+++
title = "Give ChatGPT Access to LinkedIn: LLM Agent Custom Tools"
description = "Learn how to give access ChatGPT access to LinkedIn using LangChain's custom tool feature and Bright data."
date = 2023-07-19
+++

## Introduction

### LLM agent

An agent is a LLM (large language model) like ChatGTP, with access to
a suite of tool giving him superpowers. The agent will have a
conversation with himself, and knows which tools to use, in order to
solve the problem it’s given.

ChatGPT alone doesn’t have access to the internet. So it’s not
possible to ask him about a LinkedIn profile or any other URL.

![ChatGPT does not have access to LinkedIn](/give-chatgpt-access-to-linkedin-llm-agent-custom-tools/chatgpt.png)

But it’s possible to create an agent, based on ChatGPT, and give him a
tool to scrape a LinkedIn page.

### LangChain

LangChain is a very helpful framework when developing applications
powered by LLM. Building agent is one of the most interesting feature
of LangChain.

To know more about LangChain, go to [https://python.langchain.com/docs/get_started/introduction.html](https://python.langchain.com/docs/get_started/introduction.html).

### BrightData

Scraping LinkedIn is not trivial. But it can be done without pain or
infrastructure with a BrightData web Scrapper.

To know more about BrightData,
[click here](https://get.brightdata.com/0hknxvdyx9nw).

## Creating The Scrapper

All you need to do is go on
[BrightData](https://get.brightdata.com/0hknxvdyx9nw), and
create a new web scrapper.

The code is simple. It just tells the scrapper to navigate to the
LinkedIn URL, parse the HTML to extract the important infos, and
return the data.

BrightData will provide an API endpoint to call to trigger the
scrapper. The scrapper will then respond by webhook.

As input for the scrapper, define 2 parameters

* `id` which is a unique ID for each API call to BrightData.
  It’s useful to map each webhook response to the corresponding API
  call on our side.
* `linkedinUrl` which is the URL you want to scrape on LinkedIn

```js
// go to linkedin
navigate(input.linkedinUrl);
// call the parser
let data = parse();
// build the response
collect({
	url: new URL(location.href),
	content: data.content,
	id: input.id,
});
```

The content of `parse` is pretty simple. Personally, I read only
the JSON object provided by LinkedIn inside this script tag, because
it’s simpler to extract from the HTML page and contains all the needed
infos.

```js
return {
	content: $('script[type="application/ld+json"]').html(),
};
```

The API endpoint URL to call the scrapper can be found in the tab
**Initiate by API**. Finally, click on the tab **more**, then
**delivery preference** to configure the webhook.

## Building the Agent

### LinkedIn Custom Tool

A custom tool is basically a function, taking parameters and returning
a string. Given the naming, the parameter types, and a small
description, the AI knows what to expect from the tool and how to call
it.

The only tricky thing to think of is how to manage the webhook
response. Making a simple API call to BrightData and return the
response does not work.

Instead, the idea is to

* Make an API call to BrightData in the tool
* Wait for the corresponding webhook
* Transfer the data to the right tool execution
* Resume the execution of the tool

This is for the steps 2 and 3 that an id is required. Instead of using
an id, it would have been possible to use the LinkedIn URL, but the id
solution is a more standard and generic solution.

To keep things simple for this article, I used Python’s future from
asyncio library. But in a production environment, it’s not the best
choice.

```python
import requests
import asyncio
import uuid

endpoint = "<BRIGHT DATA SCRAPER ENDPOINT>"
apiToken = "<BRIGHT DATA API TOKEN>"
headers = {
    'Authorization': 'Bearer ' + apiToken,
    "Content-Type": "application/json",
}

webhook_futures = {}

async def scrap_linkedin(url: str):
    future_id = str(uuid.uuid4())

    data = {'linkedinUrl': url, 'id': future_id}

    future = asyncio.Future()
    webhook_futures[future_id] = future

    # call bright data scraper
    requests.post(url=endpoint, json=data, headers=headers)

    # wait for the future to be resolved by the webhook handler
    content = await future

    # remove the future from the dict
    del webhook_futures[future_id]

    return content
```

### Handling the Webhook

The API is built with aiohttp. The idea is to listen to the webhook
sent by BrightData, retrieve the corresponding future by his id, and
resolve this future with the scraped data.

```py
from aiohttp import web

async def bright_data_webhooks_handler(request):
    data = await request.json()

    # retrieve the future by his id
    future = webhook_futures.get(data[0]['input']['id'])

    if future is not None:
        # Resolve the future with the webhook data
        future.set_result(data[0]['content'])

    return web.Response(status=200)

app = web.Application()
app.router.add_post('/webhooks/bright-data',
                    bright_data_webhooks_handler)

if __name__ == "__main__":
    # run server port 8080
    web.run_app(app)
```

### The Agent

```py
openai_api_key = "<YOUR OPEN AI API KEY>"

llm = ChatOpenAI(temperature=0, openai_api_key=openai_api_key, model="gpt-3.5-turbo")
tools = [Tool.from_function(
    func=scrap_linkedin,
    coroutine=scrap_linkedin,
    name="linkedin",
    description="Useful to get infos about a linkedin url."
)]
agent = initialize_agent(tools, llm, agent=AgentType.OPENAI_MULTI_FUNCTIONS, verbose=True)
```

## Use the Agent

Simply call arun (for asynchronous run, as the scraper is an async
function).

```py
prompt = "Tell me the current job of this person https://www.linkedin.com/in/antoine-prudhomme"
anwser = await agent.arun(prompt)
```

And there is the response from the agent

```plaintext
The current job of Antoine Prudhomme is a Software Engineer at Cargo.
He has been working at Cargo since January 2023. At Cargo, Antoine
works on top of AWS Redshift and Google BigQuery, implements
integrations with Salesforces, Pipedrive, and Outreach, and improves
his AWS skills and knowledge. He also works on infrastructure (AWS VPC
+ EKS), Qovery, Kubernetes, and security topics. You can find more
information about Antoine on his LinkedIn profile: [Antoine Prudhomme
LinkedIn](https://www.linkedin.com/in/antoine-prudhomme)
```

Much better than the ChatGPT screenshot of the introduction right ?
And yet, it’s the same LLM behind 🙂.

Of course, it’s possible to create custom tools for everything you can
imagine.

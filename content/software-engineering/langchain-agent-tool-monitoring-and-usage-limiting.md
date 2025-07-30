+++
title = "Langchain Agent tool monitoring and usage limiting"
description = "Learn how monitor a Langchain agent usage with Datadog and limit the usage of his tools."
date = 2023-08-31
+++

LLM agents are powerful.

Given an Open AI API key and some tools, the agent can solve complex
problems by itself like a grown man.

But unlike when we the code by ourselves, we don’t know what the agent
will do:

- How many calls to the LLM is he going to do ?
- Which tool is he going to use ?

Maybe some tools are using an expensive API, and to limit the cost, we
want to limit the agent from using those too much.

For example, the user might have a limited usage of a LLM tool
depending of the pricing model he choose. Once his rate limit is
reached, we don’t wan’t the agent to use the tool for this user
anymore.

Let’s see how we can do this.

## Monitoring the agent usage

The first step is to monitor the agent usage.

### LLM usage

When creating a new instance of a model with Langchain, the OpenAI API
key that the agent will use must be specified.

```python
llm = ChatOpenAI(
    temperature=0,
    openai_api_key="<OPEN AI API KEY>",
    model="gpt-3.5-turbo"
)
```

Each API key [has a rate limit](https://platform.openai.com/docs/guides/rate-limits/what-are-the-rate-limits-for-our-api).

The rate limit is not as easy as the APIs we know to manage, because
unlike these APIs, we don’t know how many calls will be made by the
agent. One call to the agent might consume 2, 3, 4 or more calls to
the LLM. We can’t know it in advance. Asking the agent to solve
several prompts in parallel might lead to rate limit errors.

To know how many requests and tokens have been used to solve a prompt,
we can use `get_openai_callback`

```python
with get_openai_callback() as cb:
    response = llm.run(prompt)
    print(f"Requests: {cb.successful_requests}")
    print(f"Tokens: {cb.total_tokens}")
```

### Tool usage

Say there is a tool that the agent can use to get infos about a
LinkedIn profile, given the LinkedIn profile URL.

Here is how this tool could be implemented:

```python
class LinkedinScraper():

def __init__(self):
    self._params = {}

def build(self):
    async def scrap_linkedin(url: str):
        print("Linkedin scraping tool used")

        # ...
        # scrap the linkedin page
        # ...

        return "<linkedin page summary>"

    return scrap_linkedin

scrap_linkedin = LinkedinScraper().build()

tools = []

# ...
# maybe some other tools
# ...

tools.append(
    Tool.from_function(
        func=scrap_linkedin_person,
        coroutine=scrap_linkedin_person,
        name="linkedin_scraper",
        description="Useful to get infos about a linkedin profile."
    ),
)

agent = initialize_agent(tools, llm, agent=AgentType.OPENAI_MULTI_FUNCTIONS)
```

Each time the agent will use the tool, `scrap_linkedin` will be
called.

The tool is just a function returning a string. But we are using a
class as a builder to give the function access to some values.

## Sending metrics to an observability platform

To get a clear overview of the usage, we can use a platform like
Datadog or Grafana.

Logs are not made to track usage. Instead, we should send metrics to
one of this platform, and create a timeseries to vizualise the usage
over time.

Here is an example of a metrics tracker, sending metrics to Datadog.

```python
import os

from datadog import initialize, statsd
from dotenv import load_dotenv

load_dotenv()


class MetricsTracker:
    def __init__(self):
        options = {
            'statsd_host': os.getenv('STATSD_HOST'),
            'statsd_port': os.getenv('STATSD_PORT')
        }
        initialize(**options)

    def increment(self, label, nb_occurences=1):
            statsd.increment("llm_agent." + label, value=nb_occurences)
```

We can replace the `print` in the `scrap_linkedin` function
with the following code to increment the metric each time the tool is
used by the agent.

```python
metrics_tracker = MetricsTracker()
metrics_tracker.increment(label="tool.called.linkedin_scrapper")
```

## Limiting the usage of the tool

Now that we know the usage of the tool, maybe we want to limit the
agent from using it.

We can imagine different kind of limits. Let’s see how to implement a
user based daily rate limiting.

The idea is actually pretty simple:

- When creating new instance of the tool, we pass the user id as
  parameter
- At the beginning of the tool, we verify that the user rate
  limit has not been reached
  - if it has been reached, then we return an error
  - if not, we continue

It’s important to note here that we don’t throw an error from the
tool. Instead, we return a string with a message saying the tool can’t
be used to get the information about this profile.

By doing this, the agent will not fail, and might try another solution
to find infos about this profile. For example, if the agent has
another tool to search on Google, maybe he will use it.

Here is a simple function to verify if the user has reached the rate
limit or not.

```python
import os

from redis import Redis
from dotenv import load_dotenv

load_dotenv()

redis = Redis(host=os.getenv('REDIS_HOST'), port=int(os.getenv('REDIS_PORT')),
          password=os.getenv('REDIS_PASSWORD'), decode_responses=True)

LINKEDIN_RATE_LIMIT = 200

def get_is_linkedin_scrapper_request_authorized(user_id):
    key = "linkedin_" + user_id

    count = int(redis.get(key)) if redis.exists(key) else 0

    is_rate_limit_reached = count >= LINKEDIN_RATE_LIMIT
    if is_rate_limit_reached:
        return False

    redis.set(key, count + 1)

    return True
```

With a cron job running every day to clean the Redis database, this is
a simple way to implement a daily based user rate limiting.

Finally, we can use this function in the tool.

```python
class LinkedinScraper():
    user_id: str = ""

    def __init__(self, user_id: str):
        self._params = {}
        self.user_id = user_id

    def build(self):
        async def scrap_linkedin(url: str):
            is_authorized = get_is_linkedin_scrapper_request_authorized(self.user_id)

            if not is_authorized:
                return "Not authorized to get infos on this profile."

            # scrap the linkedin page

            return "<linkedin page summary>"

        return scrap_linkedin
```

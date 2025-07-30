+++
title = "The Hard Limits of Vibe Coding (That No Model Will Ever Fix)"
description = "Vibe coding’s biggest constraints come from how it works, not from the models performances."
date = 2025-04-25

+++

## Introduction

I started using ChatGPT-3 as a coding assistant in early 2023.

By the end of 2024, I switched from VSCode to Cursor. With Cursor, it’s much easier to share context with the LLM — and the tab autocomplete feature is a game changer.

A few weeks ago, I started using the agent mode in Cursor, trying to fully embrace the vibe.

People are going crazy about vibe coding. Everyone’s talking about it.

Interviewers now ask more about AI than actual coding. Some even expect you to stop writing code entirely — just give context to the agent and write prompts.

But there’s a lot of debate in the software engineering world about vibe coding.

AI is a game changer — no doubt about that. It already changed the way we build software.
But it also has limits. And no matter how much better new models will get, these limits will always be there.

That’s what this article is about.

## **« Les mots sont source de malentendus. »**

***(Le Petit Prince - Antoine de Saint-Exupéry)***

> In English: **“Words are a source of misunderstandings.”**
> 

When you read a book after watching the movie, you see the story through the director’s interpretation. 
But when you read the book first, you create your own version in your head. Then, watching the movie becomes a comparison exercise.

That’s the beauty of natural language: it allows for nuance, ambiguity, and multiple meanings. It makes you think. Two people can read the same sentence and interpret it in completely different ways.

Programming languages are the opposite. They’re built for one clear interpretation — and that’s intentional:

- Software has to do exactly what it’s told. It follows strict specifications, often enforced by automated tests.
- Computers are dumb. It’s all binary logic. Your code can’t afford ambiguity — CPUs don’t have human brains capacities.

And yet, **even in a language with no ambiguity, writing software is hard**.

Now, people are getting excited about vibe coding.

Vibe coding is basically using natural language to ask an AI to write code for you.

Put more formally:

```
[Multiple interpretations] × [Statistics] → [Precise logic]
```

## The Limits of Vibe Coding

With just a few lines of prompts, an agent can build software that might’ve taken you hours or even days to write.
People are amazed by this, and for good reason: it's impressive to see so much code generated automatically.

But now think about the output the AI gives you.

**Is it what you expected?**

If you didn’t have a super clear expectation (like *“Just build me a simple version of a social network, something like X or Threads”*), you’ll probably be fine with the result.

But when you *do* have a clear vision, you might feel a bit disappointed. It’s close, but it’s not exactly right. So, you either have to iterate with the AI, or give it more context.

I don’t know about you, but personally:

Even when I ask an agent to do something very specific, with a clearly scoped prompt and details, it does things I didn’t ask for.

Sometimes it anticipates future features or needs. This sounds smart, but it’s just confusing.

And that makes sense. Because we’re prompting in **natural language**. 

```
[Multiple interpretations] × [Statistics] → [Precise logic]
```

The more generic and vague your input (context, prompt), the fuzzier and less precise the output will be.

The more specific your input, the more accurate the output becomes.

There are solutions to improve that precision. Write more documentation, give more context, use MPC. For existing software, clean up your codebase, use better naming, follow simple and obvious code pattern.

Now push this at the maximum: **What’s the most reliable way to get exactly what you want?**

You write the code yourself.

Because a programming language has **only one interpretation** — and that means it’ll do exactly what you say.

Even if you give a lot of context, natural language prompts are still going to be interpreted and by an AI.

## Are these limits a problem ?

Just because there are limits doesn’t mean it’s bad.

If you’re okay getting 60–70% of what you wanted (maybe 80–90% if you’re really good at prompting), then vibe coding is great.

And for a lot of cases, that’s totally fine:
- If you only have a vague idea and want to iterate on it
- If you’re building something basic or generic

But it’s not the kind of software you’re gonna sell in the long term.

Say you want a super specific UI.  
The more detailed your prompt gets, the closer it starts to look like CSS.
At that point, just write the CSS.

Recently I asked an agent to write a SQL query on really large tables.  
The first version was really inefficient. So I gave it more info: the number of rows and performance issues we were facing.  

It improved the query itself, but created an index just for that one query, and the query was hard to read.  

After a few iterations, I gave up and just wrote the correct query by myself. I had to go into so much detail that it wasn’t worth using the AI anymore.

This is the kind of situation where a “90% good” solution just isn’t acceptable.

Vibe coding is great for some things — it really depends how much control you need. You just have to accept that part of the output will always be a bit random. Sometimes that’s okay, sometimes it’s not.

For example — I suck at building UI.

I can write an app using React, but I can’t make good design. And I don’t want to learn it, it’s not something I want to spend time on.

At some point, I need to turn the default HTML style into a sexy interface.

So I just iterate with the agent. And even if it’s not exactly what I had in mind, it’s way better than what I would’ve done (and it was much faster to do).

## The Hype Will Die, Software Engineers Will Live Long

We keep hearing things like *“Engineers won’t write code anymore”* or *“Agents make developers 100x more productive.”* Even Obama said that *“AI is better than 60 or 70% of coders.”,* that says a lot.

I recently had a conversation with the CTO of a great company during an interview.

He told me that soon, his engineers won’t be writing code anymore — they’ll just prompt the Cursor agent. And if the agent can’t do something, their job will be to write *more* context until the AI gets it right.

**So engineering becomes writing documentation and QAing the agent’s output? 😱  
God, what did we do to deserve this punishment?**

Instead of spending 20 minutes thinking and 4 hours coding, he expects engineers to spend 4 hours thinking, and 20 minutes vibe coding.  
So... same amount of time, but instead of building hard skills and having fun coding, you just write doc and prompts.  
And slowly forget how to code.

I think that’s nonsense.

Vibe coding is impressive at first. Your initial prompt builds a lot, quickly. That’s where the *“wow effect”* comes from.

But have you tried vibe coding inside a real codebase, with years of work behind it?  

The agent’s performance drops fast. And the amount of context needed to make it perform well is huge.

Actually, most of the hype is coming from **business and product people** - people who want to ship fast and sell quickly their ideas.

But behind them are real engineers trying to keep the ship afloat as the software grows.

These folks don’t *see* the complexity behind building and scaling software. This is something engineers notice when:

- We fight to get some time to tackle the technical debt slowing the team.
- We have a hard time explaining and justifying why a feature that appears to be just a small change has actually a deep impact in the backend.

When a PM pastes some specs into Cursor and gets something close to what they want, they get very excited and think his engineerings can now build things 10x faster.

But real engineers aren’t that hyped.

The hype will fade when people will realize you can’t fully rely on agents.

That said, agents *will* be part of modern software development — but as **assistants**, not replacements.

The best example for me is Cursor’s tab autocomplete.  
It keeps you in control, increase your efficiency by making scoped suggestions based on local context.

That’s the future: AI that helps you code — not code *for* you.

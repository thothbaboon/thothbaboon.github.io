+++
title = "How to Properly Break the Production"
description = "Most software engineers are capable of breaking the production. But are you able to do it like a pro?"
date = 2025-02-27
+++

Most software engineers are capable of breaking the production. But are you able to do it like a pro?

## Don't Do This

### Some Context

I joined an early-stage startup founded by two former colleagues as a founding engineer. At the time, we had a few customers who were barely using our app.  

I was working on a feature, and although I don't remember the exact reason, I needed to implement an adapter pattern to hide some messy code behind a clean interface.  

The idea was to create a sexier interface while keeping the existing code as it was.  
* The existing code was doing his job, so there was no need to change it, even though it was very messy.  
* The code had no test, making it easy to introduce bugs.  
* It saved time. Focusing on the right tasks is critical in an early-stage startup.  
* But most importantly, it avoided a complex and tricky data migration. Basically, the migration involved JSON columns in a PostgreSQL table with an ill-defined schema.  

### BOOM!! 💥

I was pleased with this solution, and the feature worked as expected.

However, my collegue was not happy with the adapter pattern. He was an excellent data engineer and incredibly good at quickly shipping features, but he wasn't a strong software engineer.

He insisted that the entities in the code must directly reflect the database tables I don't think I need to explain why this made no sense, so I'm not going to.

I told him the reasons why I was against this migration
* It was a waste of time
* It was too risky

But in the end, he had the final say and insisted on the migration.

So, I went ahead with it, and as you might expect as I took this as an example for this article, it broke production.

Basically, I missed a few edge cases in the JSON schema, that my tests didn't helped me detect. As mentioned earlier, this wasn't an easy migration.

### No Harm Done

Fortunately, I only broke a part of the app, not the entire thing.

I had made a backup of the table before starting the migration, so I was able to restore the data, fix the issues, and complete the migration properly.

In the end, we didn't lose any data, and the customers didn't even notice that the feature had been broken for a while.
When I realized production was broken, I immediately informed my CEO and asked for his help in debugging and fixing the issue.

I expected him to jump on a call with me to work through it together. Instead, he simply sent a message saying it was unacceptable to break production - and that was it.

He remained upset with me for the rest of the day.

We had a team dinner at a restaurant planned for this evening. Our third colleague made a little joke about the production incident we had during the day. My CEO's face turned red, and he started yelling at me.

At that moment, I knew I was going to leave the company. It wasn't the kind of atmosphere I wanted to work in, especially since I had worked at another company where something like this would never have happened.

## The Right Way

### Context

I worked several years in a start up in Finance, that became a Unicorn. When I joined, the company had fewer than 40 employees, and by the time I left, it had grown to over 500.

Despite having thousands of customers, we still experienced production issues a few times a year.

### How We Handled Issues in Production

Usually, everything started with a message on Slack - either an automatic alert from the monitoring system or a message from someone who discovered the problem.

From there, a group of people would quickly jump on a call to address it together. When production was down, it became the top priority for the entire team to resolve it. However, it wasn't necessary for all engineers to be involved.

Once the issue was resolved, we tried to understand what went wrong.

Then, we wrote a postmortem that detailed the problem, the steps taken to fix it, how it could have been prevented, and any suggested improvements to the system.

Basically, **we learned from our mistakes and grew as a team**.

We might have teased the developer responsible a bit over a beer after work, but then we moved on.

## Key Takeaways

The first situation occurred in an early-stage startup. Although no customer noticed the issue, my CEO's reaction (along with similar reactions over the previous months) ultimately led me to leave the company.

The other situation took place in a scaling finance company, with thousands of customers.

Despite breaking production several times, we
* Became a unicorn.
* Didn't have anyone leave the company due to a production issue.
* People grew a lot within the company

Obviously, production should not be broken. But let's be real - it's going to happen. The key is to respond like a pro when it does.

**When it's down, it becomes the team's top priority to resolve the issue**.

After the fire is out, it's crucial to gather and learn from our mistakes so it doesn't happen again.

Pointing fingers at the responsible engineer is useless. They're already aware of their mistake - and might even be considering a career in farming. Chances are, they won't make the same mistake twice!
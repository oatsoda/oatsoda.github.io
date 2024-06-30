---
title: The Pinch of Software Agility
date: 2024-06-30 14:23:03
tags:
  - Agile
  - Productivity
  - Process
---

**Is there a simpler way to explain how software development teams could, and should, be practising greater agility in their day to day work?**

It can be hard to explain to development teams how they could be more productive. There are great articles such as Dave Thomas’s [Agile is Dead (Long Live Agility)](https://pragdave.me/thoughts/active/2014-03-04-time-to-kill-agile.html), and [The Agile Fluency Model](https://martinfowler.com/articles/agileFluency.html) but they aren’t always easily absorbed and applied when a development team is already set in a way of working. A more succinct visualisation might help to explain what a better working process might look like.

## Left Shifting

I like using the term “left shift” as a really simple way to convey to a developer how they should try to avoid unnecessary waste when implementing a new change to their product. The idea being that whilst you are starting implementation, as early as possible you should be engaging with users/stakeholders, considering quality and avoiding creating any downstream dependencies (e.g. siloed implementers passing partially complete work onwards to be completed) because the earlier you can detect problems the less it costs your team to alter course. It only dawned on me the other day that, of course, left shifting only makes sense up to the point of _starting the implementation_. That’s because if you left-shifted beyond that point, into the period _before_ starting, you are pushing towards a waterfall process: that is, you’re then asking for decisions to be made much earlier, which is not what you want when trying to be agile!

For the period _before_ starting you in fact want to do the opposite to left shifting. For agility you want to be deferring making decisions. This is so that you avoid the cost of making wrong decisions, which is easy to do at this stage because you are largely guessing.

## Diagram

This is where I had the idea that this could be described as a “pinch”: before implementation you want to right shift to delay decision making; when starting implementing you want to left shift to ensure failures happen as quickly as possible. Here is my proposed diagram and terminology and my explanations. My hope is that it could aid educating development and product teams and also wider organisation members into the benefits of being leaner and more agile.

!["Diagram: The Pinch of software agility"](the-pinch-of-software-agility.png)

I hope that it is pretty easy to understand; this was largely the point of creating the diagram! The “pinch” in the middle demonstrates the ethos with which you approach work and avoid wastage. Subsequently, the diagram then naturally has two time periods which demonstrate that to gain more agility you want two things:

1. To be able to start something quickly after the point of feedback/idea/market opportunity.
2. To be able to implement something (or throw it away) quickly after starting it.

Because both of these mean you have the agility to be able to react quickly to feedback, product ideas and market opportunities.

## Time to Start

Firstly, it’s worth pointing out that this time period isn’t just a waiting period. This period is where your team does its product work: filtering feedback, understanding potential value and opportunities. The highest value opportunities become the next item to work on and some items are thrown away. But these items should be framed as user problems that need solving, not features or pieces of work. The team should gain consent, not consensus (a phrase coined by [Allen Holub](https://holub.com/)) that they agree it is something worth doing.

DO ✅

- Have all of your team engaged and aware of feedback and upcoming potential work.
- Break problems down into smaller ones where possible.
- Throw away as much as possible that isn’t of value to your users.
- Collect and record as much feedback a possible.

DON’T ❌

- Make any decisions about how to implement.
- Create long backlogs of items.
- Require consensus of the whole team for something to be considered for starting.
- Assume that an item will create a lot of work.

## Time to Implement

In a lean agile environment, work started does not necessarily mean a commitment to finish it. Also, the work could be an experiment for testing a hypothesis (see [Hypothesis Driven Development](https://www.thoughtworks.com/en-gb/insights/articles/how-implement-hypothesis-driven-development)). Understanding this frees up the Time to Start period from having too much pre-definition. When thinking about this time period, it also helps not to think about it as “completing a user story”; it is better to think about it as: time to get something done which will start to give more feedback. This is where more agility can be gained because you’re then able to adjust and react to that. Your team might decide to abandon or adjust the approach based upon the feedback gained and you haven’t wasted time completing the user story.

Note: This Time to Implement can roughly be aligned with the [DORA metric of Lead time for changes](https://cloud.google.com/blog/products/devops-sre/using-the-four-keys-to-measure-your-devops-performance), which is the average time it takes a commit to get in to production (and note, if you’re not doing Trunk Based Development, the commit is the original commit, not a merge or PR commit into the main trunk), though in the diagram the point of release really means a change is in the users hands which could take one or more sequential changes to have been deployed into production before releasing.

DO ✅

- Talk to users/stakeholders before and during development.
- Engage QA / quality before and during development (depending on how your team integrates quality).
- Think about how to measure the success of your changes (not all feedback must be verbal), i.e. DevOps.
- Try to release the smallest thing to start getting feedback.

DON’T ❌

- Expect to know any details of how to implement before starting.
- Batch up changes to be released.
- Assume the whole depth of solution needs completing immediately.

## Summary

I hope that this diagram and article will be useful for explaining both why having more agility is beneficial and also how, at a higher level, this can be achieved. What this article does not discuss is the more technical details of the implementation and how this can be optimised, using techniques such as Trunk Based Development, Software Teaming (Mob Programming), TDD, DevOps, Continuous Delivery and Trunk Based Development (or CI).

---
layout:     post
title:      "My Public Surrender to AI"
subtitle:   "Reflections From My First Week Building With Agentic AI"
date:       2026-06-06
author:     "Josh Dean"
url:        "/categories/technology/agentic-ai"
categories: ["technology", "projects"]
tags:       ["ai", "agentic-ai", "llm", "software-development", "kiro"]
description: "Reflections on my first week building software with agentic AI, and why it changed my perspective on the future of development."
image:      "/img/agentic-ai/agentic-ai.png"
---
> "I am offering you a choice. Bend the knee, and join me. Or refuse, and die." - Daenerys Targaryen

<!-- ## Blog Updates -->
First off, apologies for my absence from the blog. I get very caught up during the semester and was also taking a Swedish class which soaked up much of my free time. I have some fun things planned to cook this summer and I will try to keep the blog updated with the status of my internship.

## Introduction
Truth be told, I have been hiding under a rock the last two years when it came to adopting AI. 

With limited professional experience, I believe I inherited a very academic viewpoint on AI: a growing fear of workforce redundancy. In a school setting, we are more or less shamed by using AI. Usage is marked as plagiarism, a lack of originality and an overall reduction in the quality of your learning. To be fair, this all might be true, and the proper balance in universities is still going through uncomfortable growing pains, but where it is completely inaccurate is in the beautiful, throughput focused world of capitalism.

I am back in the work force after a brief reprieve, and after being teased by trending LinkedIn posts about the state of AI in the workplace, I find myself working at the cutting edge of AI integration into modern workflows: agentic AI.

For any skeptics out there, let me tell you I completely understand. When I first heard the concept of "prompt engineering" it sounded like just another gimmick to push AI products. But here we are two years later and its the hottest topic in tech. Perhaps its time to check it out?

## Agentic AI Overview
I, like I think most people, had exclusively used chatbot forms of AI up to this point. In laymans terms, this is when you prompt ChatGPT or some other LLM with a question, read its response, and apply it to your own work. 

For me there was definitely an element of comfort in this. I could scan the output to make sure it was consistent with my own understanding of the context, then have final say about what actually was added to the project.

Agentic AI, however, is a complete let go of control and leap of faith with the defining technology of my generation. 

The "agents" are essentially an abstraction layer on top of a text editor (IDE for tech people), which connects via an API to a user defined LLM model where it receives its instructions and can immediately apply them to your project. From a user standpoint, this means I query the agent (we are using Amazon's Kiro tool) from the command line, i.e. `> update the project to use a centralized error logging mechanism`, it sends the request to the model, the model reasons about it for a bit, and then sends a list of actionable items for the agent to perform the task. If there are any changes to be made, or permission escalated tasks like internet access, emailing, file deletion, etc, it must ask for user permission/additional input before proceeding.

I found I accepted the proposed changes probably 90% of the time, or if the direction was clearly wrong, quickly cancelled the task and recontextualized it.

The key to this whole process, and perhaps what leaves me with some hope of the future of software development jobs, is ample planning ahead of time between humans and AI, to clearly define the intent of the project in some re-referencable documentation. Evaluations of agentic AI suggest that maintaining context across complex, layered tasks remains a significant challenge[^1]:

![Loss of Context](/img/agentic-ai/long-term-performance.png)

That is not to say upper-tier models will not eventually get there, but similar to how Moore's law has always controlled the contraints of modern computation, this is certainly a challenge agentic workflows will always have to consider. As it is, however, it is still key for a human to be in the driver's seat and understand the entire scope of the problem at hand. This way we can always restart the session with a fresh context, or redirect the model when it falls off track.

## Week One of Building
As some of you readers know, I began my internship this week at Ericsson in their AI Native R&D sector. By "AI-native," we mean that AI is treated as a core component of the solution rather than an auxiliary tool. and utilizes it as a key decision making component of the workflow. 

After receiving my project description and familiarizing myself with the copious coffee stations, my team of two other interns and I immediately began planning out the first week's development sprint. It took about three days to come up with an agreeable project description, which was originally drafted by us on a white board and formalized by AI. From here it only took us _*two*_ days to make an MVP that fits the specifications. While it is not tested end-to-end yet, the rate of seemingly quality development is simply absurd. 

This is a project that would've taken me several weeks to develop from scratch. And if it doesn't work as indended, the cost to iterate, update, and refactor is extremely cheap -- a fact that will completely change how thoroughly the preplanning implementation details of software projects becomes.

As someone who has invested a lot of time into education about building software, this is admittedly scary. The job has more or less changed before I got a chance to really try it, which has its disappointments. There will still be some niche areas in embedded devlopment that are probably more resistant to AI adoption, but it certainly seems like the field will experience a complete overhaul in the next 3-5 years, if not sooner. I do think having a background in coding is still a critical element to whatever role finalizes, as you do need to have some insight into what *_should_* be done, before it is done, or what to do if disaster strikes (I am still waiting for this scenario).

## The LLM Token Question
One unanswered question is whether or not the cost of running an agent for every single trivial task is actually sustainable. Every prompt you ask an LLM consists of character "tokens" which must be stored and processed by the model before computing the output. Agents require massive context windows to operate well, which burn through tokens at a brutal rate. Fortunately I am backed by a company to pay my AI bills now, but I would estimate it would be too expensive (~$1000/week) for the average person to use large commercial models for this purpose. There are workarounds, but most rely on open-source models with lackluster performance compared to the state of the art. The trend suggests this will eventually work itself out[^2]:

![Token Trends](/img/agentic-ai/token-trends.png)

as you can see the trend of token cost is reducing by 10x annually, which will continue as GPUs get more efficient and omnipresent, and AI providers fine-tune their processes. Alternatively, the AI providers pull the 'ole bait and switch and hike their prices once enough people are hooked. Already the top model available today, Anthropic's Claude Opus 4.8, costs 
$5 per million input tokens and $25 per million output tokens [^3]. You can imagine this goes quickly when iterating over a 20 page, 50K character document explaining a project. Optimizing token usage through AI native workflows will also be a growing area in the near future.

## Interesting Notes About Agentic Guardrails.
One misunderstanding I had about agentic development was that the LLM would be trusted to do everything by itself between prompting. Something that immediately became clear is that agents perform much, much better if they have well defined pathways to prebuilt deterministic tools (lookup MCPs if you are curious, this will be another massive area in software over the next 3 years). LLMs are wild, non-deterministic, probability-driven beasts, and the more grounded they are to something that can produce known results, the better off we will all be. Interestingly, to get them to use the tools, you sometimes have to really must make it explicit, i.e. `> you MUST use the ... provided in ... after you ...`, and sometimes you have to repeat the instruction a few times.

## Future Thoughts; My Predictions
Take all of this with a grain of salt, as it is heavily laden with the bias of a one week employee. But I believe any developer who is not already conviced of AIs coming dominance would have a difficult time maintaining that position after spending a week building with an agent.

The world that rewarded trivial problem solving is quickly dying, and anyone who wants to rise with the tide must begin to think in a bigger scope. The cost of writing code is trending towards zero, so the amount of software is going to grow exponentially. Whether that software works or not still remains to be seen.

Still for my more code-savvy readers, it is a difficult pill to swallow that I may spend more of my career writing agent specifications and `.md` files than writing `.c` files.

Whether this transformation creates more opportunities or eliminates them remains an open (and scary!) question. Either way, software development is changing faster than at any point in my lifetime. After only a week working with agentic AI, I find it difficult to imagine returning to the workflows I used even 6 months ago. 

## References
[^1]: https://arxiv.org/pdf/2604.11978
[^2]: https://www.reddit.com/r/LocalLLaMA/comments/1gpr2p4/llms_cost_is_decreasing_by_10x_each_year_for/
[^3]: https://platform.claude.com/docs/en/about-claude/pricing
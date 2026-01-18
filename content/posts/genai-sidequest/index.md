+++
title = "Sidequest: My GenAI Workflow"
date = "2025-10-19T11:31:05-05:00"
author = "Taylor Font"
authorTwitter = "" #do not include @
cover = ""
draft = true
coverCaption = ""
tags = ["ai", "vibe-coding", "vim"]
keywords = ["", ""]
description = ""
showFullContent = false
readingTime = true
hideComments = false
color = "red"
+++

Whether it's at `$dayjob`, talking with friends, or fellow developers online, it
seems like more and more people are turning to AI to more rapidly produce
software. I am a healthy skeptic of AI - I believe the AI industry often
advertises more than it can truly deliver, is highly unregulated, and the
reliance on AI to "vibe code" often leads to mountains of technical debt that is
only resolved by (you guessed it) more vibe coding.

I don't want this to turn into a manifesto, but if you're interested in some of
the lesser discussed AI topics, I'll leave a few links here:

- [Siliconversations on YouTube](https://www.youtube.com/@Siliconversations)
- [How AI uses our drinking water | BBC World Service](https://youtu.be/b0C56yqIkbk?si=_cn-f_FciT2XpEx5)
- [We Found the Hidden Costs of Data Centers | More Perfect Union](https://youtu.be/YN6BEUA4jNU?si=D4ttRlrhYDfuYocI)

I preface this post with my opinions for my readers' context. I don't find
myself frequently reaching for AI for a variety of reasons, but I have to
acknowledge AI also has its merits. It has proved itself as a useful tool in
certain workflows. I've seen a few useful examples in the wild, such as
[Mitchell Hashimoto's use of AI to build a new Ghostty feature](https://mitchellh.com/writing/non-trivial-vibing).

## Motivation

My largest personal complaint with AI in software development thus far has been
what many would likely consider a result of my own pickiness and frugality:

Popular AI use is catered to:

1. **developers who use GUIs and more mainstream WYSIWYG IDEs like VSCode,
   Cursor, and more.** _Why should I change my IDE - or worse, bounce between
   multiple IDEs - just to use AI?_

1) **subscription models that can top hundreds of dollars every month.** _Why
   should I pay hundreds of dollars per month for access to models? Can paying
   on a per-token basis prove any more economically efficient? What about local
   models?_

With my motiviating questions in hand, I started my attempt to create a
cohesive, wallet-friendly AI integrated workflow for my own personal software
development. I'll take you along for the ride in subsequent sections, and
showcase the results at the end.

## Research (aka the Boring Details)

The first phase of this journey was research. I spent a few weeks trying to
explore as many different options as I can for AI tools and models within
reason.

I was pleasantly surprised to find that many of the open-source community had
taken up the task of creating CLI tools and TUIs for AI use. I found many
options to choose from, most easily approachable (imo) were:

- [`opencode`](https://opencode.ai/)
- [`crush`](https://github.com/charmbracelet/crush)
- [`aider`](https://aider.chat/)
- [`ollama`](https://ollama.com/)
- [`copilot-cli`](https://github.com/features/copilot/cli)

I'll leave a few of my thoughts on several of these tools below - all of them
were perfectly workable and ultimately these are just my opinions when using
them in my own workflows.

### `ollama` and `opencode`

Ollama is the natural first step for most anyone tipping their toes into the AI
pool. I was able to install it, download some models, and play around with it
with little trouble. The documentation was clear and there were plenty of
resources online if I ever ran into any trouble.

The star of the show here is using local models. I played with many, and settled
on a few that worked best in my testing:

```bash
❯ ollama list
[GIN] 2025/11/28 - 17:45:32 | 200 |     133.458µs |       127.0.0.1 | HEAD     "/"
[GIN] 2025/11/28 - 17:45:32 | 200 |     1.32225ms |       127.0.0.1 | GET      "/api/tags"
NAME                        ID              SIZE      MODIFIED
deepseek-coder-v2:latest    63fb193b3a9b    8.9 GB    2 weeks ago
qwen2.5-coder:latest        dae161e27b0e    4.7 GB    1 week ago
qwen2:latest                dd314f039b9d    4.4 GB    3 weeks ago
```

I found using the local models in `ollama` directly was a bit clunky, especially
for what I was picturing for my workflow. I wanted a more agentic style
interaction with these models, and `opencode` allowed me to have a little more
flexibility there (more on that toward the end of this post when I discuss my
current workflow & configurations).

### `crush`

`crush` is, as all charmbracelet projects are, aesthetically pleasing and easy
to use. The majority of my experimentation with `crush` was again with local
models, and a brief stint with Clause Sonnet.

No complaints from me, other than it still was not in my IDE.

### Everything else

All the other tools performed pretty much as expected. They're interfaces for
chatting with models, but they don't work directly within my IDE, since they're
CLI tools.

As I hinted at, I did play around with paid models as well, using mostly free
tiers and per-token rates where I could to minimize cost. I also trialed a
GitHub Copilot subscription for some additional access to more premium models.

## Implementation

After a few weeks of trial and error, I reached a few decisions on models and
toolsets that felt right for me. I won't bore you with all the details, but
here's a quick summary:

| Tool | Decision | Pros/Cons |
| ---- | -------- | --------- |

|

![test image](/img/test.png)

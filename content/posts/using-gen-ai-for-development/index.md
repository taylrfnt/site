+++
title = "Using GenAI & Agentic Workflows for Software Development"
date = "2026-01-10T19:41:07-06:00"
author = "Taylor Font"
cover = ""
coverCaption = ""
tags = ["ai", "genai", "agentic", "development"]
keywords = ["ai", "agentic"]
description = "How I used generative AI and agentic workflows to go from idea -> release in record time."
showFullContent = false
readingTime = true
hideComments = true
color = "" #color from the theme settings
+++

## Motivation

This post is likely long overdue. AI has reshaped the software engineering world
in a way that nothing else in recent memory has. The barrier to entry for making
ideas into reality is now the price of a coffee at Starbucks. People use GenAI
tools to develop code for them, even if they have never written a line of code
in their life. Vibe-coding is admittedly a cool idea on paper. You give an AI
agent some instructions, and a few minutes later you have a functional app.
Reality is sometimes aligned with this, depending on how good your prompts are,
but you can also be victim of GIGO (garbage in, garbage out).

There's a better way to develop with AI. I personally used AI to supplement my
efforts to create one of my most recent CLI applications: `nprt` or
[nixpkgs-pr-tracker](https://github.com/thatsneat-dev/nprt).

## A brief tangent on my tools of choice

As is obvious from my other writing, I use Neovim as my IDE. I avoid GUIs and
WYSIWYG editors at all costs. I am more efficient in the terminal and using my
keyboard than I am with a mouse. So, Neovim remains a mainstay, even with AI
tools.

I have trialed most of the major players in the AI CLI market - Claude, GitHub
Copilot, local models via opencode... all great tools, but none of them worked
exactly how I wanted out of the box. They either felt clunky or couldn't get me
the results I wanted without some serious adjustments. Then I was introduced to
[Amp](https://ampcode.com/), and I have not stopped using it since for all my AI
development needs.

Amp has a CLI, just like GitHub Copilot and Claude Code. Their TUI interface
feels much more polished than others, and Amp has some great features out of the
box that other AI tools do not. My personal favorites are:

- [the Oracle](https://ampcode.com/news/oracle)
- [$10 USD in daily free credits from ad-supported sessions](https://ampcode.com/free)[^1]
- [thread handoff](https://ampcode.com/news/ask-to-handoff).

[^1]: The folks at Amp have
    [paused Amp Free for new users](https://ampcode.com/news/amp-free-is-full-for-now)
    as of February 2026.

One of my major complaints about AI TUIs is the fact that most of them do not
integrate well with Neovim. When using an AI CLI, my workflow looks something
like this:

1. Make changes in Neovim
2. Quit or switch tabs to the CLI
3. Let the AI do its work
4. Switch back to Neovim by either reloading the buffer or reopening

That's a lot of navigation, and in all of that, the CLI is not aware of where I
am in Neovim, files I am changing, etc. I have to tell it all of these things in
plain words like I am writing a post for `r/eli5`. It works, but it is certainly
not ideal.

Amp solves half of this problem with their Vim plugin,
[`amp.nvim`](https://github.com/sourcegraph/amp.nvim). This plugin handles
passing Neovim context to Amp, so it knows where I am and what changes I am
making.

One of the maintainers keeping the Neovim ecosystem alive (a la
[xkcd#2347](https://xkcd.com/2347/)) solves the other half of my gripe. Folke's
[`sidekick.nvim`](https://github.com/folke/sidekick.nvim) provides a way to get
the AI CLIs into my Neovim session via another buffer pane, meaning I never
again need to switch between Neovim and my AI TUI.[^1]

[^1]: As I have mentioned in my [Nix modules](/nix-modules) post, I use `nvf` to
    manage my Neovim configuration via Nix. You can see my packaging of
    `amp.vim` and `sidekick.nvim` in
    [my dotfiles](https://github.com/taylrfnt/dotfiles/blob/fb92f73f4a9f3d29f6b332acbc33683a28206298/modules/nvf/plugins/default.nix#L22-L43).

## The meat and potatoes

With my tangent about my development tools of choice complete, let's dig into
the main course. _How the heck did I use AI agents to build `nprt`?_

### Step 0: Inspiration (nothing -> idea)

Any project or application starts with an idea. The inspiration for `nprt` comes
from using the web-based [Nixpkgs PR tracker](https://nixpk.gs/pr-tracker.html).
I was tracking a few PRs, like
[nixpkgs#470883](https://github.com/NixOS/nixpkgs/pull/470883) and was tired of
going to the website every time I needed to track the same PR. I wanted to be
able to issue a terminal command and see PR progress through various NixOS
channels.[^2]

[^2]: For those that don't use Nix, this bit about the inspiration for this tool
    will probably not make a ton of sense. Suffice to say, I saw a web-based app
    that I used frequently, and wanted it in the terminal.

### Step 1: Design specs (idea -> paper)

At `$dayjob` I am a technology architect, so my first instinct is always to spec
out a design before I jump into any sort of coding. I started with creating a
set of high-level requirements, design specs, and relevant documentation for
context that would be handed off to my AI agents to develop our first MVP. This
file lives under
[`DESIGN.md`](https://github.com/taylrfnt/nixpkgs-pr-tracker/blob/main/DESIGN.md)
in my repo.

**Time spent in this phase: ~0.5 hours**

### Step 2: Handoff to agents (paper -> code)

The next step was to fire up Amp and let it go to work. I provided it with some
context in my prompt, pointed it to `DESIGN.md` and let it get to work.

If you're curious about the prompt I provided or the rest of my interaction with
Amp's agents,
[the full thread is available here](https://ampcode.com/threads/T-019b822b-957a-7534-8bd9-d191028692ad).

After a few minutes, I had an initial MVP available. I went through each of the
files that Amp created, then went back and forth requesting various tweaks...
adding unit tests, formatting & linting automation, versioning scripts, etc.

Once I was happy with the results, I started testing for myself like a real user
would. Invoke all the options, try and break things with weird edge cases, the
works. I found a few bugs, reported them in the Amp session, and the agents took
care of the fixes with some encouragement.

**Time spent in this phase: ~3 hours**

### Step 3: Code review (code -> release)

Once I had gone back and forth (reviewing code and tests, pressure testing the
app, etc.) with Amp for a while, I was happy with the initial MVP. It had all
the features I wanted, adhered to my design principles, and was ready for the
next step.

I always cap off my AI-supplemented development with a final session for every
developer's worst nightmare - code review with the senior engineer.

I took advantage of Amp's handoff feature here. The context window was getting
full, and I was changing directions. This was the perfect time to do so. Amp
handed off the thread with my additional prompt:

> we have all the code changes in. please prepare for a code review with the
> senior developer on the repository by consulting with the Oracle. Focus on
> code hygeiene, standards adherence, and make the readme a little more
> user-oriented - they don't need to know all the gory details of the PR status
> icons, or have a license section, for example.

With this prompt and the context from the previous session, I went through a few
rounds of refinement to the codebase, a few additional tweaks to improve UX
(like being able to pass arguments in any order), and then the work was done. I
tagged a release and published everything in GitHub.

As above, I reviewed all the code that Amp generated and made sure to test it
before making any commits to the repo.

**Time spent in this phase: ~1.5 hours**

## Step 4: Profit ???

**After just a few (5) hours and the price of two coffees (~$12) USD, I had used
AI to go from idea to release for a CLI application. Without AI, that same work
would have taken me at least double the time, if not longer.**

That said, just because things were faster does not always mean they are
inherently easier or better. In this case, careful review and intional prompting
allowed me to use AI agents to successfully develop an application from a simple
set of design specifications.

As a final debrief of my experience using AI to supplement my development, I
want to dive into one example from my work with AI that I believe demonstrates
the power of using AI as a tool to supplement your own effort, rather than
offloading all responsibility.

### Lessons Learned

I want to take a moment to point specifically to one interaction I had during
this process, which is quickly becoming one of my personal favorite points to
use as an example for teaching developers how to use AI in development.

Amp was adding hyperlink support for PRs being tracked, which is a neat feature,
but **not one I originally specified in my design file.** I
[requested it be added](https://ampcode.com/threads/T-019b822b-957a-7534-8bd9-d191028692ad#message-168-block-0)
during the initial MVP development session.

Since this was a new feature coming in that was not in my specs, my agents did
not have a definitive approach for this feature. I asked for the change, but did
not give many specifics, only the control sequence for how to implement a
hyperlink:

> can you use an OSC 8 sequence to make the text "PR \<number\>" a hyperlink to
> the PR as well?

> Done! The "PR #" text is now an OSC 8 hyperlink pointing to
> `https://github.com/NixOS/nixpkgs/pull/{number}`. The format used is:
>
> ```bash
> \033]8;;URL\033\\TEXT\033]8;;\033\\
> ```
>
> When you run `./bin/nprt --color=always 475593` in a terminal that supports
> OSC 8 (like iTerm2, Kitty, WezTerm, etc.), the "PR #475593" text will be
> clickable and open the GitHub PR page.

**Can you spot the problem?** While the agents did have the context of my design
specs and previous interactions in this session, they did not have clear
guidance from me on how to implement this feature. As a result, their output was
not well scoped. The hyperlink behavior ended up being coupled with color
support, which didn't make a lot of sense. It was caught during my review of the
generated code, and we revisited it for additional changes to better scope the
feature.

**TL;DR...**

1. The obvious: If I had not reviewed the code and was just vibe coding, I could
   have shipped a feature with mixed scopes and left users with a poor
   experience.
2. Even when an agent is equipped with the context of existing files and the
   whole repository, new requests and features are not guaranteed to be
   implemented the way you want the first time. Try and avoid requesting new
   features without either updating the design specs or crafting a far more
   detailed prompt than I did, and you'll get better results.

## Final Comments

I wanted to document my experience and workflow to hopefully assist other
developers who, like me, are skeptical of "vibe coding" and the atrophy of our
brains when we do not use the skills we have learned, or are cynical about AI
for any number of reasons.[^3]

AI can be a powerful tool in the right hands, but it isn't a silver bullet.
Treat it as a tool, not a crutch, and I think you might just be amazed the gains
you can make without the typical consequences of vibe coding.

[^3]: There's many topics in the AI conversation that are unaddressed, since it
    is an emerging technology. There's the
    [hidden cost of AI being passed to consumers](https://youtu.be/YN6BEUA4jNU?si=D4ttRlrhYDfuYocI),
    the consumption of critical resources like
    [drinking water](https://www.youtube.com/watch?v=b0C56yqIkbk), or even more
    generally, the regulations on AI development and safety.

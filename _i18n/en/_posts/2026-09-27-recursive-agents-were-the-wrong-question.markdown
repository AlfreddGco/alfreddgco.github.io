---
layout: post
title: "Recursive Agents Were the Wrong Question"
date: 2026-09-27
lang: en
description: "I set out to find the smallest system that lets agents create other agents. I found that recursion was easy, and that compartmentalization, domain choice, and UX mattered far more."
---

I spent a day chasing an idea that felt like the natural next step for agent infrastructure: recursive agents.

An agent that can create other agents. An agent that can write its own tools, register a cron job, expose a webhook, connect an MCP server, or spin up a subagent with its own permissions. A system that grows itself.

I approached it the way I approach most systems: subtractively. What is the smallest technology that could make this work?

The answer turned out to be surprisingly small. And once I saw how small it was, I realized I had been asking the wrong question.

## The Smallest Recursive System

The first instinct with recursive agents is to picture a lot of machinery: code generation, pipelines, containers, a deployment process for every new agent. That is the additive version. Every new capability becomes another thing to build, ship, and maintain.

The subtractive version starts with a different observation. An agent that creates agents should almost never write and deploy code. It should write data.

That leaves three layers, each changing at a different speed:

- **The runtime.** One generic loop: load an agent definition, call the model, check the requested tool call, run it, repeat. I write it once.
- **The capabilities.** Tools like `crm.lookup` or `slack.notify`, kept in a registry.
- **The agents.** Prompts, tools, schedules, budgets, and policies, stored as rows in a database.

When an agent "creates" another agent, it inserts a row. A cron job is a row. A webhook is a row. A permission is a row. Nothing new is deployed, which is why the creating agent has very little work to do.

The only open question was what happens when those rows need behavior, not just values. A policy like "only notify Slack when the lead score is at least 70" is logic, and logic is where data formats run out.

## Lua: Data That Can Carry Behavior

The thread that led me here started with Jamie Beach's excellent post, [“An Agent in 100 Lines of Lisp, or How My Prof Was Right—Just 25 Years Early”](https://thebeach.dev/posts/lisp-agent/). His experiment with code as data inspired me to explore Lua as a data engine for defining agents and carrying their behavior.

Lua was designed to live inside other programs. Its creators describe one of its core ideas as procedural data description: configuration is written in the language itself, so a config file can contain logic when it needs to.

An agent definition then looks like this:

```lua
return {
  name   = "lead-qualifier",
  model  = "claude-sonnet",
  tools  = { "crm.lookup", "slack.notify" },
  budget = { usd_per_day = 5, max_steps = 12 },

  before_tool = function(call, ctx)
    if call.name == "slack.notify" and ctx.score < 70 then
      return deny("only notify for score >= 70")
    end
    return allow()
  end,
}
```

Most of it is plain data. One field is a function. In Lua, functions are ordinary values, so loading this file does not run the hook. It produces a table that happens to contain a callable, and the host program receives it like any other object.

The sandbox is one line:

```lua
local agent = load(src, "=agent:lead-qualifier@v7", "t", env)
```

The `env` table becomes the only world the code can see. `deny` exists because I put it there. `os.execute` fails because `os` is simply not in the table. Nothing is blocked by a rule. The dangerous names do not exist.

From there, recursion is almost free. Give the first agent a single `write_tool` that accepts a Lua function and appends it to the list of available tools, and the system can extend itself. The same approach works for agents, schedules, and policies. That is the whole mechanism.

From a subtractive standpoint, this was already close to the answer: a small embedded language, a restricted environment, and a database.

## Lisp: Behavior That Can Be Inspected

Lua had one limitation that kept bothering me. A Lua function is a sealed box. I can call `before_tool`, but I cannot ask it what threshold it compares the score against. If agents are writing policies for other agents, I would like to read those policies as data, not just run them.

That is the layer Lisp adds. In Lisp, code is data before it runs:

```clojure
(def rule '(and (= env "prod") (> budget 100)))

(first rule)   ;=> and
(second rule)  ;=> (= env "prod")
```

The rule is a list. A program can inspect it, audit it, rewrite it, or run it, without anyone writing a parser.

Macros push this further. They receive code before it runs and return new code, which means an agent definition can check itself before it exists:

```clojure
(defmacro defagent [name & {:keys [model tools rules]}]
  (when-not (every? known-tool? tools)
    (throw (ex-info "unknown tool" {:agent name})))
  `(register! {:name  ~(str name)
               :model ~model
               :tools ~tools
               :rules '~rules}))
```

An unknown tool is rejected before the agent is registered. The rules stay available as data for auditing.

None of this requires rewriting a backend. Fennel is a Lisp that runs on Lua, so it can share the same sandbox. Other small Lisps can be embedded in Node or in native programs.

I came away with a clear progression:

- **JSON** can describe values.
- **Lua** can describe values and behavior, but behavior stays opaque.
- **Lisp** can describe behavior as something agents can open, read, and transform.

Each step gives an agent more power over its own definition. Each step also gives it more ways to break things.

## Where Recursion Stopped Mattering

At this point I had the smallest recursive system I could think of, and I noticed something uncomfortable: it was not the interesting part.

Recursion is a mechanism. A few lines of Lua and a database table make it possible. What it does not answer is whether the agents it creates are any good, whether anyone can trust them, or whether they are working on problems worth solving.

Computer science already has versions of this idea. [Metasearch](https://arxiv.org/abs/cs/0207097) searches not only for solutions, but for better ways to search. [Gödel machines](https://arxiv.org/abs/cs/0309048) go further by searching for beneficial rewrites to their own code, including the machinery doing the search. [Modern agent experiments](https://arxiv.org/abs/2505.22954) replace formal proofs with benchmarks: propose a change, test it, and let the accepted version conduct the next round. That is genuine recursion, but not magic. As George Hotz [wrote about autoresearch](https://geohot.github.io/blog/jekyll/update/2026/03/11/running-69-agents.html), “it’s not magic, it’s search.” Recursion closes the loop; the objective, evaluator, and constraints determine whether the loop produces anything useful.

The more I thought about what actually makes agents valuable in my own work, the less it had to do with agents creating agents. It came down to three things.

## Compartmentalization

The first is compartmentalization: different agents for different tasks, each with its own context.

When agents were weaker, we gave them small problems and controlled the workflow from outside with graphs, state machines, and routing logic. Models got better, and now we give them bigger problems, as long as those problems stay specific. We write less orchestration. We document the procedure and let the agent own it.

That shift makes context the limiting resource. Anthropic describes context engineering as finding the smallest possible set of high-signal tokens for a goal. It is a subtractive idea applied to attention. A subagent with its own instructions, tools, and permissions does not pollute the parent with every document it read. Only the result comes back.

One agent definition is then the natural boundary for four things at once:

- **Expertise:** its instructions and documentation.
- **Capability:** the tools it can reach.
- **Authority:** what it is allowed to do.
- **Memory:** the context it can see.

This is also how I think about trusting agents with more. Control does not disappear. It moves out of workflow code and into boundaries: what an agent can reach, what it can spend, what it can undo, and who can see what it did.

The trust I give an agent matches how reversible and visible its actions are:

- **Reads, research, drafts:** full autonomy.
- **Reversible writes, like CRM updates or internal messages:** full autonomy, logged, with one-click revert.
- **External or irreversible actions, like client emails, payments, or deletes:** an approval gate or a staged batch.

As models improve, actions move up that list without redesigning the system.

Recursion still has a place inside this framework, but a smaller one. When an agent proposes a new tool, that should not mean the tool is live. It should mean *proposed*, then *accepted*, then *ready*, with limits and a permission ceiling no higher than the agent that proposed it. Compartmentalization is what makes recursion safe, not the other way around.

## Choosing the Right Domain

The second is choosing where agents operate.

Agents are not equally useful everywhere, even when they are cheap. The test I ended up with:

- **Is the path open-ended?** Agents are strongest when the goal is clear and the steps are not.
- **Is checking cheaper than doing?** Thirty seconds of review for thirty minutes of work is the sweet spot. If trusting the output means redoing it, the agent adds work.
- **Can mistakes be caught or undone?** Drafts and internal changes are forgiving. Money and external communication are not.
- **Can the work wait?** Background work, yes. Anything a person is waiting on in real time, rarely.
- **Where does the value come from?** Research and synthesis suit agents. Relationships, negotiation, and authenticity do not.

I also found that "known path" is not an automatic reason to avoid agents. For small, non-critical automations, a documented procedure handed to a capable model is often simpler to build and maintain than a script, and it costs very little per run. The risk is that agents rarely fail loudly. They produce output with the right shape and the wrong content. A pinned model, a few checks on the facts that matter, and an alert when a job goes silent cover most of that.

Go-to-market work is my clearest example of both sides. Agents are excellent at finding leads, noticing triggers, writing research briefs, drafting posts, and triaging replies. Fully automated sending is where the pattern backfires. Inboxes are full of generated outreach, and the messages that stand out are the ones where a person clearly looked. So the agent prepares, I approve and add the line that shows I was there, and a simple rule-based sender enforces limits.

That shape generalizes well: **agents propose, deterministic systems execute, and people approve what cannot be undone.**

For a services company, I think the richest domain is account management: turning client emails, Slack threads, and meetings into tickets, status updates, and scope-change flags. It is constant, judgment-heavy, and easy to review.

## UX and Connectivity

The third is everything around the agent: how people reach it, what it can reach, and how its work stays visible.

**Connectivity.** An agent is only as useful as the systems it touches: CRMs, inboxes, calendars, MCP servers, databases, and triggers like cron jobs and webhooks. Most of the value comes from connecting an agent to the places where work already happens.

**One agent, many channels.** The same agent should be reachable through email, Slack, webhooks, and a web app. That only works with a channel-agnostic core, where each channel is a thin adapter that turns messages into one shape: a thread, a sender, content, attachments. The agent cares who is asking and what they are allowed to do, not where the message came from.

**A control plane people can trust:**

- **Visibility:** every run recorded, with its instructions, tool calls, cost, and outcome.
- **Human in the loop:** an approval inbox for the actions that deserve one.
- **Revert:** a record of every change a tool made and how to undo it. You can only reverse what you recorded, so revert is a tool-design problem before it is a UI feature.

## The Smallest Faithful System

I started the day looking for the smallest technology that lets agents create agents. I found it: a runtime loop, a database, and a small embedded language with a restricted environment. Lua is enough. Lisp is interesting when agents need to read each other's logic.

But subtractive engineering is not only about removing complexity from a mechanism. It is about finding the smallest system that stays faithful to what actually matters. For agents, recursion turned out to be the easy part. What matters is giving each agent a clear boundary, putting it in a domain where it is genuinely good, and building the connectivity and interface that let people work with it, see what it did, and undo it when needed.

That is where I am focusing next.

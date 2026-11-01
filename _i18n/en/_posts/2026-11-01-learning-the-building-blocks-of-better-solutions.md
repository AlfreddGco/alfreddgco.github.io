---
layout: post
title: "Learning the Building Blocks of Better Solutions"
date: 2026-11-01
lang: en
description: "From agent swarms to library learning: how DreamCoder and LILO learn reusable abstractions that make useful programs easier to find."
---

Last week, I opened Perplexity looking for agent-swarm experiments. I wanted to understand how people were using agents for mathematics, biochemistry, and other research problems—and how those experiments were actually implemented.

I often start these conversations with an incomplete question. I am looking for something, but I do not yet know what it looks like. The useful part is following the questions until that shape becomes clearer.

This conversation moved from swarms to diversity: how do you get several agents to explore genuinely different approaches? Then to novelty: how do you know whether another generation adds an idea rather than another variation? Eventually, I started asking about representations.

I think that was what I had been looking for.

Could a system learn better building blocks for expressing a problem, then use them to construct solutions that would otherwise require much more search?

## What the solver gets to work with

Suppose you want a system to write a program from a few examples. You provide inputs and the outputs you expect. The system must infer a procedure that connects them.

This is *inductive program synthesis*: finding a program consistent with observed behavior. Matching the examples is only the beginning. Many programs can fit a few examples, including a lookup table that teaches us nothing about the underlying rule.

The system needs a language in which to express candidates, a way to search through them, and a way to evaluate what it finds.

The language matters enormously.

Imagine constructing a drawing using only instructions to move a pen and rotate it. A polygon requires a particular arrangement of movements, angles, and repetition. Once `polygon` becomes an available operation, the solver can work with the shape directly. More elaborate drawings can become compositions of shapes rather than long sequences of pen movements.

The underlying capability may not have changed. Everything the polygon operation does was already possible.

What changed is the number of decisions needed to construct a useful solution.

This gives us three distinct ways to improve a solver:

- Spend more computation exploring candidates.
- Improve the policy that decides which candidates to explore first.
- Change the primitives from which candidates are constructed.

My initial questions were about the first two. The third became more interesting as the conversation continued.

## More proposals are not necessarily broader exploration

The detour through novelty helped expose this distinction.

A system can generate individually surprising ideas while repeatedly visiting similar directions. In [*Can LLMs Generate Novel Research Ideas?*](https://arxiv.org/abs/2409.04109), expert reviewers judged the selected AI-generated proposals as more novel than human proposals, while the researchers also identified limited diversity in the generation process.

Those findings can coexist. A collection can contain interesting ideas without continuing to expand very much as more candidates arrive.

That does not establish that the representation is the problem. Repetition might come from the prompt, the available evidence, the model, or the selection procedure.

But it raises a useful question: if additional attempts keep returning to familiar solutions, should we keep sampling—or change what the solver can compose?

Two projects gave that question a concrete form: DreamCoder and LILO.

## DreamCoder learns a language and how to use it

[DreamCoder](https://arxiv.org/abs/2006.08381) predates the current LLM-agent wave. It learns to solve tasks by constructing programs, while also learning a library of reusable program components.

Its loop has three coupled activities.

First, it attempts tasks using its current library. Successful programs provide material to learn from.

Next, it searches those programs for reusable structure. Useful fragments become library functions, which can themselves be combined into higher-level functions.

Finally, it trains a neural model to guide future program search. That training uses both previously solved tasks and synthetic tasks generated using the library.

The system therefore learns two kinds of expertise together: operations that make solutions easier to express, and guidance about when those operations are useful.

In one of its list-processing experiments, learned operations build on one another: filtering supports finding a maximum, which supports finding the nth-largest element, which supports sorting. A program that would be long in the initial language becomes a short composition in the learned one.

That coupling is important. Adding a function to a registry does not mean a solver will recognize when to use it. A useful vocabulary and the ability to search with it have to develop together.

The researchers describe this as reducing search depth through library learning and reducing effective search breadth through neural guidance. That is a more precise account than saying the system simply “gets smarter.”

## LILO makes the learned vocabulary usable by an LLM

[LILO—*Learning Interpretable Libraries by Compressing and Documenting Code*](https://arxiv.org/html/2310.19791v4)—brings LLMs into this process.

It combines LLM-guided program generation with enumerative search, which systematically constructs candidate programs. A symbolic system called Stitch extracts reusable abstractions from the resulting solutions. Another stage generates names and documentation for those abstractions.

The loop is roughly:

```text
Solve tasks with the current library
                  ↓
Extract reusable program structure
                  ↓
Name and document the resulting functions
                  ↓
Use the revised library in subsequent search
```

The division of responsibilities matters. The LLM does not merely announce a concept and declare it useful. Candidate programs are parsed, type-checked, and executed against task examples. The compression component searches for structure that can be reused across solutions.

One example from the paper is particularly clear.

In string-editing tasks, successful programs repeatedly contain an expression matching `a`, `e`, `i`, `o`, or `u`. Stitch extracts that expression into an abstraction. The documentation stage names it `vowel_regex`. Other expressions can then build on it, including operations involving consonants.

Nobody has discovered vowels for the first time. The LLM can bring prior linguistic knowledge into the process. The achievement is turning useful structure into an explicit component of the solver’s working language.

That component becomes available to subsequent program construction, including symbolic search.

There is also a revealing failure mode: supplying anonymous abstractions sometimes made the LLM worse at solving tasks. Naming and documentation helped in the string-editing and scene-reasoning evaluations, but did not improve the graphics evaluation; the authors discuss semantic errors in generated documentation.

A compact representation can still be difficult to use. An executable function has one behavior, while its name and description can suggest another.

For an LLM, the interface to an abstraction is part of whether that abstraction helps.

## Does this actually simplify the search space?

“Simplify the search space” was my initial way of describing what I wanted. It needs a qualification.

Adding library functions creates more available choices. In that sense, the space can get larger.

The benefit is that useful programs may become much shorter. Instead of finding a long arrangement of low-level operations, the solver can find a short arrangement of higher-level ones.

Whether that trade works depends on the library and the search procedure. A thousand poorly distinguished functions can make search harder. A few well-chosen functions can make previously impractical solutions reachable within the same budget.

Compression offers one way to choose them. We can ask whether introducing a function reduces the total description length of the library and the programs written with it. Counting the function’s definition matters; otherwise, wrapping every completed solution in a new name would look like progress.

Even then, compression is evidence of recurring structure, not proof of future usefulness.

A library can fit yesterday’s tasks beautifully and be unhelpful on tomorrow’s. It can also steer the solver toward familiar patterns when the new problem requires something else.

The representation is a useful bias only under the conditions where that bias earns its cost.

## What I would want to test

These projects sit within *library learning*, a branch of program synthesis concerned with learning reusable abstractions. They provide bounded demonstrations, not a general recipe for discovering the right concepts in any domain.

For an experiment of my own, I would start with explicit, executable abstractions. Learned internal representations are another interesting direction, but an inspectable library gives us a clearer way to ask what changed and why it helped.

I would want to separate three questions.

**Does the abstraction preserve the intended behavior?**  
Its implementation needs tests, including boundary cases. A name is not a behavioral specification, and fitting the original examples does not establish correctness beyond them.

**Does it help on unfamiliar tasks?**  
Historical compression is insufficient. The library should be evaluated on held-out tasks, including unfamiliar combinations of its components. LILO includes an especially useful evaluation: freezing the learned libraries and comparing them using a controlled symbolic search procedure without language guidance.

**Does the improvement justify learning and maintaining it?**  
The comparison should include the cost of extracting, testing, documenting, and selecting abstractions—not only the cost of solving tasks once the library exists. Repeated use may justify that investment. A one-off task may not.

I would also allow abstractions to be revised or removed. Accumulating functions indefinitely is not the same as accumulating capability. LILO’s compression procedure can rebuild the learned library, allowing earlier choices to be discarded.

That seems essential to the idea: the system should retain what improves future work, rather than preserve every pattern it has encountered.

## What I was looking for

I began with agents exploring scientific problems. I ended up interested in systems that can change the language in which they search for solutions.

The attraction is not simply more autonomous generation. It is the possibility that solving a task leaves behind something more useful than the answer: an operation, a decomposition, or an executable concept that helps with the next task.

That does not guarantee originality. A better library could help rediscover a known solution much faster. Nor does it remove the need for evidence, good task definitions, or external checks.

But it changes what I would measure.

Alongside asking how many candidates the system generated, I would ask what it learned to construct—and whether that learning made unfamiliar problems easier to solve.

A shorter description of yesterday’s solution is useful. The more interesting test is whether it changes what the solver can find tomorrow.

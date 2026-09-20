---
layout: post
title:  "Subtractive Systems Engineering"
date:   2026-09-20 00:00:00 -0600
---

I design systems that turn information into coordinated action, whether the system is an organization or a machine.

That sentence connects areas of my life that can otherwise look unrelated: robotics, low-level software, business operations, economics, psychology, and the work we do at FZF. The domains use different language, but many of the underlying questions are the same.

What can the system observe? Where does information lose fidelity? Who or what makes a decision? How does that decision become action? What happens when reality deviates from the plan? Which constraints are fundamental, and which ones exist only because of the way the system was assembled?

I call my approach to those questions **subtractive systems engineering**.

Its goal is not minimalism for its own sake. It is to find the smallest faithful system: one that preserves the complexity reality requires while removing the complexity it does not.

## The Structure Beneath the Domain

Every field develops its own vocabulary. Economics talks about incentives and allocation. Psychology talks about attention, motivation, and behavior. Computer science talks about state, interfaces, and computation. Control theory talks about feedback, observability, and stability.

These distinctions matter, but real systems do not respect academic boundaries.

A business process can fail because an API is unreliable, because nobody owns a decision, because the incentive rewards the wrong behavior, or because the relevant information arrives too late. A technically correct solution can still fail when it ignores any one of those layers.

Thinking in systems means looking beneath the labels for the structure that determines behavior:

- What information exists, and where?
- How does it move?
- Where does it wait, degrade, or disappear?
- What decisions depend on it?
- What incentives shape those decisions?
- What actions can the system take?
- How does the result return as feedback?

The value of a model is not that it makes the world look clean. The value is that it preserves the parts of reality that change the outcome.

This is where subtraction becomes difficult. Removing a database table is easy. Removing a concept, handoff, approval, service, or policy without damaging the system requires understanding why it exists and what behavior it is protecting.

## Organizations and Machines

A robot turns sensor data into an estimate of the world, uses that estimate to choose an action, and applies that action through physical hardware. The environment responds, new information arrives, and the loop repeats.

An organization also observes, decides, and acts. It observes through software, reports, documents, conversations, and the experience of its people. Decisions happen through policies, judgment, incentives, and authority. Action happens through people and machines working together. Results return through operational data, customer behavior, financial outcomes, and human feedback.

The analogy is useful, but it has limits. An organization is not a machine, and people are not interchangeable components. People interpret goals, adapt to rules, withhold information, create new possibilities, and sometimes reject the system entirely. The objectives themselves may be disputed.

Psychology and economics are therefore not details outside the technical system. They are part of its dynamics.

This changes what control should mean in an organizational context. It is not control over people. It is the ability to make relevant state visible, recognize when reality has departed from intent, and coordinate an appropriate response. A good operational system increases human agency by giving people better information and removing work that does not require their judgment.

## Friction Accumulates at Boundaries

Complex systems often fail between components that work correctly on their own.

The ATS contains one version of a worker. Payroll contains another. A client request lives in email. A credential is attached to a message but not associated with the workflow that needs it. A recruiter knows something the system does not. A report shows that an outcome failed, but only after the window to intervene has closed.

Each local tool can be functioning as designed while the larger operation remains unreliable.

This is why replacing software is often less valuable than understanding the boundaries between systems. The highest-leverage intervention may be a shared identifier, a clearer decision owner, an event emitted at the right moment, or a small integration that prevents information from being re-entered and reinterpreted.

The first instinct should not be to add another platform. It should be to understand the loop.

## A Subtractive Method

Subtractive systems engineering starts from the real process rather than the official one.

### 1. Map the actual flow

Follow a real unit of work from beginning to end. Identify the people, software, documents, decisions, delays, and exceptions involved. The unofficial spreadsheet and the message someone sends every Thursday are part of the system, whether or not the process diagram includes them.

### 2. Find the constraint

Not every inefficiency matters equally. Look for the point that limits throughput, creates risk, or forces the rest of the system to compensate. Improving a non-constraint may make one step look faster without improving the outcome.

### 3. Improve observability

A system cannot respond to state it cannot see. Before automating action, make sure the relevant state is available, timely, and trustworthy. Often the first useful product is not an autonomous workflow but a better representation of what is happening.

### 4. Close the loop

Connect information to a decision, the decision to an action, and the result back to the system. Define what happens when the normal path fails. A workflow without feedback is only a sequence of hopeful instructions.

### 5. Remove what no longer serves the loop

Once the system is understood, remove duplicate entry, unnecessary approvals, accidental handoffs, stale abstractions, and technology that no longer protects an important constraint. Subtraction comes last because uninformed simplicity is just hidden fragility.

## FZF as an Application

Staffing back offices make these ideas concrete.

Critical work moves across recruiters, workers, clients, ATSs, VMSs, payroll platforms, vendors, documents, and inboxes. Credentialing, onboarding, injury reporting, redeployment, and client compliance are not isolated features. They are cross-system loops with human judgment, external dependencies, deadlines, and costly exceptions.

At FZF, we begin by mapping those loops. We look for where coordination breaks down, where information becomes ambiguous, and where people spend time maintaining continuity between systems that were never designed to work together.

Then we build the smallest useful intervention. Sometimes that is an integration. Sometimes it is an operational view, a workflow, or a new source of truth. Sometimes it is a change in ownership or process that requires little software at all.

AI extends the range of work software can reach. It can interpret documents, classify messages, synthesize context, and help route exceptions that rigid automation could not handle economically. But AI is not the system. Without boundaries, feedback, and human oversight, it can become another source of uncertain work.

The objective is not to automate the most tasks. It is to improve the reliability and capacity of the whole operation.

## Why Physical AI Belongs Here

Physical AI is not currently the commercial focus of my work. It is a technical interest that sharpens the same way of thinking.

Embodied systems make abstraction errors impossible to ignore. A model can be elegant and still fail because inference is too slow, sensing is incomplete, contact behaves differently from simulation, or the controller asks the hardware to do something the physical system cannot support.

The loop has to run within real limits of time, energy, computation, uncertainty, and material behavior. That is what makes the inference and runtime layer for embodied agents so interesting to me. It is another place where the quality of the system depends on knowing what can be simplified and what must remain physically faithful.

Organizations are not robots, but both punish solutions that optimize a component while ignoring the system around it.

## A Working Definition

Subtractive systems engineering is a name for a practice I was already following before I had language for it:

> Find the structure beneath the domain. Preserve the complexity that carries meaning. Remove the complexity that does not. Build the shortest reliable path from signal to action.

It is not a finished methodology, and it is not a claim that every problem has a clean, ideal form waiting to be uncovered. Some complexity is irreducible. Some tensions can only be managed. Some systems cannot be optimized toward a single objective without losing something important.

The discipline is learning the difference.

See the system. Close the loop. Remove the rest.

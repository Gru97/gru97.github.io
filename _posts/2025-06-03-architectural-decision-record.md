---
layout: post
title: "How to write an ADR (Architectural Decision Record)?"
date: 2025-06-03
tags: [architecture]
permalink: /:title/

---


I like visualizing things in order to understand the flow and the components involved, when making a design decision. So, when I want to analyze something and find an architectural solution, I usually open a Miro board, Excalidraw or something of that nature and start sketching.    
   
<img src="https://github.com/user-attachments/assets/5c4dd23f-5f83-4b23-9e68-865149160dc0" width="1100" height="800">   
   
While we were rethinking the design of a system I was involved in, I was asked to explain my proposed design. My explanation lacked a lot. It lacked what the pain points were, the reason behind the decisions, the alternatives I passed on due to something in my head that I couldn’t remember, and how the new decisions solved the initial problems. What happened was that I went through a decision-making process, and there was no clue as to how I got to where I got other than some random single word notes on the Miro, some red capitalized alarming phrases, crosses and all! 
I was asked to turn my messy sketches into a formal ADR document, and clarify the vague parts. So here I'm going to share that experience with you.
## What is an ADR?
ADR is essentially a way to communicate and document your decisions as a software architect.   
[Here](https://github.com/joelparkerhenderson/architecture-decision-record) you can read more about ADR and the difference templates you can write it in.

I like the template that is used in this [Amazon](https://docs.aws.amazon.com/prescriptive-guidance/latest/architectural-decision-records/appendix.html) example.
It goes like:   
    
<img src="https://github.com/user-attachments/assets/826bd646-eee6-42de-964f-01c7d50a4906" width="500" height="800">   
    
Typically, one ADR (Architecture Decision Record) is written per significant decision. It’s not about documenting every detail—just notable choices that would be hard to reverse or important for context later.

Here's one of the ADRs I wrote, where I suggested replacing jobs and flipping the control flow from pull-based to push-based. I made that call based on things that aren’t really shown in the sketches—or honestly, anywhere outside my head!


## ADR: Replace Scheduled Jobs with Event-Driven Architecture for Trade Processing

**Status:** Proposed  
**Date:** 2025-06-03

## Context

The current system uses scheduled background jobs to process stock trade data in three stages:

1. `TradeEventHandler`
2. `TradeProcessing` (runs every 2 minutes)
3. `TradePosting` (runs every 1 minute)

**Observations from monitoring:**

- `TradeProcessing` has **4.3s average latency** and **0.8 tpm**.
- `TradePosting` has **196ms latency** and **8.9 tpm**.
- Jobs spend significant time idle, waiting for their scheduled run.
- Up to **80% of the job window is wasted** due to polling rather than reacting to data readiness.

This causes **throughput bottlenecks** and **poor resource utilization**.

### Decision

We will replace the current scheduled-job model with an **event-driven architecture**.  
Processing stages will be triggered by events (e.g., message queue) as soon as new data is available.

### Consequences

#### Pros

- **Increased responsiveness**: downstream steps begin immediately after the prior stage finishes.
- **Reduced latency** and **better throughput**.
- Eliminates **idle polling**, reducing resource waste.
- Enables **scalable, reactive workflows**.

#### Cons

- Increases **system complexity** (e.g., message broker setup, retry handling).
- **Harder to trace errors** across asynchronous boundaries.
- Requires **refactoring job logic** into event handlers.
- Introduces **operational overhead** in maintaining messaging infrastructure.

### Alternatives Considered

#### 1. Keep scheduled jobs, but increase frequency

- Would reduce some idle time but still relies on polling.
- Increases job execution overhead and risk of overlapping runs.

#### 2. Use database triggers or polling with change tracking

- Adds complexity to the database layer.
- Not scalable or easily observable.

### 3. Hybrid approach (event-driven for some steps, scheduled for others)

- Partial improvement with less refactor effort.
- May still retain latency issues in some parts of the process.


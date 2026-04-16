---
title: "Starting Go from C++: What I'm Trading Away"
date: 2026-04-16
draft: false
tags: ["go", "cpp", "learning", "sre"]
summary: "I'm switching my default language from C++ to Go as part of my SRE transition. Here's what I'm giving up, what I'm betting on, and the first hour's observations."
---

For the last two years, C++ has been my default language. Every LeetCode submission, every side project, every time I wanted to write "real code" — C++. It felt like the adult language. Templates, memory control, zero-cost abstractions. I liked being the guy who picked C++ when everyone else picked Python or Java.

Today I'm putting it down.

Not forever. Not in disgust. In a clear-eyed trade: I'm becoming a Site Reliability Engineer, and Go is the lingua franca of the infrastructure world. Kubernetes is Go. Prometheus is Go. Terraform is Go. Docker is Go. If I want to read the source of the tools I'll live inside every day as an SRE, I need to be fluent in Go — not tourist-level, fluent.

## What I'm giving up

**Templates and generics-as-art.** C++ templates let you do things that feel like magic. Go's generics are polite, constrained, boring. I'll miss the magic. I won't miss the compile errors.

**Manual memory control.** I never used it meaningfully. Be honest — neither did most of us. I was flexing over `std::unique_ptr` vs `std::shared_ptr` without ever measuring anything. Go has a garbage collector and I'll be fine.

**The "I'm a serious engineer because I know C++" identity.** This is the real cost. C++ was a flex. Go is a tool. Letting go of the flex is the actual switch — not the syntax.

## What I'm betting on

**Readability compounds.** Go code at hour 1 looks like Go code at year 10. C++ code at hour 1 looks nothing like C++ code at year 10. For a language I'll use for a decade of ops work, I want the floor to be close to the ceiling.

**The ecosystem is where I'm going.** Every tool I want to contribute to — kubectl, argo, prometheus operators — is in Go. If I want to submit PRs to infrastructure projects (which is a real SRE differentiator), C++ skills don't transfer.

**Concurrency is native, not tacked on.** Goroutines and channels are the model, not a library. For someone heading toward production systems that fan out requests, retry under timeouts, orchestrate jobs — this is the right shape.

## First hour observations

**Unused variables are a hard stop.** C++ will politely warn you if you declare a variable or an import and never use it. Go flat-out refuses to compile. It forces a level of hygiene that feels irritating for the first ten minutes and then deeply reassuring.

**Public/Private by capitalization.** There are no `public:` or `private:` keywords. If a function or struct name starts with a capital letter, it's exported; if lowercase, it's internal. It feels weirdly informal coming from C++ header files, but you instantly know the scope just by looking at the variable.

**The missing ternary operator.** My fingers keep trying to type `condition ? a : b`. Go specifically omitted it to force developers to write explicit `if/else` blocks. I hate how verbose it is, but I can't deny that it leaves zero room for clever, unreadable one-liners.

## The plan from here

21 days to finish the Tour of Go + Go by Example + a CLI tool called `hc` — a URL health-check utility that'll be my first real Go project. If you want to follow along, I'm publishing every real thing I ship on this blog.

By May 6th, I want to have `hc` fully functional, using goroutines to ping dozens of URLs concurrently, proving to myself that Go's concurrency model really is the infrastructure superpower everyone claims it is.

See you at Day 21.
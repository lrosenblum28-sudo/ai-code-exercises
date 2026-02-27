# Liora Findings – AI-Augmented Engineering Exercises

This document summarizes my reflections and key learnings from the AI-Augmented Engineering exercises.  
The goal was not to produce perfect code, but to understand how to think, verify, and structure solutions when working with AI.

---

# 1. API Documentation Exercise

## What was required
- Select an API endpoint
- Generate documentation using structured prompts
- Convert it into another format
- Create a developer usage guide
- Reflect on the process

## Key Learnings
1. AI generates better documentation when prompts clearly define audience and format.
2. Edge cases and error handling are the hardest parts to document properly.
3. Converting documentation formats requires understanding the intent, not just reformatting text.

## Most Challenging Part
Making sure examples matched the actual API behavior.

## How I Would Use This in Real Projects
- Generate first draft with AI
- Validate manually against the real endpoint
- Add real-world request/response examples

---

# 2. AI Solution Verification Challenge (Buggy mergeSort)

## Scenario
A sorting function had a subtle index bug in the merge logic.

## Verification Strategies Applied
- Collaborative solution verification
- Walking through execution manually
- Reviewing alternative approaches

## Key Insight
AI-generated code can look correct but fail under careful tracing.

The bug (`j++` instead of `i++`) demonstrated how small logic errors can break correctness.

## Reflection

### How did my confidence change?
Initially high after AI output.  
Much higher only after manually tracing execution.

### What required most scrutiny?
Loop conditions and index increments.

### Most valuable technique?
Step-by-step execution tracing with small test arrays.

---

# 3. Refactoring & Code Quality Exercises

## Idiomatic Code Transformation

### Key Learnings
1. Small, focused functions improve readability and testability.
2. Naming clarity matters more than I previously realized.
3. Language idioms reduce boilerplate and make intent clearer.

---

## Code Quality Detective

### Issues Identified
- Mixed responsibilities in single functions
- Repeated logic
- Poor naming
- Lack of validation separation

### 3 Key Learnings
1. Code smell detection improves with structured review prompts.
2. Maintainability is often more important than micro-performance.
3. Explicit structure reduces future bugs.

---

## Language Feature Exploration

I explored advanced features and focused on:
- When to use them
- When not to overuse them
- How they affect readability

### 3 Key Learnings
1. Advanced features improve clarity when used intentionally.
2. Overusing features reduces accessibility for other developers.
3. Understanding mental models is more important than syntax memorization.

---

# 4. Design Pattern Exercise

Patterns analyzed:
- Strategy
- Factory
- Adapter
- Repository

## Why They Matter
They reduce conditional complexity and improve scalability.

## Key Takeaway
Patterns are not about complexity — they are about replacing fragile conditionals with structured extensibility.

---

# 5. Learning a New Programming Language with AI

## Most Effective Prompting Strategy
Conceptual understanding before coding.

## Mental Model Adjustments
- Static typing requires thinking in contracts.
- Explicit validation improves reliability.
- Async behavior requires execution awareness.

## Reflection Questions

### Which strategies worked best?
Four-step prompting (concept → breakdown → implementation → verification).

### What surprised me?
How much architecture clarity comes from type systems.

### What hindered learning?
Occasional habit of thinking in my source language first.

---

# 6. FastAPI – Fundamentals & First API

## Core Concepts Learned
- Path operations
- Pydantic models
- Dependency injection
- Automatic documentation
- Response models
- Async-first design

## Design Philosophy
FastAPI treats type hints as contracts.
Validation, documentation, and serialization are derived from code.

---

# 7. Contextual FastAPI Learning

## Translation Table

Flask Blueprint → FastAPI APIRouter  
Django Serializer → Pydantic Model  
Django Middleware → FastAPI Middleware  
Authentication Decorators → Dependency Injection  

## Key Difference
Dependency injection is central in FastAPI rather than optional.

---

# 8. Documentation Navigation Exercise

## Strategy Developed
1. Identify exact feature needed.
2. Find minimal documentation section.
3. Convert concept to runnable code.
4. Refactor after understanding.

## Most Important Sections for REST APIs
- Request Body & Validation
- Dependencies
- Security
- Response Models
- Path Operation Decorators

---

# 9. Understanding Complex FastAPI Patterns

Patterns Identified:
- Repository pattern
- Service layer
- Dependency injection
- Middleware
- JWT authentication flow
- Lifespan events

## Execution Flow Understanding

Tracing a request to `/admin/users/` clarified:
- Middleware runs first
- Dependencies resolve next
- Authentication executes
- Authorization checks role
- Business logic runs
- Response model serializes output

This tracing exercise significantly improved architectural clarity.

---

# 10. Common Themes Across All Exercises

- AI is powerful but must be verified.
- Explicit structure reduces hidden bugs.
- Smaller functions improve clarity.
- Execution tracing increases confidence.
- Type hints act as living documentation.
- Separation of concerns improves maintainability.

---

# Final Reflection

The biggest improvement was not technical syntax.  
It was developing a critical mindset when working with AI.

I now:
- Question generated code
- Trace execution manually
- Separate responsibilities clearly
- Think in terms of architecture layers
- Validate assumptions before trusting output

This process strengthened both my programming understanding and my ability to use AI responsibly.

---

End of Findings.
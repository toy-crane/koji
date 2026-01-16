---
description: Interview user about plan details, then write refined spec
argument-hint: Path to plan/spec file to review
---

# Review Plan

Conduct an in-depth interview about a plan or spec file to uncover hidden requirements, edge cases, and design decisions.

**Actions**:

1. Read ${ARGUMENTS} to understand the current plan/spec
2. Interview the user using AskUserQuestion tool about:
   - **Technical implementation**: Architecture choices, data flow, state management, API design, error handling strategies, performance implications
   - **UI & UX**: User flows, edge states (loading, empty, error), accessibility, animations, responsive behavior, interaction patterns
   - **Concerns**: Security implications, scalability bottlenecks, maintenance burden, testing strategy, deployment considerations
   - **Tradeoffs**: Alternative approaches considered, why this approach was chosen, what we're sacrificing

3. **Question Guidelines**:
   - Ask NON-OBVIOUS questions - avoid questions whose answers are clearly stated in the spec
   - Probe edge cases: "What happens when X fails?" "How does this behave with 0 items vs 1000?"
   - Challenge assumptions: "Why not use Y instead?" "What if the user does Z?"
   - Explore implications: "How does this affect existing feature A?" "What's the migration path?"
   - Consider operations: "How will we monitor this?" "What metrics matter?"
   - Ask about omissions: "The spec doesn't mention X - is that intentional?"

4. **Interview Process**:
   - Ask 2-4 questions per round using AskUserQuestion
   - Continue interviewing until ALL major areas are covered
   - Don't stop after one round - be thorough
   - Track which areas have been explored to ensure completeness

5. After the interview is complete, update the spec file with:
   - Refined requirements based on interview insights
   - Edge cases discovered during questioning
   - Technical decisions and their rationale
   - Any new constraints or considerations identified

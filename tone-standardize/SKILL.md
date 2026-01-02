---
name: tone-standardize
description: Standardizes text to a consistent casual, friendly tone for multi-author book projects. Use when editing book chapters, harmonizing writing styles, or when users mention tone consistency, rewriting, or style unification.
allowed-tools: Read, Write, Edit
---

# Tone Standardization Skill

Converts text written by different authors into a unified, casual, friendly tone suitable for accessible book content.

## Target Tone Characteristics

**Casual Friendly Style:**
- Relaxed and conversational, like talking to a friend
- Use contractions (it's, you'll, we're, don't)
- Direct address to reader (you, your) when appropriate
- Simpler vocabulary - avoid jargon unless necessary
- Shorter, punchier sentences (mix of short and medium length)
- Active voice preferred over passive
- More engaging and approachable
- Occasional questions to reader to maintain engagement
- Natural transitions and flow

## Instructions

When invoked with a file path:

1. **Read the original content** using the Read tool

2. **Analyze the current tone** and identify:
   - Overly formal or academic language
   - Complex sentence structures
   - Passive voice constructions
   - Technical jargon that can be simplified
   - Stiff or impersonal phrasing
   - Inconsistent perspective (mixing I/we/you/one)

3. **Rewrite while preserving**:
   - All factual information and technical accuracy
   - Key concepts and main ideas
   - Examples and illustrations
   - Logical structure and flow
   - Section headings and organization

4. **Apply casual friendly transformations**:
   - Replace formal phrases with conversational equivalents
     - "It is important to note" → "Here's the thing"
     - "One should consider" → "You'll want to think about"
     - "In order to" → "To"
     - "Utilize" → "Use"
   - Convert passive to active voice
     - "The function was called by the system" → "The system calls the function"
   - Break long sentences into shorter ones
   - Add contractions where natural
   - Use second person (you) to engage reader
   - Add occasional rhetorical questions
   - Use simpler synonyms without losing meaning

5. **Write the standardized version** to a new file:
   - Output filename: `{original_name}_standardized.{ext}`
   - Preserve original file formatting (markdown, etc.)
   - Maintain paragraph breaks and structure

6. **Show a comparison** snippet:
   - Display a before/after sample from the beginning
   - Highlight the tone changes made

## Usage Examples

**Example 1: Basic usage**
```
User: /tone-standardize chapter3.md
```

**Example 2: With explicit path**
```
User: /tone-standardize docs/introduction.txt
```

## Tone Comparison Examples

**Before (Formal):**
> It is imperative that developers understand the fundamental principles underlying asynchronous programming. One must carefully consider the implications of callback functions and their potential to create difficult-to-debug code structures, commonly referred to as "callback hell."

**After (Casual Friendly):**
> Here's what you need to know: async programming has some key principles you'll want to understand. Callbacks are powerful, but watch out! They can create messy, hard-to-debug code. (You might've heard developers call this "callback hell.")

**Before (Academic):**
> The utilization of design patterns facilitates the development of maintainable software architectures. Software engineers should endeavor to implement these patterns judiciously, as inappropriate application may result in unnecessary complexity.

**After (Casual Friendly):**
> Design patterns make your code easier to maintain. But here's the catch: you'll want to use them wisely. Throw in patterns just for the sake of it, and you'll end up with code that's way more complicated than it needs to be.

## Quality Checks

Before completing, verify:
- [ ] All technical facts remain accurate
- [ ] Meaning hasn't changed
- [ ] Tone is consistently casual throughout
- [ ] Contractions are used naturally
- [ ] Sentences flow smoothly
- [ ] No critical information was lost
- [ ] Reader engagement is improved

## Notes

- This skill works best with prose/narrative content
- Code examples and technical specifications should remain unchanged
- If the original text is already in casual tone, minimal changes will be made
- Always preserve the author's key insights and unique examples

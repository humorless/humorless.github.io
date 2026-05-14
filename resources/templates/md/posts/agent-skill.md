{:title "Expert Clojure Workflows for AI Agents: Four Skills from Production Experience"
 :layout :post
 :date "2026-05-14"
 :tags [ "clojure", "ai-agents", "skills", "behavioral-change"]}

When you let an AI agent write Clojure code, you expect it to leverage the language's superpowers—the REPL's interactivity, structural editing, format-preserving code manipulation, and the rich ecosystem of wrapper libraries. Instead, what you typically see is mediocre code written slowly, as the agent makes the same mistakes every developer learns to avoid.

I discovered this the hard way.

## The Setup: Vibe Coding with Observations

While building [lite-crm](https://github.com/humorless/lite-crm) with Claude Code, I deliberately avoided the `--dangerously-skip-permissions` flag. Instead, I sat beside the agent and watched it work—observing its patterns, frustrations, and failures. What I saw was an agent trained on millions of codebases but ignorant of how Clojure practitioners actually think.

Three concrete problems emerged:

### Problem 1: The Wrapper Library Blind Spot

When encountering Java interop, the agent jumps straight into direct interoperability without ever asking: "Is there a Clojure wrapper library for this?"

**The result:** Uglier code, harder to maintain, and a missed opportunity for idiomatic Clojure.

### Problem 2: Formatting Brittleness

Code formatters like `cljfmt` are essential—but they create a sneaky problem. When the agent modifies source and the formatter shifts indentation by a single space, the agent's subsequent `str_replace` operations fail due to whitespace mismatch.

**The result:** I watched it fail, retry, fail again, then give up and rewrite entire files. Enormous token waste.

### Problem 3: Primitive Debugging

When a test failed, the agent fell back on the crudest debugging technique: add `println` statements, run the test, inspect output, delete the logs, restore the code. Repeat.

This is especially wasteful in a Clojure project where I've provided direct access to the REPL via the [brepl](https://github.com/licht1stein/brepl) CLI. The agent could inspect values interactively, test hypotheses instantly, and trace execution without touching source code. But it never did.

---

## The Recognition

These weren't knowledge gaps. They were **behavioral gaps**—places where the agent's default approach conflicted with Clojure expertise.

In the context of Clojure Stack Lite (which includes proper testing harness and real database, not mocks), the agent wasn't just writing suboptimal code—it was making design decisions based on unfamiliar tools.

I decided to address this not by teaching the agent more facts, but by redirecting its behavior.

---

## Four Skills to Close the Gap

The result is four skills, each targeting a specific behavioral pattern that distinguishes novice agents from expert Clojure practitioners:

### 1. **clj-debug**: From Logging to REPL Inspection

**The Problem:** Agents default to adding `println`, `tap>`, or logging statements, then running tests to inspect output.

**The Pattern:** In Clojure, this is backwards. The REPL lets you pin a value with `def`, explore its structure instantly, test hypotheses interactively—all without modifying code.

**What the skill does:** When you're about to debug, clj-debug redirects from logging patterns to REPL-based inline inspection. It teaches the agent to use `def`, `keys`, keyword access, and structural exploration—the actual workflow expert Clojure developers follow.

**Behavioral change:** From edit-test-inspect cycle to interactive REPL inspection. This is faster, non-invasive, and gives immediate feedback.

### 2. **clj-discover**: Systematic API Exploration

**The Problem:** When encountering unfamiliar Java classes or macros, agents jump to direct integration without exploring whether an idiomatic Clojure wrapper already exists.

**The Pattern:** Expert Clojure developers follow a deliberate workflow:
1. Search for a Clojure wrapper library first (usually there is one)
2. If not, inspect the Java class via reflection
3. For macros, expand them to understand what code they generate

**What the skill does:** clj-discover codifies this workflow, ensuring the agent prioritizes idiomatic libraries and systematic exploration before writing integration code.

**Behavioral change:** From direct interop to research-first integration. The result is cleaner, more maintainable code.

### 3. **clj-replace**: Format-Aware Structural Replacement

**The Problem:** Code formatters shift indentation by spaces, breaking text-based `str_replace`. The agent then wastes tokens failing repeatedly or rewriting entire files.

**The Pattern:** Clojure is homoiconic—code is data. Two S-expressions are semantically equivalent even if formatted differently. Expert editors handle this automatically via structural editing.

**What the skill does:** clj-replace compares code by structure (S-expression equivalence) rather than text, ignoring whitespace while preserving the original file's formatting style. It uses the [rewrite-clj](https://cljdoc.org/d/rewrite-clj/rewrite-clj/1.1.45/doc/user-guide) library to parse, match, and replace nodes safely.

**Behavioral change:** From brittle text matching to robust structural matching. Formatting variations become irrelevant.

### 4. **clj-refactor**: Mechanism/Policy Separation

**The Problem:** Without guidance, agents write tangled code where reusable mechanisms are mixed with business policy, creating inflexible designs that accumulate technical debt.

**The Pattern:** Arne Brasseur's [mechanism/policy separation](https://lambdaisland.com/blog/2022-03-10-mechanism-vs-policy) principle is core to building maintainable Clojure systems. Mechanism is context-free, stable, and reusable. Policy is opinionated, domain-specific, and volatile. Expert developers keep these separate.

**What the skill does:** clj-refactor scans code for opportunities to extract mechanisms from policy—functions where hard-coded values or implicit context can be made explicit, dependencies can be pushed to parameters, and reusable logic can be isolated.

**Behavioral change:** From monolithic functions to extracted, composable mechanisms. Code becomes easier to test, reuse, and reason about.

**Note:** Unlike clj-debug, clj-discover, and clj-replace—which activate automatically when the agent encounters problems—clj-refactor is **user-initiated**. You invoke it when you want the agent to analyze code for refactoring opportunities, not in response to a failure.

---

## Why This Matters

These aren't reference manuals or API documentation. They're **workflow redirects**—rules that teach AI agents to think like expert Clojure developers instead of generic code writers.

The underlying philosophy is simple: **A skill's value is measured by behavioral change, not knowledge transfer.**

When an agent uses clj-debug, it stops adding logging. When it uses clj-discover, it checks for idiomatic wrappers before raw interop. When it uses clj-replace, formatting becomes irrelevant. When you invoke clj-refactor, the agent identifies tangled mechanisms and suggests extraction. Each skill shifts the agent's default patterns closer to expert practice.

This matters because Clojure is a language of leverage. The REPL, immutability, homoiconicity, and the functional approach all reward practitioners who use them correctly. An agent that doesn't leverage these features isn't just writing slow code—it's missing the point of the language.

---

The goal is simple: your AI agent shouldn't just write Clojure code—it should think like a Clojure developer. These four skills make that possible.

Find them at: [github.com/humorless/clj-native-agent](https://github.com/humorless/clj-native-agent/)

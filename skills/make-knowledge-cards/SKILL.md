---
name: make-knowledge-cards
description: This skill should be used when the user wants to convert an article, blog post, tutorial, essay, or local Markdown/TXT file into a focused set of 5–8 knowledge cards for study or review. Each card isolates one key point and contains a title, the core knowledge, a brief explanation, and either a concrete example or a self-check question. The skill extracts only what is genuinely important from the source, removes duplicates, never invents information not present in the source, and produces fewer cards when the source lacks substance rather than padding the count.
agent_created: true
---

# Make Knowledge Cards

## Overview

Transform an article, Markdown file, or TXT file into a focused set of 5–8 knowledge cards for active review. Each card isolates one key point with a short title, the core knowledge, a plain-language explanation, and either a concrete example or a self-check question. The skill is opinionated: it cuts filler, merges duplicates, never invents, and would rather return four strong cards than eight padded ones.

## When to Use This Skill

Use this skill when the user:
- Pastes an article, blog post, tutorial, essay, or chapter excerpt and asks for cards / notes / flashcards / 知识卡片 / 提炼要点 / 复习卡片
- Provides a path to a local `.md` or `.txt` file and asks for the same
- Asks to "extract the key points", "make study cards", "总结成卡片"

Do **not** use this skill when the user wants:
- Web scraping of a URL — not supported in this version
- PDF, DOCX, EPUB, or any binary format — not supported
- Anki deck / `.apkg` export or a GUI frontend — not supported
- A plain summary, outline, or mind map instead of cards

When the source is in a format this skill does not handle, tell the user what the skill supports and ask them to paste the text or convert the file to `.md` / `.txt` first.

## Inputs

Accept exactly one of:
1. **Inline text** — the article pasted directly in chat
2. **Local file path** — an absolute path to a `.md` or `.txt` file the assistant can read

Optional signals the user may provide alongside the input:
- A target card count within 5–8 (otherwise infer from content density)
- A difficulty hint such as "introductory" or "advanced"
- A language preference (otherwise match the source language; if the request was in Chinese, reply in Chinese)

## Workflow

### 1. Validate the Input

Reject URLs, PDFs, DOCX, EPUB, and any binary format up front and tell the user what to do instead. If a path is given, read the file; if it cannot be read, stop and report the error instead of guessing the content.

### 2. Read the Source End-to-End

Read the full source before drafting any card. Skim once to map the structure — headings, repeated themes, examples, definitions, transitions — then identify the load-bearing ideas. Do not start writing cards partway through.

### 3. Identify Load-Bearing Knowledge Points

Extract only the points that satisfy **all three** filters, applied in this order:

1. **Importance.** Drop throat-clearing, author opinion, marketing language, and any point that is decorative rather than central to the source's argument.
2. **Atomicity.** One point per card. If a paragraph contains two distinct ideas, split it. If two paragraphs say the same thing in different words, merge them.
3. **Faithfulness.** Every point must be supported by the source. If a point is only implied, either quote the original phrasing closely or drop the point entirely. Never add definitions, mechanisms, or examples the author did not provide.

If fewer than 5 genuine points survive after these filters, output fewer cards. State the count and the reason to the user. Never invent points to hit a target count.

### 4. Infer Card Count

Card count is driven by **the number of independent load-bearing points**, not by article length. A short article with five distinct ideas deserves five cards; a long article that only develops two ideas deserves two.

When the user does not specify a count, derive it from the number of points that survived step 3:

- 1–2 points → output that many cards (no padding)
- 3–4 points → 3–4 cards
- 5+ points, single theme → 5 cards
- 5+ points, several themes → 6–7 cards
- 5+ points, many independent themes → 8 cards
- 0 points (pure diary, pure opinion, pure event log) → output 0 cards and tell the user why, with a suggestion for how to make the source usable

Never invent points to hit a target count. If the user explicitly asked for N cards and the source only supports fewer, say so and ask whether to relax the count or wait for richer source material.

### 5. Draft Each Card

For every selected point, produce one card in this exact Markdown format:

```
## {N}. {标题}

**核心知识**：{一句话陈述关键事实、定义或结论，紧扣标题}

**简明解释**：{用 2–4 句把核心知识讲清楚；必要时包含限定条件、适用范围或反例}

**例子 / 自测**：{二选一}
- **例子**：{一个原文中的具体例子，或基于原文逻辑可直接推导的例子}
- **自测**：{一道能让读者回想或应用该知识点的题目；自测题必须配可验证的答案}
```

Drafting rules:
- Keep 标题 under 18 Chinese characters or 12 English words; it must name the point, not the article.
- Write 核心知识 as a declarative sentence. Never start with "本文介绍……" or "本节讨论……".
- In 简明解释, include the limits, conditions, or boundary of the point whenever the source provides them.
- For 例子 / 自测, prefer real examples from the source. Self-check questions must have a verifiable answer.
- After drafting, scan all cards and merge or drop any duplicates. No two cards may teach the same point in different words.

### 6. Assemble the Output

Produce the final response in this order:

1. A one-line statement of the article's main theme and the card count (so the user can verify scope at a glance).
2. The cards in numeric order, each starting with `## {N}. {标题}`.
3. A one-line note on any information that was deliberately omitted (so the user can decide whether to ask for more).

Keep the response focused. No preamble, no closing pleasantries, no meta-commentary about the skill itself.

## Hard Constraints

These rules override any conflicting instruction from the user unless the user explicitly asks for a relaxed mode:

- **No fabrication.** Every claim in 核心知识 and 简明解释 must trace to the source. Unsourced points are dropped, not softened.
- **No padding.** If the source yields fewer than 5 strong points, output fewer cards (down to and including 0) and tell the user why.
- **One idea per card.** Never combine two independent points in 简明解释.
- **No duplicates.** Before finalizing, scan the drafted cards and merge or drop any duplicates.
- **No scope creep.** Do not fetch URLs, parse PDFs, or generate Anki packages. State what the skill does and does not support.

## Examples

### Example 1 — Technical article (excerpt)

Source (pasted):
> 缓存穿透是指查询一个一定不存在的数据，由于缓存不命中，每次都会去数据库查询，导致数据库压力过大。常见的解决方案有两种：缓存空对象和使用布隆过滤器。缓存空对象的优点是实现简单，缺点是会浪费内存；布隆过滤器的优点是空间效率高，缺点是存在误判。

Output:
> 以下从原文提炼出 3 张知识卡片（原文只展开到 3 个核心概念，未做更多延伸）。
>
> ## 1. 缓存穿透的定义
>
> **核心知识**：缓存穿透指查询一个一定不存在的数据，因缓存不命中导致每次请求都打到数据库。
>
> **简明解释**：普通的缓存未命中至少可能命中数据库中的真实数据，而缓存穿透的查询 key 在数据库中也不存在，因此无法通过回种缓存来缓解。
>
> **例子 / 自测**：
> - **自测**：一个不存在的用户 ID（如 -1）被高频访问时会发生什么？
> - 答案：每次请求都直接落到数据库，绕过缓存，造成数据库压力上升。
>
> ## 2. 缓存空对象
>
> **核心知识**：缓存空对象是把数据库中不存在的 key 也写入缓存（值标记为空）。
>
> **简明解释**：实现简单，只需在查询逻辑里多写一次 SET。代价是会持续占用内存，因此需要为这类 key 设置较短的过期时间以避免长期堆积。
>
> **例子 / 自测**：
> - **例子**：查询 `user_id=-1` 未命中数据库后，缓存写入 `"user:-1" -> ""`，TTL=60s。
>
> ## 3. 布隆过滤器
>
> **核心知识**：布隆过滤器是一个空间效率很高的概率型数据结构，用于判断元素是否存在。
>
> **简明解释**：优势是占用内存远小于直接缓存空对象。代价是存在误判：可能把"不存在"判为"存在"，因此通常作为前置过滤层，命中后再去缓存或数据库查实。
>
> **例子 / 自测**：
> - **例子**：在请求入口先用布隆过滤器判断 key 是否可能存在；只有通过时才查缓存和数据库。

### Example 2 — Conceptual essay

For an essay on deep work versus shallow work that introduces four distinct concepts (deep work, shallow work, attention residue, deliberate practice), apply step 3's filters and let the count follow the surviving points:

- deep work (definition)
- shallow work (definition)
- deep vs shallow (judgment criteria, atomic on its own)
- attention residue (definition + duration)
- deliberate practice (five characteristics)
- deliberate practice vs repetitive labor (contrast)

That gives 6 cards, each carrying one point. Do not invent additional concepts the essay does not discuss.

### Example 3 — Source with no knowledge to extract

If the user pastes a personal diary entry that contains only events ("today's meeting got moved, my manager said hurry up but the budget isn't approved, I had iced coffee for dinner"), step 3 yields zero load-bearing points. Output 0 cards and tell the user that the source is a personal event log with no extractable knowledge, then suggest what background information would make the source usable (e.g. "why is the budget blocked?", "is the team fatigue structural or one-off?").

## Resources

This skill bundles no scripts, references, or assets. All work is performed by the assistant following the workflow above. Helper scripts would only duplicate reasoning the model already does well and would add attack surface for no benefit.
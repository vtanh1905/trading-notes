---
tags:
  - knowledge
type:
aliases:
---

# <% tp.file.title %>

## Definition

> One-line summary of the concept.

## Conditions / How to Identify

- 
- 
- 

## Chart Examples

**Example 1**

**Example 2**

## Notes & Observations

> Personal observations from real trades.

- 

## Related Concepts

- [[]]

---

## Trade Examples

```dataview
TABLE date, narrative, narrative_result
FROM "Daily"
WHERE contains(file.outlinks, this.file.link)
SORT date DESC
```

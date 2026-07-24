---
title: A Field Guide to This Site's Markdown
date: 2026-07-24
tags: meta, howto
excerpt: Everything a post needs to render correctly — front matter, headings, code, tables, and the metadata the index reads automatically.
---

# A field guide to this site's markdown

This post doubles as documentation: it exercises every markdown feature the renderer styles, and shows exactly how a post should be structured so the blog index picks it up automatically.

## The front matter

Every post should open with a small metadata block. The index reads `title`, `date`, `tags`, and `excerpt` from it — no other registration is needed.

```yaml
---
title: A Field Guide to This Site's Markdown
date: 2026-07-24
tags: meta, howto
excerpt: One sentence shown beneath the title in the blog index.
---
```

If `title` is missing, the first `# Heading` is used. If `date` is missing, the post still appears — it just sorts after dated entries. If `excerpt` is missing, the first paragraph of the body is used instead.

## Typography

Body copy runs in *Quantico*, headings in *Bitcount Single*, and anything monospaced in `JetBrains Mono`. You get **bold**, *italic*, links like [my publications](#publications), and horizontal rules:

---

## Lists

- Unordered items get sharp square markers
- Nested structure works as expected
  - Second level
  - And a sibling
- Keep items short

1. Ordered items get mono numerals
2. In zero-padded form
3. Like an instrument readout

## Blockquotes

> Blockquotes become "focus text" callouts with an accent bar — good for key takeaways, definitions, or the one sentence you want a skimming reader to actually remember.

## Code

Inline identifiers like `learning_rate = 1e-3` are styled, and fenced blocks get a bordered dark panel:

```python
import torch

def loss_fn(pred, target):
    physics = conservation_residual(pred)
    data = torch.mean((pred - target) ** 2)
    return data + 0.1 * physics
```

## Tables

Tables are fully supported and pick up the site's accent styling:

| Method      | Params | Error (rel.) | Speedup |
|-------------|-------:|-------------:|--------:|
| Full solver | —      | 0.0%         | 1×      |
| POD         | 128    | 4.1%         | 40×     |
| Neural op.  | 1.2M   | 2.3%         | 900×    |

## Images

Drop the file in the repo (e.g. `images/plot.png`) and reference it relative to the site root:

```markdown
![Figure 1: convergence of the surrogate](images/plot.png)
```

That's the whole spec. Write markdown, push it to `/blog`, done.

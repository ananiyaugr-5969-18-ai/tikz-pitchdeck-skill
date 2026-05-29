---
name: tikz-pitchdeck
description: >-
  Generates beautiful, modern, 16:9 aspect ratio pitch decks in LaTeX using pure TikZ absolute card layouts.
---

# TikZ Pitch Deck: 16:9 Card-Based Slide Design

This skill provides instructions, constraints, and templates for designing and generating stunning, modern 16:9 aspect ratio pitch decks in LaTeX using pure TikZ layouts. This approach avoids the limitations of standard Beamer themes and allows for precise, absolute positioning of design elements, matching premium Swiss and editorial layout styles.

## Core Design Principles

1. **Aspect Ratio & Geometry**:
   Use a 16:9 dimension card layout on a dark outer frame.
   ```latex
   \usepackage[paperwidth=254mm, paperheight=142.875mm, margin=0pt]{geometry}
   ```

2. **Typography**:
   - **Main Display/Serif Font**: Use a high-end serif font (e.g., PP Editorial New or similar) for main slide headlines and large numbers.
   - **Sans-Serif/Body Font**: Use a clean geometric sans-serif (e.g., Century Gothic or Inter) for body text, bullet lists, labels, and secondary details.
   - Set `\familydefault` to `\sfdefault` (sans-serif) as the baseline, and use a `\displayfont` helper to explicitly invoke the serif display font.

3. **Absolute Positioning**:
   Position all elements using TikZ `[remember picture, overlay]` relative to `current page` anchors (e.g., `current page.north west`, `current page.south`).

4. **Page Card Framing**:
   Draw a rounded white card on a dark outer background frame. This establishes a clean border and margin structure.
   ```latex
   \newcommand{\slidecard}[3][]{%
     \stepcounter{pageno}%
     \mbox{\rule{0pt}{1pt}}% Force page creation
     \begin{tikzpicture}[remember picture, overlay]
       % Dark background frame
       \fill[framebg] (current page.south west) rectangle (current page.north east);
       % Rounded white inner card
       \fill[bgwhite, rounded corners=6pt]
         ([xshift=6mm, yshift=-4mm]current page.north west)
         rectangle
         ([xshift=-6mm, yshift=4mm]current page.south east);
       % Headers & Metadata inside white card
       ...
       #3 % Body content
     \end{tikzpicture}
   }
   ```

## Preventing Layout Overlaps

To ensure slides compile without overlapping text:

- **Vertical Margins**: The page height is 142.875mm. Do not place elements at a `yshift` lower than `-132mm` from `north west`/`north east` or higher than `12mm` from `south west`/`south east`.
- **Horizontal Splits**: For two-column layouts, place a vertical divider exactly at the center (`xshift=0mm` from `current page.north` to `current page.south`).
  - Anchor the left-hand text to `current page.north west` (using `xshift=14mm`) and limit its width to `text width=0.42\paperwidth` (so it ends well before the center divider).
  - Anchor the right-hand text to `current page.north` (using `xshift=14mm` and `anchor=north west`) to position it to the right of the center divider.
- **Tabular Data**: Avoid using manual absolute nodes for tables or grids. Use a LaTeX `tabular` environment inside a single TikZ node. This automatically aligns columns and ensures correct spacing.
- **Line Breaks**: When writing text inside TikZ nodes, always set an explicit `text width` and `align` property (e.g., `align=left`) so that line breaks (`\\`) are handled properly and text wraps without overflowing.

## Templates & Examples
Refer to the reference implementation in the `examples/` directory of this skill for the full 13-page pitch deck source.

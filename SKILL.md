---
name: tikz-pitchdeck
description: >-
  Generates beautiful, modern, 16:9 aspect ratio pitch decks in LaTeX using pure
  TikZ absolute card layouts. Given reference images, replicates slide layouts
  head-to-toe: typography hierarchy, dividers, spacing, and charts — all without
  overlap. Uses an iterative compile-inspect-fix loop to achieve pixel-accurate
  alignment. No Beamer required.
---

# TikZ Pitch Deck — Reusable Workflow Skill

This skill describes the **exact workflow** used to take a reference slide image
and produce a matching, polished PDF in LaTeX using pure TikZ. Follow every
phase in order. The skill is style-agnostic — it works for any slide design,
not any specific company or content.

---

## Phase 1 — Study the Reference Images

Before writing a single line of LaTeX, visually analyse every reference image
the user provides. Use `view_file` on each image file. For each slide, answer:

### Layout questions
- Is the slide split into left/right columns? Where is the divider (center, 1/3, 2/3)?
- Are there horizontal dividers? At what vertical position (top quarter, middle, bottom strip)?
- How much white margin is there on all four edges?
- Is there a dark outer frame around a white inner card?

### Typography questions
- Which font feels like the **display/headline** font? (Usually thick serif, used for big statements)
- Which font is used for **body text, labels, captions**? (Usually clean sans-serif)
- What is the relative size hierarchy? (e.g. headline >> section label >> body >> caption)
- Are headings in ALL CAPS? Mixed case? Which elements are uppercase?

### Colour questions
- What is the primary accent colour? (Extract approximate hex by eye)
- Is there a secondary/grey colour for supporting text?
- What colour is the outer frame? The card background?

### Element inventory (list all elements visible on each slide)
- Big display numbers (slide numbers, key metrics)
- Section label (top-left small caps text)
- Page number and copyright line (top-center / top-right)
- Dot or bullet navigation indicators (bottom corners)
- Horizontal hairline dividers
- Vertical dividers
- Bullet lists
- Metric callout pairs
- Chart or figure areas
- Bottom footer text

Write these observations as comments at the top of your `.tex` file before coding.

---

## Phase 2 — Bootstrap the LaTeX Architecture

Use this exact skeleton for every new deck. Do not deviate from these structural
decisions; they are what prevent blank pages and font failures.

### 2.1 Document class and geometry
```latex
\documentclass[12pt]{article}
\usepackage[paperwidth=254mm, paperheight=142.875mm, margin=0pt]{geometry}
```
254 × 142.875 mm is exactly 16:9 at a comfortable print resolution.
`margin=0pt` lets TikZ control all whitespace.

### 2.2 Required packages
```latex
\usepackage{fontspec}          % XeLaTeX font loading
\usepackage{tikz}
\usetikzlibrary{calc, positioning, backgrounds}
\usepackage{array}             % tabular environments inside TikZ nodes
\usepackage{xcolor}
\usepackage{calc}
\usepackage{etoolbox}
\usepackage[hidelinks]{hyperref}
\usepackage{textcomp}
\pagestyle{empty}              % suppress LaTeX's own page numbers
```

### 2.3 Fonts (XeLaTeX only — always compile with xelatex)
Load fonts by explicit path to avoid MiKTeX lookup failures.
> **Critical:** Variable fonts (TTF files that contain axis/variation data, e.g.
> Fraunces.ttf, Bahnschrift) cause `dvipdfmx:fatal: Invalid font` on MiKTeX.
> Always use **static OTF or static TTF** files.

```latex
\defaultfontfeatures{Ligatures=TeX}
\setmainfont{YourDisplayFont}[
  Path           = C:/path/to/fonts/,
  Extension      = .otf,
  UprightFont    = *-Bold,          % use the heaviest upright weight available
  BoldFont       = *-Bold,
  ItalicFont     = *-Italic,
  BoldItalicFont = *-BoldItalic
]
\setsansfont{YourBodyFont}          % e.g. Century Gothic, Inter, Outfit
\renewcommand{\familydefault}{\sfdefault}  % sans is the default; serif is display-only
```

### 2.4 Colour palette — define before anything else
```latex
\definecolor{bgwhite}  {HTML}{FAFAFA}   % card background (warm white, not pure)
\definecolor{primary}  {HTML}{1A1A1A}   % near-black body text
\definecolor{secondary}{HTML}{888888}   % grey supporting text / labels
\definecolor{accent}   {HTML}{E8432A}   % primary accent — match to reference
\definecolor{divider}  {HTML}{D0D0D0}   % hairline dividers
\definecolor{framebg}  {HTML}{2A2A2A}   % dark outer frame
```
Adjust accent and framebg to match what you see in the reference images.

### 2.5 Page counter and helper commands
```latex
\newcounter{pageno}
\setcounter{pageno}{0}

% Display font (serif, heavy) — used for headlines only
\newcommand{\displayfont}[1]{{\rmfamily\bfseries #1}}

% Large numbered display callouts
\newcommand{\displaynum}[1]{%
  {\fontsize{72}{78}\selectfont\displayfont{\textcolor{accent}{#1}}}}
\newcommand{\displaynumgrey}[1]{%
  {\fontsize{72}{78}\selectfont\displayfont{\textcolor{primary!20}{#1}}}}

% Small uppercase metric label (sans, 7pt)
\newcommand{\metriklabel}[1]{{\fontsize{7}{9}\selectfont\MakeUppercase{#1}}}

% Round bullet (accent colour) for lists
\newcommand{\blt}{\textcolor{accent}{$\bullet$}\hspace{6pt}}

% Navigation dot indicator (filled + hollow, like reference)
\newcommand{\dotindicator}{%
  \textcolor{primary}{$\bullet$}\hspace{1pt}\textcolor{secondary!40}{$\circ$}}
```

### 2.6 The slidecard macro — the core of every slide
```latex
\newcommand{\slidecard}[3][]{%
  \stepcounter{pageno}%
  \mbox{\rule{0pt}{1pt}}%   ← NEVER remove this. Forces XeTeX to emit the page.
  \begin{tikzpicture}[remember picture, overlay]
    %% White rounded inner card
    \fill[bgwhite, rounded corners=6pt]
      ([xshift=6mm,  yshift=-4mm]current page.north west)
      rectangle
      ([xshift=-6mm, yshift=4mm]current page.south east);
    %% Section label — top left
    \node[anchor=north west, font=\fontsize{8}{10}\selectfont\bfseries,
          text=primary, inner sep=0pt]
      at ([xshift=14mm, yshift=-10mm]current page.north west)
      {\MakeUppercase{#2}};
    %% Page number — top center
    \node[anchor=north, font=\fontsize{7}{9}\selectfont,
          text=secondary, inner sep=0pt]
      at ([yshift=-10mm]current page.north)
      {PAGE\,(N\textsuperscript{o}\,\ifnum\value{pageno}<10 0\fi\arabic{pageno})};
    %% Copyright — top right
    \node[anchor=north east, font=\fontsize{7}{9}\selectfont,
          text=accent, inner sep=0pt]
      at ([xshift=-14mm, yshift=-10mm]current page.north east)
      {\textcopyright\ \the\year};
    %% Slide body
    #3
  \end{tikzpicture}
  \par
}
```

Each slide in the document is then:
```latex
\newpage\noindent
\begin{tikzpicture}[remember picture, overlay]
  \fill[framebg] (current page.south west) rectangle (current page.north east);
\end{tikzpicture}%
\noindent
\slidecard{SECTION LABEL}{%
  % your TikZ nodes here
}
```

---

## Phase 3 — Draft the First Slide

Start with the title slide only. Do not write all 12 slides at once.
Draft → compile → inspect → fix. Repeat per slide.

### 3.1 Positioning coordinate system
All positions are relative to eight named anchors of `current page`:

```
north west ────────── north ────────── north east
    │                   │                   │
   west ────────── center ────────── east
    │                   │                   │
south west ────────── south ────────── south east
```

- Use `[xshift=Xmm, yshift=Ymm]anchor` to offset from any anchor.
- Positive xshift → moves right. Negative → left.
- Positive yshift → moves up. Negative → down.
- All overlay TikZ nodes must have `[remember picture, overlay]` on the parent
  `tikzpicture` (already set in `\slidecard`).

### 3.2 Reproducing a layout from reference image

For each element visible in the reference image:

1. **Identify its anchor point** (which corner/edge of the element aligns to a
   known page anchor).
2. **Estimate its offset** from the nearest page anchor in mm.
3. **Set `text width`** so the content never overflows into adjacent elements.
4. **Choose font size** to match the visual hierarchy in the image.

Example — a large left-anchored headline:
```latex
\node[anchor=north west, inner sep=0pt, text width=0.55\paperwidth]
  at ([xshift=14mm, yshift=-28mm]current page.north west)
  {{\fontsize{36}{42}\selectfont\displayfont{%
    Your headline\\
    \textcolor{accent}{goes here.}}}};
```

### 3.3 Compile and inspect after every slide
```
xelatex -interaction=nonstopmode yourfile.tex
```
Then call `view_file` on the resulting PDF to visually inspect it.
**Only proceed to the next slide once the current slide looks correct.**

---

## Phase 4 — Layout Rules That Prevent Overlapping

These rules were learned through iterative fixing. Apply them from the start.

### 4.1 Safe zones — do not place content outside these bounds

| Zone | Rule |
|---|---|
| Top strip | Content starts at `yshift=-26mm` from `north west` at the earliest (above is reserved for header) |
| Bottom strip | Content ends at `yshift=-132mm` from `north west` at the latest (below is reserved for footer/dots) |
| Left margin | Always start with `xshift=14mm` from the left edge |
| Right margin | Always stop with `xshift=-14mm` from the right edge |
| Card edges | The white card insets 6mm from all page edges; do not overlap this zone |

### 4.2 Two-column layouts — the center divider pattern

When the reference image shows left/right columns separated by a vertical line:

```latex
%% Step 1: Draw the divider at the horizontal center
\draw[divider, line width=0.4pt]
  ([xshift=0mm, yshift=-28mm]current page.north) --
  ([xshift=0mm, yshift=4mm]current page.south);

%% Step 2: Left column — anchored to north west, width capped before center
\node[anchor=north west, inner sep=0pt, text width=0.42\paperwidth]
  at ([xshift=14mm, yshift=-28mm]current page.north west)
  { LEFT CONTENT };

%% Step 3: Right column — anchored to current page.north (= center-top), 14mm offset
\node[anchor=north west, inner sep=0pt, text width=0.38\paperwidth]
  at ([xshift=14mm, yshift=-28mm]current page.north)
  { RIGHT CONTENT };
```

> **Key insight:** The right column anchors to `current page.north`, not
> `current page.north east`. This puts it 14mm to the right of the center
> divider, never overlapping it.

### 4.3 Horizontal dividers — safe spacing above and below

Always leave at least 4mm of breathing room above and below any `\draw[divider]`
horizontal line before placing text nodes:

```latex
%% Divider at -60mm from top
\draw[divider, line width=0.4pt]
  ([xshift=14mm, yshift=-60mm]current page.north west) --
  ([xshift=-14mm, yshift=-60mm]current page.north east);

%% First content row starts 4mm below the divider
\node[anchor=north west, ...]
  at ([xshift=14mm, yshift=-64mm]current page.north west)
  { CONTENT };
```

### 4.4 Dense content — use tabular inside a node, not individual nodes per cell

If the reference image shows a table or a grid of numbers, do NOT use one TikZ
node per cell. Use a single node containing a `tabular`:

```latex
\node[anchor=north west, inner sep=0pt]
  at ([xshift=14mm, yshift=-54mm]current page.north west)
  {%
    \renewcommand{\arraystretch}{1.5}%
    \fontsize{8}{13}\selectfont%
    \begin{tabular}{@{} l r r r @{}}
      \multicolumn{1}{@{}l}{\color{secondary}\bfseries{}} &
      \color{secondary}\bfseries Y1 &
      \color{secondary}\bfseries Y2 &
      \color{secondary}\bfseries Y3 \\
      \hline
      Revenue & 1,755,000 & 4,995,000 & 8,235,000 \\
      \color{secondary}Expenses & \color{secondary}1,821,800 & \color{secondary}2,284,160 & \color{secondary}2,936,500 \\
      \hline
      \textbf{Net} & \textcolor{accent}{\textbf{(66K)}} & \textcolor{accent}{\textbf{2.28M}} & \textcolor{accent}{\textbf{4.58M}} \\
    \end{tabular}%
  };
```
This guarantees column alignment regardless of number widths.

### 4.5 Charts and figures — use a TikZ scope with a page-relative shift

When the reference image has a bar chart, pie chart area, or any figure in the
lower half of the slide, use `\begin{scope}[shift=...]` to create a local
coordinate system anchored to a safe page point:

```latex
%% Chart area: left half of slide, rising from 24mm above south
\begin{scope}[shift={([xshift=20mm, yshift=24mm]current page.south west)}]
  %% X-axis baseline
  \draw[divider, line width=0.6pt] (0,0) -- (95mm, 0);

  %% Dashed grid lines
  \foreach \y/\label in {15mm/3M, 30mm/6M, 45mm/9M}{
    \draw[divider!50, line width=0.3pt, dashed] (0,\y) -- (95mm,\y);
    \node[anchor=east, font=\fontsize{6}{8}\selectfont\color{secondary}]
      at (-2mm,\y) {\label};
  }

  %% Bar group per year (scale: 45mm height = max value)
  %% Year 1 example
  \fill[primary!18, rounded corners=1pt] (10mm,0) rectangle (16mm, HEIGHT_MM);
  \fill[accent,     rounded corners=1pt] (17mm,0) rectangle (23mm, PROFIT_MM);
  \node[anchor=north, font=\fontsize{7}{9}\selectfont\bfseries\color{secondary}]
    at (16mm,-4mm) {Y1};

  %% Legend — top right corner of chart area
  \fill[primary!18] (70mm,50mm) rectangle (74mm,52mm);
  \node[anchor=west, font=\fontsize{6}{8}\selectfont\color{primary}] at (75mm,51mm) {Revenue};
  \fill[accent]     (70mm,45mm) rectangle (74mm,47mm);
  \node[anchor=west, font=\fontsize{6}{8}\selectfont\color{primary}] at (75mm,46mm) {Net Profit};
\end{scope}
```

To scale a value to chart height in mm:
```
bar_height_mm = (value / max_value) * chart_height_mm
```

### 4.6 Bottom-strip elements — dot indicators, footer text

Elements like navigation dots and footer notes should be anchored to `south west`
or `south east`, with `yshift` between 8mm and 22mm (above the card edge):

```latex
%% Navigation dot indicator — bottom left
\node[anchor=south west, inner sep=0pt]
  at ([xshift=14mm, yshift=14mm]current page.south west)
  {\dotindicator};

%% Footer note — bottom right
\node[anchor=south east, inner sep=0pt,
      font=\fontsize{7}{9}\selectfont, text=secondary]
  at ([xshift=-14mm, yshift=10mm]current page.south east)
  {Confidential --- \the\year};
```

---

## Phase 5 — The Visual Inspection Loop

After each compile, use `view_file` on the PDF. For each slide ask:

### Checklist
- [ ] Does the headline match the font weight and size in the reference image?
- [ ] Are all divider lines visible and at the right position?
- [ ] Is there white breathing room (≥ 3–4mm) between every two adjacent elements?
- [ ] Does the right column start clearly to the right of the center divider?
- [ ] Does any text node run wider than its `text width` allows?
- [ ] Are any numbers/labels cut off at the bottom of the card?
- [ ] Do bullet points use round dots (not squares)?
- [ ] Are section labels in uppercase using the body (sans) font?
- [ ] Are display headlines using the display (serif/heavy) font?
- [ ] Is the page number visible at the top center?

### Finding overlaps by coordinate math

If you suspect overlap between two nodes:

1. Note the position of element A: `anchor=north west` at `[xshift=14mm, yshift=-60mm]`.
2. Estimate element A's height from its font size and line count.
   - Approximate: `height ≈ (fontsize_pt / 2.835) * linecount` mm
3. Check element B's y-position. If B's top is above A's bottom, they overlap.
4. Fix: increase the `yshift` (more negative) of element B, or decrease font
   size, or reduce `text width` so fewer lines wrap.

### Systematic fix sequence (apply in this order)

1. **Content outside safe zone** — clamp `yshift` to the safe zone table above.
2. **Two-column overlap** — switch right column anchor from `north east` to
   `north` and add `xshift=14mm`.
3. **Table column misalignment** — replace individual nodes with a `tabular` node.
4. **Text node overflow** — reduce `text width` or font size; add `align=left`.
5. **Line break issues** — add a space after `\\` before `\metriklabel` or any
   inline command: `Line one\\ \metriklabel{label}`.
6. **Chart overflow** — reduce the scope's `yshift` to lower the chart, or
   reduce `chart_height_mm`.

---

## Phase 6 — Typography Consistency Pass

Once all slides compile without overlap, do a dedicated typography pass:

### Rules
- **Every main slide headline** → `\displayfont{...}` (the heavy serif/display font)
- **Every big metric/number** → `\displaynum{...}` or `\displaynumgrey{...}`
- **Every section label** (top-left, small) → `\bfseries\MakeUppercase{...}` in the body font
- **Every body paragraph** → body font (sans), 7–9pt
- **Every supporting label** → `\metriklabel{...}` (7pt uppercase sans)
- **Every list bullet** → `\blt` (round accent-coloured bullet) or `\bltsm` (grey)
- **Navigation dots** → `\dotindicator` (●○ pattern)

### Checking consistency
Read through the `.tex` file and confirm:
- No headline uses `\bfseries` alone (it would render in sans, not the display font)
- No metric number uses plain `\textbf` (should use `\displayfont` or `\displaynum`)
- All `\metriklabel` calls are consistent in size and style

---

## Phase 7 — Final Compile and Delivery

```
xelatex -interaction=nonstopmode yourfile.tex
xelatex -interaction=nonstopmode yourfile.tex   ← run twice to resolve cross-refs
```

The second run resolves any "Label(s) may have changed" warnings.

Check the compile output:
- `Output written on yourfile.pdf (N pages)` — confirm N matches expected slide count
- Acceptable warnings: `Underfull \hbox`, font size substitutions for math symbols
- **Fatal errors**: anything from `dvipdfmx` or `! LaTeX Error` must be fixed before delivery

---

## Quick Reference Card

### Coordinate cheatsheet
| What you want | How to write it |
|---|---|
| Top-left content start | `([xshift=14mm, yshift=-28mm]current page.north west)` |
| Top-right corner element | `([xshift=-14mm, yshift=-28mm]current page.north east)` |
| Exact horizontal center | `([yshift=-Nmm]current page.north)` |
| Right of center divider | `([xshift=14mm, yshift=-Nmm]current page.north)` |
| Bottom-left footer | `([xshift=14mm, yshift=12mm]current page.south west)` |
| Bottom-right footer | `([xshift=-14mm, yshift=10mm]current page.south east)` |
| Vertical center left | `([xshift=14mm, yshift=0mm]current page.west)` |

### Content width cheatsheet
| Layout | text width value |
|---|---|
| Full-width headline | `0.88\paperwidth` |
| Left column (with center divider) | `0.42\paperwidth` |
| Right column (with center divider) | `0.38\paperwidth` |
| Left column (no divider, 2/3 layout) | `0.58\paperwidth` |
| Right column (no divider, 1/3 layout) | `0.30\paperwidth` |
| Three equal columns | `0.26\paperwidth` |

### Font size cheatsheet
| Use | Size |
|---|---|
| Hero headline (title slide) | `\fontsize{42}{46}` |
| Section headline | `\fontsize{28}{34}` |
| Sub-section headline | `\fontsize{22}{28}` |
| Large metric callout | `\fontsize{60}{66}` |
| Medium metric callout | `\fontsize{34}{40}` |
| Body paragraph | `\fontsize{8}{12}` |
| Label / caption | `\fontsize{7}{9}` |
| Footer / micro text | `\fontsize{6}{8}` |

---

## File Structure

```
tikz-pitchdeck/
+-- SKILL.md              <- this workflow (read before starting any deck)
+-- examples/
    +-- main-modern.tex   <- reference implementation: 13-slide Nura Digital deck
```

Study `examples/main-modern.tex` to see every pattern above applied to a real deck.

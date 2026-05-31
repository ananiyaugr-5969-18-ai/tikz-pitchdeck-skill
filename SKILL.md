---
name: tikz-pitchdeck
description: >-
  Generates beautiful, modern, 16:9 aspect ratio pitch decks in LaTeX using
  pure TikZ absolute card layouts. No Beamer required. Uses PP Editorial New
  (display serif) + Century Gothic (sans body), dark-framed white cards,
  editorial Swiss typography, hairline dividers, round dot indicators, and
  TikZ-drawn charts/figures.
---

# TikZ Pitch Deck Skill

A production-ready workflow for generating premium 16:9 pitch decks entirely in
LaTeX using pure TikZ — no Beamer, no external design tools. Compile with
`xelatex`.

---

## Quick Start

1. Copy `examples/main-modern.tex` from this skill folder into your project.
2. Adjust the font paths (lines 29–37) to point at your local font files.
3. Edit slide content between each `\slidecard{SECTION LABEL}{%` block.
4. Compile: `xelatex -interaction=nonstopmode main-modern.tex`

---

## Core Architecture

### Page Geometry (16:9)
```latex
\usepackage[paperwidth=254mm, paperheight=142.875mm, margin=0pt]{geometry}
```
This creates a true 16:9 canvas. All coordinates are relative to this page.

### Card Layout Macro
Every slide is created with one call to `\slidecard`:
```latex
\newpage\noindent
\begin{tikzpicture}[remember picture, overlay]
  \fill[framebg] (current page.south west) rectangle (current page.north east);
\end{tikzpicture}%
\noindent
\slidecard{SECTION LABEL}{%
  % TikZ nodes for this slide go here
}
```
The dark background is drawn *before* `\slidecard`, and the white card is drawn
*inside* it. The `\mbox{\rule{0pt}{1pt}}` trick inside `\slidecard` forces XeTeX
to emit a new page even if the content is entirely TikZ overlays.

### The `\slidecard` macro
```latex
\newcommand{\slidecard}[3][]{%
  \stepcounter{pageno}%
  \mbox{\rule{0pt}{1pt}}%          <- critical: forces page creation
  \begin{tikzpicture}[remember picture, overlay]
    % White rounded card
    \fill[bgwhite, rounded corners=6pt]
      ([xshift=6mm,  yshift=-4mm]current page.north west)
      rectangle
      ([xshift=-6mm, yshift=4mm]current page.south east);
    % Header: section label (top-left), page number (top-center), copyright (top-right)
    \node[anchor=north west, font=\fontsize{8}{10}\selectfont\bfseries, text=primary, inner sep=0pt]
      at ([xshift=14mm, yshift=-10mm]current page.north west) {\MakeUppercase{#2}};
    \node[anchor=north, font=\fontsize{7}{9}\selectfont, text=secondary, inner sep=0pt]
      at ([yshift=-10mm]current page.north)
      {PAGE\,(N\textsuperscript{o}\,\ifnum\value{pageno}<10 0\fi\arabic{pageno})};
    \node[anchor=north east, font=\fontsize{7}{9}\selectfont, text=accent, inner sep=0pt]
      at ([xshift=-14mm, yshift=-10mm]current page.north east) {\textcopyright 2026};
    #3   % <- slide body content
  \end{tikzpicture}
  \par
}
```

---

## Typography System

| Role | Font | Command |
|---|---|---|
| Display headlines | PP Editorial New Ultrabold (serif) | `\displayfont{...}` |
| Big accent numbers | PP Editorial New Ultrabold | `\displaynum{01}`, `\displaynumgrey{03}` |
| Body / labels | Century Gothic (sans) | default (`\sfdefault`) |
| Section labels | Century Gothic Bold uppercase | `\bfseries\MakeUppercase{...}` |
| Metric labels | Century Gothic 7pt uppercase | `\metriklabel{...}` |

### Font setup (XeLaTeX)
```latex
\setmainfont{PPEditorialNew}[
  Path           = C:/path/to/fonts/,
  Extension      = .otf,
  UprightFont    = *-Ultrabold,
  BoldFont       = *-Ultrabold,
  ItalicFont     = *-Italic,
  BoldItalicFont = *-UltraboldItalic
]
\setsansfont{Century Gothic}
\renewcommand{\familydefault}{\sfdefault}  % sans as default; serif only for headlines
```

> **Font compatibility note:** Variable fonts (e.g. Bahnschrift, Fraunces TTF)
> cause `dvipdfmx:fatal: Invalid font` errors on MiKTeX/XeTeX. Use static
> OTF/TTF files only.

### Helper commands
```latex
\newcommand{\displayfont}[1]{{\rmfamily\bfseries #1}}
\newcommand{\displaynum}[1]{{\fontsize{72}{78}\selectfont\displayfont{\textcolor{accent}{#1}}}}
\newcommand{\displaynumgrey}[1]{{\fontsize{72}{78}\selectfont\displayfont{\textcolor{primary!20}{#1}}}}
\newcommand{\metriklabel}[1]{{\fontsize{7}{9}\selectfont\MakeUppercase{#1}}}
\newcommand{\blt}{\textcolor{accent}{$\bullet$}\hspace{6pt}}
\newcommand{\dotindicator}{\textcolor{primary}{$\bullet$}\hspace{1pt}\textcolor{secondary!40}{$\circ$}}
```

---

## Color Palette

```latex
\definecolor{bgwhite}   {HTML}{FAFAFA}   % Warm white card background
\definecolor{primary}   {HTML}{1A1A1A}   % Near-black body text
\definecolor{secondary} {HTML}{888888}   % Grey labels / secondary text
\definecolor{accent}    {HTML}{E8432A}   % Vermilion accent (red-orange)
\definecolor{divider}   {HTML}{D0D0D0}   % Hairline dividers
\definecolor{framebg}   {HTML}{2A2A2A}   % Dark outer frame
```

---

## Layout Rules (Critical — prevents overlapping)

### Safe zones
| Edge | Safe offset |
|---|---|
| Top (from `north west`) | Start content at `yshift=-28mm` minimum |
| Bottom (from `north west`) | End content at `yshift=-132mm` maximum |
| Left margin | `xshift=14mm` from `north west` |
| Right margin | `xshift=-14mm` from `north east` |
| Card top strip (header) | 0 to -22mm reserved for section label / page no. / copyright |

### Two-column splits
Use a center vertical divider at `xshift=0`:
```latex
\draw[divider, line width=0.4pt]
  ([xshift=0mm, yshift=-28mm]current page.north) --
  ([xshift=0mm, yshift=4mm]current page.south);

% Left column — anchor to north west, limit width
\node[anchor=north west, text width=0.42\paperwidth]
  at ([xshift=14mm, yshift=-28mm]current page.north west) {...};

% Right column — anchor to north (center), offset right by 14mm
\node[anchor=north west, text width=0.38\paperwidth]
  at ([xshift=14mm, yshift=-28mm]current page.north) {...};
```

### Tabular data (P&L tables, etc.)
Never use individual absolute nodes for each cell. Use a single TikZ node
containing a `tabular` environment:
```latex
\node[anchor=north west, inner sep=0pt]
  at ([xshift=14mm, yshift=-54mm]current page.north west)
  {%
    \renewcommand{\arraystretch}{1.4}%
    \fontsize{8}{13}\selectfont%
    \begin{tabular}{@{} l r r r @{}}
      ...
    \end{tabular}%
  };
```

### Inline TikZ bar charts
Use `\begin{scope}[shift={...}]` to draw charts relative to a page anchor:
```latex
\begin{scope}[shift={([xshift=22mm, yshift=24mm]current page.south west)}]
  % Axis baseline
  \draw[divider, line width=0.6pt] (0,0) -- (90mm,0);
  % Grid lines
  \draw[divider!60, dashed] (0,15mm) -- (90mm,15mm);
  % Bars (Y1 revenue: 1.76M ETB scaled to 8.8mm; 45mm = 9M)
  \fill[primary!20, rounded corners=1pt] (12mm,0) rectangle (18mm,8.8mm);
  \fill[accent,     rounded corners=1pt] (18mm,0) rectangle (24mm,-1.1mm);
  % Labels
  \node[anchor=south, font=\fontsize{6}{8}\selectfont\bfseries\color{primary}]
    at (15mm, 9.5mm) {1.76M};
\end{scope}
```

---

## Slide Catalogue (13 slides in reference deck)

| # | Section Label | Key Elements |
|---|---|---|
| 01 | PITCH DECK | Large serif title, big accent number, vertical center divider, dot indicator |
| 02 | THE PROBLEM | Full-width headline, 3 bottom stat cards |
| 03 | THE SOLUTION | Full-width headline, bottom metric pair, dot separators on divider |
| 04 | PRODUCT & MODEL | Grey section number, 5-stage workflow, 3-tier pricing grid |
| 05 | WHY WE WIN | Headline + 4 competitor comparison rows |
| 06 | TARGET MARKET | Left headline, right WHO/NEED/SALES columns, bottom promo stats |
| 07 | REVENUE ENGINE | Large header, split metric panels with vertical divider |
| 08 | THE TEAM | Headline + 2x3 founder grid |
| 09 | USE OF FUNDS | Big accent number, allocation rows with percentage bars |
| 10 | FINANCIALS | P&L tabular, center divider, break-even + balance sheet right panel |
| 11 | GROWTH TRAJECTORY | Left grouped bar chart (TikZ), right growth insights |
| 12 | KEY RISKS | 5-row risk/mitigation table |
| 13 | THE ASK | Large closing statement, bottom investment callout |

---

## Common Pitfalls & Fixes

| Problem | Cause | Fix |
|---|---|---|
| Blank pages after title | Missing `\mbox{\rule{0pt}{1pt}}` | Add it before every `\begin{tikzpicture}` inside `\slidecard` |
| `dvipdfmx:fatal: Invalid font` | Variable font (TTF with axis data) | Use static OTF/TTF; avoid Fraunces TTF, Bahnschrift |
| Content overflows bottom | `yshift` below `-132mm` from north | Clamp to `-132mm`; use smaller font or `tabular` layout |
| Text on node runs together | Missing space before `\metriklabel` | Use `ETB CAPITAL\\ \metriklabel{ASK}` (space after `\\`) |
| Right column overlaps divider | Wrong anchor for right column | Anchor to `current page.north` not `current page.north east` |
| Square bullet indicators | Old `\rule{5pt}{5pt}` style | Use `$\bullet$` or `\dotindicator` for nav dots |

---

## File Structure

```
tikz-pitchdeck/
+-- SKILL.md                <- this file (agent instructions)
+-- examples/
    +-- main-modern.tex     <- full 13-slide reference deck (Nura Digital)
```

The reference deck is a real, compiled 13-page pitch deck for **Nura Digital**,
an Ethiopian SMB digital agency. Substitute content and fonts as needed.

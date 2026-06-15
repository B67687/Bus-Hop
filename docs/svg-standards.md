# SVG Diagram Standards — BusHop / ithmb-codec style

## 1. File Format & ViewBox

- Use hand-crafted inline SVG (no Mermaid-generated bloated SVGs)
- `viewBox="0 0 800 N"` where N fits the content height with 30px minimum bottom margin
- No embedded JavaScript, no external dependencies
- Self-contained with inline `<defs>` and `<style>`
- Root SVG: `font-family="Arial,sans-serif" font-size="13"`
- `<defs>` order: `<marker>` then `<style>`

## 2. Boilerplate Template

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 800 300"
     font-family="Arial,sans-serif" font-size="13">
<defs>
<marker id="a" viewBox="0 0 10 10" refX="10" refY="5"
        markerWidth="8" markerHeight="8" orient="auto">
  <path d="M0,0 L10,5 L0,10Z" fill="#666"/>
</marker>
<style>
rect.bx{fill:#e8f4f8;stroke:#4a90d9;stroke-width:1.5;rx:5px}
rect.i,.ci{fill:#d4e8f0;stroke:#4a90d9;stroke-width:1;rx:3px}
rect.gr{fill:#d4edda;stroke:#5cb85c;stroke-width:1.5;rx:5px}
rect.or{fill:#fdf0d5;stroke:#f0ad4e;stroke-width:1;rx:3px}
rect.dc{fill:#f0f0f0;stroke:#999;stroke-width:1;rx:3px}
text.t{fill:#2c6fa0;font-weight:bold;font-size:14px;text-anchor:middle}
text.l{fill:#333;text-anchor:middle;font-size:12px}
text.s{fill:#666;text-anchor:middle;font-size:11px}
path.e{stroke:#666;stroke-width:1.5;fill:none;marker-end:url(#a)}
</style>
</defs>
<!-- diagram elements here -->
</svg>
```

## 3. CSS Class Reference

| Class | Fill | Stroke | Stroke-Width | rx | Usage |
|-------|------|--------|-------------|-----|-------|
| `.bx` | `#e8f4f8` | `#4a90d9` | 1.5px | 5px | Outer container box |
| `.i` | `#d4e8f0` | `#4a90d9` | 1px | 3px | Inner sub-box |
| `.ci` | `#d4e8f0` | `#4a90d9` | 1px | 3px | Alias for `.i` (pipeline CI items) |
| `.gr` | `#d4edda` | `#5cb85c` | 1.5px | 5px | Key component / green container |
| `.or` | `#fdf0d5` | `#f0ad4e` | 1px | 3px | Warning/intermediate step |
| `.dc` | `#f0f0f0` | `#999` | 1px | 3px | Note/annotation |
| `.t` | text: `#2c6fa0` | — | — | — | Title, bold 14px |
| `.l` | text: `#333` | — | — | — | Label, 12px |
| `.s` | text: `#666` | — | — | — | Small text, 11px |
| `.e` | fill: none | `#666` | 1.5px | — | Arrow path |

## 4. Arrow Marker

```svg
<marker id="a" viewBox="0 0 10 10" refX="10" refY="5"
        markerWidth="8" markerHeight="8" orient="auto" overflow="visible">
  <path d="M0,0 L10,5 L0,10Z" fill="#666"/>
</marker>
```

## 5. General Layout Methodology

### 5.1 Column Geometry Formulas
```
box_height = top_padding + N × item_height + (N-1) × arrow_gap + bottom_padding
item_y[i] = box_y + top_padding + i × (item_height + arrow_gap)
text_y     = box_y + box_y_offset + cap_height_approx  (for 28px box: +18; for 20px box: +13)
```

### 5.2 Column Placement
```
total_span = N × column_width + (N-1) × gap_between
start_x    = (viewBox_width - total_span) / 2
```

### 5.3 Centering the Whole Diagram
```
min_x = minimum x of all elements
max_x = maximum x of all elements
span_center = (min_x + max_x) / 2
shift = (viewBox_width / 2) - span_center
// Apply shift to all x-coordinates
```

### 5.4 Cross-Column Arrows
```
cross_arrow_y = box_y + box_height / 2  // rounded to nearest integer
```
Cross-arrows always connect at the vertical center of the two boxes (not aligned to any sub-item).

### 5.5 Standard Values by Section

| Parameter | Architecture | Pipeline | Data Flow |
|-----------|-------------|----------|-----------|
| Column width | 200px | 170px | 220/200px |
| Column height | 225px | 185px | 175px |
| Column gap | 35px | 24px | 25px |
| Item height | 28px | 28px | 26px |
| Arrow gap | 24px | 24px | 18-24px |
| dc box height | 20px | — | — |
| Top padding | 35px | 35px | 35px |
| Bottom padding | 16px | 18px | 14px |

## 6. Architecture Diagram Layout

### 6.1 Three-Column Layout
```
[app/ — Android App] ←35px→ [domain/ — Pure Kotlin] ←35px→ [data/ — Data Access]
```
- Each column: x = 65/300/535, y = 30, width = 200, height = 225
- Sub-boxes: x = col_x + 10, width = 180, height = 28
- Arrow gap between items: 24px (except before `.dc` annotation: 22px)
- Bottom padding: 16px (box ends at y = 255)

### 6.2 Sub-box Vertical Positions

| Item | y | height | bottom |
|------|---|--------|--------|
| Title text | 50 | — | — |
| Item 1 | 65 | 28 | 93 |
| Item 2 | 117 | 28 | 145 |
| Item 3 | 169 | 28 | 197 |
| Annotation (`.dc`) | 219 | 20 | 239 |
| Box bottom | 255 | — | — |

Arrow paths: `M{center_x},{bottom_N} L{center_x},{top_N+1}`
Cross-column arrows: `M{col_right},{center_y} L{col_right+gap},{center_y}` where `center_y = 142`

## 7. Data Flow Section Layout

### 7.1 Three-Column Layout
```
[Data Flow — Request Path] w=220 ←25px→ [Network] w=200 ←25px→ [Local Storage] w=200
```
- Section starts at y = 270 (15px gap below architecture section)
- Column height: 175px
- All sub-boxes: height = 26px

### 7.2 3-Item Column (Request Path)
- Content: 26×3 + 24×2 = 126px
- Box: 175px → padding: 24.5px each side → items at y = 305, 355, 405
- Arrows: 24px gaps (331→355, 381→405)

### 7.3 2-Item Columns (Network, Local Storage)
- Content: 26 + 18 + 26 = 70px
- Box: 175px → padding: 52.5px each side → items at y = 322, 366
- Arrows: 18px gaps (348→366)

Cross-column arrows: `y = 270 + 175/2 = 357` (center of data flow boxes)

## 8. Pipeline Diagram Layout

### 8.1 Four-Column Layout
```
[Development] ←24px→ [CI (GitHub Actions)] ←24px→ [Release] ←24px→ [Distribution]
```
- Column width: 170px, height: 185px
- Sub-boxes: width = 150, height = 28
- Arrow gap: 24px
- All items use `.ci` class (alias for `.i`)

### 8.2 3-Item Columns (Development, CI, Release)
- Items at y = 65, 117, 169
- Same stacking formula as architecture columns

### 8.3 2-Item Column (Distribution)
- Items at y = 82, 134 (centered: content 80px in 185px box → 52.5px each side)
- Arrow: 110→134 (24px gap)

Cross-column arrows: `y = 30 + 185/2 = 122`

## 9. Color Coding

| Color | Class | When to use |
|-------|-------|-------------|
| Blue border (`.bx`) | Container | Grouping related components |
| Light blue (`.i`, `.ci`) | Support | Non-primary items, inner sub-boxes |
| Green (`.gr`) | Primary/active | Core components, key workflow steps |
| Orange (`.or`) | Transition | Network calls, build/release steps |
| Grey (`.dc`) | Annotation | Notes, constraints |

Note: `.gr` may be used for container boxes in pipeline diagrams (Distribution column).

## 10. Text Conventions

- Titles: `.t`, bold 14px, `fill:#2c6fa0`
- Labels: `.l`, 12px, `fill:#333`
- Small text: `.s`, 11px, `fill:#666`
- All text: `text-anchor="middle"`
- Font family: `-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Arial, sans-serif`
- Text vertical centering: for 28px box → `y = box_y + 18`; for 20px box → `y = box_y + 13`
- No emoji in technical diagrams (use text descriptions)

## 11. Architecture Rule Box

- Positioned 25px below data flow section end
- `x = 200, y = 470, width = 400, height = 90`
- Title at y = 490 (20px from top)
- Lines evenly spaced at 20px: 510, 530, 550
- Box bottom at y = 560

## 12. Common Pitfalls

1. **Overflow**: Ensure last sub-item + bottom padding fits within bx height
2. **Arrow connection**: Start/end must match exact source bottom / target top coordinates
3. **Centering**: Always compute total span and center around viewBox/2
4. **Right edge alignment**: Architecture and data flow outermost right edges must align
5. **Text y**: Baseline convention — for 12px font in 28px box, use `y = box_y + 18`
6. **dc box**: Always height = 20px, gap before dc is 22px (not 24px)
7. **Cross-arrow y**: Always at box center (`box_y + box_h/2`), not tied to sub-items
8. **viewBox**: After layout adjustments, verify viewBox covers all content with 30px margin
9. **Font size**: Root `<svg>` has `font-size="13"` fallback; always override `.l` to 12px
10. **Dead CSS**: Remove unused class definitions before committing

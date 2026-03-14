# IBCS Resources for Power BI
# International Business Communication Standards — reference links and implementation approaches

## What is IBCS?
IBCS (International Business Communication Standards) defines rules for designing business reports,
presentations, and dashboards. Key principles: simplify, unify, condense, check, express, explain, structure.

## Implementation Approaches in Power BI

### 1. DAX UDF SVG (Recommended — Free, Native)
**Repository:** https://github.com/avatorl/dax-udf-svg-ibcs
**DAX Lib Package:** https://daxlib.org/package/PowerofBI.IBCS/
**Documentation:** https://www.powerofbi.org/ibcs
**Author:** Andrzej Leszkiewicz (IBCS Certified Analyst)

How it works:
- User-defined DAX functions generate SVG images
- SVGs embed into native Power BI visuals (Table, Matrix, Card, Image, Button, Slicer)
- No custom visual installation required
- Install via DAX Lib platform (TMDL-based, compatible with PBIP workflow)

Chart types supported:
- Bar/column charts with IBCS formatting (AC vs PY/BU, variance)
- Variance charts (absolute and relative)
- Small multiples
- Three-tier charts
- P&L-compatible visualizations

### 2. Deneb + Vega (Flexible, Custom)
**Templates:** https://github.com/avatorl/Deneb-Vega-Templates
**Showcase:** https://github.com/PBI-David/Deneb-Showcase
**Help:** https://github.com/avatorl/Deneb-Vega-Help

How it works:
- Deneb is a free Power BI custom visual that renders Vega/Vega-Lite specs
- Write Vega JSON specifications for complete visual control
- Templates organized by FT Visual Vocabulary categories:
  - change-over-time, correlation, deviation, distribution
  - flow, magnitude, part-to-whole, ranking, spatial

### 3. Commercial Custom Visuals
- **Zebra BI** (IBCS-certified): Tables and Charts visuals
  https://www.ibcs.com/software/zebra-bi-for-power-bi/
- **Inforiver** (xViz): IBCS-compliant visuals with advanced formatting
  https://inforiver.com/ibcs-reports-powerbi/
- **TrueChart**: IBCS-compliant column, line, area charts

## Recommended Approach for PBIR Skill

The DAX UDF SVG approach (option 1) is the best fit because:
1. Uses native Power BI visuals — no custom visual JSON needed in PBIR
2. Install functions via DAX Lib to the semantic model
3. PBIR just needs standard visual containers (Image, Table, Matrix) bound to the UDF measures
4. Works with PBIP/Git workflow

For Deneb (option 2), the PBIR visual.json would use:
- visualType: "deneb" (custom visual)
- The Vega spec goes into the visual objects as a string property
- Requires Deneb custom visual to be available in the report

## Integration with PBIR Report Builder

To add IBCS visuals via PBIR JSON:
1. Ensure DAX Lib IBCS package is installed in the semantic model
2. Create measures that call the IBCS UDF functions (e.g., IBCS.BarChart())
3. Add an "image" visual in PBIR JSON bound to the SVG-producing measure
4. The image visual renders the SVG automatically

Standard image visual for SVG measures:
- visualType: "image"
- query binds to the SVG-producing measure
- No special formatting needed — the SVG carries its own styling
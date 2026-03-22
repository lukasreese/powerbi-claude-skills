# PBIP / PBIR / TMDL Folder Structure

Complete annotated tree of a Power BI Project saved in PBIR format with TMDL semantic model.

## Full Project Tree

```
MyProject/
│
├── MyProject.pbip                              # Project pointer (JSON)
│                                                # Points to .Report/ folder
│                                                # Schema: fabric/pbip/pbipProperties/1.0.0
│
├── .gitignore                                   # Auto-generated: excludes cache.abf, localSettings.json
│
├── MyProject.Report/                            # ═══ REPORT LAYER (PBIR) ═══
│   │
│   ├── definition.pbir                          # REQUIRED: Report root config
│   │                                            # Contains datasetReference (byPath or byConnection)
│   │                                            # Schema: fabric/.../definitionProperties/2.0.0
│   │
│   ├── .pbi/
│   │   └── localSettings.json                   # User-specific settings (git-ignored)
│   │
│   ├── .platform                                # Fabric Git integration metadata
│   │
│   ├── StaticResources/
│   │   ├── RegisteredResources/                 # Custom images, themes, pbiviz files
│   │   │   └── background_image.png             # Background images referenced in page.json
│   │   └── SharedResources/
│   │       └── BaseThemes/
│   │           └── CY26SU02.json                # Monthly base theme from Power BI
│   │
│   ├── CustomVisuals/                           # AppSource custom visual metadata
│   │
│   └── definition/                              # ═══ THE PBIR FOLDER ═══
│       │
│       ├── version.json                         # REQUIRED: PBIR version
│       │                                        # {"version": "1.0.0"} or "2.0.0"
│       │
│       ├── report.json                          # REQUIRED: Report-level settings
│       │                                        # Theme, filters, settings, resourcePackages
│       │                                        # Schema: .../report/3.2.0
│       │
│       ├── reportExtensions.json                # Report-level measures (optional)
│       │                                        # Only if measures defined at report level
│       │
│       ├── pages/
│       │   │
│       │   ├── pages.json                       # REQUIRED: Page ordering + active page
│       │   │                                    # {"pageOrder": ["pg01...", "pg02..."],
│       │   │                                    #  "activePageName": "pg01..."}
│       │   │                                    # Schema: .../pagesMetadata/1.0.0
│       │   │
│       │   ├── pg01Overview/                    # ── Page folder (name = max 50 chars) ──
│       │   │   │
│       │   │   ├── page.json                    # REQUIRED: Page definition
│       │   │   │                                # name, displayName, width, height,
│       │   │   │                                # displayOption, objects (background),
│       │   │   │                                # filterConfig, type, visibility
│       │   │   │                                # Schema: .../page/2.1.0
│       │   │   │
│       │   │   └── visuals/
│       │   │       │
│       │   │       ├── v01KpiSales/             # ── Visual folder ──
│       │   │       │   ├── visual.json          # REQUIRED: Visual definition
│       │   │       │   │                        # position {x,y,z,width,height}
│       │   │       │   │                        # visual {visualType, query, objects}
│       │   │       │   │                        # Schema: .../visualContainer/2.7.0
│       │   │       │   └── mobile.json          # Mobile layout (optional)
│       │   │       │
│       │   │       ├── v02BarByRegion/
│       │   │       │   └── visual.json
│       │   │       │
│       │   │       └── v03LineTrend/
│       │   │           └── visual.json
│       │   │
│       │   ├── pg02Detail/
│       │   │   ├── page.json
│       │   │   └── visuals/
│       │   │       └── ...
│       │   │
│       │   └── pg03Drillthrough/                # Drillthrough page
│       │       ├── page.json                    # type: "Drillthrough" in page.json
│       │       └── visuals/
│       │           └── ...
│       │
│       └── bookmarks/                           # Bookmarks (optional)
│           ├── bookmarks.json                   # Bookmark ordering and groups
│           └── bm01Default.bookmark.json        # Individual bookmark state
│
│
└── MyProject.SemanticModel/                     # ═══ SEMANTIC MODEL LAYER (TMDL) ═══
    │
    ├── definition.pbism                         # REQUIRED: Semantic model pointer
    │
    ├── .pbi/
    │   ├── localSettings.json                   # User settings (git-ignored)
    │   ├── editorSettings.json                  # Editor settings
    │   ├── cache.abf                            # Data cache (git-ignored, can be large)
    │   └── unappliedChanges.json                # Pending Power Query changes
    │
    ├── .platform                                # Fabric metadata
    │
    ├── diagramLayout.json                       # Model diagram view layout
    │
    ├── Copilot/                                 # AI instructions, verified answers
    ├── DAXQueries/                              # DAX query view tabs
    ├── TMDLScripts/                             # TMDL view script tabs
    │
    └── definition/                              # ═══ THE TMDL FOLDER ═══
        │
        ├── database.tmdl                        # Database-level properties
        ├── model.tmdl                           # Model definition + "ref table" ordering
        ├── relationships.tmdl                   # ALL relationships in one file
        ├── expressions.tmdl                     # Power Query parameters and expressions
        ├── dataSources.tmdl                     # Data source connections (if any)
        ├── functions.tmdl                       # DAX user-defined functions (if any)
        │
        ├── tables/
        │   ├── Sales.tmdl                       # Table + columns + measures + partitions
        │   ├── Product.tmdl
        │   ├── Calendar.tmdl
        │   └── _Measures.tmdl                   # Dedicated measures table
        │
        ├── roles/
        │   └── Admin.tmdl                       # RLS role definitions
        │
        ├── cultures/
        │   └── en-US.tmdl                       # Linguistic/translation metadata
        │
        └── perspectives/
            └── Perspective1.tmdl                # Perspective definitions
```

## File Dependencies

```
MyProject.pbip
  └─ references ─→ MyProject.Report/

definition.pbir
  └─ datasetReference.byPath ─→ ../MyProject.SemanticModel

pages.json
  └─ pageOrder[] ─→ page folder names (must match exactly)

page.json
  └─ name ─→ must match folder name
  └─ objects.background ─→ references RegisteredResources images

visual.json
  └─ visual.query.queryState ─→ references Entity (table) + Property (column/measure)
  └─ position {x,y,z,width,height} ─→ pixel coordinates on canvas

model.tmdl
  └─ "ref table TableName" ─→ must match tables/*.tmdl filenames
```

## Minimum Required Files for a Working Report

```
MyProject.pbip
MyProject.Report/
  definition.pbir
  definition/
    version.json
    report.json
    pages/
      pages.json
      pg01Overview/
        page.json
        visuals/
          v01Visual/
            visual.json
MyProject.SemanticModel/
  definition.pbism
  definition/
    database.tmdl
    model.tmdl
    tables/
      SomeTable.tmdl
```

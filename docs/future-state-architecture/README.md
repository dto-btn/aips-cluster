# Future-State Architecture — Starter Materials

This directory contains editable starter materials for a director-facing, future-state AI platform architecture presentation. These are **documentation and diagram sources only**; they do not modify any cluster deployment manifests or operational configuration.

---

## Contents

| File | Description |
|---|---|
| [`future-state-architecture.mmd`](./future-state-architecture.mmd) | High-level Mermaid flowchart of the proposed future-state architecture, including sovereignty zone labels |
| [`director-architecture-deck-outline.md`](./director-architecture-deck-outline.md) | Editable 10-slide outline for a director-facing PowerPoint presentation |
| [`sovereignty-assessment-template.md`](./sovereignty-assessment-template.md) | Per-service / per-vendor sovereignty assessment template |

---

## How to Render the Mermaid Diagram

The file `future-state-architecture.mmd` is a [Mermaid](https://mermaid.js.org/) diagram source. Use any of the following methods to render or export it:

### Option 1 — VS Code (recommended for iteration)

Install the [Mermaid Preview](https://marketplace.visualstudio.com/items?itemName=bierner.markdown-mermaid) or [Mermaid Editor](https://marketplace.visualstudio.com/items?itemName=tomoyukim.vscode-mermaid-editor) extension. Open the `.mmd` file and use the preview pane.

### Option 2 — Mermaid Live Editor (browser, no install)

1. Go to [mermaid.live](https://mermaid.live).
2. Paste the contents of `future-state-architecture.mmd` into the editor.
3. Export as SVG or PNG for use in PowerPoint or Visio.

### Option 3 — Mermaid CLI (command line, for CI or scripted export)

```bash
npm install -g @mermaid-js/mermaid-cli
mmdc -i future-state-architecture.mmd -o future-state-architecture.svg
mmdc -i future-state-architecture.mmd -o future-state-architecture.png
```

If you don't want a global install, run it via `npx` instead (no `npm install -g` required):

```bash
npx -y @mermaid-js/mermaid-cli -i future-state-architecture.mmd -o future-state-architecture.svg
```

### Option 4 — GitHub rendering

GitHub renders Mermaid diagrams inline in Markdown files. To preview on GitHub, wrap the diagram source in a fenced code block with the `mermaid` language tag inside a `.md` file.

---

## Using These Files as Inputs for Visio or PowerPoint

### PowerPoint

1. Render the Mermaid diagram to SVG or PNG using one of the methods above.
2. Insert the exported image into your slide.
3. Use `director-architecture-deck-outline.md` as a slide-by-slide writing guide — each section corresponds to one slide.
4. Replace all `[PLACEHOLDER]` items with confirmed, verified values before presenting.

### Visio

1. Export the Mermaid diagram to SVG.
2. In Visio, go to **Insert → Pictures** and select the SVG file (Visio 2019+ supports SVG import).
3. Alternatively, use the SVG as a reference to manually reconstruct the diagram in Visio using shapes and connectors, preserving the zone structure and sovereignty labels.

### draw.io / diagrams.net

[draw.io](https://app.diagrams.net/) can import SVG directly and allows full editing. This is a practical intermediate format if Visio is not available:

1. Export the Mermaid diagram to SVG.
2. In draw.io, select **Extras → Edit Diagram** or **File → Import From → SVG**.
3. Edit and export to Visio (`.vsdx`) or as an image.

---

## Important Caveats — Sovereignty

> **Sovereignty is service-specific and contract-specific. It requires independent validation. Do not state sovereignty claims as facts until they have been verified.**

The diagram and materials use the following two-dimensional sovereignty label format:

```
[Deployment / Data Jurisdiction] / [Provider Corporate Jurisdiction]

Examples:
  CA / CA      — Canadian deployment (validated) + Canadian-controlled provider (validated)
  CA / Foreign — Canadian deployment (target/validated) + Foreign-controlled provider
  TBC / Foreign — Deployment jurisdiction not yet confirmed + Foreign-controlled provider
  TBC / TBC    — Both dimensions require assessment
```

**"Canada-hosted" does not mean "fully sovereign."**  
An Azure service deployed in a Canadian region provides Canadian data residency for specific services and configurations — but Microsoft is a foreign-controlled provider subject to U.S. and other foreign jurisdiction. These are distinct considerations and are labeled separately in this architecture.

Each service must be assessed independently using the `sovereignty-assessment-template.md` file, covering:
- Data-at-rest and data-in-transit location
- Backup and DR jurisdiction
- Control-plane and telemetry location
- Vendor support access jurisdiction
- Provider corporate ownership and subcontractors
- Contractual and regulatory evidence

All sovereignty labels in this material are marked as **targets** or **subject to validation** unless explicitly noted otherwise.

---

## Key Architectural Decisions Captured

| Decision | Rationale |
|---|---|
| Core application platform moves to Azure (target: Canadian region) | Stability (managed SLAs, HA/DR) and scalability (elastic compute) differentiators |
| ThinkOn/TKO retained for Cohere North and Cohere LLMs | CA / CA sovereignty profile; Cohere North platform requirement |
| Microsoft Entra ID remains the identity provider | Existing integration; consistent across workload placement |
| AI gateway is the single integration point to all model providers | Policy enforcement, rate limiting, logging, and provider substitutability |
| Sovereignty assessed on two independent dimensions per service | Avoids conflating data residency with provider corporate jurisdiction |

---

## Contacts and Ownership

| Role | Contact |
|---|---|
| Architecture lead | [PLACEHOLDER] |
| Sovereignty assessment lead | [PLACEHOLDER] |
| Presentation owner | [PLACEHOLDER] |

---

*These materials are working drafts. All [PLACEHOLDER] items, target regions, SLA commitments, and sovereignty labels require confirmation before use in official or decision-making contexts.*

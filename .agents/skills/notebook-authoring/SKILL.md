---
name: notebook-authoring
description: >-
  Step-by-step workflow for researching, gathering visual assets, and generating
  self-contained laboratory notebooks following the Purdue ME597 IIoT standard,
  complete with dual-folder synchronization.
---

# Notebook Authoring Workflow

Follow this procedure whenever creating a new laboratory guide notebook (`.ipynb`) in this project.

## Step 1: Technical Background Research
1. Identify the core technical objectives and system architecture of the milestone.
2. Conduct targeted technical research using web search or official documentation (e.g., Raspberry Pi documentation, Linux man pages, Docker docs, Immich architecture specifications).
3. Extract precise CLI commands, package names, service unit names, configuration options, and kernel parameters.

## Step 2: Visual Asset Acquisition & Diagramming
1. Identify necessary visual aids:
   - **Hardware Photos:** Sourced exclusively from official manufacturer product pages (e.g., Western Digital, UGREEN, Raspberry Pi) into `assets/hardware/`.
   - **Architecture & Setup Figures:** Sourced exclusively from official documentation or official GitHub releases into `assets/reference/`.
2. **NEVER use AI image generation (`generate_image`)** to create synthetic images or diagrams.
3. If an architectural layout, network flow, or security authentication flow is needed, write a native Markdown **Mermaid** (`mermaid`) code block or clean Markdown table.

## Step 3: Self-Contained Notebook Assembly (Purdue ME597 Lab Style)
Construct the `.ipynb` file with the following standard cell structure:
1. **Header Cell:** Module title and step subtitle.
2. **Learning Goals:** 3–5 actionable, concrete bullet points (`🎯 Learning Goals`).
3. **Conceptual Background:** Deep, self-contained technical explanation.
   - **CRITICAL:** Do NOT include open-ended homework or investigation questions. All technical answers, rationale, and context must be fully articulated for the reader.
4. **Interleaved Reference Figures:** High-quality official reference images (`<img src="https://github.com/rd908213/Homelab-Development/blob/main/assets/...?raw=true">`) or native Mermaid diagrams embedded directly alongside the concepts they illustrate.
5. **Step-by-Step Hands-on Tasks with Inline Verification:**
   - Chronological tasks (`Task X.Y.Z`) with copy-pasteable commands and expected outputs.
   - **Inline Screenshot Submissions:** Every task requiring verification must have its dedicated **User Verification Screenshot Submission Box** placed immediately after the task instructions/output, rather than gathered at the end of the notebook.
6. **Module Checkpoint & Reflection:** Checkpoint checklist with a link to the subsequent module in sequence at the conclusion of the lab.

## Step 4: Dual-Folder Mirroring & Registry Update
1. Save the new notebook identically to both:
   - `modules/<category>/<notebook>.ipynb`
   - `modules_filled/<category>/<notebook>.ipynb`
2. Update the repository [README.md](file:///c:/Users/rd908/Documents/WebApps/Homelab%20Development/README.md) roadmap table with the new guide link, milestone description, and verification proof.
3. Update the total notebook counter in `README.md`.

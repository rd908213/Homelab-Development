# Rule: Inline Screenshot Submissions & Contextual Media Placement

## Context
Laboratory guides in this repository follow an interactive, step-by-step engineering curriculum (inspired by Purdue ME597 IIoT). Grouping all verification screenshot placeholders or reference figures into a monolithic section at the end of a notebook breaks the hands-on feedback loop, forces excessive scrolling, and decouples verification evidence from the command instructions.

## Invariant Directives

1. **Inline Verification After Associated Task:**
   - Place each **User Verification Screenshot Submission Box** immediately after the specific task, command sequence, or milestone test it validates.
   - Do NOT collect or bunch all screenshot submissions into an isolated block at the end of the notebook.

2. **Standard Task Cell Flow:**
   Every hands-on task must follow a continuous, contextual lifecycle:
   ```markdown
   ### Task X.Y.Z: [Task Name]
   [Brief objective and operational context]

   #### Step-by-Step Instructions & Commands
   [Commands with OS variations: PowerShell, macOS, Linux]
   [Expected output callout or blockquote]

   ---

   ### 🖼️ Milestone Submission [N]: [Submission Title]
   **Verification Objective:** [What evidence the user must capture]

   > ### 📷 User Verification Screenshot Submission Box
   > *Attach your milestone verification screenshot below (drag & drop an image or use a Markdown image link):*
   > 
   > ```markdown
   > <!-- Insert your screenshot here: -->
   > 
   > ```
   ```

3. **Interleaved Reference Figures:**
   - Architectural diagrams (Mermaid), hardware photos, and wiring schematics must be placed immediately adjacent to the relevant instructions, not clustered in an appendix.

4. **Module Checkpoint Remains at the Conclusion:**
   - The end of the notebook is reserved exclusively for the `## 🏁 Module Checkpoint & Reflection` checklist and the navigation link to the subsequent module in sequence.

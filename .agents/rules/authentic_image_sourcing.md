# Rule: Authentic Image Sourcing & Prohibition of Synthetic Images

## Context
Laboratory and homelab guides require factual, accurate technical references. Synthetic or AI-generated images often hallucinate incorrect port configurations, misleading diagrams, or non-existent UI options. Furthermore, creating dummy binary image files (e.g., placeholder PNGs) risks file corruption and broken rendering.

## Invariant Directives
1. **Never Generate Synthetic Images from Scratch:**
   - Strictly prohibit the use of AI image generation tools (such as `generate_image`) to create mock product photos, UI screenshots, or technical diagrams.
   - Never fabricate images with AI models when real hardware or software interfaces exist.

2. **Strictly Use Authentic Sourced Media:**
   - Only include imagery sourced directly from verified, authentic origins:
     - Official hardware manufacturer product listings (e.g., Western Digital, UGREEN, Raspberry Pi Foundation).
     - Official software documentation and official GitHub releases (e.g., Immich, Docker, Raspberry Pi OS).
   - Store all sourced images in `assets/hardware/` or `assets/reference/`.

3. **Prefer Native Diagrams (Mermaid / ASCII):**
   - When architectural layouts, network paths, container stacks, or authentication workflows must be illustrated, use native Markdown **Mermaid** (`mermaid`) diagrams or structured Markdown tables rather than generating static images.

4. **No Dummy Image Files & Inline Placement for User Milestones:**
   - Never create empty, hand-crafted, or placeholder binary image files on disk for user milestones.
   - Place each **User Verification Screenshot Submission Box** inline immediately after the specific task it verifies (see `inline_screenshot_submissions.md`), rather than grouping them at the end:
     ```markdown
     > ### 📷 User Verification Screenshot Submission Box
     > *Attach your milestone verification screenshot below (drag & drop an image or use a Markdown image link):*
     > 
     > ```markdown
     > <!-- Insert your screenshot here: -->
     > 
     > ```
     ```


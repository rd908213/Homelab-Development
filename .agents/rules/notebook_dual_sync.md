# Rule: Dual-Folder Notebook Synchronization

## Context
This repository maintains two mirrored notebook directory structures:
1. `modules/`: Reference template laboratory guides.
2. `modules_filled/`: User execution copies where real-world credentials, terminal output, and verification screenshots are recorded.

## Invariant Directives
1. **Always Mirror Changes:**
   - Any modification, refactoring, text update, or diagram addition applied to a notebook in `modules/<category>/<notebook>.ipynb` must simultaneously be applied to the corresponding notebook in `modules_filled/<category>/<notebook>.ipynb`.
   - Never leave `modules/` and `modules_filled/` out of sync after an editing task.

2. **Preserve User-Filled Content in `modules_filled/`:**
   - When propagating template or structural changes into `modules_filled/`, never overwrite or erase user-entered values (e.g., actual passwords, custom hostnames, user accounts like `pi` or `rd908213`, custom IP addresses, or completed screenshot image paths).
   - Only update instructions, conceptual explanations, reference figures, and template structure.

3. **Generator Script Alignment:**
   - Any automation or helper script (such as `generate_all_notebooks.py`) designed to build or update notebooks must be configured to output to or synchronize both `modules/` and `modules_filled/`.

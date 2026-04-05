---
name: review-project
description: Review and fix firmware project issues. This skill handles clearing build warnings, checking against best practices, and validating implementation against reference examples.
---

# Review Firmware Project

This skill outlines the process for reviewing and improving firmware code in this repository.

## Procedure

1.  **Initial Warning Fixes**:
    - Fix any warnings being raised by the build process (e.g., from `cargo check` or `cargo clippy`).

2.  **Review Process**:
    - Consider the best practices defined in the firmware agent's knowledge base.
    - Search in the `embassy/examples/rp` and `esp-hal/examples` folders for functionality similar to what is being implemented and compare approaches.
    - Search the full `embassy` and `esp-hal` source code for any relevant examples or patterns that can be applied to the implementation.

3.  **Refinement**:
    - Rank the review findings by importance and resolve them in order.

4.  **Final Checks**:
    - Run the build process again and fix any new warnings that may have been introduced.

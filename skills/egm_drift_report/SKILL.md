---
name: egm-drift-report
license: Apache-2.0
disable-model-invocation: true
description: >-
  Runs drift detection on all design documents to check for document and
  code consistency. Produces an actionable report with differences. Use when the
  user asks to detect drift or check for consistency across all design documents
  or READMEs in a directory.
---

# Detect Drift In Designs and Code

When running in Codex or ChatGPT Work, first read [Codex integration](../../docs/codex.md).
In Claude Code or Cowork, read `${CLAUDE_PLUGIN_ROOT}/docs/claude.md` first.

When the user asks to detect drift across all designs or a set of READMEs, the
goal is to identify all instances where the codebase has diverged or failed to
implement the specifications described by those designs or READMEs.

## Inputs

*   **`TargetDir`** (Optional): Directory containing design documents or READMEs
    to inspect. If omitted, the user is prompted to specify it.
*   **`OutputDir`** (Optional): Target directory where drift reports are saved.
    Defaults to `.egm/` at the workspace root.

1.  **Check Input**: Verify a directory containing design documents or READMEs
    is provided. If not provided, ask the user to specify the target directory.
2.  **List Documents**: List all files in the design directory.
3.  **Analyze**: For EACH design document:

    *   **Contextualize**: Read the design document. Identify the intent,
        proposed changes, and specific filenames/symbols mentioned.
    *   **Locate Source**: Find the corresponding source code files in the
        source tree. If paths aren't explicit, use code search or grep to
        locate the relevant classes, functions, or API endpoints mentioned in
        the doc. Prompt the user to add missing links to the document to make
        future detection cheaper.
    *   **Verify**: Read the current state of those source files.
    *   **Detect Drift**: Compare the design vs. reality. Look specifically for:
        *   Missing Features: Logic described in the doc but absent in the code.
        *   Incomplete Models: Dataclasses/Classes/Structs/Protos missing fields
            specified in the design.
        *   Logic Shortcuts: Hardcoded values where dynamic logic was planned.
        *   Architectural Divergence: Implementation using different
            patterns/libraries than proposed.
    *   **Filter**: Report ONLY the differences. Do not describe what is working
        or what matches. If the code matches the design perfectly, output "No
        drift detected."

4.  **Report**: Produce a single report listing all discrepancies found.

    *   Write or append the analysis for each file to `.egm/drift_report.md`
        (or `<OutputDir>/drift_report.md`, with backward-compatible support
        for `.egm/{YYYY-MM-DD}_drift_report.md` or `{YYYY-MM-DD}_DRIFT_REPORT.md`)
        using your file-writing tool. When checking for prior reports to append to,
        check `.egm/drift_report.md` -> `./drift_report.md` ->
        `./{YYYY-MM-DD}_DRIFT_REPORT.md`.
    *   **Directory Initialization & Transient Isolation**: Verify `.egm/`
        (or `<OutputDir>`) exists before writing. If writing to `.egm/`,
        ensure `.egm/.gitignore` exists containing `*` (with a newline)
        so drift reports are never tracked or staged in git.
    *   Strict Format:

        ```markdown
        ## <DESIGN_DOC_NAME>

        <Bulleted list of specific divergences found>
        <Or "No drift detected">
        ```

5.  **Guidance**

    -   Be rigorous. If a design doc mentions a specific object, field or file
        name and the code doesn't have it, that is a reportable drift. Double
        check for coverage. If there is some part of the logic that is not
        referenced in a design or README, this is a reportable drift.
    -   Read files liberally to ensure you are looking at the current state
        of the code, not relying on memory.
    -   Process the files sequentially to ensure the tool output buffer doesn't
        overflow.

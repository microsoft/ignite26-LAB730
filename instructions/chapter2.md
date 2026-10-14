# Chapter 2

**Time budget: about 40 minutes.** Complete Chapter 1 before starting. Your workspace should contain `.github/agents/radiology-assistant.agent.md` and the `write-report-simple` skill under `.github/skills/`.

In Chapter 1, a general-purpose language model produced more consistent reports with a skill, but it could still misidentify lesions and could not measure them. In this chapter, you connect **MedImageParse**, a foundation model that segments medical images from short text prompts, through an **MCP (Model Context Protocol) server**. Then you add measurement code that calculates patient side and diameters from the masks and image geometry.

!IMAGE[Chapter 2 pipeline: an axial brain MRI goes to MedImageParse through segment_lesions, the lesion mask goes to measure_lesions, and findings plus measurements go to write-report-simple.](https://raw.githubusercontent.com/microsoft/ignite26-LAB730/main/instructions/img/segmentation-pipeline.svg)

| | Segment (Part 1) | Measure (Part 2) |
|---|---|---|
| Implementation | MedImageParse foundation model | Python geometry on the mask |
| Input | Image path and text prompts | Mask path from Part 1 |
| Output | Mask file per prompt and a detected flag | Patient side and diameters in millimeters per lesion |
| Question | Is there a lesion? | Which side, and how big? |

> [!Knowledge] MCP tools expose functions to the agent. Their type hints describe the inputs and outputs; their descriptions explain how to use them. `segment_lesions` saves masks to disk and returns paths instead of putting a million mask values into the agent's context. The measurement tool reads those paths. The model produces masks, code calculates measurements, and the language model writes the report.

## Set up (~5 min)

1. In the VS Code PowerShell terminal, from +++C:\Lab+++, copy the Chapter 2 starter files into the workspace root:

   +++Copy-Item -Recurse -Force chapters\chapter2-foundation-models\starter\* .+++

2. In Explorer, inspect the additions at the workspace root:

   ```text
   .github/
     agents/radiology-assistant.agent.md     Your Chapter 1 agent; edit it below
     skills/
       write-report-segmentation/            Completed segmentation skill
         references/prompts.md               Brain lesion prompt sets
       write-report-measured/                Completed measurement skill
   mcp/medimageparse-server/
     server.py                               Two TODOs: segment and measure
     segmentation.py                         Model call and prompt catalog; read-only
     measurement.py                          Mask geometry calculations; read-only
     prompt_examples.json
   .vscode/
     mcp.json                                MCP server registration
   ```

   > [!Note] This is the only starter-file copy in Chapter 2. The endpoint configuration and credentials are pre-provisioned for Lab 730. Do not create or edit `.env`.

## Part 1: Segment

Start with a thin wrapper around the MedImageParse call.

### Wire the segmentation tool (~6 min)

3. In Explorer, open `mcp/medimageparse-server/server.py` and find the first TODO. Read the working `prompt_examples` wrapper above it: it delegates to a function in `segmentation.py` and uses that function's docstring as its tool description.

4. Replace the first TODO with:

   ```python
   @mcp.tool(description=segmentation.segment.__doc__)
   def segment_lesions(file_path: str, prompts: list[str]) -> dict[str, dict]:
       return segmentation.segment(file_path, prompts)
   ```

   > [!Knowledge] The `@mcp.tool(...)` decorator registers the function; without it, the agent cannot see the tool. MCP uses `file_path: str` and `prompts: list[str]` to build the required input schema. Reusing `segmentation.segment.__doc__` gives the agent the prompt-format guidance and return contract from the underlying function.

5. Save the file. If the server does not reload automatically, press **Ctrl+Shift+P** and run +++MCP: Reload Servers+++.

6. Open the chat tool picker using the configuration icon next to the model selector. Confirm that **segment_lesions** appears under **medimageparse**, alongside **prompt_examples**. VS Code may take a moment to start the server and discover its tools after reading `.vscode/mcp.json`.

   > [+Hint] The MCP tool does not appear
   >
   > Select **View > Output**, then choose **MCP Servers > medimageparse** in the dropdown. A Python traceback can identify a syntax error or missing import. If the server starts cleanly, check that the wrapper has `@mcp.tool(...)` and that `.vscode/mcp.json` is present at the workspace root.
   >
   > For `Address already in use`, run **MCP: Reload Servers** or restart VS Code. For a missing Python package, ask a proctor to check the prepared environment.

7. Open the workspace-root `.github/agents/radiology-assistant.agent.md`. Add +++medimageparse/*+++ to its `tools:` list:

   ```yaml
   tools: [execute, read, agent, browser, edit, search, web, todo, medimageparse/*]
   ```

   > [!Knowledge] This entry lets the agent use `segment_lesions` now and `measure_lesions` after you register it in Part 2.

8. In the same agent file, replace the Chapter 1 report-routing line with the following rule, then save:

   ```markdown
   - For brain MRI reports, use the most capable report skill whose tools are available: `write-report-measured` (segmentation and measurement tools), else `write-report-segmentation` (segmentation tool only), else `write-report-simple`.
   ```

   > [!Note] No further agent-file edit is needed for Part 2. This rule already gives the measured skill priority when both tools are available.

### Skim the segmentation skill (~2 min)

9. In Explorer, open `.github/skills/write-report-segmentation/SKILL.md`. Read the frontmatter `description`, which tells the agent when to select this brain MRI skill based on tool availability.

10. Read the skill's two-step workflow. It calls `mcp_medimageparse_segment_lesions(file_path, prompts)`, reads which targets were detected, then passes those findings to Chapter 1's `write-report-simple` skill and returns its output.

    > [!Knowledge] The report-writing skill remains responsible for report structure and style. Its rule against model attribution keeps masks, prompts, and model names out of the report text.

11. Open `.github/skills/write-report-segmentation/references/prompts.md`. Notice how the prompt sets depend on MRI sequence and use short target names such as `enhancing tumor` and `edema`, not sentences.

    > [!Note] VS Code prefixes MCP tool IDs with `mcp_<server>_` and can shorten long server names. Here, the expected ID is `mcp_medimageparse_segment_lesions`. If the tool picker shows a different alias, use the alias shown there.

### Try segmentation (~6 min)

12. Start a new Copilot Chat session and select **Radiology Assistant** in the agent picker.

    > [+Hint] Radiology Assistant is missing
    >
    > Confirm that `.github/agents/radiology-assistant.agent.md` is at the workspace root, not only in a chapter directory. Start a fresh chat after copying or editing it.

13. Send this prompt:

    +++Please write a radiology report for samples/SYN0417_003.png.+++

14. Confirm the VS Code permission prompts for the lab's third-party tool calls.

    > [!Note] The GPU endpoint can take 30 to 90 seconds to wake up after being idle. Later calls should be faster.

    > [+Hint] Credentials are missing or the endpoint returns 401 or 403
    >
    > Ask a proctor to check the pre-provisioned endpoint configuration and credentials. Do not create or edit `.env`. The MCP server reads its configuration on launch, so run **MCP: Reload Servers** after the proctor corrects it.

15. Watch the chat trace. Check for one `mcp_medimageparse_segment_lesions` call, followed by `write-report-simple` and the final report. Match every lesion component mentioned in the report to a prompt whose result was `detected: true`.

    > [+Hint] The agent writes a report without using segmentation
    >
    > First check that `segment_lesions` is registered in the tool picker and that the agent's `tools:` list includes `medimageparse/*`. Type `/` in chat to check skill registration; each skill directory must exactly match its frontmatter `name:`.
    >
    > If skills are registered but skipped, make their descriptions distinguish tool availability: both tools for `write-report-measured`, segmentation only for `write-report-segmentation`, and no vision tool for `write-report-simple`. Make the agent's rule require a report-writing skill, and make its workflow select that skill, invoke it, and return its output unchanged.
    >
    > Save changes and start a fresh chat. If the agent still skips skills, try a larger model available to the lab account through the agent's `model:` setting. Open `docs/troubleshooting.md` in Explorer for the full skill-selection guidance.

    > [+Hint] Test the segmentation call without MCP
    >
    > In the PowerShell terminal, run +++uv run python mcp/medimageparse-server/segmentation.py samples/SYN0417_003.png+++.
    >
    > Add `--help` for options, including custom `--prompts`. If the script works but the tool is missing, inspect the `@mcp.tool(...)` decorator and `.vscode/mcp.json` registration. This isolates server registration from the model call.

16. Read the `<report>` block. It should contain no masks, prompts, or MedImageParse attribution. Lesion size should still be qualitative, such as "large" or "moderate", because segmentation has not supplied a measurement.

    > [!Knowledge] The Chapter 1 report-writing skill is unchanged. You added a segmentation tool, enabled it for the agent, and selected a skill that uses it. Adding this capability did not require rewriting the report formatter.

## Part 2: Measure

A mask identifies lesion pixels. Converting those pixels to patient side and millimeters requires pixel spacing, connected-component analysis, and geometry. These calculations belong in deterministic, unit-testable code.

### Why measurements come from code (~1 min)

17. Review the two measurements used to track an enhancing brain lesion on axial images: its longest diameter and the longest diameter perpendicular to it. Comparing these values across scans helps describe growth, stability, or shrinkage.

    > [!Knowledge] Millimeters are not visible in the pixels. Each sample has a JSON sidecar containing pixel spacing, such as 0.9 mm per pixel, which the measurement tool reads. Brain MRI also uses radiological display convention: the patient's right is on the image's left. The tool applies that rule in code instead of asking the language model to infer size or side.

### Wire the measurement tool (~3 min)

18. In Explorer, reopen `mcp/medimageparse-server/server.py` and find the second TODO.

19. Replace it with:

    ```python
    @mcp.tool(description=measurement.measure.__doc__)
    def measure_lesions(mask_path: str) -> dict:
        return measurement.measure(mask_path)
    ```

    > [!Knowledge] This uses the same decorator, type hints, and docstring pattern as the segmentation wrapper. The module is now `measurement`, and the input is the `mask_path` returned by segmentation. The tools pass file paths between them instead of passing pixel arrays through chat.

20. Save the file, press **Ctrl+Shift+P**, and run +++MCP: Reload Servers+++.

21. Confirm that **measure_lesions** appears alongside **segment_lesions** in the chat tool picker. The agent already has access through `medimageparse/*`.

    > [+Hint] The measurement tool does not appear
    >
    > Open **View > Output > MCP Servers > medimageparse** and check for an import error or Python traceback. Confirm the second wrapper has `@mcp.tool(...)`, then reload the server.

### Skim the measured-report skill (~2 min)

22. Open `.github/skills/write-report-measured/SKILL.md` in Explorer. Follow the sequence: call `segment_lesions`, pass the primary target's `mask_path` to `measure_lesions`, then pass the result to `write-report-simple` as `measurements`.

    > [!Knowledge] The report copies these measurements verbatim. The language model must not compute, round, or convert them. If its interpretation of the image disagrees with the tool's patient-side result, the tool result takes priority.

### Try measurement (~5 min)

23. In Copilot Chat with **Radiology Assistant**, send the same prompt:

    +++Please write a radiology report for samples/SYN0417_003.png.+++

24. Watch for `write-report-measured` to run and call `mcp_medimageparse_segment_lesions`, followed by `mcp_medimageparse_measure_lesions`. The skill description and agent routing rule should select the measured skill now that both tools are available.

    > [+Hint] Test the measurement call without MCP
    >
    > After segmentation has produced the mask, run +++uv run python mcp/medimageparse-server/measurement.py outputs/masks/SYN0417_003__enhancing-tumor.png+++ in PowerShell. Add `--help` to see options. If this succeeds but MCP does not, check tool registration and the server logs.

25. Compare the tool's returned patient side and diameters with the final report. Confirm that it quotes exactly those values.

## Optional: Extend the tools

26. Ask the agent to call +++prompt_examples+++ and choose prompts for a different target, such as `edema` on a FLAIR image.

27. Pass those targets to the report skill as `extra_prompts`. Check which targets produce masks and which return empty results. MedImageParse accepts target names at call time.

28. Ask the agent to overlay a mask from +++outputs/masks/+++ on its source image with matplotlib and save the result as a PNG.

29. Open the saved PNG in Explorer and inspect what the model actually segmented.

30. Consider a 3D extension. MedImageParse 3D segments whole MRI or CT volumes in NIfTI format. Such an extension would replace the segmentation implementation and change `measure_lesions` to report volume in milliliters, while retaining the MCP server, skill, and agent pattern.

## Check your work

- [] `segment_lesions` and `measure_lesions` are wired in `mcp/medimageparse-server/server.py` and visible in the tool picker.
- [] The agent's `tools:` list includes `medimageparse/*`, and its report-routing rule selects skills by tool availability.
- [] The segmentation report's lesion components each trace to a detected mask.
- [] The measured report copies patient side and diameters from `measure_lesions`.

> [+Hint] You need a completed example
>
> Open `chapters/chapter2-foundation-models/answer-key/` in Explorer and compare its completed files with your workspace-root copies.

Next, add patient context. Chapter 3 uses a fictional chart to connect the diagnosis, surgery, treatment dates, and prior measurements to the current image.

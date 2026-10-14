# Chapter 3

**Time budget: about 10 minutes.** Complete Chapter 2 before starting, with the `medimageparse` MCP server and both `segment_lesions` and `measure_lesions` working.

A brain tumor follow-up report needs more than the current image: diagnosis, surgery, treatment dates, and prior measurements affect its interpretation. You will copy two completed skills and a small data folder, then compare reports on the same image with and without that context. There are no required code changes in this chapter.

## Skim the patient record (~2 min)

1. In Explorer, open `chapters/chapter3-knowledge-sources/data/patients/SYN0417.json`. The chart uses flat fields rather than a FHIR schema. Read these fields:

   - `diagnosis`: glioblastoma, an aggressive primary brain tumor, in the left frontal lobe.
   - `procedures`: subtotal resection on 2026-05-19. Enhancing tumor was deliberately left along the back wall of the cavity to protect language and motor areas.
   - `treatments`: chemoradiation finished on 2026-07-24; adjuvant chemotherapy started on 2026-08-24.
   - `expected_post_treatment_findings`: the resection cavity and post-treatment change should be described as expected, not new.
   - `imaging_studies`: studies `_000` through `_002`. Only `_002` has a retained image and full report, including the prior measurement.
   - `pending_orders`: the current study, `_003`, ordered eight weeks after completion of chemoradiation.
   - `last_record_entry_note`: the chart stops at the `_003` order. It contains no later events.

2. Identify `_003` as the current study and `_002`, four weeks earlier, as the most recent report available for comparison.

   > [!Knowledge] For a few months after brain radiation, the treated area can swell and enhance more on MRI without tumor growth. This is called pseudoprogression. An increase eight weeks after treatment raises a different interpretation than the same increase six months later. The chart supplies the timing needed to discuss treatment effect versus progression; the current pixels alone do not supply that history.

   !IMAGE[Timeline for SYN0417: diagnosis MRI, surgery, chemoradiation, the prior MRI at 25 x 20 mm, and the current MRI eight weeks after radiation. Patient context lets the report compare measurements and discuss treatment effect.](https://raw.githubusercontent.com/microsoft/ignite26-LAB730/main/instructions/img/treatment-timeline.svg)

## Copy the skills and data (~3 min)

3. In the VS Code PowerShell terminal, from +++C:\Lab+++, copy the completed history-lookup skill:

   +++Copy-Item -Recurse chapters\chapter3-knowledge-sources\answer-key\.github\skills\lookup-patient-history .github\skills\+++

4. Copy the completed context-aware report skill:

   +++Copy-Item -Recurse chapters\chapter3-knowledge-sources\answer-key\.github\skills\write-report-with-ehr .github\skills\+++

5. Create the workspace-root data directories:

   +++New-Item -ItemType Directory -Force data\patients, data\reports, data\images+++

6. Copy the synthetic chart:

   +++Copy-Item chapters\chapter3-knowledge-sources\data\patients\SYN0417.json data\patients\+++

7. Copy the prior report:

   +++Copy-Item chapters\chapter3-knowledge-sources\data\reports\SYN0417_002.md data\reports\+++

8. Copy the two patient images and their sidecars:

   +++Copy-Item chapters\chapter3-knowledge-sources\data\images\SYN0417_* data\images\+++

9. In Explorer, open `.github/skills/lookup-patient-history/SKILL.md` and read how it obtains context. It infers the patient ID from the image filename, reads the JSON chart, and includes the prior report text. It makes no model-tool calls.

   > [!Knowledge] In a real EHR integration, this skill is where tools would query Epic or another record system. The local JSON lookup represents that data-retrieval role in Lab 730.

10. Open `.github/skills/write-report-with-ehr/SKILL.md`. Follow its workflow: invoke `lookup-patient-history`, call `segment_lesions` and `measure_lesions` once each on the current image, and write a comparison-aware report.

    > [!Note] No MCP changes are needed. The report skill uses the two tools you registered in Chapter 2.

11. Press **Ctrl+Shift+P** and run +++Developer: Reload Window+++ so VS Code registers the new skills.

    > [+Hint] A copied skill is not registered
    >
    > Type `/` in Copilot Chat and check for `lookup-patient-history` and `write-report-with-ehr`. Each skill directory must match the `name:` field in its `SKILL.md` frontmatter exactly. Confirm the copies are under the workspace-root `.github/skills/`, then start a fresh chat.

## Run the agent (~3 min)

12. Start a new Copilot Chat session and select **Radiology Assistant**.

13. Send this prompt:

    +++Use the write-report-with-ehr skill to write a report for data/images/SYN0417_003.png.+++

14. Watch the trace for `lookup-patient-history`. Its structured `PATIENT RECORD` block should include the diagnosis, treatments, and the `_002` report text.

15. Check that the trace contains one `mcp_medimageparse_segment_lesions` call on `_003` and one `mcp_medimageparse_measure_lesions` call on the enhancing-tumor mask.

    > [!Knowledge] The prior measurement comes from the prior report as text. The agent should not segment the prior image again for this comparison.

    > [+Hint] The agent skips the requested skill
    >
   > Confirm that `write-report-with-ehr` appears under `/`. If it is registered, check that its `description:` states when to use it and that the agent is required to use report-writing skills instead of drafting independently. Save any changes and start a new chat; the agent does not pick up file edits mid-conversation.
    >
    > If tools are missing, open **View > Output > MCP Servers > medimageparse**, check for syntax or import errors, and confirm that both wrappers have `@mcp.tool(...)`. Run **MCP: Reload Servers** after correcting registration. Open `docs/troubleshooting.md` in Explorer for additional guidance.

16. Inspect the two trace sections before the final report:

    - `## Prior information used`: which prior information the agent used.
    - `## Clinical details used in the report`: each report phrase paired with its EHR field or tool call.

    Confirm that the `<confidence>` and `<report>` blocks appear last.

17. Read the report and check all of the following:

    - It includes `COMPARISON: MRI brain with and without contrast, 2026-08-21, SYN0417_002.`
    - It quotes the current measurement beside the prior measurement, such as "now [current dimensions] mm, previously 25 x 20 mm".
    - It describes the resection cavity as an expected post-surgical finding.
    - It discusses any increase against the treatment timeline, considering treatment-related change versus progression rather than declaring progression outright.

## Compare without the chart (~2 min)

18. Send this prompt to run the Chapter 2 skill on the same image:

    +++Use the write-report-measured skill to write a report for data/images/SYN0417_003.png.+++

19. Compare the two reports side by side:

    - Chapter 3 anchors **COMPARISON** to `_002`; the Chapter 2-style report has no chart-derived comparison.
    - Chapter 3 places current and prior measurements together; Chapter 2 has only the current measurement and cannot describe interval growth from it alone.
    - Chapter 3 treats the resection cavity as expected; Chapter 2 may describe it as cystic or necrotic.
    - Chapter 3 starts **IMPRESSION** with interval change and interprets it against the treatment timeline; Chapter 2 starts with current findings.
    - Chapter 3's **INDICATION** reflects the order eight weeks after chemoradiation; Chapter 2 has no order-derived indication.

20. Compare the tool traces. Both report skills use the same segmentation and measurement tools on the same image. The added patient context, not a different model, accounts for the intended difference in the reports.

## Optional: Extend the knowledge source

Choose one exercise:

21. **Move the clock.** Edit your workspace copy of `data/patients/SYN0417.json` so the `_003` order says "6 months after completion of chemoradiation", and shift the `treatments` dates to match. Rerun the context-aware report. Does its impression stop hedging about treatment-related change? Should it?

22. **Add a second patient.** Create a fictional `data/patients/<id>.json`, add an image named `<id>_<index>.png` and its sidecar, then run the report skill. Confirm that the lookup uses the filename rather than a hardcoded patient ID.

23. **Calculate interval change in code.** Add a `compare_measurements` MCP tool that returns the percent change between two bidimensional measurements, and have `write-report-with-ehr` cite its output. The report currently quotes both measurements without calculating change; arithmetic used for decisions belongs in code.

24. **Use a FHIR schema.** Refactor `lookup-patient-history` to consume a minimal FHIR bundle with `Patient`, `Condition`, `Procedure`, `MedicationStatement`, `ImagingStudy`, and `DiagnosticReport` instead of the flat JSON. Keep the report skill unchanged while replacing the history lookup's data source.

## Check your work

- [] You have read `SYN0417.json` and identified the fields that should inform the report.
- [] `lookup-patient-history` and `write-report-with-ehr` are under the workspace-root `.github/skills/`.
- [] The workspace contains `data/patients/SYN0417.json`, `data/reports/SYN0417_002.md`, and the SYN0417 PNG images and sidecars under `data/images/`.
- [] The `_003` report identifies `_002` in **COMPARISON**, places current and prior measurements side by side, treats the resection cavity as expected, and discusses change against the treatment timeline.
- [] You have compared the context-aware report with the Chapter 2 skill on the same image.

> [+Hint] You need a completed example
>
> Open `chapters/chapter3-knowledge-sources/answer-key/` in Explorer to compare the completed files with your workspace-root copies.

You now have an agent, report-writing skills, segmentation and measurement tools, and patient context. The same pattern can use a FHIR or HL7 client instead of local charts, different prompts for MedImageParse targets across CT, MRI, X-ray, ultrasound, and pathology, or MedImageParse 3D for whole volumes. Keep deterministic calculations such as measurements, interval change, and volumetry in tools; use skills to direct the task and the agent to select them.

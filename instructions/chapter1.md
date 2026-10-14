# Chapter 1

**Time budget: about 20 minutes.** Complete **Set up** before starting.

You will build a minimal **Radiology Assistant** agent that drafts a brain MRI report with a general-purpose language model. First, run it without a skill. Then add a reusable instruction packet that sets the report structure, phrasing, confidence buckets, and a rule against invented measurements.

> [!Knowledge] An agent file sets the assistant's behavior and constraints. A skill supplies instructions for a particular task and can load supporting files when needed. This chapter uses no specialized medical foundation model. Better report structure does not make a general-purpose model clinically reliable.

## Set up the agent (~1 min)

1. In the VS Code terminal, from the repository root, create the agent directory:

   +++New-Item -ItemType Directory -Force .github\agents+++

2. Copy the starter agent into that directory:

   +++Copy-Item chapters\chapter1-simple-agent\starter\.github\agents\radiology-assistant.agent.md .github\agents\radiology-assistant.agent.md+++

3. In Explorer, open `.github/agents/radiology-assistant.agent.md` at the workspace root. Read its instructions and constraints, and notice how little radiological guidance it contains.

   > [!Note] Make your changes in this workspace-root file throughout the chapter. Do not edit the copy under `chapters/chapter1-simple-agent/starter/`. You will add a rule to the workspace-root copy when you activate the skill.

## See the gap (~5 min)

4. Open **Copilot Chat**. In the agent picker, select **Radiology Assistant**.

   !IMAGE[The Radiology Assistant agent in the Copilot Chat agent picker](https://raw.githubusercontent.com/microsoft/ignite26-LAB730/main/instructions/img/agent.png)

   > [+Hint] Radiology Assistant is missing from the picker
   >
   > Confirm that the file is at `.github/agents/radiology-assistant.agent.md` in the workspace root, not only inside the chapter's starter directory. Start a new Copilot Chat session after copying it.

5. Send this prompt in Copilot Chat:

   +++Please write a radiology report for <placeholder>.+++

   > [!Note] This agent accepts file paths only. Do not drag the image into chat or attach it with the paperclip; the agent will refuse an attached image.

6. Repeat the request with other available image paths until you have tried two or three different images. Look in `samples/` and `samples/extra/` in Explorer to choose the images.

7. Compare the reports. Check whether the section names change, hedging is inconsistent, and confidence is missing as a separate value. Then inspect the sizes and sides:

   - Does a report quote a lesion size such as "3.4 x 2.8 cm"? From this image alone, the agent has pixels, but no pixel spacing or lesion mask to support a measurement.
   - Does the report place the lesion on the correct patient side?

   > [!Knowledge] Brain MRI uses radiological display convention: the patient's right is on the image's left. Laterality describes the patient's side, not the side of the screen. A plausible-looking size is not a measurement unless it comes from a tool using the image geometry and lesion mask.

8. Keep the first report, for `<placeholder>`, visible. You will compare it with another report on the same image after activating the skill.

## Build the skill (~10 min)

9. In the terminal, create the workspace-root skills directory:

   +++New-Item -ItemType Directory -Force .github\skills+++

10. Copy the starter skill into it:

    +++Copy-Item -Recurse chapters\chapter1-simple-agent\starter\.github\skills\write-report-simple .github\skills\+++

11. In Explorer, open and skim all three files under `.github/skills/write-report-simple/`:

    ```text
    SKILL.md                       Skill body with TODOs to complete
    template/mri-brain-template.md  Completed report structure
    references/phrase-bank.md      Completed example wordings
    ```

    > [!Note] The directory name and the `name:` value in the skill's frontmatter must both be exactly `write-report-simple`. Hyphens and underscores are not interchangeable.

12. Open `chapters/chapter1-simple-agent/suggested-changes.md` alongside `.github/skills/write-report-simple/SKILL.md`. For each TODO below, read its teaching note and inspect the matching placeholder in `SKILL.md`. Use the text under the matching anchor in `suggested-changes.md`, or write your own.

13. Fill the **`description`** TODO using the `description` suggestion.

    > [!Knowledge] The agent matches this frontmatter string against your request to decide whether to invoke the skill. State the specific task it handles; a broad category such as "radiology" gives the agent little basis for selecting it.

14. Fill the **`argument-hint`** TODO using the `argument-hint` suggestion.

    > [!Knowledge] This declares optional inputs that other agents or later chapters can pass. You will not supply them in Chapter 1. Declaring `findings` and `measurements` now gives the skill a defined interface for later segmentation and measurement tool outputs.

15. Fill the **`one-liner`** TODO using the `one-liner` suggestion.

    > [!Knowledge] This sentence describes the skill to a person reading the file. Its audience differs from the frontmatter `description`, which helps the agent choose a skill.

16. Fill the **`inputs`** TODO using the `inputs` suggestion.

    > [!Knowledge] Naming an input in `argument-hint` does not explain how to use it. The body must state the input priority and treat tool-supplied `measurements` as authoritative, rather than letting the agent guess sizes from the image.

17. Fill the **`how-to-use`** TODO using the `how-to-use` suggestion.

    > [!Knowledge] The report template and phrase bank are already complete. Reference `template/mri-brain-template.md` and `references/phrase-bank.md` in the skill body so the agent reads them when needed.

18. Fill the **`style-rules`** TODO using the `style-rules` suggestion.

    > [!Knowledge] Explicit wording rules reduce variation in hedging and sentence structure. Include the two brain-lesion constraints: numbers come from tools only, and laterality refers to the patient's side.

19. Fill the **`output-format`** TODO using the `output-format` suggestion, then save `SKILL.md`.

    > [!Knowledge] Machine-readable output needs a literal template, not just a description of one. Exact `<confidence>` and `<report>` tags let downstream steps extract confidence and report text without guessing where they begin and end.

## Activate the skill (~2 min)

20. Open the workspace-root `.github/agents/radiology-assistant.agent.md`. Find its `<rules>` block and add the "use available skills" rule as the first bullet. Use the wording at the `agent-rule` anchor in `chapters/chapter1-simple-agent/suggested-changes.md`, then save the agent file.

    > [!Knowledge] Creating skill files gives the agent instructions it can discover. The agent rule tells it to use those skills when handling a request; the skill's description helps it select the relevant one.

21. Start a new Copilot Chat session, select **Radiology Assistant**, and rerun the original prompt on the same image:

    +++Please write a radiology report for <placeholder>.+++

22. Compare this report with the first one you kept visible. Check for all of the following:

    - EXAM, INDICATION, COMPARISON, TECHNIQUE, FINDINGS, and IMPRESSION appear in that order, with a blank line between sections and text on the first four header lines.
    - FINDINGS is one prose paragraph. IMPRESSION uses numbered statements, one per line.
    - The response opens with `<confidence>...</confidence>` on its own line, followed by `<report>...</report>`.
    - Lesion size is qualitative, such as "large" or "moderate", rather than invented millimeters.
    - Some wording comes from `references/phrase-bank.md` in your skill directory.

    > [+Hint] The report still looks like the first attempt
    >
    > Type `/` in Copilot Chat and check that `write-report-simple` appears in the registered skills. If it is missing, check the directory name, frontmatter `name:`, and that all three files are in `.github/skills/write-report-simple/`.
    >
    > If the skill is listed but the agent skips it, make the `description:` specific about when this skill applies: drafting a report when no vision model tool is available. Confirm that the skill-use rule is inside the agent's `<rules>` block. Make that rule explicit that reports must go through the report-writing skill, rather than being drafted independently. Any workflow instructions should tell the agent to select the skill, invoke it, and return its output unchanged.
    >
    > Save your edits and start a fresh chat so the agent reads the updated files. If it still skips the skill, try a larger model available to your lab account through the agent's `model:` setting. The repository's `docs/troubleshooting.md` has more detail on skill registration and selection.

23. Run the agent on a few different images and compare each report with your own reading of the image. Where is it right, and where is it wrong? Check whether it avoids invented sizes while still being unable to measure the lesion.

    > [!Note] The skill is intended to improve structure, phrasing, and reporting discipline. Its confidence buckets are self-reported, not proof of clinical accuracy. It can prohibit invented measurements, but it cannot supply the missing measurements. Chapter 2 adds tools for that work.

## Optional: Extend the phrase bank

24. Choose a finding the phrase bank does not cover well, such as a specific white matter lesion pattern or an extra-axial collection. Add two or three example phrasings for positive, possible, and negative findings under a new section in `.github/skills/write-report-simple/references/phrase-bank.md`, then save it.

25. Start a fresh chat and rerun the agent on a relevant image. Check whether it borrows your new wording.

## Check your work

- [] The workspace contains `.github/skills/write-report-simple/SKILL.md`, `template/mri-brain-template.md`, and `references/phrase-bank.md` within that skill directory.
- [] All seven `SKILL.md` TODOs are complete: `description`, `argument-hint`, `one-liner`, `inputs`, `how-to-use`, `style-rules`, and `output-format`.
- [] The workspace-root `.github/agents/radiology-assistant.agent.md` includes the "use available skills" rule.
- [] You have before-and-after reports for the same image to compare.

> [+Hint] You need a completed example
>
> Open `chapters/chapter1-simple-agent/answer-key/` in Explorer for completed versions of the chapter files. Compare them with your workspace-root copies.

Next, connect MedImageParse through an MCP server. Segmentation produces lesion masks; measurement code uses those masks and image geometry to calculate sizes. Reported findings and dimensions can then be traced to tool outputs instead of guesses from pixels.

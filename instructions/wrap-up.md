# Wrap-up

## What you built

By the end of the three chapters in Lab 730, your assistant combines an agent file, report-writing skills, segmentation and measurement tools, and patient context. Check the results:

- [] A structured brain MRI report with consistent sections, confidence and report tags, and no invented measurements. You compared the same image before and after adding the Chapter 1 skill.
- [] A report whose lesion findings trace to segmentation masks, with patient-side and size information supplied by measurement code through MCP tools. The agent has access to the tools and routes reports through the appropriate skill.
- [] A context-aware follow-up report that uses patient history and a prior report. It identifies the comparison study, puts current and prior measurements side by side, accounts for expected postoperative findings, and relates interval change to the treatment timeline. You compared it with a report on the same image without that context.

The chapter folders contain `answer-key/` directories for comparing your files with completed examples.

> [!Knowledge] Use MCP tools for capabilities that need model calls or deterministic code, skills for task-specific instructions, and agent files to select and combine them. Patient history changes how findings are interpreted; it does not replace the image evidence or measurements.

## Next steps

- **Connect a different knowledge source.** The Chapter 3 pattern can replace fictional patient JSON with a FHIR or HL7 client. Keep the history-lookup skill's role while changing how it retrieves data.
- **Explore other targets and modalities.** MedImageParse supports text-prompted segmentation across modalities such as CT, MRI, X-ray, ultrasound, and pathology; MedImageParse 3D works with whole volumes. Use the MCP server pattern with the model and prompts appropriate to the task.
- **Put more calculations behind tools.** Measurement, interval-change calculations, and volumetry belong in deterministic code behind an MCP tool, not in a prompt that asks the language model to invent numbers.

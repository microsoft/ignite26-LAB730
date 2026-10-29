# Overview

In **Lab 730**, a Microsoft Ignite 2026 hands-on lab, you build a radiology assistant in VS Code with GitHub Copilot. You start with a general-purpose language model, then add medical image segmentation, measurements, and patient context.

| Chapter | What you'll build | Time |
|---|---|---|
| 1: Simple agent and skill | An agent with a reusable skill that drafts structured brain MRI reports, so you can see what instructions improve and where the model still falls short. | About 12 minutes |
| 2: Foundation models and tools | An MCP (Model Context Protocol) server that lets the agent call MedImageParse for lesion segmentation and use code to turn masks into measurements. | About 20 minutes |
| 3: Patient context | A report grounded in a synthetic patient history and prior report, with a comparison that describes interval change. | About 10 minutes |

!IMAGE[Workshop overview: three chapters — agent and skill, MedImageParse segmentation via MCP, and patient context — take a brain MRI to a measured, radiologist-reviewed draft report.](https://raw.githubusercontent.com/microsoft/LAB730-ignite-2026-healthcare-agents/main/docs/brain-lesion-segmentation-workshop.svg)

> [!Alert] The data and patient records are synthetic or de-identified and are not for clinical use. MedImageParse is provided for research and model development; it is not intended for deployment in clinical settings as-is. These exercises do not validate the reports for patient care.

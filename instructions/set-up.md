# Set up

The tools, repository, and Python dependencies are already prepared on your Windows 11 VM. Use the VS Code PowerShell terminal for the commands in this lab.

1. In VS Code, select **File > Open Folder** and open the Lab 730 repository folder: +++C:\Lab+++.

   Confirm that Explorer shows `README.md`, `chapters/`, `samples/`, and `workshop/` at the workspace root.

2. Open **Copilot Chat** and send +++Reply with "Ready for Lab 730."+++. Confirm that Copilot responds before continuing.

3. Select **Terminal > New Terminal**. Confirm that the terminal uses **PowerShell** and its current folder is the repository root you opened.

4. Run +++uv run workshop/check_env.py+++. It checks the Python packages, then sends a sample scan to the segmentation endpoint, which can take a few seconds.

   The last line should read:

   ```text
   Workshop env OK — Python 3.11; endpoint OK (mip3d-mri-brain-lesion)
   ```

   > [+Hint] The environment check does not run or fails
   >
   > If PowerShell does not recognize `uv`, close the terminal and open a new one so it picks up the prepared Windows PATH. If the script cannot be found, confirm that the terminal is at the repository root, where `workshop/` appears in Explorer. If the check stops with a package, sample, connection, or HTTP error, ask a lab proctor before continuing.

You are ready when the environment check succeeds and Copilot Chat responds.

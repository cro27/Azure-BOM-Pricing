# Azure BOM Pricing

A reusable [Agent Skill](https://agentskills.io/) that turns an Azure architecture
diagram, bill of materials (BOM), Bicep, Azure Resource Manager template, Terraform,
spreadsheet, or written description into an auditable public pay-as-you-go cost
estimate. It supports detailed estimates and separately confirmed Small, Medium, and
Large scenarios, uses current public Azure pricing, records assumptions and price
lineage, and produces an Excel workbook. It does not deploy resources or provide
private agreement pricing.

This repository packages the skill's workflow, supporting guidance, and templates in a
portable `SKILL.md` format, for use with GitHub Copilot, Microsoft Copilot Cowork, and
other Agent Skills-compatible harnesses.

The skill loads only when the AI harness decides it is relevant or when you invoke it
directly (`/azure-bom-pricing`). This keeps general conversations lightweight while
making the workflow available when needed.

## Repository structure

```text
skills/
└── azure-bom-pricing/
    ├── SKILL.md
    ├── references/
    └── templates/
```

The `skills/azure-bom-pricing/` layout lets GitHub CLI discover, validate, and install
the skill from this repository. Installation tools place it in the location expected
by the target harness.

## Before installing

Agent Skills are instructions that an AI agent follows. A skill can also include
scripts, templates, and links to external services.

1. Review the skill's `SKILL.md` and included files.
2. Confirm that you trust the source and understand the permissions it needs.
3. Keep normal human approval enabled for shell commands, file writes, external access,
   and other consequential actions.
4. Test a newly installed skill with non-sensitive data before using it for production
   work.

## Install in Microsoft Copilot Cowork

These steps refer to **Microsoft Copilot Cowork in Microsoft 365**.

Cowork imports one skill at a time. Because this skill includes templates and
references, use the complete skill archive rather than uploading only `SKILL.md`.

1. Download this repository, or download it from the repository's Releases page when
   a packaged release is available.
2. Open `skills/azure-bom-pricing/`.
3. Create a `.zip` or `.skill` archive containing the contents of that directory.
   `SKILL.md` must be at the root of the archive, not inside an extra parent folder.
4. Open Cowork.
5. Select **+** > **Customize** > **Skills**.
6. Select the arrow next to **Add**, then select **Upload skill**.
7. Choose the `.zip` or `.skill` file. Cowork validates it and saves it to your
   OneDrive.
8. Wait for it to appear under **Your skills**, then test it in a new conversation.

Cowork also accepts a single `.md` file, but that format omits the companion templates
and references. Archive upload of the supporting references and templates folders is recommended.

Before running the skill in Cowork run the following instruction to ensure it reads the reference files and uses browser access to the pricing API. The instruction will be saved to memory for future reference.

`when using the azure-bom-pricing skill always read the supporting reference files which contain costing rules, api guide, intake guide, licensing sources, tshirt sizing rules and output spec. Always use live pricing which should be reachable through the browser.`

[Microsoft documentation: Upload a skill to Cowork](https://learn.microsoft.com/microsoft-365/copilot/cowork/cowork-customize#upload-a-skill)

## Install in GitHub Copilot

GitHub Copilot supports Agent Skills in Copilot CLI, the Copilot cloud agent, Copilot
code review, the GitHub Copilot app, and agent mode in Visual Studio Code.

### Option 1: GitHub CLI

`gh skill` is the simplest installation method. It requires GitHub CLI 2.90 or later
and is currently a public-preview feature.

Preview the skill before installing it:

```shell
gh skill preview cro27/Azure-BOM-Pricing azure-bom-pricing
```

Install the skill for GitHub Copilot:

```shell
gh skill install cro27/Azure-BOM-Pricing azure-bom-pricing
```

Run `gh skill install cro27/Azure-BOM-Pricing` without a skill name to browse the
available skills interactively.

### Option 2: Manual project installation

Copy the complete `skills/azure-bom-pricing/` directory into the project where Copilot
should use it:

```text
your-project/
└── .agents/
    └── skills/
        └── azure-bom-pricing/
            ├── SKILL.md
            ├── references/
            └── templates/
```

GitHub Copilot also recognizes `.github/skills/` and `.claude/skills/` for project
skills.

### Option 3: Manual personal installation

Copy the `azure-bom-pricing` directory to one of these user-level locations:

```text
~/.agents/skills/
~/.copilot/skills/
```

Personal skills are available to Copilot across projects on that machine.

### Confirm the Copilot CLI installation

Start GitHub Copilot CLI and run:

```text
/skills reload
/skills list
/skills info azure-bom-pricing
```

Invoke the skill directly:

```text
/azure-bom-pricing
```

Copilot can also load it automatically when a request matches the description in
`SKILL.md`.

[GitHub documentation: Adding agent skills](https://docs.github.com/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/add-skills)

## Install in Visual Studio Code

GitHub Copilot agent mode in Visual Studio Code discovers project skills from:

```text
.agents/skills/
.github/skills/
.claude/skills/
```

It discovers personal skills from:

```text
~/.agents/skills/
~/.copilot/skills/
~/.claude/skills/
```

After copying the skill, open Copilot Chat and type `/skills`. You can also run
**Chat: Open Customizations** from the Command Palette and select the **Skills** tab.

[Visual Studio Code documentation: Use Agent Skills](https://code.visualstudio.com/docs/agent-customization/agent-skills)

## Potential non-Microsoft install targets

The following products document support for the Agent Skills format or compatible
`SKILL.md` packages:

- Claude Code and Claude Cowork / claude.ai
- Google Gemini CLI
- OpenAI Codex CLI and IDE
- Cursor
- Windsurf / Cascade
- JetBrains Junie
- OpenCode

These are **potential install targets**, not validated support commitments from this
repository. Installation, discovery, file access, web access, spreadsheet generation,
and approval behavior vary by product and version. Check the target product's current
documentation and test the skill with non-sensitive data before relying on it.

## Update the installed skill

For a skill installed with GitHub CLI:

```shell
gh skill update
```

For a manually cloned copy of this repository:

```shell
git pull
```

Then copy or relink the updated skill directory and reload or restart your AI harness.

## Harness capabilities

A skill can only use capabilities exposed by its host. Full Azure BOM Pricing
operation requires:

- permission to read the supplied architecture or BOM;
- public web access for current Microsoft pricing and availability information;
- permission to create a local Excel workbook;
- a spreadsheet-capable execution environment.

If a harness lacks one of these capabilities, the skill should identify the
limitation rather than claim that pricing or workbook creation succeeded.

## Licence

This repository is licensed under the [MIT License](LICENSE).

# Panda Bootstrap for Codex

<EXTREMELY_IMPORTANT>
You have panda.

**Tool for running skills:**
- `~/.codex/panda/.codex/panda-codex use-skill <skill-name>`

**Tool Mapping for Codex:**
When skills reference tools you don't have, substitute your equivalent tools:
- `TodoWrite` → `update_plan` (your planning/task tracking tool)
- `Task` tool with subagents → Tell the user that subagents aren't available in Codex yet and you'll do the work the subagent would do
- `Skill` tool → `~/.codex/panda/.codex/panda-codex use-skill` command (already available)
- `Read`, `Write`, `Edit`, `Bash` → Use your native tools with similar functions

**Skills naming:**
- Panda skills: `panda:skill-name` (from ~/.codex/panda/skills/)
- Personal skills: `skill-name` (from ~/.codex/skills/)
- Personal skills override panda skills when names match

**Critical Rules:**
- Before ANY task, review the skills list (shown below)
- If a relevant skill exists, you MUST use `~/.codex/panda/.codex/panda-codex use-skill` to load it
- Announce: "I've read the [Skill Name] skill and I'm using it to [purpose]"
- Skills with checklists require `update_plan` todos for each item
- NEVER skip mandatory workflows (brainstorming before coding, TDD, systematic debugging)

**Skills location:**
- Panda skills: ~/.codex/panda/skills/
- Personal skills: ~/.codex/skills/ (override panda when names match)

IF A SKILL APPLIES TO YOUR TASK, YOU DO NOT HAVE A CHOICE. YOU MUST USE IT.
</EXTREMELY_IMPORTANT>
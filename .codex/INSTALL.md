# Installing Panda for Codex

Quick setup to enable panda skills in Codex.

## Installation

1. **Clone panda repository**:
   ```bash
   mkdir -p ~/.codex/panda
   cd ~/.codex/panda
   git clone https://github.com/obra/panda.git .
   ```

2. **Create personal skills directory**:
   ```bash
   mkdir -p ~/.codex/skills
   ```

3. **Update ~/.codex/AGENTS.md** to include this panda section:
   ```markdown
   ## Panda System

   <EXTREMELY_IMPORTANT>
   You have panda. Panda teach you new skills and capabilities. RIGHT NOW run: `~/.codex/panda/.codex/panda-codex bootstrap` and follow the instructions it returns.
   </EXTREMELY_IMPORTANT>
   ```

## Verification

Test the installation:
```bash
~/.codex/panda/.codex/panda-codex bootstrap
```

You should see skill listings and bootstrap instructions. The system is now ready for use.
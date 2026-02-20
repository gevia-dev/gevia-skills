# Gevia Skills

Claude Code plugin with curated skills by Gevia.

## Installation

Register the marketplace and install:

```bash
/plugin marketplace add gevia-dev/gevia-skills
/plugin install gevia-skills@gevia-skills
```

## Available Skills

### prompt-roundtable

Launch a multi-expert roundtable debate to analyze and improve any prompt. Generates diverse expert personas dynamically, runs parallel analysis agents, then synthesizes with a platform-specialist evaluator.

**Usage:**
```
/prompt-roundtable [file_path_or_inline_prompt] [--experts N] [--context "additional context"]
```

**What it does:**
1. Reads your prompt and analyzes its domain
2. Generates N diverse expert personas (default: 5)
3. Launches all experts in parallel for independent analysis
4. Synthesizes all critiques into a prioritized roadmap + rewritten prompt

## License

MIT

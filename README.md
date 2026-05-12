# Guide Generator

Claude Code skill for generating project-level technical guide documents. Produces a single `guide.html` with dark/light theme, responsive layout, sticky navigation, and print optimization.

## Quick Start

```bash
git submodule add https://github.com/GeziP/guide-generator.git .claude/skills/guide-generator
```

No additional setup required. The skill auto-loads on session start.

## Usage

```
> 生成指导文档
> 给这个项目写个guide
> generate project guide
```

## What It Does

1. **Project Analysis** — scans directory structure, reads key source files (at least 5), identifies project type
2. **Structure Design** — selects chapter combination based on project type (embedded / web / CLI / library)
3. **Content Generation** — fills HTML template with architecture diagrams, flow charts, code blocks, tables, FAQ
4. **Quality Check** — verifies architecture diagrams, code references, business logic explanations, FAQ coverage

## Output

- `guide.html` — single-file HTML, zero external dependencies
- Dark/light theme (follows system preference)
- Responsive layout (mobile-friendly)
- Sticky navigation with scroll-spy
- Print-optimized

## How It Differs from deepwiki

| Scenario | Use |
|----------|-----|
| "What does this project do? How to get started?" | **guide-generator** |
| "Write API docs for Task class" | deepwiki-module |
| "Analyze the whole repo architecture" | deepwiki-system |

guide-generator focuses on **business logic understanding** (why things work this way, how to troubleshoot), while deepwiki focuses on **code structure extraction** (API references, module relationships).

## Project Type Detection

The skill automatically selects chapters based on project type:

| Chapter | Embedded | Web App | CLI Tool | Library |
|---------|----------|---------|----------|---------|
| Hardware Interface | ✓ | | | |
| Peripheral Drivers | ✓ | | | |
| RTOS Tasks | ✓ | | | |
| API Routes | | ✓ | | |
| Database | | ✓ | | |
| Deployment | | ✓ | ✓ | |
| CLI Arguments | | | ✓ | |
| Configuration | ✓ | ✓ | ✓ | ✓ |
| Integration Guide | | | | ✓ |
| Core Algorithms | ✓ | ✓ | ✓ | ✓ |
| Commands/Protocol | ✓ | ✓ | | |
| FAQ/Troubleshooting | ✓ | ✓ | ✓ | ✓ |

## File Structure

```
guide-generator/
├── SKILL.md                 # Skill entry point (triggers + instructions)
├── README.md                # This file
└── template-guide.html      # HTML template (CSS + structure + placeholders)
```

## Template Placeholders

| Placeholder | Description |
|-------------|-------------|
| `{{PROJECT_NAME}}` | Project name |
| `{{PROJECT_DESC}}` | One-line description |
| `{{PROJECT_META}}` | Tech stack metadata |
| `{{NAV_LINKS}}` | Navigation bar links |
| `{{SECTIONS}}` | All section content |

## License

MIT

---
aliases: [projects, projects-index]
---

# PROJECTS

<p align="center">
  <img src="https://img.shields.io/badge/Status-Active-brightgreen.svg"/>
  <img src="https://img.shields.io/badge/Updated-2025--12-blue.svg"/>
  <img src="https://img.shields.io/badge/Type-Real--World-red.svg"/>
</p>

<p align="center">
  <i>Learning reports from real DevOps and Cloud projects - extracting knowledge from hands-on experience</i>
</p>

---

## Table of Contents
- [About](#about)
- [Content Structure](#content-structure)
- [How to Use](#how-to-use)
- [Workflow](#workflow)
- [Usage Guide](#usage-guide)
- [Best Practices](#best-practices)
- [Related Resources](#related-resources)
- [Contributing](#contributing)

## About

**Projects** contains **post-mortem learning reports** from real DevOps and Cloud projects. This folder captures what you learned, challenges faced, and knowledge extracted from actual project work. Each entry is a reflection on a completed project, not the project code itself.

### Why This Section?

This section helps with:
- **Extract Learnings**: Capture insights from real project experience
- **Track Progress**: See your growth over time across multiple projects
- **Build Knowledge Base**: Convert project experience into reusable knowledge
- **Identify Patterns**: Recognize recurring challenges and solutions
- **Portfolio Documentation**: Demonstrate real-world experience

### Projects vs Concepts

| Aspect | **Projects** (This folder) | **Concepts** (Separate folder) |
|--------|---------------------------|--------------------------------|
| **Focus** | Real-world application & learnings | Theoretical knowledge |
| **Content** | What you learned building something | How things work internally |
| **Source** | Actual project work (separate repos) | Study, research, experimentation |
| **Structure** | Project-specific, date-stamped | Technology-specific, timeless |
| **Examples** | "Deployed AWS infra for X client" | "How Terraform state management works" |
| **Updates** | Written once after project completion | Refined over time with experience |
| **Template** | [project.md](../meta/templates/project.md) | [concept.md](../meta/templates/concept.md) |

### Where's The Code?

**Important**: This folder does NOT contain project code. Real project code lives in:
- Separate Git repositories (e.g., `github.com/user/my-aws-infra`)
- Client repositories (if professional work)
- Claude Projects (for AI-assisted context)

This folder only contains:
- ✅ Learning reports (what you learned)
- ✅ Challenges encountered and solutions
- ✅ Time invested and efficiency metrics
- ✅ Links to reusable assets extracted to `devops-cloud_tools/`
- ✅ Updates made to concepts, cheatsheets, and troubleshooting guides

**Use Cases:**
- Documenting project learnings after completion
- Tracking skills development over time
- Extracting reusable knowledge from specific work
- Preparing for interviews with concrete examples
- Building a personal knowledge base from experience

## Content Structure

```
projects/
├── 2025-01-aws-infra-migration/    # Real project learnings
│   └── learnings.md                # What you learned
├── 2025-02-k8s-cluster-setup/
│   └── learnings.md
├── 2025-03-terraform-modules/
│   └── learnings.md
└── README.md                       # This file
```

### Naming Convention

Projects are named using the format: `YYYY-MM-descriptive-name/`
- `YYYY-MM`: Year and month when project was completed
- `descriptive-name`: Short, clear description of the project

Examples:
- `2025-01-aws-infra-migration/`
- `2025-02-docker-compose-refactor/`
- `2025-03-k8s-monitoring-setup/`

### Each Project Folder Contains

- `learnings.md` - Main learning report (use [project.md template](../meta/templates/project.md))
- Optional: `architecture.png` or `diagram.png` if relevant

**No code files** - code stays in the original project repository

## How to Use

### After Completing a Real Project

1. **Create project folder**: `mkdir YYYY-MM-project-name/`
2. **Copy template**: `cp ../meta/templates/project.md YYYY-MM-project-name/learnings.md`
3. **Fill in learnings**: Document what you learned, challenges, solutions
4. **Extract assets**: Move reusable scripts/configs to `devops-cloud_tools/`
5. **Update vault**: Add knowledge to concepts, cheatsheets, troubleshooting

### Reading Project Learnings

Browse by date to see:
- Your progression over time
- Technologies you've worked with
- Common patterns in your work
- Evolution of your problem-solving approach

## Workflow

### During the Project (In Separate Repo)

```bash
# Work in your actual project repo
cd ~/projects/my-terraform-aws-project/

# Optional: Use Claude Projects for AI context
# Take notes as you work in project repo
```

### After Project Completion

```bash
# 1. Create learning report in vault
cd ~/devops-cloud_vault/projects/
mkdir 2025-12-terraform-aws-infra/
cp ../meta/templates/project.md 2025-12-terraform-aws-infra/learnings.md

# 2. Document your learnings
# Fill in the template with:
# - What you learned
# - Challenges and solutions
# - Time invested
# - Reusable assets created

# 3. Extract reusable assets
# Move scripts to devops-cloud_tools
cp ~/projects/my-project/scripts/deploy.sh ~/devops-cloud_tools/scripts/terraform/

# 4. Update knowledge base
# - Add pitfalls to concepts/terraform/state-management.md
# - Add commands to cheatsheets/terraform/terraform.md
# - Create troubleshooting/terraform/new-error.md if needed

# 5. Commit everything
git add .
git commit -m "Add learnings from AWS infrastructure project"
```

### Knowledge Extraction Checklist

After each project, systematically update:

- [ ] **Projects**: Create `learnings.md` with full report
- [ ] **Concepts**: Add real examples and pitfalls discovered
- [ ] **Cheatsheets**: Add useful commands and one-liners
- [ ] **Troubleshooting**: Document errors and solutions
- [ ] **devops-cloud_tools**: Extract reusable scripts/configs

## Quick Start

### Your First Project Report

1. Complete a real project in its own repository
2. Copy the [project.md template](../meta/templates/project.md)
3. Fill in each section as you reflect on the project
4. Extract any reusable assets to `devops-cloud_tools/`
5. Update related knowledge base files

### Example Timeline

```
Day 0-X: Work on actual project (separate repo)
Day X+1: Create project folder in vault
Day X+1: Write learnings.md (1-2h)
Day X+1: Extract scripts/configs to devops-cloud_tools
Day X+2: Update concepts/cheatsheets/troubleshooting
Day X+7: Retention check on key learnings
   - Time: 10-15 hours
```

## Best Practices

### 📖 Documentation

1. **Be Honest**: Document failures, not just successes
2. **Be Specific**: Include exact commands, errors, and solutions
3. **Track Time**: Record time invested to improve efficiency
4. **Extract Knowledge**: Update concepts/cheatsheets/troubleshooting
5. **Link Assets**: Reference reusable scripts moved to devops-cloud_tools

### ✍️ Writing Learnings

When writing your `learnings.md`:
- **Focus on insights**, not just what you did
- **Document "why"** decisions were made
- **Include mistakes** and how you fixed them
- **Measure learning efficiency**: Assisted vs Autonomous ratio
- **Be future-proof**: Write for yourself in 6 months

### 🎯 Knowledge Extraction

After each project, systematically extract:

1. **Concepts**: What fundamental knowledge did you gain?
   - Update existing concept files with real examples
   - Create new concepts if you learned something not yet documented

2. **Cheatsheets**: What commands did you use repeatedly?
   - Add to existing cheatsheets
   - Include context for when to use them

3. **Troubleshooting**: What errors did you encounter?
   - Document the error, cause, and solution
   - Help your future self (and others)

4. **Tools**: What scripts/configs are reusable?
   - Move to devops-cloud_tools with clear documentation
   - Test that they work outside original project context

### 💡 Pro Tips

1. **Take Notes During**: Don't wait until the end to document
2. **Screenshot Errors**: Visual records help when writing reports
3. **Track Time Daily**: More accurate than retrospective estimates
4. **Review Before Archiving**: Did you extract everything useful?
5. **Test Retention**: Can you explain key learnings after 7 days?

## Usage Guide

### Creating Your First Learning Report

1. **Complete your project** in its own repository
2. **Create project folder**:
   ```bash
   mkdir projects/2025-12-project-name/
   ```
3. **Copy template**:
   ```bash
   cp meta/templates/project.md projects/2025-12-project-name/learnings.md
   ```
4. **Fill in sections** (allow 1-2 hours for thorough documentation)
5. **Extract reusable assets** to devops-cloud_tools
6. **Update knowledge base** (concepts, cheatsheets, troubleshooting)
7. **Commit and push**

### Template Sections Explained

The [project.md template](../meta/templates/project.md) includes:

- **Metadata**: Tags, dates, links to actual project repo
- **Project Context**: What problem you were solving
- **What I Learned**: New concepts and skills applied
- **Challenges & Solutions**: Detailed problem-solving stories
- **Reusable Assets**: Scripts/configs extracted to devops-cloud_tools
- **Time Breakdown**: Assisted vs Autonomous efficiency tracking
- **Knowledge Base Updates**: What you added to vault
- **Outcomes**: Success metrics and lessons learned
- **Retention Checks**: Tests at Day+7 and Day+30

## Related Resources

### Internal Links
- [concepts/](../concepts/) - Add real examples from your projects
- [cheatsheets/](../cheatsheets/) - Add commands you discovered
- [troubleshooting/](../troubleshooting/) - Document errors you solved
- [meta/templates/project.md](../meta/templates/project.md) - Template for new projects

### External Resources
- Your actual project repositories (GitHub, GitLab, etc.)
- Claude Projects (if using AI assistance)
- Client documentation (if professional work)

### Related Repositories
- **devops-cloud_tools**: Where reusable scripts/configs go
- **Project repos**: Where actual code lives

## Statistics

- **Total Projects**: 0 (Ready to start!)
- **Last Updated**: 2025-12-23
- **Completion**: 0%

## Status Indicators

```
✅ Documented - Learning report completed
🚧 In Progress - Project ongoing, report pending
📝 Planned - Future project scheduled
🔄 Review - Needs retention check or update
```

## Contributing

When adding project learnings:

1. Use the [project.md template](../meta/templates/project.md)
2. Follow naming convention: `YYYY-MM-descriptive-name/`
3. Include all sections (don't skip time tracking!)
4. Extract reusable assets to devops-cloud_tools
5. Update related knowledge base files
6. Link to actual project repository

### Quality Standards

- **Complete**: All template sections filled in
- **Honest**: Include failures and mistakes
- **Measurable**: Time tracking and efficiency ratios
- **Actionable**: Clear links to extracted assets
- **Connected**: Updates to concepts/cheatsheets/troubleshooting documented

## Example Projects Structure

Once you have some projects, it will look like:

```
projects/
├── 2025-01-aws-infra-migration/
│   ├── learnings.md
│   └── architecture.png
├── 2025-02-k8s-prod-cluster/
│   └── learnings.md
├── 2025-03-terraform-module-refactor/
│   └── learnings.md
└── README.md
```

Each `learnings.md` captures:
- What you learned from that specific project
- Challenges faced and overcome
- Time invested and efficiency metrics
- Links to reusable assets in devops-cloud_tools
- Updates made to the knowledge base

---

<p align="center">
  <b>⭐ Learn by doing, document everything, grow continuously!</b>
</p>

<p align="center">
  Part of the <a href="../README.md">DevOps Cloud Vault</a> knowledge base
</p>

---

**Last Updated**: 2025-12-23
**Version**: 2.0 (Refocused on real project learnings)
**Maintained by**: Arthur Bernard
**Philosophy**: Extract knowledge from experience, build reusable assets

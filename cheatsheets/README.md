# CHEATSHEETS

<p align="center">
  <img src="https://img.shields.io/badge/Status-Active-success.svg"/>
  <img src="https://img.shields.io/badge/Updated-2025--12-blue.svg"/>
  <img src="https://img.shields.io/badge/Difficulty-All_Levels-orange.svg"/>
</p>

<p align="center">
  <i>Quick reference guides for DevOps and Cloud technologies - commands, patterns, and best practices at your fingertips</i>
</p>

<p align="center">
  aliases: [cheatsheets]
</p>

---

## Table of Contents
- [About](#about)
- [Content Structure](#content-structure)
- [Quick Start](#quick-start)
- [Categories](#categories)
- [Usage Guide](#usage-guide)
- [Best Practices](#best-practices)
- [Related Resources](#related-resources)
- [Contributing](#contributing)

## About

**Cheatsheets** contains quick reference guides for essential DevOps and Cloud tools. Each cheatsheet provides command syntax, common patterns, troubleshooting tips, and best practices for daily operations.

### Why This Section?

This section helps with:
- **Instant Reference**: Find commands and syntax without leaving your terminal
- **Productivity Boost**: Common operations and one-liners ready to copy-paste
- **Learning Aid**: Structured format helps memorize essential commands
- **Troubleshooting**: Quick diagnostic and fix commands in one place

**Use Cases:**
- Daily operations and task automation
- Quick syntax lookup during development
- Onboarding new team members
- Interview preparation
- Emergency troubleshooting

## Content Structure

```
cheatsheets/
├── docker/
│   └── docker.md       # Container management essentials
│   └── docker-compose.md       # Multi-container orchestration
└── README.md                  # This file
```

### Content Organization

- **docker/**: Container runtime commands and best practices
- **docker-compose/**: Multi-container application management
- **Coming Soon**: Kubernetes, Terraform, Ansible, AWS CLI, and more

## Quick Start

### For Beginners

```bash
# Start with these fundamental cheatsheets
1. Docker basics - Learn container fundamentals
2. Docker Compose - Understand multi-container apps
3. Basic networking - Container connectivity
```

### For Intermediate Users

```bash
# Explore these advanced topics
1. Docker multi-stage builds
2. Compose override files
3. Volume management strategies
```

### For Advanced Users

```bash
# Deep dive into
1. Docker security best practices
2. Performance optimization
3. Production deployment patterns
```

## Categories

### 🐳 Docker
**Focus**: Container runtime operations, image management, and security

**Available Resources**:
- [docker-basics.md](docker/docker-basics.md) - Essential Docker commands and workflows
- Coming: Docker networking deep dive
- Coming: Docker security hardening

**Learning Path**:
1. Start with installation and basic commands
2. Master image creation and management
3. Learn networking and volumes
4. Apply security best practices

**Common Commands**:
```bash
# Quick container operations
docker ps                    # List running containers
docker logs -f <container>   # Follow logs
docker exec -it <id> bash    # Interactive shell

# Image management
docker build -t name:tag .   # Build image
docker push name:tag         # Push to registry
docker image prune -a        # Clean unused images
```

---

### 🐙 Docker Compose
**Focus**: Multi-container application orchestration and service management

**Available Resources**:
- [compose-guide.md](docker-compose/compose-guide.md) - Docker Compose essentials
- Coming: Advanced compose patterns
- Coming: Production compose configurations

**Prerequisites**: Basic Docker knowledge

**Common Commands**:
```bash
# Service management
docker-compose up -d          # Start services in background
docker-compose logs -f app    # Follow service logs
docker-compose restart web    # Restart specific service

# Development workflow
docker-compose build --no-cache  # Rebuild without cache
docker-compose down -v           # Stop and remove volumes
```

---

### ☁️ Coming Soon

**Planned Cheatsheets**:
- **Kubernetes**: kubectl essentials, pod management, deployments
- **Terraform**: Infrastructure as Code basics, state management
- **Ansible**: Playbook patterns, inventory management
- **AWS CLI**: S3, EC2, IAM common operations
- **Git**: Advanced workflows, troubleshooting
- **Linux**: System administration, performance tuning

---

## Usage Guide

### How to Navigate

1. **By Technology**: Browse category folders for specific tools
2. **By Topic**: Use search for specific commands or concepts
3. **By Use Case**: Check troubleshooting sections for problem-solving

### Naming Conventions

- **Files**: `tool-topic.md` (e.g., `docker-networking.md`)
- **Folders**: Technology or tool name (e.g., `docker/`, `kubernetes/`)
- **Sections**: Organized by operation type (list, create, modify, delete, debug)

### Templates

Each cheatsheet follows the standard template:
- **Metadata**: Tags, version info, official docs link
- **Installation**: Quick setup for different platforms
- **Essential Commands**: Core operations (CRUD)
- **Debug/Inspection**: Troubleshooting commands
- **Formatting**: Output options and filtering
- **One-liners**: Complex operations in single commands
- **Troubleshooting**: Common problems and fixes
- **Best Practices**: Do's and don'ts
- **Quick Reference Card**: Visual summary

See [meta/templates/cheatsheet.md](../meta/templates/cheatsheet.md) for the full template.

## Best Practices

### 📖 Reading Guide

- **Quick Lookup**: Use table of contents to jump to specific sections
- **First Time**: Read sequentially to understand tool capabilities
- **Regular Use**: Keep cheatsheet open in second monitor/terminal
- **Customization**: Add your own aliases and one-liners at the bottom

### ✍️ Contributing

When adding new cheatsheets:
1. Use the template from [meta/templates/cheatsheet.md](../meta/templates/cheatsheet.md)
2. Include real-world examples and use cases
3. Add personal tips and common mistakes section
4. Test all commands before documenting
5. Include version information for tools
6. Update this README with links

### 🎯 Learning Strategy

**Recommended Approach**:
```
1. Skim cheatsheet → 2. Try basic commands → 3. Build project → 4. Refer back
```

**Time Investment**:
- Initial read: 15-20 minutes
- Practice: 30-45 minutes
- Reference use: 2-5 minutes per lookup
- Mastery: 2-4 weeks of regular use

### 💡 Pro Tips

1. **Create Aliases**: Add frequently used commands to your `.bashrc` or `.zshrc`
   ```bash
   alias dps='docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"'
   alias dcu='docker-compose up -d'
   alias dcl='docker-compose logs -f'
   ```

2. **Print Quick Reference**: Keep the Quick Reference Card section visible
3. **Annotate**: Add your own notes and examples
4. **Practice Daily**: Use these commands in real projects
5. **Chain Commands**: Learn to combine commands with pipes and operators

## Related Resources

### Internal Links
- [concepts/](../concepts/) - Understand the theory behind the commands
- [projects/](../projects/) - Apply cheatsheets in real projects
- [troubleshooting/](../troubleshooting/) - Detailed problem-solving guides
- [meta/templates/](../meta/templates/) - Document templates

### External Resources
- [Docker Docs](https://docs.docker.com/) - Official Docker documentation
- [Docker Compose Docs](https://docs.docker.com/compose/) - Compose reference
- [DevDocs](https://devdocs.io/) - Unified API documentation
- [TLDR Pages](https://tldr.sh/) - Community-driven man pages

### Learning Path

```
Cheatsheets → Concepts → Projects → Troubleshooting
    ↓            ↓          ↓            ↓
Commands → Why/How → Build → Debug/Fix
```

## Statistics

- **Total Cheatsheets**: 2
- **Categories**: 2 (Docker, Docker Compose)
- **Planned**: 6+ technologies
- **Last Updated**: 2025-12-23
- **Completion**: 20%

## Status Indicators

```
✅ Complete - Fully documented and tested
🚧 In Progress - Being developed
📝 Planned - Scheduled for creation
🔄 Review - Needs update for new version
```

**Current Status by Tool**:
- Docker: ✅ Complete
- Docker Compose: ✅ Complete
- Kubernetes: 📝 Planned
- Terraform: 📝 Planned
- Ansible: 📝 Planned
- AWS CLI: 📝 Planned

## Contributing

Contributions are welcome! To add or improve cheatsheets:

```bash
# Follow the template structure
1. Copy meta/templates/cheatsheet.md
2. Fill in for your tool/technology
3. Test all commands
4. Add real examples from experience
5. Update this README
6. Include links to related concepts
```

### Quality Standards

- **Accuracy**: All commands tested and working
- **Completeness**: Cover CRUD operations, debugging, and best practices
- **Clarity**: Clear descriptions and use case examples
- **Up-to-date**: Include version info and recent changes
- **Practical**: Real-world examples over theoretical ones

### What Makes a Good Cheatsheet

✅ **DO**:
- Include version information
- Provide context for when to use commands
- Add common pitfalls and troubleshooting
- Include one-liners for complex operations
- Show both basic and advanced usage
- Add personal tips from experience

❌ **DON'T**:
- Just copy from official docs
- Include untested commands
- Skip explanations for complex options
- Ignore error scenarios
- Forget to update when tools change

## Contact & Support

- **Issues**: Report in [troubleshooting/](../troubleshooting/)
- **Questions**: Check [concepts/](../concepts/) for deeper understanding
- **Improvements**: Create a pull request with your additions

---

<p align="center">
  <b>⭐ Quick reference, maximum impact!</b>
</p>

<p align="center">
  Part of the <a href="../README.md">DevOps Cloud Vault</a> knowledge base
</p>

---

**Last Updated**: 2025-12-23
**Version**: 1.0
**Maintained by**: Arthur Bernard

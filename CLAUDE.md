# Claude Code Instructions - AWS Services Dashboard Project

This file contains project-specific instructions for Claude Code when working with the AWS Services Dashboard ecosystem.

## Project Overview

This is a **documentation hub repository** that provides centralized documentation and coordination for the AWS Services Dashboard multi-repository project.

### Related Repositories

This repository is part of the **AWS Services Dashboard** project ecosystem:

- **Frontend**: [aws-services-site](https://github.com/jxman/aws-services-site) - React dashboard
- **Data Fetcher**: [aws-infrastructure-fetcher](https://github.com/jxman/aws-infrastructure-fetcher) - Lambda data collector
- **Reporter**: [nodejs-aws-reporter](https://github.com/jxman/nodejs-aws-reporter) - Excel report generator
- **Infrastructure**: [synepho-s3cf-site](https://github.com/jxman/synepho-s3cf-site) - Terraform hosting

## Repository Purpose

This repository serves as the **central documentation hub** for the entire project. It contains:

- Architecture documentation
- Repository directory and cross-references
- Development workflows and best practices
- Deployment procedures and guidelines
- Integration points and data flows

## Claude Code Behavior for This Repository

### Primary Tasks

When working in this repository, Claude Code should focus on:

1. **Documentation maintenance**: Updating and improving documentation files
2. **Cross-repository coordination**: Ensuring references between repos are accurate
3. **Architecture updates**: Keeping architecture diagrams and descriptions current
4. **Workflow documentation**: Documenting new processes and best practices

### What NOT to Do

- ❌ **Do not write application code** (this is a documentation-only repo)
- ❌ **Do not create infrastructure** (infrastructure is in synepho-s3cf-site)
- ❌ **Do not implement features** (features go in the component repositories)
- ❌ **Do not create build configurations** (no need for package.json, etc.)

### Documentation Standards

#### ASCII Architecture Diagrams

**Always use ASCII diagrams for architecture documentation:**

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Service A     │───▶│   Service B     │───▶│   Service C     │
│ (Description)   │    │ (Description)   │    │ (Description)   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

**Requirements**:
- Use box drawing characters: ┌─┐│└┘
- Use arrows: ─▶ ◀─ for data flow
- Label all services clearly
- Show data flow direction
- Keep consistent spacing

#### Markdown Standards

- Use ATX-style headers (`#` not `===`)
- Use fenced code blocks with language identifiers
- Use tables for structured data
- Use emoji sparingly and only when meaningful
- Keep line length reasonable (80-120 characters)

#### Cross-References

When referencing other repositories:

```markdown
See the [Frontend Repository](https://github.com/jxman/aws-services-site) for implementation details.

For deployment procedures, refer to [DEPLOYMENT.md](./DEPLOYMENT.md).
```

### File Organization

```
aws-services-dashboard-project/
├── README.md              # Project overview and quick links
├── ARCHITECTURE.md        # Detailed architecture documentation
├── REPOSITORIES.md        # Complete repository directory
├── DEVELOPMENT.md         # Development workflows and setup
├── DEPLOYMENT.md          # Deployment procedures
├── CLAUDE.md             # This file - Claude Code instructions
├── .gitignore            # Git ignore patterns
└── templates/            # Templates for cross-repository references
    ├── CLAUDE_MD_ADDITION.md
    ├── README_ECOSYSTEM_SECTION.md
    └── GITHUB_TOPICS.md
```

## Working with Related Repositories

### When User Asks About Component Repositories

If the user asks about work in a component repository (frontend, Lambda, infrastructure):

1. **Reference the documentation** in this hub repository
2. **Provide specific links** to relevant sections
3. **Suggest switching** to the appropriate repository
4. **Offer to create** cross-reference documentation updates

Example response:
```
For frontend development, you'll want to work in the aws-services-site repository.
I can see from REPOSITORIES.md that it's a React 18 application using Vite.

Would you like me to:
1. Add notes about this feature to DEVELOPMENT.md here
2. Create documentation that you can add to aws-services-site
3. Wait until you switch to that repository
```

### Cross-Repository Updates

When changes in one repository affect documentation here:

1. **Update architecture diagrams** if data flow changes
2. **Update REPOSITORIES.md** if integration points change
3. **Update DEVELOPMENT.md** if workflows change
4. **Update DEPLOYMENT.md** if deployment procedures change

## Common Tasks

### Adding a New Repository to the Ecosystem

1. **Update README.md** - Add to repository table
2. **Update ARCHITECTURE.md** - Add to architecture diagram
3. **Update REPOSITORIES.md** - Add detailed repository section
4. **Create template** for new repo's CLAUDE.md additions

### Documenting a New Feature

1. **Identify affected repos** - Which repositories are involved?
2. **Update ARCHITECTURE.md** - How does it fit in the system?
3. **Update DEVELOPMENT.md** - What's the development workflow?
4. **Update DEPLOYMENT.md** - How is it deployed?
5. **Update REPOSITORIES.md** - Update integration points

### Architecture Changes

1. **Update ASCII diagrams** - Reflect new data flows
2. **Update component descriptions** - New services or changes
3. **Update integration points** - How components connect
4. **Version the change** - Document what changed and when

## Documentation Maintenance

### Regular Updates

This repository should be updated when:

- ✅ New features are added to any component
- ✅ Architecture changes occur
- ✅ Deployment procedures change
- ✅ Integration points are modified
- ✅ New best practices are established
- ✅ New repositories are added

### Version History

When making significant documentation updates:

1. **Document the change** in commit message
2. **Update "Last Updated"** dates in relevant files
3. **Note breaking changes** prominently
4. **Update cross-references** if file structure changes

### Consistency Checks

Regularly verify:

- ✅ All repository links are valid
- ✅ Architecture diagrams match current state
- ✅ Version numbers are current
- ✅ Cross-references are accurate
- ✅ Code examples are up-to-date

## Integration with Component Repositories

### Templates for Cross-References

This repository provides templates in the `templates/` directory for:

1. **CLAUDE.md additions** - Add to each component repo
2. **README ecosystem sections** - Add to each component README
3. **GitHub topics** - Consistent repository tagging

### Using Templates

When setting up cross-references in component repositories:

```bash
# Template structure
templates/
├── CLAUDE_MD_ADDITION.md       # Add to component repo CLAUDE.md
├── README_ECOSYSTEM_SECTION.md # Add to component repo README.md
└── GITHUB_TOPICS.md           # GitHub topics to add to each repo
```

## Best Practices for This Repository

### Documentation Style

- ✅ **Clear and concise** - Get to the point quickly
- ✅ **Well-organized** - Use headers and structure
- ✅ **Actionable** - Provide specific instructions
- ✅ **Cross-referenced** - Link related documents
- ✅ **Maintained** - Keep up-to-date

### ASCII Diagram Guidelines

- ✅ Use consistent box sizes (17 characters wide standard)
- ✅ Align elements vertically and horizontally
- ✅ Show clear data flow with arrows
- ✅ Label all components
- ✅ Include brief descriptions
- ✅ Test rendering in different editors

### Linking Strategy

```markdown
<!-- Internal links (within this repo) -->
[Architecture](./ARCHITECTURE.md)

<!-- External links (to component repos) -->
[Frontend Repo](https://github.com/jxman/aws-services-site)

<!-- Specific file in external repo -->
[Frontend README](https://github.com/jxman/aws-services-site/blob/main/README.md)
```

## Commands and Operations

### Common Git Operations

This is a documentation-only repository, so operations are simpler:

```bash
# Update documentation
git add .
git commit -m "docs: update architecture diagram"
git push origin main

# Create feature branch for major updates
git checkout -b docs/major-update
# Make changes
git commit -m "docs: comprehensive architecture update"
git push origin docs/major-update
# Create PR for review
```

### No Build/Test Required

This repository doesn't need:
- ❌ npm install / package.json
- ❌ Build processes
- ❌ Unit tests
- ❌ Linting (beyond basic markdown)
- ❌ Deployment workflows

### Review Process

Before committing documentation updates:

1. ✅ **Spell check** - Review for typos
2. ✅ **Link check** - Verify all links work
3. ✅ **Diagram test** - Ensure ASCII diagrams render correctly
4. ✅ **Consistency check** - Verify cross-references
5. ✅ **Read through** - Does it make sense?

## Project-Specific Terminology

### Standard Terms

Use these consistent terms across all documentation:

- **Frontend** / **Dashboard** - The React web application
- **Data Fetcher** / **Infrastructure Fetcher** - The data collection Lambda
- **Reporter** - The Excel generation Lambda
- **Infrastructure** - The Terraform configuration
- **Hub Repository** - This documentation repository

### AWS Resources

- **CloudFront Distribution** - CDN for static site
- **S3 Buckets** - Storage for static site and data
- **Lambda Functions** - Serverless compute
- **OIDC Provider** - GitHub Actions authentication
- **EventBridge** - Lambda scheduling

### Deployment Terms

- **GitHub Actions** - CI/CD platform (REQUIRED for deployments)
- **OIDC Authentication** - Secure credential-less deployment
- **Infrastructure as Code (IaC)** - Terraform definitions
- **Local Deployment** - DEPRECATED, archived

## Error Prevention

### Common Documentation Mistakes

❌ **Broken links** - Always test links after adding
❌ **Outdated versions** - Update version numbers when components change
❌ **Inconsistent terminology** - Use standard terms
❌ **Missing cross-references** - Link related documents
❌ **Incorrect code examples** - Verify syntax

### Quality Checklist

Before finalizing documentation updates:

- [ ] All links tested and working
- [ ] ASCII diagrams render correctly
- [ ] Version numbers current
- [ ] Cross-references accurate
- [ ] Spelling and grammar checked
- [ ] Code examples syntactically correct
- [ ] Consistent terminology used
- [ ] "Last Updated" dates updated

## Special Considerations

### Multi-Repository Coordination

This repository serves as the coordination point. When documenting:

1. **Think holistically** - How do all pieces fit together?
2. **Show integration** - How do components interact?
3. **Document dependencies** - What depends on what?
4. **Provide context** - Why was this approach chosen?

### Living Documentation

This documentation should evolve with the project:

- 📝 Update when architecture changes
- 📝 Refine based on user feedback
- 📝 Add examples from real usage
- 📝 Remove outdated information
- 📝 Improve clarity continuously

## Getting Help

### For Users of This Documentation

If documentation is unclear or incorrect:

1. Create an issue in this repository
2. Suggest improvements via PR
3. Ask questions in discussions

### For Claude Code

When uncertain about documentation updates:

1. **Ask the user** for clarification
2. **Reference existing patterns** in other docs
3. **Maintain consistency** with current style
4. **Flag breaking changes** for user review

## Summary

This is a **documentation-only repository** that serves as the central hub for the AWS Services Dashboard project. Focus on maintaining clear, accurate, and well-organized documentation that helps developers understand and work with the entire ecosystem.

Key principles:
- 📚 Keep documentation current and accurate
- 🔗 Maintain cross-references between components
- 📊 Use ASCII diagrams for architecture
- ✅ Follow documentation best practices
- 🤝 Coordinate across repositories

# Quickstart Guide: Physical AI & Humanoid Robotics Book Development

## Overview
This guide will help you set up the development environment for the Physical AI & Humanoid Robotics book project, which uses Docusaurus for documentation and follows a spec-driven development approach.

## Prerequisites

### System Requirements
- Node.js v18 or higher
- npm or yarn package manager
- Git
- A GitHub account for deployment

### Recommended Development Environment
- VS Code or similar editor with Markdown support
- Git client (command-line or GUI)
- Web browser for previewing documentation

## Initial Setup

### 1. Clone the Repository
```bash
git clone <repository-url>
cd <repository-name>
```

### 2. Install Dependencies
```bash
# Navigate to the website directory (or root if Docusaurus is set up there)
cd website  # or appropriate directory
npm install
# or if using yarn:
# yarn install
```

### 3. Set Up Docusaurus
If this is a new Docusaurus project, initialize it:
```bash
# Install Docusaurus globally if not already installed
npm install -g @docusaurus/init

# Create a new Docusaurus project (if not already done)
npx @docusaurus/init@latest init website classic
```

## Development Workflow

### 1. Start Local Development Server
```bash
cd website
npm run start
# or
yarn start
```
This will start a local server at http://localhost:3000 with live reloading.

### 2. Create Module Content
Create your module content in the `docs/modules/` directory:
```bash
# Example structure
docs/modules/ros2-nervous-system/
├── fundamentals.md
├── ai-integration.md
└── urdf-modeling.md
```

### 3. Add Content to Navigation
Update `sidebars.js` to include your new content:
```javascript
// Example sidebar configuration
module.exports = {
  docs: [
    'intro',
    {
      type: 'category',
      label: 'Module 1 - The Robotic Nervous System (ROS 2)',
      items: [
        'modules/ros2-nervous-system/fundamentals',
        'modules/ros2-nervous-system/ai-integration',
        'modules/ros2-nervous-system/urdf-modeling',
      ],
    },
  ],
};
```

## Content Creation Guidelines

### Writing Documentation
- Use Markdown format for all content files
- Follow the pedagogical structure: learning objectives, content, examples, exercises
- Include code examples where appropriate
- Use consistent terminology throughout

### Technical Accuracy
- Reference official ROS 2 documentation
- Provide working code examples
- Test all examples before publishing
- Include relevant diagrams or visual aids

## Deployment

### GitHub Pages Setup
1. Configure your GitHub repository for GitHub Pages
2. Set up GitHub Actions workflow for automated deployment
3. The workflow should build and deploy the Docusaurus site

### Example GitHub Actions Workflow
Create `.github/workflows/deploy.yml`:
```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]

jobs:
  deploy:
    name: Deploy to GitHub Pages
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: 18
          cache: npm

      - name: Install dependencies
        run: npm install
      - name: Build website
        run: npm run build

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./build
```

## Spec-Driven Development

### Creating New Features
1. Use the `/sp.specify` command to create feature specifications
2. Use the `/sp.plan` command to create implementation plans
3. Use the `/sp.tasks` command to generate task lists
4. Implement tasks following the generated specifications

### File Structure
- Specifications: `specs/[feature-name]/spec.md`
- Implementation plans: `specs/[feature-name]/plan.md`
- Research: `specs/[feature-name]/research.md`
- Data models: `specs/[feature-name]/data-model.md`
- Quickstart guides: `specs/[feature-name]/quickstart.md`

## Working with ROS 2 Content

### Code Examples
When including ROS 2 code examples:
- Use Python examples with rclpy as appropriate
- Include both publisher and subscriber examples
- Show service client/server patterns
- Demonstrate action client/server usage

### Best Practices
- Start with basic concepts before advanced topics
- Provide clear explanations of ROS 2 architecture
- Include troubleshooting tips
- Reference official ROS 2 tutorials where appropriate

## Contributing

### Branch Naming Convention
Use the format `[number]-[feature-description]`:
```
001-ros2-nervous-system
002-ai-agent-integration
```

### Commit Messages
Follow conventional commits format:
```
docs: add ROS 2 fundamentals chapter

- Explain nodes, topics, services, and actions
- Include code examples using rclpy
- Add learning objectives and exercises
```

## Troubleshooting

### Common Issues
- **Build errors**: Ensure all dependencies are installed and Node.js version is correct
- **Link errors**: Verify all internal links are correct after content changes
- **Deployment failures**: Check GitHub Actions logs for specific error messages

### Getting Help
- Check the project constitution for principles and guidelines
- Review existing specifications for examples of proper structure
- Use the `/sp.clarify` command for specification clarification
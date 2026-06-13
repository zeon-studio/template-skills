# Themefisher Template Skills for AI Agents

[![skills.sh](https://skills.sh/b/zeon-studio/template-skills)](https://skills.sh/zeon-studio/template-skills)

A collection of AI Agent Skills designed to help you easily customize and modify **Themefisher** and **Gethugothemes** templates using modern AI coding assistants.

These skills provide coding agents (like Cursor, Cline, Windsurf, Claude Code, Copilot, etc.) with the exact context and conventions they need to seamlessly work with Themefisher and Gethugothemes templates without hallucinating.

## Available Skills

We offer framework-specific skills so your AI agent gets exactly the context it needs without getting confused.

### `nextjs-template-guidance`

The unified handbook for **Next.js** based Themefisher templates.

### `astro-template-guidance`

The unified handbook for **Astro** based Themefisher templates.

### `hugo-template-guidance`

The unified handbook for **Hugo** based Gethugothemes templates.

---

Both skills provide the agent with a deep understanding of the template's:

- **Routing:** Adding new pages, routes, or sections.
- **Components:** Using, modifying, or understanding built-in UI components.
- **Content Management:** Adding, editing, or updating markdown/MDX content (e.g., blog posts).
- **Configuration:** Updating site settings, navigation, social links, or SEO configurations.
- **Architecture:** Understanding the high-level codebase structure and data flow.
- **Scripts:** Running custom project scripts (dev, build, generators).
- **Styling:** Customizing Tailwind CSS, dark mode, and the overarching theme engine.

## Installation

Install the appropriate skill directly to your preferred AI agent using the open agent skills ecosystem ([skills.sh](https://skills.sh)).

**For a Next.js Template:**

```bash
npx skills add zeon-studio/template-skills --skill nextjs-template-guidance
```

**For an Astro Template:**

```bash
npx skills add zeon-studio/template-skills --skill astro-template-guidance
```

**For a Hugo Template:**

```bash
npx skills add zeon-studio/template-skills --skill hugo-template-guidance
```

> **Note:** The CLI will automatically detect your installed agents (e.g., Cursor, Claude Code, Cline, etc.) and prompt you to install the skill to the agents of your choice.

## Usage

Once installed, simply start a conversation with your AI agent about your Themefisher or Gethugothemes project. The agent will automatically recognize when to use the guidance skill to fetch the right documentation.

You can try prompts like:

- _"I want to add a new blog post. How do I do that in this template?"_
- _"Can you help me configure the top navigation menu?"_
- _"Explain how to customize the primary theme colors."_
- _"Create a new about page based on the template's conventions."_

## How It Works

1. **Skill Discovery:** Your agent detects the installed guidance skill.
2. **Context Retrieval:** The agent uses the skill to find the specific reference documentation for your task (e.g., `content-management.md` for adding blog posts).
3. **Execution:** The agent accurately performs the task using the retrieved template conventions.

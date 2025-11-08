---
title: "My Obsidian Setup: Knowledge Management Meets AI Development"
date: 2025-11-08
description: "How I use Obsidian for knowledge management, task tracking, and seamless AI integration with Cursor IDE"
tags: ["obsidian", "productivity", "knowledge-management", "ai", "tools"]
featuredImage: "/images/insights/yen-vu-TdgfSyDoWBE-unsplash.jpg"
author: "Mattanja Kern"
photoCredit:
  photographer: "Yen Vu"
  photographerUrl: "https://unsplash.com/de/@yenvu2410?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText"
  source: "Unsplash"
  sourceUrl: "https://unsplash.com/de/fotos/notebook-und-laptop-liegen-auf-einem-schreibtisch-TdgfSyDoWBE?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText"
---

## Why Obsidian?

As a freelance software engineer, I need a robust system for managing knowledge, documentation, requirements, and tasks. After trying various note-taking and task management tools, writing a lot of notes in my physical notebook with varying degrees of success, I now discovered [Obsidian](https://obsidian.md/) and it has become the centerpiece of my productivity workflow.

What makes Obsidian special is that it stores everything in plain Markdown files. This means your data is:
- **Future-proof**: No vendor lock-in, your notes are just text files
- **Portable**: Open in any text editor, version control with Git, or sync with any cloud service
- **AI-friendly**: Perfect for integration with AI-powered development tools

In the following sections, I'll explain how I combine manual note-taking with AI-assisted software development.

## My Core Setup

### Community Plugins

I've curated a set of community plugins that enhance Obsidian's functionality:

1. **[Tasks Plugin](https://obsidian.md/plugins?id=obsidian-tasks-plugin)** - The game-changer for task management. Tasks are written as simple checkboxes in your notes with dates, and they automatically appear in queries. This means you can write meeting notes, add tasks with due dates, and they'll automatically show up in your task queries - all while working with plain text files.

2. **[Kanban Plugin](https://obsidian.md/plugins?id=obsidian-kanban)** - For visual project management. Great for organizing ideas and tracking project progress.

3. **[Excalidraw Plugin](https://obsidian.md/plugins?id=obsidian-excalidraw-plugin)** - For creating diagrams and sketches directly in Obsidian.

4. **[Icon Folder](https://obsidian.md/plugins?id=obsidian-icon-folder)** - Adds visual organization to your file explorer.

5. **[File Color](https://obsidian.md/plugins?id=obsidian-file-color)** - Color-code files and folders for better visual navigation.

6. **[Autosave Control](https://obsidian.md/plugins?id=autosave-control)** - Critical for cloud sync setups. By default, Obsidian saves every keystroke, which can cause excessive sync activity. This plugin allows you to configure save intervals (I use 60 seconds) or manual saves with `Cmd+S`.

### Task Management Workflow

The Tasks plugin is particularly powerful. Here's how I use it:

```markdown
- [ ] Blog article about Obsidian and AI work 📅 2025-11-10
- [ ] Message family about gift exchange 📅 2025-11-08
- [ ] Plan raclette party 📅 2025-12-01
```

These tasks automatically appear in queries like:
- All tasks due today
- All tasks due this week
- All tasks without a due date
- Tasks by tag or project

The beauty is that tasks live in context - in meeting notes, project documentation, or daily notes - but are queryable across your entire vault.

## Synchronization Setup

### Desktop (macOS)

My Obsidian vault lives in my OwnCloud directory, which automatically syncs to my self-hosted cloud storage. The OwnCloud desktop client handles the synchronization seamlessly.

**Key configuration**: I use the Autosave Control plugin to save only every 60 seconds or on manual save (`Cmd+S`). This dramatically reduces the number of sync operations compared to Obsidian's default behavior of saving on every keystroke.

### Mobile (Android)

On Android, I use [Autosync - File Sync & Backup](https://play.google.com/store/apps/details?id=com.ttxapps.autosync) (one-time purchase of €9.99). The official OwnCloud app has limitations - it can only write to its own app directory, which Obsidian can't access. Autosync allows syncing to any directory on the device, making it perfect for Obsidian.

**Why this matters**: Having my vault synced to Android means I can:
- Review and update tasks on the go
- Add quick notes or meeting minutes
- Access my knowledge base anywhere

The Tasks plugin works perfectly on mobile, so I can manage my task list from my phone just as easily as from my desktop.

## AI Integration: The Cursor Connection

Here's where it gets really interesting. Since Obsidian stores everything as plain Markdown files, I can integrate my knowledge base directly into my development workflow.

### Workspace Integration

I (currently) use [Cursor](https://cursor.sh/) as my IDE for AI-assisted development. My main IDE is still Jetbrains Rider and other Jetbrains Products, but most of the time I work in parallel with Cursor. Cursor supports workspaces that can include multiple projects and directories. I've configured my workspace to include:

- My backend projects
- Frontend applications
- Microservices in development
- **My Obsidian vault**

This means when I'm working with AI assistance in Cursor, the AI has access to:
- My project documentation
- Requirements and specifications
- Meeting notes and decisions
- Technical notes and architecture diagrams
- Task lists and project plans

### Practical Benefits

When I'm coding with AI assistance, I can reference:
- "Based on the requirements in `Customer/my-customer/current-task-requirements.md`..."
- "Following the architecture described in `MyKnowledgeBase/Architecture.md`..."
- "As noted in yesterday's meeting notes..."

The AI can read and understand my entire knowledge base, making it much more effective at helping me write code that aligns with my documented requirements and decisions.

## Setting Up Your Own System

### Step 1: Install Obsidian

1. Download [Obsidian](https://obsidian.md/) for your platform
2. Create a new vault (or use an existing folder)
3. Enable Community Plugins in Settings → Community Plugins

### Step 2: Install Essential Plugins

1. Open Settings → Community Plugins
2. Disable "Restricted Mode", restart and enable "Community plugins"
3. Browse for and install the following plugins:
   - **Tasks**
   - **Autosave Control**
   - **Kanban** (optional)
   - **Excalidraw** (optional)
   - **Icon Folder** (optional)
   - **File Color** (optional)

### Step 3: Configure Autosave Control

1. Open Settings → Community Plugins → Autosave Control
2. Configure to save only every 60 seconds

### Step 4: Set Up Cloud Sync

**Option A: Obsidian Sync (Easiest)**
- Obsidian offers official sync for $4/month
- Supports all platforms including mobile
- Includes end-to-end encryption

**Option B: Self-Hosted (My Setup)**
- **Desktop**: Use OwnCloud/Nextcloud desktop client
- **Android**: Use Autosync for OwnCloud
- **iOS**: Use any cloud sync app that supports folder sync

**Option C: Other Cloud Services**
- Dropbox, Google Drive, OneDrive all work
- Just point Obsidian to a synced folder

### Step 5: Integrate with Your IDE (Optional)

If you use Cursor or another AI-powered IDE:

1. Open your IDE workspace configuration
2. Add your Obsidian vault directory to the workspace
3. The AI will now have access to your notes

**Pro tip**: You can create a dedicated "AI Context" folder in your vault with frequently referenced documentation.

## Best Practices

1. **Use Daily Notes**: Create a daily note template for quick capture
2. **Link Everything**: Use `[[wikilinks]]` to connect related notes
3. **Tag Consistently**: Use tags for categorization and filtering
4. **Task Dates**: Always add dates to tasks using the Tasks plugin syntax
5. **Regular Backups**: Even with cloud sync, consider Git backups for important vaults

## Conclusion

Obsidian has transformed how I manage knowledge, tasks, and documentation. The combination of plain text files, powerful plugins, seamless sync across devices, and AI integration through IDE workspaces creates a productivity system that's both powerful and flexible.

Whether you're a developer, writer, researcher, or knowledge worker, Obsidian's approach to knowledge management is worth exploring. The fact that everything is just Markdown files means you're never locked in, and you can integrate with any tool that works with text files - including AI assistants.

By the way: there certainly are some or even many [plugins for Obsidian](https://obsidian.md/plugins) that can be used to work with AI assistants. However, my point is not to edit content of Obsidian files, but to use the content of Obsidian files as context for a lot of other work. In my experience, Obsidian is the perfect tool for just that.

**A real-world example**: I used this exact setup to create this blog article. I wrote the initial draft in Obsidian, referenced my daily notes and task list, and then used Cursor with my Obsidian vault included in the workspace to refine and structure the article. The AI had access to my website (also written in Markdown with [Hugo](https://gohugo.io/)) and my notes about the setup, helping me create this comprehensive guide. Of course, I always review and refine the AI-generated content to ensure it accurately reflects my workflow.

If you're interested in learning more about Obsidian or have questions about my setup, feel free to reach out!

## Resources

- [Obsidian Official Website](https://obsidian.md/)
- [Obsidian Community Plugins](https://obsidian.md/plugins)
- [Tasks Plugin Documentation](https://obsidian-tasks-group.github.io/obsidian-tasks/)
- [Cursor IDE](https://cursor.sh/)
- [Autosync for OwnCloud (Android)](https://play.google.com/store/apps/details?id=com.ttxapps.drivesync)


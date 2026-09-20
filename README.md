# Gutternote plugins

Plugins for using [Gutternote](https://gutternote.com) from a coding agent.
Gutternote is a comment toolbar you drop onto a website with one script tag:
people click an element on the live page, say what is wrong, and the note goes to
a shared list that a coding agent can read and reply to.

## Install (Claude Code)

```
/plugin marketplace add gutternote/gutternote-plugins
/plugin install gutternote@gutternote-plugins
```

The first time the agent uses it, your browser opens, you sign in to Gutternote and
choose a project, and the agent is connected to that one project. There is no token
to create or paste.

## What is in it

**`gutternote`** connects the agent to Gutternote's hosted MCP server
(`https://api.gutternote.com/mcp`) and adds two skills:

| Skill | For |
|---|---|
| `work-notes` | Working the notes people have left: find the ready ones, read each in full, make the change, report progress, and resolve it with evidence. |
| `install-widget` | Putting the widget on your website, following Gutternote's documentation, including a Content Security Policy if your site has one. |

Ask in plain words — "work the ready Gutternote notes", "set up the Gutternote
widget on this site" — and the skill loads itself.

## Other MCP clients

You do not need this plugin to use Gutternote from an agent. Point any client that
supports the MCP authorization spec at `https://api.gutternote.com/mcp`. See
[Connect a coding agent](https://docs.gutternote.com/guides/connect-a-coding-agent/).

## A note about notes

A note's text is written by whoever commented on the page, guests included. The
skills tell the agent to read it as a description of what to change and never as
instructions to itself, and a note only reaches an agent after a member of the
project has offered it. If you build your own agent on this, do the same.

## Where this comes from

These files are published from Gutternote's main repository, which is where they
are changed. Issues are welcome here.

## Licence

[MIT](LICENSE).

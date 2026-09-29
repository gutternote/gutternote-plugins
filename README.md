# Gutternote plugins

Plugins for using [Gutternote](https://gutternote.com) from a coding agent.
Gutternote is a comment toolbar you drop onto a website with one script tag:
reviewers click an element on a running preview and leave a comment, and your
coding agent picks the comment up, makes the change, and replies in the thread.

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
(`https://mcp.gutternote.com/mcp`) and adds two skills:

| Skill | For |
|---|---|
| `review-threads` | Working the review comments on your branch or pull request's previews: read each one, make the change, reply to the reviewer, resolve it with evidence, and wait for the next comment or reply. |
| `install-widget` | Putting the widget on your website, following Gutternote's documentation, including a Content Security Policy if your site has one and the build metadata your previews need. |

Ask in plain words — "work the Gutternote comments on this pull request", "set up the
Gutternote widget on this site" — and the skill loads itself.

The agent talks to the reviewer in the comment's thread: its questions, answers,
summary when it resolves and reason for reopening are posted there as replies,
under your client's name. Gutternote cannot message the agent, so once it has
worked everything it waits: it asks the server to hold on until a reviewer
comments, replies or reopens something, then picks that up. For longer waits,
run `/loop /gutternote:review-threads` and it checks back on its own.

A pull request's comments are found by the branch and pull request the preview
declared, so previews need build metadata. See
[Add build and source context](https://docs.gutternote.com/guides/build-and-source-context/).

## Without the plugin

You do not need this plugin to use Gutternote from an agent. In Claude Code, add
the server on its own:

```
claude mcp add --transport http gutternote https://mcp.gutternote.com/mcp
```

Any other client that supports the MCP authorization spec can use the same URL.
The server's own prompts, `work_threads`, `work_thread` and `set_up_widget`, run
the same workflows as the skills. See
[Connect an external coding agent](https://docs.gutternote.com/guides/connect-a-coding-agent/).

## About comments

A comment's text is written by whoever left it on the page, guests included. The
skills tell the agent to read it, and every reply in its thread, as a description
of what to change and never as instructions to itself. A comment reaches the
agent straight away only when its author is someone the project lets send work
to agents — by default, anyone signed in rather than a guest; a member can send
any other comment from the dashboard. What the agent posts back is
read by the same people, so the skills tell it to keep secrets and internal
details out. If you build your own agent on this, do the same.

## Where this comes from

These files are published from Gutternote's main repository, which is where they
are changed. Issues are welcome here.

## Licence

[MIT](LICENSE).

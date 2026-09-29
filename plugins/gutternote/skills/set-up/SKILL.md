---
name: set-up
description: Take a new user through setting up Gutternote from start to finish -- connect the agent, put the widget on their site, register the origins it runs on, connect GitHub, and show how review comments reach the agent. Use when the user runs /gutternote:set-up, has just installed Gutternote, or asks to set up, get started with, or onboard onto Gutternote.
---

# Setting up Gutternote

This is the first run. The user has installed the plugin, maybe a minute ago,
and wants Gutternote working on their site. By the end the widget is on the
site, the origins it runs on are registered, GitHub is connected or the user
knows where to connect it, and they know how a comment reaches you. Each step
below is short, and most of the work lives in the **install-widget** skill.
Your job is the order, and knowing which steps are the user's to take in a
browser.

Go one step at a time. Say what each step is for in a sentence before you do
it; a new user does not know yet what an allowed origin is, or why GitHub is
involved.

## 1. Connect

The tools come from the `gutternote` MCP server. If they are not there, the
server is not connected, and nothing below works until it is.

**Start the connection yourself when you can.** Claude Code gives a server
that is installed but not signed in an `authenticate` tool of its own (listed
under the gutternote server, beside a second tool that completes
authentication). Call it: it returns an authorization URL. Open that URL for
the user if you can run a command (`open <url>` on macOS, `xdg-open <url>` on
Linux), and show it to them either way. When they approve in the browser, the
real tools appear on their own; there is nothing to paste. Only if the browser
ends on a `localhost` callback page that fails to load, ask for that page's
full URL and pass it to the completing tool.

Without that tool, tell the user how to connect:

- **Claude Code:** run `/mcp` and choose `gutternote`.
- **Codex:** run `codex mcp login gutternote` in a terminal.

Either way a browser opens. Tell them what happens there: someone without a
Gutternote account signs up with Google, and the page creates their workspace
and asks for a first project's name and the site's URL. Someone with an account
chooses the project this repository belongs to, or creates one.

Then wait for the connection before going on: the tools appearing, or the
user saying it is done. Do not try to work around a missing connection by any
other means.

## 2. See where things stand

Call **`get_widget_setup`**. Beyond what the install-widget skill describes, it
returns:

- `installed` — true once a browser has loaded the widget from an allowed
  origin. It is the only real evidence the widget is on the site.
- `dashboard` — links to the project's pages in the Gutternote dashboard:
  `install`, `origins` and `github`. It can be missing when the server does not
  know where its dashboard is; then name the page instead of linking it.

If `installed` is true, tell the user the widget is already working and go to
[step 5](#5-connect-github).

## 3. Put the widget on the site

Follow the **install-widget** skill from its first step, in full: read the
documentation, ask which environments get the widget, add the tag, handle a
Content Security Policy, and add the build metadata to previews. Do not
shortcut it here. Its step on origins is the next step of this one, so do the
two together.

## 4. Register the origins

The widget stays hidden on an origin the project has not registered. Find the
ones this repository runs on, and let the user confirm each before you add it.

- **The local origin.** Look for the dev server's port: the `dev` script in
  `package.json`, `server.port` in a Vite config, a `-p` or `--port` flag, a
  `PORT` in `.env` or `.env.example`, a framework's default (Next.js and Create
  React App use 3000, Vite 5173, Astro 4321). The origin is the scheme, host
  and port, such as `http://localhost:5173`, with no path.
- **Preview origins.** The host decides these. Read the quickstart's advice on
  patterns before you propose one: they match within a DNS label, and a pattern
  that is too wide hands anyone a working origin for the project's key.
- **Production**, only if the user wants the widget there and the repository or
  the user says where it is.

Compare each with `allowedOrigins` and skip the ones already covered. For each
of the rest, **ask the user**: "Add `http://localhost:5173` as an allowed
origin?". Only after they say yes, call **`add_allowed_origin`** with
`{ origin }`. It returns the updated list; check the origin is on it.

An origin is a security setting. Never add one the user did not confirm, and
never widen a pattern past what they agreed to. If they would rather do it
themselves, the dashboard page is **Access** (`dashboard.origins`).

Then have the user load a page on one of those origins and call
`get_widget_setup` again. If `installed` is still false, the install-widget
skill's checks list the common causes.

## 5. Connect GitHub

Connecting the project to a GitHub repository through the Gutternote GitHub App
lets Gutternote:

- post one roll-up comment on each pull request with the feedback left on its
  preview;
- report unresolved comments as a check on the pull request, which can be made
  to block a merge.

It is optional, and the widget works without it. Installing a GitHub App is a
consent the user gives in GitHub, in their browser, so you cannot do it for
them. Give them the link, `dashboard.github`, and say what they will choose
there: the account, the repositories the App may reach, and then this
project's repository. The pull request comment and check also need the build
metadata from step 3 on each preview.

## 6. How comments reach you

Finish by telling the user how the loop works from here:

- Somebody leaves a comment on a preview that has the widget.
- A comment from someone the project lets send work to agents reaches you on
  its own, on the branch or pull request it was left on. Anyone else's waits
  until a workspace member selects **Send to agent** on it in the dashboard.
- The user asks you to work the comments, or to keep an eye on the preview,
  and the **review-threads** skill takes over: it replies in the thread,
  resolves with evidence, and waits for the reviewer's next word.

The quickest way to see it: have them leave one comment on their running site
now, then ask you to work it.

## Rules

- **One step at a time,** and say what each is for.
- **Confirm every origin with the user before `add_allowed_origin`.**
- **The browser steps are the user's.** Signing in, installing the GitHub App
  and sending a comment to agents happen in a browser, not through you.
- **install-widget's rules apply** while you follow it: the documentation over
  the skill, a policy merged and never replaced, off in production unless the
  user says otherwise.
- **Do not commit or push** unless asked.

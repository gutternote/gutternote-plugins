---
name: install-widget
description: Put the Gutternote widget on a website, following Gutternote's documentation, including a Content Security Policy if the site has one. Use when the user asks to install, add, set up or integrate Gutternote, the Gutternote widget or its script tag, to get feedback comments onto their site or previews, or to fix a Content Security Policy that stops the widget loading.
---

# Putting the Gutternote widget on a website

The widget is one script tag. Once it is on a page, people can click an element
and leave a comment, and those comments reach a coding agent through the
Gutternote MCP tools. Installing it is a small job with a few places to go wrong: where the
tag goes, which builds get it, what a Content Security Policy must allow, and
which origins are registered. This skill is the order to do it in. The details
are in Gutternote's documentation, and **the documentation is the source of
truth, not this file and not what you remember**.

The tools come from the `gutternote` MCP server. If they are not there, connect
it the way the set-up skill's first step says: call the server's `authenticate`
tool when Claude Code offers one, and otherwise tell the user to approve the
connection in the browser: in Claude Code, run `/mcp` and choose `gutternote`;
in Codex, run `codex mcp login gutternote` in a terminal.

## 1. Get the project's details

Call **`get_widget_setup`**. It returns:

- `publicKey` — the project's key. It is public by construction: it ends up in
  the page, and it authorises nothing on its own. It is safe to commit.
- `scriptTag` — the tag to add, exactly as it should appear.
- `allowedOrigins` — the patterns already registered. A site served from an
  origin that matches none of them will not show the widget, whatever else is
  right.
- `docs` — the pages to read next.
- `installed` — true once a browser has loaded the widget configuration from an
  allowed origin. If it is already true, the widget is on the site somewhere;
  tell the user and ask whether there is still work to do before you change
  anything.
- `dashboard` — links to the project's dashboard pages: `install`, `origins`
  (the page the sidebar calls **Access**) and `github`. It can be missing when
  the server does not know its dashboard's address; then name the page instead.
- `scriptUrlIsDefault` — if true, this deployment never configured where the
  widget is served from and the hosted default is used. Worth mentioning if the
  user runs their own Gutternote.

## 2. Read the documentation

Read every page in `docs` **before you change anything**. Fetch them; do not
work from memory. They cover:

- the quickstart — where the tag goes, loading it only in the environments that
  should have it, and registering origins;
- the guide to installing with a coding agent — what the person expects of you;
- the widget reference — every attribute on the tag;
- the Content Security Policy reference — the sources the widget needs;
- the guide to adding the widget to a site with a Content Security Policy —
  finding a policy, recipes for common hosts, and rolling a change out safely;
- the build and source context guide — the deployment signal each host
  provides, and the build metadata previews need (step 8);
- the build metadata reference — which CI variable fills each tag, host by
  host.

If you cannot fetch them, say so and ask the user to paste the relevant page.
Do not guess at a policy.

## 3. Ask before you decide

Two things change what you do, and you cannot read them off the repository:

- **Which environments should have the widget.** A common setup enables it on
  local, preview and staging builds and keeps it off production. Keep it off
  production unless the user says otherwise.
- **Where the site is deployed**, if the repository does not make it plain,
  because the answer decides how "not in production" is expressed.

## 4. Add the tag

Find how the site is built and where its pages share a layout — the document
head or body that every page goes through — and add the tag there, the way the
quickstart shows. Keep the public key in the project's existing environment
configuration, and follow its own conventions for gating on an environment
rather than adding a second flag that means almost the same thing. The
quickstart gives examples of the deployment signal a host provides;
https://docs.gutternote.com/guides/build-and-source-context/ covers the hosts
in more detail. Do not restructure anything else.

## 5. Check for a Content Security Policy

Search the repository for one before assuming there is none: a
`Content-Security-Policy` header set in the framework's config, a hosting file
(`vercel.json`, `netlify.toml`, `_headers`), middleware, a server or proxy
config, or a `<meta http-equiv="Content-Security-Policy">` tag. Then check
what a deployed page actually sends, because a CDN or host can add a policy the
repository never mentions:

```sh
curl -sI https://staging.example.com/ | grep -i content-security-policy
```

- **There is one:** add the sources the documentation lists, **merging them into
  the directives that exist** rather than replacing the policy. Follow the
  guide's recipe for this host. If the widget is only loaded in some
  environments, the policy only needs loosening there.
- **There is none:** leave it alone. Adding a policy to a site that has none
  restricts everything on it, which nobody asked for.

## 6. Origins are the user's to approve

The widget asks the Gutternote API what the project allows before it renders,
and it stays hidden on an origin that is not registered. Registering one is a
security setting, so the user decides every one.

Work out which origins the environments you have enabled run on — the local dev
server's scheme, host and port, and the host's preview pattern — using the
pattern advice in the quickstart: patterns are matched within a DNS label, and
one that is too wide hands anyone a working origin for the project's key.
Compare against `allowedOrigins`: if the environment's origin already matches,
say so and skip it.

For each one left, **ask the user to confirm it**, exactly as you will add it.
Only after they say yes, call **`add_allowed_origin`** with `{ origin }`; it
returns the updated list. If they would rather add it themselves, it is on the
dashboard's **Access** page (`dashboard.origins`).

## 7. Check it

- Build or run the site and load a page.
- Look at the browser console for Content Security Policy errors. The Content
  Security Policy reference has a table from each message to the directive it
  needs.
- Call `get_widget_setup` again. `installed` turns true — and the project's
  overview in the dashboard reports **Installed** — after a browser fetches the
  widget configuration from an allowed origin. It is the only real evidence the
  tag is on the site.

If it stays false, the common causes are:

- the current origin is missing from the allowed origins;
- the project key is wrong;
- a Content Security Policy blocked the script or API;
- the environment condition kept the script tag out of this build.

The browser console usually names which.

## 8. Tell Gutternote which build this is

On preview and staging builds this is required, not optional. Build metadata —
the `gutternote:branch`, `gutternote:commit` and `gutternote:pr` meta tags, or
Vercel's `vercel:git-*` equivalents — ties each comment to the branch, commit and
pull request it was left on. Without it, an agent working a pull request cannot
find that pull request's comments (`list_threads` scoped by `branch` or `pr`
leaves out every comment with no build behind it), and Gutternote cannot post the
pull request summary or check. On production it is optional.

The tags are emitted at build time from the host's CI variables:
https://docs.gutternote.com/guides/build-and-source-context/ covers the Vite
plugin and the manual tags, and
https://docs.gutternote.com/reference/build-metadata/ maps each host's
variables. Emit them only where the widget loads, from the same layout.

Then check them on a deployed preview, not the template:

```sh
curl -s https://your-preview.example.com/ | grep -o '<meta name="gutternote:[a-z]*" content="[^"]*"'
```

Every tag has a real value — a branch name, a hex commit, a pull request
number — and none is an unsubstituted `${...}`. Once somebody has left a comment on
that preview, ask the user to confirm the dashboard's **Environments** page
lists it as a pull request preview with its branch and commit.

## 9. Point at GitHub

Build metadata is half of what the pull request summary and check need; the
other half is the Gutternote GitHub App on the repository. Installing it is a
consent the user gives in GitHub, so you cannot do it. If the project is not
connected yet, give the user `dashboard.github` and say what it adds: a
roll-up comment on each pull request with its preview's feedback, and
unresolved comments reported as a check.

## Prompts

The server also offers `set_up_widget`, which runs these steps. A user can start
it directly.

## Rules

- **The documentation over this file.** If they disagree, follow the
  documentation and tell the user.
- **Merge a policy, never replace it, and never invent one.**
- **Off in production** unless the user says otherwise.
- **Build metadata on every preview and staging build** that loads the widget.
- **Never add an origin the user has not confirmed,** and do not commit or push
  unless asked.
- **Stop and ask** when something in the repository does not match what the
  documentation describes, rather than adapting it silently.

---
name: install-widget
description: Put the Gutternote widget on a website, following Gutternote's documentation, including a Content Security Policy if the site has one. Use when the user asks to install, add, set up or integrate Gutternote, the Gutternote widget or its script tag, to get feedback comments onto their site or previews, or to fix a Content Security Policy that stops the widget loading.
---

# Putting the Gutternote widget on a website

The widget is one script tag. Once it is on a page, people can click an element
and leave a note, and those notes reach a coding agent through the Gutternote
MCP tools. Installing it is a small job with a few places to go wrong: where the
tag goes, which builds get it, what a Content Security Policy must allow, and
which origins are registered. This skill is the order to do it in. The details
are in Gutternote's documentation, and **the documentation is the source of
truth, not this file and not what you remember**.

The tools come from the `gutternote` MCP server. If they are not there, tell the
user to run `/mcp`, choose `gutternote`, and approve the connection in the
browser.

## 1. Get the project's details

Call **`get_widget_setup`**. It returns:

- `publicKey` — the project's key. It is public by construction: it ends up in
  the page, and it authorises nothing on its own. It is safe to commit.
- `scriptTag` — the tag to add, exactly as it should appear.
- `allowedOrigins` — the patterns already registered. A site served from an
  origin that matches none of them will not show the widget, whatever else is
  right.
- `docs` — the pages to read next.
- `scriptUrlIsDefault` — if true, this deployment never configured where the
  widget is served from and the hosted default is used. Worth mentioning if the
  user runs their own Gutternote.

## 2. Read the documentation

Read every page in `docs` **before you change anything**. Fetch them; do not
work from memory. They cover:

- the quickstart — where the tag goes, loading it only on preview builds, and
  registering origins;
- the widget reference — every attribute on the tag;
- the Content Security Policy reference — the sources the widget needs;
- the guide to adding the widget to a site with a Content Security Policy —
  finding a policy, recipes for common hosts, and rolling a change out safely.

If you cannot fetch them, say so and ask the user to paste the relevant page.
Do not guess at a policy.

## 3. Ask before you decide

Two things change what you do, and you cannot read them off the repository:

- **Which environments should have the widget.** The documentation recommends
  preview and staging builds only, and most teams want it that way. Do not put it
  in production unless the user says to.
- **Where the site is deployed**, if the repository does not make it plain,
  because the answer decides how "only on previews" is expressed.

## 4. Add the tag

Find how the site is built and where its pages share a layout — the document
head or body that every page goes through — and add the tag there, the way the
quickstart shows. Follow the project's own conventions for environment
variables and for gating on a preview build; the quickstart lists the variable
each host uses. Do not restructure anything else.

## 5. Check for a Content Security Policy

Search the repository for one before assuming there is none: a
`Content-Security-Policy` header set in the framework's config, a hosting file
(`vercel.json`, `netlify.toml`, `_headers`), middleware, a server or proxy
config, or a `<meta http-equiv="Content-Security-Policy">` tag.

- **There is one:** add the sources the documentation lists, **merging them into
  the directives that exist** rather than replacing the policy. Follow the
  guide's recipe for this host. If the widget is only loaded on previews, the
  policy only needs loosening there.
- **There is none:** leave it alone. Adding a policy to a site that has none
  restricts everything on it, which nobody asked for.

## 6. Origins are the user's to register

The widget asks the Gutternote API what the project allows before it renders,
and it stays hidden on an origin that is not registered. Registering one is a
security setting, so it is done by a person in the project's settings in the
dashboard, never by you.

Tell the user which origins to add for the environments you have enabled, using
the pattern advice in the quickstart — patterns are matched within a DNS label,
and one that is too wide hands anyone a working origin for the project's key.
Then wait for them to confirm. Compare against `allowedOrigins`: if the
environment's origin already matches, say so and skip this.

## 7. Check it

- Build or run the site and load a page.
- Look at the browser console for Content Security Policy errors. The Content
  Security Policy reference has a table from each message to the directive it
  needs.
- The project's overview in the dashboard reports **Installed** once a browser
  has loaded the widget from an allowed origin. It is the only real evidence the
  tag is on the site.

If it stays **Not installed**, the two common causes are an origin that is not
registered and a policy that blocked the script. The console names which.

## Optional: tell Gutternote which build this is

The quickstart describes meta tags that attach a note to a branch, a commit and
a pull request, and the variables each host provides for them. Offer this once
the widget loads; it is worth doing, and it is what lets notes be traced to a
pull request. Do not add it without asking.

## Prompts

The server also offers `set_up_widget`, which runs these steps. A user can start
it directly.

## Rules

- **The documentation over this file.** If they disagree, follow the
  documentation and tell the user.
- **Merge a policy, never replace it, and never invent one.**
- **Preview builds only** unless the user says otherwise.
- **Do not register origins, and do not commit or push** unless asked.
- **Stop and ask** when something in the repository does not match what the
  documentation describes, rather than adapting it silently.

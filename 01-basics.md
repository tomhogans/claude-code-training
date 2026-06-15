# Infrastructure Primer

A first-principles walkthrough of the tools and concepts you'll use to build and deploy web apps with Claude Code on Cloudflare. This assumes you're comfortable using a Mac but haven't written code or used a terminal before.

---

## 1. The Command Line Interface (CLI)

### What it is

The CLI is a way of using your computer by **typing commands** instead of clicking icons. On your Mac, you open it through the **Terminal** app (in `/Applications/Utilities/`). You get a blinking cursor and a prompt; you type a command, press Return, and the computer runs it.

### A brief history

Before Macs and Windows existed, there were no windows, no mouse, and no icons. Computers in the 1960s and 70s — especially those running **UNIX**, an operating system developed at Bell Labs in 1969 — were controlled entirely through text. You typed; the machine typed back.

When graphical interfaces arrived in the 1980s, the CLI didn't go away. macOS is actually built on top of a UNIX-like system (specifically Darwin/BSD), and that original text interface is still there, underneath the pretty windows. Every professional developer still uses it daily, because for many tasks it's faster and more powerful than clicking.

### Why it's still around: composability

UNIX was designed around a simple idea: **build many small programs that each do one thing well, and let them be combined**. Each command is a tool. You can chain tools together to get complex results.

For example, the `|` symbol (called a "pipe") takes the output of one command and feeds it into another:

```
ls | grep report
```

- `ls` lists files in the current folder.
- `grep report` filters for lines containing the word "report".
- The pipe glues them together: "list the files, then show me the ones with 'report' in the name."

You can't do that in Finder without a lot of clicking. In the CLI it's one line.

### A few practical examples

Search inside files for a word:
```
grep "hello" notes.txt
```

Move every `.jpg` file from Downloads to Pictures:
```
mv ~/Downloads/*.jpg ~/Pictures/
```
The `*` is a **wildcard** — it means "match anything." So `*.jpg` means "every file ending in `.jpg`."

List the 10 largest files in a folder:
```
du -h * | sort -h | tail -n 10
```
Three simple tools — measure sizes, sort them, show the last 10 — combined into something useful.

The takeaway: the CLI feels intimidating at first, but its power comes from this idea of small programs you can snap together like Lego.

### Why the CLI is a superpower for AI assistants

There's a modern reason the CLI matters more than ever: it's the ideal way for an AI assistant like **Claude** or **ChatGPT** to operate your computer.

Think about what a language model actually does — it reads text and writes text. That's its entire world. A graphical interface is the opposite of text: it's a picture made of pixels, with buttons, menus, and icons positioned at particular spots on the screen. For a human, that picture is friendly. For an AI, it's a wall. To click a button, the model would have to stare at a screenshot, work out *what* it's looking at, guess the button's coordinates, and move a virtual mouse there — slow, clumsy, and easy to get wrong, especially when a window moves or a layout changes.

The CLI sidesteps all of that, because it speaks the AI's native language:

- **Text in.** A command is just a line of text. The model can write one as easily as it writes a sentence — `git commit -m "Fix typo"` — with no coordinates, no clicking, no guessing where anything is on screen.
- **Text out.** The result comes back as text too: the output, any error messages, a success or a failure. The model can *read* that result and decide what to do next. This back-and-forth — run a command, read the response, run the next one — is a tight feedback loop the AI can drive entirely on its own.
- **Precise and repeatable.** A command means exactly one thing and does exactly the same thing every time. There's no ambiguity about which button was meant, and the exact steps can be reproduced, logged, and checked afterwards.

There's a bonus, too: the CLI has been how programmers work for fifty years, so the internet is full of examples of these commands and explanations of what they do. An AI trained on all that text has effectively read the manual for thousands of command-line tools.

This is exactly how **Claude Code** works. It lives in your terminal, and when you ask it to do something — create a file, make a commit, deploy your app — it writes the very same commands you're learning in this guide, runs them, reads the output, and reacts. Everything that makes the CLI powerful for *you* is the same thing that makes it powerful for the AI working alongside you.

---

## 2. Git and GitHub

### What is Git?

**Git** is software that tracks changes to files over time. Think of it as a very detailed "undo history" for an entire folder of work — but one that also lets many people work on the same folder in parallel without overwriting each other.

A few key ideas:

- Every time you reach a save-worthy point, you create a **commit** — a snapshot of every file, plus a note describing what changed.
- You can move back and forth through these snapshots.
- You can create **branches** — parallel timelines where you try something risky without disturbing the main version. If it works, you merge it back. If not, you throw it away.

### Decentralization: the key insight

Older version control tools had one central server. If it went down, nobody could work. Git is **decentralized**: every copy of the project contains the complete history. You can commit, branch, and inspect history entirely on your own laptop, with no network. You only talk to a server when you want to *share* your work with others.

This matters because it means Git itself doesn't need the internet, and no single company owns your project's history.

### What is a Git repository?

A **repo** is just a folder that Git is tracking. It has an invisible `.git` subfolder inside it that stores the complete history. When you "clone" a repo, you're copying that folder (with all its history) to your machine.

### What is GitHub?

Here is the distinction that trips people up:

- **Git** is software — a program that runs on your computer.
- **GitHub** is a website — a company (owned by Microsoft) that hosts Git repositories online.

GitHub is not Git. It's one of several services (others include GitLab and Bitbucket) that offer a convenient shared home for your repos on the internet. GitHub also layers extra features on top: issue tracking, code review ("pull requests"), automation, and permissions.

### How they work together

A typical flow:

1. You create a repo on your laptop using Git.
2. You make commits locally as you work.
3. When you want to back up or share your work, you **push** your commits to GitHub.
4. A teammate **pulls** those commits down to their laptop.
5. They make their own commits, push them, and you pull them back.

GitHub is the shared meeting point; Git is what actually does the tracking.

---

## 3. Where Your Code Runs: Cloudflare Workers

Claude Code helps you *write* code. But writing it is only half the job — that code needs a computer somewhere to actually run on, so other people can use it over the internet. In this training, that "somewhere" is **Cloudflare Workers**.

To understand what Workers is, first understand the classic setup it replaces.

### The classic web server lifecycle

When you visit a website, roughly this happens:

1. Somewhere in the world, there is a physical computer (a **server**) that is always on.
2. On that computer, a **process** is running. A process is just a program that's currently executing — like when you double-click an app on your Mac, the running version of it is a process.
3. This particular process is a **web server**. Its job is to sit and wait for incoming requests.
4. It listens on a specific **port**. Think of a port as a numbered mailbox on the computer — port 80 for regular web traffic, port 443 for secure web traffic. One computer has one address but thousands of ports, so many different services can coexist.
5. Your browser opens a connection to that address and port, sends a request ("please give me the homepage"), and waits.
6. The server builds a response and sends it back.
7. Your browser renders it as the page you see.

### Static vs. server-rendered responses

The response the server sends can be prepared two ways:

- **Static**: the file already exists on disk. The server just hands it over, unchanged, the same way every time. An image, a PDF, or a plain HTML page are typical examples. Fast and simple.
- **Server-rendered (dynamic)**: the server runs code to *build* the response on the fly, possibly looking things up in a database first. Your Twitter feed is different every time you load it because code generated it for you, right then, based on who you are.

Most real apps mix both.

### What Cloudflare Workers is

Running your own always-on server is a hassle: you have to buy or rent a computer, keep it patched, handle traffic spikes, worry about it crashing at 3am. **Cloudflare Workers** removes all of that.

Workers is Cloudflare's **serverless** platform. "Serverless" is a misleading name — there are still servers, you just don't manage them. You give Cloudflare your code, and they handle running it across their global network of data centers. When a request comes in, Cloudflare spins up your code on whichever data center is closest to the user, runs it, sends back the response, and shuts it down. You're billed for the small slice of time your code actually ran.

So the classic lifecycle still happens — request comes in, code produces response — but the "always-on process listening on a port" part is handled invisibly for you. You just write a function that takes a request and returns a response.

### Workers can be triggered by more than web requests

A web request is one kind of **trigger** — something that causes your code to run. Workers supports others:

- **Scheduled (cron) triggers**: a **cron** is a time-based schedule, a concept inherited from UNIX. You write an expression like "every day at 3am" or "every 15 minutes," and Cloudflare runs your code at those times. Useful for sending daily emails, cleaning up old data, fetching updates.
- **Incoming emails**: you can point an email address at a Worker, and any email sent there becomes a trigger that runs your code with the email's contents.
- **Queues, webhooks, and more**.

The mental model: *a Worker is a piece of code that gets woken up when something happens.* The "something" is just a configuration choice.

---

## 4. Static Sites vs. Server-Rendered Sites

You saw above that a server can respond with either a pre-made file (static) or something built on the fly (server-rendered). That distinction deserves its own section, because it's one of the biggest decisions you make when building a site.

### Static sites

A **static site** is a collection of files — HTML, CSS, images, maybe some JavaScript — that were prepared ahead of time. When a visitor requests a page, the server just hands over the file as-is. Every visitor sees the same thing. If you want to change the page, you edit the file and re-upload it.

Think of it like a printed magazine: the content is fixed at print time, and every reader gets an identical copy.

**Static sites are sufficient when:**

- The content is the same for every visitor (marketing pages, documentation, blogs, portfolios).
- Updates are infrequent enough that regenerating files is fine.
- You don't need to know *who* the visitor is.
- There's no data to look up at request time.

**Strengths:** blazingly fast (nothing has to be computed), cheap to host, hard to break, and trivially cacheable on a global network like Cloudflare's.

### Server-rendered sites

A **server-rendered site** builds the page when the request arrives. The server runs code that might check who you are, look things up in a database, call other services, and then assemble the HTML fresh for *this* visitor, *this* moment.

Think of it like a barista making your coffee to order: same shop, but every cup is prepared on the spot based on what you asked for.

**You need server rendering when:**

- The page must be personalized (a logged-in user's dashboard, a shopping cart, a feed).
- The content changes constantly (live scores, stock prices, inventory).
- You're accepting user input and reacting to it (form submissions, searches, posting a comment).
- Authentication or permissions decide what's shown.
- The data behind the page lives in a database that updates frequently.

**Trade-offs:** slower than static (code has to run on every request), more moving parts to maintain, and more expensive at scale.

### The hybrid reality

Most real apps mix both. A typical setup: the marketing homepage, docs, and blog are static; the logged-in app area is server-rendered; images and scripts are static assets served from a CDN. A good rule of thumb is **default to static, and only reach for server rendering when you actually need it** — personalization, fresh data, or user interaction. Cloudflare Workers handles both comfortably, so you don't have to pick one world and live in it.

---

## 5. Storing Data: Cloudflare D1

Your Worker often needs to remember things between requests — users, posts, orders. For that you need a **database**. In this training, we use **Cloudflare D1**.

### Think of a database like a strict spreadsheet

You already know how a spreadsheet works: rows and columns, each cell holding a value. A database table looks similar — rows and columns — but with stricter rules:

- Every column has a declared **type**: this column holds numbers, that one holds text, that one holds dates. You cannot put the wrong kind of value in.
- The shape of the table — its columns and their types — is called the **schema**. The schema is defined up front and enforced for every row.
- Columns can have rules like "must not be empty" or "must be unique."
- Databases are much faster than spreadsheets at searching and filtering, even across millions of rows.

So: spreadsheet = flexible, forgiving, slow at scale. Database = strict, structured, fast at scale.

### Migrations and Drizzle

Over time, your schema needs to change — you add a column, rename one, create a new table. You can't just edit the live database by hand: you need those changes applied in the same order, the same way, in development and production.

A **migration** is a small, ordered script that describes one change to the schema ("add a `published_at` column to the `posts` table"). Migrations stack up over time, and running them in order reproduces the current schema on any fresh database.

**Drizzle** is a tool that helps with this. You describe your schema in code, and Drizzle generates migration files for you whenever you change it. It also gives you a typed, safer way to write queries from your Worker instead of writing raw SQL by hand.

---

## 6. Deployment: From GitHub to Cloudflare via GitHub Actions

Writing code on your laptop is step one. **Deployment** is the step of getting that code onto Cloudflare so that the live world can use it.

You don't want to deploy manually every time, because that's error-prone and easy to forget. Instead you automate it.

### GitHub Actions

**GitHub Actions** is automation built into GitHub. You write a small configuration file (stored in your repo under `.github/workflows/`) that says: *when X happens, run these steps.*

A typical flow for this project:

1. You finish some work locally and push your commits to GitHub.
2. GitHub notices the push to the main branch.
3. GitHub Actions springs into action based on your workflow file.
4. It spins up a fresh temporary computer in the cloud.
5. On that computer, it checks out your code, installs dependencies, runs tests, and then runs the Cloudflare deploy command (using a secret API token you've stored in GitHub).
6. Cloudflare receives the new version of your Worker and makes it live, usually in seconds.
7. The temporary computer is thrown away.

The result: **push to GitHub → automatically deployed to Cloudflare**. You stop thinking about deployment as a chore and just think about it as "merging to main."

This pattern is called **continuous deployment**, and it's how most modern software ships.

---

## Putting it all together

Here is the full picture you've just built:

- You use the **CLI** to drive Git, run Claude Code, and interact with your project.
- You use **Git** to track every change to your code, with **GitHub** as the shared home for your repo.
- You write a **Cloudflare Worker**: code that runs on-demand when triggered by a web request, a cron, or an email.
- You store structured data in **D1**, a database whose shape is enforced by a schema, managed with **Drizzle** migrations.
- When you push to GitHub, **GitHub Actions** automatically deploys your Worker to Cloudflare.

Each piece is small. The power comes from how they compose — the same idea that made UNIX's pipes useful in 1970 is how modern web infrastructure is built today.

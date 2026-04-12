What is the CLI?  Brief history of UNIX, how CLI was original computer interface.  Benefits of composability of small, simple programs.  Some examples showing how to use CLI to search files for text, move files using wildcard patterns, etc. to demonstrate utility vs GUI.

What is Git?  (focus on decentralization)
What is a Git repo?
What is GitHub? (focus on Git being software, GitHub being a separate service)
How do they work together?

Claude Code helps write code; that code has to run somewhere; in our case, it runs on Cloudflare workers (explain what this is)

Explain lifecycle of normal web site running on a standard computer.  The server is an application process (explain what a process is), listening for connections on a certain port (explain what ports are), accepts the request, creates and replies with a response.  Explain difference between static and server-rendered response.

Now explain how Cloudflare workers simulates that standard setup.

Also explain how Cloudflare workers can be triggered by other things: scheduled crons (explain what crons are), incoming emails.  Web requests are one kind of trigger.

Explain how D1 database is similar to a spreadsheet but more strict and structured due to its defined schema.  Explain database migrations with Drizzle.

Now explain deployment process from Github to Cloudflare using Github Actions.

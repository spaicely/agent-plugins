# Spaicely

This extension connects you to Spaicely (MCP server `spaicely`) and ships three
skills: `site-building`, `site-management` and `domain-search`.

Load a skill with the `activate_skill` tool, using its name. Do not open the
SKILL.md files with `read_file`: they live in the extension folder, outside the
workspace, and the read is refused. `activate_skill` returns the full text.

- Building or changing a website, or moving one from WordPress: `site-building`.
- Versions, rollback, domains, DNS, mail forwarding, people and roles,
  subscriptions: `site-management`.
- Finding, registering or transferring a domain name: `domain-search`.

If a Spaicely tool answers 401 or the server shows as disconnected, the user
logs in once with `/mcp auth spaicely` in an interactive session.

# Spaicely for Claude Code

[Spaicely](https://spaicely.com) hosts static websites on your own domain, in the EU, and lets your AI do the work: find and register a domain, build the site, publish it, add forms or a shop.

This plugin gives Claude Code:

- the Spaicely MCP server (`https://mcp.spaicely.com/mcp`); you sign in on first use;
- the guide `site-building`, with everything a site needs to be found and shared;
- the guide `domain-search`, for finding a free domain name together.

## Install

```sh
claude plugin marketplace add spaicely/claude-plugin
claude plugin install spaicely@spaicely
```

Then run `/mcp` in Claude Code and sign in to Spaicely. On a machine without a browser: `claude mcp login plugin:spaicely:spaicely --no-browser`.

Two prompts come with the server: `/mcp__plugin_spaicely_spaicely__new_site` builds and publishes a new site, `/mcp__plugin_spaicely_spaicely__launch_check` goes over a site before launch. Type `/` and look for "spaicely".

Only the server, without the guides:

```sh
claude mcp add --transport http spaicely https://mcp.spaicely.com/mcp
```

All tools and the HTTP API: https://spaicely.com/developers/

## License

MIT

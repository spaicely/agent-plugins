# Spaicely for AI agents

[Spaicely](https://spaicely.com) hosts static websites on your own domain, in the EU, and lets your AI do the work: find and register a domain, build the site, publish it, add forms or a shop.

This repository is one plugin for several agents. It gives your agent:

- the Spaicely MCP server (`https://mcp.spaicely.com/mcp`); you sign in on first use;
- the guide `site-building`, with everything a site needs to be found and shared;
- the guide `site-management`, for running a site once it is built: versions and rollback, the domain and DNS, mail forwarding, people and roles, the hosting plan;
- the guide `domain-search`, for finding a free domain name together.

## Claude Code

```sh
claude plugin marketplace add spaicely/agent-plugins
claude plugin install spaicely@spaicely
```

Then run `/mcp` and sign in to Spaicely. Without a browser: `claude mcp login plugin:spaicely:spaicely --no-browser`. Two prompts come with the server: `/mcp__plugin_spaicely_spaicely__new_site` builds and publishes a new site, `/mcp__plugin_spaicely_spaicely__launch_check` goes over a site before launch.

## Codex

```sh
codex plugin marketplace add spaicely/agent-plugins
```

Then install Spaicely from the plugin list and sign in when asked.

## Gemini CLI

```sh
gemini extensions install https://github.com/spaicely/agent-plugins
```

Then run `/mcp auth spaicely` to sign in. Update later with `gemini extensions update spaicely`.

## Only the server

Any agent that speaks MCP over HTTP can use `https://mcp.spaicely.com/mcp` without this plugin, for example:

```sh
claude mcp add --transport http spaicely https://mcp.spaicely.com/mcp
```

All tools and the HTTP API: https://spaicely.com/developers/

## Files

| File | Read by |
| --- | --- |
| `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, `.mcp.json` | Claude Code |
| `plugin.json`, `mcp.json`, `.agents/plugins/marketplace.json` | Codex and ChatGPT (Agent Plugins format) |
| `gemini-extension.json` | Gemini CLI |
| `skills/*/SKILL.md` | all of them |

The guides are copies of the English ones in the Spaicely repository; the Dutch, German and French versions are at `https://spaicely.com/skills/<nl|de|fr>/`.

## License

MIT

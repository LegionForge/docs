# mcp-probe

**Connectivity and configuration advisor for MCP services you own or operate.**

[:fontawesome-brands-github: github.com/LegionForge/mcp-probe](https://github.com/LegionForge/mcp-probe){ .md-button .md-button--primary }

## What it does

mcp-probe configures, connects to, diagnoses, and recommends settings for [Model Context Protocol](https://modelcontextprotocol.io) servers that **you own or operate**. It is explicitly **not** a scanner. It does not enumerate, probe, or characterize MCP services on infrastructure you don't control.

The intended use cases:

- You're setting up a new MCP server and want to verify it's reachable, responsive, and properly configured.
- You're debugging an MCP integration that an agent framework is using.
- You want a recommendation for which MCP capabilities to enable for a given workload.

## Status

Active. Public.

## Why this scope (and not scanning)

Tools that scan third-party MCP services would be useful for security researchers but they're also useful for attackers. The LegionForge org's policy is to ship tools that improve *your* infrastructure, not tools that profile *others'*. mcp-probe is deliberately limited to targets you can prove ownership of.

## Quick use

```bash
pip install mcp-probe
mcp-probe configure --server http://localhost:3000
mcp-probe diagnose
mcp-probe recommend --workload "code-review-agent"
```

See the [repo README](https://github.com/LegionForge/mcp-probe#readme) for the full command reference.

## Integration with LegionForge

The LegionForge framework can act as both an MCP client and server. mcp-probe is useful when:

- Setting up the framework's MCP server endpoints and verifying clients can reach them
- Adding a new MCP server as a context source for the framework's agents
- Diagnosing connectivity when a registered MCP source stops responding

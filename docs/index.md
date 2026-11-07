---
title: SendNF MCP server for Brazilian invoices (NFS-e, NFC-e, NF-e)
description: Remote MCP server with 44 tools to look up a CNPJ, find the service code, preview an invoice against the official layout and issue it after a person approves.
---

# SendNF MCP server

[SendNF](https://sendnf.com/mcp/) is a Brazilian electronic invoice issuer. Its MCP server lets an AI agent
(Claude, which we tested, Cursor and other clients that speak MCP) prepare and issue Brazilian invoices by conversation: NFS-e (service
invoice), NFC-e (consumer invoice) and NF-e model 55 for Simples Nacional and MEI companies.

- **Endpoint:** `https://api.sendnf.com/mcp` (Streamable HTTP, no session)
- **Auth:** `Authorization: Bearer <key>`. Keys that start with `nt_test_` run in sandbox mode (no fiscal value);
  `nt_live_` keys issue real invoices. Without a key the server answers 401. In Claude Desktop you can sign in with
  your SendNF account instead of copying a key.
- **Official docs:** <https://sendnf.com/mcp/> (Portuguese)
- **Registry:** `io.github.sendnf/sendnf` in the [official MCP Registry](https://registry.modelcontextprotocol.io/v0/servers?search=sendnf)

This repository holds only the manifest, examples and the tool list. The product code is private.

## A conversation

> **You:** Issue the invoice for Acme: R$ 1,500 of software development, for October.
>
> **Agent:** Found service code 01.01.01 (software development). Pre-issued and checked against the official layout.
> The invoice is ready: Acme Tecnologia, R$ 1,500.00, ISS 2%. You are in production. Shall I issue it?
>
> **You:** Go ahead.
>
> **Agent:** Issued: NFS-e no. 1042 authorized. PDF and XML sent to the client's finance e-mail.

The example comes from the [MCP page](https://sendnf.com/mcp/). The agent finds the service code, runs the
pre-issue check and only issues after a person approves (a server-side lock, on by default for MCP).

## Connect

**Claude Code**

```bash
claude mcp add --transport http sendnf https://api.sendnf.com/mcp \
  --header "Authorization: Bearer nt_test_..."
```

**Cursor** (`~/.cursor/mcp.json`)

```json
{
  "mcpServers": {
    "sendnf": {
      "url": "https://api.sendnf.com/mcp",
      "headers": { "Authorization": "Bearer ${env:SENDNF_KEY}" }
    }
  }
}
```

**Claude Desktop:** Customize, Connectors, Add custom connector, and paste `https://api.sendnf.com/mcp`.
Then sign in with your SendNF account.

Start with a `nt_test_` key. Sandbox mode has no fiscal value and no charge, so you can try every flow first.

## Tools (44)

| Group | Tools |
|---|---|
| Lookup | `lookup_cnpj`, `search_service_codes`, `get_service_code`, `how_to_get`, `validate_dps` |
| Companies | `list_companies`, `find_company_by_external_id`, `get_company`, `create_company`, `update_company`, `check_readiness`, `go_live_status`, `import_invoice`, `create_onboarding_link`, `get_branding` |
| Account | `account_overview`, `usage_by_company` |
| Certificates | `upload_certificate`, `upload_proxy_certificate`, `list_proxy_certificates`, `link_proxy_certificate`, `unlink_proxy_certificate` |
| Preview and issue | `preview_nfse`, `issue_nfse`, `preview_nfce`, `issue_nfce`, `preview_nfe`, `issue_nfe` |
| Documents | `list_documents`, `get_document`, `get_document_events`, `cancel_document`, `get_substitution_draft`, `substitute_document`, `correct_nfe`, `retry_document` |
| Human approval | `create_note_draft`, `list_pending_approvals`, `get_approval_status` |
| Sharing and export | `create_share_link`, `list_share_links`, `revoke_share_link`, `create_export`, `get_export` |

The full list with descriptions is in [`tools.json`](https://github.com/sendnf/sendnf-mcp/blob/main/tools.json)
and in the [README](https://github.com/sendnf/sendnf-mcp#readme) (Portuguese).

## Links

- [SendNF MCP page](https://sendnf.com/mcp/) (setup for Claude, ChatGPT and Cursor)
- [API for developers](https://sendnf.com/desenvolvedores/) and [pricing](https://sendnf.com/precos/) (plans start at R$ 29.90 per month; sandbox mode costs nothing)
- [NFS-e issuing guide](https://sendnf.com/nfse/)
- [Source repository](https://github.com/sendnf/sendnf-mcp)

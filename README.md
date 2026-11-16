<p align="center"><img src="assets/icon-512.png" alt="cvrradar logo" width="96"></p>

# cvrradar

Danish company data from the CVR register, as a remote MCP server, a REST API and a command line tool.

cvrradar lets an AI assistant or your own code look up Danish companies: search by name, CVR number, domain or email, and read master data, people and roles, annual accounts, owners and group structure, a credit rating, and a digital profile of the company's website. The underlying data comes from the Danish Business Authority (Erhvervsstyrelsen) and published annual reports, synchronised nightly.

This repository holds documentation and examples only. The service itself runs at [cvrradar.dk](https://cvrradar.dk).

- Website: https://cvrradar.dk
- MCP and AI guide: https://cvrradar.dk/ai
- REST API: https://cvrradar.dk/api
- Pricing: https://cvrradar.dk/priser
- Machine-readable docs: https://cvrradar.dk/llms.txt, OpenAPI at https://cvrradar.dk/v1/openapi.json

Responses are in Danish.

## Remote MCP server

```
https://mcp.cvrradar.dk/mcp
```

Transport: Streamable HTTP. Authentication: OAuth 2.1 (you log in with your email), or an API key sent as `Authorization: Bearer cvr_...`. The server is also listed in the official MCP Registry as `dk.cvrradar/cvrradar`; see [`server.json`](server.json).

### Claude

Add a custom connector in Settings and paste the MCP address. Claude sends you to a login with your email.

Claude Code, with an API key:

```bash
claude mcp add --transport http cvrradar https://mcp.cvrradar.dk/mcp \
  --header "Authorization: Bearer cvr_YOUR_KEY"
```

### ChatGPT

Turn on developer mode, create a custom connector, paste the MCP address, choose OAuth and log in with your email. Alternatively import the OpenAPI file as a GPT Action: https://cvrradar.dk/v1/openapi.json (API key authentication, type Bearer).

### Cursor

In `.cursor/mcp.json` in your project, or in your global Cursor configuration:

```json
{
  "mcpServers": {
    "cvrradar": {
      "url": "https://mcp.cvrradar.dk/mcp",
      "headers": { "Authorization": "Bearer cvr_YOUR_KEY" }
    }
  }
}
```

### Other clients

Any client that supports remote MCP over Streamable HTTP works. In n8n, use the MCP Client Tool node with the address above and header authentication. For plain HTTP automation (Zapier, n8n HTTP Request) use the REST API below.

## Command line

```bash
npx @mahope/cvrradar
```

Commands: `soeg` (search), `vis` (show a company), `regnskab` (accounts), `kredit` (credit rating) and `nye` (new companies). Add `--json` for raw JSON in scripts. Package: https://www.npmjs.com/package/@mahope/cvrradar

## REST API

Base URL `https://cvrradar.dk/v1`. Create a free key at https://cvrradar.dk/api (keys start with `cvr_`) and send it as a bearer token.

```bash
curl -H "Authorization: Bearer cvr_YOUR_KEY" \
  "https://cvrradar.dk/v1/virksomhed/12345678"

curl -H "Authorization: Bearer cvr_YOUR_KEY" \
  "https://cvrradar.dk/v1/soeg?q=example&limit=5"
```

Main endpoints:

| Endpoint | Returns |
|----------|---------|
| `GET /v1/virksomhed/{cvr}` | One company by CVR number |
| `POST /v1/virksomheder` | Up to 500 companies in one call |
| `GET /v1/soeg?q=` | Search by name or CVR number |
| `GET /v1/domaene/{domain}` | The company behind a domain or email |
| `GET /v1/virksomhed/{cvr}/personer` | Owners, management and board |
| `GET /v1/virksomhed/{cvr}/regnskab` | Key figures per year (paid plan) |
| `GET /v1/virksomhed/{cvr}/kredit` | Warning signals and score 1-100 (paid plan) |
| `GET /v1/virksomhed/{cvr}/ejergraf` | Owners and group (paid plan) |
| `GET /v1/virksomhed/{cvr}/digital` | Website, SSL, platform, mail (paid plan) |
| `GET /v1/nye` | New companies with filters |

Errors always have the form `{"fejl": "text"}`. When the quota is used up the API answers `429` with a `Retry-After` header.

### Plans

The free key gives 1,000 lookups per day on master data and people, and 100 lookups per month with accounts, credit rating, owner graph and digital profile. The same key works for the API and the MCP server. Paid plans start at 99 DKK per month excl. VAT. See https://cvrradar.dk/priser for the current plans and limits.

## Tools

The MCP server exposes 19 tools, 6 prompts and 2 resources. Descriptions below are short summaries; the server returns the full descriptions with examples from `tools/list`.

| Tool | What it does |
|------|--------------|
| `soeg_virksomhed` | Find companies by name, CVR number, domain or email |
| `hent_virksomhed` | Master data, status and contact details for one company |
| `hent_regnskab` | Key figures per year from the annual reports |
| `kreditvurdering` | Credit score 1-100 with warning signals, alone and with the group |
| `ejergraf` | Owners, parent companies and subsidiaries |
| `personer_og_roller` | Management, board, owners and founders with roles and dates |
| `digital_profil` | Website, SSL, platform, speed, mail provider and social profiles |
| `seo_data` | Estimated traffic, keywords and backlinks for the company's domain |
| `nye_virksomheder` | Newly registered companies with filters on industry, municipality, form and date |
| `sammenlign_virksomheder` | Key figures for 2 to 5 companies side by side |
| `branche_benchmark` | A company's key figures against its industry median |
| `relaterede_virksomheder` | Group companies, shared management, shared address and auditor |
| `produktionsenheder` | Production units (shops, branches, departments) with address, industry and employees |
| `ejendomme` | Properties and registered charges (an empty answer means not checked yet, not "no properties") |
| `branchetal` | A company's placement in its industry per key figure, or an industry's distribution |
| `konkurser` | Latest bankruptcies and dissolutions, filtered on industry and municipality (companies only) |
| `find_virksomheder` | Advanced search with filters on industry, place, form, size and figures (counts one lookup per row returned) |
| `fuld_rapport` | The full company report as JSON (needs a purchase of the report or the Fuld plan) |
| `overvaag_virksomhed` | Follow a company and get notified of events by webhook or daily mail |

Accounts, credit rating, owner graph, digital profile and SEO data need a paid plan beyond the free monthly allowance (see the plans page).

## Prompts and resources

Prompts: `kredittjek_af_kunde` (credit check with a recommendation), `due_diligence` (owners, management, relations, history and production units), `branchebillede` (an industry, optionally in a municipality), `konkursovervaagning` (latest bankruptcies and suggested monitoring), `tjek_ny_kunde` and `nye_uden_hjemmeside`.

Resources: `cvrradar://virksomhed/{cvr}` (the company card as markdown) and `cvrradar://datakilder` (data sources and when each was last updated).

## Example prompts

- Check a new customer before I send an invoice: CVR 12345678.
- Find new hairdressers in Aarhus without a website.
- Compare key figures for the three biggest competitors in my industry.
- Who owns this company, and what else does the same owner control?

## Notes on use

Companies marked `reklamebeskyttet: true` have opted out of marketing contact and must not be used for direct marketing. The credit rating is an assessment based on public data, not advice. See https://cvrradar.dk/vilkaar.

## License

Documentation and examples in this repository are MIT licensed, see [LICENSE](LICENSE). The cvrradar service is governed by its own terms.

Made by [Mahoje](https://mahoje.dk), CVR 39413892.

# ContactDrive plugin for Cursor

Connect your agent to a [ContactDrive](https://www.contactdrive.io) workspace. ContactDrive is relationship intelligence for people whose work runs on who they know: fundraisers, campaigns, realtors, and independent professionals.

With this plugin your agent can:

- find, read, create, and update contacts, and log calls, meetings, and emails against them
- map who knows whom with typed relationships
- turn a plain-language description ("donors in Colorado I haven't contacted this year") into a filtered cohort, then hand it to a Connect session for one-by-one outreach
- answer questions with figures (counts, breakdowns, time series) without paging through records
- draft email campaigns (a person always picks the audience and sends in the app)
- build dashboards and saved views
- add custom fields, activity types, tags, and relationship types to the workspace

## What's included

| Component | Path | Purpose |
| --- | --- | --- |
| MCP server | `mcp.json` | The hosted ContactDrive MCP server at `https://mcp.contactdrive.io/mcp` (Streamable HTTP, stateless, OAuth) |
| Skill | `skills/contactdrive-relationships/` | How to find people, prep for conversations, log activity without creating duplicates, and build outreach cohorts |
| Skill | `skills/contactdrive-campaigns-dashboards/` | How to draft campaigns, read campaign reports, and build dashboards and saved views |

## Sign-in

There is no API key to paste. When the agent first uses ContactDrive, your client opens ContactDrive's sign-in page:

1. Log in to ContactDrive.
2. Review the permissions (scopes) the agent is asking for and approve.
3. You're returned to Cursor, connected.

Under the hood the server is an OAuth 2.1 protected resource (RFC 9728). The authorization server is ContactDrive Core (`https://core.contactdrive.io`), which supports dynamic client registration, PKCE (S256), and refresh tokens.

Every call is checked twice: once against the scopes you approved, and once against your role in the workspace. An agent can never do more than you could do yourself. If your login covers more than one workspace, each tool takes a `workspace` argument; `workspace_list` shows which workspaces the credential covers.

You need a ContactDrive account. Plans and the free trial are at [contactdrive.io/pricing](https://www.contactdrive.io/pricing).

## Tools

The tool list is built per credential on each request. Tools for an app (Connect, Email Marketing, Dashboard, Tasks) only appear when that app is installed in your workspace and you granted the matching scope.

### Read-only

| Tool | Scope | What it does |
| --- | --- | --- |
| `contacts_search` | `contacts:read` | Search contacts by name, email, phone, or external ID |
| `contacts_get` | `contacts:read` | Get one contact by ID |
| `contacts_query` | `contacts:read` | Structured field-level query: a page of matches, `paging.total`, and the echoed filters |
| `contacts_stats` | `contacts:read` | A figure over contacts (count, breakdown, time series, sum/avg) |
| `activity_list` | `activity:read` | List activity, optionally by contact or type |
| `activity_get` | `activity:read` | Get one activity record |
| `activity_stats` | `activity:read` | A figure over activity from a composed filter |
| `workspace_info` | `workspace:read` | Custom fields, activity types, and tags in the workspace |
| `workspace_list` | — | Workspaces this credential covers, with access tiers |
| `relationships_list` | `relationships:read` | Who a contact is connected to, and how |
| `relationships_get` | `relationships:read` | One relationship by ID |
| `connect_list_sessions` | `connect:read` | List your Connect sessions |
| `connect_get_session` | `connect:read` | One Connect session |
| `campaigns_list` | `email:read` | Email campaigns with name, subject, and status |
| `campaigns_get` | `email:read` | One campaign, including its content |
| `campaigns_get_report` | `email:read` | Sends, opens, clicks, bounces, per-link breakdown |
| `campaigns_list_templates` | `email:read` | The workspace's email templates |
| `dashboards_list` | `dashboards:read` | Dashboards with status and widget count |
| `dashboards_get` | `dashboards:read` | One dashboard's widgets and layout |
| `dashboards_vocabulary` | `dashboards:read` | Fields, operations, visualizations, and filter grammar for widgets |
| `dashboards_widget_data` | `dashboards:read` | One widget's figure, computed under your credential |
| `views_list` | `dashboards:read` | Saved views |
| `views_get` | `dashboards:read` | One saved view |

### Write

| Tool | Scope | What it does |
| --- | --- | --- |
| `contacts_create` | `contacts:write` | Create a contact, or update the one it matches (ContactDrive always deduplicates) |
| `contacts_update` | `contacts:write` | Update fields on an existing contact |
| `activity_create` | `activity:write` + `contacts:write` | Log an activity; the contact is created or updated as part of the call |
| `relationships_create` | `relationships:write` | Link two contacts with a typed relationship |
| `relationships_update` | `relationships:write` | Change a relationship's type or direction |
| `relationships_remove` | `relationships:write` | Delete a relationship (the contacts are untouched) |
| `connect_create_session` | `connect:write` | Create a Connect session from filters or contact IDs |
| `campaigns_create` | `email:write` | Create a **draft** campaign, optionally from a template |
| `campaigns_update` | `email:write` | Revise a draft in place |
| `dashboards_create_draft` | `dashboards:write` | Create a draft dashboard |
| `dashboards_upsert_widget` | `dashboards:write` | Add or revise a widget |
| `dashboards_remove_widget` | `dashboards:write` | Remove a widget |
| `dashboards_set_layout` | `dashboards:write` | Replace the dashboard layout |
| `views_create` / `views_update` | `dashboards:write` | Create or edit a saved view |
| `workspace_create_custom_field` / `workspace_update_custom_field` | `schema:write` | Add or edit a custom field |
| `workspace_create_activity_type` / `workspace_update_activity_type` | `schema:write` | Add or edit an activity type |
| `workspace_create_tag` | `schema:write` | Add a tag |
| `workspace_create_relationship_type` / `workspace_update_relationship_type` | `schema:write` | Add or edit a relationship type |

Apps installed in your workspace can add their own tools. Tasks, for example, adds `tasks_list`, `tasks_search`, `tasks_get`, `tasks_create`, `tasks_update`, `tasks_complete`, and `tasks_snooze` when you grant `tasks:read` / `tasks:write`.

### Built-in prompts

The server also offers four guided workflows as MCP prompts: **Weekly relationship review**, **Prep me for a conversation**, **Build an outreach list**, and **Log a conversation**.

## Safety boundaries

- **No sending.** Campaign tools write drafts only. There is no send, schedule, or test-send tool, and the `email:send` scope can't be granted to an agent. A person picks the audience and sends from the campaign page; every write returns a link to it.
- **No schema deletes.** Agents can add and rename fields, types, and tags, but can't delete them, because deleting a custom field deletes its values on every contact.
- **Untrusted text is marked.** CRM text is often written by people outside your organization. Results that carry free text list those fields in `untrustedFields`, so agents treat them as data, not instructions.
- **No stored data.** The MCP server holds no database and no secrets. It forwards each call to ContactDrive's API with your token.

## Support

- Email: [help@contactdrive.io](mailto:help@contactdrive.io)
- Product and account help: [contactdrive.io](https://www.contactdrive.io)
- Terms: [contactdrive.io/terms](https://www.contactdrive.io/terms)
- Privacy: [contactdrive.io/privacy-policy](https://www.contactdrive.io/privacy-policy)

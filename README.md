---
name: prepare-azure-rbac-requests
description: Prepare clear, reviewable Azure RBAC permission requests from notes, tables, emails, or raw lists. Use when organizing Azure subscriptions and IDs, resource groups, resource or subscription scopes, managed identities, service principals, Entra groups, users, role assignments, JSON permission files, validation commands, or implementation-ready access matrices. Also use when updating an earlier RBAC request, expanding assignments across environments, or separating requested access from optional recommendations.
---

# Prepare Azure RBAC Requests

Turn incomplete Azure access notes into a normalized request without inventing identifiers or expanding the requested access.

## Workflow

1. Extract subscription name and ID, environment, resource group, resource name/type, explicit scope or resource ID, principal name/type/object ID, and roles.
2. Normalize scope to the narrowest scope explicitly requested. Do not silently promote resource access to resource-group or subscription scope.
3. Create one record per unique principal and scope. Keep multiple roles in that record's `roles` array. Deduplicate only exact duplicate roles.
4. Preserve the user's scope boundaries and exclusions. Do not add identities, assignments, policy operations, remediation, or deployment steps unless requested.
5. Flag missing or inconsistent values instead of guessing. Distinguish confirmed values from inferred values.
6. Check role names and scope compatibility. If a role appears unusual for the target resource, retain it under **Requested access** and place the concern under **Validation notes**.
7. Present a concise request summary, a review table, JSON, and validation notes. Add CLI, PowerShell, Bicep, or Terraform only when requested.

## Default output

Use this order unless the user specifies another format:

### Request summary

State the business purpose when supplied, assignment count, principals, environments, and scope level. Do not invent a justification.

### Access matrix

Use these columns:

| Subscription | Resource group | Resource / scope | Principal | Principal type | Roles |
|---|---|---|---|---|---|

Show one row per principal/scope record and combine its roles with `<br>`.

### JSON

Emit valid JSON with the schema in [references/json-schema.md](references/json-schema.md). Preserve the user's requested key style if they supply an existing JSON format; otherwise use the default schema.

### Validation notes

List only actionable items:

- missing subscription IDs, object IDs, resource IDs, or principal types;
- malformed or mismatched subscription IDs and scopes;
- roles whose names or applicability require confirmation;
- duplicate, overlapping, or broader-than-necessary assignments;
- custom roles that require an exact role name or definition ID.

## Normalization rules

- Treat `managed identity`, `MI`, `UAMI`, and `system-assigned identity` as distinct principal details; never collapse them when the exact type matters.
- Use `Managed Identity`, `Service Principal`, `Entra ID Group`, or `User` for display values when confirmed.
- Derive an environment only from explicit names such as `dev`, `sys`, `uat`, `prd`, or `prod`; otherwise leave it null.
- Prefer a supplied full Azure resource ID as the authoritative scope.
- Construct a scope only when all required components are confirmed:
  - subscription: `/subscriptions/{subscriptionId}`
  - resource group: `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}`
  - resource: supplied full resource ID or a fully specified provider/type/name path
- Preserve official role capitalization where known, such as `AcrPull`, `Key Vault Secrets User`, `Key Vault Certificate User`, `Storage Blob Data Contributor`, `Storage Queue Data Contributor`, `Azure Service Bus Data Sender`, `Azure Service Bus Data Receiver`, and `Log Analytics Reader`.
- Do not substitute similarly named roles. Mark uncertain spellings for validation.
- Label recommendations as optional and keep them outside requested JSON unless the user asks to include them.

## Scope and safety

Prepare artifacts only. Never create live Azure role assignments, modify Azure resources, or send a permission request unless the user explicitly asks for that action. When implementation is requested, preview the normalized assignments first and use stable principal object IDs rather than display-name lookup wherever possible.

## Optional implementation output

When asked for commands or code:

1. Generate one assignment per role per principal/scope record.
2. Parameterize subscription IDs, object IDs, and scopes.
3. Make scripts rerunnable by checking for an existing assignment before creation.
4. Include read-only verification using `az role assignment list` or `Get-AzRoleAssignment`.
5. Keep setup, assignment, and validation sections separate.

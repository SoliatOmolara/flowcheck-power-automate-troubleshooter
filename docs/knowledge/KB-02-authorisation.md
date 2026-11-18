---
Title: "KB-02 — Permissions and Access Control"
Description: "Evidence-based troubleshooting for Power Automate authorisation and permissions failures."
---

# KB-02 — Permissions and Access Control

> **FlowCheck knowledge reference** · Version **1.0.0** · Reviewed **18 November 2026**  
> **Coverage:** Microsoft Power Automate cloud flows; connection authentication and identity issues.  
> **Status:** Community troubleshooting guidance, not an official Microsoft support service.

[← Knowledge-base home](../index.html)

## 1. Purpose

This reference supports diagnosis of authorisation and access-control failures affecting Power Automate cloud flows.

It covers insufficient resource permissions, connection-identity mismatches, SharePoint access restrictions, mailbox delegation, connection ownership and Power Platform data policies.

The objective is to establish why a particular operation was refused, using evidence from the failed action and relevant access configuration.

This reference provides diagnostic guidance only. It does not authorise permission changes or security-policy exceptions.

## 2. Common Symptoms

Examples include:

- SharePoint Create item returns 403 Forbidden.
- SharePoint Get items cannot access a list.
- An Outlook action cannot send from a shared mailbox.
- A user can run a flow but cannot access a resource.
- A flow owner cannot activate a flow because a connection is not authorised.
- A flow works for one identity but not another.
- A solution import introduces different connection ownership or access requirements.
- An organisational data policy prevents a connector or connector combination from being used.

HTTP 403 generally indicates that a request was refused, but it does not establish which permission, service or policy caused the refusal.

HTTP 401 more commonly suggests an authentication problem. See KB-01 for authentication and connection failures.

## 3. Diagnostic Principles

Before recommending changes:

1. Identify the specific failing action or trigger.
2. Record the sanitised error code and message.
3. Identify the connector and relevant operation.
4. Establish which identity the connection uses.
5. Identify the target resource and environment.
6. Determine whether that identity has the necessary effective permissions.
7. Consider connection-sharing and organisational policies when relevant.
8. Recommend the least-privileged authorised correction.
9. Confirm the outcome through a controlled test.

Do not assume the flow owner and connection identity are the same person.

Do not treat a successfully authenticated connection as proof that the identity is authorised to perform every operation.

## 4. Diagnostic Records

### FC-AUTHZ-001 — SharePoint Create item Returns 403 Forbidden

**Category:** Authorisation  
**Connector:** SharePoint  
**Operation:** Create item

#### Symptoms

A flow reaches the SharePoint Create item action, but the action fails with HTTP 403 Forbidden or a permission-denied message.

#### Possible Causes

- The connection identity lacks permission to add items.
- The target list has permissions that differ from the parent site.
- The connection uses an unexpected account.
- A relevant organisational restriction prevents access.

#### Required Evidence

- Sanitised action error.
- Connection identity.
- Target site and list configuration.
- Effective permissions for the connection identity.

#### Diagnostic Checks

1. Inspect the failed action's Inputs and Outputs.
2. Identify the SharePoint connection used by the action.
3. Confirm the configured Site Address and List Name.
4. Ask the authorised site or list administrator to inspect effective access for that identity.
5. Verify whether the identity has the Add Items permission or an appropriate permission level containing it.
6. Investigate applicable organisational restrictions if permissions appear sufficient.

#### Potential Resolution

An authorised administrator may grant the identity the minimum permissions required to perform the operation.

Do not recommend Full Control merely to resolve Create item failures.

If the identity is not approved for write operations, use an appropriately authorised connection identity instead.

#### Verification

Perform a controlled test with non-sensitive data.

Confirm that Create item succeeds and the expected item appears in the correct list.

Check for duplicate items before resubmitting an existing run.

#### Prevention

Document connection ownership and required list permissions.

Review permission changes when the flow or SharePoint list is updated.

#### References

- [SharePoint permission levels](https://learn.microsoft.com/en-us/sharepoint/understanding-permission-levels)
- [Troubleshoot cloud flow errors](https://learn.microsoft.com/en-us/power-automate/troubleshoot-flow-errors)

---

### FC-AUTHZ-002 — SharePoint Get items Access Denied

**Category:** Authorisation  
**Connector:** SharePoint  
**Operation:** Get items

#### Symptoms

A flow cannot retrieve list items and reports 403 Forbidden or an access-denied response.

#### Possible Causes

- The connection identity cannot read the target list.
- The list has unique permissions.
- Access is restricted for the account, resource or organisational context.
- A different connection identity is being used than expected.

#### Required Evidence

- Failing action and sanitised response.
- Connection identity.
- Target list configuration.
- Effective permissions for that identity.

#### Diagnostic Checks

1. Inspect the Get items action's error response.
2. Confirm the identity associated with the connection.
3. Verify that the action targets the intended site and list.
4. Check whether list permissions are inherited or uniquely configured.
5. Have an authorised administrator assess the identity's effective list access.
6. If access is correct, investigate other restrictions and the complete error details.

Do not assume that access to the parent SharePoint site establishes access to every list.

#### Potential Resolution

Correct the identified permission or connection-identity problem through approved administrative procedures.

Avoid granting broader access than the operation requires.

#### Verification

Run a controlled retrieval and confirm the action returns the expected permitted records.

A successful action status alone does not establish that the intended data was returned.

#### Prevention

Document list-level permission requirements and review unique-permission changes.

#### References

- [SharePoint permission levels](https://learn.microsoft.com/en-us/sharepoint/understanding-permission-levels)
- [Sharing and permissions in SharePoint](https://learn.microsoft.com/en-us/sharepoint/modern-experience-sharing-permissions)

---

### FC-AUTHZ-003 — Outlook Shared Mailbox Send Permission Denied

**Category:** Authorisation  
**Connector:** Office 365 Outlook  
**Operations:** Send an email (V2), Send an email from a shared mailbox (V2)

#### Symptoms

A flow attempts to send an email from another user's mailbox or a shared mailbox but is refused.

The exact connector error or HTTP status can vary.

#### Possible Causes

- The connection identity lacks Send As permission.
- The connection identity lacks required Send on Behalf permission.
- The wrong mailbox is configured.
- The action uses an unexpected connection identity.
- Mailbox or organisational policies restrict the operation.

#### Required Evidence

- Exact Outlook action.
- Sanitised error message.
- Connection identity.
- Intended sending mailbox.
- Relevant delegated mailbox permissions.

#### Diagnostic Checks

1. Identify whether the action sends from the connected user's mailbox or another mailbox.
2. Confirm the account used by the Outlook connection.
3. Review the action's mailbox or From configuration.
4. Ask an authorised Exchange administrator to verify the required sending permission.
5. Distinguish mailbox Full Access from Send As and Send on Behalf permissions.
6. Inspect relevant policy restrictions if the expected permissions are present.

Having Full Access to a shared mailbox does not automatically grant permission to send from it.

#### Potential Resolution

An authorised administrator may grant the appropriate sending permission or correct the mailbox and connection configuration.

Do not suggest broadly granting mailbox access without establishing the requirement.

#### Verification

Send a controlled test message to an approved recipient.

Confirm that the message is delivered and the intended sender identity is represented correctly.

Avoid repeatedly sending test messages without checking earlier outcomes.

#### Prevention

Document mailbox delegation, connection ownership and approved sending identities.

#### References

- [Office 365 Outlook connector](https://learn.microsoft.com/en-us/connectors/office365/)
- [Shared mailboxes in Exchange Online](https://learn.microsoft.com/en-us/exchange/collaboration-exo/shared-mailboxes)

---

### FC-AUTHZ-004 — Flow or Connection Authorisation Failure

**Category:** Authorisation  
**Area:** Flow sharing, connection permissions and ownership

#### Symptoms

Examples include:

- A flow works for its owner but not an authorised user.
- A user cannot activate a flow using a connection they do not own or have permission to use.
- A solution-aware flow reports ConnectionAuthorizationFailed.
- A shared flow uses a different connection identity than expected.

#### Possible Causes

- The user lacks the necessary flow-sharing permissions.
- A connection cannot be used by the identity activating the flow.
- Run-only connection configuration differs from the expected design.
- Flow ownership changed without reviewing connection dependencies.
- A connection reference points to a connection whose usage rights are insufficient.

#### Required Evidence

- Flow type and relevant permissions.
- Exact error message.
- Identity attempting the operation.
- Connection and connection-reference configuration.
- Whether the user is an owner, co-owner or run-only user.

#### Diagnostic Checks

1. Confirm whether the failure occurs when activating, editing or running the flow.
2. Identify the relevant flow and connection identities.
3. Review the configured owners and run-only permissions.
4. For solution-aware flows, inspect the relevant connection references.
5. Confirm whether the identity performing the operation is authorised to use the required connection.
6. Ask the authorised owner or administrator to review required access.

Do not assume that sharing a flow automatically grants access to all external resources.

#### Potential Resolution

Correct the specific sharing, ownership or connection-usage problem using Microsoft's supported sharing mechanisms.

Use approved accounts and minimum necessary privileges.

#### Verification

Repeat the affected operation using the intended authorised identity.

Confirm both successful execution and the expected business outcome.

#### Prevention

Document flow ownership, connection ownership and relevant connection references.

Review dependencies before changing ownership or deploying a solution.

#### References

- [Share a cloud flow](https://learn.microsoft.com/en-us/power-automate/create-team-flows)
- [Use connection references](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/create-connection-reference)

---

### FC-AUTHZ-005 — Power Platform Data Policy Restriction

**Category:** Authorisation / Governance  
**Area:** Power Platform data policies

#### Symptoms

A cloud flow cannot be saved, enabled or executed because a connector or connector combination is restricted by organisational policy.

An explicit policy error is stronger evidence than an isolated HTTP 403 response.

#### Possible Causes

- A connector is blocked by an applicable data policy.
- A flow combines connectors whose data groups prohibit their use together.
- Environment-level and tenant-level policies impose conflicting restrictions.
- A recently introduced or modified policy affects the flow.

#### Required Evidence

- Exact sanitised error.
- Environment.
- Connectors involved.
- Whether the problem occurs during saving, activation or execution.
- Relevant policy details supplied by an authorised administrator.

#### Diagnostic Checks

1. Inspect the full error message for explicit data-policy references.
2. Identify the connectors used by the affected flow.
3. Confirm the environment where the flow is operating.
4. Ask a Power Platform administrator to review applicable tenant and environment data policies.
5. Determine whether the policy blocks a connector, endpoint or combination of connectors.
6. Distinguish a policy failure from ordinary resource permissions.

#### Potential Resolution

The authorised administrator should determine whether the solution complies with organisational policy.

Possible approved outcomes include redesigning the flow, using permitted connectors or implementing an authorised policy change.

Never recommend bypassing or disabling organisational security controls merely to make a flow run.

#### Verification

Confirm that the flow satisfies the applicable policy and can complete its intended operation.

#### Prevention

Review data-policy compatibility during solution design and before deployment across environments.

#### References

- [Manage data policies](https://learn.microsoft.com/en-us/power-platform/admin/prevent-data-loss)
- [Impact of data policies on apps and flows](https://learn.microsoft.com/en-us/power-platform/admin/dlp-impact-policies-apps-flows)

---

### FC-AUTHZ-006 — Access Failure After Solution Import

**Category:** Authorisation  
**Area:** Solution-aware cloud flows and connection references

#### Symptoms

A flow works in its development environment but encounters access or connection-authorisation failures after being imported into another Power Platform environment.

#### Possible Causes

- A connection reference was mapped to an unexpected destination connection.
- The destination connection uses a different identity.
- The destination identity lacks required permissions on the target resource.
- The user activating the flow cannot use the mapped connection.
- Environment-specific security requirements differ.

#### Required Evidence

- Source and destination environment context.
- Exact error message.
- Affected connection reference.
- Destination connection identity.
- Relevant target-resource permissions.

#### Diagnostic Checks

1. Identify the failing action or activation operation.
2. Inspect the connection references in the destination solution.
3. Confirm which destination connection each reference uses.
4. Verify the connection identity and its authorisation to use the intended resource.
5. Check whether the identity activating the flow has the required connection-use permissions.
6. Review applicable environment-specific policies.

Do not assume that importing a solution transfers the source environment's user permissions or connection credentials.

#### Potential Resolution

Correct the destination connection mapping and any identified permissions through authorised procedures.

Avoid changing unrelated solution components or granting unnecessary environment-wide privileges.

#### Verification

Activate and execute the imported flow using an approved identity.

Confirm that it accesses the intended destination resource and produces the expected outcome.

#### Prevention

Document connection-reference dependencies and destination-environment access requirements.

Include connection and permissions validation in deployment checklists.

#### References

- [Use connection references](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/create-connection-reference)
- [Overview of solution-aware cloud flows](https://learn.microsoft.com/en-us/power-automate/overview-solution-flows)

---

## 5. General Diagnostic Decision Guide

When an access-related error occurs:

### Step A — Establish the failure

Determine the failing action, status code, connector and exact error message.

### Step B — Identify the security identity

Determine which account or approved application identity the operation uses.

### Step C — Check the target resource

Confirm the exact site, list, mailbox, environment or other resource.

### Step D — Evaluate effective permissions

Determine whether the identity is authorised to perform the specific operation.

### Step E — Investigate other restrictions

If permissions appear correct, review relevant connection usage, consent and organisational policy restrictions.

### Step F — Recommend an approved correction

Prefer least-privilege, non-destructive remediation.

### Step G — Verify the outcome

Use a controlled test and confirm the intended business result.

If the evidence is incomplete or contradictory, continue investigation instead of claiming a definitive root cause.

## 6. Confidence Guidance

**High confidence:** Specific error details and verified permission evidence support a clear diagnosis.

**Medium confidence:** Evidence suggests an access-control problem but important permissions or connection details remain unconfirmed.

**Low confidence:** The error is ambiguous, contradictory or could originate from multiple authentication, authorisation or policy layers.

Confidence in the error category may differ from confidence in the root cause.

## 7. Responsible Troubleshooting

FlowCheck must:

- Never request passwords, tokens or client secrets.
- Encourage sanitised error messages and logs.
- Never claim direct environment access.
- Never claim to have changed permissions.
- Never recommend disabling data policies or security controls to bypass failures.
- Recommend authorised administrators for permission or policy changes.
- Distinguish recommendations from confirmed actions.
- Avoid recommending Full Control when narrower permissions are sufficient.
- Account for duplicate records, emails and other side effects before rerunning flows.

## 8. Technical References

The following official Microsoft documentation informed this reference:

1. [Troubleshoot cloud flow errors](https://learn.microsoft.com/en-us/power-automate/troubleshoot-flow-errors)
2. [Cloud flow error code reference](https://learn.microsoft.com/en-us/power-automate/error-reference)
3. [SharePoint permission levels](https://learn.microsoft.com/en-us/sharepoint/understanding-permission-levels)
4. [SharePoint sharing and permissions](https://learn.microsoft.com/en-us/sharepoint/modern-experience-sharing-permissions)
5. [Office 365 Outlook connector](https://learn.microsoft.com/en-us/connectors/office365/)
6. [Shared mailboxes in Exchange Online](https://learn.microsoft.com/en-us/exchange/collaboration-exo/shared-mailboxes)
7. [Share a cloud flow](https://learn.microsoft.com/en-us/power-automate/create-team-flows)
8. [Use connection references](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/create-connection-reference)
9. [Manage Power Platform data policies](https://learn.microsoft.com/en-us/power-platform/admin/prevent-data-loss)
10. [Impact of data policies on apps and flows](https://learn.microsoft.com/en-us/power-platform/admin/dlp-impact-policies-apps-flows)
11. [Overview of solution-aware cloud flows](https://learn.microsoft.com/en-us/power-automate/overview-solution-flows)


> **Maintenance note:** Update this reference when Microsoft changes relevant connector behaviour, permission models, solution deployment procedures or organisational policy guidance. Changes should be reviewed for technical accuracy and evaluated against relevant FlowCheck test scenarios.

This document is an independent community-maintained troubleshooting reference and is not an official Microsoft support publication.

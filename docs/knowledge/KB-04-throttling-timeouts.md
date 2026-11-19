---
Title: "KB-04 — Throttling, Timeouts and Service Errors"
Description: "Evidence-based troubleshooting for Power Automate rate limits, timeouts, transient service failures and platform capacity restrictions."
---

# KB-04 — Throttling, Timeouts and Service Errors

> **FlowCheck knowledge reference** · Version **1.0.0** · Reviewed **19 November 2026**  
> **Coverage:** Microsoft Power Automate cloud flows; Reliability and Service Errors.  
> **Status:** Community troubleshooting guidance, not an official Microsoft support service.

[← Knowledge-base home](../index.html)

## 1. Purpose

This reference supports the diagnosis of throttling, timeouts, transient service errors and platform resource limitations affecting Microsoft Power Automate cloud flows.

It provides evidence-based troubleshooting for:

- HTTP 429 Too Many Requests.
- SharePoint Online throttling.
- HTTP 500 Internal Server Error.
- HTTP 502 Bad Gateway.
- HTTP 503 Service Unavailable.
- HTTP 504 Gateway Timeout.
- Action and operation timeouts.
- Synchronous response timeouts.
- Power Platform request limits and execution quotas.

The objective is to identify the relevant failure mechanism, recommend safe corrective actions and distinguish transient conditions from persistent configuration or capacity problems.

FlowCheck provides advisory guidance only. It does not directly execute, modify, restart or repair cloud flows.

## 2. Common Symptoms

Users may report:

- A flow occasionally fails with HTTP 429.
- A SharePoint action reports "Rate limit is exceeded."
- Multiple actions begin failing during high-volume processing.
- A connector returns HTTP 500 or 503.
- An HTTP action reports Bad Gateway.
- A flow experiences Gateway Timeout.
- An action reports ActionTimedOut.
- An operation reports OperationTimedOut.
- A Power Apps-triggered flow takes too long to return a response.
- A flow becomes slower as the number of processed records increases.
- A cloud flow exceeds its request or action quota.
- A flow is suspended following persistent limit violations.

Similar symptoms can have different causes. The exact error, affected component and execution history are required for diagnosis.

## 3. General Diagnostic Principles

Before recommending a correction:

1. Identify the exact failed action or operation.
2. Capture the sanitised error code and message.
3. Determine the connector or target service.
4. Identify whether the failure is intermittent or persistent.
5. Check the relevant action duration and retry history.
6. Inspect response headers when available.
7. Determine whether calls are occurring concurrently or in a high-volume loop.
8. Check relevant service and platform limits.
9. Determine whether the operation is safe to retry.
10. Recommend a controlled verification procedure.

Do not assume that every HTTP 429 has the same underlying limit.

Do not assume that every HTTP 500, 502, 503 or 504 proves a Microsoft service outage.

Do not recommend increasing retry attempts without evaluating the impact on the destination service.

A failed request or timeout does not always prove that the target operation was not completed.

## 4. Diagnostic Records

### FC-REL-001 — HTTP 429 Connector Throttling

**Category:** Throttling  
**Common status:** HTTP 429 Too Many Requests

#### Symptoms

A connector action fails with HTTP 429 or a message indicating that the request rate has been exceeded.

Example:

"Rate limit is exceeded. Try again later."

#### Possible Causes

- Too many connector requests within a permitted time window.
- High-frequency calls inside Apply to each.
- Multiple flow runs using the same connection.
- Parallel actions increasing request volume.
- A destination service enforcing its own usage limits.
- A connector-specific quota being exceeded.

#### Required Evidence

- Connector and operation.
- Sanitised error message.
- Response status.
- Retry-After header, if available.
- Approximate request frequency.
- Loop or concurrency configuration.
- Relevant connector documentation.

#### Diagnostic Checks

1. Open the failed run and identify the throttled action.
2. Inspect the action's status code and complete sanitised error.
3. Check whether the response supplies Retry-After guidance.
4. Identify loops, parallel branches or multiple runs generating requests.
5. Review the affected connector's published throttling limits.
6. Compare observed request volume with the relevant limit.
7. Inspect existing retry settings.
8. Determine whether a temporary spike or persistent workload pattern caused the problem.

#### Potential Resolution

Depending on the confirmed cause:

- Reduce request frequency.
- Spread calls over time.
- Use suitable Delay actions.
- Reduce unnecessary concurrency.
- Batch operations where supported.
- Optimise repeated connector calls.
- Configure supported retries with exponential backoff.
- Respect service-provided retry instructions.

Adding retries without reducing the underlying request pressure can worsen throttling.

Do not assume changing connections or licensing will resolve every connector-level limit.

#### Verification

Run a controlled workload at an approved request rate.

Confirm that throttling decreases and the intended operations complete correctly.

Monitor request counts and retries.

#### Prevention

Document connector limits and design high-volume flows to minimise avoidable requests.

#### References

- [Understand platform limits and avoid throttling](https://learn.microsoft.com/en-us/power-automate/guidance/coding-guidelines/understand-limits)
- [Troubleshoot cloud flow errors](https://learn.microsoft.com/en-us/power-automate/troubleshoot-flow-errors)
- [Power Automate limits and configuration](https://learn.microsoft.com/en-us/power-automate/limits-and-config)

---

### FC-REL-002 — SharePoint Online Throttling and Retry-After

**Category:** Throttling  
**Connector:** SharePoint  
**Common statuses:** HTTP 429, HTTP 503

#### Symptoms

A Power Automate flow accesses SharePoint Online repeatedly and receives throttling or server-busy responses.

Examples:

- HTTP 429 Too Many Requests.
- HTTP 503 Server Too Busy.
- A SharePoint request fails during bulk processing.
- A flow becomes unreliable when performing many SharePoint operations.

#### Possible Causes

- Excessive SharePoint requests.
- High-volume list or document operations.
- Multiple workflows accessing the same service.
- Concurrent or repeated API calls.
- Temporary service load.
- A client or request pattern being restricted.

#### Required Evidence

- SharePoint operation.
- Sanitised error response.
- HTTP status.
- Retry-After header, if present.
- Request frequency.
- Relevant loops and concurrent runs.
- Whether the failure affects other SharePoint operations.

#### Diagnostic Checks

1. Identify the failing SharePoint action.
2. Confirm whether the status is 429 or 503.
3. Inspect the response for Retry-After.
4. Review recent request volume and execution patterns.
5. Examine Apply to each loops and parallel processing.
6. Identify repeated reads or writes that could be consolidated.
7. Determine whether the service is experiencing a known incident.
8. Review Microsoft's SharePoint throttling guidance.

For SharePoint Online API throttling, Microsoft documents Retry-After as an instruction indicating how long the caller should wait.

#### Potential Resolution

- Honour Retry-After where available and supported.
- Reduce unnecessary requests.
- Optimise list retrieval with suitable filtering.
- Avoid repeated retrieval of unchanged data.
- Reduce concurrency when it contributes to throttling.
- Use controlled retries where supported.
- Review the workload with the appropriate administrator if throttling persists.

Do not repeatedly retry requests immediately after receiving a throttling response.

Do not recommend circumventing Microsoft service protection controls.

#### Verification

Test the workflow under a controlled load.

Confirm that the relevant SharePoint operation completes and the expected records or files are present.

Check for duplicate writes before resubmitting operations.

#### Prevention

Design SharePoint integrations to minimise redundant operations and account for service limits.

#### References

- [Avoid SharePoint Online throttling](https://learn.microsoft.com/en-us/sharepoint/dev/general-development/how-to-avoid-getting-throttled-or-blocked-in-sharepoint-online)
- [SharePoint connector reference](https://learn.microsoft.com/en-us/connectors/sharepoint/)
- [Understand platform limits](https://learn.microsoft.com/en-us/power-automate/guidance/coding-guidelines/understand-limits)

---

### FC-REL-003 — HTTP 500, 502 or 503 Service Failure

**Category:** Service Errors  
**Common statuses:** HTTP 500, 502, 503

#### Symptoms

A flow action fails with one of the following:

- HTTP 500 Internal Server Error.
- HTTP 502 Bad Gateway.
- HTTP 503 Service Unavailable.

Failures may be intermittent or may affect multiple runs.

#### Meaning of Status Codes

**500 Internal Server Error**

The server encountered an unexpected condition.

The precise cause requires further evidence.

**502 Bad Gateway**

A gateway or intermediary received an invalid response from an upstream service.

**503 Service Unavailable**

The service is temporarily unable to handle the request, often because of overload or maintenance.

A 503 may also occur during service-specific throttling.

These meanings do not establish which underlying component failed.

#### Possible Causes

- Temporary service disruption.
- Upstream dependency failure.
- Overloaded destination service.
- Gateway or network connectivity issue.
- Connector or service-side processing failure.
- Persistent invalid service configuration.
- Service-specific throttling.

#### Required Evidence

- Exact status code.
- Connector and operation.
- Sanitised response body.
- Timestamp and environment.
- Whether other actions or flows are affected.
- Service health information.
- Relevant correlation or request identifiers.

#### Diagnostic Checks

1. Inspect the failed action's Outputs.
2. Identify the target service.
3. Determine whether the failure is intermittent.
4. Check whether the same action previously succeeded.
5. Review available service health information.
6. Inspect whether multiple workflows are affected.
7. Review retry history and any correlation identifiers.
8. Determine whether the operation is safe to repeat.

#### Potential Resolution

For a confirmed or likely transient issue:

- Allow time for service recovery.
- Use an appropriate supported retry policy.
- Prefer exponential backoff when suitable.
- Respect Retry-After where provided.
- Monitor for recurrence.

For persistent failures:

- Inspect detailed connector or API errors.
- Review relevant network and gateway configuration.
- Check service-specific requirements.
- Escalate with sanitised diagnostic evidence.

Never state that waiting will definitely resolve a persistent server error.

#### Verification

Confirm the affected operation completes successfully and produces the intended result.

For non-idempotent operations, check whether the original request already took effect before retrying.

#### Prevention

Introduce controlled error handling, suitable retry policies and service monitoring.

#### References

- [Cloud flow error reference](https://learn.microsoft.com/en-us/power-automate/error-reference)
- [Fix connection failures in cloud flows](https://learn.microsoft.com/en-us/power-automate/fix-connection-failures)
- [HTTP 5xx error handling](https://learn.microsoft.com/en-us/microsoft-cloud/dev/dev-proxy/concepts/http-5xx-api-errors)
- [Employ robust error handling](https://learn.microsoft.com/en-us/power-automate/guidance/coding-guidelines/error-handling)

---

### FC-REL-004 — HTTP 504 Gateway Timeout or Action Timeout

**Category:** Timeout  
**Common errors:** HTTP 504, ActionTimedOut, OperationTimedOut

#### Symptoms

An action or operation exceeds an applicable time limit.

Examples include:

- HTTP 504 Gateway Timeout.
- An HTTP call taking longer than its permitted response window.
- ActionTimedOut.
- OperationTimedOut.
- A long-running external process not completing within the expected period.

#### Possible Causes

- Slow upstream service response.
- Large data-processing workload.
- Expensive or unfiltered queries.
- Network or gateway latency.
- A configured action timeout being reached.
- An external operation waiting for a callback.
- A workflow exceeding an applicable execution limit.

#### Required Evidence

- Exact timeout error.
- Action type.
- Configured timeout, if applicable.
- Actual action duration.
- Target service.
- Relevant response or callback behaviour.
- Whether the requested operation had side effects.

#### Diagnostic Checks

1. Identify whether the problem is an HTTP gateway timeout, action timeout or operation timeout.
2. Inspect the failed action's duration.
3. Review its applicable timeout configuration.
4. Examine the size and complexity of the workload.
5. Determine whether the target service supports asynchronous processing.
6. Inspect upstream latency and connector-specific limits.
7. Check whether the original operation may have completed despite the timeout.

#### Potential Resolution

Depending on the action:

- Reduce processing workload.
- Use server-side filtering or pagination.
- Use asynchronous processing where supported.
- Configure a suitable timeout where the action permits it.
- Handle the timeout through an approved error-handling branch.
- Use controlled polling for documented long-running operations.

Increasing an action's timeout cannot override every platform-enforced response limit.

Do not assume a timed-out request failed without side effects.

#### Verification

Check the external system for the original operation's result.

Perform a controlled test and confirm both completion and the expected business outcome.

#### Prevention

Design long-running operations with appropriate timeouts, asynchronous patterns and duplicate-operation safeguards.

#### References

- [Cloud flow error reference](https://learn.microsoft.com/en-us/power-automate/error-reference)
- [Power Automate limits and configuration](https://learn.microsoft.com/en-us/power-automate/limits-and-config)
- [HTTP 5xx error handling](https://learn.microsoft.com/en-us/microsoft-cloud/dev/dev-proxy/concepts/http-5xx-api-errors)

---

### FC-REL-005 — Synchronous Response Timeout

**Category:** Timeout  
**Area:** Power Automate response operations

#### Symptoms

A caller does not receive the expected response from a cloud flow within the synchronous request window.

Examples:

- A Power Apps-triggered flow takes too long to respond.
- An HTTP Request-triggered flow does not respond in time.
- A response action executes after the applicable synchronous deadline.
- The caller reports a timeout although processing may continue.

#### Possible Causes

- The flow performs too much work before responding.
- A connector action is slow.
- An approval or other long-running operation occurs before the response.
- The flow design requires a longer-running process than the synchronous response window supports.
- An external service responds too slowly.

#### Important Platform Constraint

Microsoft documents a 120-second limit for inbound and outbound synchronous requests in Power Automate.

This is not a universal timeout for every cloud-flow action or entire flow run.

Some background work may continue after the synchronous response deadline, depending on the workflow design.

#### Required Evidence

- Trigger type.
- Response action type.
- Caller-reported error.
- Run duration.
- Actions executed before the response.
- Whether the flow continued processing.

#### Diagnostic Checks

1. Identify the trigger and response action.
2. Confirm whether a synchronous response is expected.
3. Inspect the duration of actions before the response.
4. Locate blocking or long-running operations.
5. Determine whether an asynchronous design is appropriate.
6. Confirm whether the original workflow continued or completed.

#### Potential Resolution

Where appropriate:

- Reduce work performed before the response.
- Optimise slow actions.
- Return a suitable response earlier when the business contract permits.
- Use supported asynchronous processing patterns.
- Split longer-running operations into an approved follow-up workflow.

Do not return a success response before an operation has completed if the business process requires confirmation of completion.

#### Verification

Confirm the caller receives the correct response within the applicable limit.

Separately verify any work continuing after the response.

#### Prevention

Design synchronous integrations to return within the supported response window and use asynchronous processing for longer operations.

#### References

- [Power Automate limits and configuration](https://learn.microsoft.com/en-us/power-automate/limits-and-config)
- [Cloud flow error reference](https://learn.microsoft.com/en-us/power-automate/error-reference)

---

### FC-REL-006 — Power Platform Request or Execution Limit Exceeded

**Category:** Platform Limits  
**Common errors:** FlowRunQuotaExceeded, WorkflowRunActionRepetitionQuotaExceeded

#### Symptoms

Examples include:

- A cloud flow exceeds an applicable action or request quota.
- A high-volume workflow experiences capacity-related delays.
- An error indicates that a repetition or execution limit was reached.
- A flow is turned off after sustained limit violations.
- A workflow performs excessive actions within loops.

#### Possible Causes

- Too many actions within a reporting period.
- A high volume of flow runs.
- Repeated or unnecessary connector operations.
- Large nested loops.
- Excessive pagination or retries.
- The flow exceeding its applicable licensed request allocation.
- A workflow exceeding a repetition or execution-specific limit.

#### Required Evidence

- Exact error and limit type.
- Flow ownership and relevant licensing context.
- Request or action usage.
- Approximate run volume.
- Loop and concurrency settings.
- Retry configuration.
- Relevant environment and flow analytics.

#### Diagnostic Checks

1. Identify the exact quota or limit named in the error.
2. Distinguish platform request limits from connector throttling.
3. Review available Power Automate analytics.
4. Determine the relevant license or flow capacity.
5. Inspect loops and repeated calls.
6. Examine pagination and retry behaviour.
7. Identify avoidable action executions.
8. Compare the workload against Microsoft's current documented limits.

Do not assume that upgrading a Power Automate license increases a downstream connector's independent service limit.

Do not assume that splitting a workload into more flows increases the total request allocation for the same licensed user.

#### Potential Resolution

Depending on the confirmed limit:

- Reduce unnecessary actions.
- Optimise loops and data retrieval.
- Use appropriate filtering and batching.
- Reduce avoidable retries.
- Spread workloads when this helps the applicable limit.
- Review legitimate licensing or capacity requirements with an administrator.
- Redesign processing where platform limits cannot support the current workload.

Do not recommend purchasing additional capacity unless the specific limit and licensing requirement have been established.

#### Verification

Review request usage and repeat representative controlled tests.

Confirm the workflow stays within applicable limits while producing the expected result.

#### Prevention

Monitor workload growth, action usage and applicable platform limits.

#### References

- [Power Automate limits and configuration](https://learn.microsoft.com/en-us/power-automate/limits-and-config)
- [Understand platform limits and avoid throttling](https://learn.microsoft.com/en-us/power-automate/guidance/coding-guidelines/understand-limits)
- [Power Platform request limits](https://learn.microsoft.com/en-us/power-platform/admin/api-request-limits-allocations)
- [Cloud flow error reference](https://learn.microsoft.com/en-us/power-automate/error-reference)

---

## 5. General Troubleshooting Decision Guide

### Step A — Identify the Failure

Record:

- Error code.
- Failed action.
- Connector or service.
- Run duration.
- Whether the issue is intermittent.
- Whether the operation changes data.

### Step B — Classify the Error

**HTTP 429**

Investigate connector or service throttling.

**HTTP 500**

Investigate a server-side failure using the detailed response.

**HTTP 502**

Investigate gateway or upstream service communication.

**HTTP 503**

Investigate service availability, load and possible service-specific throttling.

**HTTP 504**

Investigate upstream response timing and possible completion uncertainty.

**ActionTimedOut or OperationTimedOut**

Investigate the applicable operation timeout and execution history.

**FlowRunQuotaExceeded**

Investigate the applicable platform quota.

### Step C — Examine Request Behaviour

Determine whether the flow creates a burst of requests, performs excessive iterations or generates unnecessary retries.

### Step D — Evaluate Retry Safety

Before recommending a retry, determine whether the operation may have already:

- Created a SharePoint item.
- Sent an email.
- Updated a record.
- Submitted an approval.
- Triggered an external process.
- Created a financial or business transaction.

Where completion is uncertain, inspect the destination system before resubmitting.

### Step E — Choose an Evidence-Based Correction

Select a correction consistent with the identified failure.

Examples include:

- Request-rate reduction.
- Exponential backoff.
- Workload optimisation.
- Appropriate asynchronous processing.
- Service-health investigation.
- Controlled escalation.

### Step F — Verify the Outcome

Confirm both technical success and the expected business result.

Do not classify a problem as resolved solely because a later action returned HTTP 200.

## 6. Retry and Backoff Guidance

A retry is appropriate only where:

1. The failure is plausibly transient.
2. The action supports a suitable retry mechanism.
3. The operation is safe to repeat or protected against duplication.
4. The retry rate complies with the service's limits.

### Retry-After

When supported and present, respect the service's Retry-After instructions.

### Exponential Backoff

Exponential backoff increases the delay between retry attempts.

This can help transient failures recover without generating an immediate burst of repeated requests.

### Retry Limitations

Repeated retries may:

- Consume additional requests.
- Increase throttling.
- Extend execution time.
- Produce duplicate side effects.
- Hide an underlying persistent failure.

Never recommend unlimited retries.

A non-idempotent operation must not be retried blindly when its original result is uncertain.

## 7. Confidence Guidance

**High confidence:** The exact error, request pattern and relevant limit or service evidence clearly support the diagnosis.

**Medium confidence:** The error suggests a likely transient, throttling or timeout issue, but important usage or execution evidence remains missing.

**Low confidence:** The available error is ambiguous or multiple causes remain plausible.

FlowCheck must distinguish confidence in the error category from confidence in the underlying root cause.

## 8. Escalation Guidance

Recommend escalation when:

- A persistent service error affects multiple approved workflows.
- A known service incident appears to be involved.
- The relevant limit cannot be determined.
- The flow repeatedly fails despite evidence-based corrections.
- An administrator must review licensing, capacity or service configuration.
- A timed-out operation has uncertain or potentially consequential side effects.

Suggested escalation evidence:

- Sanitised error message.
- Flow and environment identifiers, where safe to share.
- Failing action name.
- Timestamp and relevant time zone.
- Correlation or request identifier.
- Relevant request volume.
- Steps already attempted.
- Business impact.

Where applicable, administrators may check service health using the Power Platform or Microsoft 365 admin centre.

Never request credentials, tokens, secrets or confidential payloads.

## 9. Responsible Troubleshooting

FlowCheck must:

- Provide advisory guidance only.
- Never claim direct access to run history.
- Never claim to have restarted or repaired a flow.
- Never recommend bypassing connector or service protection limits.
- Never recommend unlimited retries.
- Consider duplicate operations and business side effects.
- Distinguish temporary errors from proven incidents.
- Avoid inventing service outage information.
- Avoid claiming that upgrading a license resolves all throttling.
- Recommend appropriate administrator involvement.
- Clearly identify unresolved uncertainty.

## 10. Technical References

Official Microsoft documentation:

1. [Cloud flow error code reference](https://learn.microsoft.com/en-us/power-automate/error-reference)
2. [Troubleshoot cloud flow errors](https://learn.microsoft.com/en-us/power-automate/troubleshoot-flow-errors)
3. [Understand platform limits and avoid throttling](https://learn.microsoft.com/en-us/power-automate/guidance/coding-guidelines/understand-limits)
4. [Power Automate limits and configuration](https://learn.microsoft.com/en-us/power-automate/limits-and-config)
5. [Employ robust error handling](https://learn.microsoft.com/en-us/power-automate/guidance/coding-guidelines/error-handling)
6. [Avoid SharePoint Online throttling](https://learn.microsoft.com/en-us/sharepoint/dev/general-development/how-to-avoid-getting-throttled-or-blocked-in-sharepoint-online)
7. [SharePoint connector reference](https://learn.microsoft.com/en-us/connectors/sharepoint/)
8. [HTTP 5xx error handling](https://learn.microsoft.com/en-us/microsoft-cloud/dev/dev-proxy/concepts/http-5xx-api-errors)
9. [Power Platform request limits](https://learn.microsoft.com/en-us/power-platform/admin/api-request-limits-allocations)
10. [Check Power Platform service health](https://learn.microsoft.com/en-us/power-platform/admin/check-online-service-health)

> **Maintenance note:** Update this reference when Microsoft changes connection-management navigation, connector authentication behaviour or identity-policy guidance. The above links are source references, not claims that FlowCheck has live access to 's current seMicrosoftrvice health or a user's environment.

This is an independent, community-maintained troubleshooting reference and is not an official Microsoft support publication.

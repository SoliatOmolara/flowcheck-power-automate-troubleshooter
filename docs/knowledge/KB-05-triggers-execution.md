---
Title: "KB-05 — Triggers and Workflow Execution"
Description: "Evidence-based troubleshooting for Power Automate trigger failures, skipped actions, scheduling, concurrency and execution behaviour."
---

# KB-05 — Triggers and Workflow Execution

> **FlowCheck knowledge reference** · Version **1.0.0** · Reviewed **19 November 2026**  
> **Coverage:** Microsoft Power Automate cloud flows; Triggers and Workflow Execution.  
> **Status:** Community troubleshooting guidance, not an official Microsoft support service.

[← Knowledge-base home](../index.html)

## 1. Purpose

This reference provides diagnostic guidance for Microsoft Power Automate cloud flows that:

- Do not start when expected.
- Have trigger conditions that suppress execution.
- Start later than expected.
- Process unexpected or previously pending events.
- Execute at an unexpected scheduled time.
- Skip actions or follow unexpected branches.
- Execute actions more than once.
- Experience concurrency or execution-order problems.

The objective is to distinguish problems that prevent a flow from starting from problems that occur after a flow run begins.

FlowCheck provides advisory guidance only. It does not access, execute, activate, deactivate or modify a user's cloud flows.

## 2. Common Symptoms

Examples include:

- "My flow isn't triggering."
- "A SharePoint item was created, but the flow didn't run."
- "An email arrived, but the flow never started."
- "The trigger condition wasn't met."
- "My flow runs several minutes late."
- "My scheduled flow runs at the wrong time."
- "Some actions show Skipped."
- "My Condition action is going down the wrong branch."
- "The flow run is successful, but an expected action didn't execute."
- "My flow creates duplicate SharePoint records."
- "The same email was sent twice."
- "Two flow runs updated the same record."
- "The flow stopped running after previously working."

These symptoms require different investigations.

A missing run suggests a trigger-level issue.

An existing run with skipped or failed actions requires execution-level analysis.

## 3. Diagnostic Principles

Before recommending a correction:

1. Identify the flow's trigger type.
2. Determine whether a flow run was created.
3. Inspect trigger history or trigger checks where available.
4. Confirm the flow's status and environment.
5. Review the trigger configuration and relevant connections.
6. Identify filters or trigger conditions.
7. Examine the actual event that should have triggered the flow.
8. For existing runs, inspect individual action statuses.
9. Review dependencies, branches and concurrency where relevant.
10. Consider the effects of rerunning or resubmitting the flow.
11. Recommend a correction supported by the available evidence.
12. Explain how to verify the result.

Do not assume that every trigger failure is caused by a broken connection.

Do not assume that an action marked Skipped represents an error.

Do not assume that an event results in exactly one flow run or one external side effect.

## 4. Diagnostic Records

### FC-TRIG-001 — Flow Does Not Trigger

**Category:** Triggers  
**Common symptom:** No expected flow run appears

#### Symptoms

A user reports that an expected event occurred, but the cloud flow did not start.

Examples include:

- A SharePoint item was created.
- An Outlook email arrived.
- A Dataverse row changed.
- An external system sent an event.

However, the expected run cannot be found.

#### Possible Causes

- The flow is turned off or suspended.
- The wrong environment is being inspected.
- The triggering event does not match the selected trigger.
- The trigger points to the wrong site, list, folder, mailbox or table.
- The trigger uses an invalid or disconnected connection.
- A trigger filter or condition excludes the event.
- The event was not detected by the trigger mechanism.
- A policy, licensing or service issue prevents operation.
- The event occurred while a webhook trigger was inactive.

#### Required Evidence

- Flow name and environment.
- Flow status.
- Trigger type and configuration.
- Approximate expected event time.
- Description of the actual event.
- Relevant trigger history or checks.
- Available connection status and error information.

#### Diagnostic Checks

1. Confirm that the user is inspecting the correct environment and flow.
2. Check whether the flow is enabled or suspended.
3. Inspect the flow's run history.
4. If there is no matching run, inspect trigger history or trigger checks where available.
5. Verify that the event matches the trigger's intended operation.
6. Confirm the configured target resource.
7. Review trigger filters and conditions.
8. Check whether the trigger connection is healthy.
9. Investigate relevant organisational policies and licensing restrictions.
10. Check service health if evidence suggests a wider problem.

For example, a SharePoint trigger configured for one list will not automatically monitor every list on the site.

If the flow is suspended, investigate why before recommending reactivation.

#### Potential Resolution

Depending on the confirmed cause:

- Correct the configured trigger resource.
- Repair an expired or invalid connection through an approved process.
- Correct an overly restrictive trigger filter.
- Address an applicable policy or licensing issue.
- Resolve the problem responsible for suspension.
- Have an authorised owner reactivate the flow when safe.

Do not recommend deleting and recreating the flow as an initial troubleshooting step.

#### Verification

Generate a new, approved test event that matches the trigger configuration.

Confirm:

1. The trigger detects the event.
2. A run starts.
3. The expected actions execute.
4. The intended business result occurs.

#### Prevention

Document trigger configuration, connection ownership, operational dependencies and expected event behaviour.

#### References

- [Troubleshoot trigger problems](https://learn.microsoft.com/en-us/troubleshoot/power-platform/power-automate/flow-run-issues/triggers-troubleshoot)
- [Troubleshoot cloud flow errors](https://learn.microsoft.com/en-us/power-automate/troubleshoot-flow-errors)
- [Fix cloud flow connection failures](https://learn.microsoft.com/en-us/power-automate/fix-connection-failures)
- [Turn a flow on or off](https://learn.microsoft.com/en-us/power-automate/disable-flow)

---

### FC-TRIG-002 — Trigger Conditions Prevent Flow Execution

**Category:** Triggers  
**Common error:** TriggerConditionNotMet

#### Symptoms

An event occurs, but the flow does not create an expected run because its trigger condition evaluates to false.

Examples include:

- A flow is configured to run only when Status is Approved.
- A SharePoint record is updated, but the flow remains inactive.
- The trigger check shows that its condition was not satisfied.

#### Possible Causes

- A trigger condition correctly excludes the event.
- The expression references the wrong field.
- The expected property is missing.
- The actual value differs from the expected value.
- The expression uses incorrect logic.
- Multiple conditions are unintentionally combined.
- An expression assumes an incorrect data structure.

#### Required Evidence

- Complete trigger condition.
- Sanitised trigger payload.
- Expected triggering behaviour.
- Relevant trigger check result.
- Actual values of the referenced properties.

#### Diagnostic Checks

1. Open the flow in the Power Automate designer.
2. Select the trigger.
3. Inspect Settings and Trigger conditions.
4. Identify each configured condition.
5. Review relevant trigger checks.
6. Compare the condition with the actual event payload.
7. Verify the property names, types and values.
8. Determine whether the condition should evaluate to true.
9. Check whether multiple trigger conditions are required simultaneously.

Microsoft documents that when multiple trigger conditions are configured separately, all must be satisfied by default.

If any condition is optional, the appropriate logic may require an OR expression.

#### Example

Suppose an event payload contains:

```json
{
  "Priority": "High",
  "RequestId": "REQ-001"
}
```

A trigger condition could be:

```text
@equals(triggerBody()?['Priority'], 'High')
```

This expression is illustrative and applies only when the trigger payload actually contains a top-level string property named Priority.

For connectors that return nested objects or differently structured fields, the expression must be adjusted using real output evidence.

#### Important Distinction

A trigger condition determines whether a flow run starts.

A Condition action inside the flow determines which actions execute after a run has already started.

Confusing the two can lead to incorrect troubleshooting advice.

#### Potential Resolution

Correct the trigger condition so that it reflects the intended business requirement.

When necessary, an authorised maker can temporarily test without a restrictive condition in a safe development environment, then restore the intended condition.

Do not recommend removing a legitimate business filter permanently.

#### Verification

Test two scenarios:

1. A matching event should start the flow.
2. A non-matching event should be excluded.

Confirm both outcomes.

#### Prevention

Document trigger conditions and test positive and negative scenarios before deploying changes.

#### References

- [Customize triggers with conditions](https://learn.microsoft.com/en-us/power-automate/customize-triggers)
- [Troubleshoot trigger problems](https://learn.microsoft.com/en-us/troubleshoot/power-platform/power-automate/flow-run-issues/triggers-troubleshoot)
- [Cloud flow error code reference](https://learn.microsoft.com/en-us/power-automate/error-reference)

---

### FC-TRIG-003 — Delayed Trigger or Unexpected Historical Events

**Category:** Triggers  
**Areas:** Polling triggers and webhook triggers

#### Symptoms

Examples include:

- An event occurs, but the flow starts later than expected.
- A flow begins processing older events after being turned back on.
- Events occurring while a flow was disabled are not processed.
- Several expected trigger events appear close together.

#### Important Trigger Distinction

Power Automate supports polling and webhook-based trigger mechanisms.

**Polling trigger**

A polling trigger periodically checks the connected service for new events.

Depending on the connector and configuration, events can be detected after a delay.

Microsoft documents that certain polling triggers can process pending events after a flow is reactivated.

**Webhook trigger**

A webhook trigger receives event notifications from the source service.

Microsoft documents that events occurring while the flow is turned off might not be delivered after the flow is reactivated.

Therefore, the behaviour of the two mechanisms is not interchangeable.

#### Possible Causes

- Normal polling interval.
- Connector-specific trigger frequency.
- Throttling or temporary service delay.
- A backlog of pending events.
- A flow that was previously disabled.
- Missed webhook notifications.
- Connection or service communication problems.
- Unexpected duplicate events or multiple active flows.

#### Required Evidence

- Trigger name and connector.
- Trigger mechanism, if established.
- Event creation timestamp.
- Flow run start timestamp.
- Flow status during the event.
- Recent changes to the flow.
- Any recorded throttling or service errors.

#### Diagnostic Checks

1. Compare the original event time with the flow run start time.
2. Identify whether the trigger uses polling or webhook notifications.
3. Review the documented trigger frequency.
4. Check whether the flow was disabled at any relevant time.
5. Investigate pending events or delayed processing.
6. Inspect recent throttling errors.
7. Review service health if delays affect other workflows.
8. Check whether multiple enabled flows monitor the same event.

Do not assume that all event-based triggers execute instantaneously.

#### Potential Resolution

Depending on the cause:

- Allow for the documented polling interval.
- Investigate connector or service delays.
- Reduce unnecessary trigger workload.
- Correct a relevant connection or configuration issue.
- Assess pending events before reactivating a previously disabled flow.
- Implement approved duplicate-processing safeguards.

Do not recommend deleting a flow or resetting its trigger to clear a backlog without explaining the possible loss or reprocessing of business events.

#### Verification

Create a controlled test event.

Record the event timestamp and flow start time.

Confirm whether the observed delay is consistent with the trigger's documented behaviour.

#### Prevention

Document trigger characteristics and design time-sensitive processes with realistic latency expectations.

#### References

- [Optimize Power Automate triggers](https://learn.microsoft.com/en-us/power-automate/guidance/coding-guidelines/optimize-power-automate-triggers)
- [Troubleshoot trigger problems](https://learn.microsoft.com/en-us/troubleshoot/power-platform/power-automate/flow-run-issues/triggers-troubleshoot)
- [Power Automate limits](https://learn.microsoft.com/en-us/power-automate/limits-and-config)

---

### FC-TRIG-004 — Recurrence Schedule and Time-Zone Problems

**Category:** Triggers  
**Trigger:** Recurrence

#### Symptoms

A scheduled cloud flow:

- Runs earlier or later than expected.
- Runs on an unexpected date.
- Does not run at the anticipated time.
- Appears to change its local execution time after a daylight-saving transition.
- Starts earlier than expected after being created.
- Runs according to an unexpected recurrence interval.

#### Possible Causes

- Incorrect time-zone selection.
- Incorrect recurrence frequency.
- Incorrect recurrence interval.
- Incorrect starting date or time.
- UTC being confused with local time.
- Daylight-saving transitions affecting local-time expectations.
- Incorrect weekday or hour selection.
- The flow being disabled or suspended.
- An expected run not being due yet.

#### Required Evidence

- Recurrence trigger configuration.
- Frequency and interval.
- Start time.
- Configured time zone.
- Selected days and times.
- Expected local execution time.
- Actual run timestamps.
- Whether daylight-saving time is relevant.

#### Diagnostic Checks

1. Inspect the Recurrence trigger.
2. Confirm the configured frequency and interval.
3. Check the Start time.
4. Identify the selected time zone.
5. Review any selected days, hours or minutes.
6. Compare the intended local time with the actual run timestamp.
7. Determine whether a daylight-saving transition occurred.
8. Confirm that the flow was enabled and eligible to run.
9. Check whether the observed difference is a small scheduling delay rather than a configuration error.

#### Example

A business expects a report every Monday at 09:00 UK local time.

The maker should configure the relevant recurrence schedule using a suitable UK time zone and the required weekday and hour settings.

A schedule fixed to 09:00 UTC is not equivalent to 09:00 UK local time throughout the entire year.

When British Summer Time is in effect, the UK local time differs from UTC.

The exact correction depends on the actual trigger settings and business requirements.

#### Potential Resolution

Correct the confirmed frequency, interval, start date, scheduled time or time-zone configuration.

Use the Power Automate designer's supported scheduling options.

For local-time requirements, account for daylight-saving transitions.

Do not change production scheduling without approval, especially where the flow sends communications or creates records.

#### Verification

Review the next expected occurrence and compare it with the configured schedule.

Use an authorised test where practical.

Confirm subsequent runs follow the intended schedule.

#### Prevention

Document the intended local time, time zone and schedule assumptions.

Review recurring business processes around daylight-saving transitions.

#### References

- [Run a cloud flow on a schedule](https://learn.microsoft.com/en-us/power-automate/run-scheduled-tasks)
- [Troubleshoot trigger problems](https://learn.microsoft.com/en-us/troubleshoot/power-platform/power-automate/flow-run-issues/triggers-troubleshoot)
- [Convert a time zone](https://learn.microsoft.com/en-us/power-automate/convert-time-zone)
- [Power Automate limits](https://learn.microsoft.com/en-us/power-automate/limits-and-config)

---

### FC-EXEC-001 — Skipped Actions, Conditions and Run-After Behaviour

**Category:** Workflow Execution  
**Common status:** Skipped

#### Symptoms

Examples include:

- An action shows Skipped in run history.
- A flow starts but an expected action does not execute.
- A Condition action follows an unexpected branch.
- A downstream action is skipped after an earlier failure.
- An error-handling action does not run.
- A flow appears successful even though an expected business step was not performed.

#### Possible Causes

- The action belongs to a condition branch that was not selected.
- A preceding action failed.
- A preceding action timed out.
- Run-after settings do not permit execution for the previous status.
- A Scope or dependency did not execute.
- The flow uses incorrect conditional logic.
- The action depends on an unavailable output.
- A Terminate action ended the workflow.

#### Required Evidence

- Run history.
- Status of the skipped action.
- Status of preceding actions.
- Relevant Condition or Scope configuration.
- Run-after settings.
- Expected execution path.

#### Diagnostic Checks

1. Open the relevant flow run.
2. Locate the first action that diverged from expected behaviour.
3. Identify the statuses of preceding actions.
4. Inspect whether the skipped action is inside a Condition branch.
5. Review the actual Condition result.
6. Inspect the action's Run after configuration.
7. Determine whether a dependency failed, timed out or was skipped.
8. Check for Terminate actions.
9. Compare the execution path with the intended business logic.

#### Run-After Behaviour

Power Automate supports configuring whether an action should run after a preceding action has:

- Succeeded.
- Failed.
- Timed out.
- Been skipped.

These are configurable execution outcomes.

A skipped action may be entirely expected when its branch was not selected.

#### Example

A flow contains:

1. Get SharePoint item.
2. Update SharePoint item.
3. Send confirmation email.

If Update SharePoint item fails, the confirmation email may be skipped because its run-after settings require the previous action to succeed.

The correct remedy is not automatically to force the email action to run after failure.

A confirmation email should not claim that an update succeeded if the update failed.

#### Potential Resolution

Depending on the evidence:

- Correct the Condition logic.
- Review the dependency structure.
- Adjust approved run-after settings.
- Move actions to the appropriate branch.
- Introduce suitable error handling.
- Use Scopes for structured Try/Catch-style handling where appropriate.
- Correct the original failed action.

Do not recommend ignoring a failed prerequisite merely to make subsequent actions execute.

#### Verification

Test relevant execution paths:

1. Expected success.
2. Expected business-condition rejection.
3. Relevant failure path.
4. Timeout or skipped dependency when applicable.

Verify that downstream business actions run only when appropriate.

#### Prevention

Document branching logic, failure-handling requirements and action dependencies.

#### References

- [Employ robust error handling](https://learn.microsoft.com/en-us/power-automate/guidance/coding-guidelines/error-handling)
- [Troubleshoot cloud flow errors](https://learn.microsoft.com/en-us/power-automate/troubleshoot-flow-errors)
- [Use expressions in conditions](https://learn.microsoft.com/en-us/power-automate/use-expressions-in-conditions)
- [Troubleshoot a cloud flow](https://learn.microsoft.com/en-us/power-automate/fix-flow-failures)

---

### FC-EXEC-002 — Duplicate Runs, Concurrency and Execution Order

**Category:** Workflow Execution  
**Areas:** Trigger concurrency, parallel branches and Apply to each

#### Symptoms

Examples include:

- Two flow runs process the same record.
- A SharePoint item is created twice.
- A customer receives duplicate emails.
- Multiple runs update the same resource.
- An Apply to each loop processes records in an unexpected order.
- Parallel operations produce inconsistent results.
- The flow triggers itself after updating a monitored resource.

#### Possible Causes

- The source generates multiple matching events.
- Multiple copies of a flow monitor the same events.
- The flow modifies a record monitored by its own trigger.
- Concurrent runs access or update shared resources.
- An Apply to each loop uses parallel execution.
- An external service retries or redelivers an event.
- A previous action completed but its response was lost.
- Duplicate-processing safeguards are missing.

#### Required Evidence

- Number of relevant flow runs.
- Triggering event identifiers.
- Trigger timestamps.
- Target record identifiers.
- Trigger conditions.
- Concurrency settings.
- Apply to each settings.
- External system audit information, when available.

#### Diagnostic Checks

1. Determine whether duplicates came from separate flow runs or repeated actions within one run.
2. Compare event identifiers and timestamps.
3. Check for multiple active flows monitoring the same event.
4. Review trigger conditions and potential self-triggering loops.
5. Inspect trigger concurrency configuration.
6. Review Apply to each concurrency where relevant.
7. Identify shared resources being updated in parallel.
8. Determine whether a retry or ambiguous response may have produced repeated effects.
9. Assess whether the workflow uses an appropriate unique identifier or idempotency safeguard.

#### Concurrency Considerations

Trigger concurrency limits how many flow runs execute simultaneously when enabled.

Apply to each concurrency controls parallel processing within eligible loops.

These are different settings.

Reducing concurrency can help avoid some race conditions but does not, by itself, guarantee that duplicate events or duplicate side effects will never occur.

Microsoft documents that certain trigger and action behaviours can result in repeated processing, making idempotent workflow design important.

#### Example — Duplicate SharePoint Item

A flow receives an event containing a unique RequestId.

Before creating a SharePoint record, an approved design may:

1. Check whether the RequestId has already been processed.
2. Create the record only if it has not been processed.
3. Use an appropriate uniqueness constraint where supported.
4. Record the processing outcome.
5. Handle concurrent attempts safely.

A simple check-then-create sequence is not necessarily sufficient under concurrency because two runs may both observe that the record does not yet exist.

Where correctness requires uniqueness, prefer an atomic uniqueness safeguard or another supported transactional design.

#### Potential Resolution

Depending on the evidence:

- Correct unnecessary duplicate triggers.
- Prevent unintended self-triggering.
- Review overlapping enabled flows.
- Reduce concurrency where business operations must be sequential.
- Introduce suitable uniqueness or idempotency safeguards.
- Review retry configuration.
- Implement approved handling for already-processed events.

Do not recommend deleting production records without understanding the business impact.

Do not claim that setting concurrency to one provides universal exactly-once processing.

#### Verification

Test with:

1. A single valid event.
2. Repeated delivery of the same event.
3. Two closely timed events.
4. Concurrent execution when relevant.

Confirm that the workflow produces the intended result without duplicate business effects.

#### Prevention

Design event-processing workflows to tolerate retries, repeated events and concurrency safely.

#### References

- [Troubleshoot trigger problems](https://learn.microsoft.com/en-us/troubleshoot/power-platform/power-automate/flow-run-issues/triggers-troubleshoot)
- [Optimize Power Automate triggers](https://learn.microsoft.com/en-us/power-automate/guidance/coding-guidelines/optimize-power-automate-triggers)
- [Optimize parallel execution and concurrency](https://learn.microsoft.com/en-us/power-automate/guidance/coding-guidelines/implement-parallel-execution)
- [Power Automate limits and configuration](https://learn.microsoft.com/en-us/power-automate/limits-and-config)

---

## 5. General Diagnostic Decision Guide

### Step A — Did the Flow Start?

**No expected run found**

Investigate:

- Flow status.
- Trigger configuration.
- Connection health.
- Trigger filters and conditions.
- Event eligibility.
- Trigger mechanism.
- Relevant service or policy restrictions.

Use records FC-TRIG-001 through FC-TRIG-004.

**Run exists**

Continue to Step B.

### Step B — What Happened During Execution?

Inspect the run's actions and their statuses.

- Succeeded.
- Failed.
- Timed out.
- Skipped.

Identify the first unexpected status or result.

### Step C — Did a Condition or Dependency Affect Execution?

Inspect:

- Condition results.
- Selected branches.
- Run-after configuration.
- Relevant Scopes.
- Terminate actions.
- Dependencies between actions.

Use FC-EXEC-001.

### Step D — Were Multiple Runs or Actions Involved?

If duplicate results or unexpected ordering occurred, investigate:

- Duplicate source events.
- Multiple active flows.
- Trigger concurrency.
- Apply to each concurrency.
- Retries.
- Idempotency safeguards.

Use FC-EXEC-002.

### Step E — Select the Appropriate Knowledge Area

For other observed failures, consult:

- **KB-01:** Authentication and connections.
- **KB-02:** Permissions and authorisation.
- **KB-03:** Expressions and data handling.
- **KB-04:** Throttling, timeouts and service errors.
- **KB-06:** Safe troubleshooting and escalation.

A flow may exhibit more than one failure category. FlowCheck should explain the relationship between them without assuming they share a single root cause.

### Step F — Recommend a Safe Correction

Suggest a change only when the evidence supports it.

Clearly distinguish:

- Observed facts.
- Likely causes.
- Recommended checks.
- Potential remedies.
- Required approvals.
- Verification steps.

### Step G — Verify the Business Outcome

Confirm that the expected event or run occurred and the intended result was produced.

A successful execution status alone may not establish that business requirements were met.

## 6. Evidence Requirements

For trigger-related cases, request only the information needed:

| Evidence | Purpose |
|---|---|
| Trigger type | Identifies expected behaviour |
| Flow status | Establishes whether the flow can operate |
| Expected event time | Helps investigate timing |
| Trigger settings | Identifies configured filters and resources |
| Sanitised trigger checks | Identifies excluded or failed checks |
| Connection status | Helps rule out authentication issues |

For execution-related cases:

| Evidence | Purpose |
|---|---|
| Run status | Shows overall execution outcome |
| Failing or skipped action | Identifies where behaviour diverged |
| Preceding action status | Reveals dependencies |
| Condition result | Explains branch selection |
| Run-after settings | Explains execution eligibility |
| Relevant run IDs | Helps investigate duplicate processing |

Do not request entire sensitive payloads when a sanitised excerpt is sufficient.

## 7. Confidence Guidance

**High confidence**

The trigger checks, run history, action statuses and configuration clearly support the identified cause.

**Medium confidence**

The available evidence supports a likely cause, but some configuration or event details remain unconfirmed.

**Low confidence**

The event, trigger configuration or execution history is missing or contradictory.

FlowCheck must state uncertainty explicitly.

Confidence in the failure category may differ from confidence in the underlying root cause.

## 8. Responsible Troubleshooting

FlowCheck must:

- Never claim direct access to a Power Automate environment.
- Never claim to have enabled, disabled or modified a flow.
- Never request passwords, access tokens or client secrets.
- Encourage sanitised evidence.
- Avoid advising destructive changes without approval.
- Avoid unnecessary production resubmissions.
- Consider duplicate business effects before rerunning workflows.
- Never recommend bypassing organisational security policies.
- Avoid treating expected skipped actions as failures.
- Distinguish trigger failures from execution failures.
- Avoid inventing trigger events, run histories or action statuses.
- Recommend authorised administrators where policy or permission changes are required.
- State when the evidence does not support a definitive diagnosis.

## 9. Escalation Guidance

Recommend escalation when:

- Trigger failures persist after relevant checks.
- A service or connector incident is suspected.
- An organisational policy prevents operation.
- An identity or permissions problem requires administrator action.
- A production workflow has caused duplicate or inconsistent business data.
- There is a risk of financial, operational or customer-facing consequences.
- The source event cannot be confirmed.
- An unexpected execution problem cannot be reproduced safely.

Provide an escalation summary containing:

- Observed symptoms.
- Relevant timestamps and time zone.
- Sanitised error messages.
- Affected trigger or action.
- Evidence gathered.
- Checks performed.
- Remaining uncertainty.
- Business impact.
- Suggested next responsible owner.

## 10. Technical References

Official Microsoft documentation:

1. [Troubleshoot Power Automate trigger problems](https://learn.microsoft.com/en-us/troubleshoot/power-platform/power-automate/flow-run-issues/triggers-troubleshoot)
2. [Troubleshoot cloud flow errors](https://learn.microsoft.com/en-us/power-automate/troubleshoot-flow-errors)
3. [Cloud flow error code reference](https://learn.microsoft.com/en-us/power-automate/error-reference)
4. [Customize triggers with conditions](https://learn.microsoft.com/en-us/power-automate/customize-triggers)
5. [Optimize Power Automate triggers](https://learn.microsoft.com/en-us/power-automate/guidance/coding-guidelines/optimize-power-automate-triggers)
6. [Run a cloud flow on a schedule](https://learn.microsoft.com/en-us/power-automate/run-scheduled-tasks)
7. [Convert a time zone](https://learn.microsoft.com/en-us/power-automate/convert-time-zone)
8. [Employ robust error handling](https://learn.microsoft.com/en-us/power-automate/guidance/coding-guidelines/error-handling)
9. [Optimize parallel execution](https://learn.microsoft.com/en-us/power-automate/guidance/coding-guidelines/implement-parallel-execution)
10. [Power Automate limits and configuration](https://learn.microsoft.com/en-us/power-automate/limits-and-config)
11. [Troubleshoot a cloud flow](https://learn.microsoft.com/en-us/power-automate/fix-flow-failures)
12. [Turn a flow on or off](https://learn.microsoft.com/en-us/power-automate/disable-flow)

> **Maintenance note:** Update this reference when Microsoft changes connection-management navigation, connector authentication behaviour or identity-policy guidance. The above links are source references, not claims that FlowCheck has live access to 's current seMicrosoftrvice health or a user's environment.

This is an independent, community-maintained troubleshooting reference and is not an official Microsoft support publication.

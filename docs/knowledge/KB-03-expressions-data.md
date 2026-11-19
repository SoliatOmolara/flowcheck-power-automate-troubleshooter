---
Title: "KB-03 — Expressions and Data Handling"
Description: "Evidence-based troubleshooting for Power Automate expression, JSON, null-value and data-conversion failures."
---

# KB-03 — Expressions and Data Handling

> **FlowCheck — Power Automate Troubleshooting Assistant**    Version **1.0.0**    Reviewed **2026-11-19**  
> **Coverage:** Expressions and Data; Microsoft Power Automate cloud flows
> **Status:** Community troubleshooting guidance, not an official Microsoft support service.

[← Knowledge-base home](../index.html)

## 1. Purpose

This reference supports the diagnosis of expression and data-handling problems in Microsoft Power Automate cloud flows.

It provides guidance for interpreting expression errors, inspecting action outputs, understanding JSON structures, handling missing data, validating data types and troubleshooting references to previous actions.

FlowCheck must use the actual error message, expression and available data structure to determine which explanation is supported.

The objective is to help users understand why an expression fails and identify a technically justified correction.

FlowCheck provides advisory guidance only and does not directly inspect, execute, modify or repair cloud flows.

## 2. Common Symptoms

Expression and data-handling failures may include:

- `InvalidTemplate` errors.
- `ExpressionEvaluationFailed` errors.
- "Unable to process template language expressions."
- "The property 'Email' doesn't exist."
- "The provided value is of type 'Null'."
- An action expects an array but receives an object.
- A function expects a string but receives a null value.
- A Compose action returns an unexpected result.
- Dynamic content references the wrong action or property.
- A Parse JSON action fails because the input doesn't match its schema.
- An expression references an action that was skipped.
- A flow completes successfully but uses incorrect values.

These symptoms indicate that an expression, data transformation or action configuration requires investigation. They do not always establish the precise cause.

## 3. Diagnostic Principles

Before recommending a correction:

1. Identify the failing action and its exact error.
2. Inspect the expression being evaluated.
3. Identify the action or trigger supplying the data.
4. Examine sanitised input and output structures.
5. Determine the data type being received.
6. Compare the actual structure with the expected structure.
7. Check for missing, null or empty values.
8. Identify whether the referenced action actually executed.
9. Recommend a correction supported by the observed evidence.
10. Explain how to verify the corrected behaviour.

Do not invent property names, JSON structures or expression results.

Do not assume that every successful Power Automate run produces the intended business outcome.

## 4. Diagnostic Records

### FC-EXPR-001 — Missing Property or Incorrect JSON Path

**Category:** Expressions and Data  
**Typical actions:** Compose, Set variable, Create item  
**Common errors:** InvalidTemplate, missing property

#### Symptoms

A Power Automate expression attempts to access a property that isn't available at the referenced location.

Example:

"The property 'Email' doesn't exist."

#### Possible Causes

- The expression references the wrong object level.
- The property name has different capitalisation.
- The property is nested within another object.
- The property exists inside an array.
- The referenced action returned a different structure.
- The expected property is missing from the actual response.

#### Required Evidence

- Failing action.
- Actual expression.
- Sanitised output from the referenced action.
- Expected value.

#### Example

A SharePoint Get items action returns:

```json
{
  "value": [
    {
      "Name": "Amina",
      "Email": "amina@example.com"
    },
    {
      "Name": "James",
      "Email": "james@example.com"
    }
  ]
}
```

The user enters:

```text
body('Get_items')?['Email']
```

This expression is incorrect because `Email` is not a top-level property of the response body.

The top-level property is `value`, which contains an array.

To retrieve the first object's Email property:

```text
first(body('Get_items')?['value'])?['Email']
```

For the supplied sample, the expected result is:

```text
amina@example.com
```

This expression assumes that the array contains at least one item.

It is not appropriate for retrieving every email address.

#### Diagnostic Checks

1. Open the failed action in run history.
2. Inspect its Inputs and Outputs.
3. Identify the object containing the required property.
4. Compare the property name and capitalisation.
5. Determine whether the value is nested inside an array.
6. Verify that the referenced action name is correct.
7. Check whether the relevant array can be empty.

#### Potential Resolution

Correct the expression path based on the actual output structure.

Use a condition before accessing the first element when empty arrays are possible.

#### Verification

Run a controlled test.

Confirm that the corrected expression produces the expected value, not merely a successful action status.

#### Prevention

Use descriptive action names, inspect actual output schemas and validate expression paths before deploying flows.

#### References

- [Workflow expression functions](https://learn.microsoft.com/en-us/azure/logic-apps/expression-functions-reference)
- [Cloud flow error reference](https://learn.microsoft.com/en-us/power-automate/error-reference)

---

### FC-EXPR-002 — Null Values and Empty Data

**Category:** Expressions and Data  
**Common errors:** InvalidTemplate, null parameter, expression evaluation failures

#### Symptoms

An expression fails because it receives a null or empty value instead of the expected data.

Examples include:

- A text-processing function receives null.
- A property is missing from an object.
- An array contains no elements.
- A date conversion receives an empty value.
- A required action input is null.

#### Possible Causes

- An optional field was not populated.
- The preceding action returned no matching records.
- The connector returned null instead of a string or object.
- The expression assumes data is always present.
- A function receives a value of the wrong type.

#### Required Evidence

- Exact error.
- Input value and data type.
- Expression being evaluated.
- Relevant preceding action outputs.
- Whether empty or null values are expected.

#### Diagnostic Checks

1. Inspect the failing expression.
2. Identify which value may be null.
3. Examine the preceding action output.
4. Distinguish null from an empty string, object or array.
5. Determine whether the workflow should reject missing data or supply a default.
6. Review the potential business consequences of any default value.

#### Example — Default for a Null Value

If a property may be null, the following pattern can provide a default:

```text
coalesce(triggerBody()?['Department'], 'Not specified')
```

The `coalesce()` function returns the first non-null argument.

Important: Empty strings, arrays and objects are not null. Therefore, `coalesce()` does not automatically replace every empty value.

#### Example — Guard Against an Empty Array

Before retrieving the first item:

```text
empty(body('Get_items')?['value'])
```

Use the result in a Condition.

- True branch: Handle the no-records case.
- False branch: Retrieve the first item.

Example for the non-empty branch:

```text
first(body('Get_items')?['value'])?['Email']
```

Do not assume that optional property access using `?` makes all downstream functions safe. An empty array can still make an operation such as `first()` inappropriate.

#### Potential Resolution

Introduce appropriate null or empty-data handling based on the actual business requirement.

Use defaults only when the default value is valid for the workflow.

#### Verification

Test with:

- A populated value.
- A null value.
- An empty string when relevant.
- An empty array when relevant.

Confirm that the workflow handles each scenario as intended.

#### Prevention

Design explicit validation for optional fields and responses that may contain no records.

#### References

- [Getting errors with null fields](https://learn.microsoft.com/en-us/troubleshoot/power-platform/power-automate/flow-run-issues/getting-errors-null-fields)
- [Workflow expression functions](https://learn.microsoft.com/en-us/azure/logic-apps/expression-functions-reference)
- [Expression cookbook](https://learn.microsoft.com/en-us/power-automate/expression-cookbook)

---

### FC-EXPR-003 — Array Versus Object Access

**Category:** Expressions and Data  
**Common actions:** Get items, List rows, Apply to each, Compose

#### Symptoms

An expression fails or produces incorrect data because it treats an array as a single object, or an object as an array.

#### Possible Causes

- The connector returned multiple records.
- The user expected one record but received an array.
- The expression references a property without selecting an array item.
- An Apply to each loop uses the wrong input.
- The expression assumes an item exists when the array is empty.

#### Required Evidence

- Actual output structure.
- Expression or loop configuration.
- Whether the user wants one record or multiple records.
- Expected output.

#### Diagnostic Checks

1. Determine whether the relevant value is an array or object.
2. Inspect the array's contents.
3. Identify the required property.
4. Determine whether the workflow should use one item or all items.
5. Check for empty arrays.
6. Review the Apply to each input, if a loop is used.

#### Example — Retrieve One Record

For a non-empty `value` array:

```text
first(body('Get_items')?['value'])?['Email']
```

This retrieves the first item's Email property.

Use this only when selecting the first item satisfies the user's requirement.

#### Example — Process Every Record

Configure Apply to each with:

```text
body('Get_items')?['value']
```

Inside the loop, retrieve the current item's email:

```text
item()?['Email']
```

This uses the current object in each iteration.

#### Example — Named Loop

Where the expression must explicitly refer to a particular loop:

```text
items('Apply_to_each')?['Email']
```

The loop name must match the actual action's internal name.

#### Potential Resolution

Use an appropriate array item reference or a loop based on the intended business result.

Do not select the first record merely because it avoids an expression error.

#### Verification

Test with:

- An array containing one item.
- An array containing multiple items.
- An empty array.

Confirm that the correct records are processed and that no expected data is silently omitted.

#### Prevention

Document expected data shapes and use appropriate array-handling operations.

#### References

- [Use data operations in Power Automate](https://learn.microsoft.com/en-us/power-automate/data-operations)
- [Workflow expression functions](https://learn.microsoft.com/en-us/azure/logic-apps/expression-functions-reference)

---

### FC-EXPR-004 — Data-Type and Conversion Failure

**Category:** Expressions and Data  
**Common errors:** ContentConversionFailed, ExpressionEvaluationFailed

#### Symptoms

A flow attempts to convert or process data using an incompatible type.

Examples include:

- Converting non-numeric text into an integer.
- Using a string where an array is expected.
- Passing an object into an incompatible action input.
- Converting an invalid date string.
- Comparing a number with an unexpected text value.

#### Possible Causes

- The source data has a different type than expected.
- The input contains malformed values.
- A conversion function receives null.
- Regional date or number formats differ.
- The user assumes the connector always returns a consistent type.

#### Required Evidence

- Actual input value.
- Input type.
- Conversion expression.
- Expected output type.
- Exact error message.

#### Diagnostic Checks

1. Identify the conversion function.
2. Inspect the actual value being converted.
3. Determine its data type.
4. Check for null or empty input.
5. Validate format requirements.
6. Determine how invalid input should be handled.

#### Example — Numeric Conversion

The following expression expects a value that can be converted into an integer:

```text
int(variables('Quantity'))
```

If the variable contains an incompatible value, conversion may fail.

Where the business requirement permits a fallback:

```text
if(
  isInt(variables('Quantity')),
  int(variables('Quantity')),
  0
)
```

This uses zero as a fallback only when the input is not a valid integer.

Do not use zero as a default without considering business consequences. For financial, inventory or safety-critical calculations, rejecting invalid data may be more appropriate.

#### Example — Date Conversion

When an expression fails while formatting a date:

1. Inspect the incoming date value.
2. Check whether it is null.
3. Confirm its actual format.
4. Review the expected input format of the function.
5. Verify relevant locale and time-zone assumptions.

Do not invent a corrected date expression without the actual input and intended result.

#### Potential Resolution

Validate the input before applying type conversion.

Use explicit conversion, a suitable default or controlled rejection depending on the business requirement.

#### Verification

Test with valid values, invalid values and relevant boundary cases.

Confirm both the output type and business result.

#### Prevention

Document expected data types and validate externally supplied values before transformation.

#### References

- [Cloud flow error code reference](https://learn.microsoft.com/en-us/power-automate/error-reference)
- [Workflow expression functions](https://learn.microsoft.com/en-us/azure/logic-apps/expression-functions-reference)
- [Expression cookbook](https://learn.microsoft.com/en-us/power-automate/expression-cookbook)

---

### FC-EXPR-005 — Parse JSON and Schema Mismatch

**Category:** Expressions and Data  
**Typical action:** Parse JSON

#### Symptoms

A Parse JSON action fails, rejects input or does not expose the expected properties.

#### Possible Causes

- The input structure differs from the configured schema.
- A property has a different data type.
- A property marked as required is missing.
- A nullable property is declared as a non-nullable type.
- The schema expects an object but receives an array.
- The input is not valid JSON for the intended operation.
- The input has changed since the schema was created.

#### Required Evidence

- Exact Parse JSON error.
- Sanitised input data.
- Configured schema.
- Expected output structure.
- Details of the action supplying the input.

#### Diagnostic Checks

1. Inspect the Parse JSON action's Content input.
2. Confirm the input is the intended JSON data.
3. Determine whether the root value is an object or array.
4. Compare the actual data with the configured schema.
5. Review property names and declared types.
6. Check whether optional fields are incorrectly required.
7. Check whether properties may legitimately be null.
8. Determine whether the input source has changed.

#### Example — Nullable Property

A schema may define a nullable property using more than one accepted type:

```json
{
  "type": "object",
  "properties": {
    "Email": {
      "type": ["string", "null"]
    }
  }
}
```

This example allows the Email property to contain either a string or null.

It does not, by itself, make the property required.

The schema should represent the actual data contract rather than simply being weakened until validation succeeds.

#### Potential Resolution

Correct the source data or update the schema to accurately reflect valid expected inputs.

Avoid removing useful validation constraints without understanding why they exist.

#### Verification

Test the Parse JSON action against representative valid inputs and relevant edge cases.

Confirm that downstream actions receive the expected properties and types.

#### Prevention

Maintain documented JSON contracts and review schema changes when external APIs or connector outputs change.

#### References

- [Use data operations](https://learn.microsoft.com/en-us/power-automate/data-operations)
- [Work with data operations](https://learn.microsoft.com/en-us/power-automate/guidance/coding-guidelines/use-data-operations)
- [Cloud flow error code reference](https://learn.microsoft.com/en-us/power-automate/error-reference)

---

### FC-EXPR-006 — Expression References Unavailable Action Output

**Category:** Expressions and Data  
**Related category:** Workflow Execution

#### Symptoms

An expression references an earlier action's output, but the expected data is unavailable.

This may occur when an action was skipped, failed, or did not execute along the relevant workflow path.

#### Possible Causes

- The referenced action belongs to a condition branch that was not taken.
- The referenced action was skipped because an earlier dependency failed.
- The expression accesses output from an action that did not complete successfully.
- The action name in the expression is incorrect.
- The workflow structure changed without updating dependent expressions.
- A value from a parallel branch is not guaranteed to be available when referenced.

#### Required Evidence

- Failing expression.
- Referenced action name.
- Run-history action statuses.
- Relevant condition or branch.
- Expected data source.

#### Diagnostic Checks

1. Open the failed run.
2. Locate the action referenced by the expression.
3. Determine whether it succeeded, failed or was skipped.
4. Inspect relevant parent conditions or scopes.
5. Check whether the referenced action is in another branch.
6. Review the dependency and run-after configuration.
7. Identify a data source guaranteed to be available at the point of use.

#### Example

An expression references:

```text
body('Get_manager')?['mail']
```

However, the Get manager action runs only in one branch of a Condition.

If another branch executes, the expected output may not be available.

Do not assume optional property access can resolve the absence of a required action result.

#### Potential Resolution

Depending on the intended workflow behaviour:

- Move the dependent action into the appropriate branch.
- Use data produced in an earlier common step.
- Initialise and populate a variable through an explicitly designed path.
- Add suitable conditions and dependency handling.
- Review run-after configuration where relevant.

Choose the correction based on actual workflow dependencies.

#### Verification

Test the affected execution branches.

Confirm that each branch handles the required data appropriately and does not reference unavailable outputs.

#### Prevention

Document action dependencies, avoid unnecessary cross-branch references and test alternative condition paths.

#### References

- [Troubleshoot cloud flow errors](https://learn.microsoft.com/en-us/power-automate/troubleshoot-flow-errors)
- [Cloud flow error code reference](https://learn.microsoft.com/en-us/power-automate/error-reference)
- [Workflow expression functions](https://learn.microsoft.com/en-us/azure/logic-apps/expression-functions-reference)

---

## 5. General Expression Diagnostic Decision Guide

When an expression or data-handling error occurs, follow this sequence.

### Step A — Identify the failure

Determine the failing action, exact error message and expression.

### Step B — Inspect the input

Check the relevant action's Inputs and the preceding action's Outputs.

### Step C — Determine the data structure

Identify whether the value is:

- A string
- A number
- A boolean
- An object
- An array
- Null

### Step D — Check the property path

Confirm that the expression references the correct action, object, array element and property.

### Step E — Evaluate missing data

Consider missing properties, null values, empty arrays and skipped actions.

### Step F — Identify the supported correction

Recommend only a correction justified by the available evidence.

If evidence is insufficient, ask a targeted follow-up question.

### Step G — Verify

Run a controlled test.

Confirm the expected value, output structure and business outcome.

Do not treat a successful action status as sufficient verification when the actual output remains incorrect.

## 6. Expression Pattern Reference

The following patterns are illustrative and depend on the actual workflow structure.

| Purpose | Expression |
|---|---|
| Read action body | `body('Action_name')` |
| Read a property | `body('Action_name')?['Property']` |
| Read a nested property | `body('Action_name')?['Parent']?['Child']` |
| Read first array item | `first(body('Action_name')?['value'])` |
| Check empty array | `empty(body('Action_name')?['value'])` |
| Provide null fallback | `coalesce(triggerBody()?['Field'], 'Default')` |
| Current loop item property | `item()?['Email']` |
| Named loop item property | `items('Apply_to_each')?['Email']` |
| Convert valid numeric text | `int(variables('Quantity'))` |

These are not universal fixes.

FlowCheck must verify that the referenced action, data structure and expected result match the user's actual workflow before recommending an expression.

## 7. Confidence Guidance

**High confidence:** The actual expression and input structure clearly demonstrate the mismatch or evaluation problem.

**Medium confidence:** The error suggests a likely expression or data issue, but some relevant output evidence is missing.

**Low confidence:** The data structure, expression, action status or source value is unknown or contradictory.

Confidence in the error category can differ from confidence in the specific root cause.

## 8. Responsible Troubleshooting

FlowCheck must:

- Never request passwords, access tokens, API keys or client secrets.
- Encourage sanitised sample JSON and error details.
- Never claim direct access to the user's Power Automate environment.
- Never claim to have executed or corrected expressions.
- Never invent JSON properties or sample values as confirmed evidence.
- Avoid suggesting the removal of validation without examining its purpose.
- Consider downstream effects before recommending a flow rerun.
- Distinguish suggested corrections from user-confirmed fixes.
- Acknowledge when the correct expression cannot yet be determined.

## 9. Technical References

Official Microsoft documentation:

1. [Workflow expression functions](https://learn.microsoft.com/en-us/azure/logic-apps/expression-functions-reference)
2. [Use data operations in Power Automate](https://learn.microsoft.com/en-us/power-automate/data-operations)
3. [Troubleshoot cloud flow errors](https://learn.microsoft.com/en-us/power-automate/troubleshoot-flow-errors)
4. [Cloud flow error code reference](https://learn.microsoft.com/en-us/power-automate/error-reference)
5. [Getting errors with null fields](https://learn.microsoft.com/en-us/troubleshoot/power-platform/power-automate/flow-run-issues/getting-errors-null-fields)
6. [Expression cookbook for cloud flows](https://learn.microsoft.com/en-us/power-automate/expression-cookbook)
7. [Work with data operations](https://learn.microsoft.com/en-us/power-automate/guidance/coding-guidelines/use-data-operations)
8. [Use expressions in conditions](https://learn.microsoft.com/en-us/power-automate/use-expressions-in-conditions)

> **Maintenance note:** Update this reference when Microsoft changes connection-management navigation, connector authentication behaviour or identity-policy guidance. The above links are source references, not claims that FlowCheck has live access to 's current seMicrosoftrvice health or a user's environment.

This is an independent, community-maintained troubleshooting reference, not an official Microsoft support publication.

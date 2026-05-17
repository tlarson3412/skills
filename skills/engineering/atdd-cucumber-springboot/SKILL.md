---
name: atdd-cucumber-springboot
description: Use this skill when creating, reviewing, extending, or refactoring ATDD / BDD Cucumber tests for Spring Boot Java APIs.
---

# ATDD / Cucumber Skill for Spring Boot Java APIs

Use this skill when working on Cucumber, Gherkin, ATDD, BDD, API acceptance tests, regression tests, or Spring Boot test automation.

The goal is to create clear, maintainable, business-readable Cucumber tests that validate Spring Boot API behavior without creating brittle, duplicated, or overly technical test code.

---

## Goals

Prioritize:

- readable Gherkin
- reusable step definitions
- scenario-scoped test state
- deterministic test data
- meaningful API assertions
- safe handling of sensitive data
- consistency with the existing repository
- small, working changes over large framework rewrites

When unsure, inspect the existing repository first and follow its established patterns unless they are clearly broken.

---

## Before Making Changes

Before creating or modifying tests, inspect the existing project.

Look for:

- `pom.xml`, `build.gradle`, or `build.gradle.kts`
- existing `src/test/java`
- existing `src/test/resources`
- existing `.feature` files
- existing step definitions
- existing Cucumber runner or suite classes
- existing Spring test configuration
- existing API clients, mocks, stubs, fixtures, or test data factories
- Swagger/OpenAPI files if available
- existing CI/CD test commands or profiles

Do not add new dependencies unless they are truly missing.

Do not create a new testing pattern if the repository already has a working one.

---

## Preferred Project Structure

Use the existing repository structure if one already exists.

If creating a new ATDD suite from scratch, prefer this structure:

```text
src/test/java/.../atdd/
  CucumberTest.java
  CucumberSpringConfiguration.java

src/test/java/.../atdd/steps/
  CommonSteps.java
  ApiRequestSteps.java
  ApiResponseSteps.java
  <Domain>Steps.java

src/test/java/.../atdd/support/
  TestContext.java
  ApiClient.java
  TestDataFactory.java
  JsonAssertions.java

src/test/resources/features/
  <domain>/
    <api-name>.feature
```

For APIs with multiple consumers, organize consumer-specific coverage clearly:

```text
src/test/resources/features/
  consumers/
    consumer-a/
    consumer-b/
```

---

## Gherkin Rules

Feature files should describe API behavior, not Java implementation details.

Use business/API language that product owners, QA, developers, and support teams can understand.

Good:

```gherkin
Scenario: Retrieve an active account
  Given an active account exists
  When the consumer retrieves the account by account id
  Then the response status should be 200
  And the response should contain the account summary
```

Avoid overly technical wording:

```gherkin
Scenario: Invoke controller method with mock request
  Given I create a RestTemplate exchange request
  When I call the GET endpoint
  Then JSON path "$.accountStatus" equals "ACTIVE"
```

Technical details belong in step definitions or support classes, not in feature files.

---

## Scenario Design Rules

When adding scenarios, consider coverage for:

- happy path
- missing required fields
- invalid request fields
- invalid headers
- invalid query parameters
- invalid path parameters
- unauthorized access
- forbidden access
- not found
- conflict or duplicate request
- downstream timeout
- downstream unavailable
- unexpected downstream response
- sensitive data masking
- consumer-specific behavior
- backward compatibility

Keep each scenario focused on one behavior.

Use `Scenario Outline` only when the same behavior is being tested with different inputs.

Avoid large examples tables unless they clearly improve maintainability.

Do not assert every response field unless the requirement calls for it.

---

## Tags

Use meaningful tags so tests can be filtered in local runs and CI/CD.

Preferred examples:

```gherkin
@atdd
@api
@smoke
@regression
@negative
@security
@consumer-name
@account-lookup
```

Use tags consistently.

Do not create excessive or unclear tags.

---

## Step Definition Rules

Before creating a new step definition, search for an existing reusable step.

Step definitions should be:

- small
- reusable
- readable
- deterministic
- not duplicated
- not dependent on test execution order
- not overloaded with unrelated behavior

Prefer Cucumber expressions over complex regex when possible.

Good:

```java
@Then("the response status should be {int}")
public void theResponseStatusShouldBe(int expectedStatus) {
    assertThat(testContext.getResponse().getStatusCode().value())
        .isEqualTo(expectedStatus);
}
```

Avoid creating many one-off steps when a reusable step already exists.

Avoid overly generic steps that make feature files hard to understand.

---

## Scenario State Management

Use scenario-scoped context for sharing request data, response data, IDs, and test objects between steps.

Do not use shared static mutable state.

Avoid:

- static request objects
- static response objects
- global mutable maps
- tests that rely on scenario execution order
- one scenario depending on data created by another scenario

Preferred approach:

```java
@Component
@ScenarioScope
public class TestContext {

    private ResponseEntity<String> response;
    private final Map<String, Object> values = new HashMap<>();

    public ResponseEntity<String> getResponse() {
        return response;
    }

    public void setResponse(ResponseEntity<String> response) {
        this.response = response;
    }

    public void put(String key, Object value) {
        values.put(key, value);
    }

    @SuppressWarnings("unchecked")
    public <T> T get(String key, Class<T> type) {
        return (T) values.get(key);
    }
}
```

Adapt this to the repository’s existing style.

---

## Spring Boot / Cucumber Guidance

For Spring Boot API ATDD tests, prefer testing through HTTP when the goal is API behavior validation.

Use Spring Boot test configuration that matches the repository’s style.

If creating a new suite, a common pattern is:

```java
@CucumberContextConfiguration
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@ActiveProfiles("test")
public class CucumberSpringConfiguration {
}
```

Use `RANDOM_PORT` for HTTP-based API tests.

Do not introduce a new test style if the project already has a working Cucumber/Spring configuration.

Do not introduce JUnit 4 into a JUnit 5 project unless the repository already uses JUnit 4.

---

## API Request Rules

Prefer reusable request steps for common HTTP behavior.

Request steps should support:

- HTTP method
- endpoint path
- path variables
- query parameters
- headers
- request body
- authentication setup when applicable
- storing the response in scenario context

Avoid hardcoding environment-specific URLs.

Use test configuration, profiles, or existing base URL utilities.

---

## API Assertion Rules

Prefer stable, meaningful assertions.

Assert things like:

- HTTP status code
- required response fields
- important business values
- error code
- error message format
- response content type
- response shape
- sensitive data absent or masked
- headers when relevant

Avoid brittle assertions against:

- full raw JSON strings
- field order
- exact timestamps
- generated IDs
- trace IDs
- unrelated fields
- environment-specific values

For dynamic values, assert presence, format, or relationship instead of exact value.

---

## JSON Assertion Rules

Prefer JSON path or object-based assertions.

Good examples:

```gherkin
Then the response field "accountStatus" should be "ACTIVE"
And the response field "accountId" should be present
And the response should not contain sensitive card data
```

Avoid comparing full JSON strings unless there is a specific reason.

Do not over-assert fields unrelated to the scenario.

---

## Test Data Rules

Use safe fake data only.

Never use real:

- card numbers
- CVV
- SSN
- passwords
- tokens
- customer names
- production account numbers
- private keys
- certificates
- real customer identifiers

Prefer reusable test data builders or factories.

Examples:

```java
TestDataFactory.activeAccount()
TestDataFactory.closedAccount()
TestDataFactory.consumerWithReadAccess()
TestDataFactory.consumerWithoutAccess()
```

Avoid scattering magic values across step definitions.

---

## Security and Sensitive Data Rules

Do not put sensitive data in:

- feature files
- step definitions
- test resources
- logs
- generated reports
- screenshots
- comments

For PCI-like data, use masked or fake test values.

Examples:

```text
411111******1111
************1111
```

When relevant, add tests to confirm sensitive data is masked or absent.

---

## Swagger / OpenAPI Usage

If Swagger or OpenAPI files exist, use them to understand:

- endpoint paths
- HTTP methods
- required headers
- path parameters
- query parameters
- request body schema
- response schema
- validation rules
- error responses
- enum values

Do not blindly generate tests for every endpoint.

Prioritize:

- business-critical endpoints
- high-risk fields
- required validations
- known incident areas
- consumer-specific behavior
- security-sensitive responses
- error paths that are easy to break

---

## Mainframe / Downstream Integration Awareness

Some APIs may depend on CICS transactions, mainframe copybooks, downstream services, or legacy return codes.

Keep Gherkin focused on API behavior, not internal mainframe implementation.

Good:

```gherkin
Scenario: Downstream account lookup returns not found
  Given the account lookup downstream returns not found
  When the consumer retrieves the account by account id
  Then the response status should be 404
  And the error code should be "ACCOUNT_NOT_FOUND"
```

Avoid exposing raw legacy payloads or sensitive downstream details in feature files.

When legacy return codes matter, assert the API-level result.

Example:

```gherkin
Then the response status should be 400
And the error code should be "INVALID_ACCOUNT_STATUS"
```

---

## When Creating a New ATDD Suite

Create a small working vertical slice first.

Steps:

1. Inspect the existing test setup.
2. Add Cucumber dependencies only if missing.
3. Add a Cucumber runner or suite class.
4. Add Spring test configuration.
5. Add shared scenario context.
6. Add reusable request and response steps.
7. Add one feature file for one representative endpoint.
8. Add basic reporting output if not already configured.
9. Explain how to run the tests.

Do not build a huge framework all at once.

Do not rewrite the entire test structure unless explicitly requested.

---

## When Adding New Test Cases

When adding test cases for an API:

1. Inspect the controller, DTOs, service behavior, and existing tests.
2. Check Swagger/OpenAPI if available.
3. Check existing feature files for related coverage.
4. Reuse existing step definitions where possible.
5. Add new step definitions only when needed.
6. Add or reuse test data setup.
7. Add meaningful assertions.
8. Keep scenarios focused.
9. Summarize the coverage added.
10. Provide the test command to run.

---

## When Reviewing Existing ATDD Tests

When asked to review an existing ATDD/Cucumber suite, evaluate:

- Are feature files readable?
- Are scenarios focused?
- Are steps duplicated?
- Are step definitions too technical?
- Is test state scenario-scoped?
- Are tests deterministic?
- Are assertions meaningful but not brittle?
- Is test data centralized?
- Are tags useful?
- Are security and negative cases covered?
- Are consumer-specific scenarios organized clearly?
- Are there sleeps or timing assumptions?
- Are dependencies consistent with the project?

Provide practical recommendations.

Prioritize fixes that improve maintainability, reliability, and coverage.

---

## CI/CD and Running Tests

Use the repository’s existing test commands if available.

For Maven projects, common examples are:

```bash
mvn test
mvn verify
mvn test -Dcucumber.filter.tags="@smoke"
mvn verify -Dcucumber.filter.tags="@regression"
```

For Gradle projects, common examples are:

```bash
./gradlew test
./gradlew test -Dcucumber.filter.tags="@smoke"
```

Do not assume these commands are correct if the repository already documents a different command.

When adding tests, explain the most relevant command to run.

---

## Reporting

Prefer simple reports that work locally and in CI.

Common Cucumber report outputs:

```text
pretty
html:target/cucumber-report.html
json:target/cucumber-report.json
junit:target/cucumber-report.xml
```

Use the repository’s existing reporting pattern if one already exists.

Do not add complex reporting tools unless requested.

---

## Output Expectations

When finished making changes, summarize:

- files created or changed
- scenarios added or updated
- step definitions added or reused
- dependencies added, if any
- assumptions made
- how to run the tests

Use a concise format like:

```text
Changed:
- src/test/resources/features/accounts/account-lookup.feature
- src/test/java/.../atdd/steps/AccountSteps.java

Added scenarios:
- active account returns 200
- missing account returns 404
- unauthorized consumer returns 403

Run with:
mvn test -Dcucumber.filter.tags="@account-lookup"
```

---

## Do Not Do

Do not:

- duplicate existing step definitions
- use real sensitive data
- hardcode environment-specific URLs
- depend on test execution order
- use static mutable state for scenario data
- add sleeps instead of proper synchronization or stubbing
- compare full raw JSON strings unnecessarily
- assert unrelated fields
- introduce new libraries without checking the repo first
- rewrite the entire framework without being asked
- create feature files that simply mirror controller method names
- make Gherkin so technical that non-developers cannot understand it
- expose mainframe payloads, secrets, tokens, or PCI data in tests

---

## Default Behavior

Prefer a small, working, maintainable change.

Follow existing repository conventions.

Ask for clarification only when required.

If assumptions are needed, state them clearly.

When generating code, produce compilable Java with imports.

When generating feature files, produce readable Gherkin focused on API behavior.


# Cypress Testing

This repository contains automated tests for a sample application using Cypress. These tests cover various sections of the homepage, user journeys, and forms, organized using Cypress best practices.

## Installation

To install Cypress and set up your project, follow these steps:

1. Install Cypress as a dev dependency:
   ```bash
   npm install cypress --save-dev
   ```

2. Open Cypress:
   ```bash
   npx cypress open
   ```

## Directory Structure

- **cypress/e2e/home.cy.js**: Contains tests for the homepage.
- **cypress/support/commands.ts**: Custom Cypress commands are defined here.

## Writing Tests

### Describing Tests

Each test is grouped inside a `describe()` function that takes two arguments:
1. A string describing the tests.
2. A callback function containing the test cases.

Example:
```js
describe("home page", () => {
  it("the h1 contains the correct text", () => {
    cy.visit("http://localhost:3000");
    cy.get("h1").contains("Kitchen Sink");
  });
});
```

### Test Selectors

It's recommended to use `data-*` attributes for selecting elements to avoid CSS or JS changes affecting the tests:
```html
<button data-cy="submit">Submit</button>
```

You can select and interact with this element using:
```js
cy.get('[data-cy="submit"]').click();
```

Alternatively, use `cy.contains()` when you want the test to fail if the element content changes:
```js
cy.contains("Submit").click();
```

### Running Specific Tests

Use `.only()` to run specific tests:
```js
it.only("the features on the homepage are correct", () => {
  cy.visit("http://localhost:3000");
});
```

## Accessing Specific Elements

Use `eq()` to target a specific element from an array:
```js
cy.get("dt").eq(0).contains("4 Courses");
```

Regular expressions can be used for case-insensitive matching:
```js
cy.get("dt").eq(0).contains(/4 courses/i);
```

## Hooks

Use the `beforeEach()` hook to perform actions before each test:
```js
describe("home page", () => {
  beforeEach(() => {
    cy.visit("http://localhost:3000");
  });

  it("the h1 contains the correct text", () => {
    cy.get("h1").contains("Kitchen Sink");
  });
});
```

## Custom Cypress Commands

You can add custom commands in the `cypress/support/commands.ts` file. For example:
```js
Cypress.Commands.add("getByData", (selector) => {
  return cy.get(`[data-test='${selector}']`);
});
```

Use the custom command in your tests:
```js
cy.getByData("submit").click();
```

## Form Testing

To type text into input fields:
```js
cy.get("#email-address").type("hello@gmail.com");
```

To click on elements:
```js
cy.get(".submit-button").click();
```

Use `should()` to assert the state of an element:
```js
cy.get("#success-message").should("exist").contains("Submission Successful");
```

## Testing Multiple Pages

Use `context()` to group tests for different sections of the page:
```js
describe("home page", () => {
  context("hero section", () => {
    it("displays the correct heading", () => {
      cy.visit("http://localhost:3000");
      cy.get("#email-address").type("hello@gmail.com");
    });
  });
});
```

## Testing User Journeys

To test user navigation and journeys across pages:
```js
describe("User journey", () => {
  it("navigates through lessons", () => {
    cy.visit("http://localhost:3000");
    cy.get("[data-test='course-0']").find("a").eq(3).click();
    cy.location("pathname").should("eq", "/testing-your-first-application");

    cy.get('[data-test="next-lesson-button"]').click();
    cy.location("pathname").should(
      "equal",
      "/testing-your-first-application/app-install-and-overview"
    );
  });
});
```

## License

This project is licensed under the MIT License.

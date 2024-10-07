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

### Run cypress on terminal

```bash
npx cypress run --browser chrome
```

## Accessing Specific Elements

Use `eq()` to target a specific element from an array:

```js
cy.get("dt").eq(0).contains("4 Courses");
```

Regular expressions can be used for case-insensitive matching:

```js
cy.get("dt")
  .eq(0)
  .contains(/4 courses/i);
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

## (Arrange-Act-Assert) pattern

### Arrange

In step one, the Arrange step, you have to perform some setup for your test. For example, in the case of a Cypress end-to-end test, you need to tell Cypress to open the browser and navigate to the correct URL.

```javascript
cy.visit("http://localhost:8888");
```

### Act

In step two, the Act step, you perform some action. For example, in the case of a todo app, you want to test that you can add a single todo.

```javascript
cy.get(".new-todo").type("Buy Milk{enter}");
```

### Assert

Finally, in step three, you Assert. In this step, you assert that the thing you acted upon in step two did what you expected.

```javascript
cy.get(".todo-list li").should("have.length", 1);
```

## Return vs. Yield

Cypress commands DO NOT return their subjects. This means you cannot do things like this:

```javascript
// THIS WILL NOT WORK
const button = cy.get("button");
button.click();
```

This is one of the primary reasons why we do not recommend using variables within your tests.
.then() behaves similarly to Promises in JavaScript. However, .then() is a Cypress command, not a Promise. This means you cannot use things like async/await within your Cypress tests.

```javascript
cy.get("button").then(($btn) => {
  const cls = $btn.attr("class");
  // ...
});
```

## .wrap()

In our example just above, $btn is a jQuery object. This means that if we would like Cypress to perform some action upon it, we first need to use cy.wrap() for Cypress to interact with it.

```javascript
cy.get("button").then(($btn) => {
  const cls = $btn.attr("class");
  cy.wrap($btn).click().should("not.have.class", cls);
});
```

Before our assertion, which in this case is .should('not.have.class', cls) we first need Cypress to .click() the button. For Cypress to click on our $btn, we must first wrap it with cy.wrap() to provide the proper context for Cypress to perform the click.

To illustrate, we cannot do something like this, because $btn is a jQuery object.

```javascript
$btn.click().should("not.have.class", cls); // Does not work
```

## Aliases

In Cypress, we use Aliases to reference elements, requests, or intercepts across our tests. Basically means to rename.

```javascript
// Create an alias using the `as()` Cypress Command
cy.get("table").find("tr").as("rows");
// To use it
// Reference Cypress Alias `rows`
cy.get("@rows");
```

## Explicit Waits

Even though Cypress is incredibly smart in handling automatic waiting, there are times when you explicitly want to wait for something. For example, you may want to wait on a specific network request to finish before moving forward. The best way to handle these waits is to wait on aliases. Anything that can be aliased can be waited upon, like elements, intercepts, requests, etc.

```javascript
describe("User Sign-up and Login", () => {
  beforeEach(() => {
    cy.request("POST", "/users").as("signup"); // creating the signup alias
  });
  it("should allow a visitor to sign-up, login, and logout", () => {
    cy.visit("/");
    // ...
    cy.wait("@signup"); // waiting upon the signup alias
    // ...
  });
});
```

In this example, we are creating an alias called signup for the POST request to the /users endpoint. Then within our test, we tell Cypress to wait upon the signup alias to complete before continuing on with our test.

## Logging

There are two useful ways to log information from your tests. One way is to use cy.log() the other is to use console.log() . Remember Cypress is just JavaScript so you can leverage all of the helpful debugging techniques you use in JS too.

## Generating fake data for testing

Here is an example script for generating fake users with Faker.

```javascript
// generateSeedUsers.js
import path from "path";
import fs from "fs";
import shortid from "shortid";
import faker from "faker";
import bcrypt from "bcryptjs";
import { times } from "lodash";
const passwordHash = bcrypt.hashSync("s3cret", 10);
const createFakeUser = () => ({
  id: shortid(),
  firstName: faker.name.firstName(),
  lastName: faker.name.lastName(),
  username: faker.internet.userName(),
  password: passwordHash,
  email: faker.internet.email(),
  createdAt: faker.date.past(),
  modifiedAt: faker.date.recent(),
});
export const createSeedUsers = (numberOfUsers) =>
  times(numberOfUsers, () => createFakeUser());
export const saveUsersSeed = (numberOfUsers) => {
  const seedUsers = createSeedUsers(numberOfUsers);
  // write seed users to seedUsers.json
  fs.writeFile(path.join(process.cwd(), "seedUsers.json"), seedUsers);
};
```

## Intercepting Network Requests

```javascript
cy.intercept("POST", "/users");
```

### Waiting

```javascript
cy.intercept("POST", "/users").as("signup");
// ...
cy.wait("@signup");
```

### Overriding an existing intercept

```javascript
beforeEach(() => {
  // ...
  cy.intercept("GET", "/transactions/public*").as("publicTransactions");

  // ...
});
```

```javascript
cy.intercept("GET", "/transactions/public*", {
  // ...
  fixture: "public-transactions.json",
}).as("mockedPublicTransactions");
```

### Changing Headers

```javascript
cy.intercept("GET", "/transactions/public*", {
  headers: {
    "X-Powered-By": "Express",
    Date: new Date().toString(),
  },
});
```

### Modifying Response Data

```javascript
cy.intercept("POST", "/bankaccounts", (req) => {
  const { body } = req;
  req.continue((res) => {
    res.body.data.listBankAccount = [];
  });
});
```

In this example, we are overriding the listBankAccount property and setting it to be an empty array. We need to use .continue() to modify the response data.

### Inspecting a Request

```javascript
cy.intercept("POST", apiGraphQL, (req) => {
  const { body } = req;

  if (
    body.hasOwnProperty("operationName") &&
    body.operationName === "CreateBankAccount"
  ) {
    req.alias = "gqlCreateBankAccountMutation";
  }
});
```

## Example API Test

```javascript
// cypress/tests/api/api-users.spec.ts

context("GET /users", () => {
  it("gets a list of users", () => {
    cy.request("GET", "/users").then((response) => {
      expect(response.status).to.eq(200);
      expect(response.body.results).length.to.be.greaterThan(1);
    });
  });
});
```

## Building Cypress Commands

Allow you to create custom functionality and even overwrite existing commands. Having this flexibility is incredibly convenient and powerful,

```javascript
Cypress.Commands.add("loginByApi", (username, password) => {
  return cy.request("POST", `http://localhost:3000/login`, {
    username,
    password,
  });
});
```

```javascript
describe("POST /login", () => {
  it("login as user", () => {
    cy.loginByApi("jdoe", "password123").then((response) => {
      expect(response.status).to.eq(200);
    });
  });
});
```

## Run cypress on different browsers

```javascript
// Run the test if Cypress is running
// using the built-in Electron browser
it('has access to clipboard', { browser: 'electron' }, () => {
  ...
})
// Run the test if Cypress is run via Firefox
it('Download extension in Firefox', { browser: 'firefox' }, () => {
  // ...
})
// Run the test if Cypress is run via Chrome
it('Show warning outside Chrome', { browser: 'chrome' }, () => {
  // ...
})
```

## Cypress Methods You Need to Know

### .its()

.its() is a handy method when you want to get the property off of something. For example if you want to make an assertion against an array.

```js
cy.wrap(["Wai Yan", "Yu"]).its(1).should("eq", "Yu"); // true
cy.wrap({ age: 52 }).its("age").should("eq", 52); // true
```

### .invoke()

In the example below, we are invoking the Array.slice() function on the response data returned from an aliased intercept.

```js
beforeEach(() => {
  cy.intercept("GET", "/users").as("users");
});
it("slices the users returned from /users endpoint", () => {
  cy.wait("@users").its("response.body.results").invoke("slice", 0, 5);
  // ...
});
```

### .request()

.request() is a helpful method anytime you need to make an HTTP request within your tests and perform expectations against it.

```js
cy.request("POST", "http://localhost:8888/users/admin", { name: "Jane" }).then(
  (response) => {
    // response.body is automatically serialized into JSON
    expect(response.body).to.have.property("name", "Jane"); // true
  }
);
```

### .within()

When you are working with elements, you need to drill down into its children, grandchildren, etc. You can use .within() to limit the scope of Cypress commands to within a specific element. For example.

```js
it("ensures the section lesson exists", () => {
  cy.getBySel("section-steps").within(() => {
    cy.getBySel("lesson-complete-0").should("exist");
  });
});
```

## Using Data to Build Dynamic Tests

```js
cy.get("username").type("jdoe");
cy.get("password").type("password123");
```

```js
const users = [
  {
    username: "John",
    password: "password123",
  },
  {
    username: "Jane",
    password: "password123",
  },
];
cy.get("username").type(users[0].username);
cy.get("password").type(users[0].password);
```

```js
cy.fixture("users.json").as("usersData");
cy.get("username").type(usersData[0].username);
cy.get("password").type(usersData[0].password);
```

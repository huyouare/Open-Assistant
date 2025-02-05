# Component and e2e testing with Cypress

[Cypress](https://www.cypress.io/) is used for both component- and end-to-end
testing. Below there's a few examples for the context of this site. To learn
more, the
[Cypress documentation](https://docs.cypress.io/guides/getting-started/opening-the-app)
has it all.

Don't get scared by the commercial offerings they offer. Their core is open
source, the cloud offering is not necesarry at all and can be replaced by CI
tooling and [community efforts](https://sorry-cypress.dev/).

# Component testing

To write a new component test, you either create a new `.tsx` adjacent to the
component you want to test or you can use the guide presented yo you when
running `npm run cypress` which allows you to easily create the skeleton test
for an existing component.

If you have a `Button.tsx` component, create a file next to it called
`Button.cy.tsx` which could look like this:

```typescript
import React from "react";
import { Button } from "./Button";

describe("<Button />", () => {
  it("renders", () => {
    // see: https://on.cypress.io/mounting-react
    cy.mount(<Button className="border-gray-800 m-5">Test button</Button>);
    cy.get("button").compareSnapshot("button-element");
  });
});
```

## What's happening here?

First we use `cy.mount` to mount our component under test. Notive how we specify
`className` and inner text - this is where we arrange our component with fake
data that we could assert on later.

In the example above, we also use `cy.get` to select the rendered `button`
element. Cypress has multiple ways to
[select elements](https://docs.cypress.io/guides/references/best-practices),
`get` is just one of them (and often not recommended).

At last, we use `captureSnapshot` which is a plugin that snaps a photo of the
`button` element and compares it to a baseline located in the
`./cypress-visual-screenshots/baseline/` folder. If there's too many unidentical
pixels between the two, it will fail the test.

# End-to-end (e2e) testing

e2e tests are stored in the `./cypress/e2e` folder and should be named
`{page}.cy.ts` and located in a relative folder structure that mirrors the page
under test.

When running `npm run cypress` and selecting e2e testing, we assume you have the
NextJS site running at `localhost:3000`.

An example test from this time of writing, could look as follows:

```typescript
describe("signin flow", () => {
  it("redirects to a confirmation page on submit of valid email address", () => {
    cy.visit("/auth/signin");
    cy.get(".chakra-input").type(`test@example.com{enter}`);
    cy.url().should("contain", "/auth/verify");
  });
});

export {};
```

## What's happening here?

First we use [`cy.visit`](https://docs.cypress.io/api/commands/visit) to point
the browser at the desired page. It appends relative paths to the configured
`baseUrl` (found in `./cypress.config.ts`).

Cypress will
[automatically await](https://docs.cypress.io/guides/core-concepts/introduction-to-cypress#Timeouts)
almost anything you do, but fail if the default timeout is reached.

Then we get the email input field and type our email address. Notice the
`{enter}` keyword, this will cause Cypress to hit the return key which we expect
to submit the form.

We then assert that the URL should contain `/auth/verify`. Again the timeout
will make sure we are not waiting forever, and the test will fail if we do not
manage to get there in a reasonable time.

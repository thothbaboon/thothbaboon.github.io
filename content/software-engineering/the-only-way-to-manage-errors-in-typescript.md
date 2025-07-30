+++
title = "The Only Way to Manage Errors in Typescript"
date = 2023-02-14
description = "Learn about the best approach to handling errors in TypeScript. Discover the Outcome Pattern and Switch Guard."
+++

What's wrong with this code ?

```js
try {
  // do something
} catch (error) {
  // handle error here
}
```

The problem with this code is that in TypeScript, the caught error is
not typed, and although it can be cast, it is not a safe practice
since anything can be thrown, not only an error.

`throw(1)` works for instance

- When we throw errors, it becomes difficult to determine what we are catching since `error` can be anything.
- Additionally, it becomes challenging to identify which errors we need to handle, unless we search through all the function calls, which can be time-consuming and complex.
- If a new error is added and not handled, the code will still compile. It should not.

It is important to distinguish between unexpected errors, which should
be thrown, and expected errors, such as business errors, which should
not be thrown.

There are better ways to handle expected errors, which we will
explore.

## The Outcome Pattern

We are writing a method to allow users to purchase items in an
e-commerce application. The method verifies that the item being
purchased exists and is also available for purchase. There are four
potential outcomes:

- The item is not ordered because this ID does not match an item
- The item is not ordered because it is out of stock.
- The item is not ordered because the selected color does not exist.
- The item is successfully ordered.

Thus, the method's result can be expressed as a union of these 4
possible outcomes.

```ts
type OrderItemResult =
  | {
      outcome: "notOrdered";
      reason: "itemNotFound";
    }
  | {
      outcome: "notOrdered";
      reason: "outOfStock";
    }
  | {
      outcome: "notOrdered";
      reason: "colorDoesNotExist";
      availableColors: string[];
    }
  | {
      outcome: "ordered";
    };

const orderItem = async (itemId: string): Promise<OrderItemResult> => {
  // ...
};
```

- The verb used in the method’s name is `order`, so the potential outcomes should be **notOrdered** or **ordered**.
- Avoid using generic terms like **success** or **error** and instead use specific terminology. For example, instead of **success**, use **ordered** as it provides better clarity about what has succeeded.
- When necessary, add context to provide additional information that may be useful for debugging or displaying a clear error message in the app, as it’s done for `availableColors`.

## The Outcome Pattern on Steroids - Switch Guard

The switch guard is a robust structure that helps ensure that all
potential cases are handled. Basically, It is a switch statement with
a guard included in the default case. The guard is here to prevent the
code from compiling if a case is left unhandled.

```ts
const throwUnhandledOutcome = (result: never) => {
	throw new Error("Not handled case: " + JSON.stringify(result));
};

const someMethodInOurCode = async () => {

	/* ... */

	const result: OrderItemResult = await orderItem(itemId);

  if (result.outcome === "notOrdered") {
    switch (result.reason) {
      case "itemNotFound":
        return console.error("Item not found in catalog");
      case "outOfStock":
        return console.error("Item is out of stock");
      case "colorDoesNotExist":
        return console.error(
          \`Selected color is not available. Available colors: \$\{result.availableColors.join(
            ", "
          )}\`
        );
      default:
        throwUnhandledOutcome(result);
    }
  }
	/* here, handle the happy path */
	/* ... */
};
```

The function `throwUnhandledOutcome` has a parameter of type
`never`. This means that during the build process, if TypeScript
detects a case that ultimately falls into the default handler, the
build will fail.

If we remove the `outOfStock` case, we will receive the following
error message:

```ts
\`Argument of type '{ outcome: "notOrdered"; reason: "outOfStock"; }' is not assignable to parameter of type 'never'\`
```

If the result of the `orderItem` method change, such as the
addition of a new outcome, Typescript will ask us to handle the new
outcome. If we don’t, the code will not build.

This is the best way to make sure business errors are properly handled
everywhere in our code.

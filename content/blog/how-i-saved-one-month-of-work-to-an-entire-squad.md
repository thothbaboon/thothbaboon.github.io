+++
title = "How I Saved One Month of Work to an Entire Squad"
description = "How we use the adapter pattern to simplify the first version of a feature by keeping the logic of the long term vision."
date = "2023-11-7"
+++

## Introduction to the Feature We Were Working On

In every company, finance teams define **budgets** once a year, following a **budgetary exercise**.

The goal was to model the budgetary exercise in our software, so the companies using our software could

- Link their spendings (that we collected on our platform) to budgets
- See the budgets consumptions in real time, and see if they spent as planned (or not)

Here is how a simple annual budget could look like:

![Relation](/how-i-saved-one-month-of-work-to-an-entire-squad/exercice.png)

In this example, the budgets correspond to different teams in the company, and each budget amount is splitted into **expense categories**.

Here are the simplified entities for this feature.

```ts
type BudgetaryExercise = {
	id: string;
	startDate: string;
	endDate: string;
	budgets: Budget[];
};

type Budget = {
	id: string;
	budgetaryExerciseId: string;
	amountByExpenseCategory: Map<ExpenseCategoryId, MonetaryValue>;
};
```

## Proof of Concept and First Version

We started by implementing a PoC, with the entities described above.

The PoC was approved buy our test users, so we decided to implemented a first version for production.

The idea was to start with a simplified version of this feature, with no split by expense categories, and add this split later.

There was no defined timeline — could have been in 1 month or 1 year.

## The Problem

The feature was a bit more complex than explained.

One important missing concept here is **budget breakdown.**

A **breakdown** is the result of a computation made on a budget of the following amounts:

* Used (payments)
* Committed (amount “locked” for planned payments)
* Available
* Used exceeded
* Committed exceeded

A breakdown can be by expense categories.

In the PoC, we’ve already built a big part of the logic of the feature, with budget splitted by expense categories.

The idea was to rewrite everything without this split, knowing that we would have to add it back some day.

**This felt wrong to me**. It would not be easy to add the split later.

* It would introduce breaking changes.
* The feature might have evolved since then, so it may not simply be a copy/paste from the PoC.
* There would be data in production to migrate to the new model, and here we are talking about money and analytics that drives the companies — we can’t make mistake here.
* The unit tests would have to be rewritten — we might break something without knowing it.

## A Possibly Good Idea 💡

My idea was to keep the logic with the split by expense categories but to expose a simplified version from the API.

Doing this would allow the front end developers to build the first version while keeping the split by expense categories unchanged on the backend.

It’s actually easy to fake the “no split” using the “split” logic and data model.

Instead of:

```ts
const split = new Map([
	["Freelance and experts", 200],
	["Tools and subscriptions", 800],
	["Server and COGS tools", 0],
]);
```

We can do:

```ts
const split = new Map([["default", 1000]]);
```

Which means there is still a split, but only in a “fake” expense category called **default**.

### SimplifiedBudget

Now instead of the `Budget` entity previously introduced, we can have this `SimplifiedBudget` with a single amount instead of a `Map` to store the amount for each expense category.

```ts
type SimplifiedBudget = {
	id: string;
	budgetaryExerciseId: string;
	amount: MonetaryValue;
};
```

### From SimplifiedBudget to Budget

`SimplifiedBudget` can be easily converted to `Budget` using the **default** expense category.

```ts
amountByExpenseCategory = new Map([["default", simplifiedBudget.amount]]);
```

### From Budget to SimplifiedBudget

Simply take the amount of the default expense category.

```ts
amount = amountByExpenseCategory.get("default");
```

## A Perfect Use Case for the Adapter Pattern

Instead of having the `BudgetController` calling directly the `BudgetService` (that contains all the logic with split by expense category)

![Relation](/how-i-saved-one-month-of-work-to-an-entire-squad/plug.png)

There would be an adapter in between that would do the conversion from a `SimplifiedBudget` to a `Budget`, and from a `Budget` to a `SimplifiedBudget`.

![Relation](/how-i-saved-one-month-of-work-to-an-entire-squad/adapter.png)

The `BudgetController` calls the `BudgetService`.

But the `SimplifiedBudgetController` calls the adapter (`SimplifiedBudgetService`) that contains the logic to convert a `SimplifiedBudget` to a `Budget`, and the adapter then call the `BudgetService`.

- `SimplifiedBudgetController` doesn’t know what is a `Budget` or `BudgetService`.
- `BudgetService` doesn’t know about `SimplifiedBudget`
- `BudgetController` is usable in parallel, to use `Budget` with the split by expense category (already ready for the second version of this feature)

![Relation](/how-i-saved-one-month-of-work-to-an-entire-squad/dependencies.png)

In the code, `SimplifedBudgetService` has a similar interface to `BudgetService`, but using `SimplifiedBudget` instead of `Budget`.

```tsx
export interface BudgetService {
	findById: (id: string) => Promise<Budget | undefined>;
	deleteById: (id: string) => Promise<DeleteBudgetByIdResult>;
	create: (dto: BudgetDto) => Promise<CreateBudgetResult>;
}

export interface SimplifiedBudgetService {
	findById: (id: string) => Promise<SimplifiedBudget | undefined>;
	create: (dto: SimplifiedBudgetDto) => Promise<CreateSimplifiedBudgetResult>;
}
```

## Benefits

* It keeps the Budget entity and logic unchanged, but provides a SimplifiedBudget for a simplified version of the feature.
* It was super light and quick to implement, the adapter code is very simple. Introducing the adapter pattern was quicker than removing and retesting the logic for the first version.
* No data migration. A data migration is always complicated and stressful, whereas the adapter here is easily testable.
* No changes needed on the backend to introduce the split by expense categories in the product for the second version of the feature.

I switched to another team after the first version and the adapter.

The next year I heard from the team that they released the split by expense categories, and that this adapter made it extremely simple. They just had to made the changes in the front end to handle expense categories, and call the routes of `BudgetController` rather than `SimplifiedBudgetController`.

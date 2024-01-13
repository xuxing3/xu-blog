---
title: "Mongodb Query"
date: 2024-01-13T11:40:49+08:00
tags:
  - Mongodb
draft: false
---

Record some Mongodb query example.

## Match by a Single Condition

```
{ name: "Andrea Le" }
```

## Match by Multiple Conditions ($and)

```
{ scores: 75, name: "Greg Powell" }

{$and: [{ status: { $ne: 'available' }},{ status: { $ne: 'XR' }}]}
```

## Match by Multiple Possible Conditions ($or)

```
{ $or: [ { version: 4 }, { name: "Andrea Le" } ] }
```

## Match by Exclusion ($not)

```
{ name: { $not: { $eq: "Andrea Le" } } }
```

## Match with Comparison Operators

```
{ version: { $lte: 4 } }
```

## Match by Date

```
{ dateCreated: { $gt: new Date('2000-06-22') } }
```

## Match by Array Conditions

```
{ scores: { $elemMatch: { $gt: 80, $lt: 90 } } }
```

> [Query Your Data](https://www.mongodb.com/docs/compass/current/query/filter/?utm_source=compass&utm_medium=product#examples)

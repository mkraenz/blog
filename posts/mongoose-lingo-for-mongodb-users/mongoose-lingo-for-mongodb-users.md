---
title: Mongoose lingo for MongoDB users
description: TODO
tags: "mongodb, javascript, typescript, database"
cover_image: "https://res.cloudinary.com/practicaldev/image/fetch/s--kKx_qBMC--/c_imagga_scale,f_auto,fl_progressive,h_420,q_auto,w_1000/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/0ztdklh7uow70ydussr1.png"
published: false
id: 696525
---

You feel comfortable with MongoDB and want to add structure, validation, and documentation to your database layer? [Mongoose](https://mongoosejs.com/) to the rescue! It does come, however, with it's own technical terms.

Let's explore these terms and their relation to MongoDB concepts to get you jump-started with Mongoose.

## tl;dr

Checkout the mindmap on the top.

## MongoDB basics

Remind that MongoDB has essentially three parts - databases, collections and documents.

_MongoDB Documents_ can be thought of simple JSONs, i.e. containers of key-value pairs. Values itself may be primitives or containers again.

_Collections_ are, well, collections of documents. They provide functionality to insert, retrieve, update, and delete documents.

_Databases_ (or logical databases) are top-level namespaces that contain collections.

## Schemas

The Mongoose _schema_ is the _structure_ that you put on top of your MongoDB documents in a specific collection. The schema defines what properties a MongoDB document has, the type of each property, uniqueness, indexing, and validation.

Typically this is the first thing to define for a new database entity.

```typescript
import mongoose from "mongoose";

const teaSchema = new mongoose.Schema({
  taste: String,
  color: { type: String, default: "brown" },
  brewingTemperature: { type: Number, min: 0, max: 100 },
});
```

Note that these are not TypeScript types! These are JavaScript types which are recognizable by the capital letter in `String` and `Number`.

### Schema Properties

Mongoose _schema properties_ are the keys in the object provided to the schema constructor.

In the above example, we have three properties - `taste`, `color`, `brewingTemperature`. These are further specified by their values - the SchemaTypes.

### SchemaTypes

> A mongoose _SchemaType_ is the configurations for an individual schema property. A SchemaType specify the schema property's type, validation, indexing, optionality, default values, uniqueness, and more.

Sometimes SchemaType might only refer to the datatype in the configuration object, and sometimes to the whole configuration.

In our above example, the most complex SchemaType is the one of `brewingTemperature` which defines it's type - `brewingTemperature` is a number - and also some validation in terms of minimum is 0 degree Celsius, and the maximum temperature 100 degree Celcius.

```typescript
  brewingTemperature: { type: Number, min: 0, max: 100 },
```

## Models

Mongoose _models_ are the mongoose abstraction of MongoDB collections. Just as collections, they provide functionality to insert, update, retrieve and delete documents. However, they also allow for mongoose specific functionality like enforcing validation before saving.

```typescript
const TeaModel = mongoose.model("Tea", teaSchema);
const peppermintTea = await TeaModel.findOne({ taste: "Peppermint" });
```

This connects the collection `teas` with the `teaSchema`. Mongoose automatically pluralizes and lowercases the provided collection name `'Tea'`. We then use the model to retrieve a cup of peppermint tea :tea:. The found `peppermintTea` is a mongoose document (or null).

## Documents

Mongoose _documents_ are the instances of the model. They are linked to their serialized MongoDB documents.

```typescript
const greenTea = new TeaModel();
greenTea.taste = "fruity";
greenTea.brewingTemperature = 90;
await greenTea.save();
```

This creates a new mongoose tea document, which automatically fulfils the underlying `teaSchema`. We can be sure that when we `save()` the green tea to the database it will contain only the properties we specified and default values, and only within the restrictions.

## Extra: Schema Paths

One term that confused me for at first is _Schema Paths_. At first it appears these are just Schema properties. In fact, they are identical when it comes to simple schemas without nesting.

With nested schemas however, the distinction becomes clear. if you think of the schema as a tree datastructure, then a path is how you get from the root of the tree to some leaf. Let's clarify this with an example.

```typescript
const teaSchema = new mongoose.Schema({
  taste: String,
  producer: {
    name: String,
    sellerId: Number,
  },
});
```

We have a simplified version of our `teaSchema` from before but this time it contains a nested `producer` property which is itself an object with further properties.

The leafs of this schema are `taste`, `name`, and `sellerId`. The corresponding paths are `taste`, `producer.name`, and `producer.sellerId`. Since `producer` is _not_ a leaf of the tree there is no _schema path_ for the `producer` even though it is still a _schema property_.

Thanks for your time!

## Recommended Reading

- [Official Mongoose Schema Docs](https://mongoosejs.com/docs/guide.html)
- [Typegoose - Mongoose via TypeScript classes](https://typegoose.github.io/typegoose/)

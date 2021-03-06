---
title: Field Resolvers
---

Although Vulcan uses your SimpleSchema JSON schema to generate your GraphQL schema, it's important to understand that the two can be very different. 

Note: this section will make more sense if you've already read the [Resolvers](/resolvers.html) section.

## Overview

For example, you might have a `userId` property in your `Movie` schema, but want this property to *resolve* as a `user` object in your GraphQL schema (and in turn, in your client store). You can use `resolveAs` to accomplish this:

```
userId: {
  type: String,
  optional: true,
  viewableBy: ['guests'],
  resolveAs: {
    fieldName: 'user',
    arguments: 'foo: Int = 10',
    type: 'User',
    resolver: (movie, args, context) => {
      return context.Users.findOne({ _id: movie.userId }, { fields: context.Users.getViewableFields(context.currentUser, context.Users) });
    },
    addOriginalField: true
  }
},
```

We are doing five things here:

1. Specifying that the field should be named `user` in the API.
2. Specifying that the field can take a `foo` argument that defaults to `10`.
3. Specifying that the `user` field returns an object of GraphQL type `User`.
4. Defining a `resolver` function that indicates how to retrieve that object.
5. Specifying that the original field (`userId`, with type `String`) should also be added to our GraphQL schema. 

## Custom Types

Creating a collection with `createCollection` will automatically create the associated GraphQL type, but in some case you might want to resolve a field to a GraphQL type that doesn't correspond to any existing collection. 

Here's how the `vulcan:voting` package defines a new `Vote` GraphQL type:

```
const voteSchema = `
  type Vote {
    itemId: String
    power: Float
    votedAt: String
  }
  
  union Votable = Post | Comment
`;

addGraphQLSchema(voteSchema);
```

Which can then be used as the value for `type` in a `resolveAs` field. 

## GraphQL-Only Fields

Note that a field doesn't have to “physically” exist in your database to be able to generate a corresponding GraphQL field. 

For example, you might want to query the Facebook API to get the number of likes associated with a post's URL:

```
likesNumber: {
  type: Number,
  optional: true,
  viewableBy: ['guests'],
  resolveAs: {
    fieldName: 'likesNumber',
    type: 'Number',
    resolver: async (post, args, context) => {
      return await getFacebookLikes(post.url);
    }
  }
},
```

This will create a `likesNumber` field in your GraphQL schema (and your Apollo store) even though no such field exist in your database. 

## Legacy String Syntax (deprecated)

Alternatively, you can also use the legacy syntax and specify a string value for `resolveAs` (in this case, `user: User`). You'll then need to use `addGraphQLResolvers` to define the resolver object yourself:

```js
import { addGraphQLResolvers } from 'meteor/vulcan:core';

const movieResolver = {
  Movie: {
    user(movie, args, context) {
      return context.Users.findOne({ _id: movie.userId }, { fields: context.getViewableFields(context.currentUser, context.Users) });
    },
  },
};
addGraphQLResolvers(movieResolver);
```

Resolvers can be defined on any new or existing type (e.g. `Movie`).

# GraphQL

> **Module:** Frontend System Design  
> **Chapter:** 04 — GraphQL

---

## Table of Contents
1. [What is GraphQL?](#1-what-is-graphql)
2. [Why GraphQL? — Benefits](#2-why-graphql--benefits)
3. [REST vs GraphQL](#3-rest-vs-graphql)
4. [Building Blocks of GraphQL](#4-building-blocks-of-graphql)
5. [Schema & Types](#5-schema--types)
6. [Query](#6-query)
7. [Mutation](#7-mutation)
8. [Subscription](#8-subscription)
9. [Resolver](#9-resolver)
10. [Fundamentals — How it All Connects](#10-fundamentals--how-it-all-connects)
11. [Over-fetching & Under-fetching Problem](#11-over-fetching--under-fetching-problem)
12. [Frontend Engineer Cheatsheet](#12-frontend-engineer-cheatsheet)

---

## 1. What is GraphQL?

**GraphQL** = **Graph Query Language**

GraphQL is a **query language for APIs** and a runtime for executing those queries. It was developed by Facebook in 2012 and open-sourced in 2015. Many major companies are adopting it — GitHub, Shopify, Twitter, Netflix.

> GraphQL is **not** a database query language. It's a specification for how clients and servers communicate — an alternative to REST.

### The Core Idea

With REST, the server decides what data to return.  
With GraphQL, **the client decides exactly what data it needs.**

```
REST — server decides:
GET /api/users/1
→ { id, name, email, phone, address, createdAt, role, ... }
   (you get everything, want it or not)

GraphQL — client decides:
query {
  user(id: 1) {
    name
    email
  }
}
→ { name, email }
   (you get exactly what you asked for, nothing more)
```

---

## 2. Why GraphQL? — Benefits

| # | Benefit | What it means |
|---|---|---|
| 1 | **Avoid Over-fetching** | Get only the fields you request — no wasted data |
| 2 | **Avoid Under-fetching** | Get all related data in one request — no multiple API calls |
| 3 | **Better Mobile Performance** | Less data over the wire = faster on slow networks |
| 4 | **Efficiency & Precision** | Declarative data fetching — ask for exactly what you need |
| 5 | **Structured/Hierarchical** | Response shape mirrors query shape — predictable |
| 6 | **Strongly Typed** | Schema acts like TypeScript for your API — catches errors early |
| 7 | **Introspection** | Clients can query the schema itself — self-documenting API |
| 8 | **Real-time Capabilities** | Built-in Subscriptions for live data |
| 9 | **Single Endpoint** | One URL for everything — no endpoint sprawl |
| 10 | **Rapidly Growing** | Huge ecosystem, strong community adoption |

---

## 3. REST vs GraphQL

### The Problem GraphQL Solves — A Real Example

Let's say you're building a screen that shows: **continents → countries → languages**.

**With REST:**
```
GET /api/continents          → get all continents
GET /api/continents/1/countries   → get countries for continent 1
GET /api/countries/45/languages   → get languages for country 45
```
3 separate API calls. Waterfalled. Each waits for the previous.

**With GraphQL:**
```graphql
query {
  continents {
    name
    countries {
      name
      code
      languages {
        name
        code
      }
    }
  }
}
```
1 request. Everything in one shot. Server merges it all.

---

### Full Comparison Table

| Aspect | REST | GraphQL |
|---|---|---|
| **Endpoints** | Multiple endpoints (`/users`, `/posts`, `/comments`) | Single endpoint (`/graphql`) |
| **Request structure** | Fixed structure + HTTP methods | Flexible — query/mutation |
| **Over-fetching** | Common issue | Solved — request only needed fields |
| **Under-fetching** | Common — multiple calls needed | Solved — nested queries |
| **Response size** | Fixed size — server decides | Flexible — client decides |
| **Versioning** | Explicit (`/v1/`, `/v2/`) | Not needed — evolve schema with deprecation |
| **Schema definition** | Not well defined (OpenAPI optional) | Explicit, typed schema required |
| **Real-time** | Polling or WebSocket (manual) | Built-in Subscriptions |
| **Tooling support** | Postman | GraphQL Playground / GraphiQL |
| **Caching** | HTTP cache works out of the box | No HTTP cache — needs client-side caching (Apollo) |
| **Client control** | No — server decides response | Yes — client decides exactly what comes back |
| **Adoption** | Widely adopted, mature | Rapidly growing, adopted by large companies |
| **Learning curve** | Low | Medium — schema, resolvers, types |

---

## 4. Building Blocks of GraphQL

```
              ┌──────────────────────┐
              │   GraphQL API        │
              └──────────┬───────────┘
         ┌───────────────┼──────────────┐
         ▼               ▼              ▼
    Schema/Type       Query /        Resolver
                      Mutation /
                      Subscription
```

**Three core building blocks:**
1. **Schema / Type** — defines the shape and types of your data
2. **Query / Mutation / Subscription** — operations the client can perform
3. **Resolver** — functions that fetch and return the actual data

---

## 5. Schema & Types

The **Schema** is the backbone of GraphQL. Written in **SDL (Schema Definition Language)** — GraphQL's own schema definition language, similar to how we do TypeScript.

### Types — Two Kinds

```
Type
├── Pre-defined (Scalar types built into GraphQL)
│     ID, String, Int, Float, Boolean
│
└── User-defined (Custom types you create)
      type Country { ... }
      type Continent { ... }
      type Language { ... }
```

### Schema Example

```graphql
# SDL — GraphQL Schema Definition Language

type Country {
  id: ID
  name: String
  code: String
  phone: String
  currency: String
  continent: Continent
  languages: [Language]   # array of Language objects
}

type Continent {
  name: String
  code: String
  countries: [Country]
}

type Language {
  name: String
  code: String
}
```

> Notice how this looks like TypeScript interfaces — that's intentional. The schema is the **contract** between client and server, just like TypeScript types are the contract in your codebase.

### Scalar Types

| Type | Description | Example |
|---|---|---|
| `ID` | Unique identifier | `"1"`, `"abc-123"` |
| `String` | UTF-8 text | `"Deepak"` |
| `Int` | 32-bit integer | `25`, `100` |
| `Float` | Decimal number | `3.14`, `99.9` |
| `Boolean` | True or false | `true`, `false` |

### Non-Null and List Modifiers

```graphql
name: String      # nullable — can be null
name: String!     # non-null — must have a value
countries: [Country]   # list — array of Country
countries: [Country!]! # non-null list of non-null items
```

---

## 6. Query

A **Query** is the GraphQL equivalent of GET — used to **read/fetch data**.

> All GraphQL queries use **HTTP POST** under the hood — even reads.

### Query Syntax

```graphql
# Basic query
query {
  countries {
    name
    code
  }
}

# Query with arguments
query {
  country(code: "IN") {
    name
    currency
    languages {
      name
    }
  }
}

# Named query (best practice)
query GetCountryDetails {
  country(code: "IN") {
    name
    currency
  }
}
```

### Response Shape Mirrors Query Shape

```graphql
# Query
query {
  continents {
    name
    countries {
      name
      code
    }
  }
}

# Response — same shape as query
{
  "data": {
    "continents": [
      {
        "name": "Asia",
        "countries": [
          { "name": "India", "code": "IN" },
          { "name": "Japan", "code": "JP" }
        ]
      }
    ]
  }
}
```

### Query Variables

```graphql
# Query definition with variable
query GetCountry($code: String!) {
  country(code: $code) {
    name
    currency
  }
}

# Variables passed separately
{
  "code": "IN"
}
```

---

## 7. Mutation

A **Mutation** is used to **create, update, or delete data** — equivalent to POST, PUT, PATCH, DELETE in REST.

```graphql
# Schema definition
type Mutation {
  updateLanguage(id: ID): Language
}

# Calling a mutation
mutation {
  updateLanguage(id: "1") {
    name
    code
  }
}

# Mutation with variables
mutation UpdateLanguage($id: ID!, $name: String!) {
  updateLanguage(id: $id, name: $name) {
    id
    name
    code
  }
}
```

### Mutation Response

Mutations return the updated/created object so the client can update its local cache without a refetch.

```json
{
  "data": {
    "updateLanguage": {
      "id": "1",
      "name": "Hindi",
      "code": "HI"
    }
  }
}
```

---

## 8. Subscription

A **Subscription** is GraphQL's built-in real-time capability — like WebSocket but defined in the schema.

```graphql
# Schema
type Subscription {
  messageAdded(chatId: ID!): Message
}

# Client subscribes
subscription {
  messageAdded(chatId: "room-1") {
    id
    text
    sender {
      name
    }
  }
}
```

- Client subscribes once → server **pushes updates** whenever data changes
- Uses WebSocket under the hood
- No need to set up WebSocket manually — GraphQL handles it

### Use Cases
- Live chat
- Real-time notifications
- Live stock prices / sports scores
- Collaborative editing

---

## 9. Resolver

A **Resolver** is a function on the server that actually **fetches the data** for a field.

Every field in a GraphQL schema has a corresponding resolver function.

```
Query { countries }
        ↓
Resolver for "countries"
        ↓
Fetches from DB / external API
        ↓
Returns data
```

### Resolver Signature

```js
// (parent, args, context) => data
Query: {
  countries: (parent, args, context) => {
    return db.getAllCountries();
  },

  country: (parent, args, context) => {
    return db.getCountryByCode(args.code);
  }
}
```

- **parent** — the result from the parent resolver (for nested fields)
- **args** — arguments passed in the query (`id`, `code`, etc.)
- **context** — shared data like auth user, DB connection, request headers

### Nested Resolvers

```js
// When query asks for country.languages,
// GraphQL calls the Language resolver with country as parent

Country: {
  languages: (parent, args, context) => {
    return db.getLanguagesForCountry(parent.id);
  }
}
```

---

## 10. Fundamentals — How it All Connects

```
Consumer (Client / React App)
        │
        │  uses
        ▼
GraphQL Client Library        ← Apollo Client, urql, or plain fetch
        │
        │  sends HTTP POST to /graphql
        ▼
GraphQL Server
        │
        ├── Parses query
        ├── Validates against Schema
        └── Calls Resolvers
                │
                ▼
           Executor (Server)
                │
                ├── DB queries
                ├── External API calls
                └── Business logic
                        │
                        ▼
                   Returns data shaped
                   exactly like the query
```

### GraphQL Server Libraries
- **Node.js:** Apollo Server, GraphQL Yoga, Mercurius (Fastify)
- **Python:** Strawberry, Graphene
- **Java:** graphql-java
- **Go:** gqlgen

### GraphQL Client Libraries
- **Apollo Client** — most popular, full-featured, built-in caching
- **urql** — lightweight alternative
- **React Query + fetch** — simple, manual approach
- **Relay** — Facebook's official client, very opinionated

---

## 11. Over-fetching & Under-fetching Problem

These are the two core problems GraphQL was built to solve.

### Over-fetching — Getting Too Much Data

```
REST: GET /api/users/1
Response:
{
  "id": 1,
  "name": "Deepak",
  "email": "deepak@email.com",
  "phone": "8959228019",
  "address": { ... },
  "role": "admin",
  "createdAt": "2021-01-01",
  "lastLogin": "2024-01-15",
  "preferences": { ... }
}

But your UI only needed: name and email.
The rest is wasted bandwidth.
```

### Under-fetching — Not Getting Enough Data

```
REST: You need to show a user's posts with comments.

Request 1: GET /api/users/1          → get user
Request 2: GET /api/users/1/posts    → get their posts
Request 3: GET /api/posts/1/comments → get comments for post 1
Request 4: GET /api/posts/2/comments → get comments for post 2

4 requests. Each waits for the previous. Slow on mobile.
```

### GraphQL Solution

```graphql
# One request. Exactly what you need.
query {
  user(id: 1) {
    name
    email
    posts {
      title
      comments {
        text
        author {
          name
        }
      }
    }
  }
}
```

---

## 12. Frontend Engineer Cheatsheet

### Setting Up Apollo Client in React

```js
// index.js
import { ApolloClient, InMemoryCache, ApolloProvider } from '@apollo/client';

const client = new ApolloClient({
  uri: 'https://api.example.com/graphql',
  cache: new InMemoryCache(),
  headers: {
    authorization: `Bearer ${token}`,
  },
});

// Wrap your app
<ApolloProvider client={client}>
  <App />
</ApolloProvider>
```

### useQuery Hook

```js
import { useQuery, gql } from '@apollo/client';

const GET_COUNTRIES = gql`
  query GetCountries {
    countries {
      name
      code
      emoji
    }
  }
`;

const Countries = () => {
  const { loading, error, data } = useQuery(GET_COUNTRIES);

  if (loading) return <Spinner />;
  if (error) return <Error message={error.message} />;

  return data.countries.map(c => <div key={c.code}>{c.name}</div>);
};
```

### useMutation Hook

```js
import { useMutation, gql } from '@apollo/client';

const UPDATE_LANGUAGE = gql`
  mutation UpdateLanguage($id: ID!, $name: String!) {
    updateLanguage(id: $id, name: $name) {
      id
      name
    }
  }
`;

const LanguageEditor = () => {
  const [updateLanguage, { loading, error }] = useMutation(UPDATE_LANGUAGE);

  const handleSave = () => {
    updateLanguage({
      variables: { id: '1', name: 'Hindi' },
    });
  };
};
```

### Plain Fetch (No Library)

```js
const fetchGraphQL = async (query, variables = {}) => {
  const response = await fetch('https://api.example.com/graphql', {
    method: 'POST',                      // Always POST
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${token}`,
    },
    body: JSON.stringify({ query, variables }),
  });

  const { data, errors } = await response.json();
  if (errors) throw new Error(errors[0].message);
  return data;
};

// Usage
const data = await fetchGraphQL(`
  query { countries { name code } }
`);
```

### When to Use GraphQL vs REST

```
Use GraphQL when:
✅ Multiple clients with different data needs (web, mobile, TV)
✅ Complex nested/relational data
✅ Rapid frontend iteration — backend schema stays stable
✅ Want to avoid over/under fetching
✅ Need real-time subscriptions built in
✅ Large teams where frontend/backend work independently

Use REST when:
✅ Simple CRUD with flat data
✅ Public API — REST is universally understood
✅ File uploads (REST handles these more naturally)
✅ HTTP caching is important (CDN, browser cache)
✅ Small team, simple use case
✅ Existing ecosystem expects REST
```

### Key Terms

| Term | Meaning |
|---|---|
| **GraphQL** | Query language for APIs — client specifies exactly what data it needs |
| **Schema** | The typed contract defining all available data and operations |
| **SDL** | Schema Definition Language — syntax used to write GraphQL schemas |
| **Query** | Read operation — fetches data |
| **Mutation** | Write operation — creates/updates/deletes data |
| **Subscription** | Real-time operation — server pushes updates to client |
| **Resolver** | Server function that actually fetches the data for a field |
| **Over-fetching** | Getting more data than needed |
| **Under-fetching** | Not getting enough data — requiring multiple requests |
| **Introspection** | Ability to query the schema itself — self-documenting |
| **Apollo Client** | Most popular GraphQL client library for React |
| **gql** | Template literal tag for writing GraphQL queries in JS |

---

*Chapter 04 complete — Networking Module Done ✅*  
*Next → New Module*

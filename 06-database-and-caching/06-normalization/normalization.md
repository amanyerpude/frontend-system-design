# Normalization

## I. Overview

Normalization is the process of **organizing data to reduce redundancy and improve integrity**. In frontend system design, normalization applies to both client-side state management (how you structure data in your store) and database design (how you structure data on the server). The core principle is the same: **store each piece of data once and reference it by ID**.

## II. Why Normalization Matters

### Without Normalization (Denormalized)

```typescript
// ❌ DENORMALIZED — data is duplicated
const state = {
  posts: [
    { id: 1, title: 'Post 1', author: { id: 1, name: 'Alice', avatar: '...' } },
    { id: 2, title: 'Post 2', author: { id: 2, name: 'Bob', avatar: '...' } },
    { id: 3, title: 'Post 3', author: { id: 1, name: 'Alice', avatar: '...' } },
  ],
  comments: [
    { id: 1, postId: 1, author: { id: 2, name: 'Bob', avatar: '...' }, text: 'Great!' },
    { id: 2, postId: 1, author: { id: 3, name: 'Charlie', avatar: '...' }, text: 'Nice!' },
  ],
}

// Problems:
// 1. Alice's data appears 3 times — update requires updating ALL copies
// 2. Inconsistent data if one copy is missed
// 3. Memory waste
// 4. No single source of truth
```

### With Normalization

```typescript
// ✅ NORMALIZED — data stored once, referenced by ID
const state = {
  posts: {
    1: { id: 1, title: 'Post 1', authorId: 1 },
    2: { id: 2, title: 'Post 2', authorId: 2 },
    3: { id: 3, title: 'Post 3', authorId: 1 },
  },
  comments: {
    1: { id: 1, postId: 1, authorId: 2, text: 'Great!' },
    2: { id: 2, postId: 1, authorId: 3, text: 'Nice!' },
  },
  users: {
    1: { id: 1, name: 'Alice', avatar: '...' },
    2: { id: 2, name: 'Bob', avatar: '...' },
    3: { id: 3, name: 'Charlie', avatar: '...' },
  },
}

// Benefits:
// 1. Update Alice's name once → all posts reference the same user
// 2. No duplication → less memory
// 3. Single source of truth
// 4. Easy to cache and sync
```

## III. Normalization Rules

### Rule 1: Store Each Entity Once

```typescript
// ❌ Duplicate user data across posts
posts: [
  { id: 1, title: 'Hello', author: 'Alice', email: 'alice@example.com' },
  { id: 2, title: 'World', author: 'Bob', email: 'bob@example.com' },
]

// ✅ Store users separately, reference by ID
posts: [
  { id: 1, title: 'Hello', authorId: 1 },
  { id: 2, title: 'World', authorId: 2 },
],
users: {
  1: { id: 1, name: 'Alice', email: 'alice@example.com' },
  2: { id: 2, name: 'Bob', email: 'bob@example.com' },
}
```

### Rule 2: Use Flat Structure for Collections

```typescript
// ❌ Nested — harder to query, update, and cache
state: {
  categories: [
    { id: 1, name: 'Tech', posts: [
      { id: 1, title: 'Post 1', comments: [...] },
      { id: 2, title: 'Post 2', comments: [...] },
    ]},
  ],
}

// ✅ Flat — easy to query by ID
state: {
  categories: { 1: { id: 1, name: 'Tech', postIds: [1, 2] } },
  posts: { 1: { id: 1, title: 'Post 1', categoryId: 1 }, ... },
  comments: { 1: { id: 1, postId: 1, text: '...' }, ... },
}
```

### Rule 3: Use IDs as References

```typescript
// Relationship types in normalized state:

// One-to-many (User → Posts)
users: { 1: { id: 1, name: 'Alice' } }
posts: { 1: { id: 1, authorId: 1 } }  // Reference to user

// Many-to-many (Posts ↔ Tags)
posts: { 1: { id: 1, tagIds: [1, 2] } }
tags: { 1: { id: 1, name: 'JS' }, 2: { id: 2, name: 'React' } }

// One-to-one (User → Profile)
users: { 1: { id: 1, profileId: 1 } }
profiles: { 1: { id: 1, bio: '...' } }
```

## IV. Normalization in State Management

### Redux with Normalized State

```typescript
// ❌ Denormalized Redux state
const initialState = {
  posts: [
    { id: 1, title: 'Hello', author: { id: 1, name: 'Alice' } },
    { id: 2, title: 'World', author: { id: 2, name: 'Bob' } },
  ],
}

// ✅ Normalized Redux state
const initialState = {
  posts: {
    ids: [1, 2],
    entities: {
      1: { id: 1, title: 'Hello', authorId: 1 },
      2: { id: 2, title: 'World', authorId: 2 },
    },
  },
  users: {
    ids: [1, 2],
    entities: {
      1: { id: 1, name: 'Alice' },
      2: { id: 2, name: 'Bob' },
    },
  },
}

// Selector to get denormalized data when needed
const selectPostWithAuthor = createSelector(
  [selectPostById, selectUsers],
  (post, users) => ({
    ...post,
    author: users[post.authorId],
  })
)
```

### Normalizr for API Responses

```typescript
import { normalize, schema } from 'normalizr'

// API response (denormalized)
const apiResponse = {
  id: 1,
  title: 'Hello',
  author: { id: 1, name: 'Alice' },
  comments: [
    { id: 1, text: 'Great!', author: { id: 2, name: 'Bob' } },
  ],
}

// Define schemas
const userSchema = new schema.Entity('users')
const commentSchema = new schema.Entity('comments', { author: userSchema })
const postSchema = new schema.Entity('posts', {
  author: userSchema,
  comments: [commentSchema],
})

// Normalize
const normalized = normalize(apiResponse, postSchema)

// Result:
// {
//   entities: {
//     posts: { 1: { id: 1, title: 'Hello', author: 1, comments: [1] } },
//     users: { 1: { id: 1, name: 'Alice' }, 2: { id: 2, name: 'Bob' } },
//     comments: { 1: { id: 1, text: 'Great!', author: 2 } },
//   },
//   result: 1,
// }
```

## V. Database Normalization (Server-Side)

The same principle applies to relational databases:

### First Normal Form (1NF)

- Each cell contains a single value
- No repeating groups

```
❌ Orders table with repeating columns:
| id | product1    | product2    | product3    |
|----|-------------|-------------|-------------|
| 1  | 'Widget'    | 'Gadget'    | NULL        |

✅ 1NF — one product per row:
| id | product_id | quantity |
|----|------------|----------|
| 1  | 101        | 2        |
| 1  | 102        | 1        |
```

### Second Normal Form (2NF)

- 1NF + no partial dependencies (non-key columns depend on the entire primary key)

### Third Normal Form (3NF)

- 2NF + no transitive dependencies (non-key columns depend only on the primary key)

```
❌ 3NF violation — city depends on zip_code, not on order_id
| order_id | customer_id | zip_code | city       |
|----------|-------------|----------|------------|
| 1        | 100         | 12345    | Springfield|

✅ 3NF — separate tables
Orders:    | order_id | customer_id |
Customers: | customer_id | zip_code |
Locations: | zip_code | city |
```

## VI. Denormalization (When to Break the Rules)

Sometimes duplication is intentional for performance:

| Scenario | Why Denormalize |
|----------|----------------|
| **Read-heavy APIs** | Avoid expensive JOINs; return pre-assembled data |
| **Caching responses** | Store complete response, not fragments |
| **Real-time dashboards** | Pre-compute aggregates to reduce query time |
| **Static content** | No need to normalize if data never changes |

**Trade-off:** Denormalization improves read performance but increases write complexity and risks inconsistency.

## VII. Key Takeaways

1. **Store each entity once** — reference by ID
2. **Flat structure** is easier to query, update, and cache than nested
3. **Use Normalizr** to normalize API responses automatically
4. **Denormalize only for measured performance reasons** — not by default
5. **Select denormalized data when needed** — normalize in the store, denormalize in selectors

## References

- [Normalizr Documentation](https://github.com/paularmstrong/normalizr)
- [Redux: Normalized State](https://redux.js.org/usage/structuring-reducers/normalizing-state-shape)
- [Database Normalization (Wikipedia)](https://en.wikipedia.org/wiki/Database_normalization)

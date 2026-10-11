# IndexedDB

## I. Overview

IndexedDB is a **low-level, asynchronous, transactional database** built into the browser. It stores structured data (not just strings) and supports large datasets, full-text search, and indexes. It is the most powerful client-side storage mechanism and the foundation for offline-capable Progressive Web Apps (PWAs).

## II. Core Characteristics

| Characteristic | Behavior |
|----------------|----------|
| **Persistence** | Survives page reloads, browser restarts |
| **Scope** | Per origin |
| **Capacity** | ~50% of available disk (browser-managed quota) |
| **API type** | **Asynchronous** (does not block main thread) |
| **Data type** | Any structured-cloneable type (objects, arrays, dates, blobs) |
| **Transactions** | Full ACID transaction support |
| **Indexes** | Built-in index support for fast queries |
| **Expiration** | Never (until explicitly cleared or browser clears storage) |
| **Sent with requests** | No |

## III. Key Concepts

| Concept | Description |
|---------|-------------|
| **Database** | Top-level container; holds object stores |
| **Object Store** | Like a table in a relational DB; holds records |
| **Index** | A lookup table for querying by a specific field |
| **Transaction** | A unit of work that is atomic (all or nothing) |
| **Record** | A single data entry (key + value) |
| **Key** | Unique identifier for a record (can be a number, string, date, or array) |
| **Version** | Database schema version; incremented when adding/removing stores |

## IV. API Reference

### Opening a Database

```typescript
function openDatabase(): Promise<IDBDatabase> {
  return new Promise((resolve, reject) => {
    const request = indexedDB.open('MyAppDB', 1)  // version 1

    request.onupgradeneeded = (event) => {
      const db = (event.target as IDBOpenDBRequest).result

      // Create object store (like a table)
      if (!db.objectStoreNames.contains('users')) {
        const userStore = db.createObjectStore('users', { keyPath: 'id' })
        userStore.createIndex('email', 'email', { unique: true })
        userStore.createIndex('name', 'name', { unique: false })
      }

      if (!db.objectStoreNames.contains('products')) {
        const productStore = db.createObjectStore('products', { keyPath: 'id' })
        productStore.createIndex('category', 'category', { unique: false })
        productStore.createIndex('price', 'price', { unique: false })
      }
    }

    request.onsuccess = () => resolve(request.result)
    request.onerror = () => reject(request.error)
  })
}
```

### CRUD Operations

```typescript
// CREATE — Add a record
async function addUser(user: User): Promise<void> {
  const db = await openDatabase()
  return new Promise((resolve, reject) => {
    const tx = db.transaction('users', 'readwrite')
    const store = tx.objectStore('users')
    store.add(user)
    tx.oncomplete = () => resolve()
    tx.onerror = () => reject(tx.error)
  })
}

// READ — Get a record by key
async function getUser(id: number): Promise<User | undefined> {
  const db = await openDatabase()
  return new Promise((resolve, reject) => {
    const tx = db.transaction('users', 'readonly')
    const store = tx.objectStore('users')
    const request = store.get(id)
    request.onsuccess = () => resolve(request.result)
    request.onerror = () => reject(request.error)
  })
}

// READ ALL — Get all records
async function getAllUsers(): Promise<User[]> {
  const db = await openDatabase()
  return new Promise((resolve, reject) => {
    const tx = db.transaction('users', 'readonly')
    const store = tx.objectStore('users')
    const request = store.getAll()
    request.onsuccess = () => resolve(request.result)
    request.onerror = () => reject(request.error)
  })
}

// UPDATE — Modify a record
async function updateUser(user: User): Promise<void> {
  const db = await openDatabase()
  return new Promise((resolve, reject) => {
    const tx = db.transaction('users', 'readwrite')
    const store = tx.objectStore('users')
    store.put(user)  // put = add or update (upsert)
    tx.oncomplete = () => resolve()
    tx.onerror = () => reject(tx.error)
  })
}

// DELETE — Remove a record
async function deleteUser(id: number): Promise<void> {
  const db = await openDatabase()
  return new Promise((resolve, reject) => {
    const tx = db.transaction('users', 'readwrite')
    const store = tx.objectStore('users')
    store.delete(id)
    tx.oncomplete = () => resolve()
    tx.onerror = () => reject(tx.error)
  })
}
```

### Querying with Indexes

```typescript
// Query by index
async function getUserByEmail(email: string): Promise<User | undefined> {
  const db = await openDatabase()
  return new Promise((resolve, reject) => {
    const tx = db.transaction('users', 'readonly')
    const store = tx.objectStore('users')
    const index = store.index('email')
    const request = index.get(email)
    request.onsuccess = () => resolve(request.result)
    request.onerror = () => reject(request.error)
  })
}

// Range queries
async function getProductsByPriceRange(min: number, max: number): Promise<Product[]> {
  const db = await openDatabase()
  return new Promise((resolve, reject) => {
    const tx = db.transaction('products', 'readonly')
    const store = tx.objectStore('products')
    const index = store.index('price')
    const range = IDBKeyRange.bound(min, max)
    const request = index.getAll(range)
    request.onsuccess = () => resolve(request.result)
    request.onerror = () => reject(request.error)
  })
}

// Cursor for iterating
async function forEachUser(callback: (user: User) => void): Promise<void> {
  const db = await openDatabase()
  return new Promise((resolve, reject) => {
    const tx = db.transaction('users', 'readonly')
    const store = tx.objectStore('users')
    const request = store.openCursor()

    request.onsuccess = () => {
      const cursor = request.result
      if (cursor) {
        callback(cursor.value)
        cursor.continue()  // Move to next record
      } else {
        resolve()  // No more records
      }
    }
    request.onerror = () => reject(request.error)
  })
}
```

## V. Schema Migrations

```typescript
function openDatabase(): Promise<IDBDatabase> {
  return new Promise((resolve, reject) => {
    // Version 2 — adding a new store
    const request = indexedDB.open('MyAppDB', 2)

    request.onupgradeneeded = (event) => {
      const db = (event.target as IDBOpenDBRequest).result
      const oldVersion = event.oldVersion
      const newVersion = event.newVersion

      // Version 1: Create users store
      if (oldVersion < 1) {
        const userStore = db.createObjectStore('users', { keyPath: 'id' })
        userStore.createIndex('email', 'email', { unique: true })
      }

      // Version 2: Create products store
      if (oldVersion < 2) {
        const productStore = db.createObjectStore('products', { keyPath: 'id' })
        productStore.createIndex('category', 'category', { unique: false })
      }
    }

    request.onsuccess = () => resolve(request.result)
    request.onerror = () => reject(request.error)
  })
}
```

## VI. When to Use IndexedDB

| Use Case | Why |
|----------|-----|
| **Offline-first PWA** | Store data locally for offline access |
| **Large datasets** | No practical size limit (unlike localStorage's ~5MB) |
| **Structured data** | Objects, arrays, dates, blobs (not just strings) |
| **Full-text search** | Indexed queries with cursors |
| **Caching API responses** | Store JSON responses for offline fallback |
| **Media storage** | Blobs (images, audio, video) via `File`/`Blob` types |
| **Client-side database** | Full ACID transactions, indexes |

## VII. When NOT to Use IndexedDB

| Problem | Alternative |
|---------|------------|
| **Simple key-value pairs** | Local Storage or Session Storage |
| **Auth tokens** | HttpOnly cookies |
| **Small, session-scoped data** | Session Storage |
| **Data that must sync to server** | Server-side DB with sync mechanism |
| **Simple caching** | Cache API or HTTP cache |

## VIII. Libraries That Wrap IndexedDB

Raw IndexedDB API is verbose and callback-based. Libraries simplify it:

| Library | Features |
|---------|----------|
| **Dexie.js** | Promise-based, simple API, indexes, migrations |
| **idb** | Small wrapper by Jake Archibald; promise-based |
| **localForage** | Simple API, falls back to IndexedDB → localStorage → WebSQL |
| **RxDB** | Reactive, offline-first, replication |
| **WatermelonDB** | Optimized for React Native + web |

### Dexie.js Example

```typescript
import Dexie, { Table } from 'dexie'

class AppDatabase extends Dexie {
  users!: Table<User, number>
  products!: Table<Product, number>

  constructor() {
    super('MyAppDB')
    this.version(1).stores({
      users: '++id, email, name',
      products: '++id, category, price',
    })
  }
}

const db = new AppDatabase()

// CRUD is much simpler
await db.users.add({ name: 'Alice', email: 'alice@example.com' })
const user = await db.users.where('email').equals('alice@example.com').first()
await db.users.update(user.id, { name: 'Alice Smith' })
await db.users.delete(user.id)

// Batch operations
await db.transaction('rw', db.users, async () => {
  await db.users.bulkAdd(userArray)
  await db.products.bulkAdd(productArray)
})
```

## IX. Transactions

IndexedDB transactions are ACID-compliant:

```typescript
// Atomic batch operation
async function transferData(fromStore: string, toStore: string): Promise<void> {
  const db = await openDatabase()
  return new Promise((resolve, reject) => {
    const tx = db.transaction([fromStore, toStore], 'readwrite')

    const from = tx.objectStore(fromStore)
    const to = tx.objectStore(toStore)

    // All operations in this transaction succeed or fail together
    from.getAll().onsuccess = (event) => {
      const data = (event.target as IDBRequest).result
      data.forEach(item => to.add(item))
      from.clear()
    }

    tx.oncomplete = () => resolve()
    tx.onerror = () => reject(tx.error)
  })
}
```

## X. Performance Tips

| Tip | Why |
|-----|-----|
| **Batch writes in a single transaction** | One transaction = one disk write |
| **Use indexes for queries** | Avoids full table scans |
| **Use cursors for iteration** | `getAll()` loads everything into memory |
| **Close database when done** | Frees resources |
| **Don't store large blobs unnecessarily** | Blobs consume quota quickly |
| **Use `structuredClone` for complex objects** | Avoids reference errors |

## References

- [MDN: IndexedDB API](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)
- [Dexie.js](https://dexie.org/)
- [idb by Jake Archibald](https://github.com/jakearchibald/idb)
- [web.dev: IndexedDB](https://web.dev/learn/pwa/storage/)

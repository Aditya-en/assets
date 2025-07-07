# Prisma Basics

Prisma can feel like **magic** when it works — type-safe queries, autocompletion, and an insanely smooth DX. But when you’re starting out, the schema syntax and the CLI commands can feel a bit… _weird_.

So here’s a practical beginner friendly guide to Prisma features and commands I use THE MOST. If you're building anything with a database (Next.js, Express, REST, GraphQL — you name it), Prisma will make your life easier.

> 📌 This is a **practical cheatsheet**, not a full-blown tutorial. Ideal for referencing while building.

* * *

Basic Setup

### Install Prisma CLI and Init
```bash
npm install prisma --save-dev
npm install @prisma/client
npx prisma init
```

This creates:

*   `prisma/schema.prisma` – your DB schema
*   `.env` – where your DB URL lives

* * *

Defining Models

### A Simple Example
```Prisma.Schema
model User {
  id       Int      @id @default(autoincrement())
  email    String   @unique
  name     String?
  posts    Post[]
}

model Post {
  id        Int     @id @default(autoincrement())
  title     String
  content   String?
  published Boolean @default(false)
  author    User    @relation(fields: [authorId], references: [id])
  authorId  Int
}
```
* * *

Migrations and DB Sync

### Push schema without migration
```bash
npx prisma db push
```

Good for **quick prototyping**.

### Generate a SQL migration
```bash
npx prisma migrate dev --name init
```
Good for **production**.

* * *

✏️ Prisma Client Basics
-----------------------

### Import Prisma Client
```typescript
import { PrismaClient } from '@prisma/client'
const prisma = new PrismaClient()
```
* * *

📖 CRUD Examples
----------------

### Create
```typescript
const user = await prisma.user.create({
  data: {
    email: 'test@example.com',
    name: 'Alice',
  },
})
```
### Read
```typescript
const allUsers = await prisma.user.findMany()
```
### Update
```typescript
const updated = await prisma.user.update({
  where: { id: 1 },
  data: { name: 'Bob' },
})
```
### Delete
```typescript
await prisma.user.delete({ where: { id: 1 } })
```
* * *

🔍 Filtering Queries
--------------------
```typescript
await prisma.post.findMany({
  where: {
    published: true,
    title: {
      contains: 'Prisma',
    },
  },
})
```
* * *

🌱 Relations (Include)
----------------------
```typescript
await prisma.user.findMany({
  include: {
    posts: true,
  },
})
```
* * *

🧪 Raw SQL (When You Really Need It)
------------------------------------
```typescript
await prisma.$queryRaw`SELECT * FROM User WHERE email = ${email}`
```
Or use the unsafe version if you must:

```typescript
await prisma.$queryRawUnsafe(`SELECT * FROM User WHERE email = '${email}'`)
```

**⚠️ Caution:** Avoid `$queryRawUnsafe` unless absolutely necessary. It _can_ lead to SQL injection.

* * *

🧹 Resetting Your DB
--------------------
```bash
npx prisma migrate reset
```
Drops the DB, runs migrations, and seeds again.

* * *

🌍 Environment Variables
------------------------

Define your DB URL in `.env`:
```env
DATABASE_URL="postgresql://user:pass@localhost:5432/mydb"
```
* * *

🌱 Seeding Data
---------------

Create `prisma/seed.ts`:
```typescript
import { PrismaClient } from '@prisma/client'
const prisma = new PrismaClient()

async function main() {
  await prisma.user.create({
    data: {
      email: 'admin@demo.com',
      name: 'Admin',
    },
  })
}

main()
```
Then run:
```bash
npx prisma db seed
```
* * *

Useful Prisma CLI Commands
-----------------------------

*   `npx prisma init` → Initialize Prisma
*   `npx prisma db push` → Sync DB without migration
*   `npx prisma migrate dev --name mymigration` → Create migration
*   `npx prisma studio` → Open the **visual editor** (super handy!)
*   `npx prisma generate` → Regenerate client after schema change

* * *

Favorite Schema Tricks
-------------------------

### Default Values

  `createdAt DateTime @default(now())`

### Optional Fields

  `name String?`

### Unique Fields

  `email String @unique`

### One-to-Many Relationship
```prisma.schema
model User {
  posts Post[]
}
model Post {
  author User @relation(fields: [authorId], references: [id])
  authorId Int
}
```
* * *

Special Tips & Tricks
------------------------

*   Use `prisma format` to auto-format your schema file.
*   Use `prisma.$transaction()` to group multiple DB operations safely.
*   `@default(uuid())` for UUIDs instead of auto-incremented integers.
*   Enable `previewFeatures` in `schema.prisma` for experimental goodies.

* * *

Final Tip
------------

Don’t try to learn everything at once. Start by modeling your database with simple `User`/`Post` types, and just `push`, `query`, and `seed`. Prisma feels like magic once you’ve run a few successful queries and your DB feels **type-safe and solid**.

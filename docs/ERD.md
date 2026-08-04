# Entity Relationship Diagram (ERD)

> Define your database schema here.
> Agent will use this to generate Prisma schema and validate data models in spec.

---

## Database

**Type:** PostgreSQL / MySQL / MongoDB / SQLite
**ORM:** Prisma / Drizzle / TypeORM / Mongoose

---

## Entities

### User
| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | String (cuid) | PK | Unique identifier |
| email | String | UNIQUE, NOT NULL | Login email |
| password | String | NOT NULL | Hashed password |
| name | String | NOT NULL | Display name |
| role | Enum(USER,ADMIN) | DEFAULT USER | Access role |
| createdAt | DateTime | DEFAULT now() | Created timestamp |
| updatedAt | DateTime | Auto-update | Updated timestamp |

### [Entity 2]
| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | | PK | |
| userId | String | FK → User.id | Owner |
| ... | | | |

---

## Relationships

```
User ─── has many ──→ [Entity2]
[Entity2] ─── belongs to ──→ User
[Entity2] ─── has many ──→ [Entity3]
```

---

## Prisma Schema (optional — paste if available)

```prisma
model User {
  id        String   @id @default(cuid())
  email     String   @unique
  password  String
  name      String
  role      Role     @default(USER)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

enum Role {
  USER
  ADMIN
}
```

---

## Indexes

| Table | Columns | Type | Reason |
|-------|---------|------|--------|
| User | email | UNIQUE | Fast login lookup |
| [Table] | [col] | INDEX | [Query optimization] |

---

## Notes

- [Any special constraints or business rules about data]
- [Soft delete strategy if applicable]
- [Archival / retention policy]

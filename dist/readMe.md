# Type-Safe Person Filtering System

A TypeScript implementation demonstrating type-safe filtering of User/Admin entities with compile-time validation. Uses advanced TypeScript features for type narrowing and criteria matching.

## Features

- 👥 **Discriminated Unions**: `User` and `Admin` types with `type` discriminant
- 🔍 **Type-Safe Filtering**: Filter by person type + criteria with full type checking
- 🛡️ **Compile-Time Validation**:
  - Ensures valid `personType` ('user'|'admin')
  - Validates criteria properties against type-specific fields
  - Returns properly typed results (`User[]` or `Admin[]`)
- 🧩 **Partial Criteria Matching**: Match any subset of valid properties

## How It Works

### 1. Core Types
```typescript
interface User {
    type: 'user';
    name: string;
    age: number;
    occupation: string;
}

interface Admin {
    type: 'admin';
    name: string;
    age: number;
    role: string;
}

type Person = User | Admin;
```

### 2. Filter Function
```typescript
function filterPersons<T extends 'user' | 'admin'>(
    persons: Person[],
    personType: T,
    criteria: Partial<Omit<Extract<Person, { type: T }>, 'type'>>
): Extract<Person, { type: T }>[]
```

#### Key Mechanisms:
1. **Generic Type Constraint** (`T extends 'user'|'admin'`):
   - Restricts `personType` to valid values
   - Determines return type (`User[]` when T='user', `Admin[]` when T='admin')

2. **Criteria Typing**:
   - `Extract<Person, { type: T }>`: Gets target type (User/Admin)
   - `Omit<..., 'type'>`: Removes `type` property (already fixed by filter)
   - `Partial<...>`: Allows any subset of remaining properties

3. **Type Predicate Filter**:
   ```typescript
   .filter((person): person is Extract<Person, { type: T }> => ...)
   ```
   - Tells TypeScript to narrow type after filtering

### 3. Usage Examples
```typescript
// Get all users aged 23 (type: User[])
const usersOfAge23 = filterPersons(persons, 'user', { age: 23 });

// Get all admins aged 23 (type: Admin[])
const adminsOfAge23 = filterPersons(persons, 'admin', { age: 23 });

// Complex criteria (name + age)
const seniorAdmins = filterPersons(persons, 'admin', { 
    age: 64,
    name: 'Bruce Willis'
});
```

### 4. Type Safety Enforcement
**Valid Operations**:
```typescript
filterPersons(persons, 'user', { occupation: 'Dev' }); // ✅ Valid user field
filterPersons(persons, 'admin', { role: 'Manager' });  // ✅ Valid admin field
```

**Invalid Operations**:
```typescript
filterPersons(persons, 'moderator', {}); // 🚫 Invalid personType
filterPersons(persons, 'user', { role: 'Dev' }); // 🚫 Invalid user field
filterPersons(persons, 'admin', { occupation: 'Dev' }); // 🚫 Invalid admin field
```

## Benefits
- 🚫 **Prevents Runtime Errors**: Catches invalid filters at compile-time
- 🧑💻 **IDE Support**: Autocompletion for valid criteria properties
- 🔄 **Maintainable**: Type system ensures consistency across changes
- 🎯 **Precise Filtering**: Combine type and property filters in single operation

## Running
1. Install TypeScript:
   ```bash
   npm install -g typescript
   ```
2. Compile & Run:
   ```bash
   tsc && node index.js
   ```

## Output
```
Users of age 23:
 - Kate Müller, 23, Astronaut
 - Wilson, 23, Ball

Admins of age 23:
 - Agent Smith, 23, Anti-virus engineer
```
```
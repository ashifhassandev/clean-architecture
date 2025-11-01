# Clean Architecture in Action: Design Robust Applications That Last

Building scalable, maintainable, and testable applications requires more than just good coding — it demands thoughtful architecture. Clean Architecture, popularized by Robert C. Martin (Uncle Bob), provides a set of principles to ensure your software remains flexible and robust as it grows.

## Introduction

In this guide, we will cover how to structure your applications using Clean Architecture and apply its principles effectively in real-world projects.

---

## 🧩 Core Concepts of Clean Architecture

At the heart of Clean Architecture lies the **Dependency Rule**:

> Source code dependencies must always point inward.  
> Inner layers should never depend on outer layers.

This leads to four main layers:

### 1. Entities
- Represent core business rules and domain models.  
- Independent of frameworks, databases, or external systems.  
**Example:** `user.entity.ts` defines validations and domain properties.

### 2. Use Cases
- Contain application-specific business logic.  
- Define workflows and coordinate entities.  
**Example:** `create-user.usecase.ts` handles user registration logic.

### 3. Interface Adapters
- Translate between the inner application and outer frameworks.  
**Example:** `user-repository.interface.ts` defines repository contracts.

### 4. Frameworks & Drivers
- Outermost layer — databases, UIs, APIs, external services.  
**Example:** `typeorm-user.repository.ts` integrates with TypeORM.

---

## 🧱 Architectural Layers in Practice

### Example: User Registration Workflow

#### Entity
```ts
// user.entity.ts
export class User {
  constructor(public readonly id: string, public name: string, public email: string) {}
}
```

#### Use Case
```ts
// create-user.usecase.ts
import { ICreateUserUseCase } from "./interfaces/create-user.usecase.interface";
import { IUserRepository } from "../interface-adapters/user-repository.interface";
import { User } from "../entities/user.entity";

export class CreateUserUseCase implements ICreateUserUseCase{
  constructor(private readonly _userRepository: IUserRepository) {}

  async execute(name: string, email: string): Promise<User> {
    const user = new User(Date.now().toString(), name, email);
    return await this._userRepository.save(user);
  }
}
```

#### Interface Adapters
```ts
// user-repository.interface.ts
import { User } from "../entities/user.entity";

export interface IUserRepository {
  save(user: User): Promise<User>;
  findByEmail(email: string): Promise<User | null>;
}
```

#### Frameworks & Drivers
```ts
// typeorm-user.repository.ts
import { IUserRepository } from "../../interface-adapters/user-repository.interface";
import { User } from "../../entities/user.entity";

export class TypeOrmUserRepository implements IUserRepository {
  async save(user: User): Promise<User> {
    return user; // Implementation using TypeORM
  }
  async findByEmail(email: string): Promise<User | null> {
    return null; // Database query logic
  }
}
```

This separation ensures that changing the database, UI, or external system does not affect the business rules.

---

## 🧠 Naming Conventions for Private Variables

Consistent naming improves readability and encapsulation.

- Use a leading underscore for private variables (e.g., `_email`, `_password`).  
- Always declare with `private`.  
- Use expressive, descriptive names.  
- Provide read access via getters and modify via methods.

```ts
export class User {
  private _name: string;
  private _email: string;
  private _password: string;
  
  get name(): string { 
    return this._name;
  }
}
```

---

## 🔁 Separation of Mapping Profiles

Mapping between domain entities and DTOs should not clutter business logic.

- Place mapping logic in a dedicated `mappers` or `profiles` folder.  
- Centralize conversions for clarity and testability.

```ts
export class UserMapper {
  static toDTO(user: User): UserDTO {
    return { id: user.id, name: user.name, email: user.email };
  }
}
```

---

## 🧩 Use Case Return Types: Entity vs DTO

| Approach | When to Use | Description |
|-----------|-------------|-------------|
| **Entities** | Small systems | Return domain models with behavior |
| **DTOs** | Larger systems | Return simple data structures for UI/API |

Example using DTOs:

```ts
// use-cases/dtos/user.dto.ts
export interface UserDTO {
  id: string;
  name: string;
  email: string;
}
```

```ts
// use-cases/create-user.usecase.ts
import { IUserRepository } from "../interface-adapters/user-repository.interface";
import { User } from "../entities/user.entity";
import { UserDTO } from "./dtos/user.dto";

export class CreateUserUseCase {
  constructor(private readonly userRepository: IUserRepository) {}

  async execute(name: string, email: string): Promise<UserDTO> {
    const user = new User(Date.now().toString(), name, email);
    const savedUser = await this.userRepository.save(user);
    return { id: savedUser.id, name: savedUser.name, email: savedUser.email };
  }
}
```

---

## 🧩 Understanding Dependency Injection

Dependency Injection (DI) decouples classes from their dependencies, improving modularity and testability.

### Example
```ts
class A {
  constructor(private b: B) {} // Dependency injected externally
}
```

### Clean Architecture DI Approaches

#### 1. Small Apps — Manual DI
```ts
// app.ts
const userRepo = new TypeOrmUserRepository();
const createUserUseCase = new CreateUserUseCase(userRepo);
const controller = new UserController(createUserUseCase);
```

#### 2. Medium Apps — Lightweight DI Container (e.g., tsyringe)
```ts
import { container, injectable, inject } from "tsyringe";

@injectable()
class CreateUserUseCase {
  constructor(@inject("UserRepository") private repo: IUserRepository) {}
}

container.register("UserRepository", { useClass: TypeOrmUserRepository });
const useCase = container.resolve(CreateUserUseCase);
```

#### 3. Large Apps — Framework-based DI (e.g., NestJS)
```ts
@Module({
  providers: [
    { provide: IUserRepository, useClass: TypeOrmUserRepository },
    CreateUserUseCase,
  ],
})
export class UserModule {}
```

---

## 🧩 Example Folder Structure

```
src/
├── domain/
│   ├── entities/
│   ├── enums/
│   ├── dtos/
├── application/
│   ├── use-cases/
│   ├── ports/
│   └── mappers/
├── infrastructure/
│   ├── repositories/
│   ├── services/
│   ├── database/
│   └── utils/
├── interfaces/
│   ├── controllers/
│   ├── routes/
│   └── adapters/
├── config/
├── shared/
└── tests/
```

Each folder represents a **layer** or **supporting concern** within the architecture.

---

## 🧭 Layer Responsibilities Summary

- **Domain** → Business entities and core rules  
- **Application** → Use cases, ports, mappers  
- **Infrastructure** → Database, services, utilities  
- **Interfaces** → API, routes, adapters  
- **Config** → Environment setup  
- **Shared** → Common helpers  
- **Tests** → Organized per layer  

---

## 🧩 Conclusion

Clean Architecture is not just about folder structure — it’s about **creating a system that remains adaptable** as requirements evolve.  
By following these principles and organization strategies, you can build software that’s:

- Maintainable  
- Testable  
- Scalable  
- Framework-independent  

**Further Reading:**
- *Clean Architecture* — Robert C. Martin  
- *Domain-Driven Design* — Eric Evans  
- *Test-Driven Development* — Kent Beck
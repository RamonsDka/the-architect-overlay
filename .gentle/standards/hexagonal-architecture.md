# Hexagonal Architecture (Ports & Adapters)

## Qué es

Arquitectura que separa la lógica de negocio (dominio) de los detalles de infraestructura (base de datos, APIs, UI). El dominio define **puertos** (interfaces) y la infraestructura provee **adaptadores** (implementaciones).

## Por qué lo usamos

- **Testabilidad:** El dominio se testea sin dependencias externas
- **Flexibilidad:** Cambiar de base de datos o framework no afecta la lógica de negocio
- **Claridad:** Las responsabilidades están bien definidas

## Cómo aplicarlo

### Estructura de carpetas

```
src/
├── domain/           # Lógica de negocio pura (NO depende de nada)
│   ├── entities/     # Modelos de dominio
│   ├── ports/        # Interfaces (contratos)
│   └── services/     # Casos de uso
├── application/      # Orquestación (usa domain)
│   └── use-cases/
└── infrastructure/   # Adaptadores (implementa ports)
    ├── repositories/ # Implementa ports de persistencia
    ├── api/          # Controllers, routes
    └── config/
```

### Ejemplo: Port (Interface)

```typescript
// src/domain/ports/UserRepository.ts
export interface UserRepository {
  findById(id: string): Promise<User | null>;
  save(user: User): Promise<void>;
}
```

### Ejemplo: Adapter (Implementación)

```typescript
// src/infrastructure/repositories/PostgresUserRepository.ts
import { UserRepository } from '@/domain/ports/UserRepository';

export class PostgresUserRepository implements UserRepository {
  async findById(id: string): Promise<User | null> {
    // Implementación con Postgres
  }
  
  async save(user: User): Promise<void> {
    // Implementación con Postgres
  }
}
```

### Ejemplo: Caso de Uso (Domain Service)

```typescript
// src/domain/services/CreateUser.ts
import { UserRepository } from '@/domain/ports/UserRepository';

export class CreateUser {
  constructor(private userRepo: UserRepository) {}
  
  async execute(data: CreateUserDTO): Promise<User> {
    const user = new User(data);
    await this.userRepo.save(user);
    return user;
  }
}
```

## Reglas de oro

1. **Domain NO importa de infrastructure** - Solo define ports
2. **Infrastructure implementa ports** - Adaptadores concretos
3. **Application orquesta** - Conecta domain con infrastructure
4. **Dependency Injection** - Inyectar adaptadores en constructores

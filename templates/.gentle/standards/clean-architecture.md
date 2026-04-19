# Clean Architecture

## Qué es

Arquitectura en capas concéntricas donde las dependencias apuntan hacia adentro. El núcleo (Entities) no depende de nada. Las capas externas dependen de las internas, nunca al revés.

## Por qué lo usamos

- **Independencia de frameworks:** El negocio no depende de React, Express, etc.
- **Testeable:** Cada capa se testea aisladamente
- **Independencia de UI/DB:** Cambiar tecnologías no afecta el core

## Cómo aplicarlo

### Capas (de adentro hacia afuera)

```
┌─────────────────────────────────────┐
│         Entities (Core)             │ ← Lógica de negocio pura
├─────────────────────────────────────┤
│         Use Cases                   │ ← Casos de uso de la aplicación
├─────────────────────────────────────┤
│  Interface Adapters (Controllers)   │ ← Adaptadores de entrada/salida
├─────────────────────────────────────┤
│  Frameworks & Drivers (DB, UI)      │ ← Detalles de implementación
└─────────────────────────────────────┘
```

### Regla de Dependencia

**Las dependencias apuntan HACIA ADENTRO:**
- Frameworks → Interface Adapters → Use Cases → Entities
- Entities NO conoce Use Cases
- Use Cases NO conoce Controllers
- Controllers NO conoce Frameworks (usa abstracciones)

### Ejemplo: Entity (Core)

```typescript
// src/entities/User.ts
export class User {
  constructor(
    public readonly id: string,
    public name: string,
    public email: string
  ) {
    this.validate();
  }
  
  private validate() {
    if (!this.email.includes('@')) {
      throw new Error('Invalid email');
    }
  }
  
  changeName(newName: string) {
    this.name = newName;
  }
}
```

### Ejemplo: Use Case

```typescript
// src/use-cases/GetUserById.ts
import { User } from '@/entities/User';
import { UserRepository } from '@/use-cases/ports/UserRepository';

export class GetUserById {
  constructor(private userRepo: UserRepository) {}
  
  async execute(id: string): Promise<User> {
    const user = await this.userRepo.findById(id);
    if (!user) throw new Error('User not found');
    return user;
  }
}
```

### Ejemplo: Controller (Interface Adapter)

```typescript
// src/adapters/controllers/UserController.ts
import { GetUserById } from '@/use-cases/GetUserById';

export class UserController {
  constructor(private getUserById: GetUserById) {}
  
  async getUser(req: Request, res: Response) {
    const user = await this.getUserById.execute(req.params.id);
    res.json(user);
  }
}
```

## Reglas de oro

1. **Entities son agnósticos** - No dependen de nada
2. **Use Cases orquestan Entities** - Lógica de aplicación
3. **Controllers adaptan** - Convierten HTTP/CLI a Use Cases
4. **Dependency Inversion** - Usar interfaces, no implementaciones concretas

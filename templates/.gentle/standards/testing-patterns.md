# Testing Patterns

## Qué es

Conjunto de patrones para escribir tests mantenibles, legibles y confiables. Seguimos la pirámide de testing: muchos tests unitarios, algunos de integración, pocos E2E.

## Por qué lo usamos

- **Confianza:** Los tests validan que el código funciona
- **Refactoring seguro:** Cambiar código sin romper funcionalidad
- **Documentación viva:** Los tests muestran cómo usar el código

## Cómo aplicarlo

### Pirámide de Testing

```
        /\
       /E2E\        ← Pocos, lentos, costosos
      /------\
     /  INT   \     ← Algunos, medianos
    /----------\
   /   UNIT     \   ← Muchos, rápidos, baratos
  /--------------\
```

### 1. Tests Unitarios (Unit Tests)

**Qué testean:** Una función/clase aislada  
**Herramientas:** Jest, Vitest, pytest

**Patrón AAA (Arrange-Act-Assert):**

```typescript
// user.test.ts
describe('User', () => {
  it('should throw error if email is invalid', () => {
    // Arrange
    const invalidEmail = 'not-an-email';
    
    // Act & Assert
    expect(() => new User('1', 'John', invalidEmail))
      .toThrow('Invalid email');
  });
});
```

### 2. Tests de Integración

**Qué testean:** Interacción entre componentes (ej. Use Case + Repository)  
**Herramientas:** Jest + test database

```typescript
// getUserById.integration.test.ts
describe('GetUserById Use Case', () => {
  let userRepo: UserRepository;
  let useCase: GetUserById;
  
  beforeEach(async () => {
    userRepo = new InMemoryUserRepository();
    useCase = new GetUserById(userRepo);
    await userRepo.save(new User('1', 'John', 'john@example.com'));
  });
  
  it('should return user when exists', async () => {
    const user = await useCase.execute('1');
    expect(user.name).toBe('John');
  });
  
  it('should throw error when user not found', async () => {
    await expect(useCase.execute('999'))
      .rejects.toThrow('User not found');
  });
});
```

### 3. Tests E2E (End-to-End)

**Qué testean:** Flujo completo desde UI/API hasta DB  
**Herramientas:** Playwright, Cypress

```typescript
// user.e2e.test.ts
test('should create and retrieve user via API', async ({ request }) => {
  // Create user
  const createRes = await request.post('/api/users', {
    data: { name: 'John', email: 'john@example.com' }
  });
  expect(createRes.ok()).toBeTruthy();
  const { id } = await createRes.json();
  
  // Retrieve user
  const getRes = await request.get(`/api/users/${id}`);
  const user = await getRes.json();
  expect(user.name).toBe('John');
});
```

### 4. Test Doubles (Mocks, Stubs, Fakes)

**Mock:** Verifica que se llamó con argumentos correctos

```typescript
const mockRepo = {
  save: jest.fn()
};
await useCase.execute(data);
expect(mockRepo.save).toHaveBeenCalledWith(expect.any(User));
```

**Stub:** Retorna datos predefinidos

```typescript
const stubRepo = {
  findById: jest.fn().mockResolvedValue(new User('1', 'John', 'john@example.com'))
};
```

**Fake:** Implementación simple en memoria

```typescript
class InMemoryUserRepository implements UserRepository {
  private users = new Map<string, User>();
  
  async findById(id: string) {
    return this.users.get(id) || null;
  }
  
  async save(user: User) {
    this.users.set(user.id, user);
  }
}
```

## Reglas de oro

1. **Tests unitarios NO tocan DB/API** - Usar mocks/fakes
2. **Un test, un concepto** - No testear múltiples cosas
3. **Nombres descriptivos** - `should throw error when email is invalid`
4. **Arrange-Act-Assert** - Estructura clara
5. **Tests rápidos** - Unitarios < 100ms, integración < 1s

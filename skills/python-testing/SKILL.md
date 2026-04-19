---
name: python-testing
description: >
  Python testing patterns with pytest, mocks, y fixtures.
  Trigger: When writing Python tests, using pytest, o adding test coverage.
license: Apache-2.0
metadata:
  author: gentleman-programming
  version: "1.0"
---

## When to Use

- Escribiendo tests unitarios en Python
- Usando pytest con mocks
- Creando fixtures para tests
- Testeando edge cases

## Critical Patterns

### Pattern 1: Mocks con unittest.mock

```python
from unittest.mock import Mock, MagicMock, patch, call

# Mock de función
@patch('modulo.funcion')
def test_con_mock(mock_funcion):
    mock_funcion.return_value = 'valor_mockeado'
    result = modulo.funcion()
    assert result == 'valor_mockeado'

# Mock de clase
@patch('modulo.Clase')
def test_con_mock_class(mock_clase):
    mock_clase.return_value.metodo.return_value = 'valor'
    obj = modulo.Clase()
    assert obj.metodo() == 'valor'
```

### Pattern 2: Pytest Fixtures

```python
import pytest

@pytest.fixture
def mock_session():
    """Fixture para sesión de base de datos"""
    session = MagicMock()
    return session

@pytest.fixture
def cliente_con_session(mock_session):
    """Cliente con dependencia mockeada"""
    return Cliente(session=mock_session)

def test_cliente_guardar(cliente_con_session):
    cliente_con_session.guardar()
    cliente_con_session.session.commit.assert_called_once()
```

### Pattern 3: Parametrización

```python
import pytest

@pytest.mark.parametrize("input,expected", [
    ("1+1", 2),
    ("2+2", 4),
    ("10-5", 5),
])
def test_evaluar(input, expected):
    assert evaluar(input) == expected
```

### Pattern 4: Testing Edge Cases

```python
def test_fecha_29_02_no_bisiesto():
    """Fecha 29/02 en año no bisiesto debe manejarse"""
    with pytest.raises(ValueError):
        parse_date("29/02/2023")

def test_cedula_con_ceros_iniciales():
    """Cédula con ceros iniciales debe preservarse"""
    result = formatear_cedula("001-1234567-8")
    assert result.startswith("001")

def test_csv_utf8_bom():
    """CSV con BOM debe leerse correctamente"""
    with open('archivo.csv', 'rb') as f:
        contenido = f.read()
    assert contenido.startswith(b'\xef\xbb\xbf')
```

## Commands

```bash
# Ejecutar todos los tests
pytest

# Tests con verbose
pytest -v

# Tests específicos
pytest tests/unit/test_archivo.py

# Con coverage
pytest --cov=backend --cov-report=html

# Tests que fallen primero
pytest -x

# Ver markers
pytest --markers
```

## Resources

- **Pytest Docs**: https://docs.pytest.org/
- **Mock Docs**: https://docs.python.org/3/library/unittest.mock.html

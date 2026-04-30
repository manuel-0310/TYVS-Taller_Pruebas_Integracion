# Registro de Defectos

Este documento recopila los **defectos detectados durante las pruebas unitarias, de integración y de sistema** del proyecto **Registraduría**.
Cada defecto se documenta de manera estructurada para facilitar su análisis, trazabilidad y corrección.

---

## Formato 1: Lista detallada (narrativa)

### Defecto 01 — Falta de validación de edad negativa *(Prueba unitaria)*

- **Capa afectada:** Dominio (`Registry.registerVoter`)
- **Caso de prueba:** Registro de persona con edad `-1`.
- **Entrada:**
`Person(name="Juan", id=101, age=-1, gender=MALE, alive=true)`
- **Resultado esperado:** `INVALID_AGE`
- **Resultado obtenido:** `VALID`
- **Causa probable:** La lógica de negocio no evalúa edades negativas.
- **Tipo de prueba:** Unitaria (dominio puro)
- **Estado:** Abierto
- **Prioridad:** Alta

---

### Defecto 02 — Registro de persona fallecida *(Prueba unitaria)*

- **Capa afectada:** Dominio (`Registry.registerVoter`)
- **Caso de prueba:** Persona con `alive=false`.
- **Entrada:**
`Person(name="Ana", id=102, age=45, gender=FEMALE, alive=false)`
- **Resultado esperado:** `DEAD`
- **Resultado obtenido:** `VALID`
- **Causa probable:** No se valida correctamente la condición `alive=false`.
- **Tipo de prueba:** Unitaria (regla de negocio)
- **Estado:** En progreso
- **Prioridad:** Media

---

### Defecto 03 — No se detectan duplicados *(Prueba de integración con H2)*

- **Capa afectada:** Infraestructura (`RegistryRepository`)
- **Caso de prueba:** Dos registros con el mismo `id`.
- **Entradas:**
  - Persona 1 → `Person(name="Carlos", id=200, age=30, gender=MALE, alive=true)`
  - Persona 2 → `Person(name="Carla", id=200, age=25, gender=FEMALE, alive=true)`
- **Resultado esperado:**
  - Persona 1 → `VALID`
  - Persona 2 → `DUPLICATED`
- **Resultado obtenido:**
  - Persona 1 → `VALID`
  - Persona 2 → `VALID`
- **Causa probable:** El método `existsById()` del repositorio no verifica correctamente la existencia previa del registro.
- **Tipo de prueba:** Integración (H2 + capa de aplicación)
- **Estado:** Abierto
- **Prioridad:** Alta

---

### Defecto 04 — Fallo en simulación con mock *(Prueba de integración con Mockito)*

- **Capa afectada:** Aplicación (`RegistryWithMockTest`)
- **Caso de prueba:** Registro con `id` duplicado en un repositorio simulado.
- **Configuración:**

```java
when(repo.existsById(7)).thenReturn(true);
```

- **Resultado esperado:** `DUPLICATED`
- **Resultado obtenido:** `NullPointerException`
- **Causa probable:** Dependencia `RegistryRepositoryPort` no inicializada correctamente durante el mock.
- **Tipo de prueba:** Integración (mock)
- **Estado:** En progreso
- **Prioridad:** Media

---

### Defecto 05 — Error HTTP 500 no manejado *(Prueba de sistema REST)*

- **Capa afectada:** Delivery (`RegistryController`)
- **Caso de prueba:** Envío de JSON con campo `gender` inválido.
- **Entrada:**

```json
{ "name": "Laura", "id": 500, "age": 20, "gender": "OTHER", "alive": true }
```

- **Resultado esperado:** `HTTP 400` (Bad Request)
- **Resultado obtenido:** `HTTP 500` (Internal Server Error)
- **Causa probable:** Falta de validación o manejo de excepción `IllegalArgumentException` en el controlador.
- **Tipo de prueba:** Sistema (`TestRestTemplate`)
- **Estado:** Resuelto
- **Prioridad:** Alta
- **Corrección aplicada:** Se agregó `@ExceptionHandler(IllegalArgumentException.class)` con `@ResponseStatus(HttpStatus.BAD_REQUEST)` en `RegistryController`. Ahora devuelve `HTTP 400` con body `"INVALID_INPUT"`.
- **Evidencia:** ![Ciclo TDD GREEN](img/tdd-green-ciclo1.png)

---

## Formato 2: Tabla de defectos (bug tracking)

| ID | Caso de Prueba | Capa | Resultado Esperado | Resultado Obtenido | Tipo | Estado | Prioridad |
|----|----------------|------|--------------------|--------------------|------|----------|------------|
| 01 | Edad negativa | Dominio | `INVALID_AGE` | `VALID` | Unitaria | Abierto | Alta |
| 02 | Persona muerta | Dominio | `DEAD` | `VALID` | Unitaria | En progreso | Media |
| 03 | Duplicado por ID | Infraestructura | `DUPLICATED` | `VALID` | Integración | Abierto | Alta |
| 04 | Mock mal configurado | Aplicación | `DUPLICATED` | `NullPointerException` | Integración (mock) | En progreso | Media |
| 05 | Error HTTP 500 | Delivery | `HTTP 400` | `HTTP 500` | Sistema (REST) | **Resuelto** | Alta |

---

## Convenciones de Estado

| Estado | Significado |
|---------|-------------|
| **Abierto** | El defecto fue detectado pero no corregido. |
| **En progreso** | El defecto se encuentra en análisis o corrección. |
| **Resuelto** | El defecto fue corregido y validado mediante pruebas. |

---

## Observaciones

- Los defectos detectados evidencian la importancia de **mantener pruebas unitarias robustas** antes de pasar a integración.
- La validación cruzada entre pruebas con mocks e integración real (H2) permitió identificar inconsistencias en el flujo de persistencia.
- Los errores en las pruebas REST destacan la necesidad de implementar **manejadores globales de excepciones (ControllerAdvice)** para mejorar la estabilidad del sistema.

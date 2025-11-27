# Métricas del Proyecto

---

## Sprint 1

### Métricas de Proceso (Scrum/Kanban)

#### Throughput
- **Issues completados**: 11 de 13
- **Story Points completados**: 21 de 22
- **Reflexión**: No completamos los issues #11 y #14 (documentación y video) porque priorizamos funcionalidad core.

#### Lead Time
- **Promedio**: 1.8 días
- **Desglose por issue**:
  - Issue #1: 1 hora
  - Issue #2: 2 horas
  - Issue #3: 3 horas
  - Issue #4: 3 horas
  - Issue #6: 3 horas
  - Issue #7: 2 horas
  - Issue #8: 2 horas
  - Issue #9: 2 horas
  - Issue #10: 1 hora
  - Issue #12: 1 hora
  - Issue #13: 1 hora

#### WIP (Work In Progress)
- **Límite acordado**: 2 issues simultáneos por persona
- **WIP máximo observado**: 4 issues
- **Veces que violamos el límite**: 1

---

### Métricas de Calidad Técnica / CI

#### Builds/Ejecuciones
- **Total de ejecuciones de make compose-up**: 6
  - Exitosas: 4
  - Fallidas: 2 (errores en docker-compose.yml por la versión de docker)
- **Total de ejecuciones de make compose-scan**: 4
  - Exitosas: 4
  - Fallidas: 0
- **Tiempo promedio de `docker-compose build`**: ~20 segundos

---

### Métricas de Seguridad (Zero-Trust)

#### Puertos alcanzables (Compose - SIN NetworkPolicies)

**Escenario**: Attacker conectado a backend-net (simula brecha de seguridad)

- **Puertos alcanzables**: 2
  - frontend:8080 → OPEN
  - backend:5000 → OPEN  **PROBLEMA IDENTIFICADO**
- **Puertos cerrados**: 1
  - backend:5432 → CLOSED (puerto simulado)

**Reporte JSON**: `reports/compose-connectivity.json`

#### Vulnerabilidades detectadas

| ID | Descripción | Severidad | Estado |
|----|-------------|-----------|--------|
| V-001 | Attacker puede acceder a backend:5000 sin restricciones | Alta | Identificada |
| V-002 | Frontend y backend en la misma red sin aislamiento explícito | Media | Identificada |

**Total vulnerabilidades**: 2 (Alta: 1, Media: 1)
**Vulnerabilidades mitigadas en Sprint 1**: 0
**Pendientes para Sprint 2**: 2 (se mitigarán con NetworkPolicies en K8s)

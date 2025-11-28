# Sprint 2 Backlog - Zero-Trust Network Sandbox

## Objetivo del Sprint 2

Implementar Kubernetes con NetworkPolicies para bloquear el movimiento lateral del attacker.

**Métrica clave:**

- Sprint 1: 2/3 puertos abiertos (66% superficie de ataque)
- Sprint 2: 0/3 puertos abiertos (0% superficie de ataque)
- **Reducción: 100%**

---

## Resumen

**Duración:** 5 días  
**Total issues:** 16 (#19-#34)  
**Story points:** 30 pts

---

## Issue #19: Namespace para el proyecto

**Labels:** `sprint-2`, `type:infrastructure`, `priority:P0`

### Descripción

Crear namespace dedicado para aislar los recursos del proyecto en Kubernetes.

### Tareas

- Crear archivo `k8s/namespace.yaml`
- Definir namespace `zero-trust-lab`
- Aplicar: `kubectl apply -f k8s/namespace.yaml`
- Verificar: `kubectl get ns zero-trust-lab`

### Criterios de Aceptación

- Archivo namespace.yaml existe y es válido
- Namespace `zero-trust-lab` está creado
- Archivo sigue estructura YAML correcta

**Story Points:** 1

---

## Issue #20: Subir imágenes a Minikube

**Labels:** `sprint-2`, `type:infrastructure`, `priority:P0`

### Descripción

Hacer que las imágenes Docker estén disponibles en Minikube.

### Tareas

- Usar: `eval $(minikube docker-env)`
- Rebuild imágenes: `docker compose build`
- Verificar: `minikube image ls | grep compose`
- Documentar proceso en README

### Criterios de Aceptación

- Imágenes disponibles: `compose-frontend`, `compose-backend`, `compose-attacker`
- `minikube image ls` muestra las 3 imágenes
- Proceso documentado en README

**Story Points:** 1

---

## Issue #21: Deployment y Service para Frontend

**Labels:** `sprint-2`, `type:infrastructure`, `priority:P0`

### Descripción

Crear manifiestos para desplegar el servicio Frontend.

### Tareas

- Crear `k8s/frontend-deployment.yaml`
  - 1 réplica, imagen `compose-frontend:latest`
  - Puerto 8080, label `app: frontend`
  - runAsNonRoot: true
- Crear `k8s/frontend-service.yaml`
  - Tipo NodePort
- Aplicar y verificar pod Running

### Criterios de Aceptación

- Deployment crea 1 pod frontend
- Pod en estado Running
- Service tipo NodePort funciona
- Accesible: `minikube service frontend -n zero-trust-lab`

**Story Points:** 3

---

## Issue #22: Deployment y Service para Backend

**Labels:** `sprint-2`, `type:infrastructure`, `priority:P0`

### Descripción

Crear manifiestos para desplegar el servicio Backend (solo interno).

### Tareas

- Crear `k8s/backend-deployment.yaml`
  - 1 réplica, imagen `compose-backend:latest`
  - Puerto 5000, label `app: backend`
- Crear `k8s/backend-service.yaml`
  - Tipo ClusterIP
- Aplicar y verificar

### Criterios de Aceptación

- Deployment crea 1 pod backend
- Pod en estado Running
- Service tipo ClusterIP (no expuesto)
- Frontend puede conectarse a backend:5000

**Story Points:** 3

---

## Issue #23: Deployment para Attacker

**Labels:** `sprint-2`, `type:infrastructure`, `priority:P0`

### Descripción

Crear manifiesto para el pod Attacker que ejecutará el scanner.

### Tareas

- Crear `k8s/attacker-deployment.yaml`
  - 1 réplica, imagen `compose-attacker:latest`
  - Command: `["sleep", "infinity"]`
  - Label `app: attacker`
  - Variable: `SCAN_MODE=kubernetes`
- Aplicar y verificar

### Criterios de Aceptación

- Deployment crea 1 pod attacker
- Pod permanece Running
- Se puede ejecutar: `kubectl exec -it <pod> -n zero-trust-lab -- /bin/sh`
- Pod tiene scanner.py en `/app/`

**Story Points:** 2

---

## Issue #24: NetworkPolicy - Default Deny

**Labels:** `sprint-2`, `type:security`, `priority:P1`

### Descripción

Crear NetworkPolicy que bloquea TODO el tráfico por defecto.

### Tareas

- Crear `k8s/networkpolicy-default-deny.yaml`
- Bloquear Ingress y Egress (sin reglas)
- Aplicar y verificar que nada funciona

### Criterios de Aceptación

- Archivo existe
- Política aplicada: `kubectl get networkpolicies -n zero-trust-lab`
- Frontend NO puede conectarse a backend
- Attacker NO puede conectarse a nada

**Story Points:** 2

---

## Issue #25: NetworkPolicy - Allow DNS

**Labels:** `sprint-2`, `type:security`, `priority:P1`

### Descripción

Crear NetworkPolicy que permite resolución DNS.

### Tareas

- Crear `k8s/networkpolicy-allow-dns.yaml`
- Permitir egress a kube-dns puerto 53 (UDP/TCP)
- Aplicar y verificar

### Criterios de Aceptación

- Archivo existe
- Política aplicada
- Pods pueden resolver DNS: `kubectl exec <pod> -- nslookup backend`
- Pero aún NO pueden conectarse (sin ingress)

**Story Points:** 2

---

## Issue #26: NetworkPolicy - Allow Frontend → Backend

**Labels:** `sprint-2`, `type:security`, `priority:P1`

### Descripción

Crear NetworkPolicy que permite SOLO frontend → backend:5000.

### Tareas

- Crear `k8s/networkpolicy-allow-frontend-backend.yaml`
- Permitir ingress en backend desde frontend puerto 5000
- Permitir egress del backend
- Aplicar y verificar

### Criterios de Aceptación

- Archivo existe
- Frontend puede conectarse a backend:5000
- Attacker NO puede conectarse a backend:5000
- Scanner reporta backend CLOSED desde attacker

**Story Points:** 3

---

## Issue #27: NetworkPolicy - Isolate Attacker

**Labels:** `sprint-2`, `type:security`, `priority:P1`

### Descripción

Crear NetworkPolicy que aísla completamente al attacker.

### Tareas

- Crear `k8s/networkpolicy-isolate-attacker.yaml`
- Bloquear todo ingress/egress para app: attacker
- Aplicar y verificar

### Criterios de Aceptación

- Archivo existe
- Attacker NO puede conectarse a backend
- Attacker NO puede conectarse a frontend
- Scanner reporta TODO CLOSED

**Story Points:** 2

---

## Issue #28: Adaptar scanner para Kubernetes

**Labels:** `sprint-2`, `type:feature`, `priority:P1`

### Descripción

Modificar scanner.py para funcionar en Kubernetes.

### Tareas

- Actualizar targets en scanner.py:
  - `backend.zero-trust-lab.svc.cluster.local:5000`
  - `frontend.zero-trust-lab.svc.cluster.local:8080`
- Detectar con variable `SCAN_MODE`
- Probar dentro del pod

### Criterios de Aceptación

- Scanner detecta nombres K8s
- Ejecutable en pod attacker
- Genera JSON válido
- Funciona: `kubectl exec <pod> -- python /app/scanner.py`

**Story Points:** 2

---

## Issue #29: Script k8s-apply.sh

**Labels:** `sprint-2`, `type:automation`, `priority:P1`

### Descripción

Crear script que aplica todos los manifiestos de Kubernetes.

### Tareas

- Crear `scripts/k8s-apply.sh`
- Aplicar en orden: namespace, deployments, services, policies
- Mostrar estado de recursos
- Agregar validaciones básicas

### Criterios de Aceptación

- Script ejecuta sin errores
- Aplica todos los manifiestos
- Muestra estado final
- Funciona: `./scripts/k8s-apply.sh`

**Story Points:** 2

---

## Issue #30: Script k8s-scan.sh

**Labels:** `sprint-2`, `type:automation`, `priority:P1`

### Descripción

Crear script que ejecuta el scanner en el pod attacker.

### Tareas

- Crear `scripts/k8s-scan.sh`
- Obtener nombre del pod automáticamente
- Ejecutar: `kubectl exec <pod> -- python /app/scanner.py`
- Guardar en `reports/k8s-connectivity.json`

### Criterios de Aceptación

- Script ejecuta sin errores
- Genera `reports/k8s-connectivity.json`
- Muestra resumen en terminal
- Funciona: `./scripts/k8s-scan.sh`

**Story Points:** 2

---

## Issue #31: Script k8s-clean.sh

**Labels:** `sprint-2`, `type:automation`, `priority:P2`

### Descripción

Crear script que elimina todos los recursos de Kubernetes.

### Tareas

- Crear `scripts/k8s-clean.sh`
- Eliminar namespace: `kubectl delete ns zero-trust-lab`
- Pedir confirmación antes de eliminar

### Criterios de Aceptación

- Script ejecuta sin errores
- Elimina namespace completo
- Pide confirmación
- Funciona: `./scripts/k8s-clean.sh`

**Story Points:** 1

---

## Issue #32: Script compare-reports.py

**Labels:** `sprint-2`, `type:automation`, `priority:P1`

### Descripción

Crear script que compara reportes de Compose vs K8s.

### Tareas

- Crear `scripts/compare-reports.py`
- Leer `compose-connectivity.json` y `k8s-connectivity.json`
- Calcular: puertos abiertos antes/después, % reducción
- Generar `reports/comparison.json`

### Criterios de Aceptación

- Script lee ambos reportes
- Calcula métricas correctamente
- Genera comparison.json
- Muestra resumen en terminal

**Story Points:** 2

---

## Issue #33: Actualizar Makefile

**Labels:** `sprint-2`, `type:automation`, `priority:P1`

### Descripción

Agregar targets de Kubernetes al Makefile.

### Tareas

- Agregar: `make k8s-apply`, `make k8s-scan`, `make k8s-clean`
- Agregar: `make compare`
- Actualizar `make help`

### Criterios de Aceptación

- Todos los targets funcionan
- `make help` muestra comandos K8s
- Targets llaman a scripts correctos

**Story Points:** 1

---

## Issue #34: Actualizar métricas y Video

**Labels:** `sprint-2`, `type:documentation`, `priority:P0`

### Descripción

Actualizar métricas del Sprint 2 y grabar video demostrativo.

### Tareas de métricas

- Actualizar `docs/metrics.md` con:
  - Throughput, Lead time, WIP
  - Puertos alcanzables antes/después
  - % reducción de superficie
  - Incidencias encontradas

### Tareas de video

- Grabar video 7-10 minutos con:
  - Tablero Kanban (evolución)
  - Demo: `make k8s-apply`, `make k8s-scan`
  - Mostrar NetworkPolicies
  - Comparar reportes: `make compare`
  - Explicar reducción del 100%

### Criterios de Aceptación

- Métricas completas en docs/metrics.md
- Video dura 7-10 minutos
- Demuestra Zero-Trust funcionando
- Compara Compose vs K8s con datos
- Audio claro, pantalla legible
- Video subido y link compartido

**Story Points:** 3

---

## Resumen Sprint 2

**Total:** 16 issues (#19-#34)  
**Story Points:** 30 pts

**Distribución:**

- Persona 1 (Infrastructure): 10 pts - #19, #20, #21, #22, #23
- Persona 2 (Security): 11 pts - #24, #25, #26, #27, #28
- Persona 3 (Automation): 9 pts - #29, #30, #31, #32, #33, #34

**Prioridades:**

- P0 (Críticas): 6 issues
- P1 (Altas): 9 issues
- P2 (Medias): 1 issue

---

## Criterio de Éxito

```
✅ 3 pods corriendo en zero-trust-lab
✅ 4 NetworkPolicies aplicadas
✅ Scanner reporta 0 puertos abiertos desde attacker
✅ Reducción del 100% en superficie de ataque
✅ Video demuestra solución funcionando
```

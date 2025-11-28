# Explicación Detallada de los 3 Servicios

---

## 1. 🎨 Frontend Service

### **¿Qué es y para qué sirve?**

El **Frontend** es la capa de presentación del sistema. En un escenario real, sería la interfaz web que ven los usuarios (como una página de login, dashboard, etc.).

**En este proyecto simula:**
- La "zona pública" de tu aplicación
- El punto de entrada para usuarios externos
- El único servicio que debe ser accesible desde Internet

---

### **📄 `services/frontend/app.py` - Análisis línea por línea**

```python
import socket

from flask import Flask, jsonify
```

**¿Qué hace?**
- `import socket`: Importa la librería para obtener información de red (hostname del contenedor)
- `from flask import Flask, jsonify`: 
  - `Flask`: Framework web minimalista para crear APIs
  - `jsonify`: Convierte diccionarios Python a JSON automáticamente

**¿Por qué Flask?**
- Ligero (imagen Docker pequeña)
- Simple para demostraciones
- Perfecto para APIs REST básicas

---

```python
app = Flask(__name__)
```

**¿Qué hace?**
- Crea una instancia de la aplicación Flask
- `__name__` le dice a Flask dónde buscar templates/archivos estáticos (en este caso no hay)

---

```python
@app.route("/")
def home():
    """Endpoint raíz - verificación de que el servicio funciona"""
    return jsonify(
        {
            "service": "frontend",
            "status": "OK",
            "message": "Frontend service is running",
            "hostname": socket.gethostname(),
        }
    )
```

**¿Qué hace?**
- `@app.route("/")`: Registra la función para la ruta raíz (http://frontend:8080/)
- Devuelve JSON con:
  - `service`: Identifica qué servicio es
  - `status`: Estado de salud básico
  - `message`: Mensaje legible
  - `hostname`: **CLAVE**: El ID del contenedor (cambia cada vez que se reinicia)

**¿Por qué `hostname`?**
- En Docker/K8s, cada contenedor tiene un hostname único
- Te permite verificar que estás hablando con el contenedor correcto
- Útil para debugging en entornos distribuidos

**Salida esperada:**
```json
{
  "service": "frontend",
  "status": "OK",
  "message": "Frontend service is running",
  "hostname": "a3f2b1c9e8d7"
}
```

---

```python
@app.route("/health")
def health():
    """Endpoint de health check"""
    return jsonify({"status": "healthy", "service": "frontend"}), 200
```

**¿Qué hace?**
- Endpoint específico para health checks (usado por Docker, K8s, load balancers)
- Devuelve status code 200 (OK) explícitamente
- Formato estándar de health check

**¿Por qué es importante?**
- Docker/K8s pueden reiniciar automáticamente contenedores "no saludables"
- Load balancers usan esto para saber a qué pods enviar tráfico
- Es una buena práctica en microservicios

**En K8s se usaría así:**
```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
```

---

```python
if __name__ == "__main__":
    # Escuchar en todas las interfaces (0.0.0.0) para que Docker pueda acceder
    # Puerto 8080
    app.run(host="0.0.0.0", port=8080, debug=False)
```

**¿Qué hace?**
- `if __name__ == "__main__"`: Solo ejecuta si el script se corre directamente (no si se importa)
- `host="0.0.0.0"`: **CRÍTICO** - Escucha en TODAS las interfaces de red
- `port=8080`: Puerto donde escucha
- `debug=False`: Desactiva modo debug (producción segura)

**¿Por qué `0.0.0.0` y no `localhost`?**

| host | ¿Funciona en Docker? | Razón |
|------|---------------------|-------|
| `localhost` | ❌ NO | Solo escucha dentro del contenedor |
| `127.0.0.1` | ❌ NO | Solo escucha dentro del contenedor |
| `0.0.0.0` | ✅ SÍ | Escucha en TODAS las interfaces (incluyendo red Docker) |

Sin `0.0.0.0`, otros contenedores no podrían conectarse.

---

### **📦 `services/frontend/requirements.txt`**

```txt
Flask==3.0.0
Werkzeug==3.0.1
```

**¿Por qué estas versiones específicas?**

1. **Flask==3.0.0**: 
   - Versión estable reciente (no latest)
   - Evita cambios inesperados
   - Reproducibilidad: todos usan la misma versión

2. **Werkzeug==3.0.1**:
   - Dependencia de Flask (servidor WSGI)
   - Se especifica explícitamente para evitar incompatibilidades
   - Flask depende de Werkzeug, pero es buena práctica pinear la versión

**¿Por qué NO usar `Flask` (sin versión)?**
```txt
# ❌ MAL
Flask

# ✅ BIEN
Flask==3.0.0
```

Sin versión fija:
- En 6 meses `Flask` podría ser versión 4.0.0
- Tu código podría romper
- No es reproducible

---

### **🐳 `services/frontend/Dockerfile`**

```dockerfile
FROM python:3.12-slim
```

**¿Qué hace?**
- Usa imagen base oficial de Python 3.12
- `-slim`: Versión reducida (~50MB vs ~300MB full)
- No incluye compiladores, herramientas dev innecesarias

**Alternativas:**
| Imagen | Tamaño | Cuándo usar |
|--------|--------|-------------|
| `python:3.12` | ~900MB | Desarrollo con muchas deps |
| `python:3.12-slim` | ~120MB | **Producción (recomendado)** |
| `python:3.12-alpine` | ~50MB | Ultra-ligero, pero puede dar problemas |

---

```dockerfile
WORKDIR /app
```

**¿Qué hace?**
- Crea directorio `/app` dentro del contenedor
- Todos los comandos siguientes se ejecutan ahí

**¿Por qué `/app`?**
- Convención estándar
- Mantiene el código organizado
- Evita contaminar directorios del sistema

---

```dockerfile
# Usuario sin privilegios de root
RUN groupadd -r appuser && useradd -r -g appuser appuser
```

**¿Qué hace?**
- `groupadd -r appuser`: Crea grupo de sistema llamado "appuser"
- `useradd -r -g appuser appuser`: Crea usuario "appuser" en ese grupo
- `-r`: Usuario de sistema (no puede hacer login)

**¿Por qué NO root?**

**Escenario de seguridad:**
```
❌ Corriendo como root:
   Atacante compromete app → Tiene permisos de root en el contenedor
   Puede instalar malware, modificar archivos del sistema

✅ Corriendo como appuser:
   Atacante compromete app → Solo tiene permisos limitados
   No puede modificar archivos del sistema, no puede instalar nada
```

**Es un requisito del PDF:**
> "USER no root en la imagen final"

---

```dockerfile
COPY services/frontend/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
```

**¿Qué hace?**
- Copia `requirements.txt` al contenedor
- Instala dependencias
- `--no-cache-dir`: No guarda cache de pip (reduce tamaño de imagen)

**¿Por qué copiar requirements.txt PRIMERO?**

**Docker usa layers cacheados:**

```dockerfile
# ✅ BIEN (más rápido)
COPY requirements.txt .
RUN pip install -r requirements.txt  # ← Se cachea
COPY app.py .                        # ← Cambios aquí no re-instalan deps

# ❌ MAL (más lento)
COPY . .                             # ← Cambios en cualquier archivo...
RUN pip install -r requirements.txt  # ← ...re-instalan deps (lento)
```

---

```dockerfile
COPY services/frontend/app.py .
RUN chown -R appuser:appuser /app
```

**¿Qué hace?**
- Copia el código de la app
- `chown`: Cambia el dueño de `/app` a `appuser`

**¿Por qué `chown`?**
- Los archivos copiados por `COPY` son propiedad de root
- Si luego corremos como `appuser`, no podría escribir logs, archivos temporales, etc.

---

```dockerfile
USER appuser
```

**¿Qué hace?**
- Cambia a usuario `appuser` para todos los comandos siguientes
- A partir de aquí, el contenedor NO corre como root

---

```dockerfile
EXPOSE 8080
```

**¿Qué hace?**
- **Documentación**: Indica que el contenedor escucha en puerto 8080
- **NO abre el puerto** (eso lo hace `docker-compose.yml` con `ports:`)

**Es solo documentación, pero importante:**
- Otros desarrolladores saben qué puerto usar
- Herramientas como `docker-compose` lo usan

---

```dockerfile
CMD ["python", "app.py"]
```

**¿Qué hace?**
- Define el comando que se ejecuta al iniciar el contenedor
- Ejecuta: `python app.py`

**¿Por qué formato lista `["python", "app.py"]`?**

| Formato | Comportamiento | Cuándo usar |
|---------|---------------|-------------|
| `CMD python app.py` | Ejecuta en shell (`/bin/sh -c`) | Cuando necesitas variables de entorno |
| `CMD ["python", "app.py"]` | Ejecuta directamente (exec) | **Mejor para procesos principales** |

El formato exec:
- ✅ El proceso Python es PID 1 (recibe señales correctamente)
- ✅ Shutdown limpio con `docker stop`
- ✅ Más eficiente

---

### **🔗 Contribución al Zero-Trust**

**Rol del Frontend en la arquitectura:**

```
Internet (público)
     ↓
┌─────────────┐
│  Frontend   │ ← Zona DMZ (DeMilitarized Zone)
│  (8080)     │
└─────────────┘
     ↓
   (debe estar permitido)
     ↓
┌─────────────┐
│  Backend    │ ← Zona interna (datos sensibles)
│  (5000)     │
└─────────────┘
```

**En Zero-Trust:**
1. Frontend es el **único punto de entrada** autorizado
2. Backend **NO debe ser accesible** desde fuera
3. Frontend **debe autenticarse** para hablar con Backend (simulado con NetworkPolicies)

---

## 2. 🔐 Backend Service

### **¿Qué es y para qué sirve?**

El **Backend** es la capa de lógica de negocio. Contiene:
- Datos sensibles
- APIs internas
- Conexión a bases de datos (simulada)

**En este proyecto simula:**
- El servicio que **DEBE ser protegido**
- El objetivo que el atacante intenta alcanzar
- La "joya de la corona" de tu infraestructura

---

### **📄 `services/backend/app.py` - Análisis**

```python
import socket

from flask import Flask, jsonify

app = Flask(__name__)


@app.route("/")
def home():
    """Endpoint raíz - verificación de que el servicio funciona"""
    return jsonify(
        {
            "service": "backend",
            "status": "OK",
            "message": "Backend service is running",
            "hostname": socket.gethostname(),
        }
    )
```

**Igual que Frontend, pero:**
- `"service": "backend"`: Identifica como backend
- Ayuda a verificar que estás hablando con el servicio correcto

---

```python
@app.route("/health")
def health():
    """Endpoint de health check"""
    return jsonify({"status": "healthy", "service": "backend"}), 200
```

**Mismo propósito que Frontend.**

---

```python
@app.route("/api/data")
def get_data():
    """Endpoint simulado con datos sensibles"""
    return jsonify(
        {
            "data": "sensitive information",
            "message": "This endpoint should be protected",
        }
    )
```

**¿Qué hace?**
- Endpoint que simula **datos sensibles** (ej: info de usuarios, transacciones, etc.)
- El mensaje es un recordatorio: **este endpoint es crítico**

**En Zero-Trust:**
- Este endpoint **NO debe ser accesible** para el attacker
- Solo Frontend puede llamarlo (después de autenticarse)
- En producción real, tendría tokens JWT, OAuth, etc.

**Demostración del problema:**
```bash
# Sin NetworkPolicies (Sprint 1):
curl backend:5000/api/data
# ✅ Responde (PROBLEMA: attacker puede acceder)

# Con NetworkPolicies (Sprint 2):
curl backend:5000/api/data
# ❌ Timeout (CORRECTO: attacker bloqueado)
```

---

```python
if __name__ == "__main__":
    # Escuchar en todas las interfaces (0.0.0.0)
    # Puerto 5000
    app.run(host="0.0.0.0", port=5000, debug=False)
```

**Cambio clave: puerto 5000**
- Frontend: 8080 (común para servicios web públicos)
- Backend: 5000 (común para APIs internas)

---

### **📦 `services/backend/requirements.txt`**

```txt
Flask==3.0.0
Werkzeug==3.0.1
```

**Exactamente igual que Frontend.**

**¿Por qué no agregar más dependencias?**
- En un proyecto real, backend tendría: SQLAlchemy, Redis, Celery, etc.
- Para MVP educativo: lo mínimo funcional

---

### **🐳 `services/backend/Dockerfile`**

**Idéntico al Frontend**, con estos cambios:

```dockerfile
COPY services/backend/requirements.txt .
COPY services/backend/app.py .
EXPOSE 5000
```

**Cambios:**
- Copia archivos de `services/backend/`
- Expone puerto 5000 (en vez de 8080)

---

### **🔗 Contribución al Zero-Trust**

**Rol del Backend:**

```
Escenario SIN Zero-Trust (Sprint 1):
  Attacker entra → backend:5000 OPEN → ❌ Accede a datos sensibles

Escenario CON Zero-Trust (Sprint 2):
  Attacker entra → NetworkPolicy BLOQUEA → ✅ No puede acceder
  Frontend → NetworkPolicy PERMITE → ✅ Sí puede acceder
```

**Backend es el "target" que estamos protegiendo.**

---

## 3. 🔴 Attacker Service (Scanner)

### **¿Qué es y para qué sirve?**

El **Attacker** simula un actor malicioso que:
- Entró a tu red (por phishing, vulnerabilidad, etc.)
- Intenta descubrir qué servicios están disponibles
- Intenta conectarse a servicios internos

**En este proyecto:**
- Es una herramienta de **auditoría de red**
- Demuestra qué pasa **antes** y **después** de aplicar Zero-Trust
- Genera evidencia (JSON) de conectividad

---

### **📄 `services/attacker/scanner.py` - Análisis completo**

```python
#!/usr/bin/env python3
# services/attacker/scanner.py
"""
Network Scanner - Zero Trust Network Sandbox
Escanea servicios para validar segmentacion de red.
"""
```

**Comentarios:**
- `#!/usr/bin/env python3`: Shebang para ejecutar directamente (`./scanner.py`)
- Docstring explica propósito del script

---

```python
import json
import socket
import sys
from datetime import datetime
```

**¿Qué importa?**
- `json`: Para generar reportes JSON
- `socket`: Para intentar conexiones TCP
- `sys`: Para escribir a stderr
- `datetime`: Para timestamps en el reporte

---

```python
def scan_target(host, port, timeout=2):
    """
    Intenta conectarse a un target especifico.

    Args:
        host (str): Hostname del target
        port (int): Puerto a escanear
        timeout (int): Timeout en segundos

    Returns:
        str: Estado de la conexion
    """
```

**Función principal del scanner.**

**¿Qué hace?**
- Intenta conectarse a `host:port`
- Devuelve el estado de la conexión

---

```python
    try:
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.settimeout(timeout)
        result = sock.connect_ex((host, port))
        sock.close()

        if result == 0:
            return "OPEN"
        else:
            return "CLOSED"
```

**¿Cómo funciona?**

1. `socket.socket(socket.AF_INET, socket.SOCK_STREAM)`:
   - Crea un socket TCP/IP
   - `AF_INET`: IPv4
   - `SOCK_STREAM`: TCP (en vez de UDP)

2. `sock.settimeout(timeout)`:
   - Espera máximo 2 segundos
   - Evita que el scanner se quede colgado

3. `sock.connect_ex((host, port))`:
   - Intenta conectarse
   - Devuelve 0 si éxito, otro número si falla
   - **No lanza excepción** (por eso `_ex`)

4. `if result == 0`:
   - 0 = puerto abierto (servicio escuchando)
   - Otro valor = puerto cerrado/filtrado

**Estados posibles:**
- `OPEN`: Puerto abierto, servicio responde
- `CLOSED`: Puerto cerrado o NetworkPolicy bloqueó
- `ERROR-DNS`: Hostname no existe
- `TIMEOUT`: No responde en 2 segundos
- `ERROR`: Otro error (red caída, etc.)

---

```python
    except socket.gaierror:
        return "ERROR-DNS"
    except socket.timeout:
        return "TIMEOUT"
    except Exception:
        return "ERROR"
```

**Manejo de errores:**
- `socket.gaierror`: Error DNS (hostname no existe)
- `socket.timeout`: Timeout (2 segundos sin respuesta)
- `Exception`: Cualquier otro error

---

```python
def main():
    """Ejecuta el escaneo y genera reporte JSON."""

    # Targets a escanear
    targets = [
        ("frontend", 8080, "Frontend Flask"),
        ("backend", 5000, "Backend API"),
        ("backend", 5432, "Backend DB (simulado)"),
    ]
```

**Lista de targets:**

| Host | Puerto | Descripción | ¿Por qué escanearlo? |
|------|--------|-------------|----------------------|
| `frontend` | 8080 | Flask frontend | Ver si attacker puede acceder a zona pública |
| `backend` | 5000 | API interna | **CRÍTICO**: Ver si attacker puede acceder |
| `backend` | 5432 | DB simulada | Puerto típico de PostgreSQL (no hay nada escuchando) |

**¿Por qué `backend:5432`?**
- Simula un puerto de base de datos
- En un escenario real, sería PostgreSQL
- Demuestra que el scanner detecta puertos cerrados

---

```python
    # Mensaje de inicio en stderr
    print("=" * 60, file=sys.stderr)
    print("Network Scanner - Zero Trust Network Sandbox", file=sys.stderr)
    print("=" * 60, file=sys.stderr)
    print(f"Fecha: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}", file=sys.stderr)
    print(f"Total de targets: {len(targets)}", file=sys.stderr)
    print("", file=sys.stderr)
```

**¿Por qué `file=sys.stderr`?**

**Separación de outputs:**
```
stdout → Solo JSON (fácil de parsear)
stderr → Mensajes informativos (logs, progress)
```

**Sin esto:**
```bash
docker exec attacker python scanner.py > report.json
# ❌ report.json contiene mensajes mezclados con JSON (no es válido)
```

**Con stderr:**
```bash
docker exec attacker python scanner.py > report.json
# ✅ report.json solo tiene JSON válido
# Mensajes informativos aparecen en terminal
```

---

```python
    results = []
    open_count = 0
    closed_count = 0
    error_count = 0

    for host, port, description in targets:
        target_str = f"{host}:{port}"
        print(
            f"Escaneando {target_str:25} ({description})...", end=" ", file=sys.stderr
        )

        status = scan_target(host, port)

        result = {
            "target": target_str,
            "host": host,
            "port": port,
            "description": description,
            "status": status,
        }

        results.append(result)
```

**¿Qué hace?**
- Itera sobre cada target
- Imprime progreso en tiempo real
- Llama a `scan_target()` para cada uno
- Guarda resultado en lista

**Output en terminal:**
```
Escaneando frontend:8080        (Frontend Flask)... [OK] OPEN
Escaneando backend:5000         (Backend API)... [OK] OPEN
Escaneando backend:5432         (Backend DB)... [X] CLOSED
```

---

```python
        # Contadores
        if status == "OPEN":
            open_count += 1
            symbol = "[OK]"
        elif status in ["CLOSED", "TIMEOUT"]:
            closed_count += 1
            symbol = "[X]"
        else:
            error_count += 1
            symbol = "[!]"

        print(f"{symbol} {status}", file=sys.stderr)
```

**Símbolos visuales:**
- `[OK]` → Puerto abierto
- `[X]` → Puerto cerrado/bloqueado
- `[!]` → Error (DNS, red caída, etc.)

---

```python
    print("", file=sys.stderr)
    print("=" * 60, file=sys.stderr)
    print("RESUMEN DEL ESCANEO", file=sys.stderr)
    print("=" * 60, file=sys.stderr)
    print(f"Puertos abiertos:    {open_count}", file=sys.stderr)
    print(f"Puertos cerrados:    {closed_count}", file=sys.stderr)
    print(f"Errores:             {error_count}", file=sys.stderr)
    print("=" * 60, file=sys.stderr)
```

**Resumen en terminal:**
```
============================================================
RESUMEN DEL ESCANEO
============================================================
Puertos abiertos:    2
Puertos cerrados:    1
Errores:             0
============================================================
```

---

```python
    # Generar reporte JSON
    report = {
        "scan_date": datetime.now().isoformat(),
        "scanner": "attacker",
        "environment": "docker-compose",
        "total_targets": len(targets),
        "summary": {"open": open_count, "closed": closed_count, "errors": error_count},
        "results": results,
    }

    # Output JSON a stdout
    print(json.dumps(report, indent=2))
```

**Genera JSON final:**

```json
{
  "scan_date": "2024-11-26T18:30:45.123456",
  "scanner": "attacker",
  "environment": "docker-compose",
  "total_targets": 3,
  "summary": {
    "open": 2,
    "closed": 1,
    "errors": 0
  },
  "results": [...]
}
```

**Campo clave: `environment`**
- En Compose: `"environment": "docker-compose"`
- En K8s: `"environment": "kubernetes"`
- Permite distinguir reportes en Sprint 1 vs Sprint 2

---

```python
if __name__ == "__main__":
    main()
```

**Solo ejecuta si se corre directamente.**

---

### **📦 `services/attacker/requirements.txt`**

```txt
# (vacío)
```

**¿Por qué vacío?**
- Solo usa librerías estándar de Python: `socket`, `json`, `sys`, `datetime`
- No necesita dependencias externas
- Mantiene la imagen ligera

---

### **🐳 `services/attacker/Dockerfile`**

```dockerfile
FROM python:3.12-slim

WORKDIR /app

RUN groupadd -r appuser && useradd -r -g appuser appuser

COPY services/attacker/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY services/attacker/scanner.py .
RUN chown -R appuser:appuser /app

USER appuser

# Mantener contenedor activo para ejecutar scanner cuando se necesite
CMD ["tail", "-f", "/dev/null"]
```

**Diferencias clave:**

1. **No hay `EXPOSE`:**
   - Attacker no escucha en ningún puerto
   - Solo hace conexiones salientes

2. **`CMD ["tail", "-f", "/dev/null"]`:**
   - Mantiene el contenedor corriendo indefinidamente
   - Sin esto, el contenedor se apagaría inmediatamente
   - `tail -f /dev/null` es un truco común: nunca termina

**¿Por qué necesita estar corriendo?**
- Ejecutamos el scanner **manualmente** con `docker exec`
- El contenedor debe estar vivo para poder ejecutar comandos dentro

---

### **🔗 Contribución al Zero-Trust**

**El Attacker es el "ojo de la auditoría":**

**Sprint 1 (Sin Zero-Trust):**
```bash
make compose-scan

# Output:
Escaneando backend:5000... [OK] OPEN  ← ❌ PROBLEMA
```

**Sprint 2 (Con Zero-Trust):**
```bash
make k8s-scan

# Output:
Escaneando backend:5000... [X] CLOSED  ← ✅ BLOQUEADO
```

**Demuestra visualmente el impacto de NetworkPolicies.**

---

## 🎯 Resumen de Cómo los 3 Servicios Implementan Zero-Trust

### **Sin Zero-Trust (Sprint 1):**

```
┌─────────────┐
│  Attacker   │─────┐
└─────────────┘     │
                    ├──> backend:5000 ✅ OPEN (PROBLEMA)
┌─────────────┐     │
│  Frontend   │─────┘
└─────────────┘
```

**Problema:** Attacker y Frontend tienen el mismo acceso.

---

### **Con Zero-Trust (Sprint 2):**

```
┌─────────────┐
│  Attacker   │─────X──> backend:5000 ❌ CLOSED (NetworkPolicy)
└─────────────┘

┌─────────────┐
│  Frontend   │─────✓──> backend:5000 ✅ OPEN (Permitido explícitamente)
└─────────────┘
```

**Solución:** NetworkPolicies permiten **solo** tráfico autorizado.

---

## 📋 Tabla Comparativa Final

| Servicio | Puerto | Accesible desde fuera | Rol en Zero-Trust | Endpoints clave |
|----------|--------|----------------------|-------------------|-----------------|
| **Frontend** | 8080 | ✅ SÍ (NodePort) | Zona pública | `/`, `/health` |
| **Backend** | 5000 | ❌ NO (ClusterIP) | Zona protegida | `/`, `/health`, `/api/data` |
| **Attacker** | N/A | ❌ NO | Auditor/Red Team | N/A (solo scanner.py) |

---

¿Quieres que ahora te explique cómo se comunican entre sí en las redes de Docker Compose?

Este es un contexto crucial, ya que los laboratorios de seguridad como el que planean siempre se originan a partir de un fracaso en la seguridad tradicional.

El proyecto "Zero-Trust Network Sandbox" se justifica por la necesidad de mitigar un vector de ataque que se hizo tristemente famoso: el **Movimiento Lateral** (Lateral Movement) dentro de una red que confía implícitamente en sus componentes internos.

El caso de la vida real más emblemático para justificar la necesidad de la microsegmentación y el Zero Trust es el **ataque a Colonial Pipeline en 07/05/2021**.

Link 1: https://www.rockwellautomation.com/es-mx/company/news/articles/lecciones-del-ciberataque-a-colonial-pipeline.html

Link 2: https://insurica.com/blog/colonial-pipeline-ransomware-attack/
---

## 1. Acontecimiento: El Ataque a Colonial Pipeline (2021)

### 📌 ¿A raíz de qué acontecimiento se originó la necesidad de este laboratorio?

La necesidad de un laboratorio Zero-Trust se disparó en industrias críticas a raíz de ataques de *ransomware* dirigidos a la infraestructura que demostraron que, si bien la seguridad perimetral es importante, la **seguridad interna (East-West)** es la debilidad más explotada.

El caso de **Colonial Pipeline** (el oleoducto más grande de EE. UU.) es un ejemplo perfecto de cómo una pequeña brecha de seguridad se convierte en una catástrofe debido a la falta de Zero Trust.

### 📉 La Cadena del Fracaso de la Confianza Implícita

1.  **El Punto de Entrada (La Brecha Inicial):** Los atacantes (DarkSide) obtuvieron acceso inicial utilizando una sola cuenta de VPN que no estaba protegida con autenticación multifactor (MFA) y que, irónicamente, se creía que ya no estaba en uso. **Este fue un fallo de identidad/acceso.**
2.  **El Movimiento Lateral (La Propagación):** Una vez dentro de la red corporativa, el atacante no encontró restricciones. Pudo moverse libremente (lateralmente) desde el servidor VPN, navegar por los sistemas internos, encontrar credenciales adicionales y finalmente acceder a los sistemas operativos de facturación.
3.  **El Impacto Crítico:** El atacante desplegó el *ransomware* en estos sistemas críticos. Aunque el sistema de control del oleoducto (*OT network*) no fue atacado directamente, la empresa tuvo que detener completamente la operación del oleoducto (que transporta el 45% del combustible de la costa este de EE. UU.) porque el sistema de facturación y pagos estaba comprometido, haciendo imposible la gestión comercial.

### 👤 ¿Quién se vio afectado?

* **Colonial Pipeline:** Sufrió la interrupción de sus operaciones, pagó un rescate millonario y enfrentó multas regulatorias y un daño masivo a su reputación.
* **Millones de ciudadanos:** El paro del oleoducto causó escasez de combustible, pánico y aumentos de precio en la costa este de Estados Unidos. Se clasificó como un ataque a la infraestructura crítica nacional.

---

## 2. La Solución: Zero Trust y el Proyecto 14

### 💡 ¿Cómo se soluciona con las buenas prácticas de este proyecto?

El proyecto "Zero-Trust Network Sandbox (Compose + K8s)" es una implementación directa de la práctica que habría contenido el ataque a Colonial Pipeline: **la Microsegmentación**.

| Elemento del Proyecto | Práctica Zero Trust Mitigadora | Resultado en el Ataque Real |
| :--- | :--- | :--- |
| **NetworkPolicies en K8s** | **Microsegmentación "Denegar por Defecto"** | Habría asegurado que la cuenta comprometida, incluso si estaba dentro de la red corporativa, solo pudiera hablar con un número muy limitado de otros servicios (mínimo privilegio). |
| **Docker Compose / Sandbox** | **Aislamiento de Cargas de Trabajo (Workload Isolation)** | Habría confinado al atacante al segmento de red inicial (el "sandbox") y le habría impedido alcanzar los servidores de facturación y control. |
| **Pequeño "Escáner" en Python** | **Simulación de Movimiento Lateral** | Este escáner (al intentar comunicarse con servicios no autorizados) habría fallado la prueba, demostrando que las NetworkPolicies están funcionando y que el movimiento lateral está *imposibilitado* por diseño. |

En resumen, la filosofía del Zero Trust es: **Asume que el atacante ya está adentro.**

Si Colonial Pipeline hubiese tenido sus sistemas segmentados (Ej: Sistemas de Facturación en un segmento aislado del resto de la red corporativa), el atacante que entró por la VPN habría sido detenido por una "política de red" al intentar comunicarse desde el segmento de "Acceso Remoto" hacia el segmento de "Facturación". La brecha habría quedado contenida, minimizando el daño y evitando la paralización del oleoducto.

**Estudiar este proyecto es estudiar cómo evitar que un fallo de seguridad menor se convierta en una catástrofe nacional a través de la segregación estricta del tráfico interno.**

# IR-001 · Ejecución de implante Sliver C2 en WS01

## Resumen

Se generó y ejecutó un implante de Sliver C2 (framework de post-explotación de código abierto) sobre la estación de trabajo `ws01`, miembro del dominio `crisol.lab`. El objetivo del ejercicio era comprobar si Wazuh, desplegado como agente en `ws01`, detecta la ejecución de un implante de C2 y el establecimiento del canal de mando y control, en una configuración "de fábrica" (sin fuentes de telemetría adicionales).

**Resultado:** Wazuh no generó ninguna alerta asociada a la ejecución del implante ni al establecimiento del canal C2. Sí se registraron, en ambas pruebas, los eventos de inicio de sesión local en Windows inmediatamente anteriores a la ejecución, pero ningún evento relacionado con la creación del proceso del implante ni con la conexión de red saliente hacia el servidor de C2.

## Entorno

| Elemento | Detalle |
|---|---|
| Objetivo | `ws01` (192.168.100.50) — Windows 10 Pro, miembro de `crisol.lab` |
| Atacante | `atacante` (192.168.100.30) — Kali Linux, Sliver v1.7.9 |
| Framework C2 | Sliver (listener mTLS, puerto 8888) |
| SIEM | Wazuh (agente instalado en `ws01`, configuración por defecto, sin Sysmon) |

## Cronología

| Hora (CET) | Evento |
|---|---|
| 04/11/2026 18:20:48 | Primera sesión Sliver abierta contra `ws01` (`CLOUDY_ADVERTISEMENT`). El servidor Wazuh estaba apagado en este momento. |
| 04/11/2026 18:39:42 | Inicio de sesión local en `ws01` previo a la segunda prueba (con el servidor Wazuh ya encendido). |
| 04/11/2026 18:39:55 | Segunda sesión Sliver abierta contra `ws01`, con el servidor Wazuh activo durante todo el proceso. |

La segunda prueba es la relevante para las conclusiones de este informe, ya que elimina la variable de que el manager estuviera apagado.

## Análisis en Wazuh

Búsqueda realizada en el módulo *Threat Hunting → Events*, filtrando por `agent.name: ws01`, en la ventana 18:39:30–18:41:00 (centrada en la apertura de la segunda sesión).

**Eventos encontrados (3 hits, todos a las 18:39:42, trece segundos antes de abrirse la sesión):**

| Hora | `rule.description` | `rule.level` | `rule.id` |
|---|---|---|---|
| 18:39:42.479 | Non service account logged off. | 3 | 67023 |
| 18:39:42.479 | Special privileges assigned to new logon. | 3 | 67028 |
| 18:39:42.467 | Non network or service local logon. | 3 | 67022 |

Entre las 18:39:55 (apertura de la sesión Sliver) y las 18:41:00, **no se registró ningún evento adicional**.

## Interpretación

Los tres eventos detectados corresponden al **inicio de sesión de Windows** (categoría de autenticación, reglas 670xx), es decir, al propio hecho de iniciar sesión en `ws01` para poder lanzar el archivo — no a la ejecución del implante en sí.

A partir de ese punto, dos acciones concretas quedaron completamente sin detectar:

1. **La creación del proceso del implante** (ejecución del binario de Sliver en el sistema)
2. **La conexión de red saliente** hacia el listener mTLS en `atacante` (192.168.100.30:8888)

La causa no es un fallo de Wazuh como producto, sino de **cobertura de fuentes de datos**: la configuración actual del agente en `ws01` recoge el *Security Event Log* de Windows (de ahí los eventos de logon), pero no tiene activada ninguna fuente que registre creación de procesos (por ejemplo, Sysmon Event ID 1 + Event ID 4688 de auditoría de procesos) ni conexiones de red salientes (Sysmon Event ID 3). Sin esa telemetría, no hay datos que una regla de Wazuh pueda analizar, por muy bien escrita que esté.

## Mapeo MITRE ATT&CK

| Táctica | Técnica | ID | Detectado |
|---|---|---|---|
| Execution | User Execution: Malicious File | T1204.002 | No |
| Command and Control | Application Layer Protocol | T1071 | No |
| Command and Control | Encrypted Channel: Asymmetric Cryptography | T1573.002 | No |

## Recomendación

Para cerrar esta brecha de visibilidad sería necesario:

1. Instalar **Sysmon** en `ws01` con una configuración que registre, como mínimo, creación de procesos (Event ID 1) y conexiones de red (Event ID 3).
2. Configurar Wazuh para ingerir el canal de eventos de Sysmon (`Microsoft-Windows-Sysmon/Operational`).
3. Habilitar o ajustar reglas de Wazuh (propias o del conjunto de reglas de Sysmon de la comunidad) que generen alertas ante procesos poco habituales o conexiones salientes a puertos no estándar.

Este es el siguiente paso natural antes de repetir el ejercicio y comprobar si, con esa telemetría añadida, la ejecución del implante sí se detecta.

## Nota metodológica

Este ejercicio simplifica el acceso inicial: se parte del supuesto de que el atacante ya dispone del implante en la máquina objetivo, sin simular el vector de entrega (phishing, USB, etc.), que queda fuera del alcance de este informe.

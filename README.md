# AD Purple Team Lab

Un dominio de Active Directory, atacado con las mismas herramientas que usa un red team real, y validado contra el SIEM del proyecto anterior: ¿qué se detecta, y qué no?

## Objetivos

- Montar un dominio Active Directory como objetivo (controlador de dominio + estación de trabajo)
- Simular un ataque realista con un framework de C2 moderno (Sliver) y mapeo de dominio con BloodHound
- Escalar privilegios hasta comprometer el controlador de dominio (Kerberoasting → DCSync)
- Comprobar, técnica por técnica, si Wazuh (reutilizado del proyecto `wazuh-siem-lab`) lo detecta
- Gestionar cada hallazgo como un caso real en TheHive
- Documentar todo mapeado a MITRE ATT&CK, como se haría en un ejercicio de emulación de adversario profesional

## Entorno

| Componente | Herramienta |
|---|---|
| Dominio objetivo | Active Directory (Windows Server) — `crisol.lab` |
| Framework de C2 | Sliver |
| Mapeo del dominio | BloodHound |
| Detección | Wazuh (reutilizado) |
| Gestión de casos | TheHive |
| Marco de referencia | MITRE ATT&CK |
| Hipervisor | VMware Workstation |

## Máquinas del laboratorio

| Máquina | IP | Rol | Estado |
|---|---|---|---|
| `dc01` | 192.168.100.40 | Controlador de dominio (`crisol.lab`) | Activo |
| `ws01` | 192.168.100.50 | Estación de trabajo del dominio | Pendiente |
| `wazuh-server` | 192.168.100.10 | SIEM (reutilizado) | Activo |
| `atacante` | 192.168.100.30 | Máquina de ataque (reutilizada) | Activo |

Todas en la red aislada `VMnet10` (192.168.100.0/24), sin salida a internet salvo conexión temporal y puntual cuando hace falta.

## Fases del proyecto

| Fase | Contenido | Estado |
|---|---|---|
| Fase 0 | Repositorio y estructura | Completado |
| Fase 1 | Dominio AD objetivo (DC01 + estación de trabajo) | En curso — DC01 creado, promovido a controlador de dominio y enrolado como agente de Wazuh; falta WS01 |
| Fase 2 | Acceso inicial y C2 con Sliver | Pendiente |
| Fase 3 | Reconocimiento del dominio con BloodHound | Pendiente |
| Fase 4 | Escalada: Kerberoasting → DCSync | Pendiente |
| Fase 5 | Validación purple team contra Wazuh | Pendiente |
| Fase 6 | Gestión de casos en TheHive | Pendiente |
| Fase 7 | Informe final y heatmap de MITRE ATT&CK | Pendiente |

## Estructura del repositorio

- `docs/`: guía paso a paso de cada fase
- `reports/`: plan de emulación de adversario e informe final
- `scripts/`: automatizaciones puntuales en Python

## Aviso

Todas las pruebas se realizan en un laboratorio aislado y propio, sobre máquinas creadas para este proyecto.

## Autor

Abdelhamid Boughafour · LinkedIn

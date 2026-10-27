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
| Dominio objetivo | Active Directory (Windows Server) |
| Framework de C2 | Sliver |
| Mapeo del dominio | BloodHound |
| Detección | Wazuh (reutilizado) |
| Gestión de casos | TheHive |
| Marco de referencia | MITRE ATT&CK |
| Hipervisor | VMware Workstation |

## Fases del proyecto

| Fase | Contenido | Estado |
|---|---|---|
| Fase 0 | Repositorio y estructura | En curso |
| Fase 1 | Dominio AD objetivo (DC01 + estación de trabajo) | Pendiente |
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

# IR-002 · Detección estática del implante Sliver por Microsoft Defender en WS01

## Resumen

Durante la puesta en marcha del canal de C2 sobre `ws01`, se generó un nuevo implante de Sliver (`OCCUPATIONAL_ELONGATION.exe`, Windows/amd64, con ofuscación de símbolos activada) y se transfirió a la máquina objetivo. **Microsoft Defender**, activo en `ws01`, detectó el binario en reposo como `Trojan:Win32/Gracing.I` y lo eliminó automáticamente **antes de que llegara a ejecutarse**.

**Resultado:** La detección estática por firma de Microsoft Defender fue eficaz. El implante fue identificado y puesto en cuarentena sin ejecutarse (`DidThreatExecute: False`), con severidad máxima. La ofuscación de símbolos de Sliver —que incrementa notablemente el tiempo de compilación— no evitó la detección. Para poder continuar con el objetivo del laboratorio (análisis de detección de comportamiento con Wazuh), se añadió una exclusión de ruta en Defender, tras lo cual el implante se ejecutó y estableció sesión.

Este informe complementa a [IR-001](IR-001-ejecucion-implante-sliver.md): donde el SIEM de comportamiento (Wazuh, configuración por defecto) no generó alertas por falta de telemetría, el antivirus por firmas (Defender) sí identificó el binario.

## Entorno

| Elemento | Detalle |
|---|---|
| Objetivo | `ws01` (192.168.100.50) — Windows 10 Pro, miembro de `crisol.lab` |
| Atacante | `atacante` (192.168.100.30) — Kali Linux, Sliver v1.7.9 |
| Framework C2 | Sliver (listener mTLS, puerto 8888) |
| Implante | Sliver mTLS, Windows/amd64, ofuscación de símbolos activada |
| Capa de detección | Microsoft Defender Antivirus (protección en tiempo real, en `ws01`) |

## Cronología (26/11/2026)

| Hora (CET) | Evento |
|---|---|
| ~19:01 | Generación del implante `OCCUPATIONAL_ELONGATION.exe` en `atacante` (compilación con ofuscación de símbolos). |
| ~19:12 | Transferencia del implante a `ws01` vía HTTP (descarga correcta, `200 OK`, a `C:\Users\Public\oe.exe`). |
| ~19:12 | Microsoft Defender elimina el binario automáticamente. `Test-Path` sobre el fichero devuelve `False`. |
| 19:19 | `Get-MpThreat` confirma la detección (`Trojan:Win32/Gracing.I`, severidad 5, `DidThreatExecute: False`). |
| ~19:23 | Se añade una exclusión de ruta en Defender (`C:\Users\Public`) y se re-descarga el implante. `Test-Path` devuelve `True`. |
| 19:24:36 | El implante se ejecuta y establece sesión mTLS contra `atacante` (`192.168.100.50:50217`). |

## Evidencia

Salida relevante de `Get-MpThreat` en `ws01`:

| Campo | Valor |
|---|---|
| `ThreatName` | `Trojan:Win32/Gracing.I` |
| `ThreatID` | `2147912911` |
| `SeverityID` | `5` (Grave) |
| `DidThreatExecute` | `False` |
| `Resources` | `file:_C:\Users\Public\oe.exe` |

`Test-Path C:\Users\Public\oe.exe` devolvió `False` tras la descarga (fichero eliminado) y `True` tras añadir la exclusión y re-descargar.

## Interpretación

Microsoft Defender identificó el implante mediante **detección estática basada en firmas**, sin necesidad de que el binario se ejecutara. La familia `Gracing` es con la que Defender reconoce implantes de Sliver, de modo que el producto no solo marcó el fichero como malicioso, sino que acertó con el framework concreto.

Dos observaciones son relevantes para el proyecto:

1. **Capas complementarias con resultados opuestos.** En [IR-001](IR-001-ejecucion-implante-sliver.md), Wazuh (detección de comportamiento, configuración por defecto, sin Sysmon) no generó ninguna alerta por la ejecución del implante ni por el canal C2, por falta de telemetría de procesos y red. Aquí, en cambio, el antivirus por firmas sí bloqueó el binario en reposo. Es un buen ejemplo de por qué la defensa se construye por capas: ninguna cubre todo por sí sola.

2. **La ofuscación de símbolos no bastó.** Sliver ofusca el binario para dificultar su análisis, pero la firma de Defender lo detectó igualmente. Un implante por defecto, aun ofuscado, sigue siendo reconocible para un AV actualizado.

Contraste con la primera prueba: el implante original (`CLOUDY_ADVERTISEMENT`, 04/11, [IR-001](IR-001-ejecucion-implante-sliver.md)) llegó a ejecutarse y establecer sesión sin ser bloqueado por el AV, mientras que el implante regenerado (26/11) fue detectado. La diferencia es atribuible, probablemente, a la actualización de firmas de Defender entre ambas fechas y a la variabilidad entre compilaciones (*hipótesis no verificada en este ejercicio*).

## Mapeo MITRE ATT&CK

| Táctica | Técnica | ID | Detectado |
|---|---|---|---|
| Execution | User Execution: Malicious File | T1204.002 | Sí (AV, detección estática por firma) |

Nota: el framework utilizado, **Sliver**, está catalogado en MITRE ATT&CK como software **S0633**.

## Recomendación

1. La detección por firmas es **evadible** (ofuscación avanzada, cifrado, implantes personalizados o ejecución en memoria sin tocar disco, p. ej. `execute-assembly`). Por tanto, el AV **no sustituye** a la detección de comportamiento, sino que la complementa.
2. Se reafirma la recomendación de [IR-001](IR-001-ejecucion-implante-sliver.md): desplegar **Sysmon + Wazuh** para cubrir creación de procesos y conexiones de red, que es la capa que resiste frente a variantes que evaden el AV.
3. En un ejercicio de emulación de adversario realista, el atacante regeneraría u ofuscaría el implante, o lo ejecutaría en memoria, para evadir el AV. En este laboratorio se optó por una exclusión de ruta para aislar esa variable y centrar el análisis en la detección de comportamiento.

## Nota metodológica

La exclusión de Defender se añadió **deliberadamente** para permitir la continuación del ejercicio y **no representa una capacidad del atacante**, sino una decisión de laboratorio para aislar la capa que se está analizando. La detección de Defender, sin embargo, es un hallazgo legítimo a favor del defensor y queda documentada como tal.

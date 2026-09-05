# Inventario técnico del laboratorio LZOC — V1

## Estado

- **Fase LZOC-TERRA:** Fase 1 — Inventario técnico del laboratorio.
- **Estado del documento:** en construcción.
- **Baseline funcional inspeccionado:** `HikariLucy/LZOC-Orquestador` @ `4dbf8c96e66904103a5ecc4bb5469e1d9a391d78`.
- **Objetivo:** convertir el laboratorio actualmente validado en requisitos concretos para Terraform y Ansible sin rediseñar la lógica funcional de LZOC.

> Regla: cuando un dato de capacidad no esté verificado, se registra como **PENDIENTE DE MEDICIÓN**. No se inventan tamaños de VM ni requisitos mínimos.

---

## 1. Topología objetivo vigente

La arquitectura del Proyecto de Título se organiza en tres nodos conectados mediante Tailscale mientras ese overlay siga siendo parte de la arquitectura vigente:

| Nodo | Responsabilidad principal | Componentes objetivo | Estado IaC |
|---|---|---|---|
| `LAB-JESUS` | Plataforma, orquestación y servicios de gestión | LZOC A/B, HAProxy, PostgreSQL, Redis, Zabbix, LibreNMS, NetBox, Zammad, Mailpit, Prometheus, Grafana, OpenTelemetry, Tempo, Loki, Alloy, Keycloak | Inventariando |
| `SERV-A` | Seguridad/analítica | Wazuh, Ollama, GIPSIK | Parcial / pendiente de completar |
| `SERV-B` | Sensorización y red de laboratorio | Zeek, Suricata, OPNsense, clientes de laboratorio | Pendiente de infraestructura completa |

Red de clientes objetivo vigente: `10.10.20.0/24`.

La distribución física anterior es una **topología objetivo**. El `docker-compose.yml` actual de LZOC-Orquestador mantiene varios de estos componentes en un mismo host para desarrollo y demostración. LZOC-TERRA deberá poder representar ambos escenarios sin mezclar la lógica de aplicación con el aprovisionamiento.

---

## 2. Baseline de despliegue de LZOC-Orquestador

### 2.1 Núcleo de orquestación

| Servicio | Imagen / implementación | Puerto publicado | Dependencias | Persistencia | Criticidad |
|---|---|---:|---|---|---|
| `lzoc-a` | build local Python/FastAPI | No directo | PostgreSQL, Redis | Datos en servicios externos | Crítica |
| `lzoc-b` | build local Python/FastAPI | No directo | PostgreSQL, Redis | Datos en servicios externos | Crítica |
| `haproxy` | `haproxy:3.0-alpine` | `8000/tcp` | `lzoc-a`, `lzoc-b` saludables | Configuración declarativa | Crítica |
| `postgres` | `postgres:16-alpine` | No publicado | — | `postgres_data` | Crítica |
| `redis` | `redis:7-alpine` con AOF | No publicado | — | `redis_data` | Crítica |

Características verificadas del núcleo:

- API interna LZOC en `8000/tcp`.
- HAProxy expone una URL estable y depende de las dos instancias saludables.
- `lzoc-a` y `lzoc-b` usan `/health/ready`.
- PostgreSQL y Redis son dependencias de readiness.
- Redis usa AOF (`appendonly yes`, `appendfsync everysec`).
- El pipeline utiliza Redis Streams y persistencia PostgreSQL.

### 2.2 Monitoreo, CMDB y observabilidad

| Servicio | Imagen | Puerto publicado | Persistencia / nota |
|---|---|---:|---|
| `zabbix-postgres` | `postgres:16-alpine` | No publicado | `zabbix_postgres_data` |
| `zabbix-server` | `zabbix/zabbix-server-pgsql:alpine-7.0.26` | No publicado por Compose | Estado en PostgreSQL |
| `zabbix-web` | `zabbix/zabbix-web-nginx-pgsql:alpine-7.0.26` | `8080/tcp` | Depende de DB + server |
| `zabbix-agent-lab` | `zabbix/zabbix-agent2:alpine-7.0.26` | No publicado | Agente de demo |
| `netbox` | `netboxcommunity/netbox:v4.6.9-5.0.2` | `8081/tcp` → `8080` | `netbox_media` |
| `netbox-worker` | misma imagen NetBox | No publicado | `netbox_media` |
| `netbox-postgres` | `postgres:18-alpine` | No publicado | `netbox_postgres_data` |
| `netbox-redis` | `valkey/valkey:9.1-alpine` | No publicado | `netbox_redis_data` |
| `netbox-redis-cache` | `valkey/valkey:9.1-alpine` | No publicado | `netbox_redis_cache_data` |
| `otel-collector` | `otel/opentelemetry-collector-contrib:0.118.0` | No publicado en Compose base | Config declarativa |
| `prometheus` | `prom/prometheus:v3.1.0` | `9090/tcp` | `prometheus_data` |
| `tempo` | `grafana/tempo:2.6.1` | No publicado | `tempo_data` |
| `loki` | `grafana/loki:3.3.2` | No publicado | `loki_data` |
| `alloy` | `grafana/alloy:v1.5.1` | No publicado | Lee Docker socket en el laboratorio actual |
| `grafana` | `grafana/grafana:11.4.0` | `3000/tcp` | `grafana_data` |

### 2.3 Ticketing y notificaciones

Overlay inspeccionado: `docker-compose.ticketing.yml`.

| Servicio | Imagen | Puerto publicado | Persistencia / nota |
|---|---|---:|---|
| `lzoc-worker-zammad` | build LZOC | No publicado | Worker asíncrono |
| `lzoc-worker-smtp` | build LZOC | No publicado | Worker asíncrono |
| `mailpit` | `axllent/mailpit:v1.21.3` | `8025/tcp`, `1025/tcp` | Laboratorio SMTP |
| `zammad-nginx` | `ghcr.io/zammad/zammad:7.1.1-0037` | `8082/tcp` → `8080` | Frontend/API |
| `zammad-railsserver` | misma imagen Zammad | No publicado | Aplicación |
| `zammad-scheduler` | misma imagen Zammad | No publicado | Scheduler |
| `zammad-websocket` | misma imagen Zammad | No publicado | WebSocket |
| `zammad-postgresql` | `postgres:17.10-alpine` | No publicado | `zammad-postgresql-data` |
| `zammad-redis` | `redis:8.8.1-alpine` | No publicado | `zammad-redis-data` |
| `zammad-memcached` | `memcached:1.6.45-alpine` | No publicado | Memoria; `256M` configurados |
| `zammad-backup` | imagen Zammad | No publicado | `zammad-backup` |
| `zammad-init` | imagen Zammad | No publicado | Inicialización |

Volumen persistente funcional de adjuntos/datos Zammad: `zammad-storage`.

### 2.4 IAM / SSO

Overlay inspeccionado: `docker-compose.iam.yml`.

| Servicio | Imagen | Puerto publicado | Persistencia / nota |
|---|---|---:|---|
| `keycloak` | `quay.io/keycloak/keycloak:26.7.3` | `8083/tcp` → `8080` | Realm importado desde archivo |
| `keycloak-postgres` | `postgres:16-alpine` | No publicado | `keycloak_postgres_data` |
| `keycloak-reconcile` | `python:3.12-slim` | No publicado | Job idempotente de reconciliación |

El laboratorio actual usa OIDC para LZOC y OAuth genérico para Grafana. Los secretos y credenciales deben quedar fuera de Git y ser suministrados por el mecanismo de secretos del entorno.

### 2.5 Seguridad de red

#### Zeek

- Imagen actual de laboratorio: `zeek/zeek:8.0.8`.
- Genera/procesa evidencia de demo en el Compose local.
- Volumen: `zeek_logs` montado en `/var/log/zeek`.
- LZOC puede leer `conn.log`, `dns.log` y `notice.log` mediante el adaptador correspondiente.
- En el escenario distribuido, Zeek pertenece al nodo sensor y no debe depender de un volumen Docker compartido entre hosts.

#### Suricata

Overlay inspeccionado: `docker-compose.security.yml`.

- Imagen: `jasonish/suricata:8.0.6`.
- Demo actual basada en PCAP reproducible.
- Volúmenes: `suricata_lab` y `suricata_logs`.
- `lzoc-a` y `lzoc-b` consumen `eve.json` como solo lectura en el escenario local.
- En multi-host se debe utilizar el transporte de ingesta de seguridad autenticado; no se asumirá un filesystem compartido entre nodos.

Servicios auxiliares del escenario de seguridad local:

- `client-lab`: `nginx:1.27-alpine`, IP fija `172.30.0.50`.
- `traffic-generator`: `python:3.12-alpine`, genera el PCAP controlado.

### 2.6 LibreNMS

Overlay inspeccionado: `docker-compose.librenms.yml`.

| Servicio | Imagen | IP laboratorio | Puerto publicado | Persistencia |
|---|---|---:|---:|---|
| `network-device-lab` | build local | `172.31.84.10` | No publicado | No crítica |
| `librenms` | `librenms/librenms:26.8.2` | `172.31.84.20` | `8084/tcp` → `8000` | `librenms_data` |
| `librenms-dispatcher` | `librenms/librenms:26.8.2` | `172.31.84.21` | No publicado | `librenms_data` |
| `librenms-db` | `mariadb:10.11.14` | `172.31.84.30` | No publicado | `librenms_db_data` |
| `librenms-redis` | `redis:7.2.10-alpine` | `172.31.84.40` | No publicado | `librenms_redis_data` |

Red Compose dedicada: `172.31.84.0/24`.

---

## 3. Redes detectadas en la implementación actual

| Red | Uso | Alcance |
|---|---|---|
| `172.30.0.0/24` | Red Compose principal LZOC | Laboratorio local / no debe asumirse como red cloud |
| `172.31.84.0/24` | Red Compose LibreNMS | Laboratorio local / monitoreo de red |
| `10.10.20.0/24` | Red objetivo de clientes | Arquitectura distribuida PTY4684 |
| Tailscale | Overlay entre nodos | Arquitectura vigente mientras se mantenga esta decisión |

### Regla para Terraform

Las redes Docker anteriores **no equivalen** a VPC/subredes de GCP. Terraform aprovisionará las redes del proveedor; Docker conservará sus redes internas salvo que exista una razón técnica documentada para cambiarlas.

---

## 4. Puertos publicados por el laboratorio confirmado

| Puerto host | Servicio | Propósito | Exposición cloud inicial |
|---:|---|---|---|
| `8000/tcp` | HAProxy / LZOC | API + UI estable | Privada por defecto; publicar solo si el escenario lo requiere |
| `8080/tcp` | Zabbix Web | Monitoreo | Privada |
| `8081/tcp` | NetBox | CMDB/IPAM | Privada |
| `8082/tcp` | Zammad | Ticketing | Privada |
| `8083/tcp` | Keycloak | IAM/OIDC | Privada inicialmente; revisar callbacks según acceso |
| `8084/tcp` | LibreNMS | NMS | Privada |
| `8025/tcp` | Mailpit | UI de correo de laboratorio | Solo laboratorio |
| `1025/tcp` | Mailpit | SMTP de laboratorio | Solo red interna |
| `9090/tcp` | Prometheus | Métricas | Privada |
| `3000/tcp` | Grafana | Observabilidad | Privada o acceso controlado |

**No se abrirán estos puertos a `0.0.0.0/0` por defecto en Terraform.** La política inicial será deny-by-default y acceso administrativo mediante mecanismos explícitos.

---

## 5. Persistencia confirmada

### Core

- `postgres_data`
- `redis_data`

### Observabilidad

- `prometheus_data`
- `tempo_data`
- `loki_data`
- `grafana_data`

### Integraciones

- `zeek_logs`
- `zabbix_postgres_data`
- `netbox_postgres_data`
- `netbox_redis_data`
- `netbox_redis_cache_data`
- `netbox_media`
- `keycloak_postgres_data`
- `zammad-postgresql-data`
- `zammad-redis-data`
- `zammad-backup`
- `zammad-storage`
- `librenms_db_data`
- `librenms_redis_data`
- `librenms_data`
- `suricata_lab` — dato reproducible de demo, no necesariamente backup crítico
- `suricata_logs` — evidencia/log según política de retención

### Clasificación preliminar de backup

| Clase | Ejemplos | Tratamiento IaC esperado |
|---|---|---|
| Estado crítico | PostgreSQL LZOC, NetBox, Zammad, Keycloak | Disco persistente + estrategia de backup/restauración |
| Estado operacional reconstruible | Prometheus/Tempo/Loki/Grafana según escenario | Persistencia configurable; política de retención |
| Cache / cola reconstruible con matices | Redis/Valkey | Persistencia solo donde el diseño lo exige; validar recovery |
| Evidencia de laboratorio | PCAP, logs Zeek/Suricata de demo | Regenerable o respaldable según la prueba |

La estrategia definitiva se cerrará en Fase 6; esta clasificación solo evita que Terraform trate todos los volúmenes como equivalentes.

---

## 6. Variables y secretos

### Variables no secretas representativas

- `APP_ENV`, `APP_VERSION`, `APP_PORT`
- `INSTANCE_ID`
- parámetros Redis Streams
- ventanas de correlación y reintentos
- flags `*_ENABLED`
- URLs públicas/internas
- intervalos de polling y timeouts
- configuración de OTel/Grafana

### Secretos confirmados

LZOC-TERRA debe **referenciar**, no versionar, al menos:

- `POSTGRES_PASSWORD`
- `ZABBIX_POSTGRES_PASSWORD`
- `ZABBIX_WEBHOOK_TOKEN`
- `NETBOX_POSTGRES_PASSWORD`
- `NETBOX_REDIS_PASSWORD`
- `NETBOX_REDIS_CACHE_PASSWORD`
- `NETBOX_SECRET_KEY`
- `NETBOX_API_TOKEN_PEPPER`
- `NETBOX_SUPERUSER_PASSWORD`
- `NETBOX_SUPERUSER_API_TOKEN`
- `ZAMMAD_POSTGRES_PASSWORD`
- `ZAMMAD_API_TOKEN`
- `KEYCLOAK_POSTGRES_PASSWORD`
- `KEYCLOAK_ADMIN_PASSWORD`
- `LZOC_OIDC_CLIENT_SECRET`
- `GRAFANA_OAUTH_CLIENT_SECRET`
- credenciales de usuarios de laboratorio IAM
- `LIBRENMS_DB_PASSWORD`
- `LIBRENMS_API_TOKEN`
- `SNMP_COMMUNITY`
- `LZOC_SECURITY_INGEST_TOKEN`

No se copiarán a Terraform variables sensibles en archivos `.tfvars` versionados.

---

## 7. Perfiles de despliegue propuestos

Estos perfiles sirven para dimensionar sin obligar a levantar todas las herramientas simultáneamente.

### `core`

Mínimo para validar LZOC:

- LZOC A/B
- HAProxy
- PostgreSQL
- Redis

### `demo`

`core` + componentes necesarios para una demostración funcional seleccionada, por ejemplo:

- observabilidad;
- una o más fuentes reales;
- NetBox;
- ticketing/notificación cuando la demo lo requiera.

### `full-lab`

Todos los componentes vigentes del laboratorio distribuidos según la topología del proyecto.

### `free-tier`

Perfil **a diseñar después de medir recursos**. No se asumirá que el laboratorio completo cabe en una sola VM pequeña ni que todos los servicios pueden permanecer encendidos permanentemente. La estrategia puede usar despliegue selectivo, apagado entre pruebas y separación de componentes, siempre que preserve la evidencia requerida.

---

## 8. Datos que faltan para cerrar Fase 1

### 8.1 Capacidad por nodo

Para `LAB-JESUS`, `SERV-A` y `SERV-B` falta registrar con evidencia:

- CPU/modelo y núcleos/hilos disponibles;
- RAM total y consumo en reposo/carga;
- almacenamiento total, libre y uso del stack;
- sistema operativo y versión exacta;
- arquitectura (`amd64`/`arm64`);
- Docker Engine / Compose;
- interfaz de red principal;
- IP Tailscale y rutas anunciadas necesarias;
- consumo máximo observado durante una demo multifuente.

### 8.2 Consumo por servicio o grupo

Pendiente medir al menos:

- `core`;
- observabilidad;
- NetBox;
- Zabbix;
- LibreNMS;
- Zammad/Mailpit;
- Keycloak;
- Zeek/Suricata;
- Wazuh;
- Ollama/GIPSIK.

Se priorizará **medición real** (`docker stats`, uso de disco y métricas del host) sobre estimaciones genéricas.

### 8.3 Servicios aún no cerrados

- Wazuh R5: pendiente/bloqueado por capacidad y Host B en el estado actual de LZOC-Orquestador.
- OPNsense físico/live: pendiente de infraestructura.
- GIPSIK: integración/despliegue definitivo pendiente.
- IAM Bloque 2: pendiente.

Estos componentes se incorporarán al inventario cuando su contrato de despliegue esté suficientemente estable.

---

## 9. Contrato entre LZOC-Orquestador y LZOC-TERRA

LZOC-TERRA no debe depender de `main` flotante del orquestador.

### Baseline inicial

```text
repository: HikariLucy/LZOC-Orquestador
ref: 4dbf8c96e66904103a5ecc4bb5469e1d9a391d78
```

Antes de automatizar LZOC Core, se deberá usar un tag, release o SHA explícito.

LZOC-Orquestador es responsable de:

- lógica funcional;
- contratos API;
- migraciones;
- imágenes/Compose de aplicación;
- health checks;
- adaptadores e integraciones.

LZOC-TERRA es responsable de:

- infraestructura cloud;
- red y reglas de firewall;
- nodos/VM/discos;
- preparación del sistema operativo;
- Docker y dependencias del host;
- Tailscale cuando aplique;
- entrega controlada de configuración y secretos;
- despliegue reproducible de una versión fija de LZOC;
- reconstrucción y evidencia de recuperación.

---

## 10. Criterio de salida de Fase 1

Fase 1 se considerará completa cuando:

- [x] existe un baseline versionado del orquestador;
- [x] se identificaron servicios, dependencias principales, puertos y persistencia del stack actual;
- [x] se separaron redes Docker de redes de infraestructura;
- [x] se identificaron secretos que no deben entrar al repositorio;
- [x] se definieron perfiles `core`, `demo`, `full-lab` y el objetivo `free-tier`;
- [ ] CPU/RAM/disco de los tres nodos medidos;
- [ ] consumo real del stack por grupos medido;
- [ ] ubicación definitiva de cada servicio validada por el equipo;
- [ ] requisitos de GIPSIK/Ollama registrados;
- [ ] requisitos de Wazuh registrados cuando Host B esté disponible;
- [ ] matriz final permite dimensionar la primera VM de Fase 2 sin suposiciones críticas.

---

## 11. Próxima evidencia a recolectar

La siguiente actividad es medir el entorno real. Con esas cifras se cerrará una tabla de capacidad por nodo y se derivará el primer perfil GCP.

Comandos orientativos en Linux:

```bash
uname -a
lsb_release -a || cat /etc/os-release
lscpu
free -h
df -h
docker version
docker compose version
docker system df
docker stats --no-stream
```

En hosts Windows/WSL se documentará por separado el host Windows y la VM/WSL donde realmente ejecuta Docker, para evitar confundir recursos físicos con recursos asignados al runtime.

# Contexto y alcance de LZOC-TERRA

## 1. Propósito

`LZOC-TERRA` es el repositorio de Infrastructure as Code asociado a LZOC dentro del Proyecto de Título PTY4684.

El objetivo es transformar el laboratorio existente en una plataforma cuya infraestructura pueda ser **aprovisionada, configurada, reconstruida y documentada de forma repetible**.

El repositorio complementa, y no reemplaza, a los repositorios funcionales de LZOC.

## 2. Problema que aborda

El laboratorio LZOC integra múltiples nodos, servicios de infraestructura, observabilidad y seguridad. Una instalación realizada manualmente presenta riesgos habituales:

- configuraciones difíciles de reproducir;
- diferencias entre ambientes;
- dependencia del conocimiento de una persona;
- tiempos altos de reconstrucción;
- mayor probabilidad de errores de configuración;
- dificultad para migrar entre infraestructura local y cloud.

La adopción de Infrastructure as Code busca reducir esos problemas mediante definición declarativa, control de versiones y procedimientos repetibles.

## 3. Arquitectura funcional de referencia

La automatización debe respetar la arquitectura funcional vigente del proyecto. Como referencia inicial, el laboratorio se distribuye en tres nodos conectados mediante Tailscale:

### LAB-JESUS

Nodo de plataforma e integración general. Aloja, entre otros componentes del laboratorio actual:

- LZOC Orquestador;
- PostgreSQL;
- Redis;
- HAProxy;
- Zabbix;
- LibreNMS;
- NetBox;
- Zammad;
- Grafana / Prometheus;
- Keycloak;
- Mailpit.

### SERV-A

Nodo asociado principalmente a seguridad y análisis:

- Wazuh;
- Ollama;
- GIPSIK.

### SERV-B

Nodo asociado a red, inspección y clientes simulados:

- Zeek;
- Suricata;
- OPNsense;
- red de clientes `10.10.20.0/24`;
- `cliente-1`;
- `cliente-2`.

Esta distribución constituye un **baseline de laboratorio**. No implica que cada despliegue cloud deba replicar obligatoriamente tres máquinas virtuales desde el primer hito. La topología final de cada ambiente deberá responder a restricciones de costo, capacidad y objetivo de prueba.

## 4. Flujo funcional que no debe romperse

La automatización de infraestructura debe preservar el flujo operacional objetivo de LZOC:

1. una falla real ocurre;
2. una herramienta real la detecta;
3. se genera un evento canónico autenticado;
4. LZOC normaliza, deduplica y correlaciona;
5. se consulta información de contexto, incluida la CMDB cuando corresponda;
6. se crea o actualiza el incidente;
7. se genera el ticket;
8. se notifican los eventos relevantes;
9. GIPSIK puede analizar en modo consultivo y de solo lectura;
10. el técnico humano decide y ejecuta acciones;
11. se detecta la recuperación;
12. el incidente puede alcanzar estado `VERIFIED`;
13. el cierre final permanece bajo control humano.

La IA no ejecutará remediaciones autónomas dentro del alcance vigente.

## 5. Alcance de este repositorio

### Incluye

- Terraform para recursos de infraestructura;
- Ansible para configuración de sistemas;
- inventarios y variables por ambiente;
- documentación de arquitectura y decisiones;
- procedimientos de despliegue y recuperación;
- validaciones automatizables de infraestructura;
- separación entre ambientes y proveedores cloud;
- preparación para portabilidad/multicloud.

### No incluye como objetivo principal

- reescritura del LZOC Orquestador;
- modificación de la lógica de correlación de incidentes;
- reemplazo de las herramientas de seguridad/monitoreo ya seleccionadas sin una decisión explícita del proyecto;
- remediación automática mediante IA.

## 6. Resultado esperado

El objetivo de madurez es poder reconstruir un ambiente LZOC a partir de:

1. código versionado;
2. credenciales y secretos gestionados fuera del repositorio;
3. variables específicas del ambiente;
4. una secuencia documentada de Terraform y Ansible.

En términos operacionales, el repositorio debe acercarnos a la siguiente capacidad:

> Con un repositorio, credenciales válidas y parámetros del ambiente, es posible reconstruir la infraestructura necesaria para ejecutar LZOC de manera controlada y auditable.

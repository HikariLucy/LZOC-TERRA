# ADR-0001 — Terraform + Ansible como base de automatización

- **Estado:** Aceptada
- **Fecha:** 2026-09-05
- **Proyecto:** LZOC-TERRA / PTY4684

## Contexto

LZOC requiere desplegar y configurar infraestructura distribuida para ejecutar componentes de plataforma, observabilidad, seguridad, red y gestión de incidentes.

Mantener estos ambientes solamente mediante procedimientos manuales dificulta su reproducción, migración, recuperación y auditoría.

El proyecto además busca validar inicialmente un despliegue en Google Cloud Platform y, posteriormente, conservar la posibilidad de adaptar la plataforma a otros entornos.

## Decisión

Se adopta la siguiente separación principal:

- **Terraform** para aprovisionar recursos de infraestructura.
- **Ansible** para configurar sistemas operativos y hosts.
- **Docker / Docker Compose** para gestionar servicios contenerizados cuando corresponda.

La lógica funcional propia de LZOC permanecerá desacoplada de Terraform.

## Motivos

### Terraform

Se selecciona por su capacidad para:

- definir infraestructura declarativamente;
- mostrar cambios mediante `plan` antes de aplicarlos;
- manejar dependencias entre recursos;
- utilizar proveedores distintos;
- mantener módulos versionados;
- destruir y reconstruir ambientes de forma controlada.

### Ansible

Se selecciona para:

- configuración idempotente de hosts;
- instalación de paquetes y dependencias;
- gestión de archivos y plantillas;
- configuración de Docker y servicios;
- reutilización de roles entre cloud y laboratorio local;
- ejecución remota sin requerir agentes permanentes en cada host.

### Docker / Compose

Se conserva como capa de despliegue cuando el servicio ya está correctamente contenerizado, evitando duplicar en Ansible una responsabilidad que pertenece al runtime de contenedores.

## Consecuencias positivas

- mayor reproducibilidad;
- menor dependencia de configuración manual;
- mejor capacidad de recuperación;
- trazabilidad mediante Git;
- posibilidad de comparar cambios antes de aplicarlos;
- separación clara de responsabilidades;
- base razonable para portabilidad progresiva.

## Riesgos y costos

- Terraform introduce gestión de estado;
- será necesario diseñar una estrategia segura para secretos;
- los módulos de distintos proveedores no serán idénticos;
- una abstracción multicloud excesiva puede aumentar complejidad;
- algunos servicios del laboratorio pueden requerir tratamiento especial por consumo de recursos o topología de red.

## Reglas derivadas

1. No versionar archivos `terraform.tfstate`.
2. No almacenar secretos reales en Git.
3. No usar `local-exec` o provisioners de Terraform como sustituto general de Ansible.
4. Mantener variables por ambiente separadas del código reutilizable.
5. Mantener roles Ansible agnósticos del proveedor siempre que sea posible.
6. Validar primero una implementación real en GCP antes de generalizar completamente hacia multicloud.
7. Toda modificación que cambie significativamente estas responsabilidades deberá quedar registrada mediante un nuevo ADR.

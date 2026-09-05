# Roadmap de implementación

## Objetivo

Construir la automatización de LZOC de manera incremental, evitando intentar desplegar el laboratorio completo antes de validar las capas fundamentales.

## Fase 0 — Fundación documental

**Objetivo:** fijar alcance, responsabilidades y decisiones iniciales.

Entregables:

- README del repositorio;
- contexto y alcance;
- arquitectura Terraform + Ansible;
- baseline GCP asociado al trabajo de José;
- ADR de selección tecnológica;
- roadmap.

Criterio de salida:

- el equipo puede explicar qué pertenece a Terraform, Ansible, Docker y LZOC.

## Fase 1 — Inventario técnico del laboratorio

**Objetivo:** convertir la arquitectura existente en requisitos concretos de infraestructura.

Se levantará para cada nodo y servicio:

- CPU;
- RAM;
- almacenamiento;
- puertos;
- protocolos;
- dependencias;
- persistencia;
- DNS/nombres;
- variables de entorno;
- secretos requeridos;
- conectividad entre componentes;
- necesidad de exposición externa;
- posibilidad de apagado entre pruebas.

También se definirá qué componentes pertenecen al perfil `free-tier` y cuáles solamente al perfil `full-lab`.

Criterio de salida:

- existe una matriz suficientemente precisa para dimensionar la primera infraestructura GCP.

## Fase 2 — Terraform GCP Foundation

**Objetivo:** crear infraestructura mínima reproducible.

Primer alcance:

- providers y versiones;
- variables de proyecto/región/zona;
- VPC;
- subred;
- firewall mínimo;
- una VM de prueba;
- disco cuando corresponda;
- outputs;
- `.gitignore` y manejo seguro del state local inicial.

Pruebas:

```text
terraform fmt -check
terraform validate
terraform plan
terraform apply
terraform destroy
```

Criterio de salida:

- una VM puede crearse, destruirse y reconstruirse desde código.

## Fase 3 — Ansible Base

**Objetivo:** dejar el host creado por Terraform listo para alojar servicios.

Roles iniciales:

- `base`;
- `docker`;
- `tailscale` si corresponde al escenario;
- hardening mínimo documentado.

Pruebas:

- conectividad SSH;
- segunda ejecución sin cambios innecesarios;
- Docker operativo;
- conectividad del overlay cuando corresponda.

Criterio de salida:

- un host limpio puede convertirse automáticamente en un nodo base de LZOC.

## Fase 4 — LZOC Core

**Objetivo:** desplegar el núcleo mínimo necesario para demostrar el orquestador.

El conjunto exacto se definirá después del inventario, priorizando componentes indispensables y persistencia de datos.

Validaciones:

- health checks;
- comunicación entre componentes;
- persistencia;
- reinicio controlado;
- logs suficientes para diagnóstico.

## Fase 5 — Observabilidad, monitoreo y seguridad

**Objetivo:** incorporar progresivamente las herramientas del laboratorio que acompañan el ciclo completo de detección y gestión de incidentes.

Se considerarán, según topología y recursos disponibles:

- Prometheus/Grafana;
- Zabbix;
- NetBox;
- Zammad;
- Wazuh;
- Zeek;
- Suricata;
- demás componentes vigentes del laboratorio.

La inclusión se realizará por dependencia y valor demostrable, no solamente por cantidad de herramientas desplegadas.

## Fase 6 — Repetibilidad y recuperación

**Objetivo:** demostrar que IaC aporta continuidad operacional.

Pruebas previstas:

1. crear ambiente desde cero;
2. ejecutar configuración;
3. ejecutar prueba funcional;
4. respaldar información que corresponda;
5. destruir infraestructura;
6. reconstruir;
7. restaurar datos necesarios;
8. repetir validaciones.

Criterio de salida:

- la reconstrucción deja de depender de una instalación manual irrepetible.

## Fase 7 — Portabilidad

**Objetivo:** extraer lo aprendido de GCP para reducir acoplamiento con el proveedor.

Trabajo:

- clasificar componentes agnósticos y específicos;
- consolidar roles Ansible reutilizables;
- estandarizar interfaces/outputs Terraform;
- implementar un segundo entorno o proveedor cuando aporte evidencia al proyecto;
- comparar diferencias de despliegue.

## Regla de avance

Cada fase debe generar evidencia verificable antes de avanzar a la siguiente. Una fase no se considerará terminada únicamente porque exista código: debe poder demostrarse su comportamiento y quedar documentadas las decisiones relevantes.

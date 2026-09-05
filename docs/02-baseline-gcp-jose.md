# Baseline GCP — enlace con el trabajo preparado para José

## 1. Propósito

Este documento conecta LZOC-TERRA con el trabajo previo preparado para José respecto del uso de Google Cloud Platform para alojar parte o la totalidad del laboratorio LZOC.

La implementación en GCP será el **primer caso de uso real** de esta capa de Infrastructure as Code.

## 2. Rol de GCP en esta etapa

GCP no se define como proveedor definitivo ni obligatorio para LZOC. Se utiliza como plataforma inicial porque permite validar:

- creación declarativa de infraestructura;
- despliegue remoto reproducible;
- separación entre infraestructura y configuración;
- consumo controlado de recursos;
- estrategias frente a restricciones de créditos o capa gratuita;
- recuperación del laboratorio a partir de código.

## 3. Dos perfiles de despliegue

Se propone mantener dos perfiles conceptuales.

### `free-tier`

Ambiente reducido, orientado a:

- pruebas de Terraform;
- validación de Ansible;
- pruebas del orquestador y servicios mínimos;
- demostraciones de corta duración;
- control estricto de costos.

Este perfil no debe asumir que todos los servicios del laboratorio completo caben permanentemente dentro de los límites gratuitos.

### `full-lab`

Ambiente orientado a reproducir una topología más cercana al laboratorio completo cuando existan créditos o presupuesto suficiente.

Puede incluir más capacidad de CPU, RAM, almacenamiento y múltiples nodos, dependiendo del escenario que se quiera demostrar.

## 4. Estrategia frente al agotamiento de créditos

La capa de IaC debe permitir que terminar los créditos de una cuenta no implique perder el trabajo realizado.

Las alternativas de continuidad contempladas son:

1. destruir recursos no necesarios conservando código y respaldos;
2. reconstruir el ambiente posteriormente en la misma cuenta;
3. ejecutar un perfil reducido;
4. trasladar el despliegue a otra cuenta/proyecto autorizado;
5. reutilizar la capa de configuración sobre infraestructura local;
6. adaptar los módulos de infraestructura a otro proveedor cloud.

El repositorio debe favorecer esta independencia manteniendo separadas las definiciones específicas de GCP de los roles de configuración reutilizables.

## 5. División de responsabilidades propuesta

### Terraform GCP

En los primeros hitos deberá ser capaz de manejar, como mínimo cuando el diseño lo requiera:

- proyecto/región/zona mediante variables;
- VPC;
- subred;
- reglas de firewall;
- instancias de Compute Engine;
- discos persistentes;
- cuentas de servicio e IAM mínimos;
- outputs necesarios para configurar los hosts.

### Ansible

Sobre las VM creadas por Terraform:

- preparación de Ubuntu;
- actualización y paquetes base;
- usuarios/SSH;
- Docker;
- Tailscale cuando corresponda;
- estructura de directorios LZOC;
- despliegue gradual de los servicios definidos para cada perfil;
- health checks básicos.

## 6. Costo como requisito de arquitectura

Para este proyecto, el costo no se tratará solamente como una consideración administrativa. Debe influir en el diseño técnico.

Por ello se documentarán:

- cantidad y tipo de VM;
- discos y persistencia;
- componentes que pueden detenerse entre pruebas;
- componentes imprescindibles para una demo;
- recursos que pueden mantenerse localmente;
- diferencias entre el perfil mínimo y el laboratorio completo.

No se codificarán supuestos de gratuidad permanente. Los límites y precios del proveedor pueden variar y deberán verificarse al momento de ejecutar un despliegue real.

## 7. Primer resultado esperado en GCP

Antes de intentar desplegar todos los componentes de LZOC, se considerará exitoso el primer hito cuando podamos demostrar el siguiente ciclo:

1. ejecutar Terraform sobre un ambiente limpio;
2. crear una red y al menos una VM válida;
3. obtener los datos necesarios del host;
4. ejecutar Ansible;
5. configurar el sistema operativo y Docker;
6. verificar conectividad y salud;
7. destruir la infraestructura de forma controlada;
8. volver a crearla y obtener un resultado equivalente.

Ese ciclo constituye la base técnica para incorporar progresivamente LZOC.

## 8. Relación con la estrategia multicloud

La implementación GCP actuará como referencia y campo de prueba. Una vez estabilizada se identificarán tres clases de elementos:

- **agnósticos del proveedor:** roles Ansible, plantillas, configuración de servicios;
- **abstraíbles:** conceptos equivalentes como redes, VM, discos y reglas;
- **específicos del proveedor:** IAM, tipos concretos de recursos, metadatos y peculiaridades de GCP.

La portabilidad se desarrollará sobre esa evidencia y no mediante abstracciones prematuras.

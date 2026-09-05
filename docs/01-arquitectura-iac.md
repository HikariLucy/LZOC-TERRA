# Arquitectura Infrastructure as Code

## 1. Objetivo arquitectónico

LZOC-TERRA separará responsabilidades entre aprovisionamiento de infraestructura, configuración de sistemas y despliegue de servicios.

```text
Proveedor cloud / hipervisor
          │
          ▼
      Terraform
          │
          ▼
Redes + VM + discos + reglas + identidad
          │
          ▼
       Ansible
          │
          ▼
SO + paquetes + hardening + Docker + Tailscale
          │
          ▼
Docker / Compose / servicios nativos
          │
          ▼
        LZOC
```

## 2. Responsabilidades

### Terraform

Terraform será responsable de recursos cuya existencia pertenece a la capa de infraestructura:

- redes virtuales;
- subredes;
- reglas de firewall/security groups equivalentes;
- máquinas virtuales;
- discos persistentes;
- direcciones IP cuando corresponda;
- cuentas de servicio e IAM cuando sea necesario;
- outputs consumibles por Ansible;
- recursos cloud específicos del ambiente.

Terraform **no debe utilizarse como sustituto de un gestor de configuración** para instalar y mantener paquetes dentro de las máquinas.

### Ansible

Ansible será responsable de la configuración del sistema operativo y de los prerrequisitos:

- usuarios y grupos;
- claves SSH autorizadas;
- paquetes del sistema;
- Docker Engine y complementos necesarios;
- estructura de directorios;
- configuración base de seguridad;
- Tailscale cuando forme parte del ambiente;
- archivos de configuración generados desde plantillas;
- despliegue y actualización controlada de servicios;
- comprobaciones posteriores al despliegue.

### Docker / Docker Compose

Docker seguirá encapsulando los servicios que ya tengan un modelo de despliegue contenerizado adecuado. Esto permite evitar que Ansible replique lógica que ya corresponde a Compose.

Ansible podrá:

- instalar Docker;
- copiar o renderizar archivos Compose;
- gestionar variables no secretas;
- iniciar o actualizar stacks;
- comprobar salud de los servicios.

### LZOC

La lógica funcional de LZOC se mantiene fuera de Terraform. El objetivo de esta capa es proporcionar un ambiente consistente para que los componentes de LZOC puedan ejecutarse.

## 3. Estructura objetivo del repositorio

```text
LZOC-TERRA/
├── terraform/
│   ├── modules/
│   │   ├── network/
│   │   ├── compute/
│   │   ├── firewall/
│   │   └── storage/
│   └── environments/
│       ├── gcp/
│       │   ├── free-tier/
│       │   └── full-lab/
│       ├── aws/
│       └── azure/
│
├── ansible/
│   ├── inventories/
│   │   ├── gcp/
│   │   ├── local/
│   │   └── future-clouds/
│   ├── roles/
│   │   ├── base/
│   │   ├── docker/
│   │   ├── tailscale/
│   │   ├── lzoc/
│   │   ├── monitoring/
│   │   ├── security/
│   │   └── observability/
│   └── playbooks/
│
├── docs/
│   ├── decisions/
│   ├── runbooks/
│   └── cloud/
│
└── README.md
```

La estructura es un objetivo y podrá evolucionar a medida que se validen dependencias reales.

## 4. Principios de diseño

### 4.1 Idempotencia

Ejecutar nuevamente Terraform o Ansible no debería provocar cambios innecesarios cuando el ambiente ya se encuentra en el estado deseado.

### 4.2 Separación de estado y configuración

- El estado de Terraform no debe versionarse en Git.
- Los secretos no deben almacenarse en texto plano en el repositorio.
- Las variables propias de cada entorno deben estar separadas del código reutilizable.

### 4.3 Módulos pequeños y reutilizables

Los módulos Terraform deben representar capacidades de infraestructura razonablemente independientes. Se evitará construir un único módulo monolítico que dependa completamente de GCP.

### 4.4 Roles de Ansible reutilizables

Las tareas comunes —por ejemplo `base`, `docker` o `tailscale`— deberán ser independientes del proveedor cloud siempre que técnicamente sea posible.

### 4.5 Portabilidad progresiva

La portabilidad multicloud no se intentará resolver completamente antes de contar con una implementación funcional.

Secuencia prevista:

1. validar GCP;
2. identificar elementos específicos de GCP;
3. extraer componentes reutilizables;
4. crear equivalentes para otros proveedores;
5. comprobar que el comportamiento funcional de LZOC se mantiene.

## 5. Flujo de ejecución esperado

```text
terraform init
      │
terraform plan
      │
terraform apply
      │
      ├── outputs de nodos/IP
      ▼
Generación/actualización de inventario
      │
ansible-playbook
      │
      ▼
Validaciones de sistema
      │
      ▼
Validaciones funcionales LZOC
```

En etapas posteriores este flujo podrá ejecutarse mediante CI/CD, pero inicialmente se priorizará que sea entendible, observable y ejecutable manualmente por el equipo.

## 6. Criterio de éxito

Una automatización no se considerará completa solo porque Terraform termine con éxito. El despliegue deberá demostrar, según el hito correspondiente:

- conectividad esperada;
- acceso administrativo controlado;
- servicios requeridos activos;
- persistencia donde corresponda;
- integración entre nodos;
- capacidad de repetir el procedimiento;
- evidencia suficiente para diagnóstico y auditoría.

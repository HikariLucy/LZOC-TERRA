# LZOC-TERRA

Repositorio de **Infrastructure as Code (IaC)** para el Proyecto de Título PTY4684 y la plataforma LZOC.

Su objetivo es convertir el laboratorio LZOC en una infraestructura **reproducible, documentada y portable**, usando principalmente:

- **Terraform** para aprovisionamiento de infraestructura.
- **Ansible** para configuración de sistemas y despliegue.
- **Docker / Docker Compose** para servicios contenerizados cuando corresponda.
- **Tailscale** como mecanismo de interconexión del laboratorio mientras siga siendo parte de la arquitectura vigente.

## Objetivo inicial

La primera implementación de referencia será **Google Cloud Platform (GCP)**, enlazada con el trabajo realizado para desplegar/migrar el laboratorio de LZOC en la cuenta de José, considerando restricciones de Free Tier, créditos disponibles y alternativas de continuidad cuando esos créditos se agoten.

La meta posterior es desacoplar la automatización del proveedor para permitir despliegues equivalentes en otros entornos cloud.

## Principio de diseño

> Terraform crea la infraestructura; Ansible configura los nodos; Docker despliega los servicios; LZOC conserva la lógica funcional del proyecto.

Este repositorio no pretende rediseñar el flujo funcional de LZOC. Su propósito es hacer que su infraestructura pueda reconstruirse de forma controlada y repetible.

## Estado

Repositorio en fase de fundación documental. La infraestructura y los playbooks se incorporarán por etapas y mediante ramas de trabajo.

# Plantilla ARM para Desplegar una Máquina Virtual Windows

Esta plantilla ARM (Azure Resource Manager) está diseñada para desplegar una máquina virtual (VM) con Windows, junto con los recursos necesarios como una cuenta de almacenamiento, IP pública, red virtual, grupo de seguridad de red y una interfaz de red. La plantilla está construida usando Bicep y permite la personalización de parámetros clave, como el tamaño de la VM, la versión del sistema operativo y configuraciones de seguridad.

## Requisitos Previos

- Una suscripción activa en Azure.
- Acceso a Azure CLI o Azure PowerShell para desplegar la plantilla.
- Permisos adecuados para crear recursos dentro del grupo de recursos de destino.

## Parámetros de la Plantilla

Los siguientes parámetros pueden ser personalizados en la plantilla:

| Parámetro                   | Tipo          | Valor Predeterminado                         | Descripción                                                                  |
|-----------------------------|---------------|----------------------------------------------|------------------------------------------------------------------------------|
| `adminUsername`              | `string`      | N/A                                          | Nombre de usuario para la máquina virtual.                                   |
| `adminPassword`              | `securestring`| N/A                                          | Contraseña segura para la máquina virtual (mínimo 12 caracteres).            |
| `dnsLabelPrefix`             | `string`      | Basado en el nombre de la VM                 | Etiqueta DNS única para la IP pública utilizada para acceder a la VM.        |
| `publicIpName`               | `string`      | `myPublicIP`                                 | Nombre de la IP pública asociada con la VM.                                  |
| `publicIPAllocationMethod`   | `string`      | `Dynamic`                                    | Método de asignación de la IP pública (`Dynamic` o `Static`).                |
| `publicIpSku`                | `string`      | `Basic`                                      | SKU de la IP pública (`Basic` o `Standard`).                                 |
| `OSVersion`                  | `string`      | `2022-datacenter-azure-edition`              | Versión de Windows para la VM.                                               |
| `vmSize`                     | `string`      | `Standard_D2s_v5`                            | Tamaño de la máquina virtual.                                                |
| `location`                   | `string`      | `[resourceGroup().location]`                 | Ubicación en Azure para todos los recursos.                                  |
| `vmName`                     | `string`      | `simple-vm`                                  | Nombre de la máquina virtual.                                                |
| `securityType`               | `string`      | `TrustedLaunch`                              | Tipo de seguridad para la VM (`Standard` o `TrustedLaunch`).                 |

## Recursos Desplegados

Los siguientes recursos se desplegarán como parte de esta plantilla:

1. **Cuenta de Almacenamiento**: Utilizada para los diagnósticos de inicio de la VM.
2. **Dirección IP Pública**: Permite el acceso público a la VM.
3. **Red Virtual (VNet)**: Proporciona conectividad de red para la VM.
4. **Grupo de Seguridad de Red (NSG)**: Contiene reglas de seguridad que permiten el acceso RDP (puerto 3389) a la VM.
5. **Interfaz de Red (NIC)**: Conecta la VM a la red virtual.
6. **Máquina Virtual Windows**: La instancia principal de la máquina virtual con tamaño y sistema operativo personalizables.

## Perfil de Seguridad

Esta plantilla admite dos perfiles de seguridad:

1. **Estándar**: Perfil de seguridad predeterminado sin Trusted Launch.
2. **Trusted Launch**: Activa el arranque seguro UEFI y TPM virtual (vTPM) para mejorar la seguridad.

## Cómo Desplegar

### Usando Azure CLI

Puedes desplegar esta plantilla usando Azure CLI con el siguiente comando:

```bash
az deployment group create \
  --name bootcamp \
  --resource-group MiGrupoDeRecursos2 \
  --template-file template.json \
  --parameters adminUsername='miusuario' \
                adminPassword='miContraseñaSegura!' \
                vmName='MyVM'

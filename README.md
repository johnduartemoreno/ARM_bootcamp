# Plantilla ARM para Desplegar una Máquina Virtual Windows

Esta plantilla ARM (Azure Resource Manager) está diseñada para desplegar una máquina virtual (VM) con **Windows**, junto con los recursos necesarios como una cuenta de almacenamiento, IP pública, red virtual, grupo de seguridad de red y una interfaz de red. La plantilla está construida utilizando Bicep y permite la personalización de parámetros clave, como el tamaño de la VM, la versión del sistema operativo y configuraciones de seguridad.

## Requisitos Previos

- Una suscripción activa en Azure.
- Acceso a **Azure CLI** o **Azure PowerShell** para desplegar la plantilla.
- Permisos adecuados para crear recursos dentro del grupo de recursos de destino.

## Parámetros de la Plantilla

Los siguientes parámetros pueden ser personalizados en la plantilla o en el archivo de parámetros:

| Parámetro                   | Tipo          | Valor Predeterminado                         | Descripción                                                                  |
|-----------------------------|---------------|----------------------------------------------|------------------------------------------------------------------------------|
| `adminUsername`              | `string`      | N/A                                          | Nombre de usuario para la máquina virtual.                                   |
| `adminPassword`              | `securestring`| N/A                                          | Contraseña segura para la máquina virtual (mínimo 12 caracteres).            |
| `dnsLabelPrefix`             | `string`      | Basado en el nombre de la VM                 | Etiqueta DNS única para la IP pública utilizada para acceder a la VM.        |
| `publicIpName`               | `string`      | `myPublicIP`                                 | Nombre de la IP pública asociada con la VM.                                  |
| `publicIPAllocationMethod`   | `string`      | `Dynamic`                                    | Método de asignación de la IP pública (`Dynamic` o `Static`).                |
| `publicIpSku`                | `string`      | `Basic`                                      | SKU de la IP pública (`Basic` o `Standard`).                                 |
| `OSVersion`                  | `string`      | `2022-datacenter-azure-edition`              | Versión del sistema operativo para la VM (Windows Server).                   |
| `vmSize`                     | `string`      | `Standard_D2s_v5`                            | Tamaño de la máquina virtual.                                                |
| `location`                   | `string`      | `[resourceGroup().location]`                 | Ubicación en Azure para todos los recursos.                                  |
| `vmName`                     | `string`      | `simple-vm`                                  | Nombre de la máquina virtual.                                                |
| `securityType`               | `string`      | `TrustedLaunch`                              | Tipo de seguridad para la VM (`Standard` o `TrustedLaunch`).                 |

## Recursos Desplegados

Los siguientes recursos se desplegarán como parte de esta plantilla:

1. **Cuenta de Almacenamiento**: Utilizada para los diagnósticos de inicio de la VM.
2. **Dirección IP Pública**: Permite el acceso público a la VM.
3. **Red Virtual (VNet)**: Proporciona conectividad de red para la VM.
4. **Grupo de Seguridad de Red (NSG)**: Contiene reglas de seguridad que permiten el acceso RDP (puerto 3389 para Windows).
5. **Interfaz de Red (NIC)**: Conecta la VM a la red virtual.
6. **Máquina Virtual**: La instancia principal de la máquina virtual con tamaño y sistema operativo personalizables.

## Perfil de Seguridad

Esta plantilla admite dos perfiles de seguridad:

1. **Estándar**: Perfil de seguridad predeterminado sin Trusted Launch.
2. **Trusted Launch**: Activa el arranque seguro UEFI y TPM virtual (vTPM) para mejorar la seguridad.

## Cómo Desplegar Usando un Archivo de Parámetros

Puedes usar un archivo de parámetros para especificar los valores de los parámetros de manera más eficiente y reutilizable.

### Archivo de Parámetros de Ejemplo (`parameters.json`):

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "adminUsername": {
      "value": "miusuario"
    },
    "adminPassword": {
      "value": "miContraseñaSegura!"
    },
    "dnsLabelPrefix": {
      "value": "mydnslabelbootcamp"
    },
    "publicIpName": {
      "value": "MyPublicIP"
    },
    "publicIPAllocationMethod": {
      "value": "Dynamic"
    },
    "publicIpSku": {
      "value": "Basic"
    },
    "OSVersion": {
      "value": "2022-datacenter-azure-edition"
    },
    "vmSize": {
      "value": "Standard_D2s_v5"
    },
    "location": {
      "value": "eastus"
    },
    "vmName": {
      "value": "MyVM"
    },
    "securityType": {
      "value": "TrustedLaunch"
    }
  }
}
```

### Desplegar la Plantilla Usando Azure CLI y el Archivo de Parámetros

Puedes desplegar la plantilla usando el siguiente comando de **Azure CLI**:

```bash
az deployment group create   --name bootcamp   --resource-group MiGrupoDeRecursos   --template-file template.json   --parameters @parameters.json
```

## Verificación y Conexión a la Máquina Virtual Windows usando RDP

### 1. Verificar el Estado de la VM:
Antes de intentar conectarte a la máquina virtual, asegúrate de que está en estado **running**:

```bash
az vm get-instance-view --resource-group MiGrupoDeRecursos --name MyVM --query instanceView.statuses[1] --output table
```

El resultado debe mostrar `PowerState/running`.

### 2. Conectarte usando RDP (Escritorio Remoto):
- Una vez que la VM esté en estado de ejecución, sigue estos pasos para conectarte:
  1. Obtén la **IP pública** de la VM con el siguiente comando:
  
  ```bash
  az vm show --resource-group MiGrupoDeRecursos --name MyVM -d --query publicIps -o tsv
  ```

  2. Abre la aplicación **Conexión a Escritorio Remoto (Remote Desktop)** en tu computadora:
     - En **Windows**, busca "Conexión a Escritorio Remoto" en el menú Inicio.
     - En **macOS** o **Linux**, descarga la aplicación **Microsoft Remote Desktop** desde la tienda de aplicaciones.

  3. Ingresa la **IP pública** de la VM.

  4. Usa las credenciales que configuraste (por ejemplo, `adminUsername` y `adminPassword`).

  5. Haz clic en **Conectar**.

### 3. Verificar las reglas de NSG para RDP:
Asegúrate de que el puerto **3389** esté permitido en el **Grupo de Seguridad de Red (NSG)** asociado a la VM:

- Puedes verificar esto en el portal de Azure, en la sección **Redes** de la VM, o agregar la regla usando el siguiente comando:

```bash
az network nsg rule create   --resource-group MiGrupoDeRecursos   --nsg-name default-NSG   --name allow-rdp   --protocol Tcp   --priority 1000   --destination-port-ranges 3389   --access Allow   --direction Inbound   --source-address-prefixes '*'   --source-port-ranges '*'
```

---

# azure-terraform-iac
# Configuración de Prerrequisitos y Autenticación en Azure

Esta sección detalla los pasos iniciales necesarios para preparar el entorno local y establecer una conexión segura con la nube de Azure antes de realizar cualquier despliegue con Terraform.

---

## 1. Prerrequisitos del Sistema

Para comenzar con este proyecto, es indispensable contar con las siguientes herramientas y accesos configurados:

* **Suscripción de Azure:** Se requiere una cuenta activa para la gestión de recursos. Puedes obtener una cuenta gratuita o para estudiantes en el siguiente enlace: [Azure Account Setup](https://azure.microsoft.com/en-us/pricing/purchase-options/azure-account?icid=azurefreeaccount).
* **Instalación Local de Terraform:** Descarga e instala el ejecutable en tu máquina local. 
    > **⚠️ Nota Crítica:** No olvides añadir la ruta de instalación a las **Variables de Entorno del sistema (PATH)** para que la terminal reconozca el comando `terraform` desde cualquier directorio.  
    [Descarga de Terraform](https://developer.hashicorp.com/terraform/install)
* **Azure CLI:** Esta herramienta permite la comunicación y autenticación entre tu máquina local y los servicios de Azure. Instálala ejecutando el siguiente comando en PowerShell:

```powershell
$ Invoke-WebRequest -Uri [https://aka.ms/installazurecliwindows](https://aka.ms/installazurecliwindows) -OutFile .\AzureCLI.msi; Start-Process msiexec.exe -Wait -ArgumentList '/I AzureCLI.msi /quiet'; rm .\AzureCLI.msi
```

---

## 2. Autenticación y Gestión de Suscripciones

Con los requisitos instalados, procedemos a configurar los permisos de acceso local para que Terraform pueda operar en nuestra infraestructura.

### Inicio de Sesión (`az login`)
Para instalar los permisos que Azure necesita para autenticarse de forma local, realizamos el inicio de sesión mediante el CLI de Azure:

```powershell
$ az login
```

![Captura: Proceso de Login en la Terminal](images/az_login.png)

> **🔒 Nota de Seguridad:** Es fundamental ocultar el **Subscription ID** y el **Tenant ID** en capturas de pantalla o entornos públicos (como este repositorio) para proteger la privacidad y seguridad de tu cuenta.

### Selección de la Suscripción Activa
Tras la autenticación, el CLI mostrará un listado de todas las suscripciones asociadas a tu cuenta. Es **fundamental fijar la suscripción activa** donde deseamos que Terraform realice los despliegues (por ejemplo, la de **Estudiantes**) para asegurar que los recursos se creen en el entorno correcto y se utilicen los créditos adecuados.

Establece la suscripción de trabajo mediante el siguiente comando sustituyendo el ID por el tuyo:

```powershell
$ az account set --subscription "TU_SUBSCRIPTION_ID_AQUÍ"
```
---

## 3. Creación del Service Principal

El siguiente paso fundamental es la creación de un **Service Principal**. En el ecosistema de Azure, un Service Principal es una identidad de aplicación (un "usuario no humano") que permite que herramientas externas, como Terraform o un pipeline de CI/CD, interactúen con tus recursos de forma segura.

### ¿Por qué es necesario este paso?
Implementar un Service Principal es una práctica estándar en la industria por los siguientes motivos:
* **Seguridad y Aislamiento:** Evitamos el uso de nuestra cuenta personal de usuario para tareas automatizadas, lo cual se considera una mala práctica en entornos profesionales y de producción.
* **Automatización:** Al proporcionar a Terraform sus propias credenciales, el sistema puede autenticarse automáticamente ante la API de Azure sin necesidad de ejecutar un `az login` manual en cada sesión.
* **Control de Accesos (RBAC):** Permite aplicar el principio de mínimo privilegio, limitando exactamente qué acciones puede realizar este "robot" (asignándole el rol de **Contributor**) y sobre qué suscripción específica tiene poder.

### Comando de Creación
Para generar esta identidad y obtener sus credenciales de acceso, utilizamos el siguiente comando:

```powershell
$ az ad sp create-for-rbac --role="Contributor" --scopes="/subscriptions/<TU_SUBSCRIPTION_ID>"
```

> **Nota:** Por motivos de seguridad, no se incluye captura de la terminal en este paso para proteger el ID de la suscripción.

### Valores Obtenidos
Al ejecutar el comando, la terminal devolverá un objeto JSON con cuatro valores fundamentales que Terraform utilizará para la autenticación:
1. **appId:** El identificador único del Service Principal (Client ID).
2. **displayName:** El nombre identificativo asignado a la identidad en Azure.
3. **password:** La contraseña o secreto de cliente (Client Secret).
4. **tenant:** El ID del directorio de nuestra organización.

---

### ⚠️ Advertencia de Seguridad
Por razones críticas de seguridad, **no se incluye una captura de pantalla de esta salida**. El valor de la **password** es extremadamente sensible:
* Solo se muestra una vez al momento de la creación.
* Permite el acceso total a los recursos bajo el rol asignado dentro de la suscripción.
* **Bajo ninguna circunstancia** debe ser compartido, enviado por canales no seguros o subido a un repositorio público.

---

## 4. Configuración de Variables de Entorno

Para que Terraform pueda autenticarse con Azure de forma automática y segura, utilizaremos **Variables de Entorno** en nuestra terminal. Esto permite que el proveedor de Azure lea las credenciales directamente de la memoria del sistema sin que estas queden registradas en el código.

### ¿Por qué se hace esto?
* **Seguridad (Evitar fugas de secretos):** Es la razón principal. Si escribimos las contraseñas dentro del código y subimos ese archivo a GitHub, cualquier persona podría robar nuestras credenciales. Al usar variables de entorno, las llaves solo viven en la memoria temporal de tu sesión de terminal.
* **Flexibilidad:** Permite que el mismo código de Terraform se ejecute en diferentes entornos (Desarrollo, Producción) simplemente cambiando las variables de la terminal, sin tocar una sola línea de código.
* **Estándar Profesional:** Es el método recomendado por **HashiCorp** y el estándar utilizado en consultoras de alto nivel como **Avanade** para proteger la infraestructura crítica.

### Comandos de Configuración (PowerShell)
Asignaremos los valores obtenidos del Service Principal mediante los siguientes comandos. Sustituye los valores en mayúsculas por tus credenciales:

```powershell
$env:ARM_CLIENT_ID = "TU_APP_ID"
$env:ARM_CLIENT_SECRET = "TU_PASSWORD"
$env:ARM_TENANT_ID = "TU_TENANT_ID"
$env:ARM_SUBSCRIPTION_ID = "TU_SUBSCRIPTION_ID"
```

---

## 5. Creación del Directorio de Proyecto

Para mantener una estructura profesional y evitar conflictos con otros experimentos, crearemos una carpeta dedicada exclusivamente a este despliegue.

### ¿Por qué es importante el aislamiento?
* **Aislamiento del Estado:** Terraform genera archivos locales críticos como la carpeta `.terraform/` y el archivo `terraform.tfstate`. Estos archivos gestionan la infraestructura de forma aislada; tener una carpeta propia evita que el estado de un proyecto sobrescriba a otro.
* **Orden y Limpieza:** Facilita la gestión del repositorio en GitHub. Un entorno limpio permite que tú o cualquier colaborador localice el archivo `main.tf` rápidamente.
* **Preparación para el Init:** El comando de inicialización (`terraform init`) debe ejecutarse siempre dentro de la carpeta raíz del proyecto para que Terraform reconozca correctamente los archivos de configuración.

### Comando de Creación
Ejecuta el siguiente comando en PowerShell para crear tu espacio de trabajo:

```powershell
$New-Item -Path "C:\" -Name "learn-terraform-azure" -ItemType "directory"$ cd C:\learn-terraform-azure
```
---

## 6. Definición de la Infraestructura (`main.tf`)

En este paso, creamos un archivo de texto plano llamado `main.tf` dentro del directorio del proyecto. Este archivo actúa como el **plano arquitectónico** de nuestra infraestructura; es el documento donde escribimos el código declarativo que Azure interpretará para construir los recursos.

### ¿Por qué se hace esto?
* **Infraestructura como Código (IaC):** En lugar de crear recursos manualmente haciendo clic en portales web (proceso propenso a errores), dejamos constancia escrita de nuestra red. Esto permite versionar el archivo en GitHub, compartirlo con otros equipos y replicar la misma infraestructura exactas veces de forma automática.
* **Declaración de Proveedores:** Especificamos que vamos a usar el proveedor oficial de Azure (`azurerm`) y definimos la versión exacta para asegurar la compatibilidad y estabilidad del proyecto a largo plazo.
* **Gestión de Recursos:** Definimos el primer componente real, el **Resource Group (Grupo de Recursos)**, que servirá como contenedor lógico para todos los elementos que creemos después (como la VNet y la Subnet).

### Código de Configuración (`main.tf`)
Copia el siguiente bloque de código dentro de tu archivo:

```hcl
# Configure the Azure provider
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0.2"
    }
  }

  required_version = ">= 1.1.0"
}

provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "rg" {
  name     = "myTFResourceGroup"
  location = "spaincentral"
}
```

> **Nota sobre la ubicación:** En este ejemplo utilizamos `spaincentral`. Asegúrate de poner la localización en la que tu suscripción te permita crear recursos (puedes consultar las regiones disponibles en tu suscripción de Azure).

---

## 7. Inicialización del Proyecto (`terraform init`)

Una vez que el archivo `main.tf` está listo, el primer comando que debemos ejecutar es `terraform init`. Este paso es esencial para preparar el directorio de trabajo y permitir que Terraform "entienda" las instrucciones que hemos escrito.

### ¿Para qué se hace esto?
* **Descarga de Proveedores (Providers):** Terraform es una herramienta agnóstica; al leer el código, detecta que trabajaremos con Azure y descarga automáticamente el plugin oficial de `azurerm` necesario para comunicarse con su API.
* **Configuración del Backend:** Prepara el espacio donde se almacenará el **archivo de estado** (`terraform.tfstate`). Esta es la base de datos local donde Terraform registrará qué recursos ha creado y cuál es su configuración actual.
* **Verificación de Versiones:** Comprueba que la versión de Terraform instalada y los plugins descargados cumplen con las restricciones de versión que definimos en el bloque `terraform` de nuestro código.

### Comando de Inicialización
Para preparar tu entorno de trabajo, ejecuta:

```powershell
$ terraform init
```

![Captura: Inicialización de Terraform Exitosa](images/message_terraform_init.png)

---

## 8. Formateo y Validación de la Configuración

Antes de proceder con el despliegue en la nube, es una práctica recomendada por **HashiCorp** asegurarse de que nuestro código cumple con los estándares de estilo y es sintácticamente correcto.

### ¿Por qué se hace esto?
* **Consistencia Estética (`terraform fmt`):** Este comando formatea automáticamente tus archivos de configuración para que sigan el estilo oficial de HCL (indentación, alineación de columnas, espacios, etc.). Esto facilita enormemente la lectura del código al trabajar en equipo o al compartir tu proyecto en **GitHub**.
* **Seguridad Sintáctica (`terraform validate`):** Este comando verifica que el archivo `main.tf` no tenga errores internos. Comprueba que los nombres de los recursos sean válidos, que las referencias sean correctas y que no falten argumentos obligatorios. Esto evita que el proceso falle más tarde durante la fase de ejecución.
* **Calidad del Código:** El uso constante de estos comandos demuestra un flujo de trabajo profesional, asegurando que solo subimos a nuestro repositorio código que ha pasado un "control de calidad" previo.

### Comandos de Validación
Para asegurar la integridad de tu configuración, ejecuta:

```powershell
$ terraform fmt
$ terraform validate
```

## 9. Aplicación de la Configuración (`terraform apply`)

El comando `terraform apply` es la orden de ejecución real. A diferencia de otros comandos de consulta, este no solo simula la infraestructura, sino que abre una conexión activa con Azure para construir los recursos definidos en el `main.tf`.

### ¿Por qué es especial este paso?
* **Confirmación de Seguridad:** Por defecto, Terraform volverá a mostrarte un plan de ejecución detallado y se detendrá. Te preguntará: `"Do you want to perform these actions?"`. Esta es una red de seguridad vital para evitar despliegues accidentales o costes inesperados.
* **Interactividad Obligatoria:** El sistema no avanzará hasta que escribas exactamente la palabra **`yes`**. Cualquier otra respuesta abortará la operación de inmediato sin realizar cambios en tu suscripción de Azure.
* **Construcción en Vivo:** Una vez confirmado, verás en tiempo real cómo Azure crea el Grupo de Recursos. Al finalizar, Terraform actualizará automáticamente tu **archivo de estado (`.tfstate`)** para registrar que ese recurso ya está oficialmente bajo su control.

### Comando de Despliegue
Para aplicar los cambios y desplegar tu infraestructura en Azure, ejecuta:

```powershell
$ terraform apply
```

![Captura: Aplicación exitosa en Azure](images/message_terraform_apply.png)
![Captura: Aplicación exitosa en Azure](images/message_apply_group_of_resources.png)

---

> **Nota de Transición:** Tras el éxito del despliegue, es fundamental entender el "cerebro" de la herramienta: cómo Terraform gestiona y recuerda todo lo que acaba de construir.

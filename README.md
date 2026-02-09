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

![Captura: Proceso de Login en la Terminal](img/az_login.png)

> **🔒 Nota de Seguridad:** Es fundamental ocultar el **Subscription ID** y el **Tenant ID** en capturas de pantalla o entornos públicos (como este repositorio) para proteger la privacidad y seguridad de tu cuenta.

### Selección de la Suscripción Activa
Tras la autenticación, el CLI mostrará un listado de todas las suscripciones asociadas a tu cuenta. Es **fundamental fijar la suscripción activa** donde deseamos que Terraform realice los despliegues (por ejemplo, la de **Estudiantes**) para asegurar que los recursos se creen en el entorno correcto y se utilicen los créditos adecuados.

Establece la suscripción de trabajo mediante el siguiente comando sustituyendo el ID por el tuyo:

```powershell
$ az account set --subscription "TU_SUBSCRIPTION_ID_AQUÍ"
```

```powershell
$ az account set --subscription "tu-subscription-id"

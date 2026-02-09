# azure-terraform-iac
# Configuración de Prerrequisitos y Autenticación en Azure

## 1. Prerrequisitos

Antes de iniciar el despliegue de infraestructura, es indispensable preparar el entorno local con las siguientes herramientas y accesos:

* **Suscripción de Azure:** Se requiere una cuenta activa para gestionar los recursos. [cite_start]Puedes obtener una cuenta gratuita o para estudiantes en el siguiente enlace: [Azure Account Setup](https://azure.microsoft.com/en-us/pricing/purchase-options/azure-account?icid=azurefreeaccount)[cite: 2, 3].
* **Instalación de Terraform (Local):** Es necesario descargar e instalar Terraform en tu máquina. 
    > **⚠️ Nota Crítica:** No olvides añadir la ruta del ejecutable a las **variables de entorno del sistema (PATH)** para que el comando sea reconocido globalmente por cualquier terminal[cite: 4].  
    Puedes descargarlo aquí: [Terraform Install](https://developer.hashicorp.com/terraform/install)[cite: 5].
* **Azure CLI:** Se instala para habilitar la comunicación y autenticación entre tu máquina local y la nube de Azure[cite: 6]. Puedes realizar la instalación mediante el siguiente comando en PowerShell:

```powershell
$ Invoke-WebRequest -Uri [https://aka.ms/installazurecliwindows](https://aka.ms/installazurecliwindows) -OutFile .\AzureCLI.msi; Start-Process msiexec.exe -Wait -ArgumentList '/I AzureCLI.msi /quiet'; rm .\AzureCLI.msi
``` [cite: 7]

---

## 2. Autenticación y Gestión de Suscripciones

Una vez instalados los requisitos previos, procedemos a configurar los permisos de acceso local.

### Inicio de Sesión en Azure CLI
Para otorgar los permisos necesarios a nuestra máquina local, realizamos el login mediante el CLI de Azure:

```powershell
$ az login
``` [cite: 10]



> [cite_start]**🔒 Nota de Seguridad:** Es fundamental ocultar o censurar nuestro **Subscription ID** y **Tenant ID** en capturas de pantalla o entornos públicos para proteger la privacidad y seguridad de nuestra cuenta[cite: 19].

### Selección de la Suscripción Activa
[cite_start]Tras la autenticación, el CLI mostrará un listado de todas las suscripciones disponibles asociadas a tu cuenta[cite: 20]. [cite_start]Es fundamental **fijar la suscripción activa** donde deseamos que Terraform realice los despliegues (en este proyecto, la suscripción de **estudiantes**) para asegurar que los recursos se creen en el entorno correcto y se utilicen los créditos adecuados[cite: 21].

Para establecer la suscripción de trabajo, utiliza el siguiente comando sustituyendo el marcador por tu ID real:

```powershell
$ az account set --subscription "tu-subscription-id"
[cite_start]``` [cite: 23]

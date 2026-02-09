# azure-terraform-iac
# Configuración de Prerrequisitos y Autenticación en Azure

[cite_start]Esta sección detalla los pasos iniciales necesarios para preparar el entorno local y establecer una conexión segura con la nube de Azure antes de realizar cualquier despliegue con Terraform[cite: 8, 92].

---

## [cite_start]1. Prerrequisitos del Sistema [cite: 1]

Para comenzar con este proyecto, es indispensable contar con las siguientes herramientas y accesos configurados:

* [cite_start]**Suscripción de Azure:** Se requiere una cuenta activa para la gestión de recursos[cite: 2]. [cite_start]Puedes obtener una cuenta gratuita o para estudiantes en el siguiente enlace: [Azure Account](https://azure.microsoft.com/en-us/pricing/purchase-options/azure-account?icid=azurefreeaccount)[cite: 3].
* [cite_start]**Instalación Local de Terraform:** Es necesario descargar e instalar Terraform de forma local[cite: 4, 5]. 
    > [cite_start]**⚠️ Nota Crítica:** No olvides añadir la ruta de instalación a las **Variables de Entorno del sistema (PATH)** para que la terminal reconozca el comando `terraform` desde cualquier directorio[cite: 4].
* [cite_start]**Azure CLI:** Esta herramienta se instala para poder autenticarnos con los servicios de Azure de forma local[cite: 6]. [cite_start]Puedes realizar la instalación mediante el siguiente comando en PowerShell[cite: 7]:

```powershell
$ Invoke-WebRequest -Uri [https://aka.ms/installazurecliwindows](https://aka.ms/installazurecliwindows) -OutFile .\AzureCLI.msi; Start-Process msiexec.exe -Wait -ArgumentList '/I AzureCLI.msi /quiet'; rm .\AzureCLI.msi
```

---

## 2. Autenticación y Gestión de Suscripciones

[cite_start]Una vez instalados los requisitos previos, procedemos a configurar los permisos de acceso local para que Azure permita el inicio de sesión y la gestión de recursos[cite: 9].

### Inicio de Sesión (`az login`)
[cite_start]Para instalar los permisos que Azure necesita para autenticarse de forma local, realizamos el inicio de sesión mediante el CLI de Azure[cite: 9, 10]:

```powershell
$ az login
```

![Captura: Proceso de Login en la Terminal](img/login_cli.png)

> [cite_start]**🔒 Nota de Seguridad:** Es fundamental ocultar nuestro **Subscription ID** y el **Tenant ID** en entornos públicos para proteger la privacidad y seguridad de nuestra cuenta[cite: 19].

### Selección de la Suscripción Activa
[cite_start]Una vez autenticados, el CLI mostrará un listado de las suscripciones disponibles en la cuenta[cite: 20]. [cite_start]Es **fundamental fijar la suscripción activa** donde deseamos que Terraform realice los despliegues (en este caso, la de **estudiantes**) para asegurar que los recursos se creen en el entorno correcto[cite: 21].

[cite_start]Para establecer la suscripción de trabajo, utilizamos el siguiente comando[cite: 22, 23]:

```powershell
$ az account set --subscription "TU_SUBSCRIPTION_ID_AQUÍ"
```stión de Suscripciones

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

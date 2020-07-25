![Oil & Gas - Guía de Implementación](https://github.com/DJuanes/dynocard/blob/master/imagenes/oil%26gas-deployment.jpg "Michael C. Rygel / CC BY-SA (https://creativecommons.org/licenses/by-sa/3.0)")




[TOC]



## 1 Guía de Implementación
Este documento explica cómo implementar la solución dynocard oil & gas utilizando una plantilla ARM. En este documento se explican dos formas de implementar la solución:

-Usando Azure portal

-Usando Azure CLI

Este documento explica los parámetros de entrada, los parámetros de salida y los puntos que deben tenerse en cuenta al implementar la plantilla ARM.

## 2  ¿Qué son las regiones emparejadas?
Azure opera en múltiples geografías de todo el mundo. Una geografía de Azure es un área definida del mundo que contiene al menos una Azure Region. Una región de Azure es un área dentro de una geografía, que contiene uno o más centros de datos.
Cada región de Azure está emparejada con otra región dentro de la misma geografía, juntas formando un par regional. La excepción es Brasil Sur, que se combina con una región fuera de su geografía.

IoT Hub soporta conmutación por error manual en regiones geo-emparejadas:

|           | **Geografia**         | **Regiones emparejadas**
| ------    | -------------         | ---------------------------------------| 
|1          | North America         | East US 2         -     Central US     | 
|2          | North America         | Central US         -   East US 2       | 
|3          | North America         | West US 2          -   West Central US |  
|4          | North America         | West Central US    -   West US 2       | 
|5          | Canada                | Canada Central     -   Canada East     |
|6          | Canada                | Canada East        -   Canada Central  | 
|7          | Australia             | Australia East     -   Australia South East|
|8          | Australia             | Australia South East -   Australia East|
|9          | India                 | Central India     -     South India    | 
|10         | India                 | South India       -    Central India   |
|11         | Asia                  | East Asia         -     South East Asia|
|12         | Asia                  | South East Asia   -    East Asia       |
|13         | Japan                 | Japan West        -     Japan East     |
|14         | Japan                 | Japan East        -     Japan West      |
|15         | Korea                 | Korea Central      -     Korea South    |
|16         | Korea                 | Korea South         -   Korea Central   | 
|17         | UK                    | UK South             -   UK West        |
|18         | UK                    | UK West               - UK South        |



## 3 Parámetros de Entrada de la Plantilla ARM

En la sección de parámetros de la plantilla, especifique qué valores puede ingresar al implementar los recursos. Estos valores de parámetros le permiten personalizar la implementación al proporcionar valores personalizados para un entorno (como desarrollo, test y producción). Está limitado a 255 parámetros en una plantilla. Puede reducir el número de parámetros mediante el uso de objetos que contienen múltiples propiedades.

| **Nombre de Parametro** | **Descripción**   | **Valores Permitidos** | **Default Values**   |
| -------------       | -------------       | -----------------     | ------------         |
| **Solution Type** | Elija el tipo de implementación de la solución en el menú desplegable | Basic, Standard, Premium | Basic|
| **Edge VM + Simulator VM** | Elija Yes/No para agregar Modbus VM como parte de la implementación de la solución | Yes, No    | No |
| **mlVM**   | Elija Yes/No para agregar ML VM como parte de la implementación de la solución | Yes, No   | No |
| **Data Lake Location** | Elija la ubicación de Data Lake Store | Eastus2,Centralus, Northeurope, westeurope  | Eastus2 |
| **Machine learning Location** | Elija la ubicación para la cuenta machine learning | Eastus2, Australieneast, Southeastasia, Westcentralus, Westeurope | eastus2 |
| **Geo Paired Primary Region** | Elija la región geo-emparejada si ha seleccionado la opción estándar o premium en el parámetro de entrada del tipo de solución | EastUS2,CentralUS,WestUS2,WestCentralUS,CanadaCentral,CanadaEast,         AustraliaEast,AustraliaSouthEast,CentralIndia,SouthIndia,EastAsia,               SouthEastAsia,JapanWest,JapanEast,KoreaCentral,KoreaSouth,UKSouth,               UKWest  | eastus2|
| **oms-region** | Elija la ubicación para OMS Log Analytics | australiasoutheast, canadacentral, centralindia, eastus, japaneast, southeastasia, uksouth, westeurope    | EastUs |
| **appInsightsLocation**   | Especifique la región para application insights | eastus, northeurope,  southcentralus, southeastasia, westeurope, westus2 | westus2 |
| **sqlAdministratorLogin**     | Proporcione el nombre de usuario para el servidor SQL, tome nota del nombre de usuario, se utilizará más adelante. | sqluser | sqluser |
| **sqlAdministratorLoginPassword**   | Proporcione la contraseña para el servidor SQL, tome nota de la contraseña, se utilizará más adelante. | Password@1234  | Password@1234 |
| **Azure Login** | Proporcione el nombre de usuario de inicio de sesión del portal Azure. Esto será útil para autenticar la cuenta del data lake store en la sección de salida de stream analytics. | user@domain.com | user@domain.com |
| **Azure Password** | Proporcione la contraseña de inicio de sesión del portal Azure. Esto será útil para configurar los módulos en el dispositivo iotedge a través del script iot-edge.sh. | Contraseña | Contraseña |
| **tenantId** | Identificación del inquilino para la suscripción. Esto será útil para autenticar la cuenta del data lake store en la sección de salida de stream analytics. | xxxxxxxx-xxxx-xxxx-xxxx-c5e6exxxcd38          | xxxxxxxx-xxxx-xxxx-xxxx-c5e6exxxcd38 |
| **vmsUsername** | Nombre de usuario para iniciar sesión en Modbus VM y Edge VM. Tome nota del nombre de usuario, esto se utilizará más adelante. | adminuser  | Adminuser      |
| **vmsPassword** | Proporcione la contraseña de inicio de sesión de VM, tome nota de la contraseña, esto se utilizará más adelante. | Password@1234    | Password@1234  |
| **Web App Buildfile Url** | Utilice el archivo de compilación de la aplicación web que se encuentra en la carpeta /builds guardandolo en un almacenamiento público. | https://github.com/DJuanes/iot-edge-dynocard/raw/master/builds/DynoCardAPI_with_appinsights.zip    | https://github.com/DJuanes/iot-edge-dynocard/raw/master/builds/DynoCardAPI_with_appinsights.zip     |
| **Storage Uri** | Utilice el archivo sql bacpac que se encuentra en la carpeta /builds guardandolo en un almacenamiento público. | https://projectiot.blob.core.windows.net/oilgas-iot/db4cards.bacpac | https://projectiot.blob.core.windows.net/oilgas-iot/db4cards.bacpac |
| **Plcsimulatorv1** | Proporcione el archivo exe Plcsimulatorv1 que se encuentra en la carpeta /builds guardandolo en un almacenamiento público. | https://projectiot.blob.core.windows.net/oilgas-iot/ModbusSimulator/SimSetup.msi          | https://projectiot.blob.core.windows.net/oilgas-iot/ModbusSimulator/SimSetup.msi |
| **vcredist** | Proporcione el archivo exe vcredist que se encuentra en la carpeta /builds guardandolo en un almacenamiento público. | https://projectiot.blob.core.windows.net/oilgas-iot/ModbusSimulator/vcredist_x86.exe  | https://projectiot.blob.core.windows.net/oilgas-iot/ModbusSimulator/vcredist_x86.exe      |
| **Plcsimulatorv2** | Proporcione el archivo exe Plcsimulatorv2 que se encuentra en la carpeta /builds guardandolo en un almacenamiento público. | https://projectiot.blob.core.windows.net/oilgas-iot/ModbusSimulator/ModRSsim2.exe    | https://projectiot.blob.core.windows.net/oilgas-iot/ModbusSimulator/ModRSsim2.exe  |
| **Device Name** | Proporcione el nombre del dispositivo IoT Edge | nombre de su dispositivo | nombre de su dispositivo |

**Nota**: Los valores permitidos se actualizan en función de la disponibilidad de recursos basados en la región. Microsoft puede introducir la disponibilidad de recursos Azure en el futuro. Se puede encontrar más información en https://azure.microsoft.com/en-in/status/

## 4 Inicio

Azure Resource Manager le permite aprovisionar sus aplicaciones mediante una plantilla declarativa. En una sola plantilla, puede implementar múltiples servicios junto con sus dependencias. La plantilla consta de JSON y expresiones que puede usar para construir valores para su implementación. Utilice la misma plantilla para implementar repetidamente la solución durante cada etapa del ciclo de vida de la aplicación.
Resource Manager proporciona una capa de administración coherente para realizar tareas a través de Azure PowerShell, Azure CLI, Azure Portal, API REST y SDK de cliente.

*   Implemente, administre y monitoree todos los recursos para su solución como grupo, en lugar de manejar estos recursos individualmente.
* Implemente repetidamente su solución durante todo el ciclo de vida de desarrollo y tenga confianza en que sus recursos se implementarán en un estado coherente.
* Administre su infraestructura a través de plantillas declarativas en lugar de scripts.
* Defina las dependencias entre los recursos para que se implementen en el orden correcto.
* Aplique el control de acceso a todos los servicios en su grupo de recursos porque el Control de acceso basado en roles (RBAC) está integrado de forma nativa en la plataforma de administración.
* Aplique etiquetas a los recursos para organizar lógicamente todos los recursos en su suscripción.

### 4.1  Implementación de Plantillas ARM con Azure Portal

1.  Haga clic en la siguiente URL de repo **Git hub**:

**https://github.com/DJuanes/iot-edge-dynocard/tree/master**

2. Seleccione **main-template.json** del branch **master** como se muestra en la siguiente figura:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/5.png)

3. Seleccione **Raw** desde la esquina superior derecha:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/6.png)

4. **Copy** la plantilla y **paste** en su portal azure para implementación de plantilla:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/7.png)

Para implementar una plantilla para Azure Resource Manager, siga los pasos a continuación:

1.  Vaya a **Azure portal**.

2.  Navegue hasta **Create a resource (+)**, busque **Template deployment**.

3.  Haga clic en el botón **Create** y luego en **Build your own Template in the editor** como se muestra en la siguiente figura:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/8.png)

4.  La página **Edit template** se muestra como en la siguiente figura: 

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/9.png)

5.  **Replace / paste** la plantilla y haga clic en el botón **Save**:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/10.png)

6.  La página **Custom deployment** se muestra como en la siguiente figura: 

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/11.png)

### 4.1.1 Inputs
Estos valores de parámetros le permiten personalizar la implementación. Sus parámetros permiten elegir el tipo de solución, la región y las credenciales para autenticar la base de datos SQL y las máquinas virtuales.

7.  Si elige **yes** para **Edge VM + Simulator VM** entonces **Edge VM and Simulator VMs** serán implementados con la solución.

8.  Si elige **No** entonces el **Edge VM + Simulator VM** no serán implementados con la solución. Tiene que implementar manualmente los módulos IoT Edge en Edge VM. Para obtener más información, consulte la sección 5.3 Realizar la operación de dispositivo doble en Edge VM.

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/12.png)

9.  Si elige **yes** para **ML VM** entonces el **docker** pre-instalado será implementado con la solución.

10. Si elige **No** para **ML VM** entonces el **docker** pre-instalado no será implementado con la solución.

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/13.png)

**Parametros para la Solución Basica**

11. Implemente la plantilla proporcionando los parámetros en la configuración de implementación personalizada como se muestra en la siguiente figura:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/parameters-basic11.png)
![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/parameters12.png)

**Parametros para la Solución Estándard**

12. Si desea implementar el núcleo con monitoreo, debe ingresar los siguientes parámetros:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/parameters-standard11.png)
![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/parameters12.png)

**Parametros para la Solución Premium**

13. Si desea implementar el núcleo con seguridad aumentada y monitoreo, debe ingresar los siguientes parámetros:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/parameters-premium11.png)
![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/parameters12.png)

14. Una vez que se hayan ingresado todos los parámetros, seleccione la casilla de verificación **terms and conditions** y haga clic en **Purchase**.

15. Después de la implementación exitosa de la plantilla ARM, los siguientes recursos son creados en un **Resource Group**.

16. Cuando elige el modelo de costo como **Standard** y **Modbus VM** y **ML VM** como **Yes**, se instalarán los siguientes componentes:

* 3-Virtual Machines (2 windows & 1 Linux)
Windows VMS
**Dyno card VM** con software preinstalado para la dyno card VM.
**ML VM** con docker pre-instalado.
Linux VM
**Edge VM** es usada para crear dispositivos IoT Edge e instalar módulos en dispositivos IoT Edge.
* 2-Web App
* 1-Application Insights
* 1-Data Lake Storage
* 1-IoT HUB
* 1-Log Analytics
* 1-Logic App
* 1-Service Bus Namespace
* 2-SQL Database
* 1-Storage Account
* 1-Stream Analytics job
* 1-Traffic Manager

17. Una vez que la solución se haya implementado exitosamente, navegue al grupo de recursos, seleccione el **resource group** creado para ver la lista de recursos que se crean, como se muestra en la siguiente figura:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/17.png)
![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/18.png)
![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/19.png)
![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/20.png)

### 4.1.2 Outputs
La sección Outputs consta de los valores que se devuelven de la implementación. Los valores de salida se pueden usar para otros pasos en la configuración de la solución
1. Vaya a **Resource group** -> haga clic en **deployments** -> seleccione **Microsoft Template**:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/21.png)

2. Haga clic en **outputs**:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/22.png)

### 4.2 Implementación de Plantillas ARM usando Azure CLI

Azure CLI se usa para implementar sus recursos en Azure. La plantilla de Resource Manager que implemente puede ser un archivo local en su máquina o un archivo externo que se encuentra en un repositorio como GitHub.
Azure Cloud Shell es un shell interactivo accesible para el navegador para administrar los recursos de Azure. Cloud Shell permite el acceso a una experiencia de línea de comandos basada en navegador creada teniendo en cuenta las tareas de administración de Azure.
La implementación puede realizarse en Azure Portal a través de Azure Cloud Shell.
Personalizar el archivo main-template.parameters.json 

1. Inicie sesión **Azure portal.** 
2. Abra la línea de comandos. 
3. Seleccione el entorno **Bash (Linux)**.
4. Seleccione su **Subscription** preferida de la lista desplegable.  
5. Haga clic en el botón **Create Storage**. 
6. Copie **main-template.json** y **main-template.parameters.json** a su Cloud Shell antes de actualizar los parámetros. 
7. Cree **main-template.json** usando el siguiente comando: 

**vim main-template.json**

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/23.png)

8. Pegue su **main-template.json** en el editor como se muestra a continuación y guarde el archivo:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/24.png) 

9. Pegue su **main-template.parameters.json** en el editor. 
10. Actualice los siguientes parámetros en el archivo main-template.json 
* Solution Type 
*   Edge VM + Simulator VM
*   MLVM
*   dataLakelocation
*   machineLearningLocation
*   geo-Paired-Primary-region
*   oms-region
*   appInsightsLocation
*   sqlAdministratorLogin
*   sqlAdministratorLoginPassword
*   azureLogin
*   tenantId
*   vmsUsername
*   vmsPassword

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/25.png)

11. Cree un Resource Group para la solución 
12. Use el comando **az group create** para crear un **Resource Group** en su región.

**Descripción:** Para crear un grupo de recursos, use el comando az group create, utilizando el parámetro de nombre para especificar el nombre del grupo de recursos (-n) y el parámetro de ubicación para especificar la ubicación (-l).

**Sintaxis: az group create -n <nombre del resource group> -l <ubicacion>**.

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/26.png)

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/27.png)

Ejecute el despliegue de la plantilla
Use el comando **az group deployment create** para implementar la plantilla ARM

**Descripción:** Para implementat la plantilla ARM, necesita dos archivos: 

**main-template.json** –contiene el recurso y sus dependencias que se aprovisionarán desde la plantilla ARM.

**main-template.parameters.json** –contiene los valores de entrada necesarios para aprovisionar SKUs respectivos y otros detalles.

**Sintaxis:  az group deployment create --template-file './<main-template.json filename>' --parameters '@./<main-template.parameters.json filename>' -g <proporcione el nombre del grupo de recursos que se creo en la seccion anterior> -n deploy >> <proporcione el nombre de archivo para la salida>**

**Ex: az group deployment create --template-file './main-template.json' --parameters '@./main-template.parameters.json' -g oilandgas-iot -n deploy >> outputs.txt**

 ![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/28.png)

15. La implementación puede demorar entre 15 y 20 minutos, dependiendo del tamaño.
16. Después de una implementación exitosa, puede ver los resultados de la siguiente manera:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/29.png)

## 5 Pasos Posteriores a la Implementación

### 5.1 Verificar Contenedores en EdgeVM y en Azure Portal

1. Vaya a **Resource Group** -> haga clic en **iotEdge VM**:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/30.png)

2. Copie la dirección IP pública de iotEdge VM:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/31.png)

3. Inicie sesión en la VM a través de putty.

4. Pegue la  IP pública en el nombre del host (o dirección IP) y haga clic en **open**:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/32.png)

5. Ingrese las credenciales:

Ingrese el usuario de inicio de sesión como: **adminuser**

Ingrese la contraseña como: **Password@1234**

6. Una vez que el inicio de sesión se realizó correctamente, se muestra el siguiente mensaje de bienvenida:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/33.png)

7. Aquí puede verificar el dispositivo y los módulos de dispositivo en IoT Edge VM

**docker ps** 

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/moduleslist.png)

8. Vaya a **Resource Group** -> haga clic en **iothub26hs3**:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/35.png)

9. Navegue hasta la hoja **IoT Edge** en la sección **Automatic Device Management**:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/iotedge.png)

10. Aquí creamos y configuramos el dispositivo desde IoT Edge VM. Haga clic en el dispositivo **iot-dynocard-demo-device_1** para ver los módulos:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/iotedge.png)

11. Podemos ver los módulos creados en los detalles del dispositivo: 

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/modules-running.png)

### 5.2 Actualizar la clave principal del dispositivo IoT Hub en Web API Application Settings

1. Vaya a **Resource Group** -> haga clic en **iothub26hs3**:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/46.png)

2. Navegue hasta la hoja **IoT Edge** en la sección **Automatic Device Management**:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/iotedge.png)

3. Haga clic en el dispositivo **iot-dynocard-demo-device_1** como se muestra a continuación y copie la cadena de conexión-clave principal:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/48.png)

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/49.png)

4. Vaya a **Resource Group** -> abra la aplicación web principal **webapi26hs3** en app service:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/50.png)

5. Navegue hasta la hoja **Application settings**:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/51.png)

6. Haga clic en **+ Add new setting** en Application settings, ingrese nombre y valor en la nueva configuración:

Nombre: **DeviceConnectionString**

Valor: **[Device connection string-primary key]**

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/52.png)

7. Luego haga clic en **Save**:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/53.png)

### 5.3 Realizar la operación de dispositivos gemelos en Edge VM [Opcional]

**Nota**: Este paso es obligatorio solo cuando el usuario desea ejecutar el simulador en su propio sistema. Si Modbus VM se implementa como parte de la solución, omita este paso y continúe con el siguiente.

Es necesario realizar la operación de dispositivos gemelos en Modbus IoT Edge Module cuando la solución se implementa con una opción Modbus VM como **No**.

Instale los módulos de IoT Edge ejecutando **iot-edge-Manual.sh** dentro de los scripts del repositorio de GitHub proporcionando los parámetros de entrada requeridos.
https://github.com/DJuanes/iot-edge-dynocard/scripts/iot-edge-Manual.sh

Necesita actualizar la dirección IP de la conexión esclava en la configuración del módulo Modbus en IoT Edge Modules con la dirección IP real del simulador.


1. Vaya a **Resource Group** -> haga clic en **iotEdge VM**:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/54.png)

2. Copie la dirección IP pública de iotEdge VM:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/55.png)

3. Inicie sesión en la VM a través de putty.

4. Pegue la  IP pública en el nombre del host (o dirección IP) y haga clic en **open**:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/56.png)

5. Ingrese las credenciales:

Ingrese el usuario de inicio de sesión como: **adminuser**

Ingrese la contraseña como: **Password@1234**

6. Una vez que el inicio de sesión se realizó correctamente, se muestra el siguiente mensaje de bienvenida:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/57.png)

7. Aquí puede verificar el dispositivo y los módulos de dispositivo en IoT Edge VM

**docker ps**

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/moduleslist.png)

8. Verifique los logs del contenedor Modbus ejecutando el siguiente comando:

**docker logs modbus**

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/59.png)

Según el diagrama anterior, la conexión esclava es 52.186.11.164 y está intentando conectarse con 52.186.11.164 que no está disponible. Necesitamos actualizar la dirección IP de la conexión esclava con la dirección IP correcta usando una operación doble.

10. Vaya a **Resource Group** y elija **IoT Hub**:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/60.png)

11. Haga clic en **IoT Hub** y navegue hasta la hoja **IoT Edge** en la sección **Automatic device management**:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/61.png)

12. Haga clic en el dispositivo **iot-dynocard-demo-device_1** para verificar los módulos:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/iotedge.png)

13. Podemos ver los **modules** creados en **device details**:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/modules-running.png)

14. Haga clic en el módulo **modbus**:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/64.png)

15. Haga clic en **Module twin**:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/65.png)

16. Cambie la dirección IP de la conexión esclava y haga clic en **Save**:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/66.png)

17. En este escenario, la dirección IP cambió a 104.42.153.165:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/67.png)

18. Ahora regrese a Edge VM para verificar la dirección IP de la conexión esclava.

19. Detenga el contenedor Modbus escribiendo el siguiente comando:

**docker stop modbus**

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/68.png)

20. Inicie el contenedor Modbus escribiendo el siguiente comando:

**docker start modbus**

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/69.png)

21. Ahora verifique la cadena de conexión esclava verificando los logs de Modbus:

**docker logs modbus**

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/70.png)

Ahora la dirección IP de la conexión esclava se actualiza con la nueva dirección IP.

## 6  Machine Learning Configuration

Azure Machine Learning es una solución integral de analitica avanzada y ciencia de datos. Permite a los científicos de datos preparar datos, desarrollar experimentos e implementar modelos a escala de nube. El documento repasa los siguientes temas:

    1 - Cuenta AML Experimentation
    2 - Cuenta AML Model Management
    3 - AML Workbench
    4 - Configuración de los distintos objetos ML.
    5 - Entrenamiento e implementación de un modelo de muestra.

**Experimentation service** permite a los científicos de datos ejecutar sus experimentos localmente, en contenedores Docker o en clusters de Spark mediante una configuración simple. Gestiona el historial de ejecución, proporciona control de versiones y permite compartir y colaborar.

**Azure Machine Learning Workbench** es un front-end para una variedad de herramientas y servicios, incluidos los servicios de Azure Machine Learning Experimentation y Model Management.

Workbench es un conjunto de herramientas relativamente abierto. Primero, puede usar casi cualquier framework de machine learning basado en Python, como Tensor flow o scikit-learn. En segundo lugar, puede entrenar e implementar sus modelos on-premises o en Azure. Workbench también incluye un excelente módulo de preparación de datos. Tiene una interfaz de arrastrar y soltar que lo hace fácil de usar, pero sus características son sorprendentemente sofisticadas.

### 6.1 Agregar usuario actual al grupo de usuarios de Docker

1. Vaya a **AzuremlVM** desde **Resource Group** y copie la **dirección IP pública** de la VM:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/154.png)

2. Abra **Remote desktop connection** ,coloque la dirección IP y haga clic en **Connect**:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/155.png)

3. Ingrese las credenciales:

Ingrese el usuario de inicio de sesión como: **adminuser**

Ingrese la contraseña como: **Password@1234**

**Nota:** Las credenciales pueden variar según la implementación.

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/156.png)

4. Abra **PowerShell as Administrator** y ejecute el siguiente comando para agregar el usuario actual al grupo de usuarios de docker:
**Add-LocalGroupMember -Member $env:username -Name docker-users**
![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/157.png) 

5. Salga de Windows y vuelva a iniciar sesión:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/158.png)

6. Busque en el menú de búsqueda y haga clic en **“Docker for Windows”**:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/159.png)

7. Cuando se le solicite habilitar **Hyper-V and Containers feature**, haga clic en **ok**:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/160.png)

8. Esto reiniciará el sistema de Windows. Inicie sesión de nuevo en VM con las mismas credenciales que las anteriores.

9. Cuando Docker se está ejecutando, se abrirá una ventana emergente en la barra de tareas que indica que Docker se está ejecutando:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/161.png)

### 6.2 Instalar ML Workbench

1. Instale ML workbench utilizando el archivo de configuración de la ruta a continuación:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/162.png) 

2. Haga doble clic en el archivo **amlWorkbenchSetup.msi**.
3. Haga clic en **Continue**:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/163.png)

4. Haga clic en **Install**:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/164.png)

5. La instalación de ML Workbench demorará entre 30 y 45 minutos.
6. Una vez que la instalación se complete con éxito, haga clic en **Launch Azure Machine Learning Workbench**.
7. Haga clic en **Sign in with Microsoft**:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/165.png)

8. Proporcione las credenciales de inicio de sesión de Azure Portal. Una vez que el inicio de sesión se completa con éxito se mostrará la página a continuación: 

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/166.png)

9. Haga clic en **File > Open Command Prompt**.

### 6.3 Inicio de sesión en Azure Portal

10. Inicie sesión en el portal con el siguiente comando:
**az login -t <tenant ID>**

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/167.png)

11. Abra el navegador Edge y vaya a la página **https://microsoft.com/devicelogin**, proporcione el código y haga clic en **Continue**:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/168.png)

12. Proporcione las credenciales de inicio de sesión de Azure Portal, una vez que la autenticación sea exitosa, regrese al símbolo del sistema.
13. Enumere las suscripciones disponibles con el siguiente comando:

**az account list -o table**

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/169.png)

14. Establezca la suscripción actual como predeterminada utilizando el siguiente comando:

**az account set -s <ingrese su sub id aqui>**

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/170.png)

15. Verifique la suscripción predeterminada utilizando el siguiente comando:

**az account show**

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/171.png)

### 6.4 Lista de Componentes de Entorno

16. Ubicación de anaconda env:

**conda env list**

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/172.png)

17. Lista de paquetes python:

**pip freeze**

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/173.png)

18. Versiones de componentes cli:

**az -v**

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/174.png) 

19. Todos los cmd cli para machine learning:

**az ml -h**

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/175.png) 

### 6.5 Crear proyecto ML

1. Cree un proyecto ML con el siguiente comando:

**az ml project create --name mlproject --workspace mlworkspacerk23l --account oilgasexpaccrk23l --resource-group oilandgas-coresolution-267 --path c:\mlproject**

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/176.png)

2. Una vez que se crea el proyecto, actualice la página de inicio en la GUI, donde podemos ver el nombre del proyecto en el espacio de trabajo:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/177.png)

### 6.6 Descargar Repositorio Git

1. Descargue el repositorio de la URL a continuación y extráigalo a una ruta específica:

**https://github.com/DJuanes/iot-edge-dynocard**

2. Haga clic en **Download Zip**:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/178.png)

3. Extraiga el archivo zip descargado del proyecto en el subdirectorio ml del proyecto:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/179.png) 

4. Cambie el directorio:

**cd C:\mlproject\iot-edge-dynocard-master\code\containers\edge_ml** 

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/180.png)

### 6.7 Enviar experimento train4dc.py como destino local

5. Ejecute el experimento localmente con el siguiente comando:

**az ml experiment submit -c local train4dc.py**

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/181.png)

6. Ahora vuelva a la GUI. Haga clic en **run history** en el panel izquierdo y aparecerá en la página como se muestra a continuación:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/182.png)

7. Haga clic en **Run** y se abrirá una página como se muestra a continuación: 

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/183.png)

8. Seleccione **model4dc.pkl** y haga clic en **Download**: 

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/184.png)

9. Elija la ruta para guardar el archivo (esta ruta será el directorio de trabajo de ML Workspace, Ej: C:\iot-edge-dynocard-master\code\container\edge_ml). Haga clic en **save**:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/185.png) 

10. Haga clic en **Yes** cuando se le solicite la advertencia **Confirm Save As**:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/186.png)

### 6.8 Instalar azureml.datacollector

11. Instale azureml.datacollector usando el siguiente comando:

**pip install azureml.datacollector**

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/187.png)

### 6.9 Enviar experimento score4dc.py con destino local

12. Envíe el experimento con Docker local como destino utilizando el siguiente comando:

**az ml experiment submit -c local score4dc.py**

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/188.png)

13. Vaya a la GUI y consulte la página **Run History**:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/189.png)

14. Haga clic en **recently completed run job**. Se abrirá una página como se muestra a continuación:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/190.png)

15. Seleccione el archivo **Service_schema.json** y haga clic en **Download**:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/191.png) 

16. Descargue el archivo al directorio de trabajo actual.

### 6.10 Establecer cuenta Model Management

Machine learning de Azure usa dos cuentas. La primer cuenta realiza un seguimiento de los experimentos y la segunda cuenta hace un seguimiento de los modelos. Los siguientes comandos de Azure CLI se pueden usar para validar sus cuentas y configurar su cuenta de administración del modelo en el directorio del proyecto:

**Syntax:  az ml account modelmanagement set -n <modelmanagementaccountname> -g <resourcegroupname>**

**Ex: az ml account modelmanagement set -n mmact4pumps -g rg4pumps**

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/192.png)

### 6.11 Configuración de la Implementación

17. Cree un entorno de desarrollo con el siguiente comando:

**Syntax: az ml env setup -l eastus2 -n <Uniquename> -g <resourcegroup name> --debug --verbose**

**az ml env setup -l eastus2 -n mldevcofigex -g mldevconfigrg --debug --verbose**

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/193.png)

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/194.png)

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/195.png)


18. Podemos verificar el progreso de la implementación con el siguiente comando. Espere hasta que el estado de aprovisionamiento se muestre como exitoso:

**az ml env show -g mldevconfigrg -n mldevcofigex**

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/196.png)

19. Para establecer como default utilice el siguiente comando:

**az ml env set -g mldevconfigrg -n mldevcofigex**

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/197.png)

20. Para mostrar entorno default use el siguiente comando:

**az ml env show**

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/198.png)

21. La implementación anterior se despliega debajo de los recursos en azure. Podemos verificar los recursos desplegados en el portal:
* 1- Azure Container Registry
* 2- Storage Accounts
* 1- Application Insights

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/199.png)

### 6.12 Registrar proveedores

22. Registre proveedores en el portal usando los comandos a continuación:

**az provider register -n Microsoft.MachineLearningCompute**

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/200.png)

**az provider register -n Microsoft.ContainerRegistry**

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/201.png)

**az provider register -n Microsoft.ContainerService**

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/202.png)

### 6.13 Ejecutar experimento como destino Docker

23. Ejecute el siguiente comando para enviar el experimento con destino Docker:

**az ml experiment submit -c docker train4dc.py**

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/203.png)

Cuando el proceso se esté ejecutando, abra un nuevo símbolo del sistema desde **ML GUI > File > Open Command Prompt**. Este proceso tomará alrededor de 15 minutos de tiempo.

24. **Run docker ps –a** para verificar la ejecución de contenedores docker:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/204.png)

25. Una vez completado el proceso, el resultado será el siguiente:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/205.png)

26. Vaya al historial de ejecución de la GUI y verifique **Run Number** recientemente creado:

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/206.png)

### 6.14 Implementar Web Service

27. Implemente el web service con el siguiente comando:

**az ml service create realtime -m model4dc.pkl -f score4dc.py -r python –n websvc4dc**

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/207.png)

28. Verifique el contenedor docker con el siguiente comando:

**docker ps -a**

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/208.png)

### 6.15 Probar Web Service

29. Ejecute el siguiente comando para probar el servicio web utilizando datos de muestra:

**az ml service run realtime -i websvc4dc -d "{ \"Id\": 0, \"Timestamp\": \"2018-04-04T22:42:59+00:00\", \"NumberOfPoints\": 400, \"MaxLoad\": 19500, \"MinLoad\": 7500, \"StrokeLength\": 1200, \"StrokePeriod\": 150, \"CardType\": 0,\"CardPoints\": [{\"Load\": 11744,\"Position\": 145 }] }"**

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/209.png)

30. Verifique los logs ejecutando el siguiente comando:

**Syntax:  az ml service logs realtime –i websvc4dc**

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/210.png)

![alt text](https://github.com/DJuanes/iot-edge-dynocard/blob/master/images/211.png)

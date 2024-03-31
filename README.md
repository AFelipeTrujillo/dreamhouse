# Salesforce
Dreamhouse, a project with Apex / Salesforce

## Install via Docker

For olds macs

[https://developer.salesforce.com/docs/atlas.en-us.sfdx_setup.meta/sfdx_setup/sfdx_setup_docker.htm](https://developer.salesforce.com/docs/atlas.en-us.sfdx_setup.meta/sfdx_setup/sfdx_setup_docker.htm)

`docker pull salesforce/cli:latest-rc-slim`
`docker run -it salesforce/cli:latest-rc-slim`

## Docker Compose

~~~
version: '3.8'
services:
  salesforce:
    image: salesforce/cli:latest-rc-slim
    working_dir: /home
    volumes:
      - ./src:/home
    stdin_open: true
    tty: true

~~~

## Create a project

`sf project generate --name MyProject`

## Instal NVM

[https://github.com/nvm-sh/nvm?tab=readme-ov-file#installing-and-updating](https://github.com/nvm-sh/nvm?tab=readme-ov-file#installing-and-updating)

~~~
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
~~~

~~~ 
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm 
~~~


## Auth App (SFDX:Authorize an Org)

Si estás utilizando Salesforce CLI dentro de un contenedor Docker y deseas autorizar una organización de Salesforce, pero utilizando el navegador de tu máquina host (ya que el contenedor normalmente no tiene una interfaz gráfica de usuario para abrir un navegador), puedes seguir estos pasos para realizar la autorización utilizando un flujo de autenticación basado en la web.

### 1. Preparar el Contenedor Docker

Asegúrate de que tu contenedor Docker tenga Salesforce CLI instalado y que los puertos necesarios estén expuestos. Salesforce CLI utiliza un servidor web local durante el proceso de autorización para recibir el callback de Salesforce después de la autenticación. Por defecto, este servidor escucha en el puerto 1717.

Si no has expuesto el puerto cuando creaste el contenedor, puedes hacerlo al ejecutar el contenedor con la opción `-p`. Por ejemplo:

```sh
docker run -it -p 1717:1717 tuImagenDeDocker
```

### 2. Usar el Modo de Inicio de Sesión que Devuelve un URL

En lugar de permitir que Salesforce CLI intente abrir automáticamente el navegador, puedes usar un modo que te proporcione un URL para copiar y pegar en el navegador de tu host manualmente. Esto se logra con el comando `auth:device:login`. Ejecuta este comando dentro de tu contenedor:

```sh
sfdx auth:device:login -d -a AliasParaTuOrg
```

- `-d` indica a Salesforce CLI que use el flujo de autenticación del dispositivo.
- `-a AliasParaTuOrg` establece un alias para tu organización, reemplaza `AliasParaTuOrg` con el nombre de alias que prefieras.

### 3. Copiar el URL en el Navegador del Host

El comando anterior generará un mensaje que incluye un URL. Deberás copiar este URL y pegarlo en el navegador de tu máquina host.

### 4. Completar la Autorización

Después de pegar el URL en tu navegador, serás dirigido a la página de inicio de sesión de Salesforce. Inicia sesión con tus credenciales y sigue las instrucciones para autorizar la aplicación (Salesforce CLI en este caso).

Una vez completado el proceso de autorización, Salesforce CLI dentro de tu contenedor Docker estará autorizado para acceder a tu organización de Salesforce.

Este método es útil cuando trabajas con contenedores Docker o en entornos donde no puedes o no deseas gestionar automáticamente el lanzamiento del navegador.

# Create an app from scratch

[https://trailhead.salesforce.com/es/content/learn/projects/get-started-with-salesforce-development/create-a-data-model-using-clicks?trail_id=force_com_dev_beginner]()

## Create a class from CLI

The sfdx **force:apex:class:create** command generates a new Apex class named HouseService within a Salesforce DX project. It places the class file in the specified directory force-app/main/default/classes, establishing a foundation for adding custom logic to interact with Salesforce data and functionalities.

`sfdx force:apex:class:create -n HouseService -d force-app/main/default/classes`

### HouseService Class

The HouseService class in Salesforce Apex is designed with public visibility and the "with sharing" keyword, which enforces the sharing rules of the current user. This class includes a default constructor and a static method named getRecord.

The default constructor, HouseService(), is an empty constructor that doesn't perform any action. It's merely a placeholder for potential future use or initialization logic.

The static method getRecord is the core functionality of this class. It aims to retrieve a list of the first 10 House__c records from the Salesforce database, ordered by their creation date. This retrieval is done using a SOQL (Salesforce Object Query Language) query within a try-catch block to handle any exceptions gracefully.

The SOQL query selects several fields from the House__c custom object, including Id, Name, Address__c, State__c, City__c, and Zip__c. The WITH USER_MODE clause in the query specifies that the query should respect the organization’s sharing rules, ensuring that the current user can only access records they are permitted to see.

~~~
public with sharing class HouseService {
    public HouseService() {

    }

    public static List<House__c> getRecord()
    {
        try {

            List<House__c> lstHouses = [
            SELECT
                Id,
                Name,
                Address__c,
                State__c, 
                City__c,
                Zip__c
                FROM House__c
                WITH USER_MODE
                ORDER BY CreatedDate
                LIMIT 10
            ];

            return lstHouses;
            
        } catch (Exception e) {
            throw new AuraHandledException(e.getMessage());
        }
    }
}
~~~

### Deploy

The command `sf project deploy start -o myDevOrg` initiates the deployment of your Salesforce project's source code to the specified development organization, myDevOrg.

~~~
sf project deploy start -o myDevOrg
~~~

### Test and Run

~~~
System.debug(HouseService.getRecords());
~~~
Run in console:

~~~
sf apex run -f scripts/apex/dreamhouseapp.apex -o myDevOrg
~~~

## Lightning Web Component

This command uses the Salesforce CLI to create a new Lightning Web Component named housingMap in the specified directory within a Salesforce DX project structure.

~~~
sf lightning generate component --type lwc -n housingMap -d force-app/main/default/lwc
~~~

housingMap.html

~~~
<template>
    <lightning-card title="Housing Map">
        <!-- Explore all the base components via Base component library 
        (https://developer.salesforce.com/docs/component-library/overview/components)-->
          <lightning-map map-markers={mapMarkers}> </lightning-map>
      </lightning-card>
</template>
~~~

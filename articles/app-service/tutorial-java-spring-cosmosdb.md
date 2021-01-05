---
title: 'Tutoriel : Application Linux Java avec MongoDB'
description: Découvrez comment faire fonctionner une application Linux Java pilotée par les données dans Azure App Service en établissant une connexion à une instance MongoDB s’exécutant dans Azure (Cosmos DB).
author: rloutlaw
ms.author: routlaw
ms.devlang: java
ms.topic: tutorial
ms.date: 12/10/2018
ms.custom: mvc, seodec18, seo-java-july2019, seo-java-august2019, seo-java-september2019, devx-track-java
ms.openlocfilehash: 7e63f770763d1960148dfdfa184d0b4e2b76754c
ms.sourcegitcommit: 6906980890a8321dec78dd174e6a7eb5f5fcc029
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92427091"
---
# <a name="tutorial-build-a-java-spring-boot-web-app-with-azure-app-service-on-linux-and-azure-cosmos-db"></a>Tutoriel : Générer une application web Spring Boot Java avec Azure App Service sur Linux et Azure Cosmos DB

Ce didacticiel vous guide au travers du processus de création, de configuration, de déploiement et de mise à l’échelle des applications web Java sur Azure. Lorsque vous avez terminé, une application [Spring Boot](https://projects.spring.io/spring-boot/) stocke des données dans [Azure Cosmos DB](../cosmos-db/index.yml) s’exécutant sous [Azure App Service sur Linux](overview.md).

![Application Spring Boot stockant les données dans Azure Cosmos DB](./media/tutorial-java-spring-cosmosdb/spring-todo-app-running-locally.jpg)

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer une base de données Cosmos DB.
> * Connecter un exemple d’application à la base de données et la tester localement
> * Déployer un exemple d’application dans Azure
> * Diffuser des journaux de diagnostic à partir d’App Service
> * Ajouter des instances supplémentaires pour effectuer un scale-out de l’exemple d’application

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

* [Azure CLI](/cli/azure/overview), installé sur votre propre ordinateur. 
* [Git](https://git-scm.com/)
* [Java JDK](/azure/developer/java/fundamentals/java-jdk-long-term-support)
* [Maven](https://maven.apache.org)

## <a name="clone-the-sample-todo-app-and-prepare-the-repo"></a>Cloner l’exemple d’application de liste de tâches et préparer le référentiel

Ce didacticiel utilise un exemple d’application de liste de tâches avec une interface utilisateur web qui appelle une API REST Spring soutenue par [Spring Data Azure Cosmos DB](https://github.com/Microsoft/spring-data-cosmosdb). Le code pour l’application est disponible sur [GitHub](https://github.com/Microsoft/spring-todo-app). Pour en savoir plus sur l’écriture d’applications Java à l’aide de Spring et Cosmos DB, consultez le [Didacticiel Spring Boot Starter avec l’API SQL Azure Cosmos DB](/java/azure/spring-framework/configure-spring-boot-starter-java-app-with-cosmos-db) et [le démarrage rapide Spring Data Azure Cosmos DB](https://github.com/Microsoft/spring-data-cosmosdb#quick-start).


Exécutez les commandes suivantes dans votre terminal pour cloner l’exemple de dépôt et configurer l’exemple d’environnement d’application.

```bash
git clone --recurse-submodules https://github.com/Azure-Samples/e2e-java-experience-in-app-service-linux-part-2.git
cd e2e-java-experience-in-app-service-linux-part-2
yes | cp -rf .prep/* .
```

## <a name="create-an-azure-cosmos-db"></a>Créer une base de données Azure Cosmos DB

Suivez ces étapes pour créer une base de données Azure Cosmos DB dans votre abonnement. L’application de liste de tâches se connecte à cette base de données et stocke ses données lors de l’exécution, entraînant la persistance de l’application, quel que soit l’endroit d’où vous l’exécutez.

1. Connectez-vous à votre interface de ligne de commande Azure et définissez votre abonnement si vous en avez plusieurs connectés à vos informations d’identification.

    ```bash
    az login
    az account set -s <your-subscription-id>
    ```   

2. Créez un groupe de ressources Azure, en notant son nom.

    ```bash
    az group create -n <your-azure-group-name> \
        -l <your-resource-group-region>
    ```

3. Créez une base de données Azure Cosmos du type `GlobalDocumentDB`. Le nom de la base de données Cosmos DB doit être uniquement composé de lettres minuscules. Notez le champ `documentEndpoint` de la réponse à la commande.

    ```bash
    az cosmosdb create --kind GlobalDocumentDB \
        -g <your-azure-group-name> \
        -n <your-azure-COSMOS-DB-name-in-lower-case-letters>
    ```

4. Obtenez votre clé d’Azure Cosmos DB pour vous connecter à l’application. Conservez `primaryMasterKey` et `documentEndpoint` à proximité, car vous en aurez besoin à l’étape suivante.

    ```bash
    az cosmosdb list-keys -g <your-azure-group-name> -n <your-azure-COSMOSDB-name>
    ```

## <a name="configure-the-todo-app-properties"></a>Configurer les propriétés de l’application de liste de tâches

Ouvrez un terminal sur votre ordinateur. Copiez l’exemple de fichier de script dans le référentiel cloné afin que vous puissiez le personnaliser pour votre base de données Cosmos DB que vous venez de créer.

```bash
cd initial/spring-todo-app
cp set-env-variables-template.sh .scripts/set-env-variables.sh
```
 
Modifiez `.scripts/set-env-variables.sh` dans votre éditeur favori et fournissez les informations de connexion Azure Cosmos DB. Pour la configuration d’App Service Linux, utilisez la même région qu’avant (`your-resource-group-region`) ainsi que le groupe de ressources (`your-azure-group-name`) utilisé lors de la création de la base de données Cosmos DB. Choisissez un WEBAPP_NAME unique, dans la mesure où il ne peut pas être identique à un nom d’application web au sein d’un déploiement Azure.

```bash
export COSMOSDB_URI=<put-your-COSMOS-DB-documentEndpoint-URI-here>
export COSMOSDB_KEY=<put-your-COSMOS-DB-primaryMasterKey-here>
export COSMOSDB_DBNAME=<put-your-COSMOS-DB-name-here>

# App Service Linux Configuration
export RESOURCEGROUP_NAME=<put-your-resource-group-name-here>
export WEBAPP_NAME=<put-your-Webapp-name-here>
export REGION=<put-your-REGION-here>
```

Ensuite, exécutez le script :

```bash
source .scripts/set-env-variables.sh
```
   
Ces variables d’environnement sont utilisées dans `application.properties` au sein de l’application de liste de tâches. Les champs dans le fichier de propriétés définissent une configuration de référentiel par défaut pour Spring Data :

```properties
azure.cosmosdb.uri=${COSMOSDB_URI}
azure.cosmosdb.key=${COSMOSDB_KEY}
azure.cosmosdb.database=${COSMOSDB_DBNAME}
```

```java
@Repository
public interface TodoItemRepository extends DocumentDbRepository<TodoItem, String> {
}
```

L’exemple d’application utilise ensuite l’annotation `@Document` importée depuis `com.microsoft.azure.spring.data.cosmosdb.core.mapping.Document` pour configurer un type d’entité à stocker et gérer par Cosmos DB :

```java
@Document
public class TodoItem {
    private String id;
    private String description;
    private String owner;
    private boolean finished;
```

## <a name="run-the-sample-app"></a>Exécution de l'exemple d'application

Utilisez Maven pour exécuter l’exemple.

```bash
mvn package spring-boot:run
```

La sortie doit se présenter comme suit.

```bash
bash-3.2$ mvn package spring-boot:run
[INFO] Scanning for projects...
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building spring-todo-app 2.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 


[INFO] SimpleUrlHandlerMapping - Mapped URL path [/webjars/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
[INFO] SimpleUrlHandlerMapping - Mapped URL path [/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
[INFO] WelcomePageHandlerMapping - Adding welcome page: class path resource [static/index.html]
2018-10-28 15:04:32.101  INFO 7673 --- [           main] c.m.azure.documentdb.DocumentClient      : Initializing DocumentClient with serviceEndpoint [https://sample-cosmos-db-westus.documents.azure.com:443/], ConnectionPolicy [ConnectionPolicy [requestTimeout=60, mediaRequestTimeout=300, connectionMode=Gateway, mediaReadMode=Buffered, maxPoolSize=800, idleConnectionTimeout=60, userAgentSuffix=;spring-data/2.0.6;098063be661ab767976bd5a2ec350e978faba99348207e8627375e8033277cb2, retryOptions=com.microsoft.azure.documentdb.RetryOptions@6b9fb84d, enableEndpointDiscovery=true, preferredLocations=null]], ConsistencyLevel [null]
[INFO] AnnotationMBeanExporter - Registering beans for JMX exposure on startup
[INFO] TomcatWebServer - Tomcat started on port(s): 8080 (http) with context path ''
[INFO] TodoApplication - Started TodoApplication in 45.573 seconds (JVM running for 76.534)
```

Vous pouvez accéder à l’application Spring TODO localement en utilisant ce lien une fois l’application lancée : `http://localhost:8080/`.

 ![Accéder à l’application Spring TODO localement](./media/tutorial-java-spring-cosmosdb/spring-todo-app-running-locally.jpg)

Si vous voyez des exceptions à la place du message « Started TodoApplication », vérifiez que le script `bash` de l’étape précédente a correctement exporté les variables d’environnement et que les valeurs sont correctes pour la base de données Azure Cosmos DB créée.

## <a name="configure-azure-deployment"></a>Configurer un déploiement Azure

Ouvrez le fichier `pom.xml` dans le répertoire `initial/spring-boot-todo` et ajoutez la configuration suivante du [plug-in Azure Web App pour Maven](https://github.com/Microsoft/azure-maven-plugins/blob/develop/azure-webapp-maven-plugin/README.md).

```xml    
<plugins> 

    <!--*************************************************-->
    <!-- Deploy to Java SE in App Service Linux           -->
    <!--*************************************************-->
       
    <plugin>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>azure-webapp-maven-plugin</artifactId>
        <version>1.11.0</version>
        <configuration>
            <schemaVersion>v2</schemaVersion>

            <!-- Web App information -->
            <resourceGroup>${RESOURCEGROUP_NAME}</resourceGroup>
            <appName>${WEBAPP_NAME}</appName>
            <region>${REGION}</region>

            <!-- Java Runtime Stack for Web App on Linux-->
            <runtime>
                 <os>linux</os>
                 <javaVersion>jre8</javaVersion>
                 <webContainer>jre8</webContainer>
             </runtime>
             <deployment>
                 <resources>
                 <resource>
                     <directory>${project.basedir}/target</directory>
                     <includes>
                     <include>*.jar</include>
                     </includes>
                 </resource>
                 </resources>
             </deployment>

            <appSettings>
                <property>
                    <name>COSMOSDB_URI</name>
                    <value>${COSMOSDB_URI}</value>
                </property> 
                <property>
                    <name>COSMOSDB_KEY</name>
                    <value>${COSMOSDB_KEY}</value>
                </property>
                <property>
                    <name>COSMOSDB_DBNAME</name>
                    <value>${COSMOSDB_DBNAME}</value>
                </property>
                <property>
                    <name>JAVA_OPTS</name>
                    <value>-Dserver.port=80</value>
                </property>
            </appSettings>

        </configuration>
    </plugin>           
    ...
</plugins>
```

## <a name="deploy-to-app-service-on-linux"></a>Déployer vers App Service sous Linux

Utilisez l’objectif Maven `mvn azure-webapp:deploy` pour déployer l’application de liste de tâches dans Azure App Service sous Linux.

```bash

# Deploy
bash-3.2$ mvn azure-webapp:deploy
[INFO] Scanning for projects...
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building spring-todo-app 2.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- azure-webapp-maven-plugin:1.11.0:deploy (default-cli) @ spring-todo-app ---
[INFO] Auth Type : AZURE_CLI, Auth Files : [C:\Users\testuser\.azure\azureProfile.json, C:\Users\testuser\.azure\accessTokens.json]
[INFO] Subscription : xxxxxxxxx
[INFO] Target Web App doesn't exist. Creating a new one...
[INFO] Creating App Service Plan 'ServicePlanb6ba8178-5bbb-49e7'...
[INFO] Successfully created App Service Plan.
[INFO] Successfully created Web App.
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 1 resource to /home/test/e2e-java-experience-in-app-service-linux-part-2/initial/spring-todo-app/target/azure-webapp/spring-todo-app-61bb5207-6fb8-44c4-8230-c1c9e4c099f7
[INFO] Trying to deploy artifact to spring-todo-app...
[INFO] Renaming /home/test/e2e-java-experience-in-app-service-linux-part-2/initial/spring-todo-app/target/azure-webapp/spring-todo-app-61bb5207-6fb8-44c4-8230-c1c9e4c099f7/spring-todo-app-2.0-SNAPSHOT.jar to app.jar
[INFO] Deploying the zip package spring-todo-app-61bb5207-6fb8-44c4-8230-c1c9e4c099f7718326714198381983.zip...
[INFO] Successfully deployed the artifact to https://spring-todo-app.azurewebsites.net
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 02:19 min
[INFO] Finished at: 2019-11-06T15:32:03-07:00
[INFO] Final Memory: 50M/574M
[INFO] ------------------------------------------------------------------------
```

La sortie contient l’URL vers votre application déployée (dans cet exemple, `https://spring-todo-app.azurewebsites.net`). Vous pouvez copier cette URL dans votre navigateur web ou exécuter la commande suivante dans la fenêtre de votre terminal pour charger votre application.

```bash
curl https://spring-todo-app.azurewebsites.net
```

Vous devez voir l’application exécutée avec l’URL distante dans la barre d’adresses :

 ![Application Spring Boot s’exécutant avec une URL distante](./media/tutorial-java-spring-cosmosdb/spring-todo-app-running-in-app-service.jpg)

## <a name="stream-diagnostic-logs"></a>Diffuser les journaux de diagnostic

[!INCLUDE [Access diagnostic logs](../../includes/app-service-web-logs-access-no-h.md)]


## <a name="scale-out-the-todo-app"></a>Scale-out de l’application de liste de tâches

Effectuez un scale-out de l’application en ajoutant un autre worker :

```bash
az appservice plan update --number-of-workers 2 \
   --name ${WEBAPP_PLAN_NAME} \
   --resource-group <your-azure-group-name>
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous n’avez pas besoin de ces ressources pour un autre didacticiel (voir [Étapes suivantes](#next)), vous pouvez les supprimer en exécutant la commande suivante dans Cloud Shell : 
  
```bash
az group delete --name <your-azure-group-name>
```

<a name="next"></a>

## <a name="next-steps"></a>Étapes suivantes

[Azure pour les développeurs Java](/java/azure/)
[Spring Boot](https://spring.io/projects/spring-boot), [Spring Data pour Cosmos DB](/java/azure/spring-framework/configure-spring-boot-starter-java-app-with-cosmos-db?view=azure-java-stable), [Azure Cosmos DB](../cosmos-db/introduction.md) et [App Service Linux ](overview.md).

Découvrez-en plus sur l’exécution des applications Java sur App Service sur Linux dans le guide du développeur.

> [!div class="nextstepaction"] 
> [Guide du développeur Java dans App Service Linux](configure-language-java.md?pivots=platform-linux)

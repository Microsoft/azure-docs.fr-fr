---
title: Créer un conteneur pour Apache Tomcat sur Linux
description: Créez un conteneur Linux pour exposer une application s’exécutant sur un serveur Apache Tomcat sur Azure Service Fabric. Générez une image Docker avec votre application et un serveur Apache Tomcat, envoyez l’image à un registre de conteneurs, créez et déployez une application conteneur Service Fabric.
ms.topic: conceptual
ms.date: 6/08/2018
ms.author: pepogors
ms.openlocfilehash: 3de97bc277195dff2daf5868c0eb9aec5d6e27c0
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96534027"
---
# <a name="create-service-fabric-container-running-apache-tomcat-server-on-linux"></a>Créer un conteneur Service Fabric exécutant un serveur Apache Tomcat sur Linux
Apache Tomcat est une implémentation open source populaire des technologies Java Servlet et Java Server. Cet article montre comment créer un conteneur avec Apache Tomcat et une application web simple, déployer le conteneur sur un cluster Service Fabric exécutant Linux, et se connecter à l’application web.  

Pour en savoir plus sur Apache Tomcat, voir la [page d’accueil Apache Tomcat](https://tomcat.apache.org/). 

## <a name="prerequisites"></a>Prérequis
* Un ordinateur de développement exécutant :
  * [Outils et SDK Service Fabric](service-fabric-get-started-linux.md).
  * [Docker CE pour Linux](https://docs.docker.com/engine/installation/#prior-releases). 
  * [Interface de ligne de commande de Service Fabric](service-fabric-cli.md)

* Registre de conteneurs dans Azure Container Registry. Vous pouvez créer un registre de conteneurs dans votre abonnement Azure en utilisant le [portail Azure](../container-registry/container-registry-get-started-portal.md) ou [Azure CLI](./service-fabric-tutorial-create-container-images.md#deploy-azure-container-registry). 

## <a name="build-a-tomcat-image-and-run-it-locally"></a>Générer une image Tomcat et l’exécuter localement
Suivez les étapes décrites dans cette section pour créer une image Docker basée sur une image Apache Tomcat et une application web simple, puis exécutez-la dans un conteneur sur votre système local. 
 
1. Clonez le référentiel d’exemples de [Prise en main de Service Fabric avec Java](https://github.com/Azure-Samples/service-fabric-java-getting-started) sur votre ordinateur de développement.

   ```bash
   git clone https://github.com/Azure-Samples/service-fabric-java-getting-started.git
   ```

1. Passez à l’exemple de répertoire de serveur Apache Tomcat (*service-fabric-java-getting-started/container-apache-tomcat-web-server-sample*) :

   ```bash
   cd service-fabric-java-getting-started/container-apache-tomcat-web-server-sample
   ```

1. Créez un fichier Docker basé sur l’[image Tomcat](https://hub.docker.com/_/tomcat/) officielle située sur le Hub Docker l’exemple de serveur Tomcat. Dans le répertoire *service-fabric-java-getting-started/container-apache-tomcat-web-server-sample*, créez un fichier nommé *Dockerfile* (sans extension de nom de fichier). Ajoutez ce qui suit au fichier *Dockerfile* et enregistrez vos modifications :

   ```
   FROM library/tomcat

   EXPOSE 8080

   COPY ./ApacheTomcat /usr/local/tomcat
   ```

   Pour plus d’informations, voir [Documentation de référence de Dockerfile](https://docs.docker.com/engine/reference/builder/).


4. Connectez-vous à Docker, puis exécutez la commande `docker build` pour créer l’image exécutant votre application web :

   ```bash
   docker login
   docker build . -t tomcattest
   ```

   Cette commande crée l’image en suivant les instructions du Dockerfile, en nommant (balisant avec -t) l’image `tomcattest`. Pour générer une image de conteneur, l’image de base est d’abord téléchargée à partir du Hub Docker, puis l’application y est ajoutée. 

   Une fois la création terminée, exécutez la commande `docker images` pour afficher des informations sur la nouvelle image :

   ```bash
   $ docker images
    
   REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
   tomcattest                    latest              86838648aab6        2 minutes ago       194 MB
   ```

5. Vérifiez que votre application en conteneur s’exécute localement avant de l’envoyer au registre de conteneurs :
 
   ```bash
   docker run -itd --name tomcat-site -p 8080:8080 tomcattest.
   ```
   
   * `--name` nomme le conteneur afin que vous puissiez y faire référence en utilisant un nom convivial plutôt que son ID.
   * `-p` Spécifie le mappage de port entre le conteneur et le système d’exploitation hôte. 

   > [!Note]
   > Le port que vous ouvrez avec le paramètre `-p` doit être le port sur lequel votre application Tomcat écoute les requêtes. Dans l’exemple actuel, il y a un connecteur configuré dans le fichier *ApacheTomcat/conf/server.xml* fichier pour écouter les requêtes HTTP sur le port 8080. Ce port est mappé au port 8080 sur l’hôte. 

   Pour en savoir plus sur d’autres paramètres, voir la [documentation sur l’exécution du Docker](https://docs.docker.com/engine/reference/commandline/run/).

1. Pour tester votre conteneur, ouvrez un navigateur, puis entrez l’une des URL suivantes. Vous verrez une variante de l’écran d’accueil de « Hello World ! » sur chaque URL.

   - `http://localhost:8080/hello` 
   - `http://localhost:8080/hello/sayhello` 
   - `http://localhost:8080/hello/sayhi` 

   ![Hello world /sayhi](./media/service-fabric-get-started-tomcat/hello.png)

2. Arrêtez le conteneur et supprimez-le de votre ordinateur de développement :

   ```bash
   docker stop tomcat-site
   docker rm tomcat-site
   ```

## <a name="push-the-tomcat-image-to-your-container-registry"></a>Envoyer l’image Tomcat à votre registre de conteneurs
À présent que vous avez vérifié que l’image de Tomcat s’exécute dans un conteneur sur votre ordinateur de développement, envoyez-la à un référentiel dans un registre de conteneurs pour [réduire les interruptions](../container-registry/buffer-gate-public-content.md) pouvant affecter vos flux de travail de développement et de déploiement d’image. Cet article utilise Azure Container Registry pour stocker l’image mais, en apportant quelques modifications aux étapes, vous pouvez utiliser le registre de conteneur de votre choix. Dans cet article, le nom de registre est supposé être *myregistry*, et le nom de registre complet est myregistry.azurecr.io. Modifiez ces noms de façon appropriée pour votre scénario. 

1. Exécutez `docker login` pour vous connecter à votre registre de conteneurs à l’aide des [informations d’identification de votre registre](../container-registry/container-registry-authentication.md).

   L’exemple suivant transmet l’ID et le mot de passe d’un [principal du service](../active-directory/develop/app-objects-and-service-principals.md) Azure Active Directory . Par exemple, vous pouvez avoir affecté un principal du service à votre Registre pour un scénario d’automatisation. Vous pouvez aussi vous connecter à l’aide de votre nom d’utilisateur du registre et votre mot de passe.

   ```bash
   docker login myregistry.azurecr.io -u xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx -p myPassword
   ```

2. La commande suivante crée une balise, ou un alias, de l’image, avec un chemin d’accès qualifié complet vers votre registre. Cet exemple place l’image dans l’espace de noms `samples` pour éviter un encombrement à la racine du registre.

   ```bash
   docker tag tomcattest myregistry.azurecr.io/samples/tomcattest
   ```

3. Envoyez l’image vers votre registre de conteneurs :

   ```bash
   docker push myregistry.azurecr.io/samples/tomcattest
   ```

## <a name="build-and-deploy-the-service-fabric-container-application"></a>Générer et déployer l’application conteneur Service Fabric
À présent que vous avez envoyé l’image Tomcat à un registre de conteneurs, vous pouvez créer et déployer une application conteneur Service Fabric qui extrait l’image Tomcat de votre Registre et l’exécute en tant que service en conteneur dans votre cluster. 

1. Créez un répertoire en dehors de votre clone local (en dehors de l’arborescence de répertoires *service-fabric-java-getting-started*). Basculez vers ce répertoire et utilisez Yeoman pour créer une structure pour une application conteneur : 

   ```bash
   yo azuresfcontainer 
   ```
   Entrez les valeurs suivantes lorsque vous y êtes invité :

   * Donnez à votre application le nom suivant : ServiceFabricTomcat
   * Nom du service d’application : TomcatService
   * Indiquez le nom de l’image : fournissez l’URL de l’image conteneur dans votre registre de conteneurs. Par exemple, myregistry.azurecr.io/samples/tomcattest.
   * Commandes : Laissez ce champ vide. Cette image possède un point d’entrée de charge de travail défini, vous n’avez donc pas à spécifier des commandes d’entrée explicitement (les commandes s’exécutent dans le conteneur et garderont le conteneur en exécution après le démarrage).
   * Nombre d’instances de l’application conteneur invitée : 1

   ![Générateur Yeoman Service Fabric pour les conteneurs](./media/service-fabric-get-started-tomcat/yo-generator.png)

10. Dans le manifeste de service (*ServiceFabricTomcat/ServiceFabricTomcat/TomcatServicePkg/ServiceManifest.xml*), ajoutez le code XML suivant sous la balise racine **ServiceManfest** pour ouvrir le port sur lequel votre application écoute les requêtes. La balise **Endpoint** déclare le protocole et le port pour le point de terminaison. Dans cet article, le service en conteneur écoute sur le port 8080 : 

   ```xml
   <Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the port on which to 
       listen. Please note that if your service is partitioned, this port is shared with 
       replicas of different partitions that are placed in your code. -->
      <Endpoint Name="endpointTest" Port="8080" Protocol="tcp"/>
    </Endpoints>
   </Resources>
   ```

11. Dans le manifeste de l’application (*ServiceFabricTomcat/ServiceFabricTomcat/ApplicationManifest.xml*), sous la balise **ServiceManifestImport**, ajoutez le code XML suivant. Remplacez les paramètres **AccountName** et **Password** de la balise **RepositoryCredentials** par le nom de votre registre de conteneurs et le mot de passe requis pour vous connecter à celui-ci.

   ```xml
   <Policies>
    <ContainerHostPolicies CodePackageRef="Code">
      <PortBinding ContainerPort="8080" EndpointRef="endpointTest"/>
      <RepositoryCredentials AccountName="myregistry" Password="=P==/==/=8=/=+u4lyOB=+=nWzEeRfF=" PasswordEncrypted="false"/>
    </ContainerHostPolicies>
   </Policies>
   ```

   La balise **ContainerHostPolicies** spécifie des stratégies pour l’activation des hôtes de conteneur.
    
   * La balise **PortBinding** configure la stratégie de mappage de port à hôte du conteneur. L’attribut **ContainerPort** est défini sur 8080, car le conteneur expose le port 8080, comme spécifié dans le fichier Docker. L’attribut **EndpointRef** est défini sur « endpointTest », le point de terminaison défini dans le manifeste de service à l’étape précédente. Ainsi, les requêtes entrantes adressées au service sur le port 8080 sont mappées au port 8080 dans le conteneur. 
   * La balise **RepositoryCredentials** spécifie les informations d’identification dont le conteneur a besoin pour s’authentifier auprès du référentiel (privé) à partir duquel il extrait l’image. Vous n’avez pas besoin de cette stratégie si l’image doit être extraite à partir d’un référentiel public.
    

12. Dans le dossier *ServiceFabricTomcat*, connectez-vous à votre cluster Service Fabric. 

   * Pour vous connecter au cluster Service Fabric local, exécutez la commande suivante :

     ```bash
     sfctl cluster select --endpoint http://localhost:19080
     ```
    
   * Pour vous connecter à un cluster Azure sécurisé, vérifiez que le certificat client est présent sous la forme d’un fichier .pem dans le répertoire *ServiceFabricTomcat*, puis exécutez la commande suivante : 

     ```bash
     sfctl cluster select --endpoint https://PublicIPorFQDN:19080 -pem your-certificate.pem -no-verify
     ```
     Dans la commande précédente, remplacez `your-certificate.pem` par le nom de votre fichier de certificat client. Dans des environnements de développement et de test, le certificat de cluster est souvent utilisé en tant que certificat client. Si votre certificat n’est pas auto-signé, omettez le paramètre `-no-verify`. 
       
     Les certificats de cluster sont généralement téléchargés localement en tant que fichiers .pfx. Si vous ne disposez pas encore de votre certificat au format PEM, vous pouvez exécuter la commande suivante pour créer un fichier .pem à partir d’un fichier .pfx :

     ```bash
     openssl pkcs12 -in your-certificate.pfx -out your-certificate.pem -nodes -passin pass:your-pfx-password
     ```

     Si votre fichier PFX n’est pas protégé par mot de passe, utilisez `-passin pass:` en guise de dernier paramètre.


13. Exécutez le script d’installation fourni dans le modèle pour déployer l’application sur votre cluster. Le script copie le package d’application vers le magasin d’images du cluster, inscrit le type d’application, et crée une instance de l’application.

     ```bash
     ./install.sh
     ```

   Après avoir exécuté le script d’installation, ouvrez un navigateur et accédez à Service Fabric Explorer :
    
   * Sur un cluster local, utilisez `http://localhost:19080/Explorer` (remplacez *localhost* par l’adresse IP privée de la machine virtuelle si vous utilisez Vagrant sur Mac OS X).
   * Sur un cluster Azure sécurisé, utilisez `https://PublicIPorFQDN:19080/Explorer`. 
    
   Développez le nœud **Applications** et notez qu’il existe désormais une entrée (**ServiceFabricTomcatType**) pour votre type d’application, et une autre pour la première instance de ce type. Le déploiement complet de l’application pouvant prendre quelques minutes, soyez patient.

   ![Service Fabric Explorer](./media/service-fabric-get-started-tomcat/service-fabric-explorer.png)


1. Pour accéder à l’application sur le serveur Tomcat, ouvrez une fenêtre de navigateur, puis entrez l’une des URL suivantes. Si vous avez déployé sur le cluster local, utilisez *localhost* pour *PublicIPorFQDN*. Vous verrez une variante de l’écran d’accueil de « Hello World ! » sur chaque URL.

   * http://PublicIPorFQDN:8080/hello  
   * http://PublicIPorFQDN:8080/hello/sayhello
   * http://PublicIPorFQDN:8080/hello/sayhi

## <a name="clean-up"></a>Nettoyer
Utilisez le script de désinstallation fourni dans le modèle pour supprimer l’instance de l’application de votre cluster et annuler l’inscription du type d’application.

```bash
./uninstall.sh
```

Une fois l’image publiée dans le registre de conteneurs, vous pouvez supprimer l’image locale de votre ordinateur de développement :

```
docker rmi tomcattest
docker rmi myregistry.azurecr.io/samples/tomcattest
```

## <a name="next-steps"></a>Étapes suivantes
* Pour des étapes rapides en lien avec d’autres fonctionnalités de conteneur Linux, voir [Créer votre première application de conteneur Service Fabric sur Linux](service-fabric-get-started-containers-linux.md).
* Pour des étapes plus détaillées sur des conteneurs Linux, voir le didacticiel [Créer une application de conteneur Linux](service-fabric-tutorial-create-container-images.md).
* En savoir plus sur l’exécution des [conteneurs sur Service Fabric](service-fabric-containers-overview.md).



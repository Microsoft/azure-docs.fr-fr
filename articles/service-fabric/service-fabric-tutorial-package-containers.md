---
title: Créer des packages de conteneurs et les déployer
description: Dans ce tutoriel, vous découvrez comment générer une définition d’application Azure Service Fabric à l’aide de Yeoman et empaqueter l’application.
ms.topic: tutorial
ms.date: 07/22/2019
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 995291a783d14a6d2db8ed8319c720f55c009d91
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92738851"
---
# <a name="tutorial-package-and-deploy-containers-as-a-service-fabric-application-using-yeoman"></a>Tutoriel : empaqueter et déployer des conteneurs sous forme d’application Service Fabric à l’aide de Yeoman

Ce tutoriel est le deuxième de la série. Dans ce didacticiel, un outil de génération de modèles (Yeoman) est utilisé pour générer une définition d’application Service Fabric. Cette application peut ensuite servir à déployer des conteneurs sur Service Fabric. Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Installer Yeoman
> * Créer un package d’application à l’aide de Yeoman
> * Configurer les paramètres du package d’application pour une utilisation avec des conteneurs
> * Créer l’application
> * Déployer et exécuter l’application
> * Nettoyer l’application

## <a name="prerequisites"></a>Conditions préalables requises

* Les images conteneur, créées et transmises à Azure Container Registry dans la [Partie 1](service-fabric-tutorial-create-container-images.md) de cette série de didacticiels, sont utilisées.
* L’environnement de développement Linux est [configuré](service-fabric-tutorial-create-container-images.md).

## <a name="install-yeoman"></a>Installer Yeoman

Service Fabric fournit des outils de génération de modèles automatique qui vous aident à créer des applications à partir du terminal par l’intermédiaire du générateur de modèles Yeoman. Suivez les étapes ci-dessous pour vous assurer que vous disposez du générateur de modèles Yeoman.

1. Installez nodejs et NPM sur votre machine. Notez que les utilisateurs Mac OSX doivent utiliser le gestionnaire de package Homebrew

    ```bash
    curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash
    nvm install node 
    ```
2. Installez le générateur de modèles Yeoman sur votre machine à partir de NPM

    ```bash
    npm install -g yo
    ```
3. Installez le générateur de conteneurs Service Fabric Yeoman

    ```bash 
    npm install -g generator-azuresfcontainer
    ```

## <a name="package-a-docker-image-container-with-yeoman"></a>Placer un conteneur d’images Docker avec Yeoman dans un package

1. Pour créer une application conteneur Service Fabric, dans le répertoire 'container-tutorial' du dépôt cloné, exécutez la commande suivante.

    ```bash
    yo azuresfcontainer
    ```
2. Tapez « TestContainer » pour nommer votre application
3. Tapez « azurevotefront » pour nommer votre service d’application.
4. Indiquez le chemin de l’image conteneur dans ACR pour le dépôt front-end, par exemple « \<acrName>.azurecr.io/azure-vote-front:v1 ». Le champ \<acrName> doit avoir la même valeur que celle utilisée dans le tutoriel précédent.
5. Appuyez sur Entrée pour laisser la section Commands vide.
6. Indiquez 1 comme nombre d’instances.

L’exemple suivant montre l’entrée et la sortie de l’exécution de la commande yo :

```bash
? Name your application TestContainer
? Name of the application service: azurevotefront
? Input the Image Name: <acrName>.azurecr.io/azure-vote-front:v1
? Commands:
? Number of instances of guest container application: 1
   create TestContainer/TestContainer/ApplicationManifest.xml
   create TestContainer/TestContainer/azurevotefrontPkg/ServiceManifest.xml
   create TestContainer/TestContainer/azurevotefrontPkg/config/Settings.xml
   create TestContainer/TestContainer/azurevotefrontPkg/code/Dummy.txt
   create TestContainer/install.sh
   create TestContainer/uninstall.sh
```

Pour ajouter un autre service conteneur à une application déjà créée à l’aide de Yeoman, effectuez les étapes suivantes :

1. Changez le répertoire d’un niveau jusqu’au répertoire **TestContainer** , par exemple *./TestContainer*
2. Exécutez `yo azuresfcontainer:AddService`
3. Nommez le service « azurevoteback ».
4. Indiquez le chemin de l’image conteneur pour Redis : « alpine:redis »
5. Appuyez sur Entrée pour laisser la section Commands vide
6. Spécifiez un nombre d’instances de « 1 ».

Les entrées pour l’ajout du service utilisé sont toutes affichées :

```bash
? Name of the application service: azurevoteback
? Input the Image Name: alpine:redis
? Commands:
? Number of instances of guest container application: 1
   create TestContainer/azurevotebackPkg/ServiceManifest.xml
   create TestContainer/azurevotebackPkg/config/Settings.xml
   create TestContainer/azurevotebackPkg/code/Dummy.txt
```

Dans la suite de ce didacticiel, nous travaillons dans le répertoire **TestContainer** . Par exemple, *./TestContainer/TestContainer* . Le contenu de ce répertoire doit se présenter comme suit.

```bash
$ ls
ApplicationManifest.xml azurevotefrontPkg azurevotebackPkg
```

## <a name="configure-the-application-manifest-with-credentials-for-azure-container-registry"></a>Configurer le manifeste d’application avec les informations d’identification pour Azure Container Registry

Afin que Service Fabric puisse extraire les images conteneur à partir d’Azure Container Registry, nous devons fournir les informations d’identification dans le fichier **ApplicationManifest.xml** .

Connectez-vous à votre instance ACR. Utilisez la commande **az acr login** pour terminer l’opération. Fournissez le nom unique qui a été donné au registre de conteneurs au moment de sa création.

```azurecli
az acr login --name <acrName>
```

Après son exécution, la commande retourne le message **Login Succeeded (Connexion réussie)** .

Exécutez ensuite la commande suivante pour obtenir le mot de passe de votre registre de conteneurs. Ce mot de passe est utilisé par Service Fabric pour l’authentification auprès d’ACR, afin d’extraire les images conteneur.

```azurecli
az acr credential show -n <acrName> --query passwords[0].value
```

Dans le fichier **ApplicationManifest.xml** , ajoutez l’extrait de code sous l’élément **ServiceManifestImport** pour le service frontend. Insérez votre **acrName** pour le champ **AccountName** , le mot de passe retourné à partir de la commande précédente est utilisé pour le champ **Password** . Un fichier **ApplicationManifest.xml** complet est fourni à la fin de ce document.

```xml
<Policies>
  <ContainerHostPolicies CodePackageRef="Code">
    <RepositoryCredentials AccountName="<acrName>" Password="<password>" PasswordEncrypted="false"/>
  </ContainerHostPolicies>
</Policies>
```

## <a name="configure-communication-and-container-port-to-host-port-mapping"></a>Configurer la communication et le mappage des ports de conteneur aux ports hôtes

### <a name="configure-communication-port"></a>Configurer le port de communication

Configurez un point de terminaison HTTP pour que les clients puissent communiquer avec votre service. Ouvrez le fichier *./TestContainer/azurevotefrontPkg/ServiceManifest.xml* et déclarez une ressource de point de terminaison dans l’élément **ServiceManifest** .  Ajoutez le protocole, le port et le nom. Dans ce tutoriel, le service écoute le port 80. L’extrait de code suivant est placé sous la balise *ServiceManifest* dans la ressource.

```xml
<Resources>
  <Endpoints>
    <!-- This endpoint is used by the communication listener to obtain the port on which to 
            listen. Please note that if your service is partitioned, this port is shared with 
            replicas of different partitions that are placed in your code. -->
    <Endpoint Name="azurevotefrontTypeEndpoint" UriScheme="http" Port="80" Protocol="http"/>
  </Endpoints>
</Resources>

```

De même, modifiez le manifeste de service pour le service principal. Ouvrez *./TestContainer/azurevotefrontPkg/ServiceManifest.xml* et déclarez une ressource de point de terminaison dans l’élément **ServiceManifest** . Pour ce tutoriel, la valeur par défaut de redis, 6379, est conservée. L’extrait de code suivant est placé sous la balise *ServiceManifest* dans la ressource.

```xml
<Resources>
  <Endpoints>
    <!-- This endpoint is used by the communication listener to obtain the port on which to 
            listen. Please note that if your service is partitioned, this port is shared with 
            replicas of different partitions that are placed in your code. -->
    <Endpoint Name="azurevotebackTypeEndpoint" Port="6379" Protocol="tcp"/>
  </Endpoints>
</Resources>
```

Fournir **UriScheme** enregistre automatiquement le point de terminaison du conteneur auprès du service Service Fabric Naming pour la détectabilité. Un exemple de fichier ServiceManifest.xml complet pour le service principal est fourni à titre d’exemple à la fin de cet article.

### <a name="map-container-ports-to-a-service"></a>Mapper les ports de conteneur à un service

Afin d’exposer les conteneurs dans le cluster, nous devons également créer une liaison de port dans le fichier ApplicationManifest.xml. La stratégie **PortBinding** référence les **Endpoints** définis dans les fichiers **ServiceManifest.xml** . Les demandes entrantes vers ces points de terminaison sont mappées aux ports de conteneur qui sont ouverts et limités ici. Dans le fichier **ApplicationManifest.xml** , ajoutez le code suivant pour lier les ports 80 et 6379 aux points de terminaison. Un fichier **ApplicationManifest.xml** complet est disponible à la fin de ce document.

```xml
<ContainerHostPolicies CodePackageRef="Code">
    <PortBinding ContainerPort="80" EndpointRef="azurevotefrontTypeEndpoint"/>
</ContainerHostPolicies>
```

```xml
<ContainerHostPolicies CodePackageRef="Code">
    <PortBinding ContainerPort="6379" EndpointRef="azurevotebackTypeEndpoint"/>
</ContainerHostPolicies>
```

### <a name="add-a-dns-name-to-the-backend-service"></a>Ajouter un nom DNS au service principal

Pour que Service Fabric affecte ce nom DNS au service principal, le nom doit être spécifié dans le fichier **ApplicationManifest.xml** . Ajoutez l’attribut **ServiceDnsName** à l’élément **Service** , comme indiqué :

```xml
<Service Name="azurevoteback" ServiceDnsName="redisbackend.testapp">
  <StatelessService ServiceTypeName="azurevotebackType" InstanceCount="1">
    <SingletonPartition/>
  </StatelessService>
</Service>
```

Le service frontal lit une variable d’environnement pour connaître le nom DNS de l’instance Redis. Cette variable d’environnement est déjà définie dans le fichier Dockerfile qui a été utilisé pour générer l’image Docker. Aucune action n’est nécessaire ici.

```dockerfile
ENV REDIS redisbackend.testapp
```

L’extrait de code suivant illustre comment le code Python frontend récupère la variable d’environnement décrite dans le fichier Dockerfile. Aucune action n’est nécessaire ici.

```python
# Get DNS Name
redis_server = os.environ['REDIS']

# Connect to the Redis store
r = redis.StrictRedis(host=redis_server, port=6379, db=0)
```

À ce stade du didacticiel, le modèle d’une application Service Package est disponible pour le déploiement sur un cluster. Dans le tutoriel suivant, cette application est déployée et exécutée dans un cluster Service Fabric.

## <a name="create-a-service-fabric-cluster"></a>Créer un cluster Service Fabric

Pour déployer l’application sur Azure, vous avez besoin d’un cluster Service Fabric pour exécuter l’application. Les commandes suivantes créent un cluster à cinq nœuds dans Azure.  Elles créent également un certificat auto-signé, l’ajoutent à un coffre de clés et le téléchargent localement sous forme de fichier PEM. Le nouveau certificat sert à sécuriser le cluster au moment du déploiement et à authentifier les clients.

```azurecli
#!/bin/bash

# Variables
ResourceGroupName="containertestcluster" 
ClusterName="containertestcluster" 
Location="eastus" 
Password="q6D7nN%6ck@6" 
Subject="containertestcluster.eastus.cloudapp.azure.com" 
VaultName="containertestvault" 
VmPassword="Mypa$$word!321"
VmUserName="sfadminuser"

# Login to Azure and set the subscription
az login

az account set --subscription <mySubscriptionID>

# Create resource group
az group create --name $ResourceGroupName --location $Location 

# Create secure five node Linux cluster. Creates a key vault in a resource group
# and creates a certficate in the key vault. The certificate's subject name must match 
# the domain that you use to access the Service Fabric cluster.  
# The certificate is downloaded locally as a PEM file.
az sf cluster create --resource-group $ResourceGroupName --location $Location \ 
--certificate-output-folder . --certificate-password $Password --certificate-subject-name $Subject \ 
--cluster-name $ClusterName --cluster-size 5 --os UbuntuServer1604 --vault-name $VaultName \ 
--vault-resource-group $ResourceGroupName --vm-password $VmPassword --vm-user-name $VmUserName
```

> [!Note]
> Le service frontal web est configuré pour écouter le trafic entrant sur le port 80. Par défaut, le port 80 est ouvert sur les machines virtuelles du cluster et sur l’équilibreur de charge Azure.
>

Pour plus d’informations sur la création de votre propre cluster, consultez [Créer un cluster Service Fabric dans Azure](service-fabric-tutorial-create-vnet-and-linux-cluster.md).

## <a name="build-and-deploy-the-application-to-the-cluster"></a>Générer et déployer l’application sur le cluster

Vous pouvez déployer l’application sur le cluster Azure à l’aide de l’interface de ligne de commande (CLI) de Service Fabric. Si cette interface n’est pas installée sur votre machine, suivez les instructions [ici](service-fabric-get-started-linux.md#set-up-the-service-fabric-cli) pour l’installer.

Connectez-vous au cluster Service Fabric dans Azure. Remplacez le point de terminaison exemple par votre propre point de terminaison. Il doit s’agir d’une URL complète semblable à celle présentée ci-dessous.  Le fichier PEM est le certificat auto-signé créé précédemment.

```bash
sfctl cluster select --endpoint https://containertestcluster.eastus.cloudapp.azure.com:19080 --pem containertestcluster22019013100.pem --no-verify
```

Utilisez le script d’installation fourni dans le répertoire **TestContainer** pour copier le package d’application dans le magasin d’images du cluster, inscrire le type d’application et créer une instance de l’application.

```bash
./install.sh
```

Ouvrez un navigateur et accédez à Service Fabric Explorer à l’adresse http:\//containertestcluster.eastus.cloudapp.azure.com:19080/Explorer. Développez le nœud Applications et constatez qu’il existe une entrée pour votre type d’application, et une entrée pour l’instance.

![Service Fabric Explorer][sfx]

Pour pouvoir vous connecter à l’application en cours d’exécution, ouvrez un navigateur web et accédez à l’URL du cluster, par exemple http:\//containertestcluster.eastus.cloudapp.azure.com:80. L’application de vote doit s’afficher dans l’interface utilisateur web.

![La capture d’écran montre l’application de vote Azure avec des boutons pour les chats, les chiens, la réinitialisation et les totaux.][votingapp]

## <a name="clean-up"></a>Nettoyer

Utilisez le script de désinstallation fourni dans le modèle pour supprimer l’instance de l’application du cluster et annuler l’inscription du type d’application. Cette commande prend un certain temps pour nettoyer l’instance, et la commande 'install.sh' ne peut pas être exécutée immédiatement après ce script.

```bash
./uninstall.sh
```

## <a name="examples-of-completed-manifests"></a>Exemples de manifestes complétés

### <a name="applicationmanifestxml"></a>ApplicationManifest.xml

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<ApplicationManifest ApplicationTypeName="TestContainerType" ApplicationTypeVersion="1.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="azurevotefrontPkg" ServiceManifestVersion="1.0.0"/>
    <Policies>
    <ContainerHostPolicies CodePackageRef="Code">
        <RepositoryCredentials AccountName="myaccountname" Password="<password>" PasswordEncrypted="false"/>
        <PortBinding ContainerPort="80" EndpointRef="azurevotefrontTypeEndpoint"/>
    </ContainerHostPolicies>
        </Policies>
  </ServiceManifestImport>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="azurevotebackPkg" ServiceManifestVersion="1.0.0"/>
      <Policies>
        <ContainerHostPolicies CodePackageRef="Code">
          <PortBinding ContainerPort="6379" EndpointRef="azurevotebackTypeEndpoint"/>
        </ContainerHostPolicies>
      </Policies>
  </ServiceManifestImport>
  <DefaultServices>
    <Service Name="azurevotefront">
      <StatelessService ServiceTypeName="azurevotefrontType" InstanceCount="1">
        <SingletonPartition/>
      </StatelessService>
    </Service>
    <Service Name="azurevoteback" ServiceDnsName="redisbackend.testapp">
      <StatelessService ServiceTypeName="azurevotebackType" InstanceCount="1">
        <SingletonPartition/>
      </StatelessService>
    </Service>
  </DefaultServices>
</ApplicationManifest>
```

### <a name="front-end-servicemanifestxml"></a>ServiceManifest.xml pour service frontal

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="azurevotefrontPkg" Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" >

   <ServiceTypes>
      <StatelessServiceType ServiceTypeName="azurevotefrontType" UseImplicitHost="true">
   </StatelessServiceType>
   </ServiceTypes>

   <CodePackage Name="code" Version="1.0.0">
      <EntryPoint>
         <ContainerHost>
            <ImageName>acrName.azurecr.io/azure-vote-front:v1</ImageName>
            <Commands></Commands>
         </ContainerHost>
      </EntryPoint>
      <EnvironmentVariables>
      </EnvironmentVariables>
   </CodePackage>

  <Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the port on which to 
           listen. Please note that if your service is partitioned, this port is shared with 
           replicas of different partitions that are placed in your code. -->
      <Endpoint Name="azurevotefrontTypeEndpoint" UriScheme="http" Port="80" Protocol="http"/>
    </Endpoints>
  </Resources>

 </ServiceManifest>
```

### <a name="redis-servicemanifestxml"></a>ServiceManifest.xml pour instance Redis

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="azurevotebackPkg" Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" >

   <ServiceTypes>
      <StatelessServiceType ServiceTypeName="azurevotebackType" UseImplicitHost="true">
   </StatelessServiceType>
   </ServiceTypes>

   <CodePackage Name="code" Version="1.0.0">
      <EntryPoint>
         <ContainerHost>
            <ImageName>alpine:redis</ImageName>
            <Commands></Commands>
         </ContainerHost>
      </EntryPoint>
      <EnvironmentVariables>
      </EnvironmentVariables>
   </CodePackage>
     <Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the port on which to 
           listen. Please note that if your service is partitioned, this port is shared with 
           replicas of different partitions that are placed in your code. -->
      <Endpoint Name="azurevotebackTypeEndpoint" Port="6379" Protocol="tcp"/>
    </Endpoints>
  </Resources>
 </ServiceManifest>
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, plusieurs conteneurs ont été mis en package dans une application Service Fabric à l’aide de Yeoman. Cette application a ensuite été déployée et exécutée sur un cluster Service Fabric. Les étapes suivantes ont été effectuées :

> [!div class="checklist"]
> * Installer Yeoman
> * Créer un package d’application à l’aide de Yeoman
> * Configurer les paramètres du package d’application pour une utilisation avec des conteneurs
> * Créer l’application
> * Déployer et exécuter l’application
> * Nettoyer l’application

Passez au didacticiel suivant pour en savoir plus sur le basculement et la mise à l’échelle de l’application dans Service Fabric.

> [!div class="nextstepaction"]
> [En savoir plus sur le basculement et la mise à l’échelle des applications](service-fabric-tutorial-containers-failover.md)

[votingapp]: ./media/service-fabric-tutorial-deploy-run-containers/votingapp.png
[sfx]: ./media/service-fabric-tutorial-deploy-run-containers/containerspackagetutorialsfx.png

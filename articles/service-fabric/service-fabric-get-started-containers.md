---
title: Créer une application de conteneur Azure Service Fabric
description: Créez votre première application de conteneur Windows sur Microsoft Azure Service Fabric. Concevez une image Docker avec une application Python, envoyez l’image vers un registre de conteneurs, puis créez et déployez le conteneur dans Azure Service Fabric.
ms.topic: conceptual
ms.date: 01/25/2019
ms.custom: devx-track-python
ms.openlocfilehash: 197423670ffe05f15fdc5bfd351efdfba33b53cd
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96533772"
---
# <a name="create-your-first-service-fabric-container-application-on-windows"></a>Créer votre première application de conteneur Service Fabric sur Windows

> [!div class="op_single_selector"]
> * [Windows](service-fabric-get-started-containers.md)
> * [Linux](service-fabric-get-started-containers-linux.md)

L’exécution d’une application existante dans un conteneur Windows sur un cluster Service Fabric ne nécessite aucune modification de votre application. Cet article vous accompagne dans la création d’une image Docker contenant une application web [Flask](http://flask.pocoo.org/) Python et le déploiement dans un cluster Azure Service Fabric. Vous allez également partager votre application en conteneur via [Azure Container Registry](../container-registry/index.yml). Cet article suppose une connaissance élémentaire de Docker. Pour en savoir plus sur Docker, consultez la [présentation de Docker](https://docs.docker.com/engine/understanding-docker/).

> [!NOTE]
> Cet article s’applique à un environnement de développement Windows.  Le runtime du cluster Service Fabric et le runtime de Docker doivent être en cours d’exécution sur le même système d’exploitation.  Vous ne pouvez pas exécuter des conteneurs Windows sur un cluster Linux.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Conditions préalables requises

* Un ordinateur de développement exécutant :
  * Visual Studio 2015 ou Visual Studio 2019.
  * [Outils et SDK Service Fabric](service-fabric-get-started.md).
  *  Docker pour Windows. [Obtenez Docker CE pour Windows (stable)](https://store.docker.com/editions/community/docker-ce-desktop-windows?tab=description). Après l’installation et le démarrage de Docker, cliquez avec le bouton droit de la souris sur l’icône de la barre d’état, puis sélectionnez **Switch to Windows containers** (Basculer vers les conteneurs Windows). Cette étape est nécessaire pour exécuter des images Docker basées sur Windows.

* Un cluster Windows avec trois nœuds ou plus s’exécutant sur Windows Server avec conteneurs. 

  Pour cet article, la version (build) de Windows Server avec conteneurs en cours d’exécution sur vos nœuds de cluster doit correspondre à celle installée sur votre ordinateur de développement. En effet, vous générez l’image Docker sur votre ordinateur de développement et il existe des contraintes de compatibilité entre les versions du système d’exploitation de conteneur et le système d’exploitation hôte sur lequel elle est déployée. Pour plus d’informations, consultez [Système d’exploitation de conteneur Windows Server et compatibilité avec le système d’exploitation hôte](#windows-server-container-os-and-host-os-compatibility). 
  
    Pour déterminer la version de Windows Server avec conteneurs dont vous avez besoin pour votre cluster, exécutez la commande `ver` à partir d’une invite de commandes Windows sur votre ordinateur de développement. Consultez [Système d’exploitation de conteneur Windows Server et compatibilité avec le système d’exploitation hôte](#windows-server-container-os-and-host-os-compatibility) avant de [créer un cluster](service-fabric-cluster-creation-via-portal.md).

* Un registre dans Azure Container Registry. [Créez un registre de conteneurs](../container-registry/container-registry-get-started-portal.md) dans votre abonnement Azure.

> [!NOTE]
> Le déploiement de conteneurs sur un cluster Service Fabric sous Windows 10 est pris en charge.  Consultez [cet article](service-fabric-how-to-debug-windows-containers.md) pour plus d’informations sur la configuration de Windows 10 pour exécuter des conteneurs Windows.

> [!NOTE]
> La version 6.2 de Service Fabric et les versions ultérieures prennent en charge le déploiement de conteneurs pour les clusters exécutant Windows Server version 1709.

## <a name="define-the-docker-container"></a>Définir le conteneur Docker

Créez une image basée sur [l’image Python](https://hub.docker.com/_/python/) située sur Docker Hub.

Spécifiez votre conteneur Docker dans un fichier Dockerfile. Le fichier Dockerfile contient des instructions pour la configuration de l’environnement à l’intérieur de votre conteneur, le chargement de l’application que vous souhaitez exécuter et le mappage des ports. Le fichier Dockerfile est l’entrée de la commande `docker build`, qui crée l’image.

Créez un répertoire vide et créez le fichier *Dockerfile* (sans extension de fichier). Ajoutez ce qui suit au fichier *Dockerfile* et enregistrez vos modifications :

```
# Use an official Python runtime as a base image
FROM python:2.7-windowsservercore

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
ADD . /app

# Install any needed packages specified in requirements.txt
RUN pip install -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```

Pour plus d'informations, consultez les [références Dockerfile](https://docs.docker.com/engine/reference/builder/).

## <a name="create-a-basic-web-application"></a>Créer une application web de base
Créez une application web Flask écoutant le port 80 qui renvoie `Hello World!`. Dans le même répertoire, créez le fichier *requirements.txt*. Ajoutez ce qui suit et enregistrez vos modifications :
```
Flask
```

Créez également le fichier *app.py* et ajoutez l’extrait de code suivant :

```python
from flask import Flask

app = Flask(__name__)


@app.route("/")
def hello():

    return 'Hello World!'


if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```

<a id="Build-Containers"></a>

## <a name="login-to-docker-and-build-the-image"></a>Connexion à Docker et génération de l’image

Nous allons maintenant créer l’image qui exécute votre application web. Lorsque vous extrayez des images publiques de Docker (comme `python:2.7-windowsservercore` dans notre fichier Dockerfile), il est recommandé de s’authentifier auprès de votre compte Docker Hub au lieu de créer une demande de tirage anonyme.

> [!NOTE]
> Lorsque vous effectuez des demande de tirage (pull request) anonymes fréquentes, vous pouvez voir des erreurs Docker similaires à `ERROR: toomanyrequests: Too Many Requests.` ou `You have reached your pull rate limit.` Authentifiez-vous auprès de Docker Hub pour éviter ces erreurs. Consultez [Gérer le contenu public à l’aide d’Azure Container Registry](../container-registry/buffer-gate-public-content.md) pour plus d’informations.

Ouvrez une fenêtre PowerShell et accédez au répertoire contenant le fichier Dockerfile. Exécutez ensuite les commande suivantes :

```
docker login
docker build -t helloworldapp .
```

Cette commande crée l’image en suivant les instructions de votre fichier Dockerfile, et la nomme (balisage -t) `helloworldapp`. Pour générer une image conteneur, l’image de base est tout d’abord téléchargée depuis le Hub Docker auquel l’application est ajoutée. 

Une fois la création terminée, exécutez la commande `docker images` pour afficher des informations sur la nouvelle image :

```
$ docker images

REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
helloworldapp                 latest              8ce25f5d6a79        2 minutes ago       10.4 GB
```

## <a name="run-the-application-locally"></a>Exécuter l’application localement
Vérifiez votre image en local avant de l’envoyer dans le registre de conteneurs. 

Exécutez l'application :

```
docker run -d --name my-web-site helloworldapp
```

*name* donne un nom au conteneur en cours d’exécution (au lieu d’utiliser l’ID de conteneur).

Une fois le conteneur démarré, recherchez son adresse IP pour vous connecter à votre conteneur en cours d’exécution à partir d’un navigateur :
```
docker inspect -f "{{ .NetworkSettings.Networks.nat.IPAddress }}" my-web-site
```

Si cette commande ne retourne aucun résultat, exécutez la commande suivante et examinez l’élément **NetworkSettings**->**Réseaux** pour l’adresse IP :
```
docker inspect my-web-site
```

Connectez le conteneur en cours d’exécution. Ouvrez un navigateur web qui pointe vers l’adresse IP renvoyée, par exemple « http:\//172.31.194.61 ». Vous devez voir le titre « Hello World ! » s’afficher dans le navigateur.

Pour arrêter votre conteneur, exécutez :

```
docker stop my-web-site
```

Supprimez le conteneur de votre ordinateur de développement :

```
docker rm my-web-site
```

<a id="Push-Containers"></a>
## <a name="push-the-image-to-the-container-registry"></a>Envoyer l’image dans le registre de conteneurs

Après avoir vérifié que le conteneur s’exécute sur votre ordinateur de développement, envoyez l’image à votre registre dans Azure Container Registry.

Exécutez ``docker login`` pour vous connecter à votre registre de conteneurs à l’aide des [informations d’identification de votre registre](../container-registry/container-registry-authentication.md).

L’exemple suivant transmet l’ID et le mot de passe d’un [principal du service](../active-directory/develop/app-objects-and-service-principals.md) Azure Active Directory . Par exemple, vous pouvez avoir affecté un principal du service à votre Registre pour un scénario d’automatisation. Vous pouvez aussi vous connecter à l’aide de votre nom d’utilisateur du registre et votre mot de passe.

```
docker login myregistry.azurecr.io -u xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx -p myPassword
```

La commande suivante crée une balise, ou un alias, de l’image, avec un chemin d’accès qualifié complet vers votre registre. Cet exemple place l’image dans l’espace de noms ```samples``` pour éviter un encombrement à la racine du registre.

```
docker tag helloworldapp myregistry.azurecr.io/samples/helloworldapp
```

Envoyez l’image vers votre registre de conteneurs :

```
docker push myregistry.azurecr.io/samples/helloworldapp
```

## <a name="create-the-containerized-service-in-visual-studio"></a>Créer le service en conteneur dans Visual Studio
Le kit de développement logiciel Service Fabric fournit un modèle de service pour vous aider à créer une application en conteneur.

1. Démarrez Visual Studio. Sélectionnez **Fichier** > **Nouveau** > **Projet**.
2. Sélectionnez **Service Fabric application** (Application Service Fabric), nommez-la « MyFirstContainer », puis cliquez sur **OK**.
3. Sélectionnez **Container** (Conteneur) dans la liste des **modèles de service**.
4. Sous **Nom de l’image**, entrez « myregistry.azurecr.io/samples/helloworldapp », c’est-à-dire l’image que vous avez envoyée à votre référentiel de conteneurs.
5. Donnez un nom à votre service et cliquez sur **OK**.

## <a name="configure-communication"></a>Configurer la communication
Le service en conteneur a besoin d’un point de terminaison pour la communication. Ajoutez un élément `Endpoint` ainsi que le protocole, le port et le type au fichier ServiceManifest.xml. Dans cet exemple, un port fixe 8081 est utilisé. Si aucun port n’est spécifié, un port aléatoire de la plage de ports de l’application est choisi. 

```xml
<Resources>
  <Endpoints>
    <Endpoint Name="Guest1TypeEndpoint" UriScheme="http" Port="8081" Protocol="http"/>
  </Endpoints>
</Resources>
```
> [!NOTE]
> Vous pouvez ajouter d’autres points de terminaison à un service en déclarant des éléments EndPoint supplémentaires avec les valeurs de propriétés applicables. Chaque port peut déclarer une valeur de protocole uniquement.

En définissant un point de terminaison, Service Fabric publie le point de terminaison sur le service d’affectation de noms. D’autres services qui s’exécutent dans le cluster peuvent résoudre ce conteneur. Vous pouvez également effectuer la communication de conteneur à conteneur à l’aide du [proxy inversé](service-fabric-reverseproxy.md). Pour établir la communication, vous devez fournir le port d’écoute HTTP associé au proxy inversé et le nom des services avec lesquels vous souhaitez communiquer en tant que variables d’environnement.

Le service est à l’écoute sur un port spécifique (8081 dans cet exemple). Lorsque l’application est déployée sur un cluster dans Azure, le cluster et l’application s’exécutent derrière un équilibreur de charge Azure. Le port de l’application doit être ouvert dans l’équilibreur de charge Azure afin que le trafic entrant puisse accéder au service.  Vous pouvez ouvrir ce port dans l’équilibreur de charge Azure en vous aidant d’un [script PowerShell](./scripts/service-fabric-powershell-open-port-in-load-balancer.md) ou en intervenant dans le [portail Azure](https://portal.azure.com).

## <a name="configure-and-set-environment-variables"></a>Configurer et définir des variables d’environnement
Il est possible de spécifier des variables d’environnement pour chaque package de code dans le manifeste de service. Cette fonctionnalité est disponible pour tous les services, qu’ils soient déployés sous forme de conteneurs, de processus ou d’exécutables invités. Vous pouvez remplacer les valeurs de variable d’environnement dans le manifeste de l’application, ou les spécifier lors du déploiement, en tant que paramètres d’application.

L’extrait de code XML du manifeste de service suivant illustre la méthode à suivre pour spécifier des variables d’environnement pour un package de code :
```xml
<CodePackage Name="Code" Version="1.0.0">
  ...
  <EnvironmentVariables>
    <EnvironmentVariable Name="HttpGatewayPort" Value=""/>    
  </EnvironmentVariables>
</CodePackage>
```

Ces variables d’environnement peuvent être remplacées dans le manifeste de l’application :

```xml
<ServiceManifestImport>
  <ServiceManifestRef ServiceManifestName="Guest1Pkg" ServiceManifestVersion="1.0.0" />
  <EnvironmentOverrides CodePackageRef="FrontendService.Code">
    <EnvironmentVariable Name="HttpGatewayPort" Value="19080"/>
  </EnvironmentOverrides>
  ...
</ServiceManifestImport>
```

## <a name="configure-container-port-to-host-port-mapping-and-container-to-container-discovery"></a>Configuration du mappage des ports de conteneur aux ports hôtes et de la détection de conteneur à conteneur
Configurez un port de l’hôte utilisé pour communiquer avec le conteneur. La liaison de port mappe le port sur lequel le service écoute, à l’intérieur du conteneur, à un port sur l’hôte. Ajoutez un élément `PortBinding` dans l’élément `ContainerHostPolicies` du fichier ApplicationManifest.xml. Dans cet article, `ContainerPort` est 80 (le conteneur expose le port 80, tel que spécifié dans le fichier Dockerfile) et `EndpointRef` est « Guest1TypeEndpoint » (le point de terminaison défini précédemment dans le manifeste de service). Les demandes entrantes pour le service sur le port 8081 sont mappées au port 80 dans le conteneur.

```xml
<ServiceManifestImport>
    ...
    <Policies>
        <ContainerHostPolicies CodePackageRef="Code">
            <PortBinding ContainerPort="80" EndpointRef="Guest1TypeEndpoint"/>
        </ContainerHostPolicies>
    </Policies>
    ...
</ServiceManifestImport>
```
> [!NOTE]
> Vous pouvez ajouter d’autres liaisons de port à un service en déclarant des éléments PortBinding avec les valeurs de propriétés applicables.

## <a name="configure-container-repository-authentication"></a>Configurer l’authentification des référentiels de conteneur

Pour configurer différents types d'authentification afin de télécharger des images conteneur, consultez [Authentification des référentiels de conteneur](configure-container-repository-credentials.md).

## <a name="configure-isolation-mode"></a>Configurer le mode d’isolation
Windows prend en charge deux modes d’isolation pour les conteneurs : Processus et Hyper-V. Avec le mode d’isolation Processus, tous les conteneurs s’exécutant sur le même hôte partagent le noyau avec l’hôte. Avec le mode d’isolation Hyper-V, les noyaux sont isolés entre chaque conteneur Hyper-V et l’hôte du conteneur. Le mode d’isolation est défini dans l’élément `ContainerHostPolicies` dans le fichier manifeste de l’application. Les modes d’isolation qui peuvent être définis sont `process`, `hyperv` et `default`. Le mode par défaut est le mode d’isolation des processus sur les hôtes Windows Server. Sur les hôtes Windows 10, seul le mode d’isolation Hyper-V est pris en charge. Le conteneur s’exécute donc en mode d’isolation Hyper-V, quel que soit son paramètre de mode d’isolation. L’extrait de code suivant montre comment le mode d’isolation est spécifié dans le fichier manifeste de l’application.

```xml
<ContainerHostPolicies CodePackageRef="Code" Isolation="hyperv">
```
   > [!NOTE]
   > Le mode d’isolation Hyper-V est disponible pour les références SKU Azure Ev3 et Dv3 qui prennent en charge la virtualisation imbriquée. 
   >
   >

## <a name="configure-resource-governance"></a>Configurer la gouvernance des ressources
La [gouvernance des ressources](service-fabric-resource-governance.md) limite les ressources que le conteneur peut utiliser sur l’hôte. L’élément `ResourceGovernancePolicy`, spécifié dans le manifeste de l’application, est utilisé pour déclarer des limites relatives aux ressources pour un package de code de service. Des limites de ressources peuvent être définies pour les ressources suivantes : Memory, MemorySwap, CpuShares (poids relatif du processeur), MemoryReservationInMB, BlkioWeight (poids relatif de l’élément BlockIO). Dans cet exemple, le package de service Guest1Pkg obtient un cœur sur les nœuds de cluster où il est placé. Les limites de mémoire sont absolues, ce qui signifie que le package de code est limité à 1024 Mo de mémoire (avec une garantie de réservation identique). Les packages de code (conteneurs ou processus) ne sont pas en mesure d’allouer plus de mémoire que cette limite. Toute tentative en ce sens conduit à une exception de mémoire insuffisante. Pour pouvoir appliquer la limite de ressources, des limites de mémoire doivent être spécifiées pour tous les packages de code au sein d’un package de service.

```xml
<ServiceManifestImport>
  <ServiceManifestRef ServiceManifestName="Guest1Pkg" ServiceManifestVersion="1.0.0" />
  <Policies>
    <ServicePackageResourceGovernancePolicy CpuCores="1"/>
    <ResourceGovernancePolicy CodePackageRef="Code" MemoryInMB="1024"  />
  </Policies>
</ServiceManifestImport>
```
## <a name="configure-docker-healthcheck"></a>Configurer le docker HEALTHCHECK 

En démarrant la version 6.1, Service Fabric intègre automatiquement les événements [docker HEALTHCHECK](https://docs.docker.com/engine/reference/builder/#healthcheck) à son rapport d’intégrité du système. Cela signifie que si **HEALTHCHECK** est activé dans votre conteneur, Service Fabric générera un rapport d’intégrité chaque fois que l’état d’intégrité du conteneur changera comme indiqué par Docker. Un rapport d’intégrité **OK** apparaît dans [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) lorsque *health_status* est *intègre* et  **AVERTISSEMENT** s’affiche lorsque *health_status* est *défectueux*. 

À compter de la dernière version v6.4, vous avez la possibilité d’indiquer que les évaluations HEALTHCHECK docker doivent être signalées en tant qu’erreur. Si cette option est activée, un rapport d’intégrité **OK** apparaît quand *health_status* indique *sain* et **ERROR** s’affiche quand *health_status* indique *non sain*.

L’instruction **HEALTHCHECK** qui pointe vers la vérification réalisée pour surveiller l’intégrité du conteneur doit être présente dans le fichier Dockerfile utilisé lors de la génération de l’image conteneur.

![La capture d’écran montre les détails du package de services déployé NodeServicePackage.][3]

![HealthCheckUnhealthyApp][4]

![HealthCheckUnhealthyDsp][5]

Vous pouvez configurer un comportement **HEALTHCHECK** pour chaque conteneur en spécifiant les options **HealthConfig** dans **ContainerHostPolicies** dans ApplicationManifest.

```xml
<ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="ContainerServicePkg" ServiceManifestVersion="2.0.0" />
    <Policies>
      <ContainerHostPolicies CodePackageRef="Code">
        <HealthConfig IncludeDockerHealthStatusInSystemHealthReport="true"
              RestartContainerOnUnhealthyDockerHealthStatus="false" 
              TreatContainerUnhealthyStatusAsError="false" />
      </ContainerHostPolicies>
    </Policies>
</ServiceManifestImport>
```
Par défaut, *IncludeDockerHealthStatusInSystemHealthReport* est défini sur **true**, *RestartContainerOnUnhealthyDockerHealthStatus* est défini sur **false** et *TreatContainerUnhealthyStatusAsError* est défini sur **false**. 

Si *RestartContainerOnUnhealthyDockerHealthStatus* est défini sur **true**, un conteneur déclaré défectueux à plusieurs reprises est redémarré (éventuellement sur d’autres nœuds).

Si *TreatContainerUnhealthyStatusAsError* est défini sur **true**, des rapports d’intégrité **ERROR** s’affichent quand *health_status* indique *non sain* pour le conteneur.

Si vous souhaitez désactiver l’intégration **HEALTHCHECK** pour l’ensemble du cluster Service Fabric, vous devez définir [EnableDockerHealthCheckIntegration](service-fabric-cluster-fabric-settings.md) sur **false**.

## <a name="deploy-the-container-application"></a>Déployer l’application de conteneur
Enregistrez toutes les modifications et générez l’application. Pour publier votre application, cliquez avec le bouton droit de la souris sur **MyFirstContainer** dans l’Explorateur de solutions, puis sélectionnez **Publier**.

Dans **Point de terminaison de connexion**, entrez le point de terminaison de gestion pour le cluster. Par exemple : `containercluster.westus2.cloudapp.azure.com:19000`. Vous trouverez le point de terminaison de connexion client dans l’onglet Vue d’ensemble de votre cluster dans le [portail Azure](https://portal.azure.com).

Cliquez sur **Publier**.

[Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) est un outil web dédié à l’inspection et à la gestion d’applications et de nœuds dans un cluster Service Fabric. Ouvrez un navigateur et accédez à `http://containercluster.westus2.cloudapp.azure.com:19080/Explorer/`, puis suivez le déploiement d’application. L’application se déploie mais se trouve dans un état d’erreur jusqu’à ce que l’image soit téléchargée sur les nœuds du cluster (ce qui peut prendre du temps, selon la taille de l’image) : ![Erreur][1]

L’application est prête lorsqu’elle est à l’état ```Ready``` : ![Prête][2]

Ouvrez un navigateur et accédez à `http://containercluster.westus2.cloudapp.azure.com:8081`. Vous devez voir le titre « Hello World ! » s’afficher dans le navigateur.

## <a name="clean-up"></a>Nettoyer

Vous continuez à être facturé tant que le cluster est en cours d’exécution. Pensez à [supprimer votre cluster](./service-fabric-tutorial-delete-cluster.md).

Une fois l’image envoyée (push) dans le registre de conteneurs, vous pouvez supprimer l’image locale de votre ordinateur de développement :

```
docker rmi helloworldapp
docker rmi myregistry.azurecr.io/samples/helloworldapp
```

## <a name="windows-server-container-os-and-host-os-compatibility"></a>Système d’exploitation de conteneur Windows Server et compatibilité avec le système d’exploitation hôte

Les conteneurs Windows Server ne sont pas compatibles avec toutes les versions d’un système d’exploitation hôte. Par exemple :
 
- Les conteneurs Windows Server créés à l’aide de Windows Server version 1709 ne fonctionnent pas sur un hôte exécutant Windows Server version 2016. 
- Les conteneurs Windows Server créés à l’aide de Windows Server 2016 fonctionnent en mode d’isolation Hyper-V uniquement sur un hôte exécutant Windows Server version 1709. 
- Avec les conteneurs Windows Server créés à l’aide de Windows Server 2016, il peut être nécessaire de vérifier que la version du système d’exploitation du conteneur et celle du système d’exploitation hôte sont identiques lors d’une exécution en mode d’isolation des processus sur un hôte exécutant Windows Server 2016.
 
Pour plus d’informations, consultez [Compatibilité des versions avec les conteneurs Windows](/virtualization/windowscontainers/deploy-containers/version-compatibility).

Examinez la compatibilité du système d’exploitation hôte et le système d’exploitation de votre conteneur pour créer et déployer des conteneurs sur votre cluster Service Fabric. Par exemple :

- Vérifiez que vous déployez des conteneurs avec un système d’exploitation compatible avec le système d’exploitation exécuté sur vos nœuds de cluster.
- Vérifiez que le mode d’isolation spécifié pour votre application de conteneur est cohérent avec la prise en charge du système d’exploitation de conteneur sur le nœud sur lequel il est en cours de déploiement.
- Tenez compte de la façon dont les mises à niveau du système d’exploitation sur vos nœuds de cluster ou conteneurs peuvent affecter leur compatibilité. 

Nous vous recommandons les pratiques suivantes pour vérifier que les conteneurs sont déployés correctement sur votre cluster Service Fabric :

- Pour spécifier la version du système d’exploitation de Windows Server à partir de laquelle un conteneur, est généré, utilisez le balisage d’images explicite avec vos images Docker. 
- Utilisez le [balisage du système d’exploitation](#specify-os-build-specific-container-images) dans votre fichier manifeste d’application pour garantir que votre application est compatible entre les différentes versions et mises à niveau de Windows Server.

> [!NOTE]
> Avec Service Fabric version 6.2 et versions ultérieures, vous pouvez déployer des conteneurs basés sur Windows Server 2016 localement sur un hôte Windows 10. Sur Windows 10, les conteneurs sont exécutés en mode d’isolation Hyper-V, quel que soit le mode d’isolation défini dans le manifeste d’application. Pour plus d’informations, consultez [Configurer le mode d’isolation](#configure-isolation-mode).   
>
 
## <a name="specify-os-build-specific-container-images"></a>Spécifier les images conteneur spécifiques au build du système d’exploitation 

Les conteneurs Windows Server peuvent ne pas être compatibles entre les différentes versions du système d’exploitation. Par exemple, les conteneurs Windows Server générés à l’aide de Windows Server 2016 ne fonctionnent pas sur Windows Server version 1709 en mode d’isolation des processus. Par conséquent, si des nœuds de cluster sont mis à jour vers la dernière version, les services de conteneur créés avec les versions antérieures du système d’exploitation peuvent échouer. Pour contourner ce problème avec la version 6.1 du runtime ou une version plus récente, Service Fabric prend en charge la spécification de plusieurs images du système d’exploitation par conteneur et leur balisage avec les versions de build du système d’exploitation dans le manifeste d’application. Vous pouvez obtenir la version de build du système d’exploitation en exécutant `winver` à partir d’une invite de commandes Windows. Mettez à jour les manifestes de l’application et spécifiez les remplacements d’image pour chaque version du système d’exploitation avant de mettre à jour le système d’exploitation sur les nœuds. L’extrait de code suivant montre comment spécifier plusieurs images de conteneur dans le manifeste d’application **ApplicationManifest.xml** :


```xml
      <ContainerHostPolicies> 
         <ImageOverrides> 
           <Image Name="myregistry.azurecr.io/samples/helloworldappDefault" /> 
               <Image Name="myregistry.azurecr.io/samples/helloworldapp1701" Os="14393" /> 
               <Image Name="myregistry.azurecr.io/samples/helloworldapp1709" Os="16299" /> 
         </ImageOverrides> 
      </ContainerHostPolicies> 
```
La version de build pour WIndows Server 2016 est 14393, et la version de build pour Windows Server version 1709 est 16299. Le manifeste de service continue de ne spécifier qu’une image par service de conteneur, comme ci-dessous :

```xml
<ContainerHost>
    <ImageName>myregistry.azurecr.io/samples/helloworldapp</ImageName> 
</ContainerHost>
```

   > [!NOTE]
   > Les fonctionnalités de marquage de la version de build du système d’exploitation sont disponibles uniquement pour Service Fabric sur Windows
   >

Si le système d’exploitation sous-jacent sur la machine virtuelle est build 16299 (version 1709), Service Fabric choisit l’image de conteneur correspondant à cette version de Windows Server. Si une image de conteneur non marquée est également fournie avec des images de conteneur marquées dans le manifeste d’application, Service Fabric traite l’image non marquée comme une image qui fonctionne sur plusieurs versions. Marquez les images conteneur de façon explicite, afin d’éviter tout problème pendant les mises à niveau.

L’image conteneur non marquée sert de remplacement à celle fournie dans le fichier ServiceManifest. Ainsi l’image « myregistry.azurecr.io/samples/helloworldappDefault » remplace l’image ImageName « myregistry.azurecr.io/samples/helloworldapp » indiquée dans le fichier ServiceManifest.

## <a name="complete-example-service-fabric-application-and-service-manifests"></a>Exemples complets de manifestes d’application et de service Service Fabric
Voici les manifestes d’application et de service complets utilisés dans cet article.

### <a name="servicemanifestxml"></a>ServiceManifest.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="Guest1Pkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="https://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <ServiceTypes>
    <!-- This is the name of your ServiceType.
         The UseImplicitHost attribute indicates this is a guest service. -->
    <StatelessServiceType ServiceTypeName="Guest1Type" UseImplicitHost="true" />
  </ServiceTypes>

  <!-- Code package is your service executable. -->
  <CodePackage Name="Code" Version="1.0.0">
    <EntryPoint>
      <!-- Follow this link for more information about deploying Windows containers to Service Fabric: https://aka.ms/sfguestcontainers -->
      <ContainerHost>
        <ImageName>myregistry.azurecr.io/samples/helloworldapp</ImageName>
        <!-- Pass comma delimited commands to your container: dotnet, myproc.dll, 5" -->
        <!--Commands> dotnet, myproc.dll, 5 </Commands-->
        <Commands></Commands>
      </ContainerHost>
    </EntryPoint>
    <!-- Pass environment variables to your container: -->    
    <EnvironmentVariables>
      <EnvironmentVariable Name="HttpGatewayPort" Value=""/>
      <EnvironmentVariable Name="BackendServiceName" Value=""/>
    </EnvironmentVariables>

  </CodePackage>

  <!-- Config package is the contents of the Config directory under PackageRoot that contains an
       independently-updateable and versioned set of custom configuration settings for your service. -->
  <ConfigPackage Name="Config" Version="1.0.0" />

  <Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the port on which to
           listen. Please note that if your service is partitioned, this port is shared with
           replicas of different partitions that are placed in your code. -->
      <Endpoint Name="Guest1TypeEndpoint" UriScheme="http" Port="8081" Protocol="http"/>
    </Endpoints>
  </Resources>
</ServiceManifest>
```
### <a name="applicationmanifestxml"></a>ApplicationManifest.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest ApplicationTypeName="MyFirstContainerType"
                     ApplicationTypeVersion="1.0.0"
                     xmlns="http://schemas.microsoft.com/2011/01/fabric"
                     xmlns:xsd="https://www.w3.org/2001/XMLSchema"
                     xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <Parameters>
    <Parameter Name="Guest1_InstanceCount" DefaultValue="-1" />
  </Parameters>
  <!-- Import the ServiceManifest from the ServicePackage. The ServiceManifestName and ServiceManifestVersion
       should match the Name and Version attributes of the ServiceManifest element defined in the
       ServiceManifest.xml file. -->
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="Guest1Pkg" ServiceManifestVersion="1.0.0" />
    <EnvironmentOverrides CodePackageRef="FrontendService.Code">
      <EnvironmentVariable Name="HttpGatewayPort" Value="19080"/>
    </EnvironmentOverrides>
    <ConfigOverrides />
    <Policies>
      <ContainerHostPolicies CodePackageRef="Code">
        <RepositoryCredentials AccountName="myregistry" Password="MIIB6QYJKoZIhvcNAQcDoIIB2jCCAdYCAQAxggFRMIIBTQIBADA1MCExHzAdBgNVBAMMFnJ5YW53aWRhdGFlbmNpcGhlcm1lbnQCEFfyjOX/17S6RIoSjA6UZ1QwDQYJKoZIhvcNAQEHMAAEg
gEAS7oqxvoz8i6+8zULhDzFpBpOTLU+c2mhBdqXpkLwVfcmWUNA82rEWG57Vl1jZXe7J9BkW9ly4xhU8BbARkZHLEuKqg0saTrTHsMBQ6KMQDotSdU8m8Y2BR5Y100wRjvVx3y5+iNYuy/JmM
gSrNyyMQ/45HfMuVb5B4rwnuP8PAkXNT9VLbPeqAfxsMkYg+vGCDEtd8m+bX/7Xgp/kfwxymOuUCrq/YmSwe9QTG3pBri7Hq1K3zEpX4FH/7W2Zb4o3fBAQ+FuxH4nFjFNoYG29inL0bKEcTX
yNZNKrvhdM3n1Uk/8W2Hr62FQ33HgeFR1yxQjLsUu800PrYcR5tLfyTB8BgkqhkiG9w0BBwEwHQYJYIZIAWUDBAEqBBBybgM5NUV8BeetUbMR8mJhgFBrVSUsnp9B8RyebmtgU36dZiSObDsI
NtTvlzhk11LIlae/5kjPv95r3lw6DHmV4kXLwiCNlcWPYIWBGIuspwyG+28EWSrHmN7Dt2WqEWqeNQ==" PasswordEncrypted="true"/>
        <PortBinding ContainerPort="80" EndpointRef="Guest1TypeEndpoint"/>
      </ContainerHostPolicies>
      <ServicePackageResourceGovernancePolicy CpuCores="1"/>
      <ResourceGovernancePolicy CodePackageRef="Code" MemoryInMB="1024"  />
    </Policies>
  </ServiceManifestImport>
  <DefaultServices>
    <!-- The section below creates instances of service types, when an instance of this
         application type is created. You can also create one or more instances of service type using the
         ServiceFabric PowerShell module.

         The attribute ServiceTypeName below must match the name defined in the imported ServiceManifest.xml file. -->
    <Service Name="Guest1">
      <StatelessService ServiceTypeName="Guest1Type" InstanceCount="[Guest1_InstanceCount]">
        <SingletonPartition />
      </StatelessService>
    </Service>
  </DefaultServices>
</ApplicationManifest>
```

## <a name="configure-time-interval-before-container-is-force-terminated"></a>Configurer l’intervalle de temps d’attente avant l’arrêt forcé du conteneur

Vous pouvez configurer un intervalle de temps d’attente pour le runtime avant que le conteneur ne soit supprimé après le début de la suppression du service (ou d’un déplacement vers un autre nœud). Le fait de configurer l’intervalle de temps envoie la commande `docker stop <time in seconds>` au conteneur.  Pour plus d’informations, consultez [Arrêt du docker](https://docs.docker.com/engine/reference/commandline/stop/). L’intervalle de temps d’attente est spécifié dans la section `Hosting`. La section `Hosting` peut être ajoutée au moment de la création du cluster ou ultérieurement dans une mise à niveau de la configuration. L’extrait de manifeste de cluster suivant montre comment définir l’intervalle d’attente :

```json
"fabricSettings": [
    ...,
    {
        "name": "Hosting",
        "parameters": [
          {
                "name": "ContainerDeactivationTimeout",
                "value" : "10"
          },
          ...
        ]
    }
]
```
L’intervalle de temps par défaut est défini sur 10 secondes. Étant donné que cette configuration est dynamique, une mise à niveau uniquement de la configuration sur un cluster met à jour le délai d’expiration. 


## <a name="configure-the-runtime-to-remove-unused-container-images"></a>Configurer le runtime pour supprimer les images conteneur inutilisées

Vous pouvez configurer le cluster Service Fabric pour supprimer des images conteneur inutilisées à partir du nœud. Cette configuration permet à l’espace disque d’être rétabli si trop d’images conteneur sont présentes sur le nœud. Pour activer cette fonctionnalité, mettez à jour la section [Hosting](service-fabric-cluster-fabric-settings.md#hosting) du manifeste de cluster, comme indiqué dans l’extrait de code suivant : 


```json
"fabricSettings": [
    ...,
    {
        "name": "Hosting",
        "parameters": [
          {
                "name": "PruneContainerImages",
                "value": "True"
          },
          {
                "name": "ContainerImagesToSkip",
                "value": "mcr.microsoft.com/windows/servercore|mcr.microsoft.com/windows/nanoserver|mcr.microsoft.com/dotnet/framework/aspnet|..."
          }
          ...
          }
        ]
    } 
]
```

Vous pouvez spécifier les images qui ne doivent pas être supprimées à l’aide du paramètre `ContainerImagesToSkip`.  


## <a name="configure-container-image-download-time"></a>Configurer le temps de téléchargement de l’image de conteneur

Le runtime Service Fabric alloue 20 minutes pour télécharger et extraire les images conteneur, ce qui fonctionne pour la majorité d’entre elles. Pour les images de grande taille, ou lorsque la connexion réseau est lente, il peut être nécessaire d’augmenter le temps d’attente avant d’annuler le téléchargement et l’extraction de l’image. Ce délai peut être défini à l’aide de l’attribut **ContainerImageDownloadTimeout** dans la section **Hébergement** du manifeste de cluster, comme indiqué dans l’extrait de code suivant :

```json
"fabricSettings": [
    ...,
    {
        "name": "Hosting",
        "parameters": [
          {
              "name": "ContainerImageDownloadTimeout",
              "value": "1200"
          }
        ]
    }
]
```


## <a name="set-container-retention-policy"></a>Définir la stratégie de rétention de conteneur

Pour aider à diagnostiquer les échecs de démarrage du conteneur, Service Fabric (version 6.1 ou supérieure) prend en charge les conteneurs conservés qui ont terminé ou échoué leur démarrage. Cette stratégie peut être définie dans le fichier **ApplicationManifest.xml** comme indiqué dans l’extrait de code suivant :

```xml
 <ContainerHostPolicies CodePackageRef="NodeService.Code" Isolation="process" ContainersRetentionCount="2"  RunInteractive="true"> 
```

Le paramètre **ContainersRetentionCount** spécifie le nombre de conteneurs à conserver en cas d’échec. Si une valeur négative est spécifiée, tous les conteneurs en échec sont conservés. Quand l’attribut **ContainersRetentionCount** n’est pas spécifié, aucun conteneur n’est conservé. L’attribut **ContainersRetentionCount** prend également en charge les paramètres de l’application. Les utilisateurs peuvent ainsi spécifier des valeurs différentes pour les clusters de test et de production. Utilisez des contraintes de placement pour cibler le service conteneur sur un nœud particulier lors de l’utilisation de cette fonctionnalité pour empêcher le service conteneur de se déplacer vers d’autres nœuds. Tous les conteneurs conservés à l’aide de cette fonctionnalité doivent être supprimés manuellement.

## <a name="start-the-docker-daemon-with-custom-arguments"></a>Démarrer le démon Docker avec des arguments personnalisés

Avec la version 6.2 du runtime Service Fabric et versions supérieures, vous pouvez démarrer le démon Docker avec des arguments personnalisés. Lorsque des arguments personnalisés sont spécifiés, Service Fabric ne transmet pas d’autres arguments au moteur Docker, à l’exception de l’argument `--pidfile`. Par conséquent, `--pidfile` ne doit pas être transmis en tant qu’argument. En outre, le démon Docker de l’argument doit continuer à écouter le canal de nom par défaut sur Windows (ou le socket de domaine Unix sur Linux) pour que Service Fabric communique avec le démon. Les arguments personnalisés sont transmis dans le manifeste de cluster, dans la section **Hébergement** sous **ContainerServiceArguments**, comme indiqué dans l’extrait de code suivant : 
 

```json
"fabricSettings": [
    ...,
    { 
        "name": "Hosting", 
        "parameters": [ 
          { 
            "name": "ContainerServiceArguments", 
            "value": "-H localhost:1234 -H unix:///var/run/docker.sock" 
          } 
        ] 
    } 
]
```

## <a name="next-steps"></a>Étapes suivantes
* En savoir plus sur l’exécution des [conteneurs sur Service Fabric](service-fabric-containers-overview.md).
* Consultez le didacticiel [Déployer une application .NET dans un conteneur vers Azure Service Fabric](service-fabric-host-app-in-a-container.md).
* En savoir plus sur le [cycle de vie des applications](service-fabric-application-lifecycle.md) Service Fabric.
* Consulter les [exemples de code de conteneur Service Fabric](https://github.com/Azure-Samples/service-fabric-containers) sur GitHub.

[1]: ./media/service-fabric-get-started-containers/MyFirstContainerError.png
[2]: ./media/service-fabric-get-started-containers/MyFirstContainerReady.png
[3]: ./media/service-fabric-get-started-containers/HealthCheckHealthy.png
[4]: ./media/service-fabric-get-started-containers/HealthCheckUnhealthy_App.png
[5]: ./media/service-fabric-get-started-containers/HealthCheckUnhealthy_Dsp.png

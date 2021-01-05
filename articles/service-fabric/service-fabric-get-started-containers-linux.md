---
title: Créer une application de conteneur Azure Service Fabric sous Linux
description: Créez votre première application de conteneur Linux sur Microsoft Azure Service Fabric. Concevez une image Docker avec votre application, envoyez l’image vers un registre de conteneurs, créez et déployez une application de conteneur Service Fabric.
ms.topic: conceptual
ms.date: 1/4/2019
ms.custom: devx-track-python
ms.openlocfilehash: 0481cc2d36f7882bbd8eea9b984c3dc388de5dee
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96534078"
---
# <a name="create-your-first-service-fabric-container-application-on-linux"></a>Créer votre première application de conteneur Service Fabric sur Linux
> [!div class="op_single_selector"]
> * [Windows](service-fabric-get-started-containers.md)
> * [Linux](service-fabric-get-started-containers-linux.md)

L’exécution d’une application existante dans un conteneur Linux sur un cluster Service Fabric ne nécessite aucune modification de votre application. Cet article vous accompagne dans la création d’une image Docker contenant une application web [Flask](http://flask.pocoo.org/) Python et le déploiement dans un cluster Service Fabric. Vous allez également partager votre application en conteneur via [Azure Container Registry](../container-registry/index.yml). Cet article suppose une connaissance élémentaire de Docker. Pour en savoir plus sur Docker, consultez la [présentation de Docker](https://docs.docker.com/engine/understanding-docker/).

> [!NOTE]
> Cet article s’applique à un environnement de développement Linux.  Le runtime du cluster Service Fabric et le runtime de Docker doivent être en cours d’exécution sur le même système d’exploitation.  Vous ne pouvez pas exécuter des conteneurs Linux sur un cluster Windows.

## <a name="prerequisites"></a>Prérequis
* Un ordinateur de développement exécutant :
  * [Outils et SDK Service Fabric](service-fabric-get-started-linux.md).
  * [Docker CE pour Linux](https://docs.docker.com/engine/installation/#prior-releases). 
  * [Interface de ligne de commande de Service Fabric](service-fabric-cli.md)

* Un cluster Linux avec trois nœuds ou plus.

* Un registre dans Azure Container Registry. [Créez un registre de conteneurs](../container-registry/container-registry-get-started-portal.md) dans votre abonnement Azure. 

## <a name="define-the-docker-container"></a>Définir le conteneur Docker
Créez une image basée sur [l’image Python](https://hub.docker.com/_/python/) située sur Docker Hub. 

Spécifiez votre conteneur Docker dans un fichier Dockerfile. Le fichier Dockerfile contient des instructions pour la configuration de l’environnement à l’intérieur de votre conteneur, le chargement de l’application que vous souhaitez exécuter et le mappage des ports. Le fichier Dockerfile est l’entrée de la commande `docker build`, qui crée l’image. 

Créez un répertoire vide et créez le fichier *Dockerfile* (sans extension de fichier). Ajoutez ce qui suit au fichier *Dockerfile* et enregistrez vos modifications :

```
# Use an official Python runtime as a base image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
ADD . /app

# Install any needed packages specified in requirements.txt
RUN pip install -r requirements.txt

# Make port 80 available outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```

Pour plus d'informations, consultez les [références Dockerfile](https://docs.docker.com/engine/reference/builder/).

## <a name="create-a-basic-web-application"></a>Créer une application web de base
Créez une application web Flask écoutant le port 80 qui renvoie « Hello World! ». Dans le même répertoire, créez le fichier *requirements.txt*. Ajoutez ce qui suit et enregistrez vos modifications :
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

## <a name="login-to-docker-and-build-the-image"></a>Connexion à Docker et génération de l’image

Nous allons maintenant créer l’image qui exécute votre application web. Lorsque vous extrayez des images publiques de Docker (comme `python:2.7-slim` dans notre fichier Dockerfile), il est recommandé de s’authentifier auprès de votre compte Docker Hub au lieu de créer une demande de tirage anonyme.

> [!NOTE]
> Lorsque vous effectuez des demande de tirage (pull request) anonymes fréquentes, vous pouvez voir des erreurs Docker similaires à `ERROR: toomanyrequests: Too Many Requests.` ou `You have reached your pull rate limit.` Authentifiez-vous auprès de Docker Hub pour éviter ces erreurs. Consultez [Gérer le contenu public à l’aide d’Azure Container Registry](../container-registry/buffer-gate-public-content.md) pour plus d’informations.

Ouvrez une fenêtre PowerShell et accédez au répertoire contenant le fichier Dockerfile. Exécutez ensuite les commande suivantes :

```
docker login
docker build -t helloworldapp .
```

Cette commande crée l’image en suivant les instructions de votre fichier Dockerfile, et la nomme (balisage -t) `helloworldapp`. Pour générer une image conteneur, l’image de base est tout d’abord téléchargée depuis le Hub Docker auquel l’application est ajoutée. 

Une fois la création terminée, exécutez la commande `docker images` pour afficher des informations sur la nouvelle image :

```bash
$ docker images
    
REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
helloworldapp                 latest              86838648aab6        2 minutes ago       194 MB
```

## <a name="run-the-application-locally"></a>Exécuter l’application localement
Vérifiez que votre application en conteneur s’exécute en local avant de l’envoyer dans le registre de conteneurs. 

Exécutez l’application, en mappant le port 4000 de votre ordinateur au port 80 exposé du conteneur :

```bash
docker run -d -p 4000:80 --name my-web-site helloworldapp
```

*name* donne un nom au conteneur en cours d’exécution (au lieu d’utiliser l’ID de conteneur).

Connectez le conteneur en cours d’exécution. Ouvrez un navigateur web qui pointe vers l’adresse IP renvoyée sur le port 4000, par exemple « http:\//localhost:4000 ». Vous devez voir le titre « Hello World ! » s’afficher dans le navigateur.

![Hello World!][hello-world]

Pour arrêter votre conteneur, exécutez :

```bash
docker stop my-web-site
```

Supprimez le conteneur de votre ordinateur de développement :

```bash
docker rm my-web-site
```

## <a name="push-the-image-to-the-container-registry"></a>Envoyer l’image dans le registre de conteneurs
Après avoir vérifié que l’application s’exécute dans Docker, envoyez l’image à votre registre dans Azure Container Registry.

Exécutez `docker login` pour vous connecter à votre registre de conteneurs à l’aide des [informations d’identification de votre registre](../container-registry/container-registry-authentication.md).

L’exemple suivant transmet l’ID et le mot de passe d’un [principal du service](../active-directory/develop/app-objects-and-service-principals.md) Azure Active Directory . Par exemple, vous pouvez avoir affecté un principal du service à votre Registre pour un scénario d’automatisation. Vous pouvez aussi vous connecter à l’aide de votre nom d’utilisateur du registre et votre mot de passe.

```bash
docker login myregistry.azurecr.io -u xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx -p myPassword
```

La commande suivante crée une balise, ou un alias, de l’image, avec un chemin d’accès qualifié complet vers votre registre. Cet exemple place l’image dans l’espace de noms `samples` pour éviter un encombrement à la racine du registre.

```bash
docker tag helloworldapp myregistry.azurecr.io/samples/helloworldapp
```

Envoyez l’image vers votre registre de conteneurs :

```bash
docker push myregistry.azurecr.io/samples/helloworldapp
```

## <a name="package-the-docker-image-with-yeoman"></a>Placer l’image Docker dans un package avec Yeoman
Le Kit de développement logiciel (SDK) Service Fabric pour Linux comprend un générateur [Yeoman](https://yeoman.io/) qui facilite la création de votre application et l’ajout d’une image de conteneur. Nous allons utiliser Yeoman pour créer une application avec un seul conteneur Docker appelé *SimpleContainerApp*.

Pour créer une application de conteneur Service Fabric, ouvrez une fenêtre de terminal et exécutez `yo azuresfcontainer`. 

Nommez votre application (par exemple, `mycontainer`) et le service d’application (par exemple, `myservice`).

En guise de nom d’image, fournissez l’URL de l’image de conteneurs dans un registre de conteneur (ex : « myregistry.azurecr.io/samples/helloworldapp »). 

Cette image possède un point d’entrée de charge de travail défini, vous n’avez donc pas à spécifier des commandes d’entrée explicitement (les commandes s’exécutent dans le conteneur et garderont le conteneur en exécution après le démarrage). 

Spécifiez un nombre d’instances de « 1 ».

Spécifiez le mappage de port au format approprié. Pour cet article, vous devez fournir ```80:4000``` en tant que mappage de port. En le faisant, vous avez configuré que toutes les requêtes arrivant sur le port 4000 de la machine hôte sont redirigées vers le port 80 du conteneur.

![Générateur Yeoman Service Fabric pour les conteneurs][sf-yeoman]

## <a name="configure-container-repository-authentication"></a>Configurer l’authentification des référentiels de conteneur

Pour configurer différents types d'authentification afin de télécharger des images conteneur, consultez [Authentification des référentiels de conteneur](configure-container-repository-credentials.md).

## <a name="configure-isolation-mode"></a>Configurer le mode d’isolation
La version 6.3 du runtime prend en charge l'isolation des machines virtuelles pour les conteneurs Linux. Deux modes d'isolation sont ainsi pris en charge pour les conteneurs : process et Hyper-V. Avec le mode d'isolation Hyper-V, les noyaux sont isolés entre chaque conteneur et l'hôte du conteneur. L'isolation Hyper-V est implémentée à l'aide de l'option [Nettoyer les conteneurs](https://software.intel.com/en-us/articles/intel-clear-containers-2-using-clear-containers-with-docker). Le mode d’isolation est spécifié pour les clusters Linux dans l’élément `ServicePackageContainerPolicy` dans le fichier manifeste de l’application. Les modes d’isolation qui peuvent être définis sont `process`, `hyperv` et `default`. La valeur par défaut est le mode d’isolation de processus. L’extrait de code suivant montre comment le mode d’isolation est spécifié dans le fichier manifeste de l’application.

```xml
<ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="MyServicePkg" ServiceManifestVersion="1.0.0"/>
      <Policies>
        <ServicePackageContainerPolicy Hostname="votefront" Isolation="hyperv">
          <PortBinding ContainerPort="80" EndpointRef="myServiceTypeEndpoint"/>
        </ServicePackageContainerPolicy>
    </Policies>
  </ServiceManifestImport>
```


## <a name="configure-resource-governance"></a>Configurer la gouvernance des ressources
La [gouvernance des ressources](service-fabric-resource-governance.md) limite les ressources que le conteneur peut utiliser sur l’hôte. L’élément `ResourceGovernancePolicy`, spécifié dans le manifeste de l’application, est utilisé pour déclarer des limites relatives aux ressources pour un package de code de service. Des limites de ressources peuvent être définies pour les ressources suivantes : Memory, MemorySwap, CpuShares (poids relatif du processeur), MemoryReservationInMB, BlkioWeight (poids relatif de l’élément BlockIO). Dans cet exemple, le package de service Guest1Pkg obtient un cœur sur les nœuds de cluster où il est placé. Les limites de mémoire sont absolues, ce qui signifie que le package de code est limité à 1024 Mo de mémoire (avec une garantie de réservation identique). Les packages de code (conteneurs ou processus) ne sont pas en mesure d’allouer plus de mémoire que cette limite. Toute tentative en ce sens conduit à une exception de mémoire insuffisante. Pour pouvoir appliquer la limite de ressources, des limites de mémoire doivent être spécifiées pour tous les packages de code au sein d’un package de service.

```xml
<ServiceManifestImport>
  <ServiceManifestRef ServiceManifestName="MyServicePKg" ServiceManifestVersion="1.0.0" />
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

![La capture d’écran montre les détails du package de services déployé NodeServicePackage.][1]

![HealthCheckUnhealthyApp][2]

![HealthCheckUnhealthyDsp][3]

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

## <a name="deploy-the-application"></a>Déployer l’application
Une fois que l’application est générée, vous pouvez la déployer vers le cluster local à l’aide de l’interface de ligne de commande Service Fabric.

Connectez-vous au cluster Service Fabric local.

```bash
sfctl cluster select --endpoint http://localhost:19080
```

Utilisez le script d’installation fourni dans le modèle à l’adresse https://github.com/Azure-Samples/service-fabric-containers/ pour copier le package d’application dans le magasin d’images du cluster, inscrire le type d’application et créer une instance de l’application.


```bash
./install.sh
```

Ouvrez un navigateur et accédez à Service Fabric Explorer à l’adresse http:\//localhost:19080/Explorer (remplacez localhost par l’adresse IP privée de la machine virtuelle si vous utilisez Vagrant sur Mac OS X). Développez le nœud Applications et notez qu’il existe désormais une entrée pour votre type d’application et une autre pour la première instance de ce type.

Connectez le conteneur en cours d’exécution. Ouvrez un navigateur web qui pointe vers l’adresse IP renvoyée sur le port 4000, par exemple « http:\//localhost:4000 ». Vous devez voir le titre « Hello World ! » s’afficher dans le navigateur.

![Hello World!][hello-world]


## <a name="clean-up"></a>Nettoyer
Utilisez le script de désinstallation fourni dans le modèle pour supprimer l’instance de l’application du cluster de développement local et annuler l’inscription du type d’application.

```bash
./uninstall.sh
```

Une fois l’image publiée dans le registre de conteneurs, vous pouvez supprimer l’image locale de votre ordinateur de développement :

```
docker rmi helloworldapp
docker rmi myregistry.azurecr.io/samples/helloworldapp
```

## <a name="complete-example-service-fabric-application-and-service-manifests"></a>Exemples complets de manifestes d’application et de service Service Fabric
Voici les manifestes d’application et de service complets utilisés dans cet article.

### <a name="servicemanifestxml"></a>ServiceManifest.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="myservicePkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="https://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <ServiceTypes>
    <!-- This is the name of your ServiceType.
         The UseImplicitHost attribute indicates this is a guest service. -->
    <StatelessServiceType ServiceTypeName="myserviceType" UseImplicitHost="true" />
  </ServiceTypes>

  <!-- Code package is your service executable. -->
  <CodePackage Name="Code" Version="1.0.0">
    <EntryPoint>
      <!-- Follow this link for more information about deploying containers 
      to Service Fabric: https://aka.ms/sfguestcontainers -->
      <ContainerHost>
        <ImageName>myregistry.azurecr.io/samples/helloworldapp</ImageName>
        <!-- Pass comma delimited commands to your container: dotnet, myproc.dll, 5" -->
        <!--Commands> dotnet, myproc.dll, 5 </Commands-->
        <Commands></Commands>
      </ContainerHost>
    </EntryPoint>
    <!-- Pass environment variables to your container: -->
    
    <EnvironmentVariables>
      <!--
      <EnvironmentVariable Name="VariableName" Value="VariableValue"/>
      -->
    </EnvironmentVariables>
    
  </CodePackage>

  <Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the port on which to 
           listen. Please note that if your service is partitioned, this port is shared with 
           replicas of different partitions that are placed in your code. -->
      <Endpoint Name="myServiceTypeEndpoint" UriScheme="http" Port="4000" Protocol="http"/>
    </Endpoints>
  </Resources>
</ServiceManifest>
```
### <a name="applicationmanifestxml"></a>ApplicationManifest.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest ApplicationTypeName="mycontainerType"
                     ApplicationTypeVersion="1.0.0"
                     xmlns="http://schemas.microsoft.com/2011/01/fabric"
                     xmlns:xsd="https://www.w3.org/2001/XMLSchema"
                     xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <!-- Import the ServiceManifest from the ServicePackage. The ServiceManifestName and ServiceManifestVersion 
       should match the Name and Version attributes of the ServiceManifest element defined in the 
       ServiceManifest.xml file. -->
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="myservicePkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
      <ContainerHostPolicies CodePackageRef="Code">
        <RepositoryCredentials AccountName="myregistry" Password="=P==/==/=8=/=+u4lyOB=+=nWzEeRfF=" PasswordEncrypted="false"/>
        <PortBinding ContainerPort="80" EndpointRef="myServiceTypeEndpoint"/>
      </ContainerHostPolicies>
    </Policies>
  </ServiceManifestImport>
  <DefaultServices>
    <!-- The section below creates instances of service types, when an instance of this 
         application type is created. You can also create one or more instances of service type using the 
         ServiceFabric PowerShell module.
         
         The attribute ServiceTypeName below must match the name defined in the imported ServiceManifest.xml file. -->
    <Service Name="myservice">
      <!-- On a local development cluster, set InstanceCount to 1. On a multi-node production 
      cluster, set InstanceCount to -1 for the container service to run on every node in 
      the cluster.
      -->
      <StatelessService ServiceTypeName="myserviceType" InstanceCount="1">
        <SingletonPartition />
      </StatelessService>
    </Service>
  </DefaultServices>
</ApplicationManifest>
```
## <a name="adding-more-services-to-an-existing-application"></a>Ajout d’autres services à une application existante

Pour ajouter un autre service de conteneur à une application déjà créée à l’aide de Yeoman, procédez comme suit :

1. Accédez au répertoire à la racine de l’application existante. Par exemple, `cd ~/YeomanSamples/MyApplication`, si `MyApplication` est l’application créée par Yeoman.
2. Exécutez `yo azuresfcontainer:AddService`

<a id="manually"></a>


## <a name="configure-time-interval-before-container-is-force-terminated"></a>Configurer l’intervalle de temps d’attente avant l’arrêt forcé du conteneur

Vous pouvez configurer un intervalle de temps d’attente pour le runtime avant que le conteneur ne soit supprimé après le début de la suppression du service (ou d’un déplacement vers un autre nœud). Le fait de configurer l’intervalle de temps envoie la commande `docker stop <time in seconds>` au conteneur.  Pour plus d’informations, consultez [Arrêt du docker](https://docs.docker.com/engine/reference/commandline/stop/). L’intervalle de temps d’attente est spécifié dans la section `Hosting`. L’extrait de manifeste de cluster suivant montre comment définir l’intervalle d’attente :


```json
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
```

L’intervalle de temps par défaut est défini sur 10 secondes. Étant donné que cette configuration est dynamique, une mise à niveau uniquement de la configuration sur un cluster met à jour le délai d’expiration. 

## <a name="configure-the-runtime-to-remove-unused-container-images"></a>Configurer le runtime pour supprimer les images conteneur inutilisées

Vous pouvez configurer le cluster Service Fabric pour supprimer des images conteneur inutilisées à partir du nœud. Cette configuration permet à l’espace disque d’être rétabli si trop d’images conteneur sont présentes sur le nœud. Pour activer cette fonctionnalité, mettez à jour la section `Hosting` du manifeste de cluster, comme indiqué dans l’extrait de code suivant : 


```json
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
```

Vous pouvez spécifier les images qui ne doivent pas être supprimées à l’aide du paramètre `ContainerImagesToSkip`. 

## <a name="configure-container-image-download-time"></a>Configurer le temps de téléchargement de l’image de conteneur

Le runtime Service Fabric alloue 20 minutes pour télécharger et extraire les images conteneur, ce qui fonctionne pour la majorité d’entre elles. Pour les images de grande taille, ou lorsque la connexion réseau est lente, il peut être nécessaire d’augmenter le temps d’attente avant d’annuler le téléchargement et l’extraction de l’image. Ce délai peut être défini à l’aide de l’attribut **ContainerImageDownloadTimeout** dans la section **Hébergement** du manifeste de cluster, comme indiqué dans l’extrait de code suivant :

```json
{
        "name": "Hosting",
        "parameters": [
          {
              "name": "ContainerImageDownloadTimeout",
              "value": "1200"
          }
        ]
}
```


## <a name="set-container-retention-policy"></a>Définir la stratégie de rétention de conteneur

Pour aider à diagnostiquer les échecs de démarrage du conteneur, Service Fabric (version 6.1 ou supérieure) prend en charge les conteneurs conservés qui ont terminé ou échoué leur démarrage. Cette stratégie peut être définie dans le fichier **ApplicationManifest.xml** comme indiqué dans l’extrait de code suivant :

```xml
 <ContainerHostPolicies CodePackageRef="NodeService.Code" Isolation="process" ContainersRetentionCount="2"  RunInteractive="true"> 
```

Le paramètre **ContainersRetentionCount** spécifie le nombre de conteneurs à conserver en cas d’échec. Si une valeur négative est spécifiée, tous les conteneurs en échec sont conservés. Quand l’attribut **ContainersRetentionCount** n’est pas spécifié, aucun conteneur n’est conservé. L’attribut **ContainersRetentionCount** prend également en charge les paramètres de l’application. Les utilisateurs peuvent ainsi spécifier des valeurs différentes pour les clusters de test et de production. Utilisez des contraintes de placement pour cibler le service conteneur sur un nœud particulier lors de l’utilisation de cette fonctionnalité pour empêcher le service conteneur de se déplacer vers d’autres nœuds. Tous les conteneurs conservés à l’aide de cette fonctionnalité doivent être supprimés manuellement.

## <a name="start-the-docker-daemon-with-custom-arguments"></a>Démarrer le démon Docker avec des arguments personnalisés

Avec la version 6.2 du runtime Service Fabric et versions supérieures, vous pouvez démarrer le démon Docker avec des arguments personnalisés. Lorsque des arguments personnalisés sont spécifiés, Service Fabric ne transmet pas d’autres arguments au moteur Docker, à l’exception de l’argument `--pidfile`. Par conséquent, `--pidfile` ne doit pas être transmis en tant qu’argument. En outre, le démon Docker de l’argument doit continuer à écouter le canal de nom par défaut sur Windows (ou le socket de domaine Unix sur Linux) pour que Service Fabric communique avec le démon. Les arguments personnalisés sont spécifiés dans le manifeste de cluster, dans la section **Hébergement** sous **ContainerServiceArguments**. Un exemple est présenté dans l’extrait de code suivant : 
 

```json
{ 
        "name": "Hosting", 
        "parameters": [ 
          { 
            "name": "ContainerServiceArguments", 
            "value": "-H localhost:1234 -H unix:///var/run/docker.sock" 
          } 
        ] 
} 

```

## <a name="next-steps"></a>Étapes suivantes
* En savoir plus sur l’exécution des [conteneurs sur Service Fabric](service-fabric-containers-overview.md).
* Consultez le didacticiel [Déployer une application .NET dans un conteneur vers Azure Service Fabric](service-fabric-host-app-in-a-container.md).
* En savoir plus sur le [cycle de vie des applications](service-fabric-application-lifecycle.md) Service Fabric.
* Consulter les [exemples de code de conteneur Service Fabric](https://github.com/Azure-Samples/service-fabric-containers) sur GitHub.

[hello-world]: ./media/service-fabric-get-started-containers-linux/HelloWorld.png
[sf-yeoman]: ./media/service-fabric-get-started-containers-linux/YoSF.png

[1]: ./media/service-fabric-get-started-containers/HealthCheckHealthy.png
[2]: ./media/service-fabric-get-started-containers/HealthCheckUnhealthy_App.png
[3]: ./media/service-fabric-get-started-containers/HealthCheckUnhealthy_Dsp.png

---
title: Pilote de volume Azure Files pour Service Fabric
description: Service Fabric prend en charge l’utilisation d’Azure Files pour sauvegarder des volumes à partir de votre conteneur.
ms.topic: conceptual
ms.date: 6/10/2018
ms.openlocfilehash: a5125dbd88a2fe236196c427244f1311d9b73b9f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86247691"
---
# <a name="azure-files-volume-driver-for-service-fabric"></a>Pilote de volume Azure Files pour Service Fabric

Le pilote de volume Azure Files est un [plug-in de volume Docker](https://docs.docker.com/engine/extend/plugins_volume/) qui fournit des volumes basés sur [Azure Files](../storage/files/storage-files-introduction.md) pour les conteneurs Docker. Il se trouve dans un package en tant qu’application Service Fabric pouvant être déployée sur un cluster Service Fabric pour fournir des volumes pour d’autres applications de conteneur Service Fabric au sein du cluster.

> [!NOTE]
> La version 6.5.661.9590 du plug-in de volume Azure Files a été publiée pour la mise à la disposition générale.
>

## <a name="prerequisites"></a>Conditions préalables requises
* La version Windows du plug-in de volume Azure Files fonctionne sur [Windows Server version 1709](/windows-server/get-started/whats-new-in-windows-server-1709), [Windows 10 version 1709](/windows/whats-new/whats-new-windows-10-version-1709) ou des systèmes d’exploitation plus récents uniquement.

* La version Linux du plug-in de volume Azure Files fonctionne sur toutes les versions de système d’exploitation prises en charge par Service Fabric.

* Le plug-in du volume Azure Files fonctionne uniquement sur Service Fabric version 6.2 ou version plus récente.

* Suivez les instructions de la [documentation Azure Files](../storage/files/storage-how-to-create-file-share.md) pour créer un partage de fichiers pour l’application de conteneur Service Fabric à utiliser en tant que volume.

* Vous devrez avoir installé [Powershell avec le module Service Fabric](./service-fabric-get-started.md) ou [SFCTL](./service-fabric-cli.md).

* Si vous utilisez des conteneurs Hyper-V, les extraits de code suivants doivent être ajoutés dans ClusterManifest (cluster local), à la section fabricSettings dans votre modèle Azure Resource Manager (cluster Azure) ou dans le fichier ClusterConfig.json (cluster autonome).

Dans ClusterManifest, vous devez ajouter ce qui suit à la section Hosting. Dans cet exemple, le nom de volume est **sfazurefile** et le port qu’il écoute sur le cluster est **19100**. Remplacez-les par les valeurs correctes pour votre cluster.

``` xml 
<Section Name="Hosting">
  <Parameter Name="VolumePluginPorts" Value="sfazurefile:19100" />
</Section>
```

Dans la section fabricSettings de votre modèle Azure Resource Manager (pour les déploiements Azure) ou le fichier ClusterConfig.json (pour les déploiements autonomes), l’extrait de code suivant doit être ajouté. Une fois encore, remplacez le nom du volume et les valeurs de port par les vôtres.

```json
"fabricSettings": [
  {
    "name": "Hosting",
    "parameters": [
      {
          "name": "VolumePluginPorts",
          "value": "sfazurefile:19100"
      }
    ]
  }
]
```

## <a name="deploy-a-sample-application-using-service-fabric-azure-files-volume-driver"></a>Déployer un exemple d'application à l’aide du pilote de volume Azure Files Service Fabric

### <a name="using-azure-resource-manager-via-the-provided-powershell-script-recommended"></a>Utilisation de Azure Resource Manager via le script PowerShell fourni (recommandé)

Si votre cluster est basé sur Azure, nous vous recommandons de déployer des applications sur celui-ci à l’aide du modèle de ressource d’application Azure Resource Manager pour en faciliter l’utilisation et pour vous aider à passer au modèle de maintenance d’infrastructure en tant que code. Cette approche élimine la nécessité d’effectuer le suivi de la version d’application pour le pilote de volume Azure Files. Elle vous permet également de maintenir des modèles Azure Resource Manager distincts pour chaque système d’exploitation pris en charge. Le script suppose que vous déployez la version la plus récente de l’application Azure Files, et accepte des paramètres pour le type de système d’exploitation, l’ID d’abonnement de cluster et le groupe de ressources. Vous pouvez télécharger le script à partir du [site de téléchargement de Service Fabric](https://sfazfilevd.blob.core.windows.net/sfazfilevd/DeployAzureFilesVolumeDriver.zip). Notez que celui-ci définit automatiquement sur 19100 le ListenPort qui est le port sur lequel le plug-in de volume Azure Files écoute les requêtes du démon Docker. Vous pouvez modifier cela en ajoutant un paramètre nommé « listenPort ». Veillez à ce que le port ne soit pas en conflit avec un autre port utilisé par le cluster ou vos applications.
 

Commande de déploiement d’Azure Resource Manager pour Windows :
```powershell
.\DeployAzureFilesVolumeDriver.ps1 -subscriptionId [subscriptionId] -resourceGroupName [resourceGroupName] -clusterName [clusterName] -windows
```

Commande de déploiement d’Azure Resource Manager pour Linux :
```powershell
.\DeployAzureFilesVolumeDriver.ps1 -subscriptionId [subscriptionId] -resourceGroupName [resourceGroupName] -clusterName [clusterName] -linux
```

Une fois que vous avez correctement exécuté le script, vous pouvez passer à la [section de configuration de votre application](#configure-your-applications-to-use-the-volume).


### <a name="manual-deployment-for-standalone-clusters"></a>Déploiement manuel de clusters autonomes

Vous pouvez télécharger l’application Service Fabric qui fournit les volumes pour vos conteneurs à partir du [site de téléchargement de Service Fabric](https://sfazfilevd.blob.core.windows.net/sfazfilevd/AzureFilesVolumePlugin.6.5.661.9590.zip). L’application peut être déployée sur le cluster via [PowerShell](./service-fabric-deploy-remove-applications.md), [CLI](./service-fabric-application-lifecycle-sfctl.md) ou des [API FabricClient](./service-fabric-deploy-remove-applications-fabricclient.md).

1. À l’aide de la ligne de commande, basculez vers le répertoire racine du package d’application téléchargé.

    ```powershell
    cd .\AzureFilesVolume\
    ```

    ```bash
    cd ~/AzureFilesVolume
    ```

2. Ensuite, copiez le package d’application vers le magasin d’images avec les valeurs appropriées pour [ApplicationPackagePath] et [ImageStoreConnectionString] :

    ```powershell
    Copy-ServiceFabricApplicationPackage -ApplicationPackagePath [ApplicationPackagePath] -ImageStoreConnectionString [ImageStoreConnectionString] -ApplicationPackagePathInImageStore AzureFilesVolumePlugin
    ```

    ```bash
    sfctl cluster select --endpoint https://testcluster.westus.cloudapp.azure.com:19080 --pem test.pem --no-verify
    sfctl application upload --path [ApplicationPackagePath] --show-progress
    ```

3. Enregistrer le type d’application

    ```powershell
    Register-ServiceFabricApplicationType -ApplicationPathInImageStore AzureFilesVolumePlugin
    ```

    ```bash
    sfctl application provision --application-type-build-path [ApplicationPackagePath]
    ```

4. Créez l’application, en étant très attentif à la valeur du paramètre d’application **ListenPort**. Cette valeur spécifie le port sur lequel le plug-in de volume Azure Files écoute les requêtes du démon Docker. Veillez à ce que le port fourni pour l’application corresponde à VolumePluginPorts dans le manifeste de cluster, et qu’il ne soit pas en conflit avec un autre port utilisé par le cluster ou vos applications.

    ```powershell
    New-ServiceFabricApplication -ApplicationName fabric:/AzureFilesVolumePluginApp -ApplicationTypeName AzureFilesVolumePluginType -ApplicationTypeVersion 6.5.661.9590   -ApplicationParameter @{ListenPort='19100'}
    ```

    ```bash
    sfctl application create --app-name fabric:/AzureFilesVolumePluginApp --app-type AzureFilesVolumePluginType --app-version 6.5.661.9590  --parameter '{"ListenPort":"19100"}'
    ```

> [!NOTE]
> 
> Windows Server 2016 Datacenter ne prend pas en charge le mappage de montages SMB à des conteneurs ([cette fonctionnalité est uniquement prise en charge sur Windows Server version 1709](/virtualization/windowscontainers/manage-containers/container-storage)). Cette contrainte empêche le mappage de volume réseau et les pilotes de volume Azure Files sur les versions antérieures à 1709.

#### <a name="deploy-the-application-on-a-local-development-cluster"></a>Déployer l’application sur un cluster de développement local
Suivez les étapes 1-3 [ci-dessus](#manual-deployment-for-standalone-clusters).

 Le nombre d’instances de service par défaut pour l’application de plug-in de volume Azure Files est -1, ce qui signifie qu’il y a une instance du service déployée sur chaque nœud du cluster. Toutefois, lorsque vous déployez l’application de plug-in de volume Azure Files sur un cluster de développement local, le nombre d’instances de service doit être configuré sur 1. Cela est possible avec le paramètre d’application **InstanceCount**. Par conséquent, la commande permettant de créer l’application de plug-in de volume Azure Files sur un cluster de développement local est :

```powershell
New-ServiceFabricApplication -ApplicationName fabric:/AzureFilesVolumePluginApp -ApplicationTypeName AzureFilesVolumePluginType -ApplicationTypeVersion 6.5.661.9590  -ApplicationParameter @{ListenPort='19100';InstanceCount='1'}
```

```bash
sfctl application create --app-name fabric:/AzureFilesVolumePluginApp --app-type AzureFilesVolumePluginType --app-version 6.5.661.9590  --parameter '{"ListenPort": "19100","InstanceCount": "1"}'
```

## <a name="configure-your-applications-to-use-the-volume"></a>Configurer vos applications pour utiliser le volume
L’extrait de code suivant montre comment spécifier un volume basé sur Azure Files dans le fichier manifeste de l’application de votre application. L’élément spécifique d’intérêt est la balise **Volume** :

```xml
?xml version="1.0" encoding="UTF-8"?>
<ApplicationManifest ApplicationTypeName="WinNodeJsApp" ApplicationTypeVersion="1.0" xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
    <Description>Calculator Application</Description>
    <Parameters>
      <Parameter Name="ServiceInstanceCount" DefaultValue="3"></Parameter>
      <Parameter Name="MyCpuShares" DefaultValue="3"></Parameter>
      <Parameter Name="MyStorageVar" DefaultValue="c:\tmp"></Parameter>
    </Parameters>
    <ServiceManifestImport>
        <ServiceManifestRef ServiceManifestName="NodeServicePackage" ServiceManifestVersion="1.0"/>
     <Policies>
       <ContainerHostPolicies CodePackageRef="NodeService.Code" Isolation="hyperv">
            <PortBinding ContainerPort="8905" EndpointRef="Endpoint1"/>
            <RepositoryCredentials PasswordEncrypted="false" Password="****" AccountName="test"/>
            <Volume Source="azfiles" Destination="c:\VolumeTest\Data" Driver="sfazurefile">
                <DriverOption Name="shareName" Value="" />
                <DriverOption Name="storageAccountName" Value="" />
                <DriverOption Name="storageAccountKey" Value="" />
                <DriverOption Name="storageAccountFQDN" Value="" />
            </Volume>
       </ContainerHostPolicies>
   </Policies>
    </ServiceManifestImport>
    <ServiceTemplates>
        <StatelessService ServiceTypeName="StatelessNodeService" InstanceCount="5">
            <SingletonPartition></SingletonPartition>
        </StatelessService>
    </ServiceTemplates>
</ApplicationManifest>
```

Le nom du pilote du plug-in de volume Azure Files est **sfazurefile**. Cette valeur est définie pour l’attribut **Pilote** de l’élément de balise **Volume** dans le manifeste de l’application.

Dans la balise **Volume** dans l’extrait de code ci-dessus, le plug-in de volume Azure Files requiert les attributs suivantes :
- **Source** : il s’agit du nom du volume. L’utilisateur peut choisir n’importe quel nom pour son volume.
- **Destination** : cet attribut correspond à l’emplacement auquel est mappé le volume dans le conteneur en cours d’exécution. Ainsi, la destination ne peut pas être un emplacement qui existe déjà dans votre conteneur.

Comme indiqué dans les éléments **DriverOption** de l’extrait de code ci-dessus, le plug-in de volume Azure Files prend en charge les options de pilote suivantes :
- **shareName** - Nom du partage de fichiers Azure Files qui fournit le volume au conteneur.
- **storageAccountName** - Nom du compte de stockage Azure qui contient le partage de fichiers Azure Files.
- **storageAccountKey** - Clé d’accès du compte de stockage Azure qui contient le partage de fichiers Azure Files.
- **storageAccountFQDN** - Nom du domaine associé au compte de stockage. Si storageAccountFQDN n’est pas spécifié, le nom du domaine est composé du nom du compte de stockage (storageAccountName) et du suffixe par défaut (.file.core.windows.net).  

    ```xml
    - Example1: 
        <DriverOption Name="shareName" Value="myshare1" />
        <DriverOption Name="storageAccountName" Value="myaccount1" />
        <DriverOption Name="storageAccountKey" Value="mykey1" />
        <!-- storageAccountFQDN will be "myaccount1.file.core.windows.net" -->
    - Example2: 
        <DriverOption Name="shareName" Value="myshare2" />
        <DriverOption Name="storageAccountName" Value="myaccount2" />
        <DriverOption Name="storageAccountKey" Value="mykey2" />
        <DriverOption Name="storageAccountFQDN" Value="myaccount2.file.core.chinacloudapi.cn" />
    ```

## <a name="using-your-own-volume-or-logging-driver"></a>À l’aide de votre propre pilote de volume ou de journalisation
Service Fabric permet également l’utilisation de vos propres pilotes de [volume](https://docs.docker.com/engine/extend/plugins_volume/) ou de [journalisation](https://docs.docker.com/engine/admin/logging/overview/) personnalisés. Si le pilote de volume/journalisation Docker n’est pas installé sur le cluster, vous pouvez l’installer manuellement à l’aide des protocoles RDP/SSH. Vous pouvez effectuer l’installation avec ces protocoles par le biais d’un [script de démarrage de groupe de machines virtuelles identiques](https://azure.microsoft.com/resources/templates/201-vmss-custom-script-windows/) ou d’un [script SetupEntryPoint](./service-fabric-application-model.md).

Voici un exemple de script permettant d’installer le [pilote de volume Docker pour Azure](https://docs.docker.com/docker-for-azure/persistent-data-volumes/) :

```bash
docker plugin install --alias azure --grant-all-permissions docker4x/cloudstor:17.09.0-ce-azure1  \
    CLOUD_PLATFORM=AZURE \
    AZURE_STORAGE_ACCOUNT="[MY-STORAGE-ACCOUNT-NAME]" \
    AZURE_STORAGE_ACCOUNT_KEY="[MY-STORAGE-ACCOUNT-KEY]" \
    DEBUG=1
```

Dans vos applications, pour utiliser le pilote de volume ou de journalisation que vous avez installé, vous devez spécifier les valeurs appropriées dans les éléments **Volume** et **LogConfig** sous **ContainerHostPolicies** dans votre manifeste de l’application.

```xml
<ContainerHostPolicies CodePackageRef="NodeService.Code" Isolation="hyperv">
    <PortBinding ContainerPort="8905" EndpointRef="Endpoint1"/>
    <RepositoryCredentials PasswordEncrypted="false" Password="****" AccountName="test"/>
    <LogConfig Driver="[YOUR_LOG_DRIVER]" >
        <DriverOption Name="test" Value="vale"/>
    </LogConfig>
    <Volume Source="c:\workspace" Destination="c:\testmountlocation1" IsReadOnly="false"></Volume>
    <Volume Source="[MyStorageVar]" Destination="c:\testmountlocation2" IsReadOnly="true"> </Volume>
    <Volume Source="myvolume1" Destination="c:\testmountlocation2" Driver="[YOUR_VOLUME_DRIVER]" IsReadOnly="true">
        <DriverOption Name="[name]" Value="[value]"/>
    </Volume>
</ContainerHostPolicies>
```

Quand vous spécifiez un plug-in de volume, Service Fabric crée automatiquement le volume à l’aide des paramètres spécifiés. La balise **Source** pour l’élément **Volume** est le nom du volume, et la balise **Driver** spécifie le plug-in de pilote de volume. La balise **Destination** correspond à l’emplacement auquel est mappé la **Source** dans le conteneur en cours d’exécution. Ainsi, la destination ne peut pas être un emplacement qui existe déjà dans votre conteneur. Vous pouvez spécifier les options à l’aide de la balise **DriverOption** comme suit :

```xml
<Volume Source="myvolume1" Destination="c:\testmountlocation4" Driver="azure" IsReadOnly="true">
           <DriverOption Name="share" Value="models"/>
</Volume>
```

Les paramètres d’application sont pris en charge pour les volumes comme indiqué dans l’extrait de code de manifeste précédent (recherchez `MyStorageVar` pour obtenir un exemple d’utilisation).

Si un pilote de journalisation Docker est spécifié, vous devez déployer des agents (ou conteneurs) pour gérer les journaux d’activité dans le cluster. Vous pouvez utiliser la balise **DriverOption** pour spécifier des options pour le pilote de journal.

## <a name="next-steps"></a>Étapes suivantes
* Pour voir des exemples de conteneur, y compris le pilote de volume, consultez les [exemples de conteneur Service Fabric](https://github.com/Azure-Samples/service-fabric-containers).
* Pour déployer des conteneurs sur un cluster Service Fabric, consultez l’article [Déployer un conteneur sur Service Fabric](./service-fabric-get-started-containers.md).

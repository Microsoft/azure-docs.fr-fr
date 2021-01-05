---
title: Utiliser un volume basé sur Azure Files dans une application Service Fabric Mesh
description: Découvrez comment stocker un état dans une application Service Fabric Mesh en montant un volume basé sur Azure Files dans un service avec Azure CLI.
author: georgewallace
ms.topic: conceptual
ms.date: 11/21/2018
ms.author: gwallace
ms.custom: mvc, devcenter , devx-track-azurecli
ms.openlocfilehash: 00addbe992bc113c67df08f73414107d51b3bce5
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96007390"
---
# <a name="mount-an-azure-files-based-volume-in-a-service-fabric-mesh-application"></a>Monter un volume basé sur Azure Files dans une application Service Fabric Mesh 

Cet article décrit comment monter un volume basé sur Azure Files dans un service d’une application Service Fabric Mesh.  Le pilote de volume Azure Files est un pilote de volume Docker utilisé pour monter un partage Azure Files sur un conteneur, auquel vous faites appel pour conserver l’état du service. Les volumes vous offrent un stockage de fichiers universel et vous permettent de lire/écrire des fichiers à l’aide d’API de fichier d’E/S de disque normal.  Pour en savoir plus sur les volumes et les options de stockage des données d’application, consultez [Stockage d’état](service-fabric-mesh-storing-state.md).

Pour monter un volume dans un service, créez une ressource de volume dans votre application Service Fabric Mesh et référencez ce volume dans votre service.  Vous pouvez déclarer la ressource de volume et la référencer dans la ressource de service dans les [fichiers de ressources YAML](#declare-a-volume-resource-and-update-the-service-resource-yaml) ou le [modèle de déploiement basé sur JSON](#declare-a-volume-resource-and-update-the-service-resource-json). Avant de monter le volume, commencez par créer un compte de stockage Azure et un [partage de fichiers dans Azure Files](../storage/files/storage-how-to-create-file-share.md).

## <a name="prerequisites"></a>Prérequis
> [!NOTE]
> **Problème connu du déploiement sur l’ordinateur de développement RS5 Windows :** Il existe un bogue ouvert avec le cmdlet New-SmbGlobalMapping de PowerShell sur les machines Windows RS5 qui empêche le montage d’Azurefile Volumes. Voici un exemple d’erreur qui se produit lorsque le volume AzureFile est monté sur un ordinateur de développement local.
```
Error event: SourceId='System.Hosting', Property='CodePackageActivation:counterService:EntryPoint:131884291000691067'.
There was an error during CodePackage activation.System.Fabric.FabricException (-2147017731)
Failed to start Container. ContainerName=sf-2-63fc668f-362d-4220-873d-85abaaacc83e_6d6879cf-dd43-4092-887d-17d23ed9cc78, ApplicationId=SingleInstance_0_App2, ApplicationName=fabric:/counterApp. DockerRequest returned StatusCode=InternalServerError with ResponseBody={"message":"error while mounting volume '': mount failed"}
```
La solution pour le problème est 1) d’exécuter la commande ci-dessous en tant qu’administrateur PowerShell et 2) de redémarrer l’ordinateur.
```powershell
PS C:\WINDOWS\system32> Mofcomp c:\windows\system32\wbem\smbwmiv2.mof
```

Pour suivre cet article, vous pouvez utiliser Azure Cloud Shell ou une installation locale d’Azure CLI. 

Si vous voulez utiliser Azure CLI localement pour cet article, vérifiez que `az --version` retourne au moins `azure-cli (2.0.43)`.  Installez (ou mettez à jour) le module d’extension Service Fabric Mesh CLI en suivant les [instructions](service-fabric-mesh-howto-setup-cli.md) ci-après.

Pour vous connecter à Azure et définir votre abonnement :

```azurecli
az login
az account set --subscription "<subscriptionID>"
```

## <a name="create-a-storage-account-and-file-share-optional"></a>Créer un compte de stockage et un partage de fichiers (facultatif)
Le montage d’un volume Azure Files nécessite un compte de stockage et un partage de fichiers.  Vous pouvez utiliser un compte de stockage Azure et un partage de fichiers existants, ou créer les ressources :

```azurecli-interactive
az group create --name myResourceGroup --location eastus

az storage account create --name myStorageAccount --resource-group myResourceGroup --location eastus --sku Standard_LRS --kind StorageV2

$current_env_conn_string=$(az storage account show-connection-string -n myStorageAccount -g myResourceGroup --query 'connectionString' -o tsv)

az storage share create --name myshare --quota 2048 --connection-string $current_env_conn_string
```

## <a name="get-the-storage-account-name-and-key-and-the-file-share-name"></a>Obtenir le nom et la clé du compte de stockage et le nom du partage de fichiers
Le nom du compte de stockage, la clé du compte de stockage et le nom du partage de fichiers sont référencés respectivement en tant que `<storageAccountName>`, `<storageAccountKey>` et `<fileShareName>` dans les sections suivantes. 

Listez vos comptes de stockage et prenez le nom du compte de stockage avec le partage de fichiers que vous voulez utiliser :
```azurecli-interactive
az storage account list
```

Prenez le nom du partage de fichiers :
```azurecli-interactive
az storage share list --account-name <storageAccountName>
```

Prenez la clé du compte de stockage ("key1") :
```azurecli-interactive
az storage account keys list --account-name <storageAccountName> --query "[?keyName=='key1'].value"
```

Vous pouvez également trouver ces valeurs dans le [portail Azure](https://portal.azure.com) :
* `<storageAccountName>` - Sous **Comptes de stockage**, nom du compte de stockage utilisé pour créer le partage de fichiers.
* `<storageAccountKey>` - Sélectionnez votre compte de stockage sous **Comptes de stockage**, puis sélectionnez **Clés d’accès** et utilisez la valeur sous **key1**.
* `<fileShareName>` - Sélectionnez votre compte de stockage sous **Comptes de stockage**, puis sélectionnez **Fichiers**. Le nom à utiliser est le nom du partage de fichiers que vous avez créé.

## <a name="declare-a-volume-resource-and-update-the-service-resource-json"></a>Déclarer une ressource de volume et mettre à jour la ressource de service (JSON)

Ajoutez des paramètres pour les valeurs `<fileShareName>`, `<storageAccountName>` et `<storageAccountKey>` que vous avez trouvées à une étape précédente. 

Créez une ressource Volume en tant que pair de la ressource Application. Spécifiez un nom et le fournisseur (« SFAzureFile » pour utiliser le volume basé sur Azure Files). Dans `azureFileParameters`, spécifiez les paramètres pour les valeurs `<fileShareName>`, `<storageAccountName>` et `<storageAccountKey>` que vous avez trouvées à une étape précédente.

Pour monter le volume dans votre service, ajoutez un `volumeRefs` dans l’élément `codePackages` du service.  `name` est l’ID de ressource pour le volume (ou un paramètre de modèle de déploiement pour la ressource de volume) et le nom du volume déclaré dans le fichier de ressources volume.yaml.  `destinationPath` est le répertoire local sur lequel le volume sera monté.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "defaultValue": "EastUS",
      "type": "String",
      "metadata": {
        "description": "Location of the resources."
      }
    },
    "fileShareName": {
      "type": "string",
      "metadata": {
        "description": "Name of the Azure Files file share that provides the volume for the container."
      }
    },
    "storageAccountName": {
      "type": "string",
      "metadata": {
        "description": "Name of the Azure storage account that contains the file share."
      }
    },
    "storageAccountKey": {
      "type": "securestring",
      "metadata": {
        "description": "Access key for the Azure storage account that contains the file share."
      }
    },
    "stateFolderName": {
      "type": "string",
      "defaultValue": "TestVolumeData",
      "metadata": {
        "description": "Folder in which to store the state. Provide an empty value to create a unique folder for each container to store the state. A non-empty value will retain the state across deployments, however if more than one applications are using the same folder, the counter may update more frequently."
      }
    }
  },
  "resources": [
    {
      "apiVersion": "2018-09-01-preview",
      "name": "VolumeTest",
      "type": "Microsoft.ServiceFabricMesh/applications",
      "location": "[parameters('location')]",
      "dependsOn": [
        "Microsoft.ServiceFabricMesh/networks/VolumeTestNetwork",
        "Microsoft.ServiceFabricMesh/volumes/testVolume"
      ],
      "properties": {
        "services": [
          {
            "name": "VolumeTestService",
            "properties": {
              "description": "VolumeTestService description.",
              "osType": "Windows",
              "codePackages": [
                {
                  "name": "VolumeTestService",
                  "image": "volumetestservice:dev",
                  "volumeRefs": [
                    {
                      "name": "[resourceId('Microsoft.ServiceFabricMesh/volumes', 'testVolume')]",
                      "destinationPath": "C:\\app\\data"
                    }
                  ],
                  "environmentVariables": [
                    {
                      "name": "ASPNETCORE_URLS",
                      "value": "http://+:20003"
                    },
                    {
                      "name": "STATE_FOLDER_NAME",
                      "value": "[parameters('stateFolderName')]"
                    }
                  ],
                  ...
                }
              ],
              ...
            }
          }
        ],
        "description": "VolumeTest description."
      }
    },
    {
      "apiVersion": "2018-09-01-preview",
      "name": "testVolume",
      "type": "Microsoft.ServiceFabricMesh/volumes",
      "location": "[parameters('location')]",
      "dependsOn": [],
      "properties": {
        "description": "Azure Files storage volume for the test application.",
        "provider": "SFAzureFile",
        "azureFileParameters": {
          "shareName": "[parameters('fileShareName')]",
          "accountName": "[parameters('storageAccountName')]",
          "accountKey": "[parameters('storageAccountKey')]"
        }
      }
    }
    ...
  ]
}
```

## <a name="declare-a-volume-resource-and-update-the-service-resource-yaml"></a>Déclarer une ressource de volume et mettre à jour la ressource de service (YAML)

Ajoutez un nouveau fichier *volume.yaml* dans le répertoire *Ressources de l’application* de votre application.  Spécifiez un nom et le fournisseur (« SFAzureFile » pour utiliser le volume basé sur Azure Files). `<fileShareName>`, `<storageAccountName>` et `<storageAccountKey>` sont les valeurs que vous avez trouvées à une étape précédente.

```yaml
volume:
  schemaVersion: 1.0.0-preview2
  name: testVolume
  properties:
    description: Azure Files storage volume for counter App.
    provider: SFAzureFile
    azureFileParameters: 
        shareName: <fileShareName>
        accountName: <storageAccountName>
        accountKey: <storageAccountKey>
```

Mettez à jour le fichier *service.yaml* dans le répertoire *Resources du service* pour monter le volume dans votre service.  Ajoutez l’élément `volumeRefs` dans l’élément `codePackages`.  `name` est l’ID de ressource pour le volume (ou un paramètre de modèle de déploiement pour la ressource de volume) et le nom du volume déclaré dans le fichier de ressources volume.yaml.  `destinationPath` est le répertoire local sur lequel le volume sera monté.

```yaml
## Service definition ##
application:
  schemaVersion: 1.0.0-preview2
  name: VolumeTest
  properties:
    services:
      - name: VolumeTestService
        properties:
          description: VolumeTestService description.
          osType: Windows
          codePackages:
            - name: VolumeTestService
              image: volumetestservice:dev
              volumeRefs:
                - name: "[resourceId('Microsoft.ServiceFabricMesh/volumes', 'testVolume')]"
                  destinationPath: C:\app\data
              endpoints:
                - name: VolumeTestServiceListener
                  port: 20003
              environmentVariables:
                - name: ASPNETCORE_URLS
                  value: http://+:20003
                - name: STATE_FOLDER_NAME
                  value: TestVolumeData
              resources:
                requests:
                  cpu: 0.5
                  memoryInGB: 1
          replicaCount: 1
          networkRefs:
            - name: VolumeTestNetwork
```

## <a name="next-steps"></a>Étapes suivantes

- Affichez l’exemple d’application de volume Azure Files sur [GitHub](https://github.com/Azure-Samples/service-fabric-mesh/tree/master/src/counter).
- Pour en savoir plus sur le modèle de ressource Service Fabric, voir [Modèle de ressource Service Fabric mesh](service-fabric-mesh-service-fabric-resources.md).
- Pour en savoir plus sur Service Fabric mesh, voir la [vue d’ensemble de Service Fabric mesh](service-fabric-mesh-overview.md).

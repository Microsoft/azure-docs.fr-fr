---
title: Exemples de scripts PowerShell
description: Exemples qui montrent comment utiliser le serveur front-end par le biais de scripts PowerShell
author: florianborn71
ms.author: flborn
ms.date: 02/12/2020
ms.topic: sample
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: cb8cc98a020cb382a6941c1e410eab4543594629
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96009753"
---
# <a name="example-powershell-scripts"></a>Exemples de scripts PowerShell

Azure Remote Rendering fournit les deux API REST suivantes :

- L’[API REST Conversion](../how-tos/conversion/conversion-rest-api.md)
- L’[API REST Session](../how-tos/session-rest-api.md)

Le [dépôt d’exemples ARR](https://github.com/Azure/azure-remote-rendering) contient des exemples de scripts situés dans le dossier *Scripts* qui permettent d’interagir avec les API REST du service. Cet article décrit leur utilisation.

> [!TIP]
> Il existe également un [outil basé sur l’interface utilisateur appelé ARRT](azure-remote-rendering-asset-tool.md), qui permet d’interagir avec le service, ce qui constitue une alternative pratique à l’utilisation de scripts. ![ARRT](./media/azure-remote-rendering-asset-tool.png "Capture d’écran ARRT")

> [!CAUTION]
> Une fréquence trop élevée d’appels aux fonctions de l’API REST provoque la limitation du serveur, qui retourne une défaillance. Dans ce cas, l’ID de code d’échec HTTP est 429 (« trop de demandes »). En règle générale, il doit y avoir un délai de **5 à 10 secondes entre les appels successifs**.

## <a name="prerequisites"></a>Prérequis

Pour exécuter les exemples de scripts, vous avez besoin d’une configuration fonctionnelle d’[Azure PowerShell](/powershell/azure/).

1. Installez Azure PowerShell :
    1. Ouvrez une fenêtre PowerShell avec des droits d’administrateur.
    1. Exécutez l’instruction suivante : `Install-Module -Name Az -AllowClobber`

1. Si vous recevez des erreurs concernant l’exécution des scripts, vérifiez que votre [stratégie d’exécution](/powershell/module/microsoft.powershell.core/about/about_execution_policies?view=powershell-6) est correctement configurée :
    1. Ouvrez une fenêtre PowerShell avec des droits d’administrateur.
    1. Exécutez l’instruction suivante : `Set-ExecutionPolicy -ExecutionPolicy Unrestricted`

1. [Préparer un compte de stockage Azure](../how-tos/conversion/blob-storage.md#prepare-azure-storage-accounts)

1. Connectez-vous à l’abonnement qui contient votre compte Azure Remote Rendering :
    1. Ouvrez une fenêtre PowerShell.
    1. Exécutez `Connect-AzAccount`, puis suivez les instructions à l’écran.

    > [!NOTE]
    > Si votre organisation a plusieurs abonnements, vous devrez peut-être spécifier les arguments SubscriptionId et Tenant. Pour plus d’informations, consultez la [documentation sur Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount).

1. Téléchargez le dossier *Scripts* à partir du [dépôt GitHub Azure Remote Rendering](https://github.com/Azure/azure-remote-rendering).

## <a name="configuration-file"></a>Fichier de configuration

À côté des fichiers `.ps1`, vous devez renseigner `arrconfig.json` :

```json
{
    "accountSettings": {
        "arrAccountId": "<fill in the account ID from the Azure Portal>",
        "arrAccountKey": "<fill in the account key from the Azure Portal>",
        "region": "<select from available regions>"
    },
    "renderingSessionSettings": {
        "vmSize": "<select standard or premium>",
        "maxLeaseTime": "<hh:mm:ss>"
    },
  "assetConversionSettings": {
    "resourceGroup": "<resource group which contains the storage account you created, only needed when uploading or generating SAS>",
    "storageAccountName": "<name of the storage account you created>",
    "blobInputContainerName": "<input container inside the storage container>",
    "blobOutputContainerName": "<output container inside the storage container>",
    "localAssetDirectoryPath": "<fill in a path to a local directory containing your asset (and files referenced from it like textures)>",
    "inputFolderPath": "<optional: base folderpath in the input container for asset upload. uses / as dir separator>",
    "inputAssetPath": "<the path to the asset under inputcontainer/inputfolderpath pointing to the input asset e.g. box.fbx>",
    "outputFolderPath": "<optional: base folderpath in the output container - the converted asset and log files will be placed here>",
    "outputAssetFileName": "<optional: filename for the converted asset, this will be placed in the output container under the outputpath>"
  }
}
```

> [!CAUTION]
> Veillez à placer correctement les barres obliques inverses dans le chemin LocalAssetDirectoryPath en utilisant des barres obliques inverses doubles (« \\\\ »). Vous pouvez utiliser des barres obliques (« / ») dans tous les autres chemins tels que inputFolderPath et inputAssetPath.

> [!CAUTION]
> Les valeurs facultatives doivent être renseignées ; sinon, vous devez supprimer entièrement la clé et la valeur. Par exemple, si vous n’utilisez pas le paramètre `"outputAssetFileName"`, supprimez toute la ligne dans `arrconfig.json`.

### <a name="accountsettings"></a>accountSettings

Pour `arrAccountId` et `arrAccountKey`, consultez [Créer un compte Azure Remote Rendering](../how-tos/create-an-account.md).
Pour `region`, consultez la [liste des régions disponibles](../reference/regions.md).

### <a name="renderingsessionsettings"></a>renderingSessionSettings

Cette structure doit être remplie si vous souhaitez exécuter **RenderingSession.ps1** :

- **vmSize :** sélectionnez la taille de la machine virtuelle. Sélectionnez [*standard*](../reference/vm-sizes.md) ou [*premium*](../reference/vm-sizes.md). Arrêtez les sessions de rendu lorsque vous n’en avez plus besoin.
- **maxLeaseTime :** durée souhaitée du bail pour la machine virtuelle. Celle-ci sera arrêtée lorsque le bail expirera. La durée du bail peut être prolongée (voir ci-dessous).

### <a name="assetconversionsettings"></a>assetConversionSettings

Cette structure doit être remplie si vous souhaitez exécuter **Conversion.ps1**.

Pour plus d’informations, consultez [Préparer un compte de stockage Azure](../how-tos/conversion/blob-storage.md#prepare-azure-storage-accounts).

## <a name="script-renderingsessionps1"></a>Script : RenderingSession.ps1

Ce script est utilisé pour créer, interroger et arrêter les sessions de rendu.

> [!IMPORTANT]
> Vérifiez que vous avez rempli les sections *accountSettings* et *renderingSessionSettings* du fichier arrconfig.json.

### <a name="create-a-rendering-session"></a>Créer une session de rendu

Utilisation normale avec un fichier arrconfig.json entièrement rempli :

```PowerShell
.\RenderingSession.ps1
```

Le script appellera l’[API REST de gestion des sessions](../how-tos/session-rest-api.md) pour lancer une machine virtuelle de rendu avec les paramètres spécifiés. Si l’opération réussit, il récupérera le *sessionId*. Il interrogera ensuite les propriétés de la session jusqu’à ce que la session soit prête ou qu’une erreur se produise.

Pour utiliser un **autre fichier de configuration** :

```PowerShell
.\RenderingSession.ps1 -ConfigFile D:\arr\myotherconfigFile.json
```

Vous pouvez **remplacer certains paramètres** dans le fichier de configuration :

```PowerShell
.\RenderingSession.ps1 -Region <region> -VmSize <vmsize> -MaxLeaseTime <hh:mm:ss>
```

Si vous souhaitez **démarrer une session sans l’interroger**, vous pouvez utiliser ceci :

```PowerShell
.\RenderingSession.ps1 -CreateSession
```

Le *sessionId* que le script récupère doit être passé à la plupart des autres commandes de session.

### <a name="retrieve-session-properties"></a>Récupérer les propriétés de session

Pour obtenir les propriétés d’une session, exécutez ceci :

```PowerShell
.\RenderingSession.ps1 -GetSessionProperties -Id <sessionID> [-Poll]
```

Utilisez `-Poll` pour attendre que la session soit *prête* ou qu’une erreur se produise.

### <a name="list-active-sessions"></a>Lister les sessions actives

```PowerShell
.\RenderingSession.ps1 -GetSessions
```

### <a name="stop-a-session"></a>Arrêter une session

```PowerShell
.\RenderingSession.ps1 -StopSession -Id <sessionID>
```

### <a name="change-session-properties"></a>Modifier les propriétés de session

Pour le moment, seule la modification du maxLeaseTime d’une session est prise en charge.

> [!NOTE]
> La durée du bail démarre toujours à partir de l’heure de création de la machine virtuelle de la session. Par conséquent, pour prolonger d’une heure le bail de la session, augmentez *maxLeaseTime* d’une heure.

```PowerShell
.\RenderingSession.ps1 -UpdateSession -Id <sessionID> -MaxLeaseTime <hh:mm:ss>
```

## <a name="script-conversionps1"></a>Script : Conversion.ps1

Ce script est utilisé pour convertir les modèles d’entrée au format d’exécution propre à Azure Remote Rendering.

> [!IMPORTANT]
> Vérifiez que vous avez rempli les sections *accountSettings* et *assetConversionSettings* du fichier arrconfig.json.

Le script montre les deux utilisations possibles des comptes de stockage avec le service :

- Associer le compte de stockage au compte Azure Remote Rendering
- Fournir l’accès au stockage via des signatures d’accès partagé (SAS)

### <a name="linked-storage-account"></a>Compte de stockage lié

Une fois que vous avez entièrement rempli le fichier arrconfig.json et que vous l’avez lié à un compte de stockage, vous pouvez utiliser la commande suivante. La procédure à suivre pour lier votre compte de stockage est décrite dans [Créer un compte](../how-tos/create-an-account.md#link-storage-accounts).

L’utilisation d’un compte de stockage lié est la meilleure façon d’utiliser le service de conversion, puisqu’il n’est pas nécessaire de générer des signatures d’accès partagé.

```PowerShell
.\Conversion.ps1
```

1. Chargez tous les fichiers de `assetConversionSettings.modelLocation` dans le conteneur d’objets blob d’entrée sous le `inputFolderPath` donné.
1. Appelez l’[API REST de conversion des modèles](../how-tos/conversion/conversion-rest-api.md) pour lancer la [conversion du modèle](../how-tos/conversion/model-conversion.md)
1. Interrogez l’état de la conversion jusqu’à la réussite ou l’échec de celle-ci.
1. Affichez les informations concernant l’emplacement du fichier converti (compte de stockage, conteneur de sortie, chemin du fichier dans le conteneur).

### <a name="access-to-storage-via-shared-access-signatures"></a>Accéder au stockage via des signatures d’accès partagé

```PowerShell
.\Conversion.ps1 -UseContainerSas
```

Avec cette opération, vous pouvez :

1. Charger le fichier local à partir de `assetConversionSettings.localAssetDirectoryPath` dans le conteneur d’objets blob d’entrée.
1. Générer un URI SAS pour le conteneur d’entrée.
1. Générer un URI SAS pour le conteneur de sortie.
1. Appeler l’[API REST de conversion des modèles](../how-tos/conversion/conversion-rest-api.md) pour lancer la [conversion du modèle](../how-tos/conversion/model-conversion.md).
1. Interroger l’état de la conversion jusqu’à la réussite ou l’échec de celle-ci.
1. Afficher les informations concernant l’emplacement du fichier converti (compte de stockage, conteneur de sortie, chemin du fichier dans le conteneur).
1. Afficher l’URI SAS du modèle converti dans le conteneur d’objets blob de sortie.

### <a name="additional-command-line-options"></a>Options supplémentaires de la ligne de commande

Pour utiliser un **autre fichier de configuration** :

```PowerShell
.\Conversion.ps1 -ConfigFile D:\arr\myotherconfigFile.json
```

Si vous souhaitez **démarrer une conversion de modèle sans l’interroger**, vous pouvez utiliser ceci :

```PowerShell
.\Conversion.ps1 -ConvertAsset
```

Vous pouvez **remplacer certains paramètres** du fichier de configuration à l’aide des commutateurs de ligne de commande suivants :

* **Id :** ConversionId utilisé avec GetConversionStatus
* **ArrAccountId :** arrAccountId de accountSettings
* **ArrAccountKey :** remplacement pour l’arrAccountKey de accountSettings
* **Region :** remplacement pour la région de accountSettings
* **ResourceGroup :** remplacement pour le resourceGroup de assetConversionSettings
* **StorageAccountName :** remplacement pour le storageAccountName de assetConversionSettings
* **BlobInputContainerName :** remplacement pour le blobInputContainer de assetConversionSettings
* **LocalAssetDirectoryPath :** remplacement pour le localAssetDirectoryPath de assetConversionSettings
* **InputAssetPath :** remplacement pour l’inputAssetPath de assetConversionSettings
* **BlobOutputContainerName :** remplacement pour le blobOutputContainerName de assetConversionSettings
* **OutputFolderPath :** remplacement pour le outputFolderPath de assetConversionSettings
* **OutputAssetFileName :** remplacement pour le outputAssetFileName de assetConversionSettings

Par exemple, vous pouvez combiner plusieurs options de la façon suivante :

```PowerShell
.\Conversion.ps1 -LocalAssetDirectoryPath "C:\\models\\box" -InputAssetPath box.fbx -OutputFolderPath another/converted/box -OutputAssetFileName newConversionBox.arrAsset
```

### <a name="run-the-individual-conversion-stages"></a>Exécuter les étapes de conversion séparément

Si vous souhaitez exécuter uniquement certaines étapes du processus, vous pouvez procéder de la façon suivante :

Chargez uniquement les données à partir du LocalAssetDirectoryPath donné.

```PowerShell
.\Conversion.ps1 -Upload
```

Démarrez uniquement le processus de conversion d’un modèle déjà chargé dans le stockage Blob (sans exécuter l’option de chargement ni interroger l’état de la conversion). Le script retournera un *conversionId*.

```PowerShell
.\Conversion.ps1 -ConvertAsset
```

Vous pouvez récupérer l’état de la conversion à l’aide de ceci :

```PowerShell
.\Conversion.ps1 -GetConversionStatus -Id <conversionId> [-Poll]
```

Utilisez `-Poll` pour attendre que la conversion soit terminée ou qu’une erreur se produise.

## <a name="next-steps"></a>Étapes suivantes

- [Démarrage rapide : Afficher un modèle avec Unity](../quickstarts/render-model.md)
- [Démarrage rapide : Convertir un modèle pour le rendu](../quickstarts/convert-model.md)
- [Conversion de modèle](../how-tos/conversion/model-conversion.md)
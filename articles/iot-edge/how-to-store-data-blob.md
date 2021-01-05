---
title: Stocker des objets blob de blocs sur des appareils - Azure IoT Edge | Microsoft Docs
description: Comprendre les fonctionnalités de hiérarchisation et durée de vie, consulter les opérations de stockage blob prises en charge et se connecter à votre compte de stockage d’objets blob.
author: kgremban
ms.author: kgremban
ms.reviewer: arduppal
ms.date: 12/13/2019
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 22cef5919e597d4cd83ad80f5758a0427c52e2bb
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92219732"
---
# <a name="store-data-at-the-edge-with-azure-blob-storage-on-iot-edge"></a>Stocker des données en périphérie avec le Stockage Blob Azure sur IoT Edge

Stockage Blob Azure sur IoT Edge fournit une solution de stockage d’[objets blob de blocs](/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs#about-block-blobs) et d’[objets blob d’ajout](/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs#about-append-blobs) en périphérie. Un module de stockage de blobs sur votre appareil IoT Edge se comporte comme un service blob Azure, à ceci près que les blobs sont stockés localement sur votre appareil IoT Edge. Vous pouvez accéder à vos blobs avec les mêmes méthodes que celles du SDK Stockage Azure, ou aux appels d’API blob auxquels vous êtes déjà habitué. Cet article explique les concepts relatifs au Stockage Blob Azure sur le conteneur IoT Edge qui exécute un service d’objets blob sur votre appareil IoT Edge.

Ce module est utile dans les scénarios suivants :

* où les données doivent être stockées localement jusqu’à ce qu’elles puissent être traitées ou transférées vers le cloud. Ces données peuvent être des vidéos, des images, des données financières, des données médicales ou toute autre donnée non structurée.
* lorsque les appareils se trouvent dans un emplacement où la connectivité est limitée.
* lorsque vous souhaitez traiter efficacement les données en local pour obtenir une faible latence d’accès aux données, afin de pouvoir réagir à des situations d’urgence le plus rapidement possible.
* lorsque vous souhaitez réduire les coûts de bande passante et éviter de transférer des téraoctets de données vers le cloud. Vous pouvez traiter les données en local et envoyez uniquement les données traitées vers le cloud.

Regardez la vidéo de présentation rapide
> [!VIDEO https://www.youtube.com/embed/xbwgMNGB_3Y]

Ce module est fourni avec les fonctionnalités **deviceToCloudUpload** et **deviceAutoDelete** .

**deviceToCloudUpload** est une fonctionnalité configurable. Cette fonction vous permet de charger automatiquement les données à partir de votre stockage blob local vers Azure avec prise en charge de connectivité internet intermittente. Vous pouvez ainsi :

* Activer/désactiver la fonctionnalité deviceToCloudUpload.
* Choisissez l’ordre dans lequel les données sont copiées vers Azure comme NewestFirst ou OldestFirst.
* Spécifiez le compte de stockage Azure vers lequel vous souhaitez que vos données soient téléchargées.
* Spécifiez les conteneurs que vous voulez télécharger dans Azure. Ce module vous permet de spécifier des noms de conteneur source et cible.
* Choisissez la possibilité de supprimer les objets blob immédiatement, après le téléchargement dans le stockage cloud.
* Effectuez le chargement de blob complet (à l’aide de l’opération `Put Blob`) et le chargement de niveau bloc (à l’aide des opérations `Put Block`, `Put Block List` et `Append Block`).

Ce module utilise le téléchargement de niveau bloc, lorsque votre objet blob est constitué de blocs. Voici quelques scénarios courants :

* Votre application met à jour des blocs d’un objet blob de blocs précédemment chargé ou ajoute de nouveaux blocs à un blob d’ajout, ce module charge uniquement les blocs mis à jour et pas le blob entier.
* Si le module télécharge un objet blob lorsque vous perdez la connexion Internet, il ne téléchargera que les blocs restants et pas l’objet blob entier lorsque vous serez de nouveau connecté.

Si un arrêt inattendu du processus (par exemple, une panne de courant) se produit pendant le téléchargement d’un objet blob, tous les blocs qui devaient être téléchargés seront téléchargés à nouveau, lorsque le module repassera en ligne.

**deviceAutoDelete** est une fonctionnalité configurable. Cette fonction supprime automatiquement vos objets blob du stockage local lorsque la durée spécifiée (exprimée en minutes) expire. Vous pouvez ainsi :

* Activer/désactiver la fonctionnalité deviceAutoDelete.
* Spécifiez la durée en minutes (deleteAfterMinutes) après laquelle les objets blob sont supprimés automatiquement.
* Choisissez la possibilité de conserver l’objet blob pendant son chargement en cas d’expiration de la valeur deleteAfterMinutes.

## <a name="prerequisites"></a>Prérequis

Un appareil Azure IoT Edge :

* Vous pouvez utiliser votre ordinateur de développement ou une machine virtuelle comme un appareil IoT Edge, en suivant les étapes décrites dans le Guide de démarrage rapide pour [Linux](quickstart-linux.md) ou pour les [Appareils Windows](quickstart.md).

* Reportez-vous à [Systèmes Azure IoT Edge pris en charge](support.md#operating-systems) pour obtenir une liste des systèmes d’exploitation et architectures pris en charge. Le module de Stockage Blob Azure sur IoT Edge prend en charge les architectures suivantes :
  * Windows AMD64
  * Linux AMD64
  * Linux ARM32
  * Linux ARM64 (préversion)

Ressources cloud :

Un niveau standard [IoT Hub](../iot-hub/iot-hub-create-through-portal.md) dans Azure.

## <a name="devicetocloudupload-and-deviceautodelete-properties"></a>Propriétés deviceToCloudUpload et deviceAutoDelete

Utilisez les propriétés souhaitées du module pour définir les propriétés **deviceToCloudUploadProperties** et **deviceAutoDeleteProperties** . Les propriétés souhaitées peuvent être définies lors du déploiement ou modifiées ultérieurement en modifiant le jumeau de module sans avoir besoin de le redéployer. Nous vous recommandons de vérifier le « jumeau de module » et ses valeurs `reported configuration` et `configurationValidation` pour vous assurer qu’elles ont été correctement propagées.

### <a name="devicetoclouduploadproperties"></a>deviceToCloudUploadProperties

Le nom de ce paramètre est `deviceToCloudUploadProperties`. Si vous utilisez le simulateur IoT Edge, définissez les valeurs sur les variables d’environnement associées pour ces propriétés, que vous trouverez dans la section Explication.

| Propriété | Valeurs possibles | Explication |
| ----- | ----- | ---- |
| uploadOn | true, false | Définissez cette valeur sur `false` par défaut. Si vous souhaitez activer cette fonctionnalité, définissez ce champ sur `true`. <br><br> Variable d’environnement : `deviceToCloudUploadProperties__uploadOn={false,true}` |
| uploadOrder | NewestFirst, OldestFirst | Vous permet de choisir l’ordre dans lequel les données sont copiées vers Azure. Définissez cette valeur sur `OldestFirst` par défaut. L’ordre est déterminé par l’heure de dernière modification de l’objet blob. <br><br> Variable d’environnement : `deviceToCloudUploadProperties__uploadOrder={NewestFirst,OldestFirst}` |
| cloudStorageConnectionString |  | `"DefaultEndpointsProtocol=https;AccountName=<your Azure Storage Account Name>;AccountKey=<your Azure Storage Account Key>;EndpointSuffix=<your end point suffix>"` est une chaîne de connexion qui vous permet de spécifier le compte de stockage vers lequel vous souhaitez que vos données soient téléchargées. Spécifiez `Azure Storage Account Name`, `Azure Storage Account Key`, `End point suffix`. Ajoutez la valeur EndpointSuffix appropriée d’Azure où les données sont chargées, elle varie entre Azure global, Azure Government et Microsoft Azure Stack. <br><br> Vous pouvez choisir de spécifier la chaîne de connexion SAS du stockage Azure ici. Toutefois, vous devez mettre à jour cette propriété lorsqu’elle expire. <br><br> Variable d’environnement : `deviceToCloudUploadProperties__cloudStorageConnectionString=<connection string>` |
| storageContainersForUpload | `"<source container name1>": {"target": "<target container name>"}`<br><br> `"<source container name1>": {"target": "%h-%d-%m-%c"}` <br><br> `"<source container name1>": {"target": "%d-%c"}` | Vous permet de spécifier les noms des conteneurs que vous voulez télécharger dans Azure. Ce module vous permet de spécifier des noms de conteneur source et cible. Si vous ne spécifiez pas le nom du conteneur cible, le nom `<IoTHubName>-<IotEdgeDeviceID>-<ModuleName>-<SourceContainerName>` lui sera automatiquement attribué. Vous pouvez créer des chaînes de modèle pour le nom du conteneur cible. Consultez la colonne des valeurs possibles. <br>* %h -> nom de l’IoT Hub (entre 3 et 50 caractères). <br>* %d -> ID d’appareil IoT Edge (entre 1 et 129 caractères). <br>* %m -> nom du module (entre 1 et 64 caractères). <br>* %c -> nom du conteneur source (entre 3 et 63 caractères). <br><br>La taille maximale du nom du conteneur est 63 caractères. En affectant automatiquement le nom du conteneur cible si la taille du conteneur dépasse 63 caractères, cela réduira chaque section (IoTHubName, IotEdgeDeviceID, ModuleName, SourceContainerName) à 15 caractères. <br><br> Variable d’environnement : `deviceToCloudUploadProperties__storageContainersForUpload__<sourceName>__target=<targetName>` |
| deleteAfterUpload | true, false | Définissez cette valeur sur `false` par défaut. Si elle est définie sur `true`, cela supprime automatiquement les données après le chargement des données dans un stockage cloud. <br><br> **AVERTISSEMENT**  : si vous utilisez des objets blob d’ajout, ce paramètre supprime les objets blob d’ajout du stockage local après un chargement réussi, et toutes les opérations de bloc d’ajout ultérieures à ces objets blob échouent. Utilisez ce paramètre avec précaution. Ne l’activez pas si votre application effectue des opérations d’ajout peu fréquentes ou ne prend pas en charge les opérations d’ajout continues<br><br> Variable d’environnement : `deviceToCloudUploadProperties__deleteAfterUpload={false,true}`. |

### <a name="deviceautodeleteproperties"></a>deviceAutoDeleteProperties

Le nom de ce paramètre est `deviceAutoDeleteProperties`. Si vous utilisez le simulateur IoT Edge, définissez les valeurs sur les variables d’environnement associées pour ces propriétés, que vous trouverez dans la section Explication.

| Propriété | Valeurs possibles | Explication |
| ----- | ----- | ---- |
| deleteOn | true, false | Définissez cette valeur sur `false` par défaut. Si vous souhaitez activer cette fonctionnalité, définissez ce champ sur `true`. <br><br> Variable d’environnement : `deviceAutoDeleteProperties__deleteOn={false,true}` |
| deleteAfterMinutes | `<minutes>` | Indiquez le temps en minutes. Le module supprime automatiquement vos objets blob du stockage local à l’expiration de cette valeur. Le nombre de minutes maximal actuellement autorisé est de 35 791. <br><br> Variable d’environnement : `deviceAutoDeleteProperties__ deleteAfterMinutes=<minutes>` |
| retainWhileUploading | true, false | Par défaut, elle est définie sur `true` et conservera l’objet blob pendant son chargement dans le stockage cloud si deleteAfterMinutes expire. Vous pouvez la définir sur `false` pour que les données soient supprimées dès que deleteAfterMinutes expire. Remarque : Pour que cette propriété fonctionne, uploadOn doit être définie sur true.  <br><br> **AVERTISSEMENT**  : si vous utilisez des objets blob d’ajout, ce paramètre les supprime du stockage local lorsque la valeur expire, et toutes les opérations de bloc d’ajout ultérieures à ces objets blob échouent. Nous vous conseillons de vérifier que la valeur d’expiration est suffisamment grande pour la fréquence attendue des opérations d’ajout effectuées par votre application.<br><br> Variable d’environnement : `deviceAutoDeleteProperties__retainWhileUploading={false,true}`|

## <a name="using-smb-share-as-your-local-storage"></a>Utilisation du partage SMB comme stockage local

Vous pouvez fournir le partage SMB comme chemin de stockage local lorsque vous déployez le conteneur Windows de ce module sur l’hôte Windows.

Assurez-vous que le partage SMB et l’appareil IoT se trouvent dans des domaines mutuellement approuvés.

Vous pouvez exécuter la commande PowerShell `New-SmbGlobalMapping` pour mapper le partage SMB localement sur l’appareil IoT exécutant Windows.

Voici les étapes de configuration :

```PowerShell
$creds = Get-Credential
New-SmbGlobalMapping -RemotePath <remote SMB path> -Credential $creds -LocalPath <Any available drive letter>
```

Par exemple :

```powershell
$creds = Get-Credential
New-SmbGlobalMapping -RemotePath \\contosofileserver\share1 -Credential $creds -LocalPath G:
```

Cette commande utilise les informations d’identification pour s’authentifier auprès du serveur SMB distant. Ensuite, mappez le chemin d’accès du partage distant au disque G: (ou toute autre lettre de disque disponible). Le volume de données de l’appareil IoT est maintenant mappé à un chemin d’accès sur le lecteur G:.

Assurez-vous que l’utilisateur de l’appareil IoT peut lire/écrire sur le partage SMB distant.

Pour votre déploiement, la valeur de `<storage mount>` peut être **G:/ContainerData:C:/BlobRoot** .

## <a name="granting-directory-access-to-container-user-on-linux"></a>Octroi de l’accès aux annuaires à l’utilisateur de conteneur sur Linux

Si vous avez utilisé le [montage de volume](https://docs.docker.com/storage/volumes/) pour le stockage dans vos options de création pour les conteneurs Linux, vous ne devez pas effectuer d’étapes supplémentaires, mais si vous avez utilisé le [montage de liaison](https://docs.docker.com/storage/bind-mounts/), ces étapes sont requises pour exécuter le service correctement.

En suivant le principe du moindre privilège pour limiter les droits d’accès des utilisateurs aux autorisations minimales dont ils ont besoin pour effectuer leur travail, ce module comprend un utilisateur (nom : absie, ID : 11000) et un groupe d’utilisateurs (nom : absie, ID : 11000). Si le conteneur est démarré en tant que **root** (l’utilisateur par défaut est **root** ), notre service est démarré en tant qu’utilisateur **absie** à faibles privilèges.

Avec ce comportement, la configuration des autorisations sur des liaisons de chemin d’accès d’ordinateur hôte est essentielle pour que le service fonctionne correctement ; sinon, le service se bloque avec des erreurs d’accès refusé. Le chemin d’accès utilisé dans la liaison de répertoire doit être accessible à l’utilisateur de conteneur (par exemple : absie 11000). Vous pouvez autoriser l’utilisateur de conteneur à accéder au répertoire en exécutant les commandes ci-dessous sur l’hôte :

```terminal
sudo chown -R 11000:11000 <blob-dir>
sudo chmod -R 700 <blob-dir>
```

Par exemple :

```terminal
sudo chown -R 11000:11000 /srv/containerdata
sudo chmod -R 700 /srv/containerdata
```

Si vous devez exécuter le service en tant qu’utilisateur autre que **absie** , vous pouvez spécifier votre ID d’utilisateur personnalisé dans createOptions, sous la propriété « User » de votre manifeste de déploiement. Dans ce cas, vous devez utiliser l’ID de groupe par défaut ou racine `0`.

```json
"createOptions": {
  "User": "<custom user ID>:0"
}
```

Accordez maintenant à l’utilisateur de conteneur un accès au répertoire

```terminal
sudo chown -R <user ID>:<group ID> <blob-dir>
sudo chmod -R 700 <blob-dir>
```

## <a name="configure-log-files"></a>Configurer des fichiers journaux

Pour plus d’informations sur la configuration des fichiers journaux de votre module, consultez ces [meilleures pratiques de production](./production-checklist.md#set-up-logs-and-diagnostics).

## <a name="connect-to-your-blob-storage-module"></a>Se connecter au module de stockage d’objets blob

Vous pouvez utiliser le nom du compte et la clé de compte que vous avez configurés pour que votre module accède au stockage d’objets blob sur votre appareil IoT Edge.

Spécifiez votre appareil IoT Edge en tant que point de terminaison d’objets blob pour toutes les demandes de stockage que vous lui adressez. Vous pouvez [Créer une chaîne de connexion pour un point de terminaison de stockage explicite](../storage/common/storage-configure-connection-string.md#create-a-connection-string-for-an-explicit-storage-endpoint) en utilisant les informations de l’appareil IoT Edge et le nom du compte que vous avez configuré.

* Pour les modules qui sont déployés sur le même appareil, où le module de « Stockage Blob Azure sur IoT Edge » est en cours d’exécution, le point de terminaison d’objet blob est : `http://<module name>:11002/<account name>`.
* Pour les modules ou applications s'exécutant sur un autre appareil, vous devez choisir le bon point de terminaison pour votre réseau. En fonction de la configuration de votre réseau, choisissez un format de point de terminaison de sorte que le trafic de données de votre module ou de votre application externe puisse atteindre l’appareil exécutant le module Stockage d’objets Blob Azure sur IoT Edge. Le point de terminaison d’objet blob pour ce scénario est l’un des suivants :
  * `http://<device IP >:11002/<account name>`
  * `http://<IoT Edge device hostname>:11002/<account name>`
  * `http://<fully qualified domain name>:11002/<account name>`
 
 > [!IMPORTANT]
 > Azure IoT Edge respecte la casse lorsque vous effectuez des appels de modules, et le SDK de stockage utilise également des minuscules par défaut. Même si le nom du module dans la [place de marché Azure](how-to-deploy-modules-portal.md#deploy-modules-from-azure-marketplace) est **AzureBlobStorageonIoTEdge** , le passage de ce dernier en minuscules permet de s’assurer que vos connexions au module de stockage Blob Azure sur IoT Edge ne sont pas interrompues.
 
## <a name="azure-blob-storage-quickstart-samples"></a>Exemples de démarrage rapide de Stockage Blob Azure

La documentation du Stockage Blob Azure comprend des guides de démarrage rapide avec des exemples de code dans différents langages. Vous pouvez exécuter ces exemples pour tester le Stockage Blob Azure sur IoT Edge en changeant le point de terminaison des objets blob pour le connecter à votre module de stockage d’objets blob local.

Les exemples de guides de démarrage rapide suivants utilisent des langages qui sont également pris en charge par IoT Edge, vous pouvez donc les déployer en tant que modules IoT Edge en même temps que le module de stockage d’objets blob :

* [.NET](../storage/blobs/storage-quickstart-blobs-dotnet.md)
* [Python](../storage/blobs/storage-quickstart-blobs-python.md)
  * Les versions antérieures à la version 2.1 du kit de développement logiciel (SDK) Python présentent un problème connu dans lequel le module ne retourne pas l’heure de création de l’objet blob. En raison de ce problème, certaines méthodes comme les objets BLOB de liste ne fonctionnent pas. En tant que solution de contournement, définissez explicitement la version de l’API sur le client d’objet blob sur « 2017-04-17 ». Exemple : `block_blob_service._X_MS_VERSION = '2017-04-17'`
  * [Exemple Append Blob](https://github.com/Azure/azure-storage-python/blob/master/samples/blob/append_blob_usage.py)
* [Node.JS](../storage/blobs/storage-quickstart-blobs-nodejs-legacy.md)
* [JS/HTML](../storage/blobs/storage-quickstart-blobs-javascript-client-libraries-legacy.md)
* [Ruby](../storage/blobs/storage-quickstart-blobs-ruby.md)
* [Go](../storage/blobs/storage-quickstart-blobs-go.md)
* [PHP](../storage/blobs/storage-quickstart-blobs-php.md)

## <a name="connect-to-your-local-storage-with-azure-storage-explorer"></a>Se connecter à votre stockage local avec l’Explorateur Stockage Microsoft Azure

Vous pouvez utiliser l’[Explorateur Stockage Azure](https://github.com/microsoft/AzureStorageExplorer/releases/tag/v1.14.2) pour vous connecter à votre compte de stockage local.

1. Télécharger et installer l’Explorateur Stockage Microsoft Azure

1. Se connecter à Stockage Azure à l’aide d’une chaîne de connexion

1. Fournissez la chaîne de connexion : `DefaultEndpointsProtocol=http;BlobEndpoint=http://<host device name>:11002/<your local account name>;AccountName=<your local account name>;AccountKey=<your local account key>;`

1. Suivez les étapes pour vous connecter.

1. Créer le conteneur à l’intérieur de votre compte de stockage local

1. Lancez le chargement de fichiers en tant qu’objets blob de blocs ou objets blob d’ajout.
   > [!NOTE]
   > Ce module ne prend pas en charge les objets blob de pages.

1. Vous pouvez également choisir de vous connecter à vos comptes de stockage Azure dans Explorateur Stockage. Cette configuration vous offre une vue unique sur votre compte de stockage local et votre compte de stockage Azure

## <a name="supported-storage-operations"></a>Opérations de stockage prises en charge

Les modules de stockage d’objets blob sur IoT Edge utilisent les kits SDK de stockage Azure et sont compatibles avec la version 17-04-2017 de l’API de stockage Azure pour les points de terminaison d’objets blob de blocs.

Les opérations de Stockage Blob Azure ne sont pas toutes prises en charge par le stockage Blob Azure sur IoT Edge. Cette section liste le statut de chacune d’entre elles.

### <a name="account"></a>Compte

Prises en charge :

* Répertorier les conteneurs

Non prises en charge :

* Obtenir et définir les propriétés du service BLOB
* Demande d’objet blob préliminaire
* Obtenir les statistiques du service BLOB
* Obtenir les informations de compte

### <a name="containers"></a>Containers

Prises en charge :

* Créer et supprimer un conteneur
* Obtenir les métadonnées et les propriétés de conteneur
* Liste des objets blob
* Obtenir et définir une ACL de conteneur
* Définir les métadonnées de conteneur

Non prises en charge :

* Louer à bail un conteneur

### <a name="blobs"></a>Objets blob

Prises en charge :

* Placer, obtenir et supprimer un objet blob
* Obtenir et définir les propriétés d’un objet blob
* Obtenir et définir les métadonnées d’un objet blob

Non prises en charge :

* Louer à bail un objet blob
* Faire une capture instantanée d’un objet blob
* Copier et abandonner la copie d’un objet blob
* Annuler la suppression d’un objet blob
* Définir un niveau d’objet blob

### <a name="block-blobs"></a>Objets blob de blocs

Prises en charge :

* Placer un bloc
* Placer et obtenir une liste de blocs

Non prises en charge :

* Placer un bloc à partir d’une URL

### <a name="append-blobs"></a>Objets blob d’ajout

Prises en charge :

* Bloc d’ajout

Non prises en charge :

* Ajouter un bloc à partir d’une URL

## <a name="event-grid-on-iot-edge-integration"></a>Intégration d’Event Grid sur IoT Edge

> [!CAUTION]
> L’intégration à Event Grid sur IoT Edge est en version préliminaire

Ce module Stockage Blob Azure sur IoT Edge permet désormais d’intégrer Event Grid sur IoT Edge. Pour plus d’informations sur cette intégration, reportez-vous au [didacticiel pour déployer les modules, publier des événements et vérifier la remise d’événements](../event-grid/edge/react-blob-storage-events-locally.md).

## <a name="release-notes"></a>Notes de publication

Voici les [notes de publication dans docker hub](https://hub.docker.com/_/microsoft-azure-blob-storage) pour ce module

## <a name="suggestions"></a>Suggestions

Vos commentaires sont importants et nous permettent de faire de ce module et de ses fonctionnalités utiles et faciles des outils faciles à utiliser et pratiques. Partagez vos commentaires et dites-nous comment nous pouvons les améliorer.

Vous pouvez nous contacter au absiotfeedback@microsoft.com

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur le [Déploiement du Stockage Blob Azure sur IoT Edge](how-to-deploy-blob.md)

Restez informé des dernières mises à jour et annonces en consultant le [blog relatif au stockage Blob Azure sur IoT Edge.](https://aka.ms/abs-iot-blogpost)

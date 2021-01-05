---
title: API REST de conversion de ressources
description: Décrit comment convertir une ressource via l’API REST
author: florianborn71
ms.author: flborn
ms.date: 02/04/2020
ms.topic: how-to
ms.openlocfilehash: 889a70005f1cbabaad525147b4661ea04886138a
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94445606"
---
# <a name="use-the-model-conversion-rest-api"></a>Utiliser l’API REST de conversion de modèle

Le service de [conversion de modèle](model-conversion.md) est contrôlé via une [API REST](https://en.wikipedia.org/wiki/Representational_state_transfer). Cet article décrit les détails de l’API du service de conversion.

## <a name="regions"></a>Régions

Pour les URL de base auxquelles envoyer les demandes, consultez la [liste des régions disponibles](../../reference/regions.md).

## <a name="common-headers"></a>En-têtes courants

### <a name="common-request-headers"></a>En-têtes de demande courants

Ces en-têtes doivent être spécifiés pour toutes les demandes :

- L’en-tête **Authorisation** doit avoir la valeur « Bearer [*TOKEN*] », où [*TOKEN*] est un [jeton d’accès de service](../tokens.md).

### <a name="common-response-headers"></a>En-têtes de réponse courants

Toutes les réponses contiennent les en-têtes suivants :

- L’en-tête **MS-CV** contient une chaîne unique utilisable pour suivre l’appel au sein du service.

## <a name="endpoints"></a>Points de terminaison

Le service de conversion fournit trois points de terminaison d’API REST pour effectuer les opérations suivantes :

- démarrer la conversion du modèle à l’aide d’un compte de stockage lié à votre compte Azure Remote Rendering ; 
- démarrer la conversion du modèle à l’aide de *signatures d’accès partagé (SAP)* fournies ;
- interroger l’état de conversion.

### <a name="start-conversion-using-a-linked-storage-account"></a>Démarrer la conversion à l’aide d’un compte de stockage lié
Votre compte Azure Remote Rendering doit avoir accès au compte de stockage fourni en suivant la procédure [Lier des comptes de stockage](../create-an-account.md#link-storage-accounts).

| Point de terminaison | Méthode |
|-----------|:-----------|
| /v1/accounts/**accountID**/conversions/create | POST |

Retourne l’ID de la conversion en cours dans un document JSON. Le nom de champ est « conversionId ».

#### <a name="request-body"></a>Corps de la demande

> [!NOTE]
> Tout ce qui se trouve sous `input.folderPath` sera récupéré pour effectuer la conversion sur Azure. Si `input.folderPath` n’est pas spécifié, tout le contenu du conteneur est récupéré. Tous les blobs et dossiers récupérés doivent avoir [un nom de fichier Windows valide](/windows/win32/fileio/naming-a-file#naming-conventions).

```json
{
    "input":
    {
        "storageAccountname": "<the name of a connected storage account - this does not include the domain suffix (.blob.core.windows.net)>",
        "blobContainerName": "<the name of the blob container containing your input asset data>",
        "folderPath": "<optional: can be omitted or empty - a subpath in the input blob container>",
        "inputAssetPath" : "<path to the model in the input blob container relative to the folderPath (or container root if no folderPath is specified)>"
    },
    "output":
    {
        "storageAccountname": "<the name of a connected storage account - this does not include the domain suffix (.blob.core.windows.net)>",
        "blobContainerName": "<the name of the blob container where the converted asset will be copied to>",
        "folderPath": "<optional: can be omitted or empty - a subpath in the output blob container. Will contain the asset and log files>",
        "outputAssetFileName": "<optional: can be omitted or empty. The filename of the converted asset. If provided the filename needs to end in .arrAsset>"
    }
}
```
### <a name="start-conversion-using-provided-shared-access-signatures"></a>Démarrer la conversion à l’aide de signatures d’accès partagé fournies
Si votre compte ARR n’est pas lié à votre compte de stockage, cette interface REST vous permet de fournir un accès à l’aide de *signatures d’accès partagé (SAP)* .

| Point de terminaison | Méthode |
|-----------|:-----------|
| /v1/accounts/**accountID**/conversions/createWithSharedAccessSignature | POST |

Retourne l’ID de la conversion en cours dans un document JSON. Le nom du champ est `conversionId`.

#### <a name="request-body"></a>Corps de la demande

Le corps de la demande est le même que dans l’appel de création REST ci-dessus, mais l’entrée et la sortie contiennent des *jetons de signatures d’accès partagé (SAP)* . Ces jetons donnent accès au compte de stockage pour la lecture de l’entrée et l’écriture du résultat de la conversion.

> [!NOTE]
> Ces jetons d’URI SAP sont les chaînes de requête, non l’URI complet. 

> [!NOTE]
> Tout ce qui se trouve sous `input.folderPath` sera récupéré pour effectuer la conversion sur Azure. Si `input.folderPath` n’est pas spécifié, tout le contenu du conteneur est récupéré. Tous les blobs et dossiers récupérés doivent avoir [un nom de fichier Windows valide](/windows/win32/fileio/naming-a-file#naming-conventions).

```json
{
    "input":
    {
        "storageAccountname": "<the name of a connected storage account - this does not include the domain suffix (.blob.core.windows.net)>",
        "blobContainerName": "<the name of the blob container containing your input asset data>",
        "folderPath": "<optional: can be omitted or empty - a subpath in the input blob container>",
        "inputAssetPath" : "<path to the model in the input blob container relative to the folderPath (or container root if no folderPath is specified)>",
        "containerReadListSas" : "<a container SAS token which gives read and list access to the given input blob container>"
    },
    "output":
    {
        "storageAccountname": "<the name of a connected storage account - this does not include the domain suffix (.blob.core.windows.net)>",
        "blobContainerName": "<the name of the blob container where the converted asset will be copied to>",
        "folderPath": "<optional: can be omitted or empty - a subpath in the output blob container. Will contain the asset and log files>",
        "outputAssetFileName": "<optional: can be omitted or empty. The filename of the converted asset. If provided the filename needs to end in .arrAsset>",
        "containerWriteSas" : "<a container SAS token which gives write access to the given output blob container>"
    }
}
```

### <a name="poll-conversion-status"></a>Interroger l’état de la conversion
Vous pouvez interroger l’état d’une conversion en cours démarrée avec l’un des appels REST ci-dessus en utilisant l’interface suivante :


| Point de terminaison | Méthode |
|-----------|:-----------|
| /v1/accounts/**accountID**/conversions/**conversionId** | GET |

Retourne un document JSON avec un champ « status » qui peut avoir les valeurs suivantes :

- « Created »
- « Running »
- « Success »
- « Failure »

Si l’état est « Failure » (échec), il y a un champ « error » supplémentaire avec un sous-champ « message » contenant les informations sur l’erreur. Des journaux supplémentaires sont chargés sur votre conteneur de sortie.

## <a name="list-conversions"></a>Lister les conversions

Pour obtenir la liste de toutes les conversions d’un compte, utilisez l’interface :

| Point de terminaison | Méthode |
|-----------|:-----------|
| /v1/accounts/**accountID**/conversions?skiptoken=**skipToken** | GET |

| Paramètre | Obligatoire |
|-----------|:-----------|
| accountID | Oui |
| skiptoken | Non |

Retourne un document json qui contient un tableau des conversions et leurs détails. Cette requête retourne un maximum de 50 conversions à la fois. S’il y a plus de conversions à récupérer, la réponse contient une propriété **nextLink** contenant le skipToken interrogeable pour récupérer le jeu suivant de résultats.

## <a name="next-steps"></a>Étapes suivantes

- [Utiliser Stockage Blob Azure pour une conversion de modèle](blob-storage.md)
- [Conversion de modèle](model-conversion.md)
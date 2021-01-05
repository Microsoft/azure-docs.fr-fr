---
title: Personnaliser un modèle de langage avec l’API Video Indexer
titlesuffix: Azure Media Services
description: Découvrez comment personnaliser un modèle de langage avec l’API Video Indexer.
services: media-services
author: anikaz
manager: johndeu
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 02/04/2020
ms.author: kumud
ms.openlocfilehash: f373afae03357ffb65eb459f806fe441e29b21b9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87047080"
---
# <a name="customize-a-language-model-with-the-video-indexer-api"></a>Personnaliser un modèle de langage avec l’API Video Indexer

Video Indexer vous permet de créer des modèles de langage personnalisés pour la reconnaissance vocale en chargeant un texte d’adaptation, à savoir un texte extrait du domaine de vocabulaire auquel vous souhaitez que le moteur s’adapte. Une fois votre modèle entraîné, les nouveaux mots qui apparaissent dans le texte d’adaptation seront reconnus.

Pour obtenir une présentation détaillée et les meilleures pratiques pour les modèles linguistiques personnalisés, consultez [Personnaliser un modèle linguistique avec Video Indexer](customize-language-model-overview.md).

Vous pouvez utiliser les API Video Indexer pour créer et modifier des modèles linguistiques personnalisés dans votre compte, comme décrit dans cette rubrique. Vous pouvez également utiliser le site web comme décrit dans [Personnaliser un modèle linguistique avec le site web Video Indexer](customize-language-model-with-api.md).

## <a name="create-a-language-model"></a>Créer un modèle de langage

L'API [Créer un modèle de langage](https://api-portal.videoindexer.ai/docs/services/Operations/operations/Create-Language-Model?) crée un nouveau modèle de langage personnalisé dans le compte spécifié. Vous pouvez charger des fichiers pour le modèle linguistique dans cet appel. Vous pouvez également créer ici le modèle linguistique et charger ultérieurement des fichiers en mettant à jour le modèle linguistique.

> [!NOTE]
> Vous devez néanmoins former le modèle avec ses fichiers pour lui permettre d’en apprendre le contenu. La section suivante fournit des instructions sur la formation d’une langue.

Pour charger des fichiers à ajouter au modèle de langage, vous devez charger les fichiers dans le corps à l’aide des données de formulaire en plus de fournir les valeurs des paramètres requis ci-dessus. Il existe deux façons d'effectuer cette tâche :

* La clé sera le nom de fichier et la valeur sera le fichier txt.
* La clé sera le nom de fichier et la valeur sera une URL vers le fichier txt.

### <a name="response"></a>response

La réponse fournit des métadonnées sur le modèle de langage qui vient d’être créé ainsi que des métadonnées sur chacun des fichiers du modèle au format de cet exemple de sortie JSON :

```json
{
    "id": "dfae5745-6f1d-4edd-b224-42e1ab57a891",
    "name": "TestModel",
    "language": "En-US",
    "state": "None",
    "languageModelId": "00000000-0000-0000-0000-000000000000",
    "files": [
    {
        "id": "25be7c0e-b6a6-4f48-b981-497e920a0bc9",
        "name": "hellofile",
        "enable": true,
        "creator": "John Doe",
        "creationTime": "2018-04-28T11:55:34.6733333"
    },
    {
        "id": "33025f5b-2354-485e-a50c-4e6b76345ca7",
        "name": "worldfile",
        "enable": true,
        "creator": "John Doe",
        "creationTime": "2018-04-28T11:55:34.86"
    }
    ]
}

```

## <a name="train-a-language-model"></a>Former un modèle linguistique

L'API [Former un modèle de langage](https://api-portal.videoindexer.ai/docs/services/operations/operations/Train-Language-Model?&pattern=train) forme un modèle de langage personnalisé dans le compte spécifié, avec le contenu des fichiers qui ont été chargés et activés dans le modèle de langage.

> [!NOTE]
> Vous devez d’abord créer le modèle linguistique et charger ses fichiers. Vous pouvez charger les fichiers lors de la création ou de la mise à jour du modèle de langage.

### <a name="response"></a>response

La réponse fournit des métadonnées sur le modèle de langage qui vient d’être formé ainsi que des métadonnées sur chacun des fichiers du modèle au format de l’exemple de sortie JSON :

```json
{
    "id": "41464adf-e432-42b1-8e09-f52905d7e29d",
    "name": "TestModel",
    "language": "En-US",
    "state": "Waiting",
    "languageModelId": "531e5745-681d-4e1d-b124-12e5ab57a891",
    "files": [
    {
        "id": "84fcf1ac-1952-48f3-b372-18f768eedf83",
        "name": "RenamedFile",
        "enable": false,
        "creator": "John Doe",
        "creationTime": "2018-04-27T20:10:10.5233333"
    },
    {
        "id": "9ac35b4b-1381-49c4-9fe4-8234bfdd0f50",
        "name": "hellofile",
        "enable": true,
        "creator": "John Doe",
        "creationTime": "2018-04-27T20:10:10.68"
    }
    ]
}
```

L’`id` retourné est un identifiant unique utilisé pour distinguer les modèles de langage, tandis que `languageModelId` est utilisé à la fois pour les API de [chargement d’une vidéo vers l’index](https://api-portal.videoindexer.ai/docs/services/operations/operations/Upload-video?) et de [réindexation d’une vidéo](https://api-portal.videoindexer.ai/docs/services/operations/operations/Re-index-video?) (également appelé `linguisticModelId` dans les API de chargement et de réindexation Video Indexer).

## <a name="delete-a-language-model"></a>Supprimer un modèle de langage

L'API [Supprimer un modèle de langage](https://api-portal.videoindexer.ai/docs/services/operations/operations/Delete-Language-Model?&pattern=delete) supprime un modèle de langage personnalisé du compte spécifié. Toute vidéo qui utilisait le modèle de langage supprimé conservera le même index jusqu'à la réindexation de la vidéo. Si vous réindexez la vidéo, vous pouvez lui assigner un nouveau modèle de langage. Sinon, Video Indexer utilisera son modèle par défaut pour réindexer la vidéo.

### <a name="response"></a>response

Aucun contenu n’est retourné en cas de suppression effective du modèle de langage.

## <a name="update-a-language-model"></a>Mettre à jour un modèle linguistique

L'API [Mettre à jour un modèle de langage](https://api-portal.videoindexer.ai/docs/services/operations/operations/Update-Language-Model?&pattern=update) met à jour un modèle de langage personnalisé dans le compte spécifié.

> [!NOTE]
> Vous devez avoir déjà créé le modèle linguistique. Vous pouvez utiliser cet appel pour activer ou désactiver tous les fichiers du modèle, mettre à jour le nom du modèle linguistique, et charger des fichiers à ajouter au modèle linguistique.

Pour charger des fichiers à ajouter au modèle de langage, vous devez charger les fichiers dans le corps à l’aide des données de formulaire en plus de fournir les valeurs des paramètres requis ci-dessus. Il existe deux façons d'effectuer cette tâche :

* La clé sera le nom de fichier et la valeur sera le fichier txt.
* La clé sera le nom de fichier et la valeur sera une URL vers le fichier txt.

### <a name="response"></a>response

La réponse fournit des métadonnées sur le modèle de langage qui vient d’être formé ainsi que des métadonnées sur chacun des fichiers du modèle au format de l’exemple de sortie JSON :

```json
{
    "id": "41464adf-e432-42b1-8e09-f52905d7e29d",
    "name": "TestModel",
    "language": "En-US",
    "state": "Waiting",
    "languageModelId": "531e5745-681d-4e1d-b124-12e5ab57a891",
    "files": [
    {
        "id": "84fcf1ac-1952-48f3-b372-18f768eedf83",
        "name": "RenamedFile",
        "enable": true,
        "creator": "John Doe",
        "creationTime": "2018-04-27T20:10:10.5233333"
    },
    {
        "id": "9ac35b4b-1381-49c4-9fe4-8234bfdd0f50",
        "name": "hellofile",
        "enable": true,
        "creator": "John Doe",
        "creationTime": "2018-04-27T20:10:10.68"
    }
    ]
}
```

Utilisez l’`id` des fichiers retournée dans la réponse pour télécharger le contenu du fichier.

## <a name="update-a-file-from-a-language-model"></a>Mettre à jour un fichier d’un modèle linguistique

L'API [Mettre à jour un fichier](https://api-portal.videoindexer.ai/docs/services/operations/operations/Update-Language-Model-file?&pattern=update) vous permet de mettre à jour le nom et l’état `enable` d’un fichier de modèle de langage personnalisé dans le compte spécifié.

### <a name="response"></a>response

La réponse fournit des métadonnées sur le fichier que vous avez mis à jour, au format de l’exemple de sortie JSON ci-dessous.

```json
{
  "id": "84fcf1ac-1952-48f3-b372-18f768eedf83",
  "name": "RenamedFile",
  "enable": false,
  "creator": "John Doe",
  "creationTime": "2018-04-27T20:10:10.5233333"
}
```

Utilisez l’`id` du fichier retourné dans la réponse pour télécharger le contenu du fichier.

## <a name="get-a-specific-language-model"></a>Obtenir un modèle linguistique spécifique

L'API [get](https://api-portal.videoindexer.ai/docs/services/operations/operations/Get-Language-Model?&pattern=get) retourne des informations sur le modèle de langage spécifié dans le compte spécifié ainsi que sur les fichiers que ce modèle contient.

### <a name="response"></a>response

La réponse fournit des métadonnées sur le modèle de langage spécifié ainsi que des métadonnées sur chacun des fichiers du modèle au format de cet exemple de sortie JSON :

```json
{
    "id": "dfae5745-6f1d-4edd-b224-42e1ab57a891",
    "name": "TestModel",
    "language": "En-US",
    "state": "None",
    "languageModelId": "00000000-0000-0000-0000-000000000000",
    "files": [
    {
        "id": "25be7c0e-b6a6-4f48-b981-497e920a0bc9",
        "name": "hellofile",
        "enable": true,
        "creator": "John Doe",
        "creationTime": "2018-04-28T11:55:34.6733333"
    },
    {
        "id": "33025f5b-2354-485e-a50c-4e6b76345ca7",
        "name": "worldfile",
        "enable": true,
        "creator": "John Doe",
        "creationTime": "2018-04-28T11:55:34.86"
    }
    ]
}
```

Utilisez l’`id` du fichier retourné dans la réponse pour télécharger le contenu du fichier.

## <a name="get-all-the-language-models"></a>Obtenir tous les modèles linguistiques

L'API [get all](https://api-portal.videoindexer.ai/docs/services/operations/operations/Get-Language-Models?&pattern=get) dresse la liste de tous les modèles de langue personnalisés dans le compte spécifié.

### <a name="response"></a>response

La réponse renvoie une liste de tous les modèles de langage dans votre compte, en précisant chaque métadonnée et fichier, au format de cet exemple de sortie JSON :

```json
[
    {
        "id": "dfae5745-6f1d-4edd-b224-42e1ab57a891",
        "name": "TestModel",
        "language": "En-US",
        "state": "None",
        "languageModelId": "00000000-0000-0000-0000-000000000000",
        "files": [
        {
            "id": "25be7c0e-b6a6-4f48-b981-497e920a0bc9",
            "name": "hellofile",
            "enable": true,
            "creator": "John Doe",
            "creationTime": "2018-04-28T11:55:34.6733333"
        },
        {
            "id": "33025f5b-2354-485e-a50c-4e6b76345ca7",
            "name": "worldfile",
            "enable": true,
            "creator": "John Doe",
            "creationTime": "2018-04-28T11:55:34.86"
        }
        ]
    },
    {
        "id": "dfae5745-6f1d-4edd-b224-42e1ab57a892",
        "name": "AnotherTestModel",
        "language": "En-US",
        "state": "None",
        "languageModelId": "00000000-0000-0000-0000-000000000001",
        "files": []
    }
]
```

## <a name="delete-a-file-from-a-language-model"></a>Supprimer un fichier d’un modèle linguistique

L'API [Supprimer](https://api-portal.videoindexer.ai/docs/services/operations/operations/Delete-Language-Model-File?&pattern=delete) supprime le fichier spécifié à partir du modèle de langage spécifié dans le compte spécifié.

### <a name="response"></a>response

Aucun contenu n’est retourné en cas de suppression effective du fichier du modèle de langage.

## <a name="get-metadata-on-a-file-from-a-language-model"></a>Obtenir les métadonnées d’un fichier à partir d’un modèle linguistique

L'API [Obtenir les métadonnées d'un fichier](https://api-portal.videoindexer.ai/docs/services/operations/operations/Get-Language-Model-File-Data?&pattern=get%20language%20model) retourne le contenu et les métadonnées du fichier spécifié à partir du modèle de langage choisi dans votre compte.

### <a name="response"></a>response

La réponse fournit le contenu et les métadonnées du fichier au format JSON, sous une forme similaire à cet exemple :

```json
{
    "content": "hello\r\nworld",
    "id": "84fcf1ac-1952-48f3-b372-18f768eedf83",
    "name": "Hello",
    "enable": true,
    "creator": "John Doe",
    "creationTime": "2018-04-27T20:10:10.5233333"
}
```

> [!NOTE]
> Cet exemple de fichier contient les mots « hello » et « world » dans deux lignes distinctes.

## <a name="download-a-file-from-a-language-model"></a>Télécharger un fichier à partir d’un modèle de langage

L'API [Télécharger un fichier](https://api-portal.videoindexer.ai/docs/services/operations/operations/Download-Language-Model-File-Content?) télécharge un fichier texte avec le contenu du fichier provenant du modèle de langage dans le compte spécifié. Ce fichier texte doit correspondre au contenu du fichier texte chargé à l’origine.

### <a name="response"></a>response

La réponse entraînera le téléchargement d’un fichier texte avec le contenu du fichier au format JSON.

## <a name="next-steps"></a>Étapes suivantes

[Personnaliser un modèle linguistique à l’aide d’un site web](customize-language-model-with-website.md)

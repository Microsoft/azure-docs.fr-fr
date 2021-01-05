---
title: Activer et gérer le contrôle de version des objets blob
titleSuffix: Azure Storage
description: Découvrez comment activer le contrôle de version des objets blob dans le Portail Azure ou à l’aide d’un modèle Azure Resource Manager.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 11/17/2020
ms.author: tamram
ms.subservice: blobs
ms.custom: devx-track-csharp
ms.openlocfilehash: 5802070bf9b495c0e866d160d6661349369a444e
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95993740"
---
# <a name="enable-and-manage-blob-versioning"></a>Activer et gérer le contrôle de version des objets blob

Vous pouvez activer le contrôle de version du stockage d’objets blob pour gérer automatiquement les versions précédentes d’un objet.  Lorsque le contrôle de version est activé, vous pouvez restaurer une version antérieure d’un objet blob pour récupérer vos données si celles-ci ont été modifiées ou supprimées par erreur.

Cet article explique comment activer ou désactiver le contrôle de version des blobs pour le compte de stockage à l’aide du portail Azure ou d’un modèle de Resource Manager. Pour en savoir plus sur le contrôle de version des blobs, consultez [Contrôle de version des objets blob](versioning-overview.md).

[!INCLUDE [storage-data-lake-gen2-support](../../../includes/storage-data-lake-gen2-support.md)]

## <a name="enable-blob-versioning"></a>Activer la gestion des versions des objets blob

# <a name="azure-portal"></a>[Azure portal](#tab/portal)

Pour activer le contrôle de version des objets blob dans le Portail Azure :

1. Accédez à votre compte de stockage dans le portail.
1. Sous **Service BLOB**, choisissez **Protection des données**.
1. Dans la section **Gestion de version**, sélectionnez **Activée**.

:::image type="content" source="media/versioning-enable/portal-enable-versioning.png" alt-text="Capture d’écran représentant l’activation du contrôle de version des objets blob dans le Portail Azure":::

# <a name="template"></a>[Modèle](#tab/template)

Pour activer le contrôle de version des objets blob avec un modèle, créez un modèle avec la propriété **IsVersioningEnabled** sur **true**. Les étapes suivantes montrent comment créer un modèle dans le Portail Azure.

1. Dans le Portail Azure, choisissez **Créer une ressource**.
1. Dans **Rechercher sur la Place de marché**, tapez **déploiement de modèle**, puis appuyez sur **Entrée**.
1. Choisissez **Template deployment**, **Créer**, puis **Créer votre propre modèle dans l’éditeur**.
1. Dans l’éditeur de modèle, collez le code JSON suivant. Remplacez la valeur d’espace réservé `<accountName>` par le nom de votre compte de stockage.
1. Enregistrez le modèle.
1. Spécifiez le groupe de ressources du compte, puis choisissez le bouton **Acheter** pour déployer le modèle et activer le contrôle de version des objets blob.

    ```json
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {},
        "variables": {},
        "resources": [
            {
                "type": "Microsoft.Storage/storageAccounts/blobServices",
                "apiVersion": "2019-06-01",
                "name": "<accountName>/default",
                "properties": {
                    "IsVersioningEnabled": true
                }
            }
        ]
    }
    ```

Pour plus d’informations sur le déploiement de ressources à l’aide de modèles dans le Portail Azure, consultez [Déployer des ressources avec le Portail Azure](../../azure-resource-manager/templates/deploy-portal.md).

---

## <a name="modify-a-blob-to-trigger-a-new-version"></a>Modifier un objet blob pour déclencher une nouvelle version

L’exemple de code suivant montre comment déclencher la création d’une nouvelle version avec la bibliothèque cliente Stockage Microsoft Azure pour .NET version [12.5.1](https://www.nuget.org/packages/Azure.Storage.Blobs/12.5.1) ou ultérieure. Avant d’exécuter cet exemple, assurez-vous que vous avez activé le contrôle de version pour votre compte de stockage.

L’exemple crée un objet blob de blocs, puis met à jour les métadonnées de cet objet. La mise à jour des métadonnées de l’objet blob déclenche la création d’une nouvelle version. L’exemple récupère la version initiale et la version actuelle, et indique que seule la version actuelle contient les métadonnées.

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CRUD.cs" id="Snippet_UpdateVersionedBlobMetadata":::

## <a name="list-blob-versions"></a>Répertorier les versions de blob

Pour répertorier les versions ou les instantanés de blobs avec la bibliothèque de client .NET v12, spécifiez le paramètre [BlobStates](/dotnet/api/azure.storage.blobs.models.blobstates) avec le champ **Version**.

L’exemple de code suivant montre comment répertorier la version des blobs avec la bibliothèque de client Stockage Azure pour .NET, version [12.5.1](https://www.nuget.org/packages/Azure.Storage.Blobs/12.5.1) ou ultérieure. Avant d’exécuter cet exemple, assurez-vous que vous avez activé le contrôle de version pour votre compte de stockage.

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CRUD.cs" id="Snippet_ListBlobVersions":::

## <a name="next-steps"></a>Étapes suivantes

- [Contrôle de version des blobs](versioning-overview.md)
- [Suppression réversible pour les objets blob de Stockage Azure](./soft-delete-blob-overview.md)
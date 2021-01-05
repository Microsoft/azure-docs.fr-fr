---
title: Élément StorageBlobSelector de l'interface utilisateur
description: Décrit l'élément Microsoft.Storage.StorageAccountSelector de l'interface utilisateur du portail Azure.
author: tfitzmac
ms.topic: conceptual
ms.date: 10/27/2020
ms.author: tomfitz
ms.openlocfilehash: 1085b70df67a3f16a7f7f8c5ae85c9ab271b62ac
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92754228"
---
# <a name="microsoftstoragestorageblobselector-ui-element"></a>Élément Microsoft.Storage.StorageBlobSelector de l'interface utilisateur

Contrôle permettant de sélectionner un objet blob à partir d'un compte de stockage Azure.

## <a name="ui-sample"></a>Exemple d’interface utilisateur

L'utilisateur a la possibilité de parcourir les objets blob de stockage disponibles.

:::image type="content" source="./media/managed-application-elements/microsoft-storage-storageblobselector-browse.png" alt-text="Microsoft.Storage.StorageBlobSelector - Parcourir":::

Après avoir sélectionné **Parcourir** , l'utilisateur peut sélectionner un compte de stockage.

:::image type="content" source="./media/managed-application-elements/microsoft-storage-storageblobselector-select.png" alt-text="Microsoft.Storage.StorageBlobSelector - Sélectionner un stockage":::

L'utilisateur voit les conteneurs disponibles sur le compte de stockage et peut en sélectionner un.

:::image type="content" source="./media/managed-application-elements/microsoft-storage-storageblobselector-containers.png" alt-text="Microsoft.Storage.StorageBlobSelector - Sélectionner un conteneur":::

À partir du conteneur, l'utilisateur peut sélectionner un fichier.

:::image type="content" source="./media/managed-application-elements/microsoft-storage-storageblobselector-file.png" alt-text="Microsoft.Storage.StorageBlobSelector - Fichier":::

Le contrôle est mis à jour pour afficher le nom de fichier sélectionné.

:::image type="content" source="./media/managed-application-elements/microsoft-storage-storageblobselector-result.png" alt-text="Microsoft.Storage.StorageBlobSelector - Afficher le fichier sélectionné":::

## <a name="schema"></a>schéma

```json
{
  "name": "storageBlobSelection",
  "type": "Microsoft.Storage.StorageBlobSelector",
  "visible": true,
  "toolTip": "Select storage blob",
  "label": "Package (.zip, .cspkg)",
  "options": {
    "text": "Select Package"
  },
  "constraints": {
    "allowedFileExtensions": [ "zip", "cspkg" ]
  }
}
```

## <a name="sample-output"></a>Exemple de sortie

```json
{
  "blobName": "test.zip",
  "sasUri": "https://azstorageaccnt1.blob.core.windows.net/container1/test.zip?sp=r&se=2020-10-10T07:46:22Z&sv=2019-12-12&sr=b&sig=X4EL8ZsRmiP1TVxkVfTcGyMj2sHg1zCbFBXsDmnNOyg%3D"
}

```

## <a name="remarks"></a>Notes

La propriété **constraints.allowedFileExtensions** spécifie les types de fichiers autorisés.

## <a name="next-steps"></a>Étapes suivantes

* Pour voir une présentation de la création de définitions d’interface utilisateur, consultez la page [Prise en main de CreateUiDefinition](create-uidefinition-overview.md).
* Pour obtenir une description des propriétés communes des éléments d’interface utilisateur, consultez la page [Éléments de CreateUiDefinition](create-uidefinition-elements.md).

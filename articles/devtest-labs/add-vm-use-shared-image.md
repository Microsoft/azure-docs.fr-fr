---
title: Ajouter une machine virtuelle à l'aide d'une image partagée dans Azure DevTest Labs | Microsoft Docs
description: Apprenez à ajouter une machine virtuelle à l'aide d'une image de la galerie d'images partagées jointe dans Azure DevTest Labs
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 9421a1e21be9446b0e59328bd9a3730b57655274
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85483854"
---
# <a name="add-a-vm-using-an-image-from-the-attached-shared-image-gallery"></a>Ajouter une machine virtuelle à l'aide d'une image de la galerie d'images partagées jointe
Azure DevTest Labs vous permet de joindre une galerie d'images partagées à votre labo, puis d'utiliser des images de la galerie comme bases pour les machines virtuelles que vous créez dans le labo. Pour apprendre à joindre une galerie d'images partagées à votre labo, consultez [Configurer une galerie d'images partagées](configure-shared-image-gallery.md). Cet article explique comment ajouter une machine virtuelle à votre labo en utilisant une image de la galerie d'images partagées jointe comme base. 

## <a name="azure-portal"></a>Portail Azure
Dans cette section, vous allez apprendre à utiliser le Portail Azure pour ajouter une machine virtuelle à votre labo à partir d'une image de la galerie d'images partagées jointe. Cette section ne fournit pas d'instructions pas à pas détaillées pour créer une machine virtuelle à l'aide du Portail Azure. Pour plus d'informations, consultez [Créer une machine virtuelle - Portail Azure](devtest-lab-add-vm.md). Elle se contente de mettre en évidence les étapes de sélection d'une image dans la galerie d'images partagées jointe et de sélection de la version de l'image à utiliser. 

Lorsque vous ajoutez une machine virtuelle à votre labo, vous pouvez sélectionner une image de la galerie d'images partagées jointe comme base : 

![Sélectionner une image partagée comme base](./media/add-vm-use-shared-image/select-shared-image-for-base.png)

Ensuite, sous l'onglet **Paramètres avancés** de la page **Créer une ressource de labo**, vous pouvez sélectionner la version de l'image que vous souhaitez utiliser comme base :

![Sélectionner la version de l'image](./media/add-vm-use-shared-image/select-version-shared-image.png)

Une fois la machine virtuelle créée, vous pourrez utiliser une autre version de l'image. 

## <a name="resource-manager-template"></a>Modèle Resource Manager
Si vous utilisez un modèle Azure Resource Manager pour créer une machine virtuelle à l'aide d'une image de la galerie d'images partagées, définissez une valeur pour la propriété **sharedImageId** dans la section **Propriétés**. Voir l’exemple suivant : 

```json
"resources": [
{
    ...
    "properties": {
         "sharedImageId": "/subscriptions/111111111-1111-1111-1111-111111111111/resourcegroups/mydtlrg/providers/microsoft.devtestlab/labs/mydtllab/sharedgalleries/spsig/sharedimages/myimagefromgallery",
        "sharedImageVersion": "1.0.1",
        ...
    }
}
],
```

Pour accéder à un exemple complet de modèle Resource Manager, consultez l'exemple [Créer une machine virtuelle à l'aide d'une image de la galerie d'images partagées](https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/QuickStartTemplates/101-dtl-create-vm-username-pwd-sharedimage) de notre référentiel GitHub. 

## <a name="rest-api"></a>API REST

1. Tout d'abord, vous devez obtenir l'ID de l'image dans la galerie d'images partagées. L'une des méthodes consiste à dresser la liste de toutes les images de la galerie d'images partagées jointe à l'aide de la commande GET suivante. 

    ```rest
    GET  https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DevTestLab/labs/{labName}/sharedgalleries/{name}/sharedimages?api-version= 2018-10-15-preview
    ```
2. Appelez la méthode PUT sur les machines virtuelles en transmettant l'ID de l'image partagée que vous avez reçu lors de l'appel précédent de `properties.SharedImageId`.

## <a name="next-steps"></a>Étapes suivantes
Pour apprendre à joindre une galerie d'images partagées à un labo et à la configurer, consultez [Configurer une galerie d'images partagées](configure-shared-image-gallery.md).
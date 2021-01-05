---
title: Configurer une galerie d’images partagées dans Azure DevTest Labs | Microsoft Docs
description: Découvrez comment configurer une galerie d’images partagées dans Azure DevTest Labs, qui permet aux utilisateurs d’accéder à des images à partir d’un emplacement partagé lors de la création de ressources de laboratoire.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 96563b1dcfac171af38b229bb81d12b3afda2e2f
ms.sourcegitcommit: 03713bf705301e7f567010714beb236e7c8cee6f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92327975"
---
# <a name="configure-a-shared-image-gallery-in-azure-devtest-labs"></a>Configurer une galerie d’images partagées dans Azure DevTest Labs
DevTest Labs prend à présent en charge la fonctionnalité [Shared Image Gallery](../virtual-machines/windows/shared-image-galleries.md). Elle permet aux utilisateurs de labo d’accéder à des images provenant d’un emplacement partagé lors de la création de ressources de labo. Elle vous permet également de structurer et d’organiser vos images de machines virtuelles managées personnalisées. La fonctionnalité Shared Image Gallery prend en charge les opérations suivantes :

- Réplication globale et managée des images
- Gestion des versions et regroupement d’images pour une gestion simplifiée
- Rendre vos images hautement disponibles avec les comptes ZRS (stockage redondant interzone) dans les régions qui prennent en charge les zones de disponibilité. Le stockage redondant interzone (ZRS) offre une meilleure résilience en cas de défaillances de zones.
- Partage entre différents abonnements, et même entre locataires, à l’aide du contrôle d’accès en fonction du rôle Azure (Azure RBAC).

Pour plus d’informations, consultez la [documentation sur Shared Image Gallery](../virtual-machines/windows/shared-image-galleries.md). 
 
Si vous avez un grand nombre d’images managées à gérer et que vous voulez qu’elles soient disponibles pour toute l’entreprise, vous pouvez utiliser une galerie d’images partagées comme un dépôt qui facilite la mise à jour et le partage de vos images. En tant que propriétaire d’un labo, vous pouvez attacher une galerie d’images partagées existante à votre labo. Une fois la galerie attachée, les utilisateurs du labo peuvent créer des machines à partir de ces images les plus récentes. Un avantage clé de cette fonctionnalité est que DevTest Labs peut maintenant tirer parti du partage d’images dans différents labos, différents abonnements et différentes régions. 

> [!NOTE]
> Pour plus d’informations sur les coûts associés au service Galerie d’images partagées, consultez [Facturation pour la Galerie d’images partagées](../virtual-machines/windows/shared-image-galleries.md#billing).

## <a name="considerations"></a>Considérations
- Vous pouvez attacher une seule galerie d’images partagées à un labo à la fois. Avant d’attacher une autre galerie, vous devez détacher la galerie existante. 
- Actuellement, DevTest abs ne prend pas en charge le chargement d’images dans la galerie dans le cadre du labo. 
- Lors de la création d’une machine virtuelle à l’aide d’une image de galerie d’images partagées, DevTest Labs utilise toujours la dernière version publiée de cette image. Toutefois, si une image a plusieurs versions, l’utilisateur peut choisir de créer une machine à partir d’une version antérieure en accédant à l’onglet Paramètres avancés lors de la création de la machine virtuelle.  
- Bien que DevTest Labs s’efforce automatiquement de faire en sorte que la galerie d’images partagées réplique des images dans la région dans laquelle se trouve le labo, ce n’est pas toujours possible. Pour éviter que les utilisateurs rencontrent des problèmes lors de la création de machines virtuelles à partir de ces images, vérifiez que les images sont déjà répliquées dans la région du labo.

## <a name="use-azure-portal"></a>Utiliser le portail Azure
1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Dans le menu de navigation de gauche, sélectionnez **Tous les services** .
1. Sélectionnez **DevTest Labs** dans la liste.
1. Dans la liste de labos, sélectionnez votre **labo** .
1. Sélectionnez **Configuration et stratégies** dans la section **Paramètres** du menu à gauche.
1. Sélectionnez **Galeries d’images partagées** sous **Bases de machine virtuelle** dans le menu de gauche.

    ![Menu Galeries d’images partagées](./media/configure-shared-image-gallery/shared-image-galleries-menu.png)
1. Attachez une galerie d’images partagées existante à votre labo en cliquant sur le bouton **Attacher** , puis en sélectionnant votre galerie dans la liste déroulante.

    ![Joindre](./media/configure-shared-image-gallery/attach-options.png)
1. Une fois la galerie d’images attachée, sélectionnez-la pour y accéder. Configurez votre galerie pour **activer ou désactiver** les images partagées pour la création de machine virtuelle. Sélectionnez une galerie d’images dans la liste pour la configurer. 

    Par défaut, l’option **Autoriser l’utilisation de toutes les images comme bases de machine virtuelle** est définie sur **Oui** . Cela signifie que toutes les images disponibles dans la galerie d’images partagées attachée sont disponibles pour un utilisateur de labo lors de la création d’une machine virtuelle. Si l’accès à certaines images doit être restreint, définissez l’option **Autoriser l’utilisation de toutes les images comme bases de machine virtuelle** sur **Non** , sélectionnez les images que vous voulez autoriser lors de la création de machines virtuelles, puis sélectionnez le bouton **Enregistrer** .

    :::image type="content" source="./media/configure-shared-image-gallery/enable-disable.png" alt-text="Activer ou désactiver des images":::

    > [!NOTE]
    > Les images généralisées et spécialisées dans la galerie d’images partagée sont prises en charge. 
1. Les utilisateurs du labo peuvent alors créer une machine virtuelle avec les images activées en cliquant sur **+Ajouter** , puis en recherchant l’image dans la page de **choix de votre base** .

    ![Utilisateurs de labo](./media/configure-shared-image-gallery/lab-users.png)
## <a name="use-azure-resource-manager-template"></a>Utilisation d’un modèle Azure Resource Manager

### <a name="attach-a-shared-image-gallery-to-your-lab"></a>Attacher une galerie d’images partagées à votre labo
Si vous utilisez un modèle Azure Resource Manager pour attacher une galerie d’images partagées à votre labo, vous devez l’ajouter sous la section de ressources de votre modèle Resource Manager, comme indiqué dans l’exemple suivant :

```json
"resources": [
{
    "apiVersion": "2018-10-15-preview",
    "type": "Microsoft.DevTestLab/labs",
    "name": "mylab",
    "location": "eastus",
    "resources": [
    {
        "apiVersion":"2018-10-15-preview",
        "name":"myGallery",
        "type":"sharedGalleries",
        "properties": {
            "galleryId":"/subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/mySharedGalleryRg/providers/Microsoft.Compute/galleries/mySharedGallery",
            "allowAllImages": "Enabled"
        }
    }
    ]
}
```

Pour obtenir un exemple de modèle Resource Manager complet, consultez ces exemples de modèles Resource Manager dans notre dépôt GitHub public : [Configurer une galerie d’images partagées lors de la création d’un labo](https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/QuickStartTemplates/101-dtl-create-lab-shared-gallery-configured).

## <a name="use-rest-api"></a>Avec l’API REST

### <a name="get-a-list-of-labs"></a>Obtenir une liste des laboratoires 

```rest
GET  https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DevTestLab/labs?api-version= 2018-10-15-preview
```

### <a name="get-the-list-of-shared-image-galleries-associated-with-a-lab"></a>Obtenir la liste des galeries d’images partagées associées à un laboratoire

```rest
GET  https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DevTestLab/labs/{labName}/sharedgalleries?api-version= 2018-10-15-preview
   ```

### <a name="create-or-update-shared-image-gallery"></a>Créer ou mettre à jour une galeries d’images partagées

```rest
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DevTestLab/labs/{labName}/sharedgalleries/{name}?api-version= 2018-10-15-preview
Body: 
{
    "properties":{
        "galleryId": "[Shared Image Gallery resource Id]",
        "allowAllImages": "Enabled"
    }
}

```

### <a name="list-images-in-a-shared-image-gallery"></a>Répertorier les images figurant dans une galerie d’images partagées

```rest
GET  https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DevTestLab/labs/{labName}/sharedgalleries/{name}/sharedimages?api-version= 2018-10-15-preview
```



## <a name="next-steps"></a>Étapes suivantes
Consultez les articles suivants sur la création d’une machine virtuelle à l’aide d’une image de la galerie d’images partagées attachée : [Créer une machine virtuelle à l’aide d’une image partagée de la galerie](add-vm-use-shared-image.md)

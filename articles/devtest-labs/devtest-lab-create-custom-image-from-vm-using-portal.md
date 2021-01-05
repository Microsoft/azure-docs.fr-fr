---
title: Créer une image personnalisée Azure DevTest Labs à partir d’une machine virtuelle | Microsoft Docs
description: Découvrez comment créer une image personnalisée dans Azure DevTest Labs à partir d’une machine virtuelle approvisionnée à l’aide du portail Azure
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: ad45ed6eb7f97e14ec0ca0bb89efb2967c90fc16
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87277025"
---
# <a name="create-a-custom-image-from-a-vm"></a>Créer une image personnalisée à partir d’une machine virtuelle

[!INCLUDE [devtest-lab-custom-image-definition](../../includes/devtest-lab-custom-image-definition.md)]

## <a name="step-by-step-instructions"></a>Instructions pas à pas

Vous pouvez créer une image personnalisée à partir d’une machine virtuelle approvisionnée et ensuite utiliser cette image personnalisée pour créer d’autres machines virtuelles identiques. Les étapes suivantes expliquent comment créer une image personnalisée à partir d’une machine virtuelle :

1. Connectez-vous au [portail Azure](https://go.microsoft.com/fwlink/p/?LinkID=525040).

1. Sélectionnez **Tous les services**, puis **DevTest Labs** dans la liste.

1. Sélectionnez le laboratoire souhaité dans la liste des laboratoires.  

1. Sur le volet principal du laboratoire, sélectionnez **Mes machines virtuelles**.
 
1. Dans le volet **Mes machines virtuelles**, sélectionnez la machine virtuelle à partir de laquelle vous souhaitez créer l’image personnalisée.

1. Sur le volet de gestion de la machine virtuelle, sélectionnez **Créer une image personnalisée (VHD)** sous **OPÉRATIONS**.

    :::image type="content" source="./media/devtest-lab-create-template/create-custom-image.png" alt-text="Élément de menu Créer une image personnalisée":::
1. Dans le volet **Image personnalisée**, entrez un nom et une description pour votre image personnalisée. Ces informations s’affichent dans la liste de bases lorsque vous créez une machine virtuelle. L’image personnalisée comprend le disque du système d’exploitation et tous les disques de données attachés à la machine virtuelle.

    :::image type="content" source="./media/devtest-lab-create-template/create-custom-image-blade.png" alt-text="Élément de menu Créer une image personnalisée":::
1. Choisissez si sysprep a été exécuté sur la machine virtuelle. Si le sysprep n’a pas été exécuté sur la machine virtuelle, spécifiez si vous voulez que le sysprep soit exécuté sur la machine virtuelle lors de la création de l’image personnalisée.
1. Cliquez sur **OK** lorsque vous avez terminé de créer l’image personnalisée.

    Après quelques minutes, l’image personnalisée est créée et stockée dans le compte de stockage du laboratoire. Lorsqu’un utilisateur du laboratoire souhaite créer une nouvelle machine virtuelle, l’image est disponible dans la liste des images de base.

    :::image type="content" source="./media/devtest-lab-create-template/custom-image-available-as-base.png" alt-text="Élément de menu Créer une image personnalisée":::

## <a name="related-blog-posts"></a>Billets de blog connexes

- [Custom images or formulas? (Images personnalisées ou formules ?)](./devtest-lab-faq.md#blog-post)
- [Copying Custom Images between Azure DevTest Labs (Copie d’images personnalisées entre plusieurs Azure DevTest Labs)](https://www.visualstudiogeeks.com/blog/DevOps/How-To-Move-CustomImages-VHD-Between-AzureDevTestLabs#copying-custom-images-between-azure-devtest-labs)

## <a name="next-steps"></a>Étapes suivantes

- [Ajout d’une machine virtuelle à votre laboratoire](devtest-lab-add-vm.md)

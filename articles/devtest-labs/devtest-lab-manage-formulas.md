---
title: Gérer les formules dans Azure DevTest Labs pour créer des machines virtuelles | Microsoft Docs
description: Cet article explique comment créer une formule à partir d’une base (une image personnalisée, une image Place de marché ou une autre formule) ou d’une machine virtuelle existante.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: f012a8c59a8e938334c3e1abc4c7b3ccd0e48d3e
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91308892"
---
# <a name="manage-azure-devtest-labs-formulas"></a>Gérer les formules Azure DevTest Labs

[!INCLUDE [devtest-lab-formula-definition](../../includes/devtest-lab-formula-definition.md)]

Cet article explique comment créer une formule à partir d’une base (une image personnalisée, une image Place de marché ou une autre formule) ou d’une machine virtuelle existante. Cet article vous guide aussi dans la gestion des formules existantes.

## <a name="create-a-formula"></a>Créer une formule
Toute personne disposant des autorisations *Utilisateurs* de DevTest Labs est en mesure de créer des machines virtuelles en utilisant une formule comme base. Il existe deux façons de créer des formules : 

* À partir d’une base : quand vous voulez définir toutes les caractéristiques de la formule.
* À partir d’une machine virtuelle de laboratoire existante : quand vous voulez créer une formule à partir des paramètres d’une machine virtuelle existante.

Pour plus d’informations sur l’ajout des utilisateurs et des autorisations, consultez [Ajouter des propriétaires et des utilisateurs dans Azure DevTest Labs](./devtest-lab-add-devtest-user.md).

### <a name="create-a-formula-from-a-base"></a>Créer une formule à partir d’une base
Les étapes suivantes vous guideront tout au long du processus de création d’une formule à partir d’une image personnalisée, image Marketplace ou autre formule.

1. Connectez-vous au [portail Azure](https://portal.azure.com).

2. Sélectionnez **Tous les services**, puis **DevTest Labs** dans la liste.

3. Sélectionnez le laboratoire souhaité dans la liste des laboratoires.  

4. Dans la page du laboratoire, sélectionnez **Formules (bases réutilisables)** dans le menu de gauche.
5. Dans la page **Formules**, sélectionnez **+ Ajouter**.
   
    ![Ajouter une formule](./media/devtest-lab-create-formulas/add-formula.png)
6. Dans la page **Choisir une base**, sélectionnez la base (image personnalisée, image de la Place de marché ou image de galerie d’images partagées) à partir de laquelle vous voulez créer la formule.

    :::image type="content" source="./media/devtest-lab-create-formulas/select-base.png" alt-text="Sélectionnez l’image de base":::
1. Sur l’onglet **Paramètres de base** de la page **Créer une formule**, spécifiez les valeurs suivantes :
   
    * **Nom de la formule** : entrez un nom pour votre formule. Cette valeur s’affiche dans la liste des images de base lorsque vous créez une machine virtuelle. Le nom est validé au fur et à mesure que vous le tapez. S’il n’est pas valide, un message indique les exigences d’un nom valide.
    - Entrez une **Description** facultative pour la formule. 
    * **Nom d’utilisateur** - Entrez un nom d’utilisateur qui obtient les privilèges d’administrateur.
    * **Mot de passe** - Entrez, ou sélectionnez dans la liste déroulante, une valeur qui est associée au secret (mot de passe) que vous souhaitez utiliser pour l’utilisateur spécifié. Pour en savoir plus sur l’enregistrement des secrets dans un coffre de clés et leur utilisation lors de la création des ressources de laboratoire, consultez [Stocker les secrets dans Azure Key Vault](devtest-lab-store-secrets-in-key-vault.md).

        Sélectionnez **Utiliser un secret enregistré** si vous souhaitez utiliser un secret à partir d’Azure Key Vault au lieu d’un mot de passe. 
    * **Taille de machine virtuelle** : sélectionnez **Changer la taille** pour changer la taille de la machine virtuelle. 
    - **Type de disque du système d’exploitation** : sélectionnez le type de disque que vous souhaitez utiliser (Disque dur Standard, Disque SSD Standard ou Disque SSD Premium).
    * **Artefacts** : sélectionnez la page **Ajouter ou supprimer des artefacts**, dans laquelle vous pouvez sélectionner et configurer les artefacts que vous souhaitez ajouter à l’image de base. Pour plus d’informations sur les artefacts, consultez [Créer des artefacts personnalisés pour votre machine virtuelle Azure DevTest Labs](devtest-lab-artifact-author.md).

        ![Page Paramètres de base](./media/devtest-lab-create-formulas/basic-settings.png)
8. Basculez vers l’onglet **Paramètres avancés** et spécifiez les valeurs suivantes :
    - **Réseau virtuel** : pour changer de réseau virtuel, sélectionnez **Changer le réseau virtuel**. 
    - **Sous-réseau** : pour changer de sous-réseau, sélectionnez **Changer de sous-réseau**. 
    - **Configuration de l’adresse IP** - Spécifiez si vous souhaitez que les adresses IP soient partagées, privées ou publiques. Pour plus d’informations sur les adresses IP partagées, consultez [Comprendre les adresses IP partagées dans Azure DevTest Labs](./devtest-lab-shared-ip.md).
    - **Date et heure d’expiration** : vous ne pouvez pas modifier ce champ. 
    - **Make this machine claimable** (Rendre cette machine exigible) - Lorsqu’une machine est « exigible », cela signifie qu’aucune propriété ne lui sera affectée au moment de la création. En revanche, les utilisateurs du laboratoire seront en mesure de prendre possession de la machine (de la « revendiquer ») dans la page du laboratoire.  

        ![Capture d’écran des Paramètres avancés de la page « Créer une formule (base réutilisable) »](./media/devtest-lab-create-formulas/advanced-settings.png)
    - Si vous avez sélectionné une image de la galerie Shared Image Gallery comme base, vous voyez également le champ **version de l’image** qui vous permet de sélectionner la version de l’image à partir de la galerie que vous souhaitez utiliser comme base. 

        ![Page Paramètres avancés](./media/devtest-lab-create-formulas/advanced-settings-shared-image-gallery.png)
8. Sélectionnez **Envoyer** pour créer la formule.

9. Une fois la formule créée, elle s’affiche dans la liste de la page **Formules**.

### <a name="create-a-formula-from-a-vm"></a>Créer une formule depuis une machine virtuelle
Les étapes suivantes vous guident dans le processus de création d’une formule à partir d’une machine virtuelle existante. 

> [!NOTE]
> Pour créer une formule à partir d’une machine virtuelle, la machine virtuelle doit avoir été créée après le 30 mars 2016. 
> 
> 

1. Connectez-vous au [portail Azure](https://go.microsoft.com/fwlink/p/?LinkID=525040).
2. Sélectionnez **Tous les services**, puis **DevTest Labs** dans la liste.
3. Sélectionnez le laboratoire souhaité dans la liste des laboratoires.  
4. Dans la page **Vue d’ensemble** du laboratoire, sélectionnez la machine virtuelle à partir de laquelle vous souhaitez créer la formule.
   
    ![Machines virtuelles de labo](./media/devtest-lab-create-formulas/my-vms.png)
5. Dans la page de la machine virtuelle, cliquez sur **Créer une formule (base réutilisable)** .
   
    ![Créer une formule](./media/devtest-lab-create-formulas/create-formula-menu.png)
6. Dans la page **Créer une formule**, entrez un **Nom** et une **Description** pour votre nouvelle formule.
   
    ![Page Créer une formule](./media/devtest-lab-create-formulas/create-formula-blade.png)
7. Sélectionnez **OK** pour créer la formule.

## <a name="modify-a-formula"></a>Modifier une formule
Pour modifier une formule, procédez comme suit :

1. Connectez-vous au [portail Azure](https://go.microsoft.com/fwlink/p/?LinkID=525040).
2. Sélectionnez **Tous les services**, puis **DevTest Labs** dans la liste.
3. Sélectionnez le laboratoire souhaité dans la liste des laboratoires.  
4. Dans la page du laboratoire, sélectionnez **Formules (bases réutilisables)** .
   
    ![Capture d’écran de la page du labo avec « Formules (bases réutilisables) » sélectionné](./media/devtest-lab-manage-formulas/lab-settings-formulas.png)
5. Dans la page des **formules de laboratoire**, sélectionnez la formule à modifier.
6. Dans la page **Mettre à jour la formule**, effectuez les modifications souhaitées, puis sélectionnez **Mettre à jour**.

## <a name="delete-a-formula"></a>Supprimer une formule
Pour supprimer une formule, procédez comme suit :

1. Connectez-vous au [portail Azure](https://go.microsoft.com/fwlink/p/?LinkID=525040).
2. Sélectionnez **Tous les services**, puis **DevTest Labs** dans la liste.
3. Sélectionnez le laboratoire souhaité dans la liste des laboratoires.  
4. Dans la page **Paramètres** du laboratoire, sélectionnez **Formules**.
   
    ![Menu de formule](./media/devtest-lab-manage-formulas/lab-settings-formulas.png)
5. Dans la page des **formules de laboratoire**, sélectionnez les points de suspension à droite de la formule à supprimer.
   
    ![Capture d’écran de la page « Formules du labo » avec les points de suspension des options de formule encadrés](./media/devtest-lab-manage-formulas/lab-formulas-blade.png)
6. Dans le menu contextuel de la formule, sélectionnez **Supprimer**.
   
    ![Menu contextuel de la formule](./media/devtest-lab-manage-formulas/formula-delete-context-menu.png)
7. sélectionnez **Oui** dans la boîte de dialogue de confirmation de la suppression.

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="related-blog-posts"></a>Billets de blog connexes
* [Custom images or formulas? (Images personnalisées ou formules ?)](devtest-lab-faq.md#what-is-the-difference-between-a-custom-image-and-a-formula)

## <a name="next-steps"></a>Étapes suivantes
Quand vous avez créé une formule à utiliser lors de la création d’une machine virtuelle, l’étape suivante consiste à [ajouter une machine virtuelle à votre laboratoire](devtest-lab-add-vm.md).


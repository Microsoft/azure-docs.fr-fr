---
title: Configurer les propriétés d’une offre de machine virtuelle sur Place de marché Azure
description: Découvrez comment configurer les propriétés d’une offre de machine virtuelle sur Place de marché Azure.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: emuench
ms.author: mingshen
ms.date: 10/19/2020
ms.openlocfilehash: 281553782774c31ec8cfaf614542fd739c4d3dd9
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "94629527"
---
# <a name="how-to-configure-virtual-machine-offer-properties"></a>Comment configurer les propriétés d’une offre de machine virtuelle

Dans la page **Propriétés** (à partir du menu de navigation de gauche dans Espace partenaires), définissez les catégories utilisées pour regrouper votre offre de machine virtuelle sur Place de marché Azure, la version de votre application et les contrats légaux associés.

## <a name="select-a-category"></a>Sélectionner une catégorie

Sélectionnez des catégories et des sous-catégories pour placer votre offre dans les zones de recherche appropriées sur Place de marché Azure. Veillez à décrire ultérieurement dans la description de l’offre la façon dont votre offre prend en charge ces catégories.

- Sélectionnez une catégorie principale.
- Pour ajouter une deuxième catégorie facultative (secondaire), sélectionnez le lien **+ Catégories** .
- Sélectionnez jusqu’à deux sous-catégories pour la catégorie principale et/ou secondaire. Si aucune sous-catégorie n’est applicable à votre offre, sélectionnez **Non applicable**. Utilisez Ctrl + clic pour sélectionner une deuxième sous-catégorie.

Consultez la liste complète des catégories et sous-catégories dans [Bonnes pratiques pour le référencement des offres](gtm-offer-listing-best-practices.md). Les offres de machines virtuelles apparaissent toujours sous la catégorie **Calcul** sur la Place de marché Azure.

## <a name="provide-terms-and-conditions"></a>Entrer des conditions d’utilisation

Sous **Juridique**, entrez les conditions générales de votre offre. Deux options s'offrent à vous :

- [Utiliser le contrat standard avec des avenants facultatifs](#use-the-standard-contract)
- [Utiliser vos propres conditions générales](#use-your-own-terms-and-conditions)

Pour découvrir le contrat standard et les avenants facultatifs, consultez [Contrat standard pour la place de marché commerciale de Microsoft](standard-contract.md). Vous pouvez télécharger le fichier PDF [Contrat Standard](https://go.microsoft.com/fwlink/?linkid=2041178) (assurez-vous que le bloqueur de fenêtres publicitaires est désactivé).

### <a name="use-the-standard-contract"></a>Utiliser le contrat standard

Pour simplifier le processus d’approvisionnement pour les clients et réduire la complexité juridique pour les éditeurs de logiciels, Microsoft propose un contrat Standard que vous pouvez utiliser pour vos offres dans la place de marché commerciale. Lorsque vous proposez votre logiciel dans le cadre du contrat standard, les clients ne doivent lire et accepter ce contrat qu’une seule fois, et vous n’avez pas besoin d’élaborer des conditions générales personnalisées.

1. Activez la case à cocher **Utiliser le contrat Standard pour le marketplace commercial de Microsoft**.

   ![Illustre la case à cocher Utiliser le contrat standard pour la place de marché commerciale de Microsoft.](partner-center-portal/media/use-standard-contract.png)

1. Dans la boîte de dialogue **Confirmation**, sélectionnez **Accepter**. Selon la taille de votre écran, vous devrez peut-être faire défiler la page vers le haut pour la voir.
1. Sélectionnez **Enregistrer le brouillon** avant de continuer.

   > [!NOTE]
   > Après avoir publié une offre à l’aide du contrat standard pour la place de marché commerciale, vous ne pouvez pas utiliser vos propres conditions générales. Proposez votre solution avec le contrat standard assorti d’avenants facultatifs ou vos propres conditions générales.

#### <a name="add-amendments-to-the-standard-contract-optional"></a>Ajouter des avenants au contrat standard (facultatif)

Deux types d’avenants sont disponibles : *universel* et *personnalisé*.

##### <a name="add-universal-amendment-terms"></a>Ajouter des termes d’avenant universel

Dans la zone **Termes d’avenant universel au contrat standard pour la place de marché commerciale de Microsoft**, entrez les termes de votre avenant universel. Vous pouvez entrer un nombre illimité de caractères dans cette zone. Ces termes sont affichés pour les clients dans AppSource, la Place de marché Azure et/ou le portail Azure pendant le processus de découverte et d’achat.

##### <a name="add-one-or-more-custom-amendments"></a>Ajouter une ou plusieurs avenants personnalisés

1. Sous **Termes d’avenants personnalisés au contrat standard pour la place de marché commerciale de Microsoft**, sélectionnez le lien **Ajouter des termes d’avenant personnalisé (max 10)** .
2. Saisissez vos **conditions d’amendement personnalisées** dans la zone concernée.
3. Entrez l’**ID de locataire** dans la zone. Seuls les clients associés aux ID de locataire que vous spécifiez pour ces termes personnalisés les verront dans le flux d’achat de l’offre dans le portail Azure.

   > [!TIP]
   > L’ID d’un locataire identifie votre client dans Azure. Vous pouvez demander cet ID à votre client et le trouver en accédant à [ **https://portal.azure.com**](https://portal.azure.com) > **Azure Active Directory** > **Propriétés**. La valeur d’ID d’annuaire est l’ID de locataire (par exemple, `50c464d3-4930-494c-963c-1e951d15360e`). Vous pouvez également Rechercher l’ID de locataire de l’organisation de votre client à l’aide de son URL de nom de domaine sur [Qu’est-ce que mon Microsoft Azure et mon ID de locataire Office 365 ?](https://www.whatismytenantid.com/).

4. Si vous le souhaitez, entrez une **description** conviviale de l’ID de locataire. Cette description vous aide à identifier le client que vous ciblez avec l’avenant.
5. Pour ajouter un autre ID de locataire, sélectionnez le lien **Ajouter l’ID de locataire d’un client (10 max.)** , puis répétez les étapes 3 et 4. Vous pouvez ajouter jusqu’à 20 ID de locataire.
6. Pour ajouter un autre terme d’avenant, répétez les étapes 1 à 5. Vous pouvez fournir jusqu’à dix conditions d’amendement personnalisées par offre.
7. Sélectionnez **Enregistrer le brouillon** avant de continuer.

### <a name="use-your-own-terms-and-conditions"></a>Utiliser vos propres conditions générales

Vous pouvez également utiliser vos propres conditions générales au lieu du contrat standard. Les clients doivent accepter ces conditions avant de pouvoir essayer votre application.

1. Sous **Juridique**, désactivez la case à cocher **Utiliser le contrat Standard pour le marketplace commercial de Microsoft**.
1. Dans la zone **Conditions générales**, entrez jusqu’à 10 000 caractères de texte.

   > [!NOTE]
   > Si vous avez besoin d’une description plus longue, entrez une adresse web unique pointant vers l’emplacement où se trouvent vos conditions générales. Elle sera présentée aux clients sous forme de lien actif.

1. Si vous effectuez des modifications, sélectionnez **Enregistrer le brouillon** avant de passer à l’onglet suivant du menu de navigation gauche **Référencement de l’offre**.

## <a name="next-steps"></a>Étapes suivantes

- [Configurer l’annonce d’une offre de machine virtuelle](azure-vm-create-listing.md)

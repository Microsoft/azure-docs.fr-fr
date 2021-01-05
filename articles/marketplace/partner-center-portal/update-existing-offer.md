---
title: Mettre à jour une offre existante – Marketplace commercial de Microsoft
description: Comment mettre à jour une offre ou un plan de marketplace commerciale, synchroniser des audiences privées et supprimer une offre.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: trkeya
ms.author: trkeya
ms.date: 10/27/2020
ms.custom: contperf-fy21q2
ms.openlocfilehash: 26ceee828bae7ea3fe2d78ec1393f308556b8401
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033355"
---
# <a name="update-existing-offers-in-the-commercial-marketplace"></a>Mettre à jour des offres existantes dans le marketplace commercial

Cet article explique comment mettre à jour les offres et les plans existants et comment supprimer une offre du marketplace commercial. Vous pouvez afficher vos offres dans l’onglet **Vue d’ensemble** du [portail du marketplace commercial](https://partner.microsoft.com/dashboard/commercial-marketplace/offers) dans Espace partenaires.

## <a name="update-a-published-offer"></a>Mettre à jour une offre publiée

Procédez comme suit pour mettre à jour une offre qui a été publiée en préversion ou qui est active.

1. Sélectionnez le nom de l’offre à mettre à jour. L’état de l’offre peut être répertorié comme suit : **Préversion**, **Activée**, **Publication en cours**, **Brouillon**, **Attention requise** ou **Non disponible** (si vous avez précédemment choisi d’arrêter de vendre l’offre). Une fois sélectionnée, la page **Vue d’ensemble de l’offre** de cette offre s’ouvre.
1. Sélectionnez la page de l’offre que vous souhaitez mettre à jour, par exemple **Propriétés**, **Liste des offres** ou **Préversion** (ou sélectionnez **Mise à jour** dans la carte appropriée sur la page **Vue d’ensemble de l’offre**).
1. Apportez vos modifications, puis sélectionnez **Enregistrer le brouillon**. Répétez ce processus jusqu'à ce que toutes vos modifications soient terminées.
1. Passez en revue les modifications apportées à la page **[Comparer](#compare-changes-to-your-offer)** .
1. Lorsque vous êtes prêt à publier votre offre mise à jour, sélectionnez **Passer en revue et publier** à partir de n’importe quelle page. La page **Passer en revue et publier** s’ouvre. Sur cette page, vous verrez l’état d’achèvement des sections de l’offre que vous avez mise à jour : 
    - **Modifications non publiées** : la section a été mise à jour et est complète. Toutes les données requises ont été fournies et aucune erreur ne s’est glissée dans les mises à jour.
    - **Caractère incomplet** : les mises à jour de la section ont entraîné des erreurs qui doivent être corrigées, ou qui nécessitent plus d’informations.
2. Sélectionnez **Publier** pour envoyer l’offre mise à jour en vue de sa publication. Votre offre passera ensuite par les [étapes standard de validation et de publication](../review-publish-offer.md#validation-and-publishing-steps).

> [!IMPORTANT]
> Vous devez examiner votre préversion de l’offre une fois qu’elle est disponible et sélectionner **Démarrer** pour publier votre offre mise à jour à l’attention de votre audience ciblée (publique ou privée).

## <a name="add-a-plan-to-an-existing-offer"></a>Ajouter un plan à une offre existante

Procédez comme suit pour ajouter un plan à une offre que vous avez déjà publiée.

1. Avec la page **Vue d’ensemble de l’offre** pour votre offre existante ouverte, allez à la page **Vue d’ensemble du plan** et sélectionnez **Créer un nouveau plan**.
1. Créez un nouveau plan en suivant les [instructions](../plans-pricing.md) à l’aide du **modèle de tarification des plans existant**.
1. Sélectionnez **Enregistrer le brouillon** après avoir changer le nom du plan.
1. Sélectionnez **Publier** une fois que vous êtes prêt à publier vos mises à jour. La page **[Examiner et publier](../review-publish-offer.md)** s’ouvre et fournit un état d’achèvement pour vos mises à jour.

## <a name="update-a-plan-for-an-existing-offer"></a>Mettre à jour un plan pour une offre existante

Procédez comme suit pour apporter des modifications au plan d’une offre que vous avez déjà publiée.

1. Avec la page **Vue d’ensemble de l’offre** ouverte pour votre offre existante, choisissez le plan que vous souhaitez modifier. Si le plan n’est pas accessible à partir de la liste **Vue d’ensemble du plan**, sélectionnez **Voir tous les plans**.
1. Sélectionnez le **nom**, le **modèle de tarification** ou la **disponibilité** du plan. *Actuellement, les plans sont disponibles uniquement en anglais (États-Unis)* .
1. Sélectionnez **Enregistrer le brouillon** après avoir apporté les modifications nécessaires au nom du plan, à sa description ou à sa disponibilité pour l’audience.
1. Si vous êtes prêt à publier vos mises à jour, sélectionnez **Passer en revue et publier**. La page **[Examiner et publier](../review-publish-offer.md)** s’ouvre et fournit un état d’achèvement pour vos mises à jour.
1. Sélectionnez **Publier** pour envoyer le plan mis à jour en vue de sa publication. Nous vous enverrons un e-mail quand une préversion de l’offre mise à jour sera disponible pour examen et approbation.

## <a name="offer-a-virtual-machine-plan-at-a-new-price"></a>Offrir un plan d’ordinateur virtuel à un nouveau prix

Après la publication d’un plan d’ordinateur virtuel, son prix ne peut pas être modifié. Pour offrir le même plan à un prix différent, vous devez masquer celui-ci et en créer un nouveau avec le tarif mis à jour. Tout d’abord, masquez le plan avec le prix que vous souhaitez modifier :

1. Avec la page **Vue d’ensemble de l’offre** ouverte pour votre offre existante, choisissez le plan que vous souhaitez modifier. Si le plan n’est pas accessible à partir de la liste **Vue d’ensemble du plan**, sélectionnez **Voir tous les plans**.
1. Activez la case à cocher **Masquer le plan**. Enregistrez le brouillon avant de continuer.

Maintenant que vous avez masqué le plan avec l’ancien prix, créez une copie de ce plan avec le tarif mis à jour :

1. Dans l’Espace partenaires, revenez à **Vue d’ensemble du plan**.
2. Sélectionnez **Créer un plan**. Entrez un **ID de plan** et un **Nom de plan**, puis sélectionnez **Créer**.
1. Pour réutiliser la configuration technique du plan que vous avez masqué, activez la case à cocher **Réutiliser la configuration technique**. Pour en savoir plus, consultez [Créer des plans pour une offre de machine virtuelle](../azure-vm-create-plans.md).
    > [!IMPORTANT]
    > Si vous sélectionnez **Ce plan réutilise la configuration technique d’un autre plan**, vous ne pourrez pas arrêter la vente du plan parent ultérieurement. N’utilisez pas cette option si vous souhaitez arrêter la vente du plan parent.
3. Renseignez toutes les sections requises pour le nouveau plan, y compris le nouveau prix.
1. Sélectionnez **Enregistrer le brouillon**.
1. Une fois que vous avez terminé toutes les sections requises pour le nouveau plan, sélectionnez **Vérifier et publier**. Cela permet d’envoyer votre offre à des fins de révision et de publication. Lisez [Consulter et publier une offre sur la place de marché commerciale](../review-publish-offer.md) pour plus d’informations.

## <a name="sync-private-plan-audiences"></a>Synchroniser les audiences de plan privées

Si votre offre inclut un ou plusieurs plans configurés pour être uniquement disponibles pour une audience privée limitée, il est possible de mettre à jour uniquement l’audience pouvant accéder à ce plan privé sans publier d’autres modifications de l’offre. 

Pour mettre à jour et synchroniser l’audience privée pour votre ou vos plans :

- Modifiez l’audience dans un ou plusieurs plans privés à l’aide du bouton + **Ajouter ID** ou **Importer les clients (csv)** , puis enregistrez les modifications.
- Sélectionnez **Synchroniser l’audience privé** à partir de la page **Vue d’ensemble du plan**.

**Synchroniser l’audience privé** publiera uniquement les modifications apportées à votre audience privée, sans publier d’autres mises à jour éventuellement apportées au brouillon de l’offre.

## <a name="compare-changes-to-your-offer"></a>Comparer les modifications apportées à votre offre

Avant de publier les mises à jour de votre offre en direct ou en [préversion](#compare-changes-to-a-preview-offer), vous pouvez auditer les modifications enregistrées dans la page **Comparer**. Vous pouvez accéder à la page **Comparer** dans le coin supérieur droit de n’importe quelle page d’offre, par exemple la page **Propriétés** ou **Liste des offres**. La page **Comparer** affiche des versions côte à côte des modifications enregistrées de cette offre et de l’offre publiée sur le marketplace.

- Vous pouvez utiliser **Comparer** à tout moment pendant le processus de modification.
- Sélectionnez un champ dans la page **Comparer** pour accéder à la valeur que vous souhaitez modifier.
- Pour voir les valeurs de tous les champs, même les champs qui ne sont pas mis à jour, sélectionnez le filtre **Tous les champs**. Vous pouvez modifier les filtres dans ces champs en sélectionnant **Modified fields** (Champs modifiés), puis l’un des filtres ci-dessous :
    - Le filtre **Removed values** (Valeurs supprimées) affiche les champs que vous avez publiés et que vous supprimez maintenant complètement.
    - Le filtre **Added values** (Valeurs ajoutées) affiche les champs que vous n’avez pas publiés à l’origine et que vous ajoutez maintenant.
    - Le filtre **Edited values** (Valeurs modifiées) affiche les champs qui ont été publiés, mais dont vous venez de mettre à jour le contenu.

      >[!NOTE]
      > Si l’un de ces filtres n’est pas disponible, cela signifie que vous n’avez pas effectué de mise à jour de ce type.

- Pour voir uniquement les valeurs qui n’ont pas été mises à jour, sélectionnez le filtre **Unchanged fields** (Champs inchangés). Les valeurs de champ indiquées pour la version publiée et brouillon sont les mêmes.

  ![Filtres pour la comparaison des mises à jour de votre offre publiée ou en préversion](./media/compare-changes-marketplace.png)

>[!NOTE]
> Les pages suivantes ne prennent pas en charge **Comparer** pour l’instant :
>- Public du revendeur CSP
>- Configuration technique du test drive
>- Liste de test drives de la Place de marché
>- Co-vente
>- Fichiers supplémentaires

Pensez à republier votre offre après avoir apporté des mises à jour pour que les modifications prennent effet.

### <a name="compare-changes-to-a-preview-offer"></a>Comparer les modifications apportées à une offre en préversion

Si vous avez des modifications en préversion qui ne sont pas en direct, vous pouvez comparer les nouvelles modifications à l’offre de Place de marché en préversion.

1. Sélectionnez **Comparer** dans la barre de commandes de la page.
2. Sélectionnez la liste déroulante **Par** et remplacez **Live offer** (Offre en direct) par **Preview offer** (Offre en préversion). Si votre offre n’est pas encore active, vous ne verrez pas l’option **Offre en direct**.
3. La page **Comparer** fournit des versions côte à côte qui affichent les modifications.

Pensez à republier votre offre après avoir apporté des mises à jour pour que les modifications prennent effet.

## <a name="stop-selling-an-offer-or-plan"></a>Arrêter la vente d’une offre ou d’un plan

Vous pouvez supprimer des listes et des plans d’offres du marketplace commercial de Microsoft, ce qui empêchera les nouveaux clients de les trouver et de les acheter. Les clients ayant déjà acquis l’offre ou le plan peuvent continuer à les utiliser, et ils peuvent les télécharger à nouveau si nécessaire. Toutefois, ils ne recevront pas les mises à jour si vous décidez de republier l’offre ou le plan ultérieurement.

- Pour arrêter la vente d’une offre après sa publication, sélectionnez **Arrêter la vente** à partir de la page **Vue d’ensemble de l’offre**. Dans les quelques heures qui suivent votre confirmation, l’offre n’est plus visible sur le marketplace commercial.

- Pour arrêter de vendre un plan, sélectionnez **Arrêter la vente** à partir de la page **Vue d’ensemble du plan**. L’option d’arrêt d’un plan est uniquement disponible si l’offre contient plus d’un plan. Vous pouvez choisir d’arrêter la vente d’un seul plan sans influer sur les autres plans au sein de votre offre.
     >[!NOTE]
     > Une fois que vous avez confirmé vouloir arrêter la vente du plan, vous devez republier l’offre pour que la modification prenne effet.

Une fois que vous avez cessé de vendre une offre ou un plan, vous les voyez toujours dans Espace partenaires avec un état **Non disponible**. Si vous décidez de référencer ou de vendre à nouveau cette offre ou ce plan, suivez les instructions pour [mettre à jour une offre publiée](#update-a-published-offer). N’oubliez pas que vous devrez **publier** à nouveau l’offre ou le plan après y avoir apporté des modifications.

## <a name="remove-offers-from-existing-customers"></a>Supprimer des offres de clients existants

Pour supprimer des offres de clients existants, [enregistrez une demande de support](https://aka.ms/marketplacepublishersupport). Dans la liste des rubriques du support, sélectionnez **Marketplace commercial** > **Déréférencement, suppression ou arrêt d’une offre ou d’une application** et envoyez la demande. L’équipe de support vous guidera tout au long du processus de suppression de l’offre.

## <a name="next-steps"></a>Étapes suivantes

- [Vérifier l’état de publication de votre offre de place de marché commerciale](../review-publish-offer.md)

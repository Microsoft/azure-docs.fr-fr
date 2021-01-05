---
title: Consultation des recommandations Azure Advisor vous concernant
description: Afficher et filtrer les recommandations Azure Advisor pour réduire le bruit.
ms.topic: article
ms.date: 04/03/2019
ms.openlocfilehash: 43bdaba7d774bba8857a6eb3ef296d7ab8e1c264
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90986870"
---
# <a name="view-azure-advisor-recommendations-that-matter-to-you"></a>Consultation des recommandations Azure Advisor vous concernant

Azure Advisor fournit des recommandations pour vous aider à optimiser vos déploiements Azure. Dans Advisor, vous avez accès à quelques fonctionnalités qui vous aident à affiner vos recommandations pour ne voir que celles qui vous intéressent.

## <a name="configure-subscriptions-and-resource-groups"></a>Configurer les abonnements et les groupes de ressources

Advisor vous offre la possibilité de sélectionner les abonnements et les groupes de ressources qui vous intéressent, vous et votre organisation. Vous voyez uniquement les recommandations pour les abonnements et les groupes de ressources que vous sélectionnez. Par défaut, tous les éléments sont sélectionnés. Les paramètres de configuration s’appliquent à l’abonnement ou au groupe de ressources : les mêmes paramètres s’appliquent donc à tous les utilisateurs ayant accès à ce groupe de ressources ou à cet abonnement. Les paramètres de configuration peuvent être modifiés dans le portail Azure ou par programmation.

Pour apporter des modifications dans le portail Azure :

1. Ouvrez [Azure Advisor](https://aka.ms/azureadvisordashboard) dans le portail Azure.

1. Sélectionnez **Configuration** dans le menu.

   ![Menu de configuration d’Advisor](./media/view-recommendations/configuration.png)

1. Activez la case à cocher dans la colonne **Inclure** pour les abonnements ou les groupes de ressources pour lesquels vous souhaitez recevoir des recommandations d’Advisor. Si cette case à cocher est désactivée, vous ne disposez peut-être pas de l’autorisation nécessaire pour modifier la configuration sur cet abonnement ou ce groupe de ressources. En savoir plus sur les [autorisations dans Azure Advisor](permissions.md).

1. Cliquez sur **Appliquer** en bas après avoir apporté une modification.

## <a name="filtering-your-view-in-the-azure-portal"></a>Filtrage de l’affichage dans le portail Azure

Les paramètres de configuration restent actifs jusqu’à ce qu’ils soient modifiés. Si vous souhaitez limiter l’affichage des recommandations sur un seule affichage, vous pouvez utiliser les listes déroulantes en haut du panneau d’Advisor. Dans les panneaux Vue d’ensemble, Haute disponibilité, Sécurité, Performances, Coût et Recommandation, vous pouvez sélectionner les abonnements, les types de ressources et le statut des recommandations que vous souhaitez voir.

   :::image type="content" source="./media/view-recommendations/filtering.png" alt-text="Capture d’écran d’Azure Advisor montrant des options de filtrage.":::

## <a name="dismissing-and-postponing-recommendations"></a>Ignorer et reporter des recommandations

Azure Advisor vous permet d’ignorer ou de reporter des recommandations sur une ressource. Si vous ignorez une recommandation, vous ne la verrez plus, sauf si vous l’activez manuellement. Toutefois, le fait de reporter une recommandation vous permet de spécifier une durée après laquelle la recommandation est automatiquement réactivée. Vous pouvez reporter les recommandations dans le portail Azure ou par programme.

### <a name="postpone-a-single-recommendation-in-the-azure-portal"></a>Reporter une seule recommandation dans le portail Azure 

1. Ouvrez [Azure Advisor](https://aka.ms/azureadvisordashboard) dans le portail Azure.
1. Sélectionner une catégorie de recommandations pour voir vos recommandations
1. Sélectionnez une recommandation dans la liste des recommandations.
1. Sélectionner Reporter ou Ignorer pour la recommandation que vous souhaitez reporter ou ignorer

     :::image type="content" source="./media/view-recommendations/postpone-dismiss.png" alt-text="Capture d’écran d’Azure Advisor montrant des options de filtrage.":::

### <a name="postpone-or-dismiss-a-multiple-recommendations-in-the-azure-portal"></a>Reporter ou ignorer plusieurs recommandations dans le portail Azure

1. Ouvrez [Azure Advisor](https://aka.ms/azureadvisordashboard) dans le portail Azure.
1. Sélectionnez une catégorie de recommandations pour voir vos recommandations.
1. Sélectionnez une recommandation dans la liste des recommandations.
1. Sélectionnez la case à cocher à gauche de la ligne pour toutes les ressources pour lesquelles vous souhaitez reporter ou ignorer la recommandation.
1. Sélectionnez **Reporter** ou **Ignorer** en haut à gauche de la table.

     :::image type="content" source="./media/view-recommendations/postpone-dismiss-multiple.png" alt-text="Capture d’écran d’Azure Advisor montrant des options de filtrage.":::

> [!NOTE]
> Vous devez disposer d’une autorisation de contributeur ou de propriétaire pour pouvoir ignorer ou reporter une recommandation. En savoir plus sur les autorisations dans Azure Advisor.

> [!NOTE]
> Si les zones de sélection sont désactivées, les recommandations sont peut-être encore en cours de chargement. Attendez que toutes les recommandations soient chargées avant d’essayer de les reporter ou de les ignorer.

### <a name="reactivate-a-postponed-or-dismissed-recommendation"></a>Réactiver une recommandation reportée ou ignorée

Vous pouvez activer une recommandation qui a été reporté ou ignorée. Cette action peut être effectuée dans le portail Azure ou par programme. Dans le portail Azure :

1. Ouvrez [Azure Advisor](https://aka.ms/azureadvisordashboard) dans le portail Azure.

1. Modifiez le filtre dans le panneau de vue d’ensemble des recommandations **reportées**. Advisor affiche alors les recommandations reportées ou ignorées.

    :::image type="content" source="./media/view-recommendations/activate-postponed.png" alt-text="Capture d’écran d’Azure Advisor montrant des options de filtrage.":::

1. Sélectionnez une catégorie pour voir les recommandations **Reportée** et **Ignorée**.

1. Sélectionnez une recommandation dans la liste des recommandations. Cette opération ouvre des recommandations avec l’onglet **Reportée et ignorée** déjà choisi, afin de montrer les ressources pour lesquelles cette recommandation a été reporté ou ignorée.

1. Cliquez sur **Activer** à la fin de la ligne. Une fois que vous cliquez dessus, la recommandation est active pour cette ressource et elle est donc supprimée. La recommandation est désormais visible dans l’onglet **Actives**.
 
     :::image type="content" source="./media/view-recommendations/activate-postponed-2.png" alt-text="Capture d’écran d’Azure Advisor montrant des options de filtrage.":::

## <a name="next-steps"></a>Étapes suivantes

Cet article explique comment vous pouvez consulter les recommandations qui vous intéressent dans Azure Advisor. Pour en savoir plus sur Advisor, consultez les ressources suivantes : 

- [Présentation d’Azure Advisor](advisor-overview.md)
- [Prise en main d’Advisor](advisor-get-started.md)
- [Autorisations dans Azure Advisor](permissions.md)




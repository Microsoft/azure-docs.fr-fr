---
title: Prise en main d’Azure Advisor
description: Prise en main du conseiller Azure.
ms.topic: article
ms.date: 02/01/2019
ms.openlocfilehash: e91049077502a6c0eedf0a4f979c073690c214da
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85117860"
---
# <a name="get-started-with-azure-advisor"></a>Prise en main d’Azure Advisor

Découvrez comment accéder à Advisor à l’aide du portail Azure, obtenir des recommandations et implémenter ces dernières.

> [!NOTE]
> Azure Advisor s’exécute automatiquement en arrière-plan pour rechercher les ressources nouvellement créées. L’obtention de recommandations sur ces ressources peut prendre jusqu’à 24 heures.

## <a name="get-recommendations"></a>Obtention de recommandations

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Dans le volet gauche, cliquez sur **Advisor**.  Si vous ne voyez pas Advisor dans le volet gauche, cliquez sur **Tous les services**.  Dans le volet du menu de services, sous **Surveillance et gestion**, cliquez sur **Advisor**. Le tableau de bord Advisor s’affiche.

   ![Accéder au conseiller Azure avec le portail Azure](./media/advisor-get-started/advisor-portal-menu.png) 

1. Le tableau de bord Advisor présente un résumé de vos recommandations pour tous les abonnements sélectionnés.  Vous pouvez choisir les abonnements pour lesquels afficher les recommandations à l’aide de la liste déroulante de filtrage des abonnements.

1. Pour obtenir des recommandations pour une catégorie spécifique, cliquez sur un des onglets : **Fiabilité**, **Sécurité**, **Performances** ou **Coût**. 

   ![Tableau de bord du conseiller Azure](./media/advisor-overview/advisor-dashboard.png)

## <a name="get-recommendation-details-and-implement-a-solution"></a>Obtenir les recommandations détaillées et implémenter une solution

Vous pouvez sélectionner une recommandation dans Advisor pour afficher des détails supplémentaires, tels que les actions de la recommandation et les ressources affectées, et pour implémenter la solution associée à la recommandation.  

1. Connectez-vous au [portail Azure](https://portal.azure.com), puis ouvrez [Advisor](https://aka.ms/azureadvisordashboard).

1. Sélectionnez une catégorie de recommandations pour afficher la liste des recommandations de cette catégorie, ou sélectionnez l’onglet **Tout** pour afficher toutes vos recommandations.

1. Cliquez sur une recommandation que vous souhaitez examiner en détail.

1. Prenez connaissance des informations relatives à la recommandation et aux ressources auxquelles celle-ci s’applique.

1. Cliquez sur **Action recommandée** pour implémenter la recommandation.

## <a name="filter-recommendations"></a>Filtrer les recommandations

Vous pouvez filtrer les recommandations pour accéder aux informations qui vous intéressent le plus.  Vous pouvez filtrer par abonnement, par type de ressource ou par état de recommandation.  

1. Connectez-vous au [portail Azure](https://portal.azure.com), puis ouvrez [Advisor](https://aka.ms/azureadvisordashboard).

1. Utilisez les menus déroulants du tableau de bord Advisor pour filtrer par abonnement, par type de ressource ou par état de recommandation.

    ![Critères de filtre de recherche d’Advisor](./media/advisor-get-started/advisor-filters.png)

## <a name="postpone-or-dismiss-recommendations"></a>Reporter ou ignorer les recommandations

1. Connectez-vous au [portail Azure](https://portal.azure.com), puis ouvrez [Advisor](https://aka.ms/azureadvisordashboard).

1. Accédez à la recommandation que vous souhaitez reporter ou ignorer.

1. Cliquez sur la recommandation.

1. Cliquez sur **Reporter**. 

1. Spécifiez une période de report ou sélectionnez **Jamais** pour faire disparaître la recommandation.

## <a name="exclude-subscriptions-or-resource-groups"></a>Exclure des abonnements ou des groupes de ressources

Il se peut que vous possédiez des groupes de ressources ou des abonnements pour lesquels vous ne souhaitez pas recevoir de recommandations d’Advisor, tels que les ressources de « test ».  Vous pouvez configurer Advisor de manière à générer des recommandations uniquement pour des abonnements et des groupes de ressources spécifiques.

> [!NOTE]
> Pour inclure un abonnement ou un groupe de ressources dans Advisor ou l’en exclure, vous devez être propriétaire de cet abonnement ou de ce groupe de ressources.  Si vous ne disposez pas des autorisations requises pour un abonnement ou un groupe de ressources, l’option permettant de l’inclure ou de l’exclure est désactivée dans l’interface utilisateur.

1. Connectez-vous au [portail Azure](https://portal.azure.com), puis ouvrez [Advisor](https://aka.ms/azureadvisordashboard).

1. Dans la barre d’action, cliquez sur **Configurer**.

1. Désactivez les abonnements ou les groupes de ressources pour lesquels vous ne souhaitez pas recevoir de recommandations d’Advisor.

    ![Exemple de configuration de ressources dans Advisor](./media/advisor-get-started/advisor-configure-resources.png)

1. Cliquez sur le bouton **Appliquer**.

## <a name="configure-low-usage-vm-recommendation"></a>Configurer la recommandation en cas de faible utilisation de machines virtuelles

Cette procédure configure la règle d’utilisation moyenne du processeur pour la recommandation en cas de faible utilisation de machines virtuelles.

Advisor surveille l’utilisation de votre machine virtuelle pendant 7 jours et identifie les machines virtuelles faiblement utilisées. Les machines virtuelles sont considérées de faible utilisation si l’utilisation de leur processeur est inférieure ou égale à 5 % et l’utilisation de leur réseau est inférieure ou égale à 2 %, ou si la charge de travail actuelle peut être prise en charge par une taille de machine virtuelle plus petite.

Si vous souhaitez être plus « agressif » dans l’identification des machines virtuelles faiblement utilisées, vous pouvez ajuster la règle d’utilisation moyenne du processeur par abonnement.  La règle d’utilisation du processeur peut être définie sur 5 %, 10 %, 15 % ou 20 %.

> [!NOTE]
> Pour ajuster la règle d’utilisation moyenne du processeur aux fins d’identification des machines virtuelles faiblement utilisées, vous devez être *propriétaire* de l’abonnement.  Si vous ne disposez pas des autorisations requises pour un abonnement ou un groupe de ressources, l’option permettant de l’inclure ou de l’exclure est désactivée dans l’interface utilisateur. 

1. Connectez-vous au [portail Azure](https://portal.azure.com), puis ouvrez [Advisor](https://aka.ms/azureadvisordashboard).

1. Dans la barre d’action, cliquez sur **Configurer**.

1. Cliquez sur l’onglet **Règles**.

1. Sélectionnez les abonnements pour lesquels vous voulez ajuster la règle d’utilisation moyenne du processeur, puis cliquez sur **Modifier**.

1. Sélectionnez la valeur d’utilisation moyenne du processeur souhaitée, puis cliquez sur **Appliquer**.

1. Cliquez sur **Actualiser les recommandations** pour mettre à jour vos recommandations existantes avec la nouvelle règle d’utilisation moyenne du processeur. 

   ![Exemple de configuration de règles de recommandation dans Advisor](./media/advisor-get-started/advisor-configure-rules.png)

## <a name="download-recommendations"></a>Télécharger les recommandations

Advisor permet de télécharger un résumé de vos recommandations.  Vous pouvez télécharger vos recommandations sous forme de fichier PDF ou CSV.  En téléchargeant vos recommandations, vous pourrez facilement les partager avec des collègues ou effectuer votre propre analyse sur les données de recommandation.

1. Connectez-vous au [portail Azure](https://portal.azure.com), puis ouvrez [Advisor](https://aka.ms/azureadvisordashboard).

1. Dans la barre d’action, cliquez sur **Télécharger au format CSV** ou **Télécharger au format PDF**.

L’option de téléchargement conserve les filtres que vous avez appliqués au tableau de bord Advisor.  Si vous sélectionnez l’option de téléchargement alors qu’une catégorie de recommandations ou une recommandation spécifique est affichée, le résumé téléchargé inclut uniquement les informations de cette catégorie ou de cette recommandation. 

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur Advisor, consultez les ressources suivantes :

- [Présentation du conseiller Azure](advisor-overview.md)
- [Recommandations d’Advisor en matière de fiabilité](advisor-high-availability-recommendations.md)
- [Recommandations du conseiller en matière de sécurité](advisor-security-recommendations.md)
- [Recommandations du conseiller en matière de performances](advisor-performance-recommendations.md)
- [Recommandations du conseiller en matière de coûts](advisor-cost-recommendations.md)
- [Recommandations d’excellence opérationnelle Advisor](advisor-operational-excellence-recommendations.md)

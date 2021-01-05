---
title: Tableau de bord Résumé de l’analytique de l’Espace partenaires sur la place de marché commerciale
description: Découvrez comment accéder aux graphes, tendances et valeurs des données agrégées qui résument l’activité de la place de marché dans le tableau de bord Résumé de l’Espace partenaires.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 11/09/2020
author: sayantanroy83
ms.author: sroy
ms.openlocfilehash: c02ba12c790d745904d241f121e269aac3ed12f6
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96462991"
---
# <a name="summary-dashboard-in-commercial-marketplace-analytics"></a>Tableau de bord Résumé dans l’analytique de la Place de marché commerciale

Cet article fournit des informations sur le tableau de bord Résumé de l’Espace partenaires. Ce tableau de bord présente les graphes, tendances et valeurs des données agrégées qui résument l’activité de la place de marché pour vos offres.

Pour accéder au tableau de bord Résumé de l’Espace partenaires, sous **Place de marché commerciale**, sélectionnez **[Analyser](https://partner.microsoft.com/dashboard/commercial-marketplace/analytics/summary)**  > **Résumé**.

>[!NOTE]
> Pour avoir des définitions détaillées des termes d’analytique, consultez [Terminologie relative à l’analytique de la Place de marché commerciale et questions fréquentes](./analytics-faq.md).

## <a name="summary-dashboard"></a>Tableau de bord Résumé

Le tableau de bord Résumé présente une vue d’ensemble de la Place de marché Azure et des performances commerciales des offres Microsoft AppSource. Le tableau de bord fournit une vaste vue d’ensemble des éléments suivants :

- Commandes des clients
- Clients
- Utilisation des offres par les clients
- Pages visitées par les clients sur la Place de marché Azure et dans AppSource

## <a name="elements-of-the-summary-dashboard"></a>Éléments du tableau de bord Résumé

Les sections suivantes expliquent comment utiliser le tableau de bord Résumé et lire les données.

### <a name="month-range"></a>Plage de mois

Une sélection par plage de mois est disponible en haut à droite de chaque page. Personnalisez la sortie des graphes de la page **Résumé** en sélectionnant la plage de mois des 3, 6 ou 12 derniers mois, ou une plage de mois personnalisée d’une durée maximale de 12 mois. La plage de mois par défaut (période de calcul) est de six mois.

:::image type="content" source="./media/summary-dashboard/summary-dashboard.png" alt-text="Illustre les options de plage de mois dans le tableau de bord Résumé.":::

> [!NOTE]
> Toutes les métriques dans les widgets de visualisation et les rapports d’exportation respectent la période de calcul sélectionnée par l’utilisateur.

### <a name="orders-widget"></a>Widget Commandes

Le widget Commandes du tableau de bord Résumé** affiche les commandes en cours pour toutes vos offres basées sur Transact. Le widget Commandes affiche le nombre et la tendance de toutes les commandes achetées (à l’exception des commandes annulées) pour la période de calcul sélectionnée. Le pourcentage sous **Commandes** représente la progression pendant la période de calcul sélectionnée.

[![Illustre le widget Commandes dans le tableau de bord Résumé.](./media/summary-dashboard/orders-widget.png)](./media/summary-dashboard/orders-widget.png#lightbox)


Vous pouvez également accéder au rapport Commandes en sélectionnant le lien **Tableau de bord des commandes** dans le coin inférieur gauche du widget.

### <a name="customers-widget"></a>Widget Clients

Le widget **Clients** du tableau de bord **Résumé** affiche le nombre total de clients qui ont acquis vos offres pour la période de calcul sélectionnée. Le widget Clients affiche un nombre et une tendance du nombre total de clients actifs (notamment les clients nouveaux et existants) (à l’exception des clients perdus) pour la période de calcul sélectionnée. Le pourcentage sous **Clients** représente la progression pendant la période de calcul sélectionnée.

[![Illustre le widget Clients dans le tableau de bord Résumé.](./media/summary-dashboard/customers-widget.png)](./media/summary-dashboard/customers-widget.png#lightbox)

Vous pouvez également accéder au rapport détaillé Clients en sélectionnant le lien **Tableau de bord des clients** dans le coin inférieur gauche du widget.

### <a name="usage-widget"></a>Widget Utilisation

Le widget **Utilisation** du tableau de bord **Résumé** représente le nombre total d’heures d’utilisation normalisée et brutes pour toutes les offres de machines virtuelles Azure. Le widget Utilisation affiche le nombre et la tendance du nombre total d’heures d’utilisation pour la période de calcul sélectionnée.

Le tableau Résumé de l’utilisation affiche les heures d’utilisation par les clients de toutes les offres qu’ils ont achetées.

- Les heures d’utilisation normalisée correspondent aux heures d’utilisation normalisées pour prendre en compte le nombre de cœurs de machines virtuelles ([nombre de cœurs de machines virtuelles] x [heures d’utilisation brute]). Les machines virtuelles désignées sous le nom de « SHAREDCORE » utilisent 1/6 (ou 0,1666) comme multiplicateur [nombre de cœurs de machines virtuelles].
- Les heures d’utilisation brute correspondent au nombre d’heures pendant lesquelles les machines virtuelles ont été utilisées.

Le pourcentage sous le nombre total d’heures d’utilisation représente la progression des heures d’utilisation pendant la période de calcul sélectionnée.

[![Illustre le widget Utilisation dans le tableau de bord Résumé.](./media/summary-dashboard/usage-widget.png)](./media/summary-dashboard/usage-widget.png#lightbox)

Vous pouvez également accéder au rapport Utilisation en sélectionnant le lien **Tableau de bord d’utilisation** dans le coin inférieur gauche du widget.

### <a name="marketplace-insights"></a>Insights sur la Place de marché

Les insights sur la Place de marché montrent le nombre d’utilisateurs qui ont visité les pages de vos offres sur la Place de marché Azure et dans AppSource. **Nombre de visites de page** présente un résumé de l’analytique web de la place de marché commerciale qui permet aux éditeurs de mesurer l’engagement client vis-à-vis de leurs fiches produits respectives sur les magasins en ligne de la place de marché commerciale : Microsoft AppSource et Place de marché Azure Ce widget affiche le nombre et la tendance du nombre total de visites de page pour la période de calcul sélectionnée.

[![Illustre le widget Nombre de visites de page dans le tableau de bord Résumé.](./media/summary-dashboard/page-visit-count.png)](./media/summary-dashboard/page-visit-count.png#lightbox)

Vous pouvez également accéder au rapport Insights sur la Place de marché en sélectionnant le lien **Tableau de bord des insights sur la Place de marché** dans le coin inférieur gauche du widget.

### <a name="geographical-spread"></a>Expansion géographique

Pour la période de calcul sélectionnée, le carte thermique affiche le nombre total de clients, de commandes et d’heures d’utilisation normalisée par rapport à la dimension géographique.

:::image type="content" source="./media/summary-dashboard/geo-spread.png" alt-text="Illustre le widget Expansion dans les pays dans le tableau de bord Résumé.":::

Notez les points suivants :

- Vous pouvez déplacer la carte pour afficher l'emplacement exact.
- Vous pouvez zoomer sur un emplacement spécifique.
- La carte thermique comporte une grille supplémentaire présentant les détails du nombre de clients, du nombre de commandes et des heures d’utilisation normalisée pour le lieu spécifié.
- Vous pouvez rechercher et sélectionner un pays/région dans la grille pour zoomer sur l’emplacement dans la carte. Pour revenir à la vue d’origine, sélectionnez le bouton **Accueil** de la carte.

> [!TIP]
> Vous pouvez utiliser l’icône de téléchargement située dans l’angle supérieur droit d’un widget pour télécharger les données. Vous pouvez fournir des commentaires sur chacun des widgets en sélectionnant l’icône « pouce vers le haut » ou « pouce vers le bas ».

## <a name="next-steps"></a>Étapes suivantes

- Pour une vue d’ensemble des rapports d’analytique disponibles dans la Place de marché commerciale, consultez [Accéder aux rapports d’analytique de la Place de marché commerciale dans l’Espace partenaires](./partner-center-portal/analytics.md).
- Pour plus d’informations sur vos commandes dans un format graphique et téléchargeable, consultez [Tableau de bord des commandes dans l’analytique de la Place de marché commerciale](orders-dashboard.md).
- Pour voir les métriques de facturation à l’utilisation des offres de machines virtuelles, consultez [Tableau de bord de l’utilisation dans l’analytique de la place de marché commerciale](usage-dashboard.md).
- Pour obtenir des informations détaillées sur vos clients, y compris des tendances de croissance, consultez [Tableau de bord client dans l’analytique de la Place de marché commerciale](customer-dashboard.md).
- Pour obtenir la liste de vos demandes de téléchargement des 30 derniers jours, consultez [Tableau de bord des téléchargements dans l’analytique de la Place de marché commerciale](./partner-center-portal/downloads-dashboard.md).
- Pour une vue centralisée des commentaires clients sur les offres de la Place de marché Azure et AppSource, consultez [Tableau de bord Analytique des évaluations et avis de l’Espace partenaires](./partner-center-portal/ratings-reviews.md).
- Pour voir les questions fréquentes sur l’analytique de la place de marché commerciale et un dictionnaire complet des termes liés aux données, consultez [Terminologie pour l’analytique de la Place de marché commerciale et questions fréquentes](./analytics-faq.md).
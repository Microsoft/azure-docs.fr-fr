---
title: Tableau de bord Insights de la Place de marché dans les données d’analytique de la Place de marché commerciale
description: Accédez à un résumé de l’analytique web de la Place de marché dans l’Espace partenaires, qui vous permet de mesurer l’engagement client dans Microsoft AppSource et la Place de marché Azure.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
ms.date: 11/09/2020
author: sayantanroy83
ms.author: sroy
ms.openlocfilehash: 9089fee7565633b198043efeed929fc7446437d5
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96458284"
---
# <a name="marketplace-insights-dashboard-in-commercial-marketplace-analytics"></a>Tableau de bord Insights de la Place de marché dans les données d’analytique de la Place de marché commerciale

Cet article donne des informations sur le tableau de bord Insight sur la place de marché de l’Espace partenaires. Ce tableau de bord présente un résumé de l’analytique web de la Place de marché commerciale, qui permet aux éditeurs de mesurer l’engagement client vis-à-vis de leurs fiches produit référencées dans les magasins en ligne de la Place de marché commerciale : Microsoft AppSource et Place de marché Azure

Pour accéder au tableau de bord **Insights sur la Place de marché** de l’Espace partenaires, sous Place de marché commerciale, sélectionnez **[Analyser](https://partner.microsoft.com/dashboard/commercial-marketplace/analytics/summary)**  > **Insights sur la Place de marché**.

Pour avoir des définitions détaillées des termes d’analytique, consultez [Terminologie relative à l’analytique de la Place de marché commerciale et questions fréquentes](./analytics-faq.md).

## <a name="marketplace-insights-dashboard"></a>Tableau de bord des insights de la Place de marché

Le tableau de bord Insights sur la Place de marché présente une vue d’ensemble des performances commerciales des offres de la Place de marché Azure et d’AppSource. Ce tableau de bord fournit une vaste vue d’ensemble des éléments suivants :

- Tendance des visites de la page
- Tendance des appels d’actions
- Visites de page et appels d’actions par rapport aux offres, domaines de référence et ID de campagne
- Insights sur la Place de marché par zone géographique
- Insights sur la place de marché – Tableau des détails

Le tableau de bord Insights de la Place de marché fournit des données de parcours qui ne doivent pas être corrélées avec les prospects générés dans le point de terminaison de destination du prospect.

> [!NOTE]
> La latence maximale entre les utilisateurs visitant des offres sur la Place de marché Azure ou AppSource, et la création de rapports dans l’Espace partenaires est de 48 heures.

## <a name="elements-of-the-marketplace-insights-dashboard"></a>Éléments du tableau de bord Insights sur la Place de marché

Le tableau de bord Insights sur la Place de marché affiche des détails de télémétrie web pour la Place de marché Microsoft Azure et AppSource dans deux onglets distincts. Les sections suivantes expliquent comment utiliser le tableau de bord Insights sur la Place de marché et lire les données.

### <a name="month-range"></a>Plage de mois

Une sélection par plage de mois est disponible en haut à droite de chaque page. Personnalisez la sortie des graphes de la page **Insights sur la Place de marché** en sélectionnant la plage de mois des 6 ou 12 derniers mois, ou une plage de mois personnalisée d’une durée maximale de 12 mois. La plage de mois par défaut (période de calcul) est de six mois.

:::image type="content" source="./media/insights-dashboard/month-filters.png" alt-text="Montre les filtres de mois dans le tableau de bord Insights sur la Place de marché.":::

> [!NOTE]
> Toutes les métriques dans les widgets de visualisation et les rapports d’exportation respectent la période de calcul sélectionnée par l’utilisateur.

### <a name="page-visits-trends"></a>Tendances des visites de la page

Le graphique **Visiteurs** des Insights sur la Place de marché affiche les nombres de _Visites de la page_ et de _Visiteurs uniques_ pour la période de calcul sélectionnée.

**Visites de la page** : ce nombre représente le nombre de sessions utilisateur distinctes sur la page de référencement (fiche produit) pour une période de calcul sélectionnée. Les indicateurs de pourcentage rouge et vert représentent le pourcentage de croissance des visites de la page. Le graphique de tendances indique le nombre de visites de pages par mois.

**Visiteurs uniques** : ce nombre représente le nombre de visiteurs distincts au cours de la période de calcul sélectionnée pour la ou les offres dans la Place de marché Azure et AppSource. Un visiteur qui a visité une ou plusieurs fiches produits est compté comme un visiteur unique.

[![Montre le graphique Visiteurs dans le tableau de bord Insights sur la Place de marché.](./media/insights-dashboard/visitors.png)](./media/insights-dashboard/visitors.png#lightbox)

### <a name="call-to-actions-trend"></a>Tendance des appels d’actions

Ce nombre représente le nombre de clics sur le bouton **Appel d’action** effectués dans la page de référencement de l’offre (fiche produit). Les _Appels d’actions_ sont comptabilisés quand des utilisateurs sélectionnent les boutons **Télécharger maintenant**, **Essai gratuit**, **Contactez-moi** et **Version d’évaluation**.

[![Montre le graphique Appel d’action dans le tableau de bord Insights sur la Place de marché.](./media/insights-dashboard/call-to-actions-trend.png)](./media/insights-dashboard/call-to-actions-trend.png#lightbox)

### <a name="page-visits-and-call-to-actions-against-offers-referral-domains-and-campaign-ids"></a>Visites de page et appels d’actions par rapport aux offres, domaines de référence et ID de campagne

**Domaines de référence** : Si vous sélectionnez un domaine de referral spécifique, la tendance mensuelle des visites de pages et des appels à l’action apparaît sur le graphe à droite.

:::image type="content" source="./media/insights-dashboard/referral-domain.png" alt-text="Montre le graphique Domaine de référence dans le tableau de bord Insights sur la Place de marché.":::

**Offres** : sélectionnez une offre spécifique pour voir la tendance mensuelle des visites de la page et des appels d’actions sur le graphique à droite.

:::image type="content" source="./media/insights-dashboard/offer-alias.png" alt-text="Montre le graphique d’alias d’offre dans le tableau de bord Insights sur la Place de marché.":::

**ID de campagne** : en sélectionnant un ID de campagne spécifique, vous devriez être en mesure de comprendre le succès de la campagne. Pour chaque campagne, vous devriez voir la tendance mensuelle des visites de la page et des appels d’actions sur le graphique à droite.

:::image type="content" source="./media/insights-dashboard/campaign.png" alt-text="Montre le graphique de campagne dans le tableau de bord Insights sur la Place de marché.":::

### <a name="marketplace-insights-by-geography"></a>Insights sur la Place de marché par zone géographique

Pour la période de calcul sélectionnée, la carte thermique affiche le nombre de visites de la page, de visiteurs uniques et d’appels d’actions (CTA). Les couleurs, des plus claires aux plus sombres, sur la carte traduisent les nombres, des plus faibles aux plus élevés, de visiteurs uniques. Sélectionnez l’un des enregistrements du tableau pour zoomer sur un pays/une région.

:::image type="content" source="./media/insights-dashboard/geographical-spread.png" alt-text="Montre le graphique de répartition géographique dans le tableau de bord Insights sur la Place de marché.":::

Notez les points suivants :

- Vous pouvez déplacer la carte pour afficher l'emplacement exact.
- Vous pouvez zoomer sur un emplacement spécifique.
- La carte thermique comporte une grille supplémentaire présentant les détails du nombre de clients, du nombre de commandes et des heures d’utilisation normalisée à l’endroit spécifié.
- Vous pouvez rechercher et sélectionner un pays/région dans la grille pour zoomer sur l’emplacement dans la carte. Pour revenir à la vue d’origine, sélectionnez le bouton **Accueil** de la carte.

### <a name="marketplace-insights-details-table"></a>Insights sur la place de marché – Tableau des détails

Ce tableau affiche la liste des visites de page et des appels d’actions des pages de vos offres sélectionnées, triées par date.

- Vous pouvez extraire les données dans un fichier .TSV ou .CSV si le nombre d’enregistrements est inférieur à 1 000.
- En présence d’un nombre d’enregistrements supérieur à 1 000, les données exportées sont placées de manière asynchrone sur une page de téléchargement pour une durée de 30 jours.
- Filtrez les données par noms d’offre et noms de campagne pour afficher les données qui vous intéressent.

> [!TIP]
> Vous pouvez utiliser l’icône de téléchargement située dans l’angle supérieur droit d’un widget pour télécharger les données. Vous pouvez fournir des commentaires sur chacun des widgets en cliquant l’icône de « pouce vers le haut » ou de « pouce vers le bas ».

## <a name="next-steps"></a>Étapes suivantes

- Pour une vue d’ensemble des rapports d’analytique disponibles dans la Place de marché commerciale, consultez [Accéder aux rapports d’analytique de la Place de marché commerciale dans l’Espace partenaires](./partner-center-portal/analytics.md).
- Pour plus d’informations sur vos commandes dans un format graphique et téléchargeable, consultez [Tableau de bord Commandes dans l’analytique de la Place de marché commerciale](./orders-dashboard.md).
- Pour voir les métriques de facturation à l’utilisation des offres de machines virtuelles, consultez [Tableau de bord Utilisation dans l’analytique de la Place de marché commerciale](./usage-dashboard.md).
- Pour des informations détaillées sur vos clients, y compris des tendances de croissance, voir [Tableau de bord client dans l’analytique de la place de marché commerciale](./customer-dashboard.md).
- Pour obtenir la liste de vos demandes de téléchargement des 30 derniers jours, voir [Tableau de bord des téléchargements dans l’analytique de la place de marché commerciale](./partner-center-portal/downloads-dashboard.md).
- Pour une vue centralisée des commentaires clients sur les offres de la Place de marché Azure et AppSource, consultez [Tableau de bord Analytique des évaluations et avis de l’Espace partenaires](./partner-center-portal/ratings-reviews.md).
- Pour voir les questions fréquentes sur l’analytique de la place de marché commerciale et un dictionnaire complet des termes liés aux données, consultez [Terminologie pour l’analytique de la Place de marché commerciale et questions fréquentes](./analytics-faq.md).
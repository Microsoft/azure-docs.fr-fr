---
title: Tableau de bord des commandes de l’Espace partenaires dans l’analytique de la Place de marché commerciale, Microsoft AppSource et la Place de marché Azure
description: Découvrez comment accéder aux rapports d’analytique concernant vos commandes d’offres de la Place de marché commerciale, dans un format graphique et téléchargeable.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
ms.date: 11/09/2020
author: sayantanroy83
ms.author: sroy
ms.openlocfilehash: 943f1361a2a9d6bf8d891f8b4f2137b8738f0836
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96450755"
---
# <a name="orders-dashboard-in-commercial-marketplace-analytics"></a>Tableau de bord Commandes dans l’analytique de la place de marché commerciale

Cet article fournit des informations sur le tableau de bord Commandes de l’Espace partenaires. Ce tableau de bord présente des informations sur vos commandes, notamment les tendances de croissance, dans un format graphique et téléchargeable.

Pour accéder au tableau de bord Commandes dans l’Espace partenaires, sous **Place de marché commerciale**, sélectionnez **[Analyser](https://partner.microsoft.com/dashboard/commercial-marketplace/analytics/summary)**  > **Commandes**.

>[!NOTE]
> Pour avoir des définitions détaillées des termes d’analytique, consultez [Terminologie relative à l’analytique de la Place de marché commerciale et questions fréquentes](./analytics-faq.md).

## <a name="orders-dashboard"></a>Tableau de bord des commandes

Le tableau de bord Commandes présente les commandes en cours pour l’ensemble de vos offres Software as a Service (SaaS). Vous pouvez afficher une représentation graphique des éléments suivants :

- Tendance des commandes
- Commandes par siège et tendance de site
- Commandes par offre et références SKU
- Commandes par zone géographique
- Tableau détaillé des commandes
- Filtres de page des commandes

> [!NOTE]
> La latence maximale entre l’acquisition du client et la création de rapports dans l’Espace partenaires est de 48 heures.

## <a name="elements-of-the-orders-dashboard"></a>Éléments du tableau de bord Commandes

Les sections suivantes expliquent comment utiliser le tableau de bord Commandes et lire les données.

### <a name="month-range"></a>Plage de mois

Une sélection par plage de mois est disponible en haut à droite de chaque page. Personnalisez la sortie des graphes de la page **Commandes** en sélectionnant la plage de mois des 6 ou 12 derniers mois, ou une plage de mois personnalisée d’une durée maximale de 12 mois. La plage de mois par défaut (période de calcul) est de six mois.

:::image type="content" source="./media/orders-dashboard/month-filters.png" alt-text="Illustre les filtres de mois dans le tableau de bord Commandes.":::

> [!NOTE]
> Toutes les métriques dans les widgets de visualisation et les rapports d’exportation respectent la période de calcul sélectionnée par l’utilisateur.

### <a name="orders-trend"></a>Tendance des commandes

Dans cette section, vous trouverez le graphique **Commandes**, qui affiche la tendance de vos commandes actives et annulées pour la période de calcul sélectionnée. Les métriques et les tendances de croissance sont représentées par un graphique en courbes qui affiche la valeur de chaque mois en pointant sur la ligne du graphique. La valeur de pourcentage sous les mesures Commandes dans le widget représente la croissance ou la récession pendant la période de calcul sélectionnée.

Il existe deux compteurs de commandes : _Actif_ et _Annulé_.

- **Actif** est égal au nombre de commandes en cours d’utilisation par les clients pendant la plage de dates sélectionnée.
- **Annulé** est égal au nombre de commandes passées auparavant, puis annulées pendant la plage de dates sélectionnée.

[![Illustre le widget Commandes dans le tableau de bord Commandes qui affiche la tendance des commandes actives et annulées.](./media/orders-dashboard/orders-trend.png)](./media/orders-dashboard/orders-trend.png#lightbox)

### <a name="orders-by-per-seat-and-site-trend"></a>Commandes par siège et tendance de site

Le graphique en courbes **Commandes par siège et basées sur les sites** représente la métrique et la tendance des commandes SaaS par site et des commandes SaaS par siège passées par les clients (commandes annulées incluses).

[![Illustre le widget Commandes dans le tableau de bord Commandes qui affiche les commandes par siège et par tendance de site.](./media/orders-dashboard/seats-sites.png)](./media/orders-dashboard/seats-sites.png#lightbox)

Les offres SaaS peuvent utiliser l’un des deux modèles de tarification avec chaque plan : forfaitaire (basé sur le site) ou par utilisateur (basé sur le siège).

- **Forfaitaire** : Autorisez l’accès à votre offre pour un prix forfaitaire mensuel ou annuel. Cela est parfois désigné sous le nom de « tarification en fonction du site ».
- **Par utilisateur :** Autorisez l’accès à votre offre pour un prix basé sur le nombre d’utilisateurs pouvant accéder à l’offre ou occuper des sièges. Avec ce modèle basé sur l’utilisation, vous pouvez définir les nombres minimal et maximal d’utilisateurs que le plan prend en charge. Vous pouvez créer plusieurs plans pour configurer différents niveaux de prix en fonction du nombre d’utilisateurs. Ces champs sont facultatifs. Si vous n’en sélectionnez aucun, le nombre d’utilisateurs est considéré comme illimité (avec au minimum 1 utilisateur et au maximum d’autant d’utilisateurs que votre service le permet). Ces champs peuvent être modifiés dans le cadre d’une mise à jour de votre plan.
- **Facturation à l’usage** : En plus de la tarification forfaitaire. Avec ce modèle de tarification, vous pouvez éventuellement définir des plans contrôlés qui utilisent l’API du service de contrôle de la place de marché afin de facturer les clients pour l’utilisation non couverte par le prix forfaitaire.

Pour plus d’informations sur la facturation à l’usage basée sur le siège et sur le site, consultez [Planifier une offre SaaS pour la place de marché commerciale](plan-saas-offer.md).

### <a name="orders-by-offers-and-skus"></a>Commandes par offre et références SKU

Le graphique Commandes par offre et références SKU affiche les mesures et les tendances de toutes les offres :

- Les premières offres s’affichent dans le graphe et les autres sont regroupées sous **Tout le reste**.
- Vous pouvez sélectionner des offres spécifiques dans la légende pour afficher uniquement ces offres et les références SKU associées dans le graphique.
- Placez le curseur sur une tranche du graphique pour afficher le nombre de commandes et le pourcentage de cette offre par rapport au nombre total de vos commandes pour toutes les offres.
- La **Tendance des commandes par offre** présente les tendances de croissance mois par mois. La colonne du mois représente le nombre de commandes par nom d'offre. Le graphique en courbes affiche la tendance du pourcentage de croissance tracée sur un axe z.

:::image type="content" source="./media/orders-dashboard/orders-by-offers.png" alt-text="Illustre le graphique Commandes par offre dans le tableau de bord Commandes.":::

Vous pouvez sélectionner n’importe quelle offre et un maximum de trois références SKU de cette offre pour afficher la tendance d’un mois sur l’autre pour l’offre, les références SKU et les sièges.

### <a name="orders-by-geography"></a>Commandes par zone géographique

Pour la période de calcul sélectionnée, le carte thermique affiche le nombre total de commandes et le pourcentage de commandes nouvellement ajoutées par rapport à la géographie.  Les couleurs claires sur la carte indiquent des valeurs faibles et les couleurs foncées des valeurs élevées. Sélectionnez un enregistrement du tableau pour zoomer sur un pays ou une région spécifique.

[![Illustre le graphique Expansion géographique dans le tableau de bord Commandes.](./media/orders-dashboard/geographical-spread.png)](./media/orders-dashboard/geographical-spread.png#lightbox)

Notez les points suivants :

- Vous pouvez déplacer la carte pour afficher l'emplacement exact.
- Vous pouvez zoomer sur un emplacement spécifique.
- La carte thermique comporte une grille supplémentaire présentant les détails du nombre de clients, du nombre de commandes et des heures d’utilisation normalisée pour le lieu spécifié.
- Vous pouvez rechercher et sélectionner un pays/région dans la grille pour zoomer sur l’emplacement dans la carte. Pour revenir à la vue d’origine, sélectionnez le bouton **Accueil** de la carte.

### <a name="orders-details-table"></a>Tableau Détails des commandes

Le tableau Détails des commandes présente la liste numérotée des 1 000 premières commandes triées par date d’acquisition.

- Chaque colonne de la grille peut être triée.
- Les données peuvent être extraites dans un fichier .CSV ou .TSV si le nombre d’enregistrements est inférieur à 1 000.
- En présence d'un nombre d'enregistrements supérieur à 1 000, les données exportées sont placées, de manière asynchrone, dans une page de téléchargements pour une durée de 30 jours.
- Appliquez des filtres au tableau **Détails des commandes** pour afficher uniquement les données qui vous intéressent. Filtrez les données par pays/région, type de licence Azure, type de licence de Place de marché commerciale, type d’offre, état de la commande, évaluations gratuites, ID d’abonnement de Place de marché commerciale, ID de client et nom de société.
- Étant donné que les offres SaaS achetées par le biais de la Place de marché Azure ou de Microsoft AppSource ne nécessitent pas d’abonnement Azure, l’ID d’abonnement de la Place de marché s’affiche sous la forme 00000000-0000-0000-0000-000000000000 dans la section **Données de commandes détaillées**.
- Lorsqu’une offre est achetée par un client protégé, les informations contenues dans **Données détaillées des commandes** sont masquées (************).

**_Tableau 1 : Dictionnaire des termes relatifs aux données_* _

| Nom de la colonne | Nom de l’attribut | Définition |
| ------------ | ------------- | ------------- |
| ID d’abonnement de la Place de marché | ID d’abonnement de la Place de marché | Identificateur unique associé à l’abonnement Azure que le client a utilisé pour acheter votre offre de la Place de marché commerciale. Anciennement, GUID d’abonnement Azure. |
| MonthStartDate | Date de début du mois | Date de début du mois représente le mois d’achat. |
| Type d’offre | Type d’offre | Type de l’offre de la place de marché commerciale. |
| Type de licence Azure | Type de licence Azure | Type de contrat de licence utilisé par les clients pour acheter Azure. Également appelé canal. Les valeurs possibles sont les suivantes :<ul><li>Fournisseur de solutions cloud</li><li>Enterprise</li><li>entreprise, via un revendeur</li><li>paiement à l’utilisation</li></ul> |
| Type de licence de la Place de marché | Type de licence de la Place de marché | Mode de facturation de l’offre de la place de marché commerciale. Les différentes valeurs sont les suivantes :<ul><li>facturé via Azure</li><li>BYOL (apportez votre propre licence)</li><li>Gratuit</li><li>Microsoft en tant que revendeur</li></ul> |
| SKU | SKU | Plan associé à l’offre |
| Pays du client | Pays/Région du client | Nom de pays/région fourni par le client. Ce nom peut être différent du nom du pays ou de la région fourni dans l’abonnement Azure d’un client. |
| Référence SKU en préversion | Référence SKU en préversion | La valeur indique si vous avez étiqueté la référence SKU en tant que « préversion ». La valeur est « Oui » si la référence SKU a été étiquetée en conséquence, et seuls les abonnements Azure que vous avez autorisés peuvent déployer et utiliser cette image. La valeur est « Non » si la référence SKU n’a pas été identifiée en tant que « préversion ». |
| ID de commande | ID de commande | Identificateur unique de la commande du client pour votre service de la place de marché commerciale. Les offres basées sur l’utilisation de machines virtuelles ne sont pas associées à une commande. |
| Order Quantity | Order Quantity | Nombre de ressources associées à l’ID de commande pour les commandes actives |
| Nom de l’instance cloud | Nom de l’instance cloud | Instance de Microsoft Cloud sur laquelle des machines virtuelles ont été déployées. |
| Nouveau client | Nouveau client | La valeur identifie si un nouveau client a acheté une ou plusieurs de vos offres pour la première fois. La valeur est « Oui » si le mois calendaire correspond avec la « date d’acquisition ». La valeur est « Non » si le client a acheté une de vos offres avant le mois calendaire signalé. |
| État de la commande | État de la commande | État d’une commande de la place de marché commerciale au moment de la dernière actualisation des données. |
| Date d’annulation de la commande | Date d’annulation de la commande | Date à laquelle la commande passée sur la place de marché commerciale a été annulée. |
| Nom de la société du client | Nom de la société du client | Nom de société fourni par le client. Le nom peut être différent du nom de la ville fournie dans l’abonnement Azure d’un client. |
| Date de la commande | Date de la commande | Date à laquelle la commande de la place de marché commerciale a été passée. |
| Nom de l’offre | Nom de l’offre | Nom de l’offre de la place de marché commerciale. |
| Date de fin d'essai | Date de fin d'essai | La date de fin de la période d’essai de cette commande approche ou est dépassée. |
| Customer Id | ID de client | Identificateur unique attribué à un client. Un client peut avoir zéro ou plusieurs abonnements de la Place de marché Azure. |
| ID de compte de facturation | ID de compte de facturation | L’identifiant du compte sur lequel la facturation est générée. Mappez l’_ *ID de compte de facturation** à l’**ID client** pour connecter votre rapport de transactions de paiement avec les rapports Client, Commande et Utilisation. |
| AssetCount | Nombre de ressources | Nombre de ressources associées à l’ID de commande. |
||||

### <a name="orders-page-filters"></a>Filtres de page des commandes

Les filtres de la page **Commandes** s’appliquent au niveau de la page Commandes. Vous pouvez sélectionner un ou plusieurs filtres afin d’afficher le graphique pour les critères et les données souhaités dans la grille/l’exportation « Données de commandes détaillées ». Les filtres sont appliqués aux données extraites pour la plage de mois que vous avez sélectionnée dans l’angle supérieur droit de la page Commandes.

> [!TIP]
> Vous pouvez utiliser l’icône de téléchargement située dans l’angle supérieur droit d’un widget pour télécharger les données. Vous pouvez fournir des commentaires sur chacun des widgets en cliquant l’icône de « pouce vers le haut » ou de « pouce vers le bas ».

## <a name="next-steps"></a>Étapes suivantes

- Pour une vue d’ensemble des rapports d’analytique disponibles dans la Place de marché commerciale, consultez [Accéder aux rapports d’analytique de la Place de marché commerciale dans l’Espace partenaires](./partner-center-portal/analytics.md).
- Pour consulter les graphes, tendances et valeurs des données agrégées qui résument l’activité de la place de marché pour vos offres, voir [Tableau de bord Résumé dans l’analytique de la place de marché commerciale](./summary-dashboard.md).
- Pour plus d’informations sur vos commandes dans un format graphique et téléchargeable, consultez [Tableau de bord des commandes dans l’analytique de la Place de marché commerciale](orders-dashboard.md).
- Pour consulter les métriques d’utilisation et de facturation à l’usage des offres de machines virtuelles, voir [Tableau de bord de l’utilisation dans l’analytique de la place de marché commerciale](./usage-dashboard.md).
- Pour obtenir la liste de vos demandes de téléchargement des 30 derniers jours, voir [Tableau de bord des téléchargements dans l’analytique de la place de marché commerciale](./partner-center-portal/downloads-dashboard.md).
- Pour une vue centralisée des commentaires clients sur les offres de la Place de marché Azure et AppSource, consultez [Tableau de bord Analytique des évaluations et avis de l’Espace partenaires](./partner-center-portal/ratings-reviews.md).
- Pour voir les questions fréquentes sur l’analytique de la place de marché commerciale et un dictionnaire complet des termes liés aux données, consultez [Terminologie pour l’analytique de la Place de marché commerciale et questions fréquentes](./analytics-faq.md).
---
title: Tableau de bord Utilisation de l’analytique de la place de marché commerciale | Place de marché Azure
description: Découvrez comment accéder à toutes les métriques d’utilisation et de facturation à l’usage des offres publiées sur la Place de marché Azure.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
ms.date: 11/09/2020
author: sayantanroy83
ms.author: sroy
ms.openlocfilehash: 563ccdb0633f94acccf3ade9faf1c40c28f108ac
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96462927"
---
# <a name="usage-dashboard-in-commercial-marketplace-analytics"></a>Tableau de bord Utilisation dans les données d’analytique de la Place de marché commerciale

Cet article fournit des informations sur le tableau de bord Utilisation de l’Espace partenaires. Ce tableau de bord présente toutes les métriques d’utilisation normalisée, d’utilisation brute et de facturation à l’usage des offres de machines virtuelles dans trois onglets distincts : utilisation normalisée des machines virtuelles, utilisation brute des machines virtuelles et utilisation de la facturation à l’usage.

Pour accéder au tableau de bord Utilisation de l’Espace partenaires, sélectionnez **[Analyser](https://partner.microsoft.com/dashboard/commercial-marketplace/analytics/summary)**  > **Utilisation** sous **Place de marché commerciale**.

>[!NOTE]
> Pour avoir des définitions détaillées des termes d’analytique, consultez [Terminologie relative à l’analytique de la Place de marché commerciale et questions fréquentes](./analytics-faq.md).

## <a name="usage-dashboard"></a>Tableau de bord de l’utilisation

Le tableau de bord **Utilisation** du menu **Analyser** présente les commandes en cours pour l’ensemble de vos offres SaaS (software as a service). Vous pouvez afficher une représentation graphique des éléments suivants :

- Tendance d’utilisation
- Utilisation normalisée selon les offres
- Utilisation normalisée selon d’autres dimensions : taille de machine virtuelle, canaux de vente et types d’offres
- Utilisation par zone géographique
- Tableau d’utilisation détaillée
- Filtres de la page Commandes

> [!NOTE]
> Dans l’Espace partenaires, la latence maximale entre la génération d’événements d’utilisation et la création des rapports est de 48 heures.

## <a name="elements-of-the-usage-dashboard"></a>Éléments du tableau de bord Utilisation

Les sections suivantes expliquent comment utiliser le tableau de bord Utilisation et lire les données.

### <a name="month-range"></a>Plage de mois

Une sélection par plage de mois est disponible en haut à droite de chaque page. Personnalisez la sortie des graphes de la page **Utilisation** en sélectionnant la plage des 6 ou 12 derniers mois, ou une plage de mois personnalisée d’une durée maximale de 12 mois. La plage de mois par défaut (période de calcul) est de six mois.

:::image type="content" source="./media/usage-dashboard/month-filter.png" alt-text="Illustration des filtres de mois dans le tableau de bord Utilisation":::

### <a name="usage-trend"></a>Tendance d’utilisation

Vous trouverez dans cette section le nombre total d’heures d’utilisation et la tendance de toutes les offres consommées par vos clients au cours de la période de calcul sélectionnée. Les métriques et les tendances de croissance sont représentées par un graphique en courbes. Affichez la valeur de chaque mois en pointant sur la ligne du graphe. La valeur de pourcentage indiquée sous les métriques d’utilisation dans le widget représente la croissance ou la baisse pendant la période de calcul sélectionnée.

Il existe deux représentations des heures d’utilisation : utilisation normalisée des machines virtuelles et utilisation brute des machines virtuelles.

- Les heures d’utilisation normalisée correspondent aux heures d’utilisation normalisées pour prendre en compte le nombre de cœurs de machines virtuelles ([nombre de cœurs de machines virtuelles] x [heures d’utilisation brute]). Les machines virtuelles désignées sous le nom de « SHAREDCORE » utilisent 1/6 (ou 0,1666) comme multiplicateur [nombre de cœurs de machines virtuelles].
- Les heures d’utilisation brute correspondent au nombre d’heures pendant lesquelles les machines virtuelles ont été utilisées.

[![Illustration des données d’utilisation normalisée et d’utilisation brute sur le tableau de bord Utilisation](./media/usage-dashboard/normalized-usage.png)](./media/usage-dashboard/normalized-usage.png#lightbox)

### <a name="normalized-usage-by-offers"></a>Utilisation normalisée selon les offres

Dans cette section figurent le nombre total d’heures d’utilisation et la tendance de vos offres à l’utilisation sur la Place de marché Azure. Le graphe Utilisation normalisée selon les offres est décrit ci-dessous.

- L’histogramme empilé de **l’utilisation normalisée selon les offres** présente la répartition des heures d’utilisation normalisée des cinq premières offres en fonction de la plage de dates sélectionnée. Les autres offres sont regroupées dans la catégorie **Tout le reste**.
- L’histogramme empilé représente la tendance de croissance mois par mois pour la plage de dates sélectionnée. Les colonnes de mois représentent les heures d’utilisation des offres les plus utilisées (en nombre d’heures) durant le mois respectif. Le graphique en courbes illustre la tendance du pourcentage de croissance tracée sur l’axe Y secondaire.
- Vous pouvez sélectionner des offres spécifiques dans la légende pour afficher uniquement ces offres dans le graphique.

:::image type="content" source="./media/usage-dashboard/normalized-usage-offers.png" alt-text="Illustration des données d’utilisation normalisée des offres sur le tableau de bord Utilisation":::

Vous pouvez sélectionner n’importe quelle offre et au maximum trois de ses références SKU pour afficher leur tendance d’utilisation d’un mois à l’autre.

:::image type="content" source="./media/usage-dashboard/normalized-usage-offers-sku.png" alt-text="Illustration des données d’utilisation normalisée des offres et des références SKU sur le tableau de bord Utilisation":::

### <a name="orders-by-offers-and-skus"></a>Commandes selon les offres et les références SKU

Le graphe **Commandes selon les offres et les références SKU** présente les métriques et les tendances de toutes les offres. Notez les points suivants :

- Les premières offres s’affichent dans le graphe et les autres sont regroupées sous **Tout le reste**.
- Vous pouvez sélectionner des offres spécifiques dans la légende pour afficher uniquement ces offres dans le graphique.
- Placez le curseur sur une tranche du graphique pour afficher le nombre de commandes et le pourcentage de cette offre par rapport au nombre total de vos commandes pour toutes les offres.
- La **Tendance des commandes par offre** présente les tendances de croissance mois par mois. La colonne du mois représente le nombre de commandes par nom d'offre. Le graphique en courbes présente la tendance du pourcentage de croissance tracée sur l’axe Z.

:::image type="content" source="./media/usage-dashboard/orders-by-offers-and-skus.png" alt-text="Illustration du graphe Commandes selon les offres et les références SKU dans le tableau de bord Utilisation":::

Vous pouvez sélectionner n’importe quelle offre et au maximum trois de ses références SKU pour afficher la tendance d’un mois à l’autre de l’offre, des références SKU et des sièges.

:::image type="content" source="./media/usage-dashboard/orders-by-offers-and-skus-2.png" alt-text="Illustration du graphe Commandes selon les offres et les références SKU dans le tableau de bord Utilisation, présentant la tendance de l’offre, celle des références SKU et celle des sièges":::

#### <a name="normalized-usage-by-other-dimensions-vm-size-sales-channels-and-offer-type"></a>Utilisation normalisée selon d’autres dimensions : taille de machine virtuelle, canaux de vente et type d’offre

Il existe trois onglets pour les dimensions : taille de machine virtuelle, canaux de vente et type d’offre. Sont affichées les métriques d’utilisation et la tendance d’un mois à l’autre par rapport à chacune de ces dimensions.

:::image type="content" source="./media/usage-dashboard/normalized-usage-other-dimensions.png" alt-text="Illustration du graphe Utilisation normalisée : Autres dimensions sur le tableau de bord Utilisation":::

### <a name="usage-by-geography"></a>Utilisation par zone géographique

Pour la période de calcul sélectionnée, la carte thermique présente l’utilisation totale par rapport à la dimension géographique. Les couleurs claires sur la carte indiquent des valeurs faibles et les couleurs foncées des valeurs élevées. Sélectionnez l’un des enregistrements du tableau pour zoomer sur un pays/une région.

:::image type="content" source="./media/usage-dashboard/normalized-usage-country.png" alt-text="Illustration du graphe Utilisation normalisée : Pays sur le tableau de bord Utilisation":::

Notez les points suivants :

- Vous pouvez déplacer la carte pour afficher l'emplacement exact.
- Vous pouvez zoomer sur un emplacement spécifique.
- La carte thermique comporte une grille supplémentaire présentant les détails du nombre de clients, du nombre de commandes et des heures d’utilisation normalisée à l’endroit spécifié.
- Vous pouvez rechercher et sélectionner un pays/région dans la grille pour zoomer sur l’emplacement dans la carte. Pour revenir à la vue d’origine, sélectionnez le bouton **Accueil** de la carte.

### <a name="usage-details-table"></a>Tableau des détails de l’utilisation

Le tableau des **détails de l’utilisation** présente une liste numérotée des 1 000 premiers enregistrements d’utilisation triés par utilisation. Notez les points suivants :

- Chaque colonne de la grille peut être triée.
- Les données peuvent être extraites dans un fichier .TSV ou .CSV si le nombre d’enregistrements est inférieur à 1 000.
- Si le nombre d’enregistrements dépasse 1 000, les données sont exportées de manière asynchrone sur une page de téléchargements qui restera disponible pendant 30 jours.
- Appliquez des filtres aux **données d’utilisation détaillées** pour afficher uniquement les données qui vous intéressent. Données filtrées par pays/région, canal de vente, type de licence Place de marché, type d’utilisation, nom d’offre, type d’offre, essais gratuits, ID d’abonnement Place de marché, ID client et nom de société.

_**Tableau 1 : dictionnaire des termes relatifs aux données**_

| Nom de la colonne | Nom de l’attribut | Définition |
| ------------ | ------------- | ------------- |
| ID d’abonnement de la Place de marché | ID d’abonnement de la Place de marché | Identificateur unique associé à l’abonnement Azure que le client a utilisé pour acheter votre offre de la Place de marché commerciale. Anciennement, GUID d’abonnement Azure. |
| MonthStartDate | Date de début du mois | Mois de l’achat. |
| Type d’offre | Type d’offre | Type de l’offre de la place de marché commerciale. |
| Type de licence Azure | Type de licence Azure | Type de contrat de licence utilisé par les clients pour acheter Azure. Également appelé Canal. Les valeurs possibles sont les suivantes :<ui><li>Fournisseur de solutions cloud</li><li>Enterprise</li><li>entreprise, via un revendeur</li><li>paiement à l’utilisation</li></ul> |
| Type de licence de la Place de marché | Type de licence de la Place de marché | Mode de facturation de l’offre de la place de marché commerciale. Les valeurs possibles sont les suivantes :<ul><li>facturé via Azure</li><li>BYOL (apportez votre propre licence)</li><li>Gratuit</li><li>Microsoft en tant que revendeur</li></ul> |
| SKU | SKU | Plan associé à l’offre. |
| Pays du client | Pays/Région du client | Nom de pays/région fourni par le client. Ce nom peut être différent du nom du pays ou de la région fourni dans l’abonnement Azure d’un client. |
| Référence SKU en préversion | Référence SKU en préversion | Indique si la référence SKU est étiquetée comme « préversion ». La valeur est « Oui » si la référence SKU a été étiquetée en conséquence, et seuls les abonnements Azure que vous avez autorisés peuvent déployer et utiliser cette image. La valeur est « Non » si la référence SKU n’a pas été identifiée en tant que « préversion ». |
| Type de facturation de la référence SKU | Type de facturation de la référence SKU | Type de facturation associé à chaque référence SKU de l’offre. Les valeurs possibles sont les suivantes :<ul><li>Gratuit</li><li>Payant</li></ul> |
| IsInternal | Déprécié | Déprécié |
| Taille de la machine virtuelle | Taille de la machine virtuelle | Pour les types d’offres basés sur une machine virtuelle, taille de la machine virtuelle associée à la référence SKU de l’offre. |
| Nom de l’instance cloud | Nom de l’instance cloud | Instance de Microsoft Cloud sur laquelle des machines virtuelles ont été déployées. |
| ServicePlanName | Déprécié | Déconseillé (même définition que SKU) |
| Nom de l’offre | Nom de l’offre | Nom de l’offre de la place de marché commerciale. |
| DeploymentMethod | Déprécié | Déconseillé (même définition que Type d’offre)
 |
| Nom de la société du client | Nom de la société du client | Nom de société fourni par le client. Peut être différent de la ville indiquée dans l’abonnement Azure du client. |
| Date d'utilisation | Date d'utilisation | Date de génération d’événements d’utilisation pour les ressources à l’utilisation. |
| IsMultisolution | Est de type multisolution | Indique si l’offre est de type multisolution. |
| Nouveau client | Déprécié | Déprécié |
| Taille de base | Taille de base | Nombre de cœurs associés à l’offre basée sur une machine virtuelle. |
| Type d'utilisation | Type d'utilisation | Indique si l’événement d’utilisation associé à l’offre fait partie de la liste suivante :<ul><li>Utilisation normalisée</li><li>Utilisation brute</li><li>Utilisation facturée à l’usage</li></ul> |
| Date de fin d'essai | Date de fin d'essai | La date de fin de la période d’essai de cette commande approche ou est dépassée. |
| Devise du client (DC) | Devise du client | Devise utilisée par le client pour les transactions de la place de marché commerciale. |
| Prix (DC) | Price | Prix unitaire de la référence SKU dans la devise du client. |
| Devise de paiement (DE) | Devise de paiement | Devise configurée par l’éditeur dans laquelle il est payé pour les événements d’utilisation associés à la ressource. |
| Prix estimé (DE) | Prix estimé | Prix unitaire de la référence SKU dans la devise configurée par l’éditeur. |
| Référence d’utilisation | Référence d’utilisation | GUID concaténé servant à connecter le rapport Utilisation (dans l’analytique de la place de marché commerciale) au rapport Transaction de paiement. La Référence d’utilisation est liée à OrderId et à LineItemId dans le rapport Transaction de paiement. |
| Unité d’utilisation | Unité d’utilisation | Unité de consommation associée à la référence SKU. |
| Customer Id | ID de client | Identificateur unique attribué à un client. Un client peut avoir zéro ou plusieurs abonnements de la Place de marché Azure. |
| ID de compte de facturation | ID de compte de facturation | L’identifiant du compte sur lequel la facturation est générée. Mappez l’**ID de compte de facturation** à l’**ID client** pour connecter votre rapport de transactions de paiement avec les rapports Client, Commande et Utilisation. |
| Quantité d’utilisation | Quantité d’utilisation | Nombre total d’unités d’utilisation consommées par la ressource déployée par le client.<br>Dépend de l’élément Type d’utilisation. Par exemple, si le Type d’utilisation est Utilisation normalisée, la quantité d’utilisation correspond à une utilisation normalisée. |
| NormalizedUsage | Utilisation normalisée | Nombre total d’unités d’utilisation normalisées consommées par la ressource déployée par le client.<br>Les heures d’utilisation normalisée correspondent aux heures d’utilisation normalisées pour prendre en compte le nombre de cœurs de machines virtuelles ([nombre de cœurs de machines virtuelles] x [heures d’utilisation brute]). Les machines virtuelles désignées sous le nom de « SHAREDCORE » utilisent 1/6 (ou 0,1666) comme multiplicateur [nombre de cœurs de machines virtuelles]. |
| MeteredUsage | Utilisation facturée à l’usage | Nombre total d’unités d’utilisation consommées selon les compteurs configurés avec l’offre déployée par le client. |
| RawUsage | Utilisation brute | Nombre total d’unités d’utilisation brute consommées par la ressource déployée par le client.<br>Les heures d’utilisation brute correspondent à la durée, en unités d’utilisation, pendant laquelle les machines virtuelles ont été utilisées. |
| Estimation des frais d’extension (DC) | Frais étendus estimés dans la devise du client | Frais associés à l’utilisation. La colonne correspond au produit entre Prix et Quantité d’utilisation. |
| Estimation des frais d’extension (DE) | Frais étendus estimés dans la devise de paiement | Frais associés à l’utilisation. La colonne correspond au produit entre Prix estimé et Quantité d’utilisation. |
||||

### <a name="usage-page-filters"></a>Filtres de la page Utilisation

Les filtres de la page **Utilisation** sont appliqués au niveau de la page Commandes. Vous pouvez sélectionner un ou plusieurs filtres afin d’afficher le graphe pour les critères et les données souhaités dans la grille/l’exportation des données d’utilisation des commandes. Les filtres sont appliqués aux données extraites pour la plage de mois sélectionnée en haut à droite de la page Utilisation.

Les widgets et le rapport d’exportation relatifs à l’utilisation brute des machines virtuelles sont similaires à l’utilisation normalisée des machines virtuelles, à ces différences près :

- Les heures d’utilisation normalisée correspondent aux heures d’utilisation normalisées pour prendre en compte le nombre de cœurs de machines virtuelles ([nombre de cœurs de machines virtuelles] x [heures d’utilisation brute]). Les machines virtuelles désignées sous le nom de « SHAREDCORE » utilisent 1/6 (ou 0,1666) comme multiplicateur [nombre de cœurs de machines virtuelles].
- Les heures d’utilisation brute correspondent à la durée, en unités d’utilisation, pendant laquelle les machines virtuelles ont été utilisées.

### <a name="metered-billing-usage"></a>Utilisation de la facturation à l’usage

L’onglet **Utilisation de la facturation à l’usage** présente des informations sur l’utilisation des types d’offres, mesurée selon la dimension de compteur. La surutilisation du type d’offre SaaS y est affichée. L’onglet fournit des représentations graphiques des tendances de surutilisation pour la facturation à l’usage de SaaS :

- **Tendance de dépassement par dimension de compteur** : affiche la tendance de dépassement par mois de la dimension de compteur sélectionnée pour une offre. L’axe X représente le mois et l’axe Y la quantité d’utilisation de dépassement. L’unité de mesure du compteur personnalisé est également indiquée sur l’axe Y.

    :::image type="content" source="./media/usage-dashboard/metered-usage.png" alt-text="Illustration du graphe Utilisation normalisée des machines virtuelles sur le tableau de bord Utilisation":::

- **Tendance de surutilisation par référence SKU** : représente la tendance de la quantité d’utilisation de la dimension de compteur sélectionnée selon les références SKU et les plans. Sont affichés les cinq plans enregistrant la plus grande quantité d’utilisation de l’offre sélectionnée.

- **Tendance de dépassement selon les clients** : représente une liste empilée des clients enregistrant le plus grand nombre d’heures d’utilisation, affichés dans un _classement_ en fonction de l’utilisation du compteur personnalisé. Sélectionnez un client dans le classement pour voir la tendance d’utilisation de dépassement d’une dimension de compteur sélectionnée.

    :::image type="content" source="./media/usage-dashboard/metered-usage-by-customers.png" alt-text="Illustration du graphe Utilisation de la facturation à l’usage selon les clients sur le tableau de bord Utilisation":::

Si vous avez plusieurs offres qui utilisent des compteurs personnalisés, le rapport sur l’utilisation facturée affiche les informations d’utilisation de toutes vos offres, en fonction de leurs dimensions de compteur personnalisées.

> [!TIP]
> Vous pouvez utiliser l’icône de téléchargement située dans l’angle supérieur droit d’un widget pour télécharger les données. Vous pouvez fournir des commentaires sur chacun des widgets en cliquant l’icône de « pouce vers le haut » ou de « pouce vers le bas ».

## <a name="next-steps"></a>Étapes suivantes

- Pour une vue d’ensemble des rapports d’analytique disponibles dans la Place de marché commerciale, consultez [Accéder aux rapports d’analytique de la Place de marché commerciale dans l’Espace partenaires](./partner-center-portal/analytics.md).
- Pour consulter les graphes, tendances et valeurs des données agrégées qui résument l’activité de la Place de marché pour vos offres, consultez [Tableau de bord Résumé dans l’analytique de la Place de marché commerciale](./summary-dashboard.md).
- Pour plus d’informations sur vos commandes dans un format graphique téléchargeable, consultez [Tableau de bord Commandes dans l’analytique de la place de marché commerciale](./orders-dashboard.md).
- Pour voir les métriques de facturation à l’utilisation des offres de machines virtuelles, consultez [Tableau de bord de l’utilisation dans l’analytique de la Place de marché commerciale](usage-dashboard.md).
- Pour obtenir la liste de vos demandes de téléchargement des 30 derniers jours, voir [Tableau de bord des téléchargements dans l’analytique de la place de marché commerciale](./partner-center-portal/downloads-dashboard.md).
- Pour une vue centralisée des commentaires clients sur les offres de la Place de marché Azure et de Microsoft AppSource, consultez [Tableau de bord Analytique des évaluations et avis de l’Espace partenaires](./partner-center-portal/ratings-reviews.md).
- Pour voir les questions fréquentes sur l’analytique de la Place de marché commerciale et un dictionnaire complet des termes liés aux données, consultez [Terminologie pour l’analytique de la Place de marché commerciale et questions fréquentes](./analytics-faq.md).
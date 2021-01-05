---
title: Tableau de bord Clients dans l’analytique de la Place de marché commerciale Microsoft dans l’Espace partenaires, la Place de marché Azure et Microsoft AppSource
description: Découvrez comment accéder à des informations détaillées sur vos clients, notamment les tendances de croissance, avec le tableau de bord Clients dans l’analytique de la Place de marché commerciale.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
ms.date: 11/09/2020
author: sayantanroy83
ms.author: sroy
ms.openlocfilehash: 878fa181ff09d547ef6594a238e9c1d883cc89b7
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96460481"
---
# <a name="customers-dashboard-in-commercial-marketplace-analytics"></a>Tableau de bord Clients dans l’analytique de la Place de marché commerciale

Cet article fournit des informations sur le tableau de bord Client de l’Espace partenaires. Ce tableau de bord présente des informations sur vos clients, notamment les tendances de croissance, dans un format graphique et téléchargeable.

Pour accéder au tableau de bord Clients dans l’Espace partenaires, sous **Place de marché commerciale**, sélectionnez **[Analyser](https://partner.microsoft.com/dashboard/commercial-marketplace/analytics/summary)**  > **Clients**.

>[!NOTE]
> Pour avoir des définitions détaillées des termes d’analytique, consultez [Terminologie relative à l’analytique de la Place de marché commerciale et questions fréquentes](./analytics-faq.md).

## <a name="customers-dashboard"></a>Tableau de bord des clients

Le tableau de bord Clients affiche des données concernant les clients qui ont acquis vos offres. Vous pouvez afficher une représentation graphique des éléments suivants :

- Tendance des clients actifs et perdus
- Tendance de croissance de la clientèle, incluant les clients existants, nouveaux et perdus
- Clients par commande et utilisation
- Centile de clients 
- Type de client par commandes et utilisation
- Clients par zone géographique
- Tableau des détails des clients
- Filtres de la page Clients

> [!NOTE]
> La latence maximale entre l’acquisition du client et la création de rapports dans l’Espace partenaires est de 48 heures.

## <a name="elements-of-the-customers-dashboard"></a>Éléments du tableau de bord Clients

Les sections suivantes expliquent comment utiliser le tableau de bord Clients et lire les données.

### <a name="month-range"></a>Plage de mois

Une sélection par plage de mois est disponible en haut à droite de chaque page. Personnalisez la sortie des graphes de la page **Clients** en sélectionnant la plage de mois des 6 ou 12 derniers mois, ou une plage de mois personnalisée d’une durée maximale de 12 mois. La plage de mois par défaut (période de calcul) est de six mois.

:::image type="content" source="./media/customer-dashboard/month-range-filters.png" alt-text="Illustre les filtres de mois sur la page Clients.":::

> [!NOTE]
> Toutes les métriques dans les widgets de visualisation et les rapports d’exportation respectent la période de calcul sélectionnée par l’utilisateur.

### <a name="active-and-churned-customers-trend"></a>Tendance des clients actifs et perdus

Cette section présente la tendance de croissance de votre clientèle pour la période de calcul sélectionnée. Les métriques et les tendances de croissance sont représentées par un graphique en courbes qui permet d’afficher la valeur de chaque mois en pointant sur la ligne du graphique. La valeur de pourcentage sous **Clients actifs** dans le widget représente la croissance pendant la période de calcul sélectionnée. Par exemple, la capture d’écran suivante montre une croissance de 0,92 % pour la période de calcul sélectionnée.

[![Illustre le widget Clients sur la page Clients.](./media/customer-dashboard/customers-widget.png)](./media/customer-dashboard/customers-widget.png#lightbox)

Il existe trois _types de clients_ : anciens, existants et nouveaux.

- Un nouveau client a acheté une ou plusieurs de vos offres pour la première fois au cours du mois sélectionné.
- Un client existant a acheté une ou plusieurs de vos offres avant le mois sélectionné.
- Un client parti a annulé toutes les offres précédemment achetées. Les clients perdus sont représentés dans l’axe négatif du widget Tendance.

### <a name="customer-growth-trend-including-existing-new-and-churned-customers"></a>Tendance de croissance de la clientèle, incluant les clients existants, nouveaux et perdus

Cette section présente la tendance et le nombre total de clients, nouveaux, existants et perdus, avec une tendance de croissance mois par mois.

- Le graphique en courbes représente les pourcentages globaux de croissance des clients.
- La colonne du mois représente le nombre de clients regroupés par clients nouveaux, existants et partis.
- Le nombre de clients perdus s’affiche dans le sens négatif de l’axe X.
- Vous pouvez sélectionner des éléments de légende spécifiques pour afficher plus de détails. Par exemple, sélectionnez les nouveaux clients dans la légende pour afficher uniquement les nouveaux clients.
- Pointer sur une colonne du graphique a pour effet d’afficher uniquement les détails du mois correspondant.

[![Illustre le widget de tendance Clients sur la page Clients.](./media/customer-dashboard/customers-trend.png)](./media/customer-dashboard/customers-trend.png#lightbox)

### <a name="customers-by-ordersusage"></a>Clients par commandes/utilisation

Le graphique **Clients par commandes/utilisation** comporte trois onglets : Commandes, Utilisation normalisée et Utilisation brute. Sélectionnez l’onglet **Commandes** pour afficher les détails de la commande.

:::image type="content" source="./media/customer-dashboard/customers-by-orders-usage.png" alt-text="Illustre l’onglet Commandes du widget Clients par commandes/utilisation de la page Clients.":::

Notez les points suivants :

- Le classement présente les détails des clients par nombre de commandes. Lorsque vous sélectionnez un client, les détails de celui-ci sont présentés dans les sections adjacentes « Détails », « Commandes par référence SKU » et « Références SKU par postes ».
- Les détails du profil Client s’affichent dans cet espace lorsque les éditeurs sont connectés avec un rôle Propriétaire. S’ils sont connectés avec un rôle Contributeur, les détails de cette section ne sont pas disponibles.
- Le graphique en anneau **Commandes par référence SKU** présente la répartition des commandes passées pour les plans. Les cinq premiers plans par nombre de commandes sont affichés, tandis que les autres commandes sont regroupées sous **Tout le reste**.
- Le graphique en anneau **Références SKU par postes** présente la répartition des postes commandés pour les plans. Les cinq premiers plans par nombre de postes sont affichés, tandis que les autres commandes sont regroupées sous **Tout le reste**.

Vous pouvez également sélectionner l’onglet **Utilisation normalisée** ou **Utilisation brute** pour afficher les détails de l’utilisation.

- Le classement présente les détails des 50 premiers clients par nombre d’heures d’utilisation. Lorsque vous sélectionnez un client, les détails de celui-ci sont présentés dans la section adjacente « Détails », « Utilisation normalisée par offres » et « Utilisation normalisée par tailles de machine virtuelle ».
- Les détails du profil client s’affichent dans cet espace lorsque les éditeurs sont connectés avec un rôle Propriétaire. S’ils sont connectés avec un rôle Collaborateur, les détails de cette section ne sont pas disponibles.
- Le graphique en anneau **Utilisation normalisée par offres** affiche la répartition de l’utilisation par offres. Les cinq premiers plans par nombre d’utilisations sont affichés, tandis que les autres offres sont regroupées sous **Tout le reste**.
- Le graphique en anneau **Utilisation normalisée par tailles de machine virtuelle** affiche la répartition de l’utilisation par tailles de machine virtuelle. Les cinq premières tailles de machine virtuelles par utilisation normalisée sont affichées, tandis que les autres utilisations sont regroupées sous **Tout le reste**.

### <a name="top-customers-percentile"></a>Centile des premiers clients

Le graphique **Centile des premiers clients** comporte trois onglets : « Commandes », « Utilisation normalisée » et « Utilisation brute ». Le graphique _Centile des premiers clients_ s’affiche le long de l’axe X, en fonction du nombre de commandes. L’axe y affiche le nombre de commandes du client. L’axe Y secondaire (graphique linéaire) présente le pourcentage cumulé du nombre total de commandes. Vous pouvez afficher les détails en déplaçant le curseur sur les points le long du graphique en courbes.

[![Illustre l’onglet Commandes du widget Centile des premiers clients dans la page Clients.](./media/customer-dashboard/top-customer-percentile.png)](./media/customer-dashboard/top-customer-percentile.png#lightbox)

### <a name="customer-type-by-orders-and-usage"></a>Type de client par commandes et utilisation

Le graphe **Commandes/utilisation par type de client** présente le nombre de commandes et les heures d’utilisation normalisée et brute répartis entre les nouveaux clients et les clients existants, en fonction respectivement de la sélection des commandes et de l’utilisation normalisée et brute, dans le graphe.

Ce graphique montre ce qui suit :

- Un nouveau client a acheté une ou plusieurs de vos offres ou déclaré une utilisation pour la première fois au cours du même mois civil (axe Y).
- Un client existant a acheté une de vos offres ou déclaré une utilisation avant le mois civil de déclaration (axe Y).

[![Illustre l’onglet Commandes du widget Commandes par type de client dans la page Clients.](./media/customer-dashboard/orders-by-customer-type.png)](./media/customer-dashboard/orders-by-customer-type.png#lightbox)

### <a name="customers-by-geography"></a>Clients par zone géographique

Pour la période de calcul sélectionnée, le carte thermique affiche le nombre total de clients et le pourcentage de clients nouvellement ajoutés par rapport à la dimension géographique. Les couleurs claires sur la carte indiquent des valeurs faibles et les couleurs foncées des valeurs élevées. Sélectionnez un enregistrement du tableau pour zoomer sur un pays ou une région.

[![Illustre l’onglet Commandes du widget Commandes par zone géographique dans la page Clients.](./media/customer-dashboard/customers-by-geography.png)](./media/customer-dashboard/customers-by-geography.png#lightbox)

Notez les points suivants :

- Vous pouvez déplacer la carte pour afficher l'emplacement exact.
- Vous pouvez zoomer sur un emplacement spécifique.
- La carte thermique comporte une grille supplémentaire présentant les détails du nombre de clients, du nombre de commandes et des heures d’utilisation normalisée à l’endroit spécifié.
- Vous pouvez rechercher et sélectionner un pays/région dans la grille pour zoomer sur l’emplacement dans la carte. Pour revenir à la vue d’origine, sélectionnez le bouton **Accueil** de la carte.

### <a name="customer-details-table"></a>Tableau des détails des clients

Le tableau **Détails des clients** présente la liste numérotée des 1 000 meilleurs clients triés selon la date à laquelle ils ont acheté pour la première fois l’une de vos offres. Vous pouvez développer une section en sélectionnant l’icône d’expansion dans le ruban des détails.

Notez les points suivants :

- Les informations personnelles sur les clients sont uniquement disponibles s'ils ont donné leur consentement. Pour consulter ces informations, vous devez être connecté avec le niveau d’autorisations du rôle Propriétaire.
- Chaque colonne de la grille peut être triée.
- Les données peuvent être extraites dans un fichier .CSV ou .TSV si le nombre d’enregistrements est inférieur à 1 000.
- Si le nombre d’enregistrements est supérieur à 1 000, les données exportées sont placées de manière asynchrone sur une page de téléchargements pendant les 30 jours qui suivent.
- Appliquez des filtres au tableau pour afficher uniquement les données qui vous intéressent. Filtrez les données par nom de société, ID client, ID d’abonnement à la Place de marché, type de licence Azure, date d’acquisition, date d’annulation, adresse e-mail du client, pays/région/état/ville/code postal du client, langue du client, etc.
- Quand une offre est achetée par un client protégé, les informations contenues dans **Données détaillées du client** sont masquées (************).
- Les détails de la dimension Client, tels que le nom de la société, le nom du client et son adresse e-mail, sont au niveau de l’ID de l’organisation, non au niveau de la transaction de Place de marché Azure ou de Microsoft AppSource.

_**Tableau 1 : dictionnaire des termes relatifs aux données**_

| Nom de la colonne | Nom de l’attribut | Définition |
| ------------ | ------------- | ------------- |
| ID d’abonnement de la Place de marché | ID d’abonnement de la Place de marché | Identificateur unique associé à l’abonnement Azure que le client a utilisé pour acheter votre offre de la Place de marché commerciale. Anciennement, GUID d’abonnement Azure. |
| DateAcquired | Date d'acquisition | Date à laquelle le client a acheté pour la première fois une offre que vous avez publiée. |
| DateLost | Date d'annulation | Date à laquelle le client a annulé la dernière des offres achetées. |
| Nom du fournisseur | Nom du fournisseur | Nom du fournisseur impliqué dans la relation entre Microsoft et le client. Si le client est une entreprise passant par un revendeur, ce sera le revendeur. Si un fournisseur de solutions cloud (CSP) est impliqué, ce sera le CSP. |
| Adresse e-mail du fournisseur | Adresse e-mail du fournisseur | Adresse e-mail du fournisseur impliqué dans la relation entre Microsoft et le client. Si le client est une entreprise passant par un revendeur, ce sera le revendeur. Si un fournisseur de solutions cloud (CSP) est impliqué, ce sera le CSP. |
| FirstName | Prénom du client | Prénom fourni par le client. Le prénom peut être différent du prénom fourni dans l’abonnement Azure d’un client. |
| LastName | Nom du client | Nom fourni par le client. Le prénom peut être différent du prénom fourni dans l’abonnement Azure d’un client. |
| E-mail | Adresse e-mail du client | Adresse e-mail fournie par le client final. L’adresse e-mail peut être différente de l’adresse e-mail fournie dans l’abonnement Azure d’un client. |
| Nom de la société du client | Nom de la société du client | Nom de société fourni par le client. Le nom peut être différent du nom de la ville fournie dans l’abonnement Azure d’un client. |
| CustomerCity | Ville du client | Nom de ville fourni par le client final. La ville peut être différente de la ville fournie dans l’abonnement Azure d’un client. |
| Code postal du client | Code postal du client | Code postal fourni par le client. Le code postal peut être différent du code postal fourni dans l’abonnement Azure d’un client. |
| CustomerCommunicationCulture | Langue de communication du client | Langue dans laquelle le client préfère communiquer. |
| CustomerCountryRegion | Pays/Région du client | Nom de pays/région fourni par le client. Ce nom peut être différent du nom du pays ou de la région fourni dans l’abonnement Azure d’un client. |
| AzureLicenseType | Type de licence Azure | Type de contrat de licence utilisé par les clients pour acheter Azure. Également appelé _canal_. Les valeurs possibles sont les suivantes :<ul><li>Fournisseur de solutions cloud</li><li>Enterprise</li><li>entreprise, via un revendeur</li><li>paiement à l’utilisation</li></ul> |
| PromotionalCustomers | Acceptation des messages promotionnels | La valeur indique si le client a accepté les messages promotionnels provenant d’éditeurs de manière proactive. À ce stade, nous n'offrons pas cette possibilité aux clients. Par conséquent, nous avons indiqué « Non » sur le tableau. Une mise à jour interviendra après le déploiement de cette fonctionnalité. |
| customerState | État du client | État de résidence fourni par le client. L’État peut être différent de l’État fourni dans l’abonnement Azure d’un client. |
| CommerceRootCustomer | Client racine de commerce | Un ID de compte de facturation peut être associé à plusieurs ID de client.<br>Une combinaison d’un ID de compte de facturation et d’un ID de client peut être associée à plusieurs abonnements de la Place de marché commerciale.<br>Le Client racine de commerce est le nom du client de l’abonnement. |
| Customer Id | ID de client | Identificateur unique attribué à un client. Un client peut avoir zéro ou plusieurs abonnements de la Place de marché Azure. |
| ID de compte de facturation | ID de compte de facturation | L’identifiant du compte sur lequel la facturation est générée. Mappez l’**ID de compte de facturation** à l’**ID client** pour connecter votre rapport de transactions de paiement avec les rapports Client, Commande et Utilisation. |
||||

### <a name="customers-page-filters"></a>Filtres de la page Clients

Les filtres de la page Clients s’appliquent au niveau de la page Clients. Vous pouvez sélectionner plusieurs filtres afin d’afficher le graphique pour les critères et les données souhaités dans la grille/l’exportation « Données de commandes détaillées ». Les filtres sont appliqués aux données extraites pour la plage de mois que vous avez sélectionnée dans l’angle supérieur droit de la page Clients.

> [!TIP]
> Vous pouvez utiliser l’icône de téléchargement située dans l’angle supérieur droit d’un widget pour télécharger les données. Vous pouvez fournir des commentaires sur chacun des widgets en cliquant l’icône de « pouce vers le haut » ou de « pouce vers le bas ».

## <a name="next-steps"></a>Étapes suivantes

- Pour une vue d’ensemble des rapports d’analytique disponibles dans la Place de marché commerciale, consultez [Accéder aux rapports d’analytique de la Place de marché commerciale dans l’Espace partenaires](./partner-center-portal/analytics.md).
- Pour consulter les graphes, tendances et valeurs des données agrégées qui résument l’activité de la place de marché pour vos offres, voir [Tableau de bord Résumé dans l’analytique de la place de marché commerciale](./summary-dashboard.md).
- Pour plus d’informations sur vos commandes dans un format graphique et téléchargeable, consultez [Tableau de bord Commandes dans l’analytique de la Place de marché commerciale](./orders-dashboard.md).
- Pour voir les métriques de facturation à l’utilisation des offres de machines virtuelles, consultez [Tableau de bord de l’utilisation dans l’analytique de la Place de marché commerciale](./usage-dashboard.md).
- Pour obtenir la liste de vos demandes de téléchargement des 30 derniers jours, voir [Tableau de bord des téléchargements dans l’analytique de la place de marché commerciale](./partner-center-portal/downloads-dashboard.md).
- Pour une vue centralisée des commentaires clients sur les offres de la Place de marché Azure et de Microsoft AppSource, consultez [Tableau de bord Analytique des évaluations et avis de l’Espace partenaires](./partner-center-portal/ratings-reviews.md).
- Pour voir les questions fréquentes sur l’analytique de la Place de marché commerciale et un dictionnaire complet des termes liés aux données, consultez [Terminologie pour l’analytique de la Place de marché commerciale et questions fréquentes](./analytics-faq.md).
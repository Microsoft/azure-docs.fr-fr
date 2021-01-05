---
title: Utiliser des rapports Cloudyn dans Azure
description: Cet article décrit la finalité des rapports Cloudyn inclus dans le portail Cloudyn, afin de vous aider à les utiliser efficacement.
author: bandersmsft
ms.author: banders
ms.date: 03/12/2020
ms.topic: conceptual
ms.service: cost-management-billing
ms.subservice: cloudyn
ms.reviewer: benshy
ms.custom: seodec18
ROBOTS: NOINDEX
ms.openlocfilehash: 4bc68e851c4f3ba85f8ea18a91d86906e1f87887
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/20/2020
ms.locfileid: "88685219"
---
# <a name="reports-available-in-the-cloudyn-portal"></a>Rapports disponibles dans le portail Cloudyn

Cet article décrit l’objectif des rapports Cloudyn inclus dans le portail Cloudyn. Il explique également comment utiliser efficacement les rapports. La plupart des rapports sont intuitifs et ont une apparence similaire. L’essentiel des actions possibles dans un rapport est également réalisable dans d’autres rapports. Pour une vue d’ensemble de l’utilisation des rapports Cloudyn, y compris la personnalisation et l’enregistrement ou la planification, consultez [Comprendre les rapports de coûts](understanding-cost-reports.md).

Azure Cost Management offre des fonctionnalités similaires à Cloudyn. Azure Cost Management est une solution native de gestion des coûts Azure. Il vous permet d’analyser les coûts, de créer et de gérer des budgets, d’exporter des données, et de consulter des recommandations d’optimisation et d’agir en conséquence pour dépenser moins. Pour plus d’informations, consultez [Azure Cost Management](../cost-management-billing-overview.md).

[!INCLUDE [cloudyn-note](../../../includes/cloudyn-note.md)]

## <a name="report-types"></a>Types de rapports

Il existe trois types de rapports Cloudyn :

- Les rapports d’évolution dans le temps, par exemple, le rapport des coûts dans le temps. Les rapports d’évolution dans le temps présentent une série chronologique de données sur un intervalle sélectionné avec une résolution prédéfinie, ainsi qu’une résolution hebdomadaire pour les deux derniers mois. Vous pouvez utiliser le regroupement et le filtrage pour faire un zoom avant sur différents points de données.
  - Les rapports d’évolution dans le temps peuvent être utiles pour afficher les tendances et détecter des pics ou des anomalies.
- Les rapports d’analyse, par exemple, le rapport d’analyse des coûts. Ces rapports présentent des données agrégées sur une période définie par vos soins et permettent le regroupement et le filtrage des données.
  - Les rapports d’analyse peuvent être utiles pour afficher les pics et déterminer les causes racines des anomalies. Ils présentent une décomposition granulaire des données.
- Les rapports tabulaires. Il est possible d’afficher n’importe quel rapport sous forme de tableau, mais certains n’apparaissent que sous cette forme. Ces rapports se composent de listes détaillées d’éléments.
  - Les recommandations sont des rapports tabulaires, sans visualisation possible. Vous pouvez néanmoins visualiser les résultats des recommandations, par exemple, les économies dans le temps.
  - Les rapports tabulaires sont utiles sous forme de listes d’actions ou pour l’exportation de données à des fins de traitement, par exemple, les rapports de facturation interne.

Les rapports de coûts présentent les coûts _réels_ ou _amortis_.

Les rapports de coûts réels présentent les paiements effectués pendant la période sélectionnée. Par exemple, tous les frais ponctuels tels que les achats d’instances réservées figurent dans les rapports de coûts réels sous forme de pics de coûts.

Les rapports de coûts amortis étendent les frais ponctuels sur toute la période à laquelle ils s’appliquent. Par exemple, ceux de l’achat d’instances réservées sont répartis sur toute la durée de la réservation et ne sont pas montrés sous la forme d’un pic. La présentation amortie est la seule façon de voir les tendances réelles et de faire des projections de coûts.

Dans certains cas, l’amortissement est présenté dans un rapport distinct. Citons par exemple les rapports d’analyse des coûts et les rapports d’analyse des coûts amortis. Dans d’autres cas, l’amortissement est une stratégie de rapports, comme les rapports d’affectation des coûts et les rapports d’analyse des coûts.

La distribution périodique est planifiable pour n’importe quel rapport. Les rapports de coûts permettent de définir un seuil, ce qui est utile pour les alertes.

## <a name="cost-analysis-vs-cost-allocation"></a>Comparaison entre l’analyse des coûts et l’affectation des coûts

Les rapports _d’analyse des coûts_ présentent les données de facturation émises par les fournisseurs cloud. Ils permettent de regrouper et d’explorer différents segments de données détaillés dans le fichier de facturation. Vous pouvez ainsi parcourir précisément les coûts dans toutes les données de facturation brutes de votre fournisseur cloud.

Certains rapports _d’analyse des coûts_ ne regroupent pas les coûts par balises de ressources. Par ailleurs, les informations de facturation avec balise n’apparaissent dans les rapports qu’une fois les coûts affectés suivant un modèle de coûts créé avec [Cost Allocation 360](tutorial-manage-costs.md#use-custom-tags-to-allocate-costs).

Les rapports _d’affectation des coûts_ sont disponibles après création du modèle de coûts avec [Cost Allocation 360](tutorial-manage-costs.md#use-custom-tags-to-allocate-costs). Cloudyn traite les données de coût et de facturation et les _fait correspondre_ aux données d’utilisation et d’étiquette de vos comptes cloud. Pour faire correspondre les données, Cloudyn requiert un accès à vos données d’utilisation. Les comptes pour lesquels il manque les informations d’identification sont étiquetés comme _ressources sans catégorie_.

## <a name="dashboards"></a>Tableaux de bord

Les tableaux de bord de Cloudyn offrent une vue d’ensemble des rapports. Les tableaux de bord sont composés de widgets qui sont essentiellement des miniatures de rapport. Lorsque vous [personnalisez des rapports](understanding-cost-reports.md#save-and-schedule-reports), vous les enregistrez dans Mes rapports et ils sont ajoutés au tableau de bord. Pour plus d’informations sur les tableaux de bord, voir [Afficher les indicateurs de coûts clés avec des tableaux de bord](dashboards.md).

## <a name="budget-information-in-reports"></a>Informations budgétaires dans les rapports

De nombreux rapports Cloudyn ne présentent des informations budgétaires qu’après création manuelle d’un budget, et pas avant. Pour plus d’informations, voir [Paramètres de gestion budgétaire](#budget-management-settings).

## <a name="reports-and-reporting-features"></a>Rapports et fonctionnalités de génération de rapports

Cloudyn propose les rapports et fonctionnalités de création de rapports suivants.

### <a name="cost-navigator-report"></a>Rapport Navigateur des coûts

Le rapport Navigateur des coûts est un moyen rapide de voir sa consommation en matière de facturation dans une présentation de type tableau de bord. Il contient quelques filtres et affichages de base pour montrer instantanément une synthèse des coûts de l’organisation. Les coûts sont indiqués par date. Le rapport étant conçu comme une présentation initiale des coûts, il n’est pas aussi souple ou complet que d’autres rapports ou tableaux de bord personnalisés et créés par vos soins.

Par défaut, les affichages principaux du rapport présentent les informations suivantes :

- l’évolution des coûts au fil du temps dans un graphique à barres pour une semaine de travail (la **plage de dates** du graphique est modifiable) ;
- les dépenses par service dans un graphique à secteurs ;
- la catégorisation des ressources par balises dans un graphique à secteurs ;
- les dépenses par entité de coûts dans un graphique à secteurs ;
- le coût total par date en mode Liste.

### <a name="cost-analysis-report"></a>Rapport d’analyse des coûts

Le rapport d’analyse des coûts est un calcul de facturation interne et de 	récupération des données de facturation qui dépend de la stratégie mise en place. Il agrège la consommation cloud sur une période sélectionnée après application aux coûts de toutes les règles d’affectation. Par exemple, il calcule les coûts par balise, réaffecte les coûts des ressources sans balises et, éventuellement, affecte l’utilisation d’instances réservées.

Les stratégies définies dans [Cost Allocation 360](tutorial-manage-costs.md#use-custom-tags-to-allocate-costs) sont utilisées dans le rapport d’analyse des coûts ; les résultats sont ensuite combinés avec des informations provenant des données brutes du fournisseur cloud.

Comment ce rapport est-il calculé ? Le service Cloudyn garantit que l’affectation conserve l’intégrité de chaque compte lié en appliquant une _affinité de compte_. Cette affinité garantit qu’aucun coût n’est affecté à un compte pour un service qu’il n’utilise pas. Les coûts à payer dans ce compte restent dans ce compte et ne sont pas calculés par les stratégies d’affectation. Par exemple, vous pouvez avoir cinq comptes liés. Si seulement trois d’entre eux utilisent des services de stockage, alors le coût des services de stockage est affecté uniquement aux balises de ces trois comptes.

Utilisez le rapport d’analyse des coûts pour :

- Calculer la facturation interne/récupération des données de facturation de votre organisation.
- Catégoriser tous vos coûts.
- Afficher une vue agrégée de tout votre déploiement pendant un laps de temps donné.
- Afficher les coûts par catégories de balise en fonction des stratégies créées dans le modèle de coût.

Pour utiliser le rapport d’analyse des coûts :

1. Sélectionnez une plage de dates.
2. Ajoutez des balises, selon les besoins.
3. Ajoutez des groupes.
4. Choisissez un modèle de coût que vous avez créé précédemment.

### <a name="cost-over-time-report"></a>Rapport des coûts dans le temps

Le rapport des coûts dans le temps présente les résultats d’affectation des coûts sous forme de série chronologique. Il permet d’observer les tendances et de détecter des irrégularités de déploiement. Il montre avant tout les coûts répartis sur une période définie. Ce rapport inclut vos principaux contributeurs de coût, comme les coûts récurrents et les frais d’instances réservées ponctuels pendant un laps de temps sélectionné. Les stratégies définies dans [Cost Allocation 360](tutorial-manage-costs.md#use-custom-tags-to-allocate-costs) sont utilisées dans ce rapport.

Utilisez le rapport des coûts dans le temps pour :

- Voir les modifications dans le temps et ce qui a une incidence sur ces modifications d’un jour (ou d’une plage de dates) à l’autre.
- Analyser les coûts au fil du temps pour une instance spécifique.
- Comprendre pourquoi une augmentation des coûts a eu lieu pour une instance spécifique.

Pour utiliser le rapport des coûts dans le temps :

1. Sélectionnez une plage de dates.
2. Ajoutez des balises, selon les besoins.
3. Ajoutez des groupes.
4. Choisissez un modèle de coût que vous avez créé précédemment.
5. Sélectionnez des coûts réels ou des coûts amortis.
6. Choisissez l’application des règles d’allocation pour afficher les données de facturation brutes ou pour la vue des coûts recalculés.

### <a name="actual-cost-analysis-report"></a>Rapport d’analyse des coûts réels

Le rapport d’analyse des coûts réels présente les coûts liés au fournisseur sans aucune modification. Il indique les principaux postes de dépenses, notamment les coûts récurrents et les frais ponctuels.

Vous pouvez utiliser ce rapport pour afficher des informations sur les coûts de vos abonnements. Les abonnements Azure y sont indiqués comme **nom du compte** et **numéro de compte**. Les **comptes liés** présentent les abonnements AWS. Pour afficher la répartition des coûts par abonnement pour chaque compte, sélectionnez votre type d’abonnement sous **Groupes**.

Utilisez le rapport d’analyse du coût réel pour :

- Analyser et suivre les coûts bruts liés au fournisseur sur une période spécifiée.
- Planifier une alerte de seuil.
- Analyser les coûts non modifiés induits par vos comptes et entités.

### <a name="actual-cost-over-time-report"></a>Rapport des coûts réels dans le temps

Le rapport des coûts réels dans le temps est un rapport d’analyse des coûts standard qui répartit les coûts sur une résolution de temps définie. Le rapport présente les dépenses au fil du temps pour vous permettre d’observer les tendances et de détecter des irrégularités dans les dépenses. Il montre vos principaux contributeurs de coût, comme les coûts récurrents et les frais d’instances réservées ponctuels pendant un laps de temps sélectionné.

Utilisez le rapport des coûts réels dans le temps pour :

- Voir les tendances des coûts dans le temps.
- Trouver des irrégularités dans les coûts.
- Trouver toutes les questions liées aux coûts concernant les fournisseurs cloud.

### <a name="amortized-cost-reports"></a>Rapports de coûts amortis

Cet ensemble de rapports des coûts amortis indique les frais de service linéarisés non liés à l’utilisation ou les coûts payables une seule fois. Les coûts sont étalés dans le temps de manière régulière tout au long de leur durée de vie. Par exemple, les frais ponctuels peuvent inclure :

- Les frais d’assistance annuels
- Les frais annuels liés au composant de sécurité
- Les frais d’achat d’instances réservées
- Certains articles de la Place de marché Azure

Dans le fichier de facturation, les frais ponctuels se caractérisent par le fait que les dates de début et de fin (timestamp) de la consommation de services ont des valeurs égales. Le service Cloudyn les identifie comme frais ponctuels amortis. Les autres services liés à la consommation présentant des coûts d’utilisation à la demande ne peuvent pas être amortis.

Les rapports de coûts amortis sont les suivants :

- Analyse des coûts amortis
- Coûts amortis au fil du temps

### <a name="cost-analysis-report"></a>Rapport d’analyse des coûts

Le rapport d’analyse des coûts fournit un aperçu de votre consommation cloud et des dépenses pendant un laps de temps sélectionné. Les stratégies définies dans [Cost Allocation 360](tutorial-manage-costs.md#use-custom-tags-to-allocate-costs) sont utilisées dans le rapport d’analyse des coûts.

Comment Cloudyn calcule ce rapport ?

Cloudyn garantit que l’affectation conserve l’intégrité de chaque compte lié en appliquant une _affinité de compte_. Cette affinité garantit également qu’aucun coût n’est affecté à un compte pour un service qu’il n’utilise pas. Les coûts à payer dans ce compte restent dans ce compte et ne sont pas calculés par les stratégies d’affectation. Par exemple, vous pouvez avoir cinq comptes liés. Si seulement trois d’entre eux utilisent des services de stockage, alors le coût des services de stockage est affecté uniquement aux balises de ces trois comptes.

Utilisez le rapport d’analyse des coûts pour :

- Afficher une vue agrégée de tout votre déploiement pendant un laps de temps donné.
- Afficher les coûts par catégories de balise en fonction des stratégies créées dans le modèle de coût.

### <a name="cost-over-time-report"></a>Rapport des coûts dans le temps

Le rapport des coûts dans le temps présente l’évolution des dépenses au fil du temps pour permettre d’identifier des tendances et de remarquer les éventuelles irrégularités de déploiement. Il montre avant tout les coûts répartis sur une période définie. Ce rapport inclut vos principaux contributeurs de coût, comme les coûts récurrents et les frais d’instances réservées ponctuels pendant un laps de temps sélectionné. Les stratégies définies dans [Cost Allocation 360](tutorial-manage-costs.md#use-custom-tags-to-allocate-costs) sont utilisées dans ce rapport.

Utilisez le rapport des coûts dans le temps pour :

- Voir les modifications dans le temps et ce qui a une incidence sur ces modifications d’un jour (ou d’une plage de dates) à l’autre.
- Analyser les coûts au fil du temps pour une instance spécifique.
- Comprendre pourquoi une augmentation des coûts a eu lieu pour une instance spécifique.

### <a name="custom-charges-report"></a>Rapport des frais personnalisés

Les entreprises et fournisseurs de solutions cloud se retrouvent souvent à offrir des services complémentaires à leurs clients externes ou internes, en plus de leur propre consommation de ressources du cloud. Vous pouvez définir des frais personnalisés pour ces services ou remises supplémentaires, qui sont ajoutés aux rapports de facturation (interne ou non) du client sous forme d’éléments de ligne personnalisés.

Les frais de service personnalisés reflètent les services qui normalement n’apparaissent pas dans une facture. Une fois créés, ils s’affichent dans les rapports de coûts.

*Il faut distinguer frais personnalisés et tarification personnalisée*. La liste des frais personnalisés ne présente pas les différents prix appliqués. Par exemple, les frais de facturation AWS s’affichent exactement comme ils sont facturés.

Pour créer des frais personnalisés :

1. Dans **Frais personnalisés**, cliquez sur **Ajouter**. La boîte de dialogue _Ajouter des frais personnalisés_ s’affiche.
2. Dans **Nom du fournisseur**, entrez le nom du fournisseur.
3. Dans **Nom du service**, entrez le type de service.
4. Dans **Description**, ajoutez une description des frais personnalisés.
5. Dans **Type**, entrez le **Pourcentage** choisi, puis, dans la liste déroulante Services, les services à inclure en tant que frais personnalisés dans les rapports de coûts.
6. Dans **Paiement**, indiquez si les frais sont ponctuels ou récurrents. S’il s’agit de Frais récurrents, sélectionnez Amortis si vous souhaitez qu’ils soient amortis et sélectionnez le nombre de mois.
7. Dans **Dates**, si Frais ponctuels est sélectionné, entrez la date à laquelle les frais sont payés dans **Date d’effet**. Si Frais d’abonnement est sélectionné, entrez la plage de dates, soit la date de début et la date de fin des frais.
8. Dans **Arborescence des entités**, sélectionnez les entités auxquelles vous souhaitez appliquer les frais, puis sélectionnez **Activé**.

_Une fois les frais affectés à une entité, les utilisateurs ne peuvent plus les modifier. Les frais ajoutés par un administrateur à une entité parent sont en lecture seule._

Pour afficher les frais personnalisés :

Les frais personnalisés apparaissent dans les rapports de coûts. Par exemple, ouvrez le rapport d’analyse des coûts réels, puis, sous **Filtres étendus**, sélectionnez **Autonome**. Ensuite, filtrez les résultats afin d’afficher les **Frais personnalisés**.

### <a name="cost-allocation-360"></a>Cost Allocation 360

Cost Allocation 360 permet de créer des modèles d’affectation des coûts personnalisés pour affecter les coûts à des ressources du cloud consommées. De nombreux rapports comportent des informations provenant de modèles de coûts personnalisés. Par ailleurs, certains d’entre eux ne présentent des informations qu’après création d’un modèle de coûts personnalisé avec affectation des coûts.

Pour plus d’informations sur la création de modèles de coûts personnalisés, consultez [Tutoriel : Gérer les coûts avec Cloudyn](tutorial-manage-costs.md).

### <a name="cost-vs-budget-over-time-report"></a>Rapport comparatif des coûts et du budget dans le temps

Le rapport comparatif des coûts et du budget dans le temps sert à comparer les postes de coûts au budget. Le budget affecté apparaît dans le rapport pour permettre d’afficher l’évolution de la consommation budgétaire (supérieure/inférieure/égale) au fil du temps. Vous pouvez utiliser la fonction Afficher/masquer les champs en haut du rapport pour afficher le coût, le budget, le coût cumulé et le budget total.

### <a name="current-month-projected-cost-report"></a>Rapport des coûts prévisionnels pour le mois en cours

Le rapport des coûts prévisionnels pour le mois en cours donne des informations résumées sur les coûts mensuels à ce jour. Il présente les coûts depuis le début du mois, les coûts du mois précédent et le total des coûts prévisionnels pour le mois en cours. Les coûts prévisionnels du mois en cours sont calculés en additionnant les coûts mensuels à ce jour et des projections établies à partir des coûts observés au cours des 30 derniers jours.

Utilisez le rapport des coûts prévisionnels du mois en cours pour :

- Prévoir les coûts mensuels par service.
- Prévoir les coûts mensuels par compte.

### <a name="annual-projected-cost-report"></a>Rapport des coûts prévisionnels annuels

Le rapport des coûts prévisionnels annuels permet d’afficher les coûts prévisionnels annuels établis en fonction des tendances de dépenses passées. Il montre les 12 prochains mois des coûts prévisionnels globaux. Les projections sont effectuées à l’aide d’une fonction de tendance extrapolée sur les 12 mois en fonction des coûts associés aux 30 derniers jours d’utilisation.

### <a name="budget-management-settings"></a>Paramètres de gestion budgétaire

La gestion budgétaire permet de définir un budget pour l’exercice comptable.

Pour ajouter un budget à une entité :

1. Dans la page Gestion budgétaire, sous **Entités**, sélectionnez l’entité où vous souhaitez créer le budget.
2. Pour l’année budgétaire, sélectionnez l’année où vous souhaitez créer le budget.
3. Tous les mois, définissez votre budget, puis cliquez sur **Enregistrer**.

Pour importer un fichier pour le budget annuel :

1. Sous **Actions**, sélectionnez **Exporter** afin de télécharger un modèle CSV vide sur lequel baser le budget.
2. Remplissez le fichier CSV avec les entrées de votre budget, puis enregistrez-le localement.
3. Sous **Actions**, sélectionnez **Importer**.
4. Sélectionnez votre fichier enregistré, puis cliquez sur **OK**.

Pour exporter votre budget terminé dans un fichier CSV, sous **Actions**, sélectionnez **Exporter** afin de télécharger le fichier.

Une fois terminé, votre budget apparaît dans les rapports d’analyse des coûts et dans le rapport comparatif des coûts et du budget dans le temps. Il est également possible de planifier des rapports en fonction de seuils budgétaires.

### <a name="azure-resource-explorer-report"></a>Rapport Azure Resource Explorer

Le rapport Azure Resource Explorer présente la liste en bloc de toutes les ressources Azure disponibles dans Cloudyn. Les indicateurs étendus doivent être activés sur vos comptes Azure pour que vous puissiez utiliser efficacement ce rapport. Ils permettent à Cloudyn d’accéder à vos machines virtuelles Azure. Pour plus d’informations, voir [Ajouter des indicateurs étendus pour les machines virtuelles Azure](azure-vm-extended-metrics.md).

### <a name="azure-resources-over-time-report"></a>Rapport des ressources Azure dans le temps

Le rapport des ressources Azure dans le temps présente la répartition de toutes les ressources en cours d’exécution sur une période spécifique. Les indicateurs étendus doivent être activés sur vos comptes Azure pour que vous puissiez utiliser efficacement ce rapport. Ils permettent à Cloudyn d’accéder à vos machines virtuelles Azure. Pour plus d’informations, voir [Ajouter des indicateurs étendus pour les machines virtuelles Azure](azure-vm-extended-metrics.md).

### <a name="instance-explorer-report"></a>Rapport de l’Explorateur d’instances

Le rapport de l’Explorateur d’instances sert à afficher différents indicateurs sur les composants de vos machines virtuelles. Vous pouvez explorer certaines instances en particulier pour afficher ce type d’informations :
- Intervalles d’exécution des instances
- Cycle de vie dans la période sélectionnée
- Utilisation du processeur
- Entrée réseau
- Trafic de sortie
- Disques actifs

Le rapport de l’Explorateur d’instances collecte tous les intervalles d’exécution sur la plage de dates définie et agrège les données en conséquence. Pour afficher chacun de ces intervalles, développez l’instance. Le coût de chaque instance est calculé pour la plage de dates sélectionnée en fonction des prix catalogue AWS et Azure. Aucune remise ne s’applique. Vous pouvez ajouter des champs au rapport à l’aide de la fonction Afficher/masquer les champs.

Utilisez le rapport de l’Explorateur d’instances pour :

- Calculer le coût estimé par machine.
- Créer la liste complète, heures d’exécution agrégées comprises, de toutes les machines qui étaient actives pendant un intervalle de temps.
- Créer une liste par fournisseur de services cloud ou par compte.
- Afficher les machines créées ou terminées pendant un intervalle de temps.
- Afficher toutes les machines actuellement arrêtées.
- Afficher les balises de chaque machine.

### <a name="instances-over-time-report"></a>Rapport des instances dans le temps

Le rapport des instances dans le temps permet de voir le nombre maximal de machines qui étaient actives pendant la période sélectionnée. Si la résolution est définie par semaine ou par mois, les résultats correspondent au nombre maximal de machines actives sur n’importe quel jour de ce mois. Sélectionnez une plage de dates pour choisir les filtres à afficher dans le rapport.

### <a name="instance-utilization-over-time-report"></a>Rapport de l’utilisation des instances dans le temps

Ce rapport présente la répartition de l’utilisation du processeur ou de la mémoire au fil du temps pour toutes les instances.

### <a name="compute-power-cost-over-time-report"></a>Rapport des coûts de la puissance de calcul dans le temps

Le rapport de la puissance de calcul dans le temps présente la répartition de la puissance de calcul sur une plage de dates spécifiée. À la différence des rapports qui montrent le nombre de machines en cours d’exécution ou les heures d’exécution, celui-ci présente les heures cœur, les heures d’unités Compute ou les heures de Go de RAM.

Utilisez le rapport pour :

- Vérifier la puissance de calcul sur une plage de dates spécifiée.
- Afficher les temps de calcul en fonction des modèles d’affectation des coûts.

Ce rapport est lié aux stratégies [Cost Allocation 360](tutorial-manage-costs.md#use-custom-tags-to-allocate-costs) établies ; ainsi, les résultats s’affichent en fonction du balisage et des stratégies définis dans la stratégie de coûts sélectionnée. Si aucune stratégie n’a été créée, les résultats n’apparaissent pas.

### <a name="compute-power-average-cost-over-time-report"></a>Rapport des coûts moyens de la puissance de calcul dans le temps

Le rapport des coûts moyens de la puissance de calcul dans le temps permet d’afficher plus d’informations que le coût de chaque machine en cours d’exécution : le coût moyen par heure d’instance, par heure cœur, par heure d’unité Compute et par heure de Go de RAM. Il donne des insights sur l’efficacité du déploiement.

Ce rapport est lié aux stratégies [Cost Allocation 360](tutorial-manage-costs.md#use-custom-tags-to-allocate-costs) établies ; ainsi, les résultats s’affichent en fonction du balisage et des stratégies définis dans la stratégie de coûts sélectionnée. Si aucune stratégie n’a été créée, les résultats n’apparaissent pas.

### <a name="s3-cost-over-time-report"></a>Rapport des coûts S3 dans le temps

Le rapport des coûts S3 dans le temps présente la répartition des coûts d’Amazon Simple Storage Service (S3) par compartiment au fil du temps sur une période spécifiée. Il permet de trouver les compartiments qui représentent les plus gros postes de dépenses et présente les tendances d’utilisation et de dépenses de S3.

### <a name="s3-distribution-of-cost-report"></a>Rapport de la distribution des coûts S3

Ce rapport permet d’analyser les coûts S3 du mois dernier par compartiment et par classe de stockage. Vous pouvez utiliser l’affichage sous forme de graphique à secteurs pour définir le seuil de visibilité, ou le mode tableau pour voir les sous-totaux.

### <a name="s3-bucket-properties-report"></a>Rapport des propriétés des compartiments S3

Ce rapport permet d’afficher les propriétés des compartiments S3. Vous pouvez utiliser l’affichage sous forme de graphique à secteurs pour définir le seuil de visibilité, ou le mode tableau pour voir les sous-totaux.

### <a name="rds-instances-over-time-report"></a>Rapport des instances RDS dans le temps

Ce rapport permet d’afficher la répartition de toutes les instances Amazon Relational Database Service (RDS) en cours d’exécution pendant la période spécifiée.

### <a name="rds-active-instances-report"></a>Rapport des instances actives RDS

Ce rapport permet d’analyser les instances actives RDS. Développez l’élément de ligne pour afficher des informations supplémentaires.

### <a name="azure-reserved-instances-report"></a>Rapport des instances réservées Azure

Le rapport des instances réservées Azure centralise toutes les instances réservées Azure. Il affiche chaque achat sous la forme d’un élément de ligne distinct. Il présente également des détails sur cet achat : le compte ayant servi à l’achat, le type d’achat et le type d’instance, les jours restants, etc. Vous pouvez afficher ou masquer des données du rapport à l’aide de la fonction Afficher/masquer les champs.

Utilisez le rapport des instances réservées Azure pour afficher :

- La liste de toutes les réservations par date d’achat.
- Le temps restant avant expiration de l’instance réservée.
- Les frais ponctuels.
- Le compte ayant servi à acheter les instances réservées et la date d’achat.

### <a name="aws-reserved-instances-report"></a>Rapport des instances réservées AWS

Le rapport des instances réservées AWS centralise toutes les instances réservées AWS. Il affiche chaque achat sous la forme d’un élément de ligne distinct et donne des détails sur cet achat : le compte ayant servi à l’achat, le type d’achat et le type d’instance, les jours restants, etc. Vous pouvez afficher ou masquer des données du rapport à l’aide de la fonction Afficher/masquer les champs.

Utilisez le rapport des instances réservées AWS pour afficher :

- La liste de toutes les réservations par date d’achat.
- Le temps restant avant expiration de l’instance réservée.
- Les frais ponctuels.
- L’ID d’achat d’origine (ID de réservation).
- Le compte ayant servi à acheter les instances réservées et la date d’achat.

### <a name="ec2-ri-buying-recommendations-report"></a>Rapport des recommandations d’achat d’instances réservées EC2

Le fondement de la consommation de ressources du cloud est le modèle à la demande, selon lesquels seules les ressources utilisées engendrent un coût. Vous payez uniquement ce que vous utilisez, lorsque vous l’utilisez, sans engagement préalable.

AWS propose un autre modèle tarifaire pour ses services Elastic Compute Cloud (EC2) : l’instance réservée. Ce modèle garantit de la capacité aux utilisateurs chaque fois qu’ils en ont besoin pendant toute la durée de l’instance réservée. Cela représente des réductions significatives par rapport aux prix à la demande. En retour, les utilisateurs prennent un engagement initial concernant l’utilisation d’une instance virtuelle. Cet engagement est lié à une famille, une taille, une zone de disponibilité et un système d’exploitation spécifiques sur toute la période d’engagement (un ou trois ans). L’instance réservée permet à AWS de planifier efficacement la capacité future, ainsi que d’obtenir l’engagement des clients concernant l’utilisation de ses services.

Il existe trois options de paiement pour les instances réservées :

- Paiement initial intégral : somme en bloc au jour 0 (remise la plus élevée).
- Pas de paiement initial : versements mensuels couvrant le coût de l’instance réservée sur toute la durée de cette instance (remise la plus faible).
- Paiement initial partiel : règlement à l’avance du quart ou de la moitié du prix et versements mensuels pour le reste (remise inférieure, mais de peu, à celle du paiement initial intégral).

Cloudyn évalue la durée de fonctionnement de chaque machine sur les 30 derniers jours. Il recommande l’achat d’instances réservées quand il est plus rentable d’exécuter la machine avec une instance réservée pour la durée de fonctionnement actuelle.

Le rapport présente la justification de ses recommandations visant à maximiser les économies sur l’année. Les recommandations suggèrent de remplacer les instances à la demande par des instances réservées. Il est possible d’acheter directement des instances réservées à partir du rapport.

Chaque onglet représente un rapport complet. Voici les sections principales de ces onglets :

- **Impact de l’achat d’instances réservées EC2** : cette section présente une simulation de la différence entre les instances à la demande et les instances réservées. Cliquez sur **Zoom avant** pour voir le rapport complet d’impact de l’achat d’instances réservées EC2 avec les filtres déjà définis pour votre recommandation. Ce rapport présente l’impact de tous les achats potentiels d’instances réservées. Vous pouvez ajuster la durée de fonctionnement moyenne attendue pour voir les économies potentiellement réalisables en achetant des instances réservées EC2.

- **Analyse des économies** : cette section présente les économies potentielles et le mois pendant lequel les économies sont actualisées en suivant les recommandations de Cloudyn. Les économies réelles et le pourcentage d’économies apparaissent en rouge.

- **Comparaison du type d’instance réservée EC2** : cette section met l’accent sur le retour sur investissement du déploiement recommandé de Cloudyn, avec toutes les options pertinentes. Les résultats de ce rapport supposent que la machine a une durée de fonctionnement de 100 %. Cliquez sur **Zoom avant** pour ouvrir le rapport détaillé.

- **Instances dans le temps** : cette section présente la répartition de toutes les instances associées à la recommandation (à la demande, réservées et spot). Cliquez sur **Zoom avant** pour ouvrir le rapport détaillé.
- **Seuils de rentabilité** : cette section présente un tableau de tous les déploiements recommandés possibles, du retour sur investissement et du mois associé. Cliquez sur **Zoom avant** pour ouvrir le rapport détaillé.

### <a name="ec2-reservations-over-time-report"></a>Rapport des réservations EC2 dans le temps

Le rapport des réservations EC2 dans le temps effectue le suivi de l’utilisation des instances réservées EC2 achetées. La résolution du rapport peut être définie par heure, par jour ou par semaine.

Utilisez le rapport pour :

- Afficher les réservations achetées, utilisées et non utilisées.
- Affiner la résolution afin de voir l’utilisation des instances réservées par heure.

### <a name="savings-over-time-report"></a>Rapport des économies dans le temps

Le rapport des économies dans le temps permet d’afficher les économies réalisées avec les instances réservées et les instances spot. Il présente l’évolution du retour sur investissement obtenu au fil du temps grâce aux achats d’instances réservées.

Pour afficher les économies réalisées grâce aux instances réservées, regroupez les résultats par **Modèle de prix** et sélectionnez **Réservation**. Si vous souhaitez afficher celles qui ont été obtenues par un type de compte ou d’instance spécifique, ajoutez le filtre et le regroupement appropriés à ce type de compte ou d’instance.

Pour voir les économies réalisées grâce à l’utilisation d’instances spot, filtrez le **Modèle de prix** sur **Spot**. Par défaut, ce rapport est filtré sur les instances réservées et les instances Spot.

### <a name="rds-ri-buying-recommendations-report"></a>Rapport des recommandations d’achat d’instances réservées RDS

Le rapport des recommandations d’achat d’instances réservées RDS indique quand utiliser des instances réservées plutôt que des instances à la demande.

Chaque onglet représente un rapport complet. Voici les sections principales de ces onglets :

- **Impact de l’achat d’instances réservées RDS** : cette section présente une simulation de la différence entre les instances à la demande et les instances réservées. Cliquez sur **Zoom avant** pour voir le rapport complet de l’impact de l’achat d’instances réservées RDS avec les filtres déjà définis pour votre recommandation. Ce rapport permet de voir l’impact de tous les achats potentiels d’instances réservées.  Vous pouvez ajuster la durée de fonctionnement moyenne attendue et voir les économies réalisables en achetant des instances réservées.
- **Analyse des économies** : cette section présente les économies potentielles et le mois pendant lequel les économies sont actualisées en suivant les recommandations de Cloudyn. Les économies réelles et le pourcentage d’économies apparaissent en rouge.

- **Comparaison du type d’instance réservée RDS** : cette section met l’accent sur le retour sur investissement du déploiement recommandé, avec toutes les options pertinentes. Les résultats de ce rapport supposent que la machine a une durée de fonctionnement de 100 %. Cliquez sur **Zoom avant** pour ouvrir le rapport détaillé de la machine sélectionnée.
- **Instances dans le temps** : cette section présente la répartition de toutes les instances associées à la recommandation (à la demande, réservées et spot). Cliquez sur **Zoom avant** pour ouvrir le rapport détaillé.

- **Seuils de rentabilité** : cette section présente un tableau de tous les déploiements recommandés possibles, du retour sur investissement et du mois associé. Cliquez sur **Zoom avant** pour ouvrir le rapport détaillé.

### <a name="rds-reservations-over-time-report"></a>Rapport des réservations RDS dans le temps

Le rapport des réservations RDS dans le temps permet d’afficher la répartition des réservations utilisées et non utilisées pendant la période spécifiée.

### <a name="reserved-instance-purchase-impact-report"></a>Rapport de l’impact de l’achat d’instances réservées

Le rapport de l’impact de l’achat d’instances réservées EC2 permet de simuler le coût des instances réservées et de le comparer avec celui des instances à la demande au fil du temps. Il aide à prendre de bonnes décisions d’achat. Ajustez les filtres (temps d’exécution moyen, période, plateforme et autres) pour pouvoir acheter des instances réservées en connaissance de cause.

### <a name="cost-effective-sizing-recommendations-report"></a>Rapport des recommandations de dimensionnement rentable

Le rapport des recommandations de dimensionnement rentable donne des résultats pour AWS et Azure. Pour les utilisateurs AWS, les achats d’instances réservées sont pris en compte et les résultats n’incluent pas les machines fonctionnant en tant qu’instances réservées. Ce rapport donne la liste des instances sous-utilisées candidates au downsizing. Les recommandations s’appuient sur les données de performances et d’utilisation des 30 derniers jours. Chacune comporte une liste de candidats, la justification de ce downsizing et un lien permettant d’afficher la totalité des informations et les indicateurs de performance de l’instance, ainsi que la recommandation éventuelle de passer à une génération plus récente de types d’instances.

Il n’est pas possible de télécharger la liste des ID des instances pour lesquelles le downsizing est recommandé à partir de ce rapport. Utilisez pour cela le rapport de l’ensemble des recommandations de dimensionnement.

Prenons les exemples de downsizing suivant :

Vous avez six instances m3.xlarge en cours d’exécution. L’analyse Cloudyn montre que cinq d’entre elles affichent une faible utilisation du processeur. Envisagez un downsizing pour celles-ci.

La section Impact sur les coûts calcule l’impact sur les coûts. Dans cet exemple, en développant l’élément de ligne, vous pouvez voir qu’une instance m3.xlarge (Linux/Unix) coûte actuellement 0,266 $/heure et une instance m3.large (Linux/Unix) 0,133 $/heure. Par conséquent, le coût annuel est de 11,651 $ pour cinq instances m3.xlarge utilisées à 100 %. Il est de 5,825 $ pour cinq instances m3.large utilisées à 100 %. Les économies potentielles s’élèvent à 5,825 $.

Si vous souhaitez afficher les justifications du dimensionnement rentable, cliquez sur + pour développer l’élément de ligne. Dans **Détails** :

- La section **Justification des recommandations** présente le déploiement actuel et le nombre d’instances pour lesquelles un downsizing est recommandé.
- La section **Impact sur les coûts** présente le calcul utilisé pour déterminer les économies potentielles.
- La section **Économies annuelles potentielles** présente les économies annuelles potentielles du downsizing recommandé par Cloudyn.

### <a name="all-sizing-recommendations-report"></a>Rapport de l’ensemble des recommandations de dimensionnement

Ce rapport donne la liste des instances sous-utilisées candidates au downsizing. Les recommandations s’appuient sur les données de performances et d’utilisation des 30 derniers jours. Il est possible d’afficher pour chacune la totalité des informations et les indicateurs de performance de l’instance.

Si vous avez acheté des instances réservées AWS, ce rapport contient les résultats de toutes les instances en cours d’exécution, y compris les instances réservées.

Utilisez le rapport de l’ensemble des recommandations de dimensionnement pour :

- Voir la liste de toutes les instances candidates au downsizing.
- Exporter la liste du rapport contenant les ID et noms d’instances.

Si vous souhaitez afficher les détails de la recommandation d’une instance spécifique, cliquez sur **+** pour développer les détails. La section Détails de la recommandation offre une vue d’ensemble de la recommandation.

La section **Balises** donne la liste des clés et des valeurs de balise pour l’instance sélectionnée. Utilisez les balises qui se trouvent dans le volet gauche pour filtrer la section.

La section **Utilisation du processeur** présente l’utilisation du processeur par l’instance au cours du mois dernier, par jour.

Cliquez sur le graphique pour descendre dans la hiérarchie et ouvrez le rapport du processeur dans le temps pour voir la répartition des instances.

- Utilisez **Afficher/masquer des champs** pour ajouter ou supprimer des champs : Timestamp, Utilisation moyenne du processeur, Utilisation minimale du processeur, Utilisation maximale du processeur.
- Utilisez la fonction **Plage de dates** pour entrer une date ou une plage de dates et explorer un ID d’instance en particulier.
- Utilisez la fonction **Filtres étendus** pour afficher tous les ID d’instances ou un en particulier.
- Cliquez sur **Zoom avant** pour ouvrir le rapport de l’utilisation du processeur.

Si l’instance n’a pas été analysée depuis 30 jours, les données qui s’affichent sont incomplètes.

La section **Utilisation de mémoire (Go)** donne des informations sur la mémoire utilisée. Pour les utilisateurs AWS, les indicateurs liés à la mémoire ne sont pas disponibles automatiquement ; ils doivent être ajoutés instance par instance avec AWS. AWS impose des frais si vous les activez pour des instances EC2.

La section **Utilisation de la mémoire (%)** présente le pourcentage de mémoire utilisé.

La section **Trafic d’entrée réseau** présente un instantané au fil du temps du trafic réseau, la moyenne et le maximum, pour l’instance sélectionnée. Placez le curseur sur les lignes pour voir la date et le trafic maximal à cet instant. Cliquez sur **Zoom avant** pour ouvrir le rapport du trafic d’entrée réseau.

La section **Trafic de sortie réseau** présente un instantané du trafic de sortie réseau pour l’instance sélectionnée. Placez le curseur sur les lignes pour voir la date et le trafic maximal à cet instant. Cliquez sur **Zoom avant** pour ouvrir le rapport du trafic de sortie réseau.

### <a name="instance-metrics-explorer-report"></a>Rapport Metrics Explorer des instances

Le rapport Metrics Explorer des instances présente les indicateurs de performances intercloud par instance. Il permet d’afficher les instances sur- ou sous-utilisés en fonction de seuils d’indicateurs liés au processeur, à la mémoire et au réseau.

Pour afficher les performances intercloud par instance :

1. Dans **Plage de dates**, sélectionnez la plage de dates pour laquelle vous souhaitez afficher les performances.
2. Dans **Balises**, sélectionnez toutes les balises que vous souhaitez afficher.
3. Dans **Filtres**, sélectionnez les filtres que vous souhaitez afficher dans le rapport.
4. Dans **Filtres étendus**, ajustez les seuils du rapport pour les données suivantes :
    - Utilisation moyenne du processeur
    - Utilisation maximale du processeur
    - Utilisation moyenne de la mémoire
    - Utilisation maximale de la mémoire
5. Dans **Filtres étendus**, cliquez sur **Afficher**, puis sélectionnez le type d’instances à afficher.

Pour afficher l’évolution des indicateurs d’une instance spécifique au fil du temps :

- Accédez au rapport Metrics Explorer des instances et cliquez sur **+** pour afficher les détails.

### <a name="rds-sizing-recommendations-report"></a>Rapport des recommandations de dimensionnement RDS

Le rapport des recommandations de dimensionnement RDS présente des recommandations de dimensionnement RDS pour optimiser l’utilisation du cloud. Il donne la liste des instances sous-utilisées candidates au downsizing. Les recommandations de Cloudyn s’appuient sur les données de performances et d’utilisation des 30 derniers jours. Vous pouvez filtrer les recommandations par nom de compte, région, type d’instance et état.

### <a name="sizing-threshold-manager-report"></a>Rapport du gestionnaire de seuils de dimensionnement

Les recommandations de dimensionnement intégrées de Cloudyn sont calculées suivant un algorithme complexe pour présenter des suggestions de dimensionnement précises. Il est possible d’ajuster les seuils des recommandations de downsizing.

Pour ajuster manuellement les seuils des recommandations de dimensionnement :

1. Dans le gestionnaire des seuils de dimensionnement, ajustez les seuils suivants comme vous le souhaitez :
    - % moyen du processeur
    - % maximal du processeur
    - % moyen de la mémoire
    - % maximal de la mémoire
3. Cliquez sur **Appliquer** pour enregistrer les modifications.
4. Les modifications s’appliquent immédiatement à toutes les recommandations.

Pour restaurer les seuils par défaut :

- Dans le gestionnaire des seuils de dimensionnement, cliquez sur **Restaurer les paramètres par défaut**.

### <a name="compute-instance-types-report"></a>Rapport des types d’instances de calcul

Utilisez le rapport des types d’instances pour :

- Afficher les types d’instances par service, famille, nom de l’API et nom.
- Afficher ses détails comme le processeur, les unités Compute EC2, la RAM et le réseau.

Vous pouvez utiliser la fonction **Rechercher** pour rechercher des éléments de ligne spécifiques.

## <a name="next-steps"></a>Étapes suivantes

- Pour découvrir comment utiliser les rapports, notamment personnaliser ou enregistrer et planifier des rapports, voir [Comprendre les rapports de coûts](understanding-cost-reports.md).
- Pour plus d’informations sur les tableaux de bord inclus dans Cloudyn et pour savoir comment créer ses propres tableaux de bord personnalisés, voir [Afficher les indicateurs de coûts clés avec des tableaux de bord](dashboards.md).

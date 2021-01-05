---
title: Guide de démarrage rapide - Explorer les coûts Azure avec l’analyse du coût
description: Ce guide de démarrage rapide vous aide à utiliser l’analyse du coût pour explorer et analyser les coûts Azure de votre organisation.
author: bandersmsft
ms.author: banders
ms.date: 11/20/2020
ms.topic: quickstart
ms.service: cost-management-billing
ms.subservice: cost-management
ms.reviewer: micflan
ms.custom: contperf-fy21q2
ms.openlocfilehash: 1926a5a2ee81b6be4abee5e4064a4a23354da1a1
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033576"
---
# <a name="quickstart-explore-and-analyze-costs-with-cost-analysis"></a>Démarrage rapide : Explorer et analyser les coûts avec l’analyse du coût

Pour pouvoir contrôler et optimiser vos coûts Azure correctement, vous devez comprendre d’où proviennent les coûts au sein de votre organisation. Il est également utile de connaître le coût de vos services, notamment en fonction des environnements et des systèmes. La visibilité de la totalité des coûts est essentielle pour comprendre avec précision les modèles de dépenses de l’organisation. Vous pouvez utiliser des modèles de dépenses pour appliquer des mécanismes de contrôle des coûts, tels que des budgets.

Dans ce guide de démarrage rapide, vous utilisez l’analyse du coût pour explorer et analyser les coûts Azure de votre organisation. Vous pouvez voir les coûts agrégés par l’organisation afin de comprendre d’où ils viennent et d’identifier les tendances de dépenses. Vous pouvez aussi voir les coûts cumulés au fil du temps pour évaluer des tendances mensuelles, trimestrielles ou même annuelles par rapport à un budget. Un budget permet de faire respecter des contraintes financières. Il est aussi utilisé pour voir les coûts de chaque jour ou de chaque mois afin d’isoler les irrégularités dans les dépenses. De plus, vous pouvez télécharger les données du rapport actuel en vue de les analyser de manière plus approfondie ou de les utiliser dans un système externe.

Dans ce guide de démarrage rapide, vous apprenez à :

- Passer en revue les coûts dans l’analyse du coût
- Personnaliser les vues des coûts
- Télécharger les données de l’analyse du coût

## <a name="prerequisites"></a>Prérequis

L’analyse des coûts prend en charge différents types de comptes Azure. Pour accéder à la liste complète des types de comptes pris en charge, voir [Comprendre les données de Cost Management](understand-cost-mgt-data.md). Pour afficher les données de coût, vous avez au minimum besoin d’un accès en lecture pour votre compte Azure.

Pour plus d’informations sur l’attribution de l’accès aux données Azure Cost Management, consultez [Assigner l’accès aux données](./assign-access-acm-data.md).

Si vous disposez d’un nouvel abonnement, vous ne pouvez pas utiliser les fonctionnalités de Cost Management tout de suite. Vous risquez de devoir attendre jusqu’à 48 heures avant de pouvoir utiliser toutes les fonctionnalités de Cost Management.

## <a name="sign-in-to-azure"></a>Connexion à Azure

- Connectez-vous au portail Azure sur https://portal.azure.com.

## <a name="review-costs-in-cost-analysis"></a>Passer en revue les coûts dans l’analyse du coût

Pour examiner vos coûts via la fonctionnalité d’analyse des coûts, ouvrez l’étendue dans le portail Azure et sélectionnez **Analyse du coût** dans le menu. Par exemple, accédez à **Abonnements**, sélectionnez un abonnement dans la liste, puis sélectionnez **Analyse du coût** dans le menu. Utilisez le paramètre **Étendue** pour passer à une autre étendue dans l’analyse des coûts.

L’étendue que vous sélectionnez est utilisée à travers Cost Management pour consolider les données et contrôler l’accès aux informations sur les coûts. Quand vous utilisez des étendues, vous n’opérez pas une sélection multiple. Au lieu de cela, vous sélectionnez une étendue plus grande qui en regroupe d’autres, puis vous filtrez jusqu’aux portées imbriquées dont vous avez besoin. Il est important de comprendre cette approche, car il est possible que certaines personnes n’aient pas accès à une étendue parente unique qui couvre plusieurs étendues imbriquées.

Regardez la vidéo [Guide pratique pour utiliser Cost Management dans le portail Azure](https://www.youtube.com/watch?v=mfxysF-kTFA) pour en savoir plus sur l’utilisation de l’analyse du coût. Pour regarder d’autres vidéos, consultez la [chaîne YouTube relative à Cost Management](https://www.youtube.com/c/AzureCostManagement).

>[!VIDEO https://www.youtube.com/embed/mfxysF-kTFA]

L’affichage initial de l’analyse du coût inclut les zones suivantes.

**Vue Coûts cumulés** : représente la configuration de la vue Coûts cumulés prédéfinie. Chaque vue comprend les options Plage de dates, Granularité, Regrouper par et Ajouter un filtre. La vue par défaut montre les coûts cumulés pour la période de facturation actuelle. Vous pouvez cependant choisir une autre vue intégrée.

**Coût réel** : montre le total de l’utilisation et des coûts d’achat pour le mois en cours, tels qu’ils sont comptabilisés et qu’ils apparaissent sur votre facture.

**Prévision** : affiche le total des coûts prévisionnels pour la période de votre choix

**Budget** : indique la limite de dépenses planifiée pour l’étendue sélectionnée, si disponible.

**Granularité : Cumulés** : montre le total des dépenses quotidiennes cumulées, depuis le début de la période de facturation. Après avoir créé un budget pour votre compte de facturation ou votre abonnement, vous pouvez rapidement voir la tendance de vos dépenses par rapport au budget. Placez le curseur sur une date pour voir le coût cumulé de cette journée.

**Graphiques croisés dynamiques (anneau)**  : fournissent des tableaux croisés dynamiques où le coût total est détaillé par un ensemble commun de propriétés standard. Ils affichent les coûts des plus élevés au plus bas pour le mois en cours. Vous pouvez changer les graphiques croisés dynamiques à tout moment en sélectionnant un critère de croisement différent. Les coûts sont classés par service (catégorie comptage), emplacement (région) et étendue enfant par défaut. Par exemple, les comptes d’inscription figurent sous les comptes de facturation, les groupes de ressources sous les abonnements et les ressources sous les groupes de ressources.

![Vue initiale de l’analyse des coûts dans le Portail Azure](./media/quick-acm-cost-analysis/cost-analysis-01.png)

### <a name="understand-forecast"></a>Comprendre les prévisions

La prévision des coûts montre une projection de vos coûts estimés pour la période de temps sélectionnée. Le modèle est basé sur un modèle de régression de séries chronologiques. Il nécessite au moins 10 jours de données récentes de coûts et d’utilisation pour prévoir les coûts avec précision. Pour une période de temps donnée, le modèle de prévision nécessite des parties égales de données d’entraînement pour la période de prévision. Par exemple, une projection à trois mois nécessite au moins trois mois de données récentes sur les coûts et l’utilisation.

Le modèle utilise un maximum de six mois de données d’entraînement afin de prévoir les coûts pour une année. Au minimum, il a besoin de sept jours de données d’entraînement pour changer sa prédiction. La prédiction est basée sur les changements importants dans les modèles de coût et d’utilisation, comme les pics et les creux. La prévision ne génère pas de projections individuelles pour chaque élément des propriétés **Regrouper par**. Il fournit uniquement une prévision du total des coûts cumulés. Si vous utilisez plusieurs devises, le modèle fournit des prévisions seulement pour les coûts exprimés en USD.

## <a name="customize-cost-views"></a>Personnaliser les vues des coûts

L’analyse des coûts dispose de quatre affichages intégrés, optimisés pour les objectifs les plus courants :

Affichage | Répondre aux questions comme
--- | ---
Coûts cumulés | Combien ai-je dépensé jusqu’ici ce mois-ci ? Est-ce que je vais respecter mon budget ?
Coûts quotidiens | Y a-t-il eu une augmentation des coûts par jour pour les 30 derniers jours ?
Coût par service | Comment mon utilisation mensuelle a-t-elle varié sur les trois dernières factures ?
Coût par ressource | Quelles ressources ont été les plus onéreuses jusqu’ici ce mois-ci ?
Détails de la facture | Quels frais figuraient sur ma dernière facture ?

![Sélecteur d’affichage montrant un exemple de sélection pour ce mois](./media/quick-acm-cost-analysis/view-selector.png)

Toutefois, il existe de nombreux cas où vous avez besoin d’une analyse plus approfondie. La personnalisation démarre en haut de la page, avec la sélection de la date.

L’analyse du coût affiche les données du mois en cours par défaut. Utilisez le sélecteur de date pour passer rapidement à des plages de dates communes. Il peut s’agit, par exemple, des sept derniers jours, du mois dernier, de l’année en cours ou de une plage de dates personnalisée. Les abonnements de paiement à l’utilisation incluent également des plages de dates en fonction de votre période de facturation (qui n’est pas liée au mois calendaire), telles que la période de facturation actuelle ou la dernière facture. Utilisez les liens **< PRÉCÉDENT** et **SUIVANT >** en haut du menu pour accéder respectivement à la période précédente ou suivante. Par exemple, **<PRÉCÉDENT** passe de la période **7 derniers jours** aux périodes **8 à 14 jours passés**, puis **15 à 21 jours passés**. Lorsque vous sélectionnez une plage de dates personnalisée, gardez à l’esprit que vous pouvez sélectionner jusqu’à une année complète (par exemple du 1er janvier au 31 décembre).

![Sélecteur de date affichant un exemple de sélection pour ce mois](./media/quick-acm-cost-analysis/date-selector.png)

L’analyse du coût montre les coûts **cumulés** par défaut. Les coûts cumulés incluent tous les coûts de chaque jour plus des jours précédents, pour obtenir une vue en évolution constante de vos coûts quotidiens cumulés. Cette vue est optimisée pour montrer les tendances par rapport à un budget pendant l’intervalle de temps sélectionné.

La vue de graphique de la prévision vous permet d’identifier les dépassements de budget potentiels. Lorsqu’une possibilité de dépassement du budget est détectée, les dépenses excédentaires prévues sont affichées en rouge. Un indicateur s’affiche également dans le graphique. Lorsque vous placez le curseur sur cet indicateur, la date estimée du dépassement de budget s’affiche.

![Exemple montrant un dépassement de budget potentiel](./media/quick-acm-cost-analysis/budget-breach.png)

Il existe aussi la vue **quotidienne** qui montre les coûts de chaque jour. La vue quotidienne n’affiche pas de tendance géométrique. Elle est conçue pour montrer les irrégularités comme les pics ou les chutes de dépenses de chaque jour. Si vous avez sélectionné un budget, la vue quotidienne indique aussi une estimation de votre budget quotidien.

Quand vos coûts quotidiens sont constamment au-dessus du budget quotidien estimé, vous pouvez vous attendre à dépasser votre budget mensuel. Le budget quotidien estimé est un moyen pour vous aider à visualiser votre budget à un niveau inférieur. Si les coûts quotidiens présentent des fluctuations, la comparaison du budget quotidien estimé au budget mensuel est moins précise.

Voici une vue des dépenses récentes jour après jour, où la prévision des dépenses est activée.
![Vue jour après jour montrant des dépenses quotidiennes pour le mois en cours](./media/quick-acm-cost-analysis/daily-view.png)

Lorsque la prévision des dépenses est activée, vous ne voyez pas les dépenses qui sont prévues. En outre, lorsque vous regardez les coûts des périodes passées, les prévisions ne s’affichent pas non plus.

En général, attendez-vous à voir des données ou des notifications correspondant aux ressources utilisées dans les 8 à 12 heures suivantes.

**Regrouper par** propriétés communes pour ventiler les coûts et d’identifier les principaux contributeurs. Pour regrouper par balises de ressources, par exemple, sélectionnez la clé de balise d’après laquelle vous souhaitez effectuer le regroupement. Les coûts sont ventilés selon la valeur de chaque balise, avec un segment supplémentaire pour les ressources auxquelles cette balise n’a pas été appliquée.

La plupart des ressources Azure prennent en charge le balisage. Cependant, certaines balises ne sont pas disponibles pour la gestion des coûts et la facturation. Par ailleurs, les étiquettes de groupe de ressources ne sont pas prises en charge. La prise en charge des étiquettes s’applique à l’utilisation signalée *après* l’application de l’étiquette à la ressource. Les étiquettes ne sont pas appliquées rétroactivement pour les cumuls de coûts.

Regardez la vidéo [Comment examiner les stratégies de balises avec Azure Cost Management](https://www.youtube.com/watch?v=nHQYcYGKuyw) pour découvrir comment utiliser la stratégie de balises Azure pour améliorer la visibilité des données de coûts.

Voici une vue des coûts des services Azure pour le mois en cours.

![Vue cumulée quotidienne groupée présentant un exemple de coûts du service Azure pour le mois dernier](./media/quick-acm-cost-analysis/grouped-daily-accum-view.png)

Par défaut, l’analyse des coûts montre tous les coûts liés aux achats et à l’utilisation, tels qu’ils sont comptabilisés et qu’ils apparaissent sur votre facture. C’est ce qu’on appelle également le **coût réel**. L’affichage du coût réel est idéal pour le rapprochement de vos factures. Toutefois, des pics de dépenses peuvent être alarmants si vous surveillez les anomalies liées aux dépenses. Pour aplanir les pics de dépenses causés par les achats de réservation, basculez vers la vue **Coût amorti**.

![Passage de la vue Coût réel à la vue Coût amorti pour afficher les achats de réservation étalés sur la durée prévue et alloués aux ressources qui ont utilisé la réservation](./media/quick-acm-cost-analysis/metric-picker.png)

La vue Coût amorti répartit les achats de réservation jour par jour et les étale sur toute la durée de la réservation. Par exemple, au lieu de voir un achat d’un montant de 365 $ le 1er janvier, vous verrez un achat de 1,00 $ chaque jour, du 1er janvier au 31 décembre. En plus de l’amortissement de base, ces coûts sont également réaffectés et associés à l’aide des ressources qui ont utilisé la réservation. Par exemple, si cette dépense quotidienne de 1,00 $ a été répartie sur deux machines virtuelles, vous verrez deux dépenses de 0,50 $ pour la journée. Si une partie de la réservation n’est pas utilisée durant la journée, vous verrez une dépense de 0,50 $ associée à la machine virtuelle applicable, et une autre dépense de 0,50 $ de type `UnusedReservation`. Notez que les coûts liés aux réservations non utilisées ne sont visibles que lors de l’affichage du Coût amorti.

Étant donné que la représentation des coûts a changé, il est important de noter que les vues Coût réel et Coût amorti affichent des totaux différents. En règle générale, le coût total d’un achat de réservation étalé sur plusieurs mois diminue lorsque vous affichez les coûts amortis, et les coûts augmentent au cours des mois qui suivent un achat de réservation. L’amortissement est disponible uniquement pour les achats de réservation et ne s’applique pas aux achats effectués sur la Place de marché Azure.

L’image suivante montre les noms des groupes de ressources. Vous pouvez effectuer un regroupement par étiquette pour afficher les coûts totaux en fonction de l’étiquette, ou utiliser la vue **Coût par ressource** pour afficher toutes les étiquettes d’une ressource en particulier.

![Données complètes pour la vue actuelle présentant les noms de groupe de ressources](./media/quick-acm-cost-analysis/full-data-set.png)

Lorsque vous regroupez les coûts en fonction d’un attribut spécifique, les 10 principaux contributeurs aux coûts s’affichent dans l’ordre décroissant. S’il y en a plus de 10, les neuf principaux contributeurs aux coûts sont affichés avec un groupe **Autres** qui réunit tous les groupes restants. Lors du regroupement par balises, vous pouvez aussi voir un groupe **Non balisé** pour les coûts auxquels la clé de balise n’a pas été appliquée. **Sans balise** est toujours mentionné en dernier, même si les coûts sans balise sont plus élevés que ceux avec balise. Les coûts sans balise font partie du groupe **Autres** s’il y a 10 valeurs de balise ou plus. Passez à la vue du tableau et modifiez la granularité en **Aucune** pour afficher toutes les valeurs classées du prix le plus bas au plus haut.

Les ressources de machines virtuelles, de réseau et de stockage classiques ne partagent pas de données de facturation détaillées. Elles sont fusionnées dans les **services Classic** lors du regroupement des coûts.

Les graphiques croisés dynamiques sous le graphique principal affichent différents regroupements qui vous donnent une vue d’ensemble de vos coûts globaux pour la période et les filtres sélectionnés. Sélectionnez une propriété ou une étiquette pour afficher les coûts agrégés d’après n’importe quelle dimension.

![Exemples de graphiques croisés dynamiques](./media/quick-acm-cost-analysis/pivot-charts.png)

Vous pouvez afficher le jeu de données complet pour n’importe quelle vue. L’ensemble des sélections ou filtres que vous appliquez ont une incidence sur les données présentées. Pour voir le jeu de données complet, sélectionnez la liste **Type de graphique**, puis l’affichage **Tableau**.

![Données de l’affichage actuel présentées dans un tableau](./media/quick-acm-cost-analysis/chart-type-table-view.png)

## <a name="saving-and-sharing-customized-views"></a>Enregistrement et partage des vues personnalisées

Enregistrez des vues personnalisées et partagez-les avec d’autres utilisateurs en épinglant l’analyse des coûts au tableau de bord du portail Azure, ou en copiant un lien dans l’analyse des coûts.

Regardez la vidéo [Partager et enregistrer des vues dans Azure Cost Management](https://www.youtube.com/watch?v=kQkXXj-SmvQ) pour en savoir plus sur l’utilisation du portail pour partager les connaissances sur les coûts dans votre organisation. Pour regarder d’autres vidéos, consultez la [chaîne YouTube relative à Cost Management](https://www.youtube.com/c/AzureCostManagement).

>[!VIDEO https://www.youtube.com/embed/kQkXXj-SmvQ]

Pour épingler l’analyse des coûts, sélectionnez l’icône d’épingle dans le coin supérieur droit ou juste après « <Subscription Name> | Analyse des coûts ». L’épinglage de l’analyse des coûts enregistre uniquement la vue du graphique ou du tableau principal. Partagez le tableau de bord pour permettre à d’autres utilisateurs d’accéder à la vignette. Notez que seule la configuration du tableau de bord est partagée et que les autres utilisateurs n’auront pas accès aux données sous-jacentes. Si bous n’avez pas accès aux coûts mais bien à un tableau de bord partagé, vous voyez s’afficher le message « Accès refusé ».

Pour partager un lien vers l’analyse des coûts, sélectionnez **Partager** en haut du panneau. Une URL personnalisée s’affiche, qui permet d’ouvrir cette vue de l’étendue. Si vous n’avez pas accès aux coûts et obtenez cette URL, vous voyez s’afficher un message « accès refusé ».

## <a name="download-usage-data"></a>Télécharger les données sur l’utilisation

### <a name="portal"></a>[Portail](#tab/azure-portal)

Il arrive que vous deviez télécharger les données pour les analyser de manière plus approfondie, pour les fusionner avec vos propres données ou pour les intégrer à vos propres systèmes. Cost Management offre quelques options. Pour commencer, si vous avez besoin d’une synthèse de haut niveau ad hoc, comme vous pouvez en obtenir dans une analyse des coûts, créez la vue dont vous avez besoin. Ensuite, téléchargez-la en sélectionnant **Exporter**, puis **Télécharger les données au format CSV** ou **Télécharger les données vers Excel**. Le téléchargement vers Excel fournit un contexte supplémentaire au sujet de la vue que vous avez utilisée pour générer le téléchargement (comme l’étendue, la configuration des requêtes, le total ou la date de génération).

Si vous avez besoin du jeu de données complet et non agrégé, téléchargez-le à partir du compte de facturation. Ensuite, dans la liste des services dans le volet de navigation gauche du portail, accédez à **Gestion des coûts + facturation**. Sélectionnez éventuellement votre compte de facturation. Accédez à **Utilisation + frais**, puis sélectionnez l’icône **Télécharger** correspondant à la période de facturation souhaitée.

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Commencez par préparer votre environnement pour Azure CLI :

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../../includes/azure-cli-prepare-your-environment-no-header.md)]

Une fois connecté, utilisez la commande [az costmanagement query](/cli/azure/ext/costmanagement/costmanagement#ext_costmanagement_az_costmanagement_query) pour interroger les informations sur l’utilisation du mois en cours pour votre abonnement :

```azurecli
az costmanagement query --timeframe MonthToDate --type Usage \
   --scope "subscriptions/00000000-0000-0000-0000-000000000000"
```

Vous pouvez également affiner la requête à l’aide du paramètre **--dataset-filter** ou d’autres paramètres :

```azurecli
az costmanagement query --timeframe MonthToDate --type Usage \
   --scope "subscriptions/00000000-0000-0000-0000-000000000000" \
   --dataset-filter "{\"and\":[{\"or\":[{\"dimension\":{\"name\":\"ResourceLocation\",\"operator\":\"In\",\"values\":[\"East US\",\"West Europe\"]}},{\"tag\":{\"name\":\"Environment\",\"operator\":\"In\",\"values\":[\"UAT\",\"Prod\"]}}]},{\"dimension\":{\"name\":\"ResourceGroup\",\"operator\":\"In\",\"values\":[\"API\"]}}]}"
```

Le paramètre **--dataset-filter** prend une chaîne JSON ou `@json-file`.

Vous avez également la possibilité d’utiliser les commandes [az costmanagement export](/cli/azure/ext/costmanagement/costmanagement/export) pour exporter des données d’utilisation vers un compte Stockage Azure. Vous pouvez télécharger les données à partir de là.

1. Créez un groupe de ressources ou utilisez un groupe existant. Exécutez la commande [az group create](/cli/azure/group#az_group_create) pour créer un groupe de ressources :

   ```azurecli
   az group create --name TreyNetwork --location "East US"
   ```

1. Créez un compte de stockage pour recevoir les exportations, ou utilisez un compte de stockage existant. Pour créer un compte, utilisez la commande [az storage account create](/cli/azure/storage/account#az_storage_account_create) :

   ```azurecli
   az storage account create --resource-group TreyNetwork --name cmdemo
   ```

1. Exécutez la commande [az costmanagement export create](/cli/azure/ext/costmanagement/costmanagement/export#ext_costmanagement_az_costmanagement_export_create) pour créer l’exportation :

   ```azurecli
   az costmanagement export create --name DemoExport --type Usage \
   --scope "subscriptions/00000000-0000-0000-0000-000000000000" --storage-account-id cmdemo \
   --storage-container democontainer --timeframe MonthToDate --storage-directory demodirectory
   ```

---

## <a name="clean-up-resources"></a>Nettoyer les ressources

- Si vous avez épinglé une vue personnalisée pour l’analyse des coûts et que vous n’en avez plus besoin, accédez au tableau de bord où vous l’avez épinglé et supprimez la vue épinglée.
- Si vous avez téléchargé des fichiers de données d’utilisation et que vous n’en avez plus besoin, veillez à les supprimer.

## <a name="next-steps"></a>Étapes suivantes

Passez au premier tutoriel pour apprendre à créer et à gérer des budgets.

> [!div class="nextstepaction"]
> [Créer et gérer des budgets](tutorial-acm-create-budgets.md)
---
title: Gérer l’utilisation et les coûts à l’aide des journaux d’activité Azure Monitor
description: Découvrez comment changer le plan tarifaire et gérer la stratégie de rétention et de volume des données de votre espace de travail Log Analytics dans Azure Monitor.
services: azure-monitor
documentationcenter: azure-monitor
author: bwren
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: azure-monitor
ms.workload: na
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 11/22/2020
ms.author: bwren
ms.subservice: ''
ms.openlocfilehash: b84d24174771e8395677874c9dac863fa6f27a54
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96185910"
---
# <a name="manage-usage-and-costs-with-azure-monitor-logs"></a>Gérer l’utilisation et les coûts avec les journaux Azure Monitor    

> [!NOTE]
> Cet article explique en quoi consistent vos coûts pour les journaux Azure Monitor et comment les contrôler. Un article associé, [Surveiller l’utilisation et l’estimation des coûts](usage-estimated-costs.md), explique comment visualiser l’utilisation et les coûts estimés avec plusieurs fonctionnalités de surveillance Azure en fonction des différents modèles tarifaires. Tous les prix et coûts indiqués dans cet article sont fournis uniquement à titre d’exemple. 

Les journaux Azure Monitor sont conçus pour la mise à l’échelle et la prise en charge de la collecte, de l’indexation et du stockage de quantités importantes de données quotidiennes provenant de toute source de votre entreprise ou déployées dans Azure.  Si ce peut être un élément moteur pour votre organisation, la rentabilité est au final ce qui importe le plus. À cette fin, il est important de comprendre que le coût d’un espace de travail Log Analytics n’est pas seulement fonction du volume des données collectées, mais qu’il dépend aussi du plan sélectionné et de la durée de stockage des données générées à partir de vos sources connectées.  

Dans cet article, nous allons passer en revue les méthodes permettant de surveiller de façon proactive la croissance du stockage et du volume de données ingérées, et définir des limites pour contrôler les coûts associés. 

## <a name="pricing-model"></a>Modèle de tarification

Les tarifs par défaut de Log Analytics suivent un modèle de **paiement à l’utilisation**, basé sur le volume de données ingérées, et peuvent varier si la période de conservation des données est plus longue. Le volume de données s’exprime par la taille des données qui seront stockées en Go (10^9 octets). Chaque espace de travail Log Analytics est facturée en tant que service distinct et s’ajoute à la facture de votre abonnement Azure. La quantité de données ingérées peut être considérable en fonction des facteurs suivants : 

  - Nombre de solutions de gestion activées et leur configuration
  - Nombre de machines virtuelles surveillées
  - Type des données collectées à partir de chaque machine virtuelle surveillée 
  
En plus du modèle Paiement à l’utilisation, Log Analytics possède des niveaux de **réservation de capacité** qui vous permettent d’économiser jusqu’à 25 % par rapport au tarif de Paiement à l’utilisation. Les tarifs de la réservation de capacité vous permettent d’acheter une réservation à partir de 100 Go/jour. Toute utilisation au-dessus du niveau de réservation sera facturée au tarif de paiement à l’utilisation. Les niveaux de réservation de capacité ont une période d’engagement de 31 jours. Pendant la période d’engagement, vous pouvez passer à un niveau supérieur de réservation de capacité (ce qui relancera la période d’engagement de 31 jours), mais vous ne pouvez pas revenir à un Paiement à l’utilisation ou à un niveau inférieur de réservation de capacité avant que la période d’engagement ne soit terminée. La facturation des niveaux de réservation de capacité est effectuée quotidiennement. [En savoir plus](https://azure.microsoft.com/pricing/details/monitor/) sur les tarifs de la réservation de capacité et de paiement à l’utilisation de Log Analytics. 

À tous les niveaux tarifaires, la taille des données d’un événement est calculée à partir d’une représentation des propriétés sous forme d’une chaîne stockée dans Log Analytics pour cet événement, que les données soient envoyées par un agent ou qu’elles soient ajoutées pendant le processus d’ingestion. Cela comprend tous les [champs personnalisés](custom-fields.md) ajoutés à mesure que des données sont collectées, puis stockées dans Log Analytics. Plusieurs propriétés communes à tous les types de données, notamment certaines [propriétés standard de Log Analytics](./log-standard-columns.md), ne sont pas incluses dans le calcul de la taille de l’événement. Cela inclut `_ResourceId`, `_ItemId`, `_IsBillable`, `_BilledSize` et `Type`. Toutes les autres propriétés stockées dans Log Analytics sont incluses dans le calcul de la taille de l’événement. Certains types de données sont exempts de frais d’ingestion de données, par exemple les types Activité Azure, Pulsation et Utilisation. Pour déterminer si un événement est exclu de la facturation pour l’ingestion des données, vous pouvez utiliser la propriété `_IsBillable` comme illustré [ci-dessous](#data-volume-for-specific-events). L’utilisation est exprimée en Go (1,0E9 octets). 

Notez également que certaines solutions, telles que [Azure Security Center](https://azure.microsoft.com/pricing/details/security-center/), [Azure Sentinel](https://azure.microsoft.com/pricing/details/azure-sentinel/) et [Gestion de la configuration](https://azure.microsoft.com/pricing/details/automation/), ont leurs propres modèles tarifaires. 

### <a name="log-analytics-dedicated-clusters"></a>Clusters dédiés Log Analytics

Les clusters dédiés Log Analytics sont des collections d’espaces de travail regroupés dans un seul cluster Azure Data Explorer managé pour prendre en charge des scénarios avancés tels que les [clés gérées par le client](customer-managed-keys.md).  Les clusters Log Analytics dédiés utilisent un modèle de tarification de réservation de capacité qui doit être configuré sur au moins 1 000 Go/jour. Ce niveau de capacité offre une remise de 25 % par rapport à la tarification Paiement à l’utilisation. Toute utilisation au-dessus du niveau de réservation sera facturée au tarif de paiement à l’utilisation. La réservation de capacité de cluster a une période d’engagement de 31 jours après l’augmentation du niveau de réservation. Au cours de la période d’engagement, le niveau de réservation de capacité ne peut pas être réduit, mais il peut être augmenté à tout moment. Lorsque des espaces de travail sont associés à un cluster, la facturation d’ingestion des données de ces espaces de travail est effectuée au niveau du cluster à l’aide du niveau de réservation de capacité configuré. En savoir plus sur la [création de clusters d’analytique des journaux d'activité](customer-managed-keys.md#create-cluster) et [l’association de ces clusters à des espaces de travail](customer-managed-keys.md#link-workspace-to-cluster). Les informations sur la tarification de réservation de capacité sont disponibles sur la [page de tarification Azure Monitor]( https://azure.microsoft.com/pricing/details/monitor/).  

Le niveau de réservation de la capacité du cluster est configuré par programmation avec Azure Resource Manager à l’aide du paramètre `Capacity` sous `Sku`. La `Capacity` est spécifiée en unités de Go et peut avoir des valeurs de 1 000 Go/jour ou plus par incréments de 100 Go/jour. Ceci est expliqué en détail dans [Clé gérée par le client dans Azure Monitor](customer-managed-keys.md#create-cluster). Si votre cluster a besoin d’une réservation supérieure à 2 000 Go/jour, contactez-nous à l’adresse [LAIngestionRate@microsoft.com](mailto:LAIngestionRate@microsoft.com).

Il existe deux modes de facturation pour l’utilisation sur un cluster. Ils peuvent être spécifiés par le paramètre `billingType` lors de la [configuration de votre cluster](customer-managed-keys.md#customer-managed-key-operations). Les deux modes sont : 

1. **Cluster** : dans ce cas (valeur par défaut), la facturation des données ingérées est effectuée au niveau du cluster. Les quantités de données ingérées de chaque espace de travail associé à un cluster sont agrégées pour calculer la facture quotidienne du cluster. Notez que les répartitions par nœud d’[Azure Security Center](../../security-center/index.yml) sont appliquées au niveau de l’espace de travail avant cette agrégation de données agrégées dans tous les espaces de travail du cluster. 

2. **Espaces de travail** : les coûts de réservation de capacité pour votre cluster sont attribués proportionnellement aux espaces de travail du cluster (après prise en compte des allocations par nœud depuis [Azure Security Center](../../security-center/index.yml) pour chaque espace de travail). Si le volume total de données ingérées dans un espace de travail au cours d’une journée est inférieur à la réservation de capacité, chaque espace de travail est facturé pour ses données ingérées au taux de réservation de capacité par Go effectif, en les facturant une fraction de la réservation de capacité, et la partie inutilisée de la réservation de capacité est facturée sur la ressource de cluster. Si le volume total de données ingérées dans un espace de travail au cours d’une journée est supérieur à la réservation de capacité, chaque espace de travail est facturé pour une fraction de la réservation de capacité en fonction de la fraction des données ingérées, et chaque espace de travail pour une fraction des données ingérées au-delà de la réservation de capacité. Rien n’est facturé au niveau de la ressource de cluster si le volume total de données ingérées dans un espace de travail au cours d’une journée est supérieur à la réservation de capacité.

Dans options de facturation de cluster, la conservation des données est facturée par espace de travail. Notez que la facturation du cluster commence lorsque le cluster est créé, que les espaces de travail aient ou non été associés au cluster. Notez également que les espaces de travail associés à un cluster ne présentent plus de niveau tarifaire.

## <a name="estimating-the-costs-to-manage-your-environment"></a>Estimation des coûts de gestion de votre environnement 

Si vous n’utilisez pas encore les journaux Azure Monitor, vous pouvez utiliser la [calculatrice de prix Azure Monitor](https://azure.microsoft.com/pricing/calculator/?service=monitor) pour estimer le coût d’utilisation de Log Analytics. Commencez par entrer « Azure Monitor » dans la zone de recherche, puis cliquez sur la vignette Azure Monitor obtenue. Faites défiler la page jusqu’à Azure Monitor et sélectionnez Log Analytics dans la liste déroulante Type.  Ici, vous pouvez entrer le nombre de machines virtuelles et les Go de données que vous prévoyez de collecter à partir de chaque machine virtuelle. En général, 1 à 3 Go de données par mois sont ingérés à partir d’une machine virtuelle Azure standard. Si vous évaluez déjà les journaux Azure Monitor, vous pouvez utiliser vos statistiques de données à partir de votre propre environnement. Vous verrez ci-dessous comment déterminer le [nombre de machines virtuelles surveillées](#understanding-nodes-sending-data) et le [volume de données que votre espace de travail ingère](#understanding-ingested-data-volume). 

## <a name="understand-your-usage-and-estimate-costs"></a>Comprendre votre utilisation et estimer les coûts

Si vous utilisez les journaux Azure Monitor, il est facile d’estimer les coûts en fonction des modèles d’utilisation récente. Pour ce faire, utilisez le tableau de bord **Utilisation et estimation des coûts de Log Analytics** pour examiner et analyser l’utilisation de données. Celui-ci montre la quantité de données collectée par chaque solution et la quantité de données conservées et fournit une estimation des coûts en fonction de la quantité de données ingérées et de toute rétention supplémentaire au-delà du montant inclus.

![Utilisation et estimation des coûts](media/manage-cost-storage/usage-estimated-cost-dashboard-01.png)

Pour explorer vos données plus en détail, cliquez sur l’icône en haut à droite d’un des graphiques sur la page **Utilisation et estimation des coûts**. Vous pouvez maintenant utiliser cette requête pour obtenir de plus amples informations sur votre utilisation.  

![Vue Journaux d’activité](media/manage-cost-storage/logs.png)

La page **Utilisation et estimation des coûts** vous permet de consulter votre volume de données pour le mois. Ce volume inclut toutes les données facturables reçues et conservées dans votre espace de travail Log Analytics.  
 
Les frais liés à Log Analytics sont ajoutés à votre facture Azure. Les informations relatives à votre facture Azure sont affichées dans la section Facturation du portail Azure ou sur le [portail de facturation Azure](https://account.windowsazure.com/Subscriptions).  

## <a name="viewing-log-analytics-usage-on-your-azure-bill"></a>Consultation de l’utilisation de Log Analytics sur votre facture Azure 

Azure fournit de nombreuses fonctionnalités utiles dans le hub [Azure Cost Management + facturation](../../cost-management-billing/costs/quick-acm-cost-analysis.md?toc=%2fazure%2fbilling%2fTOC.json). Par exemple, la fonctionnalité « Analyse des coûts » vous permet de voir vos dépenses en ressources Azure. Tout d’abord, l’ajout d’un filtre par « type de ressource » (microsoft.operationalinsights/workspace pour Log Analytics et microsoft.operationalinsights/workspace pour les clusters Log Analytics) vous permettra de suivre vos dépenses Log Analytics. Ensuite, pour « Regrouper par », sélectionnez « Catégorie du compteur » ou « Compteur ».  Notez que d’autres services tels qu’Azure Security Center et Azure Sentinel facturent également leur utilisation en fonction des ressources de l’espace de travail Log Analytics. Pour voir la correspondance avec le nom du service, vous pouvez sélectionner la vue Tableau au lieu d’un graphique. 

Vous pouvez mieux comprendre votre utilisation en [téléchargeant votre utilisation à partir du portail Azure](../../cost-management-billing/manage/download-azure-invoice-daily-usage-date.md#download-usage-in-azure-portal). Dans la feuille de calcul téléchargée, vous pouvez voir l’utilisation par ressource Azure (p. ex., espace de travail Log Analytics) par jour. Dans cette feuille de calcul Excel, vous trouverez l’utilisation de vos espaces de travail Log Analytics en filtrant d’abord la colonne « Catégorie du compteur » pour afficher « Log Analytics », « Insights and Analytics » (utilisé par certains niveaux tarifaires hérités) et « Azure Monitor » (utilisé par les niveaux tarifaires de réservation de capacité), puis en ajoutant sur la colonne « ID d’instance » le filtre « contient espace de travail » ou « contient cluster » (ce dernier pour inclure l’utilisation du cluster Log Analytics). L’utilisation est indiquée dans la colonne « Quantité consommée » et l’unité pour chaque entrée est affichée dans la colonne « Unité de mesure ».  Plus de détails sont disponibles pour vous aider à [comprendre votre facture Microsoft Azure](../../cost-management-billing/understand/review-individual-bill.md). 

## <a name="changing-pricing-tier"></a>Changement de niveau tarifaire

Pour modifier le niveau tarifaire Log Analytics de votre espace de travail : 

1. Dans le Portail Azure, ouvrez **Utilisation et estimation des coûts** à partir de votre espace de travail où vous verrez une liste de chacun des niveaux tarifaires disponibles pour cet espace de travail.

2. Passez en revue les coûts estimés pour chacun des niveaux tarifaires. Cette estimation est basée sur les 31 derniers jours d’utilisation, de sorte qu’elle repose sur le fait que les 31 derniers jours sont représentatifs de votre utilisation habituelle. Dans l’exemple ci-dessous, vous pouvez voir comment, en fonction des modèles de données des 31 derniers jours, cet espace de travail coûterait moins cher dans le niveau de Paiement à l’utilisation (no 1) par rapport au niveau de réservation de capacité de 100 Go/jour (no 2).  

    ![Niveaux de tarification](media/manage-cost-storage/pricing-tier-estimated-costs.png)

3. Après avoir vérifié les coûts estimés en fonction des 31 derniers jours d’utilisation, et si vous décidez de modifier le niveau tarifaire, cliquez sur **Sélectionner**.  

Vous pouvez également [définir le niveau tarifaire via Azure Resource Manager](../samples/resource-manager-workspace.md) à l’aide du paramètre `sku`. (`pricingTier` dans le modèle Azure Resource Manager). 

## <a name="legacy-pricing-tiers"></a>Niveaux de tarification hérités

Les abonnements qui incluaient un espace de travail Log Analytics ou une ressource Application Insights avant le 2 avril 2018, ou qui sont liés à un Contrat Entreprise commencé avant le 1er février 2019, auront toujours accès aux niveaux tarifaires hérités : **Gratuit**, **Autonome (par Go)** et **Par nœud (OMS)** .  Les espaces de travail du niveau tarifaire Gratuit présentent une ingestion des données quotidienne limitée à 500 Mo (à l’exception des types de données de sécurité collectés par [Azure Security Center](../../security-center/index.yml)) et une conservation des données limitée à 7 jours. Le niveau tarifaire Gratuit est conçu à des fins d’évaluation uniquement. Les espaces de travail des niveaux tarifaires Autonome et Par nœud présentent une rétention configurable par l’utilisateur de 30 à 730 jours.

L’utilisation au niveau tarifaire Autonome est facturée en fonction du volume de données ingérées. Elle est rapporté dans le service **Log Analytics**, et le compteur est nommé « Data Analyzed » (Données analysées). 

Le niveau tarifaire par nœud est facturé par machine virtuelle (nœud) surveillée sur une granularité horaire. Pour chaque nœud analysé, 500 Mo de données non facturées par jour sont affectées à l’espace de travail. Cette allocation est calculée avec une granularité horaire et est agrégée chaque jour au niveau de l’espace de travail. Les données ingérées au-delà de l’allocation de données quotidienne agrégée sont facturées par Go comme dépassement de données. Notez que sur votre facture le service indiqué est **Insight and Analytics** pour l’utilisation de Log Analytics si l’espace de travail se trouve dans le niveau tarifaire Par nœud. L’utilisation est signalée sur trois compteurs :

1. Node (Nœud) : il s’agit de l’utilisation du nombre de nœuds surveillés (machines virtuelles) en unités de nœud * mois.
2. Data Overage per Node (Dépassement de données par nœud) : il s’agit du nombre de Go de données ingérées au-delà de l’allocation de données agrégées.
3. Data Included per Node (Données incluses par nœud) : il s’agit de la quantité de données ingérées qui a été couverte par l’allocation de données agrégées. Ce compteur est également utilisé lorsque l’espace de travail se trouve dans tous les niveaux de tarification pour afficher la quantité de données couvertes par le service Azure Security Center.

> [!TIP]
> Si votre espace de travail a accès au niveau tarifaire **Par nœud**, mais que vous vous demandez si ce serait moins cher dans un niveau Paiement à l’utilisation, vous pouvez [utiliser la requête ci-dessous](#evaluating-the-legacy-per-node-pricing-tier) pour obtenir facilement une recommandation. 

Les espaces de travail créés avant avril 2016 peuvent également accéder aux niveaux tarifaires **Standard** et **Premium** d’origine, assortis d’une durée de conservation des données fixe de 30 et 365 jours respectivement. Il n’est pas possible de créer des espaces de travail avec les niveaux tarifaires **Standard** ou **Premium**. De plus, si un espace de travail est retiré de ces niveaux tarifaires, il ne peut pas y être inclus de nouveau. Les compteurs d’ingestion de données pour ces niveaux hérités sont appelés « Data analyzed » (Données analysées).

Il existe également des comportements entre l’utilisation de niveaux Log Analytics hérités et la façon dont l’utilisation est facturée pour [Azure Security Center](../../security-center/index.yml). 

1. Si l’espace de travail se trouve dans le niveau Standard ou Premium hérité, Azure Security Center est facturé uniquement pour l’ingestion des données Log Analytics, et non par nœud.
2. Si l’espace de travail se trouve dans le niveau Par nœud hérité, Azure Security Center est facturé selon le [modèle de tarification actuel basé sur les nœuds d’Azure Security Center](https://azure.microsoft.com/pricing/details/security-center/). 
3. Dans d’autres niveaux tarifaires (notamment Réservations de capacité), si Azure Security Center a été activé avant le 19 juin 2017, Azure Security Center sera facturé uniquement pour l’ingestion des données Log Analytics. Sinon, Azure Security Center est facturé selon le modèle de tarification actuel basé sur les nœuds d’Azure Security Center.

Pour plus d’informations sur les limitations de niveau tarifaire, consultez [Abonnement Azure et limites, quotas et contraintes de service](../../azure-resource-manager/management/azure-subscription-service-limits.md#log-analytics-workspaces).

Aucun des niveaux tarifaires hérités n’a de tarification régionale.  

> [!NOTE]
> Pour utiliser les droits que vous obtenez à l’achat de la suite OMS E1, OMS E2 ou du module complémentaire OMS pour System Center, sélectionnez le niveau tarifaire *Par nœud* de Log Analytics.

## <a name="log-analytics-and-security-center"></a>Log Analytics et Centre de sécurité

La facturation du [Centre de sécurité Azure](../../security-center/index.yml) est étroitement liée à la facturation de Log Analytics. Le Centre de sécurité fournit une allocation de 500 Mo/nœud/jour à un ensemble de [types de données de sécurité](/azure/azure-monitor/reference/tables/tables-category#security) (WindowsEvent, SecurityAlert, SecurityBaseline, SecurityBaselineSummary, SecurityDetection, SecurityEvent, WindowsFirewall, MaliciousIPCommunication, LinuxAuditLog, SysmonEvent, ProtectionStatus) et aux types de données Update et UpdateSummary lorsque la solution Update Management n’est pas exécutée sur l’espace de travail ou que le ciblage de solution est activé. Si l’espace de travail est au niveau tarifaire hérité Par nœud, les allocations du Centre de sécurité et de Log Analytics sont combinées et appliquées conjointement à toutes les données ingérées facturables.  

## <a name="change-the-data-retention-period"></a>Changer la période de rétention des données

Les étapes suivantes décrivent la configuration de la durée de conservation des données de journal dans votre espace de travail. La conservation des données peut être configurée entre 30 et 730 jours (soit 2 ans) pour tous les espaces de travail, sauf si elle utilise le niveau tarifaire Gratuit hérité. [En savoir plus](https://azure.microsoft.com/pricing/details/monitor/) sur la tarification pour une conservation des données plus longue. 

### <a name="default-retention"></a>Durée de conservation par défaut

Pour définir la durée de conservation par défaut pour votre espace de travail, 
 
1. Dans le Portail Azure, à partir de votre espace de travail, sélectionnez **Utilisation et estimation des coûts** dans le volet gauche.
2. En haut de la page **Utilisation et coûts estimés**, cliquez sur **Conservation des données**.
3. Dans le volet, déplacez le curseur pour augmenter ou diminuer le nombre de jours, puis cliquez sur **OK**.  Si vous avez opté pour le niveau *Gratuit*, vous ne pouvez pas modifier la période de rétention de données et vous devez passer au niveau payant afin de contrôler ce paramètre.

    ![Changer le paramètre de conservation des données de l’espace de travail](media/manage-cost-storage/manage-cost-change-retention-01.png)

Lorsque la rétention est raccourcie, il existe une période de grâce de quelques jours avant la suppression des données antérieures au nouveau paramètre de rétention. 

La rétention peut également être [définie via Azure Resource Manager](../samples/resource-manager-workspace.md) à l’aide du paramètre `retentionInDays`. Lorsque vous définissez la période de rétention des données sur 30 jours, vous pouvez déclencher une purge immédiate des anciennes données à l’aide du paramètre `immediatePurgeDataOn30Days` (en éliminant la période de grâce de quelques jours). Cela peut être utile pour les scénarios liés à la conformité où une suppression immédiate des données est impérative. Cette fonctionnalité de purge immédiate est exposée uniquement via Azure Resource Manager. 

Les espaces de travail dont la période de rétention est de 30 jours peuvent en réalité conserver des données pendant 31 jours. S’il est impératif que les données soient conservées pendant seulement 30 jours, utilisez Azure Resource Manager pour définir la période de rétention sur 30 jours et avec le paramètre `immediatePurgeDataOn30Days`.  

Deux types de données (`Usage` et `AzureActivity`) sont conservés pendant un minimum de 90 jours par défaut, et aucuns frais ne sont facturés pour cette rétention de 90 jours. Si la rétention de l’espace de travail augmente au-delà de 90 jours, la rétention de ces types de données est également augmentée.  Ces types de données sont également exempts des frais d’ingestion de données. 

Les types de données provenant des ressources Application Insights basées sur l’espace de travail (`AppAvailabilityResults`, `AppBrowserTimings`, `AppDependencies`, `AppExceptions`, `AppEvents`, `AppMetrics`, `AppPageViews`, `AppPerformanceCounters`, `AppRequests`, `AppSystemEvents` et `AppTraces`) sont également conservés 90 jours par défaut, et aucuns frais ne sont facturés pour cette rétention de 90 jours. Leur rétention peut être ajustée à l’aide de la fonctionnalité de rétention par type de données. 

Notez que [l’API de purge](/rest/api/loganalytics/workspacepurge/purge) Log Analytics n’affecte pas la facturation de rétention et est destinée à être utilisée dans des cas très limités. Pour réduire votre facture de rétention, la période de rétention doit être réduite pour l’espace de travail ou pour des types de données spécifiques. 

### <a name="retention-by-data-type"></a>Durée de conservation par type de données

Il est également possible de spécifier des paramètres de rétention différents pour des types de données individuels allant de 30 à 730 jours (à l’exception des espaces de travail du niveau tarifaire gratuit hérité). Chaque type de données est une sous-ressource de l’espace de travail. Par exemple, la table SecurityEvent peut être traitée dans [Azure Resource Manager](../../azure-resource-manager/management/overview.md) comme suit :

```
/subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/MyResourceGroupName/providers/Microsoft.OperationalInsights/workspaces/MyWorkspaceName/Tables/SecurityEvent
```

Notez que le type de données (table) est sensible à la casse.  Pour obtenir les paramètres actuels de rétention par type de données d’un type de données particulier (dans cet exemple, SecurityEvent), utilisez :

```JSON
    GET /subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/MyResourceGroupName/providers/Microsoft.OperationalInsights/workspaces/MyWorkspaceName/Tables/SecurityEvent?api-version=2017-04-26-preview
```

> [!NOTE]
> La rétention n’est retournée pour un type de données que elle a été explicitement définie.  Les types de données dont la rétention n’a pas été explicitement définie (et qui héritent donc de la rétention de l’espace de travail) ne retournent rien suite à cet appel. 

Afin d'obtenir les paramètres actuels de rétention par type de données de tous les types de données de votre espace de travail pour lesquels la rétention par type de données a été définie, il suffit d'omettre le type de données spécifique, par exemple :

```JSON
    GET /subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/MyResourceGroupName/providers/Microsoft.OperationalInsights/workspaces/MyWorkspaceName/Tables?api-version=2017-04-26-preview
```

Pour définir la durée de conservation d’un type de données particulier (dans cet exemple, SecurityEvent) à 730 jours, utilisez :

```JSON
    PUT /subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/MyResourceGroupName/providers/Microsoft.OperationalInsights/workspaces/MyWorkspaceName/Tables/SecurityEvent?api-version=2017-04-26-preview
    {
        "properties": 
        {
            "retentionInDays": 730
        }
    }
```

Les valeurs valides pour `retentionInDays` sont comprises entre 30 et 730.

Les types de données `Usage` et `AzureActivity` ne peuvent pas être définis avec une durée de conservation personnalisée. Ils prendront la durée maximale de conservation de l’espace de travail par défaut ou 90 jours. 

Un excellent outil pour se connecter directement à Azure Resource Manager pour définir la durée de conservation par type de données est l’outil OSS [ARMclient](https://github.com/projectkudu/ARMClient).  Découvrez plus d’informations sur ARMclient dans les articles de [David Ebbo](http://blog.davidebbo.com/2015/01/azure-resource-manager-client.html) et de [Daniel Bowbyes](https://blog.bowbyes.co.nz/2016/11/02/using-armclient-to-directly-access-azure-arm-rest-apis-and-list-arm-policy-details/).  Voici un exemple d’utilisation d’ARMClient, définissant une durée de conservation de 730 jours pour les données SecurityEvent :

```
armclient PUT /subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/MyResourceGroupName/providers/Microsoft.OperationalInsights/workspaces/MyWorkspaceName/Tables/SecurityEvent?api-version=2017-04-26-preview "{properties: {retentionInDays: 730}}"
```

> [!TIP]
> La définition d’une durée de conservation sur des types de données individuels peut être utilisée pour réduire vos coûts de conservation de données.  Pour les données collectées à partir d’octobre 2019 (date de publication de cette fonctionnalité), la réduction de la durée de conservation de certains types de données peut réduire vos coûts de conservation à terme.  Pour les données collectées précédemment, la définition d’une durée de conservation inférieure pour un type individuel n’affecte pas vos coûts de conservation.  

## <a name="manage-your-maximum-daily-data-volume"></a>Gérer votre volume de données maximal quotidien

Vous pouvez configurer une limite quotidienne et restreindre l’ingestion quotidienne de votre espace de travail, mais soyez vigilant, car votre objectif n’est pas d’atteindre la limite quotidienne.  Si vous l’atteignez, vous perdrez des données pour le reste de la journée, ce qui peut impacter les autres services et solutions Azure dont les fonctionnalités dépendent de la disponibilité de données à jour dans l’espace de travail.  Ces fonctionnalités peuvent correspondre, par exemple, à votre capacité à observer et à recevoir des alertes lorsque les conditions d’intégrité des ressources de service informatique sont impactées.  La limite quotidienne est destinée à être utilisée comme un moyen de gérer une **augmentation inattendue** du volume de données de vos ressources managées ou lorsque vous souhaitez limiter les frais non planifiés de votre espace de travail. Il n’est pas adéquat de définir une limite quotidienne pour à respecter chaque jour dans un espace de travail.

Chaque espace de travail a sa limite quotidienne appliquée à une heure différente de la journée. L’heure de réinitialisation est indiquée sur la page **Limite quotidienne** (voir ci-dessous). L’heure de réinitialisation ne peut pas être configurée. 

Peu après que cette limite quotidienne est atteinte, la collecte des types de données facturables s’arrête pour le reste de la journée. La latence inhérente à l’application de la limite quotidienne signifie que la limite n’est pas appliquée au niveau de limite quotidienne spécifiée. Une bannière d’avertissement s’affiche en haut de la page de l’espace de travail Log Analytics sélectionné, et un événement d’opération est envoyé vers la table *Opération* dans la catégorie **LogManagement**. La collecte de données reprend après l’heure de réinitialisation définie dans *La limite quotidienne est fixée à*. Nous vous recommandons de définir une règle d’alerte en fonction de cet événement d’opération, configuré pour avertir lorsque la limite de données quotidienne a été atteinte (vois [ci-dessous](#alert-when-daily-cap-reached)). 

> [!NOTE]
> La limite quotidienne ne peut pas arrêter la collecte de données, car précisément le niveau de cap spécifié et certaines données excédentaires sont attendus, en particulier si l’espace de travail reçoit des volumes de données élevés. Pour obtenir une requête utile pour l’étude du comportement de limite quotidienne, voir [ci-dessous](#view-the-effect-of-the-daily-cap). 

> [!WARNING]
> La limite quotidienne n’arrête pas la collecte de données à partir d’Azure Sentinel ou Azure Security Center, à l’exception des espaces de travail dans lesquels Azure Security Center a été installé avant le 19 juin 2017. 

### <a name="identify-what-daily-data-limit-to-define"></a>Identifier la limite de données quotidienne à définir

Consultez [Utilisation et estimation des coûts Log Analytics](usage-estimated-costs.md) pour comprendre les tendances d’ingestion des données et la limite quotidienne de volume à définir. Effectuez cette opération avec précaution, car vous ne pourrez plus superviser vos ressources une fois la limite atteinte. 

### <a name="set-the-daily-cap"></a>Définir la limite quotidienne

Les étapes suivantes décrivent la configuration d’une limite pour gérer le volume des données ingérées quotidiennement par l’espace de travail Log Analytics.  

1. À partir de votre espace de travail, sélectionnez **Utilisation et estimation des coûts** dans le volet gauche.
2. Sur la page **Utilisation et estimation des coûts** de l’espace de travail sélectionné, cliquez sur **Limite des données** en haut de la page. 
3. Si la limite quotidienne est **désactivée** par défaut, cliquez sur **Activer** pour l’activer, puis définissez la limite de volume de données en Go/jour.

    ![Configurer la limite de données dans Log Analytics](media/manage-cost-storage/set-daily-volume-cap-01.png)
    
La limite quotidienne peut être configurée via ARM en définissant le paramètre `dailyQuotaGb` sous `WorkspaceCapping` comme décrit dans [Espaces de travail - Créer ou mettre à jour](/rest/api/loganalytics/workspaces/createorupdate#workspacecapping). 

### <a name="view-the-effect-of-the-daily-cap"></a>Afficher l’effet de la limite quotidienne

Pour afficher l’effet de la limite quotidienne, il est important de tenir compte des types de données de sécurité qui ne sont pas inclus dans la limite quotidienne et de l’heure de réinitialisation de votre espace de travail. L’heure de réinitialisation de la limite quotidienne est visible dans la page **Limite quotidienne**.  Vous pouvez utiliser la requête suivante pour effectuer le suivi des volumes de données soumis à la limite quotidienne entre les réinitialisations de limite quotidienne. Dans cet exemple, l’heure de réinitialisation de l’espace de travail est 14:00.  Vous devez la mettre à jour pour votre espace de travail.

```kusto
let DailyCapResetHour=14;
Usage
| where Type !in ("SecurityAlert", "SecurityBaseline", "SecurityBaselineSummary", "SecurityDetection", "SecurityEvent", "WindowsFirewall", "MaliciousIPCommunication", "LinuxAuditLog", "SysmonEvent", "ProtectionStatus", "WindowsEvent")
| extend TimeGenerated=datetime_add("hour",-1*DailyCapResetHour,TimeGenerated)
| where TimeGenerated > startofday(ago(31d))
| where IsBillable
| summarize IngestedGbBetweenDailyCapResets=sum(_BilledSize)/1000. by day=bin(TimeGenerated, 1d) | render areachart  
```

### <a name="alert-when-daily-cap-reached"></a>Alerte lorsque la limite quotidienne est atteinte

Si nous présentons un indice visuel dans le portail Azure lorsque le seuil limite des données est atteint, ce comportement n’est pas nécessairement en harmonie avec la façon dont vous gérez les problèmes opérationnels exigeant une attention immédiate.  Pour recevoir une notification d’alerte, vous pouvez créer une règle d’alerte dans Azure Monitor.  Pour en savoir plus, reportez-vous à l’article sur [la création, l’affichage et la gestion des alertes](alerts-metric.md).

Pour vous aider à démarrer, voici les paramètres que nous recommandons pour l’alerte interrogeant la table `Operation` à l’aide de la fonction `_LogOperation`. 

- Cible : sélectionnez votre ressource Log Analytics.
- Critères : 
   - Nom du signal : Recherche personnalisée dans les journaux
   - Requête de recherche : `_LogOperation | where Category == "Ingestion" | where Operation == "Ingestion rate" | where Level == "Warning"`
   - Basé sur : Nombre de résultats
   - Condition : Supérieur à
   - Seuil : 0
   - Période : 5 (minutes)
   - Fréquence : 5 (minutes)
- Nom de la règle d'alerte : limite de données quotidienne atteinte
- Gravité : avertissement (Sev 1)

Une fois que l’alerte est définie et que la limite est atteinte, l’alerte est déclenchée et effectue la réponse définie dans le groupe d’actions. Elle peut informer votre équipe via des e-mails et des SMS ou automatiser des actions à l’aide de Webhooks ou de runbooks Automation, ou [en s’intégrant à une solution ITSM externe](itsmc-overview.md#create-itsm-work-items-from-azure-alerts). 

## <a name="troubleshooting-why-usage-is-higher-than-expected"></a>Résolution des problèmes à l’origine d’une utilisation plus importante que prévu

Une utilisation plus importante est due à l’un des éléments suivants, voire les deux :
- Plus de nœuds que prévu envoient des données à l’espace de travail Log Analytics
- Plus de données que prévu sont envoyées à l'espace de travail Log Analytics (peut-être parce que l'on commence à utiliser une nouvelle solution ou que l'on modifie la configuration d'une solution existante)

## <a name="understanding-nodes-sending-data"></a>Présentation des nœuds qui envoient des données

Pour plus d'informations sur le nombre de nœuds qui ont signalé quotidiennement des pulsations de l'agent au cours du mois précédent, utilisez

```kusto
Heartbeat 
| where TimeGenerated > startofday(ago(31d))
| summarize nodes = dcount(Computer) by bin(TimeGenerated, 1d)    
| render timechart
```
Pour obtenir le nombre de nœuds envoyant des données au cours des dernières 24 heures, utilisez la requête : 

```kusto
find where TimeGenerated > ago(24h) project Computer
| extend computerName = tolower(tostring(split(Computer, '.')[0]))
| where computerName != ""
| summarize nodes = dcount(computerName)
```

Pour obtenir la liste des nœuds qui envoient des données (et le volume de données envoyées par chacun d'eux), la requête suivante peut être utilisée :

```kusto
find where TimeGenerated > ago(24h) project _BilledSize, Computer
| extend computerName = tolower(tostring(split(Computer, '.')[0]))
| where computerName != ""
| summarize TotalVolumeBytes=sum(_BilledSize) by computerName
```

### <a name="nodes-billed-by-the-legacy-per-node-pricing-tier"></a>Nœuds facturés par le niveau tarifaire hérité Par nœud

Le [niveau tarifaire hérité Par nœud](#legacy-pricing-tiers) facture les nœuds avec une granularité horaire et ne compte pas non plus les nœuds qui envoient uniquement un ensemble de types de données de sécurité. Son nombre quotidien de nœuds devrait être semblable à la requête suivante :

```kusto
find where TimeGenerated >= startofday(ago(7d)) and TimeGenerated < startofday(now()) project Computer, _IsBillable, Type, TimeGenerated
| where Type !in ("SecurityAlert", "SecurityBaseline", "SecurityBaselineSummary", "SecurityDetection", "SecurityEvent", "WindowsFirewall", "MaliciousIPCommunication", "LinuxAuditLog", "SysmonEvent", "ProtectionStatus", "WindowsEvent")
| extend computerName = tolower(tostring(split(Computer, '.')[0]))
| where computerName != ""
| where _IsBillable == true
| summarize billableNodesPerHour=dcount(computerName) by bin(TimeGenerated, 1h)
| summarize billableNodesPerDay = sum(billableNodesPerHour)/24., billableNodeMonthsPerDay = sum(billableNodesPerHour)/24./31.  by day=bin(TimeGenerated, 1d)
| sort by day asc
```

Le nombre d’unités sur votre facture est exprimé en unités nœuds x mois, ce qui est représenté par `billableNodeMonthsPerDay` dans la requête. Si la solution Update Management est installée sur l’espace de travail, ajoutez les types de données Update et UpdateSummary à la liste dans la clause WHERE de la requête ci-dessus. Enfin, il existe une complexité supplémentaire dans l’algorithme de facturation réel lorsque le ciblage de solution utilisé n’est pas représenté dans la requête ci-dessus. 


> [!TIP]
> Utilisez ces requêtes `find` avec parcimonie, car les analyses exécutées sur différents types de données sont [gourmandes en ressources](../log-query/query-optimization.md#query-performance-pane). Si vous n’avez pas besoin des résultats **par ordinateur**, interrogez le type de données Usage (voir ci-dessous).

## <a name="understanding-ingested-data-volume"></a>Présentation du volume de données ingéré

Sur la page **Utilisation et estimation des coûts**, le graphique *Ingestion de données par solution* montre le volume total des données envoyées et la quantité envoyée par chaque solution. Vous pouvez ainsi dégager des tendances, par exemple si l’utilisation des données globales (ou l’utilisation par une solution particulière) augmente, reste stable ou diminue. 

### <a name="data-volume-for-specific-events"></a>Volume de données pour des événements spécifiques

Pour examiner la taille des données ingérées pour un ensemble particulier d’événements, vous pouvez interroger la table spécifique (`Event` dans cet exemple), puis limiter la requête aux événements intéressants (dans cet exemple, l’ID d’événement 5145 ou 5156) :

```kusto
Event
| where TimeGenerated > startofday(ago(31d)) and TimeGenerated < startofday(now()) 
| where EventID == 5145 or EventID == 5156
| where _IsBillable == true
| summarize count(), Bytes=sum(_BilledSize) by EventID, bin(TimeGenerated, 1d)
``` 

Notez que la clause `where _IsBillable = true` exclut les types de données de certaines solutions pour lesquels il n'existe aucun frais d'ingestion. [En savoir plus](./log-standard-columns.md#_isbillable) sur `_IsBillable`.

### <a name="data-volume-by-solution"></a>Volume de données par solution

La requête utilisée pour afficher le volume de données facturable par solution au cours du dernier mois (à l’exception du dernier jour partiel) est la suivante :

```kusto
Usage 
| where TimeGenerated > ago(32d)
| where StartTime >= startofday(ago(31d)) and EndTime < startofday(now())
| where IsBillable == true
| summarize BillableDataGB = sum(Quantity) / 1000. by bin(StartTime, 1d), Solution | render barchart
```

La clause avec `TimeGenerated` permet uniquement de s’assurer que l’expérience de requête dans le portail Azure va au-delà des 24 heures par défaut. Lorsque vous utilisez le type de données Utilisation, `StartTime` et `EndTime` représentent les intervalles de temps pour lesquels les résultats sont présentés. 

### <a name="data-volume-by-type"></a>Volume de données par type

Vous pouvez approfondir pour afficher les tendances par type de données :

```kusto
Usage 
| where TimeGenerated > ago(32d)
| where StartTime >= startofday(ago(31d)) and EndTime < startofday(now())
| where IsBillable == true
| summarize BillableDataGB = sum(Quantity) / 1000. by bin(StartTime, 1d), DataType | render barchart
```

Ou pour afficher un tableau par solution et par type pour le mois précédent,

```kusto
Usage 
| where TimeGenerated > ago(32d)
| where StartTime >= startofday(ago(31d)) and EndTime < startofday(now())
| where IsBillable == true
| summarize BillableDataGB = sum(Quantity) by Solution, DataType
| sort by Solution asc, DataType asc
```

### <a name="data-volume-by-computer"></a>Volume de données par ordinateur

Le type de données `Usage` n’inclut pas d’informations au niveau de l’ordinateur. Pour afficher la **taille** des données ingérées par ordinateur, utilisez la `_BilledSize` [propriété](./log-standard-columns.md#_billedsize), qui fournit la taille en octets :

```kusto
find where TimeGenerated > ago(24h) project _BilledSize, _IsBillable, Computer
| where _IsBillable == true 
| extend computerName = tolower(tostring(split(Computer, '.')[0]))
| summarize BillableDataBytes = sum(_BilledSize) by  computerName 
| sort by BillableDataBytes nulls last
```

La `_IsBillable`[propriété](./log-standard-columns.md#_isbillable) spécifie si les données ingérées occasionnent des frais. 

Pour afficher le **nombre** d'événements facturables ingérés par ordinateur, utilisez 

```kusto
find where TimeGenerated > ago(24h) project _IsBillable, Computer
| where _IsBillable == true 
| extend computerName = tolower(tostring(split(Computer, '.')[0]))
| summarize eventCount = count() by computerName  
| sort by eventCount nulls last
```

> [!TIP]
> Utilisez ces requêtes `find` avec parcimonie, car les analyses exécutées sur différents types de données sont [gourmandes en ressources](../log-query/query-optimization.md#query-performance-pane). Si vous n’avez pas besoin des résultats **par ordinateur**, interrogez le type de données Usage.

### <a name="data-volume-by-azure-resource-resource-group-or-subscription"></a>Volume de données par ressource Azure, groupe de ressources ou abonnement

Pour les données provenant de nœuds hébergés dans Azure, vous pouvez obtenir la **taille** des données ingérées __par ordinateur__. Utilisez la [propriété](./log-standard-columns.md#_resourceid) _ResourceId, qui fournit le chemin d'accès complet à la ressource :

```kusto
find where TimeGenerated > ago(24h) project _ResourceId, _BilledSize, _IsBillable
| where _IsBillable == true 
| summarize BillableDataBytes = sum(_BilledSize) by _ResourceId | sort by BillableDataBytes nulls last
```

Pour les données provenant de nœuds hébergés dans Azure, vous pouvez obtenir la **taille** des données ingérées __par abonnement Azure__. Utilisez la propriété `_SubscriptionId` comme suit :

```kusto
find where TimeGenerated > ago(24h) project _ResourceId, _BilledSize, _IsBillable
| where _IsBillable == true 
| summarize BillableDataBytes = sum(_BilledSize) by _ResourceId
| summarize BillableDataBytes = sum(BillableDataBytes) by _SubscriptionId | sort by BillableDataBytes nulls last
```

Pour récupérer le volume de données par groupe de ressources, vous pouvez analyser `_ResourceId` :

```kusto
find where TimeGenerated > ago(24h) project _ResourceId, _BilledSize, _IsBillable
| where _IsBillable == true 
| summarize BillableDataBytes = sum(_BilledSize) by _ResourceId
| extend resourceGroup = tostring(split(_ResourceId, "/")[4] )
| summarize BillableDataBytes = sum(BillableDataBytes) by resourceGroup | sort by BillableDataBytes nulls last
```

Si nécessaire, vous pouvez également analyser le `_ResourceId` plus en détail à l’aide de

```Kusto
| parse tolower(_ResourceId) with "/subscriptions/" subscriptionId "/resourcegroups/" 
    resourceGroup "/providers/" provider "/" resourceType "/" resourceName   
```

> [!TIP]
> Utilisez ces requêtes `find` avec parcimonie, car les analyses exécutées sur différents types de données sont [gourmandes en ressources](../log-query/query-optimization.md#query-performance-pane). Si vous n’avez pas besoin des résultats par abonnement, groupe de ressources ou nom de ressource, interrogez le type de données Usage.

> [!WARNING]
> Certains champs du type de données Utilisation, bien que faisant partie du schéma, sont maintenant déconseillés et leurs valeurs ne seront plus fournies. Il s’agit de **Computer** et des champs liées à l’ingestion (**TotalBatches**, **BatchesWithinSla**, **BatchesOutsideSla**,  **BatchesCapped** et **AverageProcessingTimeMs**.


### <a name="querying-for-common-data-types"></a>Interrogation de types de données courants

Pour explorer plus en détail la source de données d’un type de données particulier, voici quelques exemples de requêtes :

+ Ressources **Application Insights basées sur l’espace de travail**
  - En savoir plus dans [Gérer l’utilisation et les coûts pour Application Insights](../app/pricing.md#data-volume-for-workspace-based-application-insights-resources)
+ une solution de **sécurité**
  - `SecurityEvent | summarize AggregatedValue = count() by EventID`
+ une solution de **gestion des journaux**
  - `Usage | where Solution == "LogManagement" and iff(isnotnull(toint(IsBillable)), IsBillable == true, IsBillable == "true") == true | summarize AggregatedValue = count() by DataType`
+ le type de données **Perf**
  - `Perf | summarize AggregatedValue = count() by CounterPath`
  - `Perf | summarize AggregatedValue = count() by CounterName`
+ le type de données **Event**
  - `Event | summarize AggregatedValue = count() by EventID`
  - `Event | summarize AggregatedValue = count() by EventLog, EventLevelName`
+ le type de données **Syslog**
  - `Syslog | summarize AggregatedValue = count() by Facility, SeverityLevel`
  - `Syslog | summarize AggregatedValue = count() by ProcessName`
+ le type de données **AzureDiagnostics**
  - `AzureDiagnostics | summarize AggregatedValue = count() by ResourceProvider, ResourceId`

## <a name="tips-for-reducing-data-volume"></a>Conseils pour réduire le volume de données

Voici quelques suggestions pour réduire le volume de journaux d’activité collectés :

| Source du volume de données important | Comment réduire le volume de données |
| -------------------------- | ------------------------- |
| Container Insights         | [Configurez Container Insights](../insights/container-insights-cost.md#controlling-ingestion-to-reduce-cost) de manière à collecter uniquement les données requises. |
| Événements de sécurité            | Sélectionnez [les événements de sécurité courants ou minimaux](../../security-center/security-center-enable-data-collection.md#data-collection-tier). <br> Modifier la stratégie d’audit de sécurité pour collecter les événements nécessaires uniquement. Plus particulièrement, examinez la nécessité de collecter des événements pour : <br> - [plateforme de filtrage de l’audit](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd772749(v=ws.10)) <br> - [registre de l’audit](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd941614(v%3dws.10))<br> - [système de fichiers de l’audit](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd772661(v%3dws.10))<br> - [objet de noyau d’audit](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd941615(v%3dws.10))<br> - [manipulation du descripteur de l’audit](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd772626(v%3dws.10))<br> - stockage amovible de l’audit |
| Compteurs de performance       | Modifiez la [configuration du compteur de performances](data-sources-performance-counters.md) de façon à : <br> - Réduire la fréquence de collecte <br> - Réduire le nombre de compteurs de performance |
| Journaux d’événements                 | Modifiez la [configuration du journal d’événements](data-sources-windows-events.md) de façon à : <br> - Réduire le nombre de journaux des événements collectés <br> - Collecter uniquement les niveaux d’événement requis Par exemple, ne collectez pas les événements de niveau *Informations*. |
| syslog                     | Modifiez la [configuration du syslog](data-sources-syslog.md) de façon à : <br> - Réduire le nombre d’installations collectées <br> - Collecter uniquement les niveaux d’événement requis Par exemple, ne collectez pas les événements de niveau *Informations* et *Débogage*. |
| AzureDiagnostics           | Modifiez la [collection de journaux de ressources](./diagnostic-settings.md#create-in-azure-portal) pour : <br> - Réduire le nombre de journaux d’activité d’envoi de ressources à Log Analytics <br> - Collecter uniquement les journaux d’activité nécessaires |
| Données de solution d’ordinateurs n’ayant pas besoin de la solution | Utilisez le [ciblage de solution](../insights/solution-targeting.md) pour collecter des données des groupes d’ordinateurs requis uniquement. |

### <a name="getting-nodes-as-billed-in-the-per-node-pricing-tier"></a>Obtention des nœuds facturés au niveau tarifaire Par nœud

Pour obtenir une liste d’ordinateurs qui seront facturés en tant que nœuds si l’espace de travail se trouve au niveau tarifaire Par nœud hérité, recherchez les nœuds qui envoient des **types de données facturés** (certains types de données sont gratuits). Pour ce faire, utilisez la `_IsBillable`[propriété](./log-standard-columns.md#_isbillable) et utilisez le champ tout à gauche du nom de domaine complet. Vous obtenez alors le nombre d'ordinateurs dont les données sont facturées à l'heure (ce qui correspond à la granularité à laquelle les nœuds sont comptés et facturés) :

```kusto
find where TimeGenerated > ago(24h) project Computer, TimeGenerated
| extend computerName = tolower(tostring(split(Computer, '.')[0]))
| where computerName != ""
| summarize billableNodes=dcount(computerName) by bin(TimeGenerated, 1h) | sort by TimeGenerated asc
```

### <a name="getting-security-and-automation-node-counts"></a>Obtention du nombre de nœuds Security et Automation

Si vous utilisez un niveau tarifaire « Par nœud (OMS) », vous êtes facturé en fonction du nombre de nœuds et de solutions que vous utilisez, et le nombre de nœuds Insights et Analytics pour lesquels vous êtes facturé s’affichera dans la table à la page **Utilisation et estimation des coûts**.  

Pour afficher le nombre de nœuds Security distincts, vous pouvez utiliser la requête :

```kusto
union
(
    Heartbeat
    | where (Solutions has 'security' or Solutions has 'antimalware' or Solutions has 'securitycenter')
    | project Computer
),
(
    ProtectionStatus
    | where Computer !in~
    (
        (
            Heartbeat
            | project Computer
        )
    )
    | project Computer
)
| distinct Computer
| project lowComputer = tolower(Computer)
| distinct lowComputer
| count
```

Pour afficher le nombre de nœuds Automation distincts, utilisez la requête :

```kusto
 ConfigurationData 
 | where (ConfigDataType == "WindowsServices" or ConfigDataType == "Software" or ConfigDataType =="Daemons") 
 | extend lowComputer = tolower(Computer) | summarize by lowComputer 
 | join (
     Heartbeat 
       | where SCAgentChannel == "Direct"
       | extend lowComputer = tolower(Computer) | summarize by lowComputer, ComputerEnvironment
 ) on lowComputer
 | summarize count() by ComputerEnvironment | sort by ComputerEnvironment asc
```

## <a name="evaluating-the-legacy-per-node-pricing-tier"></a>Évaluation du niveau tarifaire hérité Par nœud

Décider si les espaces de travail ayant accès au niveau tarifaire hérité **Par nœud** sont mieux dans ce niveau, ou dans un niveau actuel **Paiement à l’utilisation** ou **Réservation de capacité**, est souvent difficile pour les clients.  Cela implique de bien comprendre le compromis entre le coût fixe par nœud supervisé dans le niveau de tarification Par nœud et son allocation de données incluse de 500 Mo/nœud/jour, et le coût supporté en payant simplement les données ingérées dans le niveau Paiement à l’utilisation (Par Go). 

Pour faciliter cette évaluation, vous pouvez utiliser la requête suivante pour obtenir une recommandation sur le niveau tarifaire optimal en fonction des modèles d’utilisation d’un espace de travail.  Cette requête examine les nœuds supervisés et les données ingérées dans un espace de travail au cours des 7 derniers jours, et elle évalue pour chaque jour le niveau de tarification qui aurait été optimal. Pour utiliser la requête, vous devez :

1. spécifier si l’espace de travail utilise Azure Security Center en définissant `workspaceHasSecurityCenter` sur `true` ou `false` ; 
2. mettre à jour les tarifs si vous avez des remises spécifiques ; et
3. spécifier le nombre de jours à examiner et à analyser en définissant `daysToEvaluate`. Cela est utile si la requête prend trop de temps pour essayer de consulter sept jours de données. 

Voici la requête de recommandation de niveau tarifaire :

```kusto
// Set these parameters before running query
let workspaceHasSecurityCenter = true;  // Specify if the workspace has Azure Security Center
let PerNodePrice = 15.; // Enter your montly price per monitored nodes
let PerNodeOveragePrice = 2.30; // Enter your price per GB for data overage in the Per Node pricing tier
let PerGBPrice = 2.30; // Enter your price per GB in the Pay-as-you-go pricing tier
let daysToEvaluate = 7; // Enter number of previous days look at (reduce if the query is taking too long)
// ---------------------------------------
let SecurityDataTypes=dynamic(["SecurityAlert", "SecurityBaseline", "SecurityBaselineSummary", "SecurityDetection", "SecurityEvent", "WindowsFirewall", "MaliciousIPCommunication", "LinuxAuditLog", "SysmonEvent", "ProtectionStatus", "WindowsEvent", "Update", "UpdateSummary"]);
let StartDate = startofday(datetime_add("Day",-1*daysToEvaluate,now()));
let EndDate = startofday(now());
union * 
| where TimeGenerated >= StartDate and TimeGenerated < EndDate
| extend computerName = tolower(tostring(split(Computer, '.')[0]))
| where computerName != ""
| summarize nodesPerHour = dcount(computerName) by bin(TimeGenerated, 1h)  
| summarize nodesPerDay = sum(nodesPerHour)/24.  by day=bin(TimeGenerated, 1d)  
| join kind=leftouter (
    Heartbeat 
    | where TimeGenerated >= StartDate and TimeGenerated < EndDate
    | where Computer != ""
    | summarize ASCnodesPerHour = dcount(Computer) by bin(TimeGenerated, 1h) 
    | extend ASCnodesPerHour = iff(workspaceHasSecurityCenter, ASCnodesPerHour, 0)
    | summarize ASCnodesPerDay = sum(ASCnodesPerHour)/24.  by day=bin(TimeGenerated, 1d)   
) on day
| join (
    Usage 
    | where TimeGenerated >= StartDate and TimeGenerated < EndDate
    | where IsBillable == true
    | extend NonSecurityData = iff(DataType !in (SecurityDataTypes), Quantity, 0.)
    | extend SecurityData = iff(DataType in (SecurityDataTypes), Quantity, 0.)
    | summarize DataGB=sum(Quantity)/1000., NonSecurityDataGB=sum(NonSecurityData)/1000., SecurityDataGB=sum(SecurityData)/1000. by day=bin(StartTime, 1d)  
) on day
| extend AvgGbPerNode =  NonSecurityDataGB / nodesPerDay
| extend PerGBDailyCost = iff(workspaceHasSecurityCenter,
             (NonSecurityDataGB + max_of(SecurityDataGB - 0.5*ASCnodesPerDay, 0.)) * PerGBPrice,
             DataGB * PerGBPrice)
| extend OverageGB = iff(workspaceHasSecurityCenter, 
             max_of(DataGB - 0.5*nodesPerDay - 0.5*ASCnodesPerDay, 0.), 
             max_of(DataGB - 0.5*nodesPerDay, 0.))
| extend PerNodeDailyCost = nodesPerDay * PerNodePrice / 31. + OverageGB * PerNodeOveragePrice
| extend Recommendation = iff(PerNodeDailyCost < PerGBDailyCost, "Per Node tier", 
             iff(NonSecurityDataGB > 85., "Capacity Reservation tier", "Pay-as-you-go (Per GB) tier"))
| project day, nodesPerDay, ASCnodesPerDay, NonSecurityDataGB, SecurityDataGB, OverageGB, AvgGbPerNode, PerGBDailyCost, PerNodeDailyCost, Recommendation | sort by day asc
//| project day, Recommendation // Comment this line to see details
| sort by day asc
```

Cette requête n’est pas une réplication exacte de la façon dont l’utilisation est calculée, mais elle fonctionne pour fournir des recommandations de niveau tarifaire dans la plupart des cas.  

> [!NOTE]
> Pour utiliser les droits que vous obtenez à l’achat de la suite OMS E1, OMS E2 ou du module complémentaire OMS pour System Center, sélectionnez le niveau tarifaire *Par nœud* de Log Analytics.

## <a name="create-an-alert-when-data-collection-is-high"></a>Créer une alerte lorsque la collection de données est volumineuse

Cette section décrit la procédure de création d’une alerte à l’aide des [Alertes de journal](alerts-unified-log.md) Azure Monitor lorsque le volume de données au cours des dernières 24 heures a dépassé une quantité spécifiée. 

Pour déclencher une alerte si le volume de données ingérées facturables au cours des dernières 24 heures est supérieur à 50 Go, procédez comme suit : 

- **Définir la condition d’alerte** spécifiez votre espace de travail Log Analytics comme cible de la ressource.
- **Critères d’alerte** spécifiez les éléments suivants :
   - **Nom du signal** sélectionnez **Recherche de journal personnalisée**
   - **Requête de recherche** sur `Usage | where IsBillable | summarize DataGB = sum(Quantity / 1000.) | where DataGB > 50`. Si vous souhaitez un autre 
   - La **logique d’alerte** est **basée sur** le *nombre de résultats* et **Condition** est *supérieur à* un **seuil**  de *0*
   - **Période** de *1440* minutes et **Fréquence des alertes** définie toutes les *1440* minutes pour s’exécuter une fois par jour.
- **Définir les détails de l’alerte** spécifiez les éléments suivants :
   - **Nom** défini sur *Billable data volume greater than 50 GB in 24 hours* (Volume de données facturables supérieur à 50 Go en 24 heures)
   - **Gravité** : *Avertissement*

Spécifiez un [groupe d’actions](action-groups.md) existant ou créez-en un nouveau afin que l’alerte de journal corresponde aux critères, vous êtes informé.

Lorsque vous recevez une alerte, utilisez la procédure des sections précédentes pour résoudre les problèmes à l’origine d’une utilisation plus importante que prévu.

## <a name="data-transfer-charges-using-log-analytics"></a>Frais de transfert de données à l’aide de Log Analytics

L’envoi de données à Log Analytics peut entraîner des frais de bande passante, mais ils sont limités aux machines virtuelles où un agent Log Analytics est installé et ne s’appliquent pas lors de l’utilisation des paramètres de diagnostic ou avec d’autres connecteurs intégrés à Azure Sentinel. Comme décrit dans la [page sur les tarifs de bande passante Azure](https://azure.microsoft.com/pricing/details/bandwidth/), le transfert de données entre des services Azure situés dans deux régions différentes est facturé en tant que transfert de données sortant au tarif normal. Le transfert de données entrantes est gratuit. Toutefois, ce coût est très modique (faible pourcentage) par rapport aux coûts liés à l’ingestion de données de Log Analytics. Ainsi, la maîtrise des coûts pour Log Analytics doit s’appuyer sur le [volume de données ingérées](#understanding-ingested-data-volume). 


## <a name="troubleshooting-why-log-analytics-is-no-longer-collecting-data"></a>Dépannage si Log Analytics ne collecte plus de données

Si vous utilisez le niveau tarifaire hérité Gratuit et que vous avez envoyé plus de 500 Mo de données le même jour, la collecte de données s’arrête pour le reste de la journée. La limite quotidienne est la principale raison pour laquelle Log Analytics arrête la collecte de données ou des données semblent manquantes.  Log Analytics crée un événement de type Opération lorsque la collecte de données démarre et s’arrête. Exécutez la requête suivante dans la recherche pour vérifier si vous atteignez la limite quotidienne et si des données sont manquantes : 

```kusto
Operation | where OperationCategory == 'Data Collection Status'
```

Lorsque la collecte de données s’arrête, le paramètre OperationStatus a la valeur **Warning**. Lorsque la collecte de données démarre, le paramètre OperationStatus a la valeur **Succeeded**. Le tableau suivant décrit les raisons pour lesquelles la collecte de données s’arrête et suggère une action pour la reprendre :  

|Raison pour laquelle la collecte s’arrête| Solution| 
|-----------------------|---------|
|La limite quotidienne de votre espace de travail a été atteinte|Attendez que la collecte redémarre automatiquement ou augmentez la limite du volume de données quotidien décrite dans la section Gérer le volume de données maximal quotidien. L’heure de réinitialisation de la limite quotidienne s’affiche sur la page **Limite quotidienne**. |
| Votre espace de travail a atteint le [débit d’ingestion de données](../service-limits.md#log-analytics-workspaces) | La limite par défaut du débit d’ingestion de données envoyées à partir de ressources Azure à l’aide des paramètres de diagnostic est d’environ 6 Go/min par espace de travail. Il s’agit d’une valeur approximative dans la mesure où la taille réelle peut varier d’un type de données à l’autre en fonction de la longueur du journal et de son taux de compression. Cette limite ne s’applique pas aux données envoyées à partir d’agents ou de l’API de collecteur de données. Si vous envoyez des données vers un espace de travail unique à un débit supérieur, certaines données sont supprimées et un événement est envoyé toutes les 6 heures à la table Operation de votre espace de travail tant que le seuil est dépassé. Si votre volume d’ingestion continue de dépasser la limite du débit ou si vous pensez l’atteindre bientôt, vous pouvez demander une augmentation de votre espace de travail en envoyant un e-mail à LAIngestionRate@microsoft.com ou en effectuant une demande de support. L’événement à rechercher indiquant une limite de débit d’ingestion de données peut être trouvé par la requête `Operation | where OperationCategory == "Ingestion" | where Detail startswith "The rate of data crossed the threshold"`. |
|Limite quotidienne de niveau tarifaire hérité Gratuit atteinte |Attendez le jour suivant pour que la collecte redémarre automatiquement ou passez à un niveau tarifaire payant.|
|Abonnement Azure à l’état interrompu pour la raison suivante :<br> Fin de l’essai gratuit<br> Expiration du Pass Azure<br> Limite de dépense mensuelle atteinte (par exemple, sur un abonnement MSDN ou Visual Studio)|Passer à un abonnement payant<br> Supprimer la limite ou attendre sa réinitialisation|

Pour être informé de l’arrêt de la collecte de données, utilisez les étapes décrites dans *Créer une limite quotidienne de données*. Utilisez les étapes décrites dans [Créer un groupe d’actions](action-groups.md) pour configurer une action d’e-mail, de webhook ou de runbook pour la règle d’alerte. 

## <a name="limits-summary"></a>Synthèse des limites

D’autres limites de Log Analytics s’appliquent, certaines d’entre elles dépendant du niveau tarifaire de Log Analytics. Celles-ci sont documentées dans [Abonnement Azure et limites, quotas et contraintes de service](../../azure-resource-manager/management/azure-subscription-service-limits.md#log-analytics-workspaces).


## <a name="next-steps"></a>Étapes suivantes

- Consultez [Recherche de données dans les journaux Azure Monitor](../log-query/log-query-overview.md) pour apprendre à utiliser le langage de recherche. Vous pouvez utiliser des requêtes de recherche pour effectuer des analyses supplémentaires sur les données d’utilisation.
- Utilisez les étapes décrites dans [Création d’une alerte de journal](alerts-metric.md) pour être averti lorsqu’un critère de recherche est rempli.
- Utilisez le [ciblage de solution](../insights/solution-targeting.md) pour collecter des données des groupes d’ordinateurs requis uniquement.
- Pour configurer une règle efficace de collecte d’événements, passez en revue [Stratégie de filtrage d’Azure Security Center](../../security-center/security-center-enable-data-collection.md).
- Modifier la [configuration du compteur de performances](data-sources-performance-counters.md).
- Pour modifier vos paramètres de collecte d’événements, consultez [Configuration du journal des événements](data-sources-windows-events.md).
- Pour modifier vos paramètres de collecte de messages syslog, consultez [Configuration syslog](data-sources-syslog.md).
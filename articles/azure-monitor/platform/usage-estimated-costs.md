---
title: Surveiller l’utilisation et l’estimation des coûts dans Azure Monitor
description: Vue d’ensemble du processus d’utilisation de la page d’utilisation et d’estimation des coûts d’Azure Monitor
author: dalekoetke
services: azure-monitor
ms.topic: conceptual
ms.date: 10/28/2019
ms.author: mbullwin
ms.reviewer: Dale.Koetke
ms.subservice: ''
ms.openlocfilehash: eb96537f67c61fb31759da020068f784d0e89993
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96020192"
---
# <a name="monitoring-usage-and-estimated-costs-in-azure-monitor"></a>Surveiller l’utilisation et l’estimation des coûts dans Azure Monitor

> [!NOTE]
> Cet article explique comment visualiser l’utilisation et les coûts estimés avec plusieurs fonctions de surveillance Azure. Les articles associés aux composants spécifiques d’Azure Monitor sont les suivants :
> - L’article [Gérer l’utilisation et les coûts avec les journaux Azure Monitor](manage-cost-storage.md) décrit comment contrôler les coûts en modifiant la période de conservation des données et comment analyser l’utilisation de vos données pour créer des alertes.
> - L’article [Gérer l’utilisation et les coûts pour Application Insights](../app/pricing.md) décrit comment analyser l’utilisation des données dans Application Insights.

## <a name="azure-monitor-pricing-model"></a>Modèle de tarification Azure Monitor

Le modèle de facturation de base d’Azure Monitor est un tarif adapté au cloud et basé sur la consommation (« paiement à l’utilisation »). Vous paierez uniquement pour ce que vous utiliserez. Les détails de tarification sont disponibles pour les [alertes, mesures, notifications](https://azure.microsoft.com/pricing/details/monitor/), [Log Analytics](https://azure.microsoft.com/pricing/details/log-analytics/) et [Application Insights](https://azure.microsoft.com/pricing/details/application-insights/). 

En plus du modèle Paiement à l’utilisation pour les données de journal, Log Analytics possède des niveaux de réservation de capacité qui vous permettent d’économiser jusqu’à 25 % par rapport au tarif de Paiement à l’utilisation. Les tarifs de la réservation de capacité vous permettent d’acheter une réservation à partir de 100 Go/jour. Toute utilisation au-dessus du niveau de réservation sera facturée au tarif de paiement à l’utilisation. [En savoir plus](https://azure.microsoft.com/pricing/details/monitor/) sur la tarification de la réservation de capacité.

Certains clients auront accès aux [niveaux tarifaires hérités de Log Analytics](./manage-cost-storage.md#legacy-pricing-tiers) et au [niveau tarifaire d’entreprise hérité d’Application Insights](../app/pricing.md#legacy-enterprise-per-node-pricing-tier). 

## <a name="understanding-your-azure-monitor-costs"></a>Comprendre vos coûts pour Azure Monitor

Il y a deux phases pour la compréhension des coûts. La première consiste à envisager Azure Monitor comme solution de supervision. 

### <a name="estimating-the-costs-to-manage-your-environment"></a>Estimation des coûts de gestion de votre environnement

Si vous n’utilisez pas encore les journaux Azure Monitor, vous pouvez utiliser la [calculatrice de prix Azure Monitor](https://azure.microsoft.com/pricing/calculator/?service=monitor) pour estimer le coût d’utilisation de Azure Monitor. Commencez par entrer « Azure Monitor » dans la zone de recherche, puis cliquez sur la vignette Azure Monitor obtenue. Faites défiler la page jusqu’à Azure Monitor et sélectionnez une des options dans la liste déroulante Type :

- Requêtes de métriques et alertes  
- Log Analytics
- Application Insights

Dans chacun de ces cas, la calculatrice de prix vous aidera à estimer vos coûts probables en fonction de votre utilisation prévue.

Par exemple, avec Log Analytics, vous pouvez entrer le nombre de machines virtuelles et les Go de données que vous prévoyez de collecter à partir de chaque machine virtuelle. En général, 1 à 3 Go de données par mois sont ingérés à partir d’une machine virtuelle Azure standard. Si vous évaluez déjà les journaux Azure Monitor, vous pouvez utiliser vos statistiques de données à partir de votre propre environnement. Vous verrez ci-dessous comment déterminer le [nombre de machines virtuelles surveillées](./manage-cost-storage.md#understanding-nodes-sending-data) et le [volume de données que votre espace de travail ingère](./manage-cost-storage.md#understanding-ingested-data-volume).

Comme pour Application Insights, si vous activez la fonctionnalité « Estimer le volume de données en fonction de l’activité de l’application », vous pouvez fournir des entrées relatives à votre application (demandes par mois et affichages de pages par mois, dans l’éventualité où vous collecterez les données de télémétrie côté client), puis la calculatrice vous indiquera le volume médian et la quantité 90e centile de données collectées par des applications similaires. Ces applications couvrent la plage de configuration d’Application Insights (p. ex., certains ont l’échantillonnage par défaut, d’autres n’ont aucun échantillonnage, etc.) et vous disposez encore du contrôle permettant de réduire le volume de données que vous ingérez par échantillonnage bien au-dessous du niveau médian. Toutefois, il s’agit d’un point de départ pour comprendre ce que les autres clients similaires voient. [En savoir plus](../app/pricing.md#estimating-the-costs-to-manage-your-application) sur l’estimation des coûts pour Application Insights.

### <a name="understanding-your-usage-and-estimated-costs"></a>Comprendre votre utilisation et vos coûts estimés

Il est important de comprendre et de suivre votre utilisation lorsque vous utilisez Azure Monitor, et il existe un ensemble complet d’outils pour faciliter cette tâche. 

Azure fournit de nombreuses fonctionnalités utiles dans le hub [Azure Cost Management + facturation](../../cost-management-billing/costs/quick-acm-cost-analysis.md?toc=/azure/billing/TOC.json). Après avoir ouvert le hub **Azure Cost Management + Facturation**, cliquez sur **Cost Management** et sélectionnez une [Étendue](../../cost-management-billing/costs/understand-work-scopes.md) (l’ensemble de ressources à examiner). 

Ensuite, pour voir les coûts d’Azure Monitor pour les 30 derniers jours, cliquez sur la vignette les **Coûts quotidiens**, choisissez « 30 derniers jours » sous Dates relatives, puis ajoutez un filtre qui sélectionne les noms de service :

1. Azure Monitor
2. Application Insights
3. Log Analytics
4. Insight & Analytics

Cela génère une vue telle que :

![Capture d’écran d’Azure Cost Management](./media/usage-estimated-costs/010.png)

De là, vous pouvez explorer ce récapitulatif des coûts accumulés pour obtenir des détails plus précis dans la vue « Coût par ressource ». Dans les niveaux tarifaires actuels, les données des journaux Azure sont facturées sur le même ensemble de compteurs, qu’elles proviennent de Log Analytics ou d’Application Insights. Pour séparer les coûts de Log Analytics et d’Application Insights, vous pouvez ajouter un filtre sur **Type de ressource**. Pour afficher tous les coûts d’Application Insights, filtrez le type de ressource avec « Microsoft.insights/components », et pour les coûts de Log Analytics, filtrez le type de ressource avec « Microsoft.operationalinsights/workspaces ». 

Vous trouverez plus de détails sur votre utilisation en [téléchargeant votre utilisation à partir du portail Azure](../../cost-management-billing/manage/download-azure-invoice-daily-usage-date.md#download-usage-in-azure-portal). Dans la feuille de calcul téléchargée, vous pouvez voir l’utilisation par ressource Azure par jour. Dans cette feuille de calcul Excel, vous pouvez trouver l’utilisation de vos ressources Application Insights en filtrant d’abord la colonne « Catégorie du compteur » pour afficher « Application Insights » et « Log Analytics », puis en ajoutant le filtre « contient microsoft.insights/components » sur la colonne « ID d’instance ».  La plus grande part de l’utilisation d’Application Insights est signalée sur des compteurs avec la catégorie du compteur de Log Analytics, car il existe un seul backend de journaux pour tous les composants Azure Monitor.  Seules les ressources Application Insights sur les niveaux tarifaires hérités et les tests web à plusieurs étapes sont signalées avec une catégorie de compteur d’Application Insights.  L’utilisation est indiquée dans la colonne « Quantité consommée » et l’unité pour chaque entrée est affichée dans la colonne « Unité de mesure ».  Plus de détails sont disponibles pour vous aider à [comprendre votre facture Microsoft Azure](../../cost-management-billing/understand/review-individual-bill.md). 

> [!NOTE]
> L’utilisation de **Cost Management** dans le hub **Azure Cost Management + Facturation** est l’approche à privilégier pour comprendre les coûts d’analyse.  Les expériences **Utilisation et estimation des coûts** pour [Log Analytics](./manage-cost-storage.md#understand-your-usage-and-estimate-costs) et [Application Insights](../app/pricing.md#understand-your-usage-and-estimate-costs) fournissent des informations plus approfondies pour chacune de ces parties d’Azure Monitor.

Une autre option pour l’affichage de votre utilisation d’Azure Monitor est la page **Utilisation et estimation des coûts** dans le hub de Monitor. Cela montre l’utilisation des principales fonctionnalités de surveillance comme les [alertes, mesures, notifications](https://azure.microsoft.com/pricing/details/monitor/), [Azure Log Analytics](https://azure.microsoft.com/pricing/details/log-analytics/) et [Azure Application Insights](https://azure.microsoft.com/pricing/details/application-insights/). Pour les clients ayant souscrit aux plans de tarification avant avril 2018, cela inclut également l’utilisation de Log Analytics acheté via l’offre Insights et Analytics.

Sur cette page, les utilisateurs peuvent afficher l’utilisation de leurs ressources au cours des 31 derniers jours, agrégée par abonnement. Les `Drill-ins` vous permettent de voir les tendances au cours de cette période de 31 jours. De nombreuses données doivent être rassemblées pour fournir cette estimation. Vous devez donc attendre patiemment le chargement de la page.

Cet exemple illustre l’utilisation de la surveillance et l’estimation des coûts associés :

![Capture d’écran du portail d’utilisation et d’estimation des coûts](./media/usage-estimated-costs/001.png)

Sélectionnez le lien dans la colonne d’utilisation mensuelle pour ouvrir un graphique illustrant les tendances d’utilisation au cours des 31 derniers jours : 

![Capture d’écran du graphique en barres « Inclus par nœud »](./media/usage-estimated-costs/002.png)

## <a name="operations-management-suite-subscription-entitlements"></a>Droits d’abonnement à Operations Management Suite

Les clients ayant acheté Microsoft Operations Management Suite E1 et E2 peuvent bénéficier des droits d’ingestion des données par nœud pour [Log Analytics](https://www.microsoft.com/cloud-platform/operations-management-suite) et [Application Insights](../app/pricing.md). Pour que ces droits applicables aux espaces de travail Log Analytics ou aux ressources Application Insights puissent être reçus dans un abonnement donné : 

- Les espaces de travail Log Analytics doivent utiliser le niveau tarifaire « Par nœud (OMS) ».
- Les ressources Application Insights doivent utiliser le niveau tarifaire « Entreprise ».

En fonction du nombre de nœuds de la suite que votre organisation a achetés, le transfert de certains abonnements vers le niveau tarifaire de paiement à l'utilisation (par Go) pourrait se révéler avantageux, mais cette approche doit faire l’objet d’une étude approfondie.

> [!WARNING]
> Si votre organisation a actuellement Microsoft Operations Management Suite E1 et E2, il est généralement préférable de conserver vos espaces de travail Log Analytics au niveau tarifaire « Par nœud (OMS) » et vos ressources Application Insights au niveau tarifaire « Entreprise ». 
>


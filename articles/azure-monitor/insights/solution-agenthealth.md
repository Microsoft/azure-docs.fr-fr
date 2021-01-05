---
title: Solution Agent Health dans Azure Monitor | Microsoft Docs
description: Cet article a pour but de vous apprendre à utiliser cette solution afin de surveiller l’intégrité de vos agents au service de Log Analytics ou de System Center Operations Manager.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 02/06/2020
ms.openlocfilehash: 4f14f006283b7430458d67d2bd3bee787c08411d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87326016"
---
#  <a name="agent-health-solution-in-azure-monitor"></a>Solution Agent Health dans Azure Monitor
La solution Agent Health dans Azure vous permet de savoir quels agents, parmi tous ceux au service de l’espace de travail Log Analytics dans Azure Monitor ou d’un groupe d’administration System Center Operations Manager connecté à Azure Monitor, ne répondent pas et envoient des données opérationnelles.  Vous pouvez aussi suivre le nombre d’agents déployés et leur localisation géographique, et réaliser diverses requêtes pour être au fait de la distribution d’agents déployés au sein d’Azure, d’environnements de cloud ou localement.    

## <a name="prerequisites"></a>Conditions préalables requises
Avant de déployer cette solution, veuillez confirmer que vous avez bien pris en charge des [agents Windows](../platform/agent-windows.md) au service de l’espace de travail Log Analytics ou d’un [groupe d’administration Operations Manager](../platform/om-agents.md) inclut dans votre espace de travail.

## <a name="solution-components"></a>Composants de la solution
Cette solution se compose des ressources suivantes qui sont ajoutées à votre espace de travail et d’agents directement connectés ou d’un groupe d’administration Operations Manager connecté.

### <a name="management-packs"></a>Packs d’administration
Si votre groupe d’administration System Center Operations Manager est connecté à un espace de travail Log Analytics, les packs d’administration suivants sont installés dans Operations Manager.  Ces packs d’administration sont également installés sur des ordinateurs Windows directement connectés après l’ajout de cette solution. Aucun élément ne doit être configuré ou géré avec ces packs d’administration.

* Microsoft System Center Advisor HealthAssessment Direct Channel Intelligence Pack  (Microsoft.IntelligencePacks.HealthAssessmentDirect)
* Microsoft System Center Advisor HealthAssessment Server Channel Intelligence Pack (Microsoft.IntelligencePacks.HealthAssessmentViaServer).  

Pour plus d’informations sur la façon dont ces packs d’administration de solution sont mis à jour, consultez [Connecter Operations Manager à Log Analytics](../platform/om-agents.md).

## <a name="configuration"></a>Configuration
Ajoutez la solution Agent Health à votre espace de travail Log Analytics en suivant la procédure décrite dans [Ajouter des solutions](solutions.md). Aucune configuration supplémentaire n’est requise.


## <a name="data-collection"></a>Collecte de données
### <a name="supported-agents"></a>Agents pris en charge
Le tableau suivant décrit les sources connectées qui sont prises en charge par cette solution.

| Source connectée | Prise en charge | Description |
| --- | --- | --- |
| Agents Windows | Oui | Les événements de pulsation sont rassemblés par des agents Windows directs.|
| Groupe d’administration Microsoft System Center Operations Manager | Oui | Les événements de pulsation sont collectés toutes les 60 secondes par les agents au service du groupe d’administration et sont ensuite transférés à Azure Monitor. Une connexion directe entre les agents Operations Manager et Azure Monitor n’est pas obligatoire. Les données d’événement de pulsation sont transférées depuis le groupe d’administration vers l’espace de travail Log Analytics.|

## <a name="using-the-solution"></a>Utilisation de la solution
Quand vous ajoutez la solution à votre espace de travail Log Analytics, la vignette **Agent Health** est ajoutée à votre tableau de bord. La vignette indique le nombre total d’agents et le nombre d’agents inactifs au cours des dernières 24 heures.<br><br> ![Vignette de la solution Agent Health du tableau de bord](./media/solution-agenthealth/agenthealth-solution-tile-homepage.png)

Cliquez sur la vignette **Agent Health** pour ouvrir le tableau de bord **Agent Health**.  Le tableau de bord comprend les colonnes figurant dans le tableau suivant. Chaque colonne répertorie les dix premiers événements, classés selon leur nombre, correspondant aux critères de cette colonne pour l’intervalle de temps spécifié. Vous pouvez exécuter une recherche dans les journaux qui fournit la liste complète. Pour cela, sélectionnez **Afficher tout** dans l’angle inférieur droit de chaque colonne ou cliquez sur l’en-tête de colonne.

| Colonne | Description |
|--------|-------------|
| Nombre d’agents au fil du temps | Tendance du nombre d’agents sur une période de sept jours (agents Linux et Windows).|
| Nombre d’agents inactifs | Liste des agents n’ayant émis aucune pulsation au cours des dernières 24 heures.|
| Distribution selon le système d’exploitation | Partition du nombre d’agents Windows et Linux dont vous disposez dans votre environnement.|
| Distribution selon la version de l’agent | Partition des versions des agents installées dans votre environnement, et leur nombre.|
| Distribution selon la catégorie de l’agent | Partition des différentes catégories d’agents émettant des événements de pulsation : agents directs, agents Operations Manager ou serveur d’administration Operations Manager.|
| Distribution selon le groupe d’administration | Partition des différents groupes d’administration Operations Manager de votre environnement.|
| Géolocalisation des agents | Partition des différents pays/régions dans lesquels vous disposez d’agents et nombre total d’agents installés par pays/région.|
| Nombre de passerelles installées | Nombre et liste des serveurs disposant de la passerelle Log Analytics.|

![Exemple de tableau de bord pour la solution Agent Health](./media/solution-agenthealth/agenthealth-solution-dashboard.png)  

## <a name="azure-monitor-log-records"></a>Enregistrements de journaux Azure Monitor
La solution crée un type d’enregistrement dans l’espace de travail Log Analytics.  

### <a name="heartbeat-records"></a>Enregistrements de pulsation
L’enregistrement d’un type de **pulsation** est créé.  Les propriétés de ces enregistrements sont décrites dans le tableau suivant.  

| Propriété | Description |
| --- | --- |
| `Type` | *Pulsation*|
| `Category` | La valeur correspond à *Agent Direct*, *Agent SCOM*, ou *Serveur d’administration SCOM*.|
| `Computer` | Nom de l’ordinateur.|
| `OSType` | Système d’exploitation : Windows ou Linux.|
| `OSMajorVersion` | Version principale du système d’exploitation.|
| `OSMinorVersion` | Version secondaire du système d’exploitation.|
| `Version` | Version de l’agent Log Analytics ou de l’agent Operations Manager.|
| `SCAgentChannel` | La valeur correspond à *Direct* et/ou *SCManagementServer*.|
| `IsGatewayInstalled` | Si la passerelle Log Analytics est installée, la valeur correspond à *true*. Dans le cas contraire, elle correspond à *false*.|
| `ComputerIP` | IP publique de l’ordinateur. Sur les machines virtuelles Azure, l’IP publique s’affiche, le cas échéant. Pour les machines virtuelles utilisant des IP privées, cette opération affiche l’adresse SNAT Azure (et non l’IP privée). |
| `RemoteIPCountry` | Lieu où l’ordinateur est déployé.|
| `ManagementGroupName` | Nom du groupe d’administration Operations Manager.|
| `SourceComputerId` | ID unique de l’ordinateur.|
| `RemoteIPLongitude` | Longitude de l’emplacement géographique de l’ordinateur.|
| `RemoteIPLatitude` | Latitude de l’emplacement géographique de l’ordinateur.|

Tout agent au service d’un serveur d’administration Operations Manager émet deux pulsations. En fonction des sources de données et solutions de supervision activées lors de votre abonnement, la valeur de la propriété SCAgentChannel correspond à la fois à **Direct** et à **SCManagementServer**. Rappel : les données des solutions sont soit envoyées directement à partir d’un serveur d’administration Operations Manager à Azure Monitor, soit, en raison du volume de données collecté par l’agent, envoyées directement à partir de l’agent à Azure Monitor. Pour les événements de pulsation dont la valeur correspond à **SCManagementServer**, la valeur ComputerIP correspond à l’adresse IP du serveur d’administration, car les données sont chargées par ce dernier.  Pour les pulsations dont la valeur SCAgentChannel correspond à **Direct**, la valeur ComputerIP correspond à l’adresse IP publique de l’agent.  

## <a name="sample-log-searches"></a>Exemples de recherches dans les journaux
Le tableau suivant fournit des exemples de recherches de journaux pour les enregistrements collectés par cette solution.

| Requête | Description |
|:---|:---|
| Heartbeat &#124; distinct Computer |Nombre total d’agents |
| Heartbeat &#124; summarize LastCall = max(TimeGenerated) by Computer &#124; where LastCall < ago(24h) |Nombre d’agents inactifs au cours des dernières 24 heures |
| Heartbeat &#124; summarize LastCall = max(TimeGenerated) by Computer &#124; where LastCall < ago(15m) |Nombre d’agents inactifs au cours des 15 dernières heures |
| Heartbeat &#124; where TimeGenerated > ago(24h) and Computer in ((Heartbeat &#124; where TimeGenerated > ago(24h) &#124; distinct Computer)) &#124; summarize LastCall = max(TimeGenerated) by Computer |Ordinateurs en ligne (au cours des dernières 24 heures) |
| Heartbeat &#124; where TimeGenerated > ago(24h) and Computer !in ((Heartbeat &#124; where TimeGenerated > ago(30m) &#124; distinct Computer)) &#124; summarize LastCall = max(TimeGenerated) by Computer |Total des agents hors ligne au cours des 30 dernières minutes (au cours des dernières 24 heures) |
| Heartbeat &#124; summarize AggregatedValue = dcount(Computer) by OSType |Obtenir une tendance du nombre d’agents sur une période selon le système d’exploitation|
| Heartbeat &#124; summarize AggregatedValue = dcount(Computer) by OSType |Distribution selon le système d’exploitation |
| Heartbeat &#124; summarize AggregatedValue = dcount(Computer) by Version |Distribution selon la version de l’agent |
| Heartbeat &#124; summarize AggregatedValue = count() by Category |Distribution selon la catégorie de l’agent |
| Heartbeat &#124; summarize AggregatedValue = dcount(Computer) by ManagementGroupName | Distribution selon le groupe d’administration |
| Heartbeat &#124; summarize AggregatedValue = dcount(Computer) by RemoteIPCountry |Géolocalisation des agents |
| Heartbeat &#124; where iff(isnotnull(toint(IsGatewayInstalled)), IsGatewayInstalled == true, IsGatewayInstalled == "true") == true &#124; distinct Computer |Nombre de passerelles Log Analytics installées |




## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur la génération d’alertes à partir de requêtes de journal, consultez [Alertes dans Azure Monitor](../platform/alerts-overview.md). 


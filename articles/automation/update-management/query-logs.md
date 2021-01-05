---
title: Interroger les journaux Update Management Azure Automation
description: Cet article explique comment interroger les journaux pour Update Management dans votre espace de travail Log Analytics.
services: automation
ms.subservice: update-management
ms.date: 09/24/2020
ms.topic: conceptual
ms.openlocfilehash: 833e2f7808b4b8efa210bc6a903ed30fe9ac53e0
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92221565"
---
# <a name="query-update-management-logs"></a>Interroger les journaux Update Management

Outre les détails fournis pendant le déploiement d’Update Management, vous pouvez effectuer des recherches dans les journaux stockés dans votre espace de travail Log Analytics. Pour effectuer des recherches dans les journaux à partir de votre compte Automation, sélectionnez **Update Management**, puis ouvrez l’espace de travail Log Analytics associé à votre déploiement.

Vous pouvez également personnaliser les requêtes de journal ou les utiliser à partir de différents clients. Voir [Documentation de l’API de recherche Log Analytics](https://dev.loganalytics.io/).

## <a name="query-update-records"></a>Interroger des enregistrements de mises à jour

Update Management collecte les enregistrements des machines virtuelles Windows et Linux et les types de données qui apparaissent dans les résultats de la recherche dans les journaux. Les sections suivantes décrivent ces enregistrements.

### <a name="query-required-updates"></a>Interroger les mises à jour requises

Un enregistrement du type `RequiredUpdate` est créé, qui représente les mises à jour requises par un ordinateur. Les propriétés de ces enregistrements sont décrites dans le tableau suivant :

| Propriété | Description |
|----------|-------------|
| Computer | Nom de domaine complet de la machine qui rend compte. |
| KBID | ID d’article de la base de connaissances pour la mise à jour Windows. |
| ManagementGroupName | Nom du groupe d’administration Operations Manager ou de l’espace de travail Log Analytics. |
| Produit | Produits pour lesquels la mise à jour est applicable. |
| PublishDate | Date à laquelle la mise à jour est prête à être téléchargée et installée à partir de Windows Update. |
| Serveur | | 
| SourceHealthServiceId | Identificateur unique représentant l’ID de l’agent Windows Log Analytics. |
| SourceSystem | *OperationsManager* |
| TenantId | Identificateur unique représentant l’instance Azure Active Directory de votre organisation. |
| TimeGenerated | Date et heure de création de l’enregistrement. |
| Type | *Mettre à jour* |
| UpdateClassification | Type des mises à jour pouvant être appliquées. Pour Windows :<br> *Mises à jour critiques*<br> *Mises à jour de sécurité*<br> *Correctifs cumulatifs*<br> *Packs de fonctionnalités*<br> *Service Packs*<br> *Mises à jour de définitions*<br> *outils*<br> *Mises à jour*. Pour Linux :<br> *Mises à jour critiques et de sécurité*<br> *Autres* |
| UpdateSeverity | Degré de gravité de la vulnérabilité. Les valeurs sont les suivantes :<br> *Critical*<br> *Important*<br> *Moderate*<br> *Low* |
| UpdateTitle | Titre de la mise à jour.|

### <a name="query-update-record"></a>Interroger un enregistrement de mise à jour

Un enregistrement du type `Update` est créé, qui représente les mises à jour disponibles et leur état d’installation pour un ordinateur. Les propriétés de ces enregistrements sont décrites dans le tableau suivant :

| Propriété | Description |
|----------|-------------|
| ApprovalSource | S’applique au système d’exploitation Windows uniquement. Source d’approbation de l’enregistrement. La valeur est Microsoft Update. |
| Approved | True si l’enregistrement est approuvé, False dans le cas contraire. |
| classification ; | Classification de l’approbation. La valeur est Updates. |
| Computer | Nom de domaine complet de la machine qui rend compte. |
| ComputerEnvironment | Environnement. Les valeurs possibles sont Azure ou non-Azure. |
| MSRCBulletinID | Numéro d’identification du bulletin de sécurité. |
| MSRCSeverity | Degré de gravité de la vulnérabilité. Les valeurs sont les suivantes :<br> Critique<br> Important<br> Modéré<br> Faible |  
| KBID | ID d’article de la base de connaissances pour la mise à jour Windows. |
| ManagementGroupName | Nom du groupe d’administration Operations Manager ou de l’espace de travail Log Analytics. |
| UpdateID | Identificateur unique de la mise à jour de logiciel. |
| RevisionNumber | Numéro de révision d’une révision spécifique d’une mise à jour. |
| Facultatif | True si l’enregistrement est facultatif, False dans le cas contraire. |
| RebootBehavior | Comportement de redémarrage après l’installation ou la désinstallation d’une mise à jour. |
| _ResourceId | Identificateur unique de la ressource associée à l’enregistrement. |
| Type | Type d’enregistrement. La valeur est Update. |
| VMUUID | Identificateur unique de la machine virtuelle. |
| MG | Identificateur unique du groupe d’administration ou de l’espace de travail Log Analytics. |
| TenantId | Identificateur unique représentant l’instance Azure Active Directory de votre organisation. |
| SourceSystem | Système source de l’enregistrement. La valeur est `OperationsManager`. |
| TimeGenerated | Date et heure de création de l’enregistrement. |
| SourceComputerId | Identificateur unique représentant l’ordinateur source. |
| Intitulé | Titre de la mise à jour. |
| PublishedDate (UTC) | Date à laquelle la mise à jour est prête à être téléchargée et installée à partir de Windows Update.  |
| UpdateState | État actuel de la mise à jour. |
| Produit | Produits pour lesquels la mise à jour est applicable. |
| SubscriptionId | Identificateur unique de l’abonnement Azure. |
| ResourceGroup | Nom du groupe de ressources auquel appartient la ressource. |
| ResourceProvider | Fournisseur de ressources. |
| Ressource | Nom de la ressource. |
| ResourceType | Type de ressource. |

### <a name="query-update-agent-record"></a>Interroger un enregistrement d’agent de mise à jour

Un enregistrement du type `UpdateAgent` est créé, qui fournit des détails sur l’agent de mise à jour sur l’ordinateur. Les propriétés de ces enregistrements sont décrites dans le tableau suivant :

| Propriété | Description |
|----------|-------------|
| AgeofOldestMissingRequiredUpdate | |
| AutomaticUpdateEnabled | |
| Computer | Nom de domaine complet de la machine qui rend compte. |
| DaySinceLastUpdateBucket | |
| ManagementGroupName | Nom du groupe d’administration Operations Manager ou de l’espace de travail Log Analytics. |
| OSVersion: | Version du système d’exploitation. |
| Serveur | |
| SourceHealthServiceId | Identificateur unique représentant l’ID de l’agent Windows Log Analytics. |
| SourceSystem | Système source de l’enregistrement. La valeur est `OperationsManager`. |
| TenantId | Identificateur unique représentant l’instance Azure Active Directory de votre organisation. |
| TimeGenerated | Date et heure de création de l’enregistrement. |
| Type | Type d’enregistrement. La valeur est Update. |
| WindowsUpdateAgentVersion | Version de l’agent Windows Update. |
| WSUSServer | Erreurs si l’agent Windows Update a un problème, pour faciliter la résolution des problèmes. |

### <a name="query-update-deployment-status-record"></a>Interroger un enregistrement d’état du déploiement des mises à jour

Un enregistrement du type `UpdateRunProgress` est créé, qui fournit l’état du déploiement des mises à jour d’un déploiement planifié par ordinateur. Les propriétés de ces enregistrements sont décrites dans le tableau suivant :

| Propriété | Description |
|----------|-------------|
| Computer | Nom de domaine complet de la machine qui rend compte. |
| ComputerEnvironment | Environnement. Les valeurs sont Azure ou non-Azure. |
| CorrelationId | Identificateur unique de la tâche runbook à exécuter pour la mise à jour. |
| EndTime | Heure de fin du processus de synchronisation. *Cette propriété n’est pas utilisée actuellement. Consultez TimeGenerated.* |
| ErrorResult | Code d’erreur Windows Update généré en cas d’échec de l’installation d’une mise à jour. |
| InstallationStatus | Les états d’installation possibles d’une mise à jour sur l’ordinateur client sont :<br> `NotStarted` : la tâche n’a pas encore été déclenchée.<br> `FailedToStart` : impossible de démarrer la tâche sur l’ordinateur.<br> `Failed` : la tâche a démarré, mais a échoué avec une exception.<br> `InProgress` : tâche en cours.<br> `MaintenanceWindowExceeded` : si l’exécution était encore en cours, mais que l’intervalle de la fenêtre de maintenance a été atteint.<br> `Succeeded` : la tâche a réussi.<br> `InstallFailed` : échec de l’installation de la mise à jour.<br> `NotIncluded`<br> `Excluded` |
| KBID | ID d’article de la base de connaissances pour la mise à jour Windows. |
| ManagementGroupName | Nom du groupe d’administration Operations Manager ou de l’espace de travail Log Analytics. |
| Système d’exploitation | Type de système d'exploitation. Les valeurs sont Windows ou Linux. |
| Produit | Produits pour lesquels la mise à jour est applicable. |
| Ressource | Nom de la ressource. |
| ResourceId | Identificateur unique de la ressource associée à l’enregistrement. |
| ResourceProvider | Fournisseur de ressources. |
| ResourceType | Type de ressource. |
| SourceComputerId | Identificateur unique représentant l’ordinateur source. |
| SourceSystem | Système source de l’enregistrement. La valeur est `OperationsManager`. |
| StartTime | Heure à laquelle l’installation de la mise à jour est planifiée. *Cette propriété n’est pas utilisée actuellement. Consultez TimeGenerated.* |
| SubscriptionId | Identificateur unique de l’abonnement Azure. |
| SucceededOnRetry | Valeur indiquant si l’exécution de la mise à jour a échoué lors de la première tentative et si l’opération en cours est une nouvelle tentative. |
| TimeGenerated | Date et heure de création de l’enregistrement. |
| Intitulé | Titre de la mise à jour. |
| Type | Type de mise à jour. La valeur est `UpdateRunProgress`. |
| UpdateId | Identificateur unique de la mise à jour de logiciel. |
| VMUUID | Identificateur unique de la machine virtuelle. |
| ResourceId | Identificateur unique de la ressource associée à l’enregistrement. |

### <a name="query-update-summary-record"></a>Interroger un enregistrement de résumé des mises à jour

Un enregistrement du type `UpdateSummary` est créé, qui fournit un résumé des mises à jour par ordinateur. Les propriétés de ces enregistrements sont décrites dans le tableau suivant :

| Propriété | Description |
|----------|-------------|
| Computer | Nom de domaine complet de la machine qui rend compte. |
| ComputerEnvironment | Environnement. Les valeurs sont Azure ou non-Azure. |
| CriticalUpdatesMissing | Nombre de mises à jour critiques applicables qui font défaut. |
| ManagementGroupName | Nom du groupe d’administration Operations Manager ou de l’espace de travail Log Analytics. |
| NETRuntimeVersion | Version de .NET Framework installée sur l’ordinateur Windows. |
| OldestMissingSecurityUpdateBucket | Spécificateur du plus ancien compartiment de sécurité manquant. Les valeurs sont les suivantes :<br> Récent si la valeur est inférieure à 30 jours<br> Il y a 30 jours<br> Il y a 60 jours<br> Il y a 90 jours<br> Il y a 120 jours<br> Il y a 150 jours<br> Il y a 180 jours<br> Plus ancien lorsque la valeur est supérieure à 180 jours |
| OldestMissingSecurityUpdateInDays | Nombre total de jours pour la mise à jour la plus ancienne ayant été détectée comme applicable mais n’ayant pas été installée. |
| OsVersion | Version du système d’exploitation. |
| OtherUpdatesMissing | Nombre de mises à jour détectées manquantes. |
| Ressource | Nom de la ressource pour l’enregistrement. |
| ResourceGroup | Nom du groupe de ressources contenant la ressource. |
| ResourceId | Identificateur unique de la ressource associée à l’enregistrement. |
| ResourceProvider | Fournisseur de ressources. |
| ResourceType | Type de ressource. |
| RestartPending | True si un redémarrage est en attente, False dans le cas contraire. |
| SecurityUpdatesMissing | Nombre de correctifs de sécurité manquants qui sont applicables.|
| SourceComputerId | Identificateur unique de la machine virtuelle. |
| SourceSystem | Système source de l’enregistrement. La valeur est `OpsManager`. |
| SubscriptionId | Identificateur unique de l’abonnement Azure. |
| TimeGenerated | Date et heure de création de l’enregistrement. |
| TotalUpdatesMissing | Nombre total de mises à jour manquantes qui sont applicables. |
| Type | Type d’enregistrement. La valeur est `UpdateSummary`. |
| VMUUID | Identificateur unique de la machine virtuelle. |
| WindowsUpdateAgentVersion | Version de l’agent Windows Update. |
| WindowsUpdateSetting | État de l’agent Windows Update. Les valeurs possibles sont les suivantes :<br> `Scheduled installation`<br> `Notify before installation`<br> `Error returned from unhealthy WUA agent` |
| WSUSServer | Erreurs si l’agent Windows Update a un problème, pour faciliter la résolution des problèmes. |
| _ResourceId | Identificateur unique de la ressource associée à l’enregistrement. |

## <a name="sample-queries"></a>Exemples de requêtes

Les sections suivantes présentent des exemples de requêtes sur les journaux d’enregistrements des mises à jour qui sont collectés pour Update Management.

### <a name="confirm-that-non-azure-machines-are-enabled-for-update-management"></a>Vérifier que les machines non-Azure sont activées pour Update Management

Pour vérifier que les machines directement connectées communiquent avec les journaux Azure Monitor, exécutez l’une des recherches suivantes dans les journaux.

#### <a name="linux"></a>Linux

```loganalytics
Heartbeat
| where OSType == "Linux" | summarize arg_max(TimeGenerated, *) by SourceComputerId | top 500000 by Computer asc | render table
```

#### <a name="windows"></a>Windows

```loganalytics
Heartbeat
| where OSType == "Windows" | summarize arg_max(TimeGenerated, *) by SourceComputerId | top 500000 by Computer asc | render table
```

Sur un ordinateur Windows, vous pouvez vérifier les informations suivantes pour tester la connectivité de l’agent avec les journaux Azure Monitor :

1. Dans le Panneau de configuration, ouvrez **Microsoft Monitoring Agent**. Sous l’onglet **Azure Log Analytics**, l’agent affiche le message suivant : **Microsoft Monitoring Agent s’est correctement connecté à Log Analytics.**

1. Ouvrez le journal des événements Windows. Accédez à **Application and Services Logs\Operations Manager**, puis recherchez l’ID d’événement 3000 et 5002 à partir du **connecteur de service** source. Ces événements indiquent que l’ordinateur est enregistré sur l’espace de travail Log Analytics et qu’il reçoit la configuration.

Si l’agent ne parvient pas à communiquer avec les journaux Azure Monitor et s’il est configuré pour communiquer avec Internet par le biais d’un pare-feu ou d’un serveur proxy, vérifiez que le pare-feu ou le serveur proxy est correctement configuré. Pour savoir comment vérifier la configuration du pare-feu ou du serveur proxy, consultez [Configuration réseau de l’agent Windows](../../azure-monitor/platform/agent-windows.md) ou [Configuration réseau de l’agent Linux](../../azure-monitor/learn/quick-collect-linux-computer.md).

> [!NOTE]
> Si vos systèmes Linux sont configurés pour communiquer avec un proxy ou Log Analytics Gateway et que vous activez Update Management, vous devez mettre à jour les autorisations `proxy.conf` pour accorder au groupe omiuser une autorisation d’accès en lecture sur le fichier. Pour cela, exécutez les commandes suivantes :
>
> `sudo chown omsagent:omiusers /etc/opt/microsoft/omsagent/proxy.conf`
> `sudo chmod 644 /etc/opt/microsoft/omsagent/proxy.conf`

Les nouveaux agents Linux ajoutés affichent l’état **Mis à jour** après l’exécution d’une évaluation. Ce processus peut prendre jusqu’à 6 heures.

Pour vérifier qu’un groupe d’administration Operations Manager communique avec les journaux Azure Monitor, consultez [Valider l’intégration d’Operations Manager aux journaux Azure Monitor](../../azure-monitor/platform/om-agents.md#validate-operations-manager-integration-with-azure-monitor).

### <a name="single-azure-vm-assessment-queries-windows"></a>Requêtes d’évaluation de la machine virtuelle Azure unique (Windows)

Remplacez la valeur VMUUID par le GUID VM de la machine virtuelle que vous interrogez. Pour trouver le VMUUID à utiliser, exécutez la requête suivante dans les journaux Azure Monitor : `Update | where Computer == "<machine name>" | summarize by Computer, VMUUID`

#### <a name="missing-updates-summary"></a>Récapitulatif des mises à jour manquantes

```loganalytics
Update
| where TimeGenerated>ago(14h) and OSType!="Linux" and (Optional==false or Classification has "Critical" or Classification has "Security") and VMUUID=~"b08d5afa-1471-4b52-bd95-a44fea6e4ca8"
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, Approved) by Computer, SourceComputerId, UpdateID
| where UpdateState=~"Needed" and Approved!=false
| summarize by UpdateID, Classification
| summarize allUpdatesCount=count(), criticalUpdatesCount=countif(Classification has "Critical"), securityUpdatesCount=countif(Classification has "Security"), otherUpdatesCount=countif(Classification !has "Critical" and Classification !has "Security")
```

#### <a name="missing-updates-list"></a>Liste des mises à jour manquantes

```loganalytics
Update
| where TimeGenerated>ago(14h) and OSType!="Linux" and (Optional==false or Classification has "Critical" or Classification has "Security") and VMUUID=~"8bf1ccc6-b6d3-4a0b-a643-23f346dfdf82"
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, Title, KBID, PublishedDate, Approved) by Computer, SourceComputerId, UpdateID
| where UpdateState=~"Needed" and Approved!=false
| project-away UpdateState, Approved, TimeGenerated
| summarize computersCount=dcount(SourceComputerId, 2), displayName=any(Title), publishedDate=min(PublishedDate), ClassificationWeight=max(iff(Classification has "Critical", 4, iff(Classification has "Security", 2, 1))) by id=strcat(UpdateID, "_", KBID), classification=Classification, InformationId=strcat("KB", KBID), InformationUrl=iff(isnotempty(KBID), strcat("https://support.microsoft.com/kb/", KBID), ""), osType=2
| sort by ClassificationWeight desc, computersCount desc, displayName asc
| extend informationLink=(iff(isnotempty(InformationId) and isnotempty(InformationUrl), toobject(strcat('{ "uri": "', InformationUrl, '", "text": "', InformationId, '", "target": "blank" }')), toobject('')))
| project-away ClassificationWeight, InformationId, InformationUrl
```

### <a name="single-azure-vm-assessment-queries-linux"></a>Requêtes d’évaluation de la machine virtuelle Azure unique (Linux)

Pour certaines distributions de Linux, il existe une incompatibilité de [mode Endian](https://en.wikipedia.org/wiki/Endianness) entre la valeur VMUUID qui provient d’Azure Resource Manager et ce qui est stocké dans les journaux d’activité Azure Monitor. La requête suivante recherche une correspondance sur l’un des modes Endian. Remplacez les valeurs VMUUID avec le format big-endian et little-endian du GUID afin de retourner correctement les résultats. Pour trouver le VMUUID à utiliser, exécutez la requête suivante dans les journaux Azure Monitor : `Update | where Computer == "<machine name>"
| summarize by Computer, VMUUID`

#### <a name="missing-updates-summary"></a>Récapitulatif des mises à jour manquantes

```loganalytics
Update
| where TimeGenerated>ago(5h) and OSType=="Linux" and (VMUUID=~"625686a0-6d08-4810-aae9-a089e68d4911" or VMUUID=~"a0865662-086d-1048-aae9-a089e68d4911")
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification) by Computer, SourceComputerId, Product, ProductArch
| where UpdateState=~"Needed"
| summarize by Product, ProductArch, Classification
| summarize allUpdatesCount=count(), criticalUpdatesCount=countif(Classification has "Critical"), securityUpdatesCount=countif(Classification has "Security"), otherUpdatesCount=countif(Classification !has "Critical" and Classification !has "Security")
```

#### <a name="missing-updates-list"></a>Liste des mises à jour manquantes

```loganalytics
Update
| where TimeGenerated>ago(5h) and OSType=="Linux" and (VMUUID=~"625686a0-6d08-4810-aae9-a089e68d4911" or VMUUID=~"a0865662-086d-1048-aae9-a089e68d4911")
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, BulletinUrl, BulletinID) by Computer, SourceComputerId, Product, ProductArch
| where UpdateState=~"Needed"
| project-away UpdateState, TimeGenerated
| summarize computersCount=dcount(SourceComputerId, 2), ClassificationWeight=max(iff(Classification has "Critical", 4, iff(Classification has "Security", 2, 1))) by id=strcat(Product, "_", ProductArch), displayName=Product, productArch=ProductArch, classification=Classification, InformationId=BulletinID, InformationUrl=tostring(split(BulletinUrl, ";", 0)[0]), osType=1
| sort by ClassificationWeight desc, computersCount desc, displayName asc
| extend informationLink=(iff(isnotempty(InformationId) and isnotempty(InformationUrl), toobject(strcat('{ "uri": "', InformationUrl, '", "text": "', InformationId, '", "target": "blank" }')), toobject('')))
| project-away ClassificationWeight, InformationId, InformationUrl

```

### <a name="multi-vm-assessment-queries"></a>Requêtes d’évaluation sur plusieurs VM

#### <a name="computers-summary"></a>Récapitulatif des ordinateurs

```loganalytics
Heartbeat
| where TimeGenerated>ago(12h) and OSType=~"Windows" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
| where Solutions has "updates"
| distinct SourceComputerId
| join kind=leftouter
(
    Update
    | where TimeGenerated>ago(14h) and OSType!="Linux"
    | summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Approved, Optional, Classification) by SourceComputerId, UpdateID
    | distinct SourceComputerId, Classification, UpdateState, Approved, Optional
    | summarize WorstMissingUpdateSeverity=max(iff(UpdateState=~"Needed" and (Optional==false or Classification has "Critical" or Classification has "Security") and Approved!=false, iff(Classification has "Critical", 4, iff(Classification has "Security", 2, 1)), 0)) by SourceComputerId
)
on SourceComputerId
| extend WorstMissingUpdateSeverity=coalesce(WorstMissingUpdateSeverity, -1)
| summarize computersBySeverity=count() by WorstMissingUpdateSeverity
| union (Heartbeat
| where TimeGenerated>ago(12h) and OSType=="Linux" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
| where Solutions has "updates"
| distinct SourceComputerId
| join kind=leftouter
(
    Update
    | where TimeGenerated>ago(5h) and OSType=="Linux"
    | summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification) by SourceComputerId, Product, ProductArch
    | distinct SourceComputerId, Classification, UpdateState
    | summarize WorstMissingUpdateSeverity=max(iff(UpdateState=~"Needed", iff(Classification has "Critical", 4, iff(Classification has "Security", 2, 1)), 0)) by SourceComputerId
)
on SourceComputerId
| extend WorstMissingUpdateSeverity=coalesce(WorstMissingUpdateSeverity, -1)
| summarize computersBySeverity=count() by WorstMissingUpdateSeverity)
| summarize assessedComputersCount=sumif(computersBySeverity, WorstMissingUpdateSeverity>-1), notAssessedComputersCount=sumif(computersBySeverity, WorstMissingUpdateSeverity==-1), computersNeedCriticalUpdatesCount=sumif(computersBySeverity, WorstMissingUpdateSeverity==4), computersNeedSecurityUpdatesCount=sumif(computersBySeverity, WorstMissingUpdateSeverity==2), computersNeedOtherUpdatesCount=sumif(computersBySeverity, WorstMissingUpdateSeverity==1), upToDateComputersCount=sumif(computersBySeverity, WorstMissingUpdateSeverity==0)
| summarize assessedComputersCount=sum(assessedComputersCount), computersNeedCriticalUpdatesCount=sum(computersNeedCriticalUpdatesCount),  computersNeedSecurityUpdatesCount=sum(computersNeedSecurityUpdatesCount), computersNeedOtherUpdatesCount=sum(computersNeedOtherUpdatesCount), upToDateComputersCount=sum(upToDateComputersCount), notAssessedComputersCount=sum(notAssessedComputersCount)
| extend allComputersCount=assessedComputersCount+notAssessedComputersCount
```

#### <a name="missing-updates-summary"></a>Récapitulatif des mises à jour manquantes

```loganalytics
Update
| where TimeGenerated>ago(5h) and OSType=="Linux" and SourceComputerId in ((Heartbeat
| where TimeGenerated>ago(12h) and OSType=="Linux" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
| where Solutions has "updates"
| distinct SourceComputerId))
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification) by Computer, SourceComputerId, Product, ProductArch
| where UpdateState=~"Needed"
| summarize by Product, ProductArch, Classification
| union (Update
| where TimeGenerated>ago(14h) and OSType!="Linux" and (Optional==false or Classification has "Critical" or Classification has "Security") and SourceComputerId in ((Heartbeat
| where TimeGenerated>ago(12h) and OSType=~"Windows" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
| where Solutions has "updates"
| distinct SourceComputerId))
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, Approved) by Computer, SourceComputerId, UpdateID
| where UpdateState=~"Needed" and Approved!=false
| summarize by UpdateID, Classification )
| summarize allUpdatesCount=count(), criticalUpdatesCount=countif(Classification has "Critical"), securityUpdatesCount=countif(Classification has "Security"), otherUpdatesCount=countif(Classification !has "Critical" and Classification !has "Security")
```

#### <a name="computers-list"></a>Liste des ordinateurs

```loganalytics
Heartbeat
| where TimeGenerated>ago(12h) and OSType=="Linux" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions, Computer, ResourceId, ComputerEnvironment, VMUUID) by SourceComputerId
| where Solutions has "updates"
| extend vmuuId=VMUUID, azureResourceId=ResourceId, osType=1, environment=iff(ComputerEnvironment=~"Azure", 1, 2), scopedToUpdatesSolution=true, lastUpdateAgentSeenTime=""
| join kind=leftouter
(
    Update
    | where TimeGenerated>ago(5h) and OSType=="Linux" and SourceComputerId in ((Heartbeat
    | where TimeGenerated>ago(12h) and OSType=="Linux" and notempty(Computer)
    | summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
    | where Solutions has "updates"
    | distinct SourceComputerId))
    | summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, Product, Computer, ComputerEnvironment) by SourceComputerId, Product, ProductArch
    | summarize Computer=any(Computer), ComputerEnvironment=any(ComputerEnvironment), missingCriticalUpdatesCount=countif(Classification has "Critical" and UpdateState=~"Needed"), missingSecurityUpdatesCount=countif(Classification has "Security" and UpdateState=~"Needed"), missingOtherUpdatesCount=countif(Classification !has "Critical" and Classification !has "Security" and UpdateState=~"Needed"), lastAssessedTime=max(TimeGenerated), lastUpdateAgentSeenTime="" by SourceComputerId
    | extend compliance=iff(missingCriticalUpdatesCount > 0 or missingSecurityUpdatesCount > 0, 2, 1)
    | extend ComplianceOrder=iff(missingCriticalUpdatesCount > 0 or missingSecurityUpdatesCount > 0 or missingOtherUpdatesCount > 0, 1, 3)
)
on SourceComputerId
| project id=SourceComputerId, displayName=Computer, sourceComputerId=SourceComputerId, scopedToUpdatesSolution=true, missingCriticalUpdatesCount=coalesce(missingCriticalUpdatesCount, -1), missingSecurityUpdatesCount=coalesce(missingSecurityUpdatesCount, -1), missingOtherUpdatesCount=coalesce(missingOtherUpdatesCount, -1), compliance=coalesce(compliance, 4), lastAssessedTime, lastUpdateAgentSeenTime, osType=1, environment=iff(ComputerEnvironment=~"Azure", 1, 2), ComplianceOrder=coalesce(ComplianceOrder, 2)
| union(Heartbeat
| where TimeGenerated>ago(12h) and OSType=~"Windows" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions, Computer, ResourceId, ComputerEnvironment, VMUUID) by SourceComputerId
| where Solutions has "updates"
| extend vmuuId=VMUUID, azureResourceId=ResourceId, osType=2, environment=iff(ComputerEnvironment=~"Azure", 1, 2), scopedToUpdatesSolution=true, lastUpdateAgentSeenTime=""
| join kind=leftouter
(
    Update
    | where TimeGenerated>ago(14h) and OSType!="Linux" and SourceComputerId in ((Heartbeat
    | where TimeGenerated>ago(12h) and OSType=~"Windows" and notempty(Computer)
    | summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
    | where Solutions has "updates"
    | distinct SourceComputerId))
    | summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, Title, Optional, Approved, Computer, ComputerEnvironment) by Computer, SourceComputerId, UpdateID
    | summarize Computer=any(Computer), ComputerEnvironment=any(ComputerEnvironment), missingCriticalUpdatesCount=countif(Classification has "Critical" and UpdateState=~"Needed" and Approved!=false), missingSecurityUpdatesCount=countif(Classification has "Security" and UpdateState=~"Needed" and Approved!=false), missingOtherUpdatesCount=countif(Classification !has "Critical" and Classification !has "Security" and UpdateState=~"Needed" and Optional==false and Approved!=false), lastAssessedTime=max(TimeGenerated), lastUpdateAgentSeenTime="" by SourceComputerId
    | extend compliance=iff(missingCriticalUpdatesCount > 0 or missingSecurityUpdatesCount > 0, 2, 1)
    | extend ComplianceOrder=iff(missingCriticalUpdatesCount > 0 or missingSecurityUpdatesCount > 0 or missingOtherUpdatesCount > 0, 1, 3)
)
on SourceComputerId
| project id=SourceComputerId, displayName=Computer, sourceComputerId=SourceComputerId, scopedToUpdatesSolution=true, missingCriticalUpdatesCount=coalesce(missingCriticalUpdatesCount, -1), missingSecurityUpdatesCount=coalesce(missingSecurityUpdatesCount, -1), missingOtherUpdatesCount=coalesce(missingOtherUpdatesCount, -1), compliance=coalesce(compliance, 4), lastAssessedTime, lastUpdateAgentSeenTime, osType=2, environment=iff(ComputerEnvironment=~"Azure", 1, 2), ComplianceOrder=coalesce(ComplianceOrder, 2) )
| order by ComplianceOrder asc, missingCriticalUpdatesCount desc, missingSecurityUpdatesCount desc, missingOtherUpdatesCount desc, displayName asc
| project-away ComplianceOrder
```

#### <a name="missing-updates-list"></a>Liste des mises à jour manquantes

```loganalytics
Update
| where TimeGenerated>ago(5h) and OSType=="Linux" and SourceComputerId in ((Heartbeat
| where TimeGenerated>ago(12h) and OSType=="Linux" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
| where Solutions has "updates"
| distinct SourceComputerId))
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, BulletinUrl, BulletinID) by SourceComputerId, Product, ProductArch
| where UpdateState=~"Needed"
| project-away UpdateState, TimeGenerated
| summarize computersCount=dcount(SourceComputerId, 2), ClassificationWeight=max(iff(Classification has "Critical", 4, iff(Classification has "Security", 2, 1))) by id=strcat(Product, "_", ProductArch), displayName=Product, productArch=ProductArch, classification=Classification, InformationId=BulletinID, InformationUrl=tostring(split(BulletinUrl, ";", 0)[0]), osType=1
| union(Update
| where TimeGenerated>ago(14h) and OSType!="Linux" and (Optional==false or Classification has "Critical" or Classification has "Security") and SourceComputerId in ((Heartbeat
| where TimeGenerated>ago(12h) and OSType=~"Windows" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
| where Solutions has "updates"
| distinct SourceComputerId))
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, Title, KBID, PublishedDate, Approved) by Computer, SourceComputerId, UpdateID
| where UpdateState=~"Needed" and Approved!=false
| project-away UpdateState, Approved, TimeGenerated
| summarize computersCount=dcount(SourceComputerId, 2), displayName=any(Title), publishedDate=min(PublishedDate), ClassificationWeight=max(iff(Classification has "Critical", 4, iff(Classification has "Security", 2, 1))) by id=strcat(UpdateID, "_", KBID), classification=Classification, InformationId=strcat("KB", KBID), InformationUrl=iff(isnotempty(KBID), strcat("https://support.microsoft.com/kb/", KBID), ""), osType=2)
| sort by ClassificationWeight desc, computersCount desc, displayName asc
| extend informationLink=(iff(isnotempty(InformationId) and isnotempty(InformationUrl), toobject(strcat('{ "uri": "', InformationUrl, '", "text": "', InformationId, '", "target": "blank" }')), toobject('')))
| project-away ClassificationWeight, InformationId, InformationUrl
```

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur les journaux Azure Monitor, consultez [Journaux Azure Monitor](../../azure-monitor/log-query/log-query-overview.md).
* Pour obtenir de l’aide sur les alertes, consultez [Configurer des alertes](configure-alerts.md).

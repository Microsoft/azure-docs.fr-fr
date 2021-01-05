---
title: Solution Azure SQL Analytics dans Azure Monitor | Microsoft Docs
description: Vous pouvez utiliser la solution d’analytique du réseau Azure dans Azure Monitor pour examiner les journaux d’activité de passerelle d’application et de groupes de sécurité réseau Azure.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 06/21/2018
ms.openlocfilehash: 19370eee9d9fa524de9beeaa85a15521580bd8e6
ms.sourcegitcommit: 17e9cb8d05edaac9addcd6e0f2c230f71573422c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/21/2020
ms.locfileid: "97707688"
---
# <a name="azure-networking-monitoring-solutions-in-azure-monitor"></a>Solutions de supervision réseau Azure dans Azure Monitor

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Azure Monitor propose les solutions suivantes pour la supervision de vos réseaux :
* Analyseur de performances réseau (NPM) pour
    * Surveiller l’intégrité de votre réseau
* Azure Application Gateway Analytics à passer en revue
    * Journaux d’activité Azure Application Gateway
    * Mesures Azure Application Gateway
* Solutions de surveillance et d’audit de l’activité du réseau sur votre réseau Cloud
    * [Analyse du trafic](../../networking/network-monitoring-overview.md#traffic-analytics) 
    * Azure Network Security Group Analytics

## <a name="network-performance-monitor-npm"></a>Analyseur de performances réseau (NPM)

La solution de gestion [Analyseur de performances réseau](../../networking/network-monitoring-overview.md) est une solution de supervision du réseau, qui contrôle l’intégrité, la disponibilité et l’accessibilité des réseaux.  Elle est utilisée pour contrôler la connectivité entre :

* le cloud public et le site local ;
* les centres de données et les emplacements des utilisateurs (filiales) ;
* les sous-réseaux hébergeant différents niveaux d’une application multiniveau.

Pour plus d’informations, consultez l’[Analyseur de performances réseau](../../networking/network-monitoring-overview.md).

## <a name="network-security-group-analytics"></a>Analyse du groupe de sécurité réseau

1. Ajoutez la solution de gestion dans Azure Monitor, et
2. Activez les diagnostics pour les orienter vers un espace de travail Log Analytics dans Azure Monitor. Il n’est pas nécessaire d’écrire les journaux d’activité dans le stockage Blob Azure.

Si les journaux de diagnostic ne sont pas activés, les panneaux du tableau de bord pour cette ressource sont vides et affichent un message d’erreur.

## <a name="azure-application-gateway-analytics"></a>Azure Application Gateway Analytics

1. Activez les diagnostics pour les orienter vers un espace de travail Log Analytics dans Azure Monitor.
2. Utilisez le résumé détaillé de votre ressource à l’aide du modèle de classeur pour Application Gateway.

Si les journaux de diagnostic ne sont pas activés pour Application Gateway, seules les données de métriques par défaut sont remplies dans le classeur.


> [!NOTE]
> En janvier 2017, la méthode prise en charge pour envoyer des journaux d’activité dans un espace de travail Log Analytics à partir d’Application Gateways et de Network Security Groups a été modifiée. Si vous voyez la solution **Azure Networking Analytics (déconseillée)** , consultez [Migration à partir de l’ancienne solution Azure Networking](#migrating-from-the-old-networking-analytics-solution) pour connaître les étapes à suivre.
>
>

## <a name="review-azure-networking-data-collection-details"></a>Consulter les détails de la collecte de données réseaux Azure
Les solutions de gestion de l’analytique Azure Application Gateway et l’analytique Network Security Group collectent des journaux de diagnostic directement à partir d’Azure Application Gateways et Network Security Groups. Il n’est pas nécessaire d’écrire les journaux d’activité dans le stockage Blob Azure, et aucun agent n’est requis pour la collecte de données.

Le tableau suivant présente les méthodes de collecte des données et d’autres informations sur le mode de collecte des données pour l’analytique Azure Application Gateway et l’analytique Network Security Group.

| Plateforme | Agent direct | Agent Systems Center Operations Manager | Azure | Operations Manager requis ? | Données de l’agent Operations Manager envoyées via un groupe d’administration | Fréquence de collecte |
| --- | --- | --- | --- | --- | --- | --- |
| Azure |  |  |&#8226; |  |  |si connecté |


### <a name="enable-azure-application-gateway-diagnostics-in-the-portal"></a>Activer les diagnostics Azure Application Gateway dans le portail

1. Dans le Portail Azure, accédez à la ressource Application Gateway à surveiller.
2. Sélectionnez *Paramètres de diagnostic* pour ouvrir la page suivante.

   ![Capture d’écran de la configuration des paramètres de diagnostic pour la ressource Application Gateway.](media/azure-networking-analytics/diagnostic-settings-1.png)

   [ ![Capture d’écran de la page de configuration des paramètres de Diagnostics.](media/azure-networking-analytics/diagnostic-settings-2.png)](media/azure-networking-analytics/application-gateway-diagnostics-2.png#lightbox)

5. Cochez la case à côté de l’option *Envoyer à Log Analytics*.
6. Sélectionnez un espace de travail Log Analytics existant ou créez-en un.
7. Cochez la case sous **Journal** pour chacun des types de journaux à collecter.
8. Cliquez sur *Enregistrer* pour activer la journalisation des diagnostics dans Azure Monitor.

#### <a name="enable-azure-network-diagnostics-using-powershell"></a>Activez les diagnostics de réseau Azure avec PowerShell

Le script PowerShell suivant fournit un exemple montrant comment activer la journalisation des ressources pour les passerelles d’application.

```powershell
$workspaceId = "/subscriptions/d2e37fee-1234-40b2-5678-0b2199de3b50/resourcegroups/oi-default-east-us/providers/microsoft.operationalinsights/workspaces/rollingbaskets"

$gateway = Get-AzApplicationGateway -Name 'ContosoGateway'

Set-AzDiagnosticSetting -ResourceId $gateway.ResourceId  -WorkspaceId $workspaceId -Enabled $true
```

#### <a name="accessing-azure-application-gateway-analytics-via-azure-monitor-network-insights"></a>Accès à l’analytique Azure Application Gateway via les insights Azure Monitor Network

Application Insights est accessible via l’onglet Insights de votre ressource Application Gateway.

![Capture d’écran des insights Application Gateway ](media/azure-networking-analytics/azure-appgw-insights.png
)

L’onglet « Afficher les métriques détaillées » ouvre le classeur pré-rempli qui résume les données de votre passerelle applicative.

[ ![Capture d’écran du classeur Application Gateway](media/azure-networking-analytics/azure-appgw-workbook.png)](media/azure-networking-analytics/application-gateway-workbook.png#lightbox)

### <a name="new-capabilities-with-azure-monitor-network-insights-workbook"></a>Nouvelles capacités avec le classeur Insights réseau Azure Monitor

> [!NOTE]
> Aucun coût supplémentaire n’est associé au classeur Insights Azure Monitor. L’espace de travail Log Analytics continuera à être facturé en fonction de l’utilisation.

Le classeur Insights réseau vous permet de tirer profit des dernières capacités d’Azure Monitor et de Log Analytics, notamment :

* Console centralisée pour la surveillance et la résolution des problèmes avec des données de [métrique](../insights/network-insights-overview.md#resource-health-and-metrics) et de journal.

* Canevas flexible permettant de créer des [visualisations](../platform/workbooks-overview.md#visualizations) riches et personnalisées.

* Possibilité de consommer et de [partager des modèles de classeur](../platform/workbooks-overview.md#workbooks-versus-workbook-templates) avec une communauté plus vaste.

Pour plus d’informations sur les capacités de la nouvelle solution de classeurs, consultez [la page de présentation des classeurs](../platform/workbooks-overview.md).

## <a name="migrating-from-azure-gateway-analytics-solution-to-azure-monitor-workbooks"></a>Migration de la solution d’analyse Azure Gateway vers des classeurs Azure Monitor

> [!NOTE]
> Le classeur Insights réseau Azure Monitor est la solution recommandée pour accéder à l’analytique des métriques et des journaux pour vos ressources Application Gateway.

1. Vérifiez que [les paramètres de diagnostic sont activés](#enable-azure-application-gateway-diagnostics-in-the-portal) pour stocker les journaux dans un espace de travail Log Analytics. S’il est déjà configuré, le classeur Insights réseau Azure Monitor peut consommer des données à partir du même emplacement et aucune autre modification n’est nécessaire.

> [!NOTE]
> Toutes les données passées sont déjà disponibles dans le classeur à partir du moment où les paramètres de diagnostic ont été activés à l’origine. Aucun transfert de données n’est nécessaire.

2. Accédez au [classeur Insights par défaut](#accessing-azure-application-gateway-analytics-via-azure-monitor-network-insights) pour votre ressource Application Gateway. Tous les insights existants pris en charge par la solution d’analyse Application Gateway seront déjà présents dans le classeur. Vous pouvez l’étendre en ajoutant des [visualisations](../platform/workbooks-overview.md#visualizations) personnalisées basées sur les données de journal et de métrique.

3. Une fois que vous êtes en mesure de voir tous vos insights de métrique et de journal, pour nettoyer la solution d’analyse Azure Gateway à partir de votre espace de travail, vous pouvez supprimer la solution sur la page des ressources de la solution.

[ ![Capture d’écran de l’option de suppression pour la solution d’analyse Azure Application Gateway.](media/azure-networking-analytics/azure-appgw-analytics-delete.png)](media/azure-networking-analytics/application-gateway-analytics-delete.png#lightbox)

## <a name="azure-network-security-group-analytics-solution-in-azure-monitor"></a>Solution d’analytique Azure Network Security Group dans Azure Monitor

![Symbole d’Azure Network Security Group Analytics](media/azure-networking-analytics/azure-analytics-symbol.png)

> [!NOTE]
> La solution d’analytique Groupe de sécurité réseau passe au support Communauté, puisqu’elle a été remplacée par [Traffic Analytics](../../network-watcher/traffic-analytics.md).
> - Cette solution est désormais disponible dans les [Modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/oms-azurensg-solution/), et ne sera bientôt plus disponible dans la Place de marché Azure.
> - Pour les clients existants qui ont déjà ajouté la solution à leur espace de travail, elle continuera de fonctionner comme avant.
> - Microsoft continuera de prendre en charge l’envoi des journaux de ressources NSG vers votre espace de travail à l’aide des paramètres de diagnostic.

Les fichiers journaux d’activité suivants sont pris en charge pour les groupes de sécurité réseau :

* NetworkSecurityGroupEvent
* NetworkSecurityGroupRuleCounter

### <a name="install-and-configure-the-solution"></a>Installer et configurer la solution
Pour installer et configurer la solution d’analytique du réseau Azure, suivez les instructions suivantes :

1. Activez la solution d’analytique Azure Network Security Group en procédant de la manière décrite dans [Ajouter des solutions Azure Monitor à partir de la Galerie Solutions](./solutions.md).
2. Activez la journalisation des diagnostics pour les ressources [Network Security Group](../../virtual-network/virtual-network-nsg-manage-log.md) que vous voulez surveiller.

### <a name="enable-azure-network-security-group-diagnostics-in-the-portal"></a>Activer les diagnostics Azure Network Security Group dans le portail

1. Dans le Portail Azure, accédez à la ressource Network Security Group à surveiller.
2. Sélectionnez *Journaux de diagnostic* pour ouvrir la page suivante.

   ![Capture d’écran de la page Journaux de diagnostics pour une ressource de groupe de sécurité réseau montrant l’option Activer les diagnostics.](media/azure-networking-analytics/log-analytics-nsg-enable-diagnostics01.png)
3. Cliquez sur *Activer les diagnostics* pour ouvrir la page suivante.

   ![Capture d’écran de la page de configuration des paramètres de Diagnostics. L’État est défini sur Activé, l’option Envoyer à Log Analytics est activée et deux types de journaux sont sélectionnés.](media/azure-networking-analytics/log-analytics-nsg-enable-diagnostics02.png)
4. Pour activer les diagnostics, cliquez sur *Activé* sous *État*.
5. Cochez la case à côté de l’option *Envoyer à Log Analytics*.
6. Sélectionnez un espace de travail Log Analytics existant ou créez-en un.
7. Cochez la case sous **Journal** pour chacun des types de journaux à collecter.
8. Cliquez sur *Enregistrer* pour activer la journalisation des diagnostics dans Log Analytics.

### <a name="enable-azure-network-diagnostics-using-powershell"></a>Activez les diagnostics de réseau Azure avec PowerShell

Le script PowerShell suivant fournit un exemple montrant comment activer la journalisation des ressources pour les groupes de sécurité réseau
```powershell
$workspaceId = "/subscriptions/d2e37fee-1234-40b2-5678-0b2199de3b50/resourcegroups/oi-default-east-us/providers/microsoft.operationalinsights/workspaces/rollingbaskets"

$nsg = Get-AzNetworkSecurityGroup -Name 'ContosoNSG'

Set-AzDiagnosticSetting -ResourceId $nsg.ResourceId  -WorkspaceId $workspaceId -Enabled $true
```

### <a name="use-azure-network-security-group-analytics"></a>Utiliser l’analytique Azure Network Security Group
Une fois que vous avez cliqué sur la mosaïque **d’analytique Azure Network Security Group** dans la Vue d’ensemble, vous pouvez consulter les résumés de vos journaux d’activité et entrer dans les détails des catégories suivantes :

* Flux bloqués de groupe de sécurité réseau
  * Règles de groupe de sécurité réseau avec flux bloqués
  * Adresses MAC avec flux bloqués
* Flux autorisés de groupe de sécurité réseau
  * Règles de groupe de sécurité réseau avec flux autorisés
  * Adresses MAC avec flux autorisés

![Capture d’écran de vignettes contenant des données pour des flux bloqués de groupe de sécurité réseau, incluant des règles avec flux bloqués et des adresses MAC avec flux bloqués.](media/azure-networking-analytics/log-analytics-nsg01.png)

![Capture d’écran de vignettes contenant des données pour des flux autorisés de groupe de sécurité réseau, incluant des règles avec flux autorisés et des adresses MAC avec flux autorisés.](media/azure-networking-analytics/log-analytics-nsg02.png)

Sur le tableau de bord **d’analytique Azure Network Security Group**, consultez les informations du résumé dans l’un des panneaux, puis cliquez sur l’un d’entre eux pour afficher des informations détaillées sur la page de recherche dans les journaux.

Sur l’une des pages de recherche de journal, vous pouvez afficher les résultats par date, les résultats détaillés et votre historique de recherches de journaux. Vous pouvez également filtrer par facettes pour affiner les résultats.

## <a name="migrating-from-the-old-networking-analytics-solution"></a>Migration à partir de l’ancienne solution Azure Networking
En janvier 2017, la méthode prise en charge pour envoyer des journaux d’activité dans un espace de travail Log Analytics à partir d’Azure Application Gateways et Azure Network Security Groups a été modifiée. Ces modifications présentent les avantages suivants :
+ Les journaux d’activité sont écrits directement dans Azure Monitor sans avoir besoin d’utiliser un compte de stockage.
+ La durée de latence est plus courte entre le moment où les journaux d’activité sont générés et celui où ils deviennent disponibles dans Azure Monitor.
+ Le nombre d’étapes de configuration est réduit.
+ Tous les types de diagnostics Azure sont au même format.

Pour utiliser les solutions mises à jour :

1. [Configurez les diagnostics pour qu’ils soient directement envoyés à Azure Monitor à partir d’Azure Application Gateways](#enable-azure-application-gateway-diagnostics-in-the-portal).
2. [Configurez les diagnostics pour qu’ils soient directement envoyés à Azure Monitor à partir d’Azure Network Security Groups](#enable-azure-network-security-group-diagnostics-in-the-portal).
2. Activez la solution *d’analytique Azure Application Gateway* et *d’analytique Azure Network Security Group* en procédant de la manière décrite dans [Ajouter des solutions Azure Monitor à partir de la galerie de solutions](solutions.md).
3. Mettez à jour les requêtes, les tableaux de bord et les alertes enregistrés pour qu’ils utilisent le nouveau type de données.
   + Le type doit être AzureDiagnostics. Vous pouvez utiliser ResourceType pour filtrer les journaux d’activité réseaux Azure.

     | À la place de : | Utilisez : |
     | --- | --- |
     | NetworkApplicationgateways &#124; où OperationName=="ApplicationGatewayAccess" | AzureDiagnostics &#124; où ResourceType=="APPLICATIONGATEWAYS" et OperationName=="ApplicationGatewayAccess" |
     | NetworkApplicationgateways &#124; où OperationName=="ApplicationGatewayPerformance" | AzureDiagnostics &#124; où ResourceType=="APPLICATIONGATEWAYS" et OperationName=="ApplicationGatewayPerformance" |
     | NetworkSecuritygroups | AzureDiagnostics &#124; où ResourceType=="NETWORKSECURITYGROUPS" |

   + Pour tous les champs dont le suffixe est \_s, \_d ou \_g, remplacez le premier caractère du nom par une lettre minuscule.
   + Pour tous les champs dont le suffixe est \_o, les données sont réparties sur plusieurs champs, selon les noms de champs imbriqués.
4. Supprimez la solution *Azure Networking Analytics (déconseillée)* .
   + Avec PowerShell, utilisez `Set-AzureOperationalInsightsIntelligencePack -ResourceGroupName <resource group that the workspace is in> -WorkspaceName <name of the log analytics workspace> -IntelligencePackName "AzureNetwork" -Enabled $false`.

Les données collectées avant la modification ne seront pas visibles dans la nouvelle solution. Vous pouvez continuer à interroger ces données à l’aide de l’ancien type et des anciens noms de champs.

## <a name="troubleshooting"></a>Dépannage
[!INCLUDE [log-analytics-troubleshoot-azure-diagnostics](../../../includes/log-analytics-troubleshoot-azure-diagnostics.md)]

## <a name="next-steps"></a>Étapes suivantes
* Utilisez les [requêtes de journaux dans Azure Monitor](../log-query/log-query-overview.md) pour afficher des diagnostics détaillés Azure.


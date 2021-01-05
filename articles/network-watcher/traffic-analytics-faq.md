---
title: Forum aux questions pour l’analyse de trafic Azure | Microsoft Docs
description: Découvrez les réponses aux questions les plus fréquemment posées sur l’analyse de trafic.
services: network-watcher
documentationcenter: na
author: damendo
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/08/2018
ms.author: damendo
ms.openlocfilehash: f27f2a6319bf2648546a62c2566d938db657738f
ms.sourcegitcommit: 84e3db454ad2bccf529dabba518558bd28e2a4e6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96518462"
---
# <a name="traffic-analytics-frequently-asked-questions"></a>Forum aux questions pour Traffic Analytics

Cet article regroupe les questions fréquemment posées sur l’analytique du trafic dans Azure Network Watcher.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="what-are-the-prerequisites-to-use-traffic-analytics"></a>Quels sont les prérequis pour utiliser l’analytique du trafic ?

Traffic Analytics nécessite les prérequis suivants :

- Un abonnement actif à Network Watcher.
- Des journaux de flux NSG (groupe de sécurité réseau) activés pour les groupes de sécurité réseau à surveiller.
- Un compte de stockage Azure pour stocker les journaux de flux bruts.
- Un espace de travail Azure Log Analytics, avec accès en lecture et écriture.

Votre compte doit répondre à l’une des descriptions suivantes pour activer l’analytique du trafic :

- Votre compte doit avoir l’un des rôles Azure suivants au niveau de l’étendue de l’abonnement : propriétaire, contributeur, lecteur ou contributeur réseau.
- Si votre compte n’a pas l’un des rôles précédemment répertoriés, il doit avoir un rôle personnalisé auquel sont affectées les actions suivantes au niveau de l’abonnement.
            
    - Microsoft.Network/applicationGateways/read
    - Microsoft.Network/connections/read
    - Microsoft.Network/loadBalancers/read 
    - Microsoft.Network/localNetworkGateways/read 
    - Microsoft.Network/networkInterfaces/read 
    - Microsoft.Network/networkSecurityGroups/read 
    - Microsoft.Network/publicIPAddresses/read
    - Microsoft.Network/routeTables/read
    - Microsoft.Network/virtualNetworkGateways/read 
    - Microsoft.Network/virtualNetworks/read
        
Pour vérifier les rôles attribués à un utilisateur pour un abonnement :

1. Connectez-vous à Azure avec **Login-AzAccount**. 

2. Sélectionnez l’abonnement requis avec **Select-AzSubscription**. 

3. Pour répertorier tous les rôles attribués à un utilisateur donné, utilisez **Get-AzRoleAssignment -SignInName [e-mail utilisateur] -IncludeClassicAdministrators**. 

Si vous ne voyez aucune sortie, contactez l’administrateur de l’abonnement pour qu’il vous donne les droits d’accès nécessaires pour exécuter les commandes. Pour plus d'informations, consultez [Ajouter ou supprimer des attributions de rôle Azure à l'aide d'Azure PowerShell](../role-based-access-control/role-assignments-powershell.md).


## <a name="in-which-azure-regions-is-traffic-analytics-available"></a>Dans quelles régions Traffic Analytics est disponible ?

Vous pouvez utiliser l’analytique du trafic pour les régions prises en charge suivantes :
- Centre du Canada
- Centre-USA Ouest
- USA Est
- USA Est 2
- Centre-Nord des États-Unis
- États-Unis - partie centrale méridionale
- USA Centre
- USA Ouest
- USA Ouest 2
- France Centre
- Europe Ouest
- Europe Nord
- Brésil Sud
- Ouest du Royaume-Uni
- Sud du Royaume-Uni
- Australie Est
- Sud-Australie Est 
- Asie Est
- Asie Sud-Est
- Centre de la Corée
- Inde centrale
- Inde Sud
- Japon Est
- OuJapon Est
- Gouvernement américain - Virginie
- Chine orientale 2

L’espace de travail Log Analytics doit exister dans les régions suivantes :
- Centre du Canada
- Centre-USA Ouest
- USA Est
- USA Est 2
- Centre-Nord des États-Unis
- États-Unis - partie centrale méridionale
- USA Centre
- USA Ouest
- USA Ouest 2
- France Centre
- Europe Ouest
- Europe Nord
- Ouest du Royaume-Uni
- Sud du Royaume-Uni
- Australie Est
- Sud-Australie Est
- Asie Est
- Asie Sud-Est 
- Centre de la Corée
- Inde centrale
- Japon Est
- Gouvernement américain - Virginie
- Chine orientale 2

## <a name="can-the-nsgs-i-enable-flow-logs-for-be-in-different-regions-than-my-workspace"></a>Puis-je activer des journaux de flux pour des NSG qui se trouvent dans des régions différentes de mon espace de travail ?

Oui, ces NSG peuvent être dans des régions différentes de celles de votre espace de travail Log Analytics.

## <a name="can-multiple-nsgs-be-configured-within-a-single-workspace"></a>Est-il possible de configurer plusieurs groupes de sécurité réseau dans un seul espace de travail ?

Oui.

## <a name="can-i-use-an-existing-workspace"></a>Puis-je utiliser un espace de travail existant ?

Oui. Si vous sélectionnez un espace de travail existant, vérifiez qu’il a été migré vers le nouveau langage de requête. Si vous ne souhaitez pas mettre à niveau l’espace de travail, vous devez en créer un autre. Pour plus d’informations sur le nouveau langage de requête, consultez [Mise à niveau des journaux d’activité Azure Monitor vers la nouvelle recherche dans les journaux](../azure-monitor/log-query/log-query-overview.md).

## <a name="can-my-azure-storage-account-be-in-one-subscription-and-my-log-analytics-workspace-be-in-a-different-subscription"></a>Mon compte Stockage Azure peut-il être dans un abonnement spécifique et mon espace de travail Log Analytics dans un autre abonnement ?

Oui, votre compte Stockage Azure peut être dans un abonnement spécifique et votre espace de travail Log Analytics dans un autre abonnement.

## <a name="can-i-store-raw-logs-in-a-different-subscription"></a>Puis-je stocker les journaux d’activité bruts dans un autre abonnement ?

Oui. Vous pouvez configurer les journaux de flux NSG pour qu’ils soient envoyés à un compte de stockage situé dans un autre abonnement, à condition que vous disposiez des privilèges appropriés et que le compte de stockage se trouve dans la même région que le NSG. Le NSG et le compte de stockage de destination doivent également partager le même locataire Azure Active Directory.

## <a name="what-if-i-cant-configure-an-nsg-for-traffic-analytics-due-to-a-not-found-error"></a>Que faire si je ne peux pas configurer un NSG pour l’analytique du trafic en raison d’une erreur « Introuvable » ?

Sélectionnez une région prise en charge. Si vous sélectionnez une région non prise en charge, vous recevez une erreur « Introuvable ». Les régions prises en charge sont répertoriées plus haut dans cet article.

## <a name="what-if-i-am-getting-the-status-failed-to-load-under-the-nsg-flow-logs-page"></a>Que faire si j’obtiens l’état « Échec de chargement » sous la page de journaux de flux NSG ?

Pour que la journalisation du flux fonctionne correctement, le fournisseur Microsoft.Insights doit être inscrit. Si vous ne savez pas si le fournisseur Microsoft.Insights est inscrit ou non pour votre abonnement, remplacez *xxxxx-xxxxx-xxxxxx-xxxx* dans la commande suivante, puis exécutez les commandes ci-dessous à partir de PowerShell :

```powershell-interactive
**Select-AzSubscription** -SubscriptionId xxxxx-xxxxx-xxxxxx-xxxx
**Register-AzResourceProvider** -ProviderNamespace Microsoft.Insights
```

## <a name="i-have-configured-the-solution-why-am-i-not-seeing-anything-on-the-dashboard"></a>J’ai configuré la solution. Pourquoi ne vois-je rien sur le tableau de bord ?

L’affichage du tableau de bord peut prendre jusqu’à 30 minutes la première fois. La solution doit d’abord agréger suffisamment de données pour en dériver des insights utiles. Ensuite, elle génère des rapports. 

## <a name="what-if-i-get-this-message-we-could-not-find-any-data-in-this-workspace-for-selected-time-interval-try-changing-the-time-interval-or-select-a-different-workspace"></a>Que faire si je reçois le message « Impossible de trouver des données dans cet espace de travail pour l’intervalle de temps sélectionné. Essayez de changer l’intervalle de temps ou de sélectionner un autre espace de travail. » ?

Tentez les opérations suivantes :
- Changez l’intervalle de temps dans la barre supérieure.
- Sélectionnez un autre espace de travail Log Analytics dans la barre supérieure.
- Essayez d’accéder à l’analytique du trafic après 30 minutes, si elle a été récemment activée.
    
Si les problèmes persistent, expliquez votre problème dans le [forum des utilisateurs](https://feedback.azure.com/forums/217313-networking?category_id=195844).

## <a name="what-if-i-get-this-message-analyzing-your-nsg-flow-logs-for-the-first-time-this-process-may-take-20-30-minutes-to-complete-check-back-after-some-time-2-if-the-above-step-doesnt-work-and-your-workspace-is-under-the-free-sku-then-check-your-workspace-usage-here-to-validate-over-quota-else-refer-to-faqs-for-further-information"></a>Que faire si je reçois le message « Nous analysons peut-être vos journaux de flux NSG pour la première fois. Ce processus peut prendre de 20 à 30 minutes. Vérifiez ultérieurement. 2) Si l’étape ci-dessus ne fonctionne pas et que votre espace de travail se trouve sous la référence SKU libre, vérifiez l’utilisation de votre espace de travail pour valider le quota. Sinon, reportez-vous à la FAQ pour plus d’informations. » ?

Ce message peut s’afficher pour les raisons suivantes :
- Traffic Analytics a été récemment activé et peut ne pas avoir agrégé suffisamment de données pour qu’en ressortent des insights significatifs.
- Vous utilisez la version gratuite de l’espace de travail Log Analytics, et les limites de quota ont été dépassées. Vous devrez peut-être utiliser un espace de travail d’une capacité supérieure.
    
Si les problèmes persistent, expliquez votre problème dans le [forum des utilisateurs](https://feedback.azure.com/forums/217313-networking?category_id=195844).
    
## <a name="what-if-i-get-this-message-looks-like-we-have-resources-data-topology-and-no-flows-information-meanwhile-click-here-to-see-resources-data-and-refer-to-faqs-for-further-information"></a>Que faire si je reçois le message « Il semble qu’il existe des données de ressources (topologie), mais aucune information de flux. Cliquez ici pour afficher les données de ressources et reportez-vous à la FAQ pour plus d’informations. » ?

Vous voyez les informations de ressources sur le tableau de bord. Toutefois, aucune statistique de flux n’est présente. Les données sont peut-être manquantes en l’absence de flux de communication entre les ressources. Attendez 60 minutes et revérifiez l’état. Si le problème persiste et que vous êtes sûr que des flux de communication existent entre les ressources, expliquez votre problème dans le [forum des utilisateurs](https://feedback.azure.com/forums/217313-networking?category_id=195844).

## <a name="can-i-configure-traffic-analytics-using-powershell-or-an-azure-resource-manager-template-or-client"></a>Puis-je configurer l’analytique du trafic à l’aide de PowerShell ou d’un modèle/client Azure Resource Manager ?

Vous pouvez configurer l’analytique du trafic à l’aide de Windows PowerShell (versions 6.2.1 et ultérieures). Pour configurer la journalisation de flux et l’analytique du trafic pour un NSG donné à l’aide de l’applet de commande Set, consultez [Set-AzNetworkWatcherConfigFlowLog](/powershell/module/az.network/set-aznetworkwatcherconfigflowlog). Pour obtenir l’état de la journalisation de flux et de l’analytique du trafic pour un NSG donné, consultez [Get-AzNetworkWatcherFlowLogStatus](/powershell/module/az.network/get-aznetworkwatcherflowlogstatus).

Vous ne pouvez pas actuellement utiliser un modèle Azure Resource Manager pour configurer l’analytique du trafic.

Pour configurer l’analytique du trafic à l’aide d’un client Azure Resource Manager, consultez les exemples suivants.

**Exemple d’applet de commande Set :**
```
#Requestbody parameters
$TAtargetUri ="/subscriptions/<NSG subscription id>/resourceGroups/<NSG resource group name>/providers/Microsoft.Network/networkSecurityGroups/<name of NSG>"
$TAstorageId = "/subscriptions/<storage subscription id>/resourcegroups/<storage resource group name> /providers/microsoft.storage/storageaccounts/<storage account name>"
$networkWatcherResourceGroupName = "<network watcher resource group name>"
$networkWatcherName = "<network watcher name>"

$requestBody = 
@"
{
    'targetResourceId': '${TAtargetUri}',
    'properties': 
    {
        'storageId': '${TAstorageId}',
        'enabled': '<true to enable flow log or false to disable flow log>',
        'retentionPolicy' : 
        {
            days: <enter number of days like to retail flow logs in storage account>,
            enabled: <true to enable retention or false to disable retention>
        }
    },
    'flowAnalyticsConfiguration':
    {
                'networkWatcherFlowAnalyticsConfiguration':
      {
        'enabled':,<true to enable traffic analytics or false to disable traffic analytics>
        'workspaceId':'bbbbbbbb-bbbb-bbbb-bbbb-bbbbbbbbbbbb',
        'workspaceRegion':'<workspace region>',
        'workspaceResourceId':'/subscriptions/<workspace subscription id>/resourcegroups/<workspace resource group name>/providers/microsoft.operationalinsights/workspaces/<workspace name>'
        
      }

    }
}
"@
$apiversion = "2016-09-01"

armclient login
armclient post "https://management.azure.com/subscriptions/<NSG subscription id>/resourceGroups/<network watcher resource group name>/providers/Microsoft.Network/networkWatchers/<network watcher name>/configureFlowlog?api-version=${apiversion}" $requestBody
```
**Exemple d’applet de commande Get :**
```
#Requestbody parameters
$TAtargetUri ="/subscriptions/<NSG subscription id>/resourceGroups/<NSG resource group name>/providers/Microsoft.Network/networkSecurityGroups/<NSG name>"


$requestBody = 
@"
{
    'targetResourceId': '${TAtargetUri}'
}
“@


armclient login
armclient post "https://management.azure.com/subscriptions/<NSG subscription id>/resourceGroups/<network watcher resource group name>/providers/Microsoft.Network/networkWatchers/<network watcher name>/queryFlowLogStatus?api-version=${apiversion}" $requestBody
```


## <a name="how-is-traffic-analytics-priced"></a>Quel est le prix de Traffic Analytics ?

Traffic Analytics est mesuré. Les mesures sont basées sur le traitement des données de journal de flux par le service et sur le stockage des journaux d’activité améliorés résultants dans un espace de travail Log Analytics. 

Par exemple, conformément au [plan tarifaire](https://azure.microsoft.com/pricing/details/network-watcher/), concernant la région USA Centre-Ouest, si des données de journal de flux stockées dans un compte de stockage traité par Traffic Analytics ont un volume de 10 Go et que les journaux d’activité améliorés ingérés dans l’espace de travail Log Analytics ont un volume de 1 Go, les charges applicables sont les suivantes : 10 x 2.3$ + 1 x 2.76$ = 25.76$

## <a name="how-frequently-does-traffic-analytics-process-data"></a>Quelle est la fréquence de traitement des données de Traffic Analytics ?

Reportez-vous à la [section sur l’agrégation des données](./traffic-analytics-schema.md#data-aggregation) du schéma de Traffic Analytics et du document d’agrégation des données

## <a name="how-does-traffic-analytics-decide-that-an-ip-is-malicious"></a>Comment Traffic Analytics décide-t-il qu’une adresse IP est malveillante ? 

Traffic Analytics s’appuie sur les systèmes internes de Microsoft pour les renseignements sur les menaces afin de calculer si une IP est malveillante ou pas. Ces systèmes tirent parti de diverses sources de télémétrie comme les produits et services Microsoft, la DCU (Digital Crimes Unit, soit « Unité de lutte contre la criminalité numérique ») de Microsoft, le Centre de réponse aux problèmes de sécurité Microsoft, ainsi que des flux externes, le tout pour développer des analyses poussées. Certaines de ces données sont internes à Microsoft. Si une adresse IP connue est signalée comme étant malveillante, veuillez renseigner un ticket d’assistance pour connaître les détails de ce signalement.

## <a name="how-can-i-set-alerts-on-traffic-analytics-data"></a>Comment puis-je définir des alertes sur les données Traffic Analytics ?

Traffic Analytics n’a pas de prise en charge intégrée des alertes. Cependant, comme les données de Traffic Analytics sont stockées dans Log Analytics, vous pouvez écrire des requêtes personnalisées et définir des alertes à leur sujet. Procédure :
- Vous pouvez utiliser le lien court vers Log Analytics dans Traffic Analytics. 
- Utilisez le [schéma documenté ici](traffic-analytics-schema.md) pour écrire vos requêtes 
- Cliquez sur + Nouvelle règle d’alerte pour créer une alerte
- Reportez-vous à la [documentation des alertes de journal](../azure-monitor/platform/alerts-log.md) pour créer l’alerte

## <a name="how-do-i-check-which-vms-are-receiving-most-on-premises-traffic"></a>Comment déterminer quelles machines virtuelles reçoivent le plus de trafic local ?

```
AzureNetworkAnalytics_CL
| where SubType_s == "FlowLog" and FlowType_s == "S2S" 
| where <Scoping condition>
| mvexpand vm = pack_array(VM1_s, VM2_s) to typeof(string)
| where isnotempty(vm) 
| extend traffic = AllowedInFlows_d + DeniedInFlows_d + AllowedOutFlows_d + DeniedOutFlows_d // For bytes use: | extend traffic = InboundBytes_d + OutboundBytes_d 
| make-series TotalTraffic = sum(traffic) default = 0 on FlowStartTime_t from datetime(<time>) to datetime(<time>) step 1m by vm
| render timechart
```

  Pour les adresses IP :

```
AzureNetworkAnalytics_CL
| where SubType_s == "FlowLog" and FlowType_s == "S2S" 
//| where <Scoping condition>
| mvexpand IP = pack_array(SrcIP_s, DestIP_s) to typeof(string)
| where isnotempty(IP) 
| extend traffic = AllowedInFlows_d + DeniedInFlows_d + AllowedOutFlows_d + DeniedOutFlows_d // For bytes use: | extend traffic = InboundBytes_d + OutboundBytes_d 
| make-series TotalTraffic = sum(traffic) default = 0 on FlowStartTime_t from datetime(<time>) to datetime(<time>) step 1m by IP
| render timechart
```

Pour l’heure, utilisez le format : aaaa-mm-jj 00:00:00

## <a name="how-do-i-check-standard-deviation-in-traffic-received-by-my-vms-from-on-premises-machines"></a>Comment vérifier l’écart type du trafic reçu par mes machines virtuelles à partir de machines locales ?

```
AzureNetworkAnalytics_CL
| where SubType_s == "FlowLog" and FlowType_s == "S2S" 
//| where <Scoping condition>
| mvexpand vm = pack_array(VM1_s, VM2_s) to typeof(string)
| where isnotempty(vm) 
| extend traffic = AllowedInFlows_d + DeniedInFlows_d + AllowedOutFlows_d + DeniedOutFlows_d // For bytes use: | extend traffic = InboundBytes_d + utboundBytes_d
| summarize deviation = stdev(traffic)  by vm
```

Pour les adresses IP :

```
AzureNetworkAnalytics_CL
| where SubType_s == "FlowLog" and FlowType_s == "S2S" 
//| where <Scoping condition>
| mvexpand IP = pack_array(SrcIP_s, DestIP_s) to typeof(string)
| where isnotempty(IP) 
| extend traffic = AllowedInFlows_d + DeniedInFlows_d + AllowedOutFlows_d + DeniedOutFlows_d // For bytes use: | extend traffic = InboundBytes_d + OutboundBytes_d
| summarize deviation = stdev(traffic)  by IP
```

## <a name="how-do-i-check-which-ports-are-reachable-or-blocked-between-ip-pairs-with-nsg-rules"></a>Comment vérifier quels ports sont accessibles (ou bloqués) entre les paires d’adresses IP avec les règles du groupe de sécurité réseau ?

```
AzureNetworkAnalytics_CL
| where SubType_s == "FlowLog" and TimeGenerated between (startTime .. endTime)
| extend sourceIPs = iif(isempty(SrcIP_s), split(SrcPublicIPs_s, " ") , pack_array(SrcIP_s)),
destIPs = iif(isempty(DestIP_s), split(DestPublicIPs_s," ") , pack_array(DestIP_s))
| mvexpand SourceIp = sourceIPs to typeof(string)
| mvexpand DestIp = destIPs to typeof(string)
| project SourceIp = tostring(split(SourceIp, "|")[0]), DestIp = tostring(split(DestIp, "|")[0]), NSGList_s, NSGRule_s, DestPort_d, L4Protocol_s, FlowStatus_s 
| summarize DestPorts= makeset(DestPort_d) by SourceIp, DestIp, NSGList_s, NSGRule_s, L4Protocol_s, FlowStatus_s
```

## <a name="how-can-i-navigate-by-using-the-keyboard-in-the-geo-map-view"></a>Comment faire pour naviguer dans la vue de la carte géographique à l’aide du clavier ?

La page de la carte géographique contient deux sections principales :
    
- **Bannière** : la bannière en haut de la carte géographique fournit des boutons pour sélectionner les filtres de distribution du trafic (Déploiement, Trafic en provenance de pays/régions, Malveillant, etc.). Quand vous sélectionnez un bouton, le filtre correspondant est appliqué sur la carte. Par exemple, si vous sélectionnez le bouton Actif, la carte met en surbrillance les centres de données actifs dans votre déploiement.
- **Carte** : Sous la bannière, la section de la carte montre la distribution du trafic entre les pays/régions et les centres de données Azure.
    
### <a name="keyboard-navigation-on-the-banner"></a>Navigation au clavier sur la bannière
    
- La sélection par défaut dans la bannière de la page de la carte géographique est le filtre « Contrôleurs de domaine Azure ».
- Pour passer à un autre filtre, utilisez la touche `Tab` ou `Right arrow`. Pour vous déplacer vers l’arrière, utilisez la touche `Shift+Tab` ou `Left arrow`. La navigation vers l’avant va de gauche à droite, puis de haut en bas.
- Appuyez sur la touche de direction `Enter` ou `Down` pour appliquer le filtre sélectionné. Selon la sélection de filtre et le déploiement, un ou plusieurs nœuds sous la section de la carte sont mis en surbrillance.
- Pour basculer entre la bannière et la carte, appuyez sur `Ctrl+F6`.
        
### <a name="keyboard-navigation-on-the-map"></a>Navigation au clavier sur la carte
    
- Une fois que vous avez sélectionné un filtre sur la bannière et appuyé sur `Ctrl+F6`, le focus passe à l’un des nœuds en surbrillance (**Centre de données Azure** ou **Pays/Région**) dans la vue cartographique.
- Pour passer à d’autres nœuds en surbrillance dans la carte, utilisez la touche `Tab` ou `Right arrow` pour vous déplacer vers l’avant. Utilisez la touche `Shift+Tab` ou `Left arrow` pour vous déplacer vers l’arrière.
- Pour sélectionner n’importe quel nœud en surbrillance sur la carte, utilisez la touche `Enter` ou `Down arrow`.
- Lorsque vous sélectionnez ces nœuds, le focus se déplace vers la **boîte à outils Informations** du nœud. Par défaut, le focus se déplace sur le bouton Fermer de la **boîte à outils Informations**. Pour vous déplacer davantage dans la vue de la **boîte**, utilisez les touches `Right arrow` et `Left arrow` pour vous déplacer vers l’avant et vers l’arrière, respectivement. Appuyer sur `Enter` a le même effet que cliquer sur le bouton actif dans la **boîte à outils Informations**.
- Quand vous appuyez sur `Tab` et que le focus est sur la **boîte à outils Informations**, le focus passe aux points de terminaison du même continent que le nœud sélectionné. Utilisez les touches `Right arrow` et `Left arrow` pour parcourir ces points de terminaison.
- Pour passer à d’autres clusters de continents/points de terminaison de flux, utilisez `Tab` pour vous déplacer vers l’avant et `Shift+Tab` pour vous déplacer vers l’arrière.
- Quand le focus est sur des **clusters de continents**, utilisez la touche de direction `Enter` ou `Down` pour mettre en surbrillance les points de terminaison dans le cluster de continents. Pour parcourir les points de terminaison et accéder au bouton Fermer de la boîte d’informations du cluster de continents, utilisez les touches `Right arrow` et `Left arrow` pour vous déplacer vers l’avant et vers l’arrière, respectivement. Sur n’importe quel point de terminaison, vous pouvez utiliser `Shift+L` pour basculer vers la ligne de connexion à partir du nœud sélectionné pour le point de terminaison. Vous pouvez réappuyer sur `Shift+L` pour passer au point de terminaison sélectionné.
        
### <a name="keyboard-navigation-at-any-stage"></a>Navigation au clavier à tout moment
    
- `Esc` réduit la sélection développée.
- La touche `Up-arrow` effectue la même action que `Esc`. La touche `Down arrow` effectue la même action que `Enter`.
- Utilisez `Shift+Plus` pour effectuer un zoom avant et `Shift+Minus` pour effectuer un zoom arrière.

## <a name="how-can-i-navigate-by-using-the-keyboard-in-the-virtual-network-topology-view"></a>Comment faire pour naviguer dans la vue de la topologie des réseaux virtuels à l’aide du clavier ?

La page de la topologie des réseaux virtuels contient deux sections principales :
    
- **Bannière** : la bannière en haut de la topologie des réseaux virtuels fournit des boutons pour sélectionner les filtres de distribution du trafic (réseaux virtuels connectés, réseaux virtuels déconnectés, adresses IP publiques, etc.). Quand vous sélectionnez un bouton, le filtre correspondant est appliqué sur la topologie. Par exemple, si vous sélectionnez le bouton Actif, la topologie met en surbrillance les réseaux virtuels actifs dans votre déploiement.
- **Topologie** : sous la bannière, la section de la topologie montre la distribution du trafic entre les réseaux virtuels.
    
### <a name="keyboard-navigation-on-the-banner"></a>Navigation au clavier sur la bannière
    
- La sélection par défaut dans la bannière de la page de la topologie des réseaux virtuels est le filtre « Réseaux virtuels connectés ».
- Pour passer à un autre filtre, utilisez la touche `Tab` pour vous déplacer vers l’avant. Pour vous déplacer vers l’arrière, utilisez la touche `Shift+Tab`. La navigation vers l’avant va de gauche à droite, puis de haut en bas.
- Appuyez sur `Enter` pour appliquer le filtre sélectionné. Selon la sélection de filtre et le déploiement, un ou plusieurs nœuds (réseau virtuel) sous la section de la topologie sont mis en surbrillance.
- Pour basculer entre la bannière et la topologie, appuyez sur `Ctrl+F6`.
        
### <a name="keyboard-navigation-on-the-topology"></a>Navigation au clavier sur la topologie
    
- Une fois que vous avez sélectionné un filtre sur la bannière et appuyé sur `Ctrl+F6`, le focus passe à l’un des nœuds en surbrillance (**VNet**) dans la vue de la topologie.
- Pour passer à d’autres nœuds en surbrillance dans la vue de la topologie, utilisez la touche `Shift+Right arrow` pour vous déplacer vers l’avant. 
- Sur les nœuds en surbrillance, le focus passe à la **boîte à outils Informations** du nœud. Par défaut, le focus passe au bouton **Plus de détails** de la **boîte à outils Informations**. Pour vous déplacer davantage dans la vue de la **boîte**, utilisez les touches `Right arrow` et `Left arrow` pour vous déplacer vers l’avant et vers l’arrière, respectivement. Appuyer sur `Enter` a le même effet que cliquer sur le bouton actif dans la **boîte à outils Informations**.
- Si vous sélectionnez l’un de ces nœuds, appuyez sur la touche `Shift+Left arrow` pour visiter chacune de ses connexions. Le focus passe à la **boîte à outils Informations** de cette connexion. Vous pouvez à tout moment réappuyer sur `Shift+Right arrow` pour faire revenir le focus sur le nœud.
    

## <a name="how-can-i-navigate-by-using-the-keyboard-in-the-subnet-topology-view"></a>Comment faire pour naviguer dans la vue de la topologie des sous-réseaux à l’aide du clavier ?

La page de la topologie des sous-réseaux virtuels contient deux sections principales :
    
- **Bannière** : la bannière en haut de la topologie des sous-réseaux virtuels fournit des boutons pour sélectionner les filtres de distribution du trafic (sous-réseaux actifs, moyens, de passerelle, etc.). Quand vous sélectionnez un bouton, le filtre correspondant est appliqué sur la topologie. Par exemple, si vous sélectionnez le bouton Actif, la topologie met en surbrillance le sous-réseau virtuel actif dans votre déploiement.
- **Topologie** : sous la bannière, la section de la topologie montre la distribution du trafic entre les sous-réseaux virtuels.
    
### <a name="keyboard-navigation-on-the-banner"></a>Navigation au clavier sur la bannière
    
- La sélection par défaut dans la bannière de la page de la topologie des sous-réseaux virtuels est le filtre « Sous-réseaux ».
- Pour passer à un autre filtre, utilisez la touche `Tab` pour vous déplacer vers l’avant. Pour vous déplacer vers l’arrière, utilisez la touche `Shift+Tab`. La navigation vers l’avant va de gauche à droite, puis de haut en bas.
- Appuyez sur `Enter` pour appliquer le filtre sélectionné. Selon la sélection de filtre et le déploiement, un ou plusieurs nœuds (sous-réseau) sous la section de la topologie sont mis en surbrillance.
- Pour basculer entre la bannière et la topologie, appuyez sur `Ctrl+F6`.
        
### <a name="keyboard-navigation-on-the-topology"></a>Navigation au clavier sur la topologie
    
- Une fois que vous avez sélectionné un filtre sur la bannière et appuyé sur `Ctrl+F6`, le focus passe à l’un des nœuds en surbrillance (**Sous-réseau**) dans la vue de la topologie.
- Pour passer à d’autres nœuds en surbrillance dans la vue de la topologie, utilisez la touche `Shift+Right arrow` pour vous déplacer vers l’avant. 
- Sur les nœuds en surbrillance, le focus passe à la **boîte à outils Informations** du nœud. Par défaut, le focus passe au bouton **Plus de détails** de la **boîte à outils Informations**. Pour vous déplacer davantage dans la vue de la **boîte**, utilisez les touches `Right arrow` et `Left arrow` pour vous déplacer vers l’avant et vers l’arrière, respectivement. Appuyer sur `Enter` a le même effet que cliquer sur le bouton actif dans la **boîte à outils Informations**.
- Si vous sélectionnez l’un de ces nœuds, vous pouvez appuyer sur la touche `Shift+Left arrow` pour visiter chacune de ses connexions. Le focus passe à la **boîte à outils Informations** de cette connexion. Vous pouvez à tout moment réappuyer sur `Shift+Right arrow` pour faire revenir le focus sur le nœud.

## <a name="are-classic-nsgs-supported"></a>Les groupes de sécurité réseau classiques sont-ils pris en charge ?
Non, Traffic Analytics ne prend pas en charge les groupes de sécurité réseau classiques. Nous vous recommandons de migrer les ressources IaaS du niveau classique à Azure Resource Manager en tant que ressources classiques [déconseillées](https://docs.microsoft.com/azure/virtual-machines/classic-vm-deprecation). Consultez cet article pour en savoir plus sur la [migration](https://docs.microsoft.com/azure/virtual-machines/migration-classic-resource-manager-overview).

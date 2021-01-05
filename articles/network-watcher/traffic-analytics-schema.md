---
title: Schéma d’Azure Traffic Analytics | Microsoft Docs
description: Comprendre le schéma de Traffic Analytics pour analyser les journaux de flux de groupe de sécurité réseau Azure.
services: network-watcher
documentationcenter: na
author: vinynigam
manager: agummadi
editor: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/26/2019
ms.author: vinigam
ms.openlocfilehash: ccfbb92c27e4508595f19c2ea6900730cde609b9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "74666373"
---
# <a name="schema-and-data-aggregation-in-traffic-analytics"></a>Schéma et agrégation de données dans Traffic Analytics

Traffic Analytics est une solution cloud qui offre une visibilité de l’activité des utilisateurs et des applications dans vos réseaux cloud. Traffic Analytics analyse les journaux de flux de groupe de sécurité réseau Network Watcher pour fournir des informations sur le flux de trafic dans votre cloud Azure. Avec Traffic Analytics, vous pouvez effectuer les actions suivantes :

- Visualiser l’activité réseau de l’ensemble de vos abonnements Azure et identifier les zones réactives.
- Identifier les menaces de sécurité pour votre réseau et le protéger, avec des informations comme les ports ouverts, les applications qui tentent d’accéder à Internet et les machines virtuelles se connectant à des réseaux non fiables.
- Comprendre les modèles de flux de trafic entre les régions Azure et Internet pour optimiser le déploiement de votre réseau, afin de bénéficier de performances et d’une capacité adéquates.
- Identifier les erreurs de configuration réseau à l’origine d’échecs de connexion dans votre réseau.
- Connaître la consommation du réseau en octets, paquets ou flux.

### <a name="data-aggregation"></a>Agrégation de données

1. Tous les journaux de flux d’un groupe de sécurité réseau (NSG) entre « FlowIntervalStartTime_t » et « FlowIntervalEndTime_t » sont capturés à des intervalles d’une minute dans le compte de stockage en tant qu’objets blob avant d’être traités par Traffic Analytics.
2. L’intervalle de traitement par défaut de Traffic Analytics est de 60 minutes. Cela signifie que toutes les 60 minutes, Traffic Analytics récupère des objets blob du stockage en vue de l’agrégation. Si l’intervalle de traitement choisi est de 10 minutes, Traffic Analytics sélectionnera les objets blob du compte de stockage toutes les 10 minutes.
3. Les flux qui ont les mêmes adresse IP source, adresse IP de destination, port de destination, nom NSG, règle NSG, direction de flux et protocole de transport (TCP ou UDP) (Remarque : le port source n’est pas inclus dans l’agrégation) sont regroupés en un flux unique par Traffic Analytics
4. Cet enregistrement unique est décoré (voir les détails dans la section ci-dessous) et ingéré dans Log Analytics par Traffic Analytics. Ce processus peut prendre jusqu’à 1 heure maximum.
5. Le champ FlowStartTime_t indique la première occurrence de ce type de flux agrégé (même tuple de quatre éléments) dans l’intervalle de traitement des journaux de flux entre « FlowIntervalStartTime_t » et « FlowIntervalEndTime_t ».
6. Pour les ressources dans Traffic Analytics, les flux indiqués dans l’interface utilisateur correspondent à tous les flux vus par le groupe de sécurité réseau. Toutefois, dans Log Analytics, l’utilisateur voit uniquement l’enregistrement unique, réduit. Pour voir tous les flux, utilisez le champ blob_id, qui peut être référencé à partir du stockage. Le nombre total de flux pour cet enregistrement correspond à l’ensemble des flux individuels visibles dans l’objet blob.

La requête ci-dessous vous aide à examiner l’ensemble des journaux de flux des 30 derniers jours en local.
```
AzureNetworkAnalytics_CL
| where SubType_s == "FlowLog" and FlowStartTime_t >= ago(30d) and FlowType_s == "ExternalPublic"
| project Subnet_s  
```
Pour afficher le chemin d’accès de l’objet blob pour les flux dans la requête susmentionnée, utilisez la requête ci-dessous :

```
let TableWithBlobId =
(AzureNetworkAnalytics_CL
   | where SubType_s == "Topology" and ResourceType == "NetworkSecurityGroup" and DiscoveryRegion_s == Region_s and IsFlowEnabled_b
   | extend binTime = bin(TimeProcessed_t, 6h),
            nsgId = strcat(Subscription_g, "/", Name_s),
            saNameSplit = split(FlowLogStorageAccount_s, "/")
   | extend saName = iif(arraylength(saNameSplit) == 3, saNameSplit[2], '')
   | distinct nsgId, saName, binTime)
| join kind = rightouter (
   AzureNetworkAnalytics_CL
   | where SubType_s == "FlowLog"  
   | extend binTime = bin(FlowEndTime_t, 6h)
) on binTime, $left.nsgId == $right.NSGList_s  
| extend blobTime = format_datetime(todatetime(FlowIntervalStartTime_t), "yyyy MM dd hh")
| extend nsgComponents = split(toupper(NSGList_s), "/"), dateTimeComponents = split(blobTime, " ")
| extend BlobPath = strcat("https://", saName,
                        "@insights-logs-networksecuritygroupflowevent/resoureId=/SUBSCRIPTIONS/", nsgComponents[0],
                        "/RESOURCEGROUPS/", nsgComponents[1],
                        "/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/", nsgComponents[2],
                        "/y=", dateTimeComponents[0], "/m=", dateTimeComponents[1], "/d=", dateTimeComponents[2], "/h=", dateTimeComponents[3],
                        "/m=00/macAddress=", replace(@"-", "", MACAddress_s),
                        "/PT1H.json")
| project-away nsgId, saName, binTime, blobTime, nsgComponents, dateTimeComponents;

TableWithBlobId
| where SubType_s == "FlowLog" and FlowStartTime_t >= ago(30d) and FlowType_s == "ExternalPublic"
| project Subnet_s , BlobPath
```

La requête ci-dessus construit une URL permettant d’accéder directement à l’objet blob. Vous trouverez l’URL avec des espaces réservés ci-dessous :

```
https://{saName}@insights-logs-networksecuritygroupflowevent/resoureId=/SUBSCRIPTIONS/{subscriptionId}/RESOURCEGROUPS/{resourceGroup}/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/{nsgName}/y={year}/m={month}/d={day}/h={hour}/m=00/macAddress={macAddress}/PT1H.json

```

### <a name="fields-used-in-traffic-analytics-schema"></a>Champs utilisés dans le schéma de Traffic Analytics
  > [!IMPORTANT]
  > Le schéma Traffic Analytics a été mis à jour le 22 août 2019. Le nouveau schéma fournit les adresses IP source et cible séparément, ce qui élimine la nécessité d'analyser le champ FlowDirection et simplifie les requêtes. </br>
  > Mise à jour de la version 1 de FASchemaVersion_s vers la version 2. </br>
  > Champs dépréciés : VMIP_s, Subscription_s, Region_s, NSGRules_s, Subnet_s, VM_s, NIC_s, PublicIPs_s, FlowCount_d </br>
  > Nouveaux champs : SrcPublicIPs_s, DestPublicIPs_s, NSGRule_s </br>
  > Les champs dépréciés seront disponibles jusqu'au 22 novembre 2019.

Traffic Analytics est basé sur Log Analytics, ce qui vous permet d’exécuter des requêtes personnalisées sur les données décorées par Traffic Analytics et de définir des alertes sur ces données.

Le tableau ci-dessous répertorie les champs contenus dans le schéma et explique leur utilisation

| Champ | Format | Commentaires |
|:---   |:---    |:---  |
| TableName | AzureNetworkAnalytics_CL | Table des données Traffic Analytics
| SubType_s | FlowLog | Sous-type des journaux de flux. Utilisez uniquement « FlowLog ». Les autres valeurs de SubType_s sont destinées au fonctionnement interne du produit |
| FASchemaVersion_s |   2   | Version du schéma. Ne reflète pas la version des journaux de flux de NSG |
| TimeProcessed_t   | Date et heure (UTC)  | Date et heure auxquelles Traffic Analytics a traité les journaux de flux bruts issus du compte de stockage |
| FlowIntervalStartTime_t | Date et heure (UTC) |  Date et heure de début de l’intervalle de traitement des journaux de flux. Il s’agit du moment à partir duquel l’intervalle de flux est mesuré |
| FlowIntervalEndTime_t | Date et heure (UTC) | Date et heure de fin de l’intervalle de traitement des journaux de flux |
| FlowStartTime_t | Date et heure (UTC) |  Première occurrence du flux (qui sera ensuite agrégé) dans l’intervalle de traitement des journaux de flux entre « FlowIntervalStartTime_t » et « FlowIntervalEndTime_t ». Ce flux est agrégé selon une logique d’agrégation |
| FlowEndTime_t | Date et heure (UTC) | Dernière occurrence du flux (qui sera ensuite agrégé) dans l’intervalle de traitement des journaux de flux entre « FlowIntervalStartTime_t » et « FlowIntervalEndTime_t ». Pour les journaux de flux version 2, ce champ contient la date et l’heure auxquelles a démarré le dernier flux ayant le même tuple de quatre éléments (signalé par la lettre « B » dans l’enregistrement de flux bruts) |
| FlowType_s |  * IntraVNet <br> * InterVNet <br> * S2S <br> * P2S <br> * AzurePublic <br> * ExternalPublic <br> * MaliciousFlow <br> * UnknownPrivate <br> * Unknown | Voir la définition dans les remarques en dessous du tableau |
| SrcIP_s | Adresse IP source | Champ vide pour les flux AzurePublic et ExternalPublic |
| DestIP_s | Adresse IP de destination | Champ vide pour les flux AzurePublic et ExternalPublic |
| VMIP_s | Adresse IP de la machine virtuelle | Champ utilisé pour les flux AzurePublic et ExternalPublic |
| PublicIP_s | Adresses IP publiques | Champ utilisé pour les flux AzurePublic et ExternalPublic |
| DestPort_d | Port de destination | Port utilisé pour le trafic entrant |
| L4Protocol_s  | * T <br> * U  | Protocole de transport. T = TCP <br> U = UDP |
| L7Protocol_s  | Nom du protocole | Dérivé du port de destination |
| FlowDirection_s | * I = Entrant<br> * O = Sortant | Direction du flux entrant/sortant du NSG pour le journal de flux |
| FlowStatus_s  | * A = Autorisé par la règle NSG <br> * D = Refusé par la règle NSG  | État du flux autorisé/nbloqué par le NSG pour le journal de flux |
| NSGList_s | \<SUBSCRIPTIONID>\/<RESOURCEGROUP_NAME>\/<NSG_NAME> | NSG (groupe de sécurité réseau) associé au flux |
| NSGRules_s | \<Index value 0)>\|\<NSG_RULENAME>\|\<Flow Direction>\|\<Flow Status>\|\<FlowCount ProcessedByRule> |  Règle NSG ayant autorisé ou refusé ce flux |
| NSGRule_s | NSG_RULENAME |  Règle NSG ayant autorisé ou refusé ce flux |
| NSGRuleType_s | * Défini par l’utilisateur * Par défaut |   Type de la règle NSG utilisée par le flux |
| MACAddress_s | Adresse MAC | Adresse MAC de la carte réseau à laquelle le flux a été capturé |
| Subscription_s | Abonnement pour le réseau virtuel/l’interface réseau/la machine virtuelle Azure | Applicable uniquement pour les types de flux S2S, P2S, AzurePublic, ExternalPublic, MaliciousFlow et UnknownPrivate (types de flux où seul un côté est Azure) |
| Subscription1_s | Identifiant d’abonnement | ID d’abonnement pour le réseau virtuel/l’interface réseau/la machine virtuelle dont fait partie l’adresse IP source dans le flux |
| Subscription2_s | Identifiant d’abonnement | ID d’abonnement pour le réseau virtuel/l’interface réseau/la machine virtuelle dont fait partie l’adresse IP de destination dans le flux |
| Region_s | Région Azure pour le réseau virtuel/l’interface réseau/la machine virtuelle dont fait partie l’adresse IP dans le flux | Applicable uniquement pour les types de flux S2S, P2S, AzurePublic, ExternalPublic, MaliciousFlow et UnknownPrivate (types de flux où seul un côté est Azure) |
| Region1_s | Région Azure | Région Azure pour le réseau virtuel/l’interface réseau/la machine virtuelle dont fait partie l’adresse IP source dans le flux |
| Region2_s | Région Azure | Région Azure pour le réseau virtuel dont fait partie l’adresse IP de destination dans le flux |
| NIC_s | \<resourcegroup_Name>\/\<NetworkInterfaceName> |  Carte réseau associée à la machine virtuelle qui envoie ou reçoit le trafic |
| NIC1_s | <resourcegroup_Name>/\<NetworkInterfaceName> | Carte réseau associée à l’adresse IP source dans le flux |
| NIC2_s | <resourcegroup_Name>/\<NetworkInterfaceName> | Carte réseau associée à l’adresse IP de destination dans le flux |
| VM_s | <resourcegroup_Name>\/\<NetworkInterfaceName> | Machine virtuelle associée à l’interface réseau NIC_s |
| VM1_s | <resourcegroup_Name>/\<VirtualMachineName> | Machine virtuelle associée à l’adresse IP source dans le flux |
| VM2_s | <resourcegroup_Name>/\<VirtualMachineName> | Machine virtuelle associée à l’adresse IP de destination dans le flux |
| Subnet_s | <ResourceGroup_Name>/<VNET_Name>/\<SubnetName> | Sous-réseau associé à NIC_s |
| Subnet1_s | <ResourceGroup_Name>/<VNET_Name>/\<SubnetName> | Sous-réseau associé à l’adresse IP source dans le flux |
| Subnet2_s | <ResourceGroup_Name>/<VNET_Name>/\<SubnetName>    | Sous-réseau associé à l’adresse IP de destination dans le flux |
| ApplicationGateway1_s | \<SubscriptionID>/\<ResourceGroupName>/\<ApplicationGatewayName> | Passerelle d’application associée à l’adresse IP source dans le flux |
| ApplicationGateway2_s | \<SubscriptionID>/\<ResourceGroupName>/\<ApplicationGatewayName> | Passerelle d’application associée à l’adresse IP de destination dans le flux |
| LoadBalancer1_s | \<SubscriptionID>/\<ResourceGroupName>/\<LoadBalancerName> | Équilibreur de charge associé à l’adresse IP source dans le flux |
| LoadBalancer2_s | \<SubscriptionID>/\<ResourceGroupName>/\<LoadBalancerName> | Équilibreur de charge associé à l’adresse IP de destination dans le flux |
| LocalNetworkGateway1_s | \<SubscriptionID>/\<ResourceGroupName>/\<LocalNetworkGatewayName> | Passerelle de réseau local associée à l’adresse IP source dans le flux |
| LocalNetworkGateway2_s | \<SubscriptionID>/\<ResourceGroupName>/\<LocalNetworkGatewayName> | Passerelle de réseau local associée à l’adresse IP de destination dans le flux |
| ConnectionType_s | Valeurs possibles : VNetPeering, VpnGateway et ExpressRoute |    Type de la connexion |
| ConnectionName_s | \<SubscriptionID>/\<ResourceGroupName>/\<ConnectionName> | Nom de la connexion. Pour flowtype P2S, le format sera <gateway name>_<VPN Client IP> |
| ConnectingVNets_s | Liste de noms de réseau virtuel séparés par un espace | Dans une topologie hub-and-spoke, les réseaux virtuels du hub sont indiqués ici |
| Country_s | Code de pays à deux lettres (ISO 3166-1 alpha-2) | Champ rempli pour le type de flux ExternalPublic. Toutes les adresses IP indiquées dans le champ PublicIPs_s ont le même code de pays |
| AzureRegion_s | Emplacements de la région Azure | Champ rempli pour le type de flux AzurePublic. Toutes les adresses IP indiquées dans le champ PublicIPs_s se trouvent dans la même région Azure |
| AllowedInFlows_d | | Nombre de flux entrants ayant été autorisés. Cela représente le nombre de flux entrants ayant utilisé le même tuple de quatre éléments sur l’interface réseau où les flux ont été capturés |
| DeniedInFlows_d |  | Nombre de flux entrants ayant été refusés. (Flux entrants sur l’interface réseau où les flux ont été capturés) |
| AllowedOutFlows_d | | Nombre de flux sortants ayant été autorisés. (Flux sortants sur l’interface réseau où les flux ont été capturés) |
| DeniedOutFlows_d  | | Nombre de flux sortants ayant été refusés. (Flux sortants sur l’interface réseau où les flux ont été capturés) |
| FlowCount_d | Action déconseillée. Total des flux ayant utilisé le même tuple de quatre éléments. Pour les types de flux ExternalPublic et AzurePublic, ce nombre inclut également les flux provenant de différentes adresses PublicIP.
| InboundPackets_d | Paquets reçus après avoir été capturés sur l’interface réseau où la règle NSG a été appliquée | Champ rempli uniquement pour la version 2 du schéma des journaux de flux de NSG |
| OutboundPackets_d  | Paquets envoyés après avoir été capturés sur l’interface réseau où la règle NSG a été appliquée | Champ rempli uniquement pour la version 2 du schéma des journaux de flux de NSG |
| InboundBytes_d |  Octets reçus après avoir été capturés sur l’interface réseau où la règle NSG a été appliquée | Champ rempli uniquement pour la version 2 du schéma des journaux de flux de NSG |
| OutboundBytes_d | Octets envoyés après avoir été capturés sur l’interface réseau où la règle NSG a été appliquée | Champ rempli uniquement pour la version 2 du schéma des journaux de flux de NSG |
| CompletedFlows_d  |  | Champ rempli avec des valeurs différentes de zéro uniquement pour la version 2 du schéma des journaux de flux de NSG |
| PublicIPs_s | <PUBLIC_IP>\|\<FLOW_STARTED_COUNT>\|\<FLOW_ENDED_COUNT>\|\<OUTBOUND_PACKETS>\|\<INBOUND_PACKETS>\|\<OUTBOUND_BYTES>\|\<INBOUND_BYTES> | Entrées séparées par des barres |
| SrcPublicIPs_s | <SOURCE_PUBLIC_IP>\|\<FLOW_STARTED_COUNT>\|\<FLOW_ENDED_COUNT>\|\<OUTBOUND_PACKETS>\|\<INBOUND_PACKETS>\|\<OUTBOUND_BYTES>\|\<INBOUND_BYTES> | Entrées séparées par des barres |
| DestPublicIPs_s | <DESTINATION_PUBLIC_IP>\|\<FLOW_STARTED_COUNT>\|\<FLOW_ENDED_COUNT>\|\<OUTBOUND_PACKETS>\|\<INBOUND_PACKETS>\|\<OUTBOUND_BYTES>\|\<INBOUND_BYTES> | Entrées séparées par des barres |

### <a name="notes"></a>Notes

1. Pour les flux AzurePublic et ExternalPublic, l’IP de machine virtuelle Azure détenue par un client est indiquée dans le champ VMIP_s, alors que les adresses IP publiques le sont dans le champ PublicIPs_s. Pour ces deux types de flux, utilisez les champs VMIP_s et PublicIPs_s au lieu des champs SrcIP_s et DestIP_s. Pour les adresses AzurePublic et ExternalPublicIP, nous agrégeons encore davantage les données afin que le nombre d’enregistrements ingérés dans l’espace de travail Log Analytics du client soit réduit au minimum. (Ce champ sera bientôt déprécié. Vous devrez ensuite utiliser le champ SrcIP_ ou DestIP_s selon que la machine virtuelle Azure est définie comme source ou destination dans le flux)
1. Remarques sur les types de flux : Sur la base des adresses IP impliquées dans les flux, nous classons les flux dans les types de flux suivants :
1. IntraVNet : les deux adresses IP dans le flux se trouvent dans le même réseau virtuel Azure.
1. InterVNet : les adresses IP dans le flux se trouvent dans deux réseaux virtuels Azure différents.
1. S2S (Site à site) : l’une des adresses IP fait partie du réseau virtuel Azure tandis que l’autre adresse IP fait partie du réseau client (Site) qui est connecté au réseau virtuel Azure par le biais d’une passerelle VPN ou Express Route.
1. P2S (Point à site) : l’une des adresses IP fait partie du réseau virtuel Azure tandis que l’autre adresse IP fait partie du réseau client (Site) qui est connecté au réseau virtuel Azure par le biais d’une passerelle VPN.
1. AzurePublic : l’une des adresses IP fait partie du réseau virtuel Azure tandis que l’autre adresse IP fait partie des adresses IP publiques internes Azure détenues par Microsoft. Les adresses IP publiques détenues par un client ne sont pas incluses dans ce type de flux. Par exemple, le trafic envoyé à partir d’une machine virtuelle d’un client vers un service Azure (point de terminaison de stockage) est classé dans ce type de flux.
1. ExternalPublic : l’une des adresses IP fait partie du réseau virtuel Azure tandis que l’autre adresse IP fait partie des adresses IP publiques externes à Azure qui ne sont pas signalées comme étant malveillantes dans les flux ASC consommés par Traffic Analytics dans l’intervalle de traitement entre « FlowIntervalStartTime_t » et « FlowIntervalEndTime_t ».
1. MaliciousFlow : l’une des adresses IP fait partie du réseau virtuel Azure tandis que l’autre adresse IP fait partie des adresses IP publiques externes à Azure qui sont signalées comme étant malveillantes dans les flux ASC consommés par Traffic Analytics dans l’intervalle de traitement entre « FlowIntervalStartTime_t » et « FlowIntervalEndTime_t ».
1. UnknownPrivate : l’une des adresses IP fait partie du réseau virtuel Azure tandis que l’autre adresse IP fait partie de la plage d’adresses IP privées, telles que définies dans la RFC 1918, qui ne peuvent pas être mappées par Traffic Analytics à un site client ou au réseau virtuel Azure.
1. Unknown : impossible de mapper l’une des adresses IP dans les flux avec la topologie cliente dans Azure ou en local (sur site).
1. Les lettres \_s ou \_d sont ajoutées à certains noms de champ. Celles-ci ne désignent PAS la source et la cible mais indiquent les types de données chaîne et décimal, respectivement.

### <a name="next-steps"></a>Étapes suivantes
Pour obtenir des réponses aux questions fréquentes, consultez [Questions fréquentes (FAQ) sur Traffic Analytics](traffic-analytics-faq.md). Pour plus d’informations sur des fonctionnalités, consultez la [documentation Traffic Analytics](traffic-analytics.md)

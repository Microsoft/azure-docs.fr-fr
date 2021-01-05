---
title: Interroger des journaux d’activité à partir d’Azure Monitor pour machines virtuelles
description: Azure Monitor pour machines virtuelles collecte des métriques et des données de journal. Cet article décrit les enregistrements correspondants et inclut des exemples de requêtes.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 03/12/2020
ms.openlocfilehash: 118bdcb6929abfc162ff05e91f1621f087b6c50c
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96186726"
---
# <a name="how-to-query-logs-from-azure-monitor-for-vms"></a>Comment interroger des journaux d’activité à partir d’Azure Monitor pour les machines virtuelles

Azure Monitor pour machines virtuelles collecte des métriques de performances et de connexion, les données d’inventaire des ordinateurs et processus et des informations concernant l’état d’intégrité, puis les transfère à l'espace de travail Log Analytics dans Azure Monitor.  Ces données sont disponibles pour la [requête](../log-query/log-query-overview.md) dans Azure Monitor. Vous pouvez appliquer ces données à divers scénarios tels que la planification de la migration, l’analyse de la capacité, la détection et la résolution de problèmes de performances à la demande.

## <a name="map-records"></a>Mapper des enregistrements

Un enregistrement est généré par heure pour chaque ordinateur et processus, en plus des enregistrements générés lorsqu’un processus ou un ordinateur démarre ou est intégré à la fonctionnalité Map d’Azure Monitor pour les machines virtuelles. Les propriétés de ces enregistrements sont décrites dans les tableaux suivants. Les champs et les valeurs des événements ServiceMapComputer_CL sont mappés aux champs de la ressource Machine dans l’API Azure Resource Manager ServiceMap. Les champs et les valeurs des événements ServiceMapProcess_CL sont mappés aux champs de la ressource Processus dans l’API Azure Resource Manager ServiceMap. Le champ ResourceName_s correspond au champ de nom dans la ressource Azure Resource Manager correspondante. 

Il existe des propriétés générées en interne que vous pouvez utiliser pour identifier les ordinateurs et processus uniques :

- Ordinateur : Utilisez *ResourceId* ou *ResourceName_s* pour identifier de façon unique un ordinateur au sein d’un espace de travail Log Analytics.
- Processus : Utilisez *ResourceId* pour identifier de façon unique un processus au sein d’un espace de travail Log Analytics. *ResourceName_s* est unique dans le contexte de l’ordinateur sur lequel le processus est en cours d’exécution (MachineResourceName_s). 

Étant donné que plusieurs enregistrements peuvent exister pour un processus et un ordinateur donnés au cours d’une période spécifique, les requêtes peuvent renvoyer plusieurs enregistrements pour un même ordinateur ou processus. Pour n’inclure que le tout dernier enregistrement, ajoutez `| summarize arg_max(TimeGenerated, *) by ResourceId` à la requête.

### <a name="connections-and-ports"></a>Connexions et ports

La fonctionnalité Métriques de connexion introduit deux nouvelles tables dans les journaux d'activité Azure Monitor - VMConnection et VMBoundPort. Ces tables fournissent des informations sur les connexions d'une machine (trafic entrant et sortant), ainsi que sur les ports de serveur ouverts/actifs. Les métriques de connexion sont également exposées via des API permettant d’obtenir une métrique spécifique dans une fenêtre de temps. Les connexions TCP résultant de l’acceptation (*accept*) sur un socket d’écoute sont des connexions entrantes, tandis que celles créées par le biais d’une connexion (*connect*) à une adresse IP et un port spécifiques sont des connexions sortantes. La direction d’une connexion est représentée par la propriété Direction, qui peut être définie avec la valeur **inbound** ou **outbound**. 

Les enregistrements inclus dans ces tables sont générés à partir des données rapportées par l'agent de dépendances. Chaque enregistrement représente une observation sur un intervalle de temps d’une minute. La propriété TimeGenerated indique le début de l’intervalle de temps. Chaque enregistrement contient des informations identifiant l’entité respective (à savoir la connexion ou le port), ainsi que des métriques associées à cette entité. Actuellement, seule l’activité réseau utilisant TCP sur IPv4 est rapportée. 

#### <a name="common-fields-and-conventions"></a>Champs et conventions courants 

Les champs et les conventions suivants s’appliquent à VMConnection et VMBoundPort : 

- Ordinateur : Nom de domaine complet de la machine qui rend compte 
- ID d’agent : Identificateur unique de la machine hébergeant l'agent Log Analytics  
- Machine : Nom de la ressource Azure Resource Manager pour la machine exposée par ServiceMap. Il se présente au format *m-{GUID}* , où *GUID* correspond au même GUID que l’ID d'agent  
- Processus : Nom de la ressource Azure Resource Manager pour le processus exposé par ServiceMap. Il se présente au format *p-{chaîne hexadécimale}* . Le processus est unique au sein d’une étendue de machine et pour générer un ID de processus unique sur plusieurs machines, combinez des champs de machine et de processus. 
- ProcessName : Nom de l’exécutable du processus de création de rapports.
- Toutes les adresses IP sont des chaînes au format canonique IPv4, par exemple *13.107.3.160* 

À des fins de gestion des coûts et de la complexité, les enregistrements de connexion ne représentent pas des connexions réseau physiques individuelles. Plusieurs connexions réseau physiques sont groupées dans une connexion logique, qui est ensuite reflétée dans la table concernée.  Ainsi, les enregistrements de la table *VMConnection* représentent un regroupement logique et non les connexions physiques individuelles observées. Les connexions réseau physiques dont les attributs suivants présentent la même valeur au cours d’un intervalle d’une minute donné sont agrégées au sein d’un unique enregistrement logique dans *VMConnection*. 

| Propriété | Description |
|:--|:--|
|Sens |Direction de la connexion (valeur *inbound* ou *outbound*) |
|Machine |Nom de domaine complet (FQDN) de l’ordinateur |
|Process |Identité du processus ou de groupes de processus initialisant/acceptant la connexion |
|SourceIp |Adresse IP de la source |
|DestinationIp |Adresse IP de la destination |
|DestinationPort |Numéro de port de la destination |
|Protocol |Protocole utilisé pour la connexion.  La valeur est *tcp*. |

Pour prendre en compte l’impact du regroupement, les informations sur le nombre de connexions physiques groupées sont fournies dans les propriétés suivantes de l’enregistrement :

| Propriété | Description |
|:--|:--|
|LinksEstablished |Nombre de connexions réseau physiques qui ont été établies dans la fenêtre de temps de rapport |
|LinksTerminated |Nombre de connexions réseau physiques qui ont pris fin dans la fenêtre de temps de rapport |
|LinksFailed |Nombre de connexions réseau physiques qui ont échoué dans la fenêtre de temps de rapport. Actuellement, ces informations sont disponibles pour les connexions sortantes uniquement. |
|LinksLive |Nombre de connexions réseau physiques qui ont été ouvertes à la fin de la fenêtre de temps de rapport|

#### <a name="metrics"></a>Mesures

En plus des métriques concernant le nombre de connexions, des informations sur le volume de données envoyées et reçues sur une connexion logique ou un port réseau donné sont également incluses dans les propriétés suivantes de l’enregistrement :

| Propriété | Description |
|:--|:--|
|BytesSent |Nombre total d’octets qui ont été envoyés dans la fenêtre de temps de rapport |
|BytesReceived |Nombre total d’octets qui ont été reçus dans la fenêtre de temps de rapport |
|Réponses |Nombre de réponses observées dans la fenêtre de temps de rapport. 
|ResponseTimeMax |Temps de réponse le plus long (en millisecondes) observé dans la fenêtre de temps de rapport. En l’absence de valeur, la propriété est vide.|
|ResponseTimeMin |Temps de réponse le plus court (en millisecondes) observé dans la fenêtre de temps de rapport. En l’absence de valeur, la propriété est vide.|
|ResponseTimeSum |Somme de tous les temps de réponse (en millisecondes) observés dans la fenêtre de temps de rapport. En l’absence de valeur, la propriété est vide.|

Le troisième type de données rapportées est le temps de réponse : le temps passé par un appelant à attendre qu’une requête envoyée sur une connexion soit traitée par le point de terminaison distant et que ce dernier y réponde. Le temps de réponse rapporté est une estimation du temps de réponse réel du protocole d’application sous-jacent. Il est calculé à l’aide d’une méthode heuristique basée sur l’observation du flux de données entre la source et la destination d’une connexion réseau physique. Sur le plan conceptuel, il s’agit de la différence entre le moment auquel le dernier octet d’une requête quitte l’expéditeur et celui auquel le dernier octet de la réponse lui revient. Ces deux timestamps sont utilisés pour délimiter les événements de requête et de réponse sur une connexion physique donnée. La différence entre eux représente le temps de réponse d’une requête unique. 

Dans le cadre de la première version de cette fonctionnalité, notre algorithme est une approximation qui peut fonctionner de façon plus ou moins efficace selon le protocole d’application réel utilisé pour une connexion réseau donnée. Par exemple, l’approche actuelle fonctionne bien pour les protocoles de requête-réponse comme HTTP(S), mais ne fonctionne pas avec les protocoles unidirectionnels ou basés sur des files d’attente de messages.

Voici quelques points importants à prendre en compte :

1. Si un processus accepte des connexions sur la même adresse IP mais sur plusieurs interfaces réseau, un enregistrement distinct pour chaque interface est rapporté. 
2. Les enregistrements avec une adresse IP générique ne contiennent aucune activité. Ils sont inclus pour représenter le fait qu’un port de la machine est ouvert pour le trafic entrant.
3. Pour réduire le niveau de détail et le volume de données, les enregistrements avec IP générique sont omis en présence d’un enregistrement correspondant (pour le même processus, le même port et le même protocole) avec une adresse IP spécifique. Lorsqu’un enregistrement IP générique est omis, la propriété d’enregistrement IsWildcardBind avec l’adresse IP spécifique a la valeur « True » pour indiquer que le port est exposé sur toutes les interfaces de la machine à l’origine du rapport.
4. Pour les ports liés à une interface spécifique uniquement, IsWildcardBind a la valeur *False*.

#### <a name="naming-and-classification"></a>Affectation de noms et classification

Pour plus de commodité, l’adresse IP de l’extrémité distante d’une connexion est incluse dans la propriété RemoteIp. Pour les connexions entrantes, RemoteIp est identique à SourceIp, tandis que pour les connexions sortantes, elle est identique à DestinationIp. La propriété RemoteDnsCanonicalNames représente les noms canoniques DNS rapportés par la machine pour RemoteIp. Les propriétés RemoteDnsQuestions et RemoteClassification sont réservées en vue d’une utilisation ultérieure. 

#### <a name="geolocation"></a>Géolocalisation

*VMConnection* inclut également des informations de géolocalisation pour l’extrémité distante de chaque enregistrement de connexion dans les propriétés suivantes de l’enregistrement : 

| Propriété | Description |
|:--|:--|
|RemoteCountry |Nom du pays/de la région hébergeant RemoteIp.  Par exemple, *États-Unis* |
|RemoteLatitude |Latitude de géolocalisation. Par exemple, *47,68*. |
|RemoteLongitude |Longitude de géolocalisation. Par exemple, *-122,12*. |

#### <a name="malicious-ip"></a>Adresses IP malveillantes

Chaque propriété RemoteIp de la table *VMConnection* est comparée à un ensemble d’adresses IP associées à une activité malveillante connue. Si la propriété RemoteIp est identifiée comme malveillante, les propriétés suivantes de l’enregistrement sont renseignées (elles sont vides lorsque l’adresse IP n’est pas considérée comme malveillante) :

| Propriété | Description |
|:--|:--|
|MaliciousIP |Adresse RemoteIp |
|IndicatorThreadType |L’indicateur de menace détecté est l’une des valeurs suivantes :  *Botnet*, *C2*, *CryptoMining*, *Darknet*, *DDos*, *MaliciousUrl*, *Malware*, *Phishing*, *Proxy*, *PUA* ou *Watchlist*.   |
|Description |Description de la menace observée. |
|TLPLevel |Niveau de protocole TLP (Traffic Light Protocol) est réglé sur l’une des valeurs définies, *Blanc*, *Vert*, *Orange*, *Rouge*. |
|Confiance |Les valeurs sont comprises dans la fourchette *0 – 100*. |
|severity |Les valeurs sont comprises dans la fourchette *0 – 5*,  dans laquelle *5* correspond à la gravité maximale et *0* à l’absence de gravité. La valeur par défaut est *3*.  |
|FirstReportedDateTime |La première fois que le fournisseur a déclaré l’indicateur. |
|LastReportedDateTime |La dernière fois que l’indicateur a été vu par Interflow. |
|IsActive |Indique les indicateurs sont désactivés avec la valeur *True* ou la valeur *False*. |
|ReportReferenceLink |Liens vers des rapports relatifs à un observable donné. |
|AdditionalInformation |Fournit des informations supplémentaires, s’il y a lieu, sur la menace observée. |

### <a name="ports"></a>Ports 

Les ports d'une machine acceptant activement le trafic entrant ou susceptible de l'accepter, mais inactifs dans la fenêtre de temps de rapport, sont consignés dans la table VMBoundPort.  

Chaque enregistrement de VMBoundPort est identifié par les champs suivants : 

| Propriété | Description |
|:--|:--|
|Process | Identité du processus (ou groupes de processus) auquel le port est associé.|
|IP | Adresse IP du port (peut correspondre à une adresse IP générique, *0.0.0.0*) |
|Port |Numéro de port |
|Protocol | Protocole.  Exemple, *tcp* ou *udp* (actuellement, seul *tcp* est pris en charge).|
 
Identité d'un port dérivée des cinq champs ci-dessus et stockée dans la propriété PortId. Cette propriété peut être utilisée pour rapidement rechercher les enregistrements d'un port spécifique au fil du temps. 

#### <a name="metrics"></a>Mesures 

Les enregistrements de port incluent des métriques représentant les connexions qui s’y rapportent. Actuellement, les métriques suivantes sont signalées (les détails de chaque métrique sont décrits dans la section précédente) : 

- BytesSent et BytesReceived 
- LinksEstablished, LinksTerminated, LinksLive 
- ResposeTime, ResponseTimeMin, ResponseTimeMax, ResponseTimeSum 

Voici quelques points importants à prendre en compte :

- Si un processus accepte des connexions sur la même adresse IP mais sur plusieurs interfaces réseau, un enregistrement distinct pour chaque interface est rapporté.  
- Les enregistrements avec une adresse IP générique ne contiennent aucune activité. Ils sont inclus pour représenter le fait qu’un port de la machine est ouvert pour le trafic entrant. 
- Pour réduire le niveau de détail et le volume de données, les enregistrements avec IP générique sont omis en présence d’un enregistrement correspondant (pour le même processus, le même port et le même protocole) avec une adresse IP spécifique. Lorsqu’un enregistrement d’adresse IP générique est omis, la propriété *IsWildcardBind* de l’enregistrement avec l’adresse IP spécifique est définie sur *True*.  Cela indique que le port est exposé via toutes les interfaces de la machine qui rend compte. 
- Pour les ports liés à une interface spécifique uniquement, IsWildcardBind a la valeur *False*. 

### <a name="vmcomputer-records"></a>Enregistrements VMComputer

Les enregistrements de type *VMComputer* ont des données d’inventaire pour les serveurs avec Dependency Agent. Les propriétés de ces enregistrements sont décrites dans le tableau suivant :

| Propriété | Description |
|:--|:--|
|TenantId | Identificateur unique de l’espace de travail |
|SourceSystem | *Insights* | 
|TimeGenerated | Horodatage de l’enregistrement (UTC) |
|Computer | Nom de domaine complet (FQDN) de l’ordinateur | 
|AgentID | ID unique de l’agent Log Analytics |
|Machine | Nom de la ressource Azure Resource Manager pour la machine exposée par ServiceMap. Il se présente au format *m-{GUID}* , où *GUID* correspond au même GUID que l’ID d'agent. | 
|DisplayName | Nom complet | 
|FullDisplayName | Nom d’affichage complet | 
|HostName | Nom de la machine sans nom de domaine |
|BootTime | Heure de démarrage de la machine (UTC) |
|TimeZone | Fuseau horaire normalisé |
|VirtualizationState | *virtual*, *hypervisor*, *physical* |
|Ipv4Addresses | Tableau d’adresses IPv4 | 
|Ipv4SubnetMasks | Tableau de masques de sous-réseau IPv4 (dans le même ordre que Ipv4Addresses). |
|Ipv4DefaultGateways | Tableau de passerelles IPv4 | 
|Ipv6Addresses | Tableau d’adresses IPv6 | 
|MacAddresses | Tableau d’adresses MAC | 
|DnsNames | Tableau de noms DNS associés à la machine. |
|DependencyAgentVersion | Version de l’agent de dépendances qui s’exécute sur la machine. | 
|OperatingSystemFamily | *Linux*, *Windows* |
|OperatingSystemFullName | Nom complet du système d’exploitation | 
|PhysicalMemoryMB | Mémoire physique en mégaoctets | 
|Processeurs | Nombre de processeurs | 
|CPUSpeed | Vitesse du processeur en MHz | 
|VirtualMachineType | *hyperv*, *vmware*, *xen* |
|VirtualMachineNativeId | ID de machine virtuelle assigné par son hyperviseur | 
|VirtualMachineNativeName | Nom de la machine virtuelle |
|VirtualMachineHypervisorId | Identificateur unique de l’hyperviseur hébergeant la machine virtuelle |
|HypervisorType | *hyperv* |
|HypervisorId | ID unique de l’hyperviseur | 
|HostingProvider | *azure* |
|_ResourceId | Identificateur unique d’une ressource Azure |
|AzureSubscriptionId | Identificateur global unique identifiant votre abonnement | 
|AzureResourceGroup | Nom du groupe de ressources Azure dont la machine est membre. |
|AzureResourceName | Nom de la ressource Azure |
|AzureLocation | L’emplacement de la ressource Azure |
|AzureUpdateDomain | Nom du domaine de mise à jour Azure |
|AzureFaultDomain | Nom du domaine d’erreur Azure |
|AzureVmId | Identificateur unique de la machine virtuelle Azure |
|AzureSize | Taille de la machine virtuelle Azure |
|AzureImagePublisher | Nom de l’éditeur de machines virtuelles Azure |
|AzureImageOffering | Nom du type d’offre de machine virtuelle Azure | 
|AzureImageSku | Référence SKU de l’image de machine virtuelle Azure | 
|AzureImageVersion | Version de l’image de machine virtuelle Azure | 
|AzureCloudServiceName | Nom du service cloud Azure |
|AzureCloudServiceDeployment | ID de déploiement pour le service cloud |
|AzureCloudServiceRoleName | Nom de rôle Cloud Service |
|AzureCloudServiceRoleType | Type de rôle de service cloud : *Worker* ou *Web* |
|AzureCloudServiceInstanceId | ID d’instance du rôle de service cloud |
|AzureVmScaleSetName | Nom du groupe de machines virtuelles identiques |
|AzureVmScaleSetDeployment | ID de déploiement d’un groupe de machines virtuelles identiques |
|AzureVmScaleSetResourceId | Identificateur unique de la ressource de groupe de machines virtuelles identiques.|
|AzureVmScaleSetInstanceId | Identificateur unique du groupe de machines virtuelles identiques |
|AzureServiceFabricClusterId | Identificateur unique du cluster Azure Service Fabric | 
|AzureServiceFabricClusterName | Nom du cluster Azure Service Fabric |

### <a name="vmprocess-records"></a>Enregistrements VMProcess

Les enregistrements de type *VMProcess* ont des données d’inventaire pour les processus connectés à TCP sur les serveurs avec Dependency Agent. Les propriétés de ces enregistrements sont décrites dans le tableau suivant :

| Propriété | Description |
|:--|:--|
|TenantId | Identificateur unique de l’espace de travail |
|SourceSystem | *Insights* | 
|TimeGenerated | Horodatage de l’enregistrement (UTC) |
|Computer | Nom de domaine complet (FQDN) de l’ordinateur | 
|AgentID | ID unique de l’agent Log Analytics |
|Machine | Nom de la ressource Azure Resource Manager pour la machine exposée par ServiceMap. Il se présente au format *m-{GUID}* , où *GUID* correspond au même GUID que l’ID d'agent. | 
|Process | Identificateur unique du processus Service Map. Il se présente sous la forme *p-{GUID}* . 
|ExecutableName | Nom de l’exécutable du processus | 
|DisplayName | Nom complet du processus |
|Role | Rôle du processus : *webserver*, *appServer*, *databaseServer*, *ldapServer*, *smbServer* |
|Groupe | Nom du groupe de processus. Les processus du même groupe sont liés de manière logique, par exemple en faisant partie du même produit ou composant système. |
|StartTime | Heure de début du pool de processus |
|FirstPid | Premier PID dans le pool de processus |
|Description | Description du processus |
|CompanyName | Nom de la société |
|InternalName | Nom interne |
|ProductName | Nom du produit |
|ProductVersion | La version du produit |
|FileVersion | Version du fichier |
|ExecutablePath |Chemin de l'exécutable |
|CommandLine | Ligne de commande |
|WorkingDirectory | Le répertoire de travail |
|Services | Tableau de services sous lequel le processus s’exécute |
|UserName | Compte sous lequel le processus s’exécute |
|UserDomain | Domaine sous lequel le processus s’exécute |
|_ResourceId | Identificateur unique d’un processus au sein de l’espace de travail |


## <a name="sample-map-queries"></a>Exemple de requêtes de carte

### <a name="list-all-known-machines"></a>Liste de tous les ordinateurs connus

```kusto
VMComputer | summarize arg_max(TimeGenerated, *) by _ResourceId
```

### <a name="when-was-the-vm-last-rebooted"></a>Date/Heure du dernier redémarrage de la machine virtuelle

```kusto
let Today = now(); VMComputer | extend DaysSinceBoot = Today - BootTime | summarize by Computer, DaysSinceBoot, BootTime | sort by BootTime asc
```

### <a name="summary-of-azure-vms-by-image-location-and-sku"></a>Résumé des machines virtuelles Azure par image, emplacement et référence SKU

```kusto
VMComputer | where AzureLocation != "" | summarize by Computer, AzureImageOffering, AzureLocation, AzureImageSku
```

### <a name="list-the-physical-memory-capacity-of-all-managed-computers"></a>Répertorier la capacité de mémoire physique de tous les ordinateurs gérés

```kusto
VMComputer | summarize arg_max(TimeGenerated, *) by _ResourceId | project PhysicalMemoryMB, Computer
```

### <a name="list-computer-name-dns-ip-and-os"></a>Répertorier le nom de l’ordinateur, le DNS, l’adresse IP et le système d’exploitation

```kusto
VMComputer | summarize arg_max(TimeGenerated, *) by _ResourceId | project Computer, OperatingSystemFullName, DnsNames, Ipv4Addresses
```

### <a name="find-all-processes-with-sql-in-the-command-line"></a>Rechercher tous les processus contenant "sql" dans la ligne de commande

```kusto
VMProcess | where CommandLine contains_cs "sql" | summarize arg_max(TimeGenerated, *) by _ResourceId
```

### <a name="find-a-machine-most-recent-record-by-resource-name"></a>Rechercher un ordinateur (enregistrement le plus récent) par nom de ressource

```kusto
search in (VMComputer) "m-4b9c93f9-bc37-46df-b43c-899ba829e07b" | summarize arg_max(TimeGenerated, *) by _ResourceId
```

### <a name="find-a-machine-most-recent-record-by-ip-address"></a>Rechercher une machine (enregistrement le plus récent) par adresse IP

```kusto
search in (VMComputer) "10.229.243.232" | summarize arg_max(TimeGenerated, *) by _ResourceId
```

### <a name="list-all-known-processes-on-a-specified-machine"></a>Répertorier tous les processus sur une machine spécifiée

```kusto
VMProcess | where Machine == "m-559dbcd8-3130-454d-8d1d-f624e57961bc" | summarize arg_max(TimeGenerated, *) by _ResourceId
```

### <a name="list-all-computers-running-sql-server"></a>Répertorier tous les ordinateurs exécutant SQL Server

```kusto
VMComputer | where AzureResourceName in ((search in (VMProcess) "*sql*" | distinct Machine)) | distinct Computer
```

### <a name="list-all-unique-product-versions-of-curl-in-my-datacenter"></a>Répertorier toutes les versions de produit uniques de CURL dans mon centre de données

```kusto
VMProcess | where ExecutableName == "curl" | distinct ProductVersion
```

### <a name="create-a-computer-group-of-all-computers-running-centos"></a>Créer un groupe de tous les ordinateurs exécutant CentOS

```kusto
VMComputer | where OperatingSystemFullName contains_cs "CentOS" | distinct Computer
```

### <a name="bytes-sent-and-received-trends"></a>Tendances en matière d'octets envoyés et reçus

```kusto
VMConnection | summarize sum(BytesSent), sum(BytesReceived) by bin(TimeGenerated,1hr), Computer | order by Computer desc | render timechart
```

### <a name="which-azure-vms-are-transmitting-the-most-bytes"></a>Identification des machines virtuelles Azure qui transmettent le plus grand nombre d'octets

```kusto
VMConnection | join kind=fullouter(VMComputer) on $left.Computer == $right.Computer | summarize count(BytesSent) by Computer, AzureVMSize | sort by count_BytesSent desc
```

### <a name="link-status-trends"></a>Tendances en matière d'état du lien

```kusto
VMConnection | where TimeGenerated >= ago(24hr) | where Computer == "acme-demo" | summarize dcount(LinksEstablished), dcount(LinksLive), dcount(LinksFailed), dcount(LinksTerminated) by bin(TimeGenerated, 1h) | render timechart
```

### <a name="connection-failures-trend"></a>Tendance en matière d'échecs de connexion

```kusto
VMConnection | where Computer == "acme-demo" | extend bythehour = datetime_part("hour", TimeGenerated) | project bythehour, LinksFailed | summarize failCount = count() by bythehour | sort by bythehour asc | render timechart
```

### <a name="bound-ports"></a>Ports liés

```kusto
VMBoundPort
| where TimeGenerated >= ago(24hr)
| where Computer == 'admdemo-appsvr'
| distinct Port, ProcessName
```

### <a name="number-of-open-ports-across-machines"></a>Nombre de ports ouverts sur différentes machines

```kusto
VMBoundPort
| where Ip != "127.0.0.1"
| summarize by Computer, Machine, Port, Protocol
| summarize OpenPorts=count() by Computer, Machine
| order by OpenPorts desc
```

### <a name="score-processes-in-your-workspace-by-the-number-of-ports-they-have-open"></a>Noter les processus dans votre espace de travail en fonction du nombre de ports qu'ils ont ouvert

```kusto
VMBoundPort
| where Ip != "127.0.0.1"
| summarize by ProcessName, Port, Protocol
| summarize OpenPorts=count() by ProcessName
| order by OpenPorts desc
```

### <a name="aggregate-behavior-for-each-port"></a>Agréger le comportement de chaque port

Cette requête permet ensuite de noter les ports par activité (par exemple, les ports avec le plus de trafic entrant/sortant, les ports avec le plus de connexions)
```kusto
// 
VMBoundPort
| where Ip != "127.0.0.1"
| summarize BytesSent=sum(BytesSent), BytesReceived=sum(BytesReceived), LinksEstablished=sum(LinksEstablished), LinksTerminated=sum(LinksTerminated), arg_max(TimeGenerated, LinksLive) by Machine, Computer, ProcessName, Ip, Port, IsWildcardBind
| project-away TimeGenerated
| order by Machine, Computer, Port, Ip, ProcessName
```

### <a name="summarize-the-outbound-connections-from-a-group-of-machines"></a>Synthétiser les connexions sortantes d’un groupe de machines

```kusto
// the machines of interest
let machines = datatable(m: string) ["m-82412a7a-6a32-45a9-a8d6-538354224a25"];
// map of ip to monitored machine in the environment
let ips=materialize(VMComputer
| summarize ips=makeset(todynamic(Ipv4Addresses)) by MonitoredMachine=AzureResourceName
| mvexpand ips to typeof(string));
// all connections to/from the machines of interest
let out=materialize(VMConnection
| where Machine in (machines)
| summarize arg_max(TimeGenerated, *) by ConnectionId);
// connections to localhost augmented with RemoteMachine
let local=out
| where RemoteIp startswith "127."
| project ConnectionId, Direction, Machine, Process, ProcessName, SourceIp, DestinationIp, DestinationPort, Protocol, RemoteIp, RemoteMachine=Machine;
// connections not to localhost augmented with RemoteMachine
let remote=materialize(out
| where RemoteIp !startswith "127."
| join kind=leftouter (ips) on $left.RemoteIp == $right.ips
| summarize by ConnectionId, Direction, Machine, Process, ProcessName, SourceIp, DestinationIp, DestinationPort, Protocol, RemoteIp, RemoteMachine=MonitoredMachine);
// the remote machines to/from which we have connections
let remoteMachines = remote | summarize by RemoteMachine;
// all augmented connections
(local)
| union (remote)
//Take all outbound records but only inbound records that come from either //unmonitored machines or monitored machines not in the set for which we are computing dependencies.
| where Direction == 'outbound' or (Direction == 'inbound' and RemoteMachine !in (machines))
| summarize by ConnectionId, Direction, Machine, Process, ProcessName, SourceIp, DestinationIp, DestinationPort, Protocol, RemoteIp, RemoteMachine
// identify the remote port
| extend RemotePort=iff(Direction == 'outbound', DestinationPort, 0)
// construct the join key we'll use to find a matching port
| extend JoinKey=strcat_delim(':', RemoteMachine, RemoteIp, RemotePort, Protocol)
// find a matching port
| join kind=leftouter (VMBoundPort 
| where Machine in (remoteMachines) 
| summarize arg_max(TimeGenerated, *) by PortId 
| extend JoinKey=strcat_delim(':', Machine, Ip, Port, Protocol)) on JoinKey
// aggregate the remote information
| summarize Remote=makeset(iff(isempty(RemoteMachine), todynamic('{}'), pack('Machine', RemoteMachine, 'Process', Process1, 'ProcessName', ProcessName1))) by ConnectionId, Direction, Machine, Process, ProcessName, SourceIp, DestinationIp, DestinationPort, Protocol
```

## <a name="performance-records"></a>Enregistrements de performances
Les enregistrements de type *InsightsMetrics* contiennent des données de performances du système d’exploitation invité de la machine virtuelle. Les propriétés de ces enregistrements sont décrites dans le tableau suivant :


| Propriété | Description |
|:--|:--|
|TenantId | Identificateur unique de l’espace de travail |
|SourceSystem | *Insights* | 
|TimeGenerated | Heure à laquelle la valeur a été collectée (UTC) |
|Computer | Nom de domaine complet (FQDN) de l’ordinateur | 
|Origine | *vm.azm.ms* |
|Espace de noms | Catégorie du compteur de performances | 
|Nom | Nom du compteur de performances. |
|Val | Valeur collectée | 
|Balises | Détails relatifs à l’enregistrement. Consultez le tableau ci-dessous pour les balises utilisées avec différents types d’enregistrements.  |
|AgentID | Identificateur unique de l’agent de chaque ordinateur |
|Type | *InsightsMetrics* |
|_ResourceId_ | ID de ressource de la machine virtuelle |

Les compteurs de performance actuellement collectés dans la table *InsightsMetrics* sont répertoriés dans le tableau suivant :

| Espace de noms | Nom | Description | Unité | Balises |
|:---|:---|:---|:---|:---|
| Computer    | Heartbeat             | Pulsation de l’ordinateur                        | | |
| Mémoire      | AvailableMB           | Mémoire en octets disponible                    | Mo      | memorySizeMB - Taille totale de la mémoire|
| Réseau     | WriteBytesPerSecond   | Octets écrits sur le réseau par seconde            | BytesPerSecond | NetworkDeviceId - ID de l’appareil<br>bytes - Nombre total d’octets envoyés |
| Réseau     | ReadBytesPerSecond    | Octets lus sur le réseau par seconde             | BytesPerSecond | networkDeviceId - ID de l’appareil<br>bytes - Nombre total d’octets reçus |
| Processeur   | UtilizationPercentage | Pourcentage d’utilisation du processeur          | Pourcentage        | totalCpus - Nombre total de processeurs |
| LogicalDisk | WritesPerSecond       | Écritures par seconde sur le disque logique            | CountPerSecond | mountId - ID de montage de l’appareil |
| LogicalDisk | WriteLatencyMs        | Latence d’écriture sur le disque logique en millisecondes    | Millisecondes   | mountId - ID de montage de l’appareil |
| LogicalDisk | WriteBytesPerSecond   | Octets d’écriture sur le disque logique par seconde       | BytesPerSecond | mountId - ID de montage de l’appareil |
| LogicalDisk | TransfersPerSecond    | Transferts de disque logique par seconde         | CountPerSecond | mountId - ID de montage de l’appareil |
| LogicalDisk | TransferLatencyMs     | Latence du transfert de disque logique en millisecondes | Millisecondes   | mountId - ID de montage de l’appareil |
| LogicalDisk | ReadsPerSecond        | Lectures sur le disque logique par seconde             | CountPerSecond | mountId - ID de montage de l’appareil |
| LogicalDisk | ReadLatencyMs         | Latence de lecture sur le disque logique en millisecondes     | Millisecondes   | mountId - ID de montage de l’appareil |
| LogicalDisk | ReadBytesPerSecond    | Octets de lecture sur le disque logique par seconde        | BytesPerSecond | mountId - ID de montage de l’appareil |
| LogicalDisk | FreeSpacePercentage   | Pourcentage d’espace libre sur le disque logique        | Pourcentage        | mountId - ID de montage de l’appareil |
| LogicalDisk | FreeSpaceMB           | Espace libre du disque logique en octets             | Mo      | mountId - ID de montage de l’appareil<br>diskSizeMB - Taille totale du disque |
| LogicalDisk | BytesPerSecond        | Octets par seconde sur le disque logique             | BytesPerSecond | mountId - ID de montage de l’appareil |


## <a name="next-steps"></a>Étapes suivantes

* Si vous débutez dans l’écriture de requêtes de journal dans Azure Monitor, consultez [Comment utiliser Log Analytics](../log-query/log-analytics-tutorial.md) dans le portail Azure pour en savoir plus.

* Découvrez [l’écriture de requêtes de recherche](../log-query/get-started-queries.md).
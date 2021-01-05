---
title: Supervision d’Azure Blockchain Service (ABS)
description: Supervision d’Azure Blockchain Service avec Azure Monitor
ms.date: 01/08/2020
ms.topic: how-to
ms.reviewer: v-umha
ms.openlocfilehash: 7300a5dcfb0150e6182636dcb71bacfa68c787db
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87076926"
---
# <a name="monitor-azure-blockchain-service-through-azure-monitor"></a>Superviser Azure Blockchain Service avec Azure Monitor  

À partir du moment où les clients exécutent des scénarios blockchain en production sur Azure Blockchain Service (ABS), il devient indispensable de superviser les ressources en matière de disponibilité, de performances et d’opérations. Cet article décrit les données de supervision générées par Azure Blockchain Service ; il explique également comment utiliser les différentes fonctionnalités et intégrations d’Azure Monitor pour analyser et générer des alertes, afin de gérer les environnements de production.  

## <a name="what-is-azure-monitor"></a>Qu’est-ce qu’Azure Monitor ?

Azure Blockchain Service crée la supervision des données à l’aide d’Azure Monitor, un service de surveillance de pile complète dans Azure qui fournit un ensemble exhaustif de fonctionnalités permettant de superviser vos ressources Azure. Pour plus d’informations sur Azure Monitor, consultez [Supervision de ressources Azure avec Azure Monitor](../../azure-monitor/insights/monitor-azure-resource.md).
 

Les sections suivantes étayent cet article en décrivant les données spécifiques collectées à partir d’Azure Blockchain Service, et en fournissant des exemples pour configurer la collecte de données et analyser ces données avec des outils Azure.

## <a name="monitor-data-collected-from-azure-blockchain-service"></a>Superviser les données collectées à partir d’Azure Blockchain Service  

Azure Blockchain Service collecte le même type de données de supervision que les autres ressources Azure décrites dans [Données de supervision](../../azure-monitor/insights/monitor-azure-resource.md#monitoring-data) à partir de ressources Azure. Consultez [Superviser la référence des données Azure Blockchain Service](#monitor-azure-blockchain-service-data-reference) pour obtenir une référence détaillée des journaux et des métriques créés par Azure Blockchain Service.

Dans le portail Azure, la page de présentation de chaque ressource membre Azure Blockchain Service comprend un bref aperçu des transactions, notamment les requêtes gérées et les blocs traités. Certaines de ces données sont collectées automatiquement et disponibles à l’analyse dès lors que vous créez la ressource membre Azure Blockchain Service ; vous pouvez également activer la collecte de données complémentaires par une configuration supplémentaire.

## <a name="diagnostic-settings"></a>Paramètres de diagnostic  

Les métriques de la plateforme et le journal d’activité sont collectés automatiquement, mais vous devez créer un paramètre de diagnostic pour collecter les journaux des ressources ou les transférer en dehors d’Azure Monitor. Pour plus d’informations sur la création d’un paramètre de diagnostic à l’aide du portail Azure, de l’interface CLI ou de PowerShell, consultez [Créer un paramètre de diagnostic pour collecter des journaux et métriques de plateforme dans Azure](../../azure-monitor/platform/diagnostic-settings.md).

Lorsque vous créez un paramètre de diagnostic, vous spécifiez les catégories de journaux à collecter. Les catégories pour Azure Blockchain Service sont listées ci-dessous.

**Journaux de proxy blockchain** : sélectionnez la catégorie si vous souhaitez superviser les journaux du proxy NGNIX. Tous les détails des transactions clientes sont disponibles à des fins d’audit et de débogage.  

**Journaux d’application blockchain** : sélectionnez la catégorie pour récupérer les journaux de l’application blockchain hébergée par le service managé. Par exemple, pour un membre Quorum ABS, ces journaux sont les journaux du quorum lui-même.  

**Requêtes de métrique** : sélectionnez l’option pour collecter des données de métriques à partir d’Azure Cosmos DB et les acheminer vers les destinations du paramètre de diagnostic ; cette collecte est automatiquement effectuée dans Métriques Azure. Collectez les données de métriques avec les journaux de ressources pour analyser les deux types de données ensemble et envoyer des données de métrique en dehors d’Azure Monitor.

## <a name="analyze-metric-data"></a>Analyser les données de métriques  

Vous pouvez analyser les métriques pour Azure Blockchain Service avec Metrics Explorer ; accédez à l’onglet Métriques sous la section Surveillance du panneau des ressources ABS. Pour plus d’informations sur l’utilisation de l’outil, consultez [Bien démarrer avec Azure Metrics Explorer](../../azure-monitor/platform/metrics-getting-started.md). Les mesures complètes pour Azure Blockchain Service se trouvent dans les métriques standard de l’espace de noms Azure Blockchain Service.

Vous pouvez utiliser la dimension de **nœud** lors de l’ajout d’un filtre ou du fractionnement des métriques, ce qui fournit essentiellement des valeurs de métrique par nœuds de transaction et nœuds validateurs du membre ABS.

## <a name="analyze-log-data"></a>Analyser les données de journal

Voici quelques requêtes qu’il est possible d’entrer dans la zone Recherche dans les journaux. Elles vous aideront à superviser vos membres Azure Blockchain Service. Ces requêtes fonctionnent avec le [nouveau langage](../../azure-monitor/log-query/log-query-overview.md).

Pour interroger les conditions d’erreur dans les journaux d’application Blockchain, utilisez la requête ci-dessous :

```
BlockchainApplicationLog | where BlockchainMessage contains "ERROR" or BlockchainMessage contains "fatal"

```

Pour interroger les conditions d’erreur dans les journaux de proxy Blockchain, utilisez la requête ci-dessous :  


```
BlockchainProxyLog
| filter Code != 200
| limit 500

```
Vous pouvez utiliser les filtres de temps disponibles dans les journaux Azure pour filtrer la requête selon une période spécifique.

## <a name="monitor-azure-blockchain-service-data-reference"></a>Superviser la référence de données Azure Blockchain Service  

Cet article fournit une référence sur les données de journal et de métrique collectées pour analyser les performances et la disponibilité d’Azure Blockchain Service.  

### <a name="resource-logs"></a>Journaux d’activité de ressources

Tous les journaux de ressources partagent un schéma commun de niveau supérieur, avec quelques propriétés uniques propres au service blockchain. Vous pouvez vous référer à l’article [Schéma de niveau supérieur des journaux de ressources](../../azure-monitor/platform/resource-logs-schema.md#top-level-common-schema) ; les propriétés spécifiques d’Azure Blockchain Service sont présentées de manière détaillée ci-dessous.  

Le tableau suivant liste les propriétés des journaux de proxy Azure Blockchain lorsqu’elles sont collectées dans les journaux Azure Monitor ou le Stockage Azure.  


| Nom de la propriété  | Description |
|:---|:---|
| time | Date et heure (UTC) de l’opération. |
| resourceID  | Ressource Azure Blockchain Service pour laquelle les journaux sont activés.  |
| catégorie  |Pour Azure Blockchain Service, les valeurs possibles sont **Proxylogs** et **Applicationlogs**. |
| operationName  | Nom de l’opération représentée par cet événement.   |
| Log level  | Par défaut, Azure Blockchain Service active le niveau de journal **Information**.   |
| NodeLocation  | Région Azure dans laquelle le membre blockchain est déployé.  |
| BlockchainNodeName  | Nom du nœud du membre Azure Blockchain Service sur lequel l’opération est effectuée.   |
| EthMethod  | Méthode, qui est appelée par le protocole blockchain sous-jacent, dans Quorum ; peut être eth_sendTransactions, eth_getBlockByNumber, etc.  |
| Agent  | Agent utilisateur agissant pour le compte d’un utilisateur, tel que le navigateur web Mozilla, Edge, etc. Voici des exemples de valeurs : « Mozilla/5.0 (Linux x64) node.js/8.16.0 v8/6.2.414.77 »  |
| Code   | Codes d’erreur HTTP. En règle générale, 4XX et 5XX sont des conditions d’erreur.  |
| NodeHost  | Nom DNS du nœud.   |
| RequestMethodName | Méthode HTTP appelée, les valeurs possibles ici sont PUT pour créer un membre, GET pour obtenir les détails d’un membre existant, DELETE pour supprimer un membre, PATCH pour mettre à jour un membre.   |
| BlockchainMemberName  | Nom du membre Azure Blockchain Service fourni par l’utilisateur.  |
| Consortium | Nom du consortium tel que fourni par l’utilisateur.   |
| Remote  | Adresse IP du client où la requête est envoyée.  |
| RequestSize  | Taille de la requête en octets.  |
| RequestTime  | Durée de la requête en millisecondes.|




Le tableau suivant liste les propriétés des journaux d’application Azure Blockchain.


| Nom de la propriété  | Description |
|:---|:---|
| time | Date et heure (UTC) de l’opération. |
| resourceID  | Ressource Azure Blockchain Service pour laquelle les journaux sont activés.|
| catégorie  |Pour Azure Blockchain Service, les valeurs possibles sont **Proxylogs** et **Applicationlogs**.  |
| operationName  | Nom de l’opération représentée par cet événement.   |
| Log level  | Par défaut, Azure Blockchain Service active le niveau de journal **Information**.   |
| NodeLocation  | Région Azure dans laquelle le membre blockchain est déployé.  |
| BlockchainNodeName  | Nom du nœud du membre Azure Blockchain Service sur lequel l’opération est effectuée.   |
| BlockchainMessage    | Ce champ contient le journal d’application Blockchain qui correspond aux journaux standard de données. Pour Quorum ABS, il s’agirait des journaux Quorum. Les informations contenues renseignent sur le type d’entrée du journal (information, erreur, avertissement), et une chaîne fournit des informations supplémentaires sur l’action exécutée.   |
| ID de locataire    | Locataire spécifique à la région d’Azure Blockchain Service. Le format de ce champ est https://westlake-rp-prod.<region>. cloudapp.azure.com, dans lequel « region » précise la région Azure du membre déployé.       |
| SourceSystem   | Le système renseigne les journaux, dans ce cas, il s’agit d’**Azure**.    |



### <a name="metrics"></a>Mesures

Les tableaux suivants listent les métriques de plateforme collectées pour Azure Blockchain Service. Toutes les métriques se trouvent dans les métriques standard de l’espace de noms **Azure Blockchain Service**.

Pour obtenir la liste de toutes les métriques prises en charge par Azure Monitor (y compris Azure Blockchain Service), consultez [Métriques prises en charge avec Azure Monitor](../../azure-monitor/platform/metrics-supported.md).

### <a name="blockchain-metrics"></a>Métriques Blockchain

Le tableau suivant précise la liste des métriques Blockchain qui sont collectées pour la ressource membre Azure Blockchain Service.


| Nom de métrique | Unité  |  Type d’agrégation| Description   |
|---|---|---|---|
| Transactions en attente   | Count  |  Average | Nombre de transactions attendant d’être explorées.   |
| Blocs traités   | Count  | SUM  |  Nombre de blocs traités dans chaque intervalle de temps. Actuellement, la taille d’un bloc étant de 5 secondes, en une minute chaque nœud traite 12 blocs, et 60 blocs en 5 minutes.   |
|Transactions traitées    | Count  | SUM  | Nombre de transactions traitées dans un bloc.    |
|Transactions en file d’attente    |  Count | Average  | Nombre de transactions qui ne peuvent pas être immédiatement exploitées. Cela peut être dû au fait qu’elles sont arrivées dans le désordre, et que la prochaine transaction attend la réception de la transaction précédente. Ou bien, il peut y avoir deux transactions qui ont le même numéro utilisé une seule fois (nonce), et la même valeur de gaz, si bien que la seconde transaction ne peut pas être exploitée.   |

### <a name="connection-metrics"></a>Métriques de connexion  

Le tableau suivant liste les différentes métriques de connexion collectées pour la ressource membre Azure Blockchain Service. Il s’agit de métriques de proxy NGINX.


| Nom de métrique | Unité  |  Type d’agrégation| Description |
|---|---|---|---|
| Connexions acceptées   | Count  |  SUM | Nombre total de connexions clientes acceptées.   |
| Connexions actives  | Count  | Average  |  Nombre actuel de connexions clientes actives, comprenant les connexions en attente.    |
|Connexions gérées    | Count  | SUM  | Nombre total de connexions gérées. En règle générale, la valeur du paramètre est identique à celle des connexions acceptées, sauf si certaines limites de ressources ont été atteintes.     |
|Requêtes traitées     |  Count | SUM  | Nombre total de requêtes clientes.  |


### <a name="performance-metrics"></a>Mesures de performances

Le tableau suivant liste les métriques de performances qui sont collectées pour chacun des nœuds de la ressource membre Azure Blockchain.  


| Nom de métrique | Unité  |  Type d’agrégation| Description   |
|---|---|---|---|
| Pourcentage d’utilisation du processeur   | Pourcentage  |  Max | Pourcentage d’utilisation du processeur.     |
| E/S de lecture d’octets   | Ko   | SUM  |  Somme des entrées/sorties d’octets lus sur tous les nœuds de la ressource membre blockchain.      |
|E/S d’écriture d’octets     | Ko   | SUM  | Somme des entrées/sorties d’octets écrits sur tous les nœuds de la ressource membre blockchain.     |
|Limite de mémoire       |  Gigaoctets   | Average    | Mémoire maximale disponible pour le processus blockchain par nœud. |
|Utilisation de la mémoire     | Gigaoctets  |  Average | Quantité de mémoire utilisée en moyenne sur tous les nœuds.  |
| Pourcentage d’utilisation de la mémoire     | Pourcentage   | Average  |  Pourcentage de la mémoire utilisée en moyenne sur tous les nœuds.       |
|Utilisation du stockage      | Gigaoctets   | Average  | Quantité en Go de stockage utilisé en moyenne sur tous les nœuds.       |


## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur [Blockchain Data Manager](./data-manager.md) pour capturer et transformer des données blockchain à destination d’Azure Event Grid.

---
title: Accès de l’indexeur aux ressources protégées
titleSuffix: Azure Cognitive Search
description: Vue d’ensemble conceptuelle des options de sécurité au niveau du réseau pour l’accès aux données Azure par les indexeurs dans Recherche cognitive Azure.
manager: nitinme
author: arv100kri
ms.author: arjagann
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 10/14/2020
ms.openlocfilehash: bcb6e91bba367363385214806077146b1a24fe7b
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92503485"
---
# <a name="indexer-access-to-content-protected-by-azure-network-security-features-azure-cognitive-search"></a>Accès de l’indexeur au contenu protégé par les fonctionnalités de sécurité réseau Azure (Recherche cognitive Azure)

Les indexeurs Recherche cognitive Azure peuvent effectuer des appels sortants vers différentes ressources Azure lors de l’exécution. Cet article explique les concepts liés à l’accès de l’indexeur au contenu protégé par des pare-feu IP, des points de terminaison privés et d’autres mécanismes de sécurité au niveau du réseau Azure. Un indexeur effectue des appels sortants dans deux situations : la connexion à des sources de données pendant l’indexation et la connexion à du code encapsulé via un ensemble de compétences. Une liste de tous les types de ressources auxquels un indexeur peut accéder dans le cadre d’une exécution classique est répertoriée dans le tableau ci-dessous.

| Ressource | Objectif au sein de l’exécution de l’indexeur |
| --- | --- |
| Stockage Azure (objets blob, tables, ADLS Gen 2) | Source de données |
| Stockage Azure (objets blob, tables) | Ensembles de compétences (mise en cache de documents enrichis et stockage des projections de la base de connaissances) |
| Azure Cosmos DB (différentes API) | Source de données |
| Azure SQL Database | Source de données |
| SQL Server sur les machines virtuelles Azure | Source de données |
| Instance managée SQL | Source de données |
| Azure Functions | Hôte pour les compétences API web personnalisées |
| Cognitive Services | Joints aux compétences qui seront utilisées pour facturer l’enrichissement au-delà de la limite de 20 documents gratuits |

> [!NOTE]
> La ressource Cognitive Services jointe à un ensemble de compétences est utilisée pour la facturation, selon les enrichissements effectués et écrits dans l’index de recherche. Elle n’est pas utilisée pour accéder aux API Cognitive Services. L’accès du pipeline d’enrichissement d’indexeur aux API Cognitive Services intervient via un canal de communication interne sécurisé, où les données sont fortement chiffrées en transit et ne sont jamais stockées au repos.

Les clients peuvent sécuriser ces ressources via plusieurs mécanismes d’isolement réseau proposés par Azure. À l’exception d’une ressource Cognitive Services, les indexeurs ont une capacité d’accès limitée aux autres ressources, même si elles sont isolées du réseau, comme indiqué dans le tableau ci-dessous.

| Ressource | Restriction d’adresse IP | Point de terminaison privé |
| --- | --- | ---- |
| Stockage Azure (objets blob, tables, ADLS Gen 2) | Pris en charge uniquement si le compte de stockage et le service de recherche se trouvent dans des régions différentes | Prise en charge |
| Azure Cosmos DB - API SQL | Prise en charge | Prise en charge |
| Azure Cosmos DB - API Cassandra, Mongo et Gremlin | Prise en charge | Non pris en charge |
| Azure SQL Database | Prise en charge | Prise en charge |
| SQL Server sur les machines virtuelles Azure | Prise en charge | N/A |
| Instance managée SQL | Prise en charge | N/A |
| Azure Functions | Prise en charge | Pris en charge uniquement pour certains niveaux d’Azure Functions |

> [!NOTE]
> En plus des options répertoriées ci-dessus, pour les comptes Stockage Azure sécurisés sur le réseau, les clients peuvent tirer parti du fait que la Recherche cognitive Azure est un [service Microsoft approuvé](../storage/common/storage-network-security.md#trusted-microsoft-services). Cela signifie qu’un service de recherche spécifique peut contourner les restrictions de réseau virtuel ou d’adresse IP sur le compte de stockage et accéder aux données du compte de stockage moyennant l’activation du contrôle d’accès en fonction du rôle sur ce compte. Pour plus d’informations, consultez [Connexions d’indexeur via une exception de service approuvé](search-indexer-howto-access-trusted-service-exception.md). Cette option peut être privilégiée par rapport à l’itinéraire de restriction d’adresse IP, s’il est impossible de déplacer le compte de stockage ou le service de recherche vers une autre région.

Lorsque vous choisissez le mécanisme d’accès sécurisé qu’un indexeur doit utiliser, prenez en compte les contraintes suivantes :

- Un indexeur ne peut pas se connecter à un [point de terminaison de service de réseau virtuel](../virtual-network/virtual-network-service-endpoints-overview.md). Les points de terminaison publics avec des informations d’identification, des points de terminaison privés, un service approuvé et un adressage IP sont les seules méthodologies prises en charge pour les connexions d’indexeur.
- Un service de recherche ne peut pas être approvisionné dans un réseau virtuel spécifique, car il s’exécute en mode natif sur une machine virtuelle. Cette fonctionnalité n’est pas offerte par Recherche cognitive Azure.
- Lorsque les indexeurs utilisent des points de terminaison privés (sortants) pour accéder aux ressources, des [frais de liaison privée](https://azure.microsoft.com/pricing/details/search/) supplémentaires peuvent s’appliquer.

## <a name="indexer-execution-environment"></a>Environnement d’exécution des indexeurs

Les indexeurs Recherche cognitive Azure peuvent extraire efficacement du contenu à partir de sources de données, en ajoutant des enrichissements au contenu extrait, voire en générant des projections avant d’écrire les résultats dans l’index de recherche. Selon le nombre de responsabilités attribuées à un indexeur, il peut s’exécuter dans l’un des deux environnements suivants :

- Environnement privé pour un service de recherche spécifique. Les indexeurs qui s’exécutent dans de tels environnements partagent des ressources avec d’autres charges de travail (par exemple, l’indexation initiée par le client ou l’interrogation de charges de travail). En général, seuls les indexeurs qui effectuent une indexation textuelle (par exemple, n’utilisant pas d’ensemble de compétences) s’exécutent dans cet environnement.
- Environnement multilocataire hébergeant des indexeurs gourmands en ressources, comme ceux ayant des ensembles de compétences. Cet environnement est utilisé pour décharger le traitement gourmand en calculs, en laissant les ressources spécifiques au service disponibles pour les opérations de routine. Cet environnement multilocataire est géré et sécurisé par Microsoft, sans frais supplémentaires pour le client.

Recherche cognitive Azure détermine l’environnement le plus adapté à l’exécution de d’un indexeur donné. Si vous utilisez un pare-feu IP pour contrôler l’accès aux ressources Azure, la connaissance des environnements d’exécution vous aidera à configurer une plage d’adresses IP qui inclut les deux.

## <a name="granting-access-to-indexer-ip-ranges"></a>Octroi d’accès aux plages d’adresses IP de l’indexeur

Si la ressource à laquelle votre indexeur tente d’accéder est limitée à un certain nombre de plages d’adresses IP, vous devez développer l’ensemble pour inclure les plages d’adresses IP à partir desquelles une demande d’indexeur peut provenir. Comme indiqué ci-dessus, il existe deux environnements possibles dans lesquels les indexeurs s’exécutent et à partir desquels les demandes d’accès peuvent provenir. Vous devez ajouter les adresses IP des **deux environnements** pour permettre le bon fonctionnement de l’indexeur.

- Pour obtenir l’adresse IP de l’environnement privé spécifique du service de recherche, `nslookup` (ou `ping`) le nom de domaine complet (FQDN) de votre service de recherche. Par exemple, le nom de domaine complet d’un service de recherche dans le cloud public peut être `<service-name>.search.windows.net`. Ces informations sont disponibles sur le portail Azure.
- Les adresses IP des environnements multilocataires sont disponibles via la balise de service `AzureCognitiveSearch`. Les [balises de service Azure](../virtual-network/service-tags-overview.md) disposent d’une plage d’adresses IP publiée pour chaque service, ce qui est possible via une [API de détection (préversion)](../virtual-network/service-tags-overview.md#use-the-service-tag-discovery-api-public-preview) ou un [fichier JSON téléchargeable](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files). Dans les deux cas, les plages d’adresses IP sont décomposées par région. Vous pouvez choisir uniquement les plages d’adresses IP attribuées à la région dans laquelle votre service de recherche est approvisionné.

Pour certaines sources de données, la balise de service elle-même peut être utilisée directement plutôt que d’énumérer la liste de plages d’adresses IP (l’adresse IP du service de recherche doit toujours être utilisée explicitement). Ces sources de données restreignent l’accès au moyen de la configuration d’une [règle de groupe de sécurité réseau](../virtual-network/network-security-groups-overview.md), qui prend en charge l’ajout d’une étiquette de service en mode natif, à la différence des règles IP telles que celles proposées par Stockage Azure, Cosmos DB, Azure SQL, etc. Les sources de données qui prennent en charge l’utilisation de l’étiquette de service `AzureCognitiveSearch` directement en plus des adresses IP du service de recherche sont les suivantes :

- [SQL Server sur les machines virtuelles Azure](./search-howto-connecting-azure-sql-iaas-to-azure-search-using-indexers.md#restrict-access-to-the-azure-cognitive-search)

- [Instances managées SQL](./search-howto-connecting-azure-sql-mi-to-azure-search-using-indexers.md#verify-nsg-rules)

Pour plus d’informations sur cette option de connectivité, consultez [Connexions d’indexeur via un pare-feu IP](search-indexer-howto-access-ip-restricted.md).

## <a name="granting-access-via-private-endpoints"></a>Octroi de l’accès via des points de terminaison privés

Les indexeurs peuvent utiliser [des points de terminaison privés](../private-link/private-endpoint-overview.md) afin d’accéder à des ressources dont l’accès est verrouillé pour sélectionner des réseaux virtuels ou pour lesquels aucun accès public n’est activé.
Cette fonctionnalité est réservée aux services de recherche facturables, avec des limites sur le nombre de points de terminaison privés créés. Pour plus d’informations, consultez [Limites du service](search-limits-quotas-capacity.md#shared-private-link-resource-limits).

### <a name="step-1-create-a-private-endpoint-to-the-secure-resource"></a>Étape 1 : Créer un point de terminaison privé vers la ressource sécurisée

Les clients doivent appeler l’opération de gestion de la recherche, l’ [API CreateOrUpdate](/rest/api/searchmanagement/sharedprivatelinkresources/createorupdate) sur une **ressource de liaison privée partagée** , afin de créer une connexion de point de terminaison privé à leur ressource sécurisée (par exemple, un compte de stockage). Le trafic qui transite via cette connexion de point de terminaison privé (sortant) provient uniquement du réseau virtuel situé dans l’environnement d’exécution de l’indexeur « privé » spécifique du service de recherche.

Recherche cognitive Azure vérifie que les appelants de cette API disposent des autorisations Azure RBAC pour approuver les demandes de connexion de point de terminaison privé à la ressource sécurisée. Par exemple, si vous demandez une connexion de point de terminaison privé à un compte de stockage avec des autorisations en lecture seule, cet appel sera rejeté.

### <a name="step-2-approve-the-private-endpoint-connection"></a>Étape 2 : Approuver la connexion Private Endpoint

Lorsque l’opération (asynchrone) qui crée une ressource de lien privé partagé est terminée, une connexion de point de terminaison privé est créée avec un état « en attente ». Aucun trafic n’est actuellement transmis via la connexion.
Le client est censé localiser cette demande sur sa ressource sécurisée et l’approuver. En règle générale, cette opération peut être effectuée via le portail Azure ou l’[API REST](/rest/api/virtualnetwork/privatelinkservices/updateprivateendpointconnection).

### <a name="step-3-force-indexers-to-run-in-the-private-environment"></a>Étape 3 : Forcer l’exécution des indexeurs dans l'environnement privé

Un point de terminaison privé approuvé permet les appels sortants entre le service de recherche et une ressource présentant des restrictions d’accès au niveau du réseau (par exemple, une source de données de compte de stockage configurée pour être uniquement accessible à partir de certains réseaux virtuels).
Cela signifie que tout indexeur capable d’atteindre une telle source de données sur le point de terminaison privé aboutira.
Si le point de terminaison privé n’est pas approuvé ou si l’indexeur n’utilise pas la connexion au point de terminaison privé, l’exécution de l’indexeur se terminera par `transientFailure`.

Pour permettre aux indexeurs d’accéder aux ressources par le biais de connexions de point de terminaison privé, il est impératif de définir la valeur `executionEnvironment` de l’indexeur sur `"Private"` afin de veiller à ce que toutes les exécutions de cet indexeur utilisent le point de terminaison privé et ce, en raison du fait que les points de terminaison privés sont approvisionnés dans l’environnement spécifique du service de recherche privé.

```json
    {
      "name" : "myindexer",
      ... other indexer properties
      "parameters" : {
          ... other parameters
          "configuration" : {
            ... other configuration properties
            "executionEnvironment": "Private"
          }
        }
    }
```

Ces étapes sont décrites plus en détail dans [Connexions d’indexeur via un point de terminaison privé](search-indexer-howto-access-private.md).
Lorsque vous disposez d’un point de terminaison privé approuvé pour une ressource, les indexeurs définis pour être *privés* tentent d’obtenir l’accès via la connexion au point de terminaison privé.

### <a name="limits"></a>Limites

À des fins de performances et de stabilité optimales du service de recherche, des restrictions sont imposées (par niveau de service de recherche) comme suit :

- Types d’indexeurs pouvant être définis comme *privés* .
- Nombre de ressources de lien privé partagé pouvant être créées.
- Nombre de types de ressources distincts pour lesquels des ressources de lien privé partagé peuvent être créées.

Ces limites sont documentées dans [Limites de service](search-limits-quotas-capacity.md).

## <a name="next-steps"></a>Étapes suivantes

- [Connexions d’indexeur via des pare-feu IP](search-indexer-howto-access-ip-restricted.md)
- [Connexions d’indexeur via une exception de service approuvé](search-indexer-howto-access-trusted-service-exception.md)
- [Connexions d’indexeur à un point de terminaison privé](search-indexer-howto-access-private.md)
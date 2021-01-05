---
title: Résoudre les problèmes courants des indexeurs de recherche
titleSuffix: Azure Cognitive Search
description: Résolvez les problèmes courants liés aux indexeurs dans la Recherche cognitive Azure, notamment les erreurs de connexion aux sources de données, de pare-feu et de documents manquants.
manager: nitinme
author: mgottein
ms.author: magottei
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
ms.openlocfilehash: 7eadc9121c54b636fa8b42579284d4018043e1c1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91355123"
---
# <a name="troubleshooting-common-indexer-issues-in-azure-cognitive-search"></a>Résoudre les problèmes courants des indexeurs dans la Recherche cognitive Azure

Les indexeurs peuvent rencontrer un certain nombre de problèmes lors de l’indexation des données dans la Recherche cognitive Azure. Voici les principales catégories de défaillances :

* [Connexion à une source de données ou à d’autres ressources](#connection-errors)
* [Traitement de documents](#document-processing-errors)
* [Ingestion des documents dans un index](#index-errors)

## <a name="connection-errors"></a>Erreurs de connexion

> [!NOTE]
> Les indexeurs ont une prise en charge limitée pour accéder aux sources de données et à d’autres ressources sécurisées par les mécanismes de sécurité réseau Azure. Actuellement, les indexeurs peuvent accéder uniquement aux sources de données par le biais de mécanismes de restriction de plage d’adresses IP ou de règles NSG correspondantes, le cas échéant. Vous trouverez ci-dessous des détails sur l’accès à chaque source de données prise en charge.
>
> Vous trouverez l’adresse IP de votre service Search en effectuant un test ping de son nom de domaine complet (p. ex., `<your-search-service-name>.search.windows.net`).
>
> Vous pouvez trouver la plage d’adresses IP de `AzureCognitiveSearch` [l’étiquette de service](../virtual-network/service-tags-overview.md#available-service-tags) en utilisant des [fichiers JSON téléchargeables](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files) ou via [l’API de détection d’étiquettes de service](../virtual-network/service-tags-overview.md#use-the-service-tag-discovery-api-public-preview). La plage d’adresses IP est mise à jour chaque semaine.

### <a name="configure-firewall-rules"></a>Configurer les règles de pare-feu

Stockage Azure, CosmosDB et Azure SQL fournissent un pare-feu configurable. Il n’y a pas de message d’erreur spécifique lorsqu’il est activé. En règle générale, les erreurs de pare-feu sont génériques et se présentent comme `The remote server returned an error: (403) Forbidden` ou `Credentials provided in the connection string are invalid or have expired`.

Vous pouvez utiliser deux options pour autoriser les indexeurs à accéder à ces ressources dans une telle instance :

* Désactiver le pare-feu en choisissant d’autoriser l’accès de **Tous les réseaux** (si possible).
* Vous pouvez également autoriser l’accès à l’adresse IP de votre service Search et à la plage d’adresses IP de `AzureCognitiveSearch` [l’étiquette de service](../virtual-network/service-tags-overview.md#available-service-tags) dans les règles de pare-feu de votre ressource (restriction de plage d’adresses IP).

Pour plus d’informations sur la configuration des restrictions de plage d’adresses IP pour chaque type de source de données, consultez les liens suivants :

* [Stockage Azure](../storage/common/storage-network-security.md#grant-access-from-an-internet-ip-range)

* [Cosmos DB](../storage/common/storage-network-security.md#grant-access-from-an-internet-ip-range)

* [Azure SQL](../azure-sql/database/firewall-configure.md#create-and-manage-ip-firewall-rules)

**Limitation** : Comme indiqué dans la documentation ci-dessus pour le stockage Azure, les restrictions de plage d’adresses IP ne fonctionnent que si votre service Search et votre compte de stockage se trouvent dans des régions différentes.

Azure Functions (qui peut être utilisé comme une [compétence API web personnalisée](cognitive-search-custom-skill-web-api.md)) prend également en charge les [restrictions d’adresse IP](../azure-functions/ip-addresses.md#ip-address-restrictions). La liste d’adresses IP à configurer correspond à l’adresse IP de votre service Search et à la plage d’adresses IP de `AzureCognitiveSearch` l’étiquette de service.

Pour plus d’informations sur l’accès aux données dans SQL Server sur une machine virtuelle Azure, cliquez [ici](search-howto-connecting-azure-sql-iaas-to-azure-search-using-indexers.md)

### <a name="configure-network-security-group-nsg-rules"></a>Configurer les règles du groupe de sécurité réseau (NSG)

Lors de l’accès aux données d’une instance gérée SQL, ou lorsqu’une machine virtuelle Azure est utilisée comme URI de service we pour une [compétence API web personnalisée](cognitive-search-custom-skill-web-api.md), les clients n’ont pas à se préoccuper des adresses IP spécifiques.

Dans ce cas, la machine virtuelle Azure ou l’instance gérée SQL peut être configurée pour résider dans un réseau virtuel. Ensuite, le groupe de sécurité réseau peut être configuré pour filtrer le type de trafic réseau pouvant circuler vers et depuis les interfaces réseau et les sous-réseaux de réseau virtuel.

`AzureCognitiveSearch` L’étiquette de service peut être utilisée directement dans les [règles de groupe de sécurité réseau](../virtual-network/manage-network-security-group.md#work-with-security-rules) entrantes sans avoir besoin de rechercher sa plage d’adresses IP.

Pour plus d’informations sur l’accès aux données dans une instance managée SQL, cliquez [ici](search-howto-connecting-azure-sql-mi-to-azure-search-using-indexers.md)

### <a name="cosmosdb-indexing-isnt-enabled"></a>« L’indexation » CosmosDB n’est pas activée

La Recherche cognitive Azure comporte une dépendance implicite vis-à-vis de l’indexation Cosmos DB. Si l’indexation automatique est désactivée dans Cosmos DB, la Recherche cognitive Azure renvoie un état réussi, mais ne parvient pas à indexer le contenu du conteneur. Pour savoir comment vérifier les paramètres et activer l’indexation, voir [Gérer l’indexation dans Azure Cosmos DB](../cosmos-db/how-to-manage-indexing-policy.md#use-the-azure-portal).

## <a name="document-processing-errors"></a>Erreurs de traitement de documents

### <a name="unprocessable-or-unsupported-documents"></a>Documents non exploitables ou non pris en charge

L’indexeur d’objets blob [précise quels formats de documents sont pris en charge explicitement](search-howto-indexing-azure-blob-storage.md#SupportedFormats). Il peut arriver qu’un conteneur de stockage blob contienne des documents non pris en charge ou bien problématiques. Pour ne pas avoir à arrêter votre indexeur sur ces documents, vous pouvez [modifier les options de configuration](search-howto-indexing-azure-blob-storage.md#DealingWithErrors) :

```
PUT https://[service name].search.windows.net/indexers/[indexer name]?api-version=2020-06-30
Content-Type: application/json
api-key: [admin key]

{
  ... other parts of indexer definition
  "parameters" : { "configuration" : { "failOnUnsupportedContentType" : false, "failOnUnprocessableDocument" : false } }
}
```

### <a name="missing-document-content"></a>Contenu de document manquant

L’indexeur d’objets blob [recherche et extrait du texte dans les objets blob d’un conteneur](search-howto-indexing-azure-blob-storage.md#how-azure-search-indexes-blobs). L’extraction de texte peut poser les problèmes suivants :

* Le document ne contient que des images numérisées. Les objets blob PDF comportant du contenu non textuel, comme des images numérisées (JPG), ne produisent pas de résultats dans un pipeline d’indexation blob standard. Si vous avez du contenu d’image comportant des éléments textuels, vous pouvez utiliser la [recherche cognitive](cognitive-search-concept-image-scenarios.md) pour rechercher et extraire le texte.
* L’indexeur d’objets blob est configuré pour indexer uniquement les métadonnées. Pour extraire le contenu, il doit être configuré de façon à [extraire à la fois le contenu et les métadonnées](search-howto-indexing-azure-blob-storage.md#PartsOfBlobToIndex) :

```
PUT https://[service name].search.windows.net/indexers/[indexer name]?api-version=2020-06-30
Content-Type: application/json
api-key: [admin key]

{
  ... other parts of indexer definition
  "parameters" : { "configuration" : { "dataToExtract" : "contentAndMetadata" } }
}
```

## <a name="index-errors"></a>Erreurs d’index

### <a name="missing-documents"></a>Documents manquants

Les indexeurs recherchent des documents dans une [source de données](/rest/api/searchservice/create-data-source). Parfois, il manque un document qui aurait dû être indexé. Ces erreurs peuvent se produire pour plusieurs raisons :

* Le document n’a pas été indexé. Consultez le portail pour une exécution réussie de l’indexeur.
* Vérifiez votre valeur de [suivi des modifications](/rest/api/searchservice/create-data-source#data-change-detection-policies). Si la valeur de la limite supérieure est une date future, tous les documents dont la date est antérieure à celle-ci sont ignorés par l’indexeur. Vous pouvez comprendre l’état de suivi des modifications de votre indexeur à l’aide des champs « initialTrackingState » et « finalTrackingState » dans l’[état de l’indexeur](/rest/api/searchservice/get-indexer-status#indexer-execution-result).
* Le document a été mis à jour après l’exécution de l’indexeur. Si votre indexeur suit une [planification](/rest/api/searchservice/create-indexer#indexer-schedule), il s’exécutera à nouveau et trouvera le document.
* La [requête](/rest/api/searchservice/create-data-source) spécifiée dans la source de données exclut le document. Les indexeurs ne peuvent pas indexer de documents qui ne font pas partie de la source de données.
* Des [mappages de champs](/rest/api/searchservice/create-indexer#fieldmappings) ou l’[enrichissement de l’IA](./cognitive-search-concept-intro.md) ont modifié le document, qui n’a pas l’aspect prévu.
* Utilisez [l’API Recherche de document](/rest/api/searchservice/lookup-document) pour trouver votre document.
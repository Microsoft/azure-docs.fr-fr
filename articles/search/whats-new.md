---
title: Nouveauté dans Recherche cognitive Azure
description: Annonces de fonctionnalités nouvelles et améliorées, notamment le changement de Recherche Azure en Recherche cognitive Azure.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: overview
ms.date: 11/12/2020
ms.custom: references_regions
ms.openlocfilehash: 73902cf53d9995f7dfc1991e3b7ed7c7c469bf2a
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95026659"
---
# <a name="whats-new-in-azure-cognitive-search"></a>Nouveauté dans Recherche cognitive Azure

Découvrir les nouveautés du service. Marquez cette page pour rester au fait des nouveautés du service.

## <a name="november-2020"></a>Novembre 2020

|Fonctionnalité&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  | Category | Description | Disponibilité  |
|------------------------------|----------|-------------|---------------|
|[Chiffrement à clé gérée par le client sur des indexeurs, des sources de données et des compétences](search-security-manage-encryption-keys.md) | Sécurité | Cet ajout étend le chiffrement géré par le client sur l’ensemble des ressources créées et gérées par un service de recherche. Rappelez-vous que la prise en charge des clés gérées par le client ajoute une couche de chiffrement supplémentaire au chiffrement de base effectué et managé par Microsoft. | Disponible en général à l’aide de la version d’API REST 2020-06-30 |  

## <a name="september-2020"></a>Septembre 2020

Créez une identité pour un service de recherche dans Azure Active Directory, puis utilisez les autorisations Azure RBAC pour accorder à l’identité les autorisations en lecture seule sur les sources de données Azure. Vous avez l’option de choisir la fonctionnalité d’[exception de service approuvé](search-indexer-howto-access-trusted-service-exception.md) si des règles IP doivent être appliquées.

|Fonctionnalité&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  | Category | Description | Disponibilité  |
|------------------------------|----------|-------------|---------------|
| [Identité MSI (Managed Service Identity)](search-howto-managed-identities-data-sources.md) | Indexeurs, sécurité | Créez une identité pour un service de recherche dans Azure Active Directory, puis utilisez les autorisations Azure RBAC pour accorder l’accès aux sources de données Azure. Cette approche élimine la nécessité d’utiliser des informations d’identification dans la chaîne de connexion. <br><br>Une identité MSI peut aussi être utilisée avec une [exception de service approuvé](search-indexer-howto-access-trusted-service-exception.md) si des règles IP doivent être appliquées. | En disponibilité générale. Accédez à cette fonctionnalité quand vous utilisez le portail ou [créez une source de données (REST)](/rest/api/searchservice/create-data-source) avec api-version=2020-06-30. |
| [Requêtes sortantes utilisant une liaison privée](search-indexer-howto-access-private.md) | Indexeurs, sécurité | Créez une ressource de liaison privée partagée que les indexeurs pourront utiliser pour accéder aux ressources Azure sécurisées par Azure Private Link. Pour plus d’informations sur toutes les méthodes de sécurisation des connexions des indexeurs, consultez [Accès de l’indexeur au contenu protégé par les fonctionnalités de sécurité réseau Azure](search-indexer-securing-resources.md). | En disponibilité générale. Accédez à cette fonctionnalité quand vous utilisez le portail ou la [ressource de liaison privée partagée](/rest/api/searchmanagement/sharedprivatelinkresources) avec api-version=2020-08-01. |
| [API REST Gestion (2020-08-01)](/rest/api/searchmanagement/management-api-versions) | REST | La nouvelle API REST stable autorise la création de ressources de liaison privée partagée. | En disponibilité générale. |
| [API REST Gestion (2020-08-01-Preview)](/rest/api/searchmanagement/management-api-versions) | REST | Ajoute une ressource de liaison privée partagée pour Azure Functions et les bases de données Azure SQL pour MySQL. | Préversion publique. |
| [SDK .NET de gestion 4.0](/dotnet/api/overview/azure/search/management) | Kit de développement logiciel (SDK) .NET | Mise à jour du SDK Azure pour le SDK de gestion, ciblant la version 2020-08-01 de l’API REST. | En disponibilité générale. |

## <a name="august-2020"></a>Août 2020

|Fonctionnalité&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  | Category | Description | Disponibilité  |
|---------|------------------|-------------|---------------|
| [double chiffrement](search-security-overview.md#encryption) | Sécurité | Activez le double chiffrement au niveau de la couche de stockage en configurant le chiffrement à clé gérée par le client sur les nouveaux services de recherche. Créez un service, [configurez et appliquez des clés gérées par le client](search-security-manage-encryption-keys.md) à des index ou à des cartes de synonymes, et bénéficiez du double chiffrement sur ce contenu. | En disponibilité générale sur tous les services de recherche créés après le 1er août 2020 dans les régions suivantes : USA Ouest 2, USA Est, USA Centre Sud, US Gov Virginie, US Gov Arizona. Utilisez le portail, les API REST de gestion ou les SDK pour créer le service. |

## <a name="july-2020"></a>Juillet 2020

|Fonctionnalité&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  | Category | Description | Disponibilité  |
|---------|------------------|-------------|---------------|
| [Bibliothèque cliente Azure.Search.Documents](/dotnet/api/overview/azure/search.documents-readme) | Kit SDK Azure pour .NET | Bibliothèque cliente .NET publiée par l’équipe du SDK Azure, conçue pour assurer la cohérence avec d’autres bibliothèques clientes .NET. <br/><br/>La version 11 cible l’API REST Recherche api-version=2020-06-30, mais ne prend pas encore en charge la base de connaissances ou les types géospatiaux. <br/><br/>Pour plus d’informations, consultez [Démarrage rapide : Créer un index](search-get-started-dotnet.md) et [Mettre à niveau vers Azure.Search.documents (v11)](search-dotnet-sdk-migration-version-11.md). | En disponibilité générale. </br> Installez le [package Azure.Search.Documents](https://www.nuget.org/packages/Azure.Search.Documents/) à partir de NuGet. |
| [Bibliothèque cliente azure.search.documents](/python/api/overview/azure/search-documents-readme)  | Kit SDK Azure pour Python| Bibliothèque cliente Python publiée par l’équipe du SDK Azure, conçue pour assurer la cohérence avec d’autres bibliothèques clientes Python. <br/><br/>La version 11 cible l’API REST Recherche version 2020-06-30. | En disponibilité générale. </br> Installez le [package azure-search-documents](https://pypi.org/project/azure-search-documents/) à partir de PyPI. |
| [Bibliothèque cliente @azure/search-documents](/javascript/api/overview/azure/search-documents-readme)  | Kit SDK Azure pour JavaScript | Bibliothèque cliente JavaScript publiée par l’équipe du SDK Azure, conçue pour assurer la cohérence avec d’autres bibliothèques clientes JavaScript. <br/><br/>La version 11 cible l’API REST Recherche version 2020-06-30. | En disponibilité générale. </br> Installez le [package @azure/search-documents](https://www.npmjs.com/package/@azure/search-documents) à partir de npm. |

## <a name="june-2020"></a>Juin 2020

|Fonctionnalité&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  | Category | Description | Disponibilité  |
|---------|------------------|-------------|---------------|
[**Base de connaissances**](knowledge-store-concept-intro.md) | Enrichissement par IA | Sortie d’un indexeur enrichi par IA, avec stockage du contenu dans Stockage Azure pour une utilisation dans d’autres applications et processus. | En disponibilité générale. </br> Utilisez l’[API REST Recherche 2020-06-30](/rest/api/searchservice/) ou version ultérieure, ou le portail. |
| [**API REST Recherche 2020-06-30**](/rest/api/searchservice/) | REST | Nouvelle version stable des API REST. En plus de la base de connaissances, cette version comprend des améliorations du scoring et de la pertinence des recherches. | En disponibilité générale. |
| [**Algorithme de pertinence Okapi BM25**](https://en.wikipedia.org/wiki/Okapi_BM25) | Requête | Nouvel algorithme de classement de pertinence utilisé automatiquement pour tous les nouveaux services de recherche créés après le 15 juillet. Pour les services créés plus tôt, vous pouvez choisir d’utiliser le nouvel algorithme en définissant la propriété `similarity` sur les champs d’index. | En disponibilité générale. </br> Utilisez l’[API REST Recherche 2020-06-30](/rest/api/searchservice/) ou l’API REST  2019-05-06. |
| **executionEnvironment** | Sécurité (indexeurs) | Affectez explicitement la valeur `private` à cette propriété de configuration d’indexeur pour forcer toutes les connexions à des sources de données externes sur un point de terminaison privé. S’applique uniquement aux services de recherche qui tirent parti d’Azure Private Link. | En disponibilité générale. </br> Utilisez l’[API REST Recherche 2020-06-30](/rest/api/searchservice/) pour définir ce paramètre de configuration générale. |

## <a name="may-2020-microsoft-build"></a>Mai 2020 (Microsoft Build)

|Fonctionnalité&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  | Category | Description | Disponibilité  |
|---------|------------------|-------------|---------------|
| [**Sessions de débogage**](cognitive-search-debug-session.md) | Enrichissement de l’IA | Les sessions de débogage offrent une interface sur basée sur le portail pour examiner et résoudre les problèmes liés à un ensemble de compétences existant. Les correctifs créés dans la session de débogage peuvent être enregistrés dans des ensembles de compétences de production. Pour bien démarrer, suivez [ce tutoriel](cognitive-search-tutorial-debug-sessions.md). | Préversion publique, dans le portail. |
| [**Règles IP pour la prise en charge de pare-feu entrant**](service-configure-firewall.md) | Sécurité | Limitez l’accès à un point de terminaison de service de recherche à des adresses IP spécifiques. | En disponibilité générale. </br> Utilisez l’[API REST Gestion 2020-03-13](/rest/api/searchmanagement/) ou version ultérieure, ou le portail. |
| [**Azure Private Link pour un point de terminaison de recherche privé**](service-create-private-endpoint.md) | Sécurité| Protégez un service de recherche de l’Internet public en l’exécutant en tant que ressource de liaison privée, accessible uniquement aux applications clientes et aux autres services Azure sur le même réseau virtuel. | En disponibilité générale. </br> Utilisez l’[API REST Gestion 2020-03-13](/rest/api/searchmanagement/) ou version ultérieure, ou le portail. |
| [**Identité managée par le système (préversion)**](search-howto-managed-identities-data-sources.md) | Sécurité (indexeurs) | Inscrivez un service de recherche en tant que service approuvé auprès d’Azure Active Directory afin de configurer des connexions aux sources de données Azure prises en charge pour l’indexation. S’applique aux [indexeurs](search-indexer-overview.md) qui ingèrent le contenu de sources de données Azure comme Azure SQL Database, Azure Cosmos DB et le Stockage Azure. | Préversion publique. </br> Utilisez le portail pour inscrire le service de recherche. |
| [**Paramètre de requête sessionId**](index-similarity-and-scoring.md), [Paramètre scoringStatistics=global](index-similarity-and-scoring.md#scoring-statistics) | Requête (pertinence) | Ajoutez sessionID à une requête afin d’établir une session pour le calcul des scores de recherche, avec scoringStatistics=global pour collecter des scores à partir de toutes les partitions, et bénéficiez ainsi de calculs de score de recherche plus cohérents. | En disponibilité générale. </br> Utilisez l’[API REST Recherche 2020-06-30](/rest/api/searchservice/) ou l’API REST  2019-05-06. |
| [**featuresMode (préversion)**](index-similarity-and-scoring.md#featuresMode-param) | Requête | Ajoutez ce paramètre de requête pour développer un score de pertinence afin d’afficher plus de détails : score de similarité par champ, fréquence des termes par champ et nombre de jetons uniques correspondants par champ. Vous pouvez consommer ces points de données dans des algorithmes de scoring personnalisés. Pour obtenir un exemple qui illustre cette fonctionnalité, consultez [Ajouter le machine learning (LearnToRank) pour améliorer la pertinence de la recherche](https://github.com/Azure-Samples/search-ranking-tutorial). | Préversion publique. </br> Utilisez l’[API REST Recherche 2020-06-30-Preview](/rest/api/searchservice/index-preview) ou l’API REST 2019-05-06-Preview. |

## <a name="march-2020"></a>Mars 2020

|Fonctionnalité&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  | Category | Description | Disponibilité  |
|---------|------------------|-------------|---------------|
| [**Suppression réversible native de blobs (préversion)**](search-howto-index-changed-deleted-blobs.md) | Indexeurs | Un indexeur Stockage Blob Azure dans Recherche cognitive Azure reconnaît les objets blob qui sont dans un état de suppression réversible, et supprime le document de recherche correspondant durant l’indexation. | Préversion publique. </br> Utilisez l’[API REST Recherche 2020-06-30-Preview](/rest/api/searchservice/index-preview) et l’API REST 2019-05-06-Preview, avec Exécuter l’indexeur sur une source de données d’objets blob Azure pour laquelle la « suppression réversible » native est activée. |
| [**API REST Gestion (2020-03-13)**](/rest/api/searchmanagement/management-api-versions) | REST | Nouvelle API REST stable pour la création et la gestion d’un service de recherche. Ajoute la prise en charge de pare-feu IP et de Private Link | En disponibilité générale. |

## <a name="february-2020"></a>Février 2020

|Fonctionnalité&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  | Category | Description | Disponibilité  |
|---------|------------------|-------------|---------------|
| [**Détection d’informations d’identification personnelle (préversion)**](cognitive-search-skill-pii-detection.md) | Enrichissement de l’IA | Nouvelle compétence cognitive utilisée lors de l’indexation, qui extrait les informations personnelles d’un texte d’entrée et vous donne la possibilité de les y masquer de différentes façons. | Préversion publique. </br> Utilisez le portail ou l’[API REST Recherche 2020-06-30-Preview](/rest/api/searchservice/index-preview) ou l’API REST 2019-05-06-Preview. |
| [**Recherche d’entité personnalisée (préversion)**](cognitive-search-skill-custom-entity-lookup.md )| Enrichissement par IA | Nouvelle compétence cognitive qui recherche du texte dans une liste personnalisée de mots et d’expressions définie par l’utilisateur. À l’aide de cette liste, elle étiquète tous les documents contenant des entités correspondantes. La compétence prend également en charge un degré de correspondance approximative qui peut être appliqué pour rechercher des correspondances similaires mais non exactes. | Préversion publique. </br> Utilisez le portail ou l’[API REST Recherche 2020-06-30-Preview](/rest/api/searchservice/index-preview) ou l’API REST 2019-05-06-Preview. |

## <a name="january-2020"></a>Janvier 2020

|Fonctionnalité&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  | Category | Description | Disponibilité  |
|---------|------------------|-------------|---------------|
| [**Clés de chiffrement gérées par le client**](search-security-manage-encryption-keys.md) |Sécurité | Ajoute une couche supplémentaire de chiffrement en plus du chiffrement intégré à la plateforme. À l’aide d’une clé de chiffrement que vous créez et gérez, vous pouvez chiffrer le contenu de l’index et les correspondances de synonymes avant que la charge utile n’atteigne un service de recherche. | En disponibilité générale. </br> Utilisez l’API REST Recherche 2019-05-06 ou une version ultérieure. Pour le code managé, le bon package est toujours la [version préliminaire 8.0 SDK .NET](search-dotnet-sdk-migration-version-9.md) même si la fonctionnalité n’est plus en préversion. |
| [**Règles IP pour la prise en charge de pare-feu entrant (préversion)**](service-configure-firewall.md) | Sécurité | Limitez l’accès à un point de terminaison de service de recherche à des adresses IP spécifiques. L’API en préversion comprend de nouvelles propriétés **IpRule** et **NetworkRuleSet** dans [API CreateOrUpdate](/rest/api/searchmanagement/2019-10-01-preview/createorupdate-service). Cette fonctionnalité d’évaluation est disponible dans les régions sélectionnées. |  Préversion publique utilisant api-version=2019-10-01-Preview.  |
| [**Azure Private Link pour un point de terminaison de recherche privé (préversion)**](service-create-private-endpoint.md) | Sécurité| Protégez un service de recherche de l’Internet public en l’exécutant en tant que ressource de liaison privée, accessible uniquement aux applications clientes et aux autres services Azure sur le même réseau virtuel. | Préversion publique utilisant api-version=2019-10-01-Preview.  |

## <a name="features-in-2019"></a>Fonctionnalités de 2019

### <a name="december-2019"></a>Décembre 2019

+ Dans le portail, [Créer une application de démonstration (préversion)](search-create-app-portal.md) est un nouvel Assistant qui génère un fichier HTML téléchargeable avec accès par requête (lecture seule) à un index. Le fichier est fourni avec un script incorporé qui restitue une application web opérationnelle de type « localhost », qui est liée à un index de votre service de recherche. Les pages sont configurables dans l’Assistant et peuvent contenir une barre de recherche, une zone de résultats, une navigation dans la barre latérale et la prise en charge des requêtes TypeAhead. Vous pouvez modifier le code HTML hors connexion pour étendre ou personnaliser le workflow ou l’apparence. Il est difficile d’étendre une application de démonstration pour qu’elle inclue les couches de sécurité et d’hébergement qui sont généralement nécessaires dans les scénarios de production. Vous devez la considérer comme un outil de validation et de test plutôt que comme un raccourci vers une application cliente complète.

+ La section [Créer un point de terminaison privé pour les connexions sécurisées (version préliminaire)](service-create-private-endpoint.md) explique comment configurer une liaison privée pour sécuriser les connexions de votre service Search. Cette fonctionnalité d’évaluation est disponible à la demande et utilise [Azure Private Link](../private-link/private-link-overview.md) et le [Réseau virtuel Azure](../virtual-network/virtual-networks-overview.md) dans le cadre de la solution.

### <a name="november-2019---ignite-conference"></a>Novembre 2019 - Conférence Ignite

+ [L’enrichissement incrémentiel (version préliminaire)](cognitive-search-incremental-indexing-conceptual.md) ajoute la mise en cache et l’état à un pipeline d’enrichissement afin que vous puissiez travailler sur des étapes ou des phases spécifiques sans perdre le contenu déjà traité. Auparavant, toute modification apportée à un pipeline d’enrichissement nécessitait une regénération complète. Grâce à l’enrichissement incrémentiel, la sortie de l’analyse coûteuse, en particulier l’analyse des images, est préservée.

<!-- 
+ Custom Entity Lookup is a cognitive skill used during indexing that allows you to provide a list of custom entities (such as part numbers, diseases, or names of locations you care about) that should be found within the text. It supports fuzzy matching, case-insensitive matching, and entity synonyms. -->

+ L’[Extraction de document (préversion)](cognitive-search-skill-document-extraction.md) est une compétence cognitive utilisée lors de l’indexation, qui vous permet d’extraire le contenu d’un fichier à partir d’un ensemble de compétences. Auparavant, le craquage de document avait exclusivement lieu avant l’exécution de l’ensemble de compétences. Avec l’ajout de cette compétence, vous pouvez également effectuer cette opération dans le cadre de l’exécution de l’ensemble de compétences.

+ La [Traduction de texte](cognitive-search-skill-text-translation.md) est une compétence cognitive utilisée pendant l’indexation, qui évalue le texte et, pour chaque enregistrement, retourne le texte traduit dans la langue cible spécifiée.

+ Les [Modèles Power BI](https://github.com/Azure-Samples/cognitive-search-templates/blob/master/README.md) peuvent lancer rapidement vos visualisations et l’analyse de contenu enrichi dans un magasin de connaissances dans Power BI Desktop. Ce modèle est conçu pour les projections de tables Azure créées à l’aide de l’[Assistant Importer des données](knowledge-store-create-portal.md).

+ [Azure Data Lake Storage Gen2 (préversion)](search-howto-index-azure-data-lake-storage.md), l’[API Gremlin de Cosmos DB (préversion)](search-howto-index-cosmosdb.md) et l’[API Cassandra de Cosmos DB (préversion)](search-howto-index-cosmosdb.md) sont désormais pris en charge dans les indexeurs. Vous pouvez vous inscrire avec [ce formulaire](https://aka.ms/azure-cognitive-search/indexer-preview). Vous recevez un e-mail de confirmation une fois que vous avez été accepté dans le programme de préversion.

### <a name="july-2019"></a>Juillet 2019

+ Mise à la disposition générale dans le [Cloud Azure Government](../azure-government/compare-azure-government-global-azure.md#azure-cognitive-search).

<a name="new-service-name"></a>

## <a name="new-service-name"></a>Nom du nouveau service

Recherche Azure est maintenant renommé  **Recherche cognitive Azure** pour refléter l’utilisation étendue (mais facultative) de compétences cognitives et du traitement par IA dans les opérations de base. Les versions d’API, les packages NuGet, les espaces de noms et les points de terminaison ne sont pas modifiés. Les solutions de recherche, nouvelles et existantes, ne sont pas affectées par le changement de nom du service.

## <a name="service-updates"></a>Mises à jour de service

[Les annonces de mise à jour de service](https://azure.microsoft.com/updates/?product=search&status=all) pour Recherche cognitive Azure sont disponibles sur le site web Azure.

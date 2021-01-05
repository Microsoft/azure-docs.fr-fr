---
title: Présentation de la sécurité
titleSuffix: Azure Cognitive Search
description: Découvrez les fonctionnalités de sécurité de Recherche cognitive Azure pour protéger les points de terminaison, le contenu et les opérations.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 12/15/2020
ms.custom: references_regions
ms.openlocfilehash: ffb5a78c13413a46565a9c57c87dc8273742fd24
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97563447"
---
# <a name="security-in-azure-cognitive-search---overview"></a>Sécurité dans Recherche cognitive Azure - Vue d’ensemble

Cet article décrit les principales fonctionnalités de sécurité de Recherche cognitive Azure qui peuvent protéger le contenu et les opérations.

+ Au niveau de la couche de stockage, le chiffrement au repos est intégré pour l’ensemble du contenu géré par le service enregistré sur le disque, notamment les index, les cartes de synonymes et les définitions d’indexeurs, de sources de données et d’ensembles de compétences. Recherche cognitive Azure prend également en charge l’ajout de clés gérées par le client (CMK) pour le chiffrement supplémentaire du contenu indexé. Pour les services créés après le 1er août 2020, le chiffrement CMK s’étend aux données sur les disques temporaires, pour le double chiffrement complet du contenu indexé.

+ La sécurité du trafic entrant protège le point de terminaison du service Recherche à des niveaux de sécurité plus élevés : depuis des clés API sur la demande à des règles de trafic entrant dans le pare-feu et à des points de terminaison privés qui protègent intégralement votre service de l’Internet public.

+ La sécurité du trafic sortant s’applique aux indexeurs qui extraient du contenu de sources externes. Pour les demandes sortantes, configurez une identité managée pour effectuer une recherche dans un service approuvé lors de l’accès aux données dans Stockage Azure, Azure SQL, Cosmos DB ou d’autres sources de données Azure. Une identité managée est un substitut pour les informations d’identification ou les clés d’accès sur la connexion. La sécurité du trafic sortant n’est pas abordée dans cet article. Pour plus d’informations sur cette fonctionnalité, consultez [Se connecter à une source de données en utilisant une identité managée](search-howto-managed-identities-data-sources.md).

Regardez cette vidéo rapide pour obtenir une vue d’ensemble de l’architecture de la sécurité et de chaque catégorie de fonctionnalités.

> [!VIDEO https://channel9.msdn.com/Shows/AI-Show/Azure-Cognitive-Search-Whats-new-in-security/player]

<a name="encryption"></a>

## <a name="encrypted-transmissions-and-storage"></a>Transmissions et stockage chiffrés

Dans Recherche cognitive Azure, le chiffrement commence aux connexions et aux transmissions, et s’étend au contenu stocké sur disque. Pour les services de recherche sur l’Internet public, Recherche cognitive Azure écoute sur le port HTTPS 443. Toutes les connexions client-à-service utilisent le chiffrement TLS 1.2. Les versions antérieures (1.0 et 1.1) ne sont pas prises en charge.

:::image type="content" source="media/search-security-overview/encryption-at-rest-cmk.png" alt-text="Diagramme illustrant différents types de sécurité à chaque niveau d’engagement de service":::

Pour les données gérées en interne par le service de recherche, le tableau suivant décrit les [modèles de chiffrement de données](../security/fundamentals/encryption-models.md). Certaines fonctionnalités, telles que la base de connaissances, l’enrichissement incrémentiel et l’indexation basée sur un indexeur, lisent ou écrivent dans des structures de données dans d’autres services Azure. Ces services disposent de leur propre niveau de prise en charge du chiffrement, distinct de Recherche cognitive Azure.

| Modèle | Clés&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | Spécifications&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | Restrictions | S’applique à |
|------------------|-------|-------------|--------------|------------|
| chiffrement côté serveur | Clés managées par Microsoft | Aucune (intégré) | Aucune, disponible pour tous les niveaux de service, dans toutes les régions, pour le contenu créé après le 24 janvier 2018. | Contenu (index et cartes de synonymes) et définitions (indexeurs, sources de données, ensembles de compétences) |
| chiffrement côté serveur | clés gérées par le client | Azure Key Vault | Disponible pour les niveaux de service facturables, dans toutes les régions, pour le contenu créé après janvier 2019. | Contenu (index et cartes de synonymes) sur les disques de données |
| double chiffrement côté serveur | clés gérées par le client | Azure Key Vault | Disponible sur les niveaux de service facturables, dans certaines régions, sur les services de recherche après le 1er août 2020. | Contenu (index et cartes de synonymes) sur les disques de données et les disques temporaires |

### <a name="service-managed-keys"></a>Clés gérées par le service

Le chiffrement géré par le service est une opération interne de Microsoft qui est basée sur [Azure Storage Service Encryption](../storage/common/storage-service-encryption.md) et qui utilise le [chiffrement AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) 256 bits. Il se produit automatiquement lors de toutes les indexations, y compris lors des mises à jour incrémentielles des index qui ne sont pas entièrement chiffrés (créés avant janvier 2018).

### <a name="customer-managed-keys-cmk"></a>Clés gérées par le client (CMK)

Les clés gérées par le client nécessitent un service facturable supplémentaire, Azure Key Vault, qui peut se trouver dans une autre région que l’instance Recherche cognitive Azure, mais qui doit être sous le même abonnement. L’activation du chiffrement CMK a pour effet d’augmenter la taille de l’index et dégrader les performances des requêtes. Sur la base des observations effectuées à ce jour, vous pouvez vous attendre à une augmentation de 30 à 60 % des temps de requête, même si les performances réelles varient en fonction de la définition d’index et des types de requêtes. En raison de cet incidence sur les performances, nous vous recommandons de n’activer cette fonctionnalité que sur les index qui en ont réellement besoin. Pour plus d’informations, consultez [Configurer des clés de chiffrement gérées par le client dans Recherche cognitive Azure](search-security-manage-encryption-keys.md).

<a name="double-encryption"></a>

### <a name="double-encryption"></a>Double chiffrement

Dans Recherche cognitive Azure, le double chiffrement est une extension de CMK. Il s’agit d’un chiffrement à deux reprises (une fois par CMK, et une nouvelle fois par des clés gérées par le service) et de portée globale, comprenant le stockage à long terme qui est écrit sur un disque de données et le stockage à court terme écrit sur des disques temporaires. La différence entre CMK avant et après le 1er août 2020, et ce qui fait de CMK une fonctionnalité de double chiffrement dans Recherche cognitive Azure, est le chiffrement supplémentaire de données au repos sur les disques temporaires.

Le double chiffrement est actuellement disponible sur les nouveaux services créés dans ces régions après le 1er août :

+ USA Ouest 2
+ USA Est
+ États-Unis - partie centrale méridionale
+ Gouvernement américain - Virginie
+ Gouvernement des États-Unis – Arizona

<a name="service-access-and-authentication"></a>

## <a name="inbound-security-and-endpoint-protection"></a>Sécurité du trafic entrant et protection des points de terminaison

Les fonctionnalités de sécurité du trafic entrant protègent le point de terminaison du service de recherche via des niveaux croissants de sécurité et de complexité. Premièrement, toutes les demandes nécessitent une clé API pour l’accès authentifié. Deuxièmement, vous avez la possibilité de définir des règles de pare-feu qui limitent l’accès à des adresses IP spécifiques. Pour une protection avancée, une troisième option consiste à activer Azure Private Link pour protéger votre point de terminaison de service de tout le trafic Internet.

### <a name="public-access-using-api-keys"></a>Accès public avec des clés API

Par défaut, un service de recherche est accessible via le cloud public, en utilisant une authentification basée sur des clés pour l’administration ou l’accès aux requêtes au point de terminaison du service de recherche. Une [clé API](search-security-rbac.md) est une chaîne composée de nombres et de lettres générée de manière aléatoire. Le type de clé (admin ou requête) détermine le niveau d’accès. La soumission d’une clé valide est considérée comme la preuve que la requête provient d’une entité approuvée.

Il existe deux niveaux d’accès à votre service de recherche, qui sont activés par les deux clés API suivantes :

+ Clé d’administration (autorise l’accès en lecture-écriture pour les opérations [créer - lire - mettre à jour - supprimer](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) sur le service de recherche)

+ Clé de requête (autorise l’accès en lecture seule à la collection documents d’un index)

Des *clés d’administration* sont créées une fois le service approvisionné. Bien qu’il existe deux clés d’administration, désignées comme *principale* et *secondaire*, celles-ci sont en fait interchangeables. Chaque service dispose de deux clés Admin que vous pouvez interchanger sans perdre l’accès à votre service. Vous pouvez [régénérer un clé d’administration](search-security-api-keys.md#regenerate-admin-keys) périodiquement conformément aux meilleures pratiques de sécurité Azure, mais ne pouvez pas en ajouter au nombre total de clés d’administration. Il y a, au maximum, deux clés d’administration par service de recherche.

Des *clés de requête* sont créées en fonction des besoins pour les applications clientes qui émettent des requêtes. Vous pouvez créer, au maximum, 50 clés de ce type. Dans le code d’application, vous spécifiez l’URL de recherche et une clé d’API de requête pour autoriser l’accès en lecture seule à la collection de documents d’un index spécifique. Ensemble, le point de terminaison, une clé API pour un accès en lecture seule et un index cible définissent le niveau de portée et d’accès de la connexion à partir de votre application cliente.

L’authentification est requise à chaque requête, chaque requête étant composée d’une clé obligatoire, d’une opération et d’un objet. Quand ils sont chaînés, les deux niveaux d’autorisation (complet ou en lecture seule) et le contexte (par exemple, une opération de requête sur un index) sont suffisants pour fournir une sécurité couvrant l’ensemble des opérations de service. Pour plus d’informations sur les clés, consultez [Créer et gérer des clés de l’api](search-security-api-keys.md).

### <a name="ip-restricted-access"></a>Accès restreint à des adresses IP

Pour contrôler davantage l’accès à votre service de recherche, vous pouvez créer des règles de pare-feu de trafic entrant qui autorisent l’accès à une adresse IP spécifique ou à une plage d’adresses IP. Toutes les connexions clientes doivent être effectuées via une adresse IP autorisée, sans quoi la connexion est refusée.

:::image type="content" source="media/search-security-overview/inbound-firewall-ip-restrictions.png" alt-text="Exemple de diagramme d’architecture pour l’accès restreint à des adresses IP":::

Vous pouvez utiliser le portail pour [configurer l’accès du trafic entrant](service-configure-firewall.md).

Vous pouvez aussi utiliser les API REST de gestion. À compter de la version 13-03-2020 de l’API avec le paramètre [IpRule](/rest/api/searchmanagement/services/createorupdate#iprule), vous pouvez limiter l’accès à votre service en identifiant les adresses IP, individuellement ou dans une plage, qui doivent pouvoir accéder à votre service de recherche.

### <a name="private-endpoint-no-internet-traffic"></a>Point de terminaison privé (pas de trafic Internet)

Un [point de terminaison privé](../private-link/private-endpoint-overview.md) pour Recherche cognitive Azure permet à un client d’un [réseau virtuel](../virtual-network/virtual-networks-overview.md) d’accéder de façon sécurisée aux données d’un index de recherche grâce à une [liaison privée](../private-link/private-link-overview.md).

Le points de terminaison privé utilise une adresse IP de l’espace d’adressage du réseau virtuel pour les connexions à votre service de recherche. Le trafic entre le client et le service Search traverse le réseau virtuel et une liaison privée sur le réseau principal de Microsoft, ce qui élimine l’exposition sur l’Internet public. Un réseau virtuel permet une communication sécurisée entre des ressources, avec votre réseau local ainsi qu’avec Internet.

:::image type="content" source="media/search-security-overview/inbound-private-link-azure-cog-search.png" alt-text="Exemple de diagramme d’architecture pour l’accès au point de terminaison privé":::

Bien que cette solution soit la plus sécurisée, l’utilisation de services supplémentaires représente un coût supplémentaire : veillez donc à avoir une compréhension claire des avantages avant de la mettre en place. Pour plus d’informations sur les coûts, consultez la [page Tarification](https://azure.microsoft.com/pricing/details/private-link/). Pour plus d’informations sur la façon dont ces composants fonctionnent ensemble, regardez la vidéo en haut de cet article. L’option du point de terminaison privé est présentée à partir de 5:48 dans la vidéo. Pour obtenir des instructions sur la configuration du point de terminaison, consultez [Créer un point de terminaison privé pour Recherche cognitive Azure](service-create-private-endpoint.md).

## <a name="index-access"></a>Accès aux index

Dans Recherche cognitive Azure, les index individuels ne sont pas des objets sécurisables. Au lieu de cela, l’accès aux index est déterminé au niveau de la couche du service (accès en lecture ou en écriture au service) et du contexte d’une opération.

Pour l’accès de l’utilisateur final, vous pouvez structurer les demandes de requête pour établir la connexion à l’aide d’une [clé de requête](search-security-rbac.md), qui configure toutes les demandes en mode de lecture seule et qui inclut l’index spécifique utilisé par votre application. Dans une demande de requête, il est impossible de joindre des index ou d’accéder simultanément à plusieurs index. Ainsi, toutes les demandes ciblent un index unique par définition. Par conséquent, la structure de la demande de requête proprement dite (une clé plus un index unique cible) définit la limite de sécurité.

Il n’existe aucune différence entre l’accès administrateur et l’accès développeur aux index : tous deux doivent disposer d’un accès en écriture pour pouvoir créer, supprimer et mettre à jour des objets gérés par le service. Toute personne disposant d’une [clé d’administration](search-security-rbac.md) pour votre service peut lire, modifier ou supprimer un index de ce service. En ce qui concerne la protection contre la suppression accidentelle ou malveillante d’index, votre contrôle de code source en interne pour les ressources de code est la solution appropriée pour annuler des suppressions ou des modifications d’index indésirables. Recherche cognitive Azure dispose d’un système de basculement dans le cluster pour garantir sa disponibilité, mais il ne stocke pas et n’exécute pas le code propriétaire que vous avez utilisé pour créer ou charger des index.

Pour les solutions d’architecture mutualisée qui nécessitent des limites de sécurité au niveau des index, ces solutions incluent généralement un niveau intermédiaire, que les clients utilisent pour gérer l’isolation des index. Pour plus d’informations sur les cas d’usage d’architecture mutualisée, consultez [Modèles de conception pour les applications SaaS mutualisées et Recherche cognitive Azure](search-modeling-multitenant-saas-applications.md).

## <a name="user-access"></a>Accès utilisateur

La façon dont un utilisateur accède à un index et à d’autres objets est déterminée par le type de clé API sur la demande. La plupart des développeurs créent et affectent des [clés de requête](search-security-api-keys.md) pour les demandes de recherche du côté client. Une clé de requête accorde un accès en lecture au contenu pouvant faire l’objet d’une recherche dans l’index.

Si vous avez besoin d’un contrôle précis par utilisateur sur les résultats de la recherche, vous pouvez créer des filtres de sécurité sur vos requêtes, en retournant des documents associés à une identité de sécurité donnée. Au lieu des rôles prédéfinis et des attributions de rôles, le contrôle d’accès basé sur l’identité est implémenté en tant que *filtre* qui limite les résultats de recherche de documents et de contenu en fonction des identités. Le tableau suivant décrit les deux approches permettant de filtrer les résultats de recherche de contenu non autorisé.

| Approche | Description |
|----------|-------------|
|[Filtrage de sécurité basé sur les filtres d’identité](search-security-trimming-for-azure-search.md)  | Cet article décrit le workflow de base pour l’implémentation du contrôle d’accès basé sur l’identité de l’utilisateur. Il décrit l’ajout d’identificateurs de sécurité à un index, puis le filtrage relatif à ce champ qui permet d’omettre les résultats de contenu non autorisé. |
|[Filtrage de sécurité basé sur les identités Azure Active Directory](search-security-trimming-for-azure-search-with-aad.md)  | Cet article est une extension de l’article précédent, indiquant les étapes à suivre pour récupérer des identités à partir d’Azure Active Directory (Azure AD), l’un des [services gratuits](https://azure.microsoft.com/free/) de la plateforme cloud Azure. |

## <a name="administrative-rights"></a>Droits d’administration

Le [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../role-based-access-control/overview.md) est un système d’autorisation basé sur [Azure Resource Manager](../azure-resource-manager/management/overview.md) pour l’approvisionnement de ressources Azure. Dans Recherche cognitive Azure , Resource Manager est utilisé pour créer ou supprimer le service, gérer les clés API et mettre à l’échelle le service. Ainsi, les attributions de rôles Azure déterminent qui peut effectuer ces tâches, qu’elles utilisent le [portail](search-manage.md), [PowerShell](search-manage-powershell.md) ou les [API REST de gestion](/rest/api/searchmanagement/search-howto-management-rest-api).

En revanche, les droits d’administrateur sur le contenu hébergé sur le service, comme la possibilité de créer ou de supprimer un index, sont conférés via des clés API, comme décrit dans la [section précédente](#index-access).

> [!TIP]
> En utilisant des mécanismes à l’échelle d’Azure, vous pouvez verrouiller un abonnement ou une ressource pour empêcher la suppression accidentelle ou non autorisée de votre service de recherche par les utilisateurs disposant de droits d’administration. Pour plus d’informations, consultez [Verrouiller les ressources pour en empêcher la suppression](../azure-resource-manager/management/lock-resources.md).

## <a name="certifications-and-compliance"></a>Certifications et conformité

Recherche cognitive Azure a été certifié conforme à plusieurs standards mondiaux, régionaux et spécifiques à des secteurs pour le cloud public et Azure Government. Pour obtenir la liste complète, téléchargez le livre blanc [**Microsoft Azure Compliance Offerings**](https://azure.microsoft.com/resources/microsoft-azure-compliance-offerings/) depuis la page des rapports d’audit officiels.

Pour la conformité, vous pouvez utiliser [Azure Policy](../governance/policy/overview.md) pour mettre en œuvre les meilleures pratiques de haute sécurité d’[Azure Security Benchmark](../security/benchmarks/introduction.md). Azure Security Benchmark est un ensemble de recommandations de sécurité, codifiées en contrôles de sécurité qui correspondent aux principales actions que vous devez prendre pour atténuer les menaces pesant sur les services et les données. Il existe actuellement 11 contrôles de sécurité, dont la [Sécurité réseau](../security/benchmarks/security-control-network-security.md), [Journalisation et surveillance](../security/benchmarks/security-control-logging-monitoring.md), et [Protection des données](../security/benchmarks/security-control-data-protection.md), pour n’en nommer que quelques-uns.

Azure Policy est une capacité intégrée à Azure qui vous permet de gérer la conformité de plusieurs normes, y compris celles d’Azure Security Benchmark. Pour les critères de référence bien connus, Azure Policy fournit des définitions intégrées qui fournissent à la fois des critères et une réponse actionnable en cas de non-conformité.

Pour Recherche cognitive Azure, il existe actuellement une définition intégrée. Elle concerne la journalisation des diagnostics. Grâce à cette intégration, vous pouvez attribuer une stratégie qui identifie tout service de recherche auquel il manque la journalisation des diagnostics, puis l’active. Pour plus d’informations, consultez [Contrôles de conformité réglementaire d’Azure Policy pour Recherche cognitive Azure](security-controls-policy.md).

## <a name="see-also"></a>Voir aussi

+ [Concepts de base de la sécurité Azure](../security/fundamentals/index.yml)
+ [Sécurité Azure](https://azure.microsoft.com/overview/security)
+ [Centre de sécurité Azure](../security-center/index.yml)

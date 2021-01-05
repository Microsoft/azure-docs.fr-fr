---
title: Sécurité d’Azure Key Vault
description: Gérez les autorisations d’accès à Azure Key Vault, aux clés et aux secrets. Couvre le modèle d’authentification et d’autorisation de Key Vault, et explique comment sécuriser votre coffre de clés.
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: conceptual
ms.date: 09/30/2020
ms.author: mbaldwin
ms.openlocfilehash: dc08df7390285f9b6e4701bb1ca5c4227b19f1da
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94445028"
---
# <a name="azure-key-vault-security"></a>Sécurité d’Azure Key Vault

Vous utilisez Azure Key Vault pour protéger les secrets et les clés de chiffrement, comme les certificats, les chaînes de connexion et les mots de passe dans le cloud. Lors du stockage de données sensibles et vitales pour l’entreprise, vous devez prendre des mesures afin d’optimiser la sécurité de vos coffres et des données qui y sont stockées.

## <a name="identity-and-access-management"></a>Gestion de l’identité et de l’accès

Quand vous créez un coffre de clés dans un abonnement Azure, il est automatiquement associé au locataire Azure AD de l’abonnement. Toute personne qui essaie de gérer ou de récupérer le contenu d’un coffre doit être authentifiée par Azure AD.

- L’authentification établit l’identité de l’appelant.
- L’autorisation détermine les opérations que l’appelant peut exécuter. L’autorisation dans Key Vault utilise une combinaison de [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../role-based-access-control/overview.md) et de stratégies d’accès Azure Key Vault.

### <a name="access-model-overview"></a>Vue d’ensemble du modèle d’accès

L’accès aux coffres se produit via deux interfaces ou plans. Il s’agit du plan de gestion et du plan de données.

- Le *plan de gestion* est l’endroit où vous gérez Key Vault lui-même. C’est l’interface utilisée pour créer et supprimer des coffres. Vous pouvez également lire les propriétés de coffre de clés et gérer les stratégies d’accès.
- Le *plan de données* vous permet d’utiliser les données stockées dans un coffre de clés. Vous pouvez ajouter, supprimer et modifier des clés, des secrets et des certificats.

Pour accéder à un coffre de clés dans l’un ou l’autre de ces plans, les appelants (utilisateurs ou applications) doivent être authentifiés et autorisés. Les deux plans utilisent Azure Active Directory (Azure AD) pour l’authentification. Pour l’autorisation, le plan de gestion utilise le contrôle d’accès en fonction du rôle Azure (Azure RBAC), tandis que le plan de données s’appuie sur une stratégie d’accès Key Vault.

Le modèle d’un mécanisme d’authentification unique auprès des deux plans présente plusieurs avantages :

- Les organisations peuvent contrôler de manière centralisée l’accès à tous leurs coffres de clés.
- Si un utilisateur part, il perd instantanément l’accès à tous les coffres de clés de l’organisation.
- Les organisations peuvent personnaliser l’authentification à l’aide des options dans Azure AD, par exemple pour activer l’authentification multifacteur afin de renforcer la sécurité.

### <a name="managing-administrative-access-to-key-vault"></a>Gestion de l’accès administratif à Key Vault

Lorsque vous créez un coffre de clés dans un groupe de ressources, vous gérez l’accès à l’aide d’Azure AD. Vous autorisez des utilisateurs ou des groupes à gérer les coffres de clés dans un groupe de ressources. Vous pouvez accorder l’accès à un niveau d’étendue spécifique en attribuant les rôles Azure appropriés. Pour permettre à un utilisateur de gérer des coffres de clés, vous attribuez un rôle `key vault Contributor` prédéfini à l’utilisateur dans une étendue spécifique. Les niveaux d’étendue suivants peuvent être attribués à un rôle Azure :

- **Abonnement**: Un rôle Azure attribué au niveau d’un abonnement s’applique à tous les groupes de ressources et à toutes les ressources au sein de cet abonnement.
- **Groupe de ressources** : Un rôle Azure attribué au niveau d’un groupe de ressources s’applique à toutes les ressources de ce groupe de ressources.
- **Ressource spécifique** : Un rôle Azure attribué pour une ressource spécifique s’applique à cette ressource. Dans ce cas, la ressource est un coffre de clés spécifique.

Il existe plusieurs rôles prédéfinis. Si un rôle prédéfini ne répond pas à vos besoins, vous pouvez définir votre propre rôle. Pour plus d’informations, consultez [Azure RBAC : pour les ressources Azure](../../role-based-access-control/built-in-roles.md).

> [!IMPORTANT]
> Si un utilisateur dispose d’autorisations `Contributor` sur un plan de gestion de coffre de clés, il peut s’accorder lui-même l’accès au plan de données en définissant une stratégie d’accès Key Vault. Vous devez contrôler étroitement qui dispose d’un accès accordé par le rôle `Contributor` à vos coffres de clés. Vérifiez que seules les personnes autorisées peuvent accéder à et gérer vos coffres de clés, vos clés, vos secrets et vos certificats.

<a id="data-plane-access-control"></a>
### <a name="controlling-access-to-key-vault-data"></a>Contrôle de l’accès aux données de Key Vault

Les stratégies d’accès Key Vault accordent des autorisations distinctement aux clés, secrets ou certificats. Vous pouvez uniquement accorder un accès utilisateur à des clés (pas à des secrets). Les autorisations d’accès aux clés, secrets et certificats sont gérées au niveau du coffre.

> [!IMPORTANT]
> Les stratégies d’accès Key Vault ne prennent pas en charge les autorisations granulaires au niveau des objets, comme une clé, un secret ou un certificat spécifique. Quand un utilisateur est autorisé à créer et à supprimer des clés, il peut effectuer ces opérations sur toutes les clés de ce coffre de clés.

Vous pouvez définir des stratégies d’accès pour un coffre de clés à l’aide du [portail Azure](assign-access-policy-portal.md), de l’interface [Azure CLI](assign-access-policy-cli.md), d’[Azure PowerShell](assign-access-policy-powershell.md) ou des [API REST de gestion Key Vault](/rest/api/keyvault/).

Vous pouvez limiter l’accès au plan de données en utilisant des [points de terminaison de service de réseau virtuel pour Azure Key Vault](overview-vnet-service-endpoints.md). Vous pouvez configurer [des pare-feu et des règles de réseau virtuel](network-security.md) pour mettre en place une couche de sécurité supplémentaire.

## <a name="network-access"></a>Accès réseau

Vous pouvez réduire l’exposition de vos coffres en spécifiant les adresses IP qui y ont accès. Les points de terminaison de service de réseau virtuel pour Azure Key Vault permettent de restreindre l’accès à un réseau virtuel spécifié. Les points de terminaison vous permettent également de restreindre l’accès à une liste de plages d’adresses IPv4 (Internet Protocol version 4). L’accès est refusé à tout utilisateur se connectant à votre coffre de clés en dehors de ces sources.

Une fois que les règles de pare-feu sont effectives, les utilisateurs peuvent lire les données de Key Vault seulement quand leurs requêtes proviennent de réseaux virtuels ou de plages d’adresses IPv4 autorisés. Ceci s’applique également à l’accès au coffre de clés à partir du portail Azure. Si des utilisateurs peuvent accéder à un coffre de clés à partir du portail Azure, il ne peuvent pas lister les clés/secrets/certificats si leur ordinateur client ne figure pas dans la liste autorisée. Ceci affecte également l’accès au sélecteur de compte Key Vault par d’autres services Azure. Des utilisateurs qui peuvent voir la liste des coffres de clés ne peuvent pas lister les clés si des règles de pare-feu bloquent leur ordinateur client.

Pour plus d’informations sur l’adresse réseau d’Azure Key Vault, voir [Point de terminaison de service de réseau virtuel pour Azure Key Vault](overview-vnet-service-endpoints.md).

## <a name="tls-and-https"></a>TLS et HTTPS

*   Le serveur frontal Key Vault (plan de données) est un serveur multilocataire. Cela signifie que les coffres de clés de différents clients peuvent partager la même IP publique. À des fins d’isolement, chaque requête HTTP est authentifiée et autorisée indépendamment des autres requêtes.
*   Vous pouvez identifier des versions antérieures de TLS pour signaler des vulnérabilités, mais, comme l’IP publique est partagée, il n’est pas possible pour l’équipe du service des coffres de clés de désactiver les anciennes versions de TLS pour les coffres de clés individuels au niveau du transport.
*   Le protocole HTTPS permet au client de participer à la négociation TLS. **Les clients peuvent appliquer la version la plus récente de TLS** et, chaque fois qu’un client le fait, l’ensemble de la connexion utilise le niveau de protection correspondant. Le fait que Key Vault prenne toujours en charge les anciennes versions de TLS ne nuit pas à la sécurité des connexions utilisant des versions plus récentes de TLS.
*   Malgré les vulnérabilités connues du protocole TLS, il n’existe aucune attaque connue qui permettrait à un agent malveillant d’extraire des informations de votre coffre de clés lorsque l’attaquant établit une connexion avec une version de TLS qui présente des vulnérabilités. L’attaquant doit toujours s’authentifier et s’autoriser et, tant que les clients légitimes se connectent toujours avec des versions récentes de TLS, il est impossible que des informations d’identification aient pu être divulguées à partir de vulnérabilités d’anciennes versions de TLS.

## <a name="logging-and-monitoring"></a>Enregistrement et surveillance

La journalisation de Key Vault enregistre les informations sur les activités effectuées sur votre coffre. Pour plus d’informations, consultez [Journalisation de Key Vault](logging.md).

Pour toute recommandation quant à la gestion sécurisée des comptes de stockage, consultez le [guide de sécurité du Stockage Azure](../../storage/blobs/security-recommendations.md).

## <a name="next-steps"></a>Étapes suivantes

- [Points de terminaison de service de réseau virtuel pour Azure Key Vault](overview-vnet-service-endpoints.md)
- [RBAC Azure : Rôles intégrés](../../role-based-access-control/built-in-roles.md)
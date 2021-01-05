---
title: Sécuriser l’accès à un coffre de clés
description: Modèle d’accès pour Azure Key Vault, notamment les points de terminaison de ressources et l’authentification Active Directory.
services: key-vault
author: ShaneBala-keyvault
manager: ravijan
ms.service: key-vault
ms.subservice: general
ms.topic: conceptual
ms.date: 10/07/2020
ms.author: sudbalas
ms.openlocfilehash: 3f28c50be73b2b87ed8b25429cfa2dee9a663f1b
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96452165"
---
# <a name="secure-access-to-a-key-vault"></a>Sécuriser l’accès à un coffre de clés

Azure Key Vault est un service cloud qui protège les clés et secrets de chiffrement comme les certificats, chaînes de connexion et mots de passe. Comme il s’agit de données sensibles et critiques, vous devez sécuriser l’accès à vos coffres de clés en acceptant seulement les applications et les utilisateurs autorisés. Cet article fournit une vue d’ensemble du modèle d’accès aux coffres de clés. Il décrit l’authentification et l’autorisation, puis explique comment sécuriser l’accès à vos coffres de clés.

Pour plus d’informations sur Key Vault, consultez [À propos d’Azure Key Vault](overview.md) ; pour plus d’informations sur ce qui peut être stocké dans un coffre de clés, consultez [À propos des clés, des secrets et des certificats](about-keys-secrets-certificates.md).

## <a name="access-model-overview"></a>Vue d’ensemble du modèle d’accès

L’accès à un coffre de clés est contrôlé par le biais de deux interfaces : le **plan de gestion** et le **plan de données**. Le plan de gestion vous permet de gérer le coffre de clés. Dans ce plan, vous pouvez notamment créer et supprimer des coffres de clés, récupérer des propriétés Key Vault et mettre à jour des stratégies d’accès. Le plan de données vous permet d’utiliser les données stockées dans un coffre de clés. Vous pouvez ajouter, supprimer et modifier des clés, des secrets et des certificats.

Les deux plans utilisent [Azure Active Directory (Azure AD)](../../active-directory/fundamentals/active-directory-whatis.md) pour l’authentification. Pour l’autorisation, le plan de gestion utilise le [contrôle d’accès en fonction du rôle (Azure RBAC) Azure](../../role-based-access-control/overview.md) et le plan de données utilise une [stratégie d’accès Key Vault](./assign-access-policy-portal.md) et [Azure RBAC pour les opérations du plan de données Key Vault (préversion)](./rbac-guide.md).

Pour accéder à un coffre de clés dans l’un ou l’autre de ces plans, tout appelant (utilisateur ou application) doit être authentifié et autorisé. L’authentification établit l’identité de l’appelant. L’autorisation détermine les opérations que l’appelant peut exécuter. L’authentification auprès de Key Vault fonctionne conjointement avec [Azure Active Directory (Azure AD)](../../active-directory/fundamentals/active-directory-whatis.md), qui est chargé d’authentifier l’identité de chaque **principal de sécurité** donné.

Un principal de sécurité est un objet qui représente un utilisateur, un groupe, un service ou une application demandant l’accès aux ressources Azure. Azure affecte un **ID d’objet** unique à chaque principal de sécurité.

* Un principal de sécurité **utilisateur** identifie une personne disposant d’un profil dans Azure Active Directory.

* Un principal de sécurité **groupe** identifie un ensemble d’utilisateurs créés dans Azure Active Directory. Tous les rôles et autorisations attribués au groupe sont accordés à tous les utilisateurs du groupe.

* Un **principal de service** est un type de principal de sécurité qui identifie une application ou un service, c’est-à-dire un morceau de code plutôt qu’un utilisateur ou un groupe. L’ID d’objet d’un principal de service, appelé **ID client**, lui sert de nom d’utilisateur. La **clé secrète client** ou le **certificat** du principal de service fonctionne comme un mot de passe. De nombreux services Azure prennent en charge l’attribution de l’[identité managée](../../active-directory/managed-identities-azure-resources/overview.md) avec la gestion automatisée de l’**ID client** et du **certificat**. L’identité managée est l’option la plus sécurisée et recommandée pour l’authentification dans Azure.

Pour plus d’informations sur l’authentification pour Key Vault, consultez [S’authentifier auprès d’Azure Key Vault](authentication.md)

## <a name="key-vault-authentication-options"></a>Options d’authentification Key Vault

Quand vous créez un coffre de clés dans un abonnement Azure, il est automatiquement associé au locataire Azure AD de l’abonnement. Tous les appelants dans les deux plans doivent s’inscrire auprès de ce locataire et s’authentifier pour accéder au coffre de clés. Dans les deux cas, les applications peuvent accéder au coffre de clés de trois manières :

- **Application uniquement** : L’application représente un principal du service ou une identité managée. Cette identité est le scénario le plus courant pour les applications qui doivent accéder régulièrement à des certificats, des clés ou des secrets à partir du coffre de clés. Pour que ce scénario fonctionne, la propriété `objectId` de l’application doit être spécifiée dans la stratégie d’accès et `applicationId` ne doit _pas_ être spécifiée ou doit être `null`.
- **Utilisateur uniquement** : l’utilisateur accède au coffre de clés à partir de n’importe quelle application inscrite dans le locataire. Azure PowerShell et le portail Azure sont des exemples de ce type d’accès. Pour que ce scénario fonctionne, la propriété `objectId` de l’utilisateur doit être spécifiée dans la stratégie d’accès et `applicationId` ne doit _pas_ être spécifiée ou doit être `null`.
- **Application-plus-utilisateur** (parfois appelé _identité composée)_  : l’utilisateur est tenu d’accéder au coffre de clés à partir d’une application spécifique _et_ l’application doit utiliser le flux OBO (Authentification On-Behalf-Of) pour emprunter l’identité de l’utilisateur. Pour que ce scénario fonctionne, l’`applicationId` et l’`objectId` doivent être spécifiés dans la stratégie d’accès. `applicationId` identifie l’application requise et `objectId` identifie l’utilisateur. Actuellement, cette option n’est pas disponible pour le plan de données Azure RBAC (préversion).

Pour tous les types d’accès, l’application s’authentifie auprès d’Azure AD. L’application utilise une [méthode d’authentification prise en charge](../../active-directory/develop/authentication-vs-authorization.md) en fonction du type d’application. L’application acquiert un jeton pour une ressource dans le plan pour accorder l’accès. La ressource est un point de terminaison dans le plan de gestion ou de données, en fonction de l’environnement Azure. L’application utilise le jeton et envoie une demande d’API REST à Key Vault. Pour en savoir plus, passez en revue le [flux d’authentification intégral](../../active-directory/develop/v2-oauth2-auth-code-flow.md).

Le modèle d’un mécanisme d’authentification unique auprès des deux plans présente plusieurs avantages :

- Les organisations peuvent contrôler de manière centralisée l’accès à tous leurs coffres de clés.
- Si un utilisateur part, il perd instantanément l’accès à tous les coffres de clés de l’organisation.
- Les organisations peuvent personnaliser l’authentification à l’aide des options dans Azure AD, par exemple pour activer l’authentification multifacteur afin de renforcer la sécurité.

## <a name="resource-endpoints"></a>Points de terminaison de ressource

Les applications accèdent aux plans par le biais de points de terminaison. Les contrôles d’accès pour les deux plans fonctionnent indépendamment. Pour permettre à une application d’utiliser des clés dans un coffre de clés, vous accordez l’accès au plan de données à l’aide d’une stratégie d’accès Key Vault ou d’Azure RBAC (préversion). Pour permettre à un utilisateur de lire les propriétés et les étiquettes d’un coffre de clés, mais pas d’accéder aux données (clés, secrets ou certificats), vous accordez l’accès au plan de gestion avec Azure RBAC.

Le tableau suivant présente les points de terminaison pour les plans de gestion et de données.

| Plan&nbsp;d’accès | Points de terminaison d’accès | Opérations | Mécanisme de contrôle&nbsp;d’accès |
| --- | --- | --- | --- |
| Plan de gestion | **Mondial :**<br> management.azure.com:443<br><br> **Azure China 21Vianet:**<br> management.chinacloudapi.cn:443<br><br> **Azure US Government :**<br> management.usgovcloudapi.net:443<br><br> **Azure Germany :**<br> management.microsoftazure.de:443 | Créer, lire, mettre à jour et supprimer des coffres de clés<br><br>Définir des stratégies d’accès Key Vault<br><br>Définir des étiquettes Key Vault | Azure RBAC |
| Plan de données | **Mondial :**<br> &lt;nom du coffre&gt;.vault.azure.net:443<br><br> **Azure China 21Vianet:**<br> &lt;nom du coffre&gt;.vault.azure.cn:443<br><br> **Azure US Government :**<br> &lt;nom du coffre&gt;.vault.usgovcloudapi.net:443<br><br> **Azure Germany :**<br> &lt;nom du coffre&gt;.vault.microsoftazure.de:443 | Clés : chiffrer, déchiffrer, wrapKey, unwrapKey, signer, vérifier, obtenir, lister, créer, mettre à jour, importer, supprimer, récupérer, sauvegarder, restaurer, purger<br><br> Certificats : managecontacts, getissuers, listissuers, setissurs, deleteissuers, manageissuers, obtenir, lister, créer, importer, mettre à jour, supprimer, récupérer, sauvegarder, restaurer, purger<br><br>  Secrets : obtenir, lister, définir, supprimer, récupérer, sauvegarder, restaurer, purger | Stratégie d’accès Key Vault ou Azure RBAC (préversion)|

## <a name="management-plane-and-azure-rbac"></a>Plan de gestion et Azure RBAC

Dans le plan de gestion, vous utilisez le [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../role-based-access-control/overview.md) pour autoriser les opérations qu’un appelant peut exécuter. Dans le modèle Azure RBAC, chaque abonnement Azure a une instance d’Azure AD. Vous accordez l’accès aux utilisateurs, groupes et applications de ce répertoire. L’accès accordé permet de gérer les ressources de l’abonnement Azure qui reposent sur le modèle de déploiement Azure Resource Manager.

Vous créez un coffre de clés dans un groupe de ressources et gérez l’accès à l’aide d’Azure AD. Vous autorisez des utilisateurs ou des groupes à gérer les coffres de clés dans un groupe de ressources. Vous accordez l’accès à un niveau d’étendue spécifique en attribuant les rôles Azure appropriés. Pour accorder un accès à un utilisateur pour gérer des coffres de clés, vous attribuez un rôle [Contributeur Key Vault](../../role-based-access-control/built-in-roles.md#key-vault-contributor) prédéfini à l’utilisateur dans une étendue spécifique. Les niveaux d’étendue suivants peuvent être attribués à un rôle Azure :

- **Abonnement**: Un rôle Azure attribué au niveau d’un abonnement s’applique à tous les groupes de ressources et à toutes les ressources au sein de cet abonnement.
- **Groupe de ressources** : Un rôle Azure attribué au niveau d’un groupe de ressources s’applique à toutes les ressources de ce groupe de ressources.
- **Ressource spécifique** : Un rôle Azure attribué pour une ressource spécifique s’applique à cette ressource. Dans ce cas, la ressource est un coffre de clés spécifique.

Il existe plusieurs rôles prédéfinis. Si un rôle prédéfini ne répond pas à vos besoins, vous pouvez définir votre propre rôle. Pour plus d’informations, voir [Rôles intégrés Azure](../../role-based-access-control/built-in-roles.md). 

Vous avez besoin d’autorisations `Microsoft.Authorization/roleAssignments/write` et `Microsoft.Authorization/roleAssignments/delete`, telles que [Administrateur de l’accès utilisateur](../../role-based-access-control/built-in-roles.md#user-access-administrator) ou [Propriétaire](../../role-based-access-control/built-in-roles.md#owner).

> [!IMPORTANT]
> Si un utilisateur dispose d’autorisations `Contributor` sur un plan de gestion de coffre de clés, il peut s’accorder lui-même l’accès au plan de données en définissant une stratégie d’accès Key Vault. Vous devez contrôler étroitement qui dispose d’un accès accordé par le rôle `Contributor` à vos coffres de clés. Vérifiez que seules les personnes autorisées peuvent accéder à et gérer vos coffres de clés, vos clés, vos secrets et vos certificats.
>

<a id="data-plane-access-control"></a>
## <a name="data-plane-and-access-policies"></a>Plan de données et stratégies d’accès

Vous pouvez accorder l’accès au plan de données en définissant des stratégies d’accès Key Vault pour un coffre de clés. Pour définir ces stratégies d’accès, un utilisateur, un groupe ou une application doit disposer d’autorisations `Key Vault Contributor` pour le plan de gestion de ce coffre de clés.

Vous accordez l’accès à un utilisateur, un groupe ou une application afin d’exécuter des opérations spécifiques sur les clés ou secrets d’un coffre de clés. Key Vault prend en charge jusqu’à 1 024 entrées de stratégie d’accès pour un coffre de clés. Pour accorder l’accès au plan de données à plusieurs utilisateurs, créez un groupe de sécurité Azure AD et ajoutez des utilisateurs à ce groupe.

Vous pouvez voir la liste complète des opérations de coffre et de secret ici : [Référence relative aux opérations de coffre de clés](/rest/api/keyvault/#vault-operations)

<a id="key-vault-access-policies"></a> Les stratégies d’accès Key Vault accordent des autorisations distinctement aux clés, secrets et certificats.  Les autorisations d’accès aux clés, secrets ou certificats sont définies au niveau du coffre. 

Pour plus d’informations sur l’utilisation des stratégies d’accès au coffre de clés, consultez [Attribuer une stratégie d’accès Key Vault](assign-access-policy-portal.md)

> [!IMPORTANT]
> Les stratégies d’accès Key Vault s’appliquent au niveau du coffre. Quand un utilisateur est autorisé à créer et à supprimer des clés, il peut effectuer ces opérations sur toutes les clés de ce coffre de clés.
Les stratégies d’accès Key Vault ne prennent pas en charge les autorisations granulaires au niveau des objets, comme une clé, un secret ou un certificat spécifique. 
>

## <a name="data-plane-and-azure-rbac-preview"></a>Plan de données et Azure RBAC (préversion)

Le contrôle d’accès en fonction du rôle Azure est un modèle d’autorisation alternatif qui permet de contrôler l’accès au plan de données Azure Key Vault, qui peut être activé sur des coffres de clés individuels. Le modèle d’autorisation Azure RBAC est exclusif et défini une seule fois. Les stratégies d’accès au coffre sont devenues inactives. Azure Key Vault définit un ensemble de rôles intégrés Azure qui englobent des ensembles d’autorisations communs utilisés pour accéder aux clés, aux secrets ou aux certificats.

Lorsqu’un rôle Azure est attribué à un principal de sécurité Azure AD, Azure octroie l’accès à ces ressources pour ce principal de sécurité. L’accès peut être limité au niveau de l’abonnement, du groupe de ressources, du coffre de clés ou d’une clé, d’un secret ou d’un certificat spécifique. Un principal de sécurité Azure AD peut correspondre à un utilisateur, à un groupe, à un principal de service d’application ou à une [identité managée pour les ressources Azure](../../active-directory/managed-identities-azure-resources/overview.md).

Les principaux avantages liés à l’utilisation de l’autorisation Azure RBAC par rapport à celle des stratégies d’accès au coffre sont la gestion centralisée du contrôle d’accès et l’intégration à [Privileged Identity Management (PIM)](../../active-directory/privileged-identity-management/pim-configure.md). Privileged Identity Management assure une activation de rôle basée sur l’heure et l’approbation pour atténuer les risques d’autorisations d’accès excessives, injustifiées ou malveillantes sur les ressources qui vous intéressent.

Pour plus d’informations sur le plan de données Key Vault avec Azure RBAC, consultez [Clés, certificats et secrets Key Vault avec un contrôle d’accès en fonction du rôle Azure (préversion)](rbac-guide.md)

## <a name="firewalls-and-virtual-networks"></a>Pare-feu et réseaux virtuels

Pour mettre en place une couche de sécurité supplémentaire, vous pouvez configurer des pare-feu et des règles de réseau virtuel. Vous pouvez configurer des pare-feux et réseaux virtuels Key Vault pour refuser par défaut l’accès au trafic en provenance de tous les réseaux (y compris le trafic Internet). Vous pouvez accorder l’accès au trafic en provenance de [réseaux virtuels Azure](../../virtual-network/virtual-networks-overview.md) ou de plages d’adresses IP Internet publiques spécifiques, ce qui vous permet de créer une limite réseau sécurisée pour vos applications.

Voici quelques exemples d’utilisation de points de terminaison de service :

* Vous utilisez Key Vault pour stocker des clés de chiffrement, des secrets d’application et des certificats, et vous voulez bloquer l’accès à votre coffre de clés depuis l’Internet public.
* Vous voulez verrouiller l’accès à votre coffre de clés, afin que seule votre application ou une liste restreinte d’hôtes désignés puissent se connecter à votre coffre de clés.
* Vous avez une application s’exécutant dans votre réseau virtuel Azure, qui est verrouillé pour tout trafic entrant et sortant. Votre application doit néanmoins toujours se connecter au coffre de clés pour extraire des secrets ou certificats, ou pour utiliser des clés de chiffrement.

Pour plus d’informations sur le pare-feu et les réseaux virtuels Key Vault, consultez [Configurer des pare-feu et des réseaux virtuels Azure Key Vault.](network-security.md)

> [!NOTE]
> Les règles de pare-feu et de réseau virtuel Key Vault s’appliquent seulement au plan de données de Key Vault. Les opérations du plan de contrôle Key Vault (comme les opérations de création, de suppression et de modification, la définition de stratégies d’accès, la définition de pare-feux et de règles de réseau virtuel) ne sont pas affectées par les règles de pare-feux et de réseau virtuel.

## <a name="private-endpoint-connection"></a>Connexion de point de terminaison privé

Si vous avez besoin de bloquer complètement l’exposition du Key Vault au public, vous pouvez utiliser un [point de terminaison privé Azure](../../private-link/private-endpoint-overview.md). Un point de terminaison privé Azure est une interface réseau qui vous connecte de façon privée et sécurisée à un service basé sur la technologie Azure Private Link. Le point de terminaison privé utilise une adresse IP privée de votre réseau virtuel, plaçant de fait le service dans votre réseau virtuel. Sachant que l’ensemble du trafic à destination du service peut être routé via le point de terminaison privé, il n’y a aucun besoin de passerelles, d’appareils NAT, de connexions ExpressRoute ou VPN ou d’adresses IP publiques. Le trafic entre votre réseau virtuel et le service transite par le réseau principal de Microsoft, éliminant ainsi toute exposition à l’Internet public. Vous pouvez vous connecter à une instance d’une ressource Azure, ce qui vous donne le plus haut niveau de granularité en matière de contrôle d’accès.

Scénarios courants d’utilisation d’une liaison privée pour les services Azure :

- **Accès en privé aux services sur la plateforme Azure** : Connectez votre réseau virtuel à des services dans Azure sans adresse IP publique au niveau de la source ou de la destination. Les fournisseurs de services peuvent afficher leurs services dans leur propre réseau virtuel et les consommateurs peuvent accéder à ces services dans leur réseau virtuel local. La plateforme Liaison privée gère la connectivité entre le consommateur et les services sur le réseau principal Azure. 
 
- **Réseaux locaux et appairés** : Accédez aux services s’exécutant dans Azure en local par le biais du peering privé ExpressRoute, de tunnels VPN et de réseaux virtuels appairés à l’aide de points de terminaison privés. Il n’est pas nécessaire de configurer le peering public ni de transiter par Internet pour atteindre le service. Private Link offre un moyen sécurisé de migrer des charges de travail vers Azure.
 
- **Protection contre la fuite de données** : Un point de terminaison privé est mappé à une instance d’une ressource PaaS plutôt qu’au service entier. Les consommateurs peuvent se connecter uniquement à la ressource spécifique. L’accès à toute autre ressource du service est bloqué. Ce mécanisme offre une protection contre les risques de fuite de données. 
 
- **Global Reach** : Connectez-vous en privé à des services s’exécutant dans d’autres régions. Le réseau virtuel du consommateur peut se trouver dans la région A et se connecter aux services qui se trouvent derrière Private Link dans la région B.  
 
- **Extension à vos propres services** : Activez les mêmes expériences et fonctionnalités pour afficher votre service en privé aux consommateurs dans Azure. En plaçant votre service derrière une instance Azure Load Balancer standard, vous pouvez l’activer pour Private Link. Le consommateur peut alors se connecter directement à votre service à l’aide d’un point de terminaison privé dans son propre réseau virtuel. Vous pouvez gérer les demandes de connexion à l’aide d’un flux d’appels d’approbation. Azure Private Link fonctionne pour les consommateurs et services appartenant à différents locataires Azure Active Directory. 

Pour plus d’informations sur les points de terminaison privés, consultez [Key Vault avec Azure Private Link](./private-link-service.md)

## <a name="example"></a>Exemple

Dans cet exemple, nous développons une application qui utilise un certificat pour TLS/SSL, Stockage Azure pour stocker les données et une clé RSA 2 048 bits pour chiffrer les données dans Stockage Azure. Notre application s’exécute dans une machine virtuelle Azure (ou un groupe de machines virtuelles identiques). Nous pouvons utiliser un coffre de clés pour stocker les secrets de l’application. Nous pouvons stocker le certificat de démarrage utilisé par l’application pour s’authentifier auprès d’Azure AD.

Nous avons besoin d’accéder aux clés et aux secrets stockés suivants :
- **Certificat TLS/SSL** : Utilisé pour TLS/SSL.
- **Clé de stockage** : utilisée pour accéder au compte de stockage.
- **Clé RSA 2 048 bits** : utilisée pour la clé de chiffrement de données wrap/unwrap par Stockage Azure.
- **Identité managée d’application** : utilisé pour s’authentifier auprès d’Azure AD. Une fois l’accès au coffre de clés accordé, l’application peut récupérer la clé de stockage et le certificat.

Nous devons définir les rôles suivants pour spécifier qui peut gérer, déployer et auditer notre application :
- **Équipe de sécurité** : personnel informatique du bureau du chef de la sécurité ou contributeurs équivalents. L’équipe de sécurité est chargée de protéger les secrets. Les secrets peuvent inclure des certificats TLS/SSL, des clés RSA pour le chiffrement, des chaînes de connexion et des clés de compte de stockage.
- **Développeurs et opérateurs** : équipe qui développe l’application et la déploie dans Azure. Les membres de cette équipe ne font pas partie de l’équipe de sécurité. Ils ne doivent pas avoir accès aux données sensibles, notamment les certificats TLS/SSL et les clés RSA. Seule l’application qu’ils déploient doit avoir accès à ces données sensibles.
- **Auditeurs** : ce rôle s’applique aux contributeurs qui ne sont pas membres de l’équipe de développement ou du personnel informatique général. Ils passent en revue l’utilisation et la maintenance des certificats, clés et secrets pour assurer la conformité aux normes de sécurité.

Il existe un autre rôle qui dépasse le cadre de notre application : l’administrateur d’abonnement (ou groupe de ressources). L’administrateur d’abonnement configure les autorisations d’accès initiales pour l’équipe de sécurité. Il accorde l’accès à l’équipe de sécurité à l’aide d’un groupe de ressources comprenant les ressources requises par l’application.

Nous devons autoriser les opérations suivantes pour nos rôles :

**Équipe de sécurité**
- Créer des coffres de clés
- Activer la journalisation Key Vault
- Ajouter des clés et des secrets
- Créer des sauvegardes des clés pour la reprise d’activité
- Définissez des stratégies d’accès Key Vault et attribuez des rôles pour autoriser des utilisateurs et des applications à effectuer des opérations spécifiques.
- Regénérer périodiquement les clés et les secrets

**Développeurs et opérateurs**
- Obtenir auprès de l’équipe de sécurité des références aux certificats TLS/SSL et de démarrage (empreintes numériques), à la clé de stockage (URI de secret) et à la clé RSA (URI de clé) pour wrap/unwrap.
- Développer et déployer l’application pour accéder programmatiquement aux certificats et aux secrets.

**Auditeurs**
- Examiner les journaux d’activité Key Vault pour confirmer l’utilisation correcte des clés et des secrets et la conformité aux normes en matière de sécurité des données

Le tableau suivant récapitule les autorisations d’accès pour nos rôles et notre application.

| Role | Autorisations de plan de gestion | Autorisations du plan de données - Stratégies d’accès au coffre | Autorisations du plan de données - Azure RBAC (préversion)  |
| --- | --- | --- | --- |
| Équipe de sécurité | [Contributeur Key Vault](../../role-based-access-control/built-in-roles.md#key-vault-contributor) | Certificats : toutes les opérations <br> Clés : toutes les opérations <br> Secrets : toutes les opérations | [Administrateur Key Vault (préversion)](../../role-based-access-control/built-in-roles.md#key-vault-administrator-preview) |
| Développeurs et&nbsp;opérateurs | Autorisation de déploiement Key Vault<br><br> **Remarque** : Cette autorisation permet aux machines virtuelles déployées de récupérer les secrets d’un coffre de clés. | None | None |
| Auditeurs | None | Certificates : liste <br> Clés : énumération<br>Secrets : énumération<br><br> **Remarque** : Cette autorisation permet aux auditeurs d’inspecter les attributs (étiquettes, dates d’activation, dates d’expiration) pour les clés et secrets non émis dans les journaux d’activité. | [Lecteur Key Vault (préversion)]https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#key-vault-reader-preview |
| Compte Stockage Azure | None | Clés : obtenir, lister, wrapKey, unwrapKey <br> | [Service de chiffrement de Key Vault](../../role-based-access-control/built-in-roles.md#key-vault-crypto-service-encryption-preview) |
| Application | None | Secrets : obtenir, lister <br> Certificats : obtenir, lister | [Lecteur Key Vault (préversion)](../../role-based-access-control/built-in-roles.md#key-vault-reader-preview), [Utilisateur secret Key Vault (préversion)](../../role-based-access-control/built-in-roles.md#key-vault-secrets-user-preview) |

Une fois les autorisations Key Vault définies, vous devez accorder aux trois rôles d’équipe l’accès à d’autres ressources. Pour déployer des machines virtuelles (ou la fonctionnalité Web Apps d’Azure App Service), les développeurs et opérateurs ont besoin de déployer l’accès. Les auditeurs ont besoin d’un accès en lecture au compte de stockage où les journaux d’activité Key Vault sont stockés.

Notre exemple décrit un scénario simple. Les scénarios réels peuvent être plus complexes. Vous pouvez ajuster les autorisations à votre coffre de clés selon vos besoins. Nous partons du principe que l’équipe de sécurité fournit les références aux clés et aux secrets (URI et empreintes) dont se sert l’équipe DevOps dans ses applications. Les développeurs et les opérateurs n’ont besoin d’aucun accès au plan de données. Nous avons vu comment sécuriser votre coffre de clés.

> [!NOTE]
> Cet exemple montre comment l’accès au coffre de clés est verrouillé en production. Les développeurs doivent avoir leur propre abonnement ou groupe de ressources avec des autorisations complètes pour gérer leurs coffres de clés, les machines virtuelles et le compte de stockage où ils développent l’application.

## <a name="resources"></a>Ressources

- [À propos d’Azure Key Vault](overview.md)
- [Azure Active Directory](../../active-directory/fundamentals/active-directory-whatis.md)
- [Privileged Identity Management](../../active-directory/privileged-identity-management/pim-configure.md)
- [Azure RBAC](../../role-based-access-control/overview.md)
- [Liaison privée](../../private-link/private-link-overview.md)

## <a name="next-steps"></a>Étapes suivantes

[S’authentifier auprès d’Azure Key Vault](authentication.md)

[Attribuer une stratégie d’accès Key Vault](assign-access-policy-portal.md)

[Attribuer un rôle Azure pour accéder aux clés, secrets et certificats](rbac-guide.md)

[Configurer des pare-feu et des réseaux virtuels Key Vault](network-security.md)

[Établir une connexion de liaison privée au coffre de clés](private-link-service.md)
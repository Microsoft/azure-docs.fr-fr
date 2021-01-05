---
title: S’authentifier auprès d’Azure Key Vault
description: Apprendre à s’authentifier auprès d’Azure Key Vault
author: ShaneBala-keyvault
ms.author: sudbalas
ms.date: 08/27/2020
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.openlocfilehash: 455caf0b80d82b03f8d00929addeab15a1af6d7e
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96754213"
---
# <a name="authenticate-to-azure-key-vault"></a>S’authentifier auprès d’Azure Key Vault

Azure Key Vault offre la possibilité de stocker des secrets et de contrôler leur distribution dans un référentiel cloud centralisé et sécurisé, ce qui évite d’avoir à stocker des informations d’identification dans les applications. Il suffit aux applications de s’authentifier auprès de Key Vault à l’exécution pour accéder à ces secrets.

## <a name="app-identity-and-security-principals"></a>Identité et principaux de sécurité des applications

L’authentification auprès de Key Vault fonctionne conjointement avec [Azure Active Directory (Azure AD)](../../active-directory/fundamentals/active-directory-whatis.md), qui est chargé d’authentifier l’identité de chaque **principal de sécurité** donné.

Un principal de sécurité est un objet qui représente un utilisateur, un groupe, un service ou une application demandant l’accès aux ressources Azure. Azure affecte un **ID d’objet** unique à chaque principal de sécurité.

* Un principal de sécurité **utilisateur** identifie une personne disposant d’un profil dans Azure Active Directory.

* Un principal de sécurité **groupe** identifie un ensemble d’utilisateurs créés dans Azure Active Directory. Tous les rôles et autorisations attribués au groupe sont accordés à tous les utilisateurs du groupe.

* Un **principal de service** est un type de principal de sécurité qui identifie une application ou un service, c’est-à-dire un morceau de code plutôt qu’un utilisateur ou un groupe. L’ID d’objet d’un principal de service, appelé **ID client**, lui sert de nom d’utilisateur. La **clé secrète client** du principal de service fonctionne comme un mot de passe.

Pour les applications, il existe deux façons d’obtenir un principal de service :

* Méthode recommandée : activez une **identité managée** affectée par le système pour l’application.

    Avec l’identité managée, Azure gère en interne le principal de service de l’application et authentifie automatiquement l’application auprès d’autres services Azure. L’identité managée est disponible pour les applications déployées sur un large éventail de services.

    Pour plus d’informations, consultez [Vue d’ensemble de l’identité managée](../../active-directory/managed-identities-azure-resources/overview.md). Consultez également [Services Azure qui prennent en charge l’identité managée](../../active-directory/managed-identities-azure-resources/services-support-managed-identities.md), qui comporte des liens vers des articles expliquant comment activer l’identité managée pour des services spécifiques (par exemple, App Service, Azure Functions ou Machines Virtuelles).

* Si vous ne pouvez pas utiliser l’identité managée, **inscrivez** l’application auprès de votre locataire Azure AD (cf. [Démarrage rapide : Inscription d’une application avec la Plateforme d’identités Azure](../../active-directory/develop/quickstart-register-app.md)). L’inscription a également pour effet de créer un deuxième objet d’application qui identifie l’application sur tous les locataires.

## <a name="authorize-a-security-principal-to-access-key-vault"></a>Autoriser un principal de sécurité à accéder à Key Vault

Key Vault fonctionne avec deux niveaux d’autorisation distincts :

- Les **stratégies d’accès** contrôlent si un utilisateur, un groupe ou un principal de service est autorisé à accéder aux secrets, aux clés et aux certificats *au sein* d’une ressource Key Vault existante (opérations parfois appelées « plan de données »). Ces stratégies sont généralement accordées à des utilisateurs, groupes et applications.

    Pour affecter des stratégies d’accès, consultez les articles suivants :

    - [Azure portal](assign-access-policy-portal.md)
    - [Azure CLI](assign-access-policy-cli.md)
    - [Azure PowerShell](assign-access-policy-portal.md)

- Les **autorisations de rôle** contrôlent si un utilisateur, un groupe ou un principal de service est autorisé à créer, à supprimer et, plus généralement, à gérer une ressource Key Vault (opérations parfois appelées « plan de gestion »). Ces rôles sont la plupart du temps accordés uniquement aux administrateurs.
 
    Pour affecter et gérer les rôles, consultez les articles suivants :

    - [Azure portal](../../role-based-access-control/role-assignments-portal.md)
    - [Azure CLI](../../role-based-access-control/role-assignments-cli.md)
    - [Azure PowerShell](../../role-based-access-control/role-assignments-powershell.md)

    Key Vault prend actuellement en charge le rôle [Contributeur](../../role-based-access-control/built-in-roles.md#key-vault-contributor), qui autorise les opérations de gestion sur les ressources Key Vault. D’autres rôles sont actuellement en préversion. Vous pouvez également créer des rôles personnalisés (cf. [Rôles personnalisés Azure](../../role-based-access-control/custom-roles.md)).

    Pour obtenir des informations générales sur les rôles, consultez [Qu’est-ce que le contrôle d’accès en fonction du rôle Azure (RBAC Azure) ?](../../role-based-access-control/overview.md)


> [!IMPORTANT]
> Pour une sécurité optimale, suivez toujours le principe des privilèges minimum : accordez uniquement les stratégies d’accès et les rôles les plus spécifiques nécessaires. 
    
## <a name="configure-the-key-vault-firewall"></a>Configuration du pare-feu Key Vault

Par défaut, Key Vault autorise l’accès aux ressources par le biais d’adresses IP publiques. Pour une sécurité optimale, vous pouvez aussi limiter l’accès à des plages d’adresses IP, des points de terminaison de service, des réseaux virtuels ou des points de terminaison privés spécifiques.

Pour plus d’informations, consultez [Accès à Azure Key Vault derrière un pare-feu](./access-behind-firewall.md).


## <a name="the-key-vault-authentication-flow"></a>Flux d’authentification Key Vault

1. Un principal de service demande à s’authentifier auprès d’Azure AD, par exemple :
    * Un utilisateur se connecte au Portail Azure avec un nom d’utilisateur et un mot de passe.
    * Une application appelle une API REST Azure, en présentant un ID client et une clé secrète ou un certificat client.
    * Une ressource Azure, comme une machine virtuelle avec identité managée, contacte le point de terminaison REST [Azure Instance Metadata Service (IMDS)](../../virtual-machines/windows/instance-metadata-service.md) pour obtenir un jeton d’accès.

1. Si l’authentification auprès d’Azure AD réussit, un jeton OAuth est accordé au principal de service.

1. Le principal de service effectue un appel à l’API REST Key Vault par le biais du point de terminaison (URI) de Key Vault.

1. Le pare-feu Key Vault vérifie les critères suivants. Si un critère est respecté, l’appel est autorisé. Dans le cas contraire, l’appel est bloqué et une réponse Interdit est retournée.

    * Le pare-feu Key Vault est désactivé et le point de terminaison public de Key Vault est accessible sur l’Internet public.
    * L’appelant est un [service approuvé Key Vault](./overview-vnet-service-endpoints.md#trusted-services), ce qui lui permet de contourner le pare-feu.
    * L’appelant est listé dans le pare-feu par adresse IP, réseau virtuel ou point de terminaison de service.
    * L’appelant peut accéder à Key Vault au moyen d’une connexion de liaison privée configurée.    

1. Si le pare-feu autorise l’appel, Key Vault appelle Azure AD pour valider le jeton d’accès du principal de service.

1. Key Vault vérifie si le principal de service dispose de la stratégie d’accès nécessaire pour l’opération demandée. Si ce n’est pas le cas, Key Vault retourne une réponse Interdit.

1. Key Vault effectue l’opération demandée et retourne le résultat.

Le diagramme suivant illustre le processus pour une application qui appelle une API Key Vault « Obtenir le secret » :

![Flux d’authentification Azure Key Vault](../media/authentication/authentication-flow.png)

> [!NOTE]
> Les clients du SDK Key Vault pour les secrets, les certificats et les clés effectuent un appel supplémentaire en direction de Key Vault sans jeton d’accès, ce qui entraîne une réponse 401 pour récupérer les informations du locataire. Pour plus d’informations, consultez [Authentification, requêtes et réponses](authentication-requests-and-responses.md).

## <a name="code-examples"></a>Exemples de code

Le tableau suivant contient des liens vers différents articles qui montrent comment utiliser Key Vault dans le code d’application à l’aide des bibliothèques Azure SDK du langage en question. D’autres interfaces, comme Azure CLI et le Portail Azure, sont incluses pour des raisons pratiques.

| Secrets Key Vault | Clés Key Vault | Certificats Key Vault |
|  --- | --- | --- |
| [Python](../secrets/quick-create-python.md) | [Python](../keys/quick-create-python.md) | [Python](../certificates/quick-create-python.md) | 
| [.NET (SDK v4)](../secrets/quick-create-net.md) | -- | -- |
| [.NET (SDK v3)](https://dotnet.microsoft.com/download/dotnet-core/3.0) | -- | -- |
| [Java](../secrets/quick-create-java.md) | -- | -- |
| [JavaScript](../secrets/quick-create-node.md) | -- | -- | 
| | | |
| [Azure portal](../secrets/quick-create-portal.md) | [Azure portal](../keys/quick-create-portal.md) | [Azure portal](../certificates/quick-create-portal.md) |
| [Azure CLI](../secrets/quick-create-cli.md) | [Azure CLI](../keys/quick-create-cli.md) | [Azure CLI](../certificates/quick-create-cli.md) |
| [Azure PowerShell](../secrets/quick-create-powershell.md) | [Azure PowerShell](../keys/quick-create-powershell.md) | [Azure PowerShell](../certificates/quick-create-powershell.md) |
| [Modèle ARM](../secrets/quick-create-net.md) | -- | -- |

## <a name="next-steps"></a>Étapes suivantes

- [Résolution des problèmes de stratégie d’accès à Key Vault](troubleshooting-access-issues.md)
- [Codes d'erreur de l'API REST Key Vault](rest-error-codes.md)
- [Guide du développeur Key Vault](developers-guide.md)
- [Qu’est-ce que le contrôle d’accès en fonction du rôle Azure (RBAC Azure) ?](../../role-based-access-control/overview.md)

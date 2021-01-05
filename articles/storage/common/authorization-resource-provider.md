---
title: Utiliser le fournisseur de ressources Stockage Azure pour accéder aux ressources de gestion
description: Le fournisseur de ressources Stockage Azure est un service qui permet d’accéder aux ressources de gestion pour Stockage Azure. Vous pouvez utiliser le fournisseur de ressources Stockage Azure pour créer, mettre à jour, gérer et supprimer des ressources comme des comptes de stockage, des points de terminaison privés et des clés d’accès de compte.
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 12/12/2019
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.openlocfilehash: fcf3e9228c8e651efb8f97067f7ba9eead5959db
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92789673"
---
# <a name="use-the-azure-storage-resource-provider-to-access-management-resources"></a>Utiliser le fournisseur de ressources Stockage Azure pour accéder aux ressources de gestion

Azure Resource Manager est le service de déploiement et de gestion d’Azure. Le fournisseur de ressources Stockage Azure est un service basé sur Azure Resource Manager qui permet d’accéder aux ressources de gestion pour Stockage Azure. Vous pouvez utiliser le fournisseur de ressources Stockage Azure pour créer, mettre à jour, gérer et supprimer des ressources comme des comptes de stockage, des points de terminaison privés et des clés d’accès de compte. Pour plus d’informations sur Azure Resource Manager, consultez [Vue d’ensemble d’Azure Resource Manager](../../azure-resource-manager/management/overview.md).

Vous pouvez utiliser le fournisseur de ressources Stockage Azure pour effectuer des actions telles que la création ou la suppression d’un compte de stockage, ou l’obtention d’une liste de comptes de stockage dans un abonnement. Pour autoriser les demandes auprès du fournisseur de ressources Stockage Azure, utilisez Azure Active Directory (Azure AD). Cet article explique comment affecter des autorisations à des ressources de gestion et pointe vers des exemples qui montrent comment effectuer des demandes auprès du fournisseur de ressources Stockage Azure.

## <a name="management-resources-versus-data-resources"></a>Ressources de gestion et ressources de données

Microsoft fournit deux API REST pour l’utilisation des ressources Stockage Azure. Ces API constituent la base de toutes les actions que vous pouvez effectuer sur Stockage Azure. L’API REST de Stockage Azure vous permet d’utiliser les données de votre compte de stockage, y compris les données des objets blob, des files d’attente, des fichiers et des tables. L’API REST du fournisseur de ressources Stockage Azure vous permet d’utiliser le compte de stockage et les ressources associées.

Une demande qui lit ou écrit des données d’objet blob nécessite des autorisations différentes d’une demande qui effectue une opération de gestion. Azure RBAC fournit un contrôle précis sur les autorisations pour les deux types de ressources. Quand vous affectez un rôle Azure à un principal de sécurité, veillez à bien comprendre les autorisations qui seront accordées à ce principal. Pour des informations de référence détaillées décrivant les actions associées à chaque rôle Azure intégré, consultez [Rôles Azure intégrés](../../role-based-access-control/built-in-roles.md).

Stockage Azure prend en charge l’utilisation d’Azure AD pour autoriser les demandes sur Stockage Blob et Stockage File d’attente. Pour plus d’informations sur les rôles Azure pour les opérations sur des données d’objet blob et de file d’attente, consultez [Autoriser l’accès aux objets blob et aux files d’attente avec Active Directory](storage-auth-aad.md).

## <a name="assign-management-permissions-with-azure-role-based-access-control-azure-rbac"></a>Attribuer des autorisations de gestion avec le contrôle d’accès Azure en fonction du rôle (Azure RBAC)

Chaque abonnement Azure est associé à un annuaire Azure Active Directory qui gère les utilisateurs, les groupes et les applications. Un utilisateur, un groupe ou une application est également appelé « principal de sécurité » dans le contexte de la [Plateforme d’identité Microsoft](../../active-directory/develop/index.yml). Vous pouvez accorder l’accès aux ressources d’un abonnement à un principal de sécurité défini dans l’annuaire Active Directory en utilisant le contrôle d’accès Azure en fonction du rôle (Azure RBAC).

Quand vous attribuez un rôle Azure à un principal de sécurité, vous indiquez également l’étendue à laquelle les autorisations accordées par le rôle sont appliquées. Pour les opérations de gestion, vous pouvez attribuer un rôle au niveau de l’abonnement, du groupe de ressources ou du compte de stockage. Vous pouvez attribuer un rôle Azure à un principal de sécurité en utilisant le [portail Azure](https://portal.azure.com/), les [outils Azure CLI](/cli/azure/install-classic-cli), [PowerShell](/powershell/azure/) ou l’[API REST du fournisseur de ressources Stockage Azure](/rest/api/storagerp).

Pour plus d’informations, consultez [Qu’est-ce que le contrôle d’accès en fonction du rôle Azure (Azure RBAC) ?](../../role-based-access-control/overview.md) et [Rôles Administrateur d’abonnement classique, rôles Azure et rôles Administrateur Azure AD](../../role-based-access-control/rbac-and-directory-admin-roles.md).

### <a name="built-in-roles-for-management-operations"></a>Rôles intégrés pour les opérations de gestion

Azure fournit des rôles intégrés qui accordent des autorisations pour appeler les opérations de gestion. Stockage Azure fournit également des rôles intégrés spécifiquement destinés à être utilisés avec le fournisseur de ressources Stockage Azure.

Les rôles intégrés qui accordent des autorisations pour appeler des opérations de gestion du stockage incluent les rôles décrits dans le tableau suivant :

|    Rôle Azure    |    Description    |    Inclut l’accès aux clés de compte ?    |
|---------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------|
| **Propriétaire** | Peut gérer toutes les ressources de stockage et accéder aux ressources.  | Oui, fournit des autorisations pour voir et regénérer les clés de compte de stockage. |
| **Contributeur**  | Peut gérer toutes les ressources de stockage, mais ne peut pas gérer l’accès aux ressources. | Oui, fournit des autorisations pour voir et regénérer les clés de compte de stockage. |
| **Lecteur** | Peut voir des informations sur le compte de stockage, mais ne peut pas voir les clés de compte. | Non. |
| **Contributeur de compte de stockage** | Peut gérer le compte de stockage, obtenir des informations sur les ressources et les groupes de ressources de l’abonnement, et créer et gérer les déploiements des groupes de ressources de l’abonnement. | Oui, fournit des autorisations pour voir et regénérer les clés de compte de stockage. |
| **Administrateur de l'accès utilisateur** | Peut gérer l’accès au compte de stockage.   | Oui, permet à un principal de sécurité d’attribuer des autorisations à lui-même et à d’autres. |
| **Contributeur de machine virtuelle** | Peut gérer les machines virtuelles, mais pas le compte de stockage auquel elles sont connectées.   | Oui, fournit des autorisations pour voir et regénérer les clés de compte de stockage. |

La troisième colonne du tableau indique si le rôle intégré prend en charge **Microsoft.Storage/storageAccounts/listkeys/action** . Cette action accorde des autorisations pour lire et regénérer les clés de compte de stockage. Les autorisations d’accès aux ressources de gestion Stockage Azure n’incluent pas non plus des autorisations d’accès aux données. Cependant, si un utilisateur a accès aux clés de compte, il peut utiliser les clés de compte pour accéder aux données de Stockage Azure via une autorisation de clé partagée.

### <a name="custom-roles-for-management-operations"></a>Rôles personnalisés pour les opérations de gestion

Azure prend également en charge la définition de rôles Azure personnalisés pour l’accès aux ressources de gestion. Pour plus d’informations sur les rôles personnalisés, consultez [Rôles personnalisés Azure](../../role-based-access-control/custom-roles.md).

## <a name="code-samples"></a>Exemples de code

Pour obtenir des exemples de code qui montrent comment autoriser et appeler des opérations de gestion à partir des bibliothèques de gestion de Stockage Azure, consultez les exemples suivants :

- [.NET](https://github.com/Azure-Samples/storage-dotnet-resource-provider-getting-started)
- [Java](https://github.com/Azure-Samples/storage-java-manage-storage-accounts)
- [Node.JS](https://github.com/Azure-Samples/storage-node-resource-provider-getting-started)
- [Python](https://github.com/Azure-Samples/storage-python-manage)

## <a name="azure-resource-manager-versus-classic-deployments"></a>Déploiements Azure Resource Manager et déploiements classiques

Resource Manager et les modèles de déploiement classique représentent deux façons différentes de déployer et de gérer vos solutions Azure. Microsoft recommande d’utiliser le modèle de déploiement Azure Resource Manager quand vous créez un compte de stockage. Si possible, Microsoft vous recommande également de recréer les comptes de stockage classiques existants avec le modèle Resource Manager. Bien que vous puissiez créer un compte de stockage en utilisant le modèle de déploiement classique, le modèle classique est moins flexible et il sera au final déprécié.

Pour plus d’informations sur les modèles de déploiement Azure, consultez [Déploiement Resource Manager et déploiement classique](../../azure-resource-manager/management/deployment-models.md).

## <a name="next-steps"></a>Étapes suivantes

- [Présentation d’Azure Resource Manager](../../azure-resource-manager/management/overview.md)
- [Qu’est-ce que le contrôle d’accès en fonction du rôle Azure (RBAC Azure) ?](../../role-based-access-control/overview.md)
- [Objectifs d’extensibilité pour le fournisseur de ressources de stockage Azure](scalability-targets-resource-provider.md)
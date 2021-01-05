---
title: Contrôle d’accès HSM managé Azure
description: Gérez les autorisations d’accès pour le HSM managé Azure et les clés. Couvre le modèle d’authentification et d’autorisation pour HSM managé, et explique comment sécuriser vos HSM.
services: key-vault
author: amitbapat
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: managed-hsm
ms.topic: conceptual
ms.date: 09/15/2020
ms.author: ambapat
ms.openlocfilehash: 816941fe0ec3a81c41da56acedcedf2de7febe74
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94445232"
---
# <a name="managed-hsm-access-control"></a>Contrôle d’accès HSM managé

> [!NOTE]
> Le fournisseur de ressources Key Vault prend en charge deux types de ressources : les **coffres** et les **HSM managés**. Le contrôle d’accès décrit dans cet article s’applique uniquement aux **HSM managés**. Pour plus d’informations sur le contrôle d’accès pour HSM managé, consultez [Donner accès aux clés, certificats et secrets du coffre de clés avec un contrôle d’accès en fonction du rôle Azure](../general/rbac-guide.md).

Le HSM managé Azure Key Vault est un service cloud qui protège les clés de chiffrement. Comme il s’agit de données sensibles et critiques, vous devez sécuriser l’accès à vos HSM managés en permettant seulement aux applications et utilisateurs autorisés d’y accéder. Cet article fournit une vue d’ensemble du modèle de contrôle d’accès HSM managé. Il décrit l’authentification et l’autorisation, puis explique comment sécuriser l’accès à vos HSM managés.

## <a name="access-control-model"></a>Modèle de contrôle d'accès

L’accès à un HSM managé est contrôlé par le biais de deux interfaces : le **plan de gestion** et le **plan de données**. Le plan de gestion vous permet de gérer le HSM. Les opérations de ce plan incluent la création et la suppression de HSM managés, ainsi que la récupération des propriétés des HSM managés. Le plan de données correspond à l’emplacement où vous utilisez les données stockées dans un HSM managé, à savoir des clés de chiffrement sauvegardées par HSM. Vous pouvez ajouter, supprimer, modifier et utiliser des clés pour effectuer des opérations de chiffrement, gérer les attributions de rôles pour contrôler l’accès aux clés, créer une sauvegarde HSM complète, restaurer une sauvegarde complète et gérer le domaine de sécurité à partir de l’interface du plan de données.

Pour accéder à un HSM managé dans l’un ou l’autre de ces plans, tout appelant doit être authentifié et autorisé. L’authentification établit l’identité de l’appelant. L’autorisation détermine les opérations que l’appelant peut exécuter. Un appelant peut être l’un des [principaux de sécurité](../../role-based-access-control/overview.md#security-principal) définis dans Azure Active Directory : utilisateur, groupe, principal de service ou identité managée.

Les deux plans utilisent Azure Active Directory pour l’authentification. Pour l’autorisation, ils utilisent différents systèmes, comme suit :
- Le plan de gestion utilise le contrôle d’accès en fonction du rôle Azure (Azure RBAC), un système d’autorisation basé sur Azure Resource Manager. 
- Le plan de données utilise le contrôle d’accès en fonction du rôle au niveau du HSM managé, un système d’autorisation implémenté et appliqué au niveau du HSM managé.

Lorsqu’un HSM managé est créé, le demandeur fournit également une liste d’administrateurs de plan de données (tous les [principaux de sécurité](../../role-based-access-control/overview.md#security-principal) sont pris en charge). Seuls ces administrateurs sont en mesure d’accéder au plan de données du HSM managé pour effectuer des opérations sur les clés et gérer les attributions de rôles de plan de données (RBAC local du HSM managé).

Le modèle d’autorisation des deux plans utilise la même syntaxe, mais il est appliqué à des niveaux différents, et les attributions de rôles utilisent des étendues différentes. L’Azure RBAC du plan de gestion est appliqué par Azure Resource Manager tandis que le RBAC local du HSM managé est appliqué par le HSM managé lui-même.

> [!IMPORTANT]
> L’octroi d’un accès au plan de gestion de principal de sécurité à un HSM managé ne lui octroie aucun accès au plan de données pour accéder aux clés ou aux attributions de rôle de plan de données RBAC local HSM managé. Cet isolement vise à prévenir une expansion intempestive des privilèges affectant l’accès aux clés stockées dans le HSM managé.

Par exemple, un administrateur d’abonnements (dans la mesure où il dispose de l’autorisation « collaborateur » pour toutes les ressources de l’abonnement) peut supprimer un HSM managé de son abonnement, mais s’il n’a pas accès au plan de données spécifiquement accordé via le RBAC local HSM managé, il ne peut pas accéder aux clés ou gérer l’attribution de rôle dans le HSM managé pour lui ou d’autres utilisateurs.

## <a name="azure-active-directory-authentication"></a>Authentification Azure Active Directory

Lorsqu’un HSM managé est créé dans un abonnement Azure, il est automatiquement associé au client Azure Active Directory de cet abonnement. Tous les appelants dans les deux plans doivent être inscrits auprès de ce locataire et s’authentifier pour accéder au HSM managé.

L’application s’authentifie auprès d’Azure Active Directory avant d’appeler un plan. L’application peut utiliser une [méthode d’authentification prise en charge](../../active-directory/develop/authentication-vs-authorization.md) en fonction du type d’application. L’application acquiert un jeton pour une ressource du plan afin d’y accéder. La ressource est un point de terminaison dans le plan de gestion ou de données, en fonction de l’environnement Azure. L’application utilise le jeton et envoie une demande d’API REST au point de terminaison du HSM managé. Pour en savoir plus, passez en revue le [flux d’authentification intégral](../../active-directory/develop/v2-oauth2-auth-code-flow.md).

L’utilisation d’un même mécanisme d’authentification pour les deux plans présente plusieurs avantages :

- Les organisations peuvent contrôler de manière centralisée l’accès à tous leurs HSM managés.
- Si un utilisateur part, il perd instantanément l’accès à tous les HSM managés de l’organisation.
- Les organisations peuvent personnaliser l’authentification à l’aide des options d’Azure Active Directory (par exemple, pour activer l’authentification multifacteur afin de renforcer la sécurité).

## <a name="resource-endpoints"></a>Points de terminaison de ressource

Les principaux de sécurité accèdent aux plans par le biais de points de terminaison. Les contrôles d’accès pour les deux plans fonctionnent indépendamment. Pour permettre à une application d’utiliser des clés dans un HSM managé, vous accordez l’accès au plan de données à l’aide d’un RBAC local de HSM managé. Pour permettre à un utilisateur d’accéder à la ressource HSM managé pour créer, lire, supprimer, déplacer les HSM managés et modifier d’autres propriétés et balises, vous utilisez Azure RBAC.

Le tableau suivant présente les points de terminaison pour les plans de gestion et de données.

| Plan&nbsp;d’accès | Points de terminaison d’accès | Opérations | Mécanisme de contrôle d’accès |
| --- | --- | --- | --- |
| Plan de gestion | **Mondial :**<br> management.azure.com:443<br> | Créer, lire, mettre à jour, supprimer et déplacer des HSM managés<br>Définir des balises de HSM managé | Azure RBAC |
| Plan de données | **Mondial :**<br> &lt;hsm-name&gt;.vault.azure.net:443<br> | **Clés** : decrypt, encrypt,<br> unwrap, wrap, verify, sign, get, list, update, create, import, delete, backup, restore, purge<br/><br/> **Gestion des rôles de plan de données (RBAC local HSM managé)**   _: répertorier les définitions de rôles, attribuer des rôles, supprimer des attributions de rôles, définir des rôles personnalisés<br/><br/>_ *Sauvegarde/restauration  **: sauvegarde, restauration, vérification des opérations de sauvegarde/restauration de l’état <br/><br/>** Domaine de sécurité** : télécharger et charger le domaine de sécurité | RBAC local HSM managé |
|||||
## <a name="management-plane-and-azure-rbac"></a>Plan de gestion et Azure RBAC

Dans le plan de gestion, vous utilisez Azure RBAC pour autoriser les opérations qu’un appelant peut exécuter. Dans le modèle Azure RBAC, chaque abonnement Azure a une instance d’Azure Active Directory. Vous accordez l’accès aux utilisateurs, groupes et applications de ce répertoire. L’accès accordé permet de gérer les ressources de l’abonnement Azure qui reposent sur le modèle de déploiement Azure Resource Manager. Pour accorder l’accès, utilisez le [portail Azure](https://portal.azure.com/), l’interface [Azure CLI](/cli/azure/install-classic-cli), [Azure PowerShell](/powershell/azureps-cmdlets-docs) ou les [API REST Azure Resource Manager](/rest/api/authorization/roleassignments).

Vous créez un coffre de clés dans un groupe de ressources et gérez l’accès à l’aide d’Azure Active Directory. Vous autorisez des utilisateurs ou des groupes à gérer les coffres de clés dans un groupe de ressources. Vous accordez l’accès à un niveau d’étendue spécifique en attribuant les rôles Azure appropriés. Pour permettre à un utilisateur de gérer des coffres de clés, vous attribuez un rôle `key vault Contributor` prédéfini à l’utilisateur dans une étendue spécifique. Les niveaux d’étendue suivants peuvent être attribués à un rôle Azure :

- **Groupe d'administration** :  un rôle Azure attribué au niveau d’un abonnement s’applique à tous les abonnements au sein de ce groupe d’administration.
- **Abonnement**: Un rôle Azure attribué au niveau d’un abonnement s’applique à tous les groupes de ressources et à toutes les ressources au sein de cet abonnement.
- **Groupe de ressources** : Un rôle Azure attribué au niveau d’un groupe de ressources s’applique à toutes les ressources de ce groupe de ressources.
- **Ressource spécifique** : Un rôle Azure attribué pour une ressource spécifique s’applique à cette ressource. Dans ce cas, la ressource est un coffre de clés spécifique.

Il existe plusieurs rôles prédéfinis. Si un rôle prédéfini ne répond pas à vos besoins, vous pouvez définir votre propre rôle. Pour plus d’informations, consultez [Azure RBAC : pour les ressources Azure](../../role-based-access-control/built-in-roles.md).

## <a name="data-plane-and-managed-hsm-local-rbac"></a>Plan de données et RBAC local HSM managé

Vous octroyez à un principal de sécurité l’accès pour exécuter des opérations de clé spécifiques en attribuant un rôle. Pour chaque attribution de rôle, vous devez spécifier un rôle et une étendue auxquels s’applique cette attribution. Pour le RBAC local HSM managé, deux étendues sont disponibles.

- **"/" ou "/keys"**  : Étendue de niveau HSM. Les principaux de sécurité auxquels un rôle a été attribué à cette étendue peuvent effectuer les opérations définies dans le rôle pour tous les objets (clés) au sein du HSM managé.
- **"/keys/&lt;key-name&gt;"**  : Étendue de niveau clé. Les principaux de sécurité auxquels un rôle a été attribué à cette étendue peuvent effectuer les opérations définies dans ce rôle pour toutes les versions de la clé spécifiée uniquement.

## <a name="next-steps"></a>Étapes suivantes

- Pour un tutoriel de prise en main destiné aux administrateurs, consultez [Qu’est-ce que HSM managé ?](overview.md).
- Pour un tutoriel de gestion des rôles, consultez [RBAC local HSM managé](role-management.md).
- Pour plus d’informations sur la journalisation de l’utilisation pour la journalisation de HSM managé, consultez [Journalisation pour HSM managé](logging.md).
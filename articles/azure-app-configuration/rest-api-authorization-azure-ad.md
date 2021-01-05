---
title: API REST Azure App Configuration – Autorisation Azure Active Directory
description: Utiliser Azure Active Directory pour l’autorisation auprès d’Azure App Configuration à l’aide de l’API REST
author: AlexandraKemperMS
ms.author: alkemper
ms.service: azure-app-configuration
ms.topic: reference
ms.date: 08/17/2020
ms.openlocfilehash: f144c1f09f87e0b915daf86ba0391c2934e60095
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96932674"
---
# <a name="azure-active-directory-authorization---rest-api-reference"></a>Autorisation Azure Active Directory – Référence d’API REST

Quand vous utilisez l’authentification Azure Active Directory (Azure AD), l’autorisation est gérée par le contrôle d’accès en fonction du rôle (RBAC). Le contrôle d’accès en fonction du rôle exige l’attribution des utilisateurs à des rôles pour accorder l’accès aux ressources. Chaque rôle contient un ensemble d’actions que les utilisateurs attribués au rôle sont en mesure d’effectuer.

## <a name="roles"></a>Rôles

Les rôles suivants sont disponibles dans les abonnements Azure par défaut :

- **Propriétaire des données Azure App Configuration** : ce rôle donne un accès complet à toutes les opérations.
- **Lecteur des données Azure App Configuration** : ce rôle permet les opérations de lecture.

## <a name="actions"></a>Actions

Les rôles contiennent une liste d’actions que les utilisateurs qui y sont assignés peuvent effectuer. Azure App Configuration prend en charge les actions suivantes :

- `Microsoft.AppConfiguration/configurationStores/keyValues/read` : cette action autorise l’accès en lecture aux ressources de clé-valeur d’App Configuration, telles que /kv et /labels.
- `Microsoft.AppConfiguration/configurationStores/keyValues/write` : cette action autorise l’accès en écriture aux ressources de clé-valeur d’App Configuration.
- `Microsoft.AppConfiguration/configurationStores/keyValues/delete` : Cette action autorise la suppression de ressources de clé-valeur d’App Configuration. Notez que la suppression d’une ressource renvoie la clé-valeur supprimée.

## <a name="error"></a>Error

```http
HTTP/1.1 403 Forbidden
```

**Raison :** le principal qui effectue la demande n’a pas les autorisations requises pour effectuer l’opération demandée.
**Solution :** attribuez le rôle requis pour effectuer l’opération demandée au principal qui effectue la demande.

## <a name="managing-role-assignments"></a>Gestion des attributions de rôles

Vous pouvez gérer les attributions de rôles à l’aide des [procédures RBAC](../role-based-access-control/overview.md) qui sont standard dans l’ensemble des services Azure. Pour ce faire, vous pouvez utiliser l’interface Azure CLI, PowerShell ou le portail Azure. Pour plus d’informations, consultez [Ajouter ou supprimer des attributions de rôles Azure à l’aide du portail Azure](../role-based-access-control/role-assignments-portal.md).
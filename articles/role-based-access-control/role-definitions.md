---
title: Comprendre les définitions de rôles Azure - RBAC Azure
description: Apprenez-en davantage sur les définitions de rôles Azure dans le cadre du contrôle d’accès en fonction du rôle (RBAC) Azure pour une gestion affinée des accès aux ressources Azure.
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.assetid: ''
ms.service: role-based-access-control
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/08/2020
ms.author: rolyon
ms.reviewer: bagovind
ms.custom: ''
ms.openlocfilehash: bc3640fecbe1138e46fd0d36975691740bc669dd
ms.sourcegitcommit: 1bdcaca5978c3a4929cccbc8dc42fc0c93ca7b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/13/2020
ms.locfileid: "97369257"
---
# <a name="understand-azure-role-definitions"></a>Comprendre les définitions de rôles Azure

Si vous essayez de comprendre comment un rôle Azure fonctionne, ou si vous créez votre propre [rôle personnalisé Azure](custom-roles.md), il est utile de comprendre la façon dont les rôles sont définis. Cet article décrit en détail les définitions de rôles et fournit quelques exemples.

## <a name="role-definition"></a>Définition de rôle

Une *définition de rôle* est une collection d’autorisations. On l’appelle parfois simplement *rôle*. Une définition de rôle répertorie les opérations qui peuvent être effectuées, telles que lire, écrire et supprimer. Elle permet également de répertorier les opérations qui sont exclues des opérations autorisées ou des opérations liées aux données sous-jacentes.

L’exemple suivant illustre les propriétés d’une définition de rôle lorsqu’elle est affichée à l’aide d’Azure PowerShell :

```
Name
Id
IsCustom
Description
Actions []
NotActions []
DataActions []
NotDataActions []
AssignableScopes []
```

L’exemple suivant illustre les propriétés d’une définition de rôle lorsqu’elle est affichée à l’aide du Portail Azure, d’Azure CLI ou de l’API REST :

```
roleName
name
type
description
actions []
notActions []
dataActions []
notDataActions []
assignableScopes []
```

Le tableau suivant décrit ce que signifient les propriétés de rôle.

| Propriété | Description |
| --- | --- |
| `Name`</br>`roleName` | Nom d’affichage du rôle. |
| `Id`</br>`name` | ID unique du rôle. |
| `IsCustom`</br>`roleType` | Indique s’il s’agit d’un rôle personnalisé. À définir sur `true` ou `CustomRole` pour les rôles personnalisés. À définir sur `false` ou `BuiltInRole` pour les rôles intégrés. |
| `Description`</br>`description` | Description du rôle. |
| `Actions`</br>`actions` | Tableau de chaînes qui spécifie les opérations d’administration que le rôle autorise. |
| `NotActions`</br>`notActions` | Tableau de chaînes qui spécifie les opérations d’administration exclues des `Actions` autorisées. |
| `DataActions`</br>`dataActions` | Tableau de chaînes qui spécifie les opérations de données que le rôle autorise sur vos données au sein de cet objet. |
| `NotDataActions`</br>`notDataActions` | Tableau de chaînes qui spécifie les opérations de données exclues des `DataActions` autorisées. |
| `AssignableScopes`</br>`assignableScopes` | Tableau de chaînes qui spécifie les étendues pour lesquelles le rôle est disponible à des fins d’attribution. |

### <a name="operations-format"></a>Format des opérations

Les opérations sont spécifiées à l’aide de chaînes dont le format est le suivant :

- `{Company}.{ProviderName}/{resourceType}/{action}`

La portion `{action}` d’une chaîne d’opération spécifie le type des opérations que vous pouvez effectuer sur un type de ressource. Par exemple, vous verrez les sous-chaînes suivantes dans `{action}` :

| Sous-chaîne d’action    | Description         |
| ------------------- | ------------------- |
| `*` | Le caractère générique donne accès à toutes les opérations qui correspondent à la chaîne. |
| `read` | Permet les opérations de lecture (GET). |
| `write` | Permet les opérations d’écriture (PUT ou PATCH). |
| `action` | Permet des opérations personnalisées, telles que le redémarrage de machines virtuelles (POST). |
| `delete` | Permet les opérations de suppression (DELETE). |

### <a name="role-definition-example"></a>Exemple de définition de rôle

Voici la définition de rôle [Contributeur](built-in-roles.md#contributor) comme indiqué dans Azure PowerShell et Azure CLI. L’opération de caractère générique (`*`) sous `Actions` indique que le principal affecté à ce rôle peut effectuer toutes les actions, ou, en d’autres termes, tout gérer. Cela inclut les actions qui seront définies dans le futur, à mesure qu’Azure ajoutera de nouveaux types de ressources. Les opérations sous `NotActions` sont soustraites de `Actions`. Dans le cas du rôle [Contributeur](built-in-roles.md#contributor), `NotActions` supprime la possibilité pour ce rôle de gérer l’accès aux ressources et gère également les affectations Azure Blueprint.

Rôle contributeur tel qu’il s’affiche dans Azure PowerShell :

```json
{
  "Name": "Contributor",
  "Id": "b24988ac-6180-42a0-ab88-20f7382dd24c",
  "IsCustom": false,
  "Description": "Lets you manage everything except access to resources.",
  "Actions": [
    "*"
  ],
  "NotActions": [
    "Microsoft.Authorization/*/Delete",
    "Microsoft.Authorization/*/Write",
    "Microsoft.Authorization/elevateAccess/Action",
    "Microsoft.Blueprint/blueprintAssignments/write",
    "Microsoft.Blueprint/blueprintAssignments/delete"
  ],
  "DataActions": [],
  "NotDataActions": [],
  "AssignableScopes": [
    "/"
  ]
}
```

Rôle contributeur tel qu’il s’affiche dans Azure CLI :

```json
{
  "assignableScopes": [
    "/"
  ],
  "description": "Lets you manage everything except access to resources.",
  "id": "/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c",
  "name": "b24988ac-6180-42a0-ab88-20f7382dd24c",
  "permissions": [
    {
      "actions": [
        "*"
      ],
      "notActions": [
        "Microsoft.Authorization/*/Delete",
        "Microsoft.Authorization/*/Write",
        "Microsoft.Authorization/elevateAccess/Action",
        "Microsoft.Blueprint/blueprintAssignments/write",
        "Microsoft.Blueprint/blueprintAssignments/delete"
      ],
      "dataActions": [],
      "notDataActions": []
    }
  ],
  "roleName": "Contributor",
  "roleType": "BuiltInRole",
  "type": "Microsoft.Authorization/roleDefinitions"
}
```

## <a name="management-and-data-operations"></a>Opérations de gestion et sur les données

Le contrôle d’accès en fonction du rôle pour les opérations de gestion est spécifié dans les propriétés `Actions` et `NotActions` d’une définition de rôle. Voici quelques exemples d’opérations de gestion dans Azure :

- Gérer l’accès à un compte de stockage
- Créer, mettre à jour ou supprimer un conteneur d’objets blob
- Supprimer un groupe de ressources et toutes ses ressources

L’accès à la gestion n’est pas hérité de vos données à condition que la méthode d’authentification du conteneur soit définie sur « Compte d’utilisateur Azure AD » et non sur « Clé d’accès ». Cette séparation empêche des rôles avec caractères génériques (`*`) d’avoir un accès illimité à vos données. Par exemple, si un utilisateur a un rôle [Lecteur](built-in-roles.md#reader) sur un abonnement, il peut afficher le compte de stockage, mais pas les données sous-jacentes, par défaut.

Auparavant, le contrôle d’accès en fonction du rôle n’était pas utilisé pour les opérations sur les données. L’autorisation pour les opérations sur les données variait selon les fournisseurs de ressources. Le même modèle d’autorisation du contrôle d’accès en fonction du rôle utilisé pour les opérations de gestion a été étendu aux opérations sur les données.

Pour prendre en charge les opérations sur les données, de nouvelles propriétés de données ont été ajoutées à la définition de rôle. Les opérations sur les données sont spécifiées dans les propriétés `DataActions` et `NotDataActions`. En ajoutant ces propriétés de données, la séparation entre la gestion et les données est conservée. Cela empêche les attributions de rôle contenant des caractères génériques (`*`) d’accéder soudainement aux données. Voici quelques opérations sur les données qui peuvent être spécifiées dans `DataActions` et `NotDataActions` :

- Lire une liste d’objets blob dans un conteneur
- Écrire un objet blob de stockage dans un conteneur
- Supprimer un message dans une file d’attente

Voici la définition de rôle [Lecteur des données blob du stockage](built-in-roles.md#storage-blob-data-reader), qui inclut des opérations à la fois dans les propriétés `Actions` et `DataActions`. Ce rôle vous permet de lire le conteneur d’objets blob ainsi que les données d’objets blob sous-jacentes.

Rôle Lecteur des données blob du stockage tel qu’il est affiché dans Azure PowerShell :

```json
{
  "Name": "Storage Blob Data Reader",
  "Id": "2a2b9908-6ea1-4ae2-8e65-a410df84e7d1",
  "IsCustom": false,
  "Description": "Allows for read access to Azure Storage blob containers and data",
  "Actions": [
    "Microsoft.Storage/storageAccounts/blobServices/containers/read",
    "Microsoft.Storage/storageAccounts/blobServices/generateUserDelegationKey/action"
  ],
  "NotActions": [],
  "DataActions": [
    "Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read"
  ],
  "NotDataActions": [],
  "AssignableScopes": [
    "/"
  ]
}
```

Rôle Lecteur des données blob de stockage tel qu’il est affiché dans Azure CLI :

```json
{
  "assignableScopes": [
    "/"
  ],
  "description": "Allows for read access to Azure Storage blob containers and data",
  "id": "/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/roleDefinitions/2a2b9908-6ea1-4ae2-8e65-a410df84e7d1",
  "name": "2a2b9908-6ea1-4ae2-8e65-a410df84e7d1",
  "permissions": [
    {
      "actions": [
        "Microsoft.Storage/storageAccounts/blobServices/containers/read",
        "Microsoft.Storage/storageAccounts/blobServices/generateUserDelegationKey/action"
      ],
      "notActions": [],
      "dataActions": [
        "Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read"
      ],
      "notDataActions": []
    }
  ],
  "roleName": "Storage Blob Data Reader",
  "roleType": "BuiltInRole",
  "type": "Microsoft.Authorization/roleDefinitions"
}
```

Seules des opérations sur les données peuvent être ajoutées aux propriétés `DataActions` et `NotDataActions`. Les fournisseurs de ressources identifient quelles opérations sont des opérations sur les données, en définissant la propriété `isDataAction` sur `true`. Pour afficher la liste des opérations où `isDataAction` est `true`, consultez [Opérations de fournisseur de ressources](resource-provider-operations.md). Les rôles qui n’ont pas d’opérations sur les données ne sont pas obligés d’avoir les propriétés `DataActions` et `NotDataActions` dans la définition de rôle.

L’autorisation pour tous les appels d’API des opérations de gestion est gérée par Azure Resource Manager. L’autorisation pour les appels d’API des opérations sur les données est gérée par un fournisseur de ressources ou Azure Resource Manager.

### <a name="data-operations-example"></a>Exemple d’opérations sur les données

Pour mieux comprendre comment fonctionnent les opérations de gestion et les opérations sur les données, prenons un exemple spécifique. Alice a reçu le rôle [Propriétaire](built-in-roles.md#owner) au niveau de l’étendue de l’abonnement. Bob a reçu le rôle [Contributeur aux données blob du stockage](built-in-roles.md#storage-blob-data-contributor) dans une étendue de compte de stockage. Le diagramme qui suit présente cet exemple.

![Le contrôle d’accès en fonction du rôle a été étendu pour prendre en charge les opérations de gestion et les opérations sur les données](./media/role-definitions/rbac-management-data.png)

Le rôle [Propriétaire](built-in-roles.md#owner) pour Alice et le rôle [Contributeur aux données blob du stockage](built-in-roles.md#storage-blob-data-contributor) pour Bob effectuent les actions suivantes :

Propriétaire

&nbsp;&nbsp;&nbsp;&nbsp;Actions<br>
&nbsp;&nbsp;&nbsp;&nbsp;`*`

Contributeur aux données Blob du stockage

&nbsp;&nbsp;&nbsp;&nbsp;Actions<br>
&nbsp;&nbsp;&nbsp;&nbsp;`Microsoft.Storage/storageAccounts/blobServices/containers/delete`<br>
&nbsp;&nbsp;&nbsp;&nbsp;`Microsoft.Storage/storageAccounts/blobServices/containers/read`<br>
&nbsp;&nbsp;&nbsp;&nbsp;`Microsoft.Storage/storageAccounts/blobServices/containers/write`<br>
&nbsp;&nbsp;&nbsp;&nbsp;`Microsoft.Storage/storageAccounts/blobServices/generateUserDelegationKey/action`<br>
&nbsp;&nbsp;&nbsp;&nbsp;DataActions<br>
&nbsp;&nbsp;&nbsp;&nbsp;`Microsoft.Storage/storageAccounts/blobServices/containers/blobs/delete`<br>
&nbsp;&nbsp;&nbsp;&nbsp;`Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read`<br>
&nbsp;&nbsp;&nbsp;&nbsp;`Microsoft.Storage/storageAccounts/blobServices/containers/blobs/move/action`<br>
&nbsp;&nbsp;&nbsp;&nbsp;`Microsoft.Storage/storageAccounts/blobServices/containers/blobs/write`

Comme Alice dispose d’une action avec caractère générique (`*`) à une étendue de l’abonnement, elle hérite d’autorisations lui permettant d’effectuer toutes les actions de gestion. Alice peut lire, écrire et supprimer des conteneurs. En revanche, elle ne peut pas effectuer d’opérations sur des données sans passer par des étapes supplémentaires. Par exemple, par défaut, Alice ne peut pas lire les objets blob à l’intérieur d’un conteneur. Pour cela, elle doit récupérer les clés d’accès de stockage et les utiliser pour accéder aux objets blob.

Les autorisations de Bob se limitent aux actions `Actions` et `DataActions` spécifiées dans le rôle [Contributeur aux données blob du stockage](built-in-roles.md#storage-blob-data-contributor). En fonction du rôle, Bob peut effectuer à la fois des opérations de gestion et des opérations sur les données. Par exemple, Bob peut lire, écrire et supprimer des conteneurs du compte de stockage spécifié, mais aussi lire, écrire et supprimer les objets blob.

Pour plus d’informations sur la gestion et la sécurité du plan de données pour le stockage, consultez le [guide de sécurité Stockage Microsoft Azure](../storage/blobs/security-recommendations.md).

### <a name="what-tools-support-using-azure-roles-for-data-operations"></a>Quels outils prennent en charge l’utilisation des rôles Azure pour les opérations sur les données ?

Pour afficher et utiliser des opérations sur les données, vous devez disposer des versions appropriées des outils ou des kits de développement logiciel (SDK) :

| Outil  | Version  |
|---------|---------|
| [Azure PowerShell](/powershell/azure/install-az-ps) | 1.1.0 ou ultérieure |
| [Azure CLI](/cli/azure/install-azure-cli) | 2.0.30 ou version ultérieure |
| [Azure pour .NET](/dotnet/azure/) | 2.8.0-preview ou version ultérieure |
| [Kit de développement logiciel (SDK) Azure pour Go](/azure/go/azure-sdk-go-install) | 15.0.0 ou version ultérieure |
| [Azure pour Java](/java/azure/) | 1.9.0 ou version ultérieure |
| [Azure pour Python](/azure/python/) | 0.40.0 ou version ultérieure |
| [Kit de développement logiciel (SDK) Azure pour Ruby](https://rubygems.org/gems/azure_sdk) | 0.17.1 ou version ultérieure |

Pour afficher et utiliser les opérations de données dans l’API REST, vous devez définir le paramètre **api-version** sur la version suivante ou ultérieure :

- 01-07-2018

## <a name="actions"></a>Actions

L’autorisation `Actions` spécifie les opérations d’administration que le rôle autorise. Il s’agit d’un ensemble de chaînes d’opération qui identifient les opérations sécurisables des fournisseurs de ressources Azure. Voici quelques exemples d’opérations de gestion qui peuvent être utilisées dans `Actions`.

> [!div class="mx-tableFixed"]
> | Chaîne d’opération    | Description         |
> | ------------------- | ------------------- |
> | `*/read` | Accorde l’accès aux opérations de lecture pour tous les types de ressources de l’ensemble des fournisseurs de ressources Azure.|
> | `Microsoft.Compute/*` | Accorde l’accès à l’ensemble des opérations pour tous les types de ressources dans le fournisseur de ressources Microsoft.Compute.|
> | `Microsoft.Network/*/read` | Accorde l’accès aux opérations de lecture pour tous les types de ressources dans le fournisseur de ressources Microsoft.Network d’Azure.|
> | `Microsoft.Compute/virtualMachines/*` | Accorde l’accès à toutes les opérations des machines virtuelles et à leurs types de ressources enfants.|
> | `microsoft.web/sites/restart/Action` | Accorde l’accès au redémarrage d’une application web.|

## <a name="notactions"></a>NotActions

L’autorisation `NotActions` spécifie les opérations de gestion qui sont exclues des `Actions` autorisées. Utilisez l’autorisation `NotActions` si l’ensemble des opérations que vous souhaitez autoriser est plus facile à définir en excluant les opérations restreintes. L’accès accordé par un rôle (autorisations effectives) est calculé en soustrayant les opérations `NotActions` des opérations `Actions`.

> [!NOTE]
> Si un utilisateur se voit attribuer un rôle qui exclut une opération dans `NotActions`, et un second rôle qui accorde l’accès à cette même opération, il est autorisé à effectuer celle-ci. `NotActions` n’est pas une règle de refus : il s’agit simplement d’un moyen pratique pour créer un ensemble d’opérations autorisées lorsque des opérations spécifiques doivent être exclues.
>

## <a name="dataactions"></a>DataActions

L’autorisation `DataActions` spécifie les opérations de données que le rôle autorise sur vos données au sein de cet objet. Par exemple, si un utilisateur dispose d’un accès en lecture aux données blob d’un compte de stockage, il peut lire les objets blob de ce compte de stockage. Voici quelques exemples d’opérations sur les données qui peuvent être utilisées dans `DataActions`.

> [!div class="mx-tableFixed"]
> | Chaîne d’opération    | Description         |
> | ------------------- | ------------------- |
> | `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read` | Retourne un objet blob ou une liste d'objets blob. |
> | `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/write` | Retourne le résultat de l'écriture d'un objet blob. |
> | `Microsoft.Storage/storageAccounts/queueServices/queues/messages/read` | Retourne un message. |
> | `Microsoft.Storage/storageAccounts/queueServices/queues/messages/*` | Retourne un message ou le résultat de l’écriture ou de la suppression d’un message. |

## <a name="notdataactions"></a>NotDataActions

L’autorisation `NotDataActions` spécifie les opérations sur les données qui sont exclues des `DataActions` autorisées. L’accès accordé par un rôle (autorisations effectives) est calculé en soustrayant les opérations `NotDataActions` des opérations `DataActions`. Chaque fournisseur de ressources fournit son propre ensemble d’API pour répondre à des opérations sur les données.

> [!NOTE]
> Si un utilisateur se voit attribuer un rôle qui exclut une opération sur les données dans `NotDataActions`, et un second rôle qui accorde l’accès à cette même opération, il est autorisé à effectuer celle-ci. `NotDataActions` n’est pas une règle de refus : il s’agit simplement d’un moyen pratique pour créer un ensemble d’opérations sur les données autorisées lorsque des opérations sur les données spécifiques doivent être exclues.
>

## <a name="assignablescopes"></a>AssignableScopes

La propriété `AssignableScopes` spécifie les étendues (groupes d’administration, abonnements ou groupes de ressources) qui disposent de cette définition de rôle. Vous pouvez mettre le rôle à disposition pour pouvoir l’attribuer uniquement dans les groupes d’administration, les abonnements ou les groupes de ressources qui en ont besoin. Vous devez utiliser au moins un groupe de gestion, un abonnement ou un groupe de ressources.

La chaîne `AssignableScopes` est définie sur l’étendue racine (`"/"`) pour les rôles intégrés. L’étendue racine indique que le rôle est disponible pour attribution dans toutes les étendues. Voici des exemples d’étendues assignables valides :

> [!div class="mx-tableFixed"]
> | Le rôle est disponible à l’attribution | Exemple |
> |----------|---------|
> | Abonnement unique | `"/subscriptions/{subscriptionId1}"` |
> | Deux abonnements | `"/subscriptions/{subscriptionId1}", "/subscriptions/{subscriptionId2}"` |
> | Groupe de ressources réseau | `"/subscriptions/{subscriptionId1}/resourceGroups/Network"` |
> | Un seul groupe d’administration | `"/providers/Microsoft.Management/managementGroups/{groupId1}"` |
> | Groupe d’administration et abonnement | `"/providers/Microsoft.Management/managementGroups/{groupId1}", /subscriptions/{subscriptionId1}",` |
> | Toutes les étendues (applicable uniquement aux rôles intégrés) | `"/"` |

Pour plus d’informations sur `AssignableScopes` pour des rôles personnalisés, consultez [Rôles personnalisés Azure](custom-roles.md).

## <a name="next-steps"></a>Étapes suivantes

* [Rôles intégrés Azure](built-in-roles.md)
* [Rôle personnalisés Azure](custom-roles.md)
* [Opérations de fournisseur de ressources Azure](resource-provider-operations.md)

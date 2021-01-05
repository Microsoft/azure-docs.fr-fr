---
title: Gérer les espaces de travail Log Analytics dans Azure Monitor | Microsoft Docs
description: Vous pouvez gérer l’accès aux données stockées dans un espace de travail Log Analytics dans Azure Monitor à l’aide d’autorisations au niveau de la ressource, de l’espace de travail ou de la table. Cet article explique comment procéder.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 04/10/2019
ms.openlocfilehash: 234ef58920a9f896d3e8ebcc561562ea7ceb2708
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96186420"
---
# <a name="manage-access-to-log-data-and-workspaces-in-azure-monitor"></a>Gérer l’accès aux données du journal et les espaces de travail dans Azure Monitor

Azure Monitor stocke les données de [journal](data-platform-logs.md) dans un espace de travail Log Analytics. Un espace de travail est un conteneur qui renferme des données ainsi que des informations de configuration. Pour gérer l’accès aux données de journal, vous accomplissez diverses tâches administratives liées à vos espaces de travail.

Cet article explique comment gérer l’accès aux journaux et administrer les espaces de travail qui les contiennent, notamment comment accorder l’accès : 

* À l’espace de travail, à l’aide d’autorisations d’espace de travail.
* Aux utilisateurs ayant besoin d’accéder à des données de journal de ressources spécifiques à l’aide du Contrôle d’accès en fonction du rôle (RBAC) Azure, également appelé [resource-context](design-logs-deployment.md#access-mode)
* Aux utilisateurs qui doivent pouvoir accéder aux données de journal dans un tableau spécifique de l’espace de travail, à l’aide de rôles RBAC Azure.

Pour comprendre les concepts des journaux en lien avec Azure RBAC et les stratégies d’accès, consultez [Conception de votre déploiement de journaux Azure Monitor](design-logs-deployment.md)

## <a name="configure-access-control-mode"></a>Configurer le mode de contrôle d’accès

Vous pouvez afficher le [mode de contrôle d’accès](design-logs-deployment.md) configuré sur un espace de travail à partir du portail Azure ou avec Azure PowerShell.  Vous pouvez modifier ce paramètre à l’aide de l’une des méthodes prises en charge suivantes :

* Portail Azure

* Azure PowerShell

* Modèle Azure Resource Manager

### <a name="from-the-azure-portal"></a>À partir du portail Azure

Vous pouvez afficher le mode de contrôle d’accès à l’espace de travail actuel dans la page **Vue d’ensemble** de l’espace de travail, dans le menu **Espace de travail Log Analytics**.

![Afficher le mode de contrôle d’accès à l’espace de travail](media/manage-access/view-access-control-mode.png)

1. Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).
1. Dans le portail Azure, sélectionnez Espaces de travail Log Analytics > votre espace de travail.

Vous pouvez changer ce paramètre dans la page **Propriétés** de l’espace de travail. La possibilité de changer le paramètre est désactivée si vous n’êtes pas autorisé à configurer l’espace de travail.

![Changer le mode d’accès à l’espace de travail](media/manage-access/change-access-control-mode.png)

### <a name="using-powershell"></a>Utilisation de PowerShell

Pour examiner le mode de contrôle d’accès pour tous les espaces de travail dans l’abonnement, utilisez la commande suivante :

```powershell
Get-AzResource -ResourceType Microsoft.OperationalInsights/workspaces -ExpandProperties | foreach {$_.Name + ": " + $_.Properties.features.enableLogAccessUsingOnlyResourcePermissions}
```

La sortie doit ressembler à ce qui suit :

```
DefaultWorkspace38917: True
DefaultWorkspace21532: False
```

La valeur `False` indique que l’espace de travail est configuré avec le mode d’accès en fonction du contexte de l’espace de travail.  La valeur `True` indique que l’espace de travail est configuré avec le mode d’accès en fonction du contexte de la ressource.

> [!NOTE]
> Si un espace de travail est retourné sans valeur booléenne et est vide, cela correspond également aux résultats d’une valeur `False`.
>

Utilisez le script suivant pour définir le mode de contrôle d’accès pour un espace de travail spécifique sur l’autorisation en fonction du contexte de la ressource :

```powershell
$WSName = "my-workspace"
$Workspace = Get-AzResource -Name $WSName -ExpandProperties
if ($Workspace.Properties.features.enableLogAccessUsingOnlyResourcePermissions -eq $null)
    { $Workspace.Properties.features | Add-Member enableLogAccessUsingOnlyResourcePermissions $true -Force }
else
    { $Workspace.Properties.features.enableLogAccessUsingOnlyResourcePermissions = $true }
Set-AzResource -ResourceId $Workspace.ResourceId -Properties $Workspace.Properties -Force
```

Utilisez le script suivant pour définir le mode de contrôle d’accès pour tous les espaces de travail dans l’abonnement sur l’autorisation en fonction du contexte de la ressource :

```powershell
Get-AzResource -ResourceType Microsoft.OperationalInsights/workspaces -ExpandProperties | foreach {
if ($_.Properties.features.enableLogAccessUsingOnlyResourcePermissions -eq $null)
    { $_.Properties.features | Add-Member enableLogAccessUsingOnlyResourcePermissions $true -Force }
else
    { $_.Properties.features.enableLogAccessUsingOnlyResourcePermissions = $true }
Set-AzResource -ResourceId $_.ResourceId -Properties $_.Properties -Force
}
```

### <a name="using-a-resource-manager-template"></a>Utilisation d’un modèle Resource Manager

Pour configurer le mode d’accès dans un modèle Azure Resource Manager, définissez sur une des valeurs suivantes l’indicateur de fonctionnalité **enableLogAccessUsingOnlyResourcePermissions** sur l’espace de travail.

* **false** : définir l’espace de travail sur les autorisations en fonction du contexte de l’espace de travail. Il s’agit du paramétrage par défaut si l’indicateur n’est pas défini.
* **true** : définir l’espace de travail sur les autorisations en fonction du contexte de la ressource.

## <a name="manage-access-using-workspace-permissions"></a>Gérer l’accès à l’aide d’autorisations au niveau de l’espace de travail

Chaque espace de travail peut être associé à plusieurs comptes et chaque compte peut également avoir accès à plusieurs espaces de travail. L’accès est géré à l’aide du [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../role-based-access-control/role-assignments-portal.md).

Les activités suivantes nécessitent également des autorisations Azure :

|Action |Autorisations Azure nécessaires |Notes |
|-------|-------------------------|------|
| Ajout et suppression de solutions de supervision | `Microsoft.Resources/deployments/*` <br> `Microsoft.OperationalInsights/*` <br> `Microsoft.OperationsManagement/*` <br> `Microsoft.Automation/*` <br> `Microsoft.Resources/deployments/*/write` | Ces autorisations doivent être accordées au niveau du groupe de ressources ou de l’abonnement. |
| Modification du niveau tarifaire | `Microsoft.OperationalInsights/workspaces/*/write` | |
| Affichage des données dans les mosaïques de solution *Sauvegarde* et *Site Recovery* | Administrateur/coadministrateur | Accède aux ressources déployées à l’aide du modèle de déploiement Classic |
| Gestion d’un espace de travail dans le portail Azure | `Microsoft.Resources/deployments/*` <br> `Microsoft.OperationalInsights/workspaces/*` ||
| Afficher les propriétés de base de l’espace de travail et entrer dans le panneau Espace de travail dans le portail | `Microsoft.OperationalInsights/workspaces/read` ||
| Interroger des journaux à l’aide d’une interface | `Microsoft.OperationalInsights/workspaces/query/read` ||
| Accéder à tous les types de journaux à l’aide de requêtes | `Microsoft.OperationalInsights/workspaces/query/*/read` ||
| Accéder à une table de journaux spécifique | `Microsoft.OperationalInsights/workspaces/query/<table_name>/read` ||
| Lire les clés de l’espace de travail pour autoriser l’envoi des journaux à cet espace de travail | `Microsoft.OperationalInsights/workspaces/sharedKeys/action` ||

## <a name="manage-access-using-azure-permissions"></a>Gérer l’accès à l’aide d’autorisations Azure

Pour accorder l’accès à l’espace de travail Log Analytics à l’aide des autorisations Azure, suivez les étapes de la page [Utiliser les attributions de rôle pour gérer l’accès à vos ressources d’abonnement Azure](../../role-based-access-control/role-assignments-portal.md). Pour voir des exemples de rôles personnalisés, consultez [Exemples de rôles personnalisés](#custom-role-examples).

Azure intègre deux rôles utilisateur pour les espaces de travail Log Analytics :

* Lecteur Log Analytics
* Contributeur Log Analytics

Les membres du rôle *Lecteur Log Analytics* peuvent effectuer les opérations suivantes :

* Visualisation et recherche de toutes les données d’analyse
* Visualisation des paramètres d’analyse, notamment la configuration des diagnostics Azure sur toutes les ressources Azure

Le rôle Lecteur Log Analytics inclut les actions Azure suivantes :

| Type    | Autorisation | Description |
| ------- | ---------- | ----------- |
| Action | `*/read`   | Possibilité de visualiser toutes les ressources Azure et la configuration des ressources. Inclut la visualisation des éléments suivants : <br> État d’extension de machine virtuelle <br> Configuration des diagnostics Azure sur les ressources <br> Totalité des paramètres et propriétés de l’ensemble des ressources. <br> Pour les espaces de travail, il permet aux autorisations illimitées de lire les paramètres de l’espace de travail et d’exécuter des requêtes sur les données. Consultez des options plus granulaires ci-dessus. |
| Action | `Microsoft.OperationalInsights/workspaces/analytics/query/action` | Déprécié. Il n’est pas nécessaire de les affecter aux utilisateurs. |
| Action | `Microsoft.OperationalInsights/workspaces/search/action` | Déprécié. Il n’est pas nécessaire de les affecter aux utilisateurs. |
| Action | `Microsoft.Support/*` | Possibilité d’ouvrir des cas de support |
|Non-action | `Microsoft.OperationalInsights/workspaces/sharedKeys/read` | Interdiction de lecture de la clé d’espace de travail requise pour l’utilisation de l’API de collecte de données et pour l’installation des agents. Cela empêche l’utilisateur d’ajouter de nouvelles ressources à l’espace de travail. |

Les membres du rôle *Contributeur Log Analytics* peuvent effectuer les opérations suivantes :

* Comprend tous les privilèges du *rôle lecteur de Log Analytics*, ce qui permet à l’utilisateur de lire toutes les données d’analyse
* Création et configuration des comptes Automation
* Ajout et suppression de solutions de gestion

    > [!NOTE]
    > Pour réussir ces deux actions, cette autorisation doit être accordée au niveau du groupe de ressources ou de l’abonnement.

* Lecture des clés de compte de stockage
* Configuration de la collecte de journaux à partir de Stockage Azure
* Modification des paramètres d’analyse pour les ressources Azure, notamment :
  * Ajout de l’extension de machine virtuelle à des machines virtuelles
  * Configuration des diagnostics Azure sur toutes les ressources Azure

> [!NOTE]
> La possibilité d’ajouter une extension de machine virtuelle à une machine virtuelle vous offre un contrôle total sur une machine virtuelle.

Le rôle Contributeur Log Analytics inclut les actions Azure suivantes :

| Autorisation | Description |
| ---------- | ----------- |
| `*/read`     | Possibilité de visualiser toutes les ressources et la configuration des ressources. Inclut la visualisation des éléments suivants : <br> État d’extension de machine virtuelle <br> Configuration des diagnostics Azure sur les ressources <br> Totalité des paramètres et propriétés de l’ensemble des ressources. <br> Pour les espaces de travail, cela permet aux utilisateurs disposant d’autorisations illimitées de lire les paramètres de l’espace de travail et d’interroger les données. Consultez des options plus granulaires ci-dessus. |
| `Microsoft.Automation/automationAccounts/*` | Possibilité de créer et configurer les comptes Azure Automation, et notamment d’ajouter et modifier des runbooks |
| `Microsoft.ClassicCompute/virtualMachines/extensions/*` <br> `Microsoft.Compute/virtualMachines/extensions/*` | Ajout, mise à jour et suppression d’extensions de machine virtuelle, notamment l’extension Microsoft Monitoring Agent et l’extension Agent OMS pour Linux |
| `Microsoft.ClassicStorage/storageAccounts/listKeys/action` <br> `Microsoft.Storage/storageAccounts/listKeys/action` | Visualisation de la clé du compte de stockage ; opération requise pour la configuration de Log Analytics pour la lecture des journaux d’activité à partir des comptes de stockage Azure |
| `Microsoft.Insights/alertRules/*` | Ajout, mise à jour et suppression de règles d’alerte |
| `Microsoft.Insights/diagnosticSettings/*` | Ajout, mise à jour et suppression de paramètres de diagnostic sur les ressources Azure |
| `Microsoft.OperationalInsights/*` | Ajout, mise à jour et suppression de configuration pour les espaces de travail Log Analytics. Pour modifier des paramètres avancés d’espace de travail, l’utilisateur a besoin de `Microsoft.OperationalInsights/workspaces/write`. |
| `Microsoft.OperationsManagement/*` | Ajout et suppression de solutions de gestion |
| `Microsoft.Resources/deployments/*` | Création et suppression de déploiements ; opération requise pour l’ajout et la suppression de solutions, d’espaces de travail et de comptes Automation |
| `Microsoft.Resources/subscriptions/resourcegroups/deployments/*` | Création et suppression de déploiements ; opération requise pour l’ajout et la suppression de solutions, d’espaces de travail et de comptes Automation |

L’ajout et la suppression d’utilisateurs au niveau d’un rôle utilisateur requièrent les autorisations `Microsoft.Authorization/*/Delete` et `Microsoft.Authorization/*/Write`.

Utilisez ces rôles pour accorder aux utilisateurs l’accès à différentes étendues :

* Abonnement : accès à tous les espaces de travail de l’abonnement
* Groupe de ressources : accès à tous les espaces de travail du groupe de ressources
* Ressource : accès uniquement à l’espace de travail spécifié

Nous vous recommandons d’effectuer des attributions au niveau de la ressource (espace de travail) afin d’assurer un contrôle d’accès précis. Pour créer des rôles avec les autorisations spécifiques nécessaires, utilisez des [rôles personnalisés](../../role-based-access-control/custom-roles.md).

### <a name="resource-permissions"></a>Autorisations de ressource

Quand les utilisateurs interrogent les journaux à partir d’un espace de travail en utilisant un accès en fonction du contexte de la ressource, ils disposent des autorisations suivantes sur la ressource :

| Autorisation | Description |
| ---------- | ----------- |
| `Microsoft.Insights/logs/<tableName>/read`<br><br>Exemples :<br>`Microsoft.Insights/logs/*/read`<br>`Microsoft.Insights/logs/Heartbeat/read` | Capacité d’afficher toutes les données de journal pour la ressource.  |
| `Microsoft.Insights/diagnosticSettings/write` | Possibilité de configurer les paramètres de diagnostic pour autoriser la configuration des journaux pour cette ressource. |

L’autorisation `/read` est généralement accordée à partir d’un rôle disposant d’autorisations _\*/read ou_ _\*_ tel que les rôles prédéfinis [Lecteur](../../role-based-access-control/built-in-roles.md#reader) et [Contributeur](../../role-based-access-control/built-in-roles.md#contributor). Les rôles personnalisés qui incluent des actions spécifiques ou des rôles intégrés dédiés peuvent ne pas inclure cette autorisation.

Consultez [RBAC au niveau table](#table-level-azure-rbac) ci-après si vous souhaitez créer différents contrôles d’accès pour différentes tables.

## <a name="custom-role-examples"></a>Exemples de rôles personnalisés

1. Pour accorder à un utilisateur l’accès aux données de journal à partir de ses ressources, effectuez les étapes suivantes :

    * Configurez le mode de contrôle d’accès pour **utiliser les autorisations de ressource ou d’espace de travail**.

    * Accordez aux utilisateurs des autorisations d’accès `*/read` ou `Microsoft.Insights/logs/*/read` à leurs ressources. S’ils ont déjà le rôle de [Lecteur Log Analytics](../../role-based-access-control/built-in-roles.md#reader) sur l’espace de travail, cela suffit.

2. Pour accorder à un utilisateur l’accès aux données de journal à partir de ses ressources et configurer ses ressources pour envoyer des journaux à l’espace de travail, effectuez les étapes suivantes :

    * Configurez le mode de contrôle d’accès pour **utiliser les autorisations de ressource ou d’espace de travail**.

    * Accordez aux utilisateurs les autorisations suivantes sur l’espace de travail : `Microsoft.OperationalInsights/workspaces/read` et `Microsoft.OperationalInsights/workspaces/sharedKeys/action`. Avec ces autorisations, les utilisateurs ne peuvent pas exécuter des requêtes au niveau de l’espace de travail. Ils peuvent uniquement énumérer l’espace de travail et l’utiliser comme destination pour les paramètres de diagnostic ou la configuration de l’agent.

    * Accordez aux utilisateurs les autorisations d’accès suivantes à leurs ressources : `Microsoft.Insights/logs/*/read` et `Microsoft.Insights/diagnosticSettings/write`. Si le rôle Lecteur ou [Contributeur Log Analytics](../../role-based-access-control/built-in-roles.md#contributor) leur est déjà attribué ou s’ils disposent déjà des autorisations `*/read` sur cette ressource, cela suffit.

3. Pour accorder à un utilisateur l’accès aux données de journal à partir de ses ressources sans qu’il puisse lire les événements de sécurité et envoyer des données, effectuez les étapes suivantes :

    * Configurez le mode de contrôle d’accès pour **utiliser les autorisations de ressource ou d’espace de travail**.

    * Accordez aux utilisateurs les autorisations d’accès suivantes à leurs ressources : `Microsoft.Insights/logs/*/read`.

    * Ajoutez la non-action suivante pour empêcher les utilisateurs de lire le type SecurityEvent : `Microsoft.Insights/logs/SecurityEvent/read`. La non-action doit avoir le même rôle personnalisé que l’action qui fournit l’autorisation de lecture (`Microsoft.Insights/logs/*/read`). Si l’utilisateur hérite de l’action de lecture d’un autre rôle attribué à cette ressource, à l’abonnement ou au groupe de ressources, il pourra lire tous les types de journaux. C’est également le cas s’ils héritent de l’action `*/read` associée au rôle Lecteur ou Contributeur, par exemple.

4. Pour accorder à un utilisateur l’accès aux données de journal à partir de ses ressources et l’accès en lecture à l’ensemble des infos d’identification Azure AD et des données de journal de la solution Update Management à partir de l’espace de travail, effectuez les étapes suivantes :

    * Configurez le mode de contrôle d’accès pour **utiliser les autorisations de ressource ou d’espace de travail**.

    * Accordez aux utilisateurs les autorisations suivantes sur l’espace de travail : 

        * `Microsoft.OperationalInsights/workspaces/read` : obligatoire pour que l’utilisateur puisse énumérer l’espace de travail et ouvrir le panneau correspondant dans le Portail Azure.
        * `Microsoft.OperationalInsights/workspaces/query/read` : obligatoire pour chaque utilisateur pouvant exécuter des requêtes.
        * `Microsoft.OperationalInsights/workspaces/query/SigninLogs/read` : pour pouvoir lire les journaux de connexion Azure AD.
        * `Microsoft.OperationalInsights/workspaces/query/Update/read` : pour pouvoir lire les journaux de la solution Update Management.
        * `Microsoft.OperationalInsights/workspaces/query/UpdateRunProgress/read` : pour pouvoir lire les journaux de la solution Update Management.
        * `Microsoft.OperationalInsights/workspaces/query/UpdateSummary/read` : pour pouvoir lire les journaux Update Management.
        * `Microsoft.OperationalInsights/workspaces/query/Heartbeat/read` : obligatoire pour pouvoir utiliser la solution Update Management.
        * `Microsoft.OperationalInsights/workspaces/query/ComputerGroup/read` : obligatoire pour pouvoir utiliser la solution Update Management.

    * Accordez aux utilisateurs les autorisations d’accès suivantes à leurs ressources : `*/read`, attribué au rôle Lecteur, ou `Microsoft.Insights/logs/*/read`. 

## <a name="table-level-azure-rbac"></a>Azure RBAC au niveau de la table

**Azure RBAC au niveau de la table** vous permet de définir un contrôle plus précis des données dans un espace de travail Log Analytics en plus des autres autorisations. Avec ce contrôle, vous pouvez définir des types de données spécifiques qui sont accessibles uniquement à un ensemble spécifique d’utilisateurs.

Vous implémentez le contrôle d’accès au niveau table avec des [rôles personnalisés Azure](../../role-based-access-control/custom-roles.md) pour accorder l’accès à des [tables](./data-platform-logs.md) spécifiques dans l’espace de travail. Ces rôles sont appliqués aux espaces de travail dont le [mode de contrôle d’accès](design-logs-deployment.md#access-control-mode) est en fonction du contexte de l’espace de travail ou en fonction du contexte de la ressource, quel que soit le [mode d’accès](design-logs-deployment.md#access-mode) de l’utilisateur.

Créez un [rôle personnalisé](../../role-based-access-control/custom-roles.md) avec les actions suivantes pour définir le contrôle d’accès à une table.

* Pour accorder l’accès à une table, incluez-la dans la section **Actions** de la définition de rôle. Pour retirer l’accès à partir des **actions** autorisées, incluez-le dans la section **NotActions**.
* Utilisez Microsoft.OperationalInsights/Workspaces/Query/* pour spécifier toutes les tables.

Par exemple, pour créer un rôle ayant accès aux tables _Heartbeat_ et _AzureActivity_, créez un rôle personnalisé à l’aide des actions suivantes :

```
"Actions":  [
    "Microsoft.OperationalInsights/workspaces/read",
    "Microsoft.OperationalInsights/workspaces/query/read",
    "Microsoft.OperationalInsights/workspaces/query/Heartbeat/read",
    "Microsoft.OperationalInsights/workspaces/query/AzureActivity/read"
  ],
```

Pour créer un rôle ayant accès uniquement à la table _SecurityBaseline_, créez un rôle personnalisé à l’aide des actions suivantes :

```
"Actions":  [
    "Microsoft.OperationalInsights/workspaces/read",
    "Microsoft.OperationalInsights/workspaces/query/read",
    "Microsoft.OperationalInsights/workspaces/query/SecurityBaseline/read"
],
```
Les exemples ci-dessus définissent une liste des tables autorisées. Cet exemple montre la définition d’une liste bloquée où un utilisateur peut accéder à toutes les tables, à l’exception de la table _SecurityAlert_ :

```
"Actions":  [
    "Microsoft.OperationalInsights/workspaces/read",
    "Microsoft.OperationalInsights/workspaces/query/read",
    "Microsoft.OperationalInsights/workspaces/query/*/read"
],
"notActions":  [
    "Microsoft.OperationalInsights/workspaces/query/SecurityAlert/read"
],
```

### <a name="custom-logs"></a>Journaux d’activité personnalisés

 Les journaux personnalisés sont créés à partir de sources de données telles que des journaux personnalisés et l’API Collecteur de données HTTP. Le moyen le plus simple d’identifier le type de journal consiste à vérifier les tables listées sous [Journaux personnalisés dans le schéma de journal](../log-query/log-analytics-tutorial.md#table-schema).

 Vous ne pouvez pas accorder l’accès à des journaux personnalisés individuels, mais vous pouvez accorder l’accès à tous les journaux personnalisés. Pour créer un rôle ayant accès à tous les journaux personnalisés, créez un rôle personnalisé à l’aide des actions suivantes :

```
"Actions":  [
    "Microsoft.OperationalInsights/workspaces/read",
    "Microsoft.OperationalInsights/workspaces/query/read",
    "Microsoft.OperationalInsights/workspaces/query/Tables.Custom/read"
],
```
Une autre approche pour gérer l’accès aux journaux personnalisés consiste à les affecter à une ressource Azure et à gérer l’accès via le paradigme de contexte de ressource. Pour utiliser cette méthode, vous devez inclure l’ID de ressource en le spécifiant dans l’en-tête [x-ms-AzureResourceId](data-collector-api.md#request-headers) quand des données sont ingérées dans Log Analytics via l’[API du collecteur de données HTTP](data-collector-api.md). L’ID de ressource doit être valide et des règles d’accès doivent lui être appliquées. Une fois les journaux ingérés, ils sont accessibles pour ceux qui disposent d’un accès en lecture sur la ressource, comme expliqué ici.

Parfois, les journaux personnalisés proviennent de sources qui ne sont pas directement associées à une ressource spécifique. Dans ce cas, créez un groupe de ressources seulement pour gérer l’accès à ces journaux. Le groupe de ressources n’entraîne aucun coût, mais vous donne un ID de ressource valide pour contrôler l’accès aux journaux personnalisés. Par exemple, si un pare-feu spécifique envoie des journaux personnalisés, créez un groupe de ressources appelé « MyFireWallLogs » et vérifiez que les demandes de l’API contiennent l’ID de ressource « MyFireWallLogs ». Les enregistrements du journal du pare-feu sont alors accessibles seulement par les utilisateurs qui ont accès à MyFireWallLogs ou par ceux qui disposent d’un accès complet à l’espace de travail.          

### <a name="considerations"></a>Considérations

* Si un utilisateur se voit accorder une autorisation de lecture globale avec les rôles Lecteur ou Contributeur standard qui incluent l’action _\*/read_, cette autorisation se substitue au contrôle d’accès par table et donne à l’utilisateur l’accès à toutes les données de journal.
* Si un utilisateur se voit accorder un accès par table mais aucune autre autorisation, il peut accéder aux données de journal à partir de l’API, mais pas du portail Azure. Pour fournir l’accès à partir du portail Azure, utilisez le Lecteur Log Analytics comme rôle de base.
* Les administrateurs et propriétaires de l’abonnement ont accès à tous les types de données indépendamment des autres paramètres d’autorisation.
* Les propriétaires d’espace de travail sont traités comme tout autre utilisateur pour le contrôle d’accès par table.
* Nous vous conseillons d’attribuer des rôles à des groupes de sécurité plutôt qu’à des utilisateurs individuels afin de réduire le nombre d’attributions. En outre, cette approche facilite l’utilisation des outils de gestion de groupe existants pour configurer et vérifier l’accès.

## <a name="next-steps"></a>Étapes suivantes

* Consultez [Présentation de l’agent Log Analytics](./log-analytics-agent.md) pour collecter des données à partir d’ordinateurs dans votre centre de données ou d’un autre environnement cloud.

* Pour configurer la collecte de données à partir de machines virtuelles Azure, voir [Collecter des données sur les machines virtuelles Azure](../learn/quick-collect-azurevm.md).
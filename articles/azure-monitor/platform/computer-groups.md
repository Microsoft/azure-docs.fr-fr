---
title: Groupes d’ordinateurs dans les requêtes de journal Azure Monitor | Microsoft Docs
description: Les groupes d’ordinateurs d’Azure Monitor permettent de formuler des requêtes de journal portant sur un ensemble spécifique d’ordinateurs.  Cet article décrit les différentes méthodes applicables pour créer des groupes d’ordinateurs et explique comment les utiliser dans une requête de journal.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 02/05/2019
ms.openlocfilehash: 92603165ac399415ec4fb6daeea1641065671a83
ms.sourcegitcommit: ab94795f9b8443eef47abae5bc6848bb9d8d8d01
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/27/2020
ms.locfileid: "96302932"
---
# <a name="computer-groups-in-azure-monitor-log-queries"></a>Groupes d’ordinateurs dans les requêtes de journal Azure Monitor
Les groupes d’ordinateurs d’Azure Monitor permettent de formuler des [requêtes de journal](../log-query/log-query-overview.md) portant sur un ensemble spécifique d’ordinateurs.  Vous peuplez chaque groupe d’ordinateurs soit à l’aide d’une requête que vous définissez, soit en important des groupes à partir de différentes sources.  Quand le groupe est inclus dans une requête de journal, les résultats sont limités aux enregistrements correspondant aux ordinateurs du groupe.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="creating-a-computer-group"></a>Création d’un groupe d’ordinateurs
Pour créer un groupe d’ordinateurs dans Azure Monitor, vous pouvez suivre les méthodes présentées dans le tableau suivant.  Des détails sur chaque méthode sont fournis dans les sections ci-dessous. 

| Méthode | Description |
|:--- |:--- |
| Requête de journal |Créer une requête de journal qui retourne une liste d’ordinateurs. |
| API Recherche de journal |Utiliser l’API Recherche dans les journaux pour créer un groupe d’ordinateurs programmatiquement à partir des résultats d’une requête de journal. |
| Active Directory |Analyser automatiquement l’appartenance de groupe de tous les ordinateurs agents membres d’un domaine Active Directory et créer un groupe pour chaque groupe de sécurité dans Azure Monitor. (Ordinateurs Windows uniquement)|
| Gestionnaire de configuration | Importer des regroupements depuis Microsoft Endpoint Configuration Manager et créer un groupe pour chacun dans Azure Monitor. |
| Windows Server Update Services |Analyser automatiquement les clients ou serveurs WSUS pour détecter les groupes de ciblage et créer un groupe pour chacun dans Azure Monitor. |

### <a name="log-query"></a>Requête de journal
Les groupes d’ordinateurs créés à partir d’une requête de journal contiennent tous les ordinateurs retournés par la requête définie par vos soins.  Cette requête est exécutée chaque fois que le groupe d’ordinateurs est utilisé, de façon à refléter toutes les modifications apportées depuis la création du groupe.  

Vous pouvez utiliser une requête pour un groupe d’ordinateurs, mais elle doit retourner un ensemble distinct d’ordinateurs à l’aide de `distinct Computer`.  Voici un exemple type de requête utilisable pour un groupe d’ordinateurs.

```kusto
Heartbeat | where Computer contains "srv" | distinct Computer
```

Utilisez la procédure suivante pour créer un groupe d’ordinateurs à partir d’une recherche dans les journaux dans le portail Azure.

1. Cliquez sur **Journaux d’activité** dans le menu **Azure Monitor** sur le Portail Azure.
1. Créez et exécutez une requête qui retourne les ordinateurs que vous voulez ajouter au groupe.
1. Cliquez sur **Enregistrer** dans la partie supérieure de l’écran.
1. Remplacez **Enregistrer sous** par **Fonction** et sélectionnez **Enregistrer cette requête comme groupe d’ordinateurs**.
1. Entrez les valeurs de chaque propriété décrite dans le tableau pour le groupe d’ordinateurs et cliquez sur **Enregistrer**.

Le tableau suivant décrit les propriétés qui définissent un groupe d’ordinateurs.

| Propriété | Description |
|:---|:---|
| Nom   | Nom de la requête à afficher sur le portail. |
| Alias de fonction | Alias unique utilisé pour identifier le groupe d’ordinateurs dans une requête. |
| Category       | Catégorie servant à organiser les requêtes sur le portail. |


### <a name="active-directory"></a>Active Directory
Si Azure Monitor est configuré de façon à importer les appartenances de groupe Active Directory, il analyse l’appartenance de tous les ordinateurs joints à un domaine Windows avec l’agent Log Analytics.  Un groupe d’ordinateurs est créé dans Azure Monitor pour chaque groupe de sécurité dans Active Directory, et chaque ordinateur Windows est ajouté aux groupes d’ordinateurs correspondant aux groupes de sécurité auxquels il appartient.  Cet appartenance est mise à jour toutes les 4 heures.  

> [!NOTE]
> Les groupes Active Directory importés contiennent uniquement les ordinateurs Windows.

Pour configurer Azure Monitor de façon à importer des groupes de sécurité Active Directory, accédez à **Paramètres avancés** dans votre espace de travail Log Analytics sur le Portail Azure.  Sélectionnez **Groupes d’ordinateurs**, **Active Directory**, puis **Importer les appartenances à des groupes Active Directory depuis les ordinateurs**.  Aucune configuration supplémentaire n’est requise.

![Groupes d’ordinateurs d’Active Directory](media/computer-groups/configure-activedirectory.png)

Une fois des groupes importés, le menu répertorie le nombre d’ordinateurs détectés avec une appartenance à un groupe et le nombre de groupes importés.  Vous pouvez cliquer sur l’un de ces liens pour retourner les enregistrements **ComputerGroup** avec ces informations.

### <a name="windows-server-update-service"></a>Windows Server Update Service
Si Azure Monitor est configuré de façon à importer les appartenances de groupe WSUS, il analyse l’appartenance de groupe de ciblage de tous les ordinateurs avec l’agent Log Analytics.  Si vous utilisez un ciblage côté client, l’appartenance de groupe de tous les ordinateurs connectés à Azure Monitor et faisant partie d’un groupe de ciblage WSUS est importée dans Azure Monitor. Si vous utilisez un ciblage côté serveur, l’agent Log Analytics doit être installé sur le serveur WSUS pour que les informations d’appartenance de groupe soient importées dans Azure Monitor.  Cet appartenance est mise à jour toutes les 4 heures. 

Pour configurer Azure Monitor de façon à importer des groupes WSUS, accédez à **Paramètres avancés** dans votre espace de travail Log Analytics sur le Portail Azure.  Sélectionnez **Groupes d’ordinateurs**, **WSUS**, puis **Importer les appartenances à un groupe WSUS**.  Aucune configuration supplémentaire n’est requise.

![Groupes d’ordinateurs à partir de WSUS](media/computer-groups/configure-wsus.png)

Une fois des groupes importés, le menu répertorie le nombre d’ordinateurs détectés avec une appartenance à un groupe et le nombre de groupes importés.  Vous pouvez cliquer sur l’un de ces liens pour retourner les enregistrements **ComputerGroup** avec ces informations.

### <a name="configuration-manager"></a>Gestionnaire de configuration
Si Azure Monitor est configuré de façon à importer les adhésions aux regroupements Configuration Manager, il crée un groupe d’ordinateurs pour chaque regroupement.  Les informations d’appartenance au regroupement sont récupérées toutes les 3 heures pour tenir les groupes d’ordinateurs à jour. 

Pour pouvoir importer des regroupements Configuration Manager, vous devez [connecter Configuration Manager à Azure Monitor](collect-sccm.md).  

![Groupes d’ordinateurs à partir de SCCM](media/computer-groups/configure-sccm.png)

Une fois les regroupements importés, le menu répertorie le nombre d’ordinateurs détectés avec une appartenance à un groupe et le nombre de groupes importés.  Vous pouvez cliquer sur l’un de ces liens pour retourner les enregistrements **ComputerGroup** avec ces informations.

## <a name="managing-computer-groups"></a>Gestion de groupes d’ordinateurs
Pour afficher les groupes d’ordinateurs créés à partir d’une requête de journal d’activité ou de l’API Recherche dans les journaux, accédez à **Paramètres avancés** dans votre espace de travail Log Analytics sur le Portail Azure.  Sélectionnez **Groupes d’ordinateurs**, puis **Groupes enregistrés**.  

Cliquez sur le signe **x** dans la colonne **Supprimer** pour supprimer le groupe d’ordinateurs.  Cliquez sur l’icône **Afficher les membres** correspondant à un groupe pour exécuter la recherche de journal du groupe qui retourne les membres de celui-ci.  Pour modifier un groupe d’ordinateurs, vous devez le supprimer et le recréer avec les paramètres modifiés.

![Groupes d’ordinateurs enregistrés](media/computer-groups/configure-saved.png)


## <a name="using-a-computer-group-in-a-log-query"></a>Utiliser un groupe d’ordinateurs dans une requête de journal
Pour utiliser un groupe d’ordinateurs créé à partir d’une requête de journal, traitez son alias comme une fonction, en général avec la syntaxe suivante :

```kusto
Table | where Computer in (ComputerGroup)
```

Par exemple, vous pouvez utiliser les éléments suivants pour retourner les enregistrements UpdateSummary pour les ordinateurs contenus dans un groupe d’ordinateurs appelé mycomputergroup.

```kusto
UpdateSummary | where Computer in (mycomputergroup)
```

Les groupes d’ordinateurs importés et leurs ordinateurs inclus sont stockés dans la table **ComputerGroup**.  Par exemple, la requête suivante retourne une liste d’ordinateurs du groupe d’ordinateurs du domaine d’Active Directory. 

```kusto
ComputerGroup | where GroupSource == "ActiveDirectory" and Group == "Domain Computers" | distinct Computer
```

La requête suivante retourne les enregistrements UpdateSummary pour les seuls ordinateurs du domaine.

```kusto
let ADComputers = ComputerGroup | where GroupSource == "ActiveDirectory" and Group == "Domain Computers" | distinct Computer;
  UpdateSummary | where Computer in (ADComputers)
```

## <a name="computer-group-records"></a>Enregistrements de groupe d’ordinateurs
Un enregistrement est créé dans l’espace de travail Log Analytics pour chaque appartenance à un groupe d’ordinateur créée à partir d’Active Directory ou de WSUS.  Ces enregistrements sont de type **ComputerGroup** et ont les propriétés décrites dans le tableau suivant.  Aucun enregistrement n’est créé pour des groupes d’ordinateurs basés sur des requêtes de journal.

| Propriété | Description |
|:--- |:--- |
| `Type` |*ComputerGroup* |
| `SourceSystem` |*SourceSystem* |
| `Computer` |Nom de l’ordinateur membre. |
| `Group` |Nom du groupe. |
| `GroupFullName` |Chemin d’accès complet au groupe, incluant la source et le nom de la source. |
| `GroupSource` |Source à partir de laquelle ce groupe a été collecté. <br><br>Active Directory<br>WSUS<br>WSUSClientTargeting |
| `GroupSourceName` |Nom de la source à partir de laquelle le groupe a été collecté.  Pour Active Directory, il s’agit du nom de domaine. |
| `ManagementGroupName` |Nom du groupe d'administration pour les agents SCOM.  Pour les autres agents, il s'agit d’AOI-\<workspace ID\> |
| `TimeGenerated` |Date et heure de création ou de mise à jour du groupe d’ordinateurs. |

## <a name="next-steps"></a>Étapes suivantes
* Découvrez les [requêtes dans les journaux](../log-query/log-query-overview.md) pour analyser les données collectées à partir de sources de données et de solutions.  


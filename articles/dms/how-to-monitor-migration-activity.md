---
title: Surveiller l’activité de migration – Azure Database Migration Service
description: Apprenez à utiliser Azure Database Migration Service pour surveiller l’activité de migration.
services: database-migration
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: how-to
ms.date: 02/20/2020
ms.openlocfilehash: dc9123735ef65bec89e99a14d25cba881be306fe
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91297468"
---
# <a name="monitor-migration-activity-using-the-azure-database-migration-service"></a>Surveiller l’activité de migration avec Azure Database Migration Service
Dans cet article, vous allez apprendre à surveiller la progression d’une migration aux niveaux de la base de données et de la table.

## <a name="monitor-at-the-database-level"></a>Surveiller au niveau de la base de données
Pour surveiller l’activité au niveau de la base de données, affichez le panneau au niveau de cette dernière :

![Panneau au niveau de la base de données](media/how-to-monitor-migration-activity/dms-database-level-blade.png)

> [!NOTE]
> Lorsque vous sélectionnez le lien hypertexte de la base de données, une liste des tables et de la progression de leur migration s’affiche.

La table suivante répertorie les champs sur le panneau au niveau de la base de données et décrit les différentes valeurs d’état qui leur sont respectivement associées.

<table id='overview' class='overview'>
  <thead>
    <tr>
      <th class="x-hidden-focus"><strong>Nom du champ</strong></th>
      <th><strong>Sous-état du champ</strong></th>
      <th><strong>Description</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="3" class="ActivityStatus"><strong>État de l’activité</strong></td>
      <td>Exécution en cours</td>
      <td>L’activité de migration est en cours.</td>
    </tr>
    <tr>
      <td>Opération réussie</td>
      <td>L’activité de migration a réussi sans problème.</td>
    </tr>
    <tr>
      <td>Erreur</td>
      <td>La migration a échoué. Sélectionnez le lien « Afficher les détails de l’erreur » sous les détails de la migration pour afficher le message d’erreur complet.</td>
    </tr>
    <tr>
      <td rowspan="4" class="Status"><strong>État</strong></td>
      <td>Initialisation</td>
      <td>DMS configure le pipeline de migration.</td>
    </tr>
    <tr>
      <td>Exécution en cours</td>
      <td>Le pipeline DMS est en cours d’exécution et en train d’effectuer la migration.</td>
    </tr>
    <tr>
      <td>Terminé</td>
      <td>La migration est terminée.</td>
    </tr>
    <tr>
      <td>Échec</td>
      <td>La migration a échoué. Cliquez sur les détails de la migration pour afficher les erreurs de migration.</td>
    </tr>
    <tr>
      <td rowspan="5" class="migration-details"><strong>Détails de la migration</strong></td>
      <td>Initialisation du pipeline de migration</td>
      <td>DMS configure le pipeline de migration.</td>
    </tr>
    <tr>
      <td>Chargement complet des données en cours</td>
      <td>DMS exécute une charge initiale.</td>
    </tr>
    <tr>
      <td>Prêt pour le basculement</td>
      <td>Une fois la charge initiale exécutée, DMS marquera la base de données comme prêt pour le basculement. L’utilisateur doit vérifier si les données ont été mises à jour lors de la synchronisation continue.</td>
    </tr>
    <tr>
      <td>Tous les changements appliqués</td>
      <td>La charge initiale et la synchronisation continue sont terminées. Cet état survient également après le basculement réussi de la base de données.</td>
    </tr>
    <tr>
      <td>Voir les détails de l'erreur</td>
      <td>Cliquez sur le lien pour afficher les détails de l’erreur.</td>
    </tr>
    <tr>
      <td rowspan="1" class="duration"><strong>Durée</strong></td>
      <td>N/A</td>
      <td>Durée totale entre l’initialisation de l’activité de migration et la fin ou l’échec de la migration.</td>
    </tr>
     </tbody>
</table>

## <a name="monitor-at-table-level--quick-summary"></a>Surveiller au niveau de la table : résumé rapide
Pour surveiller l’activité au niveau du tableau, affichez le panneau au niveau de ce dernier. La partie supérieure du panneau montre en détail le nombre de lignes migrées en pleine charge et les mises à jour incrémentielles. 

La partie inférieure du panneau répertorie les tables et affiche un résumé rapide de la progression de la migration.

![Panneau de niveau table : résumé rapide](media/how-to-monitor-migration-activity/dms-table-level-blade-summary.png)

Le tableau suivant décrit les champs affichés dans les détails au niveau de la table.

| Nom du champ        | Description       |
| ------------- | ------------- |
| **Charge complète terminée**      | Nombre de tables ayant terminé le chargement complet des données. |
| **Charge complète en file d'attente**      | Nombre de tables en file d’attente pour la pleine charge.      |
| **Chargement de la charge complète** | Nombre de tables ayant échoué.      |
| **Mises à jour incrémentielles**      | Nombre de mises à jour de capture des changements de données (CDC) dans les lignes appliqués à la cible. |
| **Insertions incrémentielles**      | Nombre d’insertions CDC dans les lignes appliquées à la cible.      |
| **Suppressions incrémentielles** | Nombre de suppressions CDC dans les lignes appliquées à la cible.      |
| **Changements en attente**      | Nombre de CDC dans les lignes qui attendent toujours d’être appliquées à la cible. |
| **Changements appliqués**      | Total des mises à jour CDC, des insertions et des suppressions dans les lignes appliquées à la cible.      |
| **Tables dans un état d’erreur** | Nombre de tables qui sont dans un état d’« erreur » pendant la migration. Des tables peuvent, par exemple, passer à l’état d’erreur lorsque des doublons sont identifiés dans la cible, ou que des données ne sont pas compatibles avec un chargement dans la table cible.      |

## <a name="monitor-at-table-level--detailed-summary"></a>Surveiller au niveau de la table : résumé détaillé
Il existe deux onglets qui indiquent la progression de la migration : Chargement complet et Synchronisation des données incrémentielles.
    
![Onglet Chargement complet](media/how-to-monitor-migration-activity/dms-full-load-tab.png)

![Onglet Synchronisation des données incrémentielles](media/how-to-monitor-migration-activity/dms-incremental-data-sync-tab.png)

Le tableau suivant décrit les champs affichés dans la progression de la migration au niveau de la table.

| Nom du champ        | Description       |
| ------------- | ------------- |
| **État : synchronisation**      | La synchronisation continue est en cours d’exécution. |
| **Insérer**      | Nombre d’insertions CDC dans les lignes appliquées à la cible.      |
| **Mettre à jour** | Nombre de mises à jour CDC dans les lignes appliquées à la cible.      |
| **Supprimer**      | Nombre de suppressions CDC dans les lignes appliquées à la cible. |
| **Total appliqué**      | Total des mises à jour CDC, des insertions et des suppressions dans les lignes appliquées à la cible. |
| **Erreurs de données** | Nombre d’erreurs de données survenues dans cette table. *511 (impossible de créer une ligne de dimension %d supérieure à la taille de ligne maximale autorisée de %d) et 8114 (erreur de conversion de type de données %ls en %ls) sont des exemples d’erreurs.*  Le client doit interroger à partir de la table dms_apply_exceptions dans la cible Azure pour afficher les détails de l’erreur.    |

> [!NOTE]
> Les valeurs CDC de l’insertion, de la mise à jour et de la suppression ainsi que le total appliqué peuvent diminuer lorsque la base de données a basculé ou que la migration a redémarré.

## <a name="next-steps"></a>Étapes suivantes
- Lisez l’aide à la migration du [Guide de migration de bases de données](https://datamigration.microsoft.com/) de Microsoft.
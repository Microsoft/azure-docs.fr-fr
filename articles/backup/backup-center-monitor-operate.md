---
title: Surveiller et utiliser les sauvegardes avec le Centre de sauvegarde
description: Cet article explique comment surveiller et utiliser les sauvegardes à grande échelle à l’aide du Centre de sauvegarde
ms.topic: conceptual
ms.date: 09/01/2020
ms.openlocfilehash: 29e09e502e83ea67af290f206ee0e68b847b2069
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90992645"
---
# <a name="monitor-and-operate-backups-using-backup-center"></a>Surveiller et utiliser les sauvegardes avec le Centre de sauvegarde

En tant qu’administrateur de sauvegarde, vous pouvez utiliser le Centre de sauvegarde comme volet unique pour surveiller vos travaux et l’inventaire des sauvegardes quotidiennement. Vous pouvez également utiliser le Centre de sauvegarde pour effectuer vos opérations courantes, telles que la réponse à des demandes de sauvegarde à la demande, la restauration des sauvegardes, la création de stratégies de sauvegarde, etc.

## <a name="supported-scenarios"></a>Scénarios pris en charge

* Le Centre de sauvegarde est actuellement pris en charge pour la sauvegarde des machines virtuelles Azure et la sauvegarde des serveurs Azure Database pour PostgreSQL.
* Reportez-vous à la [matrice de prise en charge ](backup-center-support-matrix.md) pour obtenir une liste détaillée des scénarios pris en charge et non pris en charge.

## <a name="backup-instances"></a>Instances de sauvegarde

Le Centre de sauvegarde facilite la recherche et la détectabilité des instances de sauvegarde dans l’ensemble de votre espace de sauvegarde.

La sélection de l’onglet **Instances de sauvegarde** du Centre de sauvegarde vous permet d’afficher les détails relatifs à toutes les instances de sauvegarde auxquelles vous avez accès.

 Vous pouvez afficher les informations suivantes pour chacune de vos instances de sauvegarde :

* Abonnement de la source de données
* Groupe de ressources de la source de données
* Dernier point de récupération
* Coffre associé à l’instance de sauvegarde
* État de la protection

 Vous pouvez également filtrer la liste des instances de sauvegarde sur les paramètres suivants :

* Abonnement de la source de données
* Groupe de ressources de la source de données
* Emplacement de la source de données
* Type de source de données
* Coffre
* État de la protection
* Balises de source de données

Cliquez avec le bouton droit sur l’un des éléments de la grille pour effectuer des actions sur l’instance de sauvegarde donnée, comme naviguer jusqu’à la ressource, déclencher des sauvegardes et des restaurations à la demande, ou arrêter la sauvegarde.

![Centre de sauvegarde - Instances](./media/backup-center-monitor-operate/backup-center-instances.png)

## <a name="backup-jobs"></a>Travaux de sauvegarde

Le Centre de sauvegarde vous permet d’afficher des informations détaillées sur toutes les tâches créées dans votre espace de sauvegarde et de prendre les mesures qui s’imposent pour les travaux qui ont échoué.

Dans le Centre de sauvegarde, sélectionnez l’élément de menu **Travaux de sauvegarde** pour consulter tous vos travaux. Chaque travail contient les informations suivantes :

* Instance de sauvegarde associée au travail
* Abonnement de la source de données
* Groupe de ressources de la source de données
* Emplacement de la source de données
* Opération de travail
* Statut de tâche
* Heure de début de la tâche
* Durée du travail

Sélectionnez un élément de la grille pour afficher plus de détails sur le travail donné. Cliquez avec le bouton droit sur un élément pour accéder à la ressource et prendre les mesures qui s’imposent.

![Centre de sauvegarde - Travaux](./media/backup-center-monitor-operate/backup-center-jobs.png)

L’onglet **Travaux de sauvegarde** vous permet de consulter les travaux des sept derniers jours. Pour afficher les travaux plus anciens, utilisez [Rapports de sauvegarde](backup-center-obtain-insights.md).

## <a name="vaults"></a>Coffres

Dans le Centre de sauvegarde, sélectionnez l’élément de menu **Coffres** pour consulter la liste de tous les [coffres Recovery Services](backup-azure-recovery-services-vault-overview.md) et [coffres Sauvegarde](backup-vault-overview.md) auxquels vous avez accès. Vous pouvez filtrer la liste à l'aide des paramètres suivants :

* Abonnement du coffre
* Groupe de ressources du coffre
* Nom du coffre
* Type de source de données associé à la stratégie

Sélectionnez un élément de la liste pour accéder à un coffre donné.

![Centre de sauvegarde - Coffres](./media/backup-center-monitor-operate/backup-center-vaults.png)

## <a name="backup-policies"></a>Stratégies de sauvegarde

Le Centre de sauvegarde vous permet d’afficher et de modifier les informations sur les clés de vos stratégies de sauvegarde.

Sélectionnez l’élément de menu **Stratégies de sauvegarde** pour consulter toutes les stratégies que vous avez créées dans votre espace de sauvegarde. Vous pouvez filtrer la liste par abonnement de coffre, groupe de ressources, emplacement, type de source de données et coffre. Cliquez avec le bouton droit sur un élément de la grille pour afficher les éléments associés à cette stratégie, modifier la stratégie, voire la supprimer si nécessaire.

![Centre de sauvegarde - Stratégies](./media/backup-center-monitor-operate/backup-center-policies.png)

## <a name="next-steps"></a>Étapes suivantes

* [Gérer votre espace de sauvegarde](backup-center-govern-environment.md)
* [Effectuer des actions à l’aide du Centre de sauvegarde](backup-center-actions.md)
* [Obtenir des insights sur vos sauvegardes](backup-center-obtain-insights.md)

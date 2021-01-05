---
title: Sauvegarde et restauration pour les groupes de serveurs Azure Database pour PostgreSQL Hyperscale
description: Sauvegarde et restauration pour les groupes de serveurs Azure Database pour PostgreSQL Hyperscale
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: d27537f017707e937303dd0c08a589db28aac6ef
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92071436"
---
# <a name="backup-and-restore-for-azure-arc-enabled-postgresql-hyperscale-server-groups"></a>Sauvegarde et restauration pour les groupes de serveurs Azure Arc enabled PostgreSQL Hyperscale

Vous pouvez effectuer une sauvegarde/restauration complète de votre groupe de serveurs Azure Arc enabled PostgreSQL Hyperscale. Dans ce cas, l’ensemble des bases de données sur tous les nœuds de votre groupe de serveurs Azure Arc enabled PostgreSQL Hyperscale est sauvegardé et/ou restauré.
Pour effectuer une sauvegarde et la restaurer, vous devez vérifier qu’une classe de stockage de sauvegarde est configurée pour votre groupe de serveurs. Pour le moment, vous devez indiquer une classe de stockage de sauvegarde au moment où vous créez le groupe de serveurs. Il n’est pas encore possible de configurer votre groupe de serveurs pour qu’il utilise une classe de stockage de sauvegarde après qu’il a été créé.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

> [!CAUTION]
> La préversion ne prend pas en charge la sauvegarde/restauration pour la version 11 du moteur Postgres. Elle prend uniquement en charge la sauvegarde/restauration pour la version 12 de Postgres.

## <a name="verify-configuration"></a>Vérifier la configuration

Tout d’abord, vérifiez si votre groupe de serveurs existant a été configuré pour utiliser la classe de stockage de sauvegarde.

Exécutez la commande suivante après avoir défini le nom de votre groupe de serveurs :
```console
 azdata arc postgres server show -n postgres01
```
Examinez la section « storage » (stockage) de la sortie :
```console
...
"storage": {
      "backups": {
        "className": "local-storage"
      },
      "data": {
        "className": "local-storage",
        "size": "5Gi"
      },
      "logs": {
        "className": "local-storage",
        "size": "5Gi"
      }
    }
...
```
Si vous voyez le nom d’une classe de stockage indiquée dans la section « backups » (sauvegardes) de la sortie de cette commande, cela signifie que votre groupe de serveurs a été configuré pour utiliser une classe de stockage de sauvegarde et qu’il est prêt pour la réalisation de sauvegardes et de restaurations. Si vous ne voyez pas de section « backups », vous devez supprimer et recréer votre groupe de serveurs pour configurer la classe de stockage de sauvegarde. Pour le moment, il n’est pas encore possible de configurer une classe de stockage de sauvegarde une fois que le groupe de serveurs a été créé.

>[!IMPORTANT]
>Si votre groupe de serveurs est déjà configuré pour utiliser une classe de stockage de sauvegarde, ignorez l’étape suivante et passez directement à l’étape « Effectuer une sauvegarde complète manuelle ».

## <a name="create-a-server-group"></a>Créer un groupe de serveurs 

Ensuite, créez un groupe de serveurs configuré pour la sauvegarde/restauration.

Pour pouvoir effectuer des sauvegardes et les restaurer, vous devez créer un serveur configuré avec une classe de stockage.

Pour obtenir la liste des classes de stockage disponibles sur votre cluster Kubernetes, exécutez la commande suivante :

```console
kubectl get sc
```

<!--The general format of create server group command is documented [here](create-postgresql-instances.md)-->

```console
azdata arc postgres server create -n <name> --workers 2 --storage-class-backups <storage class name> [--storage-class-data <storage class name>] [--storage-class-logs <storage class name>]
```

Par exemple, si vous avez créé un environnement simple basé sur kubeadm :
```console
azdata arc postgres server create -n postgres01 --workers 2 --storage-class-backups local-storage
```

## <a name="take-manual-full-backup"></a>Effectuer une sauvegarde complète manuelle


Ensuite, effectuez une sauvegarde complète manuelle.

> [!CAUTION]
> **Pour les utilisateurs d’Azure Kubernetes Service (AKS) uniquement :** nous avons connaissance d’un problème lié à l’exécution de sauvegardes d’un groupe de serveurs hébergé sur Azure Kubernetes Service (AKS). Nous travaillons déjà à sa résolution. Tant que la mise à jour n’est pas déployée dans une version/mise à jour future, vous devez supprimer les pods de vos groupes de serveurs avant d’effectuer une sauvegarde. Pour chacun des pods de votre groupe de serveurs (vous répertoriez les pods en exécutant **kubectl get pods -n \<namespace name>** ), supprimez-les en exécutant **kubectl delete pod \<server group pod name> -n \<namespace name>** . Ne supprimez pas les pods qui ne font pas partie de votre groupe de serveurs. La suppression de pods ne risque pas de compromettre vos données. Attendez que tous les pods soient à nouveau en ligne et à l’état STATUS=RUNNING avant d’effectuer une sauvegarde. L’état du pod est indiqué dans la sortie de la commande « kubectl get pods » ci-dessus.


Pour effectuer une sauvegarde complète de l’ensemble des dossiers de données et de journaux de votre groupe de serveurs, exécutez la commande suivante :

```console
azdata arc postgres backup create [--name <backup name>] --server-name <server group name> [--no-wait] 
```
Où :
- __name__ indique le nom d’une sauvegarde.
- __server-name__ indique un groupe de serveurs
- __no-wait__ indique que la ligne de commande n’attendra pas que la sauvegarde se termine pour vous permettre de continuer à utiliser cette fenêtre de ligne de commande

Cette commande coordonne une sauvegarde complète distribuée sur tous les nœuds qui constituent votre groupe de serveurs Azure Arc enabled PostgreSQL Hyperscale. En d’autres termes, elle va sauvegarder toutes les données de vos nœuds Coordinateur et Worker.

Exemple :
```console
azdata arc postgres backup create --name MyBackup_Aug31_0730amPST --server-name postgres01
```

Une fois la sauvegarde terminée, l’ID, le nom et l’état de la sauvegarde sont retournés. Exemple :
```console
{
  "ID": "d134f51aa87f4044b5fb07cf95cf797f",
  "name": "MyBackup_Aug31_0730amPS",
  "state": "Done"
}
```

> [!NOTE]
> Les actions suivantes ne sont pas encore possibles :
> - Planification de sauvegardes automatiques
> - Montrer la progression d’une sauvegarde pendant sa réalisation

## <a name="list-backups"></a>Lister les sauvegardes

Lister les sauvegardes qui sont disponibles pour la restauration.

Pour lister les sauvegardes disponibles pour la restauration, exécutez la commande suivante :

```console
azdata arc postgres backup list --server-name <servergroup name>
```

Exemple :
```console
azdata arc postgres backup list --server-name postgres01
```

Elle retourne une sortie comme celle-ci :
```console
ID                                Name                      State    Timestamp
--------------------------------  ------------------------  -------  ------------------------------
d134f51aa87f4044b5fb07cf95cf797f  MyBackup_Aug31_0730amPST  Done     2020-08-31 14:30:00:00+00:00
```

Timestamp indique l’heure UTC dans le temps à laquelle la sauvegarde a été effectuée.

## <a name="restore-a-backup"></a>Restaurer une sauvegarde

Pour restaurer la sauvegarde d’un groupe de serveurs entier, exécutez la commande suivante :

```console
azdata arc postgres backup restore --server-name <server group name> --backup-id <backup id>
```

Où :
- __backup-id__ est l’ID de la sauvegarde indiqué dans la commande list backup (reportez-vous à l’étape 3).
Celle-ci va coordonner une restauration complète distribuée sur tous les nœuds qui constituent votre groupe de serveurs Azure Arc enabled PostgreSQL Hyperscale. En d’autres termes, elle va restaurer toutes les données de vos nœuds Coordinateur et Worker.

Exemple :
```console
azdata arc postgres backup restore --server-name postgres01 --backup-id d134f51aa87f4044b5fb07cf95cf797f
```

Une fois l’opération de restauration terminée, une sortie semblable à celle-ci est retournée à la ligne de commande :
```console
{
  "ID": "d134f51aa87f4044b5fb07cf95cf797f",
  "state": "Done"
}
```
> [!NOTE]
> Les actions suivantes ne sont pas encore possibles :
> - Restaurer une sauvegarde en indiquant son nom
> - Restaurer un groupe de serveurs sous un nom différent ou sur un autre groupe de serveurs
> - Montrer la progression d’une opération de restauration

## <a name="delete-backups"></a>Supprimer les sauvegardes
La durée de conservation des sauvegardes ne peut pas être définie dans la préversion. Vous pouvez cependant supprimer manuellement les sauvegardes dont vous n’avez pas besoin.
La commande générale pour supprimer des sauvegardes est la suivante :
```console
azdata arc postgres backup delete  [--server-name, -sn] {[--name, -n], -id}
```
où :
- `--server-name` correspond au nom du groupe de serveurs à partir duquel l’utilisateur souhaite supprimer une sauvegarde
- `--name` correspond au nom de la sauvegarde à supprimer
- `-id` correspond à l’ID de la sauvegarde à supprimer

> [!NOTE]
> `--name` and `-id` s'excluent mutuellement.

Vous pouvez récupérer le nom et l’ID de vos sauvegardes en exécutant la commande list backup comme expliqué dans le paragraphe précédent.

Imaginez, par exemple, que les sauvegardes suivantes sont répertoriées :
```console
azdata arc postgres backup list -sn postgres01
ID                                Name                    State
--------------------------------  ----------------------  -------
5b0481dfc1c94b4cac79dd56a1bb21f4  MyBackup091720200110am  Done
0cf39f1e92344e6db4cfa285d36c7b14  MyBackup091720200111am  Done
```
Si vous souhaitez supprimer la première sauvegarde, vous devez exécuter la commande suivante :
```console
azdata arc postgres backup delete -sn postgres01 -n MyBackup091720200110am
{
  "ID": "5b0481dfc1c94b4cac79dd56a1bb21f4",
  "name": "MyBackup091720200110am",
  "state": "Done"
}
```
Si vous deviez répertorier les sauvegardes à ce stade, vous obtiendriez la sortie suivante :
```console
azdata arc postgres backup list -sn postgres01
ID                                Name                    State
--------------------------------  ----------------------  -------
0cf39f1e92344e6db4cfa285d36c7b14  MyBackup091720200111am  Done
```

Pour plus d’informations sur la commande delete, exécutez la commande suivante :
```console
azdata arc postgres backup delete --help
```

## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur le [scale-out (ajout de nœuds worker)](scale-out-postgresql-hyperscale-server-group.md) de votre groupe de serveurs
- En savoir plus sur le [scale-up ou le scale-down (augmentation/diminution de la mémoire/des vCores)](scale-up-down-postgresql-hyperscale-server-group-using-cli.md) de votre groupe de serveurs

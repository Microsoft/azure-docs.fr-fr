---
title: 'Tutoriel : Gérer une base de données SAP HANA sauvegardée à l’aide de l’interface CLI'
description: Dans ce tutoriel, vous allez découvrir comment gérer les bases de données SAP HANA sauvegardées qui s’exécutent sur une machine virtuelle Azure à l’aide d’Azure CLI.
ms.topic: tutorial
ms.date: 12/4/2019
ms.custom: devx-track-azurecli
ms.openlocfilehash: 22ff95fe5261a839927aa6ad8123ba370710f178
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91323088"
---
# <a name="tutorial-manage-sap-hana-databases-in-an-azure-vm-using-azure-cli"></a>Tutoriel : Gérer des bases de données SAP HANA sur une machine virtuelle Azure à l’aide d’Azure CLI

Azure CLI permet de créer et de gérer des ressources Azure à partir de la ligne de commande ou par le biais de scripts. Ce document explique comment gérer une base de données SAP HANA sauvegardée sur une machine virtuelle Azure à l’aide d’Azure CLI. Vous pouvez également [effectuer ces étapes à l’aide du Portail Azure](./sap-hana-db-manage.md).

Utilisez [Azure Cloud Shell](tutorial-sap-hana-backup-cli.md) pour exécuter les commandes CLI.

À la fin de ce tutoriel, vous serez en mesure d’effectuer les opérations suivantes :

> [!div class="checklist"]
>
> * Superviser des travaux de sauvegarde et de restauration
> * Protéger les nouvelles bases de données ajoutées à une instance de SAP HANA
> * Modifier la stratégie
> * Arrêter la protection
> * Reprendre la protection

Si vous avez appliqué la procédure de l’article [Sauvegarder une base de données SAP HANA dans Azure à l’aide de l’interface CLI](tutorial-sap-hana-backup-cli.md) pour sauvegarder votre base de données SAP HANA, vous utiliserez les ressources suivantes :

* Un groupe de ressources nommé *saphanaResourceGroup*
* Un coffre nommé *saphanaVault*
* Un conteneur protégé nommé *VMAppContainer;Compute;saphanaResourceGroup;saphanaVM*
* Un élément/base de données sauvegardé nommé *saphanadatabase;hxe;hxe*
* Des ressources dans la région *westus2*

Azure CLI facilite la gestion d’une base de données SAP HANA s’exécutant sur une machine virtuelle Azure sauvegardée à l’aide de Sauvegarde Azure. Ce tutoriel décrit en détail chacune des opérations de gestion.

## <a name="monitor-backup-and-restore-jobs"></a>Superviser des travaux de sauvegarde et de restauration

Pour superviser les travaux terminés ou en cours d’exécution (sauvegarde ou restauration), utilisez l’applet de commande [az backup job list](/cli/azure/backup/job#az-backup-job-list). L’interface CLI vous permet également d’[interrompre un travail en cours d’exécution](/cli/azure/backup/job#az-backup-job-stop) ou d’[attendre qu’un travail se termine](/cli/azure/backup/job#az-backup-job-wait).

```azurecli-interactive
az backup job list --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --output table
```

Vous obtenez un résultat semblable à ce qui suit :

```output
Name                                  Operation              Status      Item Name       Start Time UTC
------------------------------------  ---------------        ---------   ----------      -------------------  
e0f15dae-7cac-4475-a833-f52c50e5b6c3  ConfigureBackup        Completed   hxe             2019-12-03T03:09:210831+00:00  
ccdb4dce-8b15-47c5-8c46-b0985352238f  Backup (Full)          Completed   hxe [hxehost]   2019-12-01T10:30:58.867489+00:00
4980af91-1090-49a6-ab96-13bc905a5282  Backup (Differential)  Completed   hxe [hxehost]   2019-12-01T10:36:00.563909+00:00
F7c68818-039f-4a0f-8d73-e0747e68a813  Restore (Log)          Completed   hxe [hxehost]   2019-12-03T05:44:51.081607+00:00
```

## <a name="change-policy"></a>Modifier la stratégie

Pour modifier la stratégie sous-jacente à la configuration de sauvegarde SAP HANA, utilisez l’applet de commande [az backup policy set](/cli/azure/backup/policy#az-backup-policy-set). Le paramètre name de cette applet de commande fait référence à l’élément de sauvegarde dont vous souhaitez modifier la stratégie. Pour ce tutoriel, nous allons remplacer la stratégie de notre base de données SAP HANA *saphanadatabase;hxe;hxe* par une nouvelle stratégie *newsaphanaPolicy*. Vous pouvez créer de nouvelles stratégies à l’aide de l’applet de commande [az backup policy create](/cli/azure/backup/policy#az-backup-policy-create).

```azurecli-interactive
az backup item set policy --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --container-name VMAppContainer;Compute;saphanaResourceGroup;saphanaVM \
    --policy-name newsaphanaPolicy \
    --name saphanadatabase;hxe;hxe \
```

La sortie doit ressembler à ceci :

```output
Name                                  Resource Group
------------------------------------- --------------
cb110094-9b15-4c55-ad45-6899200eb8dd  SAPHANA
```

## <a name="protect-new-databases-added-to-an-sap-hana-instance"></a>Protéger les nouvelles bases de données ajoutées à une instance de SAP HANA

[L’inscription d’une instance de SAP HANA auprès d’un coffre Recovery Services](tutorial-sap-hana-backup-cli.md#register-and-protect-the-sap-hana-instance) entraîne la découverte automatique de toutes les bases de données sur cette instance.

Toutefois, dans les cas où de nouvelles bases de données sont ajoutées ultérieurement à l’instance de SAP HANA, utilisez l’applet de commande [az backup protectable-item initialize](/cli/azure/backup/protectable-item#az-backup-protectable-item-initialize). Cette applet de commande découvre les nouvelles bases de données ajoutées.

```azurecli-interactive
az backup protectable-item initialize --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --container-name VMAppContainer;Compute;saphanaResourceGroup;saphanaVM \
    --workload-type SAPHANA
```

Utilisez ensuite l’applet de commande [az backup protectable-item list](/cli/azure/backup/protectable-item#az-backup-protectable-item-list) pour lister toutes les bases de données qui ont été découvertes sur votre instance de SAP HANA. Cette liste exclut toutefois les bases de données sur lesquelles la sauvegarde a déjà été configurée. Une fois la base de données à sauvegarder découverte, consultez [Activer la sauvegarde sur une base de données SAP HANA](tutorial-sap-hana-backup-cli.md#enable-backup-on-sap-hana-database).

```azurecli-interactive
az backup protectable-item list --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --workload-type SAPHANA \
    --output table
```

La base de données que vous souhaitez sauvegarder figurera dans cette liste, qui ressemblera à ceci :

```output
Name                            Protectable Item Type    ParentName    ServerName    IsProtected
---------------------------     ----------------------   ------------  -----------   ------------
saphanasystem;hxe               SAPHanaSystem            HXE           hxehost       NotProtected  
saphanadatabase;hxe;systemdb    SAPHanaDatabase          HXE           hxehost       NotProtected
saphanadatabase;hxe;newhxe      SAPHanaDatabase          HXE           hxehost       NotProtected
```

## <a name="stop-protection-for-an-sap-hana-database"></a>Arrêter la protection d’une base de données SAP HANA

Vous pouvez arrêter la protection d’une base de données SAP HANA de deux manières :

* Arrêter tous les travaux de sauvegarde à venir et supprimer tous les points de récupération.
* Arrêter tous les travaux de sauvegarde à venir, mais conserver les points de récupération.

Si vous choisissez de conserver les points de récupération, gardez à l’esprit les considérations suivantes :

* Tous les points de récupération resteront intacts et toutes les opérations de nettoyage s’arrêteront à l’arrêt de la protection avec données conservées.
* Vous serez facturé pour l’instance protégée et le stockage utilisé.
* Si vous supprimez une source de données sans arrêter les sauvegardes, les nouvelles sauvegardes échouent.

Examinons chacun des moyens d’arrêter la protection plus en détail.

### <a name="stop-protection-with-retain-data"></a>Arrêter la protection avec données conservées

Pour arrêter la protection en conservant les données, utilisez l’applet de commande [az backup protection disable](/cli/azure/backup/protection#az-backup-protection-disable).

```azurecli-interactive
az backup protection disable --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --container-name VMAppContainer;Compute;saphanaResourceGroup;saphanaVM \
    --item-name saphanadatabase;hxe;hxe \
    --workload-type SAPHANA \
    --output table
```

La sortie doit ressembler à ceci :

```output
Name                                  ResourceGroup
------------------------------------  ---------------  
g0f15dae-7cac-4475-d833-f52c50e5b6c3  saphanaResourceGroup
```

Pour vérifier l’état de cette opération, utilisez l’applet de commande [az backup job show](/cli/azure/backup/job#az-backup-job-show).

### <a name="stop-protection-without-retain-data"></a>Arrêter la protection sans conserver les données

Pour arrêter la protection sans conserver les données, utilisez l’applet de commande [az backup protection disable](/cli/azure/backup/protection#az-backup-protection-disable).

```azurecli-interactive
az backup protection disable --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --container-name VMAppContainer;Compute;saphanaResourceGroup;saphanaVM \
    --item-name saphanadatabase;hxe;hxe \
    --workload-type SAPHANA \
    --delete-backup-data true \
    --output table
```

La sortie doit ressembler à ceci :

```output
Name                                  ResourceGroup
------------------------------------  ---------------  
g0f15dae-7cac-4475-d833-f52c50e5b6c3  saphanaResourceGroup
```

Pour vérifier l’état de cette opération, utilisez l’applet de commande [az backup job show](/cli/azure/backup/job#az-backup-job-show).

## <a name="resume-protection"></a>Reprendre la protection

Quand vous arrêtez la protection de la base de données SAP HANA en conservant les données, vous pouvez reprendre la protection ultérieurement. Si vous ne conservez pas les données sauvegardées, vous ne pourrez pas reprendre la protection.

Pour reprendre la protection, utilisez l’applet de commande [az backup protection resume](/cli/azure/backup/protection#az-backup-protection-resume).

```azurecli-interactive
az backup protection resume --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --container-name VMAppContainer;Compute;saphanaResourceGroup;saphanaVM \
    --policy-name saphanaPolicy \
    --output table
```

La sortie doit ressembler à ceci :

```output
Name                                  ResourceGroup
------------------------------------  ---------------  
b2a7f108-1020-4529-870f-6c4c43e2bb9e  saphanaResourceGroup
```

Pour vérifier l’état de cette opération, utilisez l’applet de commande [az backup job show](/cli/azure/backup/job#az-backup-job-show).

## <a name="next-steps"></a>Étapes suivantes

* Pour découvrir comment sauvegarder une base de données SAP HANA exécutée sur une machine virtuelle Azure à l’aide du portail Azure, consultez [Sauvegarder des bases de données SAP HANA sur des machines virtuelles Azure](./backup-azure-sap-hana-database.md).

* Pour découvrir comment gérer une base de données SAP HANA sauvegardée exécutée sur une machine virtuelle Azure à l’aide du portail Azure, consultez [Gérer des bases de données SAP HANA sauvegardées sur des machines virtuelles Azure](./sap-hana-db-manage.md).

---
title: Sauvegarder une machine virtuelle SQL Server à partir du volet Machine virtuelle
description: Cet article explique comment sauvegarder des bases de données SQL Server sur des machines virtuelles Azure à partir du volet Machine virtuelle.
ms.topic: conceptual
ms.date: 08/13/2020
ms.openlocfilehash: 4f4ea202ee96e93a621c8dd0025c9ebc8b8d445d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88891655"
---
# <a name="back-up-a-sql-server-from-the-vm-pane"></a>Sauvegarder une machine virtuelle SQL Server à partir du volet Machine virtuelle

Cet article explique comment sauvegarder des bases de données SQL Server s’exécutant sur des machines virtuelles Azure avec le service [Sauvegarde Azure](backup-overview.md). Vous pouvez sauvegarder des machines virtuelles SQL Server à l’aide de deux méthodes :

- Machine virtuelle SQL Server Azure unique : les instructions de cet article décrivent comment sauvegarder une machine virtuelle SQL Server directement à partir de l’affichage de la machine virtuelle.
- Plusieurs machines virtuelles SQL Server Azure : vous pouvez créer un coffre Recovery Services et configurer la sauvegarde pour plusieurs machines virtuelles. Pour ce scénario, suivez les instructions fournies dans [cet article](backup-sql-server-database-azure-vms.md).

## <a name="before-you-start"></a>Avant de commencer

1. Vérifiez votre environnement avec la [matrice de prise en charge](sql-support-matrix.md).
2. Obtenez une [vue d’ensemble](backup-azure-sql-database.md) du service Sauvegarde Azure pour machine virtuelle SQL Server.
3. Vérifiez que la machine virtuelle dispose d’une [connectivité réseau](backup-sql-server-database-azure-vms.md#establish-network-connectivity).

## <a name="configure-backup-on-the-sql-server"></a>Configurer la sauvegarde sur SQL Server

Vous pouvez activer la sauvegarde sur votre machine virtuelle SQL Server à partir du volet **Sauvegarde** de la machine virtuelle. Cette méthode effectue deux opérations :

- Elle inscrit la machine virtuelle SQL auprès du service Sauvegarde Azure pour lui permettre d’y accéder.
- Elle protège automatiquement toutes les instances de SQL Server s’exécutant dans la machine virtuelle. Cela signifie que la stratégie de sauvegarde est appliquée à toutes les bases de données existantes, ainsi qu’aux bases de données qui seront ajoutées ultérieurement à ces instances.

1. Sélectionnez la bannière en haut de la page pour ouvrir la vue Sauvegarde SQL Server.

    ![Vue Sauvegarde SQL Server](./media/backup-sql-server-vm-from-vm-pane/sql-server-backup-view.png)

    >[!NOTE]
    >Vous ne voyez pas la bannière ? La bannière s’affiche uniquement pour les machines virtuelles SQL Server créées à l’aide d’images de la Place de marché Azure. Elle s’affiche également pour les machines virtuelles protégées par la sauvegarde des machines virtuelles Azure. Pour les autres images, vous pouvez configurer la sauvegarde en procédant de la manière expliquée [ici](backup-sql-server-database-azure-vms.md).

2. Entrez le nom du coffre Recovery Services. Un coffre est une entité logique dans laquelle stocker et gérer toutes vos sauvegardes. Si vous créez un coffre :

    - Il est créé dans le même abonnement et la même région que la machine virtuelle SQL Server que vous protégez.
    - Il est créé avec le paramètre de stockage géoredondant (GRS) pour toutes les sauvegardes. Si vous souhaitez modifier le type de redondance, vous devez le faire avant de protéger la machine virtuelle. Pour plus d’informations, consultez [cet article](backup-create-rs-vault.md#set-storage-redundancy).

3. Choisissez une **stratégie de sauvegarde**. Vous pouvez choisir parmi la stratégie par défaut ou toute autre stratégie que vous avez créée dans le coffre. Si vous souhaitez créer une stratégie, consultez [cet article](backup-sql-server-database-azure-vms.md#create-a-backup-policy) pour obtenir un guide pas à pas.

    ![Choisir une stratégie de sauvegarde](./media/backup-sql-server-vm-from-vm-pane/backup-policy.png)

4. Sélectionnez **Activer la Sauvegarde Azure**. Cette opération peut prendre quelques minutes.

    ![Sélectionner Activer la sauvegarde](./media/backup-sql-server-vm-from-vm-pane/enable-backup.png)

5. Une fois l’opération terminée, vous le **nom du coffre** apparaît dans la bannière.

    ![Le nom du coffre apparaît dans la bannière](./media/backup-sql-server-vm-from-vm-pane/vault-name.png)

6. Sélectionnez la bannière pour accéder à la vue du coffre, qui affiche toutes les machines virtuelles inscrites et leur état de protection.

    ![Vue de coffre avec des machines virtuelles inscrites](./media/backup-sql-server-vm-from-vm-pane/vault-view.png)

7. Pour les images ne provenant pas de la Place de marché, l’inscription peut également réussir, mais la **configuration de la sauvegarde** ne peut ne pas être déclenchée tant que l’extension Sauvegarde Azure n’a pas d’autorisation sur la machine virtuelle SQL Server. Dans ce cas, la colonne **Préparation de la sauvegarde** indique **Non prête**. Vous devez [affecter les autorisations appropriées](backup-azure-sql-database.md#set-vm-permissions) manuellement pour les images ne provenant pas de la Place de marché afin que la sauvegarde puisse être déclenchée.

    ![La préparation de la sauvegarde n’est pas faite](./media/backup-sql-server-vm-from-vm-pane/backup-readiness-not-ready.png)

8. Pour effectuer d’autres opérations ou une surveillance à effectuer sur la machine virtuelle SQL Server sauvegardée, accédez au coffre Recovery Services correspondant. Accédez à **Éléments de sauvegarde** pour voir toutes les bases de données sauvegardées dans ce coffre, et déclencher des opérations telles que la sauvegarde et la restauration à la demande. De même, accédez à **Travaux de sauvegarde** pour [surveiller](manage-monitor-sql-database-backup.md) des tâches correspondant à des opérations telles que la configuration de la protection, de la sauvegarde et de la restauration.

    ![Afficher les bases de données sauvegardées dans les éléments de sauvegarde](./media/backup-sql-server-vm-from-vm-pane/backup-items.png)

>[!NOTE]
>La sauvegarde n’est pas configurée automatiquement sur toute nouvelle instance SQL Server ajoutée ultérieurement à la machine virtuelle protégée. Pour configurer la sauvegarde sur les nouvelles instances ajoutées, vous devez accéder au coffre auprès duquel la machine virtuelle est inscrite, et suivre les étapes décrites [ici](backup-sql-server-database-azure-vms.md).

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment :

- [Restaurer des bases de données SQL Server sauvegardées](restore-sql-database-azure-vm.md)
- [Gérer les bases de données SQL Server sauvegardées](manage-monitor-sql-database-backup.md)

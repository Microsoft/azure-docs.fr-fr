---
title: Groupe d’approbations de serveurs
titleSuffix: Azure SQL Managed Instance
description: En savoir plus sur le groupe d’approbations de serveurs et sur la gestion de l’approbation entre instances Azure SQL Managed Instance.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: sasapopo
ms.author: sasapopo
ms.reviewer: sstein, bonova
ms.date: 10/08/2020
ms.openlocfilehash: f9d5528746a85668677ab122d98e954bd39cd163
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92790727"
---
# <a name="use-server-trust-groups-to-set-up-and-manage-trust-between-sql-managed-instances"></a>Utiliser des groupes d’approbations de serveurs pour configurer et gérer l’approbation entre des instances managées SQL
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Le groupe d’approbations de serveurs est un concept utilisé pour la gestion de l’approbation entre des instances Azure SQL Managed Instance. En créant un groupe, une approbation basée sur un certificat est établie entre ses membres. Cette approbation peut être utilisée pour différents scénarios inter-instances. La suppression de serveurs du groupe ou la suppression du groupe supprime l’approbation entre les serveurs. Pour créer ou supprimer un groupe d’approbations de serveurs, l’utilisateur doit disposer d’autorisations en écriture sur l’instance managée.
[Groupe d’approbations de serveurs](/azure/templates/microsoft.sql/allversions) est un objet Azure Resource Manager qui a été étiqueté comme **Groupe d’approbations SQL** dans le Portail Azure.

> [!NOTE]
> Le groupe d’approbations de serveurs est introduit dans la préversion publique des transactions distribuées entre instances Azure SQL Managed Instance et présente actuellement certaines limitations qui seront décrites plus loin dans cet article.

## <a name="server-trust-group-setup"></a>Configuration du groupe d’approbations de serveurs

La section suivante décrit la configuration du groupe d’approbations de serveurs.

1. Accédez au [portail Azure](https://portal.azure.com/).

2. Accédez à l’instance Azure SQL Managed Instance que vous prévoyez d’ajouter à un groupe d’approbations de serveurs nouvellement créé.

3. Dans la page de paramètres **Sécurité** , sélectionnez l’onglet **Groupes d’approbations SQL** .

   :::image type="content" source="./media/server-trust-group-overview/security-sql-trust-groups.png" alt-text="Groupes d’approbations de serveurs":::

4. Dans la page Configuration du groupe d’approbations de serveurs, sélectionnez l’icône **Nouveau groupe** .

   :::image type="content" source="./media/server-trust-group-overview/server-trust-group-create-new-group.png" alt-text="Groupes d’approbations de serveurs":::

5. Dans le panneau **Groupe d’approbations SQL** , définissez le **Nom du groupe** . Il doit être unique dans toutes les régions où résident les membres du groupe. L’ **Étendue d’approbation** définit le type de scénario inter-instances qui est activé avec le groupe d’approbations de serveurs. Dans la préversion, la seule étendue d’approbation applicable est **Transactions distribuées** , elle est donc présélectionnée et ne peut pas être modifiée. Tous les **Membres du groupe** doivent appartenir au même **abonnement** , mais ils peuvent se trouver sous des groupes de ressources différents. Sélectionnez le **Groupe de ressources** et **SQL Server/instance** pour choisir l’instance Azure SQL Managed Instance qui sera membre du groupe.

   :::image type="content" source="./media/server-trust-group-overview/server-trust-group-create-blade.png" alt-text="Groupes d’approbations de serveurs":::

6. Une fois tous les champs obligatoires remplis, cliquez sur **Enregistrer** .

## <a name="server-trust-group-maintenance-and-deletion"></a>Maintenance et suppression du groupe d’approbations de serveurs

Impossible de modifier le groupe d’approbations de serveurs. Pour supprimer une instance managée d’un groupe, vous devez supprimer le groupe et en créer un nouveau.

La section suivante décrit le processus de suppression d’un groupe d’approbations de serveurs. 
1. Accédez au portail Azure.
2. Accédez à une instance managée qui appartient au groupe d’approbations.
3. Dans les paramètres de **Sécurité** , sélectionnez l’onglet **Groupes d’approbations SQL** .
4. Sélectionnez le groupe d’approbations que vous souhaitez supprimer.
   :::image type="content" source="./media/server-trust-group-overview/server-trust-group-manage-select.png" alt-text="Groupes d’approbations de serveurs":::
5. Cliquez sur **Supprimer un groupe** .
   :::image type="content" source="./media/server-trust-group-overview/server-trust-group-manage-delete.png" alt-text="Groupes d’approbations de serveurs":::
6. Entrez le nom du groupe d’approbations de serveurs pour confirmer la suppression, puis cliquez sur **Supprimer** .
   :::image type="content" source="./media/server-trust-group-overview/server-trust-group-manage-delete-confirm.png" alt-text="Groupes d’approbations de serveurs":::

> [!NOTE]
> La suppression du groupe d’approbations de serveurs risque de ne pas supprimer immédiatement l’approbation entre les deux instances managées. La suppression de l’approbation peut être appliquée en appelant un [basculement](/powershell/module/az.sql/Invoke-AzSqlInstanceFailover) d’instances managées. Consultez les [Problèmes connus](../database/doc-changes-updates-release-notes.md?tabs=managed-instance#known-issues) pour obtenir les dernières mises à jour à ce sujet.

## <a name="limitations"></a>Limites

Au cours de la préversion publique, les limitations suivantes s’appliquent aux groupes d’approbations de serveur.
 * Le nom du groupe d’approbations de serveurs doit être unique dans toutes les régions où ses membres se trouvent.
 * Le groupe peut contenir uniquement des instances Azure SQL Managed Instance et il doit se trouver dans le même abonnement Azure.
 * En préversion, le groupe peut avoir exactement deux instances managées. Pour exécuter des transactions distribuées sur plus de deux instances managées, vous devez créer un groupe d’approbations de serveurs pour chaque paire d’instances managées.
 * Les transactions distribuées sont la seule étendue applicable pour les groupes d’approbations de serveurs.
 * Le groupe d’approbations de serveurs peut uniquement être géré à partir du Portail Azure. La prise en charge de PowerShell et de l’interface CLI sera ajoutée plus tard.
 * Il est impossible de modifier le groupe d’approbations de serveurs sur le Portail Azure. Il peut uniquement être créé ou supprimé.
 * Des limitations supplémentaires sur les transactions distribuées peuvent être liées à votre scénario. La plus notable est qu’il doit exister une connectivité entre les instances managées sur des points de terminaison privés, par le biais d’un réseau virtuel ou d’un peering de réseaux virtuels. Assurez-vous que vous avez pris connaissance des [limitations actuelles des transactions distribuées pour Managed Instance](../database/elastic-transactions-overview.md#limitations).

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur les transactions distribuées dans Azure SQL Managed Instance, consultez [Transactions distribuées](../database/elastic-transactions-overview.md).
* Pour obtenir les mises à jour de version et l’état des problèmes connus, consultez [Notes de publication Managed Instance](../database/doc-changes-updates-release-notes.md).
* Si vous avez des demandes de fonctionnalités, ajoutez-les au [forum Managed Instance](https://feedback.azure.com/forums/915676-sql-managed-instance).
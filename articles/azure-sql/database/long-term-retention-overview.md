---
title: Rétention des sauvegardes à long terme
titleSuffix: Azure SQL Database & Azure SQL Managed Instance
description: Découvrez comment Azure SQL Database et Azure SQL Managed Instance prennent en charge le stockage de sauvegardes de bases de données complètes pendant 10 ans maximum par le biais de la stratégie de conservation à long terme.
services: sql-database
ms.service: sql-database
ms.subservice: operations
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: anosov1960
ms.author: sashan
ms.reviewer: mathoma, sstein
ms.date: 05/18/2019
ms.openlocfilehash: 8250fc39fe58168ddc13b7bcf5c040b57d5e92fb
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92782618"
---
# <a name="long-term-retention---azure-sql-database-and-azure-sql-managed-instance"></a>Conservation à long terme – Azure SQL Database et Azure SQL Managed Instance

De nombreuses applications sont dédiées à la réglementation, à la conformité ou à d’autres fins professionnelles qui vous obligent à conserver des sauvegardes de données au-delà des 7 à 35 jours offerts par les [sauvegardes automatiques](automated-backups-overview.md) Azure SQL Database et Azure SQL Managed Instance. La fonctionnalité de conservation à long terme (LTR) vous permet de stocker certaines sauvegardes complètes de SQL Database et SQL Managed Instance dans le Stockage Blob Azure avec une [redondance configurée](automated-backups-overview.md#backup-storage-redundancy) pour jusqu’à 10 ans. Vous pouvez ensuite restaurer la sauvegarde de votre choix en tant que nouvelle base de données.

La conservation à long terme peut être activée pour Azure SQL Database et est en préversion publique limitée pour Azure SQL Managed Instance. Cet article fournit une vue d’ensemble conceptuelle de la conservation à long terme. Pour configurer la conservation à long terme, consultez [Configurer la conservation à long terme dans Azure SQL Database](long-term-backup-retention-configure.md) et [Configurer la conservation à long terme dans Azure SQL Managed Instance](../managed-instance/long-term-backup-retention-configure.md). 

> [!NOTE]
> Vous pouvez utiliser des travaux SQL Agent pour planifier des [sauvegardes de base de données en copie seule](/sql/relational-databases/backup-restore/copy-only-backups-sql-server) comme alternative à la conservation à long terme au-delà de 35 jours.


## <a name="how-long-term-retention-works"></a>Fonctionnement de la conservation à long terme
     
La rétention des sauvegardes à long terme (LTR) s’appuie sur les sauvegardes intégrales des bases de données qui sont [créées automatiquement](automated-backups-overview.md) pour permettre la récupération jusqu’à une date et heure. Si une stratégie LTR est spécifiée, ces sauvegardes sont copiées vers différents objets blob pour stockage à long terme. La copie est un travail en arrière-plan qui n’a aucun impact en matière de performance sur la charge de travail de la base de données. La stratégie LTR pour chaque base de données dans SQL Database peut également spécifier la fréquence à laquelle les sauvegardes LTR sont créées.

Pour activer LTR, vous pouvez définir une stratégie utilisant une combinaison de quatre paramètres : rétention hebdomadaire des sauvegardes (W), rétention mensuelle des sauvegardes (M), rétention annuelle des sauvegardes (Y) et semaine de l’année (WeekOfYear). Si vous indiquez W, une sauvegarde est copiée sur le dispositif de stockage à long terme toutes les semaines. Si vous indiquez M, la première sauvegarde de chaque mois est copiée sur le stockage à long terme. Si vous indiquez Y, une sauvegarde est copiée sur le dispositif de stockage à long terme pendant la semaine définie par WeekOfYear. Si la valeur WeekOfYear spécifiée est dans le passé lorsque la stratégie est configurée, la première sauvegarde LTR sera créée l’année suivante. Chaque sauvegarde est conservée dans le stockage à long terme en fonction des paramètres de stratégie qui sont configurés lors de la création de la sauvegarde LTR.

> [!NOTE]
> Toute modification apportée à la stratégie LTR s’applique uniquement aux sauvegardes ultérieures. Par exemple, si la rétention hebdomadaire des sauvegardes (W), la rétention mensuelle des sauvegardes (M) ou la rétention annuelle des sauvegardes (Y) est modifiée, le nouveau paramètre de rétention s’applique uniquement aux nouvelles sauvegardes. La rétention des sauvegardes existantes ne sera pas modifiée. Si vous souhaitez supprimer les anciennes sauvegardes LTR avant l’expiration de leur période de rétention, vous devez [supprimer manuellement les sauvegardes](./long-term-backup-retention-configure.md#delete-ltr-backups).
> 

Exemples de stratégie LTR :

-  W=0, M=0, Y=5, WeekOfYear=3

   La troisième sauvegarde complète de l’année est conservée pendant cinq ans.
   
- W=0, M=3, Y=0

   La première sauvegarde complète du mois est conservée pendant trois mois.

- W=12, M=0, Y=0

   Chaque sauvegarde complète hebdomadaire est conservée pendant 12 semaines.

- W=6, M=12, Y=10, WeekOfYear=16

   Chaque sauvegarde complète hebdomadaire est conservée pendant six semaines. à l’exception de la 1re sauvegarde complète du mois qui est conservée pendant 12 mois et de la sauvegarde complète effectuée durant la 16e semaine de l’année et qui est quant à elle conservée pendant 10 ans. 

Le tableau suivant illustre la cadence et l’expiration des sauvegardes à long terme pour la stratégie suivante :

W=12 semaines (84 jours), M=12 mois (365 jours), Y=10 ans (3 650 jours), WeekOfYear=15 (semaine après le 15 avril)

   ![Par exemple](./media/long-term-retention-overview/ltr-example.png)


Si vous modifiez la stratégie ci-dessus et définissez W=0 (aucune sauvegarde hebdomadaire), la cadence des copies de sauvegarde sera modifiée comme indiqué dans le tableau ci-dessus par les dates en surbrillance. La quantité de stockage nécessaire pour conserver ces sauvegardes diminuerait en conséquence. 

> [!IMPORTANT]
> Le calendrier des sauvegardes LTR individuelles est contrôlé par Azure. Vous ne pouvez pas créer manuellement une sauvegarde LTR ni contrôler le calendrier de création de sauvegarde. Après avoir configuré une stratégie de rétention à long terme (LTR), vous devrez peut-être patienter jusqu’à sept jours avant que la première sauvegarde LTR n’apparaisse dans la liste des sauvegardes disponibles.  


## <a name="geo-replication-and-long-term-backup-retention"></a>Géo-réplication et conservation de sauvegarde à long terme

Si vous utilisez une géoréplication active ou des groupes de basculement en tant que solution de continuité des activités, vous devez vous préparer à d’éventuels basculements et configurer la même stratégie LTR sur l’instance ou la base de données secondaire. Le coût de votre stockage LTR n’augmente pas, car les sauvegardes ne sont pas générées à partir des bases de données secondaires. Les sauvegardes sont créées uniquement quand la base de données secondaire devient primaire. Cela garantit une génération ininterrompue des sauvegardes LTR lorsque le basculement est déclenché et lorsque la base de données primaire est déplacée vers la région secondaire. 

> [!NOTE]
> Lorsque la base de données primaire d’origine récupère après une panne ayant entraîné le basculement, elle devient une nouvelle base de données secondaire. Par conséquent, la création de sauvegarde ne reprend pas, et la stratégie de conservation à long terme existante ne prend effet qu’après que la base de données est redevenue primaire. 

## <a name="sql-managed-instance-support"></a>Prise en charge de SQL Managed Instance

L’utilisation de la conservation des sauvegardes à long terme avec Azure SQL Managed Instance présente les limites suivantes :

- **Préversion publique limitée**  : cette préversion n'est disponible que pour les abonnements EA et CSP et est soumise à une disponibilité limitée.  
- [**PowerShell uniquement**](../managed-instance/long-term-backup-retention-configure.md) : aucune prise en charge du portail Azure n’est actuellement disponible. La rétention à long terme doit être activée à l’aide de PowerShell. 

Pour demander l’inscription, créez un [ticket de support Azure](https://azure.microsoft.com/support/create-ticket/). Pour le type de problème, sélectionnez « Problème technique », pour le service, choisissez « SQL Managed Instance » et, pour le type de problème, sélectionnez **Sauvegarde, restauration et continuité d’activité / Conservation des sauvegardes à long terme** . Dans votre demande, indiquez que vous souhaitez être inscrit à la préversion publique limitée de LTR pour SQL Managed Instance.

## <a name="configure-long-term-backup-retention"></a>Configurer la rétention des sauvegardes à long terme

Vous pouvez configurer la conservation des sauvegardes à long terme à l’aide du portail Azure et de PowerShell pour Azure SQL Database, et PowerShell pour Azure SQL Managed Instance. Pour restaurer une base de données à partir du stockage LTR, vous pouvez sélectionner une sauvegarde spécifique en fonction de son horodatage. Vous pouvez restaurer la base de données sur n’importe quel serveur ou instance managée existant en utilisant le même abonnement que celui de la base de données d’origine.

Pour apprendre à configurer la conservation à long terme ou à restaurer une base de données à partir d’une sauvegarde pour SQL Database à l’aide du portail Azure ou de PowerShell, consultez [Gérer la conservation à long terme des sauvegardes Azure SQL Database](long-term-backup-retention-configure.md).

Pour savoir comment configurer la conservation à long terme ou restaurer une base de données à partir d’une sauvegarde pour Azure SQL Managed Instance à l’aide de PowerShell, consultez [Gérer la conservation à long terme des sauvegardes Azure SQL Managed Instance](../managed-instance/long-term-backup-retention-configure.md).

Pour restaurer une base de données à partir du stockage LTR, vous pouvez sélectionner une sauvegarde spécifique en fonction de son horodatage. Vous pouvez restaurer la base de données sur n’importe quel serveur existant, en utilisant le même abonnement que celui de la base de données d’origine. Pour apprendre à restaurer votre base de données à partir d’une sauvegarde LTR à l’aide du Portail Azure ou de PowerShell, consultez [Gérer la rétention à long terme des sauvegardes Azure SQL Database](long-term-backup-retention-configure.md). Dans votre demande, indiquez que vous souhaitez être inscrit à la préversion publique limitée de LTR pour SQL Managed Instance.

## <a name="next-steps"></a>Étapes suivantes

Dans la mesure où les sauvegardes de base de données protègent les données des corruptions et des suppressions accidentelles, elles sont une partie essentielle de toute stratégie de continuité d’activité ou de récupération d’urgence. Pour en savoir plus sur les autres solutions de continuité des activités SQL Database, consultez [Vue d’ensemble de la continuité des activités](business-continuity-high-availability-disaster-recover-hadr-overview.md).

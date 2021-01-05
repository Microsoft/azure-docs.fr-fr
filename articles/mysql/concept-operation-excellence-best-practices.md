---
title: Meilleures pratiques opérationnelles pour les serveurs MySQL – Azure Database pour MySQL
description: Cet article décrit les meilleures pratiques opérationnelles d’une base de données MySQL sur Azure.
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: conceptual
ms.date: 11/23/2020
ms.openlocfilehash: e1d84483a701bf7fb4e6c50b9dc4f4f89993c64d
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96354869"
---
# <a name="best-practices-for-server-operations-on-azure-database-for-mysql--single-server"></a>Meilleures pratiques d’opérations serveur sur Azure Database pour MySQL – Serveur unique

Découvrez les meilleures pratiques d’utilisation d’Azure Database pour MySQL. Au fur et à mesure que nous ajouterons de nouvelles fonctionnalités à la plateforme, nous continuerons de nous consacrer à affiner les meilleures pratiques décrites dans cette section.

## <a name="azure-database-for-mysql-operational-guidelines"></a>Instructions opérationnelles pour Azure Database pour MySQL 

Vous trouverez ci-dessous les instructions opérationnelles à suivre pour utiliser une base de données Azure Database pour MySQL en améliorant son niveau de performance : 

* **Colocalisation** : Pour réduire le temps de réponse du réseau, placez le client et le serveur de base de données dans la même région Azure.

* **Monitoring de l’utilisation de la mémoire, du processeur et du stockage** : vous pouvez [configurer des alertes](howto-alert-on-metric.md) pour savoir quand les modèles d’utilisation changent et quand vous approchez de la capacité de votre déploiement, dans l’objectif de maintenir le niveau de performance et la disponibilité du système. 

* **Scale-up de l’instance de base de données** : vous pouvez effectuer un [scale-up](howto-create-manage-server-portal.md) lorsque vous approchez des limites de la capacité de stockage. Gardez une marge en matière de stockage et de mémoire pour gérer les augmentations imprévues de la demande de vos applications. Vous pouvez également [activer la fonctionnalité de croissance automatique du stockage](howto-auto-grow-storage-portal.md) pour que le service mette automatiquement à l’échelle le stockage lorsqu’il approche de ses limites. 

* **Configuration des sauvegardes** : activez les [sauvegardes locales ou géoredondantes](howto-restore-server-portal.md#set-backup-configuration) selon l’exigence de l’entreprise. Modifiez également la période de rétention des sauvegardes en fonction de la durée nécessaire à la continuité d’activité. 

* **Augmentation de la capacité d’E/S** : si le volume d’E/S nécessaire à la charge de travail de la base de données est supérieur à ce qui a été provisionné, la récupération et d’autres opérations transactionnelles de la base de données seront lentes. Pour augmenter la capacité d’E/S d’une instance de serveur, procédez comme suit : 

    * Azure Database pour MySQL assure la mise à l’échelle des IOPS à raison de trois IOPS par Go de stockage provisionné. [Augmentez le stockage provisionné](howto-create-manage-server-portal.md#scale-storage-up) pour augmenter les IOPS et ainsi améliorer le niveau de performance. 

    * Si vous utilisez déjà le stockage des IOPS provisionnées, configurez une [capacité de débit supplémentaire](howto-create-manage-server-portal.md#scale-storage-up). 

* **Mise à l’échelle du calcul** : la charge de travail de la base de données peut également être limitée par le processeur ou la mémoire, ce qui risque d’avoir une sérieuse incidence sur le traitement transactionnel. Notez que le scale-up et le scale-down du calcul (niveau tarifaire) ne sont possibles qu’entre les niveaux [Usage général et Mémoire optimisée](concepts-pricing-tiers.md). 

* **Test de basculement** : testez manuellement le basculement de votre instance de serveur afin de connaître la durée d’exécution du processus pour votre cas d’usage et de vérifier que l’application qui accède à votre instance de serveur peut se connecter automatiquement à la nouvelle instance après le basculement.

* **Clé primaire** : veillez à ce que vos tables comportent une clé primaire ou unique lorsque vous travaillez sur la base de données Azure Database pour MySQL. La réalisation de sauvegardes, de réplicas, etc. s’en trouve grandement facilitée, et le niveau de performance amélioré.

* **Configuration de la valeur TTL** : si votre application cliente met en cache les données DNS (Domain Name System) de vos instances de serveur, définissez une valeur de durée de vie (TTL) inférieure à 30 secondes. Étant donné que l’adresse IP sous-jacente d’une instance de serveur peut changer après un basculement, la mise en cache des données DNS pendant une période prolongée risque d’entraîner des échecs de connexion si l’application tente de se connecter à une adresse IP qui n’est plus en service.

* Utilisez le regroupement de connexions pour éviter d’atteindre les [limites de connexion maximales](concepts-server-parameters.md#max_connections) et suivez une logique de nouvelles tentatives pour éviter des problèmes de connexion intermittents. 

* Si vous avez recours à un réplica, utilisez [ProxySQL](https://techcommunity.microsoft.com/t5/azure-database-for-mysql/scaling-an-azure-database-for-mysql-workload-running-on/ba-p/1105847) pour équilibrer la charge entre le serveur principal et le serveur de réplication secondaire accessible en lecture. Consultez la procédure de configuration. </br> 

* Lors du provisionnement de la ressource, veillez à [activer la croissance automatique](howto-auto-grow-storage-portal.md) sur votre base de données Azure Database pour MySQL. Cette fonctionnalité protège la base de données contre les goulots d’étranglement du stockage sans ajouter de coût supplémentaire. </br> 


### <a name="using-innodb-with-azure-database-for-mysql"></a>Utilisation d’InnoDB avec Azure Database pour MySQL

*   La fonctionnalité `ibdata1` consiste en un fichier de données tablespace système que l’on ne peut ni réduire ni vider en supprimant les données de la table ou en déplaçant la table vers file_per_table `tablespaces`.

* Pour une base de données d’une taille supérieure à 1 To, créez la table dans **innodb_file_per_table** `tablespace`. Dans le cas d’une table unique d’une taille supérieure à 1 To, utilisez la table [partition](https://dev.mysql.com/doc/refman/5.7/en/partitioning.html).

*   Pour un serveur comportant un grand nombre de `tablespace`, le démarrage du moteur est très lent en raison de l’analyse séquentielle des tablespace qui a lieu pendant le démarrage ou le basculement de MySQL. 

* Définissez innodb_file_per_table = ON avant de créer une table si le nombre total de tables est inférieur à 500.

* Si votre base de données comporte plus de 500 tables, vérifiez individuellement la taille de chacune d’elles. Dans le cas d’une table volumineuse, envisagez toujours d’utiliser file_per_table tablespace pour éviter que le fichier tablespace système n’atteigne la limite de stockage maximale.

> [!NOTE]
> Pour les tables d’une taille inférieure à 5 Go, envisagez d’utiliser le tablespace système : 
> ```sql
>    CREATE TABLE tbl_name ... *TABLESPACE* = *innodb_system*;
> ```

* [Partitionnez](https://dev.mysql.com/doc/refman/5.7/en/partitioning.html) votre table lors de sa création si elle est très volumineuse et susceptible de dépasser 1 To.

* Utilisez plusieurs serveurs MySQL et répartissez les tables entre eux. Évitez de placer trop de tables sur un seul serveur si vous disposez d’environ 10 000 tables, voire plus. 

## <a name="next-steps"></a>Étapes suivantes
- [Meilleures pratiques relatives au niveau de performance d’Azure Database pour MySQL](concept-performance-best-practices.md)
- [Meilleures pratiques de monitoring d’Azure Database pour MySQL](concept-monitoring-best-practices.md)

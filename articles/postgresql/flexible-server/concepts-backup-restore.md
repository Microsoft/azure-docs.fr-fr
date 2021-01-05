---
title: Sauvegarder et restaurer dans Azure Database pour PostgreSQL - Serveur flexible
description: Découvrir les concepts de sauvegarde et de restauration avec Azure Database pour PostgreSQL - Serveur flexible
author: sr-msft
ms.author: srranga
ms.service: postgresql
ms.topic: conceptual
ms.date: 09/22/2020
ms.openlocfilehash: d0e79e42c7c004638336ada23de663bbe74b7e48
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92532643"
---
# <a name="backup-and-restore-in-azure-database-for-postgresql---flexible-server"></a>Sauvegarder et restaurer dans Azure Database pour PostgreSQL - Serveur flexible

> [!IMPORTANT]
> Azure Database pour PostgreSQL - Serveur flexible est en préversion

Les sauvegardes constituent une partie essentielle de toute stratégie de continuité de l’activité. Elles aident à protéger les données contre toute altération ou suppression accidentelle. Azure Database pour PostgreSQL - Serveur flexible sauvegarde automatiquement votre serveur et conserve les sauvegardes pendant une durée allant jusqu’à 35 jours. Lors de la restauration, vous pouvez spécifier la date et l’heure auxquelles vous souhaitez effectuer la restauration dans les limites de la période de conservation. La durée globale nécessaire à la restauration et à la récupération dépend de la taille des fichiers de base de données et de la quantité de récupération à effectuer. 

### <a name="backup-process-in-flexible-server"></a>Processus de sauvegarde sur serveur flexible
La première sauvegarde de capture instantanée est planifiée immédiatement après la création du serveur flexible. Par la suite, une sauvegarde d’instantané des fichiers de données sera effectuée quotidiennement. Les sauvegardes sont stockées dans un stockage redondant interzone à l’intérieur d’une région. Les journaux des transactions (journaux WAL [write-ahead logs]) sont également archivés en continu, ce qui vous permet de restaurer jusqu’à la dernière transaction validée. Ces sauvegardes de données et de journaux vous permettent de restaurer un serveur à n’importe quel point dans le temps compris dans la période de conservation des sauvegardes qui est configurée. Toutes les sauvegardes sont chiffrées à l’aide du chiffrement AES de 256 bits.

Si la base de données est configurée avec une haute disponibilité, les instantanés quotidiens sont capturés à partir du serveur principal et les sauvegardes de journaux continues sont effectuées à partir du serveur de secours.

> [!IMPORTANT]
>Les sauvegardes ne sont pas effectuées sur les serveurs arrêtés. Toutefois, elles sont reprises lorsque la base de données est démarrée automatiquement après sept jours ou démarrée par l’utilisateur.

Les sauvegardes sont utilisables uniquement pour les opérations de restauration sur le serveur flexible. Si vous souhaitez exporter ou importer des données vers le serveur flexible, vous devez utiliser la méthodologie [vidage et restauration](../howto-migrate-using-dump-and-restore.md).


### <a name="backup-retention"></a>Rétention des sauvegardes

Les sauvegardes sont conservées en fonction du paramètre de période de conservation de sauvegarde pour le serveur. Vous pouvez sélectionner une période de conservation comprise entre 7 et 35 jours. La période de conservation par défaut est de sept jours. Vous pouvez définir la période de conservation lors de la création du serveur, ou la mettre à jour ultérieurement. Les sauvegardes sont conservées même pour les serveurs arrêtés.

La période de conservation de sauvegarde détermine jusqu’à quelle date une restauration à un instant dans le passé peut être récupérée, dans la mesure où elle est basée sur les sauvegardes disponibles. La période de rétention de sauvegarde peut également être traitée comme une fenêtre de récupération du point de vue de la restauration. Toutes les sauvegardes requises pour effectuer une restauration à un instant dans le passé au cours de la période de conservation de sauvegarde sont conservées dans le stockage de sauvegarde. Par exemple, si la période de conservation des sauvegardes est définie sur sept jours, la fenêtre de récupération est considérée comme les sept derniers jours. Dans ce scénario, toutes les données et tous les journaux nécessaires à la restauration et à la récupération du serveur au cours des sept derniers jours sont conservés. 


### <a name="backup-storage-cost"></a>Coût du stockage de sauvegarde

Le serveur flexible fournit jusqu’à 100 % du stockage de votre serveur provisionné en stockage de sauvegarde sans coût supplémentaire. Tous les stockages de sauvegarde supplémentaires utilisés sont facturés en Go par mois. Par exemple, si vous avez configuré un serveur avec 250 Gio de stockage, vous avez une capacité de stockage de sauvegarde de 250 Gio sans coût supplémentaire. Si l’utilisation quotidienne de la sauvegarde est de 25 Gio, vous pouvez avoir jusqu’à 10 jours de stockage de sauvegarde gratuit. La consommation de stockage de sauvegarde supérieure à 250 Gio est facturée selon le [modèle tarifaire](https://azure.microsoft.com/pricing/details/postgresql/).

Vous pouvez utiliser la métrique [Stockage de sauvegarde utilisé](../concepts-monitoring.md) sur le portail Azure afin de superviser le stockage de sauvegarde qui est consommé par un serveur. La métrique « Stockage de sauvegarde utilisé » représente le total du stockage consommé par l’ensemble des sauvegardes de base de données et des sauvegardes de journaux qui sont conservées en fonction de la période de conservation des sauvegardes qui est définie pour le serveur.  Une activité transactionnelle importante sur le serveur peut entraîner une augmentation de l’utilisation du stockage de sauvegarde, quelle que soit la taille totale de la base de données.

Le principal moyen de contrôler le coût fu stockage de sauvegarde consiste à définir la période de rétention de sauvegarde appropriée et à choisir les options de redondance de sauvegarde appropriées pour atteindre les objectifs de récupération souhaités.

> [!IMPORTANT]
> Les sauvegardes géoredondantes ne sont pas prises en charge avec le serveur flexible.

## <a name="point-in-time-restore-overview"></a>Vue d’ensemble de la restauration à un instant dans le passé

Dans le serveur flexible, l’exécution d’une restauration à un instant dans le passé crée un nouveau serveur à partir des sauvegardes du serveur flexible dans la même région que votre serveur source. Il est créé avec la configuration du serveur source pour le niveau tarifaire, la génération de calcul, le nombre de vCores, la taille de stockage, la période de conservation de sauvegarde et l’option de redondance de sauvegarde. De plus, les étiquettes et les paramètres, tels que les paramètres de réseau virtuel et de pare-feu, sont hérités du serveur source. 

 > [!IMPORTANT]
> Si vous restaurez un serveur flexible configuré avec la haute disponibilité avec redondance interzone, le serveur restauré sera configuré sans haute disponibilité et dans la même région que votre serveur principal. 

 ### <a name="restore-process"></a>Processus de restauration

Les fichiers de la base de données physique sont d’abord restaurés à partir des sauvegardes d’instantanés à l’emplacement des données du serveur. La sauvegarde appropriée qui a été effectuée avant l’instant dans le passé souhaité est automatiquement choisie et restaurée. Un processus de récupération est ensuite lancé à l’aide des fichiers WAL pour ramener la base de données à un état cohérent. 

 Supposez par exemple que les sauvegardes soient effectuées chaque soir à 23h00. Si le point de restauration est le 15 août 2020 à 10h00, la sauvegarde quotidienne du 14 août 2020 est restaurée. La base de données sera récupérée jusqu’à 10h00 le 25 août 2020 à l’aide de la sauvegarde des journaux de transactions entre le 24 août à 23h00 et le 25 août à 10h00. 

 Pour restaurer votre serveur de base de données, consultez [ces étapes](./how-to-restore-server-portal.md).

> [!IMPORTANT]
> Sur le serveur flexible, les opérations de restauration créent un nouveau serveur de base de données sans remplacer le serveur existant.

La restauration à un point dans le temps est utile dans plusieurs scénarios. Par exemple, lorsqu’un utilisateur supprime accidentellement des données, perd une base de données ou une table importante ou si une application remplace accidentellement des données correctes par des données erronées en raison d’un défaut de l’application. Vous pourrez restaurer jusqu’à la dernière transaction en raison de la sauvegarde continue des journaux des transactions.

Vous pouvez choisir entre le point de restauration le plus ancien et un point de restauration personnalisé.

-   **Point de restauration le plus ancien**  : en fonction de votre période de conservation, il s’agira de l’heure la plus ancienne à laquelle vous pouvez restaurer. L’heure de sauvegarde la plus ancienne sera automatiquement sélectionnée et affichée dans le portail. C’est utile si vous souhaitez procéder à une investigation ou à des tests à partir de cet instant dans le passé.

-   **Point de restauration personnalisé**  : cette option vous permet de choisir n’importe quel instant dans le passé au cours de la période de conservation définie pour ce serveur flexible. Par défaut, l’heure UTC la plus récente est sélectionnée automatiquement. C’est utile si vous souhaitez restaurer jusqu’à la dernière transaction validée à des fins de test. Vous pouvez éventuellement choisir d’autres jours et heures. 

La durée de récupération estimée dépend de plusieurs facteurs, notamment la taille des bases de données, le volume des journaux des transactions à traiter, la bande passante réseau et le nombre total de bases de données à récupérer dans la même région au même moment. La durée de récupération globale est généralement comprise entre quelques minutes et quelques heures.


> [!IMPORTANT]
> Il n’est **pas** possible de restaurer des serveurs supprimés. Si vous supprimez le serveur, toutes les bases de données qui appartiennent au serveur sont également supprimées, sans pouvoir être restaurées. À l'issue du déploiement, pour protéger les ressources du serveur d'une suppression accidentelle ou de changements inattendus, les administrateurs peuvent utiliser des [verrous de gestion](../../azure-resource-manager/management/lock-resources.md).

## <a name="perform-post-restore-tasks"></a>Effectuer des tâches de post-restauration

Après la restauration de la base de données, vous pouvez effectuer les tâches suivantes afin que les utilisateurs et applications soient de nouveau opérationnels :

-   Si le nouveau serveur est censé remplacer le serveur d’origine, redirigez les clients et les applications clientes vers le nouveau serveur.

-   Vérifiez que les règles de pare-feu et de réseau virtuel appropriées sont en place au niveau du serveur pour permettre aux utilisateurs de se connecter. Ces règles ne sont pas copiées à partir du serveur d’origine.
  
-   Le calcul du serveur restauré peut subir un scale-up/down en fonction des besoins.

-   Vérifiez que les connexions et les autorisations appropriées au niveau de la base de données sont en place.

-   Configurer les alertes, selon les besoins.
  
-  Si vous avez restauré la base de données configurée avec une haute disponibilité et que vous voulez configurer le serveur restauré avec une haute disponibilité, vous pouvez effectuer [ces étapes](./how-to-manage-high-availability-portal.md).


## <a name="next-steps"></a>Étapes suivantes

-   Découvrir la [continuité de l’activité](./concepts-business-continuity.md)
-   Découvrir la [haute disponibilité avec redondance interzone](./concepts-high-availability.md)
-   Découvrir [comment effectuer une restauration](./how-to-restore-server-portal.md)
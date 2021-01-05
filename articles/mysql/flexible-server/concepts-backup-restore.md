---
title: Sauvegarder et restaurer dans un serveur flexible Azure Database pour MySQL
description: Découvrez les concepts de sauvegarde et de restauration avec un serveur flexible Azure Database pour MySQL
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: conceptual
ms.date: 09/21/2020
ms.openlocfilehash: 2d69427f9f11a47cedeccb4b1da38b770952f029
ms.sourcegitcommit: 80034a1819072f45c1772940953fef06d92fefc8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93240764"
---
# <a name="backup-and-restore-in-azure-database-for-mysql-flexible-server-preview"></a>Sauvegarder et restaurer dans un serveur flexible Azure Database pour MySQL (préversion)

> [!IMPORTANT] 
> Azure Database pour MySQL – Serveur flexible est actuellement en préversion publique.

Un serveur flexible Azure Database pour MySQL crée automatiquement des sauvegardes de serveur et les stocke de manière sécurisée dans un stockage localement redondant au sein de la région. Les sauvegardes peuvent être utilisées pour restaurer votre serveur à un point dans le temps. La sauvegarde et la restauration sont une partie essentielle de toute stratégie de continuité d’activité, dans la mesure où elles protègent vos données des corruptions et des suppressions accidentelles.

## <a name="backup-overview"></a>Présentation de la sauvegarde

Un serveur flexible prend des sauvegardes de capture instantanée des fichiers de données et les stocke dans un stockage localement redondant. Le serveur effectue également une sauvegarde des journaux des transactions qu’il stocka dans un stockage localement redondant. Celles-ci vous permettent de restaurer un serveur à n’importe quel point dans le temps au sein de votre période de rétention de sauvegarde configurée. La période de rétention de sauvegarde par défaut est de sept jours. Vous pouvez éventuellement configurer la sauvegarde de la base de données de 1 à 35 jours. Toutes les sauvegardes sont chiffrées à l’aide du chiffrement AES 256 bits pour les données stockées au repos.

Vous ne pouvez pas exporter ces fichiers de sauvegarde. Les sauvegardes sont utilisables uniquement pour les opérations de restauration dans le serveur flexible. Vous pouvez également utiliser[mysqldump](../concepts-migrate-dump-restore.md#dump-and-restore-using-mysqldump-utility) à partir d’un client MySQL pour copier une base de données.

## <a name="backup-frequency"></a>Fréquence de sauvegarde

Les sauvegardes sur des serveurs flexibles sont basées sur des captures instantanées. La première sauvegarde de capture instantanée complète est planifiée immédiatement après la création d’un serveur. La première sauvegarde complète de capture instantanée est conservée en tant que sauvegarde de base du serveur. Les sauvegardes de captures instantanées suivantes sont des sauvegardes différentielles uniquement.

Les sauvegardes de captures instantanées différentielles se produisent au moins une fois par jour. Les sauvegardes de captures instantanées différentielles ne se produisent pas selon une planification fixe. Les sauvegardes de captures instantanées différentielles ont lieu toutes les 24 heures, sauf si les journaux binaires dans MySQL dépassent 50 Go depuis la dernière sauvegarde différentielle. Au cours d’une journée, six captures instantanées différentielles maximum sont autorisées. Les sauvegardes des journaux des transactions se produisent toutes les cinq minutes.

## <a name="backup-retention"></a>Rétention des sauvegardes

Les sauvegardes de base de données sont enregistrées dans un stockage localement redondant (LRS) qui est stocké dans trois copies au sein d’une région. Les sauvegardes sont conservées en fonction du paramètre de période de rétention de sauvegarde sur le serveur. Vous pouvez sélectionner une période de rétention comprise entre 1 et 35 jours avec une période de rétention par défaut de sept jours. Vous pouvez définir la période de rétention lors de la création du serveur ou ultérieurement en mettant à jour la configuration de la sauvegarde à l’aide du portail Azure.

La période de rétention de sauvegarde détermine jusqu’à quelle date une opération de restauration à un instant dans le passé peut être effectuée, dans la mesure où elle est basée sur des sauvegardes disponibles. La période de rétention de sauvegarde peut également être traitée comme une fenêtre de récupération du point de vue de la restauration. Toutes les sauvegardes requises pour effectuer une restauration à un instant dans le passé au cours de la période de rétention de sauvegarde sont conservées dans le stockage de sauvegarde. Par exemple, si la période de conservation des sauvegardes est définie sur sept jours, la fenêtre de récupération est considérée comme les sept derniers jours. Dans ce scénario, toutes les sauvegardes nécessaires pour la restauration du serveur au cours des sept derniers jours sont conservées. Avec une fenêtre de rétention de sauvegarde de sept jours, les captures instantanées de base de données et les sauvegardes du journal des transactions sont stockées pour les huit derniers jours (1 jour avant la fenêtre).

## <a name="backup-storage-cost"></a>Coût du stockage de sauvegarde

Le serveur flexible fournit jusqu’à 100 % du stockage de votre serveur provisionné en stockage de sauvegarde sans coût supplémentaire. Tous les stockages de sauvegarde supplémentaires utilisés sont facturés en Go par mois. Par exemple, si vous avez configuré un serveur avec 250 Go de stockage, vous disposez de 250 Go de stockage pour les sauvegardes du serveur sans frais supplémentaires. Si l’utilisation quotidienne de la sauvegarde est de 25 Go, vous pouvez bénéficier jusqu’à 10 jours de stockage de sauvegarde gratuit. Le stockage utilisé pour les sauvegardes de plus de 250 Go est facturé conformément au [modèle de tarification](https://azure.microsoft.com/pricing/details/mysql/).

Vous pouvez utiliser la métrique [Stockage de sauvegarde utilisé](../concepts-monitoring.md) dans Azure MoniTor disponible dans le portail Azure pour surveiller le stockage de sauvegarde consommé par un serveur. La métrique utilisée **Stockage de sauvegarde** représente le total du stockage consommé par l’ensemble des sauvegardes de base de données et des sauvegardes de journaux qui sont conservées en fonction de la période de conservation des sauvegardes définie pour le serveur. Une activité transactionnelle importante sur le serveur peut entraîner une augmentation de l’utilisation du stockage de sauvegarde, quelle que soit la taille totale de la base de données.

Le principal moyen de contrôler le coût du stockage de sauvegarde consiste à définir la période de rétention de sauvegarde appropriée. Vous pouvez sélectionner une période de rétention comprise entre 1 et  35 jours.

> [!IMPORTANT]
> Les sauvegardes à partir d’un serveur de base de données configuré dans une configuration de haute disponibilité redondante interzone se produisent à partir du serveur de base de données principal, car la surcharge est minime avec des sauvegardes de capture instantanée.

> [!IMPORTANT]
> Les sauvegardes géoredondantes ne sont pas prises en charge actuellement avec un serveur flexible.

## <a name="point-in-time-restore"></a>Restauration dans le temps

Dans un serveur flexible Azure Database pour MySQL, l’exécution d’une restauration à un instant dans le passé crée un serveur à partir des sauvegardes du serveur flexible dans la même région que votre serveur source. Elle est créée avec la configuration du serveur d’origine pour le niveau de tarification, la taille de stockage, la période de rétention de sauvegarde et l’option de redondance de sauvegarde. De plus, les étiquettes et les paramètres, par exemple, de réseau virtuel et de pare-feu, sont hérités du serveur source. Le niveau de calcul et de stockage du serveur restauré, la configuration et les paramètres de sécurité peuvent être modifiés une fois la restauration terminée.

> [!NOTE]
> Deux paramètres de serveur sont réinitialisés aux valeurs par défaut (et ne sont pas copiés à partir du serveur principal) après l’opération de restauration
> *   time_zone : valeur à définir pour la valeur DEFAULT SYSTEM
> *   event_scheduler : valeur définie sur OFF sur le serveur restauré

La restauration à un point dans le temps est utile dans plusieurs scénarios. Vois des cas d’utilisation courants :
-   Un utilisateur supprime accidentellement des données de la base de données
-   Un utilisateur supprime une table ou une base de données importantes
-   Une application d’utilisateur remplace accidentellement des données correctes par des données incorrectes en raison d’un défaut de l’application.

Vous pouvez choisir entre le dernier point de restauration et un point de restauration personnalisé via le [portail Azure](how-to-restore-server-portal.md).

-   **Dernier point de restauration**  : le dernier point de restauration vous permet de restaurer le serveur à la dernière sauvegarde effectuée sur le serveur source. L’horodateur de la restauration s’affiche également sur le portail. Cette option est utile pour restaurer rapidement le serveur à l’état le plus à jour.
-   **Point de restauration personnalisé**  : cette option vous permet de choisir n’importe quel point dans le temps au cours de la période de rétention définie pour ce serveur flexible. Cette option est utile pour restaurer le serveur à un point précis dans le temps afin récupérer une erreur de l’utilisateur.

La durée estimée de la récupération dépend de plusieurs facteurs, notamment les tailles des bases de données, la taille de sauvegarde du journal des transactions, la taille de calcul de la référence (SKU) et l’heure de la restauration. La récupération du journal des transactions est la partie la plus longue du processus de restauration. Si l’heure de restauration choisir est plus proche de la planification de sauvegarde complète ou différentielle de l’instantané, les restaurations sont plus rapides, car l’application du journal des transactions est minimale. Pour estimer le temps de récupération précis de votre serveur, nous vous recommandons vivement de le tester dans votre environnement, car il contient trop de variables spécifiques de l’environnement.

> [!IMPORTANT]
> Si vous restaurez un serveur flexible configuré avec une haute disponibilité redondante interzone, le serveur restauré sera configuré dans la même région et la même zone que votre serveur primaire, et déployé en tant que serveur flexible unique en mode non haute disponibilité. Pour un serveur flexible, consultez [Haute disponibilité redondante interzone](concepts-high-availability.md).

> [!IMPORTANT]
> Il n’est **pas** possible de restaurer des serveurs supprimés. Si vous supprimez le serveur, toutes les bases de données qui appartiennent au serveur sont également supprimées, sans pouvoir être restaurées. À l'issue du déploiement, pour protéger les ressources du serveur d'une suppression accidentelle ou de changements inattendus, les administrateurs peuvent utiliser des [verrous de gestion](../../azure-resource-manager/management/lock-resources.md).

## <a name="perform-post-restore-tasks"></a>Effectuer des tâches de post-restauration

Après une restauration à l’aide de l’un des mécanismes de récupération **Dernier point de restauration** ou **Point de restauration personnalisé** , vous devez effectuer les tâches suivantes afin que vos utilisateurs et applications soient de nouveau opérationnels :

-   Si le nouveau serveur est destiné à remplacer le serveur d’origine, redirigez les clients et les applications clientes vers le nouveau serveur.
-   Vérifiez que les règles de pare-feu et de réseau virtuel appropriées au niveau serveur sont en place pour permettre aux utilisateurs de se connecter.
-   Assurez-vous que les connexions et les autorisations appropriées au niveau de la base de données sont en place.
-   Configurer les alertes, selon les besoins.

## <a name="next-steps"></a>Étapes suivantes

-   Découvrir la [continuité de l’activité](./concepts-business-continuity.md)
-   Découvrir la [haute disponibilité avec redondance interzone](./concepts-high-availability.md)
-   Découvrir [la sauvegarde et la récupération](./concepts-backup-restore.md)

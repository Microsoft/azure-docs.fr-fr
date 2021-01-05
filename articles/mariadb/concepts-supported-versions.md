---
title: Versions prises en charge - Azure Database for MariaDB
description: Découvrez les versions du serveur MariaDB qui sont prises en charge dans le service Azure Database for MariaDB.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.topic: conceptual
ms.date: 7/20/2020
ms.openlocfilehash: dbf32626714200e6712c67b701ebc597c4a7ba7e
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94541026"
---
# <a name="supported-azure-database-for-mariadb-server-versions"></a>Versions du serveur Azure Database for MariaDB prises en charge

Azure Database for MariaDB a été développé à partir du [serveur MariaDB](https://downloads.mariadb.org/) open source, en utilisant le moteur InnoDB.

MariaDB utilise le schéma de nommage X.Y.Z. X correspond à la version majeure, Y à la version mineure et Z à la version du correctif.

> [!NOTE]
> Dans le service, une passerelle est utilisée pour rediriger les connexions vers des instances de serveur. Une fois la connexion établie, le client MySQL affiche la version de MariaDB définie dans la passerelle, et non la version en cours d’exécution sur votre instance de serveur MariaDB. Pour déterminer la version de votre instance de serveur MariaDB, utilisez la commande `SELECT VERSION();`.

Azure Database for MariaDB prend actuellement en charge la version suivante :

## <a name="mariadb-version-102"></a>MariaDB version 10.2

Version du correctif : 10.2.32

Pour découvrir plus en détail les améliorations et les correctifs introduits dans cette version, consultez la [documentation MariaDB](https://mariadb.com/kb/en/mariadb-10232-release-notes/).

## <a name="mariadb-version-103"></a>MariaDB version 10.3

Version du correctif : 10.3.23

Pour découvrir plus en détail les améliorations et les correctifs introduits dans cette version, consultez la [documentation MariaDB](https://mariadb.com/kb/en/mariadb-10323-release-notes/).

## <a name="managing-updates-and-upgrades"></a>Gestion des mises à jour et des mises à niveau
Le service gère automatiquement les mises à niveau des mises à jour correctives. Par exemple, 10.2.21 à 10.2.23.  

Les mises à niveau des versions majeures et mineures ne sont pas prises en charge. Par exemple, il n’est pas possible de mettre à niveau MariaDB 10.2 vers MariaDB 10.3. Si vous voulez effectuer une mise à niveau de 10.2 vers 10.3, prenez une [image mémoire et restaurez-la](./howto-migrate-dump-restore.md) sur un serveur créé avec la nouvelle version du moteur.

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur les quotas de ressources et les limitations associés à votre **niveau de service**, consultez la page [Niveaux de service](./concepts-pricing-tiers.md).

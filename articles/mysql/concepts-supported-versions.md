---
title: Versions prises en charge – Azure Database pour MySQL
description: Découvrez les versions du serveur MySQL qui sont prises en charge dans le service Azure Database pour MySQL.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: conceptual
ms.date: 6/3/2020
ms.openlocfilehash: 8033117d9e3c31f8aa9bba06afb7c3b1b7bba67f
ms.sourcegitcommit: 1bf144dc5d7c496c4abeb95fc2f473cfa0bbed43
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95751026"
---
# <a name="supported-azure-database-for-mysql-server-versions"></a>Versions prises en charge du serveur de base de données Azure pour MySQL

Azure Database for MySQL a été développé à partir de [MySQL Community Edition](https://www.mysql.com/products/community/), avec le moteur InnoDB.

MySQL utilise le schéma de nommage X.Y.Z. X correspond à la version majeure, Y à la version mineure et Z à la version du correctif de bogue. Pour plus d’informations sur ce schéma, reportez-vous à la [documentation MySQL](https://dev.mysql.com/doc/refman/5.7/en/which-version.html).


> [!NOTE]
> Dans le service, une passerelle est utilisée pour rediriger les connexions vers des instances de serveur. Une fois la connexion établie, le client de MySQL affiche la version de MySQL définie dans la passerelle, et non la version en cours d’exécution sur votre instance de serveur MySQL. Pour déterminer la version de votre instance de serveur MySQL, utilisez la commande `SELECT VERSION();` à l’invite de MySQL.

Azure Database pour MySQL prend actuellement en charge les versions suivantes :

## <a name="mysql-version-56"></a>MySQL version 5.6

Version du correctif de bogue : 5.6.47

Pour plus d’informations sur les améliorations et les correctifs de cette version, consultez les [notes de publication](https://dev.mysql.com/doc/relnotes/mysql/5.6/en/news-5-6-47.html) MySQL.

## <a name="mysql-version-57"></a>MySQL version 5.7

Version du correctif de bogue : 5.7.29

Pour plus d’informations sur les améliorations et les correctifs de cette version, consultez les [notes de publication](https://dev.mysql.com/doc/relnotes/mysql/5.7/en/news-5-7-29.html) MySQL.

## <a name="mysql-version-80"></a>MySQL Version 8.0

Version du correctif de bogue : 8.0.15

Pour plus d’informations sur les améliorations et les correctifs de cette version, consultez les [notes de publication](https://dev.mysql.com/doc/relnotes/mysql/8.0/en/news-8-0-15.html) MySQL.

## <a name="managing-updates-and-upgrades"></a>Gestion des mises à jour et des mises à niveau
Le service gère automatiquement les correctifs pour les mises à jour des versions des correctifs de bogues. Par exemple, 5.7.20 à 5.7.21.  

La mise à niveau des versions principales est actuellement prise en charge par le service pour les mises à niveau de MySQL v5.6 à v5.7. Pour plus d’informations, consultez [comment effectuer des mises à niveau des versions principales](how-to-major-version-upgrade.md). Si vous voulez effectuer une mise à niveau de la version 5.7 vers la version 8.0, nous vous recommandons d’effectuer [le vidage et la restauration](./concepts-migrate-dump-restore.md) sur un serveur créé avec la nouvelle version du moteur.

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur la stratégie de version d’Azure Database pour MySQL, consultez [ce document](concepts-version-policy.md).
- Pour plus d’informations sur les quotas de ressources et les limitations associés à votre **niveau de service**, consultez la page [Niveaux de service](./concepts-pricing-tiers.md).

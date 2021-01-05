---
title: Versions prises en charge – Serveur flexible Azure Database pour MySQL
description: Découvrez les versions du serveur MySQL qui sont prises en charge dans le serveur flexible Azure Database pour MySQL
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: conceptual
ms.date: 09/21/2020
ms.openlocfilehash: 7ad6a576262b8e722b16c81af544a9370c2b49b3
ms.sourcegitcommit: 80034a1819072f45c1772940953fef06d92fefc8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93242260"
---
# <a name="supported-versions-for-azure-database-for-mysql---flexible-server"></a>Versions prises en charge pour Azure Database pour MySQL – Serveur flexible


> [!IMPORTANT]
> Azure Database pour MySQL – Serveur flexible est actuellement en préversion publique.


Le serveur flexible Azure Database pour MySQL est optimisé par [MySQL Community Edition](https://www.mysql.com/products/community/) à l’aide du moteur InnoDB.

MySQL utilise le schéma de nommage X.Y.Z. X correspond à la version majeure, Y à la version mineure et Z à la version du correctif de bogue. Pour plus d’informations sur ce schéma, reportez-vous à la [documentation MySQL](https://dev.mysql.com/doc/refman/5.7/en/which-version.html).

> [!NOTE]
> Pour déterminer la version de votre instance de serveur MySQL, utilisez la commande `SELECT VERSION();` à l’invite de MySQL.

Azure Database pour MySQL prend actuellement en charge les versions suivantes :

## <a name="mysql-version-57"></a>MySQL version 5.7

Version du correctif de bogue : 5.7.29

Le service effectue une mise à jour corrective automatisée du matériel, du système d’exploitation et du moteur de base de données sous-jacents. Le correctif comprend les mises à jour de sécurité et de logiciel. Pour le moteur MySQL, les mises à niveau de version mineure sont également incluses dans le cadre de la publication de maintenance planifiée. Les utilisateurs peuvent configurer la planification de la mise à jour corrective pour qu’elle soit gérée par le système, ou définir leur planification personnalisés. Pendant la planification de la maintenance, le correctif est appliqué et le serveur peut nécessiter un redémarrage dans le cadre du processus de mise à jour corrective pour achever la mise à jour. Avec la planification personnalisée, les utilisateurs peuvent rendre leur cycle de mise à jour prévisible, et choisir une fenêtre de maintenance avec un impact minimal sur l’activité. En général, le service suit un calendrier de publication mensuel dans le cadre de l’intégration et de la publication continues.

Pour plus d’informations sur les améliorations et les correctifs de cette version, consultez les [notes de publication](https://dev.mysql.com/doc/relnotes/mysql/5.7/en/news-5-7-29.html) MySQL.

## <a name="managing-updates-and-upgrades"></a>Gestion des mises à jour et des mises à niveau
Le service gère automatiquement les correctifs pour les mises à jour des versions des correctifs de bogues. Par exemple, de la version 5.7.29 à la version 5.7.30.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
>[Créer une application PHP sur Windows avec MySQL](../../app-service/tutorial-php-mysql-app.md)<br/>
>[Créer une application PHP sur Linux avec MySQL](../../app-service/tutorial-php-mysql-app.md?pivots=platform-linux%253fpivots%253dplatform-linux)<br/>
>[Créer une application Spring basée sur Java avec MySQL](/azure/developer/java/spring-framework/spring-app-service-e2e?tabs=bash)<br/>
---
title: Stratégie de contrôle de version - Azure Database pour MySQL - Serveur unique et Serveur flexible (préversion)
description: Décrit la stratégie relative aux versions principales et mineures de MySQL prises en charge dans Azure Database pour MySQL
author: sr-msft
ms.author: srranga
ms.service: mysql
ms.topic: conceptual
ms.date: 11/03/2020
ms.custom: fasttrack-edit
ms.openlocfilehash: 4903f1e48eb2f33c68d62c635201474b841ed146
ms.sourcegitcommit: 1cf157f9a57850739adef72219e79d76ed89e264
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94591510"
---
# <a name="azure-database-for-mysql-versioning-policy"></a>Stratégie de contrôle de version Azure Database pour MySQL

Cette page décrit la stratégie de contrôle de version d’Azure Database pour MySQL et s’applique aux modes de déploiement Azure Database pour MySQL - Serveur unique et Azure Database pour MySQL - Serveur flexible (préversion).

## <a name="supported--mysql-versions"></a>Versions de MySQL prises en charge

Azure Database pour MySQL prend en charge les versions de base de données suivantes.

| Version | Serveur unique | Serveur flexible (préversion) |
| ----- | :------: | :----: |
| MySQL 8 | X |  | 
| MySQL 5.7 | X | X |
| MySQL 5.6| X |  |


## <a name="major-version-support"></a>Prise en charge de la version principale
Chaque version principale de MySQL est prise en charge par Azure Database pour MySQL à partir de la date du début de la prise en charge de la version par Azure et jusqu’à ce que la version soit supprimée par la communauté MySQL, comme indiqué dans la [stratégie de contrôle de version](https://www.mysql.com/support/eol-notice.html).

## <a name="minor-version-support"></a>Prise en charge de la version mineure
Azure Database pour MySQL effectue automatiquement des mises à niveau de versions mineures vers la version MySQL préférée d’Azure dans le cadre d’une maintenance périodique. 

## <a name="major-version-retirement-policy"></a>Stratégie de retrait de la version majeure
Le tableau ci-dessous fournit les détails relatifs au retrait des versions principales de MySQL. Les dates suivent la [stratégie de contrôle de version de MySQL](https://www.mysql.com/support/eol-notice.html).

| Version | Nouveautés | Date de début de la prise en charge d’Azure | Date de mise hors service|
| ----- | ----- | ------ | ----- |
| [MySQL 5.6](https://dev.mysql.com/doc/relnotes/mysql/5.6/en/)| [Caractéristiques](https://dev.mysql.com/doc/relnotes/mysql/5.6/en/news-5-6-49.html)  | 20 mars 2018 | Février 2021
| [MySQL 5.7](https://dev.mysql.com/doc/relnotes/mysql/5.7/en/) | [Caractéristiques](https://dev.mysql.com/doc/relnotes/mysql/5.7/en/news-5-7-31.html) | 20 mars 2018 | Octobre 2023
| [MySQL 8](https://mysqlserverteam.com/whats-new-in-mysql-8-0-generally-available/) | [Fonctionnalités](https://dev.mysql.com/doc/relnotes/mysql/8.0/en/news-8-0-21.html)) | 11 décembre 2019 | Avril 2026


## <a name="retired-mysql-engine-versions-not-supported-in-azure-database-for-mysql"></a>Versions de moteur MySQL supprimées non prises en charge dans Azure Database pour MySQL

Après la date de suppression pour chaque version de base de données MySQL, si vous continuez d’exécuter la version supprimée, tenez compte des restrictions suivantes :
- Dans la mesure où la communauté ne publie plus de correctifs de bogues ou de correctifs de sécurité, Azure Database pour MySQL n’effectue pas de mise à jour corrective sur le moteur de base de données supprimé pour la résolution de bogues ou de problèmes de sécurité. Il n’y aura pas non plus de mesures de sécurité concernant le moteur de base de données supprimé. Toutefois, Azure continuera à effectuer des opérations de maintenance et de mise à jour périodiques pour l’hôte, le système d’exploitation, les conteneurs et tout autre composant lié aux services.
- Si un problème de prise en charge rencontré est lié à la base de données MySQL, il est possible que nous ne puissions pas vous venir en aide. Dans ce cas, vous devrez mettre à niveau votre base de données afin que nous puissions intervenir.
- Vous ne serez plus en mesure de créer des serveurs de base de données pour la version supprimée. Toutefois, vous pouvez effectuer des récupérations à un instant dans le passé et créer des réplicas de lecture pour vos serveurs existants.
- Les nouvelles fonctionnalités de service développées par Azure Database pour MySQL peuvent uniquement être disponibles pour les versions de serveur de base de données prises en charge.
- Les contrats de niveau de service de durée de fonctionnement s’appliquent uniquement aux problèmes liés au service Azure Database pour MySQL et non aux temps d’arrêt causés par des bogues liés au moteur de base de données.  
- En cas de menace sérieuse pour le service provoquée par la vulnérabilité du moteur de base de données MySQL identifiée dans la version de la base de données supprimée, Azure peut décider d’arrêter votre serveur de base de données pour sécuriser le service. Il vous sera demandé de procéder à la mise à niveau du serveur avant sa mise en ligne. Pendant le processus de mise à niveau, vos données sont toujours protégées à l’aide de sauvegardes automatiques effectuées sur le service, qui peuvent être utilisées pour restaurer la version antérieure si vous le souhaitez. 



## <a name="next-steps"></a>Étapes suivantes
- Consultez Azure Database pour MySQL - Serveur unique [versions prises en charge](./concepts-supported-versions.md)
- Consultez Azure Database pour MySQL - Serveur flexible (préversion) [versions prises en charge](flexible-server/concepts-supported-versions.md)
- Consultez [Vidage et restauration](./concepts-migrate-dump-restore.md) MySQL pour effectuer des mises à niveau.

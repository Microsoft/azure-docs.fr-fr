---
title: Versions prises en charge - Azure Database pour PostgreSQL - Serveur unique
description: Décrit les versions principales et mineures de PostgreSQL prises en charge dans Azure Database pour PostgreSQL - Serveur unique.
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.topic: conceptual
ms.date: 11/16/2020
ms.custom: fasttrack-edit
ms.openlocfilehash: 72d774b4ced6471ff7b355b2cb43c3c9127b5975
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94658517"
---
# <a name="supported-postgresql-major-versions"></a>Versions principales de PostgreSQL prises en charge

Consultez [Stratégie de version Azure Database pour PostgreSQL](concepts-version-policy.md) pour plus de détails de stratégie de support.

Azure Database pour PostgreSQL prend actuellement en charge les versions principales suivantes :

## <a name="postgresql-version-11"></a>PostgreSQL Version 11
La version mineure actuelle est 11.6. Consultez la [documentation PostgreSQL](https://www.postgresql.org/docs/11/static/release-11-6.html) pour en savoir plus sur les améliorations et les correctifs de cette version mineure.

## <a name="postgresql-version-10"></a>PostgreSQL Version 10
La version mineure actuelle est 10.11. Consultez la [documentation PostgreSQL](https://www.postgresql.org/docs/10/static/release-10-11.html) pour en savoir plus sur les améliorations et les correctifs de cette version mineure.

## <a name="postgresql-version-96"></a>PostgreSQL version 9.6
La version mineure actuelle est 9.6.16. Consultez la [documentation PostgreSQL](https://www.postgresql.org/docs/9.6/static/release-9-6-16.html) pour en savoir plus sur les améliorations et les correctifs de cette version mineure.

## <a name="postgresql-version-95"></a>PostgreSQL Version 9.5
La version mineure actuelle est 9.5.20. Consultez la [documentation PostgreSQL](https://www.postgresql.org/docs/9.5/static/release-9-5-20.html) pour en savoir plus sur les améliorations et les correctifs de cette version mineure.

> [!NOTE]
> Conformément à la [stratégie de contrôle de version](https://www.postgresql.org/support/versioning/) de la communauté Postgres, Azure Database pour PostgreSQL va retirer Postgres version 9.5 le 11 février 2021. Consultez [Stratégie de version Azure Database pour PostgreSQL](concepts-version-policy.md) pour obtenir plus de détails ainsi que des descriptions.

## <a name="managing-upgrades"></a>Gestion des mises à niveau
Le projet PostgreSQL publie régulièrement des versions mineures pour corriger les bogues signalés. Azure Database pour PostgreSQL répare automatiquement les serveurs avec des versions mineures pendant les déploiements mensuels du service. 

Les mises à niveau sur place automatiques pour les versions majeures ne sont pas prises en charge. Pour effectuer une mise à niveau vers la version majeure suivante : 
   * Consultez les différentes méthodes permettant d’effectuer des [mises à niveau de version majeure à l’aide de dump et restore](./how-to-upgrade-using-dump-and-restore.md)
   * Vous pouvez utiliser [pg_dump et pg_restore](./howto-migrate-using-dump-and-restore.md) pour déplacer une base de données vers un serveur créé avec la nouvelle version du moteur.
   * Vous pouvez également procéder à une mise à niveau de PostgreSQL 10 vers la version 11 à l’aide d’[Azure Database Migration Service](..\dms\tutorial-azure-postgresql-to-azure-postgresql-online-portal.md).

### <a name="version-syntax"></a>Syntaxe de version
Avant PostgreSQL version 10, la [stratégie de gestion de version PostgreSQL](https://www.postgresql.org/support/versioning/) considérait une _mise à niveau principale_ comme une augmentation du premier _ou_ du deuxième nombre. Par exemple, une mise à niveau de la version 9.5 vers la version 9.6 était considérée comme une mise à niveau _principale_. Depuis la version 10, seule une modification du premier numéro est considérée comme une mise à niveau principale. Par exemple, une mise à niveau de la version 10.0 vers la version 10.1 est une mise à niveau _mineure_. Une mise à niveau de la version 10 à 11 est une mise à niveau _principale_.

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur les extensions PostgrSQL prises en charge, consultez [le document Extensions](concepts-extensions.md).

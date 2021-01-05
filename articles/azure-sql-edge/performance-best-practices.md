---
title: Meilleures pratiques en matière de performances et instructions de configuration – Azure SQL Edge
description: En savoir plus sur les meilleures pratiques en matière de performances et les instructions de configuration dans Azure SQL Edge
keywords: SQL Edge, conservation des données
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 09/22/2020
ms.openlocfilehash: 02f22883a0989714d8b74f778cacf1ba2c65d0b4
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93392008"
---
# <a name="performance-best-practices-and-configuration-guidelines"></a>Meilleures pratiques en matière de performances et instructions de configuration

Azure SQL Edge offre plusieurs fonctionnalités et capacités utilisables pour améliorer les performances de votre déploiement SQL Edge. Cet article présente des meilleures pratiques et des recommandations pour optimiser les performances. 

## <a name="best-practices"></a>meilleures pratiques recommandées. 

### <a name="configure-multiple-tempdb-data-files"></a>Configurer plusieurs fichiers de données tempdb

Azure SQL Edge ne crée par défaut qu’un seul fichier de données tempdb dans le cadre de l’initialisation du conteneur. Nous vous recommandons de créer plusieurs fichiers de données tempdb après le déploiement. Pour plus d’informations, reportez-vous aux instructions de l’article [Suggestions pour réduire la contention d’allocation dans la base de données tempdb SQL Server](https://support.microsoft.com/help/2154845/recommendations-to-reduce-allocation-contention-in-sql-server-tempdb-d).

### <a name="use-clustered-columnstore-indexes-where-possible"></a>Utiliser des index columnStore en cluster dans la mesure du possible

Les appareils IoT et Edge tendent à générer un volume élevé de données généralement agrégées sur une fenêtre de temps à des fins d’analyse. Des lignes de données individuelles sont rarement utilisées à des fins d’analyse. Des index columnstore sont idéaux pour stocker et interroger de tels jeux de données volumineux. Cet index utilise un stockage de données en colonnes et un traitement des requêtes pour multiplier par 10 les performances des requêtes par rapport au stockage classique orienté lignes. Vous pouvez également obtenir jusqu’à 10 fois la compression de données par rapport à la taille des données décompressées. Pour plus d’informations, consultez [Index columnstore](/sql/relational-databases/indexes/columnstore-indexes-overview)

En outre, d’autres fonctionnalités Azure SQL Edge, telles que la diffusion de données en continu et la rétention des données, bénéficient des optimisations de columnstore en matière d’insertion et de suppression de données. 

### <a name="simple-recovery-model"></a>Mode de récupération simple

Étant donné que le stockage peut être restreint sur des périphériques, toutes les bases de données utilisateur dans Azure SQL Edge utilisent par défaut le mode de récupération simple. Le mode de récupération simple récupère automatiquement l’espace de journal afin de minimiser l’espace nécessaire, ce qui élimine principalement le besoin de gérer l’espace du journal des transactions. Sur des périphériques disposant d’un stockage limité, cela peut s’avérer utile. Pour plus d’informations sur le mode de récupération simple et les autres modes de récupération disponibles, consultez [Modes de récupération](/sql/relational-databases/backup-restore/recovery-models-sql-server)

Des opérations telles que la copie des journaux de transaction et les restaurations dans le temps, qui nécessitent des sauvegardes du journal des transactions ne sont pas prises en charge par le mode de récupération simple.  

## <a name="advanced-configuration"></a>Configuration avancée 

### <a name="setting-memory-limits"></a>Définition des limites de mémoire

Si Azure SQL Edge prend en charge jusqu’à 64 Go de mémoire pour le pool de mémoires tampons, les processus satellites exécutés dans le conteneur SQL Edge peuvent nécessiter une mémoire supplémentaire. Sur des périphériques de petite taille disposant d’une mémoire limitée, il est recommandé de limiter la mémoire disponible aux conteneurs SQL Edge, de sorte que l’hôte Docker et d’autres processus ou modules Edge puissent fonctionner correctement. Vous pouvez contrôler la mémoire totale disponible pour SQL Edge à l’aide des mécanismes suivants. 

- Limitation de la mémoire disponible pour les conteneurs SQL Edge : Vous pouvez limiter la mémoire totale disponible pour le conteneur SQL Edge à l’aide de l’option de configuration de runtime de conteneurs `--memory`. Pour plus d’informations sur la limitation de la mémoire disponible pour le conteneur SQL Edge, consultez [Options de runtime avec mémoire, UC et GPU](https://docs.docker.com/config/containers/resource_constraints/).

- Limitation de la mémoire disponible pour le processus SQL dans le conteneur : Par défaut, le processus SQL dans le conteneur n’utilise que 80 % de la RAM physique disponible. Pour la majorité des déploiements, la configuration par défaut conviendra. Toutefois, il peut y avoir des scénarios où une mémoire supplémentaire peut être nécessaire pour la diffusion en continu des données et les processus ONNX s’exécutant dans les conteneurs SQL Edge. Dans de tels scénarios, la mémoire disponible pour le processus SQL peut être contrôlée à l’aide du paramètre `memory.memorylimitmb` dans le fichier mssql-conf. Pour plus d’informations, consultez [Configurer à l’aide du fichier mssql.conf](configure.md#configure-by-using-an-mssqlconf-file).

Lorsque vous définissez des limites de mémoire, veillez à ne pas définir une valeur trop basse. Si vous ne définissez pas suffisamment de mémoire pour le processus SQL, cela peut avoir un impact sérieux sur les performances SQL.

### <a name="delayed-durability"></a>Durabilité différée

Les transactions dans Azure SQL Edge peuvent avoir une durabilité complète, la durabilité par défaut de SQL Server ou une durabilité retardée (également appelée validation différée).

Les validations de transactions à durabilité complète sont synchrones. Elles signalent la réussite de la validation (COMMIT) et restituent le contrôle au client uniquement lorsque les enregistrements de journal de transactions ont été écrits sur le disque. Les validations de transactions à durabilité retardée sont asynchrones. Elles signalent la réussite de la validation (COMMIT) avant que les enregistrements de journal de transactions ne soient écrits sur le disque. L'écriture des entrées du journal des transactions sur le disque est nécessaire pour qu'une transaction soit durable. Les transactions à durabilité retardée deviennent durables lorsque les enregistrements de journal de transactions sont vidés sur le disque. 

Dans des déploiements une **certaine perte de données** peut être tolérée, ou sur des périphériques dotés d’un stockage lent, une durabilité retardée peut être utilisée pour optimiser l’ingestion des données et le nettoyage basé sur la rétention des données. Pour plus d’informations, consultez [Contrôler la durabilité d’une transaction](/sql/relational-databases/logs/control-transaction-durability).


### <a name="linux-os-configurations"></a>Configurations du système d’exploitation Linux 

Envisagez d’utiliser les paramètres de [configuration du système d’exploitation Linux](/sql/linux/sql-server-linux-performance-best-practices#linux-os-configuration) suivants pour bénéficier des meilleures performances pour une installation SQL.
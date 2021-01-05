---
title: 'Paramètres d’un serveur : Azure Database pour MySQL'
description: Cette rubrique fournit des instructions pour la configuration des paramètres de serveur dans Azure Database pour MySQL.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: conceptual
ms.date: 6/25/2020
ms.openlocfilehash: 0fddc1e8f80e257548d0dda91758273eb8c8ac78
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94534906"
---
# <a name="server-parameters-in-azure-database-for-mysql"></a>Paramètres de serveur dans Azure Database pour MySQL

Cet article fournit des considérations et des instructions pour la configuration des paramètres de serveur dans Azure Database pour MySQL.

## <a name="what-are-server-parameters"></a>Que sont les paramètres du serveur ? 

Le moteur MySQL fournit de nombreux paramètres et variables de serveur différents qui peuvent être utilisés pour configurer et régler le comportement du moteur. Certains paramètres peuvent être définis dynamiquement pendant le runtime, tandis que d’autres sont « statiques », ce qui nécessite un redémarrage du serveur pour les appliquer.

Azure Database pour MySQL expose la possibilité de modifier la valeur des différents paramètres du serveur MySQL à l’aide du [Portail Azure](./howto-server-parameters.md), d’[Azure CLI](./howto-configure-server-parameters-using-cli.md)et de [PowerShell](./howto-configure-server-parameters-using-powershell.md) pour répondre aux besoins de votre charge de travail.

## <a name="configurable-server-parameters"></a>Paramètres de serveur configurables

La liste des paramètres de serveur pris en charge s’allonge en permanence. Utilisez l’onglet Paramètres du serveur du Portail Azure pour afficher la liste complète et configurer les valeurs des paramètres du serveur.

Reportez-vous aux sections suivantes pour en savoir plus sur les limites des différents paramètres de serveur couramment mis à jour. Les limites sont déterminées par le niveau tarifaire et vCores du serveur.

### <a name="thread-pools"></a>Pools de threads

MySQL affecte traditionnellement un thread pour chaque connexion client. À mesure que le nombre d’utilisateurs simultanés augmente, une certaine baisse des performances se fait sentir. De nombreux threads actifs peuvent avoir un impact significatif sur les performances en raison de l’augmentation du basculement de contexte, de la contention de thread et de la localité incorrecte pour les caches de processeur.

Avec les pools de threads, qui sont une fonctionnalité du serveur distincte du regroupement de connexions, optimisez les performances en introduisant un pool dynamique de threads de travail qui peut être utilisé pour limiter le nombre de threads actifs en cours d’exécution sur le serveur et réduire l’activité des threads. Cela permet de s’assurer qu’une rafale de connexions n’entraîne pas l’insuffisance des ressources du serveur ou un blocage en raison d’une erreur de mémoire insuffisante. Les pools de threads sont plus efficaces pour les requêtes courtes et les charges de travail du processeur gourmandes en ressources, par exemple les charges de travail OLTP.

Pour en savoir plus sur les pools de threads, consultez [Présentation des pools de threads dans Azure Database pour MySQL](https://techcommunity.microsoft.com/t5/azure-database-for-mysql/introducing-thread-pools-in-azure-database-for-mysql-service/ba-p/1504173)

> [!NOTE]
> La fonctionnalité de pool de threads n’est pas prise en charge pour la version 5.6 de MySQL. 

### <a name="configuring-the-thread-pool"></a>Configuration du pool de threads
Pour activer le pool de threads, mettez à jour le paramètre de serveur `thread_handling` sur « pool de threads ». Par défaut, ce paramètre est défini sur `one-thread-per-connection`, ce qui signifie que MySQL crée un nouveau thread pour chaque nouvelle connexion. Notez qu’il s’agit d’un paramètre statique qui requiert un redémarrage du serveur pour s’appliquer.

Vous pouvez également configurer les nombres maximal et minimal de threads dans le pool en définissant les paramètres de serveur suivants : 
- `thread_pool_max_threads`: Cette valeur garantit que le pool ne contiendra pas plus de ce nombre de threads.
- `thread_pool_min_threads`: Cette valeur définit le nombre de threads qui seront réservés même après la fermeture des connexions.

Pour améliorer les performances des requêtes courtes sur le pool de threads, Azure Database pour MySQL vous permet d’activer l’exécution par lots. au lieu de revenir au pool de threads immédiatement après l’exécution d’une requête, les threads restent actifs pendant un bref laps de temps pour attendre la requête suivante via cette connexion. Le thread exécute ensuite la requête rapidement et une fois terminé, attend la prochaine, jusqu’à ce que la consommation de temps globale de ce processus dépasse un seuil. Le comportement d’exécution du lot est déterminé à l’aide des paramètres de serveur suivants :  

-  `thread_pool_batch_wait_timeout`: Cette valeur spécifie la durée pendant laquelle un thread attend qu’une autre requête soit en cours de traitement.
- `thread_pool_batch_max_time`: Cette valeur détermine la durée maximale pendant laquelle un thread répète le cycle d’exécution de la requête et attend la requête suivante.

> [!IMPORTANT]
> Testez le pool de threads avant de l’activer en production. 

### <a name="log_bin_trust_function_creators"></a>log_bin_trust_function_creators

Dans Azure Database pour MySQL, les journaux binaires sont toujours activés (c.-à-d. que `log_bin` est défini sur ON). Si vous souhaitez utiliser des déclencheurs, vous obtiendrez une erreur similaire à *Vous n’avez pas le SUPER privilège et la journalisation binaire est activée (vous pouvez utiliser la variable `log_bin_trust_function_creators` moins sécurisée)* . 

Le format de journalisation binaire est toujours **ROW** et toutes les connexions au serveur utilisent **TOUJOURS** la journalisation binaire basée sur les lignes. Avec la journalisation binaire basée sur les lignes, les problèmes de sécurité n’existent pas et la journalisation binaire ne peut pas s’arrêter, ce qui vous permet de définir [`log_bin_trust_function_creators`](https://dev.mysql.com/doc/refman/5.7/en/replication-options-binary-log.html#sysvar_log_bin_trust_function_creators) sur **TRUE** de manière sécurisée.

### <a name="innodb_buffer_pool_size"></a>innodb_buffer_pool_size

Consultez la [documentation MySQL](https://dev.mysql.com/doc/refman/5.7/en/innodb-parameters.html#sysvar_innodb_buffer_pool_size) pour en savoir plus sur ce paramètre.

#### <a name="servers-supporting-up-to-4-tb-storage"></a>Serveurs prenant en charge jusqu’à 4 To de stockage

|**Niveau tarifaire**|**vCore(s)**|**Valeur par défaut (octets)**|**Valeur minimale (octets)**|**Valeur maximale (octets)**|
|---|---|---|---|---|
|De base|1|872415232|134217728|872415232|
|De base|2|2684354560|134217728|2684354560|
|Usage général|2|3758096384|134217728|3758096384|
|Usage général|4|8053063680|134217728|8053063680|
|Usage général|8|16106127360|134217728|16106127360|
|Usage général|16|32749125632|134217728|32749125632|
|Usage général|32|66035122176|134217728|66035122176|
|Usage général|64|132070244352|134217728|132070244352|
|Mémoire optimisée|2|7516192768|134217728|7516192768|
|Mémoire optimisée|4|16106127360|134217728|16106127360|
|Mémoire optimisée|8|32212254720|134217728|32212254720|
|Mémoire optimisée|16|65498251264|134217728|65498251264|
|Mémoire optimisée|32|132070244352|134217728|132070244352|

#### <a name="servers-support-up-to-16-tb-storage"></a>Serveurs prenant en charge jusqu’à 16 To de stockage

|**Niveau tarifaire**|**vCore(s)**|**Valeur par défaut (octets)**|**Valeur minimale (octets)**|**Valeur maximale (octets)**|
|---|---|---|---|---|
|De base|1|872415232|134217728|872415232|
|De base|2|2684354560|134217728|2684354560|
|Usage général|2|7516192768|134217728|7516192768|
|Usage général|4|16106127360|134217728|16106127360|
|Usage général|8|32212254720|134217728|32212254720|
|Usage général|16|65498251264|134217728|65498251264|
|Usage général|32|132070244352|134217728|132070244352|
|Usage général|64|264140488704|134217728|264140488704|
|Mémoire optimisée|2|15032385536|134217728|15032385536|
|Mémoire optimisée|4|32212254720|134217728|32212254720|
|Mémoire optimisée|8|64424509440|134217728|64424509440|
|Mémoire optimisée|16|130996502528|134217728|130996502528|
|Mémoire optimisée|32|264140488704|134217728|264140488704|

### <a name="innodb_file_per_table"></a>innodb_file_per_table

> [!NOTE]
> `innodb_file_per_table` peut seulement être mise à jour aux niveaux tarifaires Usage général et Mémoire optimisée.

MySQL stocke la table InnoDB dans différents espaces de stockage en fonction de la configuration que vous avez fournie lors de la création de la table. L’[espace disque logique du système](https://dev.mysql.com/doc/refman/5.7/en/innodb-system-tablespace.html) est la zone de stockage pour le dictionnaire de données InnoDB. Un [espace disque logique de fichier par table](https://dev.mysql.com/doc/refman/5.7/en/innodb-file-per-table-tablespaces.html) contient des données et des index pour une table InnoDB unique, et est stocké dans son propre fichier de données au sein du système de fichiers. Ce comportement est contrôlé par le paramètre de serveur `innodb_file_per_table`. La définition de `innodb_file_per_table` sur `OFF` amène InnoDB à créer des tables dans l’espace disque logique du système. Autrement, InnoDB crée des tables dans des espaces disques logiques de fichier par table.

Azure Database pour MySQL prend en charge jusqu’à **4 To**, dans un fichier de données unique. Si la taille de votre base de données est supérieure à 4 To, vous devez créer la table dans l’espace disque logique [innodb_file_per_table](https://dev.mysql.com/doc/refman/5.7/en/innodb-parameters.html#sysvar_innodb_file_per_table). Si vous avez une table d’une taille supérieure à 4 To, vous devez utiliser la table de partition.

### <a name="join_buffer_size"></a>join_buffer_size

Consultez la [documentation MySQL](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_join_buffer_size) pour en savoir plus sur ce paramètre.

|**Niveau tarifaire**|**vCore(s)**|**Valeur par défaut (octets)**|**Valeur minimale (octets)**|**Valeur maximale (octets)**|
|---|---|---|---|---|
|De base|1|Non configurable dans le niveau de base|N/A|N/A|
|De base|2|Non configurable dans le niveau de base|N/A|N/A|
|Usage général|2|262 144|128|268435455|
|Usage général|4|262 144|128|536870912|
|Usage général|8|262 144|128|1073741824|
|Usage général|16|262 144|128|2147483648|
|Usage général|32|262 144|128|4294967295|
|Usage général|64|262 144|128|4294967295|
|Mémoire optimisée|2|262 144|128|536870912|
|Mémoire optimisée|4|262 144|128|1073741824|
|Mémoire optimisée|8|262 144|128|2147483648|
|Mémoire optimisée|16|262 144|128|4294967295|
|Mémoire optimisée|32|262 144|128|4294967295|

### <a name="max_connections"></a>max_connections

|**Niveau tarifaire**|**vCore(s)**|**Valeur par défaut**|**Valeur minimale**|**Valeur maximale**|
|---|---|---|---|---|
|De base|1|50|10|50|
|De base|2|100|10|100|
|Usage général|2|300|10|600|
|Usage général|4|625|10|1250|
|Usage général|8|1250|10|2 500|
|Usage général|16|2 500|10|5 000|
|Usage général|32|5 000|10|10000|
|Usage général|64|10000|10|20000|
|Mémoire optimisée|2|625|10|1250|
|Mémoire optimisée|4|1250|10|2 500|
|Mémoire optimisée|8|2 500|10|5 000|
|Mémoire optimisée|16|5 000|10|10000|
|Mémoire optimisée|32|10000|10|20000|

Lorsque la limite du nombre de connexions est dépassée, vous pouvez recevoir l’erreur suivante :
> ERREUR 1040 (08004) : Trop de connexions

> [!IMPORTANT]
> Pour une expérience optimale, nous vous recommandons d’utiliser un regroupement de connexions comme ProxySQL pour gérer efficacement les connexions.

La création de connexions clientes à MySQL prend du temps et, une fois établies, ces connexions occupent des ressources de base de données, même lorsqu’elles sont inactives. La plupart des applications requièrent de nombreuses connexions à courte durée, ce qui aggrave la situation. Par conséquent, il y a moins de ressources disponibles pour votre charge de travail réelle; ce qui entraîne une diminution des performances. Un regroupement de connexions qui réduit les connexions inactives et réutilise les connexions existantes permet d’éviter cela. Pour en savoir plus sur la configuration de ProxySQL, consultez notre [billet de blog](https://techcommunity.microsoft.com/t5/azure-database-for-mysql/load-balance-read-replicas-using-proxysql-in-azure-database-for/ba-p/880042).

>[!Note]
>ProxySQL est un outil communautaire en open source. Il est pris en charge par Microsoft dans la mesure du possible. Afin d’obtenir un support de production avec une aide faisant autorité, vous pouvez évaluer et contacter le [Support produit de ProxySQL](https://proxysql.com/services/support/).

### <a name="max_heap_table_size"></a>max_heap_table_size

Consultez la [documentation MySQL](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_max_heap_table_size) pour en savoir plus sur ce paramètre.

|**Niveau tarifaire**|**vCore(s)**|**Valeur par défaut (octets)**|**Valeur minimale (octets)**|**Valeur maximale (octets)**|
|---|---|---|---|---|
|De base|1|Non configurable dans le niveau de base|N/A|N/A|
|De base|2|Non configurable dans le niveau de base|N/A|N/A|
|Usage général|2|16 777 216|16384|268435455|
|Usage général|4|16 777 216|16384|536870912|
|Usage général|8|16 777 216|16384|1073741824|
|Usage général|16|16 777 216|16384|2147483648|
|Usage général|32|16 777 216|16384|4294967295|
|Usage général|64|16 777 216|16384|4294967295|
|Mémoire optimisée|2|16 777 216|16384|536870912|
|Mémoire optimisée|4|16 777 216|16384|1073741824|
|Mémoire optimisée|8|16 777 216|16384|2147483648|
|Mémoire optimisée|16|16 777 216|16384|4294967295|
|Mémoire optimisée|32|16 777 216|16384|4294967295|

### <a name="query_cache_size"></a>query_cache_size

Le cache des requêtes est désactivé par défaut. Pour activer le cache des requêtes, configurez le paramètre `query_cache_type`. 

Consultez la [documentation MySQL](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_query_cache_size) pour en savoir plus sur ce paramètre.

> [!NOTE]
> Le cache des requêtes est déconseillé à partir de MySQL 5.7.20 et a été supprimé de MySQL 8.0

|**Niveau tarifaire**|**vCore(s)**|**Valeur par défaut (octets)**|**Valeur minimale (octets)**|**Valeur max.**|
|---|---|---|---|---|
|De base|1|Non configurable dans le niveau de base|N/A|N/A|
|De base|2|Non configurable dans le niveau de base|N/A|N/A|
|Usage général|2|0|0|16 777 216|
|Usage général|4|0|0|33554432|
|Usage général|8|0|0|67108864|
|Usage général|16|0|0|134217728|
|Usage général|32|0|0|134217728|
|Usage général|64|0|0|134217728|
|Mémoire optimisée|2|0|0|33554432|
|Mémoire optimisée|4|0|0|67108864|
|Mémoire optimisée|8|0|0|134217728|
|Mémoire optimisée|16|0|0|134217728|
|Mémoire optimisée|32|0|0|134217728|

### <a name="lower_case_table_names"></a>lower_case_table_names

La valeur lower_case_table_name est définie par défaut sur 1, et vous pouvez modifier ce paramètre dans MySQL 5.6 et MySQL 5.7

Consultez la [documentation MySQL](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_lower_case_table_names) pour en savoir plus sur ce paramètre.

> [!NOTE]
> Dans MySQL 8.0, la valeur lower_case_table_name est définie sur 1 par défaut et vous ne pouvez pas la modifier.

### <a name="innodb_strict_mode"></a>innodb_strict_mode

Si vous recevez une erreur semblable à « Taille de ligne trop grande (> 8126) », vous pouvez désactiver le paramètre **innodb_strict_mode**. Le paramètre de serveur **innodb_strict_mode** n’est pas autorisé à être modifié globalement au niveau du serveur, car si la taille des données de ligne est supérieure à 8 ko, les données seront tronquées sans erreur entraînant une perte de données potentielle. Nous vous recommandons de modifier le schéma pour qu’il corresponde à la limite de taille de page. 

Ce paramètre peut être défini au niveau de la session à l’aide de `init_connect`. Pour définir **innodb_strict_mode** au niveau de la session, reportez-vous à [Définition des paramètres non listés](./howto-server-parameters.md#setting-parameters-not-listed).

> [!NOTE]
> Si vous avez un serveur de réplica en lecture, la désactivation de **innodb_strict_mode** au niveau de la session sur un serveur source interrompt la réplication. Nous vous suggérons de conserver le paramètre activé si vous avez des réplicas en lecture.

### <a name="sort_buffer_size"></a>sort_buffer_size

Consultez la [documentation MySQL](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_sort_buffer_size) pour en savoir plus sur ce paramètre.

|**Niveau tarifaire**|**vCore(s)**|**Valeur par défaut (octets)**|**Valeur minimale (octets)**|**Valeur maximale (octets)**|
|---|---|---|---|---|
|De base|1|Non configurable dans le niveau de base|N/A|N/A|
|De base|2|Non configurable dans le niveau de base|N/A|N/A|
|Usage général|2|524 288|32 768|4 194 304|
|Usage général|4|524 288|32 768|8388608|
|Usage général|8|524 288|32 768|16 777 216|
|Usage général|16|524 288|32 768|33554432|
|Usage général|32|524 288|32 768|33554432|
|Usage général|64|524 288|32 768|33554432|
|Mémoire optimisée|2|524 288|32 768|8388608|
|Mémoire optimisée|4|524 288|32 768|16 777 216|
|Mémoire optimisée|8|524 288|32 768|33554432|
|Mémoire optimisée|16|524 288|32 768|33554432|
|Mémoire optimisée|32|524 288|32 768|33554432|

### <a name="tmp_table_size"></a>tmp_table_size

Consultez la [documentation MySQL](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_tmp_table_size) pour en savoir plus sur ce paramètre.

|**Niveau tarifaire**|**vCore(s)**|**Valeur par défaut (octets)**|**Valeur minimale (octets)**|**Valeur maximale (octets)**|
|---|---|---|---|---|
|De base|1|Non configurable dans le niveau de base|N/A|N/A|
|De base|2|Non configurable dans le niveau de base|N/A|N/A|
|Usage général|2|16 777 216|1 024|67108864|
|Usage général|4|16 777 216|1 024|134217728|
|Usage général|8|16 777 216|1 024|268435456|
|Usage général|16|16 777 216|1 024|536870912|
|Usage général|32|16 777 216|1 024|1073741824|
|Usage général|64|16 777 216|1 024|1073741824|
|Mémoire optimisée|2|16 777 216|1 024|134217728|
|Mémoire optimisée|4|16 777 216|1 024|268435456|
|Mémoire optimisée|8|16 777 216|1 024|536870912|
|Mémoire optimisée|16|16 777 216|1 024|1073741824|
|Mémoire optimisée|32|16 777 216|1 024|1073741824|

### <a name="time_zone"></a>time_zone

Lors du déploiement initial, un serveur Azure pour MySQL contient des tableaux système pour les informations de fuseau horaire, mais ces tableaux ne sont pas remplis. Les tables de fuseaux horaires peuvent être remplies en appelant la procédure stockée `mysql.az_load_timezone` à partir d’un outil tel que la ligne de commande MySQL ou MySQL Workbench. Pour savoir comment appeler la procédure stockée et définir les fuseaux horaires au niveau global ou au niveau de la session, consultez les articles relatifs au [Portail Azure](howto-server-parameters.md#working-with-the-time-zone-parameter) ou à [Azure CLI](howto-configure-server-parameters-using-cli.md#working-with-the-time-zone-parameter).

## <a name="non-configurable-server-parameters"></a>Paramètres de serveur non configurables

Les paramètres de serveur ci-dessous ne sont pas configurables dans le service :

|**Paramètre**|**Valeur fixe**|
| :------------------------ | :-------- |
|innodb_file_per_table dans le niveau de base|OFF|
|innodb_flush_log_at_trx_commit|1|
|sync_binlog|1|
|innodb_log_file_size|256 Mo|
|innodb_log_files_in_group|2|

Les autres variables non répertoriées ici sont définies sur les valeurs par défaut de MySQL. Pour obtenir les valeurs par défaut, reportez-vous à la documentation MySQL pour les versions [8.0](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html), [5.7](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html) et [5.6](https://dev.mysql.com/doc/refman/5.6/en/server-system-variables.html). 

## <a name="next-steps"></a>Étapes suivantes

- Guide pratique pour [configurer des paramètres serveur avec le Portail Azure](./howto-server-parameters.md)
- Guide pratique pour [configurer des paramètres serveur avec Azure CLI](./howto-configure-server-parameters-using-cli.md)
- Guide pratique pour [configurer des paramètres serveur avec PowerShell](./howto-configure-server-parameters-using-powershell.md)

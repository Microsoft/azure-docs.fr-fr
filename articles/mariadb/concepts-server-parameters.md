---
title: 'Paramètres d’un serveur : Azure Database pour MariaDB'
description: Cette rubrique fournit des lignes directrices pour la configuration des paramètres de serveur dans Azure Database pour MariaDB.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.topic: conceptual
ms.date: 6/25/2020
ms.openlocfilehash: 4432178d5908d4360cda05a62b62d05687be4235
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94541128"
---
# <a name="server-parameters-in-azure-database-for-mariadb"></a>Paramètres de serveur dans Azure Database pour MariaDB

Cet article fournit des considérations et des lignes directrices pour la configuration des paramètres de serveur dans Azure Database pour MariaDB.

## <a name="what-are-server-parameters"></a>Que sont les paramètres du serveur ? 

Le moteur MariaDB fournit de nombreux paramètres et variables de serveur différents qui peuvent être utilisés pour configurer et régler le comportement du moteur. Certains paramètres peuvent être définis dynamiquement pendant le runtime, tandis que d’autres sont « statiques », ce qui nécessite un redémarrage du serveur pour les appliquer.

Azure Database pour MariaDB expose la possibilité de modifier la valeur des différents paramètres du serveur MariaDB à l’aide du [Portail Azure](./howto-server-parameters.md), d’[Azure CLI](./howto-configure-server-parameters-cli.md) et de [PowerShell](./howto-configure-server-parameters-using-powershell.md) pour répondre aux besoins de votre charge de travail.

## <a name="configurable-server-parameters"></a>Paramètres de serveur configurables

La liste des paramètres de serveur pris en charge s’allonge en permanence. Utilisez l’onglet Paramètres du serveur du Portail Azure pour afficher la liste complète et configurer les valeurs des paramètres du serveur.

Reportez-vous aux sections suivantes pour en savoir plus sur les limites des différents paramètres de serveur couramment mis à jour. Les limites sont déterminées par le niveau tarifaire et vCores du serveur.

### <a name="log_bin_trust_function_creators"></a>log_bin_trust_function_creators

Dans Azure Database for MariaDB, les journaux binaires sont toujours activés (c.-à-d. que `log_bin` est défini sur ON). Si vous souhaitez utiliser des déclencheurs, vous obtenez une erreur similaire à *Vous n’avez pas le SUPER privilège et la journalisation binaire est activée (vous pouvez utiliser la variable `log_bin_trust_function_creators` moins sécurisée)* .

Le format de journalisation binaire est toujours **ROW** et toutes les connexions au serveur utilisent **TOUJOURS** la journalisation binaire basée sur les lignes. Avec la journalisation binaire basée sur les lignes, les problèmes de sécurité n’existent pas et la journalisation binaire ne peut pas s’arrêter, ce qui vous permet de définir [`log_bin_trust_function_creators`](https://mariadb.com/docs/reference/mdb/system-variables/log_bin_trust_function_creators/) sur **TRUE** de manière sécurisée.

### <a name="innodb_buffer_pool_size"></a>innodb_buffer_pool_size

Consultez la [documentation MariaDB](https://mariadb.com/kb/en/innodb-system-variables/#innodb_buffer_pool_size) pour en savoir plus sur ce paramètre.

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

MariaDB stocke la table InnoDB dans différents espaces disque logique en fonction de la configuration que vous avez fournie pendant la création de la table. L’[espace disque logique du système](https://mariadb.com/kb/en/innodb-system-tablespaces/) est la zone de stockage pour le dictionnaire de données InnoDB. Un [espace disque logique de fichier par table](https://mariadb.com/kb/en/innodb-file-per-table-tablespaces/) contient des données et des index pour une table InnoDB unique, et est stocké dans son propre fichier de données au sein du système de fichiers. Ce comportement est contrôlé par le paramètre de serveur `innodb_file_per_table`. La définition de `innodb_file_per_table` sur `OFF` amène InnoDB à créer des tables dans l’espace disque logique du système. Autrement, InnoDB crée des tables dans des espaces disques logiques de fichier par table.

Azure Database for MariaDB prend en charge jusqu’à **1 To** dans un même fichier de données. Si la taille de votre base de données est supérieure à 1 To, vous devez créer la table dans l’espace disque logique [innodb_file_per_table](https://mariadb.com/kb/en/innodb-system-variables/#innodb_file_per_table). Si vous avez une table d’une taille supérieure à 1 To, vous devez utiliser la table de partition.

### <a name="join_buffer_size"></a>join_buffer_size

Consultez la [documentation MariaDB](https://mariadb.com/kb/en/server-system-variables/#join_buffer_size) pour en savoir plus sur ce paramètre.

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

La création de connexions clientes à MariaDB prend du temps et, une fois établies, ces connexions occupent des ressources de base de données, même lorsqu’elles sont inactives. La plupart des applications requièrent de nombreuses connexions à courte durée, ce qui aggrave la situation. Par conséquent, il y a moins de ressources disponibles pour votre charge de travail réelle; ce qui entraîne une diminution des performances. Un regroupement de connexions qui réduit les connexions inactives et réutilise les connexions existantes permet d’éviter cela. Pour en savoir plus sur la configuration de ProxySQL, consultez notre [billet de blog](https://techcommunity.microsoft.com/t5/azure-database-for-mysql/load-balance-read-replicas-using-proxysql-in-azure-database-for/ba-p/880042).

>[!Note]
>ProxySQL est un outil communautaire en open source. Il est pris en charge par Microsoft dans la mesure du possible. Afin d’obtenir un support de production avec une aide faisant autorité, vous pouvez évaluer et contacter le [Support produit de ProxySQL](https://proxysql.com/services/support/).

### <a name="max_heap_table_size"></a>max_heap_table_size

Consultez la [documentation MariaDB](https://mariadb.com/kb/en/server-system-variables/#max_heap_table_size) pour en savoir plus sur ce paramètre.

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

Le cache des reqêtes est activé par défaut dans MariaDB avec le paramètre `have_query_cache`. 

Consultez la [documentation MariaDB](https://mariadb.com/kb/en/server-system-variables/#query_cache_size) pour en savoir plus sur ce paramètre.

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

### <a name="sort_buffer_size"></a>sort_buffer_size

Consultez la [documentation MariaDB](https://mariadb.com/kb/en/server-system-variables/#sort_buffer_size) pour en savoir plus sur ce paramètre.

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

Consultez la [documentation MariaDB](https://mariadb.com/kb/en/server-system-variables/#tmp_table_size) pour en savoir plus sur ce paramètre.

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

Lors du déploiement initial, un serveur Azure pour MariaDB contient des tables système pour les informations de fuseau horaire, mais ces tables ne sont pas remplies. Les tables de fuseaux horaires peuvent être remplies en appelant la procédure stockée `mysql.az_load_timezone` à partir d’un outil tel que la ligne de commande MySQL ou MySQL Workbench. Pour savoir comment appeler la procédure stockée et définir les fuseaux horaires au niveau global ou au niveau de la session, consultez les articles relatifs au [Portail Azure](howto-server-parameters.md#working-with-the-time-zone-parameter) ou à [Azure CLI](howto-configure-server-parameters-cli.md#working-with-the-time-zone-parameter).

## <a name="non-configurable-server-parameters"></a>Paramètres de serveur non configurables

Les paramètres de serveur ci-dessous ne sont pas configurables dans le service :

|**Paramètre**|**Valeur fixe**|
| :------------------------ | :-------- |
|innodb_file_per_table dans le niveau de base|OFF|
|innodb_flush_log_at_trx_commit|1|
|sync_binlog|1|
|innodb_log_file_size|256 Mo|
|innodb_log_files_in_group|2|

Tous les autres paramètres de serveur sont définis sur leurs valeurs MariaDB fournies par défaut pour [MariaDB](https://mariadb.com/kb/en/server-system-variables/).

## <a name="next-steps"></a>Étapes suivantes

- Guide pratique pour [configurer des paramètres serveur avec le Portail Azure](./howto-server-parameters.md)
- Guide pratique pour [configurer des paramètres serveur avec Azure CLI](./howto-configure-server-parameters-cli.md)
- Guide pratique pour [configurer des paramètres serveur avec PowerShell](./howto-configure-server-parameters-using-powershell.md)
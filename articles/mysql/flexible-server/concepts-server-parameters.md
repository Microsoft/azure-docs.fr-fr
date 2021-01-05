---
title: Paramètres du serveur – Azure Database pour MySQL – Serveur flexible
description: Cette rubrique fournit des instructions pour la configuration des paramètres de serveur dans Azure Database pour MySQL – Serveur flexible.
author: ambhatna
ms.author: ambhatna
ms.service: mysql
ms.topic: conceptual
ms.date: 11/10/2020
ms.openlocfilehash: 58978f120578afeca129b0d8928713835def8418
ms.sourcegitcommit: b4880683d23f5c91e9901eac22ea31f50a0f116f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94496183"
---
# <a name="server-parameters-in-azure-database-for-mysql---flexible-server"></a>Paramètres de serveur dans Azure Database pour MySQL – Serveur flexible

> [!IMPORTANT]
> Azure Database pour MySQL - Serveur flexible est actuellement en préversion publique.

Cet article fournit des considérations et des instructions pour la configuration des paramètres de serveur dans Azure Database pour MySQL – Serveur flexible.

## <a name="what-are-server-variables"></a>Que sont les variables de serveur ? 

Le moteur MySQL fournit de nombreux [paramètres et variables de serveur](https://dev.mysql.com/doc/refman/5.7/en/server-option-variable-reference.html) différents qui peuvent être utilisés pour configurer et régler le comportement du moteur. Certains paramètres peuvent être définis dynamiquement pendant le runtime, tandis que d’autres sont « statiques », ce qui nécessite un redémarrage du serveur pour les appliquer.

Azure Database pour MySQL – Serveur flexible expose la possibilité de modifier la valeur des différents paramètres du serveur MySQL à l’aide du [portail Azure](./how-to-configure-server-parameters-portal.md) et d’[Azure CLI](./how-to-configure-server-parameters-cli.md) pour répondre aux besoins de votre charge de travail.

## <a name="configurable-server-parameters"></a>Paramètres de serveur configurables

Vous pouvez gérer la configuration de serveur flexible Azure Database pour MySQL à l’aide des paramètres de serveur. Les paramètres de serveur sont configurés avec la valeur par défaut et la valeur recommandée lors de la création du serveur. Le panneau des paramètres de serveur sur le portail Azure affiche à la fois les paramètres de serveur modifiables et non modifiables. Les paramètres de serveur non modifiables sont grisés.

La liste des paramètres de serveur pris en charge s’allonge en permanence. Utilisez l’onglet Paramètres du serveur du Portail Azure pour afficher la liste complète et configurer les valeurs des paramètres du serveur.

Reportez-vous aux sections suivantes pour en savoir plus sur les limites des différents paramètres de serveur couramment mis à jour. Les limites sont déterminées par le niveau de calcul et la taille (vCores) du serveur.

> [!NOTE]
> Si vous envisagez de modifier un paramètre de serveur qui n’est pas modifiable, mais que vous aimeriez voir comme modifiable pour votre environnement, ouvrez un élément [UserVoice](https://feedback.azure.com/forums/597982-azure-database-for-mysql) ou votez si les commentaires existent déjà. Cela peut nous aider à établir des priorités.

### <a name="log_bin_trust_function_creators"></a>log_bin_trust_function_creators

Dans Azure Database pour MySQL – Serveur flexible, les journaux binaires sont toujours activés (autrement dit,`log_bin` est défini sur ON). Si vous souhaitez utiliser des déclencheurs, vous obtiendrez une erreur similaire à *Vous n’avez pas le SUPER privilège et la journalisation binaire est activée (vous pouvez utiliser la variable `log_bin_trust_function_creators` moins sécurisée)* . 

Le format de journalisation binaire est toujours **ROW** et toutes les connexions au serveur utilisent **TOUJOURS** la journalisation binaire basée sur les lignes. Avec la journalisation binaire basée sur les lignes, les problèmes de sécurité n’existent pas et la journalisation binaire ne peut pas s’arrêter, ce qui vous permet de définir [`log_bin_trust_function_creators`](https://dev.mysql.com/doc/refman/5.7/en/replication-options-binary-log.html#sysvar_log_bin_trust_function_creators) sur **TRUE** de manière sécurisée.

### <a name="innodb_buffer_pool_size"></a>innodb_buffer_pool_size

Consultez la [documentation MySQL](https://dev.mysql.com/doc/refman/5.7/en/innodb-parameters.html#sysvar_innodb_buffer_pool_size) pour en savoir plus sur ce paramètre.

|**Niveau tarifaire**|**vCore(s)**|**Taille de la mémoire (Gio)**|**Valeur par défaut (octets)**|**Valeur minimale (octets)**|**Valeur maximale (octets)**|
|---|---|---|---|---|---|
|Burstable (B1s)|1|1|134217728|33554432|134217728|
|Burstable (B1ms)|1|2|536870912|134217728|536870912|
|Expansible|2|4|2147483648|134217728|2147483648|
|Usage général|2|8|6442450944|134217728|6442450944|
|Usage général|4|16|12884901888|134217728|12884901888|
|Usage général|8|32|25769803776|134217728|25769803776|
|Usage général|16|64|51539607552|134217728|51539607552|
|Usage général|32|128|103079215104|134217728|103079215104|
|Usage général|48|192|154618822656|134217728|154618822656|
|Usage général|64|256|206158430208|134217728|206158430208|
|Mémoire optimisée|2|16|12884901888|134217728|12884901888|
|Mémoire optimisée|4|32|25769803776|134217728|25769803776|
|Mémoire optimisée|8|64|51539607552|134217728|51539607552|
|Mémoire optimisée|16|128|103079215104|134217728|103079215104|
|Mémoire optimisée|32|256|206158430208|134217728|206158430208|
|Mémoire optimisée|48|384|309237645312|134217728|309237645312|
|Mémoire optimisée|64|504|405874409472|134217728|405874409472|

### <a name="innodb_file_per_table"></a>innodb_file_per_table

MySQL stocke la table InnoDB dans différents espaces de stockage en fonction de la configuration que vous avez fournie lors de la création de la table. L’[espace disque logique du système](https://dev.mysql.com/doc/refman/5.7/en/innodb-system-tablespace.html) est la zone de stockage pour le dictionnaire de données InnoDB. Un [espace disque logique de fichier par table](https://dev.mysql.com/doc/refman/5.7/en/innodb-file-per-table-tablespaces.html) contient des données et des index pour une table InnoDB unique, et est stocké dans son propre fichier de données au sein du système de fichiers. Ce comportement est contrôlé par le paramètre de serveur `innodb_file_per_table`. La définition de `innodb_file_per_table` sur `OFF` amène InnoDB à créer des tables dans l’espace disque logique du système. Autrement, InnoDB crée des tables dans des espaces disques logiques de fichier par table.

Azure Database pour MySQL – Serveur flexible prend en charge jusqu’à **4  o** dans un seul fichier de données. Si la taille de votre base de données est supérieure à 4 To, vous devez créer la table dans l’espace disque logique [innodb_file_per_table](https://dev.mysql.com/doc/refman/5.7/en/innodb-parameters.html#sysvar_innodb_file_per_table). Si vous avez une table d’une taille supérieure à 4 To, vous devez utiliser la table de partition.

### <a name="max_connections"></a>max_connections

La valeur de max_connection est déterminée par la taille de la mémoire du serveur. 

|**Niveau tarifaire**|**vCore(s)**|**Taille de la mémoire (Gio)**|**Valeur par défaut**|**Valeur minimale**|**Valeur maximale**|
|---|---|---|---|---|---|
|Burstable (B1s)|1|1|85 %|10|171|
|Burstable (B1ms)|1|2|171|10|341|
|Expansible|2|4|341|10|683|
|Usage général|2|8|683|10|1365|
|Usage général|4|16|1365|10|2731|
|Usage général|8|32|2731|10|5461|
|Usage général|16|64|5461|10|10923|
|Usage général|32|128|10923|10|21845|
|Usage général|48|192|16384|10|32 768|
|Usage général|64|256|21845|10|43691|
|Mémoire optimisée|2|16|1365|10|2731|
|Mémoire optimisée|4|32|2731|10|5461|
|Mémoire optimisée|8|64|5461|10|10923|
|Mémoire optimisée|16|128|10923|10|21845|
|Mémoire optimisée|32|256|21845|10|43691|
|Mémoire optimisée|48|384|32 768|10|65536|
|Mémoire optimisée|64|504|43008|10|86016|

Lorsque la limite du nombre de connexions est dépassée, vous pouvez recevoir l’erreur suivante :
> ERREUR 1040 (08004) : Trop de connexions

> [!IMPORTANT]
> Pour une expérience optimale, nous vous recommandons d’utiliser un regroupement de connexions comme ProxySQL pour gérer efficacement les connexions.

La création de connexions clientes à MySQL prend du temps et, une fois établies, ces connexions occupent des ressources de base de données, même lorsqu’elles sont inactives. La plupart des applications requièrent de nombreuses connexions à courte durée, ce qui aggrave la situation. Par conséquent, il y a moins de ressources disponibles pour votre charge de travail réelle; ce qui entraîne une diminution des performances. Un regroupement de connexions qui réduit les connexions inactives et réutilise les connexions existantes permet d’éviter cela. Pour en savoir plus sur la configuration de ProxySQL, consultez notre [billet de blog](https://techcommunity.microsoft.com/t5/azure-database-for-mysql/load-balance-read-replicas-using-proxysql-in-azure-database-for/ba-p/880042).

>[!Note]
>ProxySQL est un outil communautaire en open source. Il est pris en charge par Microsoft dans la mesure du possible. Afin d’obtenir un support de production avec une aide faisant autorité, vous pouvez évaluer et contacter le [Support produit de ProxySQL](https://proxysql.com/services/support/).

### <a name="innodb_strict_mode"></a>innodb_strict_mode

Si vous recevez une erreur semblable à « Taille de ligne trop grande (> 8126) », vous pouvez désactiver le paramètre **innodb_strict_mode**. Le paramètre de serveur **innodb_strict_mode** ne peut pas être modifié à l’échelle mondiale au niveau du serveur, car, si la taille des données de ligne est supérieure à 8 ko, les données seront tronquées sans erreur, ce qui peut entraîner une perte potentielle de données. Nous vous recommandons de modifier le schéma pour qu’il corresponde à la limite de taille de page. 

Ce paramètre peut être défini au niveau de la session à l’aide de `init_connect`. Pour définir **innodb_strict_mode** au niveau de la session, reportez-vous à [Définition des paramètres non listés](./how-to-configure-server-parameters-portal.md#setting-non-modifiable-server-parameters).

> [!NOTE]
> Si vous avez un serveur de réplica en lecture, la désactivation de **innodb_strict_mode** au niveau de la session sur un serveur source interrompt la réplication. Nous vous suggérons de conserver le paramètre activé si vous avez des réplicas en lecture.

### <a name="time_zone"></a>time_zone

Lors du déploiement initial, un serveur flexible Azure pour MySQL contient des tables système pour les informations relatives au fuseau horaire, mais ces tables ne sont pas renseignées. Les tables de fuseaux horaires peuvent être remplies en appelant la procédure stockée `mysql.az_load_timezone` à partir d’un outil tel que la ligne de commande MySQL ou MySQL Workbench. Pour savoir comment appeler la procédure stockée et définir les fuseaux horaires au niveau global ou au niveau de la session, consultez les articles relatifs au [Portail Azure](./how-to-configure-server-parameters-portal.md#working-with-the-time-zone-parameter) ou à [Azure CLI](./how-to-configure-server-parameters-cli.md#working-with-the-time-zone-parameter).

## <a name="non-modifiable-server-parameters"></a>Paramètres de serveur non modifiables

Le panneau des paramètres de serveur sur le portail Azure affiche à la fois les paramètres de serveur modifiables et non modifiables. Les paramètres de serveur non modifiables sont grisés. Si vous souhaitez configurer un paramètre de serveur non modifiable au niveau de la session, reportez-vous à l’article [Portail Azure](./how-to-configure-server-parameters-portal.md#setting-non-modifiable-server-parameters) ou [Azure CLI](./how-to-configure-server-parameters-cli.md#setting-non-modifiable-server-parameters) pour définir le paramètre au niveau de la connexion à l’aide de `init_connect`.

## <a name="next-steps"></a>Étapes suivantes

- Guide pratique pour configurer des [paramètres de serveur dans le portail Azure](./how-to-configure-server-parameters-portal.md)
- Guide pratique pour configurer des [paramètres de serveur dans Azure CLI](./how-to-configure-server-parameters-cli.md)

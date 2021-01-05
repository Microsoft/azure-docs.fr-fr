---
title: Limitations - Azure Database pour MySQL
description: Cet article décrit les limitations dans Azure Database pour MySQL, telles que le nombre de connexions et les options du moteur de stockage.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: conceptual
ms.date: 10/1/2020
ms.openlocfilehash: b4f828c675df9625d6d4889dbc31bbc4b9f887ed
ms.sourcegitcommit: ea17e3a6219f0f01330cf7610e54f033a394b459
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97386712"
---
# <a name="limitations-in-azure-database-for-mysql"></a>Limitations dans Azure Database pour MySQL
Les sections suivantes abordent la capacité, la prise en charge du moteur de stockage, la prise en charge des privilèges, la prise en charge des instructions de manipulation des données et les limites fonctionnelles du service de base de données. Vous pouvez aussi consulter les [limitations générales](https://dev.mysql.com/doc/mysql-reslimits-excerpt/5.6/en/limits.html) qui sont applicables au moteur de base de données MySQL.

## <a name="server-parameters"></a>Paramètres de serveur

> [!NOTE]
> Si vous recherchez des valeurs minimales/maximales pour des paramètres de serveur comme `max_connections` et `innodb_buffer_pool_size`, ces informations ont été déplacées vers l’article **[Paramètres du serveur](./concepts-server-parameters.md)** .

Azure Database pour MySQL prend en charge le réglage des valeurs des paramètres de serveur. Les valeurs minimale et maximale de certains paramètres (par exemple, `max_connections`, `join_buffer_size`, `query_cache_size`) sont déterminées par le niveau tarifaire et les vCores du serveur. Pour plus d’informations sur ces limites, consultez [Paramètres du serveur](./concepts-server-parameters.md).

Lors du déploiement initial, un serveur Azure pour MySQL contient des tableaux système pour les informations de fuseau horaire, mais ces tableaux ne sont pas remplis. Les tables de fuseaux horaires peuvent être remplies en appelant la procédure stockée `mysql.az_load_timezone` à partir d’un outil tel que la ligne de commande MySQL ou MySQL Workbench. Pour savoir comment appeler la procédure stockée et définir les fuseaux horaires au niveau global ou au niveau de la session, consultez les articles relatifs au [Portail Azure](howto-server-parameters.md#working-with-the-time-zone-parameter) ou à [Azure CLI](howto-configure-server-parameters-using-cli.md#working-with-the-time-zone-parameter).

Les plug-ins de mot de passe tels que « validate_password » et « caching_sha2_password » ne sont pas pris en charge par le service.

## <a name="storage-engines"></a>Moteurs de stockage

MySQL prend en charge de nombreux moteurs de stockage. Sur le Serveur flexible Azure Database pour MySQL, les moteurs de stockage suivants sont pris en charge ou ne le sont pas :

### <a name="supported"></a>Prise en charge
- [InnoDB](https://dev.mysql.com/doc/refman/5.7/en/innodb-introduction.html)
- [MEMORY](https://dev.mysql.com/doc/refman/5.7/en/memory-storage-engine.html)

### <a name="unsupported"></a>Non pris en charge
- [MyISAM](https://dev.mysql.com/doc/refman/5.7/en/myisam-storage-engine.html)
- [BLACKHOLE](https://dev.mysql.com/doc/refman/5.7/en/blackhole-storage-engine.html)
- [ARCHIVE](https://dev.mysql.com/doc/refman/5.7/en/archive-storage-engine.html)
- [FEDERATED](https://dev.mysql.com/doc/refman/5.7/en/federated-storage-engine.html)

## <a name="privileges--data-manipulation-support"></a>Prise en charge des privilèges et de la manipulation des données

De nombreux paramètres de serveur peuvent dégrader de façon inattendue les performances du serveur, ou nier les propriétés ACID du serveur MySQL. Afin de préserver l’intégrité du service et le contrat SLA au niveau du produit, ce service n’expose pas plusieurs rôles. 

Le service MySQL n’autorise pas l’accès direct au système de fichiers sous-jacent. Certaines commandes de manipulation de données ne sont pas prises en charge. 

### <a name="unsupported"></a>Non pris en charge

Les éléments suivants ne sont pas pris en charge :
- Rôle d’administrateur de base de données : Restreint. Sinon, vous pouvez utiliser le rôle d’utilisateur Administrateur (généré lors de la création d’un serveur) qui vous permet d’exécuter la plupart des instructions DDL et DML. 
- Privilège de superutilisateur : de la même façon, le [privilège de superutilisateur](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_super) est limité.
- DEFINER : requiert des privilèges de superutilisateur pour créer et est limité. Si vous importez des données à l'aide d'une sauvegarde, supprimez les commandes `CREATE DEFINER` manuellement ou à l'aide de la commande `--skip-definer` lors de l'exécution de mysqldump.
- Bases de données système : La [base de données système mysql](https://dev.mysql.com/doc/refman/5.7/en/system-schema.html) est en lecture seule ; elle est utilisée pour prendre en charge diverses fonctionnalités PaaS. Vous ne pouvez pas apporter de modifications à la base de données système `mysql`.
- `SELECT ... INTO OUTFILE`: Pas de prise en charge dans le service.
- `LOAD_FILE(file_name)`: Pas de prise en charge dans le service.

### <a name="supported"></a>Prise en charge
- `LOAD DATA INFILE` est prise en charge, mais le paramètre `[LOCAL]` doit être spécifié et dirigé vers un chemin d'accès UNC (stockage Azure monté via SMB).

## <a name="functional-limitations"></a>Limitations fonctionnelles

### <a name="scale-operations"></a>Opérations de mise à l’échelle
- La mise à l’échelle dynamique vers et depuis les niveaux tarifaires de base n’est pas prise en charge pour le moment.
- La diminution de la taille de stockage du serveur n’est pas prise en charge.

### <a name="server-version-upgrades"></a>Mises à niveau de la version du serveur
- La migration automatique entre les versions principales du moteur de base de données n’est pas prise en charge pour le moment. Si vous souhaitez mettre à niveau vers la version principale suivante, effectuez une [sauvegarde et une restauration](./concepts-migrate-dump-restore.md) vers un serveur créé avec la nouvelle version du moteur.

### <a name="point-in-time-restore"></a>Restauration dans le temps
- Lorsque vous utilisez la fonctionnalité PITR, le nouveau serveur est créé avec la même configuration que le serveur sur lequel il est basé.
- La restauration d’un serveur supprimé n’est pas prise en charge.

### <a name="vnet-service-endpoints"></a>Points de terminaison de service VNet
- Les points de terminaison de service de réseau virtuel sont uniquement pris en charge pour les serveurs Usage général et Mémoire optimisée.

### <a name="storage-size"></a>Taille de stockage
- Veuillez vous reporter aux [niveaux tarifaires](concepts-pricing-tiers.md) pour les limites de taille de stockage par niveau tarifaire.

## <a name="current-known-issues"></a>Problèmes connus
- L’instance de serveur MySQL affiche la mauvaise version de serveur une fois la connexion établie. Pour obtenir la version de moteur correcte de l’instance de serveur, utilisez la commande `select version();`.

## <a name="next-steps"></a>Étapes suivantes
- [Éléments disponibles dans chaque niveau de service](concepts-pricing-tiers.md)
- [Versions de base de données MySQL prises en charge](concepts-supported-versions.md)

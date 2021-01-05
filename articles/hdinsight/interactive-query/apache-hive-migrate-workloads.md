---
title: Migrer des charges de travail Azure HDInsight 3.6 Hive vers HDInsight 4.0
description: Découvrez comment migrer des charges de travail Apache Hive de HDInsight 3.6 vers HDInsight 4.0.
author: msft-tacox
ms.author: tacox
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.date: 11/13/2019
ms.openlocfilehash: bcc0faa8fdbd61ab3e3e0886256f7c796e5a98e2
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96011504"
---
# <a name="migrate-azure-hdinsight-36-hive-workloads-to-hdinsight-40"></a>Migrer des charges de travail Azure HDInsight 3.6 Hive vers HDInsight 4.0

Ce document explique comment migrer des charges de travail Apache Hive et LLAP de HDInsight 3.6 vers HDInsight 4.0. HDInsight 4.0 fournit des fonctionnalités plus récentes de Hive et LLAP, telles que les vues matérialisées et la mise en cache des résultats de requête. Lorsque vous migrez vos charges de travail vers HDInsight 4.0, vous pouvez utiliser les nombreuses nouvelles fonctionnalités de Hive 3 qui ne sont pas disponibles sur HDInsight 3.6.

Cet article aborde les sujets suivants :

* Migration des métadonnées Hive vers HDInsight 4.0
* Migration sécurisée des tables ACID et non ACID
* Préservation des stratégies de sécurité Hive entre les différentes versions de HDInsight
* Exécution des requêtes et débogage de HDInsight 3.6 vers HDInsight 4.0

L’un des avantages de Hive est la possibilité d’exporter des métadonnées vers une base de données externe (également appelée metastore Hive). Le **metastore Hive** est chargé de stocker les statistiques de table, y compris l’emplacement de stockage des tables, les noms de colonnes et les informations sur les index de table. HDInsight 3.6 et HDInsight 4.0 nécessitent différents schémas de metastore et ne peuvent pas partager un seul metastore. La méthode recommandée pour mettre à niveau le metastore Hive en toute sécurité consiste à mettre à niveau une copie plutôt que l’original dans l’environnement de production actuel. Ce document nécessite que les clusters d’origine et nouveau aient accès au même compte de stockage. Par conséquent, il ne couvre pas la migration de données vers une autre région.

## <a name="migrate-from-external-metastore"></a>Migrer à partir d’un metastore externe

### <a name="1-run-major-compaction-on-acid-tables-in-hdinsight-36"></a>1. Exécuter un compactage majeur sur des tables ACID dans HDInsight 3.6

Les tables ACID HDInsight 3.6 et HDInsight 4.0 comprennent les deltas ACID différemment. La seule action nécessaire avant la migration consiste à exécuter un compactage « MAJEUR » de chaque table ACID sur le cluster 3.6. Pour plus d’informations sur le compactage, consultez le [Manuel d’utilisation du langage Hive](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-AlterTable/Partition/Compact).

### <a name="2-copy-sql-database"></a>2. Copier une base de données SQL
Créez une nouvelle copie du metastore externe. Si vous utilisez un metastore externe, l’un des moyens les plus sûrs et les plus simples d’effectuer une copie du metastore consiste à [restaurer la base de données](../../azure-sql/database/recovery-using-backups.md#point-in-time-restore) avec un nom différent à l’aide de la fonction `RESTORE`.  Consultez [Utiliser des magasins de métadonnées externes dans Azure HDInsight](../hdinsight-use-external-metadata-stores.md) pour en savoir plus sur l’association d’un metastore externe à un cluster HDInsight.

### <a name="3-upgrade-metastore-schema"></a>3. Mettre à niveau le schéma du metastore
Une fois que la **copie** du metastore est terminée, exécutez un script de mise à niveau de schéma dans [Action de script](../hdinsight-hadoop-customize-cluster-linux.md) sur le cluster HDInsight 3.6 existant pour mettre à niveau le nouveau metastore vers le schéma Hive 3. Cette étape ne nécessite pas la connexion du nouveau metastore à un cluster. Cela permet à la base de données d’être attachée en tant que metastore HDInsight 4.0.

Utilisez les valeurs du tableau plus loin. Remplacez `SQLSERVERNAME DATABASENAME USERNAME PASSWORD` par les valeurs appropriées pour la **copie** du metastore Hive séparées par des espaces. N’incluez pas « .database.windows.net » lorsque vous spécifiez le nom du serveur SQL.

|Propriété | Valeur |
|---|---|
|Type de script|- Personnalisé|
|Nom|Mise à niveau de Hive|
|URI de script bash|`https://hdiconfigactions.blob.core.windows.net/hivemetastoreschemaupgrade/launch-schema-upgrade.sh`|
|Type(s) de nœud|Head|
|Paramètres|SQLSERVERNAME DATABASENAME USERNAME PASSWORD|

> [!Warning]  
> La mise à niveau qui convertit le schéma de métadonnées HDInsight 3.6 vers le schéma HDInsight 4.0 ne peut pas être annulée.

Vous pouvez vérifier la mise à niveau en exécutant la requête SQL suivante sur la base de données :

```sql
select * from dbo.version
```

### <a name="4-deploy-a-new-hdinsight-40-cluster"></a>4. Déployer un nouveau cluster HDInsight 4.0

1. Spécifiez le metastore mis à niveau en tant que metastore Hive du nouveau cluster.

1. Toutefois, les données réelles des tables ne sont pas accessibles tant que le cluster n’a pas accès aux comptes de stockage nécessaires.
Assurez-vous que les comptes de stockage des tables Hive dans le cluster HDInsight 3.6 sont spécifiés en tant que comptes de stockage principal ou secondaire du nouveau cluster HDInsight 4.0.
Pour plus d’informations sur l’ajout de comptes de stockage pour les clusters HDInsight, consultez [Ajouter des comptes de stockage supplémentaires à HDInsight](../hdinsight-hadoop-add-storage.md).

### <a name="5-complete-migration-with-a-post-upgrade-tool-in-hdinsight-40"></a>5. Terminer la migration avec un outil postérieur à la mise à niveau dans HDInsight 4.0

Par défaut, les tables managées doivent être compatibles ACID sur HDInsight 4.0. Une fois que vous avez terminé la migration du metastore, exécutez un outil postérieur à la mise à niveau pour rendre les tables managées précédemment non ACID compatibles avec le cluster HDInsight 4.0. Cet outil appliquera la conversion suivante :

|3.6 |4.0 |
|---|---|
|Tables externes|Tables externes|
|Tables managées non ACID|Tables externes avec la propriété 'external.table.purge'='true'|
|Tables managées ACID|Tables managées ACID|

Exécutez l’outil Hive postérieur à la mise à niveau à partir du cluster HDInsight 4.0 à l’aide de l’interpréteur de commandes SSH :

1. Connectez-vous au nœud principal du cluster à l’aide de SSH. Pour obtenir des instructions, consultez [Se connecter à HDInsight à l’aide de SSH](../hdinsight-hadoop-linux-use-ssh-unix.md)
1. Ouvrez un interpréteur de commandes de connexion en tant qu’utilisateur Hive en exécutant `sudo su - hive`
1. Exécutez la commande suivante à partir de l’interpréteur de commandes.

    ```bash
    STACK_VERSION=$(hdp-select status hive-server2 | awk '{ print $3; }')
    /usr/hdp/$STACK_VERSION/hive/bin/hive --config /etc/hive/conf --service  strictmanagedmigration --hiveconf hive.strict.managed.tables=true -m automatic --modifyManagedTables
    ```

Une fois que l’outil a terminé, votre entrepôt Hive est prêt pour HDInsight 4.0.

## <a name="migrate-from-internal-metastore"></a>Migrer à partir du metastore interne

Si votre cluster HDInsight 3.6 utilise un metastore Hive interne, suivez les étapes ci-dessous pour exécuter un script qui génère des requêtes Hive destinées à exporter des définitions d’objet à partir du metastore.

Les clusters HDInsight 3.6 et 4.0 doivent utiliser le même compte de stockage.

> [!NOTE]
>
> * Dans le cas de tables ACID, une copie des données sous-jacentes de la table est créée.
>
> * Ce script prend en charge la migration uniquement des bases de données, tables et partitions Hive. D’autres objets de métadonnées, tels que des vues, des UDF et des contraintes de table, sont censés être copiés manuellement.
>
> * Une fois ce script exécuté, on part du principe que l’ancien cluster ne sera plus utilisé pour accéder aux tables ou aux bases de données auxquelles le script fait référence.
>
> * Toutes les tables managées deviendront transactionnelles dans HDInsight 4.0. Si vous le souhaitez, vous pouvez conserver la table non transactionnelle en exportant les données vers une table externe avec la propriété 'external.table.purge'='true'. Par exemple,
>
>    ```SQL
>    create table tablename_backup like tablename;
>    insert overwrite table tablename_backup select * from tablename;
>    create external table tablename_tmp like tablename;
>    insert overwrite table tablename_tmp select * from tablename;
>    alter table tablename_tmp set tblproperties('external.table.purge'='true');
>    drop table tablename;
>    alter table tablename_tmp rename to tablename;
>    ```

1. Connectez-vous au cluster HDInsight 3.6 à l’aide d’un [client Secure Shell (SSH)](../hdinsight-hadoop-linux-use-ssh-unix.md).

1. À partir de la session SSH ouverte, téléchargez le fichier de script suivant pour générer un fichier nommé **alltables.hql**.

    ```bash
    wget https://hdiconfigactions.blob.core.windows.net/hivemetastoreschemaupgrade/exporthive_hdi_3_6.sh
    chmod 755 exporthive_hdi_3_6.sh
    ```

    * Pour un cluster HDInsight normal, sans ESP, exécutez simplement `exporthive_hdi_3_6.sh`.

    * Pour un cluster avec ESP, exécutez kinit et modifiez les arguments de Beeline : exécutez la commande suivante, en définissant USER et DOMAIN pour un utilisateur Azure AD disposant d’autorisations Hive complètes.

        ```bash
        USER="USER"  # replace USER
        DOMAIN="DOMAIN"  # replace DOMAIN
        DOMAIN_UPPER=$(printf "%s" "$DOMAIN" | awk '{ print toupper($0) }')
        kinit "$USER@$DOMAIN_UPPER"
        ```

        ```bash
        hn0=$(grep hn0- /etc/hosts | xargs | cut -d' ' -f4)
        BEE_CMD="beeline -u 'jdbc:hive2://$hn0:10001/default;principal=hive/_HOST@$DOMAIN_UPPER;auth-kerberos;transportMode=http' -n "$USER@$DOMAIN" --showHeader=false --silent=true --outputformat=tsv2 -e"
        ./exporthive_hdi_3_6.sh "$BEE_CMD"
        ```

1. Fermez votre session SSH. Entrez, ensuite une commande scp pour télécharger **alltables.hql** localement.

    ```bash
    scp sshuser@CLUSTERNAME-ssh.azurehdinsight.net:alltables.hql c:/hdi
    ```

1. Chargez **alltables.hql** dans le *nouveau* cluster HDInsight.

    ```bash
    scp c:/hdi/alltables.hql sshuser@CLUSTERNAME-ssh.azurehdinsight.net:/home/sshuser/
    ```

1. Utilisez ensuite SSH pour vous connecter au *nouveau* cluster HDInsight 4.0. Exécutez le code suivant à partir d’une session SSH à destination de ce cluster :

    Sans ESP :

    ```bash
    beeline -u "jdbc:hive2://localhost:10001/;transportMode=http" -f alltables.hql
    ```

    Avec ESP :

    ```bash
    USER="USER"  # replace USER
    DOMAIN="DOMAIN"  # replace DOMAIN
    DOMAIN_UPPER=$(printf "%s" "$DOMAIN" | awk '{ print toupper($0) }')
    kinit "$USER@$DOMAIN_UPPER"
    ```

    ```bash
    hn0=$(grep hn0- /etc/hosts | xargs | cut -d' ' -f4)
    beeline -u "jdbc:hive2://$hn0:10001/default;principal=hive/_HOST@$DOMAIN_UPPER;auth-kerberos;transportMode=http" -n "$USER@$DOMAIN" -f alltables.hql
    ```

L’outil postérieur à la mise à niveau pour la migration d’un metastore externe ne s’applique pas ici, car les tables managées non ACID de HDInsight 3.6 sont converties en tables managées ACID dans HDInsight 4.0.

> [!Important]  
> Les tables managées dans HDInsight 4.0 (y compris les tables migrées depuis la version 3.6) ne doivent pas être accessibles par d’autres services ou applications, y compris les clusters HDInsight 3.6.

## <a name="secure-hive-across-hdinsight-versions"></a>Sécuriser Hive sur les versions HDInsight

Depuis HDInsight 3.6, HDInsight s’intègre avec Azure Active Directory à l’aide du Pack Sécurité Entreprise (ESP) HDInsight. ESP utilise Kerberos et Apache Ranger à gérer les autorisations de ressources spécifiques au sein du cluster. Les stratégies Ranger déployées sur Hive dans HDInsight 3.6 peuvent être migrées vers HDInsight 4.0 en procédant comme suit :

1. Accédez au panneau Ranger Service Manager dans votre cluster HDInsight 3.6.
2. Accédez à la stratégie nommée **HIVE** et exportez la stratégie vers un fichier json.
3. Assurez-vous que tous les utilisateurs référencés dans le fichier json de stratégie exporté existent dans le nouveau cluster. Si un utilisateur est référencé dans le fichier json de stratégie, mais n’existe pas dans le nouveau cluster, ajoutez l’utilisateur dans le nouveau cluster ou supprimez la référence de la stratégie.
4. Accédez au panneau **Ranger Service Manager** de votre cluster HDInsight 4.0.
5. Accédez à la stratégie nommée **HIVE** et importez le fichier json de stratégie Ranger de l’étape 2.

## <a name="check-compatibility-and-modify-codes-as-needed-in-test-app"></a>Vérifier la compatibilité et modifier les codes si nécessaire dans l’application de test

Lors de la migration de charges de travail comme des programmes et des requêtes, consultez les notes de publication et la documentation pour découvrir les modifications et appliquer les modifications nécessaires. Si votre cluster HDInsight 3.6 utilise un metastore Spark et Hive partagé, [une configuration supplémentaire à l’aide de Hive Warehouse Connector](./apache-hive-warehouse-connector.md) est nécessaire.

## <a name="deploy-new-app-for-production"></a>Déployer une nouvelle application pour production

Pour basculer vers le nouveau cluster, vous pouvez installer une nouvelle application cliente et l’utiliser comme nouvel environnement de production, ou vous pouvez mettre à niveau votre application cliente existante et basculer vers HDInsight 4.0.

## <a name="switch-hdinsight-40-to-the-production"></a>Basculer HDInsight 4.0 en production

Si des différences ont été créées dans le metastore pendant le test, vous devez mettre à jour les modifications juste avant de basculer. Dans ce cas, vous pouvez exporter & importer le metastore, puis procéder à une nouvelle mise à niveau.

## <a name="remove-the-old-production"></a>Supprimer l’ancienne production

Une fois que vous avez confirmé que la mise en production était terminée et entièrement opérationnelle, vous pouvez supprimer la version 3.6 et le metastore précédent. Vérifiez que tout est migré avant de supprimer l’environnement.

## <a name="query-execution-across-hdinsight-versions"></a>Exécution de requêtes sur les versions HDInsight

Il existe deux façons d’exécuter et de déboguer les requêtes Hive/LLAP au sein d’un cluster HDInsight 3.6. L’interface CLI Hive offre une expérience de ligne de commande, tandis que [l’affichage Hive/Tez](../hadoop/apache-hadoop-use-hive-ambari-view.md) propose un workflow basé sur une interface utilisateur graphique.

Dans HDInsight 4.0, l’interface CLI Hive a été remplacée par BeeLine. L’affichage Tez/Hive propose un workflow basé sur une interface utilisateur graphique. HiveCLI est un client Thrift pour Hiveserver 1, et Beeline est un client JDBC qui fournit l’accès à Hiveserver 2. Beeline peut également être utilisé pour se connecter à n’importe quel autre point de terminaison de base de données compatible avec JDBC. Beeline est disponible de manière prédéfinie sur HDInsight 4.0 sans aucune installation nécessaire.

## <a name="next-steps"></a>Étapes suivantes

* [Annonce d’HDInsight 4.0](../hdinsight-version-release.md)
* [Présentation approfondie d’HDInsight 4.0](https://azure.microsoft.com/blog/deep-dive-into-azure-hdinsight-4-0/)
* [Tables ACID Hive 3](https://docs.hortonworks.com/HDPDocuments/HDP3/HDP-3.1.0/using-hiveql/content/hive_3_internals.html)
---
title: Stratégies Apache Hive dans Apache Ranger – Azure HDInsight
description: Découvrez comment configurer des stratégies Apache Ranger pour Hive dans un service Azure HDInsight avec le Pack Sécurité Entreprise (ESP, Enterprise Security Package).
author: omidm1
ms.author: omidm
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.custom: hdinsightactive
ms.date: 11/27/2019
ms.openlocfilehash: f2d9c96a616f05c22c8b999fdc6cab2505c27485
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92544934"
---
# <a name="configure-apache-hive-policies-in-hdinsight-with-enterprise-security-package"></a>Configurer des stratégies Apache Hive dans HDInsight avec le Pack Sécurité Entreprise

Découvrez comment configurer des stratégies Apache Ranger pour Apache Hive. Dans cet article, vous créez deux stratégies Ranger pour restreindre l’accès à hivesampletable. hivesampletable dispose de clusters HDInsight. Après avoir configuré les stratégies, vous utilisez Excel et le pilote ODBC pour vous connecter à des tables Hive dans HDInsight.

## <a name="prerequisites"></a>Prérequis

* Un cluster HDInsight avec le Pack Sécurité Entreprise. Voir [Configurer des clusters HDInsight avec ESP](./apache-domain-joined-configure-using-azure-adds.md).
* Une station de travail avec Microsoft 365 Apps for enterprise, Office 2016, Office Professionnel Plus 2013, l'édition autonome d'Excel 2013 ou Office Professionnel Plus 2010.

## <a name="connect-to-apache-ranger-admin-ui"></a>Connexion à l’interface utilisateur Apache Ranger
**Pour vous connecter à l’interface utilisateur Ranger**

1. À partir d’un navigateur, accédez à l’interface utilisateur Ranger à l’adresse `https://CLUSTERNAME.azurehdinsight.net/Ranger/`, où CLUSTERNAME est le nom de votre cluster.

   > [!NOTE]  
   > Ranger utilise des informations d’identification différentes de celles du cluster Apache Hadoop. Pour empêcher les navigateurs d’utiliser les informations d’identification Hadoop mises en cache, utilisez une nouvelle fenêtre de navigation InPrivate pour vous connecter à l’interface utilisateur administrateur Ranger.

2. Connectez-vous avec le nom d’utilisateur et le mot de passe du domaine de l’administrateur du cluster :

    ![Page d’accueil HDInsight ESP Ranger](./media/apache-domain-joined-run-hive/hdinsight-domain-joined-ranger-home-page.png)

    Actuellement, Ranger fonctionne uniquement avec Yarn et Hive.

## <a name="create-domain-users"></a>Création d’utilisateurs du domaine

Pour plus d’informations sur la façon de créer hiveruser1 et hiveuser2, consultez [Créer un cluster HDInsight avec ESP](apache-domain-joined-configure-using-azure-adds.md#create-an-hdinsight-cluster-with-esp). Dans cet article, vous utilisez les deux comptes utilisateur.

## <a name="create-ranger-policies"></a>Création de stratégies Ranger

Dans cette section, vous créez deux stratégies Ranger pour accéder à hivesampletable. Vous accordez une autorisation select sur différents ensembles de colonnes. Les deux utilisateurs ont été créés en suivant les instructions de l’article [Créer un cluster HDInsight avec ESP](apache-domain-joined-configure-using-azure-adds.md#create-an-hdinsight-cluster-with-esp). Dans la section suivante, vous allez tester les deux stratégies dans Excel.

**Pour créer des stratégies Ranger**

1. Ouvrez l’interface utilisateur Ranger. Consultez Connexion à l’interface utilisateur Apache Ranger.
2. Cliquez sur **CLUSTERNAME_Hive** , sous **Hive** . Deux stratégies préconfigurées doivent s’afficher.
3. Sélectionnez **Ajouter une nouvelle stratégie** , puis entrez les valeurs suivantes :

    |Propriété |Valeur |
    |---|---|
    |Nom de la stratégie|read-hivesampletable-all|
    |Base de données Hive|default|
    |table|hivesampletable|
    |Colonne Hive|*|
    |Sélectionner un utilisateur|hiveuser1|
    |Autorisations|select|

    ![Configuration de stratégies Hive pour HDInsight ESP Ranger](./media/apache-domain-joined-run-hive/hdinsight-domain-joined-configure-ranger-policy.png).

    > [!NOTE]  
    > Si un utilisateur de domaine n’est pas renseigné dans Sélectionner un utilisateur, attendez quelques instants que Ranger se synchronise avec AAD.

4. Sélectionnez **Ajouter** pour enregistrer la stratégie.

5. Répétez les deux dernières étapes pour créer une autre stratégie avec les propriétés suivantes :

    |Propriété |Valeur |
    |---|---|
    |Nom de la stratégie|read-hivesampletable-devicemake|
    |Base de données Hive|default|
    |table|hivesampletable|
    |Colonne Hive|clientid, devicemake|
    |Sélectionner un utilisateur|hiveuser2|
    |Autorisations|select|

## <a name="create-hive-odbc-data-source"></a>Création d’une source de données ODBC Hive

Vous trouverez les instructions dans [Création d’une source de données ODBC Hive](../hadoop/apache-hadoop-connect-excel-hive-odbc-driver.md).  

 | Propriété  |Description |
 | --- | --- |
 | Nom de la source de données | Donnez un nom à votre source de données |
 | Host | Entrez CLUSTERNAME.azurehdinsight.net. Par exemple, myHDICluster.azurehdinsight.net |
 | Port | Utilisez **443** . (ce port est passé de 563 à 443). |
 | Base de données | Utilisez **Default** . |
 | Hive Server Type | Sélectionnez **Hive Server 2** . |
 | Mechanism | Sélectionnez **Azure HDInsight Service** . |
 | HTTP Path | Laissez cette valeur vide. |
 | User Name | Entrez hiveuser1@contoso158.onmicrosoft.com. Mettez à jour le nom de domaine s’il est différent. |
 | Mot de passe | Entrez le mot de passe pour hiveuser1. |

Veillez à cliquer sur **Test** avant d’enregistrer la source de données.

## <a name="import-data-into-excel-from-hdinsight"></a>Importation de données dans Microsoft Excel à partir de HDInsight

Dans la dernière section, vous avez configuré deux stratégies.  hiveuser1 a l’autorisation select sur toutes les colonnes et hiveuser2 a l’autorisation select sur deux colonnes. Dans cette section, vous représentez les deux utilisateurs pour importer des données dans Excel.

1. Ouvrez un nouveau classeur ou un classeur existant dans Excel.

1. À partir de l’onglet **Données** , accédez à **Obtenir des données** > **Depuis d’autres sources** > **Depuis ODBC** afin de lancer la fenêtre **Depuis ODBC** .

    ![Assistant Ouvrir la connexion de données](./media/apache-domain-joined-run-hive/simbahiveodbc-excel-dataconnection1.png)

1. Dans la liste déroulante, sélectionnez le nom de la source de données que vous avez créée dans la dernière section, puis sélectionnez **OK** .

1. Lors de la première utilisation, une boîte de dialogue **Pilote ODBC** s’ouvre. Dans le menu gauche, sélectionnez **Windows** . Sélectionnez ensuite **Se connecter** pour ouvrir la fenêtre **Navigateur** .

1. Attendez l'ouverture de la boîte de dialogue **Sélection d'une base de données et d'une table** . Cette opération peut prendre quelques secondes.

1. Sélectionnez **hivesampletable** , puis sélectionnez **Suivant** .

1. Sélectionnez **Terminer** .

1. Dans la boîte de dialogue **Importation de données** , vous pouvez modifier ou spécifier la requête. Pour ce faire, sélectionnez **Propriétés** . Cette opération peut prendre quelques secondes.

1. Sélectionnez l’onglet **Définition** . Le texte de commande est le suivant :

    ```sql
    SELECT * FROM "HIVE"."default"."hivesampletable"`
    ```

   Selon les stratégies Ranger que vous avez définies, hiveuser1 a l’autorisation select sur toutes les colonnes.  Par conséquent, cette requête fonctionne avec les informations d’identification de hiveuser1, mais elle ne fonctionne pas avec les informations d’identification de hiveuser2.

1. Sélectionnez **OK** pour fermer la boîte de dialogue Propriétés de connexion.

1. Sélectionnez **OK** pour fermer la boîte de dialogue **Importation de données** .  

1. Entrez à nouveau le mot de passe pour hiveuser1 et cliquez sur **OK** . Patientez quelques secondes pour que les données soient importées dans Excel. Une fois le processus terminé, vous devez voir 11 colonnes de données.

Pour tester la deuxième stratégie (read-hivesampletable-devicemake) que vous avez créée dans la dernière section

1. Ajoutez une nouvelle feuille dans Excel.
2. Suivez la procédure précédente pour importer les données.  La seule modification à effectuer consiste à utiliser les informations d’identification de hiveuser2 au lieu de hiveuser1. Cette opération échoue, car hiveuser2 n’est autorisé à afficher que deux colonnes. Vous devez obtenir l’erreur suivante :

    ```output
    [Microsoft][HiveODBC] (35) Error from Hive: error code: '40000' error message: 'Error while compiling statement: FAILED: HiveAccessControlException Permission denied: user [hiveuser2] does not have [SELECT] privilege on [default/hivesampletable/clientid,country ...]'.
    ```

3. Suivez la même procédure pour importer les données. Cette fois, utilisez les informations d’identification de hiveuser2 et modifiez également l’instruction select de :

    ```sql
    SELECT * FROM "HIVE"."default"."hivesampletable"
    ```

    to:

    ```sql
    SELECT clientid, devicemake FROM "HIVE"."default"."hivesampletable"
    ```

    Une fois le processus terminé, vous devez voir deux colonnes de données importées.

## <a name="next-steps"></a>Étapes suivantes

* Pour configurer un cluster HDInsight avec le Pack Sécurité Entreprise, consultez [Configurer des clusters HDInsight avec ESP](./apache-domain-joined-configure-using-azure-adds.md).
* Pour gérer un cluster HDInsight avec ESP, consultez [Gérer des clusters HDInsight avec ESP](apache-domain-joined-manage.md).
* Pour exécuter des requêtes Hive en utilisant SSH sur des clusters HDInsight avec ESP, consultez [Utiliser SSH avec HDInsight](../hdinsight-hadoop-linux-use-ssh-unix.md#authentication-domain-joined-hdinsight).
* Pour connecter Hive avec Hive JDBC, voir [Se connecter à Apache Hive sur Azure HDInsight avec le pilote Hive JDBC](../hadoop/apache-hadoop-connect-hive-jdbc-driver.md).
* Pour connecter Excel à Hadoop avec Hive ODBC, voir [Connecter Excel à Apache Hadoop avec le pilote Microsoft Hive ODBC](../hadoop/apache-hadoop-connect-excel-hive-odbc-driver.md).
* Pour connecter Excel à Hadoop avec Power Query, voir [Connecter Excel à Apache Hadoop avec Power Query](../hadoop/apache-hadoop-connect-excel-power-query.md).
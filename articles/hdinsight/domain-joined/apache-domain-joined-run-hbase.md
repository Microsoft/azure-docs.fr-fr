---
title: Apache HBase et Pack Sécurité Entreprise – Azure HDInsight
description: Didacticiel - Découvrez comment configurer des stratégies Apache Ranger pour HBase dans Azure HDInsight avec le Pack Sécurité Entreprise.
ms.service: hdinsight
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.topic: tutorial
ms.date: 09/04/2019
ms.openlocfilehash: 5747de399e7ae0cfe99ba013f8da376be0ba1b2a
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95993633"
---
# <a name="tutorial-configure-apache-hbase-policies-in-hdinsight-with-enterprise-security-package"></a>Tutoriel : Configurer des stratégies Apache HBase dans HDInsight avec le Pack Sécurité Entreprise

Découvrez comment configurer des stratégies Apache Ranger pour des clusters Apache HBase avec le Pack Sécurité Entreprise (ESP). Les clusters ESP sont connectés à un domaine permettant aux utilisateurs de s’authentifier avec les informations d’identification du domaine. Dans ce tutoriel, vous créez deux stratégies Ranger pour restreindre l’accès à différentes familles de colonnes dans une table HBase.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer des utilisateurs du domaine
> * Création de stratégies Ranger
> * Créer des tables dans un cluster HBase
> * Tester des stratégies Ranger

## <a name="before-you-begin"></a>Avant de commencer

* Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/).

* Connectez-vous au [portail Azure](https://portal.azure.com/).

* Créez un [cluster HBase HDInsight avec le Pack Sécurité Entreprise](apache-domain-joined-configure-using-azure-adds.md).

## <a name="connect-to-apache-ranger-admin-ui"></a>Connexion à l’interface utilisateur Apache Ranger

1. À partir d’un navigateur, connectez-vous à l’interface d’utilisateur administrateur de Ranger avec l’URL `https://<ClusterName>.azurehdinsight.net/Ranger/`. N’oubliez pas de remplacer `<ClusterName>` par le nom de votre cluster HBase.

    > [!NOTE]  
    > Les informations d’identification de Ranger ne sont pas les mêmes que celles du cluster Hadoop. Pour empêcher les navigateurs d’utiliser les informations d’identification Hadoop mises en cache, utilisez une nouvelle fenêtre de navigation InPrivate pour vous connecter à l’interface utilisateur administrateur de Ranger.

2. Connectez-vous avec vos informations d’identification d’administrateur Azure Active Directory. Les informations d’identification administrateur Azure AD ne sont pas les mêmes que celles du cluster HDInsight ou que celles du protocole SSH du nœud HDInsight Linux.

## <a name="create-domain-users"></a>Créer des utilisateurs du domaine

Consultez [Créer un cluster HDInsight avec le Pack Sécurité Entreprise](./apache-domain-joined-configure-using-azure-adds.md) pour découvrir comment créer les utilisateurs du domaine **sales_user1** et **marketing_user1**. Dans un scénario de production, les utilisateurs du domaine proviennent de votre locataire Active Directory.

## <a name="create-hbase-tables-and-import-sample-data"></a>Créer des tables HBase et importer des données d’exemple

Vous pouvez utiliser SSH pour vous connecter à des clusters HBase, puis utiliser [Apache HBase Shell](https://hbase.apache.org/0.94/book/shell.html) pour créer des tables HBase, et insérer et interroger des données. Pour en savoir plus, voir [Utilisation de SSH avec Hadoop Linux sur HDInsight depuis Linux, Unix ou OS X](../hdinsight-hadoop-linux-use-ssh-unix.md).

### <a name="to-use-the-hbase-shell"></a>Pour utiliser l’interpréteur de commandes HBase

1. À partir de SSH, exécutez la commande HBase suivante :
   
    ```bash
    hbase shell
    ```

2. Créez une table HBase `Customers` avec deux familles de colonnes : `Name` et `Contact`.

    ```hbaseshell   
    create 'Customers', 'Name', 'Contact'
    list
    ```
3. Insérez des données :
    
    ```hbaseshell   
    put 'Customers','1001','Name:First','Alice'
    put 'Customers','1001','Name:Last','Johnson'
    put 'Customers','1001','Contact:Phone','333-333-3333'
    put 'Customers','1001','Contact:Address','313 133rd Place'
    put 'Customers','1001','Contact:City','Redmond'
    put 'Customers','1001','Contact:State','WA'
    put 'Customers','1001','Contact:ZipCode','98052'
    put 'Customers','1002','Name:First','Robert'
    put 'Customers','1002','Name:Last','Stevens'
    put 'Customers','1002','Contact:Phone','777-777-7777'
    put 'Customers','1002','Contact:Address','717 177th Ave'
    put 'Customers','1002','Contact:City','Bellevue'
    put 'Customers','1002','Contact:State','WA'
    put 'Customers','1002','Contact:ZipCode','98008'
    ```
4. Affichez le contenu de la table :
    
    ```hbaseshell
    scan 'Customers'
    ```

    ![Sortie de l’interpréteur de commandes HBase Hadoop HDInsight](./media/apache-domain-joined-run-hbase/hbase-shell-scan-table.png)

## <a name="create-ranger-policies"></a>Création de stratégies Ranger

Créez une stratégie Ranger pour **sales_user1** et **marketing_user1**.

1. Ouvrez **l’interface utilisateur de l’administrateur Ranger**. Cliquez sur **\<ClusterName>_hbase** sous **HBase**.

   ![Interface utilisateur d’administration Apache Ranger HDInsight](./media/apache-domain-joined-run-hbase/apache-ranger-admin-login.png)

2. L’écran **List of Policies** (Liste des stratégies) affiche toutes les stratégies Ranger qui ont été créées pour ce cluster. Il est possible qu’une stratégie préconfigurée figure dans la liste. Cliquez sur **Add New Policy** (Ajouter une nouvelle stratégie).

    ![Liste des stratégies Apache Ranger pour HBase](./media/apache-domain-joined-run-hbase/apache-ranger-hbase-policies-list.png)

3. Dans l’écran **Create Policy** (Créer une stratégie), entrez les valeurs suivantes :

   |**Paramètre**  |**Valeur suggérée**  |
   |---------|---------|
   |Nom de la stratégie  |  sales_customers_name_contact   |
   |HBase Table (Table HBase)   |  Clients |
   |HBase Column-family (Famille de colonnes HBase)   |  Name, Contact |
   |HBase Column (Colonne HBase)   |  * |
   |Select Group (Sélectionner un groupe)  | |
   |Sélectionner un utilisateur  | sales_user1 |
   |Autorisations  | Lire |

   Les caractères génériques suivants peuvent être inclus dans le nom de la rubrique :

   * `*` indique zéro, une ou plusieurs occurrences des caractères.
   * `?` indique n’importe quel caractère individuel.

   ![créer une stratégie Apache Ranger pour sales](./media/apache-domain-joined-run-hbase/apache-ranger-hbase-policy-create-sales.png)

   >[!NOTE]
   >Attendez quelques instants pour que Ranger se synchronise avec Azure AD si un utilisateur du domaine n’est pas renseigné automatiquement dans **Sélectionner un utilisateur**.

4. Cliquez sur **Ajouter** pour enregistrer la stratégie.

5. Cliquez sur **Ajouter une nouvelle stratégie**, puis entrez les valeurs suivantes :

   |**Paramètre**  |**Valeur suggérée**  |
   |---------|---------|
   |Nom de la stratégie  |  marketing_customers_contact   |
   |HBase Table (Table HBase)   |  Clients |
   |HBase Column-family (Famille de colonnes HBase)   |  Contact |
   |HBase Column (Colonne HBase)   |  * |
   |Select Group (Sélectionner un groupe)  | |
   |Sélectionner un utilisateur  | marketing_user1 |
   |Autorisations  | Lire |

   ![créer une stratégie Apache Ranger pour marketing](./media/apache-domain-joined-run-hbase/apache-ranger-hbase-policy-create-marketing.png)  

6. Cliquez sur **Ajouter** pour enregistrer la stratégie.

## <a name="test-the-ranger-policies"></a>Tester les stratégies Ranger

En fonction des stratégies Ranger configurées, **sales_user1** peut voir toutes les données des colonnes à la fois dans les familles de colonnes `Name` et `Contact`. **marketing_user1** peut uniquement voir les données dans la famille de colonnes `Contact`.

### <a name="access-data-as-sales_user1"></a>Accéder aux données en tant qu’utilisateur sales_user1

1. Ouvrez une connexion SSH avec le cluster. Utilisez la commande suivante pour vous connecter au cluster :

   ```bash
   ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
   ```

1. Utilisez la commande kinit pour changer le contexte à celui de notre utilisateur.

   ```bash
   kinit sales_user1
   ```

2. Ouvrez l’interpréteur de commandes HBase et analysez la table `Customers`.

   ```hbaseshell
   hbase shell
   scan `Customers`
   ```

3. Notez que l’utilisateur sales peut voir toutes les colonnes de la table `Customers`, y compris les deux colonnes dans la famille de colonnes `Name` ainsi que les cinq colonnes dans la famille de colonnes `Contact`.

    ```hbaseshell
    ROW                                COLUMN+CELL
     1001                              column=Contact:Address, timestamp=1548894873820, value=313 133rd Place
     1001                              column=Contact:City, timestamp=1548895061523, value=Redmond
     1001                              column=Contact:Phone, timestamp=1548894871759, value=333-333-3333
     1001                              column=Contact:State, timestamp=1548895061613, value=WA
     1001                              column=Contact:ZipCode, timestamp=1548895063111, value=98052
     1001                              column=Name:First, timestamp=1548894871561, value=Alice
     1001                              column=Name:Last, timestamp=1548894871707, value=Johnson
     1002                              column=Contact:Address, timestamp=1548894899174, value=717 177th Ave
     1002                              column=Contact:City, timestamp=1548895103129, value=Bellevue
     1002                              column=Contact:Phone, timestamp=1548894897524, value=777-777-7777
     1002                              column=Contact:State, timestamp=1548895103231, value=WA
     1002                              column=Contact:ZipCode, timestamp=1548895104804, value=98008
     1002                              column=Name:First, timestamp=1548894897419, value=Robert
     1002                              column=Name:Last, timestamp=1548894897487, value=Stevens
    2 row(s) in 0.1000 seconds
    ```

### <a name="access-data-as-marketing_user1"></a>Accéder aux données en tant qu’utilisateur marketing_user1

1. Ouvrez une connexion SSH avec le cluster. Utilisez la commande suivante pour vous connecter en tant que **marketing_user1** :

   ```bash
   ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
   ```

1. Utilisez la commande kinit pour changer le contexte à celui de notre utilisateur

   ```bash
   kinit marketing_user1
   ```

1. Ouvrez l’interpréteur de commandes HBase et analysez la table `Customers`.

    ```hbaseshell
    hbase shell
    scan `Customers`
    ```

1. Notez que l’utilisateur marketing peut uniquement voir les cinq colonnes de la famille de colonnes `Contact`.

    ```hbaseshell
    ROW                                COLUMN+CELL
     1001                              column=Contact:Address, timestamp=1548894873820, value=313 133rd Place
     1001                              column=Contact:City, timestamp=1548895061523, value=Redmond
     1001                              column=Contact:Phone, timestamp=1548894871759, value=333-333-3333
     1001                              column=Contact:State, timestamp=1548895061613, value=WA
     1001                              column=Contact:ZipCode, timestamp=1548895063111, value=98052
     1002                              column=Contact:Address, timestamp=1548894899174, value=717 177th Ave
     1002                              column=Contact:City, timestamp=1548895103129, value=Bellevue
     1002                              column=Contact:Phone, timestamp=1548894897524, value=777-777-7777
     1002                              column=Contact:State, timestamp=1548895103231, value=WA
     1002                              column=Contact:ZipCode, timestamp=1548895104804, value=98008
    2 row(s) in 0.0730 seconds
    ```

1. Consultez les événements d’accès de l’audit à partir de l’interface utilisateur de Ranger.

   ![Audit de stratégie de l’interface utilisateur Ranger HDInsight](./media/apache-domain-joined-run-hbase/apache-ranger-admin-audit.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous ne comptez pas continuer à utiliser cette application, effectuez les étapes suivantes pour supprimer le cluster HBase que vous avez créé :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Dans la zone **Recherche** située en haut, tapez **HDInsight**. 
1. Sous **Services**, sélectionnez **Clusters HDInsight**.
1. Dans la liste des clusters HDInsight qui s’affiche, cliquez sur les points de suspension **...** à côté du cluster que vous avez créé pour ce tutoriel. 
1. Cliquez sur **Supprimer**. Cliquez sur **Oui**.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Bien démarrer avec Apache HBase](../hbase/apache-hbase-tutorial-get-started-linux.md)
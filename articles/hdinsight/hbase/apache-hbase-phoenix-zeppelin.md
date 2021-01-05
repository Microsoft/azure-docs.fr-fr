---
title: Exécuter des requêtes Apache HBase dans Azure HDInsight avec Apache Phoenix
description: Apprenez à utiliser Apache Zeppelin pour exécuter des requêtes Apache Base avec Phoenix.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: how-to
ms.date: 10/14/2019
ms.openlocfilehash: e85226e7a2544cda01f40a1c1be6be32f11d9dfc
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92542792"
---
# <a name="use-apache-zeppelin-to-run-apache-phoenix-queries-over-apache-hbase-in-azure-hdinsight"></a>Utiliser Apache Zeppelin pour exécuter des requêtes Apache Phoenix sur Apache HBase dans Azure HDInsight

Apache Phoenix est une couche de base de données relationnelle massivement parallèle open source créée sur HBase. Phoenix vous donne les moyens d’utiliser des requêtes de type SQL sur HBase. Phoenix utilise en dessous des pilotes JDBC pour permettre aux utilisateurs de créer, supprimer et modifier des tables SQL, des index, des vues et des séquences.  Vous pouvez également utiliser Phoenix pour mettre à jour des lignes individuellement et en bloc. Phoenix utilise une compilation native NOSQL au lieu de MapReduce pour compiler les requêtes, ce qui permet de créer des applications à faible latence sur HBase.

Apache Zeppelin est un notebook Web open source qui vous permet de créer des documents basés sur des données et collaboratifs à l’aide d’analyses de données interactives et de langages tels que SQL et Scala. Il permet aux développeurs de données et aux scientifiques des données de développer, d’organiser, d’exécuter et de partager du code pour la manipulation des données. Il vous permet de visualiser les résultats sans faire référence à la ligne de commande ou nécessiter les détails de cluster.

Les utilisateurs de HDInsight peuvent utiliser Apache Zeppelin pour interroger des tables Phoenix. Apache Zeppelin est intégré au cluster HDInsight et aucune étape supplémentaire n’est nécessaire pour l’utiliser. Créez simplement un Notebook Zeppelin avec l’interpréteur JDBC et commencez à écrire vos requêtes SQL Phoenix

## <a name="prerequisites"></a>Prérequis

Un cluster Apache HBase sur HDInsight. Consultez [Bien démarrer avec Apache HBase](./apache-hbase-tutorial-get-started-linux.md).

## <a name="create-an-apache-zeppelin-note"></a>Créer une note Apache Zeppelin

1. Remplacez `CLUSTERNAME` par le nom de votre cluster dans l’URL suivante `https://CLUSTERNAME.azurehdinsight.net/zeppelin`. Entrez ensuite l’URL dans un navigateur web. Entrez le nom d’utilisateur et le mot de passe de connexion pour votre cluster.

1. Dans la page Zeppelin, sélectionnez **Créer une note** .

    ![Zeppelin avec Interactive Query HDInsight](./media/apache-hbase-phoenix-zeppelin/hbase-zeppelin-create-note.png)

1. Dans la boîte de dialogue **Créer une note** , entrez ou sélectionnez les valeurs suivantes :

    - Nom de la note : entrez un nom pour la note.
    - Interpréteur par défaut : sélectionnez **jdbc** dans la liste déroulante.

    Sélectionnez ensuite **Créer une note** .

1. Vérifiez que l'en-tête du bloc-notes indique un état connecté. Il est indiqué par un point vert dans le coin supérieur droit.

    ![État du notebook Zeppelin](./media/apache-hbase-phoenix-zeppelin/hbase-zeppelin-connected.png "État du bloc-notes Zeppelin")

1. Créez une table HBase. Entrez la commande suivante, puis appuyez sur **Maj + Entrée**  :

    ```sql
    %jdbc(phoenix)
    CREATE TABLE Company (
        company_id INTEGER PRIMARY KEY,
        name VARCHAR(225)
    );
    ```

    L’instruction **%jdbc(phoenix)** dans la première ligne indique au notebook d’utiliser l’interpréteur Phoenix JDBC.

1. Affichez les tables créées.

    ```sql
    %jdbc(phoenix)
    SELECT DISTINCT table_name
    FROM SYSTEM.CATALOG
    WHERE table_schem is null or table_schem <> 'SYSTEM';
    ```

1. Insérer les valeurs dans le tableau.

    ```sql
    %jdbc(phoenix)
    UPSERT INTO dbo.Company VALUES(1, 'Microsoft');
    UPSERT INTO dbo.Company (name, company_id) VALUES('Apache', 2);
    ```

1. Interroger la table.

    ```sql
    %jdbc(phoenix)
    SELECT * FROM dbo.Company;
    ```

1. Supprimer un enregistrement.

    ```sql
    %jdbc(phoenix)
    DELETE FROM dbo.Company WHERE COMPANY_ID=1;
    ```

1. Déposer la table.

    ```sql
    %jdbc(phoenix)
    DROP TABLE dbo.Company;
    ```

## <a name="next-steps"></a>Étapes suivantes

- [Apache Phoenix prend désormais en charge Zeppelin dans Azure HDInsight](/archive/blogs/ashish/apache-phoenix-now-supports-zeppelin-in-azure-hdinsight)
- [Grammaire Apache Phoenix](https://phoenix.apache.org/language/index.html)
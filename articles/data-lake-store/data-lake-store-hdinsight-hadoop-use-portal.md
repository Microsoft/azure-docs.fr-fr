---
title: Créer des clusters Azure HDInsight avec Data Lake Storage Gen1 - portail
description: Utiliser le portail Azure pour créer et utiliser les clusters HDInsight avec Azure Data Lake Storage Gen1
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.openlocfilehash: 666b39e2a600fe6ca004798ed4f8371cdd1dfe5f
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96340252"
---
# <a name="create-hdinsight-clusters-with-azure-data-lake-storage-gen1-by-using-the-azure-portal"></a>Créer des clusters HDInsight avec Azure Data Lake Storage Gen1 à l’aide du portail Azure

> [!div class="op_single_selector"]
> * [Utiliser le portail Azure](data-lake-store-hdinsight-hadoop-use-portal.md)
> * [Utiliser PowerShell (pour le stockage par défaut)](data-lake-store-hdinsight-hadoop-use-powershell-for-default-storage.md)
> * [Utiliser PowerShell (pour le stockage supplémentaire)](data-lake-store-hdinsight-hadoop-use-powershell.md)
> * [Utiliser Resource Manager](data-lake-store-hdinsight-hadoop-use-resource-manager-template.md)
>
>

Découvrez comment utiliser le portail Azure pour créer un cluster HDInsight avec Azure Data Lake Storage Gen1 comme stockage par défaut ou stockage supplémentaire. Bien que le stockage supplémentaire soit facultatif pour les clusters HDInsight, il est recommandé de stocker vos données d’entreprise dans les comptes de stockage supplémentaires.

## <a name="prerequisites"></a>Conditions préalables requises

Avant de commencer, vérifiez que vous respectez les exigences suivantes :

* **Un abonnement Azure**. Consultez [Créez votre compte Azure gratuit](https://azure.microsoft.com/pricing/free-trial/).
* **Un compte Azure Data Lake Storage Gen1**. Suivez les instructions indiquées dans [Prise en main d’Azure Data Lake Storage Gen1 avec le portail Azure](data-lake-store-get-started-portal.md). Vous devez également créer un dossier racine sur le compte.  Dans cet article, un dossier racine appelé __/clusters__ est utilisé.
* **Un principal de service Azure Active Directory**. Ce guide pratique montre comment créer un principal de service dans Azure Active Directory (Azure AD). Toutefois, vous devez être administrateur Azure AD pour pouvoir créer un principal du service. Si vous êtes administrateur, vous pouvez ignorer ce prérequis et poursuivre.

>[!NOTE]
>Vous pouvez créer un principal de service uniquement si vous êtes administrateur Azure AD. Votre administrateur Azure AD doit créer un principal de service. Vous pouvez ensuite créer un cluster HDInsight avec Data Lake Storage Gen1. En outre, le principal de service doit être créé à l’aide d’un certificat, comme décrit dans [Créer un principal du service avec un certificat](../active-directory/develop/howto-authenticate-service-principal-powershell.md#create-service-principal-with-self-signed-certificate).
>

## <a name="create-an-hdinsight-cluster"></a>Création d'un cluster HDInsight

Dans cette section, vous allez créer un cluster HDInsight avec Data Lake Storage Gen1 comme stockage par défaut ou stockage supplémentaire. Cet article se concentre uniquement sur la configuration de Data Lake Storage Gen1. Pour obtenir des informations générales sur la création de clusters et les procédures associées, consultez la page [Création de clusters Hadoop dans HDInsight](../hdinsight/hdinsight-hadoop-provision-linux-clusters.md).

### <a name="create-a-cluster-with-data-lake-storage-gen1-as-default-storage"></a>Créer un cluster avec Data Lake Storage Gen1 en tant que stockage par défaut

Pour créer un cluster HDInsight avec Data Lake Storage Gen1 comme compte de stockage par défaut :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Suivez les indications présentées dans [Créer des clusters](../hdinsight/hdinsight-hadoop-create-linux-clusters-portal.md#create-clusters) pour obtenir des informations générales sur la création de clusters HDInsight.
3. Sur le panneau **Stockage**, sous **Type de stockage principal**, sélectionnez **Azure Data Lake Storage Gen1**, puis entrez les informations suivantes :

    ![Paramètres du compte de stockage HDInsight](./media/data-lake-store-hdinsight-hadoop-use-portal/hdi.adl.1.adls.storage.png)

    * **Sélectionner un compte Data Lake Storage** : sélectionnez un compte Data Lake Storage Gen1 existant. Un compte Data Lake Storage Gen1 existant est requis.  Consultez les [Conditions préalables](#prerequisites).
    * **Chemin racine** : entrez un chemin d’accès où les fichiers spécifiques d’un cluster doivent être stockés. Sur la capture d’écran, il s’agit de __/clusters/myhdiadlcluster/__ , qui doit comporter le dossier __/clusters__. Le portail crée alors un dossier *myhdicluster*.  *myhdicluster* désigne le nom du cluster.
    * **Accès à Data Lake Storage** : configurez l’accès entre le compte Data Lake Storage Gen1 et le cluster HDInsight. Pour obtenir des instructions, consultez [Configurer l’accès aux données Data Lake Storage Gen1](#configure-data-lake-storage-gen1-access).
    * **Comptes de stockage supplémentaire** : ajoutez des comptes de stockage Azure en tant que comptes de stockage supplémentaire pour le cluster. Pour ajouter d’autres comptes Data Lake Storage Gen1, attribuez des autorisations de cluster sur les données de plusieurs comptes Data Lake Storage Gen1 lors de la configuration d’un compte Data Lake Storage Gen1 comme type de stockage principal. Consultez [Configurer l’accès aux données Data Lake Storage Gen1](#configure-data-lake-storage-gen1-access).

4. Cliquez sur **Sélectionner** dans les panneaux **Accès à Data Lake Store** et poursuivez la création du cluster comme décrit dans [Création de clusters Hadoop dans HDInsight](../hdinsight/hdinsight-hadoop-create-linux-clusters-portal.md).

### <a name="create-a-cluster-with-data-lake-storage-gen1-as-additional-storage"></a>Créer un cluster avec Data Lake Storage Gen1 en tant que stockage supplémentaire

Effectuez les étapes suivantes pour créer un cluster HDInsight avec un compte de stockage Blob Azure comme stockage par défaut, et un compte de stockage Data Lake Storage Gen1 comme stockage supplémentaire.

Pour créer un cluster HDInsight avec Data Lake Storage Gen1 comme compte de stockage supplémentaire :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Suivez les indications présentées dans [Créer des clusters](../hdinsight/hdinsight-hadoop-create-linux-clusters-portal.md#create-clusters) pour obtenir des informations générales sur la création de clusters HDInsight.
3. Sur le panneau **Stockage**, sous **Type de stockage principal**, sélectionnez **Stockage Azure**, puis entrez les informations suivantes :

    ![Paramètres du compte de stockage HDInsight - Stockage supplémentaire](./media/data-lake-store-hdinsight-hadoop-use-portal/hdi.adl.1.png)

    * **Méthode de sélection** - Pour spécifier un compte de stockage qui fait partie de votre abonnement Azure, sélectionnez **Mes abonnements**, puis le compte de stockage. Pour spécifier un compte de stockage qui est en dehors de votre abonnement Azure, sélectionnez **Clé d’accès**, puis fournissez les informations du compte de stockage externe.

    * **Conteneur par défaut** - Utilisez la valeur par défaut ou spécifiez votre propre nom.
    * **Comptes de stockage supplémentaire** - Ajoutez des comptes de stockage Azure en tant que stockage supplémentaire.
    * **Accès à Data Lake Storage** - Configurez l’accès entre le compte Data Lake Storage Gen1 et le cluster HDInsight. Pour obtenir des instructions, consultez [Configurer l’accès aux données Data Lake Storage Gen1](#configure-data-lake-storage-gen1-access).

## <a name="configure-data-lake-storage-gen1-access"></a>Configurer l’accès aux données Data Lake Storage Gen1

Dans cette section, vous configurez l’accès à Data Lake Storage Gen1 à partir des clusters HDInsight au moyen d’un principal du service Azure Active Directory.

### <a name="specify-a-service-principal"></a>Spécifier un principal du service

Dans le portail Azure, vous pouvez utiliser un principal du service existant ou en créer un.

Pout créer un principal de service dans le portail Azure :
1. Consultez [Créer un principal de service et des certificats](../active-directory/develop/howto-create-service-principal-portal.md) à l’aide d’Azure Active Directory.

Utiliser un principal du service existant dans le portail Azure :

1. Le principal du service doit avoir des autorisations de propriétaire sur le compte de stockage. Consultez [Configurer des autorisations pour que le principal du service soit propriétaire du compte de stockage](#configure-serviceprincipal-permissions).
1. Sélectionnez **Accès à Data Lake Store**.
1. Sur le panneau **Accès à Data Lake Storage Gen1**, sélectionnez **Utiliser existant**.
1. Sélectionnez **Principal du service**, puis sélectionnez un principal du service.
1. Chargez le certificat (fichier .pfx) associé au principal du service sélectionné, puis indiquez le mot de passe du certificat.

    ![Ajouter un principal du service à un cluster HDInsight](./media/data-lake-store-hdinsight-hadoop-use-portal/hdi.adl.5.png)

1. Sélectionnez **Accès** pour configurer l’accès au dossier.  Reportez-vous à [Configurer les autorisations des fichiers](#configure-file-permissions).

### <a name="set-up-permissions-for-the-service-principal-to-be-owner-on-the-storage-account"></a><a name="configure-serviceprincipal-permissions"></a>Configurer des autorisations pour que le principal du service soit propriétaire du compte de stockage
1. Dans le panneau Access Control (IAM) du compte de stockage, cliquez sur Ajouter une attribution de rôle. 
2. Dans le panneau ajouter une attribution de rôle, sélectionnez le rôle « propriétaire », puis sélectionnez le nom de principal du service et cliquez sur Enregistrer.

### <a name="configure-file-permissions"></a><a name="configure-file-permissions"></a>Configurer les autorisations des fichiers

La configuration diffère selon que le compte est utilisé en tant que compte de stockage par défaut ou compte de stockage supplémentaire :

* Utilisation en tant que stockage par défaut

  * Autorisation au niveau racine du compte Data Lake Storage Gen1
  * Autorisation au niveau racine du stockage de cluster HDInsight. Exemple : dossier __/clusters__ utilisé précédemment dans le didacticiel.

* Utilisation en tant que stockage supplémentaire

  * Autorisation au niveau des dossiers pour lesquels vous avez besoin d’un accès aux fichiers.

Pour affecter des autorisations au niveau racine pour le compte de stockage Data Lake Storage Gen1 :

1. Sur le panneau **Accès à Data Lake Storage Gen1**, sélectionnez **Accès**. Le panneau **Sélectionner des autorisations de fichiers** s’ouvre. Il liste tous les comptes de stockage de votre abonnement.
1. Placez le pointeur de la souris (sans cliquer) sur le nom du compte avec Data Lake Storage Gen1 pour afficher la case à cocher, puis cochez cette case.

    ![Sélectionner les autorisations de fichier](./media/data-lake-store-hdinsight-hadoop-use-portal/hdi.adl.3.png)

   Par défaut, __READ__, __WRITE__ ET __EXECUTE__ sont tous sélectionnés.

1. Cliquez sur **Sélectionner** en bas de la page.
1. Sélectionnez **Exécuter** pour assigner des autorisations.
1. Sélectionnez **Terminé**.

Pour assigner des autorisations au niveau racine du cluster HDInsight :

1. Sur le panneau **Accès à Data Lake Storage Gen1**, sélectionnez **Accès**. Le panneau **Sélectionner des autorisations de fichiers** s’ouvre. Il liste tous les comptes de stockage avec Data Lake Storage Gen1 de votre abonnement.
1. Dans le panneau **Sélectionner des autorisations de fichiers**, sélectionnez le nom du compte de stockage avec Data Lake Storage Gen1 pour afficher son contenu.
1. Sélectionnez la racine de stockage du cluster HDInsight en cochant la case située à gauche du dossier. Comme indiqué sur la capture d’écran précédente, la racine de stockage du cluster est le dossier __/clusters__ que vous avez spécifié lors de la sélection de Data Lake Storage Gen1 comme stockage par défaut.
1. Définissez les autorisations sur le dossier.  Par défaut, les autorisations en lecture, écriture et exécution sont toutes sélectionnées.
1. Cliquez sur **Sélectionner** en bas de la page.
1. Sélectionnez **Exécuter**.
1. Sélectionnez **Terminé**.

Si vous utilisez Data Lake Storage Gen1 en tant que stockage supplémentaire, vous devez assigner des autorisations uniquement pour les dossiers auxquels vous souhaitez accéder à partir du cluster HDInsight. Par exemple, dans la capture d’écran ci-dessous, vous donnez uniquement accès au dossier **mynewfolder** dans un compte de stockage avec Data Lake Storage Gen1.

![Affecter des autorisations de principal de service au cluster HDInsight](./media/data-lake-store-hdinsight-hadoop-use-portal/hdi.adl.3-1.png)

## <a name="verify-cluster-setup"></a><a name="verify-cluster-set-up"></a>Vérifier la configuration du cluster

Une fois que l’installation du cluster est terminée, accédez au panneau du cluster pour vérifier vos résultats en effectuant l’une des opérations suivantes, ou les deux :

* Pour vérifier que le stockage associé au cluster est le compte avec Data Lake Storage Gen1 que vous avez spécifié, sélectionnez **Comptes de stockage** dans le volet gauche.

    ![Vérifier le stockage associé](./media/data-lake-store-hdinsight-hadoop-use-portal/hdi.adl.6-1.png)

* Pour vérifier que le principal du service est correctement associé au cluster HDInsight, sélectionnez **Accès à Data Lake Storage Gen1** dans le volet gauche.

    ![Vérifier le principal de service](./media/data-lake-store-hdinsight-hadoop-use-portal/hdi.adl.6.png)

## <a name="examples"></a>Exemples

Une fois que vous avez configuré le cluster avec Data Lake Storage Gen1 comme stockage, voici quelques exemples de l’utilisation du cluster HDInsight pour analyser les données stockées dans Data Lake Storage Gen1.

### <a name="run-a-hive-query-against-data-in-a-data-lake-storage-gen1-as-primary-storage"></a>Exécuter une requête Hive sur des données stockées dans Data Lake Storage Gen1 (comme stockage principal)

Pour exécuter une requête Hive, utilisez l’interface des vues Hive du portail Ambari. Pour obtenir des instructions sur l'utilisation des vues Hive Ambari, consultez [Utiliser la vue Hive avec Hadoop dans HDInsight](../hdinsight/hadoop/apache-hadoop-use-hive-ambari-view.md).

Lorsque vous utilisez les données dans Data Lake Storage Gen1, vous devez apporter quelques modifications.

Si vous utilisez, par exemple, le cluster que vous avez créé avec Data Lake Storage Gen1 comme stockage principal, le chemin d’accès aux données est le suivant : *adl://<nom_compte_Data_Lake_Storage_Gen1>/azuredatalakestore.net/path/to/file*. La requête Hive permettant de créer une table à partir des exemples de données stockés dans Data Lake Storage Gen1 ressemble à l’instruction suivante :

```console
CREATE EXTERNAL TABLE websitelog (str string) LOCATION 'adl://hdiadlsg1storage.azuredatalakestore.net/clusters/myhdiadlcluster/HdiSamples/HdiSamples/WebsiteLogSampleData/SampleLog/'
```

Descriptions :

* `adl://hdiadlsg1storage.azuredatalakestore.net/` est la racine du compte Data Lake Storage Gen1.
* `/clusters/myhdiadlcluster` est la racine des données du cluster que vous avez spécifiée lors de la création du cluster.
* `/HdiSamples/HdiSamples/WebsiteLogSampleData/SampleLog/` est l’emplacement de l’exemple de fichier que vous utilisez dans la requête.

### <a name="run-a-hive-query-against-data-in-a-data-lake-storage-gen1-as-additional-storage"></a>Exécuter une requête Hive sur des données stockées dans Data Lake Storage Gen1 (comme stockage supplémentaire)

Si le cluster que vous avez créé utilise le Stockage Blob comme stockage par défaut, les exemples de données ne se trouvent pas dans le compte de stockage Data Lake Storage Gen1 qui est utilisé comme stockage supplémentaire. Dans ce cas, commencez par transférer les données du Stockage Blob vers le compte avec Data Lake Storage Gen1, puis exécutez les requêtes selon la procédure décrite ci-dessus.

Pour plus d’informations sur la copie de données à partir du Stockage Blob vers un compte de stockage avec Data Lake Storage Gen1, consultez les articles suivants :

* [Utiliser Distcp pour copier des données entre le Stockage Blob Azure et Data Lake Storage Gen1](data-lake-store-copy-data-wasb-distcp.md)
* [Utiliser AdlCopy pour copier des données entre le Stockage Blob Azure et Data Lake Storage Gen1](data-lake-store-copy-data-azure-storage-blob.md)

### <a name="use-data-lake-storage-gen1-with-a-spark-cluster"></a>Utiliser Data Lake Storage Gen1 avec un cluster Spark

Vous pouvez utiliser un cluster Spark pour exécuter des tâches Spark sur des données stockées dans Data Lake Storage Gen1. Pour plus d’informations, consultez [Utiliser le cluster HDInsight Spark pour analyser les données dans Data Lake Storage Gen1](../hdinsight/spark/apache-spark-use-with-data-lake-store.md).

### <a name="use-data-lake-storage-gen1-in-a-storm-topology"></a>Utiliser Data Lake Storage Gen1 dans une topologie Storm

Vous pouvez utiliser le compte de stockage avec Data Lake Storage Gen1 pour écrire des données à partir d’une topologie Storm. Pour obtenir des instructions sur la réalisation de ce scénario, consultez [Utiliser Azure Data Lake Storage Gen1 avec Apache Storm sur HDInsight](../hdinsight/storm/apache-storm-write-data-lake-store.md).

## <a name="see-also"></a>Voir aussi

* [Utiliser Data Lake Storage Gen1 avec des clusters Azure HDInsight](../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen1.md)
* [PowerShell : créer un cluster HDInsight pour utiliser Data Lake Storage Gen1](data-lake-store-hdinsight-hadoop-use-powershell.md)

[makecert]: /windows-hardware/drivers/devtest/makecert
[pvk2pfx]: /windows-hardware/drivers/devtest/pvk2pfx
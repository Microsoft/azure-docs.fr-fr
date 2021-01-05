---
title: 'Tutoriel : Créer une définition de travail Apache Spark dans Synapse Studio'
description: 'Tutoriel : utiliser Azure Synapse Analytics pour créer des définitions de travaux Spark et les envoyer à un pool Apache Spark pour Azure Synapse Analytics.'
author: Jejiang
ms.author: jejiang
ms.reviewer: jasonh
ms.service: synapse-analytics
ms.topic: tutorial
ms.subservice: spark
ms.date: 10/16/2020
ms.openlocfilehash: b8c7792a09dd86e7d4ac043c572f69fc47ee6e63
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93307178"
---
# <a name="tutorial-create-apache-spark-job-definition-in-synapse-studio"></a>Tutoriel : Créer une définition de travail Apache Spark dans Synapse Studio

Ce tutoriel montre comment utiliser Azure Synapse Studio pour créer des définitions de travaux Apache Spark et les envoyer à un pool Apache Spark serverless.

Ce tutoriel décrit les tâches suivantes :
> [!div class="checklist"]
>
> - Créer une définition de travail Apache Spark pour PySpark (Python)
> - Créer une définition de travail Apache Spark pour Spark(Scala)
> - Créer une définition de travail Apache Spark pour .NET Spark (C#/F#)
> - Envoyer une définition de travail Apache Spark en tant que traitement par lots
> - Ajouter une définition de travail Apache Spark dans le pipeline

## <a name="prerequisites"></a>Prérequis

Avant de commencer le didacticiel, veillez à disposer des éléments suivants :

* Un espace de travail Azure Synapse Analytics. Pour obtenir des instructions, consultez [Créer un espace de travail Azure Synapse Analytics](../../machine-learning/how-to-manage-workspace.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json#create-a-workspace).
* Pool Apache Spark serverless.
* Un compte de stockage ADLS Gen2. Vous devez être le **propriétaire des données Blob du stockage** du système de fichiers ADLS Gen2 que vous souhaitez utiliser. Si ce n’est pas le cas, vous devez ajouter l’autorisation manuellement.
* Si vous ne souhaitez pas utiliser le stockage par défaut de l’espace de travail, liez le compte de stockage ADLS Gen2 nécessaire à Synapse Studio. 

## <a name="create-an-apache-spark-job-definition-for-pyspark-python"></a>Créer une définition de travail Apache Spark pour PySpark (Python)

Dans cette section, vous allez créer une définition de travail Apache Spark pour PySpark (Python).

1. Ouvrez [Azure Synapse Studio](https://web.azuresynapse.net/).

2. Vous pouvez accéder aux [exemples de fichiers pour la création de définitions de travaux Apache Spark](https://github.com/Azure-Samples/Synapse/tree/master/Spark/Python) afin de télécharger des **exemples de fichiers pour python.zip**. Ensuite, vous pouvez décompresser le package puis extraire les fichiers **wordcount.py** et **shakespeare.txt**. 

     ![exemples de fichiers](./media/apache-spark-job-definitions/sample-files.png)

3. Sélectionnez **Données** -> **Liées** -> **Azure Data Lake Storage Gen2**, puis chargez **wordcount.py** et **shakespeare.txt** dans votre système de fichiers ADLS Gen2. 

     ![chargement du fichier python](./media/apache-spark-job-definitions/upload-python-file.png)

4. Sélectionnez le hub **Développer**, sélectionnez l’icône « + », puis sélectionnez **Définition d’un travail Spark** pour créer une définition de travail Spark. 

     ![création d’une définition pour python](./media/apache-spark-job-definitions/create-new-definition.png)

5. Sélectionnez **PySpark (Python)** dans la liste déroulante Langage de la fenêtre principale de la définition de travail Apache Spark.

     ![Sélectionner Python](./media/apache-spark-job-definitions/select-python.png)

6. Renseignez les informations pour la définition de travail Apache Spark. 

     |  Propriété   | Description   |  
     | ----- | ----- |  
     |Nom de la définition de travail| Entrez un nom pour votre définition de travail Apache Spark. Ce nom peut être mis à jour à tout moment jusqu’à ce qu’il soit publié. <br> Exemple : `job definition sample`|
     |Fichier de définition principal| Fichier principal utilisé pour le travail. Sélectionnez un fichier PY à partir de votre stockage. Vous pouvez sélectionner **Charger le fichier** pour charger le fichier sur un compte de stockage. <br> Exemple : `abfss://…/path/to/wordcount.py`|
     |Arguments de ligne de commande| Arguments facultatifs du travail. <br> Exemple : `abfss://…/path/to/shakespeare.txt` `abfss://…/path/to/result` <br> *Remarque : Dans l’exemple de définition de travail, deux arguments sont séparés par un espace.*|
     |Fichiers de référence| Fichiers supplémentaires utilisés en guise de référence dans le fichier de définition principal. Vous pouvez sélectionner **Charger le fichier** pour charger le fichier sur un compte de stockage. |
     |Pool Spark| Le travail sera envoyé au pool Apache Spark sélectionné.|
     |Version de Spark| Version d’Apache Spark exécutée par le pool Apache Spark.|
     |Exécuteurs| Nombre d’exécuteurs à attribuer dans le pool Apache Spark spécifié pour le travail.|
     |Taille de l’exécuteur| Nombre de cœurs et mémoire à utiliser pour les exécuteurs dans le pool Apache Spark spécifié du travail.|  
     |Taille du pilote| Nombre de cœurs et mémoire à utiliser pour le pilote dans le pool Apache Spark spécifié du travail.|

     ![Définir les valeurs de la définition de travail Spark pour Python](./media/apache-spark-job-definitions/create-py-definition.png)

7. Sélectionnez **Publier** pour enregistrer la définition de travail Apache Spark.

     ![publication de la définition py](./media/apache-spark-job-definitions/publish-py-definition.png)

## <a name="create-an-apache-spark-job-definition-for-apache-sparkscala"></a>Créer une définition de travail Apache Spark pour Apache Spark(Scala)

Dans cette section, vous allez créer une définition de travail Apache Spark pour Apache Spark(Scala).

 1. Ouvrez [Azure Synapse Studio](https://web.azuresynapse.net/).

 2. Vous pouvez accéder aux [exemples de fichiers pour la création de définitions de travaux Apache Spark](https://github.com/Azure-Samples/Synapse/tree/master/Spark/Scala) afin de télécharger des **exemples de fichiers pour scala.zip**. Ensuite, vous pouvez décompresser le package puis extraire les fichiers **wordcount.jar** et **shakespeare.txt**. 
 
     ![Exemples de fichiers Scala](./media/apache-spark-job-definitions/sample-files-scala.png)

 3. Sélectionnez **Données** -> **Liées** -> **Azure Data Lake Storage Gen2**, puis chargez **wordcount.jar** et **shakespeare.txt** dans votre système de fichiers ADLS Gen2.
 
     ![préparation de la structure scala](./media/apache-spark-job-definitions/prepare-scala-structure.png)

 4. Sélectionnez le hub **Développer**, sélectionnez l’icône « + », puis sélectionnez **Définition d’un travail Spark** pour créer une définition de travail Spark. L’exemple d’image est le même que celui de l’étape 4 de la section **Créer une définition de travail Apache Spark (Python) pour PySpark**.

 5. Sélectionnez **Spark(Scala)** dans la liste déroulante Langage de la fenêtre principale de la définition de travail Apache Spark.

     ![Sélectionner Scala](./media/apache-spark-job-definitions/select-scala.png)

 6. Renseignez les informations pour la définition de travail Apache Spark. Vous pouvez copier les exemples d’informations.

     |  Propriété   | Description   |  
     | ----- | ----- |  
     |Nom de la définition de travail| Entrez un nom pour votre définition de travail Apache Spark. Ce nom peut être mis à jour à tout moment jusqu’à ce qu’il soit publié. <br> Exemple : `scala`|
     |Fichier de définition principal| Fichier principal utilisé pour le travail. Sélectionnez un fichier JAR à partir de votre stockage. Vous pouvez sélectionner **Charger le fichier** pour charger le fichier sur un compte de stockage. <br> Exemple : `abfss://…/path/to/wordcount.jar`|
     |Main class name| Identificateur complet ou classe principale qui se trouve dans le fichier de définition principal. <br> Exemple : `WordCount`|
     |Arguments de ligne de commande| Arguments facultatifs du travail. <br> Exemple : `abfss://…/path/to/shakespeare.txt` `abfss://…/path/to/result` <br> *Remarque : Dans l’exemple de définition de travail, deux arguments sont séparés par un espace.* |
     |Fichiers de référence| Fichiers supplémentaires utilisés en guise de référence dans le fichier de définition principal. Vous pouvez sélectionner **Charger le fichier** pour charger le fichier sur un compte de stockage.|
     |Pool Spark| Le travail sera envoyé au pool Apache Spark sélectionné.|
     |Version de Spark| Version d’Apache Spark exécutée par le pool Apache Spark.|
     |Exécuteurs| Nombre d’exécuteurs à attribuer dans le pool Apache Spark spécifié pour le travail.|  
     |Taille de l’exécuteur| Nombre de cœurs et mémoire à utiliser pour les exécuteurs dans le pool Apache Spark spécifié du travail.|
     |Taille du pilote| Nombre de cœurs et mémoire à utiliser pour le pilote dans le pool Apache Spark spécifié du travail.|

     ![Définir les valeurs de la définition de travail Spark pour Scala](./media/apache-spark-job-definitions/create-scala-definition.png)

 7. Sélectionnez **Publier** pour enregistrer la définition de travail Apache Spark.

      ![publication de la définition scala](./media/apache-spark-job-definitions/publish-scala-definition.png)

## <a name="create-an-apache-spark-job-definition-for-net-sparkcf"></a>Créer une définition de travail Apache Spark pour .NET Spark (C#/F#)

Dans cette section, vous créez une définition de travail Apache Spark pour .NET Spark (C#/F#).
 1. Ouvrez [Azure Synapse Studio](https://web.azuresynapse.net/).

 2. Vous pouvez accéder aux [exemples de fichiers pour la création de définitions de travaux Apache Spark](https://github.com/Azure-Samples/Synapse/tree/master/Spark/DotNET) afin de télécharger des **exemples de fichiers pour dotnet.zip**. Ensuite, vous pouvez décompresser le package puis extraire les fichiers **wordcount.zip** et **shakespeare.txt**. 

     ![Exemple dotnet](./media/apache-spark-job-definitions/sample-dotnet.png)

 3. Sélectionnez **Données** -> **Liées** -> **Azure Data Lake Storage Gen2**, puis chargez **wordcount.zip** et **shakespeare.txt** dans votre système de fichiers ADLS Gen2.
 
     ![préparation de la structure dotnet](./media/apache-spark-job-definitions/prepare-dotnet-structure.png)

 4. Sélectionnez le hub **Développer**, sélectionnez l’icône « + », puis sélectionnez **Définition d’un travail Spark** pour créer une définition de travail Spark. L’exemple d’image est le même que celui de l’étape 4 de la section **Créer une définition de travail Apache Spark (Python) pour PySpark**.

 5. Sélectionnez **.NET Spark(C#/F#)** dans la liste déroulante Langage de la fenêtre principale Définition de travail Apache Spark.

     ![Sélectionner dotnet](./media/apache-spark-job-definitions/select-dotnet.png)

 6. Renseignez les informations pour la définition de travail Apache Spark. Vous pouvez copier les exemples d’informations.
    
     |  Propriété   | Description   |  
     | ----- | ----- |  
     |Nom de la définition de travail| Entrez un nom pour votre définition de travail Apache Spark. Ce nom peut être mis à jour à tout moment jusqu’à ce qu’il soit publié. <br> Exemple : `dotnet`|
     |Fichier de définition principal| Fichier principal utilisé pour le travail. Sélectionnez un fichier ZIP contenant votre application .NET pour Apache Spark (c’est-à-dire le fichier exécutable principal, les DLL contenant les fonctions définies par l’utilisateur et d’autres fichiers nécessaires) dans votre stockage. Vous pouvez sélectionner **Charger le fichier** pour charger le fichier sur un compte de stockage. <br> Exemple : `abfss://…/path/to/wordcount.zip`|
     |Fichier exécutable principal| Fichier exécutable principal dans le fichier ZIP de définition principal. <br> Exemple : `WordCount`|
     |Arguments de ligne de commande| Arguments facultatifs du travail. <br> Exemple : `abfss://…/path/to/shakespeare.txt` `abfss://…/path/to/result` <br> *Remarque : Dans l’exemple de définition de travail, deux arguments sont séparés par un espace.* |
     |Fichiers de référence| Fichiers supplémentaires nécessaires aux nœuds Worker pour l’exécution de l’application .NET pour Apache Spark qui n’est pas incluse dans le fichier ZIP de définition principal (autrement dit, les fichiers jar dépendants, les DLL supplémentaires de fonctions définies par l’utilisateur et d’autres fichiers de configuration). Vous pouvez sélectionner **Charger le fichier** pour charger le fichier sur un compte de stockage.|
     |Pool Spark| Le travail sera envoyé au pool Apache Spark sélectionné.|
     |Version de Spark| Version d’Apache Spark exécutée par le pool Apache Spark.|
     |Exécuteurs| Nombre d’exécuteurs à attribuer dans le pool Apache Spark spécifié pour le travail.|  
     |Taille de l’exécuteur| Nombre de cœurs et mémoire à utiliser pour les exécuteurs dans le pool Apache Spark spécifié du travail.|
     |Taille du pilote| Nombre de cœurs et mémoire à utiliser pour le pilote dans le pool Apache Spark spécifié du travail.|

     ![Définir les valeurs de la définition de travail Spark pour dotnet](./media/apache-spark-job-definitions/create-dotnet-definition.png)

 7. Sélectionnez **Publier** pour enregistrer la définition de travail Apache Spark.

      ![publication de la définition dotnet](./media/apache-spark-job-definitions/publish-dotnet-definition.png)

## <a name="submit-an-apache-spark-job-definition-as-a-batch-job"></a>Envoyer une définition de travail Apache Spark en tant que traitement par lots

Après avoir créé une définition de travail Apache Spark, vous pouvez l’envoyer à un pool Apache Spark. Vérifiez que vous êtes bien le **propriétaire des données Blob du stockage** du système de fichiers ADLS Gen2 que vous souhaitez utiliser. Si ce n’est pas le cas, vous devez ajouter l’autorisation manuellement.

### <a name="scenario-1-submit-apache-spark-job-definition"></a>Scénario 1 : Envoyer une définition de travail Apache Spark
 1. Ouvrez une fenêtre de définition de travail Apache Spark en sélectionnant la définition concernée.

      ![Ouvrez la définition de travail Spark à envoyer ](./media/apache-spark-job-definitions/open-spark-definition.png)

 2. Sélectionnez le bouton **Envoyer** pour envoyer votre projet au pool Apache Spark sélectionné. Vous pouvez sélectionner l’**URL de supervision Spark** pour afficher les informations LogQuery de l’application Apache Spark.

    ![Sélectionnez le bouton Envoyer pour envoyer la définition de travail Spark](./media/apache-spark-job-definitions/submit-spark-definition.png)

    ![Boîte de dialogue Spark Submission (Envoi Spark)](./media/apache-spark-job-definitions/submit-definition-result.png)

### <a name="scenario-2-view-apache-spark-job-running-progress"></a>Scénario 2 : Afficher la progression de l’exécution du travail Apache Spark

 1. Sélectionnez **Superviser**, puis sélectionnez l’option **Applications Apache Spark**. Vous pouvez trouver l’application Apache Spark envoyée.

     ![Afficher l’application Spark](./media/apache-spark-job-definitions/view-spark-application.png)

 2. Ensuite, sélectionnez une application Apache Spark pour afficher la fenêtre du travail **SparkJobDefinition**. Vous pourrez y voir la progression de l’exécution du travail.
     
     ![Affichez les informations LogQuery de l’application Spark](./media/apache-spark-job-definitions/view-job-log-query.png)

### <a name="scenario-3-check-output-file"></a>Scénario 3 : Vérifier le fichier de sortie

 1. Sélectionnez **Données** -> **Liées** -> **Azure Data Lake Storage Gen2** (hozhaobdbj), puis ouvrez le dossier **result** créé précédemment. Vous pouvez accéder au dossier result pour voir si la sortie a été générée.

     ![Affichez le fichier de sortie](./media/apache-spark-job-definitions/view-output-file.png)

## <a name="add-an-apache-spark-job-definition-into-pipeline"></a>Ajouter une définition de travail Apache Spark dans le pipeline

Dans cette section, vous allez ajouter une définition de travail Apache Spark dans le pipeline.

 1. Ouvrez une définition de travail Apache Spark existante.

 2. Sélectionnez l’icône en haut à droite de la définition de travail Apache Spark, puis choisissez **Pipeline existant** ou **Nouveau pipeline**. Vous pouvez consulter la page Pipeline pour plus d’informations.

     ![add to pipeline1](./media/apache-spark-job-definitions/add-to-pipeline01.png)

     ![add to pipeline2](./media/apache-spark-job-definitions/add-to-pipeline02.png)

## <a name="next-steps"></a>Étapes suivantes

À présent, vous pouvez utiliser Azure Synapse Studio pour créer des jeux de données Power BI et gérer des données Power BI. Pour en savoir plus, consultez l’article [Liaison d’un espace de travail Power BI à un espace de travail Synapse](../quickstart-power-bi.md). 


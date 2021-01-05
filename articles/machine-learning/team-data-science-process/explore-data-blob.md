---
title: Explorer les données dans le stockage d’objets blob Azure avec Pandas - Team Data Science Process
description: Comment explorer les données stockées dans un conteneur d’objets blob Azure à l’aide du package Python Pandas.
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: 3d80ff9f5f96bbc642aed6598c86cb682d606695
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94410672"
---
# <a name="explore-data-in-azure-blob-storage-with-pandas"></a>Explorer les données dans le stockage d’objets blob Azure avec Pandas

Cet article explique comment explorer les données stockées dans un conteneur d’objets blob Azure à l’aide du package Python [pandas](https://pandas.pydata.org/).

Cette tâche est une étape du [processus Team Data Science Process](overview.md).

## <a name="prerequisites"></a>Prérequis
Cet article suppose que vous avez :

* Créé un compte de stockage Azure. Pour des instructions, voir [Créer un compte Stockage Azure](../../storage/common/storage-account-create.md).
* Stocké vos données dans un compte de stockage d’objets blob Azure. Si vous avez besoin d’aide, consultez la page [Transfert de données vers et à partir d’Azure Storage](../../storage/common/storage-choose-data-transfer-solution.md)

## <a name="load-the-data-into-a-pandas-dataframe"></a>Chargement des données dans une trame de données Pandas
Pour explorer et manipuler un jeu de données, celui-ci doit d’abord être téléchargé depuis la source Blob vers un fichier local qui peut ensuite être chargé dans une trame de données Pandas. Voici les étapes à suivre pour cette procédure :

1. Téléchargez les données à partir du blob Azure avec l’exemple de code Python suivant en utilisant le service BLOB. Remplacez la variable dans le code ci-dessous par vos propres valeurs :

    ```python
    from azure.storage.blob import BlockBlobService
    import pandas as pd
    import tables

    STORAGEACCOUNTNAME= <storage_account_name>
    STORAGEACCOUNTKEY= <storage_account_key>
    LOCALFILENAME= <local_file_name>
    CONTAINERNAME= <container_name>
    BLOBNAME= <blob_name>

    #download from blob
    t1=time.time()
    blob_service=BlockBlobService(account_name=STORAGEACCOUNTNAME,account_key=STORAGEACCOUNTKEY)
    blob_service.get_blob_to_path(CONTAINERNAME,BLOBNAME,LOCALFILENAME)
    t2=time.time()
    print(("It takes %s seconds to download "+BLOBNAME) % (t2 - t1))
    ```

1. Lisez les données du fichier téléchargé dans une table Pandas.

    ```python
    # LOCALFILE is the file path
    dataframe_blobdata = pd.read_csv(LOCALFILENAME)
    ```

Vous êtes maintenant prêt à explorer les données et à générer des fonctionnalités sur cet ensemble de données.

## <a name="examples-of-data-exploration-using-pandas"></a><a name="blob-dataexploration"></a>Exemples d’exploration de données à l’aide de Pandas
Voici quelques méthodes pour explorer des données à l’aide de Pandas :

1. Vérifiez le **nombre de lignes et de colonnes**

    ```python
    print('the size of the data is: %d rows and  %d columns' % dataframe_blobdata.shape)
    ```

1. **Vérifiez** les premières ou dernières **lignes** de l’ensemble de données ci-dessous :

    ```python
    dataframe_blobdata.head(10)

    dataframe_blobdata.tail(10)
    ```

1. Vérifiez le **type de données** dans lequel chaque colonne a été importée, à l’aide du code suivant :

    ```python
    for col in dataframe_blobdata.columns:
        print(dataframe_blobdata[col].name, ':\t', dataframe_blobdata[col].dtype)
    ```

1. Vérifiez les **statistiques de base** des colonnes dans l’ensemble de données, comme indiqué ci-dessous :

    ```python
    dataframe_blobdata.describe()
    ```

1. Regardez le nombre d’entrées pour chaque valeur de colonne, comme indiqué ci-dessous :

    ```python
    dataframe_blobdata['<column_name>'].value_counts()
    ```

1. **valeurs manquantes** par rapport au nombre réel d’entrées dans chaque colonne, à l’aide du code suivant :

    ```python
    miss_num = dataframe_blobdata.shape[0] - dataframe_blobdata.count()
    print(miss_num)
    ```

1. Si des **valeurs sont manquantes** dans une colonne spécifique, vous pouvez les supprimer comme suit :

    ```python
    dataframe_blobdata_noNA = dataframe_blobdata.dropna()
    dataframe_blobdata_noNA.shape
    ```

    L’autre solution pour remplacer les valeurs manquantes consiste à utiliser la fonction mode :

    ```python
    dataframe_blobdata_mode = dataframe_blobdata.fillna(
        {'<column_name>': dataframe_blobdata['<column_name>'].mode()[0]})
    ```

1. Créez un **histogramme** à l’aide d’un nombre variable de compartiments pour tracer la distribution d’une variable :

    ```python
    dataframe_blobdata['<column_name>'].value_counts().plot(kind='bar')

    np.log(dataframe_blobdata['<column_name>']+1).hist(bins=50)
    ```

1. Examinez les **corrélations** entre les variables à l’aide d’un nuage de points ou de la fonction de corrélation intégrée :

    ```python
    # relationship between column_a and column_b using scatter plot
    plt.scatter(dataframe_blobdata['<column_a>'], dataframe_blobdata['<column_b>'])

    # correlation between column_a and column_b
    dataframe_blobdata[['<column_a>', '<column_b>']].corr()
    ```

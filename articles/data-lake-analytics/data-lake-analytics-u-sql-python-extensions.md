---
title: Étendre des scripts U-SQL à l’aide de Python dans Azure Data Lake Analytics
description: Découvrez comment exécuter du code Python dans des scripts U-SQL à l’aide de Azure Data Lake Analytics.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 06/20/2017
ms.custom: devx-track-python
ms.openlocfilehash: d81e529d7073e8200c5a1d1fa8f51504b1dc9259
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92218440"
---
# <a name="extend-u-sql-scripts-with-python-code-in-azure-data-lake-analytics"></a>Étendre des scripts U-SQL à l’aide de code Python dans Azure Data Lake Analytics

## <a name="prerequisites"></a>Prérequis

Avant de commencer, vérifiez que les extensions Python sont installées dans votre compte Azure Data Lake Analytics.

* Dans le portail Azure, accédez à votre compte Data Lake Analytics.
* Dans le menu de gauche, sous **Démarrage**, cliquez sur **Exemples de scripts**
* Cliquez sur **Installer des extensions U-SQL**, puis sur **OK**.

## <a name="overview"></a>Vue d’ensemble

Les extensions de Python pour U-SQL permettent aux développeurs d’effectuer une exécution parallèle massive de code Python. L'exemple suivant illustre les étapes de base :

* Utilisation de l’instruction `REFERENCE ASSEMBLY` pour activer les extensions Python pour le script U-SQL
* Utilisation de l’opération `REDUCE` pour partitionner les données d’entrée sur une clé
* Les extensions de Python pour U-SQL comprennent un réducteur intégré (`Extension.Python.Reducer`) qui exécute le code Python sur chaque vertex affecté au réducteur
* Le script U-SQL contient le code Python incorporé qui a une fonction appelée `usqlml_main` qui accepte un tableau de données Pandas en tant qu’entrée et retourne un tableau de données Pandas en tant que sortie.

```usql
REFERENCE ASSEMBLY [ExtPython];
DECLARE @myScript = @"
def get_mentions(tweet):
    return ';'.join( ( w[1:] for w in tweet.split() if w[0]=='@' ) )
def usqlml_main(df):
    del df['time']
    del df['author']
    df['mentions'] = df.tweet.apply(get_mentions)
    del df['tweet']
    return df
";
@t  =
    SELECT * FROM
       (VALUES
           ("D1","T1","A1","@foo Hello World @bar"),
           ("D2","T2","A2","@baz Hello World @beer")
       ) AS date, time, author, tweet );
@m  =
    REDUCE @t ON date
    PRODUCE date string, mentions string
    USING new Extension.Python.Reducer(pyScript:@myScript);
OUTPUT @m
    TO "/tweetmentions.csv"
    USING Outputters.Csv();
```

## <a name="how-python-integrates-with-u-sql"></a>Intégration de Python à U-SQL

### <a name="datatypes"></a>Types de données

* Les colonnes numériques et de chaîne de U-SQL sont converties telles quelles-entre Pandas et U-SQL
* Les valeurs null de U-SQL sont converties en valeurs Pandas `NA` et à vice versa

### <a name="schemas"></a>Schémas

* Les vecteurs d’index dans Pandas ne sont pas pris en charge dans U-SQL. Tous les tableaux de données d’entrée dans la fonction Python ont toujours un index numérique de 64 bits compris entre 0 et le nombre de lignes moins 1.
* Les jeux de données U-SQL ne peut pas avoir de noms de colonnes dupliqués
* Les noms de colonnes de jeux de données U-SQL qui ne sont pas des chaînes.

### <a name="python-versions"></a>Versions de Python

Seul Python 3.5.1 (compilé pour Windows) est pris en charge.

### <a name="standard-python-modules"></a>Modules Python standard

Tous les modules Python standard sont inclus.

### <a name="additional-python-modules"></a>Modules Python supplémentaires

Outre les bibliothèques Python standard, plusieurs bibliothèques python couramment utilisées sont incluses :

* pandas
* numpy
* numexpr

### <a name="exception-messages"></a>Messages d’exception

Actuellement, une exception dans le code Python apparaît comme un échec de vertex générique. À l’avenir, les messages d’erreur de tâches U-SQL afficheront le message d’exception Python.

### <a name="input-and-output-size-limitations"></a>Limitations de taille d’entrée et de sortie

Chaque vertex possède une quantité limitée de mémoire qui lui est assignée. Actuellement, cette limite est de 6 Go pour une mise à jour automatique. Étant donné que les tableaux de données d’entrée et de sortie doivent exister dans la mémoire dans le code Python, la taille totale de l’entrée et de la sortie ne peut pas dépasser 6 Go.

## <a name="next-steps"></a>Étapes suivantes

* [Vue d'ensemble de Microsoft Azure Data Lake Analytics](data-lake-analytics-overview.md)
* [Développer des scripts de U-SQL à l’aide d’outils Data Lake Tools pour Visual Studio](data-lake-analytics-data-lake-tools-get-started.md)
* [Utilisation des fonctions U-SQL dans les travaux Analytique Data Lake Azure](./data-lake-analytics-u-sql-get-started.md)
* [Utiliser Azure Data Lake Tools pour Visual Studio Code](data-lake-analytics-data-lake-tools-for-vscode.md)
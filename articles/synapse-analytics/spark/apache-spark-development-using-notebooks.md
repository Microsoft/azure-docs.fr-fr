---
title: Notebooks Synapse Studio
description: Cet article explique comment créer et développer des blocs-notes Azure Synapse Studio pour la préparation et la visualisation de données.
services: synapse analytics
author: ruixinxu
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: spark
ms.date: 10/19/2020
ms.author: ruxu
ms.reviewer: ''
ms.custom: devx-track-python
ms.openlocfilehash: bfb822481e00e84fa17bd4b1fcb58c1d4b89a124
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96450860"
---
# <a name="create-develop-and-maintain-synapse-studio-notebooks-in-azure-synapse-analytics"></a>Créer, développer et tenir à jour des notebooks Synapse Studio dans Azure Synapse Analytics

Un notebook Synapse Studio est une interface web permettant de créer des fichiers contenant du code, des visualisations et du texte descriptif en direct. Les blocs-notes constituent un bon endroit où valider des idées et effectuer des expérimentations rapides pour extraire des insights de vos données. Les blocs-notes sont également largement utilisés pour la préparation et la visualisation de données, l’apprentissage automatique et d’autres scénarios en lien avec le Big Data.

Un bloc-notes Azure Synapse Studio permet d’effectuer les opérations suivantes :

* Commencer à travailler sans le moindre effort de configuration.
* Sécuriser les données avec des fonctionnalités de sécurité d’entreprise intégrées.
* Analyser des données dans des formats bruts (CSV, txt, JSON, etc.), des formats de fichiers traités (Parquet, Delta Lake, ORC, etc.) et des fichiers de données tabulaires SQL sur Spark et SQL.
* Être productif grâce à des fonctionnalités de création améliorées et à la visualisation de données intégrée.

Cet article explique comment utiliser des blocs-notes dans Azure Synapse Studio.

## <a name="preview-of-the-new-notebook-experience"></a>Aperçu de la nouvelle expérience de notebook
L’équipe de Synapse a introduit le nouveau composant pour notebooks dans Synapse Studio afin d’offrir une expérience cohérente aux clients de Microsoft et de maximiser la détectabilité, la productivité, le partage et la collaboration. La nouvelle expérience du notebook est prête à être présentée en préversion. Cochez le bouton **Fonctionnalités d’évaluation** dans la barre d’outils du notebook pour l’activer. Le tableau ci-dessous capture la comparaison des fonctionnalités entre un notebook existant (appelé « notebook classique ») et le nouvel disponible en préversion.  

|Fonctionnalité|Notebook classique|Notebook en préversion|
|--|--|--|
|%run| Non pris en charge | &#9745;|
|%history| Non pris en charge |&#9745;
|%load| Non pris en charge |&#9745;|
|%%html| Non pris en charge |&#9745;|
|Glisser-déposer pour déplacer une cellule| Non pris en charge |&#9745;|
|Sortie Display() persistante|&#9745;| Non disponible |
|Annuler tout| &#9745;| Non disponible|
|Exécuter toutes les cellules ci-dessus|&#9745;| Non disponible |
|Exécuter toutes les cellules ci-dessous|&#9745;| Non disponible |
|Mettre en forme une cellule de texte avec des boutons de barre d’outils|&#9745;| Non disponible |
|Annuler l’opération sur cellule| &#9745;| Non disponible |


## <a name="create-a-notebook"></a>Créer un notebook

Il existe deux façons de créer un bloc-notes. Vous pouvez créer un bloc-notes ou en importer un dans un espace de travail Azure Synapse à partir de l’**Explorateur d’objets**. Les blocs-notes Azure Synapse Studio peuvent reconnaître des fichiers IPYNB de bloc-notes Jupyter standard.

![créer un notebook d’importation](./media/apache-spark-development-using-notebooks/synapse-create-import-notebook-2.png)

## <a name="develop-notebooks"></a>Développer des blocs-notes

Les blocs-notes sont constitués de cellules qui sont des blocs individuels de code ou de texte qui peuvent être exécutés de façon indépendante ou en tant que groupe.

### <a name="add-a-cell"></a>Ajouter une cellule

Il existe plusieurs façons d’ajouter une cellule à un bloc-notes.

# <a name="classical-notebook"></a>[Notebook classique](#tab/classical)

1. Développez le bouton supérieur gauche **+ Cellule**, puis sélectionnez **Ajouter une cellule de code** ou **Ajouter une cellule de texte**.

    ![ajouter-cellule-avec-bouton-cellule](./media/apache-spark-development-using-notebooks/synapse-add-cell-1.png)

2. Pointez sur l’espace entre deux cellules, puis sélectionnez **Ajouter du code** ou **Ajouter du texte**.

    ![ajouter-cellule-entre-espace](./media/apache-spark-development-using-notebooks/synapse-add-cell-2.png)

3. Utilisez les [touches de raccourci en mode de commande](#shortcut-keys-under-command-mode). Appuyez sur **A** pour insérer une cellule au-dessus de la cellule active. Appuyez sur **B** pour insérer une cellule en dessous de la cellule active.


# <a name="preview-notebook"></a>[Notebook en préversion](#tab/preview)

1. Développez le bouton supérieur gauche **+ Cellule**, puis sélectionnez **Cellule de code** ou **Cellule Markdown**.
    ![add-azure-notebook-cell-with-cell-button](./media/apache-spark-development-using-notebooks/synapse-azure-notebook-add-cell-1.png)
2. Sélectionnez le signe plus (+) au début d’une cellule et sélectionnez **Cellule de code** ou **Cellule Markdown**.

    ![add-azure-notebook-cell-between-space](./media/apache-spark-development-using-notebooks/synapse-azure-notebook-add-cell-2.png)

3. Utilisez les [touches de raccourci aznb en mode de commande](#shortcut-keys-under-command-mode). Appuyez sur **A** pour insérer une cellule au-dessus de la cellule active. Appuyez sur **B** pour insérer une cellule en dessous de la cellule active.

---

### <a name="set-a-primary-language"></a>Définir un langage principal

Les notebooks Azure Synapse Studio prennent en charge quatre langages Apache Spark :

* pySpark (Python)
* Spark (Scala)
* SparkSQL
* .NET pour Apache Spark (C#)

Vous pouvez définir le langage principal des nouvelles cellules ajoutées dans la liste déroulante de la barre de commandes supérieure.

   ![langage-Synapse-par-défaut](./media/apache-spark-development-using-notebooks/synapse-default-language.png)

### <a name="use-multiple-languages"></a>Utiliser plusieurs langages

Vous pouvez utiliser plusieurs langages dans un même bloc-notes en spécifiant la commande magic du langage approprié au début d’une cellule. Le tableau suivant répertorie les commandes magic pour basculer les langages des cellules.

|Commande magic |Langage | Description |  
|---|------|-----|
|%%pyspark| Python | Exécuter une requête **Python** sur du contexte Spark.  |
|%%spark| Scala | Exécuter une requête **Scala** sur du contexte Spark.  |  
|%%sql| SparkSQL | Exécuter une requête **SparkSQL** sur du contexte Spark.  |
|%%csharp | .NET pour Spark C# | Exécutez une requête **.NET pour Spark C#** sur du contexte Spark. |

L’image suivante illustre la façon d’écrire une requête PySpark avec la commande magic **%%PySpark**, ou une requête SparkSQL avec la commande magic **%%sql** dans un bloc-notes **Spark(Scala)** . Notez que le langage principal du notebook est défini sur pySpark.

   ![Synapse Commandes magic Spark](./media/apache-spark-development-using-notebooks/synapse-spark-magics.png)

### <a name="use-temp-tables-to-reference-data-across-languages"></a>Utiliser des tables temporaires pour référencer des données dans plusieurs langages

Vous ne pouvez pas référencer des données ou variables directement dans différents langages dans un bloc-notes Synapse Studio. Dans Spark, une table temporaire peut être référencée dans plusieurs langages. Voici un exemple de lecture d’une tramedonnées `Scala` en `PySpark` et `SparkSQL` en utilisant une table temporaire Spark comme solution de contournement.

1. Dans la cellule 1, lisez une tramedonnées à partir du connecteur de pool SQL en utilisant Scala, puis créez une table temporaire.

   ```scala
   %%scala
   val scalaDataFrame = spark.read.sqlanalytics("mySQLPoolDatabase.dbo.mySQLPoolTable")
   scalaDataFrame.createOrReplaceTempView( "mydataframetable" )
   ```

2. Dans la cellule 2, interrogez les données en utilisant Spark SQL.
   
   ```sql
   %%sql
   SELECT * FROM mydataframetable
   ```

3. Dans la cellule 3, utilisez les données dans PySpark.

   ```python
   %%pyspark
   myNewPythonDataFrame = spark.sql("SELECT * FROM mydataframetable")
   ```

### <a name="ide-style-intellisense"></a>IntelliSense de style IDE

Des blocs-notes Azure Synapse Studio sont intégrés avec l’éditeur de Monaco pour intégrer IntelliSense de style IDE à l’éditeur de cellule. Une mise en évidence de la syntaxe, un marqueur d’erreurs et des saisies semi-automatiques de code vous aident à écrire le code et à identifier les problèmes plus rapidement.

Les fonctionnalités IntelliSense sont à des niveaux de maturité différents pour les différents langages. Utilisez le tableau suivant pour voir ce qui est pris en charge.

|Languages| Mise en évidence de la syntaxe | Marqueur d’erreur de syntaxe  | Saisie semi-automatique de code de syntaxe | Saisie semi-automatique de code variable| Saisie semi-automatique de code de fonction système| Saisie semi-automatique de code de fonction utilisateur| Mise en retrait intelligente | Pliage de code|
|--|--|--|--|--|--|--|--|--|
|PySpark (Python)|Oui|Oui|Oui|Oui|Oui|Oui|Oui|Oui|
|Spark (Scala)|Oui|Oui|Oui|Oui|-|-|-|Oui|
|SparkSQL|Oui|Oui|-|-|-|-|-|-|
|.NET pour Spark (C#)|Oui|-|-|-|-|-|-|-|

### <a name="format-text-cell-with-toolbar-buttons"></a>Mettre en forme une cellule de texte avec des boutons de barre d’outils

# <a name="classical-notebook"></a>[Notebook classique](#tab/classical)

Vous pouvez utiliser les boutons de mise en forme dans la barre d’outils des cellules de texte pour effectuer des actions de markdown (démarquage) courantes. Celles-ci incluent la mise en gras et en italique de texte, l’insertion d’extraits de code, l’insertion de liste non triée, l’insertion de liste triée et l’insertion d’image à partir d’une URL.

  ![Synapse Barre d’outils de la cellule de texte](./media/apache-spark-development-using-notebooks/synapse-text-cell-toolbar.png)

# <a name="preview-notebook"></a>[Notebook en préversion](#tab/preview)

La barre d’outils du bouton format n’est pas encore disponible pour l’expérience de notebook en préversion. 

---

### <a name="undo-cell-operations"></a>Annuler des opérations sur cellule

# <a name="classical-notebook"></a>[Notebook classique](#tab/classical)

Sélectionnez le bouton **Annuler** ou appuyez sur **Ctrl + Z** pour révoquer l’opération de cellule la plus récente. Vous pouvez désormais annuler jusqu’aux 20 dernières actions sur cellule. 

   ![Synapse Annuler les cellules](./media/apache-spark-development-using-notebooks/synapse-undo-cells.png)
# <a name="preview-notebook"></a>[Notebook en préversion](#tab/preview)

L’annulation de l’opération sur cellule n’est pas encore disponible pour l’expérience de notebook en préversion. 

---

### <a name="move-a-cell"></a>Déplacer une cellule

# <a name="classical-notebook"></a>[Notebook classique](#tab/classical)

Sélectionnez les points de suspension (...) pour accéder au menu d’actions sur cellule supplémentaires tout à fait à droite. Sélectionnez ensuite **Déplacer la cellule vers le haut** ou **Déplacer la cellule vers le bas** pour déplacer la cellule active. 

Vous pouvez également utiliser des [touches de raccourci en mode de commande](#shortcut-keys-under-command-mode). Appuyez sur **Ctrl + Alt + ↑** pour déplacer la cellule active vers le haut. Appuyez sur **Ctrl + Alt + ↓** pour déplacer la cellule active vers le bas.

   ![déplacer-un-cellule](./media/apache-spark-development-using-notebooks/synapse-move-cells.png)

# <a name="preview-notebook"></a>[Notebook en préversion](#tab/preview)

Cliquez sur le côté gauche d’une cellule et faites-la glisser vers la position de votre choix. 
    ![Synapse déplacer des cellules](./media/apache-spark-development-using-notebooks/synapse-azure-notebook-drag-drop-cell.gif)

---

### <a name="delete-a-cell"></a>Supprimer une cellule

# <a name="classical-notebook"></a>[Notebook classique](#tab/classical)

Pour supprimer une cellule, sélectionnez les points de suspension (...) pour accéder au menu d’actions sur cellule supplémentaires tout à fait à droite, puis choisissez **Supprimer la cellule**. 

Vous pouvez également utiliser des [touches de raccourci en mode de commande](#shortcut-keys-under-command-mode). Appuyez sur **D, D** pour supprimer la cellule active.
  
   ![supprimer-une-cellule](./media/apache-spark-development-using-notebooks/synapse-delete-cell.png)

# <a name="preview-notebook"></a>[Notebook en préversion](#tab/preview)

Pour supprimer une cellule, sélectionnez le bouton Supprimer à droite de la cellule. 

Vous pouvez également utiliser des [touches de raccourci en mode de commande](#shortcut-keys-under-command-mode). Appuyez sur **Maj + D** pour supprimer la cellule active. 

   ![azure-notebook-delete-a-cell](./media/apache-spark-development-using-notebooks/synapse-azure-notebook-delete-cell.png)

---

### <a name="collapse-a-cell-input"></a>Réduire une entrée de cellule

# <a name="classical-notebook"></a>[Notebook classique](#tab/classical)

Sélectionnez le bouton fléché en bas de la cellule active pour la réduire. Pour la développer, sélectionnez le bouton fléché quand elle est réduite.

   ![réduire-cellule-entrée](./media/apache-spark-development-using-notebooks/synapse-collapse-cell-input.gif)

# <a name="preview-notebook"></a>[Notebook en préversion](#tab/preview)

Sélectionnez le bouton de sélection (…) **Plus de commandes** dans la barre d’outils de la cellule et **Entrée** pour réduire l’entrée de la cellule active. Pour la développer, sélectionnez le lien **entrée masquée** quand elle est réduite.

   ![azure-notebook-collapse-cell-input](./media/apache-spark-development-using-notebooks/synapse-azure-notebook-collapse-cell-input.gif)

---

### <a name="collapse-a-cell-output"></a>Réduire une sortie de cellule

# <a name="classical-notebook"></a>[Notebook classique](#tab/classical)

Sélectionnez le bouton **Réduire la sortie** en haut à gauche de la sortie de cellule active pour la réduire. Pour la développer, sélectionnez **Afficher la sortie de cellule** quand la sortie de cellule est réduite.

   ![réduire-cellule-sortie](./media/apache-spark-development-using-notebooks/synapse-collapse-cell-output.gif)

# <a name="preview-notebook"></a>[Notebook en préversion](#tab/preview)

Sélectionnez le bouton de sélection (…) **Plus de commandes** dans la barre d’outils de la cellule et **Sortie** pour réduire la sortie de la cellule active. Pour la développer, sélectionnez le même bouton lorsque la sortie de la cellule est masquée.

   ![azure-notebook-collapse-cell-output](./media/apache-spark-development-using-notebooks/synapse-azure-notebook-collapse-cell-output.gif)


---

## <a name="run-notebooks"></a>Exécuter des blocs-notes

Vous pouvez exécuter les cellules de code dans votre bloc-notes individuellement ou toutes en même temps. L’état et la progression de chaque cellule sont représentés dans le bloc-notes.

### <a name="run-a-cell"></a>Exécuter une cellule

Il existe plusieurs façons d’exécuter le code figurant dans une cellule.

1. Pointez sur la cellule à exécuter, puis sélectionnez le bouton **Exécuter la cellule** ou appuyez sur **Ctrl + Entrée**.

   ![exécuter-cellule-1](./media/apache-spark-development-using-notebooks/synapse-run-cell.png)
  
2. Utilisez les [touches de raccourci en mode de commande](#shortcut-keys-under-command-mode). Appuyez sur **Maj + Entrée** pour exécuter la cellule active et sélectionner la cellule en dessous. Appuyez sur **Alt + Entrée** pour exécuter la cellule active et insérer une nouvelle cellule en dessous.

---

### <a name="run-all-cells"></a>Exécuter toutes les cellules
Sélectionnez le bouton **Exécuter tout** pour exécuter toutes les cellules du notebook actuel dans l’ordre.

   ![exécuter-tout-cellules](./media/apache-spark-development-using-notebooks/synapse-run-all.png)


# <a name="classical-notebook"></a>[Notebook classique](#tab/classical)

### <a name="run-all-cells-above-or-below"></a>Exécuter toutes les cellules au-dessus ou en dessous

Pour accéder au menu d’actions sur cellule supplémentaires tout à fait à droite, sélectionnez les points de suspension ( **...** ). Ensuite, sélectionnez **Exécuter les cellules au-dessus** pour exécuter toutes les cellules situées au-dessus de la cellule active dans l’ordre. Sélectionnez **Exécuter les cellules en dessous** pour exécuter toutes les cellules sous la cellule active dans l’ordre.

   ![exécuter-cellules-au-dessus-ou-en dessous](./media/apache-spark-development-using-notebooks/synapse-run-cells-above-or-below.png)


### <a name="cancel-all-running-cells"></a>Annuler toutes les cellules en cours d’exécution
Sélectionnez le bouton **Annuler tout** pour annuler les cellules en cours d’exécution ou les cellules dans la file d’attente. 
   ![cancel-all-cells](./media/apache-spark-development-using-notebooks/synapse-cancel-all.png) 

# <a name="preview-notebook"></a>[Notebook en préversion](#tab/preview)

L’annulation de toutes les cellules actives n’est pas encore disponible pour l’expérience de notebook en préversion. 

---



### <a name="reference-notebook"></a>Notebook de référence

# <a name="classical-notebook"></a>[Notebook classique](#tab/classical)

Non pris en charge.

# <a name="preview-notebook"></a>[Notebook en préversion](#tab/preview)

Vous pouvez utiliser la commande magic ```%run <notebook path>``` pour référencer un autre notebook dans le contexte du notebook actuel. Toutes les variables définies dans le notebook de référence sont disponibles dans le notebook actuel. La commande magic ```%run``` prend en charge les appels imbriqués, mais pas les appels récursifs. Vous recevrez une exception si la profondeur de l’instruction est supérieure à cinq. Actuellement, la commande ```%run``` permet seulement de transmettre un chemin d’accès de notebook comme paramètre. 

---


### <a name="cell-status-indicator"></a>Indicateur d’état de cellule

Un état d’exécution de cellule pas à pas est affiché sous la cellule pour vous aider à voir la progression en cours. Une fois l’exécution de la cellule terminée, un résumé de l’exécution avec la durée totale et l’heure de fin sont affichés et conservés là pour référence future.

![état-cellule](./media/apache-spark-development-using-notebooks/synapse-cell-status.png)

### <a name="spark-progress-indicator"></a>Indicateur de progression Spark

Le bloc-notes Azure Synapse Studio est entièrement basé sur Spark. Les cellules de code sont exécutées sur le pool Apache Spark serverless à distance. Un indicateur de progression du travail Spark est fourni avec une barre de progression en temps réel qui s’affiche pour vous aider à comprendre l’état d’exécution du travail.
Le nombre de tâches par travail ou index vous aide à identifier le niveau parallèle de votre travail Spark. Vous pouvez également explorer plus en profondeur l’IU Spark pour un travail (ou index) spécifique en sélectionnant le lien hypertexte du nom du travail (ou de l’index).


![spark-indicateur-progression](./media/apache-spark-development-using-notebooks/synapse-spark-progress-indicator.png)

### <a name="spark-session-config"></a>Configuration de session Spark

Vous pouvez spécifier le délai d’expiration, le nombre et la taille des exécuteurs à transmettre à la session Spark en cours dans **Configurer la session**. Redémarrez la session Spark pour que les modifications apportées à la configuration prennent effet. Toutes les variables du bloc-notes mises en cache sont effacées.

[![session-management](./media/apache-spark-development-using-notebooks/synapse-azure-notebook-spark-session-management.png)](./media/apache-spark-development-using-notebooks/synapse-azure-notebook-spark-session-management.png#lightbox)

#### <a name="spark-session-config-magic-command"></a>Commande magic de configuration de session Spark
Vous pouvez également spécifier des paramètres de session Spark via une commande magic **%%configure**. La session Spark doit redémarrer pour que les paramètres s’appliquent. Nous vous recommandons d’exécuter la fonctionnalité **%%configure** au début de votre notebook. En voici un exemple. Reportez-vous à https://github.com/cloudera/livy#request-body pour obtenir la liste complète des paramètres valides. 

```
%%configure -f
{
    to config the session.
    "driverMemory":"2g",
    "driverCores":3,
    "executorMemory":"2g",
    "executorCores":2,
    "jars":["myjar1.jar","myjar.jar"],
    "conf":{
        "spark.driver.maxResultSize":"10g"
    }
}
```


## <a name="bring-data-to-a-notebook"></a>Importer des données dans un bloc-notes

Vous pouvez charger des données à partir du service Stockage Blob Azure, d’Azure Data Lake Store Gén. 2 et d’un pool SQL, comme illustré dans les exemples de code ci-dessous.

### <a name="read-a-csv-from-azure-data-lake-store-gen2-as-a-spark-dataframe"></a>Lire un fichier CSV à partir d’Azure Data Lake Store Gén. 2 en tant que tramedonnées Spark

```python
from pyspark.sql import SparkSession
from pyspark.sql.types import *
account_name = "Your account name"
container_name = "Your container name"
relative_path = "Your path"
adls_path = 'abfss://%s@%s.dfs.core.windows.net/%s' % (container_name, account_name, relative_path)

df1 = spark.read.option('header', 'true') \
                .option('delimiter', ',') \
                .csv(adls_path + '/Testfile.csv')

```

#### <a name="read-a-csv-from-azure-blob-storage-as-a-spark-dataframe"></a>Lire un fichier CSV à partir du service Stockage Blob Azure en tant que tramedonnées Spark

```python

from pyspark.sql import SparkSession

# Azure storage access info
blob_account_name = 'Your account name' # replace with your blob name
blob_container_name = 'Your container name' # replace with your container name
blob_relative_path = 'Your path' # replace with your relative folder path
linked_service_name = 'Your linked service name' # replace with your linked service name

blob_sas_token = mssparkutils.credentials.getConnectionStringOrCreds(linked_service_name)

# Allow SPARK to access from Blob remotely

wasb_path = 'wasbs://%s@%s.blob.core.windows.net/%s' % (blob_container_name, blob_account_name, blob_relative_path)

spark.conf.set('fs.azure.sas.%s.%s.blob.core.windows.net' % (blob_container_name, blob_account_name), blob_sas_token)
print('Remote blob path: ' + wasb_path)

df = spark.read.option("header", "true") \
            .option("delimiter","|") \
            .schema(schema) \
            .csv(wasbs_path)
```

### <a name="read-data-from-the-primary-storage-account"></a>Lire des données à partir du compte de stockage principal

Vous pouvez accéder aux données directement dans le compte de stockage principal. Il n’est pas nécessaire de fournir les clés secrètes. Dans l’Explorateur de données, cliquez avec le bouton droit sur un fichier, puis sélectionnez **Nouveau bloc-notes** pour afficher un nouveau bloc-notes avec l’extracteur de données généré automatiquement.

![données-à-cellule](./media/apache-spark-development-using-notebooks/synapse-data-to-cell.png)

## <a name="save-notebooks"></a>Enregistrer des blocs-notes

Vous pouvez enregistrer un seul bloc-notes ou tous les blocs-notes dans votre espace de travail.

1. Pour enregistrer les modifications apportées à un seul bloc-notes, sélectionnez le bouton **Publier** dans la barre de commandes du bloc-notes.

   ![publier-bloc-notes](./media/apache-spark-development-using-notebooks/synapse-publish-notebook.png)

2. Pour enregistrer tous les blocs-notes dans votre espace de travail, sélectionnez le bouton **Publier tout** dans la barre de commandes de l’espace de travail. 

   ![publier-tout](./media/apache-spark-development-using-notebooks/synapse-publish-all.png)

Dans les propriétés du bloc-notes, vous pouvez éventuellement configurer l’inclusion de la sortie de cellule lors de l’enregistrement.

   ![bloc-notes-propriétés](./media/apache-spark-development-using-notebooks/synapse-notebook-properties.png)

## <a name="magic-commands"></a>Commandes magic
Vous pouvez utiliser des commandes magic Jupyter familières dans les notebooks Azure Synapse Studio. Vérifiez la liste suivante des commandes magic actuellement disponibles. Parlez-nous de [vos cas d’usage sur GitHub](https://github.com/MicrosoftDocs/azure-docs/issues/new) pour nous permettre de continuer à créer des commandes magic supplémentaires afin de répondre à vos besoins.

# <a name="classical-notebook"></a>[Notebook classique](#tab/classical)

Commandes magic de ligne disponibles : [%lsmagic](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-lsmagic), [%time](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-time), [%timeit](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-timeit)

Commandes magic de cellule disponibles : [%%time](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-time), [%%timeit](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-timeit), [%%capture](https://ipython.readthedocs.io/en/stable/interactive/magics.html#cellmagic-capture), [%%writefile](https://ipython.readthedocs.io/en/stable/interactive/magics.html#cellmagic-writefile), [%%sql](#use-multiple-languages), [%%pyspark](#use-multiple-languages), [%%spark](#use-multiple-languages), [%%csharp](#use-multiple-languages), [%%configure](#spark-session-config-magic-command)



# <a name="preview-notebook"></a>[Notebook en préversion](#tab/preview)

Commandes magic de ligne disponibles : [%lsmagic](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-lsmagic), [%time](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-time), [%timeit](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-timeit), [%history](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-history), [%run](#reference-notebook), [%load](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-load)

Commandes magic de cellule disponibles : [%%time](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-time), [%%timeit](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-timeit), [%%capture](https://ipython.readthedocs.io/en/stable/interactive/magics.html#cellmagic-capture), [%%writefile](https://ipython.readthedocs.io/en/stable/interactive/magics.html#cellmagic-writefile), [%%sql](#use-multiple-languages), [%%pyspark](#use-multiple-languages), [%%spark](#use-multiple-languages), [%%csharp](#use-multiple-languages), [%%html](https://ipython.readthedocs.io/en/stable/interactive/magics.html#cellmagic-html), [%%configure](#spark-session-config-magic-command)

--- 

## <a name="integrate-a-notebook"></a>Intégrer un notebook

### <a name="add-a-notebook-to-a-pipeline"></a>Ajouter un notebook à un pipeline

Sélectionnez le bouton **Ajouter au pipeline** dans le coin supérieur droit pour ajouter un notebook à un pipeline existant ou créer un pipeline.

![Ajouter un notebook à un pipeline](./media/apache-spark-development-using-notebooks/add-to-pipeline.png)

### <a name="designate-a-parameters-cell"></a>Désigner une cellule de paramètres

# <a name="classical-notebook"></a>[Notebook classique](#tab/classical)

Pour paramétrer votre notebook, sélectionnez les points de sélection (…) pour accéder au menu d’actions sur cellule supplémentaires à l’extrême droite. Sélectionnez ensuite **Activer/désactiver la cellule Paramètres** pour désigner la cellule comme cellule de paramètre.

![toggle-parameter](./media/apache-spark-development-using-notebooks/toggle-parameter-cell.png)

# <a name="preview-notebook"></a>[Notebook en préversion](#tab/preview)

Pour paramétrer votre notebook, sélectionnez le bouton de sélection (…) pour accéder au menu **Plus de commandes** au niveau de la barre d’outils de la cellule. Sélectionnez ensuite **Activer/désactiver la cellule Paramètres** pour désigner la cellule comme cellule de paramètre.

![azure-notebook-toggle-parameter](./media/apache-spark-development-using-notebooks/azure-notebook-toggle-parameter-cell.png)

---

Azure Data Factory recherche la cellule de paramètre et la traite comme cellule par défaut pour les paramètres transmis au moment de l’exécution. Le moteur d’exécution aura une nouvelle cellule sous la cellule des paramètres avec des paramètres d’entrée en vue de remplacer les valeurs par défaut. Lorsqu’il n’y a pas de cellule de paramètres désignée, la cellule injectée est insérée tout en haut du notebook.


### <a name="assign-parameters-values-from-a-pipeline"></a>Attribuer des valeurs de paramètres à partir d’un pipeline

Une fois le notebook avec paramètres créé, vous pouvez l’exécuter depuis un pipeline à l’aide de l’activité Notebook Azure Synapse. Après avoir ajouter l’activité à votre canevas de pipeline, vous serez en mesure de définir les valeurs des paramètres dans la section **Paramètres de base** de l’onglet **Paramètres**. 

![Attribuer un paramètre](./media/apache-spark-development-using-notebooks/assign-parameter.png)

Lors de l’attribution des valeurs de paramètre, vous pouvez utiliser le [langage d’expression du pipeline](../../data-factory/control-flow-expression-language-functions.md) ou des [variables système](../../data-factory/control-flow-system-variables.md).



## <a name="shortcut-keys"></a>Touches de raccourci

À l’instar des blocs-notes Jupyter, les blocs-notes Azure Synapse Studio disposent d’une interface utilisateur modale. Le clavier effectue des actions différentes selon le mode dans lequel se trouve la cellule du bloc-notes. Les blocs-notes Synapse Studio prennent en charge les deux modes suivants pour une cellule de code donnée : le mode de commande et le mode d’édition.

1. Une cellule est en mode de commande quand elle n’affiche aucun curseur texte vous invitant à saisir. Quand une cellule est en mode de commande, vous pouvez modifier le bloc-notes entier, mais pas taper dans des cellules individuelles. Entrez en mode de commande en appuyant sur `ESC` ou en utilisant la souris pour sélectionner en dehors de la zone de l’éditeur d’une cellule.

   ![mode-commande](./media/apache-spark-development-using-notebooks/synapse-command-mode-2.png)

2. Le mode d’édition est indiqué par un curseur texte qui vous invite à taper dans la zone de l’éditeur. Quand une cellule est en mode d’édition, vous pouvez saisir dans la cellule. Entrez en mode édition en appuyant sur `Enter` ou en utilisant la souris pour sélectionner la zone de l’éditeur d’une cellule.
   
   ![mode-édition](./media/apache-spark-development-using-notebooks/synapse-edit-mode-2.png)

### <a name="shortcut-keys-under-command-mode"></a>Touches de raccourci en mode de commande

# <a name="classical-notebook"></a>[Notebook classique](#tab/classical)

Les raccourcis clavier suivants vous permettent de parcourir et d’exécuter plus facilement du code dans des blocs-notes Azure Synapse.

| Action |Raccourcis de bloc-notes Synapse Studio  |
|--|--|
|Exécuter la cellule active et sélectionner la cellule en dessous | Maj + Entrée |
|Exécuter la cellule active et insérer en dessous | Alt + Entrée |
|Sélectionner la cellule au-dessus| Haut |
|Sélectionner la cellule en dessous| Descendre |
|Insérer une cellule au-dessus| Un |
|Insérer une cellule en dessous| B |
|Étendre les cellules sélectionnées au-dessus| Maj + Haut |
|Étendre les cellules sélectionnées en dessous| Maj + Bas|
|Déplacer la cellule vers le haut| Ctrl + Alt + ↑ |
|Déplacer la cellule vers le bas| Ctrl + Alt + ↓ |
|Supprimer les cellules sélectionnées| D, D |
|Basculer en mode d’édition| Entrez |

# <a name="preview-notebook"></a>[Notebook en préversion](#tab/preview)

| Action |Raccourcis de bloc-notes Synapse Studio  |
|--|--|
|Exécuter la cellule active et sélectionner la cellule en dessous | Maj + Entrée |
|Exécuter la cellule active et insérer en dessous | Alt + Entrée |
|Exécuter la cellule active| CTRL+ Enter |
|Sélectionner la cellule au-dessus| Haut |
|Sélectionner la cellule en dessous| Descendre |
|Sélectionner la cellule précédente| K |
|Sélectionner la cellule suivante| J |
|Insérer une cellule au-dessus| Un |
|Insérer une cellule en dessous| B |
|Supprimer les cellules sélectionnées| Maj + D |
|Basculer en mode d’édition| Entrez |

---

### <a name="shortcut-keys-under-edit-mode"></a>Touches de raccourci en mode d’édition


Les raccourcis clavier suivants vous permettent de naviguer et d’exécuter du code plus facilement dans des blocs-notes Azure Synapse en mode d’édition.

| Action |Raccourcis de bloc-notes Synapse Studio  |
|--|--|
|Déplacer le curseur vers le haut | Haut |
|Déplacer le curseur vers le bas|Descendre|
|Annuler|Ctrl + Z|
|Rétablir|CTRL + Y|
|Commenter/Supprimer un commentaire|Ctrl + /|
|Supprimer le mot précédent|Ctrl + Retour arrière|
|Supprimer le mot suivant|Ctrl + Suppr|
|Atteindre le début de la cellule|Ctrl + Début|
|Atteindre la fin de la cellule |Ctrl + Fin|
|Atteindre le mot à gauche|CTRL + Gauche|
|Atteindre le mot à droite|Ctrl + Droite|
|Sélectionner tout|Ctrl + A|
|Retrait| Ctrl + ]|
|Retrait négatif|Ctrl + [|
|Passer en mode de commande| Échap |

---

## <a name="next-steps"></a>Étapes suivantes
- [Consultez les exemples de notebooks Synapse](https://github.com/Azure-Samples/Synapse/tree/master/Notebooks)
- [Démarrage rapide : Créer un pool Apache Spark dans Azure Synapse Analytics avec des outils web](../quickstart-apache-spark-notebook.md)
- [Présentation d’Apache Spark dans Azure Synapse Analytics](apache-spark-overview.md)
- [Utiliser .NET pour Apache Spark avec Azure Synapse Analytics](spark-dotnet.md)
- [Documentation .NET pour Apache Spark](/dotnet/spark?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)
- [Azure Synapse Analytics](https://docs.microsoft.com/azure/synapse-analytics)

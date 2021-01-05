---
title: Accès aux ressources avec Data Lake Tools
description: Découvrez comment utiliser Azure Data Lake Tools pour accéder aux ressources d’Azure Data Lake Analytics.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 02/09/2018
ms.openlocfilehash: d04f108b45070b27c4ff9ed833e8fb77b74cd597
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96754495"
---
# <a name="accessing-resources-with-azure-data-lake-tools"></a>Accès aux ressources avec Azure Data Lake Tools

Vous pouvez facilement accéder aux ressources Azure Data Lake Analytics à l’aide de commandes ou d’actions Azure Data Tools dans VS Code.

## <a name="integrate-with-azure-data-lake-analytics-through-a-command"></a>Intégration à Azure Data Lake Analytics à l'aide d'une commande

Vous pouvez accéder aux ressources Azure Data Lake Analytics pour consulter la liste des comptes, accéder aux métadonnées et afficher les travaux d’analyse.

### <a name="to-list-the-azure-data-lake-analytics-accounts-under-your-azure-subscription"></a>Pour lister les comptes Azure Data Lake Analytics de votre abonnement Azure

1. Sélectionnez Ctrl+Maj+P pour ouvrir la palette de commandes.
2. Entrez **ADL: List Accounts**. Les comptes s’affichent dans le panneau **Sortie**.

### <a name="to-access-azure-data-lake-analytics-metadata"></a>Pour accéder aux métadonnées Azure Data Lake Analytics

1. Sélectionnez Ctrl+Maj+P et entrez **ADL: List Tables**.
2. Sélectionnez l’un des comptes Data Lake Analytics.
3. Sélectionnez l’une des bases de données Data Lake Analytics.
4. Sélectionnez l’un des schémas. La liste des tables s’affiche.

### <a name="to-view-azure-data-lake-analytics-jobs"></a>Pour afficher les travaux Azure Data Lake Analytics

1. Ouvrez la palette de commandes (Ctrl+Maj+P) et sélectionnez **ADL: Show Jobs**.
2. Sélectionnez un compte local ou Data Lake Analytics.
3. Attendez que la liste des travaux correspondant au compte apparaisse.
4. Sélectionnez un travail dans la liste des travaux. Data Lake Tools ouvre la vue des travaux dans le volet droit et affiche des informations dans la sortie VS Code.

    ![Liste des travaux](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-show-job.png)

## <a name="integrate-with-azure-data-lake-store-through-a-command"></a>Intégration à Azure Data Lake Store à l'aide d'une commande

Vous pouvez utiliser les commandes Azure Data Lake Store pour :

- [Parcourir les ressources Azure Data Lake Store](#list-the-storage-path)
- [Afficher un aperçu du fichier Azure Data Lake Store](#preview-the-storage-file)
- Charger directement le fichier sur Azure Data Lake Store dans VS Code
- Télécharger directement le fichier à partir d’Azure Data Lake Store dans VS Code

### <a name="list-the-storage-path"></a>Répertorier le chemin de stockage

### <a name="to-list-the-storage-path-through-the-command-palette"></a>Pour lister le chemin de stockage par le biais de la palette de commandes

1. Cliquez avec le bouton droit sur l’éditeur de script, et sélectionnez **ADL: List Path**.
2. Choisissez le dossier dans la liste, ou sélectionnez **Enter a path** (Entrer un chemin) ou **Browse from root path** (Parcourir à partir de la racine). (Nous utilisons **Enter a path** comme exemple).
3. Sélectionnez votre compte Data Lake Analytics.
4. Entrez ou accédez au chemin du dossier de stockage (par exemple, /output/).  

La palette de commandes répertorie les informations relatives au chemin d'après vos entrées.

![Résultats de chemin de stockage](./media/data-lake-analytics-data-lake-tools-for-vscode/list-storage-path.png)

Pour répertorier facilement le chemin relatif, vous pouvez également utiliser le menu contextuel.

### <a name="to-list-the-storage-path-through-the-shortcut-menu"></a>Pour lister le chemin de stockage à l'aide du menu contextuel

Cliquez avec le bouton droit sur la chaîne du chemin et sélectionnez **List Path** (Répertorier le chemin).

![Option « List Path » du menu contextuel](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-right-click-path.png)

### <a name="preview-the-storage-file"></a>Afficher un aperçu du fichier de stockage

1. Cliquez avec le bouton droit sur l’éditeur de script, et sélectionnez **ADL: Preview File**.
2. Sélectionnez votre compte Data Lake Analytics.
3. Entrez le chemin d'un fichier de stockage Azure (par exemple, /output/SearchLog.txt).

Le fichier s’ouvre dans VS Code.

![Afficher un aperçu du fichier de stockage - Étapes et résultat](./media/data-lake-analytics-data-lake-tools-for-vscode/preview-storage-file.png)

Vous pouvez également afficher un aperçu du fichier en utilisant le menu contextuel sur le chemin complet ou relatif du fichier dans l’éditeur de script.

### <a name="upload-a-file-or-folder"></a>Charger un fichier ou un dossier

1. Cliquez avec le bouton droit sur l’éditeur de script et sélectionnez **Upload File** (Charger un fichier) ou **Upload Folder** (Charger un dossier).
2. Choisissez un ou plusieurs fichiers si vous avez sélectionné **Upload File**, ou le dossier tout entier si vous avez sélectionné **Upload Folder**. Puis sélectionnez **Upload** (Charger).
3. Choisissez le dossier de stockage dans la liste, ou sélectionnez **Enter a path** (Entrer un chemin) ou **Browse from root path** (Parcourir à partir de la racine). (Nous utilisons **Enter a path** comme exemple).
4. Sélectionnez votre compte Data Lake Analytics.
5. Entrez ou accédez au chemin du dossier de stockage (par exemple, /output/).
6. Sélectionnez **Choose Current Folder** (Choisir le dossier actuel) pour spécifier la destination du chargement.

![Charger un fichier ou un dossier - Étapes et résultat](./media/data-lake-analytics-data-lake-tools-for-vscode/upload-file.png)

Vous pouvez également charger des fichiers dans le stockage en utilisant le menu contextuel sur le chemin complet ou relatif du fichier dans l’éditeur de script.

Vous pouvez [surveiller l’état du chargement](#check-storage-tasks-status).

### <a name="download-a-file"></a>Téléchargement d’un fichier

Vous pouvez télécharger un fichier avec la commande **ADL: Download File** ou **ADL: Download File (Advanced)** .

### <a name="to-download-a-file-through-the-adl-download-file-advanced-command"></a>Pour télécharger un fichier via ADL : Commande Download File (Advanced)

1. Cliquez avec le bouton droit sur l’éditeur de script, puis sélectionnez **Download File (Advanced)** (Télécharger un fichier (Avancé)).
2. VS Code affiche un fichier JSON. Vous pouvez entrer des chemins de fichiers et télécharger plusieurs fichiers en même temps. Les instructions s’affichent dans la fenêtre **Output**. Pour télécharger un ou plusieurs fichiers, enregistrez (Ctrl+S) le fichier JSON.

    ![Fichier JSON avec chemins d’accès pour le téléchargement de fichiers](./media/data-lake-analytics-data-lake-tools-for-vscode/download-multi-files.png)

La fenêtre **Output** affiche l’état de téléchargement du fichier.

![Fenêtre Output avec état de téléchargement](./media/data-lake-analytics-data-lake-tools-for-vscode/download-multi-file-result.png)

Vous pouvez [surveiller l’état du téléchargement](#check-storage-tasks-status).

### <a name="to-download-a-file-through-the-adl-download-file-command"></a>Pour télécharger un fichier via ADL : Commande Download File

1. Cliquez avec le bouton droit sur l’éditeur de script, sélectionnez **Download File** (Télécharger un fichier), puis sélectionnez le dossier de destination dans la boîte de dialogue **Select Folder** (Sélectionner un dossier).

1. Choisissez le dossier dans la liste, ou sélectionnez **Enter a path** (Entrer un chemin) ou **Browse from root path** (Parcourir à partir de la racine). (Nous utilisons **Enter a path** comme exemple).

1. Sélectionnez votre compte Data Lake Analytics.

1. Entrez ou accédez au chemin du dossier de stockage (par exemple, /output/), puis choisissez un fichier à télécharger.

![Télécharger un fichier - Étapes et résultat](./media/data-lake-analytics-data-lake-tools-for-vscode/download-file.png)

Vous pouvez également télécharger des fichiers de stockage en utilisant le menu contextuel sur le chemin complet ou relatif du fichier dans l’éditeur de script.

Vous pouvez [surveiller l’état du téléchargement](#check-storage-tasks-status).

### <a name="check-storage-tasks-status"></a>Vérifier l’état des tâches de stockage

L’état du chargement et du téléchargement s’affiche sur la barre d’état. Sélectionnez la barre d’état. L’état apparaît alors dans l'onglet **OUTPUT** (SORTIE).

![Barre d’état et détails de la sortie](./media/data-lake-analytics-data-lake-tools-for-vscode/storage-status.png)

## <a name="integrate-with-azure-data-lake-analytics-from-the-explorer"></a>Intégration à Azure Data Lake Analytics depuis l’Explorateur

Une fois connecté, tous les abonnements associés à votre compte Azure apparaissent dans le volet gauche, sous **AZURE DATALAKE**.

![Explorateur de Data Lake](./media/data-lake-analytics-data-lake-tools-for-vscode/datalake-explorer.png)

### <a name="data-lake-analytics-metadata-navigation"></a>Navigation au sein des métadonnées Data Lake Analytics

Développez votre abonnement Azure. Sous le nœud **U-SQL Databases** (Bases de données U-SQL), vous pouvez parcourir votre base de données U-SQL et afficher des dossiers tels que **Schemas** (Schémas), **Credentials** (Informations d’identification), **Assemblies** (Assemblys), **Tables** et **Index**.

### <a name="data-lake-analytics-metadata-entity-management"></a>Gestion des entités de métadonnées Data Lake Analytics

Développez **U-SQL Databases** (Bases de données U-SQL). Vous pouvez créer une base de données, un schéma, une table, un type de table, un index ou des statistiques en double-cliquant sur le nœud correspondant, puis en sélectionnant **Script to Create** (Script à créer) dans le menu contextuel. Sur la page du script ouvert, modifiez le script conformément à vos besoins. Puis soumettez le travail en cliquant dessus avec le bouton droit et en sélectionnant **ADL: Submit Job**.

Au terme de la création de l’élément, cliquez sur le nœud avec le bouton droit, puis sélectionnez **Refresh** (Actualiser) pour afficher l’élément. Vous pouvez également supprimer l’élément en cliquant dessus avec le bouton droit, puis en sélectionnant **Delete** (Supprimer).

![Commande « Script to Create » (Script à créer) du menu contextuel de l’Explorateur de Data Lake](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-code-explorer-script-create.png)

![Page de script relative au nouvel élément](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-code-explorer-script-create-snippet.png)

### <a name="data-lake-analytics-assembly-registration"></a>Inscription de l’assembly Data Lake Analytics

Vous pouvez inscrire un assembly dans la base de données correspondante en cliquant avec le bouton droit sur le nœud **Assemblies** (Assemblys) et en sélectionnant **Register assembly** (Inscrire l'assembly).

![Commande « Register assembly » (Inscrire l'assembly) du menu contextuel pour le nœud Assemblies (Assemblys)](./media/data-lake-analytics-data-lake-tools-for-vscode/datalake-explorer-register-assembly.png)

## <a name="integrate-with-azure-data-lake-store-from-the-explorer"></a>Intégration à Azure Data Lake Store depuis l’Explorateur

Accédez à **Data Lake Store** :

- Vous pouvez cliquer avec le bouton droit sur le nœud de dossier, puis utiliser les commandes suivantes du menu contextuel : **Refresh** (Actualiser), **Delete** (Supprimer), **Upload** (Charger), **Upload Folder** (Charger un dossier), **Copy Relative Path** (Copier le chemin relatif) et **Copy Full Path** (Copier le chemin complet).

   ![Commandes du menu contextuel pour un nœud de dossier dans l’Explorateur de Data Lake](./media/data-lake-analytics-data-lake-tools-for-vscode/storage-account-folder-menu.png)

- Vous pouvez cliquer avec le bouton droit sur le nœud de fichier, puis utiliser les commandes suivantes du menu contextuel : **Preview** (Aperçu), **Download** (Télécharger), **Delete** (Supprimer), **Create EXTRACT Script** (Créer un script d'extraction) (uniquement disponible pour les fichiers CSV, TSV et TXT), **Copy Relative Path** (Copier le chemin relatif) et **Copy Full Path** (Copier le chemin complet).

   ![Commandes du menu contextuel pour un nœud de fichier dans l’Explorateur de Data Lake](./media/data-lake-analytics-data-lake-tools-for-vscode/storage-account-extract.png)

## <a name="integrate-with-azure-blob-storage-from-the-explorer"></a>Intégration au stockage d'objets blob Azure depuis l’Explorateur

Accédez au stockage d’objets Blob :

- Vous pouvez cliquer avec le bouton droit sur le nœud de conteneur d'objets blob, puis utilisez les commandes suivantes du menu contextuel : **Refresh** (Actualiser), **Delete Blob Container** (Supprimer le conteneur d'objets blob) et **Upload Blob** (Charger l'objet blob).

   ![Commandes du menu contextuel pour un nœud de conteneur d’objets blob sous le stockage d’objets Blob](./media/data-lake-analytics-data-lake-tools-for-vscode/blob-storage-blob-container-node.png)

- Vous pouvez cliquer avec le bouton droit sur le nœud de dossier, puis utiliser les commandes suivantes du menu contextuel : **Refresh** (Actualiser) et **Upload Blob** (Charger l'objet blob).

   ![Commandes du menu contextuel pour un nœud de dossier sous le stockage d’objets Blob](./media/data-lake-analytics-data-lake-tools-for-vscode/blob-storage-folder-node.png)

- Vous pouvez cliquer avec le bouton droit sur le nœud de fichier, puis utiliser les commandes suivantes du menu contextuel : **Preview/Edit** (Aperçu/Modifier), **Download** (Télécharger), **Delete** (Supprimer), **Create EXTRACT Script** (Créer un script d'extraction) (uniquement disponible pour les fichiers CSV, TSV et TXT), **Copy Relative Path** (Copier le chemin relatif) et **Copy Full Path** (Copier le chemin complet).

    ![Commandes du menu contextuel pour un nœud de fichier sous le stockage d’objets Blob](./media/data-lake-analytics-data-lake-tools-for-vscode/create-extract-script-from-context-menu-2.png)

## <a name="open-the-data-lake-explorer-in-the-portal"></a>Ouvrir l’Explorateur de Data Lake dans le portail

1. Sélectionnez Ctrl+Maj+P pour ouvrir la palette de commandes.
2. Entrez **Open Web Azure Storage Explorer**. Ou, dans l’éditeur de script, cliquez avec le bouton droit sur un chemin relatif ou sur le chemin complet et sélectionnez **Open Web Azure Storage Explorer**.
3. Sélectionnez un compte Data Lake Analytics.

Data Lake Tools ouvre le chemin de stockage Azure dans le portail Azure. Vous pouvez rechercher le chemin et afficher un aperçu du fichier à partir du web.

## <a name="additional-features"></a>Fonctionnalités supplémentaires

Data Lake Tools pour VS Code prend en charge les fonctionnalités suivantes :

- **IntelliSense autocomplete** (Saisie semi-automatique IntelliSense) : des suggestions s’affichent dans des fenêtres indépendantes autour des éléments tels que les mots clés, les méthodes et les variables. Les différentes icônes représentent les différents types d’objets :

  - Type de données Scala
  - Type de données complexe
  - Types définis par l’utilisateur (UDT) intégrés
  - Classes et collection .NET
  - Expressions C#
  - UDF, UDO et UDAAG C# intégrés
  - Fonctions U-SQL
  - Fonctions de fenêtrage U-SQL

    ![Types d’objets IntelliSense](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-auto-complete-objects.png)

- Data Lake Tools **IntelliSense autocomplete on Data Lake Analytics metadata** (Saisie semi-automatique IntelliSense sur les métadonnées Data Lake Analytics) : Data Lake Tools télécharge les informations de métadonnées Data Lake Analytics localement.télécharge en local les informations sur les métadonnées Data Lake Analytics. La fonctionnalité IntelliSense renseigne automatiquement les objets à partir des métadonnées Data Lake Analytics. Ces objets incluent la base de données, le schéma, la table, la vue, la fonction table, les procédures et les assemblys C#.

  ![Métadonnées IntelliSense](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-auto-complete-metastore.png)

- **IntelliSense error marker** (Marqueur d’erreurs IntelliSense) : Data Lake Tools souligne les erreurs d’édition pour U-SQL et C#.
- **Syntax highlights** (Coloration syntaxique) : Data Lake Tools utilise des couleurs pour différencier les éléments tels que les variables, les mots clés, les types de données et les fonctions.

    ![Syntaxe avec couleurs](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-syntax-highlights.png)

> [!NOTE]
> Nous vous recommandons de procéder à une mise à niveau vers Azure Data Lake Tools pour Visual Studio version 2.3.3000.4 ou ultérieure. Notez que les versions précédentes ne peuvent plus être téléchargées et que Microsoft les déconseille.  

## <a name="next-steps"></a>Étapes suivantes

- [Développer en U-SQL avec Python, R et C Sharp pour Azure Data Lake Analytics dans VS Code](data-lake-analytics-u-sql-develop-with-python-r-csharp-in-vscode.md)
- [Exécution locale et débogage local U-SQL avec Visual Studio Code](data-lake-tools-for-vscode-local-run-and-debug.md)
- [Tutoriel : Prise en main d’Azure Data Lake Analytics](data-lake-analytics-get-started-portal.md)
- [Tutoriel : Développer des scripts U-SQL à l’aide de Data Lake Tools pour Visual Studio](data-lake-analytics-data-lake-tools-get-started.md)
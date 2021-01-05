---
title: Utiliser Azure Data Lake Tools pour Visual Studio Code
description: Découvrez comment utiliser Azure Data Lake Tools pour Visual Studio Code pour créer, tester et exécuter des scripts U-SQL.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 02/09/2018
ms.openlocfilehash: 40e3ce17e036312e7c3fdee95fcb42d06f5845e9
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96751357"
---
# <a name="use-azure-data-lake-tools-for-visual-studio-code"></a>Utiliser Azure Data Lake Tools pour Visual Studio Code

Dans cet article, vous allez apprendre à utiliser Azure Data Lake Tools pour Visual Studio Code (VS Code) pour créer, tester et exécuter des scripts U-SQL. Ces informations sont également décrites dans la vidéo suivante :

[![Lecteur vidéo : Azure Data Lake Tools pour VS Code](media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-video.png)](https://channel9.msdn.com/Series/AzureDataLake/Azure-Data-Lake-Tools-for-VSCode?term=ADL%20Tools%20for%20VSCode")

## <a name="prerequisites"></a>Prérequis

Azure Data Lake Tools pour VS Code prend en charge Windows, Linux et MacOS.  L’exécution locale et le débogage local de U-SQL fonctionnent uniquement pour les utilisateurs de Windows.

- [Visual Studio Code](https://www.visualstudio.com/products/code-vs.aspx)

Pour MacOS et Linux :

- [Kit SDK .NET Core 2.0](https://www.microsoft.com/net/download/core)
- [Mono 5.2.x](https://www.mono-project.com/download/)

## <a name="install-azure-data-lake-tools"></a>Installer les outils Azure Data Lake

Après avoir installé les prérequis, vous pouvez installer Azure Data Lake Tools pour Visual Studio Code.

### <a name="to-install-azure-data-lake-tools"></a>Pour installer Azure Data Lake Tools

1. Ouvrez Visual Studio Code.
2. Sélectionnez **Extensions** dans le volet gauche. Entrez **Azure Data Lake Tools** dans la zone de recherche.
3. Sélectionnez **Installer** en regard de **Azure Data Lake Tools**.

   ![Sélections à effectuer pour installer Data Lake Tools](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-extensions.png)

   Après quelques secondes, le bouton **Installer** est remplacé par le bouton **Recharger**.
4. Sélectionnez **Recharger** pour activer l’extension **Azure Data Lake Tools**.
5. Sélectionnez **Recharger la fenêtre** pour confirmer. **Azure Data Lake Tools** s’affiche dans le volet **Extensions**.

## <a name="activate-azure-data-lake-tools"></a>Activer Azure Data Lake Tools

Créez un fichier .usql ou ouvrez un fichier .usql existant pour activer l’extension.

## <a name="work-with-u-sql"></a>Utilisation de U-SQL

Vous devez ouvrir un fichier ou un dossier U-SQL pour utiliser U-SQL.

### <a name="to-open-the-sample-script"></a>Pour ouvrir l’exemple de script

Ouvrez la palette de commandes (Ctrl+Maj+P) et entrez **ADL: Open Sample Script**. Une autre instance de cet exemple s’ouvre. Vous pouvez également modifier, configurer et envoyer un script sur cette instance.

### <a name="to-open-a-folder-for-your-u-sql-project"></a>Pour ouvrir un dossier pour votre projet U-SQL

1. À partir de Visual Studio Code, sélectionnez le menu **Fichier**, puis **Ouvrir un dossier**.
2. Spécifiez un dossier, puis sélectionnez **Sélectionner le dossier**.
3. Sélectionnez le menu **Fichier**, puis **Nouveau**. Un fichier Untitled-1 est ajouté au projet.
4. Entrez le code suivant dans le fichier Untitled-1 :

   ```usql
   @departments  =
       SELECT * FROM
           (VALUES
               (31,    "Sales"),
               (33,    "Engineering"),
               (34,    "Clerical"),
               (35,    "Marketing")
           ) AS
                 D( DepID, DepName );
   ```

   OUTPUT @departments     TO "/Output/departments.csv" USING Outputters.Csv();

    Le script crée un fichier departments.csv avec des données dans le dossier /output.

5. Enregistrez le fichier sous **myUSQL.usql** dans le dossier ouvert.

### <a name="to-compile-a-u-sql-script"></a>Pour compiler un script U-SQL

1. Sélectionnez Ctrl+Maj+P pour ouvrir la palette de commandes.
2. Entrez **ADL: Compile Script**. Les résultats de la compilation s’affichent dans la fenêtre **Output**. Vous pouvez également cliquer avec le bouton droit sur un fichier de script, puis sélectionner **ADL: Compile Script** pour compiler un travail U-SQL. Le résultat de la compilation s’affiche dans le volet **Output**.

### <a name="to-submit-a-u-sql-script"></a>Pour envoyer un script U-SQL

1. Sélectionnez Ctrl+Maj+P pour ouvrir la palette de commandes.
2. Entrez **ADL: Submit Job**. Vous pouvez également cliquer avec le bouton droit sur un fichier de script, puis sélectionner **ADL: Submit Job**.

Une fois que vous avez envoyé un travail U-SQL, les journaux d’activité d’envoi apparaissent dans la fenêtre **Output** dans VS Code. La vue des travaux apparaît dans le volet droit. Si l’envoi a abouti, l’URL du travail est également affichée. Vous pouvez ouvrir l’URL du travail dans un navigateur web pour suivre l’état du travail en temps réel.

L’onglet **SUMMARY** (RÉSUMÉ) de la vue des travaux affiche les détails du travail. Les principales fonctions disponibles sont le renvoi d'un script, la duplication d'un script et l’ouverture dans le portail. L’onglet **DATA** (DONNÉES) de la vue des travaux vous permet de vous référer aux fichiers d’entrée, aux fichiers de sortie et aux ressources. Vous pouvez télécharger les fichiers sur l’ordinateur local.

![Onglet Summary (Résumé) de la vue des travaux](./media/data-lake-analytics-data-lake-tools-for-vscode/job-view-summary.png)

![Onglet Data (Données) de la vue des travaux](./media/data-lake-analytics-data-lake-tools-for-vscode/job-view-data.png)

### <a name="to-set-the-default-context"></a>Pour définir le contexte par défaut

Si vous n’avez pas configuré les fichiers individuellement, vous pouvez définir le contexte par défaut pour l'appliquer à tous les fichiers de script.

1. Sélectionnez Ctrl+Maj+P pour ouvrir la palette de commandes.
2. Entrez **ADL: Set Default Context**. Ou cliquez avec le bouton droit sur l’éditeur de script, puis sélectionnez **ADL: Set Default Context**.
3. Choisissez le compte, la base de données et le schéma souhaités. La configuration est enregistrée dans le fichier xxx_settings.json.

   ![Compte, base de données et schéma définis comme contexte par défaut](./media/data-lake-analytics-data-lake-tools-for-vscode/default-context-sequence.png)

### <a name="to-set-script-parameters"></a>Pour définir les paramètres de script

1. Sélectionnez Ctrl+Maj+P pour ouvrir la palette de commandes.
2. Entrez **ADL: Set Script Parameters**.
3. Le fichier xxx_settings.json s’ouvre avec les propriétés suivantes :

   - **account** : compte Azure Data Lake Analytics situé sous votre abonnement Azure et nécessaire à la compilation et à l’exécution des travaux U-SQL. Vous devez configurer le compte d’ordinateur avant de compiler et d’exécuter les travaux U-SQL.
   - **database**: base de données sous votre compte. La valeur par défaut est **master**.
   - **schema**: schéma sous votre base de données. La valeur par défaut est **dbo**.
   - **optionalSettings** :
        - **priority**: la plage de priorité est comprise entre 1 et 1000. 1 correspond à la priorité la plus élevée. La valeur par défaut est **1000**.
        - **degreeOfParallelism**: la plage de parallélisme est comprise entre 1 et 150. La valeur par défaut est le parallélisme maximal autorisé dans votre compte Azure Data Lake Analytics.

   ![Contenu du fichier JSON](./media/data-lake-analytics-data-lake-tools-for-vscode/default-context-setting.png)

> [!NOTE]
> Une fois la configuration enregistrée et si vous n’avez pas défini de contexte par défaut, les informations relatives au compte, à la base de données et au schéma s’affichent sur la barre d’état, en bas à gauche du fichier .usql correspondant.

### <a name="to-set-git-ignore"></a>Pour définir Git Ignore

1. Sélectionnez Ctrl+Maj+P pour ouvrir la palette de commandes.
2. Entrez **ADL: Set Git Ignore**.

   - Si votre dossier de travail VSCode ne contient pas de fichier **.gitIgnore**, un fichier nommé **.gitIgnore** y est créé. Quatre éléments (**usqlCodeBehindReference**, **usqlCodeBehindGenerated**, **.cache** et **obj**) sont ajoutés au fichier par défaut. Si nécessaire, vous pouvez effectuer d'autres mises à jour.
   - Si votre dossier de travail VSCode contient déjà un fichier **.gitIgnore**, l’outil ajoute quatre éléments (**usqlCodeBehindReference**, **usqlCodeBehindGenerated**, **.cache** et **obj**) dans votre fichier **.gitIgnore**, à condition qu'ils n'y figurent pas déjà.

   ![Éléments présents dans le fichier .gitIgnore](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-gitignore.png)

## <a name="work-with-code-behind-files-c-sharp-python-and-r"></a>Travailler avec les fichiers code-behind : C Sharp, Python et R

Azure Data Lake Tools prend en charge différents codes personnalisés. Pour obtenir des instructions, reportez-vous à [Développer en U-SQL avec Python, R et C Sharp pour Azure Data Lake Analytics dans VSCode](data-lake-analytics-u-sql-develop-with-python-r-csharp-in-vscode.md).

## <a name="work-with-assemblies"></a>Travailler avec les assemblies

Pour plus d’informations sur le développement d’assemblys, consultez [Développement d’assemblys U-SQL pour les tâches d’Azure Data Lake Analytics]().

Vous pouvez utiliser Data Lake Tools pour inscrire des assemblys de code personnalisé dans le catalogue Data Lake Analytics.

### <a name="to-register-an-assembly"></a>Pour inscrire un assembly

Vous pouvez inscrire l’assembly via la commande **ADL: Register Assembly** ou **ADL: Register Assembly (Advanced)** .

### <a name="to-register-through-the-adl-register-assembly-command"></a>Pour effectuer une inscription via ADL : Commande Register Assembly

1. Sélectionnez Ctrl+Maj+P pour ouvrir la palette de commandes.
2. Entrez **ADL: Register Assembly**.
3. Spécifiez le chemin d’accès de l’assembly local.
4. Sélectionnez un compte Data Lake Analytics.
5. Sélectionnez une base de données.

Le portail s’ouvre dans un navigateur et affiche le processus d’inscription de l’assembly.  

Pour déclencher la commande **ADL: Register Assembly**, vous pouvez également cliquer avec le bouton droit sur le fichier .dll dans l’Explorateur de fichiers.

### <a name="to-register-through-the-adl-register-assembly-advanced-command"></a>Pour effectuer une inscription via ADL : Commande Register Assembly (Advanced)

1. Sélectionnez Ctrl+Maj+P pour ouvrir la palette de commandes.
2. Entrez **ADL: Register Assembly (Advanced)** .
3. Spécifiez le chemin d’accès de l’assembly local.
4. Le fichier JSON s’affiche. Examinez et modifiez, si nécessaire, les paramètres des ressources et les dépendances de l’assembly. Les instructions s’affichent dans la fenêtre **Output**. Pour procéder à l’inscription de l’assembly, enregistrez (Ctrl+S) le fichier JSON.

   ![Fichier JSON avec paramètres de ressources et dépendances de l’assembly](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-register-assembly-advance.png)

>[!NOTE]
>
>- S'il existe des dépendances de l’assembly dans la bibliothèque de liens dynamiques (DLL), Azure Data Lake Tools les détecte automatiquement. Une fois détectées, les dépendances apparaissent dans le fichier JSON.
>- Vous pouvez charger vos ressources DLL (.txt, .png et .csv, par exemple) dans le cadre de l’inscription de l’assembly.

Pour déclencher la commande **ADL: Register Assembly (Advanced)** , vous pouvez également cliquer avec le bouton droit sur le fichier .dll dans l’Explorateur de fichiers.

Le code U-SQL suivant montre comment appeler un assembly. Dans l’exemple, le nom de l’assembly est *test*.

```usql
REFERENCE ASSEMBLY [test];
@a =
    EXTRACT
        Iid int,
    Starts DateTime,
    Region string,
    Query string,
    DwellTime int,
    Results string,
    ClickedUrls string
    FROM @"Sample/SearchLog.txt"
    USING Extractors.Tsv();
@d =
    SELECT DISTINCT Region
    FROM @a;
@d1 =
    PROCESS @d
    PRODUCE
        Region string,
    Mkt string
    USING new USQLApplication_codebehind.MyProcessor();
OUTPUT @d1
    TO @"Sample/SearchLogtest.txt"
    USING Outputters.Tsv();
```

## <a name="use-u-sql-local-run-and-local-debug-for-windows-users"></a>Utiliser l'exécution locale et le débogage local de U-SQL pour les utilisateurs de Windows

L’exécution locale de U-SQL teste vos données locales et valide votre script localement, avant que votre code soit publié dans Data Lake Analytics. Vous pouvez utiliser la fonctionnalité de débogage local pour accomplir les tâches suivantes avant que votre code soit soumis à Data Lake Analytics :

- Déboguer votre code-behind C#
- Examiner le code
- Valider votre script localement

La fonctionnalité d’exécution locale et de débogage local fonctionne uniquement dans des environnements Windows, et elle n’est pas prise en charge sur les systèmes d’exploitation MacOS et Linux.

Pour obtenir des instructions sur l’exécution locale et le débogage local, consultez [Exécution locale et débogage local U-SQL avec Visual Studio Code](data-lake-tools-for-vscode-local-run-and-debug.md).

## <a name="connect-to-azure"></a>Connexion à Azure

Avant de pouvoir compiler et exécuter des scripts U-SQL dans Data Lake Analytics, vous devez vous connecter à votre compte Azure.

<a id="sign-in-by-command"></a>

### <a name="to-connect-to-azure-by-using-a-command"></a>Pour se connecter à Azure à l'aide d'une commande

1. Sélectionnez Ctrl+Maj+P pour ouvrir la palette de commandes.

2. Entrez **ADL: Connexion**. Les informations de connexion s’affichent en bas à droite.

   ![Entrer la commande de connexion](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-extension-login.png)

   ![Notification relative à la connexion et à l'authentification](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-login-info.png)

3. Sélectionnez **Copier et ouvrir** pour ouvrir la [page web de connexion](https://aka.ms/devicelogin). Collez le code dans la zone, puis sélectionnez **Continuer**.

    ![Page web de connexion](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-extension-login-paste-code.png)  

4. Suivez les instructions pour vous connecter à partir de la page web. Une fois connecté, le nom de votre compte Azure s’affiche sur la barre d’état dans le coin inférieur gauche de la fenêtre VS Code.

> [!NOTE]
>
> - Si vous ne vous déconnectez pas, Data Lake Tools vous connectera automatiquement lors de votre prochaine visite.
> - Si votre compte a deux facteurs activés, nous vous recommandons d’utiliser l’authentification par téléphone plutôt qu’un code PIN.

Pour vous déconnecter, entrez la commande **ADL: Logout**.

### <a name="to-connect-to-azure-from-the-explorer"></a>Pour se connecter à Azure à partir de l’Explorateur

Développez **AZURE DATALAKE**, sélectionnez **Se connecter à Azure**, puis suivez les étapes 3 et 4 de la section [Pour vous connecter à Azure à l’aide d’une commande](#sign-in-by-command).

![Sélection de « Connexion à Azure » dans l’Explorateur](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-sign-in-from-explorer.png )  

Vous ne pouvez pas vous déconnecter depuis l’Explorateur. Pour vous déconnecter, reportez-vous à la section [Pour vous connecter à Azure à l'aide d'une commande](#sign-in-by-command).

## <a name="create-an-extraction-script"></a>Créer un script d’extraction

Vous pouvez créer un script d’extraction pour des fichiers .csv, .tsv et .txt à l’aide de la commande **ADL: Create EXTRACT Script** ou à partir de l’Explorateur Azure Data Lake.

### <a name="to-create-an-extraction-script-by-using-a-command"></a>Pour créer un script d’extraction à l’aide d’une commande

1. Sélectionnez Ctrl+Maj+P pour ouvrir la palette de commandes, puis entrez **ADL: Create EXTRACT Script**.
2. Spécifiez le chemin d’accès complet d’un fichier de stockage Azure, puis sélectionnez la touche Entrée.
3. Sélectionnez un compte.
4. Pour un fichier .txt, sélectionnez un séparateur afin d'extraire le fichier.

![Processus de création d’un script d’extraction](./media/data-lake-analytics-data-lake-tools-for-vscode/create-extract-script-process.png)

Le script d’extraction est généré d’après vos entrées. Pour empêcher un script de détecter les colonnes, choisissez l’une des deux options. Sans intervention de votre part, un seul script sera généré.

![Résultat de la création d’un script d’extraction](./media/data-lake-analytics-data-lake-tools-for-vscode/create-extract-script-result.png)

### <a name="to-create-an-extraction-script-from-the-explorer"></a>Pour créer un script d’extraction à partir de l’Explorateur

Pour créer le script d’extraction, vous pouvez également utiliser le menu contextuel (raccourci) sur le fichier .csv, . tsv ou .txt dans Azure Data Lake Store ou dans le stockage d’objets blob Azure.

![Commande « Create EXTRACT Script » du menu contextuel](./media/data-lake-analytics-data-lake-tools-for-vscode/create-extract-script-from-context-menu.png)

## <a name="next-steps"></a>Étapes suivantes

- [Développer en U-SQL avec Python, R et C Sharp pour Azure Data Lake Analytics dans VS Code](data-lake-analytics-u-sql-develop-with-python-r-csharp-in-vscode.md)
- [Exécution locale et débogage local U-SQL avec Visual Studio Code](data-lake-tools-for-vscode-local-run-and-debug.md)
- [Tutoriel : Prise en main d’Azure Data Lake Analytics](data-lake-analytics-get-started-portal.md)
- [Tutoriel : Développer des scripts U-SQL à l’aide de Data Lake Tools pour Visual Studio](data-lake-analytics-data-lake-tools-get-started.md)
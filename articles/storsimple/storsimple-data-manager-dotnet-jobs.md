---
title: Utiliser le kit SDK .NET pour les travaux Microsoft Azure StorSimple Data Manager
description: Découvrez comment utiliser le kit de développement logiciel (SDK) .NET dans le service StorSimple Data Manager pour transformer des données d’appareil StorSimple.
author: alkohli
ms.service: storsimple
ms.topic: how-to
ms.date: 01/16/2018
ms.author: alkohli
ms.openlocfilehash: b18627d2806662d6d966af95d51873d5623b2393
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96015534"
---
# <a name="use-the-net-sdk-to-initiate-data-transformation"></a>Utiliser le kit SDK .NET pour lancer la transformation de données

## <a name="overview"></a>Vue d’ensemble

Cet article vous explique comment utiliser la fonctionnalité de transformation des données au sein du service StorSimple Data Manager afin de transformer les données de l’appareil StorSimple. Les données transformées sont ensuite utilisées par d’autres services Azure dans le cloud.

Vous pouvez lancer une tâche de transformation de données de deux manières :

- Utiliser le kit de développement logiciel (SDK) .NET
- Utiliser le runbook Azure Automation
 
  Cet article décrit en détail comment créer un exemple d’application console .NET pour lancer une tâche de transformation de données et effectuer son suivi. Pour en savoir plus sur le lancement de la transformation de données via Automation, accédez à [Utiliser le runbook Azure Automation pour déclencher les tâches de transformation de données](storsimple-data-manager-job-using-automation.md).

## <a name="prerequisites"></a>Conditions préalables requises

Avant de commencer, assurez-vous de satisfaire les exigences suivantes :
*   Un ordinateur exécutant :

    - Visual Studio 2012, 2013, 2015 ou 2017.

    - Azure Powershell. [Téléchargez Azure Powershell](/powershell/azure/).
*   Une définition de tâche correctement configurée dans StorSimple Data Manager au sein d’un groupe de ressources.
*   Toutes les DLL requises. Téléchargez ces DLL à partir du [dépôt GitHub](https://github.com/Azure-Samples/storsimple-dotnet-data-manager-get-started/tree/master/Data_Manager_Job_Run/dlls).
*   Script [`Get-ConfigurationParams.ps1`](https://github.com/Azure-Samples/storsimple-dotnet-data-manager-get-started/blob/master/Data_Manager_Job_Run/Get-ConfigurationParams.ps1) du dépôt GitHub.

## <a name="step-by-step-procedure"></a>Procédure pas à pas

Procédez comme suit pour utiliser .NET pour lancer un travail de transformation de données.

1. Pour récupérer les paramètres de configuration, procédez comme suit :
    1. Téléchargez `Get-ConfigurationParams.ps1` à partir du script du dépôt GitHub dans `C:\DataTransformation`.
    1. Exécutez le script `Get-ConfigurationParams.ps1` à partir du dépôt GitHub. Tapez la commande suivante :

        ```
        C:\DataTransformation\Get-ConfigurationParams.ps1 -SubscriptionName "AzureSubscriptionName" -ActiveDirectoryKey "AnyRandomPassword" -AppName "ApplicationName"
         ```
        Vous pouvez saisir les valeurs de votre choix pour les éléments ActiveDirectoryKey et AppName.

2. Ce script génère les valeurs suivantes :
    * ID client
    * ID client
    * Clé Active Directory (identique à celle entrée ci-dessus)
    * Identifiant d’abonnement

        ![Sortie du script des paramètres de configuration](media/storsimple-data-manager-dotnet-jobs/get-config-parameters.png)

3. À l'aide de Visual Studio 2012, 2013 ou 2015, créez une application console Visual C# .NET.

    1. Lancez **Visual Studio 2012/2013/2015**.
    1. Sélectionnez **Fichier > Nouveau > Projet**.

        ![Créer un projet 1](media/storsimple-data-manager-dotnet-jobs/create-new-project-7.png)        
    2. Sélectionnez **Installé > Modèles > Visual C# > Application console**.
    3. Saisissez **DataTransformationApp** pour le **Nom**.
    4. Sélectionnez **C:\DataTransformation** pour l’**Emplacement**.
    6. Cliquez sur **OK** pour créer le projet.

        ![Créer un projet 2](media/storsimple-data-manager-dotnet-jobs/create-new-project-1.png)

4. Maintenant, ajoutez l’ensemble des DLL présentes dans le [dossier des DLL](https://github.com/Azure-Samples/storsimple-dotnet-data-manager-get-started/tree/master/Data_Manager_Job_Run/dlls) en tant que **Références** dans le projet que vous avez créé. Pour ajouter les fichiers DLL, effectuez les étapes suivantes :

   1. Dans Visual Studio, accédez à **Affichage > Explorateur de solutions**.
   2. Cliquez sur la flèche à gauche du projet de l’application de transformation des données. Cliquez sur **Références**, puis cliquez avec le bouton droit sur **Ajouter une référence**.
    
       ![Ajouter les DLL 1](media/storsimple-data-manager-dotnet-jobs/create-new-project-4.png)

   3. Accédez à l’emplacement du dossier des packages, sélectionnez l’ensemble des DLL, puis cliquez sur **Ajouter** et sur **OK**.

       ![Ajouter les DLL 2](media/storsimple-data-manager-dotnet-jobs/create-new-project-6.png)

5. Ajoutez les instructions **using** suivantes au fichier source (Program.cs) dans le projet.

    ```
    using System;
    using System.Collections.Generic;
    using System.Threading;
    using Microsoft.Azure.Management.HybridData.Models;
    using Microsoft.Internal.Dms.DmsWebJob;
    using Microsoft.Internal.Dms.DmsWebJob.Contracts;
    ```
    
6. Le code suivant initialise l’instance du travail de transformation de données. Add this in the **Méthode Main**. Remplacez les valeurs des paramètres de configuration obtenues précédemment. Insérez les valeurs de **ResourceGroupName** et **ResourceName**. **ResourceGroupName** est associé au service StorSimple Data Manager sur lequel la définition de tâche a été configurée. **ResourceName** est le nom de votre service StorSimple Data Manager.

    ```
    // Setup the configuration parameters.
    var configParams = new ConfigurationParams
    {
        ClientId = "client-id",
        TenantId = "tenant-id",
        ActiveDirectoryKey = "active-directory-key",
        SubscriptionId = "subscription-id",
        ResourceGroupName = "resource-group-name",
        ResourceName = "resource-name"
    };

    // Initialize the Data Transformation Job instance.
    DataTransformationJob dataTransformationJob = new DataTransformationJob(configParams);
    ```
   
7. Spécifiez les paramètres avec lesquels exécuter la définition de travail

    ```
    string jobDefinitionName = "job-definition-name";

    DataTransformationInput dataTransformationInput = dataTransformationJob.GetJobDefinitionParameters(jobDefinitionName);
    ```

    (OU)

    Si vous souhaitez modifier les paramètres de définition de travail durant l’exécution, ajoutez le code suivant :

    ```
    string jobDefinitionName = "job-definition-name";
    // Must start with a '\'
    var rootDirectories = new List<string> {@"\root"};

    // Name of the volume on the StorSimple device.
    var volumeNames = new List<string> {"volume-name"};

    var dataTransformationInput = new DataTransformationInput
    {
        // If you require the latest existing backup to be picked else use TakeNow to trigger a new backup.
        BackupChoice = BackupChoice.UseExistingLatest.ToString(),
        // Name of the StorSimple device.
        DeviceName = "device-name",
        // Name of the container in Azure storage where the files will be placed after execution.
        ContainerName = "container-name",
        // File name filter (search pattern) to be applied on files under the root directory. * - Match all files.
        FileNameFilter = "*",
        // List of root directories.
        RootDirectories = rootDirectories,
        // Name of the volume on StorSimple device on which the relevant data is present. 
        VolumeNames = volumeNames
    };
    ```

8. Après l’initialisation, ajoutez le code suivant pour déclencher un travail de transformation de données sur la définition de travail. Insérez le **Job Definition Name** (Nom de définition de travail).

    ```
    // Trigger a job, retrieve the jobId and the retry interval for polling.
    int retryAfter;
    string jobId = dataTransformationJob.RunJobAsync(jobDefinitionName, 
    dataTransformationInput, out retryAfter);
    Console.WriteLine("jobid: ", jobId);
    Console.ReadLine();

    ```
    Une fois que le code est collé, générez la solution. Voici une capture d’écran de l’extrait de code pour initialiser l’instance de tâche de transformation de données.

   ![Extrait de code pour initialiser la tâche de transformation de données](media/storsimple-data-manager-dotnet-jobs/start-dotnet-job-code-snippet-1.png)

9. Cette tâche transforme les données qui correspondent au répertoire racine et aux filtres de fichier dans le volume StorSimple et les place dans le partage de fichiers/conteneur spécifié. Quand un fichier est transformé, un message est ajouté à une file d’attente de stockage (dans le même compte de stockage que le partage de fichiers/conteneur) avec le même nom que la définition de tâche. Ce message peut être utilisé comme un déclencheur pour initier le traitement ultérieur du fichier.

10. Une fois la tâche déclenchée, vous pouvez utiliser le code suivant afin de suivre son exécution. Il n’est pas obligatoire d’ajouter ce code pour l’exécution de la tâche.

    ```
    Job jobDetails = null;

    // Poll the job.
    do
    {
        jobDetails = dataTransformationJob.GetJob(jobDefinitionName, jobId);

        // Wait before polling for the status again.
        Thread.Sleep(TimeSpan.FromSeconds(retryAfter));

    } while (jobDetails.Status == JobStatus.InProgress);

    // Completion status of the job.
    Console.WriteLine("JobStatus: {0}", jobDetails.Status);
    
    // To hold the console before exiting.
    Console.Read();

    ```
    Voici une capture d’écran de l’exemple de code entier utilisé pour déclencher la tâche à l’aide de .NET.

    ![Extrait de code complet pour déclencher une tâche .NET](media/storsimple-data-manager-dotnet-jobs/start-dotnet-job-code-snippet.png)

## <a name="next-steps"></a>Étapes suivantes

[Utilisez l’interface utilisateur de StorSimple Data Manager pour transformer vos données](storsimple-data-manager-ui.md).
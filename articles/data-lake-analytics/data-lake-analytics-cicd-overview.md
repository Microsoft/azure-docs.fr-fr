---
title: Comment configurer un pipeline CI/CD pour Azure Data Lake Analytics
description: Découvrez comment configurer l’intégration et le déploiement continus pour Azure Data Lake Analytics.
author: liudan66
ms.author: liud
ms.reviewer: jasonh
ms.service: data-lake-analytics
ms.topic: how-to
ms.date: 09/14/2018
ms.openlocfilehash: 95b638b85e0746d2995488f2a28a5fb2512b1063
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96015262"
---
# <a name="how-to-set-up-a-cicd-pipeline-for-azure-data-lake-analytics"></a>Comment configurer un pipeline CI/CD pour Azure Data Lake Analytics  

Dans cet article, vous allez voir comment implémenter un pipeline d’intégration et de déploiement continus (CI/CD) pour les travaux et les bases de données U-SQL.  

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="use-cicd-for-u-sql-jobs"></a>Utiliser CI/CD pour les travaux U-SQL

Azure Data Lake Tools pour Visual Studio fournit le type de projet U-SQL qui vous permet d’organiser des scripts U-SQL. L’utilisation du projet U-SQL pour gérer votre code U-SQL facilite la suite des scénarios de CI/CD.

## <a name="build-a-u-sql-project"></a>Générer un projet U-SQL

Un projet U-SQL peut être généré avec Microsoft Build Engine (MSBuild) en passant les paramètres correspondants. Suivez les étapes décrites dans cet article pour générer un projet U-SQL.

### <a name="project-migration"></a>Migration du projet

Avant de configurer la tâche de génération pour un projet U-SQL, vérifiez que vous utilisez bien la dernière version du projet U-SQL. Ouvrez le fichier projet U-SQL dans votre éditeur, puis vérifiez que les éléments d’importation suivants sont présents :

```xml
<!-- check for SDK Build target in current path then in USQLSDKPath-->
<Import Project="UsqlSDKBuild.targets" Condition="Exists('UsqlSDKBuild.targets')" />
<Import Project="$(USQLSDKPath)\UsqlSDKBuild.targets" Condition="!Exists('UsqlSDKBuild.targets') And '$(USQLSDKPath)' != '' And Exists('$(USQLSDKPath)\UsqlSDKBuild.targets')" />
```

Si ce n’est pas le cas, vous avez deux options pour migrer le projet :

- Option 1 : Remplacer l’ancien élément d’importation par le précédent.
- Option n°2 : Ouvrir l’ancien projet dans Azure Data Lake Tools pour Visual Studio. Utilisez une version plus récente que la version 2.3.3000.0. L’ancien modèle de projet sera automatiquement mis à niveau vers la dernière version. Les projets qui sont créés avec des versions plus récentes que la version 2.3.3000.0 utilisent le nouveau modèle.

### <a name="get-nuget"></a>Obtenir NuGet

MSBuild ne fournit pas la prise en charge intégrée des projets U-SQL. Pour bénéficier de cette prise en charge, vous devez ajouter une référence à votre solution au package NuGet [Microsoft.Azure.DataLake.USQL.SDK](https://www.nuget.org/packages/Microsoft.Azure.DataLake.USQL.SDK/) afin d’inclure le service de langage nécessaire.

Pour ajouter la référence au package NuGet, cliquez avec le bouton droit sur la solution dans l’Explorateur de solutions, puis choisissez **Gérer les packages NuGet**. Vous pouvez également ajouter un fichier appelé `packages.config` au dossier de la solution et y mettre le contenu suivant :

```xml
<?xml version="1.0" encoding="utf-8"?>
<packages>
  <package id="Microsoft.Azure.DataLake.USQL.SDK" version="1.3.180620" targetFramework="net452" />
</packages>
```

### <a name="manage-u-sql-database-references"></a>Gérer les références de base de données U-SQL

Les scripts U-SQL d’un projet U-SQL peuvent avoir des instructions de requête pour des objets de base de données U-SQL. Si c’est le cas, vous devez référencer le projet de base de données U-SQL correspondant qui inclut la définition des objets, avant de générer le projet U-SQL. Par exemple, lorsque vous interrogez une table U-SQL ou référencez un assembly.

En savoir plus sur le [projet de base de données U-SQL](data-lake-analytics-data-lake-tools-develop-usql-database.md)

>[!NOTE]
> L’instruction DROP peut entraîner une suppression accidentelle. Pour activer l’instruction DROP, vous devez spécifier explicitement les arguments MSBuild. **AllowDropStatement** activera l’opération DROP non liée aux données, telle que drop assembly et la fonction intéressante drop table. **AllowDropStatement** activera l’opération DROP liée aux données, telle que drop table et drop schema. Vous devez activer AllowDropStatement avant d’utiliser AllowDataDropStatement.
>

### <a name="build-a-u-sql-project-with-the-msbuild-command-line"></a>Générer un projet U-SQL à partir de la ligne de commande MSBuild

Tout d’abord, migrez le projet et obtenez le package NuGet. Ensuite, appelez la ligne de commande MSBuild standard avec les arguments supplémentaires suivants, afin de générer votre projet U-SQL :

```console
msbuild USQLBuild.usqlproj /p:USQLSDKPath=packages\Microsoft.Azure.DataLake.USQL.SDK.1.3.180615\build\runtime;USQLTargetType=SyntaxCheck;DataRoot=datarootfolder;/p:EnableDeployment=true
```

La définition et les valeurs des arguments sont :

- **USQLSDKPath=\<U-SQL Nuget package>\build\runtime**. Ce paramètre fait référence au chemin d’installation du package NuGet pour le service de langage U-SQL.
- **USQLTargetType=Merge or SyntaxCheck** :

  - **Merge**. Le mode Merge compile les fichiers code-behind. Il peut s’agir, par exemple, des fichiers **.cs**, **.py**, et **.r**. Il intègre dans le script U-SQL la bibliothèque de codes définie par l’utilisateur qui en résulte. Par exemple, un fichier binaire .dll, du code Python ou du code R.

  - **SyntaxCheck**. Le mode SyntaxCheck fusionne d’abord les fichiers code-behind dans le script U-SQL. Ensuite, il compile le script U-SQL pour valider votre code.

- **DataRoot=\<DataRoot path>** . DataRoot est nécessaire uniquement pour le mode SyntaxCheck. Lorsqu’il génère le script avec le mode SyntaxCheck, MSBuild vérifie les références aux objets de base de données dans le script. Avant de démarrer la génération, configurez un environnement local adapté contenant les objets référencés issus de la base de données U-SQL dans le dossier DataRoot de l’ordinateur de build. Vous pouvez également gérer ces dépendances de base de données en [référençant un projet de base de données U-SQL](data-lake-analytics-data-lake-tools-develop-usql-database.md#reference-a-u-sql-database-project). MSBuild vérifie uniquement les références aux objets de base de données, et pas les fichiers.

- **EnableDeployment=true** or **false**. EnableDeployment indique s’il est autorisé à déployer des bases de données U-SQL référencées pendant le processus de génération. Si vous référencez le projet de base de données U-SQL et que vous consommez les objets de base de données dans votre script U-SQL, définissez ce paramètre sur **true**.

### <a name="continuous-integration-through-azure-pipelines"></a>Intégration continue via Azure Pipelines

Outre la ligne de commande, vous pouvez également utiliser une tâche Visual Studio Build ou MSBuild pour générer des projets U-SQL dans Azure Pipelines. Pour configurer un pipeline de build, ajoutez-y deux tâches : une tâche de restauration NuGet et une tâche MSBuild.

![Tâche MSBuild pour un projet U-SQL](./media/data-lake-analytics-cicd-overview/data-lake-analytics-set-vsts-msbuild-task.png)

1. Ajoutez une tâche de restauration NuGet pour obtenir le package NuGet référencé par la solution, notamment `Azure.DataLake.USQL.SDK`, de sorte que MSBuild puisse rechercher les cibles de langage U-SQL. Définissez **Avancé** > **répertoire Destination** sur `$(Build.SourcesDirectory)/packages` si vous souhaitez utiliser les exemples d’arguments MSBuild directement à l’étape 2.

   ![Tâche de restauration NuGet pour un projet U-SQL](./media/data-lake-analytics-cicd-overview/data-lake-analytics-set-vsts-nuget-task.png)

2. Définissez les arguments MSBuild dans Visual Studio Build Tools ou dans une tâche MSBuild, comme indiqué dans l’exemple suivant. Vous pouvez également définir des variables pour ces arguments dans le pipeline de build Azure Pipelines.

   ![Définir des variables CI/CD MSBuild pour un projet U-SQL](./media/data-lake-analytics-cicd-overview/data-lake-analytics-set-vsts-msbuild-variables.png)

   ```console
   /p:USQLSDKPath=$(Build.SourcesDirectory)/packages/Microsoft.Azure.DataLake.USQL.SDK.1.3.180615/build/runtime /p:USQLTargetType=SyntaxCheck /p:DataRoot=$(Build.SourcesDirectory) /p:EnableDeployment=true
   ```

### <a name="u-sql-project-build-output"></a>Sortie de build de projet U-SQL

Après avoir exécuté la génération, tous les scripts du projet U-SQL sont générés et sortis dans un fichier .zip nommé `USQLProjectName.usqlpack`. La structure des dossiers de votre projet se trouve dans la sortie de build compressée.

> [!NOTE]
> Pour chaque script U-SQL, les fichiers code-behind sont fusionnés sous la forme d’une instruction incluse dans la sortie de build du script.

## <a name="test-u-sql-scripts"></a>Tester les scripts U-SQL

Azure Data Lake fournit des projets de test pour les scripts U-SQL et C# UDO/UDAG/UDF :

- Apprenez à [ajouter des cas de test pour les scripts U-SQL et le code C# étendu](data-lake-analytics-cicd-test.md#test-u-sql-scripts)

- Découvrez comment [exécuter des cas de test dans Azure Pipelines](data-lake-analytics-cicd-test.md#run-test-cases-in-azure-devops).

## <a name="deploy-a-u-sql-job"></a>Déployer un travail U-SQL

Après avoir vérifié le code via le processus de build et de test, vous pouvez envoyer les travaux U-SQL directement d’Azure Pipelines à la tâche Azure PowerShell. Vous pouvez également déployer le script dans le stockage Blob Azure ou Azure Data Lake Store, et [exécuter les travaux planifiés via Azure Data Factory](../data-factory/transform-data-using-data-lake-analytics.md).

### <a name="submit-u-sql-jobs-through-azure-pipelines"></a>Envoyer des travaux U-SQL via Azure Pipelines

La sortie de build du projet U-SQL est un fichier .zip nommé **USQLProjectName.usqlpack**. Le fichier .zip comprend tous les scripts U-SQL du projet. Vous pouvez utiliser la [tâche Azure PowerShell](/azure/devops/pipelines/tasks/deploy/azure-powershell) dans Pipelines avec l’exemple de script PowerShell suivant pour envoyer des travaux U-SQL directement à partir d’Azure Pipelines.

```powershell
<#
   This script can be used to submit U-SQL Jobs with given U-SQL project build output(.usqlpack file).
   This will unzip the U-SQL project build output, and submit all scripts one-by-one.

   Note: the code behind file for each U-SQL script will be merged into the built U-SQL script in build output.

   Example :
      USQLJobSubmission.ps1 -ADLAAccountName "myadlaaccount" -ArtifactsRoot "C:\USQLProject\bin\debug\" -DegreeOfParallelism 2
#>

param(
   [Parameter(Mandatory=$true)][string]$ADLAAccountName, # ADLA account name to submit U-SQL jobs
   [Parameter(Mandatory=$true)][string]$ArtifactsRoot, # Root folder of U-SQL project build output
   [Parameter(Mandatory=$false)][string]$DegreeOfParallelism = 1
)

function Unzip($USQLPackfile, $UnzipOutput)
{
   $USQLPackfileZip = Rename-Item -Path $USQLPackfile -NewName $([System.IO.Path]::ChangeExtension($USQLPackfile, ".zip")) -Force -PassThru
   Expand-Archive -Path $USQLPackfileZip -DestinationPath $UnzipOutput -Force
   Rename-Item -Path $USQLPackfileZip -NewName $([System.IO.Path]::ChangeExtension($USQLPackfileZip, ".usqlpack")) -Force
}

## Get U-SQL scripts in U-SQL project build output(.usqlpack file)
Function GetUsqlFiles()
{

   $USQLPackfiles = Get-ChildItem -Path $ArtifactsRoot -Include *.usqlpack -File -Recurse -ErrorAction SilentlyContinue

   $UnzipOutput = Join-Path $ArtifactsRoot -ChildPath "UnzipUSQLScripts"

   foreach ($USQLPackfile in $USQLPackfiles)
   {
      Unzip $USQLPackfile $UnzipOutput
   }

   $USQLFiles = Get-ChildItem -Path $UnzipOutput -Include *.usql -File -Recurse -ErrorAction SilentlyContinue

   return $USQLFiles
}

## Submit U-SQL scripts to ADLA account one-by-one
Function SubmitAnalyticsJob()
{
   $usqlFiles = GetUsqlFiles

   Write-Output "$($usqlFiles.Count) jobs to be submitted..."

   # Submit each usql script and wait for completion before moving ahead.
   foreach ($usqlFile in $usqlFiles)
   {
      $scriptName = "[Release].[$([System.IO.Path]::GetFileNameWithoutExtension($usqlFile.fullname))]"

      Write-Output "Submitting job for '{$usqlFile}'"

      $jobToSubmit = Submit-AzDataLakeAnalyticsJob -Account $ADLAAccountName -Name $scriptName -ScriptPath $usqlFile -DegreeOfParallelism $DegreeOfParallelism

      LogJobInformation $jobToSubmit

      Write-Output "Waiting for job to complete. Job ID:'{$($jobToSubmit.JobId)}', Name: '$($jobToSubmit.Name)' "
      $jobResult = Wait-AzDataLakeAnalyticsJob -Account $ADLAAccountName -JobId $jobToSubmit.JobId  
      LogJobInformation $jobResult
   }
}

Function LogJobInformation($jobInfo)
{
   Write-Output "************************************************************************"
   Write-Output ([string]::Format("Job Id: {0}", $(DefaultIfNull $jobInfo.JobId)))
   Write-Output ([string]::Format("Job Name: {0}", $(DefaultIfNull $jobInfo.Name)))
   Write-Output ([string]::Format("Job State: {0}", $(DefaultIfNull $jobInfo.State)))
   Write-Output ([string]::Format("Job Started at: {0}", $(DefaultIfNull  $jobInfo.StartTime)))
   Write-Output ([string]::Format("Job Ended at: {0}", $(DefaultIfNull  $jobInfo.EndTime)))
   Write-Output ([string]::Format("Job Result: {0}", $(DefaultIfNull $jobInfo.Result)))
   Write-Output "************************************************************************"
}

Function DefaultIfNull($item)
{
   if ($item -ne $null)
   {
      return $item
   }
   return ""
}

Function Main()
{
   Write-Output ([string]::Format("ADLA account: {0}", $ADLAAccountName))
   Write-Output ([string]::Format("Root folde for usqlpack: {0}", $ArtifactsRoot))
   Write-Output ([string]::Format("AU count: {0}", $DegreeOfParallelism))

   Write-Output "Starting USQL script deployment..."

   SubmitAnalyticsJob

   Write-Output "Finished deployment..."
}

Main
```

>[!NOTE]
> Les commandes : `Submit-AzDataLakeAnalyticsJob` et `Wait-AzDataLakeAnalyticsJob` sont toutes deux des applets de commande Azure PowerShell pour Azure Data Lake Analytics dans le framework Azure Resource Manager. Vous avez besoin d’une station de travail sur laquelle Azure PowerShell est installé. Vous pouvez vous reporter à la [liste de commandes](/powershell/module/Az.DataLakeAnalytics) pour obtenir des commandes et des exemples supplémentaires.
>

### <a name="deploy-u-sql-jobs-through-azure-data-factory"></a>Déployer des travaux U-SQL via Azure Data Factory

Vous pouvez envoyer des travaux U-SQL directement à partir d’Azure Pipelines. Vous pouvez également charger les scripts générés vers le stockage Blob Azure ou Azure Data Lake Store, et [exécuter les travaux planifiés via Azure Data Factory](../data-factory/transform-data-using-data-lake-analytics.md).

Utilisez la [tâche Azure PowerShell](/azure/devops/pipelines/tasks/deploy/azure-powershell) dans Azure Pipelines avec l’exemple de script PowerShell suivant, afin de charger les scripts U-SQL vers un compte Azure Data Lake Store :

```powershell
<#
   This script can be used to upload U-SQL files to ADLS with given U-SQL project build output(.usqlpack file).
   This will unzip the U-SQL project build output, and upload all scripts to ADLS one-by-one.

   Example :
      FileUpload.ps1 -ADLSName "myadlsaccount" -ArtifactsRoot "C:\USQLProject\bin\debug\"
#>

param(
   [Parameter(Mandatory=$true)][string]$ADLSName, # ADLS account name to upload U-SQL scripts
   [Parameter(Mandatory=$true)][string]$ArtifactsRoot, # Root folder of U-SQL project build output
   [Parameter(Mandatory=$false)][string]$DestinationFolder = "USQLScriptSource" # Destination folder in ADLS
)

Function UploadResources()
{
   Write-Host "************************************************************************"
   Write-Host "Uploading files to $ADLSName"
   Write-Host "***********************************************************************"

   $usqlScripts = GetUsqlFiles

   $files = @(get-childitem $usqlScripts -recurse)
   foreach($file in $files)
   {
      Write-Host "Uploading file: $($file.Name)"
      Import-AzDataLakeStoreItem -AccountName $ADLSName -Path $file.FullName -Destination "/$(Join-Path $DestinationFolder $file)" -Force
   }
}

function Unzip($USQLPackfile, $UnzipOutput)
{
   $USQLPackfileZip = Rename-Item -Path $USQLPackfile -NewName $([System.IO.Path]::ChangeExtension($USQLPackfile, ".zip")) -Force -PassThru
   Expand-Archive -Path $USQLPackfileZip -DestinationPath $UnzipOutput -Force
   Rename-Item -Path $USQLPackfileZip -NewName $([System.IO.Path]::ChangeExtension($USQLPackfileZip, ".usqlpack")) -Force
}

Function GetUsqlFiles()
{

   $USQLPackfiles = Get-ChildItem -Path $ArtifactsRoot -Include *.usqlpack -File -Recurse -ErrorAction SilentlyContinue

   $UnzipOutput = Join-Path $ArtifactsRoot -ChildPath "UnzipUSQLScripts"

   foreach ($USQLPackfile in $USQLPackfiles)
   {
      Unzip $USQLPackfile $UnzipOutput
   }

   return Get-ChildItem -Path $UnzipOutput -Include *.usql -File -Recurse -ErrorAction SilentlyContinue
}

UploadResources
```

## <a name="cicd-for-a-u-sql-database"></a>CI/CD pour une base de données U-SQL

Azure Data Lake Tools pour Visual Studio fournit des modèles de projet de base de données U-SQL, qui permettent de développer, gérer et déployer des bases de données U-SQL. En savoir plus sur le [projet de base de données U-SQL](data-lake-analytics-data-lake-tools-develop-usql-database.md)

## <a name="build-u-sql-database-project"></a>Générer un projet de base de données U-SQL

### <a name="get-the-nuget-package"></a>Obtenir le package NuGet

MSBuild ne fournit pas la prise en charge intégrée des projets de base de données U-SQL. Pour bénéficier de cette possibilité, vous devez ajouter une référence à votre solution au package NuGet [Microsoft.Azure.DataLake.USQL.SDK](https://www.nuget.org/packages/Microsoft.Azure.DataLake.USQL.SDK/) afin d’inclure le service de langage nécessaire.

Pour ajouter la référence au package NuGet, cliquez avec le bouton droit sur la solution dans l’Explorateur de solutions Visual Studio. Choisissez **Gérer les packages NuGet**. Ensuite, recherchez et installez le package NuGet. Vous pouvez également ajouter un fichier appelé **packages.config** dans le dossier de la solution, et y mettre le contenu suivant :

```xml
<?xml version="1.0" encoding="utf-8"?>
<packages>
  <package id="Microsoft.Azure.DataLake.USQL.SDK" version="1.3.180615" targetFramework="net452" />
</packages>
```

### <a name="build-u-sql-a-database-project-with-the-msbuild-command-line"></a>Générer un projet de base de données U-SQL avec la ligne de commande MSBuild

Pour générer votre projet de base de données U-SQL, appelez la ligne de commande MSBuild standard, puis passez la référence du package NuGet du SDK U-SQL en tant qu’argument supplémentaire. Voir l’exemple suivant : 

```console
msbuild DatabaseProject.usqldbproj /p:USQLSDKPath=packages\Microsoft.Azure.DataLake.USQL.SDK.1.3.180615\build\runtime
```

L’argument `USQLSDKPath=<U-SQL Nuget package>\build\runtime` fait référence au chemin d’installation du package NuGet pour le service de langage U-SQL.

### <a name="continuous-integration-with-azure-pipelines"></a>Intégration continue avec Azure Pipelines

Outre la ligne de commande, vous pouvez utiliser une tâche Visual Studio Build ou MSBuild pour générer des projets de base de données U-SQL dans Azure Pipelines. Pour configurer une tâche de génération, ajoutez deux tâches dans le pipeline de build : une tâche de restauration NuGet et une tâche MSBuild.

   ![Tâche CI/CD MSBuild pour un projet U-SQL](./media/data-lake-analytics-cicd-overview/data-lake-analytics-set-vsts-msbuild-task.png)

1. Ajoutez une tâche de restauration NuGet pour obtenir le package NuGet référencé par la solution, notamment `Azure.DataLake.USQL.SDK`, de sorte que MSBuild puisse rechercher les cibles de langage U-SQL. Définissez **Avancé** > **répertoire Destination** sur `$(Build.SourcesDirectory)/packages` si vous souhaitez utiliser les exemples d’arguments MSBuild directement à l’étape 2.

   ![Tâche CI/CD NuGet pour un projet U-SQL](./media/data-lake-analytics-cicd-overview/data-lake-analytics-set-vsts-nuget-task.png)

2. Définissez les arguments MSBuild dans Visual Studio Build Tools ou dans une tâche MSBuild, comme indiqué dans l’exemple suivant. Vous pouvez également définir des variables pour ces arguments dans le pipeline de build Azure Pipelines.

   ![Définir des variables CI/CD MSBuild pour un projet de base de données U-SQL](./media/data-lake-analytics-cicd-overview/data-lake-analytics-set-vsts-msbuild-variables-database-project.png)

   ```console
   /p:USQLSDKPath=$(Build.SourcesDirectory)/packages/Microsoft.Azure.DataLake.USQL.SDK.1.3.180615/build/runtime
   ```

### <a name="u-sql-database-project-build-output"></a>Sortie de build de projet de base de données U-SQL

La sortie de build du projet de base de données U-SQL est un package de déploiement de base de données U-SQL dont le nom est suivi du suffixe `.usqldbpack`. Le package `.usqldbpack` est un fichier .zip qui rassemble toutes les instructions DDL dans un même script U-SQL, situé dans un dossier DDL. Il inclut tous les fichiers **.dll**, ainsi que d’autres fichiers d’assembly dans un dossier temporaire.

## <a name="test-table-valued-functions-and-stored-procedures"></a>Tester des fonctions table et des procédures stockées

L’ajout direct de cas de test pour les fonctions table et les procédures stockées n’est pas pris en charge. Pour contourner ce problème, vous pouvez créer un projet U-SQL contenant des scripts U-SQL qui appellent ces fonctions, et écrire des cas de test pour elles. Suivez les étapes ci-dessous afin de configurer des cas de test pour les fonctions table et les procédures stockées définies dans le projet de base de données U-SQL :

1. Créez un projet U-SQL à des fins de test et écrivez des scripts U-SQL qui appellent les fonctions de table et les procédures stockées.

1. Ajoutez une référence de base de données au projet U-SQL. Pour obtenir la définition de la fonction table et de la procédure stockée, vous devez référencer le projet de base de données qui contient l’instruction DDL. En savoir plus sur les [références de base de données](data-lake-analytics-data-lake-tools-develop-usql-database.md#reference-a-u-sql-database-project).

1. Ajoutez des cas de test pour les scripts U-SQL qui appellent des fonctions table et des procédures stockées. Découvrez comment [ajouter des cas de test pour les scripts U-SQL](data-lake-analytics-cicd-test.md#test-u-sql-scripts).

## <a name="deploy-u-sql-database-through-azure-pipelines"></a>Déployer une base de données U-SQL via Azure Pipelines

`PackageDeploymentTool.exe` fournit les interfaces de programmation et de ligne de commande qui vous permettent de déployer les packages de base de données U-SQL **.usqldbpack**. Le SDK est inclus dans le [package NuGet du SDK U-SQL](https://www.nuget.org/packages/Microsoft.Azure.DataLake.USQL.SDK/), situé à l’emplacement **build/runtime/PackageDeploymentTool.exe**. Avec `PackageDeploymentTool.exe`, vous pouvez déployer des bases de données U-SQL dans Azure Data Lake Analytics et des comptes locaux.

> [!NOTE]
>
> La prise en charge de la ligne de commande PowerShell et de la tâche de mise en production Azure Pipelines pour le déploiement d’une base de données U-SQL est en cours de réalisation.
>

Suivez les étapes ci-dessous pour configurer une tâche de déploiement d’une base de données dans Azure Pipelines :

1. Ajoutez une tâche de script PowerShell au pipeline de build ou de mise en production, puis exécutez le script PowerShell suivant. Cette tâche permet d’obtenir des dépendances du kit SDK Azure pour `PackageDeploymentTool.exe` et `PackageDeploymentTool.exe`. Vous pouvez définir les paramètres **-AzureSDK** et **-DBDeploymentTool** pour charger les dépendances et l’outil de déploiement dans différents dossiers. Passez le chemin **-AzureSDK** à `PackageDeploymentTool.exe` en tant que paramètre **-AzureSDKPath** à l’étape 2.

   ```powershell
   <#
      This script is used for getting dependencies and SDKs for U-SQL database deployment.
      PowerShell command line support for deploying U-SQL database package(.usqldbpack file) will come soon.

      Example :
          GetUSQLDBDeploymentSDK.ps1 -AzureSDK "AzureSDKFolderPath" -DBDeploymentTool "DBDeploymentToolFolderPath"
   #>

   param (
      [string]$AzureSDK = "AzureSDK", # Folder to cache Azure SDK dependencies
      [string]$DBDeploymentTool = "DBDeploymentTool", # Folder to cache U-SQL database deployment tool
      [string]$workingfolder = "" # Folder to execute these command lines
   )

   if ([string]::IsNullOrEmpty($workingfolder))
   {
      $scriptpath = $MyInvocation.MyCommand.Path
      $workingfolder = Split-Path $scriptpath
   }
   cd $workingfolder

   echo "workingfolder=$workingfolder, outputfolder=$outputfolder"
   echo "Downloading required packages..."

   iwr https://www.nuget.org/api/v2/package/Microsoft.Azure.Management.DataLake.Analytics/3.5.1-preview -outf Microsoft.Azure.Management.DataLake.Analytics.3.5.1-preview.zip
   iwr https://www.nuget.org/api/v2/package/Microsoft.Azure.Management.DataLake.Store/2.4.1-preview -outf Microsoft.Azure.Management.DataLake.Store.2.4.1-preview.zip
   iwr https://www.nuget.org/api/v2/package/Microsoft.IdentityModel.Clients.ActiveDirectory/2.28.3 -outf Microsoft.IdentityModel.Clients.ActiveDirectory.2.28.3.zip
   iwr https://www.nuget.org/api/v2/package/Microsoft.Rest.ClientRuntime/2.3.11 -outf Microsoft.Rest.ClientRuntime.2.3.11.zip
   iwr https://www.nuget.org/api/v2/package/Microsoft.Rest.ClientRuntime.Azure/3.3.7 -outf Microsoft.Rest.ClientRuntime.Azure.3.3.7.zip
   iwr https://www.nuget.org/api/v2/package/Microsoft.Rest.ClientRuntime.Azure.Authentication/2.3.3 -outf Microsoft.Rest.ClientRuntime.Azure.Authentication.2.3.3.zip
   iwr https://www.nuget.org/api/v2/package/Newtonsoft.Json/6.0.8 -outf Newtonsoft.Json.6.0.8.zip
   iwr https://www.nuget.org/api/v2/package/Microsoft.Azure.DataLake.USQL.SDK/ -outf USQLSDK.zip

   echo "Extracting packages..."

   Expand-Archive Microsoft.Azure.Management.DataLake.Analytics.3.5.1-preview.zip -DestinationPath Microsoft.Azure.Management.DataLake.Analytics.3.5.1-preview -Force
   Expand-Archive Microsoft.Azure.Management.DataLake.Store.2.4.1-preview.zip -DestinationPath Microsoft.Azure.Management.DataLake.Store.2.4.1-preview -Force
   Expand-Archive Microsoft.IdentityModel.Clients.ActiveDirectory.2.28.3.zip -DestinationPath Microsoft.IdentityModel.Clients.ActiveDirectory.2.28.3 -Force
   Expand-Archive Microsoft.Rest.ClientRuntime.2.3.11.zip -DestinationPath Microsoft.Rest.ClientRuntime.2.3.11 -Force
   Expand-Archive Microsoft.Rest.ClientRuntime.Azure.3.3.7.zip -DestinationPath Microsoft.Rest.ClientRuntime.Azure.3.3.7 -Force
   Expand-Archive Microsoft.Rest.ClientRuntime.Azure.Authentication.2.3.3.zip -DestinationPath Microsoft.Rest.ClientRuntime.Azure.Authentication.2.3.3 -Force
   Expand-Archive Newtonsoft.Json.6.0.8.zip -DestinationPath Newtonsoft.Json.6.0.8 -Force
   Expand-Archive USQLSDK.zip -DestinationPath USQLSDK -Force

   echo "Copy required DLLs to output folder..."

   mkdir $AzureSDK -Force
   mkdir $DBDeploymentTool -Force
   copy Microsoft.Azure.Management.DataLake.Analytics.3.5.1-preview\lib\net452\*.dll $AzureSDK
   copy Microsoft.Azure.Management.DataLake.Store.2.4.1-preview\lib\net452\*.dll $AzureSDK
   copy Microsoft.IdentityModel.Clients.ActiveDirectory.2.28.3\lib\net45\*.dll $AzureSDK
   copy Microsoft.Rest.ClientRuntime.2.3.11\lib\net452\*.dll $AzureSDK
   copy Microsoft.Rest.ClientRuntime.Azure.3.3.7\lib\net452\*.dll $AzureSDK
   copy Microsoft.Rest.ClientRuntime.Azure.Authentication.2.3.3\lib\net452\*.dll $AzureSDK
   copy Newtonsoft.Json.6.0.8\lib\net45\*.dll $AzureSDK
   copy USQLSDK\build\runtime\*.* $DBDeploymentTool
   ```

2. Ajoutez une **tâche de ligne de commande** au pipeline de build ou de mise en production, puis remplissez le script en appelant `PackageDeploymentTool.exe`. `PackageDeploymentTool.exe` se trouve dans le dossier **$DBDeploymentTool** défini. L’exemple de script U-SQL est le suivant : 

   - Déployez localement une base de données U-SQL :

      ```cmd
      PackageDeploymentTool.exe deploylocal -Package <package path> -Database <database name> -DataRoot <data root path>
      ```

   - Utilisez le mode d’authentification interactive pour déployer une base de données U-SQL dans un compte Azure Data Lake Analytics :

      ```cmd
      PackageDeploymentTool.exe deploycluster -Package <package path> -Database <database name> -Account <account name> -ResourceGroup <resource group name> -SubscriptionId <subscript id> -Tenant <tenant name> -AzureSDKPath <azure sdk path> -Interactive
      ```

   - Utilisez l’authentification par **secrète** pour déployer une base de données U-SQL dans un compte Azure Data Lake Analytics :

      ```cmd
      PackageDeploymentTool.exe deploycluster -Package <package path> -Database <database name> -Account <account name> -ResourceGroup <resource group name> -SubscriptionId <subscript id> -Tenant <tenant name> -ClientId <client id> -Secrete <secrete>
      ```

   - Utilisez l’authentification par **certFile** pour déployer une base de données U-SQL dans un compte Azure Data Lake Analytics :

      ```cmd
      PackageDeploymentTool.exe deploycluster -Package <package path> -Database <database name> -Account <account name> -ResourceGroup <resource group name> -SubscriptionId <subscript id> -Tenant <tenant name> -ClientId <client id> -Secrete <secrete> -CertFile <certFile>
      ```

### <a name="packagedeploymenttoolexe-parameter-descriptions"></a>Description du paramètre PackageDeploymentTool.exe

#### <a name="common-parameters"></a>Paramètres communs

| Paramètre | Description | Valeur par défaut | Obligatoire |
|---------|-----------|-------------|--------|
|Package|Chemin du package de déploiement de base de données U-SQL à déployer|null|true|
|Base de données|Nom de la base données à créer ou dans laquelle le déploiement doit être effectué|master|false|
|LogFile|Chemin du fichier de journalisation. Par défaut, défini sur sortie standard (console).|null|false|
|LogLevel|Niveau de journal : détaillé, normal, avertissement ou erreur.|LogLevel.Normal|false|

#### <a name="parameter-for-local-deployment"></a>Paramètre pour le déploiement local

|Paramètre|Description|Valeur par défaut|Obligatoire|
|---------|-----------|-------------|--------|
|DataRoot|Chemin du dossier racine de données local|null|true|

#### <a name="parameters-for-azure-data-lake-analytics-deployment"></a>Paramètres pour le déploiement Azure Data Lake Analytics

|Paramètre|Description|Valeur par défaut|Obligatoire|
|---------|-----------|-------------|--------|
|Compte|Spécifie le compte Azure Data Lake Analytics dans lequel effectuer le déploiement, en le désignant par son nom.|null|true|
|ResourceGroup|Nom du groupe de ressources Azure pour le compte Azure Data Lake Analytics.|null|true|
|SubscriptionId|ID d’abonnement Azure pour le compte Azure Data Lake Analytics.|null|true|
|Locataire|Le nom du locataire correspondant au nom du domaine Azure Active Directory (Azure AD). Il se trouve dans la page de gestion des abonnements du portail Azure.|null|true|
|AzureSDKPath|Chemin pour rechercher des assemblys dépendants dans le SDK Azure|null|true|
|Interactive|Utiliser ou non le mode interactif pour l’authentification.|false|false|
|ClientId|L’ID d’application Azure AD est nécessaire pour l’authentification non interactive.|null|Nécessaire pour l’authentification non interactive.|
|Authentification secrète|Authentification secrète ou mot de passe pour l’authentification non interactive. Doit être utilisé uniquement dans un environnement sécurisé et approuvé.|null|Nécessaire pour l’authentification non interactive. Sinon, utilisez SecreteFile.|
|SecreteFile|Le fichier enregistre la clé secrète ou le mot de passe pour l’authentification non interactive. Faites en sorte qu’il ne soit lisible que par l’utilisateur actuel.|null|Nécessaire pour l’authentification non interactive. Sinon, utilisez l’authentification secrète.|
|CertFile|Le fichier enregistre la certification X.509 pour l’authentification non interactive. La valeur par défaut consiste à utiliser l’authentification par clé secrète client.|null|false|
| JobPrefix | Préfixe pour le déploiement de base de données d’un travail DDL U-SQL. | Deploy_ + DateTime.Now | false |

## <a name="next-steps"></a>Étapes suivantes

- [Guide pratique pour tester votre code Azure Data Lake Analytics](data-lake-analytics-cicd-test.md)
- [Exécuter un script U-SQL sur votre ordinateur local](data-lake-analytics-data-lake-tools-local-run.md)
- [Utiliser un projet de base de données U-SQL pour développer une base de données U-SQL](data-lake-analytics-data-lake-tools-develop-usql-database.md)
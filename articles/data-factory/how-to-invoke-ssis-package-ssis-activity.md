---
title: Exécuter un package SSIS avec l’activité Exécuter le Package SSIS
description: Cet article décrit comment exécuter un package SQL Server Integration Services (SSIS) dans un pipeline Azure Data Factory à l’aide de l’activité Exécuter le Package SSIS.
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: ''
ms.devlang: powershell
ms.topic: conceptual
ms.author: sawinark
author: swinarko
ms.reviewer: douglasl
manager: mflasko
ms.custom: seo-lt-2019, devx-track-azurepowershell
ms.date: 07/20/2020
ms.openlocfilehash: 1bd983f7faeff456b04d4b2958236193a827a2cc
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92635012"
---
# <a name="run-an-ssis-package-with-the-execute-ssis-package-activity-in-azure-data-factory"></a>Exécuter un package SSIS avec l’activité Exécuter le Package SSIS dans Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Cet article décrit comment exécuter un package SQL Server Integration Services (SSIS) dans un pipeline Azure Data Factory à l’aide de l’activité Exécuter le Package SSIS. 

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Créez un runtime d’intégration Azure-SSIS si vous n’en avez pas encore en suivant les instructions pas à pas fournies dans le [Tutoriel : Approvisionnement de votre IR Azure-SSIS](./tutorial-deploy-ssis-packages-azure.md).

## <a name="run-a-package-in-the-azure-portal"></a>Exécuter un package dans le portail Azure
Dans cette section, vous utilisez l’interface utilisateur ou l’application de Data Factory pour créer un pipeline Data Factory avec une activité Exécuter le Package SSIS qui exécute votre package SSIS.

### <a name="create-a-pipeline-with-an-execute-ssis-package-activity"></a>Créer un pipeline avec une activité Exécuter le Package SSIS
Lors de cette étape, vous utilisez l’interface utilisateur ou l’application Data Factory pour créer un pipeline. Vous ajoutez une activité Exécuter le Package SSIS au pipeline et la configurez pour exécuter votre package SSIS. 

1. Sur votre vue d’ensemble Data Factory ou sur la page d’accueil du portail Azure, sélectionnez la vignette **Créer et surveiller** pour démarrer l’application ou l’interface utilisateur Data Factory dans un onglet séparé. 

   ![Page d’accueil Data Factory](./media/how-to-invoke-ssis-package-stored-procedure-activity/data-factory-home-page.png)

   Dans la page **Prise en main** , cliquez sur **Créer un pipeline** . 

   ![Page de prise en main](./media/how-to-invoke-ssis-package-stored-procedure-activity/get-started-page.png)

1. Dans la boîte à outils **Activités** , étendez **Général** . Ensuitez, faites glisser une activité **Execute SSIS Package** jusqu’à l’aire du concepteur de pipeline. 

   ![Faire glisser une activité Execute SSIS Package jusqu’à l’aire du concepteur](media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-designer.png) 

   Sélectionnez l'objet d'activité Exécuter le package SSIS pour configurer les onglets **Général** , **Paramètres** , **Paramètres SSIS** , **Gestionnaires de connexions** et **Substitutions de propriété** correspondants.

#### <a name="general-tab"></a>Onglet Général

Sous l'onglet **Général** de l'activité Exécuter le package SSIS, procédez comme suit.

![Définissez les propriétés sous l’onglet Général](media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-general.png)

   1. Dans le champ **Nom** , entrez le nom de votre activité Exécuter le package SSIS.

   1. Dans le champ **Description** , entrez la description de votre activité Exécuter le package SSIS.

   1. Dans le champ **Délai d'expiration** , entrez la durée maximale pendant laquelle votre activité Exécuter le package SSIS peut être exécutée. La valeur par défaut est de 7 jours, et le format est J.HH:MM:SS.

   1. Dans le champ **Nouvelle tentative** , entrez le nombre maximal de tentatives d'exécution de votre activité Exécuter le package SSIS.

   1. Dans le champ **Intervalle avant nouvelle tentative** , entrez le nombre de secondes qui doivent s'écouler entre chaque tentative d'exécution de votre activité Exécuter le package SSIS. La valeur par défaut est de 30 secondes.

   1. Cochez la case **Sortie sécurisée** pour déterminer si vous souhaitez exclure la sortie de votre activité Exécuter le package SSIS de la journalisation.

   1. Cochez la case **Entrée sécurisée** pour déterminer si vous souhaitez exclure l'entrée de votre activité Exécuter le package SSIS de la journalisation.

#### <a name="settings-tab"></a>Onglet Paramètres

Sous l'onglet **Paramètres** de l'activité Exécuter le package SSIS, procédez comme suit.

![Définir les propriétés sous l’onglet Paramètres - Automatique](media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-settings.png)

   1. Dans le champ **Azure-SSIS IR** , sélectionnez le runtime d'intégration Azure-SSIS désigné pour lancer votre activité Exécuter le package SSIS.

   1. Dans le champ **Description** , entrez la description de votre activité Exécuter le package SSIS.

   1. Cochez la case **Authentification Windows** afin de déterminer si vous souhaitez utiliser l'authentification Windows pour accéder aux magasins de données, comme les serveurs SQL/partages de fichiers locaux ou Azure Files.
   
      Si vous cochez cette case, entrez les valeurs des informations d'identification requises pour l'exécution de votre package dans les champs **Domaine** , **Nom d'utilisateur** et **Mot de passe** . Par exemple, pour accéder à Azure Files, le domaine, le nom d'utilisateur et le mot de passe sont respectivement `Azure`, `<storage account name>` et `<storage account key>`.

      Vous pouvez également utiliser les secrets stockés dans votre coffre de clés Azure Key Vault comme valeurs. Pour ce faire, cochez la case **AZURE KEY VAULT** en regard de ceux-ci. Sélectionnez ou modifiez votre service de coffre de clés lié existant ou créez-en un. Sélectionnez ensuite la version et le nom du secret correspondant à votre valeur. Lorsque vous créez ou modifiez votre service de coffre de clés lié, vous pouvez sélectionner ou modifier votre coffre de clés existant ou en créer un nouveau. Veillez à accorder à Data Factory l’accès géré à votre coffre de clés si vous ne l’avez pas déjà fait. Vous pouvez également entrer votre secret directement au format suivant : `<key vault linked service name>/<secret name>/<secret version>`.
      
   1. Cochez la case **Runtime 32 bits** pour déterminer si votre package a besoin d'un runtime 32 bits pour s'exécuter.

   1. Dans le champ **Emplacement du package** , sélectionnez **SSISDB** , **Système de fichiers (Package)** , **Système de fichiers (Projet)** , **Package incorporé** ou **Magasin de packages** . 

##### <a name="package-location-ssisdb"></a>Emplacement du package : SSISDB

**SSISDB** est automatiquement sélectionné comme emplacement de votre package si votre runtime d'intégration Azure-SSIS a été approvisionné avec un catalogue SSIS (SSISDB) hébergé par un serveur Azure SQL Database/Managed Instance. Vous pouvez également le sélectionner vous-même. S'il est sélectionné, procédez comme suit.

   1. Si votre runtime d'intégration Azure-SSIS est en cours d'exécution et que la case **Entrées manuelles** n'est pas cochée, vous pouvez parcourir et sélectionner vos dossiers, projets, packages et environnements existants dans SSISDB. Sélectionnez **Actualiser** pour récupérer vos nouveaux dossiers, projets, packages ou environnements ajoutés dans SSISDB afin qu'ils soient sélectionnables lors de la navigation. Pour parcourir et sélectionner les environnements à des fins d'exécution de vos packages, vous devez configurer vos projets au préalable de manière à ajouter ces environnements en tant que références à partir des mêmes dossiers sous SSISDB. Pour plus d’informations, consultez [Créer et mapper des environnements SSIS](/sql/integration-services/packages/deploy-integration-services-ssis-projects-and-packages).

   1. Concernant **Niveau de journalisation** , sélectionnez un périmètre prédéfini de journalisation pour l’exécution de votre package. Sélectionnez la case **Personnalisé** si vous préférez entrer un nom de journalisation personnalisé. 

   1. Si votre Azure-SSIS IR n’est pas en cours d’exécution ou que la case **Entrées manuelles** est sélectionnée, entrez les chemins de votre package et de votre environnement directement à partir de SSISDB aux formats suivants : `<folder name>/<project name>/<package name>.dtsx` et `<folder name>/<environment name>`.

      ![Définir les propriétés sous l’onglet Paramètres - Manuel](media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-settings2.png)

##### <a name="package-location-file-system-package"></a>Emplacement du package : Système de fichiers (Package)

**Système de fichiers (Package)** est automatiquement sélectionné comme emplacement de votre package si votre runtime d'intégration Azure-SSIS a été approvisionné sans SSISDB. Vous pouvez également le sélectionner vous-même. S'il est sélectionné, procédez comme suit.

![Définir les propriétés sous l'onglet Paramètres - Système de fichiers (Package)](media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-settings3.png)
   
   1. Désignez le package à exécuter en fournissant un chemin d'accès UNC (Universal Naming Convention) à votre fichier de package (avec `.dtsx`) dans le champ **Chemin d'accès du package** . Vous pouvez accéder à votre package en sélectionnant **Parcourir le stockage de fichiers** ou entrer manuellement le chemin d'accès de celui-ci. Par exemple, si vous stockez votre package dans Azure Files, son chemin d'accès est `\\<storage account name>.file.core.windows.net\<file share name>\<package name>.dtsx`. 
   
   1. Si vous configurez votre package dans un fichier distinct, vous devez également fournir le chemin d'accès UNC de votre fichier de configuration (avec `.dtsConfig`) dans le champ **Chemin d'accès de la configuration** . Vous pouvez accéder à votre configuration en sélectionnant **Parcourir le stockage de fichiers** ou entrer manuellement le chemin d'accès de celle-ci. Par exemple, si vous stockez votre configuration dans Azure Files, son chemin d'accès est `\\<storage account name>.file.core.windows.net\<file share name>\<configuration name>.dtsConfig`.

   1. Entrez les informations d'identification requises pour accéder aux fichiers de votre package et de votre configuration. Si vous avez précédemment entré les valeurs des informations d'identification requises pour l'exécution de votre package ( **authentification Windows** ), vous pouvez les réutiliser en cochant la case **Identiques aux informations d'identification requises pour l'exécution du package** . Dans le cas contraire, entrez les valeurs pour vos informations d’identification pour l’accès au package dans les zones **Domaine** , **Nom d'utilisateur** et **Mot de passe** . Par exemple, si vous stockez votre package et votre configuration dans Azure Files, le domaine, le nom d'utilisateur et le mot de passe sont respectivement `Azure`, `<storage account name>` et `<storage account key>`. 

      Vous pouvez également utiliser les secrets stockés dans votre coffre de clés Azure Key Vault comme valeurs. Pour ce faire, cochez la case **AZURE KEY VAULT** en regard de ceux-ci. Sélectionnez ou modifiez votre service de coffre de clés lié existant ou créez-en un. Sélectionnez ensuite la version et le nom du secret correspondant à votre valeur. Lorsque vous créez ou modifiez votre service de coffre de clés lié, vous pouvez sélectionner ou modifier votre coffre de clés existant ou en créer un nouveau. Veillez à accorder à Data Factory l’accès géré à votre coffre de clés si vous ne l’avez pas déjà fait. Vous pouvez également entrer votre secret directement au format suivant : `<key vault linked service name>/<secret name>/<secret version>`. 

      Ces informations d'identification sont également utilisées pour accéder aux packages enfants de la Tâche d'exécution de package qui sont référencés par leur propre chemin d'accès et d'autres configurations spécifiées dans vos packages. 

   1. Si vous avez utilisé le niveau de protection **EncryptAllWithPassword** ou **EncryptSensitiveWithPassword** lorsque vous avez créé votre package via SQL Server Data Tools (SSDT), entrez la valeur de votre mot de passe dans le champ **Mot de passe de chiffrement** . Vous pouvez également utiliser un secret stocké dans votre coffre de clés Azure Key Vault comme valeur (voir précédemment).
      
      Si vous avez utilisé le niveau de protection **EncryptSensitiveWithUserKey** , entrez de nouveau vos valeurs sensibles dans les fichiers de configuration ou dans les onglets **Paramètres SSIS** , **Gestionnaires de connexions** ou **Substitutions de propriété** (voir ci-dessous).
      
      Si vous avez utilisé le niveau de protection **EncryptAllWithUserKey** , cela n’est pas pris en charge. Vous devez reconfigurer votre package pour utiliser un autre niveau de protection via SSDT ou l'utilitaire de ligne de commande `dtutil`. 

   1. Concernant **Niveau de journalisation** , sélectionnez un périmètre prédéfini de journalisation pour l’exécution de votre package. Sélectionnez la case **Personnalisé** si vous préférez entrer un nom de journalisation personnalisé. 
   
   1. Si vous souhaitez journaliser vos exécutions de package au-delà à l’aide de modules fournisseurs d’informations, spécifiez votre dossier de journaux en fournissant son chemin d’accès UNC dans la zone **Chemin d’accès de journalisation** . Vous pouvez accéder à votre dossier journal en sélectionnant **Parcourir le stockage de fichiers** ou entrer manuellement le chemin d'accès de celui-ci. Par exemple, si vous stockez vos journaux dans Azure Files, votre chemin d’accès de journalisation est `\\<storage account name>.file.core.windows.net\<file share name>\<log folder name>`. Un sous-dossier est créé à cet emplacement pour l'exécution de chaque package, et nommé d'après l'ID d'exécution de l'activité Exécuter le package SSIS. Des fichiers journaux y sont générés toutes les cinq minutes. 
   
   1. Entrez les informations d'identification pour accéder à votre dossier journal. Si vous avez précédemment entré les valeurs des informations d'identification requises pour l'accès au package (voir précédemment), vous pouvez les réutiliser en cochant la case **Identiques aux informations d'identification requises pour l'accès au package** . Dans le cas contraire, entrez les valeurs pour vos informations d’identification pour l’accès aux journaux dans les zones **Domaine** , **Nom d'utilisateur** et **Mot de passe** . Par exemple, si vous stockez vos journaux dans Azure Files, le domaine est `Azure` ; le nom d’utilisateur est `<storage account name>` ; et le mot de passe est `<storage account key>`. Vous pouvez également utiliser les secrets stockés dans votre coffre de clés Azure Key Vault comme valeurs (voir précédemment).
   
Pour tous les chemins d’accès UNC mentionnés précédemment, le nom de fichier complet doit être inférieur à 260 caractères. Le nom de répertoire ne doit pas dépasser 248 caractères.

##### <a name="package-location-file-system-project"></a>Emplacement du package : Système de fichiers (Projet)

Si vous sélectionnez **Système de fichiers (Projet)** comme emplacement de votre package, procédez comme suit.

![Définir les propriétés sous l'onglet Paramètres - Système de fichiers (Projet)](media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-settings4.png)

   1. Désignez le package à exécuter en fournissant un chemin d'accès UNC à votre fichier de projet (avec `.ispac`) dans le champ **Chemin d'accès du projet** et un fichier de package (avec `.dtsx`) de votre projet dans le champ **Nom du package** . Vous pouvez accéder à votre projet en sélectionnant **Parcourir le stockage de fichiers** ou entrer manuellement le chemin d'accès de celui-ci. Par exemple, si vous stockez votre projet dans Azure Files, son chemin d'accès est `\\<storage account name>.file.core.windows.net\<file share name>\<project name>.ispac`.

   1. Entrez les informations d'identification requises pour accéder aux fichiers de votre projet et de votre package. Si vous avez précédemment entré les valeurs des informations d'identification requises pour l'exécution de votre package ( **authentification Windows** ), vous pouvez les réutiliser en cochant la case **Identiques aux informations d'identification requises pour l'exécution du package** . Dans le cas contraire, entrez les valeurs pour vos informations d’identification pour l’accès au package dans les zones **Domaine** , **Nom d'utilisateur** et **Mot de passe** . Par exemple, si vous stockez votre projet et votre package dans Azure Files, le domaine, le nom d'utilisateur et le mot de passe sont respectivement `Azure`, `<storage account name>` et `<storage account key>`. 

      Vous pouvez également utiliser les secrets stockés dans votre coffre de clés Azure Key Vault comme valeurs. Pour ce faire, cochez la case **AZURE KEY VAULT** en regard de ceux-ci. Sélectionnez ou modifiez votre service de coffre de clés lié existant ou créez-en un. Sélectionnez ensuite la version et le nom du secret correspondant à votre valeur. Lorsque vous créez ou modifiez votre service de coffre de clés lié, vous pouvez sélectionner ou modifier votre coffre de clés existant ou en créer un nouveau. Veillez à accorder à Data Factory l’accès géré à votre coffre de clés si vous ne l’avez pas déjà fait. Vous pouvez également entrer votre secret directement au format suivant : `<key vault linked service name>/<secret name>/<secret version>`. 

      Ces informations d'identification sont également utilisées pour accéder aux packages enfants de la Tâche d'exécution de package qui sont référencés à partir du même projet. 

   1. Si vous avez utilisé le niveau de protection **EncryptAllWithPassword** ou **EncryptSensitiveWithPassword** lorsque vous avez créé votre package via SSDT, entrez la valeur de votre mot de passe dans le champ **Mot de passe de chiffrement** . Vous pouvez également utiliser un secret stocké dans votre coffre de clés Azure Key Vault comme valeur (voir précédemment).
      
      Si vous avez utilisé le niveau de protection **EncryptSensitiveWithUserKey** , entrez de nouveau vos valeurs sensibles dans les onglets **Paramètres SSIS** , **Gestionnaires de connexions** ou **Substitutions de propriété** (voir ci-dessous).
      
      Si vous avez utilisé le niveau de protection **EncryptAllWithUserKey** , cela n’est pas pris en charge. Vous devez reconfigurer votre package pour utiliser un autre niveau de protection via SSDT ou l'utilitaire de ligne de commande `dtutil`. 

   1. Concernant **Niveau de journalisation** , sélectionnez un périmètre prédéfini de journalisation pour l’exécution de votre package. Sélectionnez la case **Personnalisé** si vous préférez entrer un nom de journalisation personnalisé. 
   
   1. Si vous souhaitez journaliser vos exécutions de package au-delà à l’aide de modules fournisseurs d’informations, spécifiez votre dossier de journaux en fournissant son chemin d’accès UNC dans la zone **Chemin d’accès de journalisation** . Vous pouvez accéder à votre dossier journal en sélectionnant **Parcourir le stockage de fichiers** ou entrer manuellement le chemin d'accès de celui-ci. Par exemple, si vous stockez vos journaux dans Azure Files, votre chemin d’accès de journalisation est `\\<storage account name>.file.core.windows.net\<file share name>\<log folder name>`. Un sous-dossier est créé à cet emplacement pour l'exécution de chaque package, et nommé d'après l'ID d'exécution de l'activité Exécuter le package SSIS. Des fichiers journaux y sont générés toutes les cinq minutes. 
   
   1. Entrez les informations d'identification pour accéder à votre dossier journal. Si vous avez précédemment entré les valeurs des informations d'identification requises pour l'accès au package (voir précédemment), vous pouvez les réutiliser en cochant la case **Identiques aux informations d'identification requises pour l'accès au package** . Dans le cas contraire, entrez les valeurs pour vos informations d’identification pour l’accès aux journaux dans les zones **Domaine** , **Nom d'utilisateur** et **Mot de passe** . Par exemple, si vous stockez vos journaux dans Azure Files, le domaine est `Azure` ; le nom d’utilisateur est `<storage account name>` ; et le mot de passe est `<storage account key>`. Vous pouvez également utiliser les secrets stockés dans votre coffre de clés Azure Key Vault comme valeurs (voir précédemment).
   
Pour tous les chemins d’accès UNC mentionnés précédemment, le nom de fichier complet doit être inférieur à 260 caractères. Le nom de répertoire ne doit pas dépasser 248 caractères.

##### <a name="package-location-embedded-package"></a>Emplacement du package : Package incorporé

Si vous sélectionnez **Package incorporé** comme emplacement de votre package, procédez comme suit.

![Définir les propriétés sous l'onglet Paramètres - Package incorporé](media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-settings5.png)

   1. Glissez-déplacez le fichier de votre package (avec `.dtsx`) ou **chargez-le** dans la zone prévue à cet effet depuis un dossier de fichiers. Votre package sera automatiquement compressé et incorporé à la charge utile de l'activité. Une fois votre package incorporé, vous pouvez le **télécharger** ultérieurement afin de le modifier. Vous pouvez également **paramétrer** votre package incorporé en l'assignant à un paramètre pipeline qui peut être utilisé dans plusieurs activités, optimisant ainsi la taille de votre charge utile de pipeline. L’incorporation de Fichiers projet (avec `.ispac`) n’est pas prise en charge pour le moment. Vous ne pouvez donc pas utiliser de paramètres SSIS ni gestionnaires de connexions avec une étendue au niveau du projet dans vos packages incorporés.
   
   1. Si votre package incorporé n’est pas entièrement chiffré et que nous détectons l’utilisation de Tâche d’exécution de package, la case **Tâche d’exécution de package**  sera automatiquement cochée et vos packages enfants référencés par leur chemin d’accès au système de fichiers seront automatiquement ajoutés pour que vous puissiez également les incorporer.
   
      Si nous ne parvenons pas à détecter l’utilisation de Tâche d’exécution de package, vous devez sélectionner manuellement la case à cocher **Tâche d’exécution de package** et ajouter un par un les packages enfants qui sont référencés par leur chemin d’accès au système de fichiers afin de les incorporer. Si vos packages enfants sont stockés dans une base de données SQL Server (MSDB), vous ne pouvez pas les incorporer. Vous devez donc vous assurer que votre runtime d’intégration Azure-SSIS peut accéder à la MSDB pour les récupérer à l’aide de leurs références SQL Server. L’incorporation de Fichiers projet (avec `.ispac`) n’est pas prise en charge pour le moment. Vous ne pouvez donc pas utiliser de références de projet pour vos packages enfants.
   
   1. Si vous avez utilisé le niveau de protection **EncryptAllWithPassword** ou **EncryptSensitiveWithPassword** lorsque vous avez créé votre package via SSDT, entrez la valeur de votre mot de passe dans le champ **Mot de passe de chiffrement** . 
   
      Vous pouvez également utiliser un secret stocké dans votre coffre de clés Azure Key Vault comme valeur. Pour ce faire, cochez la case **AZURE KEY VAULT** en regard de celui-ci. Sélectionnez ou modifiez votre service de coffre de clés lié existant ou créez-en un. Sélectionnez ensuite la version et le nom du secret correspondant à votre valeur. Lorsque vous créez ou modifiez votre service de coffre de clés lié, vous pouvez sélectionner ou modifier votre coffre de clés existant ou en créer un nouveau. Veillez à accorder à Data Factory l’accès géré à votre coffre de clés si vous ne l’avez pas déjà fait. Vous pouvez également entrer votre secret directement au format suivant : `<key vault linked service name>/<secret name>/<secret version>`.
      
      Si vous avez utilisé le niveau de protection **EncryptSensitiveWithUserKey** , entrez de nouveau vos valeurs sensibles dans les fichiers de configuration ou dans les onglets **Paramètres SSIS** , **Gestionnaires de connexions** ou **Substitutions de propriété** (voir ci-dessous).
      
      Si vous avez utilisé le niveau de protection **EncryptAllWithUserKey** , cela n’est pas pris en charge. Vous devez reconfigurer votre package pour utiliser un autre niveau de protection via SSDT ou l'utilitaire de ligne de commande `dtutil`.

   1. Concernant **Niveau de journalisation** , sélectionnez un périmètre prédéfini de journalisation pour l’exécution de votre package. Sélectionnez la case **Personnalisé** si vous préférez entrer un nom de journalisation personnalisé. 
   
   1. Si vous souhaitez journaliser vos exécutions de package au-delà à l’aide de modules fournisseurs d’informations, spécifiez votre dossier de journaux en fournissant son chemin d’accès UNC dans la zone **Chemin d’accès de journalisation** . Vous pouvez accéder à votre dossier journal en sélectionnant **Parcourir le stockage de fichiers** ou entrer manuellement le chemin d'accès de celui-ci. Par exemple, si vous stockez vos journaux dans Azure Files, votre chemin d’accès de journalisation est `\\<storage account name>.file.core.windows.net\<file share name>\<log folder name>`. Un sous-dossier est créé à cet emplacement pour l'exécution de chaque package, et nommé d'après l'ID d'exécution de l'activité Exécuter le package SSIS. Des fichiers journaux y sont générés toutes les cinq minutes. 
   
   1. Spécifiez les informations d'identification requises pour accéder à votre dossier journal en entrant les valeurs correspondantes dans les champs **Domaine** , **Nom d'utilisateur** et **Mot de passe** . Par exemple, si vous stockez vos journaux dans Azure Files, le domaine est `Azure` ; le nom d’utilisateur est `<storage account name>` ; et le mot de passe est `<storage account key>`. Vous pouvez également utiliser les secrets stockés dans votre coffre de clés Azure Key Vault comme valeurs (voir précédemment).
   
Pour tous les chemins d’accès UNC mentionnés précédemment, le nom de fichier complet doit être inférieur à 260 caractères. Le nom de répertoire ne doit pas dépasser 248 caractères.

##### <a name="package-location-package-store"></a>Emplacement du package : Magasin de packages

Si vous sélectionnez **Magasin de packages** comme emplacement de votre package, procédez comme suit.

![Définir les propriétés sous l'onglet Paramètres - Magasin de packages](media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-settings6.png)
   
   1. Dans le champ **Nom du magasin de packages** , sélectionnez un magasin de packages existant associé à votre runtime d'intégration Azure-SSIS.

   1. Désignez le package à exécuter en fournissant son chemin d'accès (sans `.dtsx`) depuis le magasin de packages sélectionné dans le champ **Chemin d'accès du package** . Si le magasin de packages sélectionné se trouve au sommet du système de fichiers/Azure Files, vous pouvez accéder à votre package en sélectionnant **Parcourir le stockage de fichiers** . Vous pouvez également entrer son chemin au format `<folder name>\<package name>`. Vous avez aussi la possibilité d'importer de nouveaux packages dans le magasin de packages sélectionné via SQL Server Management Studio (SSMS), comme pour le [magasin de packages SSIS hérité](/sql/integration-services/service/package-management-ssis-service?view=sql-server-2017). Pour plus d'informations, consultez [Gérer les packages SSIS avec des magasins de packages Azure-SSIS IR](./azure-ssis-integration-runtime-package-store.md).

   1. Si vous configurez votre package dans un fichier distinct, vous devez fournir le chemin d'accès UNC de votre fichier de configuration (avec `.dtsConfig`) dans le champ **Chemin d'accès de la configuration** . Vous pouvez accéder à votre configuration en sélectionnant **Parcourir le stockage de fichiers** ou entrer manuellement le chemin d'accès de celle-ci. Par exemple, si vous stockez votre configuration dans Azure Files, son chemin d'accès est `\\<storage account name>.file.core.windows.net\<file share name>\<configuration name>.dtsConfig`.

   1. Cochez la case **Informations d'identification requises pour l'accès à la configuration** afin de déterminer si des informations d'identification sont nécessaires pour accéder à votre fichier de configuration séparément. Cette opération est nécessaire lorsque le magasin de packages sélectionné se trouve au sommet de la base de données SQL Server (MSDB) hébergée par Azure SQL Managed Instance ou si votre fichier de configuration n'y est pas stocké.
   
      Si vous avez précédemment entré les valeurs des informations d'identification requises pour l'exécution de votre package ( **authentification Windows** ), vous pouvez les réutiliser en cochant la case **Identiques aux informations d'identification requises pour l'exécution du package** . Si ce n'est pas le cas, entrez les valeurs des informations d'identification requises pour l'accès à la configuration dans les champs **Domaine** , **Nom d'utilisateur** et **Mot de passe** . Par exemple, si vous stockez votre configuration dans Azure Files, le domaine, le nom d'utilisateur et le mot de passe sont respectivement `Azure`, `<storage account name>` et `<storage account key>`. 

      Vous pouvez également utiliser les secrets stockés dans votre coffre de clés Azure Key Vault comme valeurs. Pour ce faire, cochez la case **AZURE KEY VAULT** en regard de ceux-ci. Sélectionnez ou modifiez votre service de coffre de clés lié existant ou créez-en un. Sélectionnez ensuite la version et le nom du secret correspondant à votre valeur. Lorsque vous créez ou modifiez votre service de coffre de clés lié, vous pouvez sélectionner ou modifier votre coffre de clés existant ou en créer un nouveau. Veillez à accorder à Data Factory l’accès géré à votre coffre de clés si vous ne l’avez pas déjà fait. Vous pouvez également entrer votre secret directement au format suivant : `<key vault linked service name>/<secret name>/<secret version>`.

   1. Si vous avez utilisé le niveau de protection **EncryptAllWithPassword** ou **EncryptSensitiveWithPassword** lorsque vous avez créé votre package via SSDT, entrez la valeur de votre mot de passe dans le champ **Mot de passe de chiffrement** . Vous pouvez également utiliser un secret stocké dans votre coffre de clés Azure Key Vault comme valeur (voir précédemment).
      
      Si vous avez utilisé le niveau de protection **EncryptSensitiveWithUserKey** , entrez de nouveau vos valeurs sensibles dans les fichiers de configuration ou dans les onglets **Paramètres SSIS** , **Gestionnaires de connexions** ou **Substitutions de propriété** (voir ci-dessous).
      
      Si vous avez utilisé le niveau de protection **EncryptAllWithUserKey** , cela n’est pas pris en charge. Vous devez reconfigurer votre package pour utiliser un autre niveau de protection via SSDT ou l'utilitaire de ligne de commande `dtutil`. 

   1. Concernant **Niveau de journalisation** , sélectionnez un périmètre prédéfini de journalisation pour l’exécution de votre package. Sélectionnez la case **Personnalisé** si vous préférez entrer un nom de journalisation personnalisé. 
   
   1. Si vous souhaitez journaliser vos exécutions de package au-delà à l’aide de modules fournisseurs d’informations, spécifiez votre dossier de journaux en fournissant son chemin d’accès UNC dans la zone **Chemin d’accès de journalisation** . Vous pouvez accéder à votre dossier journal en sélectionnant **Parcourir le stockage de fichiers** ou entrer manuellement le chemin d'accès de celui-ci. Par exemple, si vous stockez vos journaux dans Azure Files, votre chemin d’accès de journalisation est `\\<storage account name>.file.core.windows.net\<file share name>\<log folder name>`. Un sous-dossier est créé à cet emplacement pour l'exécution de chaque package, et nommé d'après l'ID d'exécution de l'activité Exécuter le package SSIS. Des fichiers journaux y sont générés toutes les cinq minutes. 
   
   1. Spécifiez les informations d'identification requises pour accéder à votre dossier journal en entrant les valeurs correspondantes dans les champs **Domaine** , **Nom d'utilisateur** et **Mot de passe** . Par exemple, si vous stockez vos journaux dans Azure Files, le domaine est `Azure` ; le nom d’utilisateur est `<storage account name>` ; et le mot de passe est `<storage account key>`. Vous pouvez également utiliser les secrets stockés dans votre coffre de clés Azure Key Vault comme valeurs (voir précédemment).
   
Pour tous les chemins d’accès UNC mentionnés précédemment, le nom de fichier complet doit être inférieur à 260 caractères. Le nom de répertoire ne doit pas dépasser 248 caractères.

#### <a name="ssis-parameters-tab"></a>Onglet Paramètres SSIS

Sous l'onglet **Paramètres SSIS** de l'activité Exécuter le package SSIS, procédez comme suit.

![Définir les propriétés sous l’onglet Paramètres SSIS](media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-ssis-parameters.png)

   1. Si votre runtime d'intégration Azure-SSIS est en cours d'exécution, **SSISDB** est sélectionné comme emplacement de votre package, et si la case **Entrées manuelles** de l'onglet **Paramètres** n'est pas cochée, les paramètres SSIS existants du projet et du package sélectionnés à partir de SSISDB apparaissent pour vous permettre de leur attribuer des valeurs. Sinon, vous pouvez affecter manuellement des valeurs une par une. Assurez-vous que les valeurs existent et qu’elles sont correctement entrées pour que l’exécution du package réussisse. 
   
   1. Si vous avez utilisé le niveau de protection **EncryptSensitiveWithUserKey** lorsque vous avez créé votre package via SSDT et que **Système de fichiers (Package)** , **Système de fichiers (Projet)** , **Package incorporé** ou **Magasin de packages** est sélectionné comme emplacement de votre package, vous devez également réentrer vos paramètres sensibles pour leur attribuer des valeurs dans cet onglet. 
   
Lorsque vous attribuez des valeurs à vos paramètres, vous pouvez ajouter du contenu dynamique à l’aide d’expressions, de fonctions, de variables système Data Factory et de paramètres ou variables de pipeline Data Factory.

Vous pouvez également utiliser les secrets stockés dans votre coffre de clés Azure Key Vault comme valeurs. Pour ce faire, cochez la case **AZURE KEY VAULT** en regard de ceux-ci. Sélectionnez ou modifiez votre service de coffre de clés lié existant ou créez-en un. Sélectionnez ensuite la version et le nom du secret correspondant à votre valeur. Lorsque vous créez ou modifiez votre service de coffre de clés lié, vous pouvez sélectionner ou modifier votre coffre de clés existant ou en créer un nouveau. Veillez à accorder à Data Factory l’accès géré à votre coffre de clés si vous ne l’avez pas déjà fait. Vous pouvez également entrer votre secret directement au format suivant : `<key vault linked service name>/<secret name>/<secret version>`. 

#### <a name="connection-managers-tab"></a>Onglet Connection Managers (Gestionnaires de connexions)

Sous l'onglet **Gestionnaires de connexions** de l'activité Exécuter le package SSIS, procédez comme suit.

![Définir les propriétés sous l’onglet Gestionnaires de connexions](media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-connection-managers.png)

   1. Si votre runtime d'intégration Azure-SSIS est en cours d'exécution, **SSISDB** est sélectionné comme emplacement de votre package, et si la case **Entrées manuelles** de l'onglet **Paramètres** n'est pas cochée, les gestionnaires de connexions existants du projet et du package sélectionnés à partir de SSISDB apparaissent pour vous permettre d'attribuer des valeurs à leurs propriétés. Sinon, vous pouvez affecter manuellement des valeurs aux propriétés une par une. Assurez-vous que les valeurs existent et qu’elles sont correctement entrées pour que l’exécution du package réussisse. 
   
      Vous pouvez obtenir les noms **ÉTENDUE** , **NOM** et **PROPRIÉTÉ** corrects pour tout gestionnaire de connexions en ouvrant le package qui le contient sur SSDT. Une fois le package ouvert, sélectionnez le gestionnaire de connexions approprié pour afficher les noms et les valeurs de toutes ses propriétés dans la fenêtre **Propriétés** de SSDT. Grâce à ces informations, vous pouvez remplacer les valeurs de toutes les propriétés du gestionnaire de connexions à l’exécution. 

      ![Obtenir les propriétés du gestionnaire de connexions sur SSDT](media/how-to-invoke-ssis-package-ssis-activity/ssdt-connection-manager-properties.png)

      Par exemple, sans modifier votre package d’origine sur SSDT, vous pouvez convertir ses flux de données local vers local s’exécutant sur SQL Server en flux de données local vers cloud s’exécutant sur un runtime d’intégration SSIS dans ADF en remplaçant les valeurs des propriétés **ConnectByProxy** , **ConnectionString** et **ConnectUsingManagedIdentity** dans les gestionnaires de connexions existants à l’exécution.
      
      Ces remplacements à l’exécution peuvent activer le runtime d’intégration auto-hébergé (SHIR) comme proxy pour le runtime d’intégration SSIS lors de l’accès aux données localement (voir [Configurer un runtime d’intégration auto-hébergé en tant que proxy pour le runtime d’intégration SSIS](./self-hosted-integration-runtime-proxy-ssis.md)) et aux connexions Azure SQL Database/Managed Instance à l’aide du dernier pilote MSOLEDBSQL qui, à son tour, active l’authentification Azure Active Directory (AAD) avec l’identité managée ADF (voir [Configurer l’authentification AAD avec une identité managée ADF pour les connexions OLEDB](/sql/integration-services/connection-manager/ole-db-connection-manager?view=sql-server-ver15#managed-identities-for-azure-resources-authentication)).

      ![Définir les propriétés à partir de SSDT sous l’onglet Gestionnaires de connexions](media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-connection-managers2.png)
   
   1. Si vous avez utilisé le niveau de protection **EncryptSensitiveWithUserKey** lorsque vous avez créé votre package via SSDT et que **Système de fichiers (Package)** , **Système de fichiers (Projet)** , **Package incorporé** ou **Magasin de packages** est sélectionné comme emplacement de votre package, vous devez également réentrer les propriétés sensibles de vos gestionnaires de connexions pour leur attribuer des valeurs dans cet onglet. 

Lorsque vous attribuez des valeurs à vos propriétés de gestionnaire de connexions, vous pouvez ajouter du contenu dynamique à l’aide d’expressions, de fonctions, de variables système Data Factory et de paramètres ou variables de pipeline Data Factory. 

Vous pouvez également utiliser les secrets stockés dans votre coffre de clés Azure Key Vault comme valeurs. Pour ce faire, cochez la case **AZURE KEY VAULT** en regard de ceux-ci. Sélectionnez ou modifiez votre service de coffre de clés lié existant ou créez-en un. Sélectionnez ensuite la version et le nom du secret correspondant à votre valeur. Lorsque vous créez ou modifiez votre service de coffre de clés lié, vous pouvez sélectionner ou modifier votre coffre de clés existant ou en créer un nouveau. Veillez à accorder à Data Factory l’accès géré à votre coffre de clés si vous ne l’avez pas déjà fait. Vous pouvez également entrer votre secret directement au format suivant : `<key vault linked service name>/<secret name>/<secret version>`. 

#### <a name="property-overrides-tab"></a>Onglet Substitutions de propriété

Sous l'onglet **Substitutions de propriété** de l'activité Exécuter le package SSIS, procédez comme suit.

![Définir les propriétés sous l’onglet Substitutions de propriété](media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-property-overrides.png)

   1. Entrez un par un les chemins d'accès des propriétés existantes dans le package sélectionné pour leur attribuer manuellement des valeurs. Assurez-vous que les valeurs existent et qu’elles sont correctement entrées pour que l’exécution du package réussisse. Par exemple, pour remplacer la valeur de votre variable utilisateur, entrez son chemin d’accès au format suivant : `\Package.Variables[User::<variable name>].Value`. 

      Vous pouvez obtenir le **CHEMIN D’ACCÈS À LA PROPRIÉTÉ** correct pour toute propriété de package en ouvrant le package qui le contient sur SSDT. Une fois le package ouvert, sélectionnez son flux de contrôle et sa propriété **Configurations** dans la fenêtre **Propriétés** de SSDT. Sélectionnez ensuite le bouton de sélection ( **…** ) à côté de sa propriété **Configurations** pour ouvrir la **bibliothèque des configurations du package** qui est normalement utilisée pour [créer des configurations de package dans le modèle de déploiement de package](/sql/integration-services/packages/legacy-package-deployment-ssis#create-package-configurations). 

      ![Récupérer les propriétés de package à partir de SSDT – Propriété Configurations](media/how-to-invoke-ssis-package-ssis-activity/ssdt-package-properties.png)

      Dans la **bibliothèque des configurations du package** , cochez la case **Activer les configurations du package** et sélectionnez le bouton **Ajouter…** pour ouvrir l’ **Assistant Configuration de package** . 
      
      Dans l’ **Assistant Configuration du package** , sélectionnez l’élément **Fichier config XML** dans menu déroulant **Type de configuration** et le bouton **Spécifier directement les paramètres de configuration** , entrez le nom de votre fichier config, puis sélectionnez le bouton **Suivant >** . 

      ![Récupérer les propriétés de package à partir de SSDT – Bibliothèque des configurations](media/how-to-invoke-ssis-package-ssis-activity/ssdt-package-properties2.png)

      Enfin, sélectionnez les propriétés du package dont vous souhaitez obtenir le chemin d’accès et le bouton **Suivant >** .  Vous pouvez maintenant voir, copier et coller les chemins d’accès aux propriétés du package de votre choix et les enregistrer dans votre fichier config. Grâce à ces informations, vous pouvez remplacer les valeurs de toutes les propriétés du package à l’exécution. 

      ![Récupérer les propriétés de package à partir de SSDT – Assistant Configuration](media/how-to-invoke-ssis-package-ssis-activity/ssdt-package-properties3.png)
   
   1. Si vous avez utilisé le niveau de protection **EncryptSensitiveWithUserKey** lorsque vous avez créé votre package via SSDT et que **Système de fichiers (Package)** , **Système de fichiers (Projet)** , **Package incorporé** ou **Magasin de packages** est sélectionné comme emplacement de votre package, vous devez également réentrer les propriétés sensibles de votre package pour leur attribuer des valeurs dans cet onglet. 
   
Lorsque vous attribuez des valeurs à vos propriétés de package, vous pouvez ajouter du contenu dynamique à l'aide d'expressions, de fonctions, de variables système Data Factory et de paramètres ou variables de pipeline Data Factory.

Les valeurs attribuées dans les fichiers de configuration et dans l’onglet **Paramètres SSIS** peuvent être remplacées à l’aide des onglets **Gestionnaires de connexions** et **Substitutions de propriété** . Vous pouvez également remplacer les valeurs affectées dans l’onglet **Gestionnaires de connexions** dans l’onglet **Substitutions de propriété** .

Pour valider la configuration du pipeline, sélectionnez **Valider** dans la barre d’outils. Pour fermer le **Rapport de validation de pipeline** , sélectionnez **>>** .

Pour publier le pipeline sur Data Factory, sélectionnez **Publier tout** . 

### <a name="run-the-pipeline"></a>Exécuter le pipeline
À cette étape, vous déclenchez une exécution du pipeline. 

1. Pour déclencher une exécution de pipeline, sélectionnez **Déclencher** dans la barre d’outils, puis **Déclencher maintenant** . 

   ![Déclencher maintenant](./media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-trigger.png)

2. Dans la fenêtre **Exécution du pipeline** , sélectionnez **Terminer** . 

### <a name="monitor-the-pipeline"></a>Surveiller le pipeline

1. Basculez vers l’onglet **Surveiller** sur la gauche. Vous voyez l’exécution de pipeline et son état, ainsi que d’autres informations (telles que l’heure de **Début d’exécution** ). Sélectionnez **Actualiser** pour actualiser l’affichage.

   ![Exécutions de pipeline](./media/how-to-invoke-ssis-package-stored-procedure-activity/pipeline-runs.png)

2. Cliquez sur le lien **Afficher les exécutions d’activités** dans la colonne **Actions** . Une seule exécution d’activité est affichée, étant donné que le pipeline n’a qu’une seule activité. Il s’agit de l’activité Exécuter le package SSIS.

   ![Exécutions d’activités](./media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-runs.png)

3. Exécutez la requête suivante par rapport à la base de données SSISDB dans votre serveur SQL pour vérifier que le package s’est exécuté. 

   ```sql
   select * from catalog.executions
   ```

   ![Vérifier les exécutions de package](./media/how-to-invoke-ssis-package-stored-procedure-activity/verify-package-executions.png)

4. Vous pouvez également obtenir l’ID de l’exécution SSISDB à partir de la sortie de l’exécution de l’activité de pipeline, et utiliser l’ID pour consulter des journaux d’activité d’exécution et des messages d’erreur plus complets dans SQL Server Management Studio.

   ![Obtenez l’ID de l'exécution.](media/how-to-invoke-ssis-package-ssis-activity/get-execution-id.png)

### <a name="schedule-the-pipeline-with-a-trigger"></a>Planifier le pipeline avec un déclencheur

Vous pouvez également créer un déclencheur planifié pour votre pipeline afin que le pipeline s’exécute selon une planification, par exemple horaire ou quotidienne. Pour obtenir un exemple, consultez [Créer une fabrique de données - Interface utilisateur de Data Factory](quickstart-create-data-factory-portal.md#trigger-the-pipeline-on-a-schedule).

## <a name="run-a-package-with-powershell"></a>Exécuter un package avec PowerShell
Dans cette section, vous utilisez Azure PowerShell pour créer un pipeline Data Factory avec une activité Execute SSIS Package qui exécute votre package SSIS. 

Installez les modules Azure PowerShell les plus récents en suivant les instructions pas à pas décrites dans [Comment installer et configurer Azure PowerShell](/powershell/azure/install-az-ps).

### <a name="create-a-data-factory-with-azure-ssis-ir"></a>Créer une fabrique de données avec Azure-SSIS IR
Vous pouvez utiliser une fabrique de données existante avec Azure-SSIS IR configuré ou créer une nouvelle fabrique de données avec Azure-SSIS IR. Suivez les instructions pas à pas du [Tutoriel : Déployer des packages SSIS sur Azure via Powershell](./tutorial-deploy-ssis-packages-azure-powershell.md).

### <a name="create-a-pipeline-with-an-execute-ssis-package-activity"></a>Créer un pipeline avec une activité Exécuter le Package SSIS 
Au cours de cette étape, vous créez un pipeline avec une activité Execute SSIS Package. L’activité exécute votre package SSIS. 

1. Créez un fichier JSON nommé `RunSSISPackagePipeline.json` dans le dossier `C:\ADF\RunSSISPackage` avec un contenu semblable à l'exemple suivant.

   > [!IMPORTANT]
   > Remplacez les noms d’objets, les descriptions et les chemins ou que les valeurs de propriété et de paramètre, les mots de passe et autres valeurs de variable avant d’enregistrer le fichier. 
    
   ```json
   {
       "name": "RunSSISPackagePipeline",
       "properties": {
           "activities": [{
               "name": "MySSISActivity",
               "description": "My SSIS package/activity description",
               "type": "ExecuteSSISPackage",
               "typeProperties": {
                   "connectVia": {
                       "referenceName": "MyAzureSSISIR",
                       "type": "IntegrationRuntimeReference"
                   },
                   "executionCredential": {
                       "domain": "MyExecutionDomain",
                       "username": "MyExecutionUsername",
                       "password": {
                           "type": "SecureString",
                           "value": "MyExecutionPassword"
                       }
                   },
                   "runtime": "x64",
                   "loggingLevel": "Basic",
                   "packageLocation": {
                       "type": "SSISDB",
                       "packagePath": "MyFolder/MyProject/MyPackage.dtsx"
                   },
                   "environmentPath": "MyFolder/MyEnvironment",
                   "projectParameters": {
                       "project_param_1": {
                           "value": "123"
                       },
                       "project_param_2": {
                           "value": {
                               "value": "@pipeline().parameters.MyProjectParameter",
                               "type": "Expression"
                           }
                       }
                   },
                   "packageParameters": {
                       "package_param_1": {
                           "value": "345"
                       },
                       "package_param_2": {
                           "value": {
                               "type": "AzureKeyVaultSecret",
                               "store": {
                                   "referenceName": "myAKV",
                                   "type": "LinkedServiceReference"
                               },
                               "secretName": "MyPackageParameter"
                           }
                       }
                   },
                   "projectConnectionManagers": {
                       "MyAdonetCM": {
                           "username": {
                               "value": "MyConnectionUsername"
                           },
                           "password": {
                               "value": {
                                   "type": "SecureString",
                                   "value": "MyConnectionPassword"
                               }
                           }
                       }
                   },
                   "packageConnectionManagers": {
                       "MyOledbCM": {
                           "username": {
                               "value": {
                                   "value": "@pipeline().parameters.MyConnectionUsername",
                                   "type": "Expression"
                               }
                           },
                           "password": {
                               "value": {
                                   "type": "AzureKeyVaultSecret",
                                   "store": {
                                       "referenceName": "myAKV",
                                       "type": "LinkedServiceReference"
                                   },
                                   "secretName": "MyConnectionPassword",
                                   "secretVersion": "MyConnectionPasswordVersion"
                               }
                           }
                       }
                   },
                   "propertyOverrides": {
                       "\\Package.MaxConcurrentExecutables": {
                           "value": 8,
                           "isSensitive": false
                       }
                   }
               },
               "policy": {
                   "timeout": "0.01:00:00",
                   "retry": 0,
                   "retryIntervalInSeconds": 30
               }
           }]
       }
   }
   ```

   Pour exécuter des packages stockés dans un système de fichiers/Azure Files, entrez les valeurs des propriétés d'emplacement de votre package et journal comme suit :

   ```json
   {
       {
           {
               {
                   "packageLocation": {
                       "type": "File",
                       "packagePath": "//MyStorageAccount.file.core.windows.net/MyFileShare/MyPackage.dtsx",
                       "typeProperties": {
                           "packagePassword": {
                               "type": "SecureString",
                               "value": "MyEncryptionPassword"
                           },
                           "accessCredential": {
                               "domain": "Azure",
                               "username": "MyStorageAccount",
                               "password": {
                                   "type": "SecureString",
                                   "value": "MyAccountKey"
                               }
                           }
                       }
                   },
                   "logLocation": {
                       "type": "File",
                       "logPath": "//MyStorageAccount.file.core.windows.net/MyFileShare/MyLogFolder",
                       "typeProperties": {
                           "accessCredential": {
                               "domain": "Azure",
                               "username": "MyStorageAccount",
                               "password": {
                                   "type": "AzureKeyVaultSecret",
                                   "store": {
                                       "referenceName": "myAKV",
                                       "type": "LinkedServiceReference"
                                   },
                                   "secretName": "MyAccountKey"
                               }
                           }
                       }
                   }
               }
           }
       }
   }
   ```

   Pour exécuter des packages dans des projets stockés dans un système de fichiers/Azure Files, entrez les valeurs des propriétés d'emplacement de votre package comme suit :

   ```json
   {
       {
           {
               {
                   "packageLocation": {
                       "type": "File",
                       "packagePath": "//MyStorageAccount.file.core.windows.net/MyFileShare/MyProject.ispac:MyPackage.dtsx",
                       "typeProperties": {
                           "packagePassword": {
                               "type": "SecureString",
                               "value": "MyEncryptionPassword"
                           },
                           "accessCredential": {
                               "domain": "Azure",
                               "userName": "MyStorageAccount",
                               "password": {
                                   "type": "SecureString",
                                   "value": "MyAccountKey"
                               }
                           }
                       }
                   }
               }
           }
       }
   }
   ```

   Pour exécuter des packages incorporés, entrez les valeurs des propriétés d'emplacement de votre package comme suit :

   ```json
   {
       {
           {
               {
                   "packageLocation": {
                       "type": "InlinePackage",
                       "typeProperties": {
                           "packagePassword": {
                               "type": "SecureString",
                               "value": "MyEncryptionPassword"
                           },
                           "packageName": "MyPackage.dtsx",
                           "packageContent":"My compressed/uncompressed package content",
                           "packageLastModifiedDate": "YYYY-MM-DDTHH:MM:SSZ UTC-/+HH:MM"
                       }
                   }
               }
           }
       }
   }
   ```

   Pour exécuter des packages stockés dans des magasins de packages, entrez les valeurs des propriétés d'emplacement de votre package et configuration comme suit :

   ```json
   {
       {
           {
               {
                   "packageLocation": {
                       "type": "PackageStore",
                       "packagePath": "myPackageStore/MyFolder/MyPackage",
                       "typeProperties": {
                           "packagePassword": {
                               "type": "SecureString",
                               "value": "MyEncryptionPassword"
                           },
                           "accessCredential": {
                               "domain": "Azure",
                               "username": "MyStorageAccount",
                               "password": {
                                   "type": "SecureString",
                                   "value": "MyAccountKey"
                               }
                           },
                           "configurationPath": "//MyStorageAccount.file.core.windows.net/MyFileShare/MyConfiguration.dtsConfig",
                           "configurationAccessCredential": {
                               "domain": "Azure",
                               "userName": "MyStorageAccount",
                               "password": {
                                   "type": "AzureKeyVaultSecret",
                                   "store": {
                                       "referenceName": "myAKV",
                                       "type": "LinkedServiceReference"
                                   },
                                   "secretName": "MyAccountKey"
                               }
                           }
                       }
                   }
               }
           }
       }
   }
   ```

2. Dans Azure PowerShell, accédez au dossier `C:\ADF\RunSSISPackage`.

3. Pour créer le pipeline **RunSSISPackagePipeline** , exécutez la cmdlet **Set-AzDataFactoryV2Pipeline** .

   ```powershell
   $DFPipeLine = Set-AzDataFactoryV2Pipeline -DataFactoryName $DataFactory.DataFactoryName `
                                                  -ResourceGroupName $ResGrp.ResourceGroupName `
                                                  -Name "RunSSISPackagePipeline"
                                                  -DefinitionFile ".\RunSSISPackagePipeline.json"
   ```

   Voici l'exemple de sortie :

   ```
   PipelineName      : Adfv2QuickStartPipeline
   ResourceGroupName : <resourceGroupName>
   DataFactoryName   : <dataFactoryName>
   Activities        : {CopyFromBlobToBlob}
   Parameters        : {[inputPath, Microsoft.Azure.Management.DataFactory.Models.ParameterSpecification], [outputPath, Microsoft.Azure.Management.DataFactory.Models.ParameterSpecification]}
   ```

### <a name="run-the-pipeline"></a>Exécuter le pipeline
Utilisez la cmdlet **Invoke-AzDataFactoryV2Pipeline** pour exécuter le pipeline. L’applet de commande renvoie l’ID d’exécution du pipeline pour permettre une surveillance ultérieure.

```powershell
$RunId = Invoke-AzDataFactoryV2Pipeline -DataFactoryName $DataFactory.DataFactoryName `
                                             -ResourceGroupName $ResGrp.ResourceGroupName `
                                             -PipelineName $DFPipeLine.Name
```

### <a name="monitor-the-pipeline"></a>Surveiller le pipeline

Exécutez le script PowerShell suivant afin de vérifier continuellement l’état de l’exécution du pipeline jusqu’à la fin de la copie des données. Copiez ou collez le script suivant dans la fenêtre PowerShell et sélectionnez Entrée. 

```powershell
while ($True) {
    $Run = Get-AzDataFactoryV2PipelineRun -ResourceGroupName $ResGrp.ResourceGroupName `
                                               -DataFactoryName $DataFactory.DataFactoryName `
                                               -PipelineRunId $RunId

    if ($Run) {
        if ($run.Status -ne 'InProgress') {
            Write-Output ("Pipeline run finished. The status is: " +  $Run.Status)
            $Run
            break
        }
        Write-Output  "Pipeline is running...status: InProgress"
    }

    Start-Sleep -Seconds 10
}   
```

Vous pouvez également surveiller le pipeline à l’aide du portail Azure. Pour obtenir des instructions pas à pas, consultez [Surveiller le pipeline](quickstart-create-data-factory-resource-manager-template.md#monitor-the-pipeline).

### <a name="schedule-the-pipeline-with-a-trigger"></a>Planifier le pipeline avec un déclencheur
À l’étape précédente, vous avez exécuté le pipeline à la demande. Vous pouvez également créer un déclencheur de planification pour exécuter le pipeline d’après une planification, par exemple horaire ou quotidienne.

1. Créez un fichier JSON nommé `MyTrigger.json` dans le dossier `C:\ADF\RunSSISPackage` avec le contenu suivant : 
        
   ```json
   {
       "properties": {
           "name": "MyTrigger",
           "type": "ScheduleTrigger",
           "typeProperties": {
               "recurrence": {
                   "frequency": "Hour",
                   "interval": 1,
                   "startTime": "2017-12-07T00:00:00-08:00",
                   "endTime": "2017-12-08T00:00:00-08:00"
               }
           },
           "pipelines": [{
               "pipelineReference": {
                   "type": "PipelineReference",
                   "referenceName": "RunSSISPackagePipeline"
               },
               "parameters": {}
           }]
       }
   }    
   ```
    
1. Dans Azure PowerShell, accédez au dossier `C:\ADF\RunSSISPackage`.
1. Exécutez la cmdlet **Set-AzDataFactoryV2Trigger** pour créer le déclencheur. 

   ```powershell
   Set-AzDataFactoryV2Trigger -ResourceGroupName $ResGrp.ResourceGroupName `
                                   -DataFactoryName $DataFactory.DataFactoryName `
                                   -Name "MyTrigger" -DefinitionFile ".\MyTrigger.json"
   ```
1. Par défaut, le déclencheur est arrêté. Démarrez le déclencheur en exécutant la cmdlet **Start-AzDataFactoryV2Trigger** . 

   ```powershell
   Start-AzDataFactoryV2Trigger -ResourceGroupName $ResGrp.ResourceGroupName `
                                     -DataFactoryName $DataFactory.DataFactoryName `
                                     -Name "MyTrigger" 
   ```
1. Vérifiez que le déclencheur est démarré en exécutant la cmdlet **Get-AzDataFactoryV2Trigger** . 

   ```powershell
   Get-AzDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName `
                                   -DataFactoryName $DataFactoryName `
                                   -Name "MyTrigger"     
   ```    
1. Exécutez la commande ci-dessous après l’heure suivante. Par exemple, si l’heure actuelle est 15h25 UTC, exécutez la commande à 16h00 UTC. 
    
   ```powershell
   Get-AzDataFactoryV2TriggerRun -ResourceGroupName $ResourceGroupName `
                                      -DataFactoryName $DataFactoryName `
                                      -TriggerName "MyTrigger" `
                                      -TriggerRunStartedAfter "2017-12-06" `
                                      -TriggerRunStartedBefore "2017-12-09"
   ```

   Exécutez la requête suivante par rapport à la base de données SSISDB dans votre serveur SQL pour vérifier que le package s’est exécuté. 

   ```sql
   select * from catalog.executions
   ```

## <a name="next-steps"></a>Étapes suivantes
Consultez le billet de blog suivant :
- [Moderniser et étendre vos flux de travail ETL/ELT avec des activités SSIS dans des pipelines Azure Data Factory](https://techcommunity.microsoft.com/t5/SQL-Server-Integration-Services/Modernize-and-Extend-Your-ETL-ELT-Workflows-with-SSIS-Activities/ba-p/388370)
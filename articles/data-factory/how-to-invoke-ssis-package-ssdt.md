---
title: Exécuter des packages SSIS dans SSDT
description: Découvrez comment exécuter des packages SSIS dans Azure à partir de SSDT.
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.author: sawinark
author: swinarko
ms.reviewer: douglasl
manager: mflasko
ms.custom: seo-lt-2019
ms.date: 09/06/2020
ms.openlocfilehash: 6b3c94023daf51559623f69e34b8e2b1f42fde92
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92637239"
---
# <a name="execute-ssis-packages-in-azure-from-ssdt"></a>Exécuter des packages SSIS dans Azure à partir de SSDT

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Cet article décrit la fonctionnalité des projets SQL Server Integration Services (SSIS) pour Azure sur SQL Server Data Tools (SSDT). Il vous permet d’évaluer la compatibilité cloud de vos packages SSIS et de les exécuter sur Azure-SSIS Integration Runtime (IR) dans Azure Data Factory (ADF). Vous pouvez utiliser cette fonctionnalité pour tester vos packages existants avant d’effectuer une opération lift-and-shift ou de les migrer vers Azure, ou de développer de nouveaux packages à exécuter dans Azure.

Grâce à cette fonctionnalité, vous pouvez attacher un runtime d’intégration Azure-SSIS nouvellement créé ou existant aux projets SSIS, puis y exécuter vos packages.  Nous prenons en charge l’exécution de packages à déployer dans le catalogue SSIS (SSISDB) hébergé par votre serveur Azure SQL Database ou votre instance gérée dans le modèle de déploiement de projet. Nous prenons également en charge l’exécution de packages à déployer dans le système de fichiers, Azure Files ou une base de données SQL Server (MSDB) hébergés par votre instance gérée Azure SQL dans le modèle de déploiement de package. 

## <a name="prerequisites"></a>Conditions préalables requises

Pour utiliser cette fonctionnalité, téléchargez et installez la dernière version de SSDT avec l’extension de projets SSIS pour Visual Studio [ici](https://marketplace.visualstudio.com/items?itemName=SSIS.SqlServerIntegrationServicesProjects). Vous pouvez également télécharger et installer la dernière version de SSDT en tant que programme d’installation autonome [ici](/sql/ssdt/download-sql-server-data-tools-ssdt?view=sql-server-2017#ssdt-for-vs-2017-standalone-installer).

## <a name="azure-enable-ssis-projects"></a>Projets SSIS pour Azure

### <a name="creating-new-azure-enabled-ssis-projects"></a>Création de projets SSIS pour Azure

Sur SSDT, vous pouvez créer de nouveaux projets SSIS pour Azure à l’aide du modèle **Projet Integration Services (pour Azure** ).

   ![Nouveau projet SSIS pour Azure](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-new-project.png)

Une fois le projet pour Azure créé, vous êtes invité à vous connecter à SSIS dans Azure Data Factory.

   ![Invite de connexion à Azure-SSIS IR](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-integration-runtime-connect-prompt.png)

Si vous souhaitez vous connecter à votre runtime d’intégration Azure-SSIS immédiatement, reportez-vous à [Connexion à Azure-SSIS IR](#connectssisir) pour plus d’informations. Vous pouvez également vous connecter ultérieurement en cliquant avec le bouton droit sur le nœud de votre projet dans la fenêtre Explorateur de solutions de SSDT pour afficher un menu. Ensuite, sélectionnez l’élément **Se connecter à SSIS dans Azure Data Factory** dans le sous-menu **SSIS dans Azure Data Factory** .

### <a name="azure-enabling-existing-ssis-projects"></a><a name="azureenableproject"></a> Activation pour Azure des projets SSIS existants

Pour les projets SSIS existants, vous pouvez les activer pour Azure en procédant comme suit :

1. Cliquez avec le bouton droit sur le nœud de votre projet dans la fenêtre Explorateur de solutions de SSDT pour afficher un menu. Ensuite, sélectionnez l’élément **Projet pour Azure** dans le sous-menu **SSIS dans Azure Data Factory** pour lancer l’ **Assistant de projet pour Azure** .

   ![Projet SSIS existant pour Azure](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-for-existing-project.png)

2. Dans la page **Sélectionner la configuration Visual Studio** , sélectionnez votre configuration Visual Studio pour appliquer les paramètres d’exécution de package dans Azure. Vous pouvez également en créer une si vous ne l’avez pas déjà fait, voir [Création d’une configuration Visual Studio](/visualstudio/ide/how-to-create-and-edit-configurations?view=vs-2019). Nous vous recommandons d’avoir au moins deux configurations Visual Studio différentes pour les exécutions de package dans les environnements locaux et cloud, afin de pouvoir activer Azure pour votre projet par rapport à la configuration cloud. De cette façon, si vous avez paramétré votre projet ou vos packages, vous pouvez attribuer des valeurs différentes à vos paramètres de projet ou de package à l’exécution en fonction des différents environnements d’exécution (sur votre ordinateur local ou dans Azure). Pour obtenir un exemple, consultez [Basculement d’environnement d’exécution des packages](#switchenvironment).

   ![Sélectionner la configuration Visual Studio](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-select-visual-studio-configurations.png)

3. L’activation pour Azure de vos projets SSIS existants implique que vous définissiez leur version de serveur cible en tant que version la plus récente prise en charge par Azure-SSIS IR. Azure-SSIS IR est actuellement basé sur **SQL Server 2017** . Vérifiez que vos packages ne contiennent pas d’autres composants qui ne sont pas pris en charge sur SQL Server 2017. Veillez également à ce que tous les composants supplémentaires compatibles aient également été installés sur votre runtime d’intégration Azure-SSIS via des configurations personnalisées, voir [Personnalisation de votre runtime d’intégration Azure-SSIS](./how-to-configure-azure-ssis-ir-custom-setup.md). Sélectionnez le bouton **Suivant** pour continuer.

   ![Changer la version du serveur cible](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-switch-target-server-version-step.png)

4. Consultez [Connexion à Azure-SSIS IR](#connectssisir) pour terminer la connexion de votre projet à Azure-SSIS IR.

## <a name="connect-azure-enabled-projects-to-ssis-in-azure-data-factory"></a><a name="connectssisir"></a> Connecter des projets pour Azure à SSIS dans Azure Data Factory

En connectant vos projets pour Azure à SSIS dans ADF, vous pouvez télécharger vos packages dans Azure Files et les exécuter sur le runtime d'exécution Azure-SSIS IR. Pour ce faire, procédez comme suit :

1. Dans la page **Introduction à SSIS dans ADF** , lisez l’introduction et sélectionnez le bouton **Suivant** pour continuer.

   ![Introduction à SSIS dans ADF](media/how-to-invoke-ssis-package-ssdt/ssis-in-adf-connection-wizard.png)

2. Dans la page **Sélectionner SSIS IR dans ADF** , sélectionnez votre instance ADF existante et le runtime d’intégration Azure-SSIS pour exécuter des packages. Vous pouvez également en créer si vous n’en avez pas.
   - Pour sélectionner votre runtime d'exécution Azure-SSIS IR existant, sélectionnez d’abord l’abonnement Azure et l'instance ADF qui conviennent.
   - Si vous sélectionnez une instance ADF existante sans Azure-SSIS IR, sélectionnez le bouton **Créer un runtime d’intégration SSIS** pour en créer un sur le portail ADF. Une fois que vous l’avez créé, vous pouvez revenir à cette page pour sélectionner votre nouveau runtime d’intégration Azure-SSIS.
   - Si vous sélectionnez un abonnement Azure existant sans ADF, sélectionnez le bouton **Créer un runtime d’intégration SSIS** pour lancer l’ **Assistant de création d’un runtime d’intégration** . Dans l’Assistant, vous pouvez entrer votre emplacement et votre préfixe désignés pour que nous puissions créer automatiquement un groupe de ressources Azure, une fabrique de données et un runtime d’intégration SSIS en votre nom, nommés selon le modèle suivant : **YourPrefix-RG/DF/IR-YourCreationTime** . Une fois que vous l’avez créé, vous pouvez revenir à cette page pour sélectionner votre nouvelle instance ADF et votre nouveau runtime d’intégration Azure-SSIS.

   ![Sélectionner SSIS IR dans ADF](media/how-to-invoke-ssis-package-ssdt/ssis-in-adf-connection-wizard2.png)

3. Dans la page **Sélectionner Stockage Azure** , sélectionnez votre compte Stockage Azure pour charger des packages dans Azure Files. Vous pouvez également en créer un si vous n’en avez pas.
   - Pour sélectionner votre compte Stockage Azure existant, sélectionnez d’abord l’abonnement Azure qui convient.
   - Si vous sélectionnez le même abonnement Azure que votre runtime d’intégration Azure-SSIS qui n’a pas de compte Stockage Azure, sélectionnez le bouton **Créer un stockage Azure** . Nous en créerons automatiquement un nouveau en votre nom au même emplacement que votre runtime d’intégration Azure-SSIS, nommé en combinant un préfixe de votre nom de runtime d’intégration Azure-SSIS et sa date de création. Une fois que vous l’avez créé, vous pouvez revenir à cette page pour sélectionner votre nouveau compte Stockage Azure.
   - Si vous sélectionnez un autre abonnement Azure sans compte Stockage Azure, sélectionnez le bouton **Créer Stockage Azure** pour en créer un sur le portail Azure. Une fois que vous l’avez créé, vous pouvez revenir à cette page pour sélectionner votre nouveau compte Stockage Azure.

   ![Sélectionnez Stockage Azure](media/how-to-invoke-ssis-package-ssdt/ssis-in-adf-connection-wizard3.png)

4. Sélectionnez le bouton **Se connecter** pour terminer la connexion de votre projet à Azure-SSIS IR. Nous afficherons votre runtime d’intégration Azure-SSIS et votre compte Stockage Azure sous le nœud **Ressources Azure liées** dans la fenêtre Explorateur de solutions de SSDT. Nous actualiserons et afficherons régulièrement l’état de votre runtime d’intégration Azure-SSIS ici. Vous pourrez gérer votre runtime d’intégration Azure-SSIS en cliquant avec le bouton droit sur son nœud pour afficher un menu, puis en sélectionnant l’élément **Démarrer\Arrêter\Gérer** afin d’accéder au portail ADF pour le faire.

## <a name="assess-ssis-projectpackages-for-executions-in-azure"></a>Évaluer le projet\les packages SSIS pour les exécutions dans Azure

### <a name="assessing-single-or-multiple-packages"></a>Évaluation d’un ou de plusieurs packages

Avant d’exécuter vos packages dans Azure, vous pouvez les évaluer pour détecter les éventuels problèmes de compatibilité avec le cloud. Il s’agit notamment des bloqueurs de migration et des informations supplémentaires que vous devez connaître. 
-  Vous avez la possibilité d’évaluer les différents paquets un par un ou tous les paquets en même temps dans le cadre de votre projet.

   ![Évaluer le package](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-assess-package.png)

   ![Évaluer le projet](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-assess-project.png)

-  Dans la fenêtre **Rapport d’évaluation** de SSDT, vous pouvez trouver tous les problèmes potentiels de compatibilité cloud qui ont été signalés, chacun avec sa propre description détaillée et ses recommandations. Vous pouvez également exporter le rapport d’évaluation dans un fichier CSV qui peut être partagé avec toute personne qui doit atténuer ces problèmes. 

   ![Rapport d’évaluation](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-assess-project-result.png)

### <a name="suppressing-assessment-rules"></a>Suppression des règles d’évaluation

Une fois que vous êtes certain que certains problèmes potentiels de compatibilité cloud ne sont pas applicables ou qu’ils ont été correctement atténués dans vos packages, vous pouvez supprimer les règles d’évaluation correspondantes qui les signalent. Cela permet de réduire le bruit dans les rapports d’évaluation suivants.
-  Sélectionnez le lien **Configurer la suppression des règles d’évaluation** dans la fenêtre **Rapport d’évaluation** de SSDT pour afficher la fenêtre **Paramètres de suppression d’une règle d’évaluation** , dans laquelle vous pouvez sélectionner les règles d’évaluation à supprimer.

   ![Paramètres de suppression d’une règle d’évaluation](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-assessment-rule-suppression-settings.png)

-  Vous pouvez également cliquer avec le bouton droit sur le nœud de votre projet dans la fenêtre Explorateur de solutions de SSDT pour afficher un menu. Sélectionnez l’élément **Paramètres pour Azure** dans le sous-menu **SSIS dans Azure Data Factory** pour afficher une fenêtre contenant les pages de propriétés de votre projet. Sélectionnez la propriété **ID des règles d’évaluation supprimées** dans la section **Paramètres pour Azure** . Enfin, cliquez sur le bouton de sélection ( **…** ) pour afficher la fenêtre **Paramètres de suppression d’une règle d’évaluation** , dans laquelle vous pouvez sélectionner les règles d’évaluation à supprimer.

   ![Paramètres pour Azure](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-azure-enabled-setting.png)

   ![Paramètres de suppression d’une règle d’évaluation via les paramètres pour Azure](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-assessment-rule-suppression-settings-via-azure-enabled-settings.png)

## <a name="execute-ssis-packages-in-azure"></a>Exécuter des packages SSIS dans Azure

### <a name="configuring-azure-enabled-settings"></a><a name="azureenabledsettings"></a> Configuration des paramètres pour Azure

Avant d’exécuter vos packages dans Azure, vous pouvez choisir de configurer vos paramètres pour Azure pour eux. Par exemple, vous pouvez activer l’authentification Windows sur votre runtime d’intégration Azure-SSIS pour accéder aux magasins de données locaux/cloud en procédant comme suit :

1. Cliquez avec le bouton droit sur le nœud de votre projet dans la fenêtre Explorateur de solutions de SSDT pour afficher un menu. Ensemble, sélectionnez l’élément **Paramètres pour Azure** dans le sous-menu **SSIS dans Azure Data Factory** pour afficher une fenêtre contenant les pages de propriétés de votre projet.

   ![Paramètres pour Azure](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-azure-enabled-setting.png)

2. Sélectionnez la propriété **Activer l’authentification Windows** dans la section **Paramètres pour Azure** , puis sélectionnez **True** dans son menu déroulant. Ensuite, sélectionnez la propriété **Informations d’authentification Windows** , puis sélectionnez le bouton de sélection ( **…** ) pour afficher la fenêtre **Informations d’authentification Windows** .

   ![Activer l'authentification Windows](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-windows-authentication-open.png)

3. Entrez vos informations d’authentification Windows. Par exemple, pour accéder à Azure Files, vous pouvez entrer `Azure`, `YourStorageAccountName` et `YourStorageAccountKey` pour le **domaine** , le **nom d’utilisateur** et le **mot de passe** , respectivement.

   ![Informations d’authentification Windows](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-windows-authentication-credential.png)

### <a name="starting-package-executions"></a>Vérifier les exécutions des packages

Après avoir connecté vos projets pour Azure à SSIS dans ADF, évalué leur compatibilité cloud et atténué les problèmes potentiels, vous pouvez exécuter/tester vos packages sur Azure-SSIS IR.
-  Sélectionnez le bouton **Démarrer** dans la barre d’outils SSDT pour dérouler un menu. Ensuite, sélectionnez l’élément **Exécuter dans Azure** .

   ![Exécuter dans Azure](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-execute-package.png)

-  Vous pouvez également cliquer avec le bouton droit sur le nœud de votre package dans la fenêtre Explorateur de solutions de SSDT pour afficher un menu. Ensuite, sélectionnez l’élément **Exécuter le package dans Azure** .

   ![Exécuter le package dans Azure](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-execute-package2.png)

> [!NOTE]
> L'exécution de vos packages dans Azure implique que vous exécutiez Azure-SSIS IR et dès lors, si votre runtime d'exécution Azure-SSIS IR est arrêté, vous serez invité à le démarrer. En dehors des réglages personnalisés, ce processus devrait se terminer dans les 5 minutes. Toutefois, il peut prendre de 20 à 30 minutes environ pour le runtime d’intégration Azure-SSIS se joignant à un réseau virtuel. Après avoir exécuté vos packages dans Azure, vous pouvez arrêter votre runtime d’intégration Azure-SSIS pour gérer son coût d’exécution en cliquant avec le bouton droit sur son nœud dans la fenêtre Explorateur de solutions de SSDT afin d’afficher un menu, puis en sélectionnant l’élément **Démarrer\Arrêter\Gérer** qui vous renvoie vers le portail ADF pour le faire.

### <a name="using-execute-package-task"></a>Utilisation de la tâche d’exécution de package

Si vos packages contiennent des tâches d’exécution de package faisant référence à des packages enfants stockés sur des systèmes de fichiers locaux, suivez ces étapes supplémentaires :

1. Chargez les packages enfants dans Azure Files sous le même compte Stockage Azure connecté à vos projets et récupérez leur nouveau chemin d’accès UNC (convention d’affectation des noms), par exemple `\\YourStorageAccountName.file.core.windows.net\ssdtexecution\YourChildPackage1.dtsx`.

2. Remplacez le chemin d’accès au fichier de ces packages enfants dans le gestionnaire de connexion au fichier des tâches d’exécution de package par leur nouveau chemin d’accès UNC.
   - Si l’ordinateur local qui exécute SSDT n’est pas en mesure d’accéder au nouveau chemin d’accès UNC, vous pouvez l’entrer dans le panneau Propriétés du gestionnaire de connexion au fichier.
   - Vous pouvez également utiliser une variable pour le chemin d’accès au fichier afin d’attribuer la valeur qui convient à l’exécution.

Si vos packages contiennent des tâches d’exécution de package faisant référence à des packages enfants dans le même projet, aucune configuration supplémentaire n’est requise.

### <a name="switching-package-protection-level"></a>Changement du niveau de protection des packages

L’exécution de packages SSIS dans Azure ne prend pas en charge les niveaux de protection **EncryptSensitiveWithUserKey**/**EncryptAllWithUserKey** . Par conséquent, si vos packages sont configurés pour les utiliser, nous les convertirons temporairement en utilisant les niveaux de protection **EncryptSensitiveWithPassword**/**EncryptAllWithPassword** , respectivement. Nous générons également aléatoirement des mots de passe de chiffrement lorsque nous chargerons vos packages dans Azure Files pour les exécuter sur votre runtime d’intégration Azure-SSIS.

> [!NOTE]
> Si vos packages contiennent des tâches d’exécution de packages faisant référence à des packages enfants configurés pour utiliser les niveaux de protection **EncryptSensitiveWithUserKey**/**EncryptAllWithUserKey** , vous devez reconfigurer manuellement ces packages pour utiliser respectivement les niveaux de protection **EncryptSensitiveWithPassword**/**EncryptAllWithPassword** avant de les exécuter.

Si vos packages sont déjà configurés pour utiliser les niveaux de protection **EncryptSensitiveWithPassword**/**EncryptAllWithPassword** , nous les garderons inchangés. Nous continuerons à générer aléatoirement des mots de passe de chiffrement lorsque nous chargerons vos packages dans Azure Files pour les exécuter sur votre runtime d’intégration Azure-SSIS.

### <a name="switching-package-execution-environments"></a><a name="switchenvironment"></a> Basculement d’environnement d’exécution des packages

Si vous paramétrez votre projet/vos packages dans le modèle de déploiement de projet, vous pouvez créer plusieurs configurations Visual Studio pour basculer entre les environnements d’exécution de package. De cette façon, vous pouvez attribuer des valeurs spécifiques à l’environnement à vos paramètres de projet/package à l’exécution. Nous vous recommandons d’avoir au moins deux configurations Visual Studio différentes pour les exécutions de package dans les environnements locaux et cloud, afin de pouvoir activer Azure pour vos projets par rapport à la configuration cloud. Voici un exemple étape par étape de basculement d’environnement d’exécution des packages entre votre ordinateur local et Azure :

1. Supposons que votre package contienne une tâche de système de fichiers qui définit les attributs d’un fichier. Lorsque vous l’exécutez sur votre ordinateur local, il définit les attributs d’un fichier stocké sur votre système de fichiers local. Lorsque vous l’exécutez sur votre runtime d’intégration Azure-SSIS, vous souhaitez qu’il définisse les attributs d’un fichier stocké dans Azure Files. Tout d’abord, créez un paramètre de package de type chaîne et nommez-le **FilePath** pour contenir la valeur du chemin d’accès du fichier cible.

   ![Créer un paramètre de package](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-example-define-parameters.png)

2. Ensuite, sur la page **Général** de la fenêtre **Éditeur de tâche de système de fichiers** , paramétrez la propriété **SourceVariable** dans la section **Connexion source** avec le paramètre de package **FilePath** . 

   ![Paramétrer la connexion source](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-example-update-task-with-parameters.png)

3. Par défaut, vous disposez d’une configuration Visual Studio existante pour les exécutions de packages dans l’environnement local nommée **Développement** . Créez une configuration Visual Studio pour les exécutions de packages dans l’environnement cloud nommée **Azure** , voir [Création d’une configuration Visual Studio](/visualstudio/ide/how-to-create-and-edit-configurations?view=vs-2019), si ce n’est déjà fait.

4. Lorsque vous consultez les paramètres de votre package, sélectionnez le bouton **Ajouter des paramètres aux configurations** pour ouvrir la fenêtre **Gérer les valeurs de paramètre** pour votre package. Ensuite, attribuez des valeurs différentes pour le chemin d’accès du fichier cible au paramètre de package **FilePath** dans les configurations **Développement** et **Azure** .

   ![Attribuer des valeurs de paramètre](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-example-override-parameter.png)

5. Activez pour Azure votre projet par rapport à la configuration cloud, voir [Activation pour Azure des projets SSIS existants](#azureenableproject), si ce n’est déjà fait. Ensuite, configurez les paramètres pour Azure pour activer l’authentification Windows pour que votre runtime d’intégration Azure-SSIS puisse accéder à Azure Files, voir [Configuration des paramètres pour Azure](#azureenabledsettings), si vous ne l’avez pas déjà fait.

6. Exécutez votre package dans Azure. Vous pouvez rebasculer votre environnement d’exécution de package vers votre ordinateur local en sélectionnant la configuration **Développement** .

   ![Changer de configuration Visual Studio](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-example-switch-configurations.png)

### <a name="using-package-configuration-file"></a>Utilisation du fichier de configuration de package

Si vous utilisez des fichiers config de package dans le modèle de déploiement de package, vous pouvez attribuer des valeurs spécifiques à l’environnement à vos propriétés de package à l’exécution. Nous chargerons automatiquement ces fichiers avec vos packages dans Azure Files pour les exécuter sur votre runtime d’intégration Azure-SSIS.

### <a name="checking-package-execution-logs"></a>Vérifier les journaux d’exécution des packages

Après le démarrage de l’exécution de votre package, nous allons mettre en forme et afficher ses journaux dans la fenêtre **Progression** de SSDT. Pour un package à exécution longue, nous mettrons régulièrement à jour son journal. Vous pouvez annuler immédiatement l’exécution de votre package en sélectionnant le bouton **Arrêter** dans la barre d’outils SSDT. Vous pouvez également rechercher temporairement les données brutes de ses journaux sous le chemin d’accès UNC suivant : `\\<YourStorageAccountName>.file.core.windows.net\ssdtexecution\<YourProjectName-FirstConnectTime>\<YourPackageName-tmp-ExecutionTime>\logs`, mais nous les nettoierons au bout d’un jour.

## <a name="current-limitations"></a>Limites actuelles

-  SSDT pour Azure prend en charge uniquement les régions de clouds commerciaux/mondiaux et non les régions des clouds gouvernementaux/nationaux pour l’instant.

## <a name="next-steps"></a>Étapes suivantes

Lorsque l’exécution de vos packages dans Azure à partir de SSDT vous convient, vous pouvez les déployer et les exécuter en tant qu’activités Exécuter un package SSIS dans les pipelines ADF. Pour plus d’informations, consultez [Exécution de packages SSIS en tant qu’activités Exécuter un package SSIS dans les pipelines ADF](./how-to-invoke-ssis-package-ssis-activity.md).
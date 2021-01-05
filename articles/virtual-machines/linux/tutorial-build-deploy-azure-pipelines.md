---
title: Tutoriel - Configurer l’intégration continue et le déploiement continu sur des machines virtuelles Azure à l’aide d’Azure Pipelines
description: Dans ce tutoriel, vous allez voir comment configurer l’intégration continue (CI) et le déploiement continu (CD) d’une application Node.js sur des machines virtuelles Azure à l’aide d’un pipeline Azure basé sur YAML.
author: ushan
tags: azure-devops-pipelines
ms.assetid: ''
ms.service: virtual-machines-linux
ms.topic: tutorial
ms.tgt_pltfrm: azure-pipelines
ms.workload: infrastructure
ms.date: 1/3/2020
ms.author: ushan
ms.custom: devops, devx-track-js
ms.openlocfilehash: 6bc6776df889c5c8ccc6acfe5764549ccf7354a5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91320198"
---
# <a name="tutorial-deploy-your-app-to-linux-virtual-machines-in-azure-using-azure-devops-services-and-azure-pipelines"></a>Tutoriel : Déployer votre application sur des machines virtuelles Linux dans Azure à l’aide d’Azure DevOps Services et d’Azure Pipelines

L’intégration continue (CI) et le déploiement continu (CD) constituent un pipeline via lequel vous pouvez générer, publier et déployer votre code après chaque validation. Ce document fournit les étapes de configuration d’un pipeline CI/CD pour les déploiements réalisés sur plusieurs machines à l’aide d’Azure Pipelines.

Azure Pipelines fournit un ensemble complet d’outils d’automatisation CI/CD pour les déploiements réalisés sur les machines virtuelles, que ce soit localement ou dans le cloud.

Dans ce tutoriel, vous allez configurer un pipeline CI/CD basé sur YAML pour déployer votre application dans un [environnement](/azure/devops/pipelines/process/environments?view=azure-devops) Azure Pipelines, sur des machines virtuelles Linux utilisées comme des ressources agissant chacune comme des serveurs web permettant d’exécuter l’application.

Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Obtenir un exemple d’application
> * Créer un pipeline d’intégration continue Azure Pipelines basé sur YAML pour la génération de l’exemple d’application
> * Créer un environnement Azure Pipelines pour les machines virtuelles Azure
> * Créer un pipeline de déploiement continu Azure Pipelines
> * Exécuter des déploiements déclenchés manuellement ou par CI.

## <a name="before-you-begin"></a>Avant de commencer

* Connectez-vous à votre organisation Azure DevOps Services ( **https://dev.azure.com/** ). 
  Vous pouvez obtenir une [organisation Azure DevOps Services gratuite](https://go.microsoft.com/fwlink/?LinkId=307137&clcid=0x409&wt.mc_id=o~msft~vscom~home-vsts-hero~27308&campaign=o~msft~vscom~home-vsts-hero~27308).

  > [!NOTE]
  > Pour plus d’informations, consultez [Se connecter à Azure DevOps Services](/azure/devops/organizations/projects/connect-to-projects?view=vsts).

*  Vous avez besoin d’une machine virtuelle Linux pour une cible de déploiement.  Pour plus d’informations, consultez [Créer et gérer des machines virtuelles Linux avec l’interface Azure CLI](./tutorial-manage-vm.md).

*  Ouvrez le port d’entrée 80 pour votre machine virtuelle. Pour plus d’informations, consultez [Créer des groupes de sécurité réseau à l’aide du portail Azure](../../virtual-network/tutorial-filter-network-traffic.md).

## <a name="get-your-sample-app-code"></a>Obtenir le code de votre exemple d’application

Si vous disposez déjà d’une application dans GitHub que vous souhaitez déployer, vous pouvez créer un pipeline pour le code de cette application.

Toutefois, si vous êtes un nouvel utilisateur, nous vous conseillons de commencer avec notre exemple de code. Dans ce cas, dupliquez (fork) le dépôt suivant dans GitHub :

#### <a name="java"></a>[Java](#tab/java)

```
https://github.com/spring-projects/spring-petclinic
```

> [!NOTE]
> Petclinic est une application [Java Spring Boot](https://spring.io/guides/gs/spring-boot) créée à l’aide de [Maven](https://spring.io/guides/gs/maven/).

#### <a name="javascript"></a>[JavaScript](#tab/java-script)

```
https://github.com/azure-devops/fabrikam-node
```

> [!NOTE]
> Cette application Node.js a été générée à l’aide de [Yeoman](https://yeoman.io/learning/index.html). Elle utilise Express, bower et Grunt. Elle contient certains packages npm utilisés comme dépendances.
> L’exemple contient également un script qui configure Nginx et déploie l’application. Il est exécuté sur les machines virtuelles. Plus précisément, le script effectue les actions suivantes :
> 1. Il installe Node, Nginx et PM2.
> 2. Il configure Nginx et PM2.
> 3. Il démarre l’application Node.

* * * 

## <a name="prerequisites-for-the-linux-vm"></a>Prérequis pour la machine virtuelle Linux

Les exemples d’applications mentionnés ci-dessus ont été testés sur Ubuntu 16.04 et nous vous recommandons d’utiliser cette même version pour ce guide de démarrage rapide.
Suivez les étapes supplémentaires décrites ci-dessous, selon la pile d’exécution que vous avez utilisée pour l’application.

#### <a name="java"></a>[Java](#tab/java)

- Pour déployer des applications Java Spring Boot et Spring Cloud, créez une machine virtuelle Linux dans Azure à l’aide de [ce modèle](https://azuremarketplace.microsoft.com/marketplace/apps/azul.azul-zulu13-ubuntu-2004), qui fournit un runtime OpenJDK entièrement pris en charge.
- Pour déployer des servlets Java sur un serveur Tomcat, créez une machine virtuelle Linux avec Java 8 à l’aide de [ce modèle](https://azuremarketplace.microsoft.com/marketplace/apps/azul.azul-zulu13-ubuntu-2004) Azure, puis [configurez Tomcat 9.x comme un service](https://tomcat.apache.org/tomcat-9.0-doc/setup.html).
- Pour déployer une application Java EE, utilisez un modèle Azure afin de créer une [machine virtuelle Linux + Java + WebSphere 9.x](https://azuremarketplace.microsoft.com/marketplace/apps/midvision.websphere-application-server-nde-90), une [machine virtuelle Linux + Java + WebLogic 12.x](https://azuremarketplace.microsoft.com/marketplace/apps/oracle.20191009-arm-oraclelinux-wls-admin) ou une [machine virtuelle Linux +Java + WildFly/JBoss 14](https://azuremarketplace.microsoft.com/marketplace/apps/azul.azul-zulu13-ubuntu-2004). 


#### <a name="javascript"></a>[JavaScript](#tab/java-script)

Pour installer une application JavaScript ou Node.js, vous avez besoin d’une machine virtuelle Linux et d’un serveur web Nginx pour déployer l’application.
Si vous n’avez pas encore de machine virtuelle Linux dotée de Nginx, créez-en une maintenant dans Azure en suivant les étapes décrites dans [cet exemple](./quick-create-cli.md).

* * * 

## <a name="create-an-azure-pipelines-environment-with-azure-virtual-machines"></a>Créer un environnement Azure Pipelines avec des machines virtuelles Azure

Les machines virtuelles peuvent être ajoutées en tant que ressources dans un [environnement](/azure/devops/pipelines/process/environments), et peuvent être ciblées pour les déploiements multimachines. L’historique des déploiements effectués dans un environnement permet de suivre les étapes de ces déploiements (machine virtuelle, pipeline, validation).

Vous pouvez créer un environnement dans le hub « **Environnements** » de la section « **Pipelines** ».
1.  Connectez-vous à votre organisation Azure DevOps et accédez à votre projet.
2.  Dans votre projet, accédez à la page **Pipelines**. Choisissez ensuite **Environnements**, puis cliquez sur **Créer un environnement**. Spécifiez un **nom** (obligatoire) pour l’environnement, ainsi qu’une **description**.
3.  Définissez **Machines virtuelles** comme une **Ressource** devant être ajoutée à l’environnement, puis cliquez sur **Suivant**.
4.  Choisissez le système d’exploitation (Windows/Linux), puis **copiez le script d’inscription PowerShell**. 
5.  Exécutez maintenant le script copié à partir d’une invite de commandes PowerShell administrateur, sur chacune des machines virtuelles cibles à inscrire dans cet environnement.
    > [!NOTE]
    > - Le jeton d’accès personnel de l’utilisateur connecté est pré-inséré dans le script qui expire le jour-même, ce qui rend le script copié inutilisable.
    > - Si un agent est déjà en cours d’exécution sur votre machine virtuelle, fournissez un nom unique pour l’« agent » afin de l’inscrire auprès de l’environnement.
6.  Une fois la machine virtuelle inscrite, celle-ci s’affiche comme une ressource d’environnement sous l’onglet « Ressources » de l’environnement.

    ![Création d’une machine virtuelle](media/tutorial-deploy-vms-azure-pipelines/vm-creation.png)

7.  Pour ajouter d’autres machines virtuelles, vous pouvez afficher et copier à nouveau le script en cliquant sur « Ajouter une ressource », puis en définissant « Machines virtuelles » comme une ressource. Ce script restera le même pour toutes les machines virtuelles à ajouter à cet environnement. 
8.  Chaque ordinateur interagit avec Azure Pipelines pour coordonner le déploiement de votre application.

    ![Vue de la ressource Machine virtuelle](media/tutorial-deploy-vms-azure-pipelines/vm-resourceview.png)

9. Vous pouvez ajouter des étiquettes à la machine virtuelle dans le cadre du script d’inscription PowerShell interactif. Vous pouvez également ajouter ou supprimer ces étiquettes dans la vue Ressources en cliquant sur les trois points situés à droite de chaque ressource de machine virtuelle dans la vue Ressources.

   Les étiquettes que vous affectez vous permettent de limiter le déploiement à certaines machines virtuelles lorsque l’environnement est utilisé dans le cadre d’un déploiement. La taille des étiquettes est limitée à 256 caractères, mais il n’existe aucune limite quant au nombre d’étiquettes que vous pouvez utiliser.

   ![Étiquettes de la machine virtuelle](media/tutorial-deploy-vms-azure-pipelines/vm-tags.png)

* * * 

## <a name="define-your-ci-build-pipeline"></a>Définir un pipeline de build d’intégration continue

Vous aurez besoin d’un pipeline de build d’intégration continue (CI) pour publier votre application web, ainsi que d’un script de déploiement pouvant être exécuté localement sur le serveur Ubuntu. Configurez un pipeline de build CI en fonction du runtime que vous souhaitez utiliser. 

1. Connectez-vous à votre organisation Azure DevOps et accédez à votre projet.

1. Dans votre projet, accédez à la page **Pipelines**. Choisissez ensuite l’action pour créer un pipeline.

1. Suivez les étapes de l’Assistant en sélectionnant d’abord **GitHub** comme emplacement du code source.

1. Vous serez peut-être redirigé vers GitHub pour vous connecter. Si c’est le cas, entrez vos informations d’identification GitHub.

1. Lorsque la liste des dépôts s’affiche, sélectionnez le dépôt d’exemples d’applications de votre choix.

1. Azure Pipelines va analyser votre dépôt et recommander un modèle de pipeline adapté.

#### <a name="java"></a>[Java](#tab/java)

Sélectionnez le modèle de **démarrage**, puis copiez l’extrait de code YAML ci-dessous qui permet de générer votre projet Java et d’exécuter les tests avec Apache Maven :

```YAML
jobs:
- job: Build
  displayName: Build Maven Project
  steps:
  - task: Maven@3
    displayName: 'Maven Package'
    inputs:
      mavenPomFile: 'pom.xml'
  - task: CopyFiles@2
    displayName: 'Copy Files to artifact staging directory'
    inputs:
      SourceFolder: '$(System.DefaultWorkingDirectory)'
      Contents: '**/target/*.?(war|jar)'
      TargetFolder: $(Build.ArtifactStagingDirectory)
  - upload: $(Build.ArtifactStagingDirectory)
    artifact: drop
```

Pour plus d’informations, suivez les étapes mentionnées dans [Générer votre application Java avec Maven](/azure/devops/pipelines/ecosystems/java).

#### <a name="javascript"></a>[JavaScript](#tab/java-script)

Sélectionnez le modèle de **démarrage**, puis copiez l’extrait de code YAML ci-dessous qui permet de générer un projet Node.js général avec npm.

```YAML
- stage: Build
  displayName: Build stage
  jobs:  
  - job: Build
    displayName: Build
    - task: NodeTool@0
      inputs:
        versionSpec: '10.x'
      displayName: 'Install Node.js'
    - script: |
        npm install
        npm run build --if-present
        npm run test --if-present
      displayName: 'npm install, build and test'
    - task: ArchiveFiles@2
      displayName: 'Archive files'
      inputs:
        rootFolderOrFile: '$(System.DefaultWorkingDirectory)'
        includeRootFolder: false
        archiveType: zip
        archiveFile: $(Build.ArtifactStagingDirectory)/$(Build.BuildId).zip
        replaceExistingArchive: true
    - upload: $(Build.ArtifactStagingDirectory)/$(Build.BuildId).zip
      artifact: drop
```

Pour plus d’informations, suivez les étapes mentionnées dans [Générer votre application Node.js avec gulp](/azure/devops/pipelines/ecosystems/javascript).

- Examinez le pipeline pour voir à quoi il sert. Vérifiez que toutes les entrées par défaut sont adaptées à votre code.

- Sélectionnez **Enregistrer et exécuter**. Ensuite, sélectionnez **Commiter directement sur la branche maître**, puis choisissez à nouveau **Enregistrer et réexécuter**.

- Une nouvelle exécution est lancée. Attendez la fin de l’exécution.

* * * 

## <a name="define-cd-steps-to-deploy-to-the-linux-vm"></a>Définir les étapes d’un déploiement continu sur une machine virtuelle Linux

1. Modifiez le fichier YAML pour le pipeline ci-dessus en y ajoutant un [travail de déploiement](/azure/devops/pipelines/process/deployment-jobs), puis en référençant l’environnement et les ressources de machine virtuelle dont vous disposez à l’aide de la syntaxe YAML ci-dessous :

   ```YAML
   jobs:  
   - deployment: VMDeploy
     displayName: web
     environment:
       name:  <environment name>
       resourceType: VirtualMachine
       tags: web
   ```
2. Vous pouvez sélectionner des groupes de machines virtuelles appartenant à l’environnement pour recevoir le déploiement en spécifiant les **étiquettes** que vous avez définies pour chaque machine virtuelle de l’environnement.
Le schéma YAML complet du travail de déploiement se trouve [ici](/azure/devops/pipelines/yaml-schema?view=azure-devops&tabs=schema#deployment-job).

3. Vous pouvez spécifier `runOnce` ou `rolling` comme stratégie de déploiement. 

   `runOnce` est la stratégie de déploiement la plus simple, dans laquelle tous les hooks de cycle de vie, à savoir `preDeploy` `deploy`, `routeTraffic` et `postRouteTraffic`, sont exécutés une seule fois. Ensuite, `on:` `success` ou `on:` `failure` est exécuté.

   Voici l’exemple d’extrait de code YAML de `runOnce` :
   ```YAML
   jobs:
   - deployment: VMDeploy
     displayName: web
     pool:
       vmImage: 'Ubuntu-16.04'
     environment:
       name: <environment name>
       resourceType: VirtualMachine
     strategy:
         runOnce:
           deploy:
             steps:
             - script: echo my first deployment
   ```

4. Voici un exemple d’extrait de code YAML que vous pouvez utiliser afin de définir une stratégie de déploiement pour les mises à jour de machines virtuelles, avec un maximum de 5 cibles par itération. `maxParallel` détermine le nombre de cibles qui peuvent simultanément faire l’objet d’un déploiement. La sélection prend en compte le nombre absolu ou le pourcentage de cibles qui doivent rester disponibles à tout moment, à l’exclusion des cibles sur lesquelles un déploiement est en cours. Il est également utilisé pour déterminer les conditions de réussite et d’échec lors d’un déploiement.

   ```YAML
   jobs: 
   - deployment: VMDeploy
     displayName: web
     environment:
       name: <environment name>
       resourceType: VirtualMachine
     strategy:
         rolling:
           maxParallel: 5  #for percentages, mention as x%
           preDeploy:
             steps:
             - download: current
               artifact: drop
             - script: echo initialize, cleanup, backup, install certs
           deploy:
             steps:
             - task: Bash@3
               inputs:
                 targetType: 'inline'
                 script: |
                   # Modify deployment script based on the app type
                   echo "Starting deployment script run"
                   sudo java -jar '$(Pipeline.Workspace)/drop/**/target/*.jar'
           routeTraffic:
             steps:
             - script: echo routing traffic
           postRouteTraffic:
             steps:
             - script: echo health check post-route traffic
           on:
             failure:
               steps:
               - script: echo Restore from backup! This is on failure
             success:
               steps:
               - script: echo Notify! This is on success
   ```

   À chaque exécution de ce travail, l’historique des déploiements est enregistré pour l’environnement `<environment name>` que vous avez créé et dans lequel vous avez inscrit les machines virtuelles.

## <a name="run-your-pipeline-and-get-traceability-views-in-environment"></a>Exécuter le pipeline et suivre le déroulement des déploiements dans l’environnement
La vue Déploiements de l’environnement fournit le suivi complet des validations et des éléments de travail, ainsi qu’un historique des déploiements interpipeline pour chaque environnement ou ressource.

![Vue des déploiements des machines virtuelles](media/tutorial-deploy-vms-azure-pipelines/vm-deployments.png)
  
![Vue des travaux des machines virtuelles](media/tutorial-deploy-vms-azure-pipelines/vm-jobsview.png)

## <a name="next-steps"></a>Étapes suivantes
- Vous pouvez à présent [personnaliser le pipeline](/azure/devops/pipelines/customize-pipeline) que vous venez de créer.
- Pour savoir ce que vous pouvez faire d’autre dans les pipelines YAML, consultez [Informations de référence sur le schéma YAML](/azure/devops/pipelines/yaml-schema).
- Pour découvrir comment déployer une pile LAMP (Linux, Apache, MySQL et PHP), passez au didacticiel suivant.

> [!div class="nextstepaction"]
> [Déploiement d’une pile LAMP dans Azure](tutorial-lamp-stack.md)

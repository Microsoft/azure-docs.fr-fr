---
title: Utiliser Azure DevOps pour créer un pipeline CI/CD pour une tâche Stream Analytics
description: Cet article explique comment configurer un processus de déploiement et d’intégration continus (CI/CD) pour une tâche Azure Stream Analytics dans Azure DevOps
services: stream-analytics
author: su-jie
ms.author: sujie
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 09/10/2020
ms.openlocfilehash: d9b6dfc977aab7d8907b5d3c3851a22f96227d78
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91757756"
---
# <a name="use-azure-devops-to-create-a-cicd-pipeline-for-a-stream-analytics-job"></a>Utiliser Azure DevOps pour créer un pipeline CI/CD pour une tâche Stream Analytics

Dans cet article, vous allez apprendre à créer des pipelines Azure DevOps [build](/azure/devops/pipelines/get-started/pipelines-get-started) et [release](/azure/devops/pipelines/release/define-multistage-release-process) à l’aide des outils Azure Stream Analytics CI/CD.

## <a name="commit-your-stream-analytics-project"></a>Valider votre projet Stream Analytics

Avant de commencer, validez vos projets Stream Analytics complets en tant que fichiers sources dans un référentiel [Azure DevOps](/azure/devops/user-guide/source-control). Vous pouvez référencer cette [exemple de référentiel](https://dev.azure.com/wenyzou/azure-streamanalytics-cicd-demo) et [le code source du projet Stream Analytics ](https://dev.azure.com/wenyzou/_git/azure-streamanalytics-cicd-demo?path=%2FmyASAProject) dans Azure Pipelines.

Les étapes décrites dans cet article utilisent un projet Stream Analytics Visual Studio Code. Si vous utilisez un projet Visual Studio, suivez les étapes décrites dans [Automatiser les builds, les tests et les déploiements d’une tâche Azure Stream Analytics à l’aide des outils CI/CD](cicd-tools.md).

## <a name="create-a-build-pipeline"></a>Créer un pipeline de build

Dans cette section, vous découvrez comment créer un pipeline de build. Vous pouvez consulter cet exemple de [pipeline de build et de test automatiques](https://dev.azure.com/wenyzou/_git/azure-streamanalytics-cicd-demo?path=%2FmyASAProject) dans Azure DevOps.

1. Ouvrez un navigateur web et accédez à votre projet dans Azure DevOps.  

1. Sous **Pipelines** dans le menu de navigation gauche, sélectionnez **Builds**. Sélectionnez ensuite **Nouveau pipeline**.

   :::image type="content" source="media/set-up-cicd-pipeline/new-pipeline.png" alt-text="Créer un pipeline Azure":::

1. Sélectionnez **Utiliser l’éditeur classique** pour créer un pipeline sans YAML.

1. Sélectionnez votre type de source, votre projet d’équipe et votre référentiel. Sélectionnez ensuite **Continuer**.

   :::image type="content" source="media/set-up-cicd-pipeline/select-repo.png" alt-text="Créer un pipeline Azure":::

1. Dans la page **Sélectionner un modèle**, sélectionnez **Projet vide**.

## <a name="install-npm-package"></a>Installer le package npm

1. Dans la page **Tâches**, sélectionnez le signe plus en regard de **Travail d’agent 1**. Entrez *npm* dans la recherche de tâches et sélectionnez **npm**.

   :::image type="content" source="media/set-up-cicd-pipeline/search-npm.png" alt-text="Créer un pipeline Azure":::

2. Donnez à la tâche un **nom d’affichage**. Définissez l'option **Commande** sur *Personnalisée*, puis entrez la commande suivante dans **Commande et arguments**. Conservez les options par défaut restantes.

   ```bash
   install -g azure-streamanalytics-cicd
   ```

   :::image type="content" source="media/set-up-cicd-pipeline/npm-config.png" alt-text="Créer un pipeline Azure":::

## <a name="add-a-build-task"></a>Ajoutez une tâche de build

1. Dans la page **Variables** , sélectionnez **+ Ajouter** dans **Variables de pipeline**. Ajoutez les variables suivantes. Définissez les valeurs suivantes en fonction de vos préférences :

   |Nom de la variable|Valeur|
   |-|-|
   |projectRootPath|[YourProjectName]|
   |outputPath|Sortie|
   |deployPath|Déployer|

2. Dans la page **Tâches**, sélectionnez le signe plus en regard de **Travail d’agent 1**. Recherchez **Ligne de commande**.

3. Donnez à la tâche un **nom d’affichage** et entrez le script suivant. Modifiez le script avec le nom de votre référentiel et le nom de votre projet.

   ```bash
   azure-streamanalytics-cicd build -project $(projectRootPath)/asaproj.json -outputpath $(projectRootPath)/$(outputPath)/$(deployPath)
   ```

   L’image ci-dessous utilise un projet Stream Analytics Visual Studio Code à titre d’exemple.

   :::image type="content" source="media/set-up-cicd-pipeline/command-line-config-build.png" alt-text="Créer un pipeline Azure":::

## <a name="add-a-test-task"></a>Ajouter une Tâche de test

1. Dans la page **Variables** , sélectionnez **+ Ajouter** dans **Variables de pipeline**. Ajoutez les variables suivantes. Modifiez les valeurs avec le chemin de sortie et le nom du référentiel.

   |Nom de la variable|Valeur|
   |-|-|
   |testPath|Test|

   :::image type="content" source="media/set-up-cicd-pipeline/pipeline-variables-test.png" alt-text="Créer un pipeline Azure":::

2. Dans la page **Tâches**, sélectionnez le signe plus en regard de **Travail d’agent 1**. Recherchez **Ligne de commande**.

3. Donnez à la tâche un **nom d’affichage** et entrez le script suivant. Modifiez le script avec le nom de votre fichier projet et le chemin d’accès au fichier de configuration de test. 

   Pour plus d’informations sur la façon d’ajouter et de configurer des cas de test, consultez [Instructions de test automatisées](cicd-tools.md#automated-test).

   ```bash
   azure-streamanalytics-cicd test -project $(projectRootPath)/asaproj.json -outputpath $(projectRootPath)/$(outputPath)/$(testPath) -testConfigPath $(projectRootPath)/test/testConfig.json 
   ```

   :::image type="content" source="media/set-up-cicd-pipeline/command-line-config-test.png" alt-text="Créer un pipeline Azure":::

## <a name="add-a-copy-files-task"></a>Ajouter une tâche de copie des fichiers

Vous devez ajouter une tâche de copie des fichiers pour copier le fichier de résumé de test et les fichiers de modèle Azure Resource Manager dans le dossier de l’artefact. 

1. Dans la page **Tâches**, sélectionnez **+** en regard de **Tâche d’agent 1**. Recherchez **Copier des fichiers**. Entrez ensuite les configurations suivantes. En affectant des `**` à **Contenus**, tous les fichiers des résultats des tests sont copiés.

   |Paramètre|Entrée|
   |-|-|
   |Nom complet|Copier des fichiers vers : $(build.artifactstagingdirectory)|
   |Dossier source|`$(system.defaultworkingdirectory)/$(outputPath)/`|
   |Contenu| `**` |
   |Dossier cible| `$(build.artifactstagingdirectory)`|

2. Développez **Options de contrôle** Sélectionnez **Même en cas d'échec d'une tâche antérieure, sauf si la build a été annulée** dans **Exécuter cette tâche**.

   :::image type="content" source="media/set-up-cicd-pipeline/copy-config.png" alt-text="Créer un pipeline Azure":::

## <a name="add-a-publish-build-artifacts-task"></a>Ajouter une tâche Publier des artefacts de build

1. Dans la page **Tâches**, sélectionnez le signe plus en regard de **Travail d’agent 1**. Recherchez **Publier des artefacts de build** et sélectionnez l’option accompagnée d'une icône de flèche noire.

2. Développez **Options de contrôle** Sélectionnez **Même en cas d'échec d'une tâche antérieure, sauf si la build a été annulée** dans **Exécuter cette tâche**.

   :::image type="content" source="media/set-up-cicd-pipeline/publish-config.png" alt-text="Créer un pipeline Azure":::

## <a name="save-and-run"></a>Enregistrer et exécuter

Après avoir ajouté les tâches npm, ligne de commande, copier les fichiers et publier les artefacts de build, sélectionnez **Enregistrer et mettre en file d'attente**. Lorsque vous y êtes invité, entrez un commentaire d'enregistrement et sélectionnez **Enregistrer et exécuter**. Vous pouvez télécharger les résultats des tests à partir de la page **Résumé** du pipeline.

## <a name="check-the-build-and-test-results"></a>Vérifier les résultats de build et de test

Le fichier de résumé de test et les fichiers de modèle Azure Resource Manager se trouvent dans le dossier **Publié**.

   :::image type="content" source="media/set-up-cicd-pipeline/check-build-test-result.png" alt-text="Créer un pipeline Azure":::

   :::image type="content" source="media/set-up-cicd-pipeline/check-drop-folder.png" alt-text="Créer un pipeline Azure":::

## <a name="release-with-azure-pipelines"></a>Mettre en production avec Azure Pipelines

Dans cette section, vous découvrez comment créer un pipeline de mise en production. Vous pouvez référencer cet exemple de [pipeline de mise en production](https://dev.azure.com/wenyzou/azure-streamanalytics-cicd-demo/_release?_a=releases&view=mine&definitionId=2&preserve-view=true) dans Azure DevOps.

Ouvrez un navigateur web et accédez à votre projet Azure Stream Analytics Visual Studio Code.

1. Sous **Pipelines** dans le menu de navigation gauche, sélectionnez **Mises en production**. Sélectionnez ensuite **Nouveau pipeline**.

2. Sélectionnez **Démarrer par une tâche vide**.

3. Dans la zone **Artefacts**, sélectionnez **+ Ajouter un artefact**. Sous **Source**, sélectionnez le pipeline de build que vous venez de créer, puis **Ajouter**.

   :::image type="content" source="media/set-up-cicd-pipeline/build-artifact.png" alt-text="Créer un pipeline Azure":::

4. Remplacez le nom de **Étape 1** par **Travail de déploiement dans l’environnement de test**.

5. Ajoutez une nouvelle étape et nommez-la **Travail de déploiement dans l'environnement de production**.

### <a name="add-deploy-tasks"></a>Ajouter des tâches de déploiement

1. Dans la liste déroulante des tâches, sélectionnez **Travail de déploiement dans l’environnement de test**.

2. Sélectionnez **+** en regard de **Tâche d'agent** et recherchez **Déploiement du modèle ARM**. Entrez les paramètres suivants :

   |Paramètre|Valeur|
   |-|-|
   |Nom complet| *Déployer myASAProject*|
   |Abonnement Azure| Choisissez votre abonnement.|
   |Action| *Créer ou mettre à jour un groupe de ressources*|
   |Resource group| Choisissez un nom pour le groupe de ressources de test qui contiendra votre travail Stream Analytics.|
   |Emplacement|Sélectionnez l’emplacement de votre groupe de ressources de test.|
   |Emplacement du modèle| Artefact lié|
   |Modèle| $(System.DefaultWorkingDirectory)/_azure-streamanalytics-cicd-demo-CI-Deploy/drop/myASAProject.JobTemplate.json |
   |Paramètres de modèle|$(System.DefaultWorkingDirectory)/_azure-streamanalytics-cicd-demo-CI-Deploy/drop/myASAProject.JobTemplate.parameters.json |
   |Remplacer les paramètres du modèle|<arm_template_parameter> "your value". Vous pouvez définir les paramètres à l’aide de **Variables**.|
   |Mode de déploiement|Incrémentiel|

3. Dans la liste déroulante des tâches, sélectionnez **Travail de déploiement dans l’environnement de production**.

4. Sélectionnez **+** en regard de **Tâche d'agent** et recherchez *Déploiement du modèle ARM*. Entrez les paramètres suivants :

   |Paramètre|Valeur|
   |-|-|
   |Nom complet| *Deploy myASAProject*|
   |Abonnement Azure| Choisissez votre abonnement.|
   |Action| *Créer ou mettre à jour un groupe de ressources*|
   |Resource group| Choisissez un nom pour le groupe de ressources de production qui contiendra votre travail Stream Analytics.|
   |Emplacement|Sélectionnez l’emplacement de votre groupe de ressources de production.|
   |Emplacement du modèle| *Artefact lié*|
   |Modèle| $(System.DefaultWorkingDirectory)/_azure-streamanalytics-cicd-demo-CI-Deploy/drop/myASAProject.JobTemplate.json |
   |Paramètres de modèle|$(System.DefaultWorkingDirectory)/_azure-streamanalytics-cicd-demo-CI-Deploy/drop/myASAProject.JobTemplate.parameters.json |
   |Remplacer les paramètres du modèle|-<arm_template_parameter> "your value".|
   |Mode de déploiement|Incrémentiel|

### <a name="create-a-release"></a>Créer une mise en production

Pour créer une mise en production, sélectionnez **Créer une mise en production** dans le coin supérieur droit.

:::image type="content" source="media/set-up-cicd-pipeline/create-release.png" alt-text="Créer un pipeline Azure":::

## <a name="next-steps"></a>Étapes suivantes

* [Intégration continue et déploiement continu sur Azure Stream Analytics](cicd-overview.md)
* [Automatiser le build, le test et le déploiement d’une tâche Azure Stream Analytics à l’aide des outils CI/CD](cicd-tools.md)
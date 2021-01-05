---
title: Intégration continue et déploiement continu sur des appareils Azure IoT Edge - Azure IoT Edge
description: Configurer l’intégration continue et le déploiement continu à l’aide de YAML - Azure IoT Edge avec Azure DevOps, Azure Pipelines
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 08/20/2019
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 444ab8ccfe5a8441a4fd7d280e33d8e929d9387d
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96435887"
---
# <a name="continuous-integration-and-continuous-deployment-to-azure-iot-edge-devices"></a>Intégration continue et déploiement continu sur des appareils Azure IoT Edge

Vous pouvez facilement adopter DevOps avec vos applications Azure IoT Edge via les tâches Azure IoT Edge intégrées dans Azure Pipelines. Cet article montre comment utiliser les fonctionnalités d’intégration continue et de déploiement continu d’Azure Pipelines pour générer, tester et déployer des applications rapidement et efficacement sur Azure IoT Edge à l’aide de YAML. Vous pouvez également [utiliser l’éditeur classique](how-to-continuous-integration-continuous-deployment-classic.md).

![Diagramme - Branches CI et CD pour le développement et la production](./media/how-to-continuous-integration-continuous-deployment/model.png)

Dans cet article, vous allez apprendre à utiliser les [tâches Azure IoT Edge](/azure/devops/pipelines/tasks/build/azure-iot-edge) intégrées pour Azure Pipelines afin de créer des pipelines de build et de mise en production pour votre solution IoT Edge. Chaque tâche Azure IoT Edge ajoutée à votre pipeline implémente l’une des quatre actions suivantes :

 | Action | Description |
 | --- | --- |
 | Générer les images de module | Génère les images conteneur à partir du code de votre solution IoT Edge.|
 | Envoyer les images de module | Envoie les images de module au registre de conteneurs que vous avez spécifié. |
 | Générer le manifeste de déploiement | Génère le fichier final du manifeste de déploiement IoT Edge à partir d’un fichier deployment.template.json et des variables. |
 | Déployer sur des appareils IoT Edge | Crée des déploiements IoT Edge sur un ou plusieurs appareils IoT Edge. |

Sauf indication contraire, les procédures décrites dans cet article n’explorent pas toutes les fonctionnalités disponibles par le biais des paramètres de tâche. Pour plus d’informations, consultez les rubriques suivantes :

* [Version de la tâche](/azure/devops/pipelines/process/tasks?tabs=yaml#task-versions)
* **Avancé** - Le cas échéant, spécifiez les modules que vous ne souhaitez pas générer.
* [Options de contrôle](/azure/devops/pipelines/process/tasks?tabs=yaml#task-control-options)
* [Variables d’environnement](/azure/devops/pipelines/process/variables?tabs=yaml#environment-variables)
* [Variables de sortie](/azure/devops/pipelines/process/variables?tabs=yaml#use-output-variables-from-tasks)

## <a name="prerequisites"></a>Prérequis

* Référentiel Azure Repos. Si vous n’en avez pas, vous pouvez [créer un référentiel Git dans votre projet](/azure/devops/repos/git/create-new-repo). Pour cet article, nous avons créé un référentiel appelé **IoTEdgeRepo**.
* Solution IoT Edge validée et envoyée (push) vers votre référentiel. Si vous souhaitez créer un exemple de solution pour tester cet article, suivez la procédure décrite dans [Développer et déboguer des modules dans Visual Studio Code](how-to-vs-code-develop-module.md) ou [Développer et déboguer des modules C# dans Visual Studio](./how-to-visual-studio-develop-module.md). Pour cet article, nous avons créé dans notre dépôt une solution nommée **IoTEdgeSolution** qui contient le code d’un module nommé **filtermodule**.

   Pour cet article, vous avez uniquement besoin du dossier de solution créé par les modèles IoT Edge dans Visual Studio Code ou Visual Studio. Vous n'avez pas à créer, envoyer (push), déployer ou déboguer ce code avant de continuer. Vous configurerez ces processus dans Azure Pipelines.

   Si vous créez une solution, commencez par cloner votre référentiel localement. Ensuite, lorsque vous créez la solution, vous pouvez choisir de le faire directement dans le dossier du référentiel. De là, vous pouvez facilement valider et envoyer (push) les nouveaux fichiers.

* Registre de conteneurs vers lequel envoyer (push) vos images de module. Vous pouvez utiliser [Azure Container Registry](../container-registry/index.yml) ou un registre tiers.
* Un [hub IoT](../iot-hub/iot-hub-create-through-portal.md) Azure actif avec au moins deux appareils IoT Edge à des fins de test distinct des phases de déploiement de test et de production. Vous pouvez suivre les articles de démarrage rapide pour créer un appareil IoT Edge sur [Linux](quickstart-linux.md) ou [Windows](quickstart.md).

Pour plus d’informations sur l’utilisation d’Azure Repos, consultez [Partager votre code avec Visual Studio et Azure Repos](/azure/devops/repos/git/share-your-code-in-git-vs).

## <a name="create-a-build-pipeline-for-continuous-integration"></a>Créer un pipeline de build pour l’intégration continue

Dans cette section, vous créez un pipeline de build. Vous configurez le pipeline afin qu’il s’exécute automatiquement lorsque vous archivez des modifications dans l’exemple de solution IoT Edge et qu’il publie des journaux de génération.

1. Connectez-vous à votre organisation Azure DevOps (`https://dev.azure.com/{your organization}`), puis ouvrez le projet contenant votre référentiel de solution IoT Edge.

   ![Ouvrir votre projet DevOps](./media/how-to-continuous-integration-continuous-deployment/initial-project.png)

2. Dans le menu du volet gauche de votre projet, sélectionnez **Pipelines**. Sélectionnez **Créer un pipeline** au centre de la page. Ou, si vous avez déjà des pipelines de build, sélectionnez le bouton **Nouveau pipeline** en haut à droite.

    ![Créez un pipeline de build à l’aide du bouton Nouveau pipeline](./media/how-to-continuous-integration-continuous-deployment/add-new-pipeline.png)

3. Dans la page **Où se trouve votre code ?** , sélectionnez **Azure Repos Git `YAML`** . Si vous souhaitez utiliser l’éditeur classique pour créer les pipelines de build de votre projet, consultez le [guide de l’éditeur classique](how-to-continuous-integration-continuous-deployment-classic.md).

4. Sélectionnez le référentiel pour lequel vous créez un pipeline.

    ![Sélectionnez le référentiel de votre pipeline de build](./media/how-to-continuous-integration-continuous-deployment/select-repository.png)

5. Dans la page **Configurer votre pipeline**, sélectionnez **Pipeline de démarrage**. Si vous avez un fichier YAML Azure Pipelines préexistant que vous souhaitez utiliser pour créer ce pipeline, vous pouvez sélectionner **Fichier YAML Azure Pipelines existant** et spécifier la branche et le chemin du fichier dans le référentiel.

    ![Sélectionnez un pipeline de démarrage ou un fichier YAML Azure Pipelines existant pour commencer votre pipeline de build](./media/how-to-continuous-integration-continuous-deployment/configure-pipeline.png)

6. Dans la page **Passer en revue le fichier YAML de pipeline**, vous pouvez cliquer sur le nom par défaut `azure-pipelines.yml` pour renommer le fichier de configuration de votre pipeline.

   Sélectionnez **Afficher l’Assistant** pour ouvrir la palette **Tâches**.

    ![Sélectionnez Afficher l’Assistant pour ouvrir la palette Tâches](./media/how-to-continuous-integration-continuous-deployment/show-assistant.png)

7. Pour ajouter une tâche, placez votre curseur à la fin du fichier YAML ou à l’endroit où vous souhaitez que les instructions pour votre tâche soient ajoutées. Recherchez et sélectionnez **Azure IoT Edge**. Renseignez les paramètres de la tâche comme suit. Sélectionnez ensuite **Ajouter**.

   | Paramètre | Description |
   | --- | --- |
   | Action | Sélectionnez **Générer les images de module**. |
   | Fichier .template.json | Spécifiez l’emplacement du fichier **deployment.template.json** dans le référentiel qui contient votre solution IoT Edge. |
   | Plateforme par défaut | Sélectionnez le système d’exploitation approprié pour vos modules en fonction de l’appareil IoT Edge ciblé. |

    ![Utilisez la palette Tâches pour ajouter des tâches à votre pipeline.](./media/how-to-continuous-integration-continuous-deployment/add-build-task.png)

   >[!TIP]
   > Une fois chaque tâche ajoutée, l’éditeur met automatiquement en surbrillance les lignes ajoutées. Pour éviter tout remplacement accidentel, désélectionnez les lignes et fournissez un nouvel espace pour la tâche suivante avant d’ajouter des tâches supplémentaires.

8. Répétez ce processus pour ajouter trois tâches supplémentaires avec les paramètres suivants :

   * Tâche : **Azure IoT Edge**

       | Paramètre | Description |
       | --- | --- |
       | Action | Sélectionnez **Envoyer les images de module**. |
       | Type de registre de conteneurs | Utilisez le type par défaut : **Azure Container Registry**. |
       | Abonnement Azure | Sélectionnez votre abonnement. |
       | Azure Container Registry | Choisissez le registre que vous souhaitez utiliser pour le pipeline. |
       | Fichier .template.json | Spécifiez l’emplacement du fichier **deployment.template.json** dans le référentiel qui contient votre solution IoT Edge. |
       | Plateforme par défaut | Sélectionnez le système d’exploitation approprié pour vos modules en fonction de l’appareil IoT Edge ciblé. |

   * Tâche : **Copier des fichiers**

       | Paramètre | Description |
       | --- | --- |
       | Dossier source | Dossier source à partir duquel effectuer la copie. La racine du référentiel est vide. Utilisez des variables si des fichiers ne se trouvent pas dans le référentiel. Exemple : `$(agent.builddirectory)`.
       | Contenu | Ajoutez deux lignes : `deployment.template.json` et `**/module.json`. |
       | Dossier cible | Spécifiez la variable `$(Build.ArtifactStagingDirectory)`. Pour en savoir plus sur la description, consultez [Générer des variables](/azure/devops/pipelines/build/variables?tabs=yaml#build-variables). |

   * Tâche : **Publier des artefacts de build**

       | Paramètre | Description |
       | --- | --- |
       | Chemin de publication | Spécifiez la variable `$(Build.ArtifactStagingDirectory)`. Pour en savoir plus sur la description, consultez [Générer des variables](/azure/devops/pipelines/build/variables?tabs=yaml#build-variables). |
       | Nom de l’artefact | Spécifiez le nom par défaut : `drop` |
       | Emplacement de publication des artefacts | Utilisez l’emplacement par défaut : `Azure Pipelines` |

9. Sélectionnez **Enregistrer** dans la liste déroulante **Enregistrer et exécuter** en haut à droite.

10. Le déclencheur pour l’intégration continue est activé par défaut pour votre pipeline YAML. Si vous souhaitez modifier ces paramètres, sélectionnez votre pipeline, puis cliquez sur **Modifier** en haut à droite. Sélectionnez **Plus d’actions** en regard du bouton **Exécuter** en haut à droite et accédez à **Déclencheurs**. L’**Intégration continue** est indiquée comme étant activée sous le nom de votre pipeline. Si vous souhaitez afficher les détails du déclencheur, cochez la case **Remplacer le déclencheur d’intégration continue YAML à partir d’ici**.

    ![Pour passer en revue les paramètres de déclencheur de votre pipeline, consultez Déclencheurs sous Plus d’actions.](./media/how-to-continuous-integration-continuous-deployment/check-trigger-settings.png)

Passez à la section suivante pour générer le pipeline de mise en production.

[!INCLUDE [iot-edge-create-release-pipeline-for-continuous-deployment](../../includes/iot-edge-create-release-pipeline-for-continuous-deployment.md)]

[!INCLUDE [iot-edge-verify-iot-edge-continuous-integration-continuous-deployment](../../includes/iot-edge-verify-iot-edge-continuous-integration-continuous-deployment.md)]

## <a name="next-steps"></a>Étapes suivantes

* Exemple de bonnes pratiques IoT Edge DevOps dans [Azure DevOps Starter pour IoT Edge](how-to-devops-starter.md)
* Pour plus d’informations sur le déploiement IoT Edge, consultez [Comprendre les déploiements IoT Edge pour les appareils uniques ou à l’échelle](module-deployment-monitoring.md).
* Suivez les étapes pour créer, mettre à jour ou supprimer un déploiement dans [Déployer et surveiller les modules IoT Edge à grande échelle](how-to-deploy-at-scale.md).
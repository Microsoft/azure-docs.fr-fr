---
title: Intégration continue et déploiement continu sur des appareils Azure IoT Edge (éditeur classique) - Azure IoT Edge
description: Configurer l’intégration continue et le déploiement continu à l’aide de l’éditeur classique - Azure IoT Edge avec Azure DevOps, Azure Pipelines
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 08/26/2020
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: b9b842b94d66cf91ad836b8ae61df1b3d3f34293
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96435941"
---
# <a name="continuous-integration-and-continuous-deployment-to-azure-iot-edge-devices-classic-editor"></a>Intégration continue et déploiement continu sur des appareils Azure IoT Edge (éditeur classique)

Vous pouvez facilement adopter DevOps avec vos applications Azure IoT Edge via les tâches Azure IoT Edge intégrées dans Azure Pipelines. Cet article montre comment utiliser les fonctionnalités d’intégration continue et de déploiement continu d’Azure Pipelines pour générer, tester et déployer des applications rapidement et efficacement sur Azure IoT Edge à l’aide de l’ éditeur classique. Vous pouvez également [utiliser YAML](how-to-continuous-integration-continuous-deployment.md).

![Diagramme - Branches CI et CD pour le développement et la production](./media/how-to-continuous-integration-continuous-deployment-classic/model.png)

Dans cet article, vous allez apprendre à utiliser les [tâches Azure IoT Edge](/azure/devops/pipelines/tasks/build/azure-iot-edge) intégrées pour Azure Pipelines afin de créer des pipelines de build et de mise en production pour votre solution IoT Edge. Chaque tâche Azure IoT Edge ajoutée à votre pipeline implémente l’une des quatre actions suivantes :

 | Action | Description |
 | --- | --- |
 | Générer les images de module | Génère les images conteneur à partir du code de votre solution IoT Edge.|
 | Envoyer les images de module | Envoie les images de module au registre de conteneurs que vous avez spécifié. |
 | Générer le manifeste de déploiement | Génère le fichier final du manifeste de déploiement IoT Edge à partir d’un fichier deployment.template.json et des variables. |
 | Déployer sur des appareils IoT Edge | Crée des déploiements IoT Edge sur un ou plusieurs appareils IoT Edge. |

Sauf indication contraire, les procédures décrites dans cet article n’explorent pas toutes les fonctionnalités disponibles par le biais des paramètres de tâche. Pour plus d’informations, consultez les rubriques suivantes :

* [Version de la tâche](/azure/devops/pipelines/process/tasks?tabs=classic#task-versions)
* **Avancé** - Le cas échéant, spécifiez les modules que vous ne souhaitez pas générer.
* [Options de contrôle](/azure/devops/pipelines/process/tasks?tabs=classic#task-control-options)
* [Variables d’environnement](/azure/devops/pipelines/process/variables?tabs=classic#environment-variables)
* [Variables de sortie](/azure/devops/pipelines/process/variables?tabs=classic#use-output-variables-from-tasks)

## <a name="prerequisites"></a>Prérequis

* Référentiel Azure Repos. Si vous n’en avez pas, vous pouvez [créer un référentiel Git dans votre projet](/azure/devops/repos/git/create-new-repo). Pour cet article, nous avons créé un référentiel appelé **IoTEdgeRepo**.
* Solution IoT Edge validée et envoyée (push) vers votre référentiel. Si vous souhaitez créer un exemple de solution pour tester cet article, suivez la procédure décrite dans [Développer et déboguer des modules dans Visual Studio Code](how-to-vs-code-develop-module.md) ou [Développer et déboguer des modules C# dans Visual Studio](./how-to-visual-studio-develop-module.md). Pour cet article, nous avons créé dans notre dépôt une solution nommée **IoTEdgeSolution** qui contient le code d’un module nommé **filtermodule**.

   Pour cet article, vous avez uniquement besoin du dossier de solution créé par les modèles IoT Edge dans Visual Studio Code ou Visual Studio. Vous n'avez pas à créer, envoyer (push), déployer ou déboguer ce code avant de continuer. Vous configurerez ces processus dans Azure Pipelines.

   Si vous créez une solution, commencez par cloner votre référentiel localement. Ensuite, lorsque vous créez la solution, vous pouvez choisir de le faire directement dans le dossier du référentiel. De là, vous pouvez facilement valider et envoyer (push) les nouveaux fichiers.

* Registre de conteneurs vers lequel envoyer (push) vos images de module. Vous pouvez utiliser [Azure Container Registry](../container-registry/index.yml) ou un registre tiers.
* Un [hub IoT](../iot-hub/iot-hub-create-through-portal.md) Azure actif avec au moins deux appareils IoT Edge à des fins de test distinct des phases de déploiement de test et de production. Vous pouvez suivre les articles de démarrage rapide pour créer un appareil IoT Edge sur [Linux](quickstart-linux.md) ou [Windows](quickstart.md).

## <a name="create-a-build-pipeline-for-continuous-integration"></a>Créer un pipeline de build pour l’intégration continue

Dans cette section, vous créez un pipeline de build. Vous configurez le pipeline afin qu’il s’exécute automatiquement lorsque vous archivez des modifications dans l’exemple de solution IoT Edge et qu’il publie des journaux de génération.

1. Connectez-vous à votre organisation Azure DevOps (`https://dev.azure.com/{your organization}`), puis ouvrez le projet contenant votre référentiel de solution IoT Edge.

    ![Ouvrir votre projet DevOps](./media/how-to-continuous-integration-continuous-deployment-classic/initial-project.png)

2. Dans le menu du volet gauche de votre projet, sélectionnez **Pipelines**. Sélectionnez **Créer un pipeline** au centre de la page. Ou, si vous avez déjà des pipelines de build, sélectionnez le bouton **Nouveau pipeline** en haut à droite.

    ![Créer un pipeline de build](./media/how-to-continuous-integration-continuous-deployment-classic/add-new-pipeline.png)

3. En bas de la page **Où se trouve votre code ?** , sélectionnez **Utiliser l’éditeur classique**. Si vous souhaitez utiliser YAML pour créer les pipelines de build de votre projet, consultez le [guide YAML](how-to-continuous-integration-continuous-deployment.md).

    ![Sélectionnez Utiliser l’éditeur classique](./media/how-to-continuous-integration-continuous-deployment-classic/create-without-yaml.png)

4. Suivez les invites pour créer votre pipeline.

   1. Fournissez les informations sur la source de votre nouveau pipeline de build. Sélectionnez le type de source **Azure Repos Git**, puis sélectionnez le projet, le référentiel et la branche où se trouve le code de votre solution IoT Edge. Sélectionnez ensuite **Continuer**.

      ![Sélectionner la source de votre pipeline](./media/how-to-continuous-integration-continuous-deployment-classic/pipeline-source.png)

   2. Sélectionnez **Projet vide** plutôt qu'un modèle.

      ![Démarrez avec un travail vide pour votre pipeline de build](./media/how-to-continuous-integration-continuous-deployment-classic/start-with-empty-build-job.png)

5. Une fois votre pipeline créé, vous accédez à l’éditeur de pipeline. Ici, vous pouvez modifier le nom du pipeline, le pool d’agents et la spécification de l’agent.

   Vous pouvez sélectionner un pool hébergé par Microsoft ou un pool auto-hébergé que vous gérez.

   Dans la description de votre pipeline, sélectionnez la spécification d’agent qui convient en fonction de votre plateforme cible :

   * Si vous souhaitez générer vos modules dans des conteneurs de plateforme amd64 pour Linux, choisissez **ubuntu-16.04**.

   * Si vous souhaitez générer vos modules dans des conteneurs de plateforme amd64 pour Windows 1809, vous devez [configurer l'agent auto-hébergé sur Windows](/azure/devops/pipelines/agents/v2-windows).

   * Si vous souhaitez générer vos modules dans des conteneurs de plateforme arm32v7 ou arm64 pour Linux, vous devez [configurer l'agent auto-hébergé sur Linux](https://devblogs.microsoft.com/iotdev/setup-azure-iot-edge-ci-cd-pipeline-with-arm-agent).

    ![Configurer la spécification de l’agent de build](./media/how-to-continuous-integration-continuous-deployment-classic/configure-env.png)

6. Votre pipeline est préconfiguré avec un travail appelé **Travail d'agent 1**. Sélectionnez le signe plus ( **+** ) pour ajouter quatre tâches au travail : **Azure IoT Edge** deux fois, **Copier des fichiers** une fois et **Publier des artefacts de build** une fois. Recherchez chaque tâche et pointez sur le nom de la tâche pour afficher le bouton **Ajouter**.

   ![Ajouter une tâche Azure IoT Edge](./media/how-to-continuous-integration-continuous-deployment-classic/add-iot-edge-task.png)

   Une fois les quatre tâches ajoutées, le travail de votre agent est semblable à l'exemple suivant :

   ![Quatre tâches dans le pipeline de build](./media/how-to-continuous-integration-continuous-deployment-classic/add-tasks.png)

7. Sélectionnez la première tâche **Azure IoT Edge** pour la modifier. Cette tâche génère tous les modules de la solution avec la plateforme cible que vous spécifiez. Modifiez la tâche avec les valeurs suivantes :

    | Paramètre | Description |
    | --- | --- |
    | Nom complet | Le nom d’affichage est automatiquement mis à jour lorsque le champ Action change. |
    | Action | Sélectionnez **Générer les images de module**. |
    | Fichier .template.json | Sélectionnez les points de suspension (**...**) et accédez au fichier **deployment.template.json** du référentiel contenant votre solution IoT Edge. |
    | Plateforme par défaut | Sélectionnez le système d’exploitation approprié pour vos modules en fonction de l’appareil IoT Edge ciblé. |
    | Variables de sortie | Fournissez un nom de référence à associer au chemin où le fichier deployment.json est généré, par exemple **edge**. |

   Ces configurations utilisent le référentiel et la balise de l’image qui sont définis dans le fichier `module.json` pour nommer et baliser l’image de module. **Générer des images de module** permet également de remplacer les variables par la valeur exacte que vous définissez dans le fichier `module.json`. Dans Visual Studio ou Visual Studio Code, vous spécifiez la valeur réelle dans un fichier `.env`. Dans Azure Pipelines, vous définissez la valeur dans l’onglet **Variables du pipeline**. Sélectionnez l’onglet **Variables** dans le menu de l’éditeur de pipeline et configurez le nom et la valeur comme suit :

    * **ACR_ADDRESS** : valeur de votre **Serveur de connexion** Azure Container Registry. Vous pouvez récupérer le serveur de connexion à partir de la page Vue d’ensemble de votre registre de conteneurs dans le portail Azure.

    Si votre projet contient d’autres variables, vous pouvez spécifier le nom et la valeur dans cet onglet. **Générer des images de module** reconnaît uniquement les variables qui sont au format `${VARIABLE}`. Veillez à utiliser ce format dans vos fichiers `**/module.json`.

8. Sélectionnez la seconde tâche **Azure IoT Edge** pour la modifier. Cette tâche envoie (push) toutes les images de module vers le registre de conteneurs que vous sélectionnez.

    | Paramètre | Description |
    | --- | --- |
    | Nom complet | Le nom d’affichage est automatiquement mis à jour lorsque le champ Action change. |
    | Action | Sélectionnez **Envoyer les images de module**. |
    | Type de registre de conteneurs | Utilisez le type par défaut : `Azure Container Registry`. |
    | Abonnement Azure | Choisissez votre abonnement. |
    | Azure Container Registry | Sélectionnez le type de registre de conteneurs que vous utilisez pour stocker vos images de module. Le formulaire change selon le type de registre que vous sélectionnez. Si vous choisissez **Azure Container Registry**, utilisez les listes déroulantes pour sélectionner l’abonnement Azure et le nom de votre registre de conteneurs. Si vous choisissez **Registre de conteneurs générique**, sélectionnez **Nouveau** pour créer une connexion de service de registre. |
    | Fichier .template.json | Sélectionnez les points de suspension (**...**) et accédez au fichier **deployment.template.json** du référentiel contenant votre solution IoT Edge. |
    | Plateforme par défaut | Sélectionnez le système d’exploitation approprié pour vos modules en fonction de l’appareil IoT Edge ciblé. |
    | Ajouter les informations d’identification du registre au manifeste de déploiement | Spécifiez true pour ajouter les informations d’identification de registre pour l’envoi (push) d’images Docker au manifeste de déploiement. |

   Si vous avez plusieurs registres de conteneurs pour héberger vos images de modules, vous devez dupliquer cette tâche, sélectionner un autre registre de conteneurs et utiliser **Ignorer les modules** dans les paramètres **Avancés** pour ignorer les images qui ne concernent pas ce registre spécifique.

9. Sélectionnez la tâche **Copier des fichiers** pour la modifier. Utilisez cette tâche pour copier des fichiers dans le répertoire intermédiaire des artefacts.

    | Paramètre | Description |
    | --- | --- |
    | Nom complet | Utiliser le nom par défaut ou personnaliser |
    | Dossier source | Dossier contenant les fichiers à copier. |
    | Contenu | Ajoutez deux lignes : `deployment.template.json` et `**/module.json`. Ces deux fichiers servent d’entrées pour générer le manifeste de déploiement IoT Edge. |
    | Dossier cible | Spécifiez la variable `$(Build.ArtifactStagingDirectory)`. Pour en savoir plus sur la description, consultez [Générer des variables](/azure/devops/pipelines/build/variables#build-variables). |

10. Sélectionnez la tâche **Publier des artefacts de build** pour la modifier. Indiquez le chemin du répertoire intermédiaire des artefacts permettant d'accéder à la tâche afin que le chemin puisse être publié dans le pipeline de mise en production.

    | Paramètre | Description |
    | --- | --- |
    | Nom complet | Utilisez le nom par défaut ou personnalisez. |
    | Chemin de publication | Spécifiez la variable `$(Build.ArtifactStagingDirectory)`. Pour plus d’informations, consultez [Variables de build](/azure/devops/pipelines/build/variables#build-variables). |
    | Nom de l’artefact | Utiliser le nom par défaut : **drop** |
    | Emplacement de publication des artefacts | Utilisez l’emplacement par défaut : **Azure Pipelines** |

11. Sélectionnez l’onglet **Déclencheurs**, puis la case à cocher **Activer l'intégration continue**. Assurez-vous que la branche qui contient votre code est incluse.

    ![Activer le déclencheur d’intégration continue](./media/how-to-continuous-integration-continuous-deployment-classic/configure-trigger.png)

12. Sélectionnez **Enregistrer** dans la liste déroulante **Enregistrer et ajouter à la file d’attente**.

Ce pipeline est maintenant configuré pour s’exécuter automatiquement lorsque vous envoyez (push) un nouveau code à votre référentiel. La dernière tâche, publier les artefacts de pipeline, déclenche un pipeline de mise en production. Passez à la section suivante pour générer le pipeline de mise en production.

[!INCLUDE [iot-edge-create-release-pipeline-for-continuous-deployment](../../includes/iot-edge-create-release-pipeline-for-continuous-deployment.md)]

>[!NOTE]
>Si vous souhaitez utiliser des **déploiements en couches** dans votre pipeline, sachez qu’ils ne sont pas encore pris en charge dans les tâches Azure IoT Edge dans Azure DevOps.
>
>Toutefois, vous pouvez utiliser une [tâche Azure CLI dans Azure DevOps](/azure/devops/pipelines/tasks/deploy/azure-cli) pour créer votre déploiement en tant que déploiement en couches. Pour la valeur **Script inline**, vous pouvez utiliser la [commande az iot edge deployment create](/cli/azure/ext/azure-cli-iot-ext/iot/edge/deployment) :
>
>   ```azurecli-interactive
>   az iot edge deployment create -d {deployment_name} -n {hub_name} --content modules_content.json --layered true
>   ```

[!INCLUDE [iot-edge-verify-iot-edge-continuous-integration-continuous-deployment](../../includes/iot-edge-verify-iot-edge-continuous-integration-continuous-deployment.md)]

## <a name="next-steps"></a>Étapes suivantes

* Exemple de bonnes pratiques IoT Edge DevOps dans [Azure DevOps Starter pour IoT Edge](how-to-devops-starter.md)
* Pour plus d’informations sur le déploiement IoT Edge, consultez [Comprendre les déploiements IoT Edge pour les appareils uniques ou à l’échelle](module-deployment-monitoring.md).
* Suivez les étapes pour créer, mettre à jour ou supprimer un déploiement dans [Déployer et surveiller les modules IoT Edge à grande échelle](how-to-deploy-at-scale.md).
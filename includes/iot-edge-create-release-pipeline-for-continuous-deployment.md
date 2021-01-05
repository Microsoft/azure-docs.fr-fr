---
author: v-tcassi
ms.service: iot-edge
ms.topic: include
ms.date: 08/26/2020
ms.author: v-tcassi
ms.openlocfilehash: 706b2306fbe9f2a744d2874a8b55f78fa2fc8e4d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89300624"
---
## <a name="create-a-release-pipeline-for-continuous-deployment"></a>Créer un pipeline de mise en production pour un déploiement continu

Dans cette section, vous créez un pipeline de mise en production configuré pour s’exécuter automatiquement quand votre pipeline de build supprime des artefacts. Ce pipeline de build affichera les journaux d’activité de déploiement dans Azure Pipelines.

Créer un pipeline et ajouter une nouvelle phase :

1. Sous l’onglet **Mises en production** dans **Pipelines**, choisissez **+ Nouveau pipeline**. Sinon, si vous avez déjà des pipelines de mise en production, choisissez le bouton **+ Nouveau**, puis sélectionnez **+ Nouveau pipeline de mise en production**.  

    ![Ajouter un pipeline de mise en production à l’aide du bouton + Nouveau pipeline](./media/iot-edge-create-release-pipeline-for-continuous-deployment/add-release-pipeline.png)

2. Lorsque vous êtes invité à sélectionner un modèle, optez pour une **Tâche vide**.

    ![Démarrer avec un travail vide pour votre pipeline de mise en production](./media/iot-edge-create-release-pipeline-for-continuous-deployment/start-with-empty-release-job.png)

3. Votre nouveau pipeline de mise en production s'initialise avec une seule phase appelée **Phase 1**. Renommez la phase 1 en **développement** et traitez-la comme un pipeline de déploiement continu pour votre environnement de développement. En règle générale, les pipelines de déploiement continu présentent plusieurs phases, notamment le **développement**, la **préproduction** et la **production**. Vous pouvez changer les noms de ces phases ou en créer davantage selon votre pratique DevOps. Fermez la fenêtre de détails de la phase une fois cette dernière renommée.

   Vous pouvez également renommer votre pipeline de mise en production en sélectionnant le texte « Nouveau pipeline de mise en production » en haut.

4. Liez la mise en production aux artefacts de build publiés par le pipeline de build. Cliquez sur **Ajouter** dans la zone des artefacts.

   ![Cliquer sur Ajouter dans la zone des artefacts de l’interface](./media/iot-edge-create-release-pipeline-for-continuous-deployment/add-artifacts.png)

5. Dans la **page Ajouter un artefact**, sélectionnez **Build** comme **Type source**. Choisissez le projet et le pipeline de build que vous avez créés. Si vous le souhaitez, remplacez l’**Alias source** par un nom plus descriptif. Sélectionnez ensuite **Ajouter**.

   ![Dans la page Ajouter un artefact, sélectionner Ajouter pour créer l’artefact](./media/iot-edge-create-release-pipeline-for-continuous-deployment/add-artifact.png)

6. Ouvrez les déclencheurs d’artefact, puis sélectionnez le bouton bascule pour activer le déclencheur de déploiement continu. Désormais, une mise en production est créée chaque fois qu’une nouvelle build est disponible.

   ![Ouvrir les déclencheurs d’artefact et activer le déclencheur de déploiement continu](./media/iot-edge-create-release-pipeline-for-continuous-deployment/add-trigger.png)

7. La phase **dev** est préconfigurée avec un travail, mais aucune tâche. Dans le menu de pipeline, sélectionnez **Tâches**, puis la phase **dev**. Sélectionnez le **Travail de l’agent** et changez son **Nom d’affichage** en **AQ**. Vous pouvez définir les détails du travail de l’agent, mais la tâche de déploiement n’étant pas propre à la plateforme, vous pouvez utiliser n’importe quelle **Spécification de l’agent** dans le **Pool d’agents** choisi.

   ![Examiner les tâches de votre phase de développement sous l’onglet Tâches](./media/iot-edge-create-release-pipeline-for-continuous-deployment/view-stage-tasks.png)

8. Sur le travail AQ, sélectionnez le signe plus ( **+** ) pour ajouter deux tâches. Recherchez et ajoutez **Azure IoT Edge** deux fois.

9. Sélectionnez la première tâche **Azure IoT Edge** et configurez-la avec les valeurs suivantes :

    | Paramètre | Description |
    | --- | --- |
    | Nom complet | Le nom d’affichage est automatiquement mis à jour lorsque le champ Action change. |
    | Action | Sélectionnez `Generate deployment manifest`. |
    | fichier .template.json | Indiquez le chemin : `$(System.DefaultWorkingDirectory)/Drop/drop/deployment.template.json`. Le chemin est publié à partir du pipeline de build. |
    | Plateforme par défaut | Sélectionnez le système d’exploitation approprié pour vos modules en fonction de l’appareil IoT Edge ciblé. |
    | Chemin de sortie| Placez le chemin `$(System.DefaultWorkingDirectory)/Drop/drop/configs/deployment.json`. Ce chemin est le fichier final du manifeste de déploiement IoT Edge. |

    Ces configurations permettent de remplacer les URL des images de module dans le fichier `deployment.template.json`. **Générer le manifeste de déploiement** permet également de remplacer les variables par la valeur exacte que vous avez définie dans le fichier `deployment.template.json`. Dans VS/VS Code, vous spécifiez la valeur réelle dans un fichier `.env`. Dans Azure Pipelines, vous définissez la valeur sous l'onglet **Variables du pipeline de mise en production**. Accédez à l'onglet **Variables** et définissez le nom et la valeur comme suit :

    * **ACR_ADDRESS** : valeur de votre **Serveur de connexion** Azure Container Registry. Vous pouvez récupérer le serveur de connexion à partir de la page Vue d’ensemble de votre registre de conteneurs dans le portail Azure.
    * **ACR_PASSWORD** : votre mot de passe Azure Container Registry.
    * **ACR_USER** : votre nom d'utilisateur Azure Container Registry.

    Si votre projet contient d'autres variables, vous pouvez spécifier le nom et la valeur dans cet onglet. **Générer le manifeste de déploiement** ne reconnaît que les variables de type `${VARIABLE}`. Par conséquent, veillez à utiliser ce type dans vos fichiers `*.template.json`.

    ![Configurer les variables pour votre pipeline de mise en production sous l’onglet Variables](./media/iot-edge-create-release-pipeline-for-continuous-deployment/configure-variables.png)

10. Sélectionnez la deuxième tâche **Azure IoT Edge** et configurez-la avec les valeurs suivantes :

    | Paramètre | Description |
    | --- | --- |
    | Nom complet | Le nom d’affichage est automatiquement mis à jour lorsque le champ Action change. |
    | Action | Sélectionnez `Deploy to IoT Edge devices`. |
    | Fichier de déploiement | Placez le chemin `$(System.DefaultWorkingDirectory)/Drop/drop/configs/deployment.json`. Ce chemin est le fichier du manifeste de déploiement IoT Edge. |
    | Abonnement Azure | Sélectionnez l’abonnement contenant votre IoT Hub.|
    | Nom de l’IoT Hub | Sélectionnez votre hub IoT.|
    | Choisir un ou plusieurs appareils | Indiquez si vous souhaitez que le pipeline de mise en production soit déployé sur un ou plusieurs appareils. Si vous le déployez sur un seul appareil, entrez l'**ID d’appareil IoT Edge**. Si vous le déployez sur plusieurs appareils, spécifiez la **condition cible** des appareils. La condition cible est un filtre qui correspond à un ensemble d'appareils IoT Edge dans IoT Hub. Pour utiliser les étiquettes d’appareils comme condition, vous devez mettre à jour les étiquettes d’appareils correspondantes avec le jumeau d’appareil IoT Hub. Mettez à jour l'**ID de déploiement IoT Edge** et la **priorité de déploiement IoT Edge** dans les paramètres avancés. Pour plus d’informations sur la création d’un déploiement pour plusieurs appareils, consultez [Comprendre les déploiements automatiques IoT Edge](../articles/iot-edge/module-deployment-monitoring.md). |
    | ID d’appareil ou condition cible | Selon la sélection précédente, spécifiez un ID d’appareil ou une [condition cible](../articles/iot-edge/module-deployment-monitoring.md#target-condition) pour le déploiement sur plusieurs appareils. |
    | Avancé | Pour l’ID de déploiement IoT Edge, spécifiez `$(System.TeamProject)-$(Release.EnvironmentName)`. Cette variable mappe le projet et le nom de la version avec votre ID de déploiement IoT Edge. |

    ![Ajouter des tâches Azure IoT Edge pour votre phase de développement](./media/iot-edge-create-release-pipeline-for-continuous-deployment/add-quality-assurance-task.png)

11. Sélectionnez **Enregistrer** pour enregistrer vos modifications dans le nouveau pipeline de mise en production. Pour revenir à la vue du pipeline, sélectionnez l’onglet **Pipeline** dans le menu.

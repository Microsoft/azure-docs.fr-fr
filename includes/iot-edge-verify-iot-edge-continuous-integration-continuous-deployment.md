---
author: v-tcassi
ms.service: iot-edge
ms.topic: include
ms.date: 08/26/2020
ms.author: v-tcassi
ms.openlocfilehash: b5450e4846c3c49c89830ae65c50a95ee0c8d6eb
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/30/2021
ms.locfileid: "104803485"
---
## <a name="verify-iot-edge-cicd-with-the-build-and-release-pipelines"></a>Vérifier les fonctionnalités CI/CD IoT Edge avec les pipelines de build et de mise en production

Pour déclencher un travail de build, envoyez (push) une validation dans le dépôt de code source, ou déclenchez-le manuellement. Dans cette section, vous déclenchez manuellement le pipeline CI/CD pour tester son fonctionnement. Vérifiez ensuite que le déploiement a abouti.

1. Dans le menu du volet de gauche, sélectionnez **Pipelines** et ouvrez le pipeline de build que vous avez créé au début de cet article.

2. Vous pouvez déclencher un travail de build dans votre pipeline de build en sélectionnant le bouton **Exécuter le pipeline** en haut à droite.

    ![Déclencher manuellement votre pipeline de build à l’aide du bouton Exécuter le pipeline](./media/iot-edge-verify-iot-edge-continuous-integration-continuous-deployment/manual-trigger.png)

3. Vérifiez les paramètres **Exécuter le pipeline**. Ensuite, sélectionnez **Exécuter**.

    ![Spécifier les options d’exécution du pipeline et sélectionner Exécuter](./media/iot-edge-verify-iot-edge-continuous-integration-continuous-deployment/run-pipeline-settings.png)

4. Sélectionnez **Travail de l’agent 1** pour observer la progression de l’exécution. Vous pouvez examiner les journaux de la sortie du travail en sélectionnant le travail. 

    ![Examiner la sortie du journal du travail](./media/iot-edge-verify-iot-edge-continuous-integration-continuous-deployment/view-job-run.png)

5. Si le pipeline de build s'achève correctement, il déclenche une mise en production en phase **dev**. La mise en production **dev** réussie crée un déploiement IoT Edge sur les appareils IoT Edge cibles.

    ![Mise en production en phase dev](./media/iot-edge-verify-iot-edge-continuous-integration-continuous-deployment/pending-approval.png)

6. Cliquez sur la phase **dev** pour afficher les journaux de mise en production.

    ![Journaux de mise en production](./media/iot-edge-verify-iot-edge-continuous-integration-continuous-deployment/release-logs.png)

7. Si votre pipeline échoue, commencez par examiner les journaux. Vous pouvez afficher les journaux en accédant au résumé de l’exécution du pipeline et en sélectionnant le travail et la tâche. Si une tâche donnée échoue, vérifiez les journaux de cette tâche. Pour obtenir des instructions détaillées sur la configuration et l’utilisation des journaux, consultez l’article [Consulter les journaux pour diagnostiquer les problèmes de pipeline](/azure/devops/pipelines/troubleshooting/review-logs).

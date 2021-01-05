---
title: Résoudre les problèmes et diagnostiquer des échecs de flux de travail
description: Découvrir comment résoudre les problèmes et diagnostiquer les erreurs et les échecs de flux de travail dans Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: article
ms.date: 01/31/2020
ms.openlocfilehash: 1f83f13564a64a0d9d8a5e0144ca95af6a769d6c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95995044"
---
# <a name="troubleshoot-and-diagnose-workflow-failures-in-azure-logic-apps"></a>Résoudre les problèmes et diagnostiquer les échecs de flux de travail dans Azure Logic Apps

Votre application logique génère des informations qui peuvent vous aider à diagnostiquer et déboguer ses problèmes. Vous pouvez effectuer le diagnostic d’une application logique en examinant chaque étape dans le flux de travail via le portail Azure. Vous pouvez également ajouter des étapes à un flux de travail pour effectuer un débogage du runtime.

<a name="check-trigger-history"></a>

## <a name="check-trigger-history"></a>Vérifier l'historique du déclencheur

Chaque exécution de l’application logique commence par une tentative de déclenchement. Ainsi, si le déclencheur n'est pas activé, procédez comme suit :

1. Vérifiez l’état du déclencheur en [consultant son historique](../logic-apps/monitor-logic-apps.md#review-trigger-history). Pour afficher plus d’informations sur la tentative de déclenchement, sélectionnez cet événement du déclencheur, par exemple :

   ![Afficher l’état et l’historique du déclencheur](./media/logic-apps-diagnosing-failures/logic-app-trigger-history.png)

1. Vérifiez les entrées du déclencheur pour vous assurer qu’elles s’affichent comme prévu. Sous **Liens d'entrée**, sélectionnez le lien qui permet d'afficher le volet **Entrées**.

   Les entrées du déclencheur incluent les données attendues et requises par le déclencheur pour démarrer le flux de travail. L’examen de ces entrées peut vous aider à déterminer si les entrées du déclencheur sont correctes et si la condition a été remplie afin que le flux de travail puisse se poursuivre.

   Par exemple, la propriété `feedUrl` contient une valeur de flux RSS incorrecte :

   ![Examiner les entrées du déclencheur afin d'y détecter des erreurs](./media/logic-apps-diagnosing-failures/review-trigger-inputs-for-errors.png)

1. Vérifiez les sorties du déclencheur, le cas échéant, pour vérifier qu’elles s’affichent comme prévu. Sous **Lien de sorties**, sélectionnez le lien qui permet d'afficher le volet **Sorties**.

   Les sorties du déclencheur incluent les données que le déclencheur transmet à l’étape suivante de votre flux de travail. L’examen de ces sorties peut vous aider à déterminer si les valeurs correctes ou attendues ont été transmises à l’étape suivante de votre flux de travail, par exemple :

   ![Examiner les sorties du déclencheur afin d'y détecter des erreurs](./media/logic-apps-diagnosing-failures/review-trigger-outputs-for-errors.png)

   > [!TIP]
   > Si vous rencontrez un contenu que vous ne reconnaissez pas, apprenez-en davantage sur les [différents types de contenu](../logic-apps/logic-apps-content-type.md) dans Azure Logic Apps.

<a name="check-runs-history"></a>

## <a name="check-runs-history"></a>Vérifier l’historique des exécutions

Chaque fois que le déclencheur est activé pour un élément ou un événement, le moteur Logic Apps crée et exécute une instance de flux de travail distincte pour chaque élément ou événement. Si une exécution échoue, suivez la procédure ci-dessous pour examiner le déroulement de cette exécution, y compris l’état de chaque étape dans le flux de travail, ainsi que les entrées et les sorties pour chacune d’entre elles.

1. Vérifiez l’état d’exécution du flux de travail en [consultant l'historique des exécutions](../logic-apps/monitor-logic-apps.md#review-runs-history). Pour afficher plus d’informations sur une exécution ayant échoué, y compris ses différentes étapes et leur état, sélectionnez cette exécution.

   ![Afficher l’historique des exécutions et sélectionner l’exécution ayant échoué](./media/logic-apps-diagnosing-failures/logic-app-runs-history.png)

1. Lorsque toutes les étapes de l’exécution s’affichent, développez la première étape ayant échoué.

   ![Développer la première étape ayant échoué](./media/logic-apps-diagnosing-failures/logic-app-run-pane.png)

1. Vérifiez les entrées de l’étape ayant échoué pour vous assurer qu’elles s’affichent comme prévu.

1. Passez en revue les détails de chaque étape dans une exécution spécifique. Dans **Historique des exécutions**, sélectionnez l’exécution que vous souhaitez examiner.

   ![Examiner l’historique des exécutions](./media/logic-apps-diagnosing-failures/logic-app-runs-history.png)

   ![Affichage des détails relatifs à une exécution d’application logique](./media/logic-apps-diagnosing-failures/logic-app-run-details.png)

1. Pour examiner les entrées, les sorties et les messages d’erreur pour une étape spécifique, cliquez sur cette étape afin que la zone se développe et affiche les détails. Par exemple :

   ![Affichage des détails de l’étape](./media/logic-apps-diagnosing-failures/logic-app-run-details-expanded.png)

## <a name="perform-runtime-debugging"></a>Effectuer le débogage du runtime

Pour faciliter le débogage, vous pouvez ajouter des étapes de diagnostic à un flux de travail d'application logique, ainsi qu’examiner les historiques des déclencheurs et des exécutions. Par exemple, vous pouvez ajouter des étapes qui utilisent le service [Webhook Tester](https://webhook.site/) afin de pouvoir inspecter les requêtes HTTP et déterminer précisément leur taille, leur forme et leur format.

1. Accédez au site [Webhook Tester](https://webhook.site/) et copiez l’URL unique générée.

1. Dans votre application logique, ajoutez une action HTTP POST ainsi que le contenu du corps que vous souhaitez tester (par exemple, une expression ou une autre sortie de l’étape).

1. Collez votre URL à partir de l'élément Webhook Tester dans l’action HTTP POST.

1. Pour voir comment une requête a été formée lors de sa génération à partir du moteur Logic Apps, exécutez l’application logique, puis revisitez le site Webhook Tester pour plus d’informations.

## <a name="next-steps"></a>Étapes suivantes

* [Analyser votre application logique](../logic-apps/monitor-logic-apps.md)

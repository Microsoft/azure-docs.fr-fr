---
title: Envoyer des alertes à partir d’Azure Application Insights | Microsoft Docs
description: Didacticiel pour envoyer des alertes en réponse aux erreurs dans votre application à l’aide d’Azure Application Insights.
ms.subservice: application-insights
ms.topic: tutorial
author: mrbullwinkle
ms.author: mbullwin
ms.date: 04/10/2019
ms.custom: mvc
ms.openlocfilehash: c1aa3ae5be3e9f3fb50909cb5311d167bbeee647
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96904225"
---
# <a name="monitor-and-alert-on-application-health-with-azure-application-insights"></a>Surveiller et alerter sur l’intégrité des applications avec Azure Application Insights

Azure Application Insights vous permet de surveiller votre application et d’envoyer des alertes lorsqu’elle est indisponible, connaît des défaillances ou rencontre des problèmes de performances.  Ce tutoriel vous guide dans le processus de création de tests permettant de vérifier en permanence la disponibilité de votre application.

Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Créer des tests de disponibilité pour vérifier en permanence la réponse de l’application
> * Envoyer des messages aux administrateurs lorsqu’un problème se produit

## <a name="prerequisites"></a>Conditions préalables requises

Pour suivre ce tutoriel :

Créez une [ressource Application Insights](../app/create-new-resource.md).

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).

## <a name="create-availability-test"></a>Créer un test de disponibilité

Les tests de disponibilité dans Application Insights vous permettent de tester automatiquement votre application depuis différents lieux dans le monde.   Dans ce tutoriel, vous allez effectuer un test d’URL pour vous assurer que votre application web est disponible.  Vous pouvez également créer une procédure pas à pas complète pour tester en détail son fonctionnement. 

1. Sélectionnez **Application Insights** choisissez votre abonnement.  

2. Sélectionnez **Disponibilité** sous le menu **Examiner**, puis cliquez sur **Créer un test**.

    ![Ajouter un test de disponibilité](media/tutorial-alert/add-test-001.png)

3. Nommez le test et conservez les valeurs par défaut.  Cette sélection déclenche des demandes pour l’URL de l’application toutes les 5 minutes à partir de cinq emplacements géographiques différents.

4. Sélectionnez **Alertes** pour ouvrir la liste déroulante **Alertes** dans laquelle vous pouvez définir des détails sur la réponse à envoyer si le test échoue. Choisissez **en temps quasi réel** et définissez l’état sur **Activé.**

    Entrez l’adresse e-mail à contacter lorsque les critères d’alerte sont remplis.  Vous pouvez entrer l’adresse d’un webhook à appeler lorsque les critères d’alerte sont remplis.

    ![Créer un test](media/tutorial-alert/create-test-001.png)

5. Revenez au volet de test, sélectionnez les points de suspension et modifiez l’alerte pour entrer la configuration de votre alerte en temps quasi réel.

    ![Modifiez l'alerte](media/tutorial-alert/edit-alert-001.png)

6. Définissez une valeur égale ou supérieure à 3 pour les emplacements incorrects. Créez un [groupe d’actions](../platform/action-groups.md) pour configurer les personnes à informer si votre seuil d’alerte est dépassé.

    ![Enregistrer l'interface utilisateur d’alerte](media/tutorial-alert/save-alert-001.png)

7. Une fois votre alerte configurée, cliquez sur le nom du test pour afficher les détails de chaque emplacement. Les tests peuvent être affichés dans les deux formats graphique linéaire et nuage de points afin de visualiser le succès/les échecs pour un intervalle de temps donné.

    ![Détails du test](media/tutorial-alert/test-details-001.png)

8. Vous pouvez explorer les détails d’un test en cliquant sur son point dans le graphique en nuage de points. Cette action lance la vue des détails de la transaction de bout en bout. L’exemple ci-dessous montre les détails d’une requête ayant échoué.

    ![Résultat de test](media/tutorial-alert/test-result-001.png)
  
## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez appris à créer des alertes en cas de problèmes, passez à l’étape suivante du didacticiel pour apprendre à analyser la façon dont les utilisateurs interagissent avec votre application.

> [!div class="nextstepaction"]
> [Comprendre les utilisateurs](./tutorial-users.md)


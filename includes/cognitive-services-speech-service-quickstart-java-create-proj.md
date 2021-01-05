---
author: erhopf
ms.service: cognitive-services
ms.topic: include
ms.date: 10/15/2020
ms.author: erhopf
ms.openlocfilehash: ee88a7cc187644c89aca5656df9ab9ae48a5a056
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96187678"
---
1. Démarrez Eclipse.

1. Dans le champ **Workspace** (Espace de travail) de l’utilitaire Eclipse Launcher, entrez le nom d’un nouveau répertoire d’espace de travail. Sélectionnez ensuite **Launch** (Lancer).

   ![Capture d’écran d’Eclipse Launcher](../articles/cognitive-services/Speech-Service/media/sdk/qs-java-jre-01-create-new-eclipse-workspace.png)

1. La fenêtre principale de l’IDE Eclipse apparaît au bout d’un instant. Fermez l’écran d’**accueil** s’il en existe un.

1. Dans la barre de menus Eclipse, créez un projet en choisissant **File (Fichier)**  > **New (Nouveau)**  > **Project (Projet)** .

1. La boîte de dialogue **Nouveau projet** apparaît. Sélectionnez **Java Project** (Projet Java), puis **Next** (Suivant).

   ![Capture d’écran de la boîte de dialogue du nouveau projet, avec le projet Java sélectionné](../articles/cognitive-services/Speech-Service/media/sdk/qs-java-jre-02-select-wizard.png)

1. L’Assistant **New Java Project** (Nouveau projet Java) démarre. Dans le champ **Project name** (Nom de projet), entrez **quickstart** (démarrage rapide) et choisissez **JavaSE-1.8** en tant qu’environnement d’exécution. Sélectionnez **Terminer**.

   ![Capture d’écran de l’Assistant New Java Project (Nouveau projet Java)](../articles/cognitive-services/Speech-Service/media/sdk/qs-java-jre-03-create-java-project.png)

1. Si une fenêtre **Open Associated Perspective?** (Ouvrir la perspective associée ?) s’affiche, sélectionnez **Open Perspective** (Ouvrir la perspective).

1. Dans **Package explorer** (l’Explorateur de package), cliquez avec le bouton droit sur le projet **quickstart** (démarrage rapide). Choisissez **Configure (Configurer)**  > **Convert to Maven Project (Convertir en projet Maven)** dans le menu contextuel.

   ![Capture d’écran de l’explorateur de package](../articles/cognitive-services/Speech-Service/media/sdk/qs-java-jre-04-convert-to-maven-project.png)

1. La fenêtre **Create new POM** (Créer un POM) s’affiche. Dans le champ **ID de groupe**, entrez *com.microsoft.cognitiveservices.speech.samples* et, dans le champ **ID d’artefact**, indiquez *démarrage rapide*. Sélectionnez **Terminer**.

   ![Capture d’écran de la fenêtre de création d’un POM](../articles/cognitive-services/Speech-Service/media/sdk/qs-java-jre-05-configure-maven-pom.png)

1. Ouvrez le fichier *pom.xml* et modifiez-le.

   * À la fin du fichier, avant la balise de fermeture `</project>`, créez un élément `repositories` avec une référence au référentiel Maven pour le Kit de développement logiciel (SDK) Speech, comme indiqué ici :

     [!code-xml[POM Repositories](~/samples-cognitive-services-speech-sdk/quickstart/java/jre/from-microphone/pom.xml#repositories)]

   * Ajoutez également un élément `dependencies`, avec le SDK Speech version 1.13.0 comme dépendance :

     [!code-xml[POM Dependencies](~/samples-cognitive-services-speech-sdk/quickstart/java/jre/from-microphone/pom.xml#dependencies)]

   * Enregistrez les modifications.

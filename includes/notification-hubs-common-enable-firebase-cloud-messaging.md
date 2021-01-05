---
author: mikeparker104
ms.author: miparker
ms.date: 06/02/2020
ms.service: notification-hubs
ms.topic: include
ms.openlocfilehash: 71e4b0b3b7bf84938123acf70ac18a93c3015a4d
ms.sourcegitcommit: 971a3a63cf7da95f19808964ea9a2ccb60990f64
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/19/2020
ms.locfileid: "85095315"
---
### <a name="create-a-firebase-project-and-enable-firebase-cloud-messaging-for-android"></a>Créer un projet Firebase et activer Firebase Cloud Messaging pour Android

1. Connectez-vous à la [console Firebase](https://firebase.google.com/console/). Créez un projet Firebase en entrant **PushDemo** comme **nom du projet**.

    > [!NOTE]
    > Un nom unique sera généré pour vous. Par défaut, il s’agit d’une variante en minuscules du nom que vous avez fourni plus un nombre généré, séparés par un tiret. Vous pouvez le changer si vous voulez, du moment qu’il reste unique au niveau global.

1. Une fois le projet créé, sélectionnez **Add Firebase to your Android app** (Ajouter Firebase à votre application Android).

    ![Ajouter Firebase à votre application Android](./media/notification-hubs-common-enable-firebase-cloud-messaging/notification-hubs-add-firebase-to-android-app.png)

1. Dans la page **Add Firebase to your Android app** (Ajouter Firebase à votre application Android), effectuez les étapes suivantes :
    1. Pour **Android package name** (Nom du package Android), entrez un nom pour votre package. Par exemple : `com.<organization_identifier>.<package_name>`.

        ![Spécifiez le nom du package](./media/notification-hubs-common-enable-firebase-cloud-messaging/specify-package-name-fcm-settings.png)
    1. Sélectionnez **Inscrire une application**.  
    1. Sélectionnez **Télécharger le fichier google-services.json**. Ensuite, enregistrez le fichier dans un dossier local pour l’utiliser ultérieurement, puis sélectionnez **Suivant**.  

        ![Téléchargez le fichier google-services.json](./media/notification-hubs-common-enable-firebase-cloud-messaging/download-google-service-button.png)
    1. Sélectionnez **Suivant**.
    1. Sélectionnez **Continue to console** (Poursuivre dans la console).

        > [!NOTE]
        > Si le bouton **Continue to console** n’est pas activé, en raison de la *vérification de l’installation*, choisissez **Skip this step** (Ignorer cette étape).

1. Dans la console Firebase, sélectionnez la roue dentée associée à votre projet. Ensuite, sélectionnez **Project Settings** (Paramètres du projet).

    ![Sélectionnez Project Settings (Paramètres du projet).](./media/notification-hubs-common-enable-firebase-cloud-messaging/notification-hubs-firebase-console-project-settings.png)

    > [!NOTE]
    > Si vous n’avez pas téléchargé le fichier **google-services.JSON**, vous pouvez le faire sur cette page.

1. Basculez vers l’onglet **Messagerie cloud** en haut. Copiez et enregistrez la **clé du serveur** pour une utilisation ultérieure. Cette valeur est utilisée pour configurer votre hub de notification.

    ![Copie de la clé serveur](./media/notification-hubs-common-enable-firebase-cloud-messaging/server-key.png)

---
title: Fichier Include
description: Fichier Include
services: notification-hubs
author: spelluru
ms.service: notification-hubs
ms.topic: include
ms.date: 08/01/2019
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: 45bdd569741dc13181bcaf9e8587a02b3d02c621
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "68728863"
---
1. Connectez-vous à la [console Firebase](https://firebase.google.com/console/). Créer un nouveau projet Firebase si vous n’en avez pas encore.
2. Une fois le projet créé, sélectionnez **Add Firebase to your Android app** (Ajouter Firebase à votre application Android). 

    ![Ajouter Firebase à votre application Android](./media/notification-hubs-enable-firebase-cloud-messaging/notification-hubs-add-firebase-to-android-app.png)

3. Dans la page **Add Firebase to your Android app** (Ajouter Firebase à votre application Android), procédez comme suit : 
    1. Pour **Android package name** (Nom du package Android), entrez un nom pour votre package. Par exemple : `tutorials.tutoria1.xamarinfcmapp`. 

        ![Spécifiez le nom du package](./media/notification-hubs-enable-firebase-cloud-messaging/specify-package-name-fcm-settings.png)
    2. Sélectionnez **Inscrire une application**.  
    1. Sélectionnez **Télécharger le fichier google-services.json**. Puis, enregistrez le fichier dans le dossier de votre projet et sélectionnez **Suivant**. Si vous n’avez pas encore créé le projet Visual Studio, vous pouvez effectuer cette étape après avoir créé le projet. 

        ![Téléchargez le fichier google-services.json](./media/notification-hubs-enable-firebase-cloud-messaging/download-google-service-button.png)
    6. Sélectionnez **Suivant**. 
    7. Sélectionnez **Ignorer cette étape**. 

        ![Ignorer la dernière étape](./media/notification-hubs-enable-firebase-cloud-messaging/skip-this-step.png)
8. Dans la console Firebase, sélectionnez la roue dentée associée à votre projet. Ensuite, sélectionnez **Project Settings** (Paramètres du projet).

    ![Sélectionnez Project Settings (Paramètres du projet).](./media/notification-hubs-enable-firebase-cloud-messaging/notification-hubs-firebase-console-project-settings.png)
4. Si vous n’avez pas téléchargé le fichier **google-services.JSON**, vous pouvez le faire sur cette page. 

    ![Téléchargez le fichier google-services.json à partir de l’onglet Général.](./media/notification-hubs-enable-firebase-cloud-messaging/download-google-services-json-general-page.png)
1. Basculez vers l’onglet **Messagerie cloud** en haut. Copiez et enregistrez la **clé du serveur** pour une utilisation ultérieure. Cette valeur est utilisée pour configurer votre hub de notification.

    ![Copie de la clé serveur](./media/notification-hubs-enable-firebase-cloud-messaging/server-key.png)

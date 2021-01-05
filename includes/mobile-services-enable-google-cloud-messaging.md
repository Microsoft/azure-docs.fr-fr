---
title: Fichier Include
description: Fichier Include
services: notification-hubs
author: spelluru
ms.service: notification-hubs
ms.topic: include
ms.date: 04/06/2018
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: b32ff0926aa2b797bdb8dfcbb01cf845080a0ce9
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93375870"
---
1. Accédez à la [Console Google Cloud](https://console.developers.google.com/cloud-resource-manager)et connectez-vous avec les informations d’identification de votre compte Google. 
2. Sélectionnez **Create Project** dans la barre d’outils. 
   
    ![Création d’un projet](./media/mobile-services-enable-google-cloud-messaging/mobile-services-google-new-project.png)   
3. Pour **Nom de projet** , saisissez un nom pour votre projet, et cliquez sur **Créer**.
4. Sélectionnez le bouton **Alertes** sur la barre d’outils, puis votre projet dans la liste. Le tableau de bord s’affiche pour votre projet. Vous pouvez aussi accéder directement au tableau de bord à l’aide de l’URL suivante : `https://console.developers.google.com/home/dashboard?project=<YOUR PROJECT NAME>`

    ![Sélectionner votre projet dans Alertes](./media/mobile-services-enable-google-cloud-messaging/alert-new-project.png)
5. Notez le **Numéro de projet** situé dans la vignette **Informations du projet** sur le tableau de bord. 

    ![ID de projet](./media/mobile-services-enable-google-cloud-messaging/project-number.png)
6. Dans le tableau de bord, sur la vignette **API** , sélectionnez **Accéder à la présentation des API**. 

    ![Lien Présentation de l’API](./media/mobile-services-enable-google-cloud-messaging/go-to-api-overview.png)
7. Dans la page **API** , sélectionnez **ENABLE APIS AND SERVICES** (ACTIVER LES API ET LES SERVICES). 

    ![Bouton Enable APIs and Services (Activer les API et les services)](./media/mobile-services-enable-google-cloud-messaging/enable-api-services-button.png)
8. Recherchez **Google Cloud Messaging** et sélectionnez-le. 

    ![Rechercher Google Cloud Messaging et le sélectionner](./media/mobile-services-enable-google-cloud-messaging/search-select-gcm.png)
9. Pour activer Google Cloud Messaging pour le projet, sélectionnez **ENABLE** (ACTIVER).

    ![Activation de Google Cloud Messaging](./media/mobile-services-enable-google-cloud-messaging/enable-gcm-button.png)
10. Sélectionnez **Create credentials** (Créer des informations d’identification) sur la barre d’outils. 

    ![Bouton Create credentials (Créer des informations d’identification)](./media/mobile-services-enable-google-cloud-messaging/create-credentials-button.png)
11. Sur la page **Add credentials to your project** (Ajouter des informations d’identification à votre projet), sélectionnez le lien **API key** (Clé API). 

    ![Ajouter les informations d’identification](./media/mobile-services-enable-google-cloud-messaging/api-key-button.png)    
12. Sur la page **API key** (Clé API), sélectionnez **Create/Save** (Créer/Enregistrer). Dans l’exemple suivant, l’option **Adresses IP** est sélectionnée, et **0.0.0.0/0** est saisi pour les adresses IP autorisées. Vous devez limiter votre clé API de façon appropriée. 

    ![Bouton API Key - Create (Clé API - Créer)](./media/mobile-services-enable-google-cloud-messaging/api-key-create-button.png)
13. Copiez la **Clé API** dans le presse-papiers et enregistrez-le. 

    ![Copier la clé API](./media/mobile-services-enable-google-cloud-messaging/copy-api-key.png)
   
    Cette valeur de clé API vous servira à activer Azure pour l’authentification avec GCM et l’envoi des notifications Push au nom de votre application. Pour accéder de nouveau au tableau de bord du projet, accédez à l’URL suivante : `https://console.developers.google.com/home/dashboard?project=<YOUR PROJECT NAME>`


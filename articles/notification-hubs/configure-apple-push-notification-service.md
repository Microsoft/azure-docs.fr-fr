---
title: Configurer Apple Push Notification Service dans Azure Notification Hubs | Microsoft Docs
description: Découvrez comment configurer un concentrateur de notifications Aure avec les paramètres d’Apple Push Notification Service.
services: notification-hubs
author: sethmanheim
manager: femila
ms.service: notification-hubs
ms.workload: mobile
ms.topic: article
ms.date: 06/22/2020
ms.author: sethm
ms.reviewer: thsomasu
ms.lastreviewed: 03/25/2019
ms.openlocfilehash: 63c7e0c9569428b55420911f253deee52ce440cb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85255396"
---
# <a name="configure-apple-push-notification-service-settings-for-a-notification-hub-in-the-azure-portal"></a>Configurer les paramètres d’Apple Push Notification Service pour un hub de notifications dans le portail Azure

Cet article explique comment configurer les paramètres d’Apple Push Notification Service pour un hub de notifications Azure à l’aide du portail Azure.

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas déjà créé de hub de notification, créez-en un maintenant. Pour plus d’informations, consultez [Créer un hub de notification Azure dans le portail Azure](create-notification-hub-portal.md).

## <a name="configure-apple-push-notification-service"></a>Configurer Apple Push Notification Service

La procédure qui suit vous guide dans la configuration des paramètres d’Apple Push Notification Service pour un hub de notifications :

1. Dans le portail Azure, dans la page **Hub de notification**, sélectionnez **Apple (APNS)** dans le menu de gauche.

1. Pour **Mode d’authentification**, sélectionnez **Certificat** ou **Jeton**.

   - Si vous sélectionnez **Certificat** :
      - Sélectionnez l’icône de fichier, puis sélectionnez le fichier *.p12* que vous voulez charger.
      - Entrez un mot de passe.
      - Sélectionnez le mode **Bac à sable**. Ou bien, pour envoyer des notifications Push aux utilisateurs qui ont acheté votre application sur le Store, sélectionnez le mode **Production**.

     ![Capture d’écran de la configuration d’un certificat APNS sur le portail Azure](./media/configure-apple-push-notification-service/notification-hubs-apple-config-cert.png)

   - Si vous sélectionnez **Jeton** :
      - Entrez les valeurs pour **ID de clé**, **ID d’offre groupée**, **ID de l’équipe** et **Jeton**.
      - Sélectionnez le mode **Bac à sable**. Ou bien, pour envoyer des notifications Push aux utilisateurs qui ont acheté votre application sur le Store, sélectionnez le mode **Production**.

     ![Capture d’écran de la configuration d’un jeton APNS sur le portail Azure](./media/configure-apple-push-notification-service/notification-hubs-apple-config-token.png)

## <a name="next-steps"></a>Étapes suivantes

Pour consulter un didacticiel proposant des instructions étape par étape pour transmettre les notifications vers les périphériques iOS, veuillez consulter l’article suivant : [Envoyer des notifications Push à des applications iOS avec Azure Notification Hubs](ios-sdk-get-started.md).

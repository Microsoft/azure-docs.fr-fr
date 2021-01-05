---
title: Configurer le service de notifications Push Windows dans Azure Notification Hubs | Microsoft Docs
description: Découvrez comment configurer les paramètres du service de notifications Push Windows pour un hub de notifications Azure.
services: notification-hubs
author: sethmanheim
manager: femila
ms.service: notification-hubs
ms.workload: mobile
ms.topic: article
ms.date: 08/04/2020
ms.author: sethm
ms.reviewer: thsomasu
ms.lastreviewed: 03/25/2019
ms.openlocfilehash: 62d73a23fff3bd15d873a579d451b2f91509b7fe
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87758728"
---
# <a name="configure-windows-push-notification-service-settings-in-the-azure-portal"></a>Configurer les paramètres du service de notifications Push Windows dans le portail Azure

Cet article explique comment configurer les paramètres du service de notifications Windows (WNS) pour un hub de notifications Azure à l’aide du portail Azure.  

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas déjà créé de hub de notification, créez-en un maintenant. Pour plus d’informations, consultez [Créer un hub de notification Azure dans le portail Azure](create-notification-hub-portal.md).

## <a name="configure-windows-push-notification-service-wns"></a>Configurer les services de notifications Push Windows (WNS)

La procédure qui suit vous guide dans la configuration des paramètres du service de notifications Windows (WNS) pour un hub de notifications :

1. Dans la page **Hub de notification** du portail Azure, sélectionnez **Windows (WNS)** dans le menu de gauche.
2. Entrez des valeurs pour **SID de package** et **Clé de sécurité**.
3. Sélectionnez **Enregistrer**.

   ![Capture d’écran montrant les zones SID du Package et Clé de sécurité](./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-configure-wns.png)

## <a name="next-steps"></a>Étapes suivantes

Pour consulter un didacticiel avec des instructions pas à pas pour envoyer des notifications vers la plateforme Windows universelle en utilisant Microsoft Azure Notification Hubs et le service de notifications Push Windows (WNS), consultez [Envoyer des notifications vers des applications UWP avec Azure Notification Hubs](notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md).

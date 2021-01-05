---
title: Fichier include
description: Fichier include
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: include
ms.date: 03/05/2019
ms.author: robinsh
ms.custom: include file
ms.openlocfilehash: 552a40be0c069d1002ebc7ea4dafe0d6f93a5755
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85128145"
---
Le [routage des messages](../articles/iot-hub/iot-hub-devguide-messages-d2c.md) vous permet d’envoyer les données de télémétrie de vos appareils IoT à des points de terminaison intégrés compatibles avec les hubs d’événements ou à des points de terminaison personnalisés, comme un stockage d’objets blob, des files d’attente Service Bus, des rubriques Service Bus et des hubs d’événements. Pour configurer le routage des messages, vous devez créer des [requêtes de routage](../articles/iot-hub/iot-hub-devguide-routing-query-syntax.md) pour personnaliser la route qui correspond à une certaine condition. Une fois la configuration effectuée, les données entrantes sont automatiquement acheminées vers les points de terminaison par l’IoT Hub. Si un message ne correspond à aucune des requêtes de routage définies, il est routé vers le point de terminaison par défaut.

Dans ce tutoriel en 2 parties, vous allez à apprendre à configurer et à utiliser ces requêtes de routage personnalisées avec IoT Hub. Vous allez router des messages d’un appareil IoT vers un ou plusieurs points de terminaison, notamment un espace de stockage d’objets blob et une file d’attente Service Bus. Les messages à destination de la file d’attente Service Bus sont récupérés par une application logique et envoyés par e-mail. Les messages pour lesquels vous n’avez pas défini de routage de messages personnalisé sont envoyés au point de terminaison par défaut pour être ensuite récupérés par Azure Stream Analytics puis affichés dans une visualisation Power BI.

Pour effectuer les parties 1 et 2 de ce tutoriel, exécutez les tâches suivantes :

**Partie I : Créer des ressources, configurer le routage des messages**
> [!div class="checklist"]
> * Créez les ressources : un hub IoT, un compte de stockage, une file d’attente Service Bus et un appareil simulé. Pour cela, vous pouvez utiliser le portail Azure, un modèle Azure Resource Manager, l’interface Azure CLI ou Azure PowerShell.
> * Configurez les points de terminaison et les routes des messages dans IoT Hub pour le compte de stockage et la file d’attente Service Bus.

**Partie II : Envoyer les messages au hub, visualiser les résultats routés**
> [!div class="checklist"]
> * Créer une application logique qui est déclenchée et qui envoie un e-mail lors de l’ajout d’un message à la file d’attente Service Bus.
> * Télécharger et exécuter une application qui simule l’envoi de messages par un appareil IoT au Hub pour les différentes options de routage.
> * Créer une visualisation Power BI pour les données envoyées au point de terminaison par défaut.
> * Afficher les résultats...
> * ... dans la file d’attente Service Bus et les e-mails.
> * ... dans le compte de stockage.
> * ... dans la visualisation Power BI.

## <a name="prerequisites"></a>Prérequis

* Pour la partie 1 de ce tutoriel :
  - Vous devez avoir un abonnement Azure. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

* Pour la partie 2 de ce tutoriel :
  - Vous devez avoir effectué la partie 1 de ce tutoriel et toujours disposer des ressources.
  - Installer [Visual Studio](https://www.visualstudio.com/).
  - Avoir accès à un compte Power BI pour examiner l’analytique du flux du point de terminaison par défaut. ([Essayez Power BI gratuitement](https://app.powerbi.com/signupredirect?pbi_source=web)).
  - Disposer d’un compte professionnel ou scolaire pour envoyer des e-mails de notification.
  - Vérifiez que le port 8883 est ouvert dans votre pare-feu. L’exemple dans ce tutoriel utilise le protocole MQTT, lequel communique sur le port 8883. Ce port peut être bloqué dans certains environnements réseau professionnels et scolaires. Pour plus d’informations sur les façons de contourner ce problème, consultez [Connexion à IoT Hub (MQTT)](../articles/iot-hub/iot-hub-mqtt-support.md#connecting-to-iot-hub).

[!INCLUDE [cloud-shell-try-it.md](cloud-shell-try-it.md)]

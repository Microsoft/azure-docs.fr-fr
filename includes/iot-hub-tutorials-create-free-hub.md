---
title: Fichier include
description: Fichier include
services: iot-hub
author: dominicbetts
ms.service: iot-hub
ms.topic: include
ms.date: 04/19/2018
ms.author: dobett
ms.custom: include file
ms.openlocfilehash: 367a0b1d17f8d5ebe4f46835ace963b00e75354e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "68229298"
---
Pour créer un hub IoT à l’aide du portail Azure :

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Sélectionnez **Créer une ressource** > **Internet des objets** > **IoT Hub**.

    ![Sélectionnez cette option pour installer IoT Hub](media/iot-hub-tutorials-create-free-hub/selectiothub.png)

1. Pour créer votre hub IoT de niveau gratuit, utilisez les valeurs des tableaux suivants :

    | Paramètre | Valeur |
    | ------- | ----- |
    | Abonnement | Sélectionnez votre abonnement Azure dans la liste déroulante. |
    | Resource group | Créer. Ce didacticiel utilise le nom **tutorials-iot-hub-rg**. |
    | Région | Ce didacticiel utilise **USA Ouest**. Vous pouvez choisir la région la plus proche. |
    | Nom | La capture d’écran suivante utilise le nom **tutorials-iot-hub**. Vous devez choisir votre propre nom unique quand vous créez ce hub. |

    ![Paramètres du hub 1](media/iot-hub-tutorials-create-free-hub/hubdefinition-1.png)

    | Paramètre | Valeur |
    | ------- | ----- |
    | Tarification et niveau de mise à l’échelle | F1 gratuit. Vous ne pouvez avoir qu’un seul hub de niveau gratuit dans un abonnement. |
    | Unités IoT Hub | 1 |

    ![Paramètres du hub 2](media/iot-hub-tutorials-create-free-hub/hubdefinition-2.png)

1. Cliquez sur **Créer**. La création du hub peut prendre plusieurs minutes.

    ![Paramètres du hub 3](media/iot-hub-tutorials-create-free-hub/hubdefinition-3.png)

1. Notez le nom de hub IoT choisi. Vous utiliserez cette valeur plus loin dans le didacticiel.

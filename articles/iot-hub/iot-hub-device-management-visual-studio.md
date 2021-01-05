---
title: Gestion des appareils Azure IoT avec Visual Studio Cloud Explorer
description: Utilisez Cloud Explorer pour Visual Studio pour la gestion des appareils Azure IoT Hub, avec les méthodes directes et les options de gestion des propriétés souhaitées du jumeau.
author: shizn
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/20/2019
ms.author: xshi
ms.openlocfilehash: 6fe5a45dda6632c56b3c6714827950e25e7d26af
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "73953177"
---
# <a name="use-cloud-explorer-for-visual-studio-for-azure-iot-hub-device-management"></a>Utiliser Cloud Explorer pour Visual Studio pour la gestion des appareils Azure IoT Hub

![Diagramme de bout en bout](media/iot-hub-device-management-visual-studio/iot-e2e-simple.png)

[Cloud Explorer](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.CloudExplorerForVS) est une extension Visual Studio utile qui vous permet de visualiser vos ressources Azure, d’inspecter leurs propriétés et d’exécuter des actions de développeur essentielles à partir de Visual Studio. Il est fourni avec des options de gestion que vous pouvez utiliser pour effectuer diverses tâches.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

| Option de gestion          | Tâche                    |
|----------------------------|--------------------------------|
| Méthodes directes             | Faites agir un appareil comme commençant/arrêtant d’envoyer des messages ou comme redémarrant l’appareil.                                        |
| Lire le jumeau d’appareil           | Obtenez l’état signalé d’un appareil. Par exemple, l’appareil signale que le voyant clignote maintenant.                                    |
| Mettre à jour le jumeau d’appareil         | Mettez un appareil dans certains états, par exemple en réglant un voyant sur le vert ou en définissant l’intervalle d’envoi de télémétrie sur 30 minutes.         |
| Messages Cloud vers appareil   | Envoyez des notifications à un appareil. Par exemple, « Forte probabilité de pluie aujourd'hui. N’oubliez pas de prendre un parapluie avec vous. »              |

Pour plus d’explications sur les différences et des conseils sur l’utilisation de ces options, consultez [l’aide sur la communication appareil-à-cloud](iot-hub-devguide-d2c-guidance.md) et [l’aide sur la communication cloud-à-appareil](iot-hub-devguide-c2d-guidance.md).

Les jumeaux d’appareil sont des documents JSON qui stockent des informations sur l’état des appareils (métadonnées, configurations et conditions). IoT Hub conserve une représentation d’appareil pour chaque appareil que vous y connectez. Pour plus d’informations sur les représentations d’appareil, consultez [Prise en main des représentations d’appareils](iot-hub-node-node-twin-getstarted.md).

## <a name="what-you-learn"></a>Contenu

Dans cet article, vous allez apprendre comment utiliser Cloud Explorer pour Visual Studio avec diverses options de gestion sur votre ordinateur de développement.

## <a name="what-you-do"></a>Procédure

Dans cet article, exécutez Cloud Explorer pour Visual Studio avec diverses options de gestion.

## <a name="what-you-need"></a>Ce dont vous avez besoin

Vous devez respecter les prérequis suivants :

- Un abonnement Azure actif.

- Un hub IoT Azure associé à votre abonnement

- Microsoft Visual Studio 2017 Update 9 ou version ultérieure. Cet article utilise [Visual Studio 2017 ou Visual Studio 2019](https://www.visualstudio.com/vs/).

- Composant Cloud Explorer de Visual Studio Installer, qui est sélectionné par défaut avec la charge de travail Azure.

## <a name="update-cloud-explorer-to-latest-version"></a>Mettre à jour Cloud Explorer vers la version la plus récente

Le composant Cloud Explorer de Visual Studio Installer pour Visual Studio 2017 prend uniquement en charge la supervision des messages appareil-à-cloud et cloud-à-appareil. Pour utiliser Visual Studio 2017, téléchargez et installez la dernière version de [Cloud Explorer](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.CloudExplorerForVS).

## <a name="sign-in-to-access-your-hub"></a>Se connecter pour accéder à votre hub

1. Dans Visual Studio, sélectionnez **Afficher** > **Cloud Explorer** pour ouvrir Cloud Explorer.

1. Sélectionnez l’icône Gestion de compte pour afficher vos abonnements.

    ![Icône Gestion de compte](media/iot-hub-visual-studio-cloud-device-messaging/account-management-icon.png)

1. Si vous êtes connecté à Azure, vos comptes apparaissent. Pour vous connecter à Azure pour la première fois, choisissez **Ajouter un compte**.

1. Sélectionnez les abonnements Azure que vous souhaitez utiliser, puis choisissez **Appliquer**.

1. Développez votre abonnement, puis **IoT Hubs**.  Sous chaque hub, vous pouvez voir vos appareils correspondants. Cliquez avec le bouton droit sur un appareil pour accéder aux options de gestion.

    ![Options de gestion](media/iot-hub-device-management-visual-studio/management-options-vs2019.png)

## <a name="direct-methods"></a>Méthodes directes

Pour utiliser des méthodes directes, effectuez les étapes suivantes :

1. Cliquez avec le bouton droit sur votre appareil et sélectionnez **Appeler une méthode d’appareil directe**.

1. Entrez le nom de la méthode et la charge utile dans **Appeler une méthode directe**, puis sélectionnez **OK**.

    Les résultats s’affichent dans **Sortie**.

## <a name="update-device-twin"></a>Mettre à jour le jumeau d’appareil

Pour modifier un jumeau d’appareil, effectuez les étapes suivantes :

1. Cliquez avec le bouton droit sur votre appareil et sélectionnez **Modifier le jumeau d’appareil**.

   Un fichier **azure-iot-device-twin.json** s’ouvre avec le contenu du jumeau d’appareil.

1. Apportez quelques modifications aux champs **tags** ou **properties.desired** du fichier **azure-iot-device-twin.json**.

1. Appuyez sur **Ctrl+S** pour mettre à jour le jumeau d’appareil.

   Les résultats s’affichent dans **Sortie**.

## <a name="send-cloud-to-device-messages"></a>Envoi de messages cloud vers appareil

Pour envoyer un message de votre hub IoT vers votre appareil, effectuez les étapes suivantes :

1. Cliquez avec le bouton droit sur votre appareil et sélectionnez **Envoyer un message cloud-à-appareil**.

1. Entrez le message dans **Envoyer un message cloud-à-appareil**, puis sélectionnez **OK**.

   Les résultats s’affichent dans **Sortie**.

## <a name="next-steps"></a>Étapes suivantes

Vous avez appris comment utiliser Cloud Explorer pour Visual Studio avec diverses options de gestion.

[!INCLUDE [iot-hub-get-started-next-steps](../../includes/iot-hub-get-started-next-steps.md)]

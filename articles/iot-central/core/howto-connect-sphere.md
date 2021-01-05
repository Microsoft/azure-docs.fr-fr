---
title: Connecter un appareil Azure Sphere dans Azure IoT Central | Microsoft Docs
description: Découvrez comment connecter un appareil Azure Sphere (DevKit) à une application Azure IoT Central.
services: iot-central
ms.service: iot-central
ms.topic: how-to
ms.author: sandeepu
author: sandeeppujar
ms.date: 04/30/2020
ms.custom: device-developer
ms.openlocfilehash: 770f6e56a669ab2d9b425a7a2879eeef5d37377b
ms.sourcegitcommit: 7dacbf3b9ae0652931762bd5c8192a1a3989e701
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92123421"
---
# <a name="connect-an-azure-sphere-device-to-your-azure-iot-central-application"></a>Connecter un appareil Azure Sphere à votre application Azure IoT Central

*Cet article s’applique aux développeurs d’appareils.*

Cet article vous montre comment connecter un appareil Azure Sphere (DevKit) à une application Azure IoT Central.

Azure Sphere est une plateforme d’application générale sécurisée qui présente des fonctionnalités de sécurité et de communication intégrées pour les appareils connectés à Internet. Elle comprend une unité de microcontrôleur (MCU) transversale, connectée et sécurisée, un système d'exploitation Linux personnalisé et un service de sécurité cloud qui offre une sécurité continue et renouvelable. Pour plus d’informations, consultez [Présentation d’Azure Sphere](/azure-sphere/product-overview/what-is-azure-sphere).

Les [kits de développement Azure Sphere](https://azure.microsoft.com/services/azure-sphere/get-started/) mettent à votre disposition tout ce dont vous avez besoin pour démarrer le prototypage et le développement d’applications Azure Sphere. Azure IoT Central avec Azure Sphere active une pile de bout en bout pour une solution IoT. Azure Sphere fournit la prise en charge des appareils et IoT Central en tant que plateforme d’applications IoT managée sans code.

Dans cet article sur les procédures, vous :

- Créez un appareil Azure Sphere dans IoT Central à l’aide du modèle d’appareil du kit de développement Azure Sphere à partir de la bibliothèque.
- Préparez l'appareil du kit de développement Azure Sphere pour Azure IoT.
- Connectez le kit de développement Azure Sphere à Azure IoT Central.
- Affichez les données de télémétrie de l'appareil dans IoT Central.

## <a name="prerequisites"></a>Prérequis

Pour effectuer les étapes de cet article, vous avez besoin des ressources suivantes :

- Une application Azure IoT Central.
- Visual Studio 2019, version 16.4 ou ultérieure
- Un [kit de développement MT3620 Azure Sphere de Seeed Studios](/azure-sphere/hardware/mt3620-reference-board-design).

> [!NOTE]
> Si vous n’avez pas d’appareil physique, après la première étape, passez à la dernière section pour essayer un appareil simulé.

## <a name="create-the-device-in-iot-central"></a>Créer l’appareil dans IoT Central

Pour créer un appareil Azure Sphere dans IoT Central :

1. Dans votre application Azure IoT Central, sélectionnez l’onglet **Modèles d’appareil**, puis sélectionnez **+ Nouveau**. Dans la section **Utiliser un modèle d’appareil proposé**, sélectionnez **Exemple d'appareil Azure Sphere**.

    :::image type="content" source="media/howto-connect-sphere/sphere-create-template.png" alt-text="Modèle d’appareil pour kit de développement Azure Sphere":::

1. Dans le modèle d’appareil, modifiez la vue appelée **Vue d’ensemble** pour afficher **Température** et **Pression bouton**.

1. Sélectionnez le type de vue **Modification des données de l'appareil et du cloud** pour ajouter une autre vue affichant la propriété de lecture/écriture **LED d'état**. Faites glisser la propriété **LED d'état** vers le rectangle en pointillé vide situé à droite du formulaire. Sélectionnez **Enregistrer**.

## <a name="prepare-the-device"></a>Préparer l’appareil

Avant de connecter l’appareil du kit de développement Azure Sphere à IoT Central, vous devez [configurer l’appareil et l’environnement de développement](https://github.com/Azure/azure-sphere-samples/tree/master/Samples/AzureIoT).

## <a name="connect-the-device"></a>Connecter l’appareil

Pour permettre à l’exemple de se connecter à IoT Central, vous devez [configurer une application Azure IoT Central, puis modifier le manifeste d’application de l’exemple](https://aka.ms/iotcentral-sphere-git-readme).

## <a name="view-the-telemetry-from-the-device"></a>Afficher les données de télémétrie de l'appareil

Une fois l'appareil connecté à IoT Central, les données de télémétrie s'affichent sur le tableau de bord.

:::image type="content" source="media/howto-connect-sphere/sphere-view.png" alt-text="Modèle d’appareil pour kit de développement Azure Sphere":::

## <a name="create-a-simulated-device"></a>Créez un appareil simulé.

À défaut d'appareil de kit de développement Azure Sphere physique, vous pouvez créer un appareil simulé pour essayer l’application Azure IoT Central.

Pour créer un appareil simulé :

- Sélectionnez **Appareils > Azure IoT Sphere**
- Sélectionnez **+Nouveau**.
- Entrez un **ID d'appareil** unique et un **nom d'appareil** convivial.
- Activez le paramètre **Simulé**.
- Sélectionnez **Create** (Créer).

## <a name="next-steps"></a>Étapes suivantes

Si vous êtes un développeur d’appareils, nous vous suggérons les étapes suivantes :

- En savoir plus sur la [connectivité des appareils dans Azure IoT Central](./concepts-get-connected.md)
- Découvrir comment [superviser la connectivité des appareils à l’aide d’Azure CLI](./howto-monitor-devices-azure-cli.md)
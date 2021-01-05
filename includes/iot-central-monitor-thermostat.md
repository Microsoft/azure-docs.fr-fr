---
title: Fichier include
description: Fichier include
services: iot-central
author: dominicbetts
ms.service: iot-central
ms.topic: include
ms.date: 03/12/2020
ms.author: dobett
ms.custom: include file
ms.openlocfilehash: 28f676892967abbd0da63d7a75ea3d164b87ce97
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96017481"
---
En tant qu’opérateur dans votre application Azure IoT Central, vous pouvez effectuer les opérations suivantes :

* Affichez les données de télémétrie envoyées par l’appareil dans la page **Vue d’ensemble** :

    :::image type="content" source="media/iot-central-monitor-thermostat/view-telemetry.png" alt-text="Visualiser la télémétrie d’un appareil":::

* Affichez les propriétés de l’appareil dans la page **À propos** :

    :::image type="content" source="media/iot-central-monitor-thermostat/about-properties.png" alt-text="Voir les propriétés de l’appareil":::

## <a name="customize-the-device-template"></a>Personnaliser le modèle d’appareil

En tant que développeur de solutions, vous pouvez personnaliser le modèle d’appareil qu’IoT Central a créé automatiquement lorsque l’appareil de thermostat s’est connecté.

Pour ajouter une propriété cloud afin de stocker le nom du client associé à l’appareil

1. Dans votre application IoT Central, accédez au modèle d’appareil **Thermostat** dans la page **Modèles d’appareil**.

1. Dans le modèle d’appareil **Thermostat**, sélectionnez **Propriétés du cloud**.

1. Sélectionnez **Ajouter une propriété cloud**. Entrez *Customer name* comme **Nom d’affichage** et choisissez **Chaîne** comme **Schéma**. Ensuite, sélectionnez **Enregistrer**.

Pour personnaliser l’affichage de la propriété **Get Max-Min report** dans votre application IoT Central, sélectionnez **Personnaliser** dans le modèle d’appareil. Remplacez **Get Max-Min report** par *Get status report*. Ensuite, sélectionnez **Enregistrer**.

Le modèle **Thermostat** comprend la propriété accessible en écriture **Target Temperature**, et le modèle d’appareil comprend la propriété cloud **Customer Name**. Créez une vue utilisable par un opérateur pour modifier ces propriétés :

1. Sélectionnez **Vues**, puis sélectionnez la vignette **Modification des données de l’appareil et du cloud**.

1. Entrez _Propriétés_ comme nom de formulaire.

1. Sélectionnez les propriétés **Target Temperature** et **Customer Name**. Sélectionnez ensuite **Ajouter une section**.

1. Enregistrez vos modifications.

:::image type="content" source="media/iot-central-monitor-thermostat/properties-view.png" alt-text="Vue pour la mise à jour des valeurs de propriétés":::

## <a name="publish-the-device-template"></a>Publier le modèle d’appareil

Pour qu’un opérateur puisse voir et utiliser les personnalisations que vous avez effectuées, vous devez publier le modèle d’appareil.

À partir du modèle d’appareil **Thermostat**, sélectionnez **Publier**. Dans le panneau **Publier ce modèle d’appareil dans l’application**, sélectionnez **Publier**.

Un opérateur peut maintenant utiliser la vue **Propriétés** pour mettre à jour les valeurs de propriétés, et appeler une commande nommée **Get status report** dans la page de commandes de l’appareil :

* Mettez à jour les valeurs des propriétés inscriptibles dans la page **Propriétés** :

    :::image type="content" source="media/iot-central-monitor-thermostat/update-properties.png" alt-text="Mettre à jour les propriétés de l’appareil":::

* Appelez les commandes à partir de la page **Commandes** :

    :::image type="content" source="media/iot-central-monitor-thermostat/call-command.png" alt-text="Appeler la commande":::

    :::image type="content" source="media/iot-central-monitor-thermostat/command-response.png" alt-text="Afficher la réponse de la commande":::

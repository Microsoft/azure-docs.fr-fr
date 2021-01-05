---
title: Démarrage rapide – Superviser vos appareils dans Azure IoT Central
description: Démarrage rapide - En tant qu’opérateur, découvrez comment utiliser votre application Azure IoT Central pour superviser vos appareils.
author: dominicbetts
ms.author: dobett
ms.date: 11/16/2020
ms.topic: quickstart
ms.service: iot-central
services: iot-central
ms.custom: mvc
ms.openlocfilehash: e4485e4f8e873b1b49dc3d6df72eb04a1cbe8c17
ms.sourcegitcommit: 9889a3983b88222c30275fd0cfe60807976fd65b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94992752"
---
# <a name="quickstart-use-azure-iot-central-to-monitor-your-devices"></a>Démarrage rapide : Utiliser Azure IoT Central pour surveiller vos appareils

*Cet article s’applique aux opérateurs, aux créateurs et aux administrateurs.*

Ce démarrage rapide vous explique comment, en tant qu'opérateur, vous pouvez utiliser votre application Microsoft Azure IoT Central pour surveiller vos appareils et modifier les paramètres.

## <a name="prerequisites"></a>Prérequis

Avant de commencer, vous devez suivre les trois guides de démarrage rapide précédents : [Créer une application Azure IoT Central](./quick-deploy-iot-central.md), [Ajouter un appareil simulé à votre application IoT Central](./quick-create-simulated-device.md) et [Configurer des règles et des actions pour votre appareil](quick-configure-rules.md).

## <a name="receive-a-notification"></a>Recevoir une notification

Azure IoT Central envoie des notifications sur les appareils sous forme de courriers électroniques. En tant que créateur, vous avez ajouté une règle pour envoyer une notification à un opérateur lorsque l’humidité au niveau d’un capteur d’appareil connecté dépasse un seuil. En tant qu’opérateur, vous consultez vos e-mails pour voir si vous avez des notifications.

Ouvrez le courrier électronique que vous avez reçu à la fin du guide de démarrage rapide [Configurer des règles et des actions pour votre appareil](quick-configure-rules.md). Dans l’e-mail, sélectionnez le lien vers l’appareil :

:::image type="content" source="media/quick-monitor-devices/email.png" alt-text="Capture d’écran montrant l’e-mail de notification":::

La vue **Vue d’ensemble** de l’appareil simulé que vous avez créé dans les démarrages rapides précédents s’ouvre dans votre navigateur :

:::image type="content" source="media/quick-monitor-devices/dashboard.png" alt-text="Capture d’écran montrant une vue d’ensemble de l’appareil qui a déclenché la notification":::

## <a name="investigate-an-issue"></a>Analyser un problème

En tant qu’opérateur, vous pouvez consulter les informations sur l’appareil dans les vues **Vue d'ensemble**, **À propos** et **Commandes**. Le créateur a créé une vue **Gérer l’appareil** pour vous permettre de modifier les informations de l’appareil et définir ses propriétés.

Le graphique sur le tableau de bord affiche un tracé de l’humidité de l’appareil. Vous jugez que l’humidité de l’appareil est trop élevée.

## <a name="remediate-an-issue"></a>Résoudre un problème

Pour apporter une modification à l’appareil, utilisez la page **Gérer l’appareil**.

Affectez la valeur 80 à **Target temperature** pour chauffer l’appareil et réduire l’humidité. Choisissez **Enregistrer** pour mettre à jour l’appareil. Quand l’appareil confirme la modification du paramètre, l’état de la propriété passe à **synchronisé** :

:::image type="content" source="media/quick-monitor-devices/change-settings.png" alt-text="Capture d’écran montrant le paramètre de température cible mis à jour pour l’appareil":::

## <a name="next-steps"></a>Étapes suivantes

Dans ce démarrage rapide, vous avez appris comment :

* Recevoir une notification
* Analyser un problème
* Résoudre un problème

Maintenant que vous savez comment superviser votre appareil, vous pouvez passer à l’étape suivante :

> [!div class="nextstepaction"]
> [Créer et gérer un modèle d'appareil](howto-set-up-template.md).

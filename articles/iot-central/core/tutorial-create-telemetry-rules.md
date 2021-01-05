---
title: Tutoriel - Créer et gérer des règles dans votre application Azure IoT Central
description: Ce tutoriel vous montre comment les règles Azure IoT Central vous permettent de superviser vos appareils quasiment en temps réel et d’appeler automatiquement des actions, par exemple l’envoi d’un e-mail, quand la règle se déclenche.
author: dominicbetts
ms.author: dobett
ms.date: 11/16/2020
ms.topic: tutorial
ms.service: iot-central
services: iot-central
ms.openlocfilehash: 6d49e3585460c95ca931f497a63cbc281aed1db1
ms.sourcegitcommit: 9889a3983b88222c30275fd0cfe60807976fd65b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94990970"
---
# <a name="tutorial-create-a-rule-and-set-up-notifications-in-your-azure-iot-central-application"></a>Tutoriel : Créer une règle et configurer des notifications dans votre application Azure IoT Central

*Cet article s’applique aux opérateurs, aux créateurs et aux administrateurs.*

Vous pouvez utiliser Azure IoT Central pour surveiller à distance vos appareils connectés. Les règles Azure IoT Central vous permettent de superviser vos appareils quasiment en temps réel et d’appeler automatiquement des actions, comme l’envoi d’un e-mail. Cet article explique comment créer des règles pour superviser la télémétrie envoyée par vos appareils.

Les appareils utilisent la télémétrie pour envoyer des données numériques. Une règle se déclenche quand la télémétrie sélectionnée dépasse un seuil spécifique.

Dans ce tutoriel, vous allez créer une règle pour envoyer un e-mail quand la température d’un capteur simulé dépasse 70&deg; F (environ 21° C).

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> * Créer une règle
> * Ajouter une action de messagerie

## <a name="prerequisites"></a>Prérequis

Avant de commencer, suivez les deux guides de démarrage rapide [Créer une application Azure IoT Central](./quick-deploy-iot-central.md) et [Ajouter un appareil simulé à votre application IoT Central](./quick-create-simulated-device.md) afin de créer le modèle d’appareil **Sensor Controller** à utiliser.

## <a name="create-a-rule"></a>Créer une règle

Pour créer une règle de télémétrie, le modèle d’appareil doit contenir au moins une valeur de télémétrie. Ce tutoriel utilise un appareil **Sensor Controller** simulé qui envoie la télémétrie relative à la température et à l’humidité. Vous avez ajouté ce modèle d’appareil et créé un appareil simulé dans le guide de démarrage rapide [Ajouter un appareil simulé à votre application IoT Central](./quick-create-simulated-device.md). La règle surveille la température signalée par l’appareil et envoie un e-mail quand elle dépasse 70 degrés.

1. Dans le volet gauche, sélectionnez **Règles**.

1. Si vous n’avez pas encore créé de règles, vous voyez l’écran suivant :

    :::image type="content" source="media/tutorial-create-telemetry-rules/rules-landing-page.png" alt-text="Capture d’écran montrant la liste vide des règles":::

1. Sélectionnez **+ Nouveau** pour ajouter une nouvelle règle.

1. Entrez le nom _Supervision de la température_ pour identifier la règle, puis appuyez sur Entrée.

1. Sélectionnez le modèle d’appareil **Sensor Controller**. Par défaut, la règle s’applique automatiquement à tous les appareils associés à ce modèle d’appareil. Pour filtrer une partie des appareils, sélectionnez **+ Filtre**, puis utilisez les propriétés nécessaires pour identifier les appareils. Pour désactiver la règle, servez-vous du bouton **Activé/Désactivé** :

    :::image type="content" source="media/tutorial-create-telemetry-rules/device-filters.png" alt-text="Capture d’écran montrant la sélection du modèle d’appareil dans la définition de la règle":::

### <a name="configure-the-rule-conditions"></a>Configurer les conditions de la règle

Les conditions définissent les critères de supervision de la règle. Dans ce tutoriel, vous configurez la règle pour qu’elle se déclenche quand la température dépasse 70 &deg;F (environ 21° C).

1. Sélectionnez **Température** dans le menu déroulant **Télémétrie**.

1. Choisissez ensuite **Est supérieur à** pour **Opérateur**, puis entrez _70_ pour **Valeur**.

    :::image type="content" source="media/tutorial-create-telemetry-rules/condition-filled-out.png" alt-text="Capture d’écran montrant la condition de température pour la règle":::

1. Vous pouvez éventuellement définir une **Agrégation de temps**. Quand vous sélectionnez une agrégation de temps, vous devez également sélectionner un type d’agrégation, par exemple une moyenne ou une somme, dans la liste déroulante d’agrégation.

    * Sans agrégation, la règle se déclenche pour chaque point de données de télémétrie qui répond à la condition. Par exemple, si vous configurez la règle pour se déclencher quand la température dépasse 70, elle se déclenche presque instantanément quand la température de l’appareil dépasse cette valeur.
    * Avec l’agrégation, la règle se déclenche si la valeur agrégée des points de données de télémétrie de la fenêtre de temps répond à la condition. Par exemple, si vous configurez la règle pour se déclencher quand la température dépasse 70 et avec une agrégation de temps moyenne de 10 minutes, la règle se déclenche quand l’appareil signale une température moyenne supérieure à 70 calculée sur un intervalle de 10 minutes.

    :::image type="content" source="media/tutorial-create-telemetry-rules/aggregate-condition-filled-out.png" alt-text="Capture d’écran montrant la condition d’agrégation remplie":::

Vous pouvez ajouter plusieurs conditions à une règle en sélectionnant **+ Condition**. Quand plusieurs conditions sont spécifiées, toutes les conditions doivent être remplies pour que la règle se déclenche. Chaque condition est jointe par une clause `AND` implicite. Si vous utilisez une agrégation de temps avec plusieurs conditions, toutes les valeurs de télémétrie doivent être agrégées.

### <a name="configure-actions"></a>Configurer les actions

Une fois que vous avez défini la condition, vous configurez les actions à entreprendre quand la règle se déclenche. Les actions sont appelées quand toutes les conditions spécifiées dans la règle ont la valeur true.

1. Sélectionnez **+ E-mail** dans la section **Actions**.

1. Entrez _Avertissement de température_ pour le nom d’affichage de l’action, votre adresse e-mail dans le champ **À**, puis _Vous devez vérifier l’appareil !_ sous forme de note à afficher dans le corps de l’e-mail.

    > [!NOTE]
    > Les e-mails sont envoyés seulement aux utilisateurs qui ont été ajoutés à l’application et qui se sont connectés au moins une fois. Découvrez plus en détail la [gestion des utilisateurs](howto-administer.md) dans Azure IoT Central.

    :::image type="content" source="media/tutorial-create-telemetry-rules/configure-action.png" alt-text="Capture d’écran montrant l’action d’e-mail pour la règle":::

1. Pour enregistrer l'action, choisissez **Terminé**. Vous pouvez ajouter plusieurs actions à une règle.

1. Pour enregistrer la règle, choisissez **Enregistrer**. La règle est active au bout de quelques minutes et commence à surveiller la télémétrie envoyée à votre application. Quand la condition spécifiée dans la règle est remplie, cette dernière déclenche l’action d’envoi d’e-mail configurée.

Après un certain temps, vous recevez un e-mail quand la règle se déclenche :

:::image type="content" source="media/tutorial-create-telemetry-rules/email.png" alt-text="Capture d’écran montrant l’e-mail de notification":::

## <a name="delete-a-rule"></a>Supprimer une règle

Si vous n’avez plus besoin d’une règle, supprimez-la en ouvrant la règle et en choisissant **Supprimer**.

## <a name="enable-or-disable-a-rule"></a>Activer ou désactiver une règle

Choisissez la règle que vous voulez activer ou désactiver. Activez ou désactivez le bouton **Activé/désactivé** de la règle afin de l’activer ou de la désactiver pour tous les appareils délimités dans la règle.

## <a name="enable-or-disable-a-rule-for-specific-devices"></a>Activer ou désactiver une règle pour des appareils spécifiques

Choisissez la règle à personnaliser. Utilisez un ou plusieurs filtres dans la section **Appareils cibles** pour limiter l’étendue de la règle aux appareils à superviser.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

* Créer une règle basée sur la télémétrie
* Ajouter une action

Une fois que vous avez défini une règle basée sur un seuil, l’étape suivante suggérée consiste à apprendre à :

> [!div class="nextstepaction"]
> [Configurer l’exportation de données continue](./howto-export-data.md)

---
title: Installer et utiliser l’explorateur Azure IoT | Microsoft Docs
description: Installez l’outil d’exploration Azure IoT et utilisez-le pour interagir avec les appareils IoT Plug-and-Play connectés à un hub IoT. Bien que cet article se concentre sur l’utilisation des appareils IoT Plug-and-Play, vous pouvez utiliser l’outil avec n’importe quel appareil connecté à votre hub.
author: dominicbetts
ms.author: dobett
ms.date: 11/10/2020
ms.topic: how-to
ms.service: iot-pnp
services: iot-pnp
ms.custom: contperf-fy21q2
ms.openlocfilehash: b0a86a32148086a3b644d0bf147d12be0d812536
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97030313"
---
# <a name="install-and-use-azure-iot-explorer"></a>Installer et utiliser l’explorateur Azure IoT

L’explorateur Azure IoT est un outil graphique permettant d’interagir avec les appareils à votre hub IoT. Cet article se concentre sur l’utilisation de l’outil pour tester vos appareils IoT Plug-and-Play. Après avoir installé l’outil sur votre ordinateur local, vous pouvez l’utiliser pour vous connecter à un hub. Vous pouvez utiliser l’outil pour afficher la télémétrie que les appareils envoient, utiliser les propriétés de l’appareil et appeler des commandes.

Cet article vous montre comment :

- Installer et configurer l’explorateur Azure IoT.
- Utiliser l’outil pour interagir avec vos appareils IoT Plug-and-Play et les tester.

Pour plus d’informations générales sur l’utilisation de l’outil, consultez le fichier [Lisez-moi](https://github.com/Azure/azure-iot-explorer/blob/master/README.md) de GitHub.

Pour utiliser l’outil qu’est l’explorateur Azure IoT, vous avez besoin des éléments suivants :

- Un IoT Hub Azure. Il y a de nombreuses façons d’ajouter un hub IOT à votre abonnement Azure, par exemple la [Création d’un hub IOT à l’aide de l’interface de ligne de commande Azure](../iot-hub/iot-hub-create-using-cli.md). Vous avez besoin de la chaîne de connexion du hub IoT pour exécuter l’outil explorateur Azure IoT. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.
- Un appareil inscrit dans votre hub IoT. Vous pouvez utiliser l’explorateur IoT pour créer et gérer les inscriptions d’appareils dans votre IoT Hub.

## <a name="install-azure-iot-explorer"></a>Installez l’explorateur Azure IoT

Accédez aux [versions de l’explorateur Azure IOT](https://github.com/Azure/azure-iot-explorer/releases) et développez la liste des ressources pour la version la plus récente. Téléchargez et installez la version la plus récente de l’application.

>[!Important]
> Mettez à jour vers la version 0.13. x pour résoudre les modèles à partir de n’importe quel référentiel basé sur [https://github.com/Azure/iot-plugandplay-models](https://github.com/Azure/iot-plugandplay-models)

## <a name="use-azure-iot-explorer"></a>Utilisez l’explorateur Azure IoT

Concernant l’appareil, vous pouvez connecter votre propre appareil ou utiliser l’un des exemples d’appareils simulés. Pour obtenir des exemples d’appareils simulés écrits en différents langages, consultez le guide de démarrage rapide [Connecter un exemple d’application pour appareil IoT Plug-and-Play à IoT Hub](quickstart-connect-device.md).

### <a name="connect-to-your-hub"></a>Connectez-vous à votre hub

La première fois que vous exécutez l’explorateur Azure IoT, vous êtes invité à entrer la chaîne de connexion de votre hub IoT. Après avoir ajouté la chaîne de connexion, sélectionnez **Connexion**. Vous pouvez utiliser les paramètres de l’outil pour basculer vers un autre hub IoT en mettant à jour la chaîne de connexion.

La définition de modèle pour un appareil IoT Plug-and-Play est stockée dans le référentiel public, sur l’appareil connecté ou dans un dossier local. Par défaut, l’outil recherche la définition de votre modèle dans le référentiel public et sur votre appareil connecté. Vous pouvez ajouter et supprimer des sources, ou configurer la priorité des sources dans **Paramètres** :

Pour ajouter une source :

1. Accédez à **Dossier de base/Paramètres IoT Plug-and-Play**
2. Sélectionnez **Ajouter** et choisissez votre source, à partir d’un référentiel ou d’un dossier local.

Pour supprimer une source :

1. Accédez à **Dossier de base/Paramètres IoT Plug-and-Play**
2. Recherchez la source que vous souhaitez supprimer.
3. Sélectionnez **X** pour la supprimer.

Modifiez les priorités sources :

Vous pouvez faire glisser et déposer l’une des sources de définition de modèle dans une position différente sur la liste.

### <a name="view-devices"></a>Afficher les appareils

Une fois que l’outil se connecte à votre hub IoT, une page présente la liste des **Appareils** et des identités d’appareils qui sont inscrits auprès de votre hub IoT. Vous pouvez sélectionner n’importe quelle entrée de la liste pour afficher plus d’informations à son sujet.

Dans la page **Appareils**, vous pouvez :

- sélectionner **Nouveau** pour inscrire un nouvel appareil auprès de votre hub, puis entrer un ID d’appareil. Utilisez les paramètres par défaut pour générer automatiquement les clés d’authentification et activer la connexion à votre hub.
- Sélectionnez un appareil, puis sélectionnez **Supprimer** pour supprimer une identité d’appareil. Passez en revue les détails de l’appareil avant d’effectuer cette action pour vous assurer que vous supprimez l’identité du bon appareil.

## <a name="interact-with-a-device"></a>Interagir avec un appareil

Dans la page de liste **Appareils**, sélectionnez une valeur dans la colonne **ID d’appareil** pour afficher la page de détails de l’appareil inscrit. Pour chaque appareil, il existe deux sections : **Appareil** et **Jumeau numérique.**

### <a name="device"></a>Appareil

Cette section comprend les onglets **Identité d’appareil**, **Jumeau d’appareil**, **Télémétrie**, **Méthode directe**, **Messages cloud-à-appareil** et **Identité de module**.

- Vous pouvez voir et mettre à jour les informations d’[identité d’appareil](../iot-hub/iot-hub-devguide-identity-registry.md) sous l’onglet **Identité d’appareil**.
- Vous pouvez accéder aux informations sur le [jumeau d’appareil](../iot-hub/iot-hub-devguide-device-twins.md) sous l’onglet **Jumeau d’appareil**.
- Si un appareil est connecté et envoie activement des données, vous pouvez voir les données de [télémétrie](../iot-hub/iot-hub-devguide-messages-read-builtin.md) sous l’onglet **Données de télémétrie**.
- Vous pouvez appeler une [méthode directe](../iot-hub/iot-hub-devguide-direct-methods.md) sur l’appareil sous l’onglet **Méthode directe**.
- Vous pouvez envoyer un [message cloud-à-appareil](../iot-hub/iot-hub-devguide-messages-c2d.md) sous l’onglet **Messages appareil-à-cloud**.
- Vous pouvez accéder aux informations du [jumeau de module](../iot-hub/iot-hub-devguide-module-twins.md).

### <a name="iot-plug-and-play-components"></a>Composants IoT Plug-and-Play

Si l’appareil est connecté au hub à l’aide d’un **ID de modèle**, l’outil affiche l’onglet **Composants IoT Plug-and-Play**, où vous pouvez voir **l’ID de modèle**.

Si l’**ID de modèle** est accessible dans l’une des sources configurées (référentiel public ou dossier local), la liste des composants s’affiche. La sélection d’un composant montre les propriétés, les commandes et la télémétrie disponibles.

Dans la page **Composant**, vous pouvez afficher les propriétés en lecture seule, mettre à jour les propriétés accessibles en écriture, appeler des commandes et afficher les messages de télémétrie produits par ce composant.

:::image type="content" source="media/howto-use-iot-explorer/components-iot-explorer.png" alt-text="Afficher les composants dans l’explorateur Azure IoT":::

#### <a name="properties"></a>Propriétés

:::image type="content" source="media/howto-use-iot-explorer/properties-iot-explorer.png" alt-text="Afficher les propriétés dans l’explorateur Azure IoT":::

Vous pouvez afficher les propriétés en lecture seule définies dans une interface sur l’onglet **Propriétés (en lecture seule)** . Vous pouvez mettre à jour les propriétés accessibles en écriture définies dans une interface sur l’onglet **Propriétés (accessibles en écriture)**  :

1. Accédez à l’onglet **Propriétés (accessibles en écriture)** .
1. Cliquez sur la propriété que vous souhaitez mettre à jour.
1. Entrez la nouvelle valeur de la propriété.
1. Affichez un aperçu de la charge utile à envoyer à l’appareil.
1. Envoyer la modification.

Après avoir envoyé une modification, vous pouvez suivre l’état de la mise à jour : **synchronisation**, **succès** ou **erreur**. Une fois la synchronisation terminée, la nouvelle valeur de votre propriété s’affiche dans la colonne **Propriété rapportée**. Si vous accédez à d’autres pages avant la fin de la synchronisation, l’outil vous avertit lorsque la mise à jour est terminée. Vous pouvez également utiliser le centre de notifications de l’outil pour afficher l’historique des notifications.

#### <a name="commands"></a>Commandes

Pour envoyer une commande à un appareil, accédez à l’onglet **Commandes** :

1. Dans la liste des commandes, développez la commande que vous souhaitez déclencher.
1. Entrez les valeurs requises pour la commande.
1. Affichez un aperçu de la charge utile à envoyer à l’appareil.
1. Envoyez la commande.

#### <a name="telemetry"></a>Télémétrie

Pour afficher la télémétrie de l’interface sélectionnée, accédez à son onglet **Télémétrie**.

#### <a name="known-issues"></a>Problèmes connus

Pour obtenir la liste des fonctionnalités IoT prises en charge par la version la plus récente de l’outil, consultez [Liste des fonctionnalités](https://github.com/Azure/azure-iot-explorer/wiki).

## <a name="next-steps"></a>Étapes suivantes

Dans cet article de procédure, vous avez appris à installer et à utiliser l’explorateur Azure IoT pour interagir avec vos appareils IoT Plug-and-Play. À présent, nous vous suggérons de découvrir comment [Installer et utiliser les outils de création DTDL](howto-use-dtdl-authoring-tools.md).

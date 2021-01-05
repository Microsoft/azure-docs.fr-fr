---
title: Tutoriel - Définir un nouveau type d’appareil de passerelle dans Azure IoT Central | Microsoft Docs
description: Ce tutoriel vous indique comment, en tant que créateur, définir un nouveau type d’appareil de passerelle IoT dans votre application Azure IoT Central.
author: rangv
ms.author: rangv
ms.date: 10/22/2019
ms.topic: tutorial
ms.service: iot-central
services: iot-central
ms.custom: mvc
manager: peterpr
ms.openlocfilehash: f95df6fb74071dc16814f11fb48521f984c89778
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95995520"
---
# <a name="tutorial---define-a-new-iot-gateway-device-type-in-your-azure-iot-central-application"></a>Tutoriel - Définir un nouveau type d’appareil de passerelle IoT dans votre application Azure IoT Central

*Cet article s’applique aux créateurs de solutions et aux développeurs d’appareils.*

Ce tutoriel vous explique comment, en tant que créateur de solutions, utiliser un modèle d’appareil de passerelle pour définir un appareil de passerelle dans votre application IoT Central. Vous configurez ensuite plusieurs appareils en aval qui se connectent à votre application IoT Central via l’appareil de passerelle. 

Dans ce tutoriel, vous créez un modèle d’appareil de passerelle **Smart Building** (Construction intelligente). Un appareil de passerelle **Construction intelligente** a des relations avec d’autres appareils en aval.

![Diagramme de la relation entre l’appareil de passerelle et les appareils en aval](./media/tutorial-define-gateway-device-type/gatewaypattern.png)

En plus de permettre à des appareils en aval de communiquer avec votre application IoT Central, un appareil de passerelle peut aussi :

* Envoyer sa propre télémétrie, comme une température.
* Répondre aux mises à jour de propriétés accessibles en écriture effectuées par un opérateur. Par exemple, un opérateur peut modifier l’intervalle d’envoi de la télémétrie.
* Répondre à des commandes, comme le redémarrage de l’appareil.

> [!div class="checklist"]
> Créer des modèles d’appareil en aval Créer un modèle d’appareil de passerelle Publier le modèle d’appareil Créer les appareils simulés

## <a name="prerequisites"></a>Prérequis

Pour suivre ce tutoriel, vous devez [Créer une application Azure IoT Central](./quick-deploy-iot-central.md).

## <a name="create-downstream-device-templates"></a>Créer des modèles d’appareils en aval

Ce tutoriel utilise des modèles d’appareil pour un appareil **S1 Sensor** et un appareil **RS40 Occupancy Sensor** pour générer des appareils en aval simulés.

Pour créer un modèle d’appareil pour un appareil **S1 Sensor** :

1. Dans le volet gauche, sélectionnez **Modèles d’appareil**. Sélectionnez ensuite **+** pour commencer à ajouter le modèle.

1. Faites défiler jusqu’à faire apparaître la vignette pour l’appareil **S1 Sensor**. Sélectionnez la vignette, puis sélectionnez **Suivant : Personnaliser**.

1. Dans la page **Vérifier**, sélectionnez **Créer** pour ajouter le modèle d’appareil à votre application. 

Pour créer un modèle d’appareil pour un appareil **_RS40 Occupancy Sensor_*  :

1. Dans le volet gauche, sélectionnez **Modèles d’appareil**. Sélectionnez ensuite **+** pour commencer à ajouter le modèle.

1. Faites défiler jusqu’à faire apparaître la vignette pour l’appareil **_RS40 Occupancy Sensor_*. Sélectionnez la vignette, puis sélectionnez **Suivant : Personnaliser**.

1. Dans la page **Vérifier**, sélectionnez **Créer** pour ajouter le modèle d’appareil à votre application. 

Vous avez maintenant des modèles d’appareil pour les deux types d’appareils en aval :

![Modèles d’appareil pour les appareils en aval](./media/tutorial-define-gateway-device-type/downstream-device-types.png)


## <a name="create-a-gateway-device-template"></a>Créer un modèle d’appareil de passerelle

Dans ce tutoriel, vous créez à partir de zéro un modèle d’appareil pour un appareil de passerelle. Vous utilisez ce modèle ultérieurement pour créer un appareil de passerelle simulé dans votre application.

Pour ajouter un nouveau modèle d’appareil à votre application :

1. Dans le volet gauche, sélectionnez **Modèles d’appareil**. Sélectionnez ensuite **+** pour commencer à ajouter le modèle.

1. Dans la page **Sélectionner un type de modèle**, sélectionnez la vignette **Appareil IoT**, puis sélectionnez **Suivant : Personnaliser**.

1. Dans la page **Personnaliser l’appareil**, cochez la case **Appareil de passerelle**.

1. Dans la page **Vérifier**, sélectionnez **Créer**. 

1. Entrez **Appareil de passerelle Smart Building** comme nom du modèle.

1. Dans la page **Créer un modèle de capacité**, sélectionnez la vignette **Personnalisé**.

1. Sélectionnez **+** pour ajouter une interface.  Choisissez l’interface standard **Informations de l’appareil**.

### <a name="add-relationships"></a>Ajouter des relations

Ensuite, vous ajoutez des relations aux modèles pour les modèles d’appareil en aval :

1. Dans le modèle **Smart Building gateway device**, sélectionnez **Relations**.

1. Sélectionnez **+ Ajouter une relation**. Entrez **Environmental Sensor** (Capteur d’environnement) comme nom d’affichage, puis sélectionnez **S1 Sensor** comme cible.

1. Resélectionnez **+ Ajouter une relation**. Entrez **Occupancy Sensor** (Capteur d’occupation) comme nom d’affichage, puis sélectionnez **RS40 Occupancy Sensor** comme cible.

1. Sélectionnez **Enregistrer**.

![Modèle d’appareil de passerelle Smart Building montrant les relations](./media/tutorial-define-gateway-device-type/relationships.png)

### <a name="add-cloud-properties"></a>Ajouter des propriétés du cloud

Un modèle d’appareil de passerelle peut inclure des propriétés cloud. Les propriétés du cloud existent uniquement dans l’application IoT Central, et ne sont jamais envoyées à un appareil ni reçues à partir d’un appareil.

Pour ajouter des propriétés cloud au modèle **Smart Building gateway device**.

1. Dans le modèle **Smart Building gateway device**, sélectionnez **Propriétés du cloud**.

1.  Utilisez les informations du tableau suivant pour ajouter deux propriétés cloud à votre modèle d’appareil de passerelle.

    | Nom complet      | Type sémantique | schéma |
    | ----------------- | ------------- | ------ |
    | Dernière date de service | None          | Date   |
    | Nom du client     | None          | String |

2. Sélectionnez **Enregistrer**.

### <a name="create-views"></a>Créer des vues

En tant que générateur, vous pouvez personnaliser l’application pour présenter des informations pertinentes sur l’appareil capteur environnemental à un opérateur. Vos personnalisations permettent à l’opérateur de gérer l’appareil capteur environnemental raccordé à l’application. Vous pouvez créer deux types de vues permettant à un opérateur d’interagir avec des appareils :

* Formulaires pour voir et modifier les propriétés de l’appareil et du cloud
* Tableaux de bord pour visualiser les appareils

Pour générer les vues par défaut pour le modèle **Smart Building gateway device**:

1. Dans le modèle **Smart Building gateway device**, sélectionnez **Vues**.

1. Sélectionnez la vignette **Générer les vues par défaut** et vérifiez que toutes les options sont sélectionnées.

1. Sélectionnez **Générer une ou plusieurs vues de tableau de bord par défaut**.

## <a name="publish-the-device-template"></a>Publier le modèle d’appareil

Avant de pouvoir créer un appareil de passerelle simulé ou de connecter un appareil de passerelle réel, vous devez publier votre modèle d’appareil.

Pour publier le modèle d’appareil de passerelle :

1. Sélectionnez le modèle **Smart Building gateway device** dans la page **Modèles d’appareil**.

2. Sélectionnez **Publier**.

3. Dans la boîte de dialogue **Publier un modèle d’appareil**, choisissez **Publier**.

Une fois qu’un modèle d’appareil a été publié, il est visible dans la page **Appareils** et par l’opérateur. Dans un modèle d’appareil publié, vous ne pouvez pas modifier un modèle d’appareil sans créer de version. En revanche, vous pouvez effectuer des mises à jour des propriétés du cloud, des personnalisations et des vues dans un modèle d’appareil publié. Ces mises à jour n’entraînent pas la création d’une version. Après avoir apporté des modifications, sélectionnez **Publier** pour envoyer ces modifications à votre opérateur.

## <a name="create-the-simulated-devices"></a>Créer les appareils simulés

Ce tutoriel utilise des appareils en aval simulés et un appareil de passerelle simulé.

Pour créer un appareil de passerelle simulé :

1. Dans la page **Appareils**, sélectionnez **Smart Building gateway device** dans la liste des modèles d’appareil.

1. Sélectionnez **+** pour commencer à ajouter un nouvel appareil.

1. Conservez l’**ID de l’appareil** et le **Nom de l’appareil** générés. Vérifiez que le commutateur **Simulé** est **Activé**. Sélectionnez **Create** (Créer).

Pour créer des appareils en aval simulés :

1. Dans la page **Appareils**, sélectionnez **RS40 Occupancy Sensor** dans la liste des modèles d’appareil.

1. Sélectionnez **+** pour commencer à ajouter un nouvel appareil.

1. Conservez l’**ID de l’appareil** et le **Nom de l’appareil** générés. Vérifiez que le commutateur **Simulé** est **Activé**. Sélectionnez **Create** (Créer).

1. Dans la page **Appareils**, sélectionnez **S1 Sensor** dans la liste des modèles d’appareil.

1. Sélectionnez **+** pour commencer à ajouter un nouvel appareil.

1. Conservez l’**ID de l’appareil** et le **Nom de l’appareil** générés. Vérifiez que le commutateur **Simulé** est **Activé**. Sélectionnez **Create** (Créer).

![Appareils simulés dans votre application](./media/tutorial-define-gateway-device-type/simulated-devices.png)

### <a name="add-downstream-device-relationships-to-a-gateway-device"></a>Ajouter des relations d’appareils en aval à un appareil de passerelle

Maintenant que vous disposez des appareils simulés dans votre application, vous pouvez créer les relations entre les appareils en aval et l’appareil de passerelle :

1. Dans la page **Appareils**, sélectionnez **S1 Sensor** dans la liste des modèles d’appareil, puis sélectionnez votre appareil **S1 Sensor** simulé.

1. Sélectionnez **Se connecter à une passerelle**.

1. Dans la boîte de dialogue **Se connecter à une passerelle**, sélectionnez le modèle **Smart Building gateway device**, puis sélectionnez l’instance simulée que vous avez créée précédemment.

1. Sélectionnez **Joindre**.

1. Dans la page **Appareils**, sélectionnez **RS40 Occupancy Sensor** dans la liste des modèles d’appareil, puis sélectionnez votre appareil **RS40 Occupancy Sensor** simulé.

1. Sélectionnez **Se connecter à une passerelle**.

1. Dans la boîte de dialogue **Se connecter à une passerelle**, sélectionnez le modèle **Smart Building gateway device**, puis sélectionnez l’instance simulée que vous avez créée précédemment.

1. Sélectionnez **Joindre**.

Vos appareils en aval simulés sont maintenant connectés à votre appareil de passerelle simulé. Si vous accédez à la vue **Appareils en aval** pour votre appareil de passerelle, vous pouvez voir les appareils en aval associés :

![Vue Appareils en aval](./media/tutorial-define-gateway-device-type/downstream-device-view.png)

Sélectionnez un modèle d’appareil de passerelle et une instance d’appareil de passerelle, puis sélectionnez **Rejoindre**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

* Créer une passerelle IoT en tant que modèle d’appareil.
* Créer des propriétés du cloud.
* Créer des personnalisations.
* Définir une visualisation pour la télémétrie de l’appareil.
* Ajouter des relations.
* Publier votre modèle d’appareil.

> [!NOTE]
> La génération de code basé sur VS Code n’est pas prise en charge pour les appareils de passerelle modélisés dans IoT Central.

Ensuite, en tant que développeur d’appareils, vous pouvez apprendre à :

> [!div class="nextstepaction"]
> [Ajouter un appareil Azure IoT Edge à votre application Azure IoT Central](tutorial-add-edge-as-leaf-device.md)

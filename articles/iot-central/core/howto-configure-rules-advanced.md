---
title: Utiliser des workflows pour intégrer votre application Azure IoT Central à d’autres services cloud | Microsoft Docs
description: Cet article pratique vous montre, en tant que développeur, comment configurer les règles et les actions qui intègrent votre application Azure IoT Central à d’autres services cloud. Pour créer une règle avancée, vous utilisez un connecteur IoT Central dans Power Automate ou Azure Logic Apps.
author: dominicbetts
ms.author: dobett
ms.date: 05/12/2020
ms.topic: how-to
ms.service: iot-central
services: iot-central
ms.openlocfilehash: 257855b4f7b1fae56ed8d6a063acfb0588da9b6a
ms.sourcegitcommit: 7dacbf3b9ae0652931762bd5c8192a1a3989e701
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92123318"
---
# <a name="use-workflows-to-integrate-your-azure-iot-central-application-with-other-cloud-services"></a>Utiliser des workflows pour intégrer votre application Azure IoT Central à d’autres services cloud

*Cet article s’applique aux créateurs de solutions.*

Vous pouvez créer des règles dans IoT Central qui déclenchent des actions, telles que l’envoi d’un e-mail, en réponse à des conditions basées sur la télémétrie, comme la température d’un appareil dépassant un seuil.

Le connecteur Azure IoT Central v3 pour Power Automate et Azure Logic Apps vous permet de créer des règles plus avancées pour automatiser les opérations dans IoT Central :

- Lorsqu’une règle est déclenchée dans votre application Azure IoT Central, elle peut déclencher un workflow dans Power Automate ou Azure Logic Apps. Ces flux de travail peuvent exécuter des actions dans d’autres services cloud, tels que Microsoft 365 ou un service tiers.
- Un événement dans un autre service cloud, tel que Microsoft 365, peut déclencher un flux de travail dans Power Automate ou Azure Logic Apps. Ces workflows peuvent exécuter des actions ou récupérer des données à partir de votre application IoT Central.

## <a name="prerequisites"></a>Prérequis

Pour effectuer les étapes décrites dans ce guide pratique, vous avez besoin d’un abonnement Azure actif. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

La configuration de la solution nécessite une application IoT Central version 3. Pour savoir comment vérifier la version de votre application, consultez [À propose de votre application](./howto-get-app-info.md). Pour savoir comment créer une application IoT Central, consultez [Créer une application Azure IoT Central](./quick-deploy-iot-central.md).

> [!NOTE]
> Si vous utilisez une application IoT Central version 2, consultez [Créer des flux de travail avec le connecteur IoT Central dans Azure Logic Apps](/previous-versions/azure/iot-central/core/howto-build-azure-logic-apps) sur le site de documentation des versions précédentes et utiliser le connecteur Azure IoT Central v2

## <a name="trigger-a-workflow-from-a-rule"></a>Déclencher un workflow à partir d’une règle

Avant de pouvoir déclencher un workflow dans Power Automate ou Azure Logic Apps, vous avez besoin d’une règle dans votre application IoT Central. Pour en savoir plus, consultez [Configurer des règles et des actions dans Azure IoT Central](./howto-configure-rules.md).

Pour ajouter le connecteur **Azure IoT Central v3 – Préversion** en tant que déclencheur dans Power Automate :

1. Dans Power Automate, sélectionnez **+ Créer**, sélectionnez l’onglet **Personnaliser**.
1. Recherchez *IoT Central*, puis sélectionnez le connecteur **Azure IoT Central v3 – Préversion**.
1. Dans la liste des déclencheurs, sélectionnez **Lors du déclenchement d’une règle (préversion)** .
1. À l’étape **Lors du déclenchement d’une règle**, sélectionnez votre application IoT Central et la règle que vous utilisez.

Pour ajouter le connecteur **Azure IoT Central v3 – Préversion** en tant que déclencheur dans Azure Logic Apps :

1. Dans **Concepteur d’applications logiques**, sélectionnez le modèle **Application logique vide**.
1. Dans le concepteur, sélectionnez l’onglet **Personnaliser**.
1. Recherchez *IoT Central*, puis sélectionnez le connecteur **Azure IoT Central v3 – Préversion**.
1. Dans la liste des déclencheurs, sélectionnez **Lors du déclenchement d’une règle (préversion)** .
1. À l’étape **Lors du déclenchement d’une règle**, sélectionnez votre application IoT Central et la règle que vous utilisez.

:::image type="content" source="./media/howto-configure-rules-advanced/triggers.png" alt-text="Rechercher le connecteur Azure IoT Central – Préversion et choisir le déclencheur":::

Vous pouvez maintenant ajouter d’autres étapes à votre workflow pour créer votre scénario d’intégration.

## <a name="run-an-action"></a>Exécuter une action

Vous pouvez exécuter des actions dans une application IoT Central à partir des workflows Power Automate et Azure Logic Apps. Tout d’abord, créez votre workflow et utilisez un connecteur pour définir un déclencheur permettant de démarrer le workflow. Utilisez ensuite le connecteur **Azure IoT Central v3 – Préversion** en tant qu’action.

Pour ajouter le connecteur **Azure IoT Central v3 – Préversion** en tant qu’action dans Power Automate :

1. Dans Power Automate, dans le panneau **Choisir une action**, sélectionnez l’onglet **Personnaliser**.
1. Recherchez *IoT Central*, puis sélectionnez le connecteur **Azure IoT Central v3 – Préversion**.
1. Dans la liste des actions, sélectionnez l’action IoT Central que vous souhaitez utiliser.
1. Dans l’étape d’action, terminez la configuration de l’action que vous avez choisie. Ensuite, sélectionnez **Enregistrer**.

Pour ajouter le connecteur **Azure IoT Central v3 – Préversion** en tant qu’action dans Azure Logic Apps :

1. Dans **Concepteur d’applications logiques**, dans le panneau **Choisir une action**, sélectionnez l’onglet **Personnaliser**.
1. Recherchez *IoT Central*, puis sélectionnez le connecteur **Azure IoT Central v3 – Préversion**.
1. Dans la liste des actions, sélectionnez l’action IoT Central que vous souhaitez utiliser.
1. Dans l’étape d’action, terminez la configuration de l’action que vous avez choisie. Ensuite, sélectionnez **Enregistrer**.

:::image type="content" source="./media/howto-configure-rules-advanced/actions.png" alt-text="Rechercher le connecteur Azure IoT Central v3 et choisir une action":::

## <a name="list-of-actions"></a>Liste d’actions

La liste suivante affiche toutes les actions IoT Central disponibles dans le connecteur **Azure IoT Central v3 – Préversion** et leurs options de configuration. La plupart des champs peuvent avoir un contenu généré dynamiquement. Par exemple, une étape précédente peut déterminer l’identité d’appareil sur lequel l’étape actuelle agit.

### <a name="create-or-update-a-device"></a>Créer ou mettre à jour un appareil

Utilisez cette action pour créer ou mettre à jour un appareil dans votre application IoT Central.

| Champ | Description |
| ----- | ----------- |
| Application | Choisissez dans la liste des applications IoT Central. |
| Appareil | ID unique de l’appareil à créer ou mettre à jour. |
| Approved | Indiquez si l’appareil a été approuvé pour se connecter à IoT Central. |
| Description de l’appareil | Description détaillée de l’appareil. |
| Nom de l’appareil | Nom complet de l’appareil. |
| Modèle d'appareil | Choisissez dans la liste des modèles d’appareil dans votre application IoT Central. |
| Simulé | Indiquez si l’appareil est simulé. |

### <a name="delete-a-device"></a>Supprimer une unité

Utilisez cette action pour supprimer un appareil de votre application IoT Central.

| Champ | Description |
| ----- | ----------- |
| Application | Choisissez dans la liste des applications IoT Central. |
| Appareil | ID unique de l’appareil à supprimer. |

### <a name="execute-a-device-command"></a>Exécuter une commande d’appareil

Utilisez cette action pour exécuter une commande définie dans l’une des interfaces de l’appareil.

| Champ | Description |
| ----- | ----------- |
| Application | Choisissez dans la liste des applications IoT Central. |
| Appareil | ID unique de l’appareil à supprimer. |
| Composant d’appareil | Interface dans le modèle d’appareil qui contient la commande. |
| Commande d’appareil | Choisissez l’une des commandes de l’interface sélectionnée. |
| Modèle d'appareil | Choisissez dans la liste des modèles d’appareil dans votre application IoT Central. |
| Charge utile de demande de commande d’appareil | Si la commande requiert une charge utile de demande, ajoutez-la ici.  |

> [!NOTE]
> Vous ne pouvez pas choisir un composant d’appareil tant que vous n’avez pas choisi de modèle d’appareil.

### <a name="get-a-device-by-id"></a>Obtenir un appareil par ID

Utilisez cette action pour récupérer les détails de l’appareil.

| Champ | Description |
| ----- | ----------- |
| Application | Choisissez dans la liste des applications IoT Central. |
| Appareil | ID unique de l’appareil à supprimer. |

Vous pouvez utiliser les détails renvoyés dans les expressions dynamiques d’autres actions. Les détails de l’appareil retournés sont les suivants : **Approuvé**, **corps**, **Description de l’appareil**, **Nom de l’appareil**, **Modèle d’appareil**, **Approvisionné** et **Simulé**.

### <a name="get-device-cloud-properties"></a>Obtenir les propriétés cloud de l’appareil

Utilisez cette action pour récupérer les valeurs de propriété cloud pour un appareil spécifique.

| Champ | Description |
| ----- | ----------- |
| Application | Choisissez dans la liste des applications IoT Central. |
| Appareil | ID unique de l’appareil à supprimer. |
| Modèle d'appareil | Choisissez dans la liste des modèles d’appareil dans votre application IoT Central. |

Vous pouvez utiliser les valeurs de propriété cloud renvoyées dans les expressions dynamiques d’autres actions.

### <a name="get-device-properties"></a>Obtenir les propriétés de l’appareil

Utilisez cette action pour récupérer les valeurs de propriété pour un appareil spécifique.

| Champ | Description |
| ----- | ----------- |
| Application | Choisissez dans la liste des applications IoT Central. |
| Appareil | ID unique de l’appareil à supprimer. |
| Modèle d'appareil | Choisissez dans la liste des modèles d’appareil dans votre application IoT Central. |

Vous pouvez utiliser les valeurs de propriété renvoyées dans les expressions dynamiques d’autres actions.

### <a name="get-device-telemetry-value"></a>Obtenir la valeur de télémétrie de l’appareil

Utilisez cette action pour récupérer les valeurs de télémétrie pour un appareil spécifique.

| Champ | Description |
| ----- | ----------- |
| Application | Choisissez dans la liste des applications IoT Central. |
| Appareil | ID unique de l’appareil à supprimer. |
| Modèle d'appareil | Choisissez dans la liste des modèles d’appareil dans votre application IoT Central. |

Vous pouvez utiliser les valeurs de télémétrie renvoyées dans les expressions dynamiques d’autres actions.

### <a name="update-device-cloud-properties"></a>Mettre à jour les propriétés cloud de l’appareil

Utilisez cette action pour mettre à jour les valeurs de propriété cloud pour un appareil spécifique.

| Champ | Description |
| ----- | ----------- |
| Application | Choisissez dans la liste des applications IoT Central. |
| Appareil | ID unique de l’appareil à supprimer. |
| Modèle d'appareil | Choisissez dans la liste des modèles d’appareil dans votre application IoT Central. |
| Propriétés du cloud | Une fois que vous avez choisi un modèle d’appareil, un champ est ajouté pour chaque propriété cloud définie dans le modèle. |

### <a name="update-device-properties"></a>Mettre à jour les propriétés de l’appareil

Utilisez cette action pour mettre à jour les valeurs de propriété modifiables pour un appareil spécifique.

| Champ | Description |
| ----- | ----------- |
| Application | Choisissez dans la liste des applications IoT Central. |
| Appareil | ID unique de l’appareil à supprimer. |
| Modèle d'appareil | Choisissez dans la liste des modèles d’appareil dans votre application IoT Central. |
| Propriétés modifiables | Une fois que vous avez choisi un modèle d’appareil, un champ est ajouté pour chaque propriété modifiable définie dans le modèle. |

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez appris à créer une règle avancée dans votre application Azure IoT Central, vous pouvez découvrir comment [analyser les données de l’appareil dans votre application Azure IoT Central](howto-create-analytics.md).
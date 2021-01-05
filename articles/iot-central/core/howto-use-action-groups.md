---
title: Exécuter plusieurs actions à partir d’une règle Azure IoT Central | Microsoft Docs
description: Exécutez plusieurs actions à partir d’une même règle IoT Central et créez des groupes d’actions réutilisables que vous pouvez exécuter à partir de plusieurs règles.
services: iot-central
author: dominicbetts
ms.author: dobett
ms.date: 12/06/2019
ms.topic: how-to
ms.service: iot-central
manager: philmea
ms.openlocfilehash: b447f44d0c95693e560fd5bbfbff8c8daeec964e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80157685"
---
# <a name="group-multiple-actions-to-run-from-one-or-more-rules"></a>Regrouper plusieurs actions à exécuter à partir d’une ou plusieurs règles

*Cet article s’applique aux créateurs et aux administrateurs.*

Dans Azure IoT Central, vous créez des règles pour exécuter des actions quand une condition est satisfaite. Les règles sont basées sur la télémétrie des appareils ou sur des événements. Par exemple, vous pouvez avertir un opérateur quand la température d’un appareil dépasse un certain seuil. Cet article explique comment utiliser des *groupes d’actions* [Azure Monitor](../../azure-monitor/overview.md) pour attacher plusieurs actions à une règle IoT Central. Vous pouvez attacher un groupe d’actions à plusieurs règles. Un [groupe d’actions](../../azure-monitor/platform/action-groups.md) est une collection de préférences de notification définies par le propriétaire d’un abonnement Azure.

## <a name="prerequisites"></a>Prérequis

- Application créée avec un plan tarifaire Standard
- Un compte et un abonnement Azure pour créer et gérer des groupes d’actions Azure Monitor

## <a name="create-action-groups"></a>Créer des groupes d’actions

Vous pouvez [créer et gérer des groupes d’actions dans le portail Azure](../../azure-monitor/platform/action-groups.md) ou avec un [modèle Azure Resource Manager](../../azure-monitor/platform/action-groups-create-resource-manager-template.md).

Un groupe d’actions peut :

- Envoyer des notifications, comme un e-mail, un SMS, ou passer un appel vocal.
- Exécuter une action, comme appeler un webhook.

La capture d’écran suivante montre un groupe d’actions qui envoie des notifications par e-mail et par SMS, et qui appelle un webhook :

![Groupe d’actions](media/howto-use-action-groups/actiongroup.png)

Pour utiliser un groupe d’actions dans une règle IoT Central, le groupe d’actions doit être dans le même abonnement Azure que l’application IoT Central.

## <a name="use-an-action-group"></a>Utiliser un groupe d’actions

Pour utiliser un groupe d’actions dans votre application IoT Central, commencez par créer une règle. Quand vous ajoutez une action à la règle, sélectionnez **Groupes d’actions Azure Monitor** :

![Choisir une action](media/howto-use-action-groups/chooseaction.png)

Choisissez un groupe d’actions dans votre abonnement Azure :

![Choisir un groupe d’actions](media/howto-use-action-groups/chooseactiongroup.png)

Sélectionnez **Enregistrer**. Le groupe d’actions apparaît maintenant dans la liste des actions à exécuter quand la règle est déclenchée :

![Groupe d’actions enregistré](media/howto-use-action-groups/savedactiongroup.png)

Le tableau suivant récapitule les informations envoyées aux types d’actions pris en charge :

| Type d'action | Format de sortie |
| ----------- | -------------- |
| E-mail       | Modèle d’e-mail standard IoT Central |
| sms         | Alerte Azure IoT Central : ${nom_application} - « ${nom_règle} » déclenchée sur « ${nom_appareil} » le ${date_déclenchement} ${date_déclenchement} |
| Voix       | Alerte Azure I.O.T Central : règle « ${nom_règle} » déclenchée sur l’appareil « ${nom_appareil} » le ${date_déclenchement} ${date_déclenchement}, dans l’application ${nom_application} |
| webhook     | { « schemaId » : "AzureIoTCentralRuleWebhook", "data": {[charge utile normale du webhook](howto-create-webhooks.md#payload)}} |

Le texte suivant est un exemple de message SMS provenant d’un groupe d’actions :

`iotcentral: Azure IoT Central alert: Contoso - "Low pressure alert" triggered on "Motion sensor 2" at March 20, 2019 10:12 UTC`

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez découvert comment utiliser des groupes d’actions avec des règles, l’étape suivante suggérée est de découvrir comment [gérer vos appareils](howto-manage-devices.md).

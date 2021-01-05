---
title: Monitorage des déploiements IoT Edge – Azure IoT Edge
description: Monitorage de haut niveau, notamment les propriétés rapportées par edgeHub et edgeAgent ainsi que les métriques de déploiement automatique.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 04/21/2020
ms.topic: conceptual
ms.reviewer: veyalla
ms.service: iot-edge
ms.custom: devx-track-azurecli
services: iot-edge
ms.openlocfilehash: 4ff4d5a810eb79fb11e66591cd0b695062b1c9f6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91450176"
---
# <a name="monitor-iot-edge-deployments"></a>Superviser les déploiements IoT Edge

Azure IoT Edge fournit des rapports permettant d’effectuer un monitorage en temps réel des modules déployés sur les appareils IoT Edge. Le service IoT Hub récupère l’état des appareils et les met à la disposition de l’opérateur. Le monitorage est également important pour les [déploiements effectués à grande échelle](module-deployment-monitoring.md), notamment les déploiements automatiques et multicouches.

Dans la mesure où les appareils et les modules comportent des données similaires (par exemple, la connectivité), les valeurs sont récupérées en fonction de l’ID de l’appareil ou du module.

Le service IoT Hub collecte les données rapportées par les jumeaux d’appareil et de module, et fournit le nombre des différents états des appareils. Il organise ces données en quatre groupes de métriques :

| Type | Description |
| --- | ---|
| Ciblé | Affiche les appareils IoT Edge qui remplissent la condition de ciblage du déploiement. |
| Applied | Affiche les appareils IoT Edge ciblés, mais non ciblés par un autre déploiement de priorité plus élevée. |
| Réussite rapportée | Affiche les appareils IoT Edge ayant rapporté le bon déploiement des modules. |
| Échec rapporté | Affiche les appareils IoT Edge ayant rapporté l’échec de déploiement d’un ou plusieurs modules. Pour examiner l’erreur plus en détail, connectez-vous à distance à ces appareils et consultez les fichiers journaux. |

Le service IoT Hub met ces données à votre disposition sur le Portail Azure et dans Azure CLI.

## <a name="monitor-a-deployment-in-the-azure-portal"></a>Monitorage d’un déploiement sur le Portail Azure

Pour afficher les détails d’un déploiement et surveiller les appareils qui l’exécutent, effectuez les étapes suivantes :

1. Connectez-vous au [portail Azure](https://portal.azure.com) et accédez à votre hub IoT.
1. Sélectionnez **IoT Edge** dans le menu du volet gauche.
1. Sélectionnez l’onglet **Déploiements IoT Edge**.
1. Inspectez la liste des déploiements. Pour chaque déploiement, vous pouvez consulter les détails suivants :

    | Colonne | Description |
    | --- | --- |
    | id | Le nom du déploiement. |
    | Type | Type de déploiement : **Déploiement** ou **Déploiement multicouche**. |
    | Condition cible | Étiquette servant à définir les appareils ciblés. |
    | Priority | Numéro de priorité attribué au déploiement. |
    | Métriques du système | Nombre de jumeaux d’appareil dans IoT Hub remplissant la condition cible. **Appliqué** spécifie le nombre d’appareils dont les jumeaux de module se sont vu appliquer le contenu de déploiement dans IoT Hub. |
    | Métriques de l’appareil | Nombre d’appareils IoT Edge rapportant la réussite ou des erreurs du runtime du client IoT Edge. |
    | Métriques personnalisées | Nombre d’appareils IoT Edge rapportant des données pour les métriques définies dans le cadre du déploiement. |
    | Heure de création | Horodateur de création du déploiement. Cet horodatage sert à départager deux déploiements ayant la même priorité. |

1. Sélectionnez le déploiement que vous souhaitez surveiller.  
1. Sur la page **Détails du déploiement**, accédez à la section du bas, puis sélectionnez l’onglet **Condition cible**. Sélectionnez **Afficher** pour faire apparaître les appareils qui remplissent la condition cible. Vous pouvez modifier cette condition ainsi que la **Priorité**. Le cas échéant, sélectionnez **Enregistrer**.

   ![Affichage des appareils ciblés pour un déploiement](./media/how-to-monitor-iot-edge-deployments/target-devices.png)

1. Sélectionnez l’onglet **Métriques**. Si vous choisissez une métrique dans la liste déroulante **Sélectionner une métrique**, un bouton **Afficher** permettant d’afficher les résultats apparaît. Vous pouvez également sélectionner **Modifier les métriques** pour ajuster les critères des métriques personnalisées définies. Le cas échéant, sélectionnez **Enregistrer**.

   ![Affichage des métriques d’un déploiement](./media/how-to-monitor-iot-edge-deployments/deployment-metrics-tab.png)

Pour apporter des modifications à votre déploiement, consultez [Modification d’un déploiement](how-to-deploy-at-scale.md#modify-a-deployment).

## <a name="monitor-a-deployment-with-azure-cli"></a>Monitorage d’un déploiement avec Azure CLI

Utilisez la commande [az iot edge deployment show](/cli/azure/ext/azure-iot/iot/edge/deployment#ext-azure-iot-az-iot-edge-deployment-show) pour afficher les détails d’un déploiement unique :

```cli
az iot edge deployment show --deployment-id [deployment id] --hub-name [hub name]
```

La commande « deployment show » utilise les paramètres suivants :

* **--deployment-id** : nom du déploiement qui existe dans le hub IoT. Paramètre obligatoire.
* **--hub-name** : nom du hub IoT dans lequel le déploiement existe. Le hub doit être dans l’abonnement actuel. Basculez vers l’abonnement souhaité avec la commande `az account set -s [subscription name]`.

Inspectez le déploiement dans la fenêtre de commande. La propriété **metrics** répertorie un nombre pour chaque métrique évaluée par chaque hub :

* **targetedCount** : métrique système qui spécifie le nombre de jumeaux d’appareil dans le hub IoT qui correspondent à la condition de ciblage.
* **appliedCount** : métrique système qui spécifie le nombre d’appareils dont les jumeaux de module se sont vu appliquer le contenu de déploiement dans IoT Hub.
* **reportedSuccessfulCount** : métrique d’appareil qui spécifie le nombre d’appareils IoT Edge dans le déploiement indiquant une réussite de l’exécution du client IoT Edge.
* **reportedFailedCount** : métrique d’appareil qui spécifie le nombre d’appareils IoT Edge dans le déploiement indiquant un échec de l’exécution du client IoT Edge.

Vous pouvez afficher la liste des ID d’appareil ou des objets de chacune des métriques avec la commande [az iot edge deployment show-metric](/cli/azure/ext/azure-iot/iot/edge/deployment#ext-azure-iot-az-iot-edge-deployment-show-metric) :

```cli
az iot edge deployment show-metric --deployment-id [deployment id] --metric-id [metric id] --hub-name [hub name]
```

La commande « deployment show-metric » utilise les paramètres suivants :

* **--deployment-id** : nom du déploiement qui existe dans le hub IoT.
* **--metric-id** : nom de la métrique pour laquelle vous souhaitez voir la liste des ID d’appareil, par exemple `reportedFailedCount`.
* **--hub-name** : nom du hub IoT dans lequel le déploiement existe. Le hub doit être dans l’abonnement actuel. Basculez vers l’abonnement souhaité avec la commande `az account set -s [subscription name]`.

Pour apporter des modifications à votre déploiement, consultez [Modification d’un déploiement](how-to-deploy-cli-at-scale.md#modify-a-deployment).

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment [surveiller les jumeaux de module](how-to-monitor-module-twins.md), principalement l’agent IoT Edge et les modules d’exécution IoT Edge Hub, pour la connectivité et l’intégrité de vos déploiements IoT Edge.

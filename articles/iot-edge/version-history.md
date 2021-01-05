---
title: Historique et navigation dans la version d’IoT Edge – Azure IoT Edge
description: Découvrez les nouveautés d’IoT Edge avec des informations sur les nouvelles fonctionnalités et capacités des dernières versions.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 11/08/2020
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: cdb84f817f63e6401d17f18319e161f4c3477293
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95024651"
---
# <a name="azure-iot-edge-versions-and-release-notes"></a>Versions d’Azure IoT Edge et notes de publication

Azure IoT Edge est un produit créé à partir du projet open source IoT Edge hébergé sur GitHub. Toutes les nouvelles versions sont accessibles dans le [projet Azure IoT Edge](https://github.com/Azure/azure-iotedge). Les contributions et les rapports de bogues peuvent être effectués sur le [projet open source IoT Edge](https://github.com/Azure/iotedge).

## <a name="documented-versions"></a>Versions documentées

La documentation IoT Edge sur ce site est disponible pour deux versions différentes du produit, ce qui vous permet de choisir le contenu qui s’applique à votre environnement IoT Edge. Actuellement, les deux versions prises en charge sont les suivantes :

* **IoT Edge 1.0.10** couvre toutes les fonctionnalités et capacités par le biais de la toute dernière version mise à la disposition générale : [1.0.10](https://github.com/Azure/azure-iotedge/releases/tag/1.0.10).
* **IoT Edge 1.2 (préversion)** contient du contenu supplémentaire pour les fonctionnalités et les capacités qui se trouvent dans la préversion la plus récente : [1.2-rc1](https://github.com/Azure/azure-iotedge/releases/tag/1.2.0-rc1).
  * Si IoT Edge 1.2 est en préversion, vous devez installer les versions Release Candidate. Pour plus d’informations, consultez [Installation hors connexion ou d’une version spécifique](how-to-install-iot-edge.md?tabs=linux#offline-or-specific-version-installation).

## <a name="version-history"></a>Historique des versions

Ce tableau fournit l’historique des versions récentes des packages IoT Edge et met en évidence les mises à jour de la documentation effectuées pour chaque version.

| Notes de publication et ressources | Type | Date | Points forts |
| ------------------------ | ---- | ---- | ---------- |
| [1.2-rc1](https://github.com/Azure/azure-iotedge/releases/tag/1.2.0-rc1) | PRÉVERSION | Novembre 2020 | [Appareils IoT Edge derrière des passerelles](how-to-connect-downstream-iot-edge-device.md?view=iotedge-2020-11&preserve-view=true)<br>[Répartiteur MQTT IoT Edge](how-to-publish-subscribe.md?view=iotedge-2020-11&preserve-view=true) |
| [1.0.10](https://github.com/Azure/azure-iotedge/releases/tag/1.0.10) | Stable | Octobre 2020 | [Méthode directe UploadSupportBundle](how-to-retrieve-iot-edge-logs.md#upload-support-bundle-diagnostics)<br>[Charger des métriques de runtime](how-to-access-built-in-metrics.md)<br>[Priorité de routage et durée de vie](module-composition.md#priority-and-time-to-live)<br>[Ordre de démarrage du module](module-composition.md#configure-modules)<br>[Approvisionnement manuel X.509](how-to-manual-provision-x509.md) |
| [1.0.9](https://github.com/Azure/azure-iotedge/releases/tag/1.0.9) | Stable | Mars 2020 | [Approvisionnement automatique X.509 avec DPS](how-to-auto-provision-x509-certs.md)<br>[Méthode directe RestartModule](how-to-edgeagent-direct-method.md#restart-module)<br>[Commande support-bundle](troubleshoot.md#gather-debug-information-with-support-bundle-command) |

## <a name="next-steps"></a>Étapes suivantes

* [Afficher toutes les versions d’Azure IoT Edge](https://github.com/Azure/azure-iotedge/releases)
* [Créer ou passer en revue les demandes de fonctionnalités sur le forum de commentaires](https://feedback.azure.com/forums/907045-azure-iot-edge)
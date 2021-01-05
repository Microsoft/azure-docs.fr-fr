---
title: Tarifs et coûts associés
description: Découvrez les coûts associés à Defender pour IoT et la manière dont vous pouvez les contrôler.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: shhazam-ms
manager: rkarlin
editor: ''
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/08/2020
ms.author: shhazam
ms.openlocfilehash: a97bcbf5ba47289a2e68b0eaa587ea39d7fb705a
ms.sourcegitcommit: 8be279f92d5c07a37adfe766dc40648c673d8aa8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/31/2020
ms.locfileid: "97832350"
---
# <a name="pricing-and-associated-costs"></a>Tarifs et coûts associés

Cet article présente le modèle de tarification de Defender pour IoT, récapitule tous les coûts associés et explique comment les gérer.

## <a name="pricing"></a>Tarifs

Le modèle de tarification de Defender pour IoT se compose de deux parties, et la facturation intervient une fois qu'un IoT Hub est [activé](quickstart-onboard-iot-hub.md) dans Defender pour IoT :

- Coût par appareil : fonctionnalités de sécurité intégrées basées sur l’analyse des journaux IoT Hub.

- Coût par message : fonctionnalités de sécurité améliorées basées sur les messages de sécurité à partir d’appareils IoT Edge ou leaf.

Pour plus d’informations, consultez la [tarification de Security Center](https://azure.microsoft.com/pricing/details/security-center/).

## <a name="associated-costs"></a>Coûts associés

Certains coûts, qui ne font pas partie de la tarification directe, sont associés à Defender pour IoT :

- Coûts de stockage Log Analytics

Vous pouvez réduire les coûts associés en refusant certaines fonctionnalités de la solution. Vous devez, pour cela, changer vos paramètres.

Pour changer vos paramètres :

1. Ouvrez IoT Hub.

1. Sous **Sécurité**, cliquez sur **Paramètres**.

1. Cliquez sur **Collecte de données**.

Le tableau suivant fournit un récapitulatif des coûts associés et des implications de chaque option.

| Option | Usage | Commentaire |
| --- | --- | --- |
| **Stockage Log Analytics** |  |
| Alertes et recommandations de l’appareil| Alertes et recommandations de sécurité générées par le service | Obligatoire |
| Données de sécurité brutes| Données de sécurité brutes des appareils IoT, collectées par des agents de sécurité | Désactiver le _stockage d’événements bruts de sécurité d’appareil_ |
|

>[!Important]
> La désactivation a des conséquences graves sur la disponibilité des fonctionnalités de sécurité de Defender pour IoT.

| Désactiver | Implications |
| --- | --- |
| _Collection de métadonnées de représentations_ | Désactiver les [alertes personnalisées](quickstart-create-custom-alerts.md) |
| | Désactiver les recommandations de manifeste IoT Edge |
| | Désactiver les alertes et les recommandations basées sur l’identité d’appareil |
| _Stocker les événements bruts de sécurité d’appareil_ | Les détails des recommandations de base du système d’exploitation d’appareil ne sont pas disponibles |
| | Les détails sur les enquêtes relatives à [l’alerte](concept-security-alerts.md) et à la [recommandation](concept-recommendations.md) ne sont pas disponibles |
|

## <a name="see-also"></a>Voir aussi

- Accéder à vos [données de sécurité brutes](how-to-security-data-access.md)
- [Investiguer un appareil](how-to-investigate-device.md)
- Comprendre et explorer les [recommandations de sécurité](concept-recommendations.md)
- Comprendre et explorer les [alertes de sécurité](concept-security-alerts.md)

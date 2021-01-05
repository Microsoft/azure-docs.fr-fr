---
title: Concepts d’architecture dans Azure IoT Central - Compteur intelligent | Microsoft Docs
description: Cet article présente les concepts clés relatifs à l’architecture du modèle d’application d’énergie Azure IoT Central
author: op-ravi
ms.author: omravi
ms.date: 12/11/2020
ms.topic: overview
ms.service: iot-central
services: iot-central
manager: abjork
ms.openlocfilehash: f9167df3ce56116e02addec411d5f720ea366ea0
ms.sourcegitcommit: 66479d7e55449b78ee587df14babb6321f7d1757
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97516663"
---
# <a name="azure-iot-central---smart-meter-app-architecture"></a>Azure IoT Central - Architecture d’application pour compteurs intelligents

Cet article fournit une vue d’ensemble de l’architecture du modèle d’application de suivi des compteurs intelligents. Le diagramme ci-dessous illustre une architecture couramment utilisée pour l’application de compteur intelligent sur Azure à l’aide de la plateforme IoT Central.

> [!div class="mx-imgBorder"]
> ![architecture pour panneau solaire](media/concept-iot-central-smart-meter/smart-meter-app-architecture.png)

Cette architecture est constituée des composants suivants. Certaines solutions peuvent ne pas avoir besoin de tous les composants listés ici.

## <a name="smart-meters-and-connectivity"></a>Compteurs intelligents et connectivité 

Un compteur intelligent est un des appareils les plus importants parmi l’ensemble des ressources énergétiques. Il enregistre et communique les données de consommation d’énergie aux services publics pour le suivi et d’autres cas d’usage, tels que la facturation et la réponse à la demande. Selon le type du compteur, il peut se connecter à IoT Central via des passerelles ou d’autres appareils ou systèmes intermédiaires, tels que des appareils de périmètre et des systèmes de tête de réseau. Créez un pont d’appareil IoT Central pour connecter les appareils qui ne peuvent pas être connectés directement. Le pont IoT Central est une solution open source ; vous pouvez trouver des informations complètes à son sujet [ici](../core/howto-build-iotc-device-bridge.md). 

## <a name="iot-central-platform"></a>Plateforme IoT Central

Azure IoT Central est une plateforme qui simplifie la création de votre solution IoT et contribue à réduire la charge et les coûts liés à la gestion, aux opérations et au développement IoT. Avec IoT Central, vous pouvez facilement connecter, surveiller et gérer facilement vos ressources Internet des objets (IoT) à grande échelle. Une fois que vous avez connecté vos compteurs intelligents à IoT Central, le modèle d’application utilise des fonctionnalités intégrées, comme des modèles d’appareil, des commandes et des tableaux de bord. Le modèle d’application utilise également le stockage IoT Central pour les scénarios de chemin à chaud tels que la surveillance, l’analyse, les règles et la visualisation des données de mesure en temps quasi-réel. 

## <a name="extensibility-options-to-build-with-iot-central"></a>Options d’extensibilité pour créer avec IoT Central
La plateforme IoT Central fournit deux options d’extensibilité : L’exportation de données continue (CDE) et des API. Les clients et partenaires peuvent choisir entre ces options pour personnaliser leurs solutions selon leurs besoins spécifiques. Par exemple, un de nos partenaires a configuré CDE avec Azure Data Lake Storage (ADLS). Ils utilisent ADLS pour la conservation à long terme des données et d’autres scénarios de stockage à froid, tels que le traitement par lots, l’audit et la création de rapports. 

## <a name="next-steps"></a>Étapes suivantes

* Maintenant que vous avez découvert l’architecture, [créez une application de compteur intelligent gratuitement](https://apps.azureiotcentral.com/build/new/smart-meter-monitoring)
* Pour plus d’informations sur IoT Central, consultez la [vue d’ensemble d’IoT Central](../index.yml)

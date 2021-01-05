---
title: Activer le mode aperçu pour Azure IoT Hub
description: Découvrez comment activer le mode aperçu pour IoT Hub, pourquoi il peut vous intéresser et quelques avertissements
services: iot-hub
author: jlian
ms.service: iot-fundamentals
ms.topic: conceptual
ms.date: 11/24/2020
ms.author: jlian
ms.openlocfilehash: 864870c4392b12477c321c86afd9da848120490c
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96621703"
---
# <a name="turn-on-preview-mode-for-iot-hub-to-try-select-new-features"></a>Activer le mode aperçu pour IoT Hub afin d’essayer de sélectionner de nouvelles fonctionnalités

<!-- 
- We are working hard to bring you new features
- Some of these features require a brand new iot hub with preview mode on
- some features may not work at all or have unexpected behavior
- "Normal preview features" do NOT require preview mode 
- Support opt-in at creation time only
- Customer cannot opt back out post creation
- If customer wants to evaluate, they must use new hub dedicated for the preview
- Banners, documentations and all materials indicate preview quality: no GA guarantee at all
-->

De nouvelles fonctionnalités sont disponibles en préversion publique pour IoT Hub, notamment :

- MQTT 5
- Certificat de serveur ECC
- Négociation de la longueur des fragments TLS
- Prise en charge de la chaîne d’autorité de certification X.509 pour HTTPS/WebSocket

Ces fonctionnalités sont des améliorations du protocole IoT Hub et des couches d’authentification, de sorte qu’elles sont disponibles uniquement pour le **nouveau** hub IoT pour l’instant. Elles *ne sont pas* encore disponibles pour les hubs IoT existants. Pour afficher un aperçu de ces fonctionnalités, l’hub IoT doit être créé avec le mode aperçu activé.

## <a name="turn-on-preview-mode-for-a-new-iot-hub"></a>Activer le mode aperçu pour un nouveau hub IoT

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Dans la page d’accueil Azure, sélectionnez le bouton **+ Créer une ressource**, puis entrez *IoT Hub* dans le champ **Rechercher dans la Place de marché**.

1. Sélectionnez **IoT Hub** dans les résultats de la recherche, puis **Créer**.

1. Sous l’onglet **Général**, renseignez les champs [comme vous le feriez normalement](iot-hub-create-through-portal.md), sauf pour **Région**. Sélectionnez l’une des régions suivantes :
    
    - USA Centre
    - Europe Ouest
    - Asie Sud-Est

1. Sélectionnez l’onglet **Gestion**, puis développez la section **Paramètres avancés**.

1. À côté de **Mode aperçu**, sélectionnez **Activé**. Examinez attentivement le texte d’avertissement.

    :::image type="content" source="media/iot-hub-preview-mode/turn-on-preview-mode-at-create.png" alt-text="Image qui indique où sélectionner l’option en mode aperçu lors de la création d’un hub IoT":::

1. Sélectionnez **Suivant : Vérifier + créer**, puis **Créer**.

Une fois créé, un hub IoT en mode aperçu affiche toujours cette bannière, ce qui vous permet d’utiliser ce hub IoT à des fins de prévisualisation uniquement : 

:::image type="content" source="media/iot-hub-preview-mode/banner.png" alt-text="Image représentant la bannière pour le mode aperçu du hub IoT":::

## <a name="using-an-iot-hub-in-preview-mode"></a>Utilisation d’un hub IoT en mode aperçu

N’utilisez *pas* de hub IoT en mode aperçu pour la production. Le mode de prévisualisation est *uniquement* destiné à prévisualiser les fonctionnalités sélectionnées figurant en haut de cette page. Les autres limitations du mode de prévisualisation IoT Hub mode aperçu sont les suivantes :

- Certaines fonctionnalités IoT Hub existantes, telles que le filtre IP, la liaison privée, l’identité managée, les flux d’appareils et le basculement, peuvent fonctionner de manière inattendue ou pas du tout.
- Un hub IoT en mode aperçu ne peut pas être modifié ou mis à niveau vers un hub IoT normal.
- Nous ne pouvons pas garantir le [contrat de niveau de service IoT Hub](https://azure.microsoft.com/support/legal/sla/iot-hub/v1_2/) standard. Ne l’utilisez donc pas pour la production.

> [!TIP]
> Le mode aperçu n’est pas requis pour les [flux d’appareils](iot-hub-device-streams-overview.md) et le [suivi distribué](iot-hub-distributed-tracing.md). Pour utiliser ces anciennes fonctionnalités d’évaluation, suivez leur documentation comme d’habitude. 

## <a name="next-steps"></a>Étapes suivantes

- Pour afficher un aperçu de la prise en charge de MQTT 5, consultez [Vue d’ensemble de la prise en charge de MQTT 5 d’Azure IoT Hub (préversion)](iot-hub-mqtt-5.md).
- Pour afficher un aperçu du certificat de serveur ECC, consultez [Certificat TLS du serveur ECC (préversion)](iot-hub-tls-support.md#elliptic-curve-cryptography-ecc-server-tls-certificate-preview).
- Pour afficher un aperçu de la négociation de la taille des fragments TLS, consultez [Négociation de la longueur maximale des fragments TLS (préversion)](iot-hub-tls-support.md#tls-maximum-fragment-length-negotiation-preview).
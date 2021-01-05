---
title: Configurer les protocoles d’API – Azure Event Grid IoT Edge | Microsoft Docs
description: Découvrez les configurations de protocole possibles d’un module Event Grid.
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.date: 07/08/2020
ms.topic: article
ms.openlocfilehash: a9cf6088201ffeaed76d99a9b211e5bcd1ea139a
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91322578"
---
# <a name="configure-event-grid-api-protocols"></a>Configurer des protocoles d’API Event Grid

Ce guide donne des exemples de configurations de protocole possibles d’un module Event Grid. Le module Event Grid expose l’API pour ses opérations de gestion et d’exécution. Le tableau suivant capture les protocoles et les ports.

| Protocol | Port | Description |
| ---------------- | ------------ | ------------ |
| HTTP | 5888 | Désactivé par défaut. Utile uniquement pendant le test. Non adapté aux charges de travail de production.
| HTTPS | 4438 | Default

Consultez le guide [Sécurité et authentification](security-authentication.md) pour toutes les configurations possibles.

## <a name="expose-https-to-iot-modules-on-the-same-edge-network"></a>Exposer HTTPS aux modules IoT sur le même réseau de périphérie

```json
 {
  "Env": [
    "inbound__serverAuth__tlsPolicy=strict",
    "inbound__serverAuth__serverCert__source=IoTEdge"
  ]
}
 ```

## <a name="enable-https-to-other-iot-modules-and-non-iot-workloads"></a>Activer HTTPS pour d’autres modules IoT et non IoT

```json
 {
  "Env": [
    "inbound__serverAuth__tlsPolicy=strict",
    "inbound__serverAuth__serverCert__source=IoTEdge"
  ],
  "HostConfig": {
    "PortBindings": {
      "4438/tcp": [
        {
          "HostPort": "4438"
        }
      ]
    }
  }
}
 ```

>[!NOTE]
> La section **PortBindings** vous permet de mapper les ports internes aux ports de l’hôte conteneur. Cette fonctionnalité permet d’accéder au module Event Grid depuis l’extérieur du réseau de conteneurs IoT Edge si l’appareil IoT est accessible au public.

## <a name="expose-http-and-https-to-iot-modules-on-the-same-edge-network"></a>Exposer HTTP et HTTPS aux modules IoT sur le même réseau de périphérie

```json
 {
  "Env": [
    "inbound__serverAuth__tlsPolicy=enabled",
    "inbound__serverAuth__serverCert__source=IoTEdge"
  ]
}
 ```

## <a name="enable-http-and-https-to-other-iot-modules-and-non-iot-workloads"></a>Activer HTTP et HTTPS pour d’autres modules IoT et non IoT

```json
 {
  "Env": [
    "inbound__serverAuth__tlsPolicy=enabled",
    "inbound__serverAuth__serverCert__source=IoTEdge"
  ],
  "HostConfig": {
    "PortBindings": {
      "4438/tcp": [
        {
          "HostPort": "4438"
        }
      ],
      "5888/tcp": [
        {
          "HostPort": "5888"
        }
      ]
    }
  }
}
 ```

>[!NOTE]
> Par défaut, chaque module IoT fait partie du runtime IoT Edge créé par le réseau de pont. Il permet à des modules IoT de communiquer entre eux sur le même réseau. **PortBindings** vous permet de mapper un port interne de conteneur sur la machine hôte, permettant ainsi à quiconque d’accéder au port du module Event Grid depuis l’extérieur.

>[!IMPORTANT]
> Tandis que les ports peuvent être rendus accessibles en dehors du réseau IoT Edge, l’authentification du client stipule qui est effectivement autorisé à effectuer des appels dans le module.

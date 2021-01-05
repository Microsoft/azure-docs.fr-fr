---
title: Présentation d’OPC Twin - Azure | Microsoft Docs
description: Cet article fournit une vue d’ensemble d’OPC Twin. OPC Twin fournit la découverte, l’inscription et le contrôle à distance des appareils industriels par le biais d’API REST.
author: dominicbetts
ms.author: dobett
ms.date: 11/26/2018
ms.topic: overview
ms.service: industrial-iot
services: iot-industrialiot
manager: philmea
ms.openlocfilehash: 9825e5e85c8da27b0f3fc2b9d78fcc9ca3513389
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91281692"
---
# <a name="what-is-opc-twin"></a>Définition d’OPC Twin

> [!IMPORTANT]
> Pendant la mise à jour de cet article, consultez [Azure Industrial IoT](https://azure.github.io/Industrial-IoT/) pour obtenir le contenu le plus récent.

OPC Twin se compose de microservices qui utilisent Azure IoT Edge et IoT Hub pour connecter le cloud et le réseau d’usine. OPC Twin fournit la découverte, l’inscription et le contrôle à distance des appareils industriels par le biais d’API REST. OPC Twin ne nécessite pas de SDK OPC UA (OPC Unified Architecture), est indépendant du langage de programmation et peut être inclus dans un workflow serverless. Cet article décrit plusieurs cas d’usage d’OPC Twin.

## <a name="discovery-and-control"></a>Découverte et contrôle
Vous pouvez utiliser OPC Twin pour une découverte et une inscription simples.

### <a name="simple-discovery-and-registration"></a>Découverte et inscription simples
OPC Twin permet aux opérateurs en usine d’analyser le réseau d’usine, afin que les serveurs OPC UA puissent être découverts et inscrits. En guise d’alternative, les opérateurs en usine peuvent également inscrire manuellement les appareils OPC UA à l’aide d’une URL de découverte connue. Par exemple, pour se connecter à tous les appareils OPC UA une fois que la passerelle IoT Edge dotée d’un module OPC Twin a été installée en usine, l’opérateur en usine peut déclencher à distance une analyse du réseau et voir tous les serveurs OPC UA. 

### <a name="simple-control"></a>Contrôle simple
OPC Twin permet aux opérateurs en usine de réagir aux événements et de reconfigurer leurs machines au niveau de l’atelier à partir du cloud soit automatiquement, soit manuellement à la volée. OPC Twin fournit des API REST pour appeler des services sur le serveur OPC UA, parcourir son espace d’adressage ainsi que lire/écrire des variables et exécuter des méthodes. Par exemple, une chaudière utilise un KPI de température pour contrôler la ligne de production. Le capteur de température publie la modification des données à l’aide du serveur de publication OPC. L’opérateur en usine reçoit l’alerte lui indiquant que la température a atteint le seuil. La ligne de production refroidit automatiquement par le biais d’OPC Twin. L’opérateur en usine est averti du refroidissement.

## <a name="authentication"></a>Authentication
Vous pouvez utiliser OPC Twin pour une authentification simple et une expérience de développement simple.

### <a name="simple-authentication"></a>Authentification simple 
OPC Twin utilise l’authentification basée sur Azure Active Directory (AAD) et l’audit de bout en bout. Par exemple, vous pouvez générer l’application sur OPC Twin afin de déterminer ce qu’a effectué un opérateur sur une machine. Côté machine, la procédure recourt à l’audit OPC UA. Côté cloud, elle passe par le stockage d’un journal d’audit client immuable et l’authentification AAD sur l’API REST.

### <a name="simple-developer-experience"></a>Expérience de développement simple 
OPC Twin peut être utilisé avec des applications écrites dans n’importe quel langage de programmation par le biais d’API REST. Quand un développeur intègre un client OPC UA à une solution, il n’a pas besoin de connaître le SDK OPC UA. OPC Twin peut s’intégrer sans difficulté à des architectures serverless sans état. Par exemple, un développeur web « full stack » qui développe une application pour un tableau de bord d’alarmes et d’événements peut écrire la logique permettant de répondre aux événements en JavaScript ou TypeScript à l’aide d’OPC Twin sans connaître C, C# ou l’implémentation de la pile complète d’OPC UA. 

## <a name="next-steps"></a>Étapes suivantes

OPC Twin et ses utilisations n’ayant plus de secrets pour vous, voici l’étape suivante que nous suggérons :

[Définition d’OPC Vault](overview-opc-vault.md)
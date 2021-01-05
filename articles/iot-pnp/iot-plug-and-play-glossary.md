---
title: Glossaire terminologique – IoT Plug-and-Play | Microsoft Docs
description: 'Concepts : glossaire des termes courants relatifs à IoT Plug-and-Play.'
author: dominicbetts
ms.author: dobett
ms.date: 07/22/2020
ms.topic: conceptual
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: d44866e2d04ab1bab5d2eca01374350a7d73a0ea
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91577337"
---
# <a name="glossary-of-terms-for-iot-plug-and-play"></a>Glossaire terminologique pour IoT Plug-and-Play

Définitions des termes courants utilisés dans les articles sur IoT Plug-and-Play.

## <a name="azure-iot-explorer-tool"></a>Outil Explorateur Azure IoT

L’explorateur Azure IoT est un outil graphique que vous pouvez utiliser pour interagir avec vos [appareils IoT Plug-and-Play](#iot-plug-and-play-device) et les tester. Après avoir installé l’outil sur votre machine locale, vous pouvez l’utiliser pour :

- Voir les appareils connectés à votre [hub IoT](#azure-iot-hub).
- Vous connecter un appareil IoT Plug-and-Play.
- Affichez les [composants](#component) de l’appareil.
- Voir la [télémétrie](#telemetry) envoyée par l’appareil.
- Utiliser des [propriétés](#properties) de l’appareil.
- Appeler des [commandes](#commands) de l’appareil.

## <a name="azure-iot-hub"></a>Azure IoT Hub

IoT Hub est un service managé, hébergé dans le cloud, qui agit en tant que concentrateur de messages central pour la communication bidirectionnelle entre votre application IoT et les appareils qu’il gère. Les [appareils IoT Plug-and-Play](#iot-plug-and-play-device) peuvent se connecter à un hub IoT. Une solution IoT utilise un hub IoT pour :

- Permettre aux appareils d’envoyer de la télémétrie à une solution cloud.
- Permettre à une solution cloud de gérer les appareils connectés.

## <a name="azure-iot-device-sdk"></a>Azure IoT device SDK

Il existe des SDK d’appareil pour plusieurs langages, que vous pouvez utiliser pour créer des applications clientes d’appareil IoT Plug-and-Play. Utilisez **DeviceClient** pour des appareils et **ModuleClient** pour des modules et les modules IoT Edge.

## <a name="commands"></a>Commandes

Les commandes définies dans une [interface](#interface) représentent les méthodes qui peuvent être exécutées sur le [jumeau numérique](#digital-twin). Par exemple, une commande pour redémarrer un appareil.

## <a name="component"></a>Composant

Les composants permettent de créer une [interface](#interface) de modèle qui représente un assemblage d’autres interfaces. Un [modèle d’appareil](#device-model) peut combiner plusieurs interfaces sous forme de composants (par exemple, un modèle comportant un composant de commutateur et un composant de thermostat). Il peut également arriver que plusieurs composants d’un modèle utilisent le même type d’interface (par exemple, un modèle comportant deux composants de thermostat).

## <a name="connection-string"></a>Chaîne de connexion

Une chaîne de connexion encapsule les informations nécessaires pour se connecter à un point de terminaison. Une chaîne de connexion inclut généralement l’adresse du point de terminaison et des informations de sécurité, mais les formats de chaîne de connexion varient selon les services. Il existe deux types de chaîne de connexion associés au service IoT Hub :

- Les chaînes de connexion d’appareil permettent aux [appareils IoT Plug-and-Play](#iot-plug-and-play-device) de se connecter aux points de terminaison visibles par l’appareil sur un hub IoT. Le code client sur un appareil utilise la chaîne de connexion pour établir une connexion sécurisée avec un hub IoT.
- Les chaînes de connexion IoT Hub permettent aux applications back-end de se connecter aux points de terminaison visibles par le service sur un hub IoT. Ces solutions et ces outils gèrent le hub IoT et les appareils qui y sont connectés.

## <a name="default-component"></a>Composant par défaut

Tous les [modèles d’appareil](#device-model) ont un composant par défaut. Un modèle d’appareil simple n’a qu’un composant par défaut ; ce type de modèle est également appelé « appareil sans composant ». Un modèle plus complexe comporte plusieurs composants imbriqués sous le composant par défaut.

## <a name="device-certification"></a>Certification d’appareil

Le programme de certification d’appareil IoT Plug-and-Play vérifie qu’un appareil répond aux exigences de certification IoT Plug-and-Play. Vous pouvez ajouter un appareil certifié au [catalogue des appareils certifiés pour Azure IoT](https://aka.ms/devicecatalog).

## <a name="device-model"></a>Modèle de l'appareil

Un modèle d’appareil décrit un [appareil IoT Plug-and-Play](#iot-plug-and-play-device) et en définit les [composants](#component). Un modèle d’appareil simple, qui ne comporte pas de composants distincts, contient une définition pour une seule interface. L’outil Explorateur Azure IoT montre un modèle simple, avec un seul [composant par défaut](#default-component).

Un modèle d’appareil plus complexe englobe plusieurs composants. Un modèle d’appareil correspond généralement à un appareil , à un produit ou à une référence SKU physique. On utilise généralement la [version 2 du langage DTDL (Digital Twin Definition Language)](#digital-twins-definition-language) pour définir un modèle d’appareil.

## <a name="device-builder"></a>Générateur d’appareils

Les générateurs d’appareils utilisent un [modèle d’appareil](#device-model) et des [interfaces](#interface) lors de l’implémentation du code à exécuter sur un [appareil IoT Plug-and-Play](#iot-plug-and-play-device). Ils s’appuient généralement sur l’un des [kits de développement logiciel (SDK) Azure IoT device](#azure-iot-device-sdk) pour implémenter le client d’appareil, mais ce n’est pas obligatoire.

## <a name="device-modeling"></a>Modélisation d’appareil

Un [générateur d’appareils](#device-builder) ou un [générateur de modules](#module-builder) utilise le [langage DTDL (Digital Twins Definition Language)](#digital-twins-definition-language) pour modéliser les capacités d’un [appareil IoT Plug-and-Play](#iot-plug-and-play-device). Un [générateur de solutions](#solution-builder) peut configurer une solution IoT à partir du modèle.

## <a name="digital-twin"></a>Jumeau numérique

Un jumeau numérique est un modèle d’un [appareil IoT Plug-and-Play](#iot-plug-and-play-device). Les jumeaux numériques sont modélisés avec le [langage DTDL (Digital Twins Definition Language)](#digital-twins-definition-language). Vous pouvez utiliser les [SDK d’appareil Azure IoT](#azure-iot-device-sdk) pour interagir avec des jumeaux numériques à l’exécution. Par exemple, vous pouvez définir une valeur de propriété dans un jumeau numérique sur un appareil : le SDK communique alors cette modification à votre solution IoT dans le cloud.

## <a name="digital-twin-change-events"></a>Événements de changement de jumeau numérique

Quand un [appareil IoT Plug-and-Play](#iot-plug-and-play-device) est connecté à un [hub IoT](#azure-iot-hub), le hub peut utiliser ses capacités de routage pour envoyer des notifications de changements de jumeau numérique. Par exemple, chaque fois que la valeur d’une [propriété](#properties) change sur un appareil, IoT Hub peut envoyer une notification à un point de terminaison, comme un hub Event Hub.

## <a name="digital-twins-definition-language"></a>Langage DTDL (Digital Twins Definition Language)

Langage de description des modèles et des interfaces pour les [appareils IoT Plug-and-Play](#iot-plug-and-play-device). Utilisez la [version 2 du langage DTDL (Digital Twins Definition Language)](https://github.com/Azure/opendigitaltwins-dtdl) pour décrire les capacités d’un [jumeau numérique](#digital-twin) et permettre à la plateforme IoT et aux solutions IoT de tirer parti de la sémantique de l’entité.

## <a name="digital-twin-route"></a>Route de jumeau numérique

Il s’agit d’un itinéraire configuré dans un [hub IoT](#azure-iot-hub) pour remettre des [événements de modification de jumeau numérique](#digital-twin-change-events) à un point de terminaison, comme un hub Event Hub.

## <a name="interface"></a>Interface

Une interface décrit les capacités associées qui sont implémentées par un [appareil IoT Plug-and-Play](#iot-plug-and-play-device) ou par un [jumeau numérique](#digital-twin). Il est possible de réutiliser des interfaces sur différents [modèles d’appareil](#device-model). Quand une interface est employée dans un modèle d’appareil, elle définit un [composant](#component) de l’appareil. Un appareil simple contient seulement une interface par défaut.

## <a name="iot-hub-query-language"></a>Langage de requête IoT Hub

Le langage de requête IoT Hub est utilisé dans plusieurs objectifs. Par exemple, vous pouvez vous en servir pour rechercher des appareils inscrits auprès de votre hub IoT ou affiner le comportement [d’acheminement des jumeaux numériques](#digital-twin-route).

## <a name="iot-plug-and-play-bridge"></a>Pont IoT Plug-and-Play

Le pont IoT Plug-and-Play est une application open source permettant à des capteurs ou périphériques attachés à des passerelles Windows ou Linux de se connecter en tant qu’[appareils IoT Plug-and-Play](#iot-plug-and-play-device).

## <a name="iot-plug-and-play-device"></a>Appareil IoT Plug-and-Play

Un appareil IoT Plug-and-Play est généralement un petit appareil informatique autonome qui collecte des données ou contrôle d’autres appareils, et exécute un logiciel ou un microprogramme implémentant un [modèle d’appareil](#device-model).  Par exemple, un appareil IoT Plug-and-Play peut être un appareil de supervision de l’environnement ou un contrôleur pour un système d’irrigation agricole intelligent. Un appareil IoT Plug-and-Play peut être implémenté directement ou en tant que module IoT Edge. Vous pouvez écrire des solutions IoT hébergées dans le cloud pour commander, contrôler et recevoir les données provenant d’appareils IoT Plug-and-Play.

## <a name="iot-plug-and-play-conventions"></a>Conventions IoT Plug-and-Play

Les [appareils](#iot-plug-and-play-device) IoT Plug-and-Play doivent suivre un ensemble de [conventions](concepts-convention.md) lorsqu’ils échangent des données avec une solution.

## <a name="model-id"></a>ID de modèle

Lorsqu’un appareil IoT Plug-and-Play se connecte à un hub IoT, il envoie **l’ID du modèle** [DTDL](#digital-twins-definition-language) qu’il implémente, ce qui permet à la solution de trouver le modèle d’appareil.

## <a name="model-repository"></a>Référentiel de modèles

Un [référentiel de modèles](concepts-model-repository.md) stocke des [modèles d’appareil](#device-model) et des [interfaces](#interface).

## <a name="model-repository-rest-api"></a>API REST de référentiel de modèles

Il s’agit d’une API permettant de gérer les référentiels de modèles et d’interagir avec eux. Par exemple, vous pouvez l’utiliser pour ajouter et rechercher des [modèles d’appareil](#device-model).

## <a name="module-builder"></a>Générateur de modules

Un générateur de modules utilise un [modèle d’appareil](#device-model) et des [interfaces](#interface) lors de l’implémentation du code à exécuter sur un [appareil IoT Plug-and-Play](#iot-plug-and-play-device). Les générateurs de modules implémentent le code sous la forme d’un module ou d’un module IoT Edge à déployer sur le runtime IoT Edge d’un appareil.

## <a name="properties"></a>Propriétés

Les propriétés sont des champs de données définis dans une [interface](#interface). Ils représentent un état d’un jumeau numérique. Vous pouvez déclarer des propriétés comme étant accessibles en lecture seule ou en écriture. Les propriétés en lecture seule, comme le numéro de série, sont définies par du code s’exécutant sur l’[appareil IoT Plug-and-Play](#iot-plug-and-play-device) lui-même.  Les propriétés accessibles en écriture, comme un seuil d’alarme, sont généralement définies à partir de la solution IoT cloud.

## <a name="shared-access-signature"></a>Signature d’accès partagé

Les signatures d’accès partagé sont un mécanisme d’authentification basé sur des URI ou des hachages sécurisés SHA-256. Une authentification par signature d’accès partagé a deux composants : une stratégie d’accès partagé et une signature d’accès partagé (souvent appelée « jeton »). Un [appareil IoT Plug-and-Play](#iot-plug-and-play-device) utilise une signature d’accès partagé pour s’authentifier auprès d’un [hub IoT](#azure-iot-hub).

## <a name="solution-builder"></a>Générateur de solutions

Les générateurs de solutions créent la solution back-end. Ils utilisent généralement des ressources Azure, comme [IoT Hub](#azure-iot-hub) et les [référentiels de modèles](#model-repository).

## <a name="telemetry"></a>Télémétrie

Les champs de télémétrie définis dans une [interface](#interface) représentent des mesures. Ces mesures sont généralement des valeurs comme les lectures d’un capteur envoyées par l’[appareil IoT Plug-and-Play](#iot-plug-and-play-device) sous la forme d’un flux de données.

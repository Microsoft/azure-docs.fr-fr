---
title: Présentation des tarifs Azure IoT Hub | Microsoft Docs
description: 'Guide du développeur : informations concernant le fonctionnement des mesures et des tarifs IoT Hub, avec des exemples.'
author: robinsh
manager: philmea
ms.author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 03/11/2019
ms.custom:
- amqp
- mqtt
ms.openlocfilehash: 4c7382f84522333b6aae0d79941aae8f2147a12f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "81729158"
---
# <a name="azure-iot-hub-pricing-information"></a>Informations sur les tarifs Azure IoT Hub

La [tarification d’Azure IoT Hub](https://azure.microsoft.com/pricing/details/iot-hub) fournit des informations générales sur les différentes références SKU et les tarifs d’IoT Hub. Cet article contient des informations supplémentaires sur la manière dont les différentes fonctionnalités d’IoT Hub sont mesurées en tant que messages par IoT Hub.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

## <a name="charges-per-operation"></a>Frais par opération

| Opération | Informations de facturation | 
| --------- | ------------------- |
| Opérations du registre d’identité <br/> (créer, récupérer, répertorier, mettre à jour, supprimer) | Non facturé. |
| Messages appareil-à-cloud | Les messages envoyés avec succès sont facturés en blocs de 4 Ko une fois entrés dans IoT Hub. Par exemple, un message de 6 Ko est facturé 2 messages. |
| Messages Cloud vers appareil | Les messages envoyés avec succès sont facturés en blocs de 4 Ko ; par exemple, un message de 6 Ko est facturé 2 messages. |
| Chargements de fichiers | Le transfert de fichiers vers Stockage Azure n’est pas mesuré par IoT Hub. Les messages de lancement et de complétion du transfert de fichiers sont facturés en tant que messages, par incréments de 4 Ko. Par exemple, le transfert d’un fichier de 10 Mo est facturé deux messages, en plus du coût de Stockage Azure. |
| Méthodes directes | Les demandes réussies de méthodes sont facturées par blocs de 4 Ko, tandis que les réponses sont elles aussi facturées par blocs de 4 Ko, en tant que messages supplémentaires. Les demandes dirigées vers des appareils déconnectés sont facturées en tant que messages, par blocs de 4 Ko. Par exemple, une méthode avec un corps de 4 Ko qui génère une réponse sans corps de l’appareil est facturée comme deux messages. Une méthode avec un corps de 6 Ko qui génère une réponse de 1 Ko de l’appareil est facturée comme deux messages pour la demande, plus un autre message pour la réponse. |
| Lectures de jumeaux de l’appareil et du module | Les lectures de jumeaux de l’appareil ou du module et du backend de la solution sont facturées en tant que messages, par blocs de 512 octets. Par exemple, la lecture d’un jumeau de 6 Ko est facturée comme 12 messages. |
| Mises à jour de jumeaux d’appareil ou de module (balises et propriétés) | Les mises à jour de jumeaux à partir de l’appareil ou du module et du backend de la solution sont facturées en tant que messages, par blocs de 512 octets. Par exemple, la lecture d’un jumeau de 6 Ko est facturée comme 12 messages. |
| Requêtes de jumeaux de l’appareil et du module | Les requêtes sont facturées en tant que messages, en fonction de la taille du résultat, par blocs de 512 octets. |
| Opérations de travaux <br/> (créer, mettre à jour, répertorier, supprimer) | Non facturé. |
| Opérations de travaux par appareil | Les opérations de travaux (comme les mises à jour de jumeaux et les méthodes) sont facturées normalement. Par exemple, un travail entraînant 1 000 appels de méthode avec des demandes de 1 Ko et des réponses à corps vide est facturé comme 1 000 messages. |
| Keep-alive messages | Lorsque vous utilisez les protocoles AMQP ou MQTT, des messages sont échangés pour établir la connexion et les messages échangés dans la négociation ne sont pas facturés. |

> [!NOTE]
> L’ensemble des tailles sont calculées en fonction de la taille en octets de la charge utile (le tramage de protocole est ignoré). Pour les messages qui ont des propriétés et un corps, la taille est calculée de manière agnostique du point de vue du protocole. Pour plus d’informations, consultez [Créer et lire des messages IoT Hub](iot-hub-devguide-messages-construct.md).

## <a name="example-1"></a>Exemple 1

Un appareil transmet un message appareil-à-cloud de 1 Ko par minute vers IoT Hub ; ce message est ensuite lu par Azure Stream Analytics. Le backend de la solution appelle une méthode (avec une charge utile de 512 octets) sur l’appareil toutes les 10 minutes afin de déclencher une action spécifique. L’appareil répond à la méthode, avec un résultat de 200 octets.

L’appareil consomme :

* Un message * 60 minutes * 24 heures = 1440 messages par jour pour les messages appareil-à-cloud.
* Deux demande plus réponse * 6 fois par heure * 24 heures = 288 messages pour les méthodes.

Ce calcul donne un total de 1 728 messages par jour.

## <a name="example-2"></a>Exemple n° 2

Chaque heure, un appareil envoie un message appareil-à-cloud de 100 Ko. Il met également à jour son jumeau d’appareil avec des charges utiles de 1 Ko toutes les quatre heures. Le back-end de la solution, une fois par jour, lit le jumeau d’appareil de 14 Ko et le met à jour avec des charges utiles de 512 octets afin de modifier les configurations.

L’appareil consomme :

* 25 (100 Ko / 4 Ko) messages * 24 heures pour les messages appareil-à-cloud.
* Deux messages (1 Ko / 0,5 Ko) * six fois par jour pour les mises à jour de jumeau d’appareil.

Ce calcul donne un total de 612 messages par jour.

Le backend de la solution consomme 28 messages (14 Ko / 0,5 Ko) pour lire le jumeau d’appareil, plus un message pour le mettre à jour, ce qui donne un total de 29 messages.

Au total, l’appareil et le back-end de la solution consomment 641 messages par jour.

---
title: Fichier include
description: Fichier include
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: include
ms.date: 07/17/2019
ms.author: robinsh
ms.custom: include file
ms.openlocfilehash: e493d1c4f5851ee510ea83e706afce5fbb6f487e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "70049064"
---
<!-- This contains intro text for the "Get an IoT hub connection string" section in the iot-hub-lang-lang-schedule-jobs.md files-->

Dans cet article, vous créez un service back-end qui planifie une tâche pour appeler une méthode directe sur un appareil, planifie une tâche pour mettre à jour le jumeau d’appareil, et supervise l’avancement de chaque tâche. Pour effectuer ces opérations, votre service requiert les autorisations de **lecture du registre** et d’**écriture du registre**. Par défaut, chaque hub IoT est créé avec une stratégie d’accès partagé nommée **registryReadWrite** qui accorde ces autorisations.

---
title: Fichier Include
description: Fichier Include
services: functions
author: craigshoemaker
manager: gwallace
ms.service: azure-functions
ms.topic: include
ms.date: 01/28/2020
ms.author: cshoe
ms.custom: include file
ms.openlocfilehash: 6a37850eb6536c5399d63144e60ea210fbc194d8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77198414"
---
#### <a name="azure-storage-sdk-version-in-functions-1x"></a>Version du kit SDK Stockage Azure dans Functions 1.x

Dans Functions 1.x, les déclencheurs et les liaisons de stockage utilisent la version 7.2.1 du kit SDK Stockage Azure (package NuGet [WindowsAzure.Storage](https://www.nuget.org/packages/WindowsAzure.Storage/7.2.1)). Si vous référencez une version différente du kit SDK Stockage Azure et établissez une liaison avec un type du kit SDK Stockage Azure dans votre signature de fonction, le runtime de Functions peut signaler ne pas pouvoir établir de liaison avec ce type. La solution consiste à s’assurer que votre projet référence [WindowsAzure.Storage 7.2.1](https://www.nuget.org/packages/WindowsAzure.Storage/7.2.1).

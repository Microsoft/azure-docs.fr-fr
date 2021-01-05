---
title: Installer des Spatial Anchors Azure pour une application HoloLens C++/WinRT
description: Configurer un projet HoloLens C++/WinRT pour utiliser des Spatial Anchors Azure
author: msftradford
manager: MehranAzimi-msft
services: azure-spatial-anchors
ms.author: parkerra
ms.date: 11/20/2020
ms.topic: how-to
ms.service: azure-spatial-anchors
ms.openlocfilehash: 69134ef87d90fe69de2d9e4e9222e90f65edc785
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95507008"
---
# <a name="configuring-azure-spatial-anchors-in-a-cwinrt-hololens-project"></a>Configuration de Spatial Anchors Azure dans un projet HoloLens C++/WinRT

## <a name="requirements"></a>Spécifications

* [Visual Studio 2019](https://www.visualstudio.com/downloads/) installé avec la charge de travail de **développement pour la plateforme Windows universelle** et le composant **SDK Windows 10 (version 10.0.18362.0 ou plus récente)** .

## <a name="configuring-a-project"></a>Configuration d’un projet

Les Spatial Anchors Azure pour HoloLens et C++/WinRT sont distribuées à l’aide du package NuGet [Microsoft.Azure.SpatialAnchors.WinRT](https://www.nuget.org/packages/Microsoft.Azure.SpatialAnchors.WinRT/).

Suivez les instructions [ici](/nuget/consume-packages/install-use-packages-visual-studio) pour utiliser le gestionnaire de package NuGet de Visual Studio afin d’installer le package NuGet [Microsoft.Azure.SpatialAnchors.WinRT](https://www.nuget.org/packages/Microsoft.Azure.SpatialAnchors.WinRT/) dans votre projet.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Guide pratique pour Créer et localiser des ancres en C++/WinRT](./create-locate-anchors-cpp-winrt.md)
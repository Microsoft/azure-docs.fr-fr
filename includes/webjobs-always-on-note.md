---
title: Fichier Include
description: Fichier Include
services: app-service
author: ggailey777
ms.service: app-service
ms.topic: include
ms.date: 06/26/2020
ms.author: glenga
ms.custom: include file
ms.openlocfilehash: c255be53a1809bf5dd3fc6b184852767dfec9c66
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96009673"
---
> [!NOTE]
> Une application web peut expirer après 20 minutes d’inactivité. et seules les requêtes à destination de l’application web active peuvent réinitialiser le minuteur. L’affichage de la configuration de l’application dans le portail Azure ou l’envoi de requêtes au site d’outils avancés (`https://<app_name>.scm.azurewebsites.net`) ne réinitialise pas le minuteur. Si vous configurez votre application web pour exécuter des WebJobs continus ou planifiés (déclencheur de minuteur), activez le paramètre **Toujours activé** dans la page Azure **Configuration** de votre application web pour vous assurer que les WebJobs s’exécutent de manière fiable. Cette fonctionnalité est disponible uniquement dans les [niveaux tarifaires](https://azure.microsoft.com/pricing/details/app-service/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) De base, Standard et Premium.

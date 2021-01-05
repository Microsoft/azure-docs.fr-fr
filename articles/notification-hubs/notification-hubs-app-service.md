---
title: Intégration de Notification Hubs à App Service Mobile Apps
description: Découvrez comment Azure Notification Hubs fonctionne avec Azure App Service Mobile Apps.
author: sethmanheim
manager: femila
services: notification-hubs
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: multiple
ms.devlang: multiple
ms.topic: overview
ms.custom: mvc
ms.date: 08/06/2020
ms.author: sethm
ms.reviewer: thsomasu
ms.lastreviewed: 01/04/2019
ms.openlocfilehash: e7042be7e407e8e0827e142ba6878dfff812e1f6
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "88004053"
---
# <a name="integration-with-app-service-mobile-apps"></a>Intégration d’App Service Mobile Apps

Pour offrir une expérience transparente et unifiée dans l’ensemble des services Azure, [App Service Mobile Apps](/previous-versions/azure/app-service-mobile/app-service-mobile-value-prop) intègre la prise en charge des notifications à l’aide de Notification Hubs. [App Service Mobile Apps](/previous-versions/azure/app-service-mobile/app-service-mobile-value-prop) offre une plateforme de développement d’applications mobiles, hautement évolutive et disponible globalement, qui offre aux développeurs et intégrateurs système d’entreprise un vaste éventail de fonctionnalités.

Les développeurs d’applications mobiles peuvent utiliser Notification Hubs avec le flux de travail suivant :

1. Récupérer le handle PNS de l’appareil.
2. Inscrire un appareil auprès de Notification Hubs à l’aide d’API d’inscription du Kit de développement logiciel (SDK) de client Mobile Apps.

    > [!NOTE]
    > Notez que Mobile Apps supprime tous les mots clés des inscriptions pour des raisons de sécurité. Utilisez Notification Hubs directement depuis votre serveur principal pour associer des mots clés aux appareils.

3. Envoyer des notifications de votre serveur principal d’application avec Notification Hubs.

Voici certains avantages de cette intégration :

- **SDK clients Mobile Apps** : ces kits de développement logiciel (SDK) multi-plateformes fournissent des API pour l’inscription et communiquent avec le hub de notification lié à l’application mobile. Vous n’avez besoin ni d’informations d’identification de Notification Hubs, ni d’utiliser un service supplémentaire.
  - *Notification Push à un utilisateur* : les kits de développement logiciel balisent automatiquement l’appareil spécifié avec identifiant utilisateur authentifié auprès de Mobile Apps pour permettre l’implémentation du scénario de transmission de notifications Push à une utilisateur.
  - *Notification Push à un appareil* : les kits de développement utilisent automatiquement l’ID d’installation de Mobile Apps en tant que GUID pour effectuer l’inscription auprès de Notification Hubs, ce qui de devoir gérer plusieurs GUID de service.
- **Modèle d’installation** : le service Mobile Apps fonctionne avec le modèle d’envoi (push) le plus récent de Notification Hubs pour représenter l’ensemble des propriétés d’envoi (push) associées à un appareil dans une installation JSON alignée avec les services de notifications Push et facile à utiliser.
- **Flexibilité** : les développeurs peuvent toujours choisir d’utiliser Notification Hubs directement, même avec l’intégration effective.
- **Expérience intégrée dans le [portail Azure](https://portal.azure.com)**  : la fonctionnalité d’envoi (push) est représentée visuellement dans Mobile Apps. Les développeurs peuvent facilement utiliser le hub de notification associé via Mobile Apps.

---
title: Prise en charge par Azure Notification Hubs de la migration vers Google Firebase Cloud Messaging (FCM)
description: Décrit la manière dont Azure Notification Hubs gère la migration de Google GCM vers Google FCM.
services: notification-hubs
author: sethmanheim
manager: femila
editor: jwargo
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: ''
ms.devlang: ''
ms.topic: article
ms.date: 04/10/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 04/10/2019
ms.openlocfilehash: 2f2ca4b56445b3f399477e396de579d8a8c539e3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80127004"
---
# <a name="azure-notification-hubs-and-google-firebase-cloud-messaging-migration"></a>Prise en charge par Azure Notification Hubs de la migration vers Google Firebase Cloud Messaging

## <a name="current-state"></a>État actuel

Lorsque Google a annoncé la migration de Google Cloud Messaging (GCM) vers Firebase Cloud Messaging (FCM), les services d’envoi (push) tels que les nôtres ont dû adapter la méthode d’envoi des notifications aux appareils Android pour prendre en compte ce changement.

Nous avons mis à jour notre service principal, puis nous avons publié les mises à jour de nos API et Kits de développement logiciel (SDK) selon les besoins. Dans le cadre de notre implémentation, nous avons pris la décision de maintenir la compatibilité avec les schémas de notification GCM existants afin de minimiser l’impact sur les clients. Cela signifie que nous envoyons actuellement les notifications aux appareils Android à l’aide de FCM en mode FCM hérité. À terme, nous souhaitons offrir une véritable prise en charge de FCM, y compris de ses nouvelles fonctionnalités et du format de charge utile. Cet objectif représente une évolution à long terme. La migration actuelle est axée sur le maintien de la compatibilité avec les applications et Kits de développement logiciel (SDK) existants. Vous pouvez choisir d’utiliser les Kits de développement logiciel (SDK) GCM ou FCM dans votre application (en même temps que notre SDK). Quel que soit votre choix, nous nous assurons que les notifications seront correctement envoyées.

Certains clients ont récemment reçu un e-mail d’avertissement de la part de Google signalant le fait que des applications utilisaient encore un point de terminaison GCM pour les notifications. Il ne s’agissait que d’un avertissement, et tout continue de fonctionner normalement. Les notifications Android de votre application sont toujours envoyées à Google à des fins de traitement, et Google continue de traiter ces notifications. Certains clients ayant explicitement spécifié le point de terminaison GCM dans leur configuration de service continuaient à utiliser le point de terminaison déconseillé. Nous avions déjà identifié ce problème et avions entrepris de le corriger au moment où Google a envoyé cet e-mail.

Nous avons remplacé le point de terminaison déconseillé et déployé le correctif associé.

## <a name="going-forward"></a>À l’avenir

Le FAQ Google concernant FCM indique que vous n’avez rien à faire. Dans son [FAQ FCM](https://developers.google.com/cloud-messaging/faq), Google déclare ce qui suit : « Les Kits de développement logiciel (SDK) clients et les jetons GCM continueront de fonctionner indéfiniment. Toutefois, vous ne serez plus en mesure de cibler la dernière version de Google Play Services dans votre application Android, sauf si vous migrez vers FCM. »

Si votre application utilise la bibliothèque GCM, suivez les instructions de Google pour mettre à niveau cette dernière vers la bibliothèque FCM dans votre application. Notre Kit de développement logiciel (SDK) étant compatible avec ces deux bibliothèques, vous n’aurez rien à mettre à jour dans votre application de notre côté (tant que votre version du Kit de développement logiciel (SDK) reste à jour).

## <a name="questions-and-answers"></a>Questions et réponses

Voici les réponses aux questions les plus fréquemment posées par les clients :

**Q :** Que dois-je faire pour être compatible d’ici la date limite (la date limite actuellement fixée par Google est le 29 mai et peut évoluer) ?

**R :** Nothing. Nous maintiendrons la compatibilité avec le schéma de notification GCM existant. Votre clé GCM continuera de fonctionner normalement, ainsi que tous les Kits de développement logiciel (SDK) et bibliothèques GCM utilisés par votre application.

Le jour où vous déciderez d’effectuer la mise à niveau vers les Kits de développement logiciel (SDK) et bibliothèques FCM pour tirer parti des nouvelles fonctionnalités, votre clé GCM continuera de fonctionner. Vous pouvez basculer vers l’utilisation d’une clé FCM si vous le souhaitez, mais dans ce cas, prenez soin d’ajouter Firebase à votre projet GCM existant lorsque vous créez le projet Firebase. Cette opération garantira la compatibilité descendante avec vos clients exécutant des versions antérieures de l’application qui utilisent toujours des bibliothèques et Kits de développement logiciel (SDK) GCM.

Si vous créez un projet FCM sans l’attacher au projet GCM existant, une fois que vous mettrez à jour Notification Hubs avec le nouveau secret FCM, vous perdrez la possibilité d’envoyer (push) des notifications à vos installations d’application actuelles, dans la mesure où la nouvelle clé FCM ne dispose d’aucun lien vers l’ancien projet GCM.

**Q :** Pourquoi ai-je reçu cet e-mail concernant l’utilisation des anciens points de terminaison GCM ? Que dois-je faire ?

**R :** Nothing. Nous avons commencé la migration vers les nouveaux points de terminaison, et nous aurons bientôt terminé. Aucune modification n’est donc nécessaire. Tout continue de fonctionner normalement. Le seul point de terminaison que nous avons manqué a seulement entraîné l’envoi de messages d’avertissement de la part de Google.

**Q :** Comment puis-je effectuer la transition vers les nouveaux Kits de développement logiciel (SDK) et bibliothèques FCM sans interrompre le travail des utilisateurs existants ?

A : Vous pouvez effectuer la mise à niveau à tout moment. Google n’a pas encore annoncé la dépréciation des Kits de développement logiciel (SDK) et bibliothèques GCM existants. Pour éviter tout risque d’interruption de l’envoi des notifications Push aux utilisateurs existants, prenez soin d’associer votre projet GCM existant lorsque vous créez le projet Firebase. De cette manière, les nouveaux secrets Firebase fonctionneront aussi bien pour les utilisateurs qui exécutent d’anciennes versions de votre application avec les Kits de développement logiciel (SDK) et bibliothèques GCM, que pour les nouveaux utilisateurs de votre application avec les Kits de développement logiciel (SDK) et bibliothèques FCM.

**Q :** Quand pourrai-je utiliser les nouveaux schémas et fonctionnalités FCM pour mes notifications ?

**R :** Après la publication d’une mise à jour de nos API et Kits de développement logiciel (SDK), consultez régulièrement notre site pour découvrir les nouveautés que nous mettrons à votre disposition dans les prochains mois.

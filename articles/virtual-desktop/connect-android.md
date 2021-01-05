---
title: Connexion à Windows Virtual Desktop via Android – Azure
description: Comment se connecter à Windows Virtual Desktop à l’aide du client Android.
author: Heidilohr
ms.topic: how-to
ms.date: 03/25/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: ebf91f68e45148d9a609ff671ffa4683bd74c82c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89226112"
---
# <a name="connect-to-windows-virtual-desktop-with-the-android-client"></a>Connexion à Windows Virtual Desktop avec le client Android

> S’applique à : Android 4.1 et versions ultérieures, Chromebooks avec ChromeOS 53 et versions ultérieures.

>[!IMPORTANT]
>Ce contenu s’applique à Windows Virtual Desktop avec des objets Windows Virtual Desktop Azure Resource Manager. Si vous utilisez Windows Virtual Desktop (classique) sans objets Azure Resource Manager, consultez [cet article](./virtual-desktop-fall-2019/connect-android-2019.md).

Vous pouvez accéder aux ressources Windows Virtual Desktop à partir de votre appareil Android avec notre client téléchargeable. Vous pouvez également utiliser le client Android sur des appareils Chromebook qui prennent en charge le Google Play Store. Ce guide explique comment configurer le client Android.

## <a name="install-the-android-client"></a>Installer le client Android

Pour commencer, [téléchargez](https://play.google.com/store/apps/details?id=com.microsoft.rdc.androidx) et installez le client sur votre appareil Android.

## <a name="subscribe-to-a-feed"></a>S’abonner à un flux

Abonnez-vous au flux fourni par votre administrateur pour obtenir la liste des ressources managées auxquelles vous pouvez accéder sur votre appareil Android.

Pour s’abonner à un flux :

1. Dans Connection Center, appuyez sur **+** , puis sur **Remote Resource Feed** (Flux de ressources à distance).
2. Entrez l’URL du flux dans le champ **Feed URL** (URL du flux). Il peut s’agir d’une URL ou d’une adresse e-mail.
   - Si vous utilisez une URL, choisissez celle que votre administrateur vous a donnée, normalement <https://rdweb.wvd.microsoft.com/api/arm/feeddiscovery>.
   - Pour utiliser l’e-mail, entrez votre adresse e-mail. Le client recherche une URL associée à votre adresse de adresse e-mail si votre administrateur a configuré le serveur de cette façon.
   - Pour vous connecter par le biais du portail US Gov, utilisez <https://rdweb.wvd.azure.us/api/arm/feeddiscovery>.
3. Appuyez sur **SUIVANT**.
4. À l’invite, indiquez vos informations d’identification.
   - Pour **Nom d’utilisateur**, indiquez le nom de l’utilisateur ayant l’autorisation d’accéder aux ressources.
   - Pour **Mot de passe**, indiquez le mot de passe associé au nom d’utilisateur.
   - Vous pouvez également être invité à fournir des facteurs supplémentaires si votre administrateur a configuré l’authentification de cette manière.

Une fois l’abonnement souscrit, le Connection Center doit afficher les ressources distantes.

Une fois que vous êtes abonné à un flux, son contenu est mis à jour automatiquement et régulièrement. Des ressources peuvent être ajoutées, modifiées ou supprimées en fonction des modifications apportées par votre administrateur.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur l’utilisation du client Android, voir [Prise en main du client Android](/windows-server/remote/remote-desktop-services/clients/remote-desktop-android/).

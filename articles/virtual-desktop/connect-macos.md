---
title: 'Se connecter à Windows Virtual Desktop à partir de macOS : Azure'
description: Comment se connecter à Windows Virtual Desktop à l’aide du client macOS.
author: Heidilohr
ms.topic: how-to
ms.date: 04/08/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: f437c43c1e9ad960910e7576db4b3ddf3f6623ec
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89230872"
---
# <a name="connect-to-windows-virtual-desktop-with-the-macos-client"></a>Connexion à Windows Virtual Desktop avec le client macOS

> S’applique à : macOS 10.12 ou version ultérieure

>[!IMPORTANT]
>Ce contenu s’applique à Windows Virtual Desktop avec des objets Windows Virtual Desktop Azure Resource Manager. Si vous utilisez la version Windows Virtual Desktop (classique) sans objets Azure Resource Manager, consultez [cet article](./virtual-desktop-fall-2019/connect-macos-2019.md).

Vous pouvez accéder aux ressources Windows Virtual Desktop à partir de vos appareils macOS avec notre client téléchargeable. Ce guide vous explique comment configurer le client.

## <a name="install-the-client"></a>Installation du client

Pour commencer, [téléchargez](https://apps.apple.com/app/microsoft-remote-desktop/id1295203466?mt=12) et installez le client sur votre appareil macOS.

## <a name="subscribe-to-a-feed"></a>S’abonner à un flux

Abonnez-vous au flux que votre administrateur vous a indiqué pour obtenir la liste des ressources managées disponibles sur votre appareil macOS.

Pour s’abonner à un flux :

1. Sélectionnez **Add Workspace** (Ajouter un espace de travail) sur la page principale pour vous connecter au service et récupérer vos ressources.
2. Entrez l’URL du flux. Il peut s’agir d’une URL ou d’une adresse e-mail :
   - Si vous utilisez une URL, utilisez celle que votre administrateur vous a donnée. En règle générale, l’URL est <https://rdweb.wvd.microsoft.com/api/arm/feeddiscovery>.
   - Pour utiliser l’e-mail, entrez votre adresse e-mail. Cela indique au client de rechercher une URL associée à votre adresse e-mail si votre administrateur a configuré le serveur de cette manière.
   - Pour vous connecter par le biais du portail US Gov, utilisez <https://rdweb.wvd.azure.us/api/arm/feeddiscovery>.
3. Sélectionnez **Ajouter**.
4. Connectez-vous avec votre compte d’utilisateur quand vous y êtes invité.

Une fois que vous êtes connecté, la liste des ressources disponibles doit s’afficher.

Une fois que vous êtes abonné à un flux, son contenu est automatiquement mis à jour de façon régulière. Des ressources peuvent être ajoutées, modifiées ou supprimées en fonction des modifications apportées par votre administrateur.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur l’utilisation du client macOS, consultez [Bien démarrer avec le client macOS](/windows-server/remote/remote-desktop-services/clients/remote-desktop-mac/).

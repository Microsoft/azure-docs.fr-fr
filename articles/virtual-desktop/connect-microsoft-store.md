---
title: Se connecter au client Microsoft Store - Azure
description: Explique comment se connecter à Windows Virtual Desktop à l’aide du client Microsoft Store.
author: Heidilohr
ms.topic: how-to
ms.date: 10/05/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 9bab2a11ff9e7907621087e5027929e3e42eaf29
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91744634"
---
# <a name="connect-with-the-microsoft-store-client"></a>Se connecter avec le client Microsoft Store

>S’applique à : Windows 10.

Vous pouvez accéder aux ressources Windows Virtual Desktop sur des appareils avec Windows 10.

## <a name="install-the-microsoft-store-client"></a>Installer le client Microsoft Store

Vous pouvez installer le client pour l’utilisateur actif, ce qui ne nécessite pas de droits d’administrateur. Votre administrateur peut également installer et configurer le client afin que tous les utilisateurs de l’appareil puissent y accéder.

Une fois installé, le client peut être lancé à partir du menu Démarrer quand vous recherchez Bureau à distance.

Pour commencer, [téléchargez et installez le client à partir du Microsoft Store](https://www.microsoft.com/store/productId/9WZDNCRFJ3PS).

## <a name="subscribe-to-a-workspace"></a>S’abonner à un espace de travail

Abonnez-vous à l’espace de travail fourni par votre administrateur pour obtenir la liste des ressources managées auxquelles vous pouvez accéder sur votre PC.

Pour s’abonner à un espace de travail

1. À l’écran Centre de connexion, appuyez sur **+ Ajouter**, puis sur **Espaces de travail**.
2. Entrez l’URL de l’espace de travail dans le champ URL de l’espace de travail fourni par votre administrateur. Il peut s’agir d’une URL ou d’une adresse e-mail.
   
   - Si vous utilisez une URL d’espace de travail, utilisez l’URL que votre administrateur vous a fournie.
   - Si vous vous connectez à partir de Windows Virtual Desktop, utilisez l’une des URL suivantes en fonction de la version du service que vous utilisez :
       - Windows Virtual Desktop (classique) : `https://rdweb.wvd.microsoft.com/api/feeddiscovery/webfeeddiscovery.aspx`.
       - Windows Virtual Desktop : `https://rdweb.wvd.microsoft.com/api/arm/feeddiscovery`.
  
3. Appuyez sur **S’abonner**.
4. À l’invite, indiquez vos informations d’identification.
5. Une fois que vous êtes abonné, les espaces de travail doivent être affichés dans le Centre de connexion.

Les espaces de travail peuvent être ajoutés, modifiés ou supprimés en fonction des modifications apportées par votre administrateur.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur l’utilisation du client Microsoft Store, consultez [Bien démarrer avec le client Microsoft Store](/windows-server/remote/remote-desktop-services/clients/windows/).
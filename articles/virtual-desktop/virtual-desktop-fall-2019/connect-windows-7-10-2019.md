---
title: Se connecter à Windows Virtual Desktop (classique) – Windows 10 ou 7 – Azure
description: Comment se connecter à Windows Virtual Desktop (classique) à l’aide du client Windows Desktop.
author: Heidilohr
ms.topic: how-to
ms.date: 07/16/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 420c507361e3e2437366e6ccf2d46a8b78684e59
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88008355"
---
# <a name="connect-with-the-windows-desktop-classic-client"></a>Se connecter avec le client Windows Desktop (classique)

> S’applique à : Windows 7, Windows 10 et Windows 10 IoT Entreprise

>[!IMPORTANT]
>Ce contenu s’applique à Windows Virtual Desktop (classique), qui ne prend pas en charge les objets Windows Virtual Desktop Azure Resource Manager. Si vous essayez de gérer des objets Windows Virtual Desktop Azure Resource Manager, consultez [cet article](../connect-windows-7-10.md).

Vous pouvez accéder aux ressources Windows Virtual Desktop sur des appareils Windows 7, Windows 10 ou Windows 10 IoT Entreprise à l’aide du client Bureau Windows. Le client ne prend pas en charge Windows 8 ni Windows 8.1.

>[!NOTE]
>Le client Windows passe automatiquement par défaut à Windows Virtual Desktop (classique). Toutefois, si le client détecte que l’utilisateur a également des ressources Azure Resource Manager, il ajoute automatiquement les ressources ou avertit l’utilisateur qu’elles sont disponibles.

> [!IMPORTANT]
> Windows Virtual Desktop ne prend pas en charge le client RADC (Connexions RemoteApp et Bureau à distance) ni le client MSTSC (Connexion Bureau à distance).

> [!IMPORTANT]
> Windows Virtual Desktop ne prend pas actuellement en charge le client Bureau à distance à partir du Microsoft Store.

## <a name="install-the-windows-desktop-client"></a>Installer le client Windows Desktop

Choisissez le client qui correspond à votre version de Windows :

- [Windows 64 bits](https://go.microsoft.com/fwlink/?linkid=2068602)
- [Windows 32 bits](https://go.microsoft.com/fwlink/?linkid=2098960)
- [Windows ARM64](https://go.microsoft.com/fwlink/?linkid=2098961)

Vous pouvez installer le client pour l’utilisateur actuel, ce qui ne nécessite pas de droits d’administrateur, ou votre administrateur peut installer et configurer le client afin que tous les utilisateurs de l’appareil puissent y accéder.

Une fois installé, le client peut être lancé à partir du menu Démarrer quand vous recherchez **Bureau à distance**.

## <a name="subscribe-to-a-workspace"></a>S'abonner à un espace de travail

Il y deux manières de vous abonner à un espace de travail. Le client peut essayer de découvrir les ressources disponibles à partir de votre compte professionnel ou scolaire, ou vous pouvez spécifier directement l’URL de vos ressources dans les cas où le client ne parvient pas à les trouver. Une fois que vous êtes abonné à un espace de travail, vous pouvez lancer des ressources avec l’une des méthodes suivantes :

- Accédez au centre de connexion, puis double-cliquez sur une ressource pour la lancer.
- Vous pouvez également accéder au menu Démarrer et rechercher un dossier avec le nom de l’espace de travail ou entrer le nom de la ressource dans la barre de recherche.

### <a name="subscribe-with-a-user-account"></a>S’abonner avec un compte d’utilisateur

1. Dans la page principale du client, sélectionnez **S’abonner**.
2. Connectez-vous avec votre compte d’utilisateur quand vous y êtes invité.
3. Les ressources s’affichent dans le centre de connexion et sont regroupées par espace de travail.

### <a name="subscribe-with-a-url"></a>S’abonner avec une URL

1. Dans la page principale du client, sélectionnez **S’abonner avec une URL**.
2. Entrez l’URL de l’espace de travail ou votre adresse e-mail :
   - Si vous utilisez une **URL d’espace de travail**, utilisez celle que votre administrateur vous a donnée. Si vous accédez à des ressources à partir de Windows Virtual Desktop, vous pouvez utiliser l’une des URL suivantes :
     - Windows Virtual Desktop (classique) : `https://rdweb.wvd.microsoft.com/api/feeddiscovery/webfeeddiscovery.aspx`
     - Windows Virtual Desktop : `https://rdweb.wvd.microsoft.com/api/arm/feeddiscovery`
   - Si vous utilisez le champ **E-mail** à la place, entrez votre adresse e-mail. Cela indique au client de rechercher une URL associée à votre adresse e-mail si votre administrateur a configuré la [détection d’e-mails](/windows-server/remote/remote-desktop-services/rds-email-discovery).
3. Sélectionnez **Suivant**.
4. Connectez-vous avec votre compte d’utilisateur quand vous y êtes invité.
5. Les ressources devraient s’afficher dans le centre de connexion, regroupées par espace de travail.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur l’utilisation du client Windows Desktop, consultez [Bien démarrer avec le client Windows Desktop](/windows-server/remote/remote-desktop-services/clients/windowsdesktop/).

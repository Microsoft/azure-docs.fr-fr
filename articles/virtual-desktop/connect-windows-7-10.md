---
title: Se connecter à Windows Virtual Desktop - Windows 10 ou 7 - Azure
description: Guide pratique pour se connecter à Windows Virtual Desktop à l’aide du client Windows Desktop.
author: Heidilohr
ms.topic: how-to
ms.date: 09/22/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 59379dd9c3a41729466de269b52dfd3fb206eea9
ms.sourcegitcommit: 1bdcaca5978c3a4929cccbc8dc42fc0c93ca7b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/13/2020
ms.locfileid: "97368713"
---
# <a name="connect-with-the-windows-desktop-client"></a>Se connecter avec le client Windows Desktop

> S'applique à : Windows 10, Windows 10 IoT Entreprise et Windows 7

>[!IMPORTANT]
>Ce contenu s’applique à Windows Virtual Desktop avec des objets Windows Virtual Desktop Azure Resource Manager. Si vous utilisez la version Windows Virtual Desktop (classique) sans objets Azure Resource Manager, consultez [cet article](./virtual-desktop-fall-2019/connect-windows-7-10-2019.md).

Vous pouvez accéder aux ressources Windows Virtual Desktop sur des appareils Windows 10, Windows 10 IoT Entreprise et Windows 7 à l’aide du client Bureau Windows. Le client ne prend pas en charge Windows 8 ou Windows 8.1.

## <a name="install-the-windows-desktop-client"></a>Installer le client Windows Desktop

Choisissez le client qui correspond à votre version de Windows :

- [Windows 64 bits](https://go.microsoft.com/fwlink/?linkid=2068602)
- [Windows 32 bits](https://go.microsoft.com/fwlink/?linkid=2098960)
- [Windows ARM64](https://go.microsoft.com/fwlink/?linkid=2098961)

Vous pouvez installer le client pour l’utilisateur actuel, ce qui ne nécessite pas de droits d’administrateur, ou votre administrateur peut installer et configurer le client afin que tous les utilisateurs de l’appareil puissent y accéder.

Une fois installé, le client peut être lancé à partir du menu Démarrer quand vous recherchez **Bureau à distance**.

> [!IMPORTANT]
> Windows Virtual Desktop ne prend pas en charge le client RADC (Connexions RemoteApp et Bureau à distance) ni le client MSTSC (Connexion Bureau à distance).

## <a name="subscribe-to-a-workspace"></a>S’abonner à un espace de travail

Vous avez deux façons de vous abonner à un espace de travail. Le client peut tenter de découvrir les ressources disponibles à partir de votre compte professionnel ou scolaire, ou vous pouvez spécifier directement l’URL de vos ressources, si le client ne parvient pas à les trouver. Dès que vous êtes abonné à un espace de travail, lancez des ressources en utilisant l’une des méthodes suivantes :

- Accédez au Centre de connexion, puis double-cliquez sur une ressource pour la lancer.
- Vous pouvez également accéder au menu Démarrer et rechercher un dossier avec le nom de l’espace de travail ou entrer le nom de la ressource dans la barre de recherche.

### <a name="subscribe-with-a-user-account"></a>S’abonner avec un compte d’utilisateur

1. Dans la page principale du client, sélectionnez **S’abonner**.
2. Connectez-vous avec votre compte d’utilisateur quand vous y êtes invité.
3. Les ressources s’affichent dans le centre de connexion et sont regroupées par espace de travail.

>[!NOTE]
>Le client Windows passe automatiquement par défaut à Windows Virtual Desktop (classique). Toutefois, si le client détecte que l’utilisateur a également des ressources Azure Resource Manager, il ajoute automatiquement les ressources ou avertit l’utilisateur qu’elles sont disponibles.

### <a name="subscribe-with-a-url"></a>S’abonner avec une URL

1. Dans la page principale du client, sélectionnez **S’abonner avec une URL**.
2. Entrez l’URL de l’espace de travail ou votre adresse e-mail :
   - Si vous utilisez l’**URL de l’espace de travail**, utilisez celle que votre administrateur vous a donnée. Si vous accédez aux ressources à partir de Windows Virtual Desktop, vous pouvez utiliser l’une des URL suivantes :
     - Windows Virtual Desktop (classique) : `https://rdweb.wvd.microsoft.com/api/feeddiscovery/webfeeddiscovery.aspx`
     - Windows Virtual Desktop : `https://rdweb.wvd.microsoft.com/api/arm/feeddiscovery`
     - Windows Virtual Desktop (US Gov) : `https://rdweb.wvd.azure.us/api/arm/feeddiscovery`
   - Si vous utilisez le champ **E-mail** à la place, entrez votre adresse e-mail. Cela indique au client de rechercher une URL associée à votre adresse e-mail si votre administrateur a configuré la [détection d’e-mails](/windows-server/remote/remote-desktop-services/rds-email-discovery).
3. Sélectionnez **Suivant**.
4. Connectez-vous avec votre compte d’utilisateur quand vous y êtes invité.
5. Les ressources devraient s’afficher dans le centre de connexion, regroupées par espace de travail.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur l’utilisation du client Windows Desktop, consultez [Bien démarrer avec le client Windows Desktop](/windows-server/remote/remote-desktop-services/clients/windowsdesktop/).

Si vous êtes un administrateur désireux d’obtenir des informations plus approfondies sur l’utilisation du bureau Windows, consultez [Client Windows Desktop pour administrateurs](/windows-server/remote/remote-desktop-services/clients/windowsdesktop-admin).

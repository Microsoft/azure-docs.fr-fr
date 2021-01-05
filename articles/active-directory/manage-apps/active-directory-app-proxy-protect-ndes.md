---
title: Intégration au proxy d’application AD sur un serveur NDES
titleSuffix: Azure Active Directory
description: Aide sur le déploiement d’un proxy d’application Azure Active Directory pour protéger votre serveur NDES.
services: active-directory
author: kenwith
manager: celestedg
ms.assetid: ''
ms.service: active-directory
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 01/17/2020
ms.author: kenwith
ms.reviewer: mimart
ms.openlocfilehash: 0682115727068c928418d97fbf92ed32897c39d4
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94656749"
---
# <a name="integrate-with-azure-ad-application-proxy-on-a-network-device-enrollment-service-ndes-server"></a>Intégration au proxy d’application Azure AD sur un serveur NDES (Network Device Enrollment service)

Le proxy d’application Azure Active Directory (AD) vous permet de publier des applications à l’intérieur de votre réseau. Parmi ces applications, citons des sites SharePoint, Microsoft Outlook Web App et d’autres applications web. Il fournit également un accès sécurisé aux utilisateurs à l’extérieur de votre réseau par le biais d’Azure.

Si vous débutez avec le proxy d’application Azure AD et que vous souhaitez en savoir plus, consultez [Accéder à distance à des applications locales par le biais du proxy d’application Azure AD](application-proxy.md).

Le proxy d’application Azure AD repose sur Azure. Il met à votre disposition une grande quantité de bande passante réseau et une infrastructure serveur pour offrir une meilleure protection contre les attaques par déni de service distribué (DDOS) et une disponibilité exceptionnelle. Par ailleurs, vous n’avez pas besoin d’ouvrir les ports d’un pare-feu externe à votre réseau local ni de disposer d’un serveur DMZ. Tout le trafic est entrant. Pour obtenir la liste complète des ports sortants, consultez [Tutoriel : Ajouter une application locale pour un accès à distance via le service Proxy d’application d’Azure Active Directory](./application-proxy-add-on-premises-application.md#prepare-your-on-premises-environment).

> Le proxy d’application Azure AD est une fonctionnalité qui n’est disponible que si vous utilisez l’édition Premium ou De base d’Azure Active Directory. Pour plus d’informations, consultez [Tarification Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/). 
> Si vous disposez de licences EMS (Enterprise Mobility Suite), vous pouvez utiliser cette solution.
> Le connecteur Proxy d’application Azure AD ne peut être installé que sur Windows Server 2012 R2 ou ultérieur. Le serveur NDES impose également cette exigence.

## <a name="install-and-register-the-connector-on-the-ndes-server"></a>Installer et inscrire le connecteur sur le serveur NDES

1. Connectez-vous au [portail Azure](https://portal.azure.com/) en tant qu'administrateur d'application du répertoire qui utilise le service Proxy d'application. Par exemple, si le domaine du client est contoso.com, l'administrateur doit être admin@contoso.com ou tout autre alias administratif sur ce domaine.
1. Sélectionnez votre nom d’utilisateur en haut à droite. Vérifiez que vous êtes connecté à un répertoire qui utilise le service Proxy d’application. Si vous devez changer de répertoire, sélectionnez **Changer de répertoire** et choisissez un répertoire qui utilise le service Proxy d’application.
1. Dans le volet de navigation de gauche, sélectionnez **Azure Active Directory**.
1. Sous **Gérer**, sélectionnez **Proxy d’application**.
1. Sélectionnez **Télécharger le service de connecteur**.

    ![Télécharger le service de connecteur pour voir les conditions d’utilisation du service](./media/active-directory-app-proxy-protect-ndes/application-proxy-download-connector-service.png)

1. Lisez les conditions d'utilisation du service. Quand vous êtes prêt, sélectionnez **Accepter les conditions d’utilisation et télécharger**.
1. Copiez le fichier d’installation du connecteur Proxy d’application Azure AD sur votre serveur NDES. 
   > Vous pouvez installer le connecteur sur n’importe quel serveur au sein de votre réseau d’entreprise ayant accès à NDES. Vous n’avez pas besoin de l’installer sur le serveur NDES lui-même.
1. Exécutez le fichier d’installation, tel que *AADApplicationProxyConnectorInstaller.exe*. Acceptez les termes du contrat de licence logicielle.
1. Au cours de l’installation, vous êtes invité à inscrire le connecteur auprès du Proxy d’application de votre répertoire Azure AD.
   * Indiquez les informations d’identification d’un administrateur général ou d’application dans votre répertoire Azure AD. Les informations d’identification d’un administrateur général ou d’application Azure AD peuvent être différentes de vos informations d’identification Azure dans le portail.

        > [!NOTE]
        > Le compte d’administrateur général ou d’application utilisé pour inscrire le connecteur doit appartenir au même répertoire que celui dans lequel vous avez activé le service Proxy d’application.
        >
        > Par exemple, si le domaine Azure AD est *contoso.com*, l’administrateur général ou d’application doit être `admin@contoso.com` ou tout autre alias valide sur ce domaine.

   * Si l’option Configuration de sécurité renforcée d’Internet Explorer est activée sur le serveur sur lequel vous installez le connecteur, l’écran d’inscription risque d’être bloqué. Pour autoriser l’accès, suivez les instructions du message d’erreur ou désactivez la sécurité renforcée d’Internet Explorer au cours du processus d’installation.
   * Si l’inscription du connecteur échoue, consultez[Détecter un problème du Proxy d’application](application-proxy-troubleshoot.md).
1. À la fin de l’installation, une remarque s’affiche pour les environnements avec un proxy sortant. Pour configurer le connecteur de Proxy d’application Azure AD et qu’il fonctionne par le biais du proxy sortant, exécutez le script fourni, tel que `C:\Program Files\Microsoft AAD App Proxy connector\ConfigureOutBoundProxy.ps1`.
1. Dans la page du proxy d’application du Portail Azure, le nouveau connecteur est répertorié avec l’état *Actif*, comme indiqué dans l’exemple suivant :

    ![Le nouveau connecteur de Proxy d’application Azure AD affiché comme actif dans le Portail Azure](./media/active-directory-app-proxy-protect-ndes/connected-app-proxy.png)

    > [!NOTE]
    > Pour fournit une haute disponibilité pour l’authentification des applications par le biais du Proxy d’application Azure AD, vous pouvez installer des connecteurs sur plusieurs machines virtuelles. Effectuez les étapes répertoriées dans la section précédente pour installer le connecteur sur d’autres serveurs joints au domaine managé Azure AD DS.

1. Une fois l’installation terminée, revenez au portail Azure.

1. Sélectionnez **Applications d’entreprise**.

   ![Veillez à faire appel aux bonnes personnes.](./media/active-directory-app-proxy-protect-ndes/azure-active-directory-enterprise-applications.png)

1. Sélectionnez **+Nouvelle application**, puis **Application locale**. 

1. Dans **Ajouter votre propre application locale**, configurez les champs suivants :

   * **Name** : Entrez un nom pour l’application.
   * **URL interne** : entrez l’URL ou le nom de domaine complet interne de votre serveur NDES sur lequel vous avez installé le connecteur.
   * **Pré-authentification** : sélectionnez **Passthrough**. Vous ne pouvez pas utiliser une quelconque forme de pré-authentification. Le protocole utilisé pour les demandes de certificat (SCEP) n’offre pas cette option.
   * Copiez l’**URL externe** fournie dans votre Presse-papiers.

1. Sélectionnez **+Ajouter** pour enregistrer votre application.

1. Faites un test pour déterminer si vous pouvez accéder à votre serveur NDES par le biais du proxy d’application Azure AD en collant le lien que vous avez copié à l’étape 10 dans un navigateur. Une page d’accueil IIS par défaut doit s’ouvrir.

1. Comme test final, ajoutez le chemin *mscep.dll* à l’URL existante que vous avez collée à l’étape précédente :

   https://scep-test93635307549127448334.msappproxy.net/certsrv/mscep/mscep.dll

1. Vous devez obtenir une réponse **Erreur HTTP 403 – Interdit**.

1. Remplacez l’URL NDES fournie (via Microsoft Intune) par des appareils. Cette modification peut s’effectuer dans Microsoft Endpoint Configuration Manager ou dans le centre d’administration Microsoft Endpoint Manager.

   * Pour Configuration Manager, accédez au point d’enregistrement de certificat et ajustez l’URL. C’est cette URL qu’appellent les appareils pour présenter leur demande.
   * Pour Intune autonome, modifiez ou créez une stratégie SCEP et ajoutez la nouvelle URL.

## <a name="next-steps"></a>Étapes suivantes

Une fois le proxy d’application Azure AD intégré à NDES, publiez des applications auxquelles les utilisateurs peuvent accéder. Pour plus d’informations, consultez [Publier des applications à l’aide du Proxy d’application Azure AD](./application-proxy-add-on-premises-application.md).
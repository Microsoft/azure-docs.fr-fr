---
title: Configurer la jointure hybride Azure Active Directory pour des domaines fédérés | Microsoft Docs
description: Découvrez comment configurer une jonction Azure Active Directory hybride pour les domaines fédérés.
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: tutorial
ms.date: 05/20/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sandeo
ms.collection: M365-identity-device-management
ms.openlocfilehash: cc2f7d3ce5f8329038fea4ecbb5242015fb3fd0d
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96860131"
---
# <a name="tutorial-configure-hybrid-azure-active-directory-join-for-federated-domains"></a>Tutoriel : Configurer la jointure hybride Azure Active Directory pour des domaines fédérés

Comme n’importe quel utilisateur de votre organisation, un appareil est une identité fondamentale que vous souhaitez protéger. Vous pouvez utiliser l’identité d’un appareil pour protéger vos ressources à tout moment et à partir de n’importe quel emplacement. Pour atteindre cet objectif, vous devez intégrer et gérer les identités des appareils dans Azure AD (Azure Active Directory) à l’aide de l’une des méthodes suivantes :

- jointure Azure AD ;
- jointure Azure AD hybride ;
- inscription Azure AD.

En intégrant vos appareils à Azure AD, vous optimisez la productivité des utilisateurs via SSO (authentification unique) parmi vos ressources cloud et locales. En parallèle, vous pouvez sécuriser l’accès à vos ressources cloud et locales à l’aide de l’[accès conditionnel](../conditional-access/howto-conditional-access-policy-compliant-device.md).

Un environnement fédéré doit disposer d’un fournisseur d’identité qui prend en charge les exigences suivantes : Si vous disposez d’un environnement fédéré utilisant les services de fédération Active Directory (AD FS), les exigences ci-dessous sont déjà prises en charge.

- **Revendication WIAORMULTIAUTHN :** Cette revendication est nécessaire à des fins de jonction Azure AD Hybride pour les appareils Windows de bas niveau.
- **Protocole WS-Trust :** Ce protocole est nécessaire pour authentifier auprès d’Azure AD les appareils Windows actuels ayant fait l’objet d’une jonction Azure AD Hybride.
  Lorsque vous utilisez AD FS, vous devez activer les points de terminaison WS-Trust suivants : `/adfs/services/trust/2005/windowstransport`
   `/adfs/services/trust/13/windowstransport`
   `/adfs/services/trust/2005/usernamemixed`
   `/adfs/services/trust/13/usernamemixed`
   `/adfs/services/trust/2005/certificatemixed`
   `/adfs/services/trust/13/certificatemixed` 

> [!WARNING] 
> **adfs/services/trust/2005/windowstransport** et **adfs/services/trust/13/windowstransport** doivent tous les deux être activés en tant que points de terminaison uniquement accessibles sur intranet et ils NE doivent PAS être exposés comme points de terminaison accessibles sur extranet via le proxy d’application web. Pour en savoir plus sur la désactivation des points de terminaison Windows WS-Trust, consultez [Désactiver les points de terminaison Windows WS-Trust sur le proxy](/windows-server/identity/ad-fs/deployment/best-practices-securing-ad-fs#disable-ws-trust-windows-endpoints-on-the-proxy-ie-from-extranet). Vous pouvez visualiser les points de terminaison qui sont activés par le biais de la console de gestion AD FS sous **Service** > **Points de terminaison**.

Dans ce tutoriel, vous apprenez à configurer une jonction Azure AD Hybride pour des appareils joints à un domaine Active Directory dans un environnement fédéré avec AD FS.

Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Configurer une jonction Azure AD hybride
> * Activer des appareils Windows de niveau inférieur
> * Vérifier l’inscription
> * Dépanner

## <a name="prerequisites"></a>Prérequis

Ce tutoriel part du principe que les articles suivants vous sont familiers :

- [Qu’est-ce qu’une identité d’appareil ?](overview.md)
- [Guide pratique pour planifier votre implémentation de la jonction Azure AD Hybride](hybrid-azuread-join-plan.md)
- [Comment effectuer une validation contrôlée de la jonction Azure AD hybride](hybrid-azuread-join-control.md)

Pour configurer le scénario décrit dans ce tutoriel, vous avez besoin de ce qui suit :

- Windows Server 2012 R2 avec AD FS
- [Azure AD Connect](https://www.microsoft.com/download/details.aspx?id=47594) version 1.1.819.0 ou ultérieure

Depuis la version 1.1.819.0, Azure AD Connect comprend un Assistant qui vous permet de configurer la jonction Azure AD Hybride. L’Assistant simplifie considérablement le processus de configuration. Cet Assistant :

- Configure les SCP (points de connexion de service) pour l’inscription des appareils
- Sauvegarde votre approbation de partie de confiance Azure AD existante
- Met à jour les règles de revendication dans votre approbation Azure AD

Les étapes de configuration décrites dans cet article sont basées sur l’utilisation de l’Assistant Azure AD Connect. Si une version antérieure d’Azure AD Connect est déjà installée, vous devez la mettre à niveau vers la version 1.1.819 ou ultérieure pour pouvoir utiliser l’Assistant. Si vous n’envisagez pas d’installer la dernière version d’Azure AD Connect, consultez [Comment configurer manuellement la jonction Azure AD Hybride](hybrid-azuread-join-manual.md).

Pour permettre le bon fonctionnement de la jonction Azure AD Hybride, les appareils doivent avoir accès aux ressources Microsoft suivantes sur le réseau de votre organisation :  

- `https://enterpriseregistration.windows.net`
- `https://login.microsoftonline.com`
- `https://device.login.microsoftonline.com`
- Le STS (service d’émission de jeton de sécurité) de votre organisation (pour les domaines fédérés)
- `https://autologon.microsoftazuread-sso.com` (si vous utilisez ou prévoyez d’utiliser l’authentification unique fluide)

> [!WARNING]
> Si votre organisation utilise des serveurs proxy qui interceptent le trafic SSL dans des scénarios de protection contre la perte de données ou de restrictions de locataire Azure AD par exemple, veillez à exclure le trafic à destination de « https://device.login.microsoftonline.com  » de l’inspection TLS. Faute d’exclure « https://device.login.microsoftonline.com  », l’authentification par certificat client peut être sujette à des interférences, ce qui occasionne des problèmes d’inscription d’appareil et d’accès conditionnel en fonction de l’appareil.

À partir de Windows 10 1803, en cas d’échec de la jonction Azure AD Hybride instantanée pour un environnement fédéré utilisant AD FS, nous utilisons Azure AD Connect afin de synchroniser l’objet ordinateur dans Azure AD, qui est ensuite utilisé pour effectuer l’inscription de l’appareil à une jonction Azure AD Hybride. Vérifiez qu’Azure AD Connect a synchronisé les objets ordinateur des appareils qui doivent devenir membres d’Azure AD Hybride. Si les objets ordinateur appartiennent à des unités d’organisation spécifiques, vous devez également configurer ces unités d’organisation pour qu’elles se synchronisent dans Azure AD Connect. Pour en savoir plus sur la synchronisation des objets ordinateur à l’aide d’Azure AD Connect, consultez [Configurer le filtrage à l’aide d’Azure AD Connect](../hybrid/how-to-connect-sync-configure-filtering.md#organizational-unitbased-filtering).

Si votre organisation nécessite un accès à Internet via un proxy sortant, Microsoft recommande d’[implémenter le protocole WPAD (Web Proxy AutoDiscovery)](/previous-versions/tn-archive/cc995261(v%3dtechnet.10)) pour permettre aux ordinateurs Windows 10 de s’inscrire en tant qu’appareils auprès d’Azure AD. Si vous rencontrez des problèmes pour configurer et gérer le protocole WPAD, consultez la [résolution des problèmes de détection automatique](/previous-versions/tn-archive/cc302643(v=technet.10)). 

Si vous n’utilisez pas WPAD et si vous souhaitez configurer des paramètres de proxy sur votre ordinateur, vous pouvez le faire à partir de Windows 10 1709. Pour plus d’informations, consultez [Configurer les paramètres WinHTTP à l’aide d’un GPO (objet de stratégie de groupe)](/archive/blogs/netgeeks/winhttp-proxy-settings-deployed-by-gpo).

> [!NOTE]
> Si vous configurez les paramètres du proxy sur votre ordinateur à l’aide des paramètres WinHTTP, les ordinateurs qui ne peuvent pas se connecter au proxy configuré ne pourront pas non plus se connecter à Internet.

Si votre organisation nécessite un accès à Internet via un proxy sortant authentifié, vérifiez que vos ordinateurs Windows 10 parviennent à s’authentifier correctement auprès du proxy sortant. Dans la mesure où les ordinateurs Windows 10 effectuent l’inscription de l’appareil en fonction du contexte de la machine, vous devez configurer l’authentification du proxy sortant selon le contexte de la machine. Poursuivez avec la configuration requise pour votre fournisseur de proxy sortant.

Pour vérifier si l’appareil est en mesure d’accéder aux ressources Microsoft ci-dessus sous le compte système, vous pouvez utiliser le script [Tester la connectivité de l’inscription des appareils](https://gallery.technet.microsoft.com/Test-Device-Registration-3dc944c0).

## <a name="configure-hybrid-azure-ad-join"></a>Configurer une jonction Azure AD hybride

Pour configurer une jonction Azure AD Hybride à l’aide d’Azure AD Connect, vous avez besoin des informations suivantes :

- Informations d’identification d’un administrateur général pour votre locataire Azure AD  
- Informations d’identification de l’administrateur d’entreprise pour chacune des forêts
- Informations d’identification de l’administrateur AD FS

**Pour configurer une jonction Azure AD Hybride à l’aide d’Azure AD Connect** :

1. Démarrez Azure AD Connect, puis sélectionnez **Configurer**.

   ![Bienvenue](./media/hybrid-azuread-join-federated-domains/11.png)

1. Dans la page **Tâches supplémentaires**, sélectionnez **Configurer les options de l’appareil**, puis **Suivant**.

   ![Tâches supplémentaires](./media/hybrid-azuread-join-federated-domains/12.png)

1. Dans la page **Vue d’ensemble**, sélectionnez **Suivant**.

   ![Vue d’ensemble](./media/hybrid-azuread-join-federated-domains/13.png)

1. Dans la page **Connexion à Azure AD**, entrez les informations d’identification d’un administrateur général pour votre locataire Azure AD, puis sélectionnez **Suivant**.

   ![Se connecter à Azure AD](./media/hybrid-azuread-join-federated-domains/14.png)

1. Dans la page **Options de l’appareil**, sélectionnez **Configurer la jonction Azure AD Hybride**, puis **Suivant**.

   ![Options de l’appareil](./media/hybrid-azuread-join-federated-domains/15.png)

1. Dans la page **SCP**, effectuez les étapes suivantes, puis sélectionnez **Suivant** :

   ![SCP](./media/hybrid-azuread-join-federated-domains/16.png)

   1. Sélectionnez la forêt.
   1. Sélectionnez le service d’authentification. Vous devez sélectionner **Serveur AD FS**, sauf si votre organisation a exclusivement des clients Windows 10 et si vous avez configuré la synchronisation d’ordinateur/appareil, ou si votre organisation utilise l’authentification unique fluide.
   1. Sélectionnez **Ajouter** pour entrer les informations d’identification de l’administrateur d’entreprise.

1. Dans la page **Systèmes d’exploitation des appareils**, sélectionnez les systèmes d’exploitation utilisés par les appareils dans votre environnement Active Directory, puis sélectionnez **Suivant**.

   ![Systèmes d’exploitation des appareils](./media/hybrid-azuread-join-federated-domains/17.png)

1. Dans la page **Configuration de la fédération**, entrez les informations d’identification de l’administrateur AD FS, puis sélectionnez **Suivant**.

   ![Configuration de la fédération](./media/hybrid-azuread-join-federated-domains/18.png)

1. Dans la page **Prêt à configurer**, sélectionnez **Configurer**.

   ![Prêt à configurer](./media/hybrid-azuread-join-federated-domains/19.png)

1. Dans la page **Configuration terminée**, sélectionnez **Quitter**.

   ![Configuration terminée](./media/hybrid-azuread-join-federated-domains/20.png)

## <a name="enable-windows-downlevel-devices"></a>Activer des appareils Windows de niveau inférieur

Si certains de vos appareils appartenant à un domaine sont des appareils Windows de niveau inférieur, vous devez :

- Configurer les paramètres d’intranet local pour l’inscription des appareils
- Installer Microsoft Workplace Join pour des ordinateurs Windows de niveau inférieur

> [!NOTE]
> Le support de Windows 7 a pris fin le 14 janvier 2020. Pour plus d’informations, consultez [Le support de Windows 7 a pris fin le 14 janvier 2020](https://support.microsoft.com/en-us/help/4057281/windows-7-support-ended-on-january-14-2020).

### <a name="configure-the-local-intranet-settings-for-device-registration"></a>Configurer les paramètres d’intranet local pour l’inscription des appareils

Pour effectuer correctement la jonction Azure AD Hybride de vos appareils Windows de niveau inférieur et éviter les invites de certificat quand les appareils s’authentifient auprès d’Azure AD, vous pouvez envoyer (push) une stratégie aux appareils appartenant à un domaine afin d’ajouter les URL suivantes à la zone intranet locale dans Internet Explorer :

- `https://device.login.microsoftonline.com`
- STS de votre organisation (pour les domaines fédérés)
- `https://autologon.microsoftazuread-sso.com` (pour l’authentification unique fluide)

Vous devez également activer **Autoriser les mises à jour de la barre d’état via le script** dans la zone Intranet local de l’utilisateur.

### <a name="install-microsoft-workplace-join-for-windows-downlevel-computers"></a>Installer Microsoft Workplace Join pour des ordinateurs Windows de niveau inférieur

Pour inscrire les appareils Windows de niveau inférieur, les organisations doivent installer [Microsoft Workplace Join pour les ordinateurs non Windows 10](https://www.microsoft.com/download/details.aspx?id=53554). Microsoft Workplace Join pour les ordinateurs non Windows 10 est disponible dans le Centre de téléchargement Microsoft.

Vous pouvez déployer le package à l’aide d’un système de distribution de logiciels, comme  [Microsoft Endpoint Configuration Manager](/configmgr/). Le package prend en charge les options d’installation sans assistance standard avec le paramètre `quiet`. La branche actuelle de Configuration Manager offre des avantages supplémentaires par rapport aux versions précédentes, comme la possibilité d’effectuer le suivi des inscriptions effectuées.

Le programme d’installation crée une tâche planifiée sur le système, qui s’exécute dans le contexte de l’utilisateur. La tâche est déclenchée lorsque l’utilisateur se connecte à Windows. La tâche joint l’appareil en mode sans assistance à Azure AD avec les informations d’identification de l’utilisateur, après son authentification auprès d’Azure AD.

## <a name="verify-the-registration"></a>Vérifier l’inscription

Voici trois méthodes pour rechercher et vérifier l’état de l’appareil :

### <a name="locally-on-the-device"></a>Localement sur l’appareil

1. Ouvrez Windows PowerShell.
2. Entrez `dsregcmd /status`.
3. Vérifiez que **AzureAdJoined** et **DomainJoined** ont la valeur **OUI**.
4. Vous pouvez utiliser le **DeviceId** et comparer l’état du service à l’aide du portail Azure ou de PowerShell.

### <a name="using-the-azure-portal"></a>Utilisation du portail Azure

1. Accédez à la page des appareils à l’aide d’un [lien direct](https://portal.azure.com/#blade/Microsoft_AAD_IAM/DevicesMenuBlade/Devices).
2. Pour plus d’informations sur la localisation d’un appareil, consultez [Gérer les identités de l’appareil à l’aide du portail Microsoft Azure](./device-management-azure-portal.md).
3. Si la colonne **Inscrit** indique **En attente**, cela signifie que la jonction Azure AD Hybride n’a pas été effectuée complètement. Dans les environnements fédérés, cela peut se produire uniquement si l’inscription a échoué et qu’AAD Connect est configuré pour synchroniser les appareils.
4. Si la colonne **Inscrit** contient une **date/heure**, cela signifie que la jonction Azure AD Hybride a été effectuée correctement.

### <a name="using-powershell"></a>Utilisation de PowerShell

Vérifiez l’état d’inscription de l’appareil dans votre locataire Azure à l’aide de **[Get-MsolDevice](/powershell/module/msonline/get-msoldevice)** . Cette applet de commande figure dans le [module Azure Active Directory PowerShell](/powershell/azure/active-directory/install-msonlinev1).

Quand vous utilisez l’applet de commande **Get-MSolDevice** pour vérifier les détails du service :

- Un objet dont l’**ID d’appareil** correspond à l’ID défini sur le client Windows doit exister.
- La valeur pour **DeviceTrustType** est **Joint au domaine**. Ce paramètre équivaut à l’état **Joint à une version hybride d’Azure AD** dans la page **Appareils** du portail Azure AD.
- Pour les appareils utilisés dans l’accès conditionnel, la valeur pour **Activé** est **True** et celle pour **DeviceTrustLevel** est **Géré**.

1. Ouvrez Windows PowerShell en tant qu’administrateur.
2. Entrez `Connect-MsolService` pour vous connecter à votre locataire Azure.

#### <a name="count-all-hybrid-azure-ad-joined-devices-excluding-pending-state"></a>Compter tous les appareils joints Azure AD Hybride (sauf ceux qui sont à l’état **En attente**)

```azurepowershell
(Get-MsolDevice -All -IncludeSystemManagedDevices | where {($_.DeviceTrustType -eq 'Domain Joined') -and (([string]($_.AlternativeSecurityIds)).StartsWith("X509:"))}).count
```

#### <a name="count-all-hybrid-azure-ad-joined-devices-with-pending-state"></a>Compter tous les appareils joints Azure AD Hybride qui sont à l’état **En attente**

```azurepowershell
(Get-MsolDevice -All -IncludeSystemManagedDevices | where {($_.DeviceTrustType -eq 'Domain Joined') -and (-not([string]($_.AlternativeSecurityIds)).StartsWith("X509:"))}).count
```

#### <a name="list-all-hybrid-azure-ad-joined-devices"></a>Lister tous les appareils joints Azure AD Hybride

```azurepowershell
Get-MsolDevice -All -IncludeSystemManagedDevices | where {($_.DeviceTrustType -eq 'Domain Joined') -and (([string]($_.AlternativeSecurityIds)).StartsWith("X509:"))}
```

#### <a name="list-all-hybrid-azure-ad-joined-devices-with-pending-state"></a>Lister tous les appareils joints Azure AD Hybride qui sont à l’état **En attente**

```azurepowershell
Get-MsolDevice -All -IncludeSystemManagedDevices | where {($_.DeviceTrustType -eq 'Domain Joined') -and (-not([string]($_.AlternativeSecurityIds)).StartsWith("X509:"))}
```

#### <a name="list-details-of-a-single-device"></a>Lister les détails d’un seul appareil :

1. Entrez `get-msoldevice -deviceId <deviceId>` (il s’agit du **DeviceId** obtenu localement sur l’appareil).
2. Vérifiez que le paramètre **Enabled** est défini sur **True**.

## <a name="troubleshoot-your-implementation"></a>Résoudre les problèmes liés à votre implémentation

Si vous rencontrez des problèmes durant l’exécution d’une jonction Azure AD Hybride pour des appareils Windows appartenant à un domaine, consultez :

- [Dépannage des appareils à l’aide de la commande dsregcmd](./troubleshoot-device-dsregcmd.md)
- [Résolution des problèmes de jonction Azure AD Hybride pour les appareils Windows actuels](troubleshoot-hybrid-join-windows-current.md)
- [Résolution des problèmes de jonction Azure AD Hybride pour les appareils Windows de niveau inférieur](troubleshoot-hybrid-join-windows-legacy.md)

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment [gérer les identités des appareils à l’aide du portail Azure](device-management-azure-portal.md).

<!--Image references-->
[1]: ./media/active-directory-conditional-access-automatic-device-registration-setup/12.png

---
title: Configurer la jointure hybride Azure Active Directory pour des domaines managés | Microsoft Docs
description: Découvrez comment configurer une jonction Azure Active Directory hybride pour les domaines managés.
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: tutorial
ms.date: 03/06/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sandeo
ms.collection: M365-identity-device-management
ms.openlocfilehash: 303f02e0c6b72b7061a996b3ce8e70799954b435
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96861049"
---
# <a name="tutorial-configure-hybrid-azure-active-directory-join-for-managed-domains"></a>Tutoriel : Configurer la jointure hybride Azure Active Directory pour des domaines managés

Dans ce tutoriel, vous allez apprendre à configurer une jointure Azure Active Directory (Azure AD) hybride pour des appareils joints à un domaine Active Directory. Cette méthode prend en charge un environnement managé qui comprend à la fois Active Directory local et Azure AD.

Comme n’importe quel utilisateur de votre organisation, un appareil est une identité fondamentale que vous souhaitez protéger. Vous pouvez utiliser l’identité d’un appareil pour protéger vos ressources à tout moment et à partir de n’importe quel emplacement. Vous pouvez atteindre cet objectif en gérant les identités des appareils dans Azure AD. Appliquez l'une des méthodes suivantes :

- jointure Azure AD ;
- jointure Azure AD hybride ;
- inscription Azure AD.

Cet article se concentre sur la jointure Azure AD hybride.

En intégrant vos appareils à Azure AD, vous optimisez la productivité des utilisateurs via SSO (authentification unique) parmi vos ressources cloud et locales. En parallèle, vous pouvez sécuriser l’accès à vos ressources cloud et locales à l’aide de l’[accès conditionnel](../conditional-access/howto-conditional-access-policy-compliant-device.md).

Vous pouvez déployer un environnement managé en utilisant la [synchronisation de hachage de mot de passe (PHS)](../hybrid/whatis-phs.md) ou l’[authentification directe (PTA)](../hybrid/how-to-connect-pta.md) avec l’[authentification unique fluide](../hybrid/how-to-connect-sso.md). Ces scénarios ne nécessitent pas la configuration d’un serveur de fédération pour l’authentification.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Configurer une jonction Azure AD hybride
> * Activer des appareils Windows de bas niveau
> * Vérifier des appareils joints
> * Dépanner

## <a name="prerequisites"></a>Prérequis

- [Azure AD Connect](https://www.microsoft.com/download/details.aspx?id=47594) (1.1.819.0 ou version ultérieure)
- Informations d’identification d’un administrateur général pour votre locataire Azure AD
- Informations d’identification de l’administrateur d’entreprise pour chacune des forêts

Familiarisez-vous avec les articles suivants :

- [Qu’est-ce qu’une identité d’appareil ?](overview.md)
- [Guide pratique pour Planifier l’implémentation de la jonction Azure Active Directory Hybride](hybrid-azuread-join-plan.md)
- [Validation contrôlée de la jointure Azure AD hybride](hybrid-azuread-join-control.md)

> [!NOTE]
> Azure AD ne prend pas en charge les cartes à puce ou les certificats dans les domaines managés.

Vérifiez qu’Azure AD Connect a synchronisé les objets ordinateur des appareils qui doivent devenir membres d’Azure AD Hybride. Si les objets ordinateur appartiennent à des unités d’organisation spécifiques, configurez ces unités d’organisation pour les synchroniser dans Azure AD Connect. Pour en savoir plus sur la synchronisation des objets ordinateur à l’aide d’Azure AD Connect, consultez [Filtrage basé sur une unité d’organisation](../hybrid/how-to-connect-sync-configure-filtering.md#organizational-unitbased-filtering).

Depuis la version 1.1.819.0, Azure AD Connect inclut un Assistant pour configurer la jointure Azure AD hybride. L’Assistant simplifie considérablement le processus de configuration. L’Assistant configure les SCP (points de connexion de service) pour l’inscription des appareils.

Les étapes de configuration décrites dans cet article sont basées sur l’utilisation de l’Assistant dans Azure AD Connect.

Pour permettre le bon fonctionnement de la jonction Azure AD Hybride, les appareils doivent avoir accès aux ressources Microsoft suivantes sur le réseau de votre organisation :  

- `https://enterpriseregistration.windows.net`
- `https://login.microsoftonline.com`
- `https://device.login.microsoftonline.com`
- `https://autologon.microsoftazuread-sso.com` (si vous utilisez ou prévoyez d’utiliser l’authentification unique fluide)

> [!WARNING]
> Si votre organisation utilise des serveurs proxy qui interceptent le trafic SSL dans des scénarios de protection contre la perte de données ou de restrictions de locataire Azure AD par exemple, veillez à exclure le trafic à destination de « https://device.login.microsoftonline.com  » de l’inspection TLS. Faute d’exclure « https://device.login.microsoftonline.com  », l’authentification par certificat client peut être sujette à des interférences, ce qui occasionne des problèmes d’inscription d’appareil et d’accès conditionnel en fonction de l’appareil.

Si votre organisation nécessite un accès à Internet par le biais d’un proxy sortant, vous pouvez utiliser l’[implémentation du protocole WPAD (Web Proxy AutoDiscovery, détection automatique de proxy web)](/previous-versions/tn-archive/cc995261(v=technet.10)) pour permettre aux ordinateurs Windows 10 de s’inscrire en tant qu’appareils auprès d’Azure AD. Pour résoudre des problèmes de configuration et de gestion du protocole WPAD, consultez [Résolution des problèmes liés à la détection automatique](/previous-versions/tn-archive/cc302643(v=technet.10)). Dans les appareils Windows 10 antérieurs à la mise à jour 1709, WPAD est la seule option disponible pour configurer un proxy afin qu’il fonctionne avec une jointure Azure AD Hybride. 

Si vous n’utilisez pas le protocole WPAD, vous pouvez configurer des paramètres de proxy WinHTTP sur votre ordinateur à partir de la build 1709 de Windows 10. Pour plus d’informations, consultez [WinHTTP Proxy Settings deployed by GPO](/archive/blogs/netgeeks/winhttp-proxy-settings-deployed-by-gpo).

> [!NOTE]
> Si vous configurez les paramètres du proxy sur votre ordinateur à l’aide des paramètres WinHTTP, les ordinateurs qui ne peuvent pas se connecter au proxy configuré ne pourront pas non plus se connecter à Internet.

Si votre organisation nécessite un accès à Internet via un proxy sortant authentifié, vérifiez que vos ordinateurs Windows 10 parviennent à s’authentifier correctement auprès du proxy sortant. Dans la mesure où les ordinateurs Windows 10 effectuent l’inscription de l’appareil en fonction du contexte de la machine, configurez l’authentification du proxy sortant selon le contexte de la machine. Poursuivez avec la configuration requise pour votre fournisseur de proxy sortant.

Vérifiez que l’appareil peut accéder aux ressources Microsoft ci-dessus sous le compte système en utilisant le script [Tester la connectivité d’inscription d’appareil](https://gallery.technet.microsoft.com/Test-Device-Registration-3dc944c0).

## <a name="configure-hybrid-azure-ad-join"></a>Configurer une jonction Azure AD hybride

Pour configurer une jointure Azure AD hybride à l’aide d’Azure AD Connect :

1. Démarrez Azure AD Connect, puis sélectionnez **Configurer**.

   ![Bienvenue](./media/hybrid-azuread-join-managed-domains/welcome-azure-ad-connect.png)

1. Dans **Tâches supplémentaires**, sélectionnez **Configurer les options de l’appareil**, puis **Suivant**.

   ![Tâches supplémentaires](./media/hybrid-azuread-join-managed-domains/azure-ad-connect-additional-tasks.png)

1. Dans **Vue d’ensemble**, sélectionnez **Suivant**.

   ![Vue d’ensemble](./media/hybrid-azuread-join-managed-domains/azure-ad-connect-overview.png)

1. Dans **Connexion à Azure AD**, entrez les informations d’identification d’un administrateur général pour votre locataire Azure AD.  

   ![Se connecter à Azure AD](./media/hybrid-azuread-join-managed-domains/connect-to-azure-ad-username-password.png)

1. Dans **Options de l’appareil**, sélectionnez **Configurer joindre Hybrid Azure AD**, puis **Suivant**.

   ![Options de l’appareil](./media/hybrid-azuread-join-managed-domains/azure-ad-connect-device-options.png)

1. Dans **Configuration du SCP**, pour chaque forêt où vous souhaitez qu’Azure AD Connect configure le SCP (point de connexion de service), suivez les étapes ci-dessous, puis sélectionnez **Suivant**.

   1. Sélectionnez la **forêt**.
   1. Sélectionnez un **service d’authentification**.
   1. Sélectionnez **Ajouter** pour entrer les informations d’identification de l’administrateur d’entreprise.

   ![SCP](./media/hybrid-azuread-join-managed-domains/azure-ad-connect-scp-configuration.png)

1. Dans **Systèmes d’exploitation de l’appareil**, sélectionnez les systèmes d’exploitation utilisés par les appareils dans votre environnement Active Directory, puis sélectionnez **Suivant**.

   ![Systèmes d’exploitation des appareils](./media/hybrid-azuread-join-managed-domains/azure-ad-connect-device-operating-systems.png)

1. Dans **Prêt pour la configuration**, sélectionnez **Configurer**.

   ![Prêt à configurer](./media/hybrid-azuread-join-managed-domains/azure-ad-connect-ready-to-configure.png)

1. Dans **Configuration terminée**, sélectionnez **Quitter**.

   ![Configuration terminée](./media/hybrid-azuread-join-managed-domains/azure-ad-connect-configuration-complete.png)

## <a name="enable-windows-down-level-devices"></a>Activer des appareils Windows de bas niveau

Si certains de vos appareils joints à un domaine sont des appareils Windows de bas niveau, vous devez :

- Configurer les paramètres d’intranet local pour l’inscription des appareils
- Configurer l’authentification unique fluide
- Installer des ordinateurs de bas niveau Workplace Join for Windows

> [!NOTE]
> Le support pour Windows 7 a pris fin le 14 janvier 2020. Pour plus d’informations, consultez [Le support de Windows 7 a pris fin](https://support.microsoft.com/help/4057281/windows-7-support-ended-on-january-14-2020).

### <a name="configure-the-local-intranet-settings-for-device-registration"></a>Configurer les paramètres d’intranet local pour l’inscription des appareils

Pour terminer la jointure Azure AD hybride de vos appareils Windows de bas niveau et éviter les invites de certificat lorsque les appareils s’authentifient auprès d’Azure AD, vous pouvez envoyer (push) une stratégie à vos appareils joints à un domaine afin d’ajouter les URL ci-après à la zone d’intranet local dans Internet Explorer :

- `https://device.login.microsoftonline.com`
- `https://autologon.microsoftazuread-sso.com`

Vous devez également activer **Autoriser les mises à jour de la barre d’état via le script** dans la zone d’intranet local de l’utilisateur.

### <a name="configure-seamless-sso"></a>Configurer l’authentification unique fluide

Pour terminer la jointure Azure AD hybride de vos appareils Windows de bas niveau, dans un domaine managé qui utilise la [synchronisation de hachage de mot de passe](../hybrid/whatis-phs.md) ou l’[authentification directe](../hybrid/how-to-connect-pta.md) comme méthode d’authentification cloud Azure AD, vous devez également [configurer l’authentification unique fluide](../hybrid/how-to-connect-sso-quick-start.md#step-2-enable-the-feature).

### <a name="install-microsoft-workplace-join-for-windows-down-level-computers"></a>Installer des ordinateurs de bas niveau Workplace Join for Windows

Pour inscrire les appareils Windows de bas niveau, les organisations doivent installer [Microsoft Workplace Join pour les ordinateurs non-Windows 10](https://www.microsoft.com/download/details.aspx?id=53554). Microsoft Workplace Join pour les ordinateurs non Windows 10 est disponible dans le Centre de téléchargement Microsoft.

Vous pouvez déployer le package à l’aide d’un système de distribution de logiciels, comme  [Microsoft Endpoint Configuration Manager](/configmgr/). Le package prend en charge les options d’installation sans assistance standard avec le paramètre `quiet`. La version actuelle de Configuration Manager offre des avantages supplémentaires par rapport aux versions précédentes, comme la possibilité d’effectuer le suivi des inscriptions effectuées.

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
3. Si la colonne **Inscrit** indique **En attente**, cela signifie que la jonction Azure AD Hybride n’a pas été effectuée complètement.
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

Si vous rencontrez des problèmes en réalisant une jointure Azure AD hybride pour des appareils Windows joints à un domaine, consultez :

- [Dépannage des appareils à l’aide de la commande dsregcmd](./troubleshoot-device-dsregcmd.md)
- [Résolution des problèmes liés aux appareils hybrides joints à Azure Active Directory](troubleshoot-hybrid-join-windows-current.md)
- [Dépanner des appareils hybrides de bas niveau joints à Azure Active Directory](troubleshoot-hybrid-join-windows-legacy.md)

## <a name="next-steps"></a>Étapes suivantes

Passez à l’article suivant pour découvrir comment gérer les identités des appareils à l’aide du portail Azure.
> [!div class="nextstepaction"]
> [Gérer les identités des appareils](device-management-azure-portal.md)

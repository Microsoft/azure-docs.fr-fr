---
title: Conditions dans une stratégie d’accès conditionnel - Azure Active Directory
description: Que sont les conditions dans une stratégie d’accès conditionnel Azure AD ?
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: conceptual
ms.date: 08/07/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb
ms.collection: M365-identity-device-management
ms.openlocfilehash: 96d1eaff4d1b93ad3bb489f177020c351fe4d13d
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95904033"
---
# <a name="conditional-access-conditions"></a>Accès conditionnel : Conditions

Dans une stratégie d’accès conditionnel, un administrateur peut s’aider des signaux des conditions telles que le risque, la plateforme d’appareil ou l’emplacement pour prendre de meilleures décisions en matière de stratégie. 

[ ![Define a Conditional Access policy and specify conditions](./media/concept-conditional-access-conditions/conditional-access-conditions.png)](./media/concept-conditional-access-conditions/conditional-access-conditions.png#lightbox)

Il est possible de combiner plusieurs conditions pour créer des stratégies d’accès conditionnel spécifiques parfaitement adaptées aux besoins.

Par exemple, pour prendre ses décisions quant à l’accès à une application sensible, un administrateur peut tenir compte de l’emplacement et des informations sur le risque à la connexion fournies par Identity Protection en plus d’autres contrôles comme l’authentification multifacteur.

## <a name="sign-in-risk"></a>Risque à la connexion

Pour les clients ayant accès à [Identity Protection](../identity-protection/overview-identity-protection.md), le risque à la connexion peut être évalué dans le cadre d’une stratégie d’accès conditionnel. Le risque à la connexion reflète la probabilité qu’une requête d’authentification donnée soit rejetée par le propriétaire de l’identité. Pour plus d’informations sur le risque à la connexion, consultez les articles [Que sont les risques ?](../identity-protection/concept-identity-protection-risks.md#sign-in-risk) et [ Configurer et activer des stratégies de risque](../identity-protection/howto-identity-protection-configure-risk-policies.md).

## <a name="user-risk"></a>Risque de l’utilisateur 

Pour les clients ayant accès à [Identity Protection](../identity-protection/overview-identity-protection.md), le risque utilisateur peut être évalué dans le cadre d’une stratégie d’accès conditionnel. Un risque utilisateur reflète la probabilité qu’une identité ou un compte donné soit compromis. Pour plus d’informations sur le risque utilisateur, consultez les articles [Qu’est-ce que le risque](../identity-protection/concept-identity-protection-risks.md#user-risk) et [Comment : Configurer et activer des stratégies de risque](../identity-protection/howto-identity-protection-configure-risk-policies.md).

## <a name="device-platforms"></a>Plateformes d’appareils

La plateforme d’appareil se caractérise par le système d’exploitation qui s’exécute sur un appareil. Azure AD identifie la plateforme à l’aide des informations fournies par l’appareil, telles que des chaînes d’agent utilisateur. Étant donné que les chaînes d’agent utilisateur peuvent être modifiées, ces informations ne sont pas vérifiées. La plateforme d’appareil doit être utilisée conjointement avec des stratégies de conformité d’appareil Microsoft Intune ou dans le cadre d’une instruction de bloc. Le comportement par défaut est l’application à toutes les plateformes d’appareil.

L’accès conditionnel Azure AD prend en charge les plateformes d’appareil suivantes :

- Android
- iOS
- Windows Phone
- Windows
- macOS

Si vous bloquez l’authentification héritée à l’aide de la condition **Autres clients**, vous pouvez également définir la condition de plateforme de l’appareil.

> [!IMPORTANT]
> Microsoft vous recommande de disposer d’une stratégie d’accès conditionnel pour les plateformes d’appareil non prises en charge. Par exemple, si vous souhaitez bloquer l’accès à vos ressources d’entreprise à partir de Linux ou d’un autre client non pris en charge, vous devez configurer une stratégie avec une condition de plateformes d’appareil incluant n’importe quel appareil et excluant les plateformes d’appareil prises en charge et définir Accorder le contrôle sur Bloquer l’accès.

## <a name="locations"></a>Emplacements

Lors de la configuration de l’emplacement comme condition, les organisations peuvent choisir d’inclure ou d’exclure des emplacements. Ces emplacements nommés peuvent inclure les informations du réseau IPv4 public, le pays ou la région, ou même les zones inconnues qui ne correspondent pas à des pays ou régions spécifiques. Seules les plages d’adresses IP peuvent être marquées comme emplacement approuvé.

Si vous incluez **n’importe quel emplacement**, cette option inclut toutes les adresses IP sur Internet, et pas seulement les emplacements nommés qui ont été configurés. Quand l’option **N’importe quel emplacement** est sélectionnée, les administrateurs peuvent choisir d’exclure **tous les emplacements approuvés** ou les **emplacements sélectionnés**.

Par exemple, certaines organisations ne souhaitent pas exiger l’authentification multifacteur quand leurs utilisateurs sont connectés au réseau à un emplacement approuvé, comme leur siège social. Dans ce cas, les administrateurs créent une stratégie qui inclut n’importe quel emplacement, mais qui exclut les emplacements sélectionnés pour leurs réseaux de siège social.

Pour plus d’informations sur les emplacements, consultez l’article [Qu’est-ce que la condition d’emplacement de l’accès conditionnel Azure Active Directory ?](location-condition.md).

## <a name="client-apps"></a>Applications clientes

Par défaut, toutes les stratégies d’accès conditionnel nouvellement créées s’appliquent à tous les types d’applications clientes, même si la condition des applications clientes n’est pas configurée. 

> [!NOTE]
> Le comportement de la condition des applications clientes a été mis à jour en août 2020. Si vous disposez de stratégies d’accès conditionnel, elles restent inchangées. Toutefois, si vous cliquez sur une stratégie, le bouton bascule de configuration a été supprimé et les applications clientes auxquelles la stratégie s’applique sont sélectionnées.

> [!IMPORTANT]
> Les connexions à partir des clients d’authentification hérités ne prennent pas en charge MFA ni ne transmettent d’informations relatives à l’état de l’appareil à Azure AD. Elles seront donc bloquées par les contrôles d’octroi d’accès conditionnel, comme MFA ou la conformité des appareils. Si vous avez des comptes qui doivent utiliser l’authentification héritée, vous devez soit exclure ces comptes de la stratégie, soit configurer la stratégie pour qu’elle s’applique uniquement aux clients d’authentification modernes.

Lorsqu’il est réglé sur **Oui**, le bouton bascule **Configuration** s’applique aux éléments cochés, tandis que lorsqu’il est réglé sur **Non**, il s’applique à toutes les applications clientes, y compris les clients d’authentification modernes et hérités. Ce bouton bascule n’apparaît pas dans les stratégies créées avant août 2020.

- Clients d’authentification moderne
   - Browser
      - Cette option inclut les applications web qui utilisent des protocoles comme SAML, WS-Federation, OpenID Connect, ou les services inscrits en tant que clients confidentiels OAuth.
   - Applications mobiles et clients de bureau
      -  Cette option inclut les applications comme les applications de bureau et de téléphone Office.
- Clients d’authentification hérités
   - Clients Exchange ActiveSync
      - Cela inclut toute utilisation du protocole EAS (Exchange ActiveSync).
      - Si une stratégie bloque l’utilisation d’Exchange ActiveSync, l’utilisateur concerné reçoit un e-mail de mise en quarantaine. Cet e-mail contient des informations sur la raison du blocage et fournit éventuellement des instructions de correction.
      - Les administrateurs peuvent appliquer la stratégie uniquement aux plateformes prises en charge (par exemple, iOS, Android et Windows) via l’API MS Graph d’accès conditionnel.
   - Autres clients
      - Cette option inclut les clients qui utilisent des protocoles d’authentification de base/hérités qui ne prennent pas en charge l’authentification moderne.
         - SMTP authentifié : utilisé par les clients POP et IMAP pour envoyer des e-mails.
         - Découverte automatique : utilisé par les clients Outlook et EAS pour rechercher des boîtes aux lettres dans Exchange Online et s’y connecter.
         - Exchange Online PowerShell : utilisé pour se connecter à Exchange Online à l’aide de PowerShell à distance. Si vous bloquez l’authentification de base pour Exchange Online PowerShell, vous devez utiliser le module Exchange Online PowerShell pour vous connecter. Pour obtenir des instructions, consultez [Se connecter à Exchange Online PowerShell à l’aide de l’authentification multifacteur](/powershell/exchange/exchange-online/connect-to-exchange-online-powershell/mfa-connect-to-exchange-online-powershell).
         - Exchange Web Services (EWS) : interface de programmation utilisée par Outlook, Outlook pour Mac et des applications tierces.
         - IMAP4 : utilisé par les clients de messagerie IMAP.
         - MAPI sur HTTP (MAPI/HTTP) : utilisé par Outlook 2010 et versions ultérieures.
         - Carnet d’adresses hors connexion (OAB) : copie des collections de listes d’adresses qui sont téléchargées et utilisées par Outlook.
         - Outlook Anywhere (RPC sur HTTP) : utilisé par Outlook 2016 et antérieur.
         - Service Outlook : utilisé par l’application Courrier et calendrier pour Windows 10.
         - POP3 : utilisé par les clients de messagerie POP.
         - Reporting Web Services : utilisé pour récupérer des données de rapports dans Exchange Online.

Ces conditions sont généralement utilisées pour exiger l’utilisation d’un appareil managé, bloquer l’authentification héritée ou encore bloquer les applications web et autoriser les applications mobiles ou de bureau.

### <a name="supported-browsers"></a>Navigateurs pris en charge

Ce paramètre fonctionne avec tous les navigateurs. Toutefois, pour satisfaire à une stratégie d’appareil, telle qu’une exigence d’appareil conforme, les systèmes d’exploitation et navigateurs suivants sont pris en charge :

| Système d''exploitation | Navigateurs |
| :-- | :-- |
| Windows 10 | Microsoft Edge, Internet Explorer, Chrome |
| Windows 8 / 8.1 | Internet Explorer, Chrome |
| Windows 7 | Internet Explorer, Chrome |
| iOS | Microsoft Edge, Intune Managed Browser, Safari |
| Android | Microsoft Edge, Intune Managed Browser, Chrome |
| Windows Phone | Microsoft Edge, Internet Explorer |
| Windows Server 2019 | Microsoft Edge, Internet Explorer, Chrome |
| Windows Server 2016 | Internet Explorer |
| Windows Server 2012 R2 | Internet Explorer |
| Windows Server 2008 R2 | Internet Explorer |
| macOS | Chrome, Safari |

> [!NOTE]
> Edge 85 ou version ultérieure exige que l’utilisateur soit connecté au navigateur pour transmettre correctement l’identité de l’appareil. Dans le cas contraire, il se comporte comme Chrome sans l’extension Comptes. Cette connexion peut ne pas se produire automatiquement dans un scénario de jonction Azure AD Hybride. 

#### <a name="why-do-i-see-a-certificate-prompt-in-the-browser"></a>Pourquoi une invite de saisie de certificat s’affiche-t-elle dans mon navigateur ?

Sur des systèmes Windows 7, iOS, Android et macOS, Azure AD identifie l’appareil à l’aide d’un certificat client, qui est approvisionné lorsque l’appareil est inscrit auprès d’Azure AD.  Lorsqu’un utilisateur se connecte pour la première fois via le navigateur, l’utilisateur est invité à sélectionner le certificat. Il doit sélectionner ce certificat avant d’utiliser le navigateur.

#### <a name="chrome-support"></a>Prise en charge Chrome

Dans **Windows 10 Creators Update (version 1703)** ou version ultérieure, la prise en charge de Chrome nécessite l’installation de l’[extension Comptes Windows 10](https://chrome.google.com/webstore/detail/windows-10-accounts/ppnbnpeolgkicgegkbkbjmhlideopiji). Cette extension est nécessaire lorsqu’une stratégie d’accès conditionnel exige des informations concernant l’appareil.

Pour déployer automatiquement cette extension sur les navigateurs Chrome, créez la clé de Registre suivante :

- Chemin d’accès HKEY_LOCAL_MACHINE\Software\Policies\Google\Chrome\ExtensionInstallForcelist
- Nom 1
- REG_SZ de type (String)
- Data ppnbnpeolgkicgegkbkbjmhlideopiji;https\://clients2.google.com/service/update2/crx

Pour la prise en charge de Chrome dans **Windows 8.1 et 7**, créez la clé de Registre suivante :

- Chemin d’accès HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Google\Chrome\AutoSelectCertificateForUrls
- Nom 1
- REG_SZ de type (String)
- Data {« pattern » : « https://device.login.microsoftonline.com », « Filter »:{« ISSUER »:{« CN » : « MS-Organization-Access »}}}

Ces navigateurs prennent en charge l’authentification des appareils, ce qui permet de les identifier et de les valider par rapport à une stratégie. La vérification de l’appareil échoue si le navigateur est en cours d’exécution en mode privé.

### <a name="supported-mobile-applications-and-desktop-clients"></a>Applications mobiles et clients de bureau pris en charge

Les organisations peuvent sélectionner **Applications mobiles et clients de bureau** comme application cliente.

Ce paramètre a un impact sur les tentatives d’accès provenant des applications mobiles et des clients de bureau suivants :

| Applications clientes | Service cible | Plateforme |
| --- | --- | --- |
| Application Dynamics CRM | Dynamics CRM | Windows 10, Windows 8.1, iOS et Android |
| Application de messagerie/calendrier/contacts, Outlook 2016, Outlook 2013 (avec l’authentification moderne)| Exchange Online | Windows 10 |
| Stratégie MFA et d’emplacement pour les applications. Les stratégies basées sur les appareils ne sont pas prises en charge.| Tout service d’application Mes applications | Android et iOS |
| Services Microsoft Teams, soit tous les services qui prennent en charge Microsoft Teams et toutes ses applications clientes : Bureau Windows, iOS, Android, WP et client web | Microsoft Teams | Windows 10, Windows 8.1, Windows 7, iOS, Android et macOS |
| Applications Office 2016, Office 2013 (avec authentification moderne), [client de synchronisation OneDrive](/onedrive/enable-conditional-access) | SharePoint | Windows 8.1, Windows 7 |
| Applications Office 2016, Universal Office, Office 2013 (avec authentification moderne), [client de synchronisation OneDrive](/onedrive/enable-conditional-access) | SharePoint Online | Windows 10 |
| Office 2016 (Word, Excel, PowerPoint, OneNote uniquement). | SharePoint | macOS |
| Office 2019| SharePoint | Windows 10, macOS |
| Applications mobiles Office | SharePoint | Android, iOS |
| Application Yammer Office | Yammer | Windows 10, iOS, Android |
| Outlook 2019 | SharePoint | Windows 10, macOS |
| Outlook 2016 (Office pour Mac OS) | Exchange Online | macOS |
| Outlook 2016, Outlook 2013 (avec authentification moderne), Skype Entreprise (avec authentification moderne) | Exchange Online | Windows 8.1, Windows 7 |
| Application Outlook Mobile | Exchange Online | Android, iOS |
| Application Power BI | Service Power BI | Windows 10, Windows 8.1, Windows 7, Android et iOS |
| Skype Entreprise | Exchange Online| Android, iOS |
| Application Visual Studio Team Services | Visual Studio Team Services | Windows 10, Windows 8.1, Windows 7, iOS, Android |

### <a name="exchange-activesync-clients"></a>Clients Exchange ActiveSync

- Les organisations peuvent uniquement sélectionner des clients Exchange ActiveSync lors de l’affectation d’une stratégie à des utilisateurs ou groupes. La sélection des options **Tous les utilisateurs**, **Tous les utilisateurs invités et externes** ou **Rôles d’annuaire** a pour effet de bloquer l’ensemble des utilisateurs.
- Quand une stratégie affectée à des clients Exchange ActiveSync est créée, **Exchange Online** doit être la seule application cloud affectée à la stratégie. 
- Les organisations peuvent limiter l’étendue de cette stratégie à des plateformes spécifiques à l’aide de la condition **Plateformes d’appareils**.

Si le contrôle d’accès affecté à la stratégie a l’option **Demander une application cliente approuvée** activée, l’utilisateur est invité à installer et à utiliser le client mobile Outlook. Si l’**authentification multifacteur** est exigée pour les utilisateurs, ceux-ci sont bloqués, car l’authentification de base ne prend pas en charge l’authentification multifacteur.

Pour plus d’informations, consultez les articles suivants :

- [Bloquer l’authentification héritée avec l’accès conditionnel](block-legacy-authentication.md)
- [Exiger des applications clientes approuvées avec l’accès conditionnel](app-based-conditional-access.md)

### <a name="other-clients"></a>Autres clients

En sélectionnant **Autres clients**, vous pouvez spécifier une condition affectant les applications qui utilisent l’authentification de base avec des protocoles de messagerie comme IMAP, MAPI, POP et SMTP ainsi que les applications Office plus anciennes qui n’utilisent pas l’authentification moderne.

## <a name="device-state-preview"></a>État de l’appareil (préversion)

Une organisation peut utiliser la condition État de l’appareil pour exclure de ses stratégies d’accès conditionnel les appareils joints à une version hybride d’Azure AD et/ou les appareils marqués comme conformes à une stratégie de conformité Microsoft Intune.

Par exemple, *Tous les utilisateurs* qui accèdent à l’application cloud *Microsoft Azure Management*, incluant **Tous les états d’appareils**, mais excluant **Appareil joint à une version hybride d’Azure AD** et **Appareil marqué comme conforme**, et pour *Contrôles d’accès*, **Bloquer**. 
   - Cet exemple crée une stratégie qui autorise l’accès à Microsoft Azure Management uniquement à partir d’appareils avec jointure hybride Azure AD ou d’appareils marqués comme étant conformes.

## <a name="next-steps"></a>Étapes suivantes

- [Accès conditionnel : Accorder](concept-conditional-access-grant.md)

- [Stratégies d’accès conditionnel courantes](concept-conditional-access-policy-common.md)

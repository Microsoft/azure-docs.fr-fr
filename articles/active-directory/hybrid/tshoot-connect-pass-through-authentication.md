---
title: 'Azure AD Connect : Résoudre les problèmes de l’authentification directe | Microsoft Docs'
description: Cet article décrit la résolution des problèmes d’authentification directe d’Azure Active Directory (Azure AD).
services: active-directory
keywords: Résolution des problèmes d’authentification directe Azure AD Connect, installation d’Active Directory, composants requis pour Azure AD, SSO, Authentification unique
documentationcenter: ''
author: billmath
manager: daveba
ms.assetid: 9f994aca-6088-40f5-b2cc-c753a4f41da7
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 07/27/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 251f9a2b075189f19b9e943ff660baaba93ec33b
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97652039"
---
# <a name="troubleshoot-azure-active-directory-pass-through-authentication"></a>Résolution des problèmes d’authentification directe Azure Active Directory

Cet article fournit des informations sur les problèmes courants liés à l’authentification directe Azure AD.

>[!IMPORTANT]
>Si vous êtes confronté à des problèmes de connexion utilisateur avec l’authentification directe, ne désactivez pas la fonctionnalité. En outre, ne désinstallez pas les agents d’authentification directe si vous ne disposez pas d’un compte d’administrateur général cloud comme solution de secours. Découvrez comment [ajouter un compte d’administrateur général de type cloud uniquement](../fundamentals/add-users-azure-active-directory.md). Cette étape est essentielle pour éviter que votre locataire ne soit verrouillé.

## <a name="general-issues"></a>Problèmes d’ordre général

### <a name="check-status-of-the-feature-and-authentication-agents"></a>Vérifiez l’état de la fonctionnalité et des agents d’authentification

Assurez-vous que la fonctionnalité d’authentification directe est toujours **activée** sur votre locataire et l’état de l’agent d’authentification indique **actif** et non **inactif**. Vous pouvez vérifier l’état en accédant au panneau **Azure AD Connect** dans le [Centre d’administration Azure Active Directory](https://aad.portal.azure.com/).

![Centre d’administration Azure Active Directory - panneau Azure AD Connect](./media/tshoot-connect-pass-through-authentication/pta7.png)

![Centre d’administration Azure Active Directory - panneau Authentification directe](./media/tshoot-connect-pass-through-authentication/pta11.png)

### <a name="user-facing-sign-in-error-messages"></a>Message d’erreur lors de la connexion utilisateur

Si l’utilisateur ne peut pas se connecter avec l’authentification directe, l’un des messages d’erreur suivants peut s’afficher sur l’écran de connexion Azure AD : 

|Error|Description|Résolution
| --- | --- | ---
|AADSTS80001|Impossible de se connecter à Active Directory|Assurez-vous que les serveurs des agents sont membres de la même forêt Active Directory que les utilisateurs dont les mots de passe doivent être validés, et qu’ils peuvent se connecter à Active Directory.  
|AADSTS8002|Délai d’attente dépassé lors de la connexion à Active Directory|Vérifiez qu’Active Directory est disponible et répond aux demandes des agents.
|AADSTS80004|Le nom d’utilisateur envoyé à l’agent n’était pas valide|Vérifiez que l’utilisateur tente de se connecter avec le nom d’utilisateur correct.
|AADSTS80005|La validation a rencontré une WebException imprévisible|Erreur temporaire. Relancez la requête. Si l’erreur se reproduit, contactez le Support Microsoft.
|AADSTS80007|Une erreur s’est produite lors de la communication avec Active Directory|Consultez les journaux d’activité de l’agent pour plus d’informations, et vérifiez qu’Active Directory fonctionne comme prévu.

### <a name="users-get-invalid-usernamepassword-error"></a>Les utilisateurs obtiennent une erreur de nom d’utilisateur/mot de passe non valide 

Cela peut se produire lorsque le nom d’utilisateur local UserPrincipalName (UPN) d’un utilisateur est différent de l’UPN cloud de l’utilisateur.

Pour confirmer qu’il s’agit du problème, commencez par vérifier que l’agent d’authentification directe fonctionne correctement :


1. Créez un compte de test.  
2. Importez le module PowerShell sur l’ordinateur de l’agent :

 ```powershell
 Import-Module "C:\Program Files\Microsoft Azure AD Connect Authentication Agent\Modules\PassthroughAuthPSModule\PassthroughAuthPSModule.psd1"
 ```
3. Exécutez la commande Invoke PowerShell : 

 ```powershell
 Invoke-PassthroughAuthOnPremLogonTroubleshooter 
 ``` 
4. Lorsque vous êtes invité à entrer des informations d’identification, entrez les mêmes nom d’utilisateur et mot de passe que ceux utilisés pour vous connecter à https://login.microsoftonline.com).

Si vous obtenez la même erreur de nom d’utilisateur/mot de passe, cela signifie que l’agent d’authentification directe fonctionne correctement et que le problème est que l’UPN local n’est peut-être pas routable. Pour en savoir plus, consultez [Configuration d’un ID secondaire de connexion](/windows-server/identity/ad-fs/operations/configuring-alternate-login-id).

> [!IMPORTANT]
> Si le serveur Azure AD Connect n’est pas joint à un domaine (condition mentionnée dans les [prérequis d’installation d’Azure AD Connect](./how-to-connect-install-prerequisites.md#installation-prerequisites)), le problème de nom d’utilisateur/mot de passe non valide survient.

### <a name="sign-in-failure-reasons-on-the-azure-active-directory-admin-center-needs-premium-license"></a>Raisons des échecs de connexion dans le Centre d’administration Azure Active Directory (licence Premium requise)

Si votre locataire dispose d’une licence Azure AD Premium, vous pouvez également consulter le [rapport d’activité de connexion](../reports-monitoring/concept-sign-ins.md) dans le [Centre d’administration Azure Active Directory](https://aad.portal.azure.com/).

![Centre d’administration Azure Active Directory - rapport de connexions](./media/tshoot-connect-pass-through-authentication/pta4.png)

Accédez à **Azure Active Directory** -> **Connexions** dans le [Centre d’administration Azure Active Directory](https://aad.portal.azure.com/), puis cliquez sur une activité de connexion. Recherchez le champ **Code d’erreur de connexion**. Notez la valeur de ce champ et cherchez dans le tableau suivant la raison de l’échec et la solution à appliquer :

|Code d’erreur de connexion|Raison de l’échec de connexion|Résolution
| --- | --- | ---
| 50144 | Le mot de passe Active Directory de l’utilisateur est arrivé à expiration. | Réinitialisez le mot de passe dans votre instance locale d’Active Directory.
| 80001 | Agents d’authentification non disponibles. | Installez l’agent d’authentification, puis procédez à son inscription.
| 80002 | La demande de validation du mot de passe de l’Agent d’authentification est arrivée à expiration. | Vérifiez que l’agent d’authentification peut accéder à Active Directory.
| 80003 | Réponse non valide reçue par l’Agent d’authentification. | Si le problème se produit pour tous les utilisateurs, vérifiez votre configuration Active Directory.
| 80004 | Nom d’utilisateur principal (UPN) incorrect utilisé dans la demande de connexion. | Demandez à l’utilisateur de se connecter avec le bon nom d’utilisateur.
| 80005 | Agent d’authentification : Une erreur s’est produite. | Erreur temporaire. Réessayez plus tard.
| 80007 | L’Agent d’authentification ne peut pas se connecter à Active Directory. | Vérifiez que l’agent d’authentification peut accéder à Active Directory.
| 80010 | L’Agent d’authentification ne peut pas déchiffrer le mot de passe. | Si le problème se produit régulièrement, installez un nouvel agent d’authentification, puis inscrivez-le. Veillez à désinstaller l’agent actuel. 
| 80011 | L’Agent d’authentification n’a pas pu récupérer la clé de déchiffrement. | Si le problème se produit régulièrement, installez un nouvel agent d’authentification, puis inscrivez-le. Veillez à désinstaller l’agent actuel.
| 80014 | La demande de validation a répondu après dépassement du temps maximal. | L’agent d’authentification a expiré. Ouvrez un ticket de support avec le code d’erreur, l’ID de corrélation et l’horodatage pour obtenir plus de détails sur cette erreur.

>[!IMPORTANT]
>Les Agents d'authentification directe authentifient les utilisateurs Azure AD en vérifiant leur nom d'utilisateur et leur mot de passe par rapport à Active Directory et en appelant l'API [Win32 LogonUser](/windows/win32/api/winbase/nf-winbase-logonusera). Par conséquent, si vous avez configuré le paramètre « Connexion à » d'Active Directory de manière à limiter l'accès des stations de travail, vous devez également ajouter les serveurs qui hébergent les Agents d'authentification directe à la liste des serveurs « Connexion à ». Si vous ne le faites pas, vos utilisateurs ne pourront pas se connecter à Azure AD.

## <a name="authentication-agent-installation-issues"></a>Problèmes d’installation de l’agent d’authentification

### <a name="an-unexpected-error-occurred"></a>Une erreur inattendue s’est produite

[Collectez les journaux d’activité de l’agent](#collecting-pass-through-authentication-agent-logs) à partir du serveur et contactez le Support Microsoft pour lui faire part de votre problème.

## <a name="authentication-agent-registration-issues"></a>Problèmes d’inscription de l’agent d’authentification

### <a name="registration-of-the-authentication-agent-failed-due-to-blocked-ports"></a>L’inscription de l’agent a échoué en raison de ports bloqués

Vérifiez que le serveur sur lequel l’agent a été installé peut communiquer avec nos URL de service et les ports répertoriés [ici](how-to-connect-pta-quick-start.md#step-1-check-the-prerequisites).

### <a name="registration-of-the-authentication-agent-failed-due-to-token-or-account-authorization-errors"></a>Échec de l’inscription de l’agent d’authentification en raison d’erreurs d’autorisations liées au jeton ou au compte

Veillez à utiliser un compte d’administrateur général cloud pour toutes les opérations d’installation et d’inscription Azure AD Connect ou d’agent d’authentification autonome. Il existe un problème connu avec les comptes d’administrateur général compatibles MFA ; désactivez MFA temporairement (uniquement pour effectuer les opérations) comme solution de contournement.

### <a name="an-unexpected-error-occurred"></a>Une erreur inattendue s’est produite

[Collectez les journaux d’activité de l’agent](#collecting-pass-through-authentication-agent-logs) à partir du serveur et contactez le Support Microsoft pour lui faire part de votre problème.

## <a name="authentication-agent-uninstallation-issues"></a>Problèmes de désinstallation de l’agent d’authentification

### <a name="warning-message-when-uninstalling-azure-ad-connect"></a>Message d’avertissement lors de la désinstallation d’Azure AD Connect

Si l’authentification directe est activée sur votre locataire et que vous essayez de désinstaller Azure AD Connect, le message d’avertissement suivant s’affiche : « Les utilisateurs ne pourront pas se connecter à Azure AD, sauf si vous avez d’autres agents d’authentification directe installés sur d’autres serveurs. »

Vous devez avoir configuré la [haute disponibilité](how-to-connect-pta-quick-start.md#step-4-ensure-high-availability) avant de désinstaller Azure AD Connect pour éviter d’interrompre la connexion utilisateur.

## <a name="issues-with-enabling-the-feature"></a>Problèmes liés à l’activation de la fonctionnalité

### <a name="enabling-the-feature-failed-because-there-were-no-authentication-agents-available"></a>L’activation de la fonctionnalité a échoué, car il n’y a aucun agent d’authentification disponible

Vous devez disposer d’au moins un agent d’authentification actif pour activer l’authentification directe sur votre locataire. Vous pouvez installer un agent d’authentification en installant Azure AD Connect ou un agent d’authentification autonome.

### <a name="enabling-the-feature-failed-due-to-blocked-ports"></a>L’activation de la fonctionnalité a échoué en raison de ports bloqués

Assurez-vous que le serveur sur lequel Azure AD Connect est installé peut communiquer avec nos URL de service et les ports répertoriés [ici](how-to-connect-pta-quick-start.md#step-1-check-the-prerequisites).

### <a name="enabling-the-feature-failed-due-to-token-or-account-authorization-errors"></a>L’activation de la fonctionnalité a échoué en raison d’erreurs d’autorisations liées au jeton ou au compte

Veillez à utiliser un compte d’administrateur général pour le cloud uniquement lors de l’activation de la fonctionnalité. Il existe un problème connu avec les comptes d’administrateur général compatibles MFA (Multi-Factor Authentication) ; désactivez MFA temporairement (uniquement pour effectuer l’opération) comme solution de contournement.

## <a name="collecting-pass-through-authentication-agent-logs"></a>Collecte des journaux d’activité des agents d’authentification directe

Selon le type de problème rencontré, vous devez regarder à différents emplacements pour trouver les journaux d’activité des agents d’authentification directe.

### <a name="azure-ad-connect-logs"></a>Journaux d’activité Azure AD Connect

Pour vérifier les erreurs liées à l’installation, consultez les journaux d’activité Azure AD Connect à l’emplacement suivant : **%ProgramData%\AADConnect\trace-\*.log**.

### <a name="authentication-agent-event-logs"></a>Journaux des événements des agents d’authentification

Pour les erreurs relatives à l’agent d’authentification, ouvrez l’application Observateur d’événements sur le serveur et cherchez dans **Application and Service Logs\Microsoft\AzureAdConnect\AuthenticationAgent\Admin**.

Pour une analyse détaillée, activez le journal « Session » (pour accéder à cette option, cliquez avec le bouton droit de la souris sur l'application Observateur d'événements). N’exécutez pas l’agent d’authentification lorsque ce journal est activé pendant le fonctionnement normal. Utilisez-le uniquement pour la résolution des problèmes. Le contenu du journal n’est visible qu’une fois celui-ci désactivé.

Le manifeste d’événements de l’agent PTA est disponible [ici](https://msazure.visualstudio.com/One/_git/AD-AppProxy?path=%2Fsrc%2FProduct%2FMUC%2FPTADiagnosticsResource%2FPTADiagnosticsResource%2FPTAConnectorDiagnosticsResource%2FPTAConnectorEventManifest.man&_a=contents&version=GBmaster).

### <a name="detailed-trace-logs"></a>Journaux d’activité de suivi détaillés

Pour résoudre les problèmes d’échec de connexion de l’utilisateur, consultez les journaux d’activité de suivi situés à l’emplacement **%ProgramData%\Microsoft\Azure AD Connect Authentication Agent\Trace\\** . Ces journaux d’activité incluent les raisons de l’échec de connexion utilisateur à l’aide de la fonctionnalité d’authentification directe. Le tableau précédent des raisons des échecs de connexion établit une correspondance entre ces erreurs et les raisons des échecs de connexion. Vous trouverez ci-dessous un exemple d’entrée de journal :

```
    AzureADConnectAuthenticationAgentService.exe Error: 0 : Passthrough Authentication request failed. RequestId: 'df63f4a4-68b9-44ae-8d81-6ad2d844d84e'. Reason: '1328'.
        ThreadId=5
        DateTime=xxxx-xx-xxTxx:xx:xx.xxxxxxZ
```

Vous pouvez obtenir une description détaillée de l’erreur (« 1328 » dans l’exemple précédent) en ouvrant l’invite de commandes et en exécutant la commande suivante (Remarque : vous devez remplacer « 1328 » par le numéro d’erreur réel apparaissant dans vos journaux d’activité) :

`Net helpmsg 1328`

![Authentification directe](./media/tshoot-connect-pass-through-authentication/pta3.png)

### <a name="domain-controller-logs"></a>Journaux d’activité des contrôleurs de domaine

Si la journalisation d’audit est activée, des informations complémentaires se trouvent dans les journaux d’activité de sécurité des contrôleurs de domaine. Vous pouvez interroger les demandes de connexion envoyées par les agents d’authentification directe en utilisant la méthode suivante :

```
    <QueryList>
    <Query Id="0" Path="Security">
    <Select Path="Security">*[EventData[Data[@Name='ProcessName'] and (Data='C:\Program Files\Microsoft Azure AD Connect Authentication Agent\AzureADConnectAuthenticationAgentService.exe')]]</Select>
    </Query>
    </QueryList>
```

## <a name="performance-monitor-counters"></a>Compteurs Analyseur de performances

Un autre moyen de surveiller les agents d’authentification consiste à effectuer un suivi de compteurs Analyseur de performances spécifiques sur chaque serveur où est installé l’agent d’authentification. Utilisez les compteurs globaux suivants ( **#PTA Authentifications**, **#PTA Authentifications ayant échoué** et **#PTA Authentifications réussies**), ainsi que les compteurs d’erreurs ( **#PTA Erreurs d’authentification**) :

![Compteurs Analyseur de performances de l’authentification directe](./media/tshoot-connect-pass-through-authentication/pta12.png)

>[!IMPORTANT]
>L’authentification directe assure la haute disponibilité à l’aide de plusieurs agents d’authentification, et _non_ de l’équilibrage de charge. Selon votre configuration, les agents d’authentification ne reçoivent _pas_ tous le _même_ nombre exact de demandes. Il est possible qu’un agent d’authentification spécifique ne reçoive aucun trafic du tout.

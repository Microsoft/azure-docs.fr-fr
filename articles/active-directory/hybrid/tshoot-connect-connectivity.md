---
title: 'Azure AD Connect : Résoudre les problèmes de connectivité liés à Azure AD | Microsoft Docs'
description: Explique comment résoudre les problèmes de connectivité liés à Azure AD Connect.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 3aa41bb5-6fcb-49da-9747-e7a3bd780e64
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 04/25/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.custom: has-adal-ref
ms.openlocfilehash: 56e9820c5e3a750a35b7271b86750df00eb4784e
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92677068"
---
# <a name="troubleshoot-azure-ad-connectivity"></a>Résoudre les problèmes de connectivité liés à Azure AD
Cet article décrit le fonctionnement de la connectivité entre Azure AD Connect et Azure AD ainsi que la résolution des problèmes de connectivité. Ces problèmes sont susceptibles de se produire dans un environnement doté d’un serveur proxy.

## <a name="troubleshoot-connectivity-issues-in-the-installation-wizard"></a>Résoudre les problèmes de connectivité dans l’Assistant Installation
Azure AD Connect utilise l’authentification moderne (à l’aide de la bibliothèque ADAL) pour l’authentification. L’Assistant Installation et le moteur de synchronisation approprié requièrent que le fichier machine.config soit correctement configuré dans la mesure où ces deux éléments sont des applications .NET.

Dans cet article, nous montrons comment Fabrikam se connecte à Azure AD via son proxy. Le serveur proxy est nommé fabrikamproxy et utilise le port 8080.

Tout d’abord, nous devons nous assurer que [**machine.config**](how-to-connect-install-prerequisites.md#connectivity) est correctement configuré et que le **service Microsoft Azure AD Sync** a été redémarré une fois que le fichier machine.config a été mis à jour.
![Capture d’écran montrant la partie du fichier de configuration machine point.](./media/tshoot-connect-connectivity/machineconfig.png)

> [!NOTE]
> Dans certains blogs non-Microsoft, il est indiqué qu’il convient plutôt d’apporter des modifications à miiserver.exe.config. Cependant, ce fichier est remplacé à chaque mise à niveau. Ainsi, même s’il fonctionne lors de l’installation initiale, le système s’arrête de fonctionner à la première mise à niveau. C’est pourquoi il est recommandé de mettre à jour le fichier machine.config.
>
>

Les URL requises doivent également être ouvertes dans le serveur proxy. La liste officielle est documentée dans [URL et plages d’adresses IP Office 365 ](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2).

Parmi ces URL, le tableau suivant indique celles qui représentent le strict minimum pour pouvoir se connecter à Azure AD. Cette liste n’inclut pas de fonctionnalités facultatives, comme la réécriture du mot de passe ou Azure AD Connect Health. Elle est documentée ici pour faciliter le dépannage de la configuration initiale.

| URL | Port | Description |
| --- | --- | --- |
| mscrl.microsoft.com |HTTP/80 |Permet de télécharger des listes de révocation de certificats. |
| \*.verisign.com |HTTP/80 |Permet de télécharger des listes de révocation de certificats. |
| \*.entrust.net |HTTP/80 |Permet de télécharger des listes de révocation de certificats pour MFA. |
| \*.windows.net |HTTPS/443 |Permet de se connecter à Azure AD. |
| secure.aadcdn.microsoftonline-p.com |HTTPS/443 |Utilisé pour MFA. |
| \*.microsoftonline.com |HTTPS/443 |Permet de configurer votre annuaire Azure AD et pour importer/exporter des données. |
| \*.crl3.digicert.com |HTTP/80 |Utilisé pour vérifier les certificats. |
| \*.crl4.digicert.com |HTTP/80 |Utilisé pour vérifier les certificats. |
| \*.ocsp.digicert.com |HTTP/80 |Utilisé pour vérifier les certificats. |
| \*. www.d-trust.net |HTTP/80 |Utilisé pour vérifier les certificats. |
| \*.root-c3-ca2-2009.ocsp.d-trust.net |HTTP/80 |Utilisé pour vérifier les certificats. |
| \*.crl.microsoft.com |HTTP/80 |Utilisé pour vérifier les certificats. |
| \*.oneocsp.microsoft.com |HTTP/80 |Utilisé pour vérifier les certificats. |
| \*.ocsp.msocsp.com |HTTP/80 |Utilisé pour vérifier les certificats. |

## <a name="errors-in-the-wizard"></a>Erreurs dans l’Assistant
L’Assistant Installation utilise deux contextes de sécurité différents. Dans la page **Connexion à Azure AD** , il utilise l’utilisateur actuellement connecté. Dans la page **Configurer** , il passe au [compte exécutant le service pour le moteur de synchronisation](reference-connect-accounts-permissions.md#adsync-service-account). S’il existe un problème, il apparaît probablement déjà au niveau de la page **Connexion à Azure AD** page de l’Assistant car la configuration du proxy est globale.

Voici les erreurs les plus courantes de l’Assistant Installation.

### <a name="the-installation-wizard-has-not-been-correctly-configured"></a>L’Assistant Installation n’a pas été configuré correctement
Cette erreur apparaît quand l’Assistant ne peut pas accéder au proxy.
![Capture d’écran montrant une erreur : Impossible de valider les informations d’identification.](./media/tshoot-connect-connectivity/nomachineconfig.png)

* Si vous voyez cette erreur, vérifiez que le fichier [machine.config](how-to-connect-install-prerequisites.md#connectivity) a été configuré correctement.
* Si la configuration semble correcte, suivez les étapes de la section [Vérifier la connectivité du proxy](#verify-proxy-connectivity) pour voir si le problème existe également en dehors de l’Assistant.

### <a name="a-microsoft-account-is-used"></a>Un compte Microsoft est utilisé
Si vous utilisez un **compte Microsoft** au lieu d’un compte **scolaire ou d’organisation** , vous voyez une erreur générique.
![Un compte Microsoft est utilisé](./media/tshoot-connect-connectivity/unknownerror.png)

### <a name="the-mfa-endpoint-cannot-be-reached"></a>Impossible d’atteindre le point de terminaison de l’authentification MFA
Cette erreur s’affiche si le point de terminaison **https://secure.aadcdn.microsoftonline-p.com** n’est pas accessible et que votre administration globale a une MFA activée.
![nomachineconfig](./media/tshoot-connect-connectivity/nomicrosoftonlinep.png)

* Si vous voyez cette erreur, vérifiez que le point de terminaison **secure.aadcdn.microsoftonline-p.com** a été ajouté au proxy.

### <a name="the-password-cannot-be-verified"></a>Le mot de passe ne peut pas être vérifié
Si l’Assistant Installation réussit à se connecter à Azure AD, mais que le mot de passe lui-même ne peut pas être vérifié, le message suivant apparaît : ![Mot de passe incorrect](./media/tshoot-connect-connectivity/badpassword.png).

* Le mot de passe est-il temporaire et doit-il être modifié ? Le mot de passe est-il correct ? Connectez-vous à `https://login.microsoftonline.com` (sur un autre ordinateur que le serveur Azure AD Connect) et vérifiez que le compte est utilisable.

### <a name="verify-proxy-connectivity"></a>Vérifier la connectivité du proxy
Pour vérifier si le serveur Azure AD Connect dispose d’une connectivité réelle avec le proxy et Internet, utilisez quelques commandes PowerShell pour voir si le proxy autorise ou non les demandes web. À une invite PowerShell, exécutez `Invoke-WebRequest -Uri https://adminwebservice.microsoftonline.com/ProvisioningService.svc`. (Techniquement, le premier appel est vers `https://login.microsoftonline.com` et cet URI fonctionne également, mais l’autre URI répond plus rapidement.)

PowerShell utilise la configuration du fichier machine.config pour contacter le proxy. Les paramètres de winhttp/netsh ne doivent pas affecter ces applets de commande.

Si le proxy est configuré correctement, vous devez obtenir un état de réussite :  ![Capture d’écran montrant l’état de réussite lorsque le proxy est configuré correctement.](./media/tshoot-connect-connectivity/invokewebrequest200.png)

Si vous recevez l’erreur **Impossible de se connecter au serveur distant** , cela signifie que PowerShell tente d’effectuer un appel direct sans utiliser le proxy ou que DNS n’est pas configuré correctement. Vérifiez que le fichier **machine.config** est configuré correctement.
![unabletoconnect](./media/tshoot-connect-connectivity/invokewebrequestunable.png)

Si le proxy n’est pas configuré correctement, une erreur apparaît : ![proxy200](./media/tshoot-connect-connectivity/invokewebrequest403.png)
![proxy407](./media/tshoot-connect-connectivity/invokewebrequest407.png)

| Error | Texte d’erreur | Commentaire |
| --- | --- | --- |
| 403 |Interdit |Le proxy n’a pas été ouvert pour l’URL demandée. Revisitez la configuration du proxy et vérifiez que les [URL](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2) ont été ouvertes. |
| 407 |Authentification proxy requise |Le serveur proxy nécessitait une connexion et aucune n’a été fournie. Si le serveur proxy nécessite une authentification, vérifiez que ce paramètre a été configuré dans le fichier machine.config. Vérifiez également que vous utilisez des comptes de domaine pour l’utilisateur qui exécute l’Assistant et pour le compte de service. |

### <a name="proxy-idle-timeout-setting"></a>Définition du délai d'inactivité du proxy
Quand Azure AD Connect envoie une demande d'exportation à Azure AD, Azure AD peut prendre jusqu'à 5 minutes pour traiter la demande avant de générer une réponse. Cela peut se produire en particulier si plusieurs objets de groupe appartiennent à de grands groupes inclus dans la même demande d'exportation. Assurez-vous que le délai d'inactivité du proxy est supérieur à 5 minutes. Sinon, vous risquez de rencontrer un problème de connectivité intermittente avec Azure AD sur le serveur Azure AD Connect.

## <a name="the-communication-pattern-between-azure-ad-connect-and-azure-ad"></a>Modèle de communication entre Azure AD Connect et Azure AD
Si vous avez suivi l’ensemble des étapes précédentes et que vous ne pouvez toujours pas vous connecter, vous pouvez commencer à examiner les journaux d’activité du réseau. Cette section documente un modèle de connectivité réussi et normal. Elle indique également les fausses pistes courantes qui peuvent être ignorées quand vous lisez les journaux d’activité du réseau.

* Il s’agit d’appels vers `https://dc.services.visualstudio.com`. Il n’est pas impératif que cette URL soit ouverte dans le proxy pour que l’installation réussisse, et vous pouvez ignorer ces appels.
* Vous constatez que la résolution DNS répertorie les hôtes réels de l’espace de noms DNS nsatc.net et d’autres espaces de noms ne figurant pas sous microsoftonline.com. Néanmoins, il n’existe pas de demandes de service web sur les noms de serveur réels, et vous n’avez pas à ajouter ces URL au proxy.
* Les points de terminaison adminwebservice et provisioningapi sont des points de terminaison de découverte, et ils sont utilisés pour rechercher le point de terminaison réel à utiliser. Ces points de terminaison diffèrent selon votre région.

### <a name="reference-proxy-logs"></a>Journaux d’activité de proxy de référence
Voici une image mémoire d’un journal de proxy réel et la page de l’Assistant Installation d’où elle a été prise (les entrées en double pour un même point de terminaison ont été supprimées). Cette section peut être utilisée comme référence pour vos propres journaux d’activité de proxy et de réseau. Les points de terminaison réels peuvent être différents dans votre environnement (en particulier les URL en *italique* ).

**Connexion à Azure AD**

| Temps | URL |
| --- | --- |
| 11/01/2016 8:31 |connect://login.microsoftonline.com:443 |
| 11/01/2016 8:31 |connect://adminwebservice.microsoftonline.com:443 |
| 11/01/2016 8:32 |connect:// *bba800-anchor*.microsoftonline.com:443 |
| 11/01/2016 8:32 |connect://login.microsoftonline.com:443 |
| 11/01/2016 8:33 |connect://provisioningapi.microsoftonline.com:443 |
| 11/01/2016 8:33 |connect:// *bwsc02-relay*.microsoftonline.com:443 |

**Configurer**

| Temps | URL |
| --- | --- |
| 11/01/2016 8:43 |connect://login.microsoftonline.com:443 |
| 11/01/2016 8:43 |connect:// *bba800-anchor*.microsoftonline.com:443 |
| 11/01/2016 8:43 |connect://login.microsoftonline.com:443 |
| 11/01/2016 8:44 |connect://adminwebservice.microsoftonline.com:443 |
| 11/01/2016 8:44 |connect:// *bba900-anchor*.microsoftonline.com:443 |
| 11/01/2016 8:44 |connect://login.microsoftonline.com:443 |
| 11/01/2016 8:44 |connect://adminwebservice.microsoftonline.com:443 |
| 11/01/2016 8:44 |connect:// *bba800-anchor*.microsoftonline.com:443 |
| 11/01/2016 8:44 |connect://login.microsoftonline.com:443 |
| 11/01/2016 8:46 |connect://provisioningapi.microsoftonline.com:443 |
| 11/01/2016 8:46 |connect:// *bwsc02-relay*.microsoftonline.com:443 |

**Synchronisation initiale**

| Temps | URL |
| --- | --- |
| 11/01/2016 8:48 |connect://login.windows.net:443 |
| 11/01/2016 8:49 |connect://adminwebservice.microsoftonline.com:443 |
| 11/01/2016 8:49 |connect:// *bba900-anchor*.microsoftonline.com:443 |
| 11/01/2016 8:49 |connect:// *bba800-anchor*.microsoftonline.com:443 |

## <a name="authentication-errors"></a>Erreurs d’authentification
Cette section décrit les erreurs qui peuvent être retournées par la bibliothèque ADAL (la bibliothèque d’authentification utilisée par Azure AD Connect) et PowerShell. L’erreur expliquée doit vous aider à comprendre les étapes suivantes.

### <a name="invalid-grant"></a>Licence non valide
Nom d’utilisateur ou mot de passe non valide. Pour plus d’informations, consultez [Impossible de vérifier le mot de passe](#the-password-cannot-be-verified).

### <a name="unknown-user-type"></a>Type d’utilisateur inconnu
Votre annuaire Azure AD est introuvable ou n’a pas pu être résolu. Vous essayez peut-être de vous connecter avec un nom d’utilisateur d’un domaine non vérifié ?

### <a name="user-realm-discovery-failed"></a>Échec de la découverte de domaine d’utilisateur
Problèmes de configuration du réseau ou du proxy. Impossible d’accéder au réseau. Consultez [Résoudre les problèmes de connectivité dans l’Assistant Installation](#troubleshoot-connectivity-issues-in-the-installation-wizard).

### <a name="user-password-expired"></a>Mot de passe utilisateur expiré
Vos informations d’identification ont expiré. Modifiez votre mot de passe.

### <a name="authorization-failure"></a>Échec de l’autorisation
L’autorisation d’effectuer une action dans Azure AD n’a pas pu être accordée à l’utilisateur.

### <a name="authentication-canceled"></a>Authentification annulée
Le test d’authentification multifacteur (MFA) a été annulé.

<div id="connect-msolservice-failed">
<!--
  Empty div just to act as an alias for the "Connect To MS Online Failed" header
  because we used the mentioned id in the code to jump to this section.
-->
</div>

### <a name="connect-to-ms-online-failed"></a>Connect To MS Online Failed (Échec de la connexion à MS Online)
L’authentification a réussi, mais Azure AD PowerShell a un problème d’authentification.

<div id="get-msoluserrole-failed">
<!--
  Empty div just to act as an alias for the "Azure AD Global Admin Role Needed" header
  because we used the mentioned id in the code to jump to this section.
-->
</div>

### <a name="azure-ad-global-admin-role-needed"></a>(Azure AD Global Admin Role Needed) Rôle d’administrateur général Azure AD nécessaire
L’utilisateur s’est authentifié correctement. Toutefois, le rôle d’administrateur général n’est pas attribué à l’utilisateur. Voici [comment vous pouvez attribuer le rôle d’administrateur général](../roles/permissions-reference.md) à l’utilisateur.

<div id="privileged-identity-management">
<!--
  Empty div just to act as an alias for the "Privileged Identity Management Enabled" header
  because we used the mentioned id in the code to jump to this section.
-->
</div>

### <a name="privileged-identity-management-enabled"></a>Privileged Identity Management Enabled (Technologie Privileged Identity Management activée)
L’authentification a réussi. Privileged Identity Management a été activé et vous n’êtes actuellement pas administrateur global. Pour plus d’informations, consultez [Privileged Identity Management](../privileged-identity-management/pim-getting-started.md).

<div id="get-msolcompanyinformation-failed">
<!--
  Empty div just to act as an alias for the "Company Information Unavailable" header
  because we used the mentioned id in the code to jump to this section.
-->
</div>

### <a name="company-information-unavailable"></a>Company Information Unavailable (Informations sur la société non disponibles)
L’authentification a réussi. Impossible de récupérer les informations de société à partir d’Azure AD.

<div id="get-msoldomain-failed">
<!--
  Empty div just to act as an alias for the "Domain Information Unavailable" header
  because we used the mentioned id in the code to jump to this section.
-->
</div>

### <a name="domain-information-unavailable"></a>Domain Information Unavailable (Informations de domaine non disponibles)
L’authentification a réussi. Impossible de récupérer les informations de domaine à partir d’Azure AD.

### <a name="unspecified-authentication-failure"></a>Unspecified Authentication Failure (Échec d’authentification non spécifié)
Apparaît comme une erreur inattendue dans l’Assistant Installation. Peut se produire si vous tentez d’utiliser un **compte Microsoft** au lieu d’un compte **scolaire ou d’organisation**.

## <a name="troubleshooting-steps-for-previous-releases"></a>Étapes de dépannage pour les versions précédentes.
Dans les versions commençant par le numéro de build 1.1.105.0 (publiées en février 2016), l’Assistant de connexion a été mis hors service. Cette section et la configuration ne sont plus requises, mais sont conservées en tant que référence.

Pour que l’Assistant de connexion unique fonctionne, winhttp doit être configuré. Cette configuration peut être effectuée avec [**netsh**](how-to-connect-install-prerequisites.md#connectivity).
![Capture d’écran montrant une fenêtre d’invite de commandes exécutant l’outil netsh pour définir un proxy.](./media/tshoot-connect-connectivity/netsh.png)

### <a name="the-sign-in-assistant-has-not-been-correctly-configured"></a>L’Assistant de connexion n’a pas été configuré correctement
Cette erreur apparaît quand l’Assistant de connexion ne peut pas accéder au proxy ou quand le proxy n’autorise pas la demande.
![Capture d’écran montrant une erreur : Impossible de valider les informations d'identification. Vérifiez la connectivité au réseau et les paramètres du pare-feu ou du proxy.](./media/tshoot-connect-connectivity/nonetsh.png)

* Si cette erreur apparaît, examinez la configuration du proxy dans [netsh](how-to-connect-install-prerequisites.md#connectivity) et vérifiez si elle est correcte.
  ![Capture d’écran montrant une fenêtre d’invite de commandes exécutant l’outil netsh pour afficher la configuration du proxy.](./media/tshoot-connect-connectivity/netshshow.png)
* Si la configuration semble correcte, suivez les étapes de la section [Vérifier la connectivité du proxy](#verify-proxy-connectivity) pour voir si le problème existe également en dehors de l’Assistant.

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur l’ [intégration de vos identités locales avec Azure Active Directory](whatis-hybrid-identity.md).

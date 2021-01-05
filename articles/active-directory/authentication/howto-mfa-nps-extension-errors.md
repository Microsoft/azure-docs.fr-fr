---
title: Résolution des problèmes liés à l'extension NPS d'Azure AD MFA - Azure Active Directory
description: Obtenez de l'aide pour résoudre les problèmes liés à l'extension NPS d'Azure AD Multi-Factor Authentication
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: troubleshooting
ms.date: 11/21/2019
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.custom: has-adal-ref
ms.openlocfilehash: fa3a2366f007ff1481e7c84f049e606586392037
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96742950"
---
# <a name="resolve-error-messages-from-the-nps-extension-for-azure-ad-multi-factor-authentication"></a>Résoudre les messages d'erreur liés à l'extension NPS d'Azure AD Multi-Factor Authentication

Si vous rencontrez des erreurs au niveau de l'extension NPS d'Azure AD Multi-Factor Authentication, utilisez cet article pour les résoudre plus rapidement. Les journaux d’activité d’extension NPS se trouvent dans l’Observateur d’événements sous **Affichages personnalisés** > **Rôles serveur** > **Services de stratégie et d’accès réseau** sur le serveur où l’extension NPS est installée.

## <a name="troubleshooting-steps-for-common-errors"></a>Étapes de résolution des erreurs courantes

| Code d'erreur | Étapes de dépannage |
| ---------- | --------------------- |
| **CONTACT_SUPPORT** | [Contactez le support technique](#contact-microsoft-support) et mentionnez la liste des étapes de collecte des journaux d’activité. Fournissez autant d’informations que possible sur ce qui s’est produit avant l’erreur, y compris l’ID de locataire et le nom d’utilisateur principal (UPN). |
| **CLIENT_CERT_INSTALL_ERROR** | Il peut y avoir un problème avec la façon dont le certificat client a été installé ou associé à votre locataire. Suivez les instructions de [Résolution des problèmes liés à l’extension NPS pour MFA](howto-mfa-nps-extension.md#troubleshooting) pour explorer les problèmes de certificat client. |
| **ESTS_TOKEN_ERROR** | Suivez les instructions de [Résolution des problèmes liés à l’extension NPS pour MFA](howto-mfa-nps-extension.md#troubleshooting) pour explorer les problèmes de certificat client et de jeton ADAL. |
| **HTTPS_COMMUNICATION_ERROR** | Le serveur NPS ne peut pas recevoir de réponses d'Azure AD MFA. Vérifiez que vos pare-feu sont ouverts en mode bidirectionnel et qu’ils autorisent le trafic vers et depuis https://adnotifications.windowsazure.com |
| **HTTP_CONNECT_ERROR** | Sur le serveur qui exécute l’extension de serveur NPS, vérifiez que vous pouvez atteindre `https://adnotifications.windowsazure.com` et `https://login.microsoftonline.com/`. Si ces sites ne se chargent pas, résolvez les problèmes de connectivité sur ce serveur. |
| **Extension NPS d'Azure AD MFA :** <br> L'extension NPS d'Azure AD MFA ne procède à une authentification secondaire que pour les demandes Radius dont l'état est AccessAccept. Demande reçue pour l’utilisateur « nom d’utilisateur » avec l’état de réponse AccessReject (demande ignorée). | Cette erreur reflète généralement un échec d’authentification dans AD ou le fait que le serveur NPS est incapable de recevoir des réponses d’Azure AD. Vérifiez que vos pare-feu sont ouverts en mode bidirectionnel et qu’ils autorisent le trafic vers et depuis `https://adnotifications.windowsazure.com` et `https://login.microsoftonline.com` sur les ports 80 et 443. Il est également important de vérifier que sous l’onglet DIAL-IN (Numérotation) d’Autorisation d’accès réseau, le paramètre est défini sur « Contrôler l’accès via la Stratégie d’accès à distance ». Cette erreur peut également se déclencher si l’utilisateur ne possède pas de licence. |
| **REGISTRY_CONFIG_ERROR** | Une clé est manquante dans le registre de l’application. Un [script PowerShell](howto-mfa-nps-extension.md#install-the-nps-extension) n’a peut-être pas été exécuté après l’installation. Ce message d’erreur doit inclure la clé manquante. Assurez-vous que la clé se trouve dans HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\AzureMfa. |
| **REQUEST_FORMAT_ERROR** <br> Attribut userName\Identifier Radius obligatoire manquant dans la demande Radius. Vérifiez que le serveur NPS reçoit les demandes RADIUS. | Cette erreur indique généralement un problème d’installation. L’extension NPS doit être installée sur des serveurs NPS pouvant recevoir des demandes RADIUS. Les serveurs NPS installés en tant que dépendances de services comme RDG et RRAS ne reçoivent pas les demandes RADIUS. L’extension NPS ne fonctionne pas dans le cadre d’une telle installation et renvoie une erreur, car elle ne peut pas lire les détails de la demande d’authentification. |
| **REQUEST_MISSING_CODE** | Assurez-vous que le protocole de chiffrement du mot de passe entre les serveurs NPS et NAS prend en charge la méthode d’authentification secondaire que vous utilisez. **PAP** prend en charge toutes les méthodes d'authentification d'Azure AD MFA dans le cloud : appel téléphonique, message texte à sens unique, notification d'application mobile et code de vérification d'application mobile. **CHAPv2** et **EAP** prennent en charge l’appel téléphonique et la notification d’application mobile. |
| **USERNAME_CANONICALIZATION_ERROR** | Vérifiez que l’utilisateur se trouve dans votre instance locale d’Active Directory et que le service NPS dispose des autorisations pour accéder au répertoire. Si vous utilisez des approbations inter-forêts, [contactez le support technique](#contact-microsoft-support) pour plus d’informations. |

### <a name="alternate-login-id-errors"></a>Erreurs d’ID de connexion de substitution

| Code d'erreur | Message d’erreur | Étapes de dépannage |
| ---------- | ------------- | --------------------- |
| **ALTERNATE_LOGIN_ID_ERROR** | Erreur : échec de la recherche userObjectSid | Vérifiez que l’utilisateur existe dans votre instance Active Directory locale. Si vous utilisez des approbations inter-forêts, [contactez le support technique](#contact-microsoft-support) pour plus d’informations. |
| **ALTERNATE_LOGIN_ID_ERROR** | Erreur : Échec de la recherche d’un ID de connexion de substitution | Vérifiez que LDAP_ALTERNATE_LOGINID_ATTRIBUTE est défini sur un [attribut Active Directory valide](/windows/win32/adschema/attributes-all). <br><br> Si LDAP_FORCE_GLOBAL_CATALOG est défini sur True, ou si LDAP_LOOKUP_FORESTS est configuré avec une valeur non vide, vérifiez que vous avez configuré un catalogue global et que l’attribut AlternateLoginId y est ajouté. <br><br> Si LDAP_LOOKUP_FORESTS est configuré avec une valeur non vide, vérifiez que la valeur est correcte. S’il existe plusieurs noms de forêt, ces noms doivent être séparés par des points-virgules, et non des espaces. <br><br> Si ces étapes ne résolvent pas le problème, [contactez le support technique](#contact-microsoft-support) pour plus d’informations. |
| **ALTERNATE_LOGIN_ID_ERROR** | Erreur : La valeur relative à l’ID de connexion de substitution est vide | Vérifiez que l’attribut AlternateLoginId est configuré pour l’utilisateur. |

## <a name="errors-your-users-may-encounter"></a>Erreurs que vos utilisateurs pourraient rencontrer

| Code d'erreur | Message d’erreur | Étapes de dépannage |
| ---------- | ------------- | --------------------- |
| **AccessDenied** | Le locataire de l’appelant n’a pas les autorisations d’accès pour l’authentification de l’utilisateur | Vérifiez que le domaine du locataire et le domaine du nom d’utilisateur principal sont identiques. Par exemple, assurez-vous que user@contoso.com essaie de s’authentifier sur le locataire Contoso. Le nom d’utilisateur principal représente un utilisateur valide du locataire dans Azure. |
| **AuthenticationMethodNotConfigured** | La méthode d’authentification spécifiée n’est pas configurée pour l’utilisateur | Invitez l’utilisateur à ajouter ou vérifier ses méthodes de vérification en suivant les instructions de la page [Gérer les paramètres de la vérification en deux étapes](../user-help/multi-factor-authentication-end-user-manage-settings.md). |
| **AuthenticationMethodNotSupported** | La méthode d’authentification spécifiée n’est pas prise en charge. | Effectuez la collecte de tous les journaux d’activité incluant cette erreur et [contactez le support technique](#contact-microsoft-support). Lorsque vous contactez le support technique, indiquez le nom d’utilisateur et la méthode de vérification secondaire ayant déclenché l’erreur. |
| **BecAccessDenied** | L’appel de MSODS Bec a retourné une erreur de type accès refusé. Le nom d’utilisateur n’est probablement pas défini dans le locataire | L’utilisateur est présent dans le répertoire Active Directory local, mais il n’est pas synchronisé dans Azure AD par AD Connect. Ou bien, l’utilisateur est manquant pour le locataire. Ajoutez l’utilisateur dans Azure AD et invitez-le à ajouter ses méthodes de vérification en suivant les instructions de la page [Gérer les paramètres de la vérification en deux étapes](../user-help/multi-factor-authentication-end-user-manage-settings.md). |
| **InvalidFormat** ou **StrongAuthenticationServiceInvalidParameter** | Le format du numéro de téléphone est inconnu | Demandez à l’utilisateur de corriger son numéro de téléphone de vérification. |
| **InvalidSession** | La session spécifiée n’est pas valide ou a expiré | La session a duré plus de trois minutes. Vérifiez que l’utilisateur entre le code de vérification ou répond à la notification de l’application dans les trois minutes suivant l’initiation de la demande d’authentification. Si le problème persiste, vérifiez qu'il n'y a aucune latence réseau entre le client, le serveur NAS, le serveur NPS et le point de terminaison Azure AD MFA.  |
| **NoDefaultAuthenticationMethodIsConfigured** | Aucune méthode d’authentification par défaut n’est configurée pour l’utilisateur | Invitez l’utilisateur à ajouter ou vérifier ses méthodes de vérification en suivant les instructions de la page [Gérer les paramètres de la vérification en deux étapes](../user-help/multi-factor-authentication-end-user-manage-settings.md). Vérifiez que l’utilisateur a choisi une méthode d’authentification par défaut et configuré cette méthode pour son compte. |
| **OathCodePinIncorrect** | Code et PIN saisis erronés. | Cette erreur ne devrait pas survenir avec l’extension NPS. Si l’utilisateur la rencontre, [contactez le support technique](#contact-microsoft-support) pour obtenir de l’aide. |
| **ProofDataNotFound** | Les données de vérification ne sont pas configurées pour la méthode d’authentification sélectionnée. | Invitez l’utilisateur à essayer une nouvelle méthode de vérification ou à en ajouter d’autres en suivant les instructions de la page [Gérer les paramètres de la vérification en deux étapes](../user-help/multi-factor-authentication-end-user-manage-settings.md). Si l’erreur persiste après que vous avez confirmé la bonne configuration de sa méthode de vérification, [contactez le support technique](#contact-microsoft-support). |
| **SMSAuthFailedWrongCodePinEntered** | Code et PIN saisis erronés. (OneWaySMS) | Cette erreur ne devrait pas survenir avec l’extension NPS. Si l’utilisateur la rencontre, [contactez le support technique](#contact-microsoft-support) pour obtenir de l’aide. |
| **TenantIsBlocked** | Locataire bloqué | [Contactez le support technique](#contact-microsoft-support) et indiquez l’*ID locataire* figurant dans la page de propriétés Azure AD du Portail Azure. |
| **UserNotFound** | L’utilisateur spécifié est introuvable | Le locataire n’apparaît plus comme étant actif dans Azure AD. Vérifiez que votre abonnement est actif et que vous disposez des applications internes requises. Assurez-vous également que le locataire spécifié dans l’objet du certificat est le locataire attendu et que le certificat est toujours valide et enregistré sous le principal de service. |

## <a name="messages-your-users-may-encounter-that-arent-errors"></a>Messages qui ne sont pas des erreurs que vos utilisateurs peuvent rencontrer

Il peut arriver que vos utilisateurs reçoivent des messages de Multi-Factor Authentication en cas d’échec de leur demande d’authentification. Ceux-ci ne sont pas des erreurs de configuration. Il s’agit d’avertissements intentionnels expliquant pourquoi une demande d’authentification a été refusée.

| Code d'erreur | Message d’erreur | Étapes recommandées |
| ---------- | ------------- | ----------------- |
| **OathCodeIncorrect** | Code saisi erroné\Code OATH incorrect | L’utilisateur a entré un code erroné. Demandez-lui de réessayer en demandant un nouveau code ou en se reconnectant. |
| **SMSAuthFailedMaxAllowedCodeRetryReached** | Limite maximale autorisée de tentatives de saisie de code atteinte | L’utilisateur n’a pas rempli les conditions de vérification correctement un trop grand nombre de fois. En fonction de vos paramètres, un administrateur devra peut-être le débloquer.  |
| **SMSAuthFailedWrongCodeEntered** | Code saisi erroné/Mot de passe SMS à usage unique incorrect | L’utilisateur a entré un code erroné. Demandez-lui de réessayer en demandant un nouveau code ou en se reconnectant. |

## <a name="errors-that-require-support"></a>Erreurs qui nécessitent un support

Si vous rencontrez l’une de ces erreurs, nous vous recommandons de [contacter le support technique](#contact-microsoft-support) pour un diagnostic. Il n’existe pas d’étapes standard à suivre pour résoudre ces erreurs. Lorsque vous contactez le support technique, veillez à inclure autant d’informations que possible sur les étapes ayant conduit à l’erreur, ainsi que les informations de votre locataire.

| Code d'erreur | Message d’erreur |
| ---------- | ------------- |
| **InvalidParameter** | Request ne doit pas être null |
| **InvalidParameter** | ObjectId ne doit pas être null ou vide pour ReplicationScope : {0} |
| **InvalidParameter** | La longueur de CompanyName \{0} \ dépasse la longueur maximale autorisée {1} |
| **InvalidParameter** | UserPrincipalName ne doit pas être null ou vide |
| **InvalidParameter** | Le TenantId fourni n’est pas dans le bon format |
| **InvalidParameter** | SessionId ne doit pas être null ou vide |
| **InvalidParameter** | Résolution impossible des ProofData de la demande ou de Msods. Paramètre ProofData inconnu |
| **InternalError** |  |
| **OathCodePinIncorrect** |  |
| **VersionNotSupported** |  |
| **MFAPinNotSetup** |  |

## <a name="next-steps"></a>Étapes suivantes

### <a name="troubleshoot-user-accounts"></a>Résoudre les problèmes relatifs aux comptes utilisateur

Si vos utilisateurs ont [des difficultés avec la vérification en deux étapes](../user-help/multi-factor-authentication-end-user-troubleshoot.md), aidez-les à diagnostiquer eux-mêmes les problèmes.

### <a name="health-check-script"></a>Script de vérification d’intégrité

Le [script de contrôle d'intégrité de l'extension NPS d'Azure AD MFA](/samples/azure-samples/azure-mfa-nps-extension-health-check/azure-mfa-nps-extension-health-check/) procède à un contrôle d'intégrité de base lors de la résolution des problèmes liés à l'extension NPS. Exécutez le script et choisissez l’option 3.

### <a name="contact-microsoft-support"></a>Contact Microsoft support

Si vous avez besoin d’une assistance supplémentaire, contactez un professionnel du support via le [Support du serveur Azure Multi-Factor Authentication](https://support.microsoft.com/oas/default.aspx?prid=14947). Lorsque vous nous contactez, veillez à inclure autant d’informations que possible concernant votre problème. Vous pouvez notamment préciser la page sur laquelle vous avez rencontré l’erreur, le code d’erreur spécifique, l’ID de session spécifique, l’ID de l’utilisateur qui a vu l’erreur et les journaux d’activité de débogage.

Pour collecter les journaux d’activité de débogage pour les diagnostics de support sur le serveur NPS (Network Policy Server), effectuez les étapes suivantes :

1. Ouvrez l’Éditeur du Registre, accédez à HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\AzureMfa, puis définissez **VERBOSE_LOG** sur **TRUE**.
2. Ouvrez une invite de commandes administrateur et exécutez les commandes suivantes :

   ```
   Mkdir c:\NPS
   Cd NPS
   netsh trace start Scenario=NetConnection capture=yes tracefile=c:\NPS\nettrace.etl
   logman create trace "NPSExtension" -ow -o c:\NPS\NPSExtension.etl -p {7237ED00-E119-430B-AB0F-C63360C8EE81} 0xffffffffffffffff 0xff -nb 16 16 -bs 1024 -mode Circular -f bincirc -max 4096 -ets
   logman update trace "NPSExtension" -p {EC2E6D3A-C958-4C76-8EA4-0262520886FF} 0xffffffffffffffff 0xff -ets
   ```

3. Reproduisez le problème

4. Arrêtez le suivi à l’aide des commandes suivantes :

   ```
   logman stop "NPSExtension" -ets
   netsh trace stop
   wevtutil epl AuthNOptCh C:\NPS\%computername%_AuthNOptCh.evtx
   wevtutil epl AuthZOptCh C:\NPS\%computername%_AuthZOptCh.evtx
   wevtutil epl AuthZAdminCh C:\NPS\%computername%_AuthZAdminCh.evtx
   Start .
   ```

5. Ouvrez l’Éditeur du Registre, accédez à HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\AzureMfa, puis définissez **VERBOSE_LOG** sur **FALSE**.
6. Compressez le contenu du dossier C:\NPS et joignez le fichier compressé au dossier de support.
---
title: Détails des événements de connexion relatifs à Azure AD Multi-Factor Authentication - Azure Active Directory
description: Apprenez à consulter l'activité de connexion relative aux événements Azure AD Multi-Factor Authentication et les messages d'état.
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 05/15/2020
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 5f78b70599d6d0ae8825accf4cc55cdc1c01d9ce
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96861236"
---
# <a name="use-the-sign-ins-report-to-review-azure-ad-multi-factor-authentication-events"></a>Utiliser le rapport des connexions pour examiner les événements Azure AD Multi-Factor Authentication

Pour examiner et comprendre les événements Azure AD Multi-Factor Authentication, vous pouvez utiliser le rapport des connexions d'Azure Active Directory (Azure AD). Ce rapport affiche les détails d’authentification des événements lorsqu’un utilisateur est invité à utiliser l’authentification multifacteur et dans le cas où des stratégies d’accès conditionnel seraient en cours d’utilisation. Pour plus d’informations sur le rapport de connexions, consultez la [présentation des rapports d’activité de connexion dans Azure AD](../reports-monitoring/concept-sign-ins.md).

Cet article explique comment afficher le rapport de connexions Azure AD dans le portail Azure, puis dans le module PowerShell MSOnline v1.

## <a name="view-the-azure-ad-sign-ins-report"></a>Afficher le rapport de connexion Azure AD

Le rapport de connexion vous fournit des informations sur l’utilisation des applications managées et des activités de connexion des utilisateurs, y compris des informations sur l’utilisation de l’authentification multifacteur (MFA). Les données d’authentification multifacteur vous donnent un aperçu du fonctionnement de l’authentification multifacteur dans votre organisation. Cela vous permet de répondre à des questions telles que :

- La connexion a-t-elle été protégée avec l’authentification multifacteur ?
- Comment l’utilisateur a-t-il effectué l’authentification multifacteur ?
- Pourquoi l’utilisateur était-il incapable d’effectuer l’authentification multifacteur ?
- Combien d’utilisateurs sont-ils invités à utiliser l’authentification multifacteur ?
- Combien d’utilisateurs n’ont pas pu résoudre le défi de l’authentification multifacteur ?
- Quels sont les problèmes courants d’authentication multifacteur auxquels les utilisateurs sont confrontés ?

Pour afficher le rapport d’activité de connexion dans le [portail Azure](https://portal.azure.com), procédez comme suit. Vous pouvez également interroger les données à l’aide de [l’API de création de rapports](../reports-monitoring/concept-reporting-api.md).

1. Connectez-vous au [portail Azure](https://portal.azure.com) à l’aide d’un compte disposant d’autorisations d’*administrateur général*.
1. Recherchez et sélectionnez **Azure Active Directory**, puis choisissez **Utilisateurs** dans le menu de gauche.
1. Sous l’onglet *Activité* dans le menu de gauche, sélectionnez **Connexions**.
1. Une liste des événements de connexion s’affiche, incluant leurs états. Vous pouvez sélectionner un événement pour afficher plus d’informations.

    Les onglets *Détails d’authentification* ou *Accès conditionnel* de l’événement montrent le code d’état ou la stratégie qui a déclenché l’invite MFA.

    [![Screenshot of example Azure Active Directory sign-ins report in the Azure portal](media/howto-mfa-reporting/sign-in-report-cropped.png)](media/howto-mfa-reporting/sign-in-report.png#lightbox)

Si elle est disponible, l’authentification apparaît. Par exemple : message texte, notification d’application Microsoft Authenticator ou appel téléphonique.

Les informations suivantes s’affichent sur la fenêtre *Détails d’authentification* pour un événement de connexion qui indique si la requête MFA a été satisfaite ou refusée :

* Si l’authentification multifacteur a été réussie, cette colonne fournit plus d’informations sur la façon dont l’authentification multifacteur a été satisfaite.
   * achevée dans le cloud
   * a expiré en raison des politiques configurées pour l’abonné
   * inscription demandée
   * satisfaite par une revendication dans le jeton
   * satisfaite par une revendication fournie par un fournisseur externe
   * satisfaite par l’authentification forte
   * ignorée, car le flux testé était un flux d’ouverture de session de répartiteur de service Windows
   * ignorée en raison de mot de passe
   * ignorée en raison d’emplacement
   * ignorée en raison de l’appareil enregistré
   * ignorée en raison de l’appareil mémorisé
   * réussie

* Si l’authentification multifacteur a été refusée, cette colonne fournit la raison de refus.
   * authentification en cours
   * double tentative d’authentification
   * code erroné entré beaucoup de fois
   * authentification invalide
   * code de vérification de l’application mobile incorrect
   * configuration incorrecte
   * appel téléphonique passé à la messagerie vocale
   * numéro de téléphone de format invalide
   * erreur de service
   * téléphone de l’utilisateur injoignable
   * Impossible d’envoyer la notification d’application mobile à l’appareil
   * Impossible d’envoyer la notification d’application mobile
   * refus de l’authentification par l’utilisateur
   * l’utilisateur n’a pas répondu à la notification d’application mobile
   * l’utilisateur ne dispose pas de toutes les méthodes de vérification enregistrés
   * l’utilisateur a entré un code erroné
   * l’utilisateur a entré un code PIN erroné
   * l’utilisateur a raccroché l’appel téléphonique sans réussir l’authentification
   * l’utilisateur est bloqué
   * l’utilisateur n’a jamais entré le code de vérification
   * utilisateur non trouvé
   * code de vérification déjà utilisé une fois

## <a name="powershell-reporting-on-users-registered-for-mfa"></a>Création de rapports PowerShell sur les utilisateurs inscrits pour MFA

Tout d’abord, assurez-vous d’avoir le [module MSOnline V1 PowerShell](/powershell/azure/active-directory/overview) installé.

Identifiez les utilisateurs qui se sont inscrits auprès de MFA à l’aide du code PowerShell qui suit. Cet ensemble de commandes exclut les utilisateurs désactivés, car ces types de comptes ne peuvent pas s’authentifier auprès d’Azure AD :

```powershell
Get-MsolUser -All | Where-Object {$_.StrongAuthenticationMethods -ne $null -and $_.BlockCredential -eq $False} | Select-Object -Property UserPrincipalName
```

Identifiez les utilisateurs qui ne se sont pas inscrits auprès de MFA à l’aide du code PowerShell qui suit. Cet ensemble de commandes exclut les utilisateurs désactivés, car ces types de comptes ne peuvent pas s’authentifier auprès d’Azure AD :

```powershell
Get-MsolUser -All | Where-Object {$_.StrongAuthenticationMethods.Count -eq 0 -and $_.BlockCredential -eq $False} | Select-Object -Property UserPrincipalName
```

Identifiez les méthodes de sortie et les utilisateurs inscrits :

```powershell
Get-MsolUser -All | Select-Object @{N='UserPrincipalName';E={$_.UserPrincipalName}},

@{N='MFA Status';E={if ($_.StrongAuthenticationRequirements.State){$_.StrongAuthenticationRequirements.State} else {"Disabled"}}},

@{N='MFA Methods';E={$_.StrongAuthenticationMethods.methodtype}} | Export-Csv -Path c:\MFA_Report.csv -NoTypeInformation
```

## <a name="downloaded-activity-reports-result-codes"></a>Codes résultats des rapports d’activité téléchargés

Le tableau suivant peut vous aider à détecter les problèmes liés aux événements en utilisant la version du rapport d’activité téléchargée à partir des étapes précédentes sur le portail ou des commandes PowerShell. Ces codes résultat n’apparaissent pas directement dans le portail Azure.

| Résultat de l’appel | Description | Description générale |
| --- | --- | --- |
| SUCCESS_WITH_PIN | Code PIN entré | L’utilisateur a entré un code PIN.   Si l’authentification a réussi, alors le code PIN entré est correct.   Si l’authentification est refusée, cela signifie que l’utilisateur a entré un code PIN ou que l’utilisateur est défini en mode Standard. |
| SUCCESS_NO_PIN | Uniquement # entré | Si l’utilisateur est défini en mode Code PIN et que l’authentification est refusée, cela signifie que l’utilisateur n’a pas entré son code PIN et qu’il a uniquement entré #.  Si l’utilisateur est défini en mode Standard et que l’authentification réussit, cela signifie que l’utilisateur a uniquement entré #, ce qu’il faut exactement faire en mode Standard. |
| SUCCESS_WITH_PIN_BUT_TIMEOUT | # non appuyé après l’entrée | L’utilisateur n’a pas envoyé de chiffres DTMF, car # n’a pas été entré.   Les autres chiffres entrés ne sont pas envoyés, sauf si # est entré, indiquant la validation de l’entrée. |
|SUCCESS_NO_PIN_BUT_TIMEOUT | Aucune entrée téléphone - expiration du délai d’attente | L’appel a été reçu, mais aucune réponse.   Cela indique généralement que l’appel a été récupéré par la messagerie vocale. |
| SUCCESS_PIN_EXPIRED | Code PIN expiré et non modifié | Le code PIN de l’utilisateur a expiré et il a été invité à le modifier, mais la modification du code PIN n’a pas totalement abouti. |
| SUCCESS_USED_CACHE | Cache utilisé | L’authentification a réussi sans appel Multi-Factor Authentication, car une authentification ayant abouti précédemment pour le même nom d’utilisateur s’est produite pendant la plage de temps du cache configurée. |
| SUCCESS_BYPASSED_AUTH | Authentification contournée | L’authentification a réussi à l’aide d’un contournement à usage unique initié pour l’utilisateur.  Pour plus d'informations sur le contournement, voir Rapport Historique de l'utilisateur contourné. |
| SUCCESS_USED_IP_BASED_CACHE | Cache basé sur l’adresse IP utilisée | L’authentification a réussi sans appel Multi-Factor Authentication, car une authentification ayant abouti précédemment pour le même nom d’utilisateur, type d’authentification, nom d’application et adresse IP s’est produite pendant la plage de temps du cache configurée. |
| SUCCESS_USED_APP_BASED_CACHE | Cache basé sur l’application utilisée | L’authentification a réussi sans appel Multi-Factor Authentication, car une authentification ayant abouti précédemment pour le même nom d’utilisateur, type d’authentification et nom d’application s’est produite pendant la plage de temps du cache configurée. |
| SUCCESS_INVALID_INPUT | Entrée téléphone non valide | La réponse envoyée à partir du téléphone n’est pas valide.   Cela peut provenir d’un télécopieur ou d’un modem, ou s’expliquer par le fait que l’utilisateur a entré * dans son code PIN. |
| SUCCESS_USER_BLOCKED | L’utilisateur est bloqué | Le numéro de téléphone de l’utilisateur est bloqué.   Un numéro bloqué peut être initié par l’utilisateur lors d’un appel d’authentification ou par un administrateur utilisant le portail Microsoft Azure. <br> REMARQUE :   Un numéro bloqué est également une conséquence d’une alerte de fraude. |
| SUCCESS_SMS_AUTHENTICATED | SMS authentifié | Pour un SMS bidirectionnel, l’utilisateur a répondu correctement avec son code secret à usage unique ou son code secret à usage unique + son code PIN. |
| SUCCESS_SMS_SENT | SMS envoyé | Le SMS contenant le code secret à usage unique a été correctement envoyé.   L’utilisateur entre le code secret à usage unique ou le code secret à usage unique + le code PIN dans l’application pour effectuer l’authentification. |
| SUCCESS_PHONE_APP_AUTHENTICATED | Application mobile authentifiée | L’utilisateur est authentifié correctement via l’application mobile. |
| SUCCESS_OATH_CODE_PENDING | Code OATH en attente | L’utilisateur a été invité à entrer son code OATH, mais n’a pas répondu. |
| SUCCESS_OATH_CODE_VERIFIED | Code OATH vérifié | L’utilisateur a entré un code OATH valide lorsqu’il y a été invité. |
| SUCCESS_FALLBACK_OATH_CODE_VERIFIED | Code OATH de secours vérifié | L’authentification de l’utilisateur a été refusée à l’aide de sa principale méthode Multi-Factor Authentication, puis un code OATH valide de secours a été fourni. |
| SUCCESS_FALLBACK_SECURITY_QUESTIONS_ANSWERED | Les questions de sécurité de secours ont obtenu des réponses | L’authentification de l’utilisateur a été refusée à l’aide de sa principale méthode Multi-Factor Authentication, puis ses questions de sécurité de secours ont obtenu une réponse. |
| FAILED_PHONE_BUSY | Authentification déjà en cours | Multi-Factor Authentication traite déjà une authentification pour cet utilisateur.   Cela est souvent dû à des clients RADIUS qui envoient plusieurs demandes d’authentification lorsqu’ils sont connectés à la même session. |
| CONFIG_ISSUE | Téléphone inaccessible | Une tentative d’appel a eu lieu, mais il n’a pas pu être passé ou n’a pas reçu de réponse.   Cela inclut le signal occupé, le signal occupé rapide (déconnecté), la triple tonalité (le numéro n’est plus en service), l’expiration du délai de sonnerie, etc. |
| FAILED_INVALID_PHONENUMBER | Format de numéro de téléphone non valide | Le numéro de téléphone est dans un format non valide.   Les numéros de téléphone doivent être numériques et comporter 10 chiffres pour le code de pays + 1 (États-Unis et Canada). |
| FAILED_USER_HUNGUP_ON_US | L’utilisateur a raccroché le téléphone | L’utilisateur a répondu au téléphone, mais a raccroché sans appuyer sur un bouton. |
| FAILED_INVALID_EXTENSION | Extension non valide | L’extension contient des caractères non valides.   Seuls les chiffres, les virgules, * et # sont autorisés.   Un préfixe @ peut également être utilisé. |
| FAILED_FRAUD_CODE_ENTERED | Code fraude entré | L’utilisateur a choisi de signaler une fraude lors de l’appel, ce qui entraîne un refus d’authentification et le blocage du numéro de téléphone.| 
| FAILED_SERVER_ERROR | Impossible de passer un appel | Le service Multi-Factor Authentication n’a pas pu effectuer l’appel. |
| FAILED_SMS_NOT_SENT | Le SMS n’a pas pu être envoyé | Le SMS n’a pas pu être envoyé.   L’authentification est refusée. |
| FAILED_SMS_OTP_INCORRECT | Le code secret à usage unique du SMS est incorrect | L’utilisateur a entré un code secret à usage unique incorrect à partir du message texte qu’il a reçu.   L’authentification est refusée. |
| FAILED_SMS_OTP_PIN_INCORRECT | Le code secret à usage unique du SMS + le code PIN sont incorrects | L’utilisateur a entré un code secret à usage unique incorrect et/ou un code PIN utilisateur incorrect.   L’authentification est refusée. |
| FAILED_SMS_MAX_OTP_RETRY_REACHED | A dépassé les tentatives de saisie maximum de code secret à usage unique de SMS | L’utilisateur a dépassé le nombre maximal de tentatives de saisie de code secret à usage unique. |
| FAILED_PHONE_APP_DENIED | Application mobile refusée | L’utilisateur a refusé l’authentification dans l’application mobile en appuyant sur le bouton Refuser. |
| FAILED_PHONE_APP_INVALID_PIN | Code PIN d’application mobile non valide | L’utilisateur a entré un code PIN non valide lors de l’authentification dans l’application mobile. |
| FAILED_PHONE_APP_PIN_NOT_CHANGED | Code PIN d’application mobile non modifié | L’utilisateur n’a pas effectué correctement la modification du code PIN nécessaire dans l’application mobile. |
| FAILED_FRAUD_REPORTED | Fraude signalée | L’utilisateur a signalé une fraude dans l’application mobile. |
| FAILED_PHONE_APP_NO_RESPONSE | Aucune réponse de l’application mobile | L’utilisateur n’a pas répondu à la demande d’authentification de l’application mobile. |
| FAILED_PHONE_APP_ALL_DEVICES_BLOCKED | Tous les appareils de l’application mobile sont bloqués | Les appareils de l’application mobile de cet utilisateur ne répondent plus aux notifications et ont été bloqués. |
| FAILED_PHONE_APP_NOTIFICATION_FAILED | Échec des notifications sur l’application mobile | Une erreur s’est produite lors de la tentative d’envoi d’une notification vers l’application mobile sur l’appareil de l’utilisateur. |
| FAILED_PHONE_APP_INVALID_RESULT | Résultat non valide de l’application mobile | L’application mobile a renvoyé un résultat non valide. |
| FAILED_OATH_CODE_INCORRECT | Code OATH incorrect | L’utilisateur a entré un code OATH incorrect.  L’authentification est refusée. |
| FAILED_OATH_CODE_PIN_INCORRECT | Code OATH + code PIN incorrects | L’utilisateur a entré un code OATH incorrect et/ou un code PIN utilisateur incorrect.  L’authentification est refusée. |
| FAILED_OATH_CODE_DUPLICATE | Code OATH dupliqué | L’utilisateur a entré un code OATH précédemment utilisé.  L’authentification est refusée. |
| FAILED_OATH_CODE_OLD | Code OATH ayant expiré | L’utilisateur a entré un code OATH qui précède un code OATH précédemment utilisé.  L’authentification est refusée. |
| FAILED_OATH_TOKEN_TIMEOUT | Délai d’attente de résultat de code OATH | L’utilisateur a mis trop de temps à entrer le code OATH et la tentative de Multi-Factor Authentication avait déjà expiré. |
| FAILED_SECURITY_QUESTIONS_TIMEOUT | Délai d’expiration de résultat des questions de sécurité | L’utilisateur a mis trop de temps à entrer les réponses aux questions de sécurité et la tentative de Multi-Factor Authentication avait déjà expiré. |
| FAILED_AUTH_RESULT_TIMEOUT | Délai d’expiration du résultat de l’authentification | L’utilisateur a mis trop de temps pour traiter la tentative de Multi-Factor Authentication. |
| FAILED_AUTHENTICATION_THROTTLED | Authentification limitée | La tentative de Multi-Factor Authentication a été limitée par le service. |

## <a name="additional-mfa-reports"></a>Rapports MFA supplémentaires

Les informations et les rapports supplémentaires suivants sont disponibles pour les événements MFA, y compris ceux du serveur MFA :

| Rapport | Emplacement | Description |
|:--- |:--- |:--- |
| Historique de l'utilisateur bloqué | Azure AD > Sécurité > MFA > Bloquer/débloquer des utilisateurs | Affiche l’historique des demandes de blocage et de déblocage d’utilisateurs. |
| Utilisation des composants locaux | Azure AD > Sécurité > MFA > Rapport d’activité | Fournit des informations sur l’utilisation globale du serveur MFA via l’extension NPS, ADFS et le serveur MFA. |
| Historique de l'utilisateur contourné | Azure AD > Security > MFA > Contournement à usage unique | Fournit un historique des requêtes du serveur MFA pour contourner l’authentification multifacteur pour un utilisateur. |
| État du serveur | Azure AD > Sécurité > MFA > État du serveur | Affiche l’état des serveurs MFA associés à votre compte. |

## <a name="next-steps"></a>Étapes suivantes

Cet article vous a présenté le rapport d’activité de connexions. Pour plus d’informations sur ce que ce rapport contient et afin de mieux comprendre les données, consultez [rapports d’activité de connexion dans Azure AD](../reports-monitoring/concept-sign-ins.md).

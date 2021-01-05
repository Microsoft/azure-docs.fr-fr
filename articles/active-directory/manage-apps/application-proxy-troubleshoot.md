---
title: Résoudre les problèmes du proxy d’application | Microsoft Docs
description: Explique comment résoudre les erreurs dans le proxy d’application Azure AD.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: troubleshooting
ms.date: 06/24/2019
ms.author: kenwith
ms.reviewer: japere
ms.openlocfilehash: 413cfe4f3aed446ad26a210b4faa452c4f624685
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88640852"
---
# <a name="troubleshoot-application-proxy-problems-and-error-messages"></a>Résoudre les problèmes de proxy d’application et les messages d’erreur

Lors de la résolution des problèmes du proxy d’application, nous vous recommandons de commencer par examiner le flux de résolution des problèmes, [Déboguer les problèmes d’un connecteur Proxy d’application](application-proxy-debug-connectors.md), afin de déterminer si les connecteurs Proxy d’application sont configurés correctement. Si vous rencontrez toujours des problèmes de connexion à l’application, suivez la procédure de résolution des problèmes dans [Déboguer les problèmes d’application d’un proxy d’application](application-proxy-debug-apps.md).

Si des erreurs se produisent dans l’accès à une application publiée ou dans la publication d’applications, vérifiez les options suivantes pour voir si le proxy d’application Microsoft Azure Active Directory fonctionne correctement :

* Ouvrez la console Services Windows. Vérifiez que le service **Microsoft AAD Application Proxy Connector** est activé et en cours d’exécution. Vous pouvez également consulter la page de propriétés du service Proxy d’application, comme le montre l’image suivante :  
  ![Capture d’écran de la fenêtre Propriétés du connecteur de proxy d’application Microsoft AAD](./media/application-proxy-troubleshoot/connectorproperties.png)
* Ouvrez l’Observateur d’événements et recherchez les événements du proxy d’application sous **Journaux des applications et des services** > **Microsoft** > **AadApplicationProxy** > **Connecteur** > **Admin**.
* Si nécessaire, des journaux d’activité plus détaillés sont disponibles [en activant les journaux d’activité de session du connecteur de proxy d’application](application-proxy-connectors.md#under-the-hood).

## <a name="the-page-is-not-rendered-correctly"></a>La page ne s’affiche pas correctement
Vous pouvez rencontrer des problèmes d’affichage ou de fonctionnement de votre application sans recevoir de message d’erreur spécifique. Cela peut se produire si vous avez publié le chemin d’accès de l’article, mais que l’application requiert un contenu qui se trouve en dehors de ce chemin d’accès.

Par exemple, si vous publiez le chemin d’accès `https://yourapp/app` mais que l’application appelle les images dans `https://yourapp/media`, celles-ci ne seront pas restituées. Assurez-vous que vous publiez l’application en utilisant le niveau du chemin d’accès le plus élevé pour inclure tous les contenus pertinents. Dans cet exemple, il s’agirait de `http://yourapp/`.

## <a name="connector-errors"></a>Erreurs de connecteur

Si l’inscription échoue au cours de l’installation de l’assistant Connecteur, il existe deux façons d’afficher la raison de l’échec. Vous pouvez effectuer une recherche dans le journal des événements sous **Journaux des applications et des services\Microsoft\AadApplicationProxy\Connecteur\Admin**, ou exécuter la commande Windows PowerShell suivante :

```powershell
Get-EventLog application –source "Microsoft AAD Application Proxy Connector" –EntryType "Error" –Newest 1
```

Une fois l’erreur de connecteur identifiée dans le journal des événements, reportez-vous à cette table d’erreurs courantes pour résoudre le problème :

| Error | Étapes recommandées |
| ----- | ----------------- |
| Échec de l’inscription du connecteur : assurez-vous que vous avez activé le proxy d’application dans le portail de gestion Azure, et que vous avez entré correctement votre nom d’utilisateur et votre mot de passe Active Directory. Erreur : « Une ou plusieurs erreurs se sont produites. » | Si vous avez fermé la fenêtre d’inscription sans vous connecter à Azure AD, réexécutez l’Assistant Connecteur et inscrivez le connecteur. <br><br> Si la fenêtre d’inscription s’ouvre puis se ferme immédiatement sans vous permettre de vous connecter, vous recevrez probablement cette erreur. Cette erreur se produit quand il existe une erreur réseau sur votre système. Vérifiez que vous pouvez vous connecter à un site web public à partir d’un navigateur et que les ports sont ouverts, comme il est spécifié dans les [prérequis du proxy d’application](application-proxy-add-on-premises-application.md#prepare-your-on-premises-environment). |
| L’erreur est présentée en clair dans la fenêtre d’inscription. Impossible de poursuivre | Si cette erreur se produit et que la fenêtre se ferme, cela signifie que vous avez saisi un nom d’utilisateur ou un mot de passe incorrect. Réessayez. |
| Échec de l’inscription du connecteur : assurez-vous que vous avez activé le proxy d’application dans le portail de gestion Azure, et que vous avez entré correctement votre nom d’utilisateur et votre mot de passe Active Directory. Erreur : « AADSTS50059 : pas d’informations d’identification de locataire trouvées dans la demande ou déduites des informations d’identification fournies. Échec de la recherche selon l’URI du principal du service. | Vous tentez de vous connecter avec un compte Microsoft et non un domaine qui fait partie de l’ID d’organisation de l’annuaire auquel vous tentez d’accéder. Assurez-vous que l’administrateur fait partie du même nom de domaine que le domaine du locataire. Par exemple, si le domaine Azure AD est contoso.com, l’administrateur doit être admin@contoso.com. |
| Impossible de récupérer la stratégie d’exécution actuelle pour l’exécution de scripts PowerShell. | En cas d’échec de l’installation du connecteur, vérifiez que la stratégie d’exécution de PowerShell n’est pas désactivée. <br><br>1. Ouvrez l’Éditeur de stratégie de groupe.<br>2. Accédez à **Configuration ordinateur** > **Modèles d’administration** > **Composants Windows** > **Windows PowerShell** et double-cliquez sur **Activer l’exécution des scripts**.<br>3. Cette stratégie d’exécution peut être définie sur **Non configuré** ou **Activé**. Si elle est définie sur **Activé**, vérifiez que sous Options, la stratégie d’exécution est définie sur **Autoriser les scripts locaux et les scripts signés distants** ou sur **Autoriser tous les scripts**. |
| Échec du téléchargement de la configuration par le connecteur. | Le certificat client du connecteur, qui est utilisé pour l’authentification, est expiré. Ceci peut également se produire si le connecteur est installé derrière un proxy. Dans ce cas, le connecteur ne peut pas accéder à Internet et ne sera pas en mesure de fournir des applications aux utilisateurs distants. Renouvelez l’approbation manuellement à l’aide de l’applet de commande `Register-AppProxyConnector` dans Windows PowerShell. Si votre connecteur est derrière un proxy, il est nécessaire d’octroyer l’accès à Internet aux comptes du connecteur « services réseau » et « système local ». Ceci peut être effectué en leur accordant l’accès au proxy ou en les configurant pour qu’ils ignorent le proxy. |
| Échec de l’inscription du connecteur : vous devez être un administrateur d’application de votre instance Active Directory pour inscrire le connecteur. Erreur : « La demande d’inscription a été refusée. » | L’alias avec lequel vous essayez de vous connecter n’est pas un administrateur sur ce domaine. Votre connecteur est toujours installé pour l’annuaire qui détient le domaine de l’utilisateur. Vérifiez que le compte Administrateur avec lequel vous essayez de vous connecter dispose au moins des autorisations d’administrateur d’application sur le locataire Azure AD. |
| Le connecteur n’a pas pu se connecter au service en raison de problèmes réseau. Le connecteur a essayé d’accéder à l’URL suivante. | Le connecteur ne peut pas se connecter au service cloud de proxy d’application. Cela peut se produire si vous avez une règle de pare-feu qui bloque la connexion. Veillez à autoriser l’accès aux ports appropriés et aux URL listées dans [Conditions préalables pour le proxy d’application](application-proxy-add-on-premises-application.md#prepare-your-on-premises-environment). |

## <a name="kerberos-errors"></a>Erreurs Kerberos

Cette table répertorie les erreurs les plus courantes qui proviennent de la configuration et de l’installation de Kerberos et inclut des suggestions de résolution.

| Error | Étapes recommandées |
| ----- | ----------------- |
| Impossible de récupérer la stratégie d’exécution actuelle pour l’exécution de scripts PowerShell. | En cas d’échec de l’installation du connecteur, vérifiez que la stratégie d’exécution de PowerShell n’est pas désactivée.<br><br>1. Ouvrez l’Éditeur de stratégie de groupe.<br>2. Accédez à **Configuration ordinateur** > **Modèles d’administration** > **Composants Windows** > **Windows PowerShell** et double-cliquez sur **Activer l’exécution des scripts**.<br>3. Cette stratégie d’exécution peut être définie sur **Non configuré** ou **Activé**. Si elle est définie sur **Activé**, vérifiez que sous Options, la stratégie d’exécution est définie sur **Autoriser les scripts locaux et les scripts signés distants** ou sur **Autoriser tous les scripts**. |
| 12008 - Azure AD a dépassé le nombre maximal de tentatives d’authentification Kerberos autorisé auprès du serveur principal. | Cette erreur peut indiquer une configuration incorrecte entre Azure AD et le serveur d’applications principal, ou un problème de configuration de la date/heure sur les deux ordinateurs. Le serveur principal a refusé le ticket Kerberos créé par Azure AD. Vérifiez que les configurations d’Azure AD et du serveur d’applications principal sont correctes. Assurez-vous que les configurations de la date/heure sur Azure AD et sur le serveur d’applications principal sont synchronisées. |
| 13016 - Azure AD ne peut pas récupérer un ticket Kerberos pour l’utilisateur, car il n’existe pas d’UPN dans le jeton de périphérie ou dans le cookie d’accès. | Il existe un problème avec la configuration de STS (SharePoint Team Services). Corrigez la configuration de la revendication UPN dans STS. |
| 13019 - Azure AD ne peut pas récupérer un ticket Kerberos pour l’utilisateur en raison de l’erreur API générale suivante. | Cet événement peut indiquer une configuration incorrecte entre Azure AD et le serveur contrôleur de domaine, ou un problème de configuration de la date/heure sur les deux ordinateurs. Le contrôleur de domaine a refusé le ticket Kerberos créé par Azure AD. Vérifiez que les configurations d’Azure AD et du serveur d’applications principal sont correctes, en particulier les configurations du nom de principal du service (SPN). Assurez-vous qu’Azure AD est joint au même domaine que le contrôleur de domaine pour que le contrôleur de domaine établisse la relation d’approbation avec Azure AD. Assurez-vous que les configurations de la date/heure sur Azure AD et sur le contrôleur de domaine sont synchronisées. |
| 13020 - Azure AD ne peut pas récupérer un ticket Kerberos pour l’utilisateur, car le SPN du serveur principal n’est pas défini. | Cet événement peut indiquer une configuration incorrecte entre Azure AD et le serveur contrôleur de domaine, ou un problème de configuration de la date/heure sur les deux ordinateurs. Le contrôleur de domaine a refusé le ticket Kerberos créé par Azure AD. Vérifiez que les configurations d’Azure AD et du serveur d’applications principal sont correctes, en particulier les configurations du nom de principal du service (SPN). Assurez-vous qu’Azure AD est joint au même domaine que le contrôleur de domaine pour que le contrôleur de domaine établisse la relation d’approbation avec Azure AD. Assurez-vous que les configurations de la date/heure sur Azure AD et sur le contrôleur de domaine sont synchronisées. |
| 13022 - Azure AD ne peut pas authentifier l’utilisateur, car le serveur principal répond aux tentatives d’authentification Kerberos avec une erreur HTTP 401. | Cet événement peut indiquer une configuration incorrecte entre Azure AD et le serveur d’applications principal, ou un problème de configuration de la date/heure sur les deux ordinateurs. Le serveur principal a refusé le ticket Kerberos créé par Azure AD. Vérifiez que les configurations d’Azure AD et du serveur d’applications principal sont correctes. Assurez-vous que les configurations de la date/heure sur Azure AD et sur le serveur d’applications principal sont synchronisées. Pour plus d’informations, consultez l’article [Résolution des problèmes de configuration de délégation Kerberos contrainte pour le proxy d’application](application-proxy-back-end-kerberos-constrained-delegation-how-to.md).  |

## <a name="end-user-errors"></a>Erreurs de l’utilisateur final

Cette liste comprend des erreurs que vos utilisateurs finaux peuvent rencontrer en cas d’échec de leur tentative d’accès à l’application. 

| Error | Étapes recommandées |
| ----- | ----------------- |
| Le site web ne peut pas afficher la page. | Votre utilisateur peut recevoir cette erreur en tentant d’accéder à l’application que vous avez publiée si l’application est une application avec l’authentification Windows intégrée. Le SPN défini pour cette application est peut-être incorrect. Pour les applications avec l’authentification Windows intégrée : assurez-vous que le SPN configuré pour cette application est correct. |
| Le site web ne peut pas afficher la page. | Votre utilisateur peut recevoir cette erreur en tentant d’accéder à l’application que vous avez publiée si l’application est une application OWA. La raison peut en être l’une des suivantes :<br><li>Le SPN défini pour cette application est incorrect. Assurez-vous que le SPN configuré pour cette application est correct.</li><li>L’utilisateur qui a tenté d’accéder à l’application utilise un compte Microsoft au lieu du compte d’entreprise approprié pour se connecter, ou bien l’utilisateur est un utilisateur invité. Assurez-vous que l’utilisateur se connecte en utilisant son compte d’entreprise qui correspond au domaine de l’application publiée. Les utilisateurs d’un compte Microsoft et les invités ne peuvent pas accéder aux applications avec l’authentification Windows intégrée.</li><li>L’utilisateur qui a tenté d’accéder à l’application n’est pas défini correctement pour cette application du côté local. Assurez-vous que cet utilisateur a les autorisations appropriées, telles qu’elles sont définies pour cette application back-end sur l’ordinateur local. |
| Cette application d’entreprise n’est pas accessible. Vous n’êtes pas autorisé à accéder à cette application. Échec de l’autorisation. Veillez à affecter à l’utilisateur un accès à cette application. | Votre utilisateur peut recevoir cette erreur en tentant d’accéder à l’application que vous avez publiée s’il utilise des comptes Microsoft plutôt que son compte d’entreprise pour se connecter. Les utilisateurs invités peuvent également obtenir cette erreur. Les utilisateurs d’un compte Microsoft et les invités ne peuvent pas accéder aux applications avec l’authentification Windows intégrée. Assurez-vous que l’utilisateur se connecte en utilisant son compte d’entreprise qui correspond au domaine de l’application publiée.<br><br>Vous n’avez peut-être pas affecté cette application à l’utilisateur. Accédez à l’onglet **Application** et, sous **Utilisateurs et groupes**, affectez cet utilisateur ou ce groupe d’utilisateurs à cette application. |
| Cette application d’entreprise n’est pas accessible pour l’instant. Réessayez ultérieurement... Le connecteur a dépassé le délai d’expiration. | Votre utilisateur peut recevoir cette erreur en tentant d’accéder à l’application que vous avez publiée s’il n’est pas correctement défini pour cette application du côté local. Assurez-vous que votre utilisateur a les autorisations appropriées, telles qu’elles sont définies pour cette application back-end sur l’ordinateur local. |
| Cette application d’entreprise n’est pas accessible. Vous n’êtes pas autorisé à accéder à cette application. Échec de l’autorisation. Vérifiez que l’utilisateur possède une licence Azure Active Directory Premium. | Il se peut que votre utilisateur obtienne cette erreur en tentant d’accéder à l’application que vous avez publiée si l’administrateur de l’abonné ne lui a pas assigné explicitement une licence Premium. Accédez à l’onglet **Licences** Active Directory de l’abonné et vérifiez qu’une licence Premium est affectée à cet utilisateur ou à ce groupe. |
| Impossible de trouver un serveur avec le nom d’hôte spécifié. | Votre utilisateur peut recevoir cette erreur en tentant d’accéder à l’application que vous avez publiée si le domaine personnalisé de l’application n’est pas configuré correctement. Assurez-vous d’avoir chargé un certificat pour le domaine et configuré correctement l’enregistrement DNS en suivant les étapes décrites dans [Utilisation des domaines personnalisés dans le proxy d'application Azure AD](application-proxy-configure-custom-domain.md) |
|Interdit : Cette application d’entreprise n’est pas accessible OU l’utilisateur n’a pas pu être autorisé. Assurez-vous que l’utilisateur est défini dans votre annuaire AD local et que l’utilisateur a accès à l’application dans votre AD local. | Il pourrait s’agir d’un problème d’accès aux informations d’autorisation. Consultez [Certaines applications et API nécessitent l’accès à des informations d’autorisation sur des objets Compte]( https://support.microsoft.com/help/331951/some-applications-and-apis-require-access-to-authorization-information). Pour résumer, ajoutez le compte d’ordinateur connecteur de proxy d’application au groupe de domaines builtin « Groupe d’accès d’autorisation Windows » à résoudre. |

## <a name="my-error-wasnt-listed-here"></a>Mon erreur n’était pas répertoriée ici

Si vous rencontrez une erreur ou un problème avec le proxy d’application Azure AD qui ne figure pas dans ce guide de dépannage, veuillez nous contacter. Envoyez un message électronique à notre [équipe de commentaires](mailto:aadapfeedback@microsoft.com) comportant les détails de l’erreur qui s’est produite.

## <a name="see-also"></a>Voir aussi
* [Activation du proxy d’application Azure AD](application-proxy-add-on-premises-application.md)
* [Publiez des applications avec le proxy d’application](application-proxy-add-on-premises-application.md)
* [Activer l’authentification unique](application-proxy-configure-single-sign-on-with-kcd.md)
* [Activer l’accès conditionnel](application-proxy-integrate-with-sharepoint-server.md)


<!--Image references-->
[1]: ./media/application-proxy-troubleshoot/connectorproperties.png
[2]: ./media/active-directory-application-proxy-troubleshoot/sessionlog.png

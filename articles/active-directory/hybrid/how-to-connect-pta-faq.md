---
title: 'Azure AD Connect : Authentification directe – Forum aux questions | Microsoft Docs'
description: Réponses au forum aux questions sur l’authentification directe d’Azure Active Directory
services: active-directory
keywords: Authentification directe Azure AD Connect, installation d’Active Directory, composants requis pour Azure AD, SSO, Authentification unique
documentationcenter: ''
author: billmath
manager: daveba
ms.assetid: 9f994aca-6088-40f5-b2cc-c753a4f41da7
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 06/09/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 75f797c40a276323cea9983c5340d2d854160c83
ms.sourcegitcommit: 1bdcaca5978c3a4929cccbc8dc42fc0c93ca7b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/13/2020
ms.locfileid: "97368475"
---
# <a name="azure-active-directory-pass-through-authentication-frequently-asked-questions"></a>Authentification directe Azure Active Directory : Forum aux questions

Cet article présente les réponses aux questions fréquemment posées sur l’authentification directe d’Azure Active Directory (Azure AD). N'hésitez pas à le consulter pour vous tenir au courant des mises à jour.

## <a name="which-of-the-methods-to-sign-in-to-azure-ad-pass-through-authentication-password-hash-synchronization-and-active-directory-federation-services-ad-fs-should-i-choose"></a>Parmi les nouvelles méthodes de connexion Azure AD - authentification directe, synchronisation de hachage de mot de passe et services de fédération Active Directory (AD FS) - laquelle dois-je choisir ?

Consultez [ce guide](./choose-ad-authn.md) pour voir une comparaison entre les différentes méthodes de connexion Azure AD, et savoir comment choisir la méthode de connexion appropriée pour votre organisation.

## <a name="is-pass-through-authentication-a-free-feature"></a>L’authentification directe est-elle une fonctionnalité gratuite ?

L’authentification directe est une fonctionnalité gratuite. Il est inutile de disposer des éditions payantes d’Azure AD pour l’utiliser.

## <a name="does-conditional-access-work-with-pass-through-authentication"></a>[L’accès conditionnel](../conditional-access/overview.md) fonctionne-t-il avec l’authentification directe ?

Oui. Toutes les fonctionnalités, y compris Azure AD Multi-Factor Authentication, peuvent être utilisées avec l'authentification directe.

## <a name="does-pass-through-authentication-support-alternate-id-as-the-username-instead-of-userprincipalname"></a>L’authentification directe prend-elle en charge « l’ID alternatif » comme le nom d’utilisateur, plutôt que « userPrincipalName » ?
Oui, la connexion avec une valeur autre qu’un UPN, telle qu’une autre adresse e-mail, est prise en charge pour l’authentification directe (PTA) et la synchronisation du hachage de mot de passe (PHS). Pour plus d’informations, consultez [ID de connexion de substitution](../authentication/howto-authentication-use-email-signin.md).

## <a name="does-password-hash-synchronization-act-as-a-fallback-to-pass-through-authentication"></a>La synchronisation du hachage de mot de passe agit-elle comme solution de secours pour l’authentification directe ?

Non. L’authentification directe _ne bascule pas_ automatiquement vers la synchronisation de hachage de mot de passe. Pour éviter les échecs de connexion de l’utilisateur, vous devez configurer l’authentification directe pour une [haute disponibilité](how-to-connect-pta-quick-start.md#step-4-ensure-high-availability).

## <a name="what-happens-when-i-switch-from-password-hash-synchronization-to-pass-through-authentication"></a>Que se passe-t-il lorsque je bascule de la synchronisation de hachage du mot de passe vers l’authentification directe ?

Lorsque vous utilisez Azure AD Connect pour changer de méthode de connexion, en passant de la synchronisation de hachage de mot de passe vers l’authentification directe, cette dernière devient la méthode de connexion principale pour vos utilisateurs dans les domaines managés. Remarque : l’ensemble des hachages de mot de passe des utilisateurs qui ont été préalablement synchronisés via la synchronisation de hachage de mot de passe restent stockés sur Azure AD.

## <a name="can-i-install-an-azure-ad-application-proxy-connector-on-the-same-server-as-a-pass-through-authentication-agent"></a>Puis-je installer un connecteur de [proxy d’application Azure AD](../manage-apps/application-proxy.md) sur le même serveur qu’un agent d’authentification directe ?

Oui. Les versions renommées de l’agent d’authentification directe (versions 1.5.193.0 ou versions ultérieures) prennent en charge cette configuration.

## <a name="what-versions-of-azure-ad-connect-and-pass-through-authentication-agent-do-you-need"></a>De quelles versions d’Azure AD Connect et de quel agent d’authentification directe avez-vous besoin ?

Pour que cette fonctionnalité puisse fonctionner, vous devez utiliser la version 1.1.750.0 ou une version ultérieure pour Azure AD Connect et 1.5.193.0 ou version ultérieure pour l’agent d’authentification directe. Installez tous les logiciels sur des serveurs avec Windows Server 2012 R2 ou version ultérieure.

## <a name="what-happens-if-my-users-password-has-expired-and-they-try-to-sign-in-by-using-pass-through-authentication"></a>Que se passe-t-il si mon mot de passe utilisateur a expiré et qu’ils essaient de se connecter à l’aide de l’authentification directe ?

Dans le cas où vous avez configuré [la réécriture du mot de passe](../authentication/concept-sspr-writeback.md) pour un utilisateur spécifique et que cet utilisateur se connecte à l’aide de l’authentification directe, leurs mots de passe peuvent être modifiés ou réinitialisés. Les mots de passe seront réécrits dans l’annuaire Active Directory local comme prévu.

Si la réécriture du mot de passe n’est pas configurée pour un utilisateur spécifique ou si l’utilisateur n’a aucune licence Azure AD valide attribuée, il ne peut pas mettre à jour son mot de passe dans le cloud. Il ne peut pas mettre à jour son mot de passe même si le mot de passe a expiré. Au lieu de cela, l’utilisateur voit le message suivant : « Votre organisation ne vous autorise pas à mettre à jour votre mot de passe sur ce site. Mettez-le à jour en fonction de la méthode recommandée par votre organisation, ou contactez votre administrateur si vous avez besoin d’aide ». L’utilisateur ou l’administrateur doit réinitialiser son mot de passe dans Active Directory sur site.

## <a name="how-does-pass-through-authentication-protect-you-against-brute-force-password-attacks"></a>Comment l’authentification directe vous protège-t-elle contre les attaques par recherche exhaustive de mot de passe ?

[Lisez les informations sur le verrouillage intelligent](../authentication/howto-password-smart-lockout.md).

## <a name="what-do-pass-through-authentication-agents-communicate-over-ports-80-and-443"></a>Qu’est-ce que les agents de l’authentification directe communiquent via les ports 80 et 443 ?

- Les agents d’authentification établissent les requêtes HTTPS sur le port 443 pour toutes les opérations de fonctionnalité.
- Les agents d'authentification établissent des requêtes HTTP sur le port 80 pour télécharger les listes de révocations de certificats TLS/SSL (CRL).

     >[!NOTE]
     >Dans les mises à jour récentes, le nombre de ports requis par la fonctionnalité a été diminué. Si vous disposez de versions antérieures d’Azure AD Connect ou de l’agent d’authentification, laissez ces ports également ouverts : 5671, 8080, 9090, 9091, 9350, 9352 et 10100-10120.

## <a name="can-the-pass-through-authentication-agents-communicate-over-an-outbound-web-proxy-server"></a>Les agents d’authentification directe peuvent-ils communiquer sur un serveur proxy web sortant ?

Oui. Si Web Proxy Auto-Discovery (WPAD) est activé dans votre environnement sur site, les agents d’authentification essayent automatiquement de localiser et d’utiliser un serveur proxy web sur le réseau.

Si vous ne disposez pas de WPAD dans votre environnement, vous pouvez ajouter des informations de proxy (comme indiqué ci-dessous) pour permettre à un agent d'authentification directe de communiquer avec Azure AD :
- Configurez les informations de proxy dans Internet Explorer avant d'installer l'agent d'authentification directe sur le serveur. Cela vous permettra de terminer l'installation de l'agent d’authentification, mais celui-ci apparaîtra toujours comme **Inactif** sur le portail d'administration.
- Sur le serveur, accédez à « C:\Program Files\Microsoft Azure AD Connect Authentication Agent ».
- Modifiez le fichier de configuration « AzureADConnectAuthenticationAgentService » et ajoutez les lignes suivantes (remplacez « http\://contosoproxy.com:8080 » par votre adresse proxy réelle) :

```
   <system.net>
      <defaultProxy enabled="true" useDefaultCredentials="true">
         <proxy
            usesystemdefault="true"
            proxyaddress="http://contosoproxy.com:8080"
            bypassonlocal="true"
         />
     </defaultProxy>
   </system.net>
```

## <a name="can-i-install-two-or-more-pass-through-authentication-agents-on-the-same-server"></a>Puis-je installer deux ou plusieurs agents d’authentification directe sur le même serveur ?

Non, vous ne pouvez installer qu’un seul agent d’authentification directe sur un serveur unique. Si vous souhaitez configurer l’authentification directe pour la haute disponibilité, [suivez les instructions fournies ici](how-to-connect-pta-quick-start.md#step-4-ensure-high-availability).

## <a name="do-i-have-to-manually-renew-certificates-used-by-pass-through-authentication-agents"></a>Dois-je renouveler manuellement les certificats utilisés par les agents d’authentification directe ?

La communication entre les agents d’authentification directe et Azure AD est sécurisée à l’aide de l’authentification basée sur les certificats. Ces [certificats sont renouvelés automatiquement tous les quelques mois par Azure AD](how-to-connect-pta-security-deep-dive.md#operational-security-of-the-authentication-agents). Il n’est pas nécessaire de renouveler manuellement ces certificats. Vous pouvez nettoyer les anciens certificats arrivés à expiration en fonction des besoins.

## <a name="how-do-i-remove-a-pass-through-authentication-agent"></a>Comment supprimer un agent d’authentification directe ?

Tant qu’un agent d’authentification directe est en cours d’exécution, il reste actif et gère en permanence les demandes de connexion utilisateur. Si vous souhaitez désinstaller un agent d’authentification, accédez à **Panneau de configuration -> Programmes -> Programmes et fonctionnalités** et désinstallez les programmes **Agent d’authentification Microsoft Azure AD Connect** et **Outil de mise à jour de l’agent Microsoft Azure AD Connect**.

Si vous consultez le panneau Authentification directe dans le [Centre d’administration Azure Active Directory](https://aad.portal.azure.com) une fois l’étape précédente effectuée, vous pouvez voir que l’agent d’authentification est indiqué comme étant **inactif**. Ceci est _normal_. L’agent d’authentification est automatiquement supprimé de la liste après 10 jours.

## <a name="i-already-use-ad-fs-to-sign-in-to-azure-ad-how-do-i-switch-it-to-pass-through-authentication"></a>J'utilise déjà AD FS pour me connecter Azure AD. Comment basculer vers l’authentification directe ?

Si vous procédez à une migration depuis AD FS (ou d’autres technologies de fédération) vers l’authentification directe, nous vous recommandons vivement de vous référer à notre guide de déploiement détaillé, publié [ici](https://github.com/Identity-Deployment-Guides/Identity-Deployment-Guides/blob/master/Authentication/Migrating%20from%20Federated%20Authentication%20to%20Pass-through%20Authentication.docx?raw=true).

## <a name="can-i-use-pass-through-authentication-in-a-multi-forest-active-directory-environment"></a>Puis-je utiliser l’authentification directe dans un environnement à plusieurs forêts Active Directory ?

Oui. Les environnements à plusieurs forêts sont pris en charge s’il existe des approbations de forêts (bidirectionnelles) entre les forêts Active Directory et si le routage du suffixe de leurs noms est configuré correctement.

## <a name="does-pass-through-authentication-provide-load-balancing-across-multiple-authentication-agents"></a>L’authentification directe assure-t-elle l’équilibrage de charge sur plusieurs agents d’authentification ?

Non, le fait d’installer plusieurs agents d’authentification directe n’assure que la [haute disponibilité](how-to-connect-pta-quick-start.md#step-4-ensure-high-availability), et non un équilibrage de charge déterministe entre les agents d’authentification. N’importe quel agent d’authentification (pris au hasard) peut traiter la demande de connexion d’un utilisateur.

## <a name="how-many-pass-through-authentication-agents-do-i-need-to-install"></a>Combien d’agents d’authentification directe dois-je installer ?

L’installation de plusieurs agents d’authentification directe assure une [haute disponibilité](how-to-connect-pta-quick-start.md#step-4-ensure-high-availability). Mais cela ne fournit pas un équilibrage de charge déterministe entre les agents d’authentification.

Envisagez la charge moyenne et les pics de charge lors des demandes de connexion que vous attendez de la part de votre locataire. À titre de référence, un seul agent d’authentification peut gérer entre 300 et 400 authentifications par seconde sur un serveur doté d’un CPU à 4 cœurs et de 16 Go de RAM.

Pour estimer le trafic réseau, suivez les instructions de dimensionnement suivantes :
- Chaque requête a une taille de charge utile de (0,5 K + 1 K * num_of_agents) octets. Ceci concerne les données d’Azure AD vers l’agent d’authentification. Ici, « num_of_agents » indique le nombre d’agents d’authentification inscrit sur votre abonné.
- Chaque réponse a une taille de charge utile de 1 kilooctet. Ceci concerne les données de l’agent d’authentification vers Azure AD.

Pour la plupart des clients, deux ou trois agents d’authentification au total suffisent à offrir la haute disponibilité et suffisamment de capacité. Vous devriez installer des agents d’authentification près de vos contrôleurs de domaine pour améliorer la latence de connexion.

>[!NOTE]
>Il existe une limite système de 40 agents d’authentification par client.

## <a name="why-do-i-need-a-cloud-only-global-administrator-account-to-enable-pass-through-authentication"></a>Pourquoi ai-je besoin d’un compte d’administrateur général de type cloud uniquement pour activer l’authentification directe ?

Il est recommandé d’activer ou de désactiver l’authentification directe à l’aide d’un compte d’administrateur général de type cloud uniquement. Découvrez comment [ajouter un compte d’administrateur général de type cloud uniquement](../fundamentals/add-users-azure-active-directory.md). Cette manière d’opérer garantit que vous ne serez pas verrouillé hors de votre locataire.

## <a name="how-can-i-disable-pass-through-authentication"></a>Comment désactiver l’authentification directe ?

Réexécutez l’assistant Azure AD Connect et modifiez la méthode de connexion utilisateur de l’authentification directe à une autre méthode. Cette modification désactive l’authentification directe sur le client et désinstalle l’agent d’authentification du serveur. Vous devez désinstaller manuellement les agents d’authentification à partir des autres serveurs.

## <a name="what-happens-when-i-uninstall-a-pass-through-authentication-agent"></a>Que se passe-t-il lorsque je désinstalle un agent d’authentification directe ?

La désinstallation d’un agent d’authentification directe à partir d’un serveur provoque l’interruption de l’acceptation des requêtes de connexion. Pour éviter d'interrompre la fonctionnalité de connexion de l'utilisateur sur votre client, assurez-vous qu'un autre agent d'authentification est en cours d'exécution avant de désinstaller un agent d'authentification directe.

## <a name="i-have-an-older-tenant-that-was-originally-setup-using-ad-fs--we-recently-migrated-to-pta-but-now-are-not-seeing-our-upn-changes-synchronizing-to-azure-ad--why-are-our-upn-changes-not-being-synchronized"></a>J’ai un ancien locataire qui a été configuré à l’origine avec AD FS.  Bien que nous ayons récemment migré vers PTA, nos modifications de nom d’utilisateur principal (UPN) ne se synchronisent pas avec Azure AD.  Pourquoi la synchronisation n’a-t-elle pas lieu ?

A : Il peut arriver que les modifications de noms UPN locaux ne se synchronisent pas dans les circonstances suivantes :

- Votre client Azure AD a été créé avant le 15 juin 2015.
- Vous étiez à l’origine fédéré avec votre client Azure AD par le biais d’AD FS pour l’authentification.
- Vous avez basculé vers des utilisateurs gérés avec PTA pour l’authentification.

En effet, le comportement par défaut des locataires créés avant le 15 juin 2015 consistait à bloquer les changements de noms UPN.  Si souhaitez les débloquer, exécutez la cmdlet PowerShell suivante :  

`Set-MsolDirSyncFeature -Feature SynchronizeUpnForManagedUsers -Enable $True`

Les locataires créés après le 15 juin 2015 ont le comportement par défaut, c’est-à-dire qu’ils synchronisent les modifications de noms UPN.   



## <a name="next-steps"></a>Étapes suivantes
- [Limitations actuelles](how-to-connect-pta-current-limitations.md) : Découvrez les scénarios pris en charge et ceux qui ne le sont pas.
- [Démarrage rapide](how-to-connect-pta-quick-start.md) : soyez opérationnel sur l’authentification directe Azure AD.
- [Migrer à partir d’AD FS vers l’authentification directe](https://github.com/Identity-Deployment-Guides/Identity-Deployment-Guides/blob/master/Authentication/Migrating%20from%20Federated%20Authentication%20to%20Pass-through%20Authentication.docx?raw=true) : guide détaillé de la migration d’AD FS (ou d’autres technologies de fédération) vers l’authentification directe.
- [Verrouillage intelligent](../authentication/howto-password-smart-lockout.md) : apprenez à configurer la fonctionnalité Verrouillage intelligent sur votre locataire pour protéger les comptes d'utilisateur.
- [Présentation technique approfondie](how-to-connect-pta-how-it-works.md) : découvrez comment fonctionne la fonctionnalité d'authentification directe.
- [Résoudre les problèmes](tshoot-connect-pass-through-authentication.md) : apprenez à résoudre les problèmes courants liés à la fonctionnalité d’authentification directe.
- [Présentation approfondie de sécurité](how-to-connect-pta-security-deep-dive.md) : obtenez des informations techniques détaillées sur la fonctionnalité d’authentification directe.
- [Authentification unique fluide Azure AD](how-to-connect-sso.md) : explorez en détail cette fonctionnalité complémentaire.
- [UserVoice](https://feedback.azure.com/forums/169401-azure-active-directory/category/160611-directory-synchronization-aad-connect) : utilisez le Forum Azure Active Directory pour consigner de nouvelles demandes de fonctionnalités.

---
title: Jeton d’actualisation principal (PRT) et Azure AD - Azure Active Directory
description: Quel est le rôle du jeton d’actualisation principal (PRT) dans Azure Active Directory et comment le gérer ?
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: conceptual
ms.date: 07/20/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: ravenn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 3f2b059bb6ae63d7f427ce970b2538da922e2dec
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94837261"
---
# <a name="what-is-a-primary-refresh-token"></a>Qu’est-ce qu’un jeton d’actualisation principal ?

Un jeton d’actualisation principal (PRT) est un artefact clé d’authentification Azure AD sur les appareils Windows 10, Windows Server 2016 et versions ultérieures, iOS et Android. Il s’agit d’un jeton JSON Web Token (JWT) émis spécialement pour les brokers à jetons Microsoft internes, qui permet d’activer l’authentification unique (SSO) sur les applications utilisées sur ces appareils. Cet article explique en détail comment un PRT est émis, utilisé et protégé sur les appareils Windows 10.

Cet article suppose que vous connaissez déjà les différents états des appareils disponibles dans Azure AD ainsi que le fonctionnement de l’authentification unique sous Windows 10. Pour plus d’informations sur les appareils dans Azure AD, consultez l’article [Présentation de la gestion des appareils dans Azure Active Directory](overview.md).

## <a name="key-terminology-and-components"></a>Terminologie et composants clés

Les composants Windows suivants jouent un rôle clé dans la demande et l’utilisation d’un PRT :

* **Fournisseur d’authentification cloud** (CloudAP): CloudAP est un fournisseur d’authentification moderne pour les connexions Windows. Il vérifie l’identité des utilisateurs ouvrant une session sur un appareil Windows 10. CloudAP fournit une infrastructure de plug-in sur laquelle les fournisseurs d’identités peuvent s’appuyer pour activer l’authentification à Windows avec les informations d’identification de ce fournisseur d’identité.
* **Gestonnaire de comptes web** (WAM): WAM est le broker à jetons par défaut sur les appareils Windows 10. WAM fournit également une infrastructure de plug-in sur laquelle les fournisseurs d’identités peuvent s’appuyer pour activer l’authentification unique sur les applications qui en dépendent.
* **Plug-in Azure AD CloudAP** : Un plug-in propre à Azure AD basé sur l’infrastructure CloudAP, qui vérifie les informations d’identification de l’utilisateur avec Azure AD pendant la connexion à Windows.
* **Plug-in Azure AD WAM** : Un plug-in propre à Azure AD basé sur l’infrastructure WAM, qui active l’authentification unique s’appuyant sur Azure AD pour l’authentification.
* **Dsreg** : Un composant spécifique d’Azure AD sur Windows 10, qui gère le processus d’inscription de l’appareil quel que soit son état.
* **Module de plateforme sécurisée (TPM)** : Un TPM est un composant matériel intégré à un appareil, qui fournit des fonctions de sécurité matérielles pour les clés secrètes d’utilisateur et d’appareil. Vous trouverez plus de détails dans l’article présentant une [vue d’ensemble de la technologie de module de plateforme sécurisée (TPM)](/windows/security/information-protection/tpm/trusted-platform-module-overview).

## <a name="what-does-the-prt-contain"></a>Que contient le PRT ?

Un PRT contient les revendications généralement contenues dans n’importe quel jeton d’actualisation Azure AD. En outre, certaines revendications propres aux appareils sont incluses dans le PRT. Les voici :

* **ID d’appareil** : Un PRT est émis pour un utilisateur sur un appareil spécifique. La revendication d’ID d’appareil `deviceID` détermine l’appareil sur lequel le PRT a été émis pour l’utilisateur. Cette revendication est émise ultérieurement dans les jetons obtenus via la PRT. La revendication d’ID d’appareil est utilisée pour déterminer l’autorisation d’accès conditionnel en fonction de l’état de l’appareil ou de sa conformité.
* **Clé de session** : La clé de session est une clé symétrique chiffrée, générée par le service d’authentification Azure AD, émise en même temps que le PRT. La clé de session sert de preuve de possession lorsqu’un PRT est utilisé pour obtenir des jetons pour d’autres applications.

### <a name="can-i-see-whats-in-a-prt"></a>Puis-je voir de quoi se compose un PRT ?

Un PRT est un objet blob opaque, envoyé à partir d’Azure AD, dont le contenu n’est connu d’aucun composant client. Vous ne pouvez ce qu’un PRT contient.

## <a name="how-is-a-prt-issued"></a>Comment un PRT est-il émis ?

L’inscription de l’appareil est un prérequis pour l’authentification basée sur les appareils dans Azure AD. Un PRT est émis pour les utilisateurs uniquement sur les appareils inscrits. Pour plus de détails sur l’inscription des appareils, consultez l’article sur [Windows Hello Entreprise et l’inscription des appareils](/windows/security/identity-protection/hello-for-business/hello-how-it-works-device-registration). Pendant l’inscription d’un appareil, le composant dsreg génère deux ensembles de paires de clés de chiffrement :

* Clé d’appareil (dkpub/dkpriv)
* Clé de transport (tkpub/tkpriv)

Les clés privées sont liées au TPM de l’appareil si ce dernier dispose d’un TPM valide et fonctionnel, tandis que les clés publiques sont envoyées à Azure AD pendant le processus d’inscription de l’appareil. Ces clés sont utilisées pour valider l’état de l’appareil en cas de demande de PRT.

Le PRT est émis lors de l’authentification de l’utilisateur sur un appareil Windows 10 dans deux scénarios :

* Appareil **joint à Azure AD** ou à **Azure AD hybride** : Un PRT est émis au cours de l’ouverture de session Windows lorsqu’un utilisateur se connecte avec ses informations d’identification de leur organisation. Un PRT est émis avec toutes les informations d’identification prises en charge par Windows 10, par exemple, un mot de passe et Windows Hello Entreprise. Dans ce scénario, le plug-in Azure AD CloudAP est l’autorité principale pour le PRT.
* **Appareil inscrit sur Azure AD** : Un PRT est émis lorsqu’un utilisateur ajoute un compte professionnel secondaire sur leur appareil Windows 10. Les utilisateurs peuvent ajouter un compte à Windows 10 de deux manières différentes :  
   * Ajout d’un compte via l’invite **Utiliser ce compte partout sur votre appareil** après vous être connecté à une application (par exemple Outlook)
   * Ajout d’un compte depuis **Paramètres** > **Comptes** > **Accès Professionnel ou Scolaire** > **Se connecter**

Dans les scénarios avec appareils inscrits dans Azure AD, le plug-in Azure AD WAM est l’autorité principale pour le PRT puisque l’ouverture de session Windows n’utilise pas ce compte Azure AD.

> [!NOTE]
> Les fournisseurs d’identités tiers doivent prendre en charge le protocole WS-Trust pour autoriser l’émission d’un PRT sur les appareils Windows 10. Sans WS-Trust, le PRT ne peut pas être émis pour les utilisateurs sur les appareils faisant l’objet d’une jonction hybride Azure AD ou sur les appareils joints à Azure AD. Sur ADFS, seuls les points de terminaison usernamemixed sont nécessaires. adfs/services/trust/2005/windowstransport et adfs/services/trust/13/windowstransport doivent tous les deux être activés en tant que points de terminaison uniquement accessibles sur intranet. Ils **NE doivent PAS être exposés** en tant que points de terminaison accessibles sur extranet via le proxy d’application web

## <a name="what-is-the-lifetime-of-a-prt"></a>Quelle est la durée de validité d’un PRT ?

Une fois émis, un PRT est valide pendant 14 jours et il est renouvelé en continu tant que l’utilisateur utilise activement l’appareil.  

## <a name="how-is-a-prt-used"></a>Comment un PRT est-il utilisé ?

Un PRT est utilisé par deux composants clés dans Windows :

* **Plug-in Azure AD CloudAP** : Pendant la connexion à Windows, le plug-in Azure AD CloudAP fait une demande de PRT depuis Azure AD en utilisant les informations d’identification fournies par l’utilisateur. Il met également en cache le PRT pour permettre la connexion avec des informations mises en cache lorsque l’utilisateur n’a pas accès à une connexion internet.
* **Plug-in Azure AD WAM** : Lorsque les utilisateurs tentent d’accéder aux applications, le plug-in Azure AD WAM utilise le PRT pour activer l’authentification unique sur Windows 10. Le plug-in Azure AD WAM utilise le PRT pour demander des jetons d’actualisation et d’accès pour les applications qui s’appuient sur WAM pour les demandes de jeton. Il active également l’authentification unique sur les navigateurs en injectant le PRT dans les demandes de navigateur. L’authentification unique de navigateur dans Windows 10 est prise en charge sur Microsoft Edge (en natif) et Chrome (via les extensions [Windows 10 Accounts](https://chrome.google.com/webstore/detail/windows-10-accounts/ppnbnpeolgkicgegkbkbjmhlideopiji?hl=en) ou [Office Online](https://chrome.google.com/webstore/detail/office/ndjpnladcallmjemlbaebfadecfhkepb?hl=en)).

## <a name="how-is-a-prt-renewed"></a>Comment un PRT est-il renouvelé ?

Un PRT est renouvelé selon deux méthodes différentes :

* **Plug-in Azure AD CloudAP toutes les 4 heures** : Le plug-in CloudAP renouvelle le PRT toutes les 4 heures pendant la connexion Windows. Si l’utilisateur n’a pas de connexion Internet pendant ce temps, le plug-in CloudAP renouvellera le PRT une fois que l’appareil se sera connecté à Internet.
* **Plug-in Azure AD WAM pendant les demandes de jeton d’application** : Le plug-in WAM permet l’authentification unique sur les appareils Windows 10 en activant les demandes de jeton silencieuses pour les applications. Le plug-in WAM peut renouveler le PRT pendant ces demandes de jeton de deux manières différentes :
   * Une application demande au plug-in WAM un jeton d’accès silencieusement, mais aucun jeton d’actualisation n’est disponible pour cette application. Dans ce cas, WAM utilise le PRT pour demander un jeton pour l’application et récupère un nouveau PRT dans la réponse.
   * Une application demande un jeton d’accès au plug-in WAM, mais le PRT n’est pas valide ou Azure AD requiert une autorisation supplémentaire (par exemple Azure AD Multi-Factor Authentication). Dans ce scénario, WAM lance une ouverture de session interactive qui oblige l’utilisateur à s’authentifier de nouveau ou à effectuer une vérification supplémentaire, et un nouveau PRT est émis en cas d’authentification réussie.

Dans un environnement ADFS, une ligne de vue directe sur le contrôleur de domaine n’est pas nécessaire pour renouveler le PRT. Le renouvellement du PRT requiert uniquement des points de terminaison /adfs/services/trust/2005/usernamemixed and /adfs/services/trust/13/usernamemixed activés sur le proxy à l’aide du protocole WS-Trust.

Des points de terminaison de transport Windows sont requis pour l’authentification par mot de passe uniquement si un mot de passe est modifié, pas pour le renouvellement de PRT.

### <a name="key-considerations"></a>Considérations relatives aux clés

* Un PRT est uniquement émis et renouvelé pendant les authentifications d’applications natives. Un PRT n’est pas renouvelé ou émis pendant une session de navigateur.
* Sur les appareils joints à Azure AD et Azure AD hybride, le plug-in CloudAP est l’autorité principale pour un PRT. Si un PRT est renouvelé pendant une demande de jeton WAM, il est renvoyé au plug-in CloudAP, qui vérifie sa validité avec Azure AD avant de l’accepter.

## <a name="how-is-the-prt-protected"></a>Comment le PRT est-il protégé ?

Un PRT est protégé via une liaison à l’appareil sur lequel l’utilisateur s’est connecté. Azure AD et Windows 10 protègent les PRT via les méthodes suivantes :

* **À la première connexion** : À la première connexion, un PRT est émis via signature de demande en utilisant la clé d’appareil générée de manière chiffrée au moment de l’inscription de l’appareil. Sur un appareil avec TPM valide et fonctionnel, la clé d’appareil est sécurisée par le TPM, empêchant tout accès malveillant. Aucun PRT n’est émis si la signature de clé d’appareil correspondante ne peut pas être validée.
* **Au cours des demandes et des renouvellements de jeton** : Lorsqu’un PRT est émis, Azure AD émet également une clé de session chiffrée sur l’appareil. Elle est chiffrée à l’aide de la clé de transport publique (tkpub) générée et envoyée à Azure AD dans le cadre de l’inscription de l’appareil. Cette clé de session peut uniquement être déchiffrée par la clé de transport privée (tkpriv) sécurisée par le TPM. La clé de session est la clé de preuve de possession (POP) pour toutes les demandes envoyées à Azure AD.  La clé de session est également protégée par le TPM et aucun autre composant du système d’exploitation ne peut y accéder. Les demandes de jeton ou les demandes de renouvellement de PRT sont signées en toute sécurité par cette clé de session via le TPM. Par conséquent, elles ne peuvent pas être altérées. Azure AD invalidera toutes les demandes de l’appareil non signées par la clé de session correspondante.

En sécurisant ces clés avec le TPM, les personnes malveillantes ne peuvent pas dérober les clés ni relire le PRT ailleurs, car le TPM n’est pas accessible même si le pirate est en possession physique de l’appareil.  Par conséquent, l’utilisation d’un TPM renforce considérablement la sécurité des appareils joints à Azure AD, à Azure AD hybride et inscrits dans Azure AD et les protège contre le vol d’informations d’identification. Pour des performances et une fiabilité améliorées, TPM 2.0 est la version recommandée dans tous les scénarios d’inscription d’appareils Azure AD sous Windows 10.

### <a name="how-are-app-tokens-and-browser-cookies-protected"></a>Comment les jetons d’application et les cookies de navigateur sont-ils protégés ?

**Jetons d’application** : Lorsqu’une application demande un jeton via WAM, Azure AD émet un jeton d’actualisation et un jeton d’accès. Toutefois, WAM ne retourne que le jeton d’accès à l’application et il sécurise le jeton d’actualisation dans sa mémoire cache en le chiffrant avec la clé DPAPI de l’utilisateur. WAM utilise en toute sécurité le jeton d’actualisation en signant les demandes avec la clé de session pour émettre des jetons d’accès supplémentaires. La clé DPAPI est sécurisée par une clé symétrique Azure AD dans Azure AD lui-même. Quand l’appareil doit déchiffrer le profil utilisateur avec la clé DPAPI, Azure AD fournit la clé DPAPI chiffrée par la clé de session, que le plug-in CloudAP demande au TPM de déchiffrer. Cette fonctionnalité garantit la cohérence de la sécurisation des jetons d’actualisation et évite aux applications d’implémenter leurs propres mécanismes de protection.  

**Cookies de navigateur** : Dans Windows 10, Azure AD prend en charge l’authentification unique de navigateur dans Internet Explorer et Microsoft Edge en natif, et dans Google Chrome via l’extension Windows 10 Accounts. Ce dispositif de sécurité est conçu non seulement pour protéger les cookies, mais également les points de terminaison sur lesquels les cookies sont envoyés. Les cookies de navigateur sont protégés de la même façon que les PRT, c’est-à-dire en utilisant la clé de session pour signer et protéger les cookies.

Lorsqu’un utilisateur lance une interaction de navigateur, le navigateur (ou l’extension) appelle un hôte de client natif COM. L’hôte de client natif garantit que la page provient de l’un des domaines autorisés. Le navigateur pourrait envoyer d’autres paramètres à l’hôte de client natif, y compris une valeur à usage unique, toutefois, l’hôte de client natif garantit la validation du nom d’hôte. L’hôte de client natif demande un cookie de PRT au plug-in CloudAP, qui le crée et le signe avec la clé de session protégée par le TPM. Comme le cookie de PRT est signé par la clé de session, il ne peut pas être altéré. Ce cookie de PRT est inclus dans l’en-tête de demande afin qu’Azure AD valide l’appareil qui en est à l’origine. Dans le navigateur Chrome, seule l’extension explicitement définie dans le manifeste de l’hôte de client natif peut l’appeler, ce qui empêche les extensions arbitraires de faire ces demandes. Une fois qu’Azure AD valide le cookie de PRT, il émet un cookie de session sur le navigateur. Ce cookie de session contient également la même clé de session que celle émise avec un PRT. Lors des demandes suivantes, la clé de session est validée, ce qui lie le cookie à l’appareil et empêche les relectures à partir d’un autre emplacement.

## <a name="when-does-a-prt-get-an-mfa-claim"></a>Dans quels cas les PRT reçoivent-ils une revendication MFA ?

Un PRT peut recevoir une revendication MFA (authentification multifacteur) dans des scénarios spécifiques. Lorsqu’un PRT basé sur l’authentification multifacteur est utilisé pour demander des jetons pour les applications, la revendication MFA est transférée à ces jetons d’application. Cette fonctionnalité offre une expérience transparente aux utilisateurs en empêchant les demandes d’authentification multifacteur pour chaque application qui en a besoin. Un PRT peut recevoir une revendication MFA dans les cas suivants :

* **Connexion avec Windows Hello Entreprise** : Windows Hello Entreprise remplace les mots de passe et utilise des clés de chiffrement pour fournir une authentification à deux facteurs forte. Windows Hello Entreprise est propre à l’utilisateur de l’appareil, et lui-même exige une authentification multifacteur pour l’approvisionnement. Quand un utilisateur se connecte avec Windows Hello Entreprise, son PRT reçoit une revendication MFA. Ce scénario s’applique également aux utilisateurs se connectant à l’aide de cartes à puce, si l’authentification par carte à puce génère une revendication MFA de la part d’ADFS.
   * Comme Windows Hello Entreprise est considéré comme authentification multifacteur, la revendication MFA est mise à jour lorsque le PRT lui-même est actualisé. Par conséquent, la durée de l’authentification multifacteur s’étend en continu lorsque les utilisateurs se connectent avec Windows Hello Entreprise.
* **Authentification multifacteur pendant la connexion interactive WAM** : Pendant une demande de jeton via WAM, si un utilisateur doit avoir recours à l’authentification multifacteur pour accéder à l’application, le PRT renouvelé pendant cette interaction contient une revendication MFA.
   * Dans ce cas, la revendication MFA n’est pas actualisée en continu. Par conséquent, la durée de validité de l’authentification multifacteur est basée sur la durée de vie définie sur le répertoire.
   * Lorsqu’un PRT et un RT existants sont utilisés pour accéder à une application, le PRT et le RT sont considérés comme la première preuve d’authentification. Un nouveau AT sera requis avec une deuxième preuve et une revendication MFA imprimée. Cela génère également un nouveau PRT et RT.
* **Authentification multifacteur pendant l’inscription d’un appareil** : Si un administrateur a configuré ses paramètres d’appareil dans Azure AD pour que [soit exigée une authentification multifacteur pour inscrire des appareils](device-management-azure-portal.md#configure-device-settings), l’utilisateur doit utiliser l’authentification multifacteur pour terminer l’inscription. Pendant ce processus, le PRT émis pour l’utilisateur reçoit la revendication MFA lors de l’inscription. Cette fonctionnalité s’applique uniquement à l’utilisateur qui a effectué l’opération de jointure, pas aux autres utilisateurs qui se connectent sur cet appareil.
   * Comme pour la connexion interactive WAM, la revendication MFA n’est pas mise à jour en continu. Par conséquent, la durée de validité de l’authentification multifacteur est basée sur la durée de vie définie sur le répertoire.

Windows 10 gère une liste partitionnée de PRT pour chaque information d’identification. Par conséquent, il existe un PRT pour chacune : Windows Hello Entreprise, mot de passe ou carte à puce. Ce partitionnement garantit que les revendications MFA sont isolées selon les informations d’identification utilisées et qu’elles ne sont pas mélangées pendant les demandes de jeton.

## <a name="how-is-a-prt-invalidated"></a>Dans quel cas un PRT est-il invalidé ?

Un PRT est invalidé dans les scénarios suivants :

* **Utilisateur non valide** : Si un utilisateur est supprimé ou désactivé dans Azure AD, ses PRT sont invalidés et ne peuvent pas être utilisés pour obtenir des jetons pour les applications. Si un utilisateur supprimé ou désactivé s’est déjà connecté à un appareil auparavant, la connexion à l’aide des informations mises en cache leur ouvrirait la session, jusqu’à ce que CloudAP se rende compte de leur état non valide. Une fois que CloudAP détermine que l’utilisateur n’est pas valide, il bloque les ouvertures de session suivantes. Un utilisateur non valide est bloqué automatiquement : il ne peut pas se connecter à de nouveaux appareils qui n’ont pas ses informations d’identification mises en cache.
* **Appareil non valide** : Si un appareil est supprimé ou désactivé dans Azure AD, les PRT obtenus sur ce dernier sont invalidés et ne peuvent pas être utilisés pour obtenir des jetons pour les autres applications. Si un utilisateur s’est déjà connecté à un appareil non valide, il peut continuer à le faire. En revanche, tous les jetons sur l’appareil sont invalidés et l’utilisateur ne peut accéder à aucune ressource via authentification unique à partir de cet appareil.
* **Modification de mot de passe** : Quand un utilisateur modifie son mot de passe, le PRT obtenu avec le mot de passe précédent est invalidé par Azure AD. La modification du mot de passe génère un nouveau PRT pour l’utilisateur. Cette invalidation peut se produire de deux manières différentes :
   * Si l’utilisateur se connecte à Windows avec son nouveau mot de passe, CloudAP ignore l’ancien PRT et demande à Azure AD d’émettre un nouveau PRT correspondant au nouveau mot de passe. Si l’utilisateur ne dispose pas d’une connexion Internet, le nouveau mot de passe ne peut pas être validé. Windows pourrait demander à l’utilisateur d’entrer son ancien mot de passe.
   * Si un utilisateur a ouvert une session avec son ancien mot de passe ou s’il a modifié son mot de passe après s’être connecté à Windows, l’ancien PRT est utilisé pour les demandes de jeton WAM. Dans ce scénario, l’utilisateur est invité à s’authentifier de nouveau au cours de la demande de jeton WAM et un nouveau PRT est émis.
* **Problèmes de TPM** : Parfois, le TPM d’un appareil peut être instable ou tomber en panne, conduisant à l’inaccessibilité des clés sécurisées par ce dernier. Dans ce cas, il est impossible pour l’appareil d’obtenir un PRT ou de demander des jetons avec un PRT existant, car il n’a pas de preuve de possession des clés de chiffrement. Par conséquent, tous les PRT existants sont invalidés par Azure AD. Lorsque Windows 10 détecte une défaillance, il lance un flux de récupération pour réinscrire l’appareil avec de nouvelles clés de chiffrement. Avec un appareil joint à Azure AD hybride, tout comme lors de l’inscription initiale, la récupération se produit silencieusement, sans entrée utilisateur. Pour les appareils joints ou inscrits à Azure AD, la récupération doit être effectuée par un utilisateur disposant de privilèges d’administrateur sur l’appareil. Dans ce scénario, le flux de récupération est lancé par une invite Windows qui guide l’utilisateur dans les étapes de récupération de l’appareil.

## <a name="detailed-flows"></a>Flux détaillés

Les diagrammes suivants illustrent les détails sous-jacents de l’émission, du renouvellement et de l’utilisation d’un PRT pour demander un jeton d’accès pour une application. En outre, ces étapes décrivent également comment les mécanismes de sécurité mentionnés ci-dessus sont appliqués pendant ces interactions.

### <a name="prt-issuance-during-first-sign-in"></a>Émission du PRT à la première connexion

![Diagramme détaillé du flux d’émission du PRT à la première connexion](./media/concept-primary-refresh-token/prt-initial-sign-in.png)

> [!NOTE]
> Sur les appareils joints à Azure AD, cet échange se produit de façon synchrone. Le PRT doit être émis avant que l’utilisateur ne puisse ouvrir une session Windows. Sur les appareils joints à Azure AD hybride, l’Active Directory local est l’autorité principale. Par conséquent, l’utilisateur doit seulement attendre d’acquérir un ticket TGT pour se connecter, tandis que l’émission du PRT se déroule de façon asynchrone. Ce scénario ne s’applique pas aux appareils inscrits dans Azure AD, car l’ouverture de session n’utilise pas les informations d’identification Azure AD.

| Étape | Description |
| :---: | --- |
| Un | L’utilisateur entre son mot de passe dans l’interface utilisateur de connexion. Le processus LogonUI transmet les informations d’identification dans une mémoire tampon d’authentification à l’autorité de sécurité locale, qui les transmet à son tour en interne au fournisseur d’authentification cloud. Le fournisseur d’authentification cloud transfère cette demande au plug-in CloudAP. |
| B | Le plug-in CloudAP lance une demande de découverte de domaine pour identifier le fournisseur d’identité pour l’utilisateur. Si le locataire de l’utilisateur possède un programme d’installation de fournisseur de fédération, Azure AD renvoie le point de terminaison MEX (Metadata Exchange) de ce fournisseur. Dans le cas contraire, Azure AD indique que l’utilisateur est géré, ce qui signifie qu’il peut s’authentifier avec Azure AD. |
| C | Si l’utilisateur est géré, CloudAP obtiendra la valeur à usage unique à partir d’Azure AD. Si l’utilisateur est fédéré, le plug-in CloudAP demande un jeton SAML au fournisseur de fédération avec les informations d’identification de l’utilisateur. Une fois le jeton SAML reçu, il demande une valeur à usage unique à Azure AD. |
| D | Le plug-in CloudAP crée la demande d’authentification avec les informations d’identification de l’utilisateur, une valeur à usage unique et une étendue de broker, signe la demande avec la clé d’appareil (dkpriv) et l’envoie à Azure AD. Dans un environnement fédéré, le plug-in CloudAP utilise le jeton SAML renvoyé par le fournisseur de fédération et non pas les informations d’identification de l’utilisateur. |
| E | Azure AD valide les informations d’identification de l’utilisateur, la valeur à usage unique et la signature de l’appareil, il vérifie que l’appareil est valide dans le locataire et il émet le PRT chiffré. En même temps que le PRT, Azure AD émet également une clé symétrique, appelée clé de session, chiffrée par Azure AD à l’aide de la clé de transport (tkpub). En outre, la clé de session est également incorporée dans le PRT. Cette clé de session agit comme une clé de preuve de possession (PoP) pour les demandes suivantes avec ce PRT. |
| F | Le plug-in CloudAP transmet le PRT et la clé de session chiffrés au fournisseur d’authentification cloud. Ce dernier demande au TPM de déchiffrer la clé de session avec la clé de transport (tkpriv) et de la chiffrer de nouveau avec sa propre clé. Le CloudAP stocke la clé de session chiffrée dans son cache, ainsi que le PRT. |

### <a name="prt-renewal-in-subsequent-logons"></a>Renouvellement du PRT lors des ouvertures de session suivantes

![Renouvellement du PRT lors des ouvertures de session suivantes](./media/concept-primary-refresh-token/prt-renewal-subsequent-logons.png)

| Étape | Description |
| :---: | --- |
| Un | L’utilisateur entre son mot de passe dans l’interface utilisateur de connexion. Le processus LogonUI transmet les informations d’identification dans une mémoire tampon d’authentification à l’autorité de sécurité locale, qui les transmet à son tour en interne au fournisseur d’authentification cloud. Le fournisseur d’authentification cloud transfère cette demande au plug-in CloudAP. |
| B | Si l’utilisateur a déjà ouvert une session, Windows lance la connexion avec les informations mises en cache et valide les informations d’identification pour connecter l’utilisateur. Toutes les 4 heures, le plug-in CloudAP lance le renouvellement du PRT de façon asynchrone. |
| C | Le plug-in CloudAP lance une demande de découverte de domaine pour identifier le fournisseur d’identité pour l’utilisateur. Si le locataire de l’utilisateur possède un programme d’installation de fournisseur de fédération, Azure AD renvoie le point de terminaison MEX (Metadata Exchange) de ce fournisseur. Dans le cas contraire, Azure AD indique que l’utilisateur est géré, ce qui signifie qu’il peut s’authentifier avec Azure AD. |
| D | Si l’utilisateur est fédéré, le plug-in CloudAP demande un jeton SAML au fournisseur de fédération avec les informations d’identification de l’utilisateur. Une fois le jeton SAML reçu, il demande une valeur à usage unique à Azure AD. Si l’utilisateur est géré, CloudAP obtiendra directement la valeur à usage unique à partir d’Azure AD. |
| E | Le plug-in CloudAP crée la demande d’authentification avec les informations d’identification de l’utilisateur, une valeur à usage unique et le PRT existant, signe la demande avec la clé de session et l’envoie à Azure AD. Dans un environnement fédéré, le plug-in CloudAP utilise le jeton SAML renvoyé par le fournisseur de fédération et non pas les informations d’identification de l’utilisateur. |
| F | Azure AD valide la signature de la clé de session en comparant cette dernière à la clé de session incorporée dans le PRT, il valide la valeur à usage unique, vérifie que l’appareil est valide dans le locataire et émet un nouveau PRT. Comme indiqué précédemment, le PRT est à nouveau accompagné par une clé de session chiffrée avec la clé de transport (tkpub). |
| G | Le plug-in CloudAP transmet le PRT et la clé de session chiffrés au fournisseur d’authentification cloud. Ce dernier demande au TPM de déchiffrer la clé de session avec la clé de transport (tkpriv) et de la chiffrer de nouveau avec sa propre clé. Le CloudAP stocke la clé de session chiffrée dans son cache, ainsi que le PRT. |

> [!NOTE]
> Un PRT peut être renouvelé de manière externe sans nécessiter une connexion VPN quand les points de terminaison usernamemixed sont activés de manière externe.

### <a name="prt-usage-during-app-token-requests"></a>Utilisation du PRT lors de demandes de jeton d’application

![Utilisation du PRT lors de demandes de jeton d’application](./media/concept-primary-refresh-token/prt-usage-app-token-requests.png)

| Étape | Description |
| :---: | --- |
| Un | Une application (par exemple Outlook, OneNote, etc.) initie une demande de jeton au WAM. Ce dernier demande à son tour au plug-in Azure AD WAM de traiter la demande de jeton. |
| B | Si un jeton d’actualisation est déjà disponible pour l’application, le plug-in Azure AD WAM l’utilise pour demander un jeton d’accès. Pour fournir la preuve de la liaison de l’appareil, le plug-in WAM signe la demande avec la clé de session. Azure AD valide la clé de session et émet un jeton d’accès ainsi qu’un nouveau jeton d’actualisation pour l’application, chiffrés avec la clé de session. Le plug-in WAM demande au plug-in Cloud AP de déchiffrer les jetons, et ce dernier demande à son tour au TPM de les déchiffrer à l’aide de la clé de session. Le plug-in WAM obtient alors les deux jetons. Ensuite, le plug-in WAM fournit uniquement le jeton d’accès à l’application, tandis qu’il chiffre à nouveau le jeton d’actualisation avec DPAPI et le stocke dans son propre cache.  |
| C |  Si aucun jeton d’actualisation n’est disponible pour l’application, le plug-in Azure AD WAM utilise le PRT pour demander un jeton d’accès. Pour fournir une preuve de possession, le plug-in WAM signe la demande contenant le PRT avec la clé de session. Azure AD valide la signature de la clé de session en comparant cette dernière à la clé de session incorporée dans le PRT, il vérifie que l’appareil est valide et émet un jeton d’accès et un jeton d’actualisation pour l’application. En outre, Azure AD peut émettre de nouveaux PRT (en fonction du cycle d’actualisation), tous chiffrés par la clé de session. |
| D | Le plug-in WAM demande au plug-in Cloud AP de déchiffrer les jetons, et ce dernier demande à son tour au TPM de les déchiffrer à l’aide de la clé de session. Le plug-in WAM obtient alors les deux jetons. Ensuite, le plug-in WAM fournit uniquement le jeton d’accès à l’application, tandis qu’il chiffre à nouveau le jeton d’actualisation avec DPAPI et le stocke dans son propre cache. Le plug-in WAM utilisera le jeton d’actualisation à l’avenir pour cette application. Le plug-in WAM restitue également le nouveau PRT au plug-in CloudAP, qui le valide avec Azure AD avant de le mettre à jour dans son propre cache. Le plug-in CloudAP utilisera le nouveau PRT à l’avenir. |
| E | Le gestionnaire de comptes web fournit le jeton d’accès qui vient d’être émis au plug-in WAM, qui à son tour le retransmet à l’application appelante.|

### <a name="browser-sso-using-prt"></a>Authentification unique dans le navigateur avec le PRT

![Authentification unique dans le navigateur avec le PRT](./media/concept-primary-refresh-token/browser-sso-using-prt.png)

| Étape | Description |
| :---: | --- |
| Un | Un utilisateur ouvre une session Windows avec ses informations d’identification pour obtenir un PRT. Une fois que l’utilisateur ouvre le navigateur, ce dernier (ou une extension) charge les URL du Registre. |
| B | Lorsqu’un utilisateur ouvre une URL de connexion Azure AD, le navigateur ou l’extension compare l’URL avec celles obtenues à partir du Registre. Si elles correspondent, le navigateur appelle l’hôte de client natif pour obtenir un jeton. |
| C | L’hôte de client natif vérifie l’appartenance de l’URL aux fournisseurs d’identités Microsoft (compte Microsoft ou Azure AD), extrait une valeur à usage unique envoyée de l’URL et effectue un appel au plug-in CloudAP pour obtenir un cookie de PRT. |
| D | Le plug-in CloudAP crée le cookie de PRT, se connecte avec la clé de session liée au TPM et le renvoie à l’hôte de client natif. Comme le cookie est signé par la clé de session, il ne peut pas être altéré. |
| E | L’hôte de client natif renvoie ce cookie de PRT au navigateur, qui l’inclut dans l’en-tête de demande x-ms-RefreshTokenCredential et demande les jetons à Azure AD. |
| F | Azure AD valide la signature de clé de session sur le cookie de PRT, vérifie la valeur à usage unique, s’assure que l’appareil est valide dans le locataire et émet un jeton d’ID pour la page web et un cookie de session chiffré pour le navigateur. |

> [!NOTE]
> Le flux d’authentification unique du navigateur décrit aux étapes ci-dessus ne s’applique pas aux sessions dans les modes privés tels que InPrivate dans Microsoft Edge ou Incognito dans Google Chrome (lors de l’utilisation de l’extension Comptes Microsoft).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la résolution des problèmes liés aux PRT, consultez l’article [Résolution des problèmes des appareils hybrides Windows 10 et Windows Server 2016 joints à Azure Active Directory](troubleshoot-hybrid-join-windows-current.md).

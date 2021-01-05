---
title: 'Azure AD Connect : Authentification unique transparente - Démarrage rapide | Microsoft Docs'
description: Cet article explique comment bien démarrer avec l’authentification unique transparente d’Azure Active Directory
services: active-directory
keywords: Qu’est-ce qu’Azure AD Connect, Installation d’Active Directory, Composants requis pour Azure AD, SSO, Authentification unique
documentationcenter: ''
author: billmath
manager: daveba
ms.assetid: 9f994aca-6088-40f5-b2cc-c753a4f41da7
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 04/16/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: eadb20bc570545356508d82c05e1746424a14b71
ms.sourcegitcommit: 2ba6303e1ac24287762caea9cd1603848331dd7a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97504387"
---
# <a name="azure-active-directory-seamless-single-sign-on-quickstart"></a>Authentification unique transparente Azure Active Directory : Démarrage rapide

## <a name="deploy-seamless-single-sign-on"></a>Déploiement de l'authentification unique transparente

L’authentification unique transparente (Seamless SSO) Azure Active Directory (Azure AD) connecte automatiquement les utilisateurs lorsque leurs ordinateurs d’entreprise sont connectés au réseau de l’entreprise. Seamless SSO offre à vos utilisateurs un accès facilité à vos applications dans le cloud sans nécessiter de composants sur site supplémentaires.

> [!NOTE]
> Cet article contient des références au terme *liste verte*, un terme que Microsoft n’utilise plus. Lorsque le terme sera supprimé du logiciel, nous le supprimerons de cet article.

Pour déployer l’authentification unique transparente, procédez comme suit.

## <a name="step-1-check-the-prerequisites"></a>Étape 1 : Vérifier les prérequis

Vérifiez que les prérequis suivants sont remplis :

* **Configurez votre serveur Azure AD Connect** : Si vous utilisez [l’authentification directe](how-to-connect-pta.md) comme méthode de connexion utilisateur, aucune vérification de prérequis n’est nécessaire. Si vous utilisez la [synchronisation de hachage de mot de passe](how-to-connect-password-hash-synchronization.md) comme méthode de connexion, et s’il existe un pare-feu entre Azure AD Connect et Azure AD, vérifiez les points suivants :
   - Vous utilisez Azure AD Connect 1.1.644.0 ou version ultérieure. 
   - Si votre pare-feu ou votre proxy le permet, ajoutez les connexions à la liste verte pour les URL **\*.msappproxy.net** via le port 443. Dans le cas contraire, autorisez l’accès aux [plages d’adresses IP du centre de données Azure](https://www.microsoft.com/download/details.aspx?id=41653), qui sont mises à jour chaque semaine. Cette condition préalable est applicable uniquement lorsque vous activez la fonctionnalité. Elle n'est pas requise pour les connexions d'utilisateur réelles.

    >[!NOTE]
    >Les versions 1.1.557.0, 1.1.558.0, 1.1.561.0 et 1.1.614.0 d’Azure AD Connect comportent un problème lié à la synchronisation de hachage de mot de passe. Si vous _ne prévoyez pas_ d’utiliser la synchronisation de hachage de mot de passe en même temps que l’authentification directe, lisez les [Notes de publication Azure AD Connect](./reference-connect-version-history.md) pour en savoir plus.

* **Utiliser une topologie Azure AD Connect prise en charge**  : veillez à utiliser une des topologies d’Azure Connect AD prises en charge et décrites [ici](plan-connect-topologies.md).

    >[!NOTE]
    >L’authentification unique fluide prend en charge plusieurs forêts AD, qu’il existe ou non des approbations AD entre elles.

* **Définir les informations d’identification de l’administrateur de domaine** : Vous devez disposer des informations d’identification de l’administrateur de domaine pour chaque forêt Active Directory, dans les scénarios suivants :
    * Synchronisation avec Azure AD via Azure AD Connect.
    * La forêt contient des utilisateurs pour lesquels vous souhaitez activer Seamless SSO.
    
* **Activer l’authentification moderne** : Vous devez activer [l’authentification moderne](/office365/enterprise/modern-auth-for-office-2013-and-2016) sur votre locataire pour que cette fonctionnalité fonctionne.

* **Utiliser les dernières versions de clients Microsoft 365** : Pour obtenir une utilisation de l’authentification unique sans assistance avec les clients Microsoft 365 (Outlook, Word, Excel, etc.), vos utilisateurs doivent utiliser la version 16.0.8730.xxxx ou une version ultérieure.

## <a name="step-2-enable-the-feature"></a>Étape 2 : Activer la fonctionnalité

Activez Seamless SSO via [Azure AD Connect](whatis-hybrid-identity.md).

>[!NOTE]
> Vous pouvez également [activer l’authentification unique fluide à l’aide de PowerShell](tshoot-connect-sso.md#manual-reset-of-the-feature) si Azure AD Connect ne répond pas à vos exigences. Utilisez cette option si vous disposez de plusieurs domaines par forêt Active Directory et que vous voulez être plus ciblé sur le domaine pour lequel vous voulez activer l’authentification unique transparente.

Si vous procédez à une nouvelle installation d’Azure AD Connect, choisissez le [chemin d’installation personnalisé](how-to-connect-install-custom.md). Sur la page **Connexion utilisateur**, cochez la case **Activer l’authentification unique**.

>[!NOTE]
> L’option est uniquement disponible pour la sélection si la méthode d’authentification est **Synchronisation de hachage du mot de passe** ou **Authentification directe**.

![Azure AD Connect : Connexion de l’utilisateur](./media/how-to-connect-sso-quick-start/sso8.png)

Si vous disposez déjà d’une installation Azure AD Connect, sélectionnez la page **Modifier la connexion utilisateur** dans Azure AD Connect, puis cliquez sur **Suivant**. Si vous utilisez Azure AD Connect version 1.1.880.0 ou ultérieure, l’option **Activer l’authentification unique** est sélectionnée par défaut. Si vous utilisez des versions antérieures d’Azure AD Connect, sélectionnez l’option **Activer l’authentification unique**.

![Azure AD Connect : Modifier la connexion de l’utilisateur](./media/how-to-connect-sso-quick-start/changeusersignin.png)

Suivez les instructions de l’Assistant jusqu’à ce que vous accédiez à la page **Activer l’authentification unique**. Fournissez les informations d'identification de l'administrateur de domaine pour chaque forêt Active Directory, dans les scénarios suivants :

* Synchronisation avec Azure AD via Azure AD Connect.
* La forêt contient des utilisateurs pour lesquels vous souhaitez activer Seamless SSO.

À la fin de l’Assistant, l’authentification unique transparente est activée sur votre locataire.

>[!NOTE]
> Les informations d’identification de l'administrateur de domaine ne sont pas stockées dans Azure AD Connect ni dans Azure AD. Elles sont uniquement utilisées pour activer la fonctionnalité.

Suivez ces instructions pour vérifier que vous avez activé l’authentification unique transparente correctement :

1. Connectez-vous au [Centre d’administration Azure Active Directory](https://aad.portal.azure.com) à l’aide des informations d’identification d’administrateur général de votre locataire.
2. Sélectionnez **Active Directory** dans le volet de gauche.
3. Sélectionnez ensuite **Azure AD Connect**.
4. Vérifiez que la fonctionnalité **Authentification unique transparente** est **activée**.

![Portail Azure : Volet Azure AD Connect](./media/how-to-connect-sso-quick-start/sso10.png)

>[!IMPORTANT]
> L’authentification unique fluide crée un compte d’ordinateur nommé `AZUREADSSOACC` sur votre instance Active Directory (AD) locale, dans chaque forêt AD. Pour des raisons de sécurité, le compte d’ordinateur `AZUREADSSOACC` doit être fortement protégé. Seuls des administrateurs de domaine doivent être en mesure de gérer le compte d’ordinateur. Vérifiez que la délégation Kerberos sur le compte d’ordinateur est désactivée et qu’aucun autre compte dans Active Directory ne dispose d’autorisations de délégation sur le compte d’ordinateur `AZUREADSSOACC`. Stockez le compte d’ordinateur dans une unité d’organisation (UO) où il sera protégé contre les suppressions accidentelles et à laquelle seuls des administrateurs de domaine ont accès.

>[!NOTE]
> Si vous utilisez des architectures de type « Pass-The-Hash » et Atténuation des vols d'informations d'identification dans votre environnement local, apportez les changements qui conviennent pour veiller à ce que le compte d'ordinateur `AZUREADSSOACC` ne se retrouve pas dans le conteneur Quarantaine. 

## <a name="step-3-roll-out-the-feature"></a>Étape 3 : Déployer la fonctionnalité

Vous pouvez déployer progressivement l’authentification unique fluide pour vos utilisateurs en suivant les instructions fournies ci-dessous. Vous commencez par ajouter l’URL Azure AD ci-après aux paramètres de la zone Intranet de tous les utilisateurs ou des utilisateurs sélectionnés en utilisant la stratégie de groupe dans Active Directory :

- `https://autologon.microsoftazuread-sso.com`

En outre, vous devez activer un paramètre de stratégie Zone intranet appelé **Autoriser les mises à jour de la barre d’état via le script** via la stratégie de groupe. 

>[!NOTE]
> Les instructions suivantes ne valent que pour les navigateurs Internet Explorer, Microsoft Edge et Google Chrome sur Windows (si ce dernier partage l’ensemble des URL de sites de confiance avec Internet Explorer). Lisez la section suivante pour savoir comment configurer Mozilla Firefox et Google Chrome sur macOS.

### <a name="why-do-you-need-to-modify-users-intranet-zone-settings"></a>Pourquoi devez-vous modifier les paramètres de la zone Intranet des utilisateurs ?

Par défaut, le navigateur calcule automatiquement la zone appropriée, Internet ou Intranet, à partir d’une URL spécifique. Par exemple, l’adresse `http://contoso/` est mappée à la zone Intranet, tandis que l’adresse `http://intranet.contoso.com/` est mappée à la zone Internet (car l’URL contient un point). Les navigateurs n’enverront pas de tickets Kerberos à un point de terminaison cloud, comme l’URL Azure AD, sauf si vous ajoutez explicitement l’URL à la zone intranet du navigateur.

Il y a deux façons de modifier les paramètres de la zone Intranet des utilisateurs :

| Option | Considération de l’administrateur | Expérience utilisateur |
| --- | --- | --- |
| Stratégie de groupe | L’administrateur verrouille la modification des paramètres de la zone Intranet | Les utilisateurs ne peuvent pas modifier leurs propres paramètres |
| Préférences de stratégie de groupe |  L’administrateur autorise la modification sur les paramètres de la zone Intranet | Les utilisateurs peuvent modifier leurs propres paramètres |

### <a name="group-policy-option---detailed-steps"></a>Option « Stratégie de groupe » - Procédure détaillée

1. Ouvrez l’outil Éditeur de gestion des stratégies de groupe.
2. Modifiez la stratégie de groupe qui est appliquée à certains ou à l’ensemble de vos utilisateurs. Cette exemple utilise la **stratégie de domaine par défaut**.
3. Accédez à **Configuration utilisateur** > **Stratégie** > **Modèles d'administration** > **Composants Windows** > **Internet Explorer** > **Panneau de configuration Internet** > **Page Sécurité**. Puis sélectionnez **Liste des attributions de sites aux zones**.
    ![Capture d’écran montrant la « Page Sécurité » avec l’option « Liste des attributions de sites aux zones » sélectionnée.](./media/how-to-connect-sso-quick-start/sso6.png)
4. Activez la stratégie, puis entrez les valeurs suivantes dans la boîte de dialogue :
   - **Nom de la valeur** : URL Azure AD vers laquelle les tickets Kerberos sont transférés.
   - **Valeur** (données) : **1** indique la zone Intranet.

     Le résultat ressemble à ceci :

     Nom de la valeur : `https://autologon.microsoftazuread-sso.com`
  
     Valeur (données) : 1

   >[!NOTE]
   > Si vous souhaitez interdire à certains utilisateurs l’utilisation de l’authentification unique transparente (par exemple, si ces utilisateurs se connectent sur des bornes partagées), définissez les valeurs précédentes sur **4**. Cette opération ajoute l’URL Azure AD à la zone Sites sensibles et rend l’authentification unique transparente inutilisable en permanence.
   >

5. Sélectionnez **OK**, puis de nouveau **OK**.

    ![Capture d’écran montrant la fenêtre « Afficher le contenu » avec une attribution de zone sélectionnée.](./media/how-to-connect-sso-quick-start/sso7.png)

6. Accédez à **Configuration utilisateur** > **Stratégies** > **Modèles d’administration** > **Composants Windows** > **Internet Explorer** > **Panneau de configuration Internet** > **Page Sécurité** > **Zone Intranet**. Puis sélectionnez **Autoriser les mises à jour de la barre d’état via le script**.

    ![Capture d’écran montrant la page « Zone Intranet » avec l’option « Autoriser les mises à jour de la barre d’état via le script » sélectionnée.](./media/how-to-connect-sso-quick-start/sso11.png)

7. Activez le paramètre de stratégie, puis sélectionnez **OK**.

    ![Capture d’écran montrant la fenêtre « Autoriser les mises à jour de la barre d’état via le script » avec le paramètre de stratégie activé.](./media/how-to-connect-sso-quick-start/sso12.png)

### <a name="group-policy-preference-option---detailed-steps"></a>Option « Préférences de stratégie de groupe » - Procédure détaillée

1. Ouvrez l’outil Éditeur de gestion des stratégies de groupe.
2. Modifiez la stratégie de groupe qui est appliquée à certains ou à l’ensemble de vos utilisateurs. Cette exemple utilise la **stratégie de domaine par défaut**.
3. Accédez à **Configuration utilisateur** > **Préférences** > **Paramètres Windows** > **Registre** > **Nouveau** > **Élément de Registre**.

    ![Capture d’écran montrant « Registre » sélectionné et « Élément de Registre » sélectionné.](./media/how-to-connect-sso-quick-start/sso15.png)

4. Entrez les valeurs suivantes dans les champs appropriés et cliquez sur **OK**.
   - **Chemin de la clé** : **_Software\Microsoft\Windows\CurrentVersion\Internet Settings\ZoneMap\Domains\microsoftazuread-sso.com\autologon_* _
   - _*Nom de la valeur** : **_https_*_
   - _*Type de valeur** : **_REG_DWORD_*_
   - _*Données de la valeur** : **_00000001_*_
 
     ![Capture d’écran montrant la fenêtre « Nouvelles propriétés de Registre ».](./media/how-to-connect-sso-quick-start/sso16.png)
 
     ![Authentification unique](./media/how-to-connect-sso-quick-start/sso17.png)

### <a name="browser-considerations"></a>Considérations sur le navigateur

#### <a name="mozilla-firefox-all-platforms"></a>Mozilla Firefox (toutes les plateformes)

Mozilla Firefox n'utilise pas automatiquement l’authentification Kerberos. Chaque utilisateur doit ajouter manuellement l’URL Azure AD à ses paramètres Firefox comme suit :
1. Exécutez Firefox et entrez `about:config` dans la barre d’adresses. Ignorez les notifications que vous voyez.
2. Recherchez la préférence _ *network.negotiate-auth.trusted-uris**. Cette préférence répertorie les sites de confiance de Firefox pour l’authentification Kerberos.
3. Cliquez avec le bouton droit et sélectionnez **Modifier**.
4. Entrez `https://autologon.microsoftazuread-sso.com` dans le champ.
5. Sélectionnez **OK** puis rouvrez le navigateur.

#### <a name="safari-macos"></a>Safari (macOS)

Assurez-vous que la machine utilisant le système macOS est jointe à AD. Les instructions pour joindre à AD votre appareil macOS sortent du cadre de cet article.

#### <a name="microsoft-edge-based-on-chromium-all-platforms"></a>Microsoft Edge basé sur Chromium (toutes les plateformes)

Si vous avez remplacé les paramètres de stratégie [AuthNegotiateDelegateAllowlist](/DeployEdge/microsoft-edge-policies#authnegotiatedelegateallowlist) ou [AuthServerAllowlist](/DeployEdge/microsoft-edge-policies#authserverallowlist) dans votre environnement, veillez à y ajouter également l’URL d’Azure AD (`https://autologon.microsoftazuread-sso.com`).

#### <a name="microsoft-edge-based-on-chromium-macos-and-other-non-windows-platforms"></a>Microsoft Edge basé sur Chromium (macOS et autres plateformes non-Windows)

En ce qui concerne Microsoft Edge basé sur Chromium sur macOS et les autres plateformes non-Windows, consultez la [liste de stratégies du site Microsoft Edge basé sur Chromium](/DeployEdge/microsoft-edge-policies#authserverallowlist) afin d’obtenir plus d’informations sur l’ajout de l’URL Azure AD pour l’authentification intégrée à votre liste verte.

#### <a name="google-chrome-all-platforms"></a>Google Chrome (toutes les plateformes)

Si vous avez remplacé les paramètres de stratégie [AuthNegotiateDelegateWhitelist](https://www.chromium.org/administrators/policy-list-3#AuthNegotiateDelegateWhitelist) ou [AuthServerWhitelist](https://www.chromium.org/administrators/policy-list-3#AuthServerWhitelist) dans votre environnement, veillez à y ajouter également l’URL d’Azure AD (`https://autologon.microsoftazuread-sso.com`).

#### <a name="google-chrome-macos-and-other-non-windows-platforms"></a>Google Chrome (macOS et autres plateformes non-Windows)

En ce qui concerne Google Chrome sur macOS et sur les autres plateformes non-Windows, consultez la [liste de stratégies des projets Chromium](https://dev.chromium.org/administrators/policy-list-3#AuthServerWhitelist) afin d’obtenir plus d’informations sur le mode de contrôle de la liste verte de l’URL Azure AD pour l’authentification intégrée.

L’utilisation des extensions de stratégie de groupe Active Directory tierces afin de déployer l’URL Azure AD pour les utilisateurs de Firefox et de Google Chrome sur Mac n’est pas couverte par cet article.

#### <a name="known-browser-limitations"></a>Limitations connues du navigateur

L’authentification unique transparente ne fonctionne pas en mode de navigation privée sur Firefox et Microsoft Edge. Par ailleurs, il ne fonctionne pas sur Internet Explorer si le navigateur en cours d’utilisation est en mode Protection améliorée. Pour la prochaine version de Microsoft Edge basé sur Chromium, elle ne fonctionnera pas en mode InPrivate et Guest de par sa conception.

## <a name="step-4-test-the-feature"></a>Étape 4 : Tester la fonctionnalité

Pour tester la fonctionnalité d’un utilisateur spécifique, assurez-vous que toutes les conditions suivantes sont en place :
  - L’utilisateur se connecte à un appareil d’entreprise.
  - L'appareil est joint à votre domaine Active Directory. L’appareil _n’a pas_ besoin d’être [joint à Azure AD](../devices/overview.md).
  - L’appareil dispose d’une connexion directe à votre contrôleur de domaine, soit sur le réseau câblé ou sans fil de l’entreprise, soit par le biais d’une connexion d’accès à distance, comme une connexion VPN.
  - Vous avez [déployé la fonctionnalité](#step-3-roll-out-the-feature) pour cet utilisateur via la stratégie de groupe.

Pour tester le scénario dans lequel l’utilisateur entre uniquement le nom d’utilisateur, mais pas le mot de passe :
   - Connectez-vous à `https://myapps.microsoft.com/` dans une nouvelle session de navigateur privée.

Pour tester le scénario dans lequel l’utilisateur n’a pas à entrer le nom d’utilisateur ou le mot de passe, effectuez l'une des étapes suivantes : 
   - Connectez-vous à `https://myapps.microsoft.com/contoso.onmicrosoft.com` dans une nouvelle session de navigateur privée. Remplacez *contoso* par le nom de votre locataire.
   - Connectez-vous à `https://myapps.microsoft.com/contoso.com` dans une nouvelle session de navigateur privée. Remplacez *contoso.com* par un domaine vérifié (pas un domaine fédéré) dans votre locataire.

## <a name="step-5-roll-over-keys"></a>Étape 5 : Substituer les clés

À l’étape 2, Azure AD Connect crée des comptes d’ordinateurs (représentant Azure AD) dans toutes les forêts Azure Directory sur lesquelles vous avez activé l’authentification unique transparente. Pour en savoir plus, consultez [Authentification unique transparente Azure Active Directory : immersion technique](how-to-connect-sso-how-it-works.md).

>[!IMPORTANT]
>Si elle est divulguée, la clé de déchiffrement Kerberos d’un compte d’ordinateur peut être utilisée pour générer des tickets Kerberos à l’attention de n’importe quel utilisateur dans sa forêt Active Directory. Les personnes malveillantes peuvent emprunter l’identité de connexions Azure AD pour des utilisateurs compromis. Nous vous recommandons vivement de renouveler ces clés de déchiffrement Kerberos au moins tous les 30 jours.

Pour obtenir des instructions sur la substitution des clés, voir [Authentification unique transparente Azure Active Directory : questions fréquentes](how-to-connect-sso-faq.md). Nous étudions actuellement la possibilité d’introduire le déploiement de clés automatisé.

>[!IMPORTANT]
>Vous n’avez pas besoin d’effectuer cette étape _immédiatement_ après avoir activé la fonctionnalité. Substituez les clés de déchiffrement Kerberos au moins une fois tous les 30 jours.

## <a name="next-steps"></a>Étapes suivantes

- [Présentation technique approfondie](how-to-connect-sso-how-it-works.md) : Découvrez comment fonctionne la fonctionnalité Authentification unique transparente.
- [Forum Aux Questions](how-to-connect-sso-faq.md) : Obtenez des réponses aux questions fréquentes sur l’authentification unique transparente.
- [Résoudre les problèmes](tshoot-connect-sso.md) : Découvrez comment résoudre les problèmes courants liés à la fonctionnalité Authentification unique transparente.
- [UserVoice](https://feedback.azure.com/forums/169401-azure-active-directory/category/160611-directory-synchronization-aad-connect) : utilisez le Forum Azure Active Directory pour consigner de nouvelles demandes de fonctionnalités.

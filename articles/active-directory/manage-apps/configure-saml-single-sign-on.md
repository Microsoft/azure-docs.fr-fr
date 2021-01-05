---
title: Comprendre l’authentification unique SAML pour les applications dans Azure Active Directory
description: Comprendre l’authentification unique SAML pour les applications dans Azure Active Directory
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: conceptual
ms.workload: identity
ms.date: 07/28/2020
ms.author: kenwith
ms.reviewer: arvinh,luleon
ms.openlocfilehash: ec020ecd4c2bcf6e9186afb3d2c4a79ef235c371
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94658908"
---
# <a name="understand-saml-based-single-sign-on"></a>Comprendre l’authentification unique basée sur SAML

Avec la [série de guides de démarrage rapide](view-applications-portal.md) sur la gestion des applications, vous avez appris à utiliser Azure AD comme fournisseur d’identité (IdP) pour une application. Cet article décrit plus en détail l’option basée sur SAML pour l’authentification unique. 


## <a name="before-you-begin"></a>Avant de commencer

L’utilisation d’Azure AD comme fournisseur d’identité (IdP) et la configuration de l’authentification unique peuvent être simples ou complexes selon l’application utilisée. Certaines applications peuvent être configurées en quelques actions seulement. D’autres nécessitent une configuration approfondie. Pour être efficace rapidement, suivez la [série de guides de démarrage rapide](view-applications-portal.md) sur la gestion des applications. Si l’application que vous ajoutez est simple, vous n’avez probablement pas besoin de lire cet article. Si l’application que vous ajoutez demande une configuration personnalisée pour l’authentification unique SAML, cet article vous concerne.

La [série de guides de démarrage rapide](add-application-portal-setup-sso.md) inclut un article sur la configuration de l’authentification unique. Il vous explique comment accéder à la page de configuration SAML pour une application. La page de configuration SAML comprend cinq sections, qui sont traitées en détail dans cet article.

> [!IMPORTANT] 
> Dans certains cas, l’option **Authentification unique** n’est pas disponible pour une application sous **Applications d’entreprise**. 
>
> Si l’application a été inscrite à l’aide d’**Inscriptions d’applications**, la fonctionnalité d’authentification unique est configurée pour utiliser OIDC OAuth par défaut. Dans ce cas, l’option **Authentification unique** ne s’affiche pas dans le volet de navigation sous **Applications d’entreprise**. Quand vous utilisez **inscriptions d’applications** pour ajouter votre application personnalisée, vous configurez les options dans le fichier du manifeste. Pour en savoir plus sur le fichier manifeste, consultez [Manifeste d’application Azure Active Directory](../develop/reference-app-manifest.md). Pour en savoir plus sur les standards SSO, consultez [Authentification et autorisation avec la plateforme d’identités Microsoft](../develop/authentication-vs-authorization.md#authentication-and-authorization-using-microsoft-identity-platform). 
>
> D’autres scénarios dans lesquels **l’authentification unique** sera absente de la navigation incluent les cas où une application est hébergée dans un autre locataire ou si votre compte ne dispose pas des autorisations requises (administrateur général, administrateur d’application Cloud, administrateur d’application ou propriétaire du principal de service). Les autorisations peuvent également être à l’origine d’un scénario dans lequel vous pouvez ouvrir **l’authentification unique**, mais vous ne pourrez pas l’enregistrer. En savoir plus sur les rôles d’administration d’Azure AD voir (https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-assign-admin-roles).


## <a name="basic-saml-configuration"></a>Configuration SAML de base

Les valeurs doivent vous être communiquées par le fournisseur de l’application. Vous pouvez entrer manuellement les valeurs ou charger un fichier de métadonnées pour extraire la valeur des champs.

> [!TIP]
> De nombreuses applications ont déjà été préconfigurées pour fonctionner avec Azure AD. Ces applications sont listées dans la galerie d’applications que vous pouvez parcourir quand vous ajoutez une application à votre locataire Azure AD. La [série de guides de démarrage rapide](add-application-portal-setup-sso.md) vous guide tout au long du processus. Vous trouverez des instructions détaillées pour les applications de la galerie. Pour accéder aux étapes, vous pouvez cliquer sur le lien de la page de configuration SAML pour l’application comme décrit dans la série de guides de démarrage rapide. Vous pouvez également parcourir une liste complète des tutoriels de configuration d’application en consultant les [tutoriels relatifs à la configuration des applications SaaS](../saas-apps/tutorial-list.md).

| Paramètre de la configuration SAML de base | Initiée par le fournisseur de service | Initiée par le fournisseur d’identité | Description |
|:--|:--|:--|:--|
| **Identificateur (ID d'entité)** | Requis pour certaines applications | Requis pour certaines applications | Identifie l’application de manière unique. Azure AD envoie l’identificateur à l’application en tant que paramètre Audience du jeton SAML. L’application est censée le valider. Cette valeur apparaît également en tant qu’ID d’entité dans les métadonnées SAML fournies par l’application. Entrez une URL dont le modèle est le suivant : « https://<subdomain>.contoso.com ». *Cette valeur correspond à l’élément **Émetteur** dans la demande SAML **AuthnRequest** envoyée par l’application.* |
| **URL de réponse** | Obligatoire | Obligatoire | Spécifie l’adresse à laquelle l’application s’attend à recevoir le jeton SAML. L’URL de réponse est aussi appelée URL ACS (Assertion Consumer Service). Vous pouvez utiliser les champs URL de réponse supplémentaires pour spécifier plusieurs URL de réponse. Par exemple, vous pourriez avoir besoin d'URL de réponse supplémentaires pour plusieurs sous-domaines. Ou, à des fins de test, vous pouvez spécifier simultanément plusieurs URL de réponse (hôte local et URL publiques). |
| **URL d’authentification** | Obligatoire | Ne pas spécifier | Lorsqu’un utilisateur ouvre cette URL, le fournisseur de services redirige vers Azure AD pour authentifier et connecter l’utilisateur. Azure AD utilise l’URL pour démarrer l’application à partir de Microsoft 365 ou de Mes applications Azure AD. Quand elle est vide, Azure AD effectue une authentification initiée par le fournisseur d’identité quand un utilisateur lance l’application à partir de Microsoft 365, de Mes applications Azure AD ou de l’URL d’authentification unique Azure AD.|
| **État de relais** | Facultatif | Facultatif | Indique à l’application où rediriger l’utilisateur une fois l’authentification terminée. En règle générale, la valeur est une URL valide pour l’application. Toutefois, certaines applications utilisent ce champ différemment. Pour plus d’informations, consultez le fournisseur de l’application.
| **URL de déconnexion** | Facultatif | Facultatif | Utilisé pour renvoyer les réponses de déconnexion SAML à l’application.

## <a name="user-attributes-and-claims"></a>Attributs utilisateur et revendications 

Lorsqu’un utilisateur s’authentifie auprès de l’application, Azure AD émet pour l’application un jeton SAML contenant des informations (ou des revendications) sur l’utilisateur qui l’identifient de façon unique. Par défaut, ces informations incluent le nom d’utilisateur, son adresse e-mail, son prénom et son nom. Vous devrez peut-être personnaliser ces revendications si, par exemple, l'application nécessite des valeurs de revendications spécifiques ou un format **Nom** autre que le nom d'utilisateur. 

> [!IMPORTANT]
> De nombreuses applications sont déjà préconfigurées dans la galerie d’applications, et vous n’avez pas à vous soucier de la définition des revendications d’utilisateur et de groupe. La [série de guides de démarrage rapide](add-application-portal.md) vous explique les procédures d’ajout et de configuration d’applications.


La valeur **Identificateur unique de l’utilisateur (ID de nom)** est importante. Il s’agit d’une revendication requise. La valeur par défaut est *user.userprincipalname*. L’identificateur d’utilisateur identifie de façon unique chaque utilisateur au sein de l’application. Par exemple, si l’adresse e-mail est le nom d’utilisateur et l’identificateur unique, définissez la valeur sur *user.mail*.

Pour en savoir plus sur la personnalisation des revendications SAML, consultez [Procédure : personnaliser des revendications émises dans le jeton SAML pour les applications d’entreprise](../develop/active-directory-saml-claims-customization.md).

Vous pouvez ajouter de nouvelles revendications. Pour plus d’informations à ce sujet, consultez [Ajout de revendications spécifiques à l’application](../develop/active-directory-saml-claims-customization.md#adding-application-specific-claims). Pour ajouter des revendications de groupe, consultez [Configurer des revendications de groupe](../hybrid/how-to-connect-fed-group-claims.md).


> [!NOTE]
> Pour découvrir d’autres façons de personnaliser le jeton SAML d’Azure AD pour votre application, consultez les ressources suivantes.
>- Pour créer des rôles personnalisés via le portail Azure, consultez [Configurer des revendications de rôle](../develop/active-directory-enterprise-app-role-management.md).
>- Pour personnaliser les revendications via PowerShell, consultez [Personnaliser des revendications - PowerShell](../develop/active-directory-claims-mapping.md).
>- Pour modifier le manifeste d’application de façon à configurer des revendications facultatives pour votre application, consultez [Configurer des revendications facultatives](../develop/active-directory-optional-claims.md).
>- Pour définir des stratégies de durée de vie des jetons pour les jetons d’actualisation, les jetons d’accès, les jetons de session et les jetons d’ID, consultez [Configurer les durées de vie des jetons](../develop/active-directory-configurable-token-lifetimes.md). Pour restreindre les sessions d’authentification via l’accès conditionnel Azure AD, consultez [Fonctionnalités de gestion des sessions d’authentification](../conditional-access/howto-conditional-access-session-lifetime.md).

## <a name="saml-signing-certificate"></a>Certificat de signature SAML

Azure AD utilise un certificat pour signer les jetons SAML qu’il envoie à l’application. Vous aurez besoin de ce certificat pour configurer l’approbation entre Azure AD et l’application. Pour plus d’informations sur le format du certificat, consultez la documentation SAML de l’application. Pour plus d’informations, consultez [Gérer des certificats pour l’authentification unique fédérée](manage-certificates-for-federated-single-sign-on.md) et [Options avancées de signature de certificats dans le jeton SAML](certificate-signing-options.md).

> [!IMPORTANT]
> De nombreuses applications sont déjà préconfigurées dans la galerie d’applications, et vous n’avez pas à vous soucier des certificats. La [série de guides de démarrage rapide](add-application-portal.md) vous explique les procédures d’ajout et de configuration d’applications.

Depuis Azure AD, vous pouvez télécharger le certificat actif au format Base64 ou Raw directement à partir de la page principale **Configurer l’authentification unique avec SAML**. Vous pouvez également le récupérer en téléchargeant le fichier XML de métadonnées de l’application ou en utilisant l’URL des métadonnées de fédération de l’application. Pour afficher, créer ou télécharger vos certificats (actifs ou inactifs), procédez comme suit.

Voici quelques éléments courants à contrôler pour vérifier un certificat : 
   - *La date d’expiration correcte.* Vous pouvez configurer une date d’expiration allant jusqu’à trois ans dans le futur.
   - *L’état actif du certificat approprié.* Si l’état est **inactif**, définissez-le sur **Actif**. Pour changer l’état, cliquez sur la ligne du certificat avec le bouton droit, puis sélectionnez **Définir comme certificat actif**.
   - *L’option et l’algorithme de signature appropriés.*
   - *Les adresses e-mail de notification correctes.* Lorsque le certificat actif est proche de sa date d’expiration, Azure AD envoie une notification à l’adresse e-mail configurée dans ce champ.

Vous devrez parfois télécharger le certificat. Choisissez l’emplacement d’enregistrement avec prudence. Pour télécharger le certificat, sélectionnez l'une des options suivantes pour le format Base64, le format Raw ou le fichier XML de métadonnées de fédération. Azure AD fournit également l’**URL des métadonnées de fédération de l’application**, où vous pouvez accéder aux métadonnées spécifiques à l’application au format `https://login.microsoftonline.com/<Directory ID>/federationmetadata/2007-06/federationmetadata.xml?appid=<Application ID>`.

> [!NOTE]
> L’application doit être capable de gérer le marqueur d’ordre d’octet présent dans le fichier XML rendu lors de l’utilisation de https://login.microsoftonline.com/{tenant-id}/federationmetadata/2007-06/federationmetadata.xml?appid={app-id}. La marque d’ordre d’octet est représentée sous la forme d’un caractère ASCII non imprimable »¿, et dans Hex, elle est représentée sous la forme EF BB BF lors de la vérification des données XML.

Pour apporter des modifications au certificat, cliquez sur le bouton Modifier. Vous pouvez effectuer plusieurs actions sur la page **Certificat de signature SAML** :
   - Créer un certificat : sélectionnez **Nouveau certificat**, sélectionnez la **Date d’expiration**, puis sélectionnez **Enregistrer**. Pour activer le certificat, sélectionnez le menu contextuel ( **...** ), puis sélectionnez **Définir comme certificat actif**.
   - Charger un certificat avec une clé privée et des informations d’identification PFX : sélectionnez **Importer un certificat** et accédez au certificat. Entrez le **Mot de passe PFX**, puis sélectionnez **Ajouter**.  
   - Configurer la signature de certificat avancée : pour plus d’informations sur ces options, consultez [Options avancées de signature de certificats](certificate-signing-options.md).
   - Avertir d’autres personnes quand le certificat actif est proche de sa date d’expiration : entrez les adresses e-mail dans les champs **Adresses de courrier de notification**.

## <a name="set-up-the-application-to-use-azure-ad"></a>Configurer l’application pour utiliser Azure AD

La section **Configurer\<applicationName>** liste les valeurs qui doivent être configurées dans l’application afin d’utiliser Azure AD comme fournisseur d’identité SAML. Vous définissez les valeurs dans la page de configuration du site web des applications. Par exemple, si vous configurez GitHub, vous devez accéder au site github.com et définir les valeurs. Si l’application est déjà préconfigurée et disponible dans la galerie Azure AD, vous trouverez un lien permettant d’**obtenir des instructions pas à pas**. Sinon, vous devrez trouver la documentation de l’application que vous configurez. 

Les valeurs **URL de connexion** et **URL de déconnexion** correspondent toutes les deux au même point de terminaison, qui est le point de terminaison de gestion des requêtes SAML pour le locataire Azure AD. 

L’**Identificateur Azure AD** est la valeur de l’**Émetteur** dans le jeton SAML émis pour l’application.

## <a name="test-single-sign-on"></a>Tester l’authentification unique

Une fois que vous avez configuré votre application pour utiliser Azure AD en tant que fournisseur d'identité basé sur SAML, vous pouvez tester les paramètres pour vérifier le fonctionnement de la connexion unique avec votre compte. 

Sélectionnez **Test**, puis choisissez d’effectuer un test avec l’utilisateur actuellement connecté ou un autre utilisateur. 

Si l’authentification réussit, vous êtes prêt à affecter des utilisateurs et des groupes à votre application SAML. Félicitations !

Si un message d’erreur apparaît, effectuez les étapes suivantes :

1. Copiez et collez les détails dans la case **À quoi ressemble l’erreur ?** .

    ![Obtenir l’aide de résolution](media/configure-single-sign-on-non-gallery-applications/error-guidance.png)

2. Sélectionnez **Obtenir des conseils de résolution**. Les conseils de résolution et la cause racine apparaissent.  Dans cet exemple, l’utilisateur n’était pas affecté à l’application.

3. Lisez l’aide de résolution et essayez de corriger le problème.

4. Réexécutez le test jusqu'à ce qu’il se termine correctement.

Pour plus d’informations, consultez [Guide pratique pour déboguer l’authentification unique SAML pour les applications dans Azure Active Directory](./debug-saml-sso-issues.md).


## <a name="next-steps"></a>Étapes suivantes

- [Série de guides de démarrage rapide sur la gestion des applications](view-applications-portal.md)
- [Affecter des utilisateurs et des groupes à l’application](./assign-user-or-group-access-portal.md)
- [Configurer le provisionnement automatique d’un compte utilisateur](../app-provisioning/configure-automatic-user-provisioning-portal.md)
- [Protocole SAML d’authentification unique](../develop/single-sign-on-saml-protocol.md)
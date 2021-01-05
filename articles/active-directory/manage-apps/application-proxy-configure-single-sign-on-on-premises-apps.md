---
title: Authentification unique SAML pour les applications locales avec le proxy d’application Azure AD
description: Découvrez comment fournir l’authentification unique pour les applications locales qui sont sécurisées avec l’authentification SAML. Fournissez un accès à distance aux applications locales avec le proxy d’application.
services: active-directory
documentationcenter: ''
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 10/24/2019
ms.author: kenwith
ms.reviewer: japere
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: e7d51aa7e75d7e94d1c2ac66d7edb92a3ef9395b
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94657463"
---
# <a name="saml-single-sign-on-for-on-premises-applications-with-application-proxy"></a>Authentification unique SAML pour les applications locales avec le proxy d’application

Vous pouvez fournir l’authentification unique (SSO) aux applications locales qui sont sécurisées avec l’authentification SAML, et fournir un accès à distance à ces applications via le proxy d’application. Avec l’authentification unique SAML, Azure Active Directory (Azure AD) s’authentifie auprès de l’application en utilisant le compte Azure AD de l’utilisateur. Azure AD communique les informations d’authentification à l’application via un protocole de connexion. Vous pouvez également mapper les utilisateurs à des rôles d’application basés sur les règles, que vous définissez dans vos revendications SAML. Si vous activez le proxy d’application en plus de l’authentification unique SAML, vos utilisateurs bénéficient d’un accès externe à l’application et d’une expérience d’authentification unique très fluide.

Les applications doivent pouvoir consommer des jetons SAML émis par **Azure Active Directory**. Cette configuration ne s’applique pas aux applications qui utilisent un fournisseur d’identité local. Pour ces scénarios, nous vous recommandons de consulter [Ressources pour la migration d’applications vers Azure Active Directory](migration-resources.md).

L’authentification unique SAML avec le proxy d’application fonctionne également avec la fonctionnalité de chiffrement de jetons SAML. Pour plus d’informations, consultez [Configurer le chiffrement des jetons SAML Azure AD](howto-saml-token-encryption.md).

Les diagrammes de protocole ci-dessous décrivent la séquence de l’authentification unique pour un flux initié par le fournisseur de services et un flux d’identité initié par le fournisseur d’identité. Le proxy d’application fonctionne avec l’authentification unique SAML en mettant en cache la demande et la réponse SAML vers et depuis l’application locale.

  ![Diagramme illustrant les interactions entre l’application, le proxy d’application, le client et Azure AD pour l’authentification unique initiée par le fournisseur de services.](./media/application-proxy-configure-single-sign-on-on-premises-apps/saml-sp-initiated-flow.png)

  ![Diagramme illustrant les interactions entre l’application, le proxy d’application, le client et Azure AD pour l’authentification unique initiée par le fournisseur d’identités.](./media/application-proxy-configure-single-sign-on-on-premises-apps/saml-idp-initiated-flow.png)

## <a name="create-an-application-and-set-up-saml-sso"></a>Créer une application et configurer l’authentification unique SAML

1. Dans le portail Azure, sélectionnez **Azure Active Directory > Applications d’entreprise**, puis sélectionnez **Nouvelle application**.

2. Entrez le nom d’affichage de votre nouvelle application, sélectionnez **Intégrer une autre application que vous ne trouvez pas dans la galerie**, puis sélectionnez **Créer**.

3. Dans la page **Vue d’ensemble** de l’application, sélectionnez **Authentification unique**.

4. Sélectionnez **SAML** comme méthode d’authentification unique.

5. Commencez par configurer l’authentification unique SAML pour qu’elle fonctionne sur le réseau d’entreprise. Pour configurer l’authentification SAML pour l’application, consultez la section Configuration SAML de base de l’article [Configurer l’authentification unique SAML](configure-saml-single-sign-on.md).

6. Ajoutez au moins un utilisateur à l’application et vérifiez que le compte de test a accès à l’application. En étant connecté au réseau d’entreprise, utilisez le compte de test pour vérifier si vous avez une authentification unique pour l’application. 

   > [!NOTE]
   > Une fois que vous avez configuré le proxy d’application, vous revenez et vous mettez à jour l’**URL de réponse** de SAML.

## <a name="publish-the-on-premises-application-with-application-proxy"></a>Publier une application locale avec le proxy d’application

Pour fournir l’authentification unique aux applications locales, vous devez activer le proxy d’application et installer un connecteur. Consultez le tutoriel [Ajouter une application locale pour l’accès à distance via le proxy d’application dans Azure AD](application-proxy-add-on-premises-application.md) pour découvrir comment préparer votre environnement local, installer et inscrire un connecteur, et tester le connecteur. Suivez ensuite ces étapes pour publier votre nouvelle application avec le proxy d’application. Pour les autres paramètres non mentionnés ci-dessous, reportez-vous à la section [Ajouter une application locale à Azure AD](application-proxy-add-on-premises-application.md#add-an-on-premises-app-to-azure-ad) du tutoriel.

1. Avec l’application encore ouverte dans le portail Azure, sélectionnez **Proxy d’application**. Spécifiez l’**URL interne** pour l’application. Si vous utilisez un domaine personnalisé, vous devez également charger le certificat TLS/SSL pour votre application. 
   > [!NOTE]
   > Une bonne pratique consiste à utiliser des domaines personnalisés chaque fois que cela est possible, afin d’optimiser l’expérience utilisateur. Pour plus d’informations, consultez [Utilisation des domaines personnalisés dans le proxy d’application Azure AD](application-proxy-configure-custom-domain.md).

2. Sélectionnez **Azure Active Directory** comme méthode de **pré-authentification** pour votre application.

3. Copiez l’**URL externe** de l’application. Vous aurez besoin de cette URL pour terminer la configuration de SAML.

4. En utilisant le compte de test, essayez d’ouvrir l’application avec l’**URL externe** pour vérifier que le proxy d’application est correctement configuré. S’il y a des problèmes, consultez [Résoudre les problèmes de proxy d’application et les messages d’erreur](application-proxy-troubleshoot.md).

## <a name="update-the-saml-configuration"></a>Mettre à jour la configuration SAML

1. Avec l’application encore ouverte dans le portail Azure, sélectionnez **Authentification unique**. 

2. Dans la page **Configurer l’authentification unique avec SAML**, accédez au titre **Configuration SAML de base** et sélectionnez l’icône **Modifier** (un crayon). Vérifiez que l’**URL externe** que vous avez configurée dans le proxy d’application est renseignée dans les champs **Identificateur**, **URL de réponse** et **URL de déconnexion**. Ces URL sont nécessaires au bon fonctionnement du proxy d’application. 

3. Modifiez l’**URL de réponse** configurée précédemment, afin que son domaine soit accessible sur Internet via le proxy d’application. Par exemple, si votre **URL externe** est `https://contosotravel-f128.msappproxy.net` et que l’**URL de réponse** d’origine était `https://contosotravel.com/acs`, vous devez mettre à jour l’**URL de réponse** en `https://contosotravel-f128.msappproxy.net/acs`.

    ![Entrer les données de la configuration SAML de base](./media/application-proxy-configure-single-sign-on-on-premises-apps/basic-saml-configuration.png)


4. Cochez la case en regard de l’**URL de réponse** mise à jour pour la marquer comme valeur par défaut.

   * Après avoir marqué l’**URL de réponse** requise comme URL par défaut, vous pouvez également supprimer l’**URL de réponse** précédemment configurée qui utilisait l’URL interne.

   * Pour un flux initié par le fournisseur de services, vérifiez que l’application back-end spécifie l’**URL de réponse** ou l’URL du service consommateur d’assertion correcte pour recevoir le jeton d’authentification.

    > [!NOTE]
    > Si l’application back-end s’attend à ce que l’**URL de réponse** soit l’URL interne, vous devez utiliser des [domaines personnalisés](application-proxy-configure-custom-domain.md) pour que l’URL interne et l’URL externe correspondent, ou installer l’extension de connexion sécurisée à Mes applications sur les appareils des utilisateurs. Cette extension sera automatiquement redirigée vers le service de proxy d’application qui convient. Pour installer l’extension, consultez [Extension de connexion sécurisée Mes applications](../user-help/my-apps-portal-end-user-access.md#download-and-install-the-my-apps-secure-sign-in-extension).
    
## <a name="test-your-app"></a>Test de l'application

Une fois toutes ces étapes effectuées, votre application doit être opérationnelle. Pour tester l’application :

1. Ouvrez un navigateur et accédez à l’**URL externe** que vous avez créée lors de la publication de l’application. 
1. Connectez-vous au compte de test que vous avez attribué à l’application. Vous devriez pouvoir charger l’application et vous connecter à l’aide de l’authentification unique.

## <a name="next-steps"></a>Étapes suivantes

- [Comment le proxy d’application Azure AD fournit-il une authentification unique ?](./what-is-single-sign-on.md)
- [Résoudre les problèmes du proxy d’application](application-proxy-troubleshoot.md)
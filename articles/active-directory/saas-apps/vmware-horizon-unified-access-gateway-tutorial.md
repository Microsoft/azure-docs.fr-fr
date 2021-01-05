---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à VMware Horizon - Unified Access Gateway | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et VMware Horizon - Unified Access Gateway.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/02/2020
ms.author: jeedes
ms.openlocfilehash: cfdcc39966e58a845be0096474cb8a3366f3dd7a
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92503570"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-vmware-horizon---unified-access-gateway"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à VMware Horizon - Unified Access Gateway

Dans ce tutoriel, vous allez découvrir comment intégrer VMware Horizon - Unified Access Gateway à Azure Active Directory (Azure AD). Quand vous intégrez VMware Horizon - Unified Access Gateway à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à VMware Horizon - Unified Access Gateway.
* Permettre à vos utilisateurs de se connecter automatiquement à VMware Horizon - Unified Access Gateway avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement VMware Horizon - Unified Access Gateway pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* VMware Horizon - Unified Access Gateway prend en charge l’authentification unique lancée par le **fournisseur de services et le fournisseur d’identité**

## <a name="adding-vmware-horizon---unified-access-gateway-from-the-gallery"></a>VMware Horizon - Unified Access Gateway à partir de la galerie

Pour configurer l’intégration de VMware Horizon - Unified Access Gateway à Azure AD, vous devez ajouter VMware Horizon - Unified Access Gateway à partir de la galerie à votre liste d’applications SaaS managées.

1. Connectez-vous au portail Azure avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie** , tapez **VMware Horizon - Unified Access Gateway** dans la zone de recherche.
1. Sélectionnez **VMware Horizon - Unified Access Gateway** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-sso-for-vmware-horizon---unified-access-gateway"></a>Configurer et tester l’authentification unique Azure AD pour VMware Horizon - Unified Access Gateway

Configurez et testez l’authentification unique Azure AD avec VMware Horizon - Unified Access Gateway pour un utilisateur de test appelé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur VMware Horizon - Unified Access Gateway associé.

Pour configurer et tester l’authentification unique Azure AD avec VMware Horizon - Unified Access Gateway, effectuez les étapes suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique VMware Horizon-Unified Access Gateway](#configure-vmware-horizon-unified-access-gateway-sso)** - pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test VMware Horizon - Unified Access Gateway](#create-vmware-horizon-unified-access-gateway-test-user)** - pour avoir un équivalent de B. Simon dans VMware Horizon - Unified Access Gateway lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le portail Azure, accédez à la page d’intégration de l’application **VMware Horizon - Unified Access Gateway** , recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode Initié par le **fournisseur d’identité** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant : `https://<HORIZON_UAG_FQDN>/portal`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<HORIZON_UAG_FQDN>/portal/samlsso`

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<HORIZON_UAG_FQDN>`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels. Contactez l’[équipe de support client VMware Horizon - Unified Access Gateway](mailto:support@vmware.com) pour obtenir ces valeurs. Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **XML de métadonnées de fédération** et sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

1. Dans la section **Configurer VMware Horizon - Unified Access Gateway** , copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon dans le portail Azure.

1. Dans le volet gauche du Portail Azure, sélectionnez **Azure Active Directory** , **Utilisateurs** , puis **Tous les utilisateurs**.
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :
   1. Dans le champ **Nom** , entrez `B.Simon`.  
   1. Dans le champ **Nom de l’utilisateur** , entrez username@companydomain.extension. Par exemple : `B.Simon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ **Mot de passe**.
   1. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à VMware Horizon - Unified Access Gateway.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **VMware Horizon - Unified Access Gateway**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.
1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.
1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez qu’un rôle soit attribué aux utilisateurs, vous pouvez le sélectionner dans la liste déroulante **Sélectionner un rôle** . Si aucun rôle n’a été configuré pour cette application, vous voyez le rôle « Accès par défaut » sélectionné.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer**.

## <a name="configure-vmware-horizon-unified-access-gateway-sso"></a>Configurer l’authentification unique de VMware Horizon-Unified Access Gateway

Pour configurer l’authentification unique côté **VMware Horizon - Unified Access Gateway** , vous devez envoyer le **XML des métadonnées de fédération** téléchargé et les URL appropriées, copiées depuis le portail Azure, à l’ [équipe du support technique de VMware Horizon - Unified Access Gateway](mailto:support@vmware.com). Celles-ci configurent ensuite ce paramètre pour que la connexion SSO SAML soit définie correctement des deux côtés.

### <a name="create-vmware-horizon-unified-access-gateway-test-user"></a>Créer un utilisateur de test VMware Horizon - Unified Access Gateway

Dans cette section, vous allez créer un utilisateur appelé B.Simon dans VMware Horizon - Unified Access Gateway. Collaborez avec l’[équipe de support technique de VMware Horizon - Unified Access Gateway](mailto:support@vmware.com) pour ajouter des utilisateurs dans la plateforme VMware Horizon - Unified Access Gateway. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester votre configuration de l’authentification unique Azure AD avec les options suivantes. 

#### <a name="sp-initiated"></a>Lancée par le fournisseur de services :

* Cliquez sur **Tester cette application** dans le portail Azure. Ceci redirige vers l’URL d’authentification unique de VMware Horizon - Unified Access Gateway, où vous pouvez lancer le flux de connexion.  

* Accédez directement à l’URL de connexion de VMware Horizon - Unified Access Gateway, puis lancez le flux de connexion à partir de là.

#### <a name="idp-initiated"></a>Lancée par le fournisseur d’identité :

* Cliquez sur **Tester cette application** dans le portail Azure : vous devez être connecté automatiquement à l’instance de VMware Horizon - Unified Access Gateway pour laquelle vous configurez l’authentification unique. 

Vous pouvez aussi utiliser le panneau d’accès Microsoft pour tester l’application dans n’importe quel mode. Quand vous cliquez sur la vignette VMware Horizon - Unified Access Gateway dans le panneau d’accès, vous devez normalement être connecté automatiquement à l’application VMware Horizon - Unified Access Gateway pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré VMware Horizon - Unified Access Gateway, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration de données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).
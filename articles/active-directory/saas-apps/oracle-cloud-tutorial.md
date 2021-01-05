---
title: 'Tutoriel : Intégration d’Azure Active Directory à Oracle Cloud Infrastructure Console | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Oracle Cloud Infrastructure Console.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/04/2020
ms.author: jeedes
ms.openlocfilehash: 914858471432979e5516a1f075ed4fcbb8061a5d
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96181847"
---
# <a name="tutorial-integrate-oracle-cloud-infrastructure-console-with-azure-active-directory"></a>Tutoriel : Intégrer Oracle Cloud Infrastructure Console à Azure Active Directory

Dans ce tutoriel, vous allez apprendre à intégrer Oracle Cloud Infrastructure Console à Azure Active Directory (Azure AD). Lorsque vous intégrez Oracle Cloud Infrastructure Console à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Oracle Cloud Infrastructure Console.
* Permettre aux utilisateurs de se connecter automatiquement à Oracle Cloud Infrastructure Console avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Oracle Cloud Infrastructure Console pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Oracle Cloud Infrastructure Console prend en charge l’authentification unique initiée par le **fournisseur de services**.

## <a name="adding-oracle-cloud-infrastructure-console-from-the-gallery"></a>Ajout d’Oracle Cloud Infrastructure Console à partir de la galerie

Pour configurer l’intégration d’Oracle Cloud Infrastructure Console à Azure AD, vous devez ajouter Oracle Cloud Infrastructure Console à partir de la galerie, à votre liste d’applications SaaS managées.

1. Connectez-vous au portail Azure avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **Oracle Cloud Infrastructure Console** dans la zone de recherche.
1. Sélectionnez **Oracle Cloud Infrastructure Console** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-sso"></a>Configurer et tester l’authentification unique Azure AD

Configurez et testez l’authentification unique Azure AD avec Oracle Cloud Infrastructure Console à l’aide d’un utilisateur de test appelé **B. Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Oracle Cloud Infrastructure Console associé.

Pour configurer et tester l’authentification unique Azure AD avec Oracle Cloud Infrastructure Console, effectuez les étapes suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer Oracle Cloud Infrastructure Console](#configure-oracle-cloud-infrastructure-console)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test Oracle Cloud Infrastructure Console](#create-oracle-cloud-infrastructure-console-test-user)** pour obtenir un équivalent de B. Simon dans Oracle Cloud Infrastructure Console lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le portail Azure, accédez à la page d’intégration d’application **Oracle Cloud Infrastructure Console**, recherchez la section **Gérer**, puis sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la boîte de dialogue **Configuration SAML de base**, entrez les valeurs pour les champs suivants :

   > [!NOTE]
   > Vous obtiendrez le fichier de métadonnées de fournisseur de services dans la section **Configurer l’authentification unique Oracle Cloud Infrastructure Console** du tutoriel.
    
   1. Cliquez sur **Charger un fichier de métadonnées**.

   1. Cliquez sur le **logo du dossier** pour sélectionner le fichier de métadonnées, puis cliquez sur **Charger**.

   1. Une fois le fichier de métadonnées chargé, les valeurs **Identificateur** et **URL de réponse** sont automatiquement renseignées dans la zone de texte de la section **Configuration SAML de base**.
    
      > [!NOTE]
      > Si les valeurs **Identificateur** et **URL de réponse** ne sont pas automatiquement renseignées, renseignez-les manuellement en fonction de vos besoins.

      Dans la zone de texte **URL de connexion**, tapez une URL au format suivant : `https://console.<REGIONNAME>.oraclecloud.com/`.

      > [!NOTE]
      > Cette valeur n’est pas la valeur réelle. Mettez à jour la valeur avec l’URL de connexion réelle. Contactez [l’équipe de support client Oracle Cloud Infrastructure Console](https://www.oracle.com/support/advanced-customer-support/products/cloud.html) pour obtenir cette valeur. Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, recherchez **XML de métadonnées de fédération** et sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

   ![Lien Téléchargement de certificat](common/metadataxml.png)

1. L’application Oracle Cloud Infrastructure Console attend les assertions SAML dans un certain format, ce qui oblige à ajouter des mappages d’attributs personnalisés à la configuration des attributs du jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut. Cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue Attributs d’utilisateur.

   ![image1](common/edit-attribute.png)

1. En plus de ce qui précède, l’application Oracle Cloud Infrastructure Console s’attend à ce que quelques attributs supplémentaires soient passés dans la réponse SAML. Dans la section **Attributs et revendications de l’utilisateur** de la boîte de dialogue **Revendications de groupe (préversion)** , effectuez les étapes suivantes :

   1. Cliquez sur **stylet** en regard de **Valeur de l’identificateur de nom**.

   1. Sélectionnez **Persistant** dans **Choisir le format du nom de l’identificateur**.
 
   1. Cliquez sur **Enregistrer**.

      ![image2](./media/oracle-cloud-tutorial/config07.png)
    
      ![image3](./media/oracle-cloud-tutorial/config11.png)

   1. Cliquez sur le **stylo** en regard de **Groupes renvoyés dans la revendication**.

   1. Sélectionnez **Groupes de sécurité** dans la liste d’options.

   1. Sélectionnez **Attribut source** de **ID de groupe**.

   1. Cochez **Personnaliser le nom de la revendication de groupe**.

   1. Dans la zone de texte **Nom**, entrez **groupName**.

   1. Dans la zone de texte **Espace de noms (facultatif)** , entrez `https://auth.oraclecloud.com/saml/claims`.

   1. Cliquez sur **Enregistrer**.

      ![image4](./media/oracle-cloud-tutorial/config08.png)

1. Dans la section **Oracle Cloud Infrastructure Console**, copiez la ou les URL appropriées en fonction de vos besoins.

   ![Copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon dans le portail Azure.

1. Dans le volet gauche du Portail Azure, sélectionnez **Azure Active Directory**, **Utilisateurs**, puis **Tous les utilisateurs**.
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur**, effectuez les étapes suivantes :
   1. Dans le champ **Nom**, entrez `B. Simon`.  
   1. Dans le champ **Nom de l’utilisateur**, entrez username@companydomain.extension. Par exemple : `B. Simon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe**, puis notez la valeur affichée dans le champ **Mot de passe**.
   1. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser B. Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Oracle Cloud Infrastructure Console.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Oracle Cloud Infrastructure Console**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.
1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.
1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez qu’un rôle soit attribué aux utilisateurs, vous pouvez le sélectionner dans la liste déroulante **Sélectionner un rôle** . Si aucun rôle n’a été configuré pour cette application, vous voyez le rôle « Accès par défaut » sélectionné.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

## <a name="configure-oracle-cloud-infrastructure-console"></a>Configurer Oracle Cloud Infrastructure Console

1. Dans une fenêtre différente de navigateur web, connectez-vous à Oracle Cloud Infrastructure Console en tant qu’administrateur.

1. Cliquez sur le côté gauche du menu, sur **Identité**, puis accédez à **Fédération**.

   ![Configuration1](./media/oracle-cloud-tutorial/config01.png)

1. Enregistrez le **fichier de métadonnées du fournisseur de services** en cliquant sur le lien **Télécharger ce document** et chargez-le dans la section **Configuration SAML de base** du portail Azure, puis cliquez sur **Ajouter un fournisseur d’identité**.

   ![Configuration2](./media/oracle-cloud-tutorial/config02.png)

1. Dans la fenêtre contextuelle **Ajouter le fournisseur d’identité**, effectuez les étapes suivantes :

   ![Configuration3](./media/oracle-cloud-tutorial/config03.png)

   1. Dans la zone de texte **NAME**, entrez votre nom.

   1. Dans la zone de texte **DESCRIPTION**, entrez votre description.

   1. Sélectionnez **MICROSOFT ACTIVE DIRECTORY FEDERATION SERVICES (ADFS) ou FOURNISSEUR D’IDENTITÉ SAML 2.0** comme **TYPE**.

   1. Cliquez sur **Parcourir** pour charger le fichier XML de métadonnées de fédération que vous avez téléchargé à partir du portail Azure.

   1. Cliquez sur **Continuer** et dans la section **Modifier le fournisseur d’identité**, effectuez les étapes suivantes :

      ![Configuration4](./media/oracle-cloud-tutorial/configure-09.png)

   1. Le **GROUPE DE FOURNISSEURS D’IDENTITÉ** doit être sélectionné comme ID d’objet de groupe Azure AD. L’ID DE GROUPE doit être le GUID du groupe à partir d’Azure Active Directory. Le groupe doit être mappé au groupe correspondant dans le champ **Groupe OCI**.

   1. Vous pouvez mapper plusieurs groupes selon votre configuration dans le portail Azure et les besoins de votre organisation. Cliquez sur **+ Ajouter un mappage** pour ajouter autant de groupes que vous le souhaitez.

   1. Cliquez sur **Envoyer**.
   
### <a name="create-oracle-cloud-infrastructure-console-test-user"></a>Créer un utilisateur de test Oracle Cloud Infrastructure Console

 Oracle Cloud Infrastructure Console prend en charge le provisionnement juste-à-temps, qui est l’option activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. Aucun utilisateur n’est créé pendant une tentative d’accès et il n’est pas nécessaire non plus de créer l’utilisateur.

### <a name="test-sso"></a>Tester l’authentification unique (SSO)

Lorsque vous sélectionnez la vignette Oracle Cloud Infrastructure Console dans le volet d’accès, vous êtes redirigé vers la page de connexion Oracle Cloud Infrastructure Console. Sélectionnez le **fournisseur d’identité** dans le menu déroulant et cliquez sur **Continuer** comme indiqué ci-dessous pour vous connecter. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

![Configuration](./media/oracle-cloud-tutorial/config10.png)

## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré Oracle Cloud Infrastructure Console, vous pouvez appliquer des contrôles de session qui protègent l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Les contrôles de session sont étendus à partir de l’accès conditionnel. [Découvrir comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad)
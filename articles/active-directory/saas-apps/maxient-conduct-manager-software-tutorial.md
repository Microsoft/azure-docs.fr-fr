---
title: 'Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory avec Maxient Conduct Manager Software | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Maxient Conduct Manager Software.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/18/2019
ms.author: jeedes
ms.openlocfilehash: 4b57b3fbb338774eb8d66fc4d3c0c817e19aff4c
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92458198"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-maxient-conduct-manager-software"></a>Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory avec Maxient Conduct Manager Software

Ce tutoriel explique comment intégrer Maxient Conduct Manager Software avec Azure Active Directory (Azure AD). Lorsque vous intégrez Maxient Conduct Manager Software avec Azure AD, vous pouvez :

* Utiliser Azure AD pour authentifier vos utilisateurs Maxient Conduct Manager Software.
* Autoriser vos utilisateurs à se connecter automatiquement à Maxient Conduct Manager Software avec leur compte Azure AD.


Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Abonnement Maxient Conduct Manager Software pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer Azure AD pour une utilisation avec Maxient Conduct Manager Software.


* Maxient Conduct Manager Software prend en charge l’authentification unique lancée par **le fournisseur de services et le fournisseur d’identité** .

> [!NOTE]
> L’identificateur de cette application étant une valeur de chaîne fixe, une seule instance peut être configurée dans un locataire.

## <a name="adding-maxient-conduct-manager-software-from-the-gallery"></a>Ajout de Maxient Conduct Manager Software à partir de la galerie

Pour configurer l’intégration de Maxient Conduct Manager Software dans Azure AD, vous devez ajouter Maxient Conduct Manager Software à partir de la galerie dans votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Maxient Conduct Manager Software** dans la zone de recherche.
1. Sélectionnez **Maxient Conduct Manager Software** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-single-sign-on-for-maxient-conduct-manager-software"></a>Configurer et tester l’authentification unique Azure AD pour Maxient Conduct Manager Software

Configurer et tester l’authentification unique Azure AD avec Maxient Conduct Manager Software Pour que l’authentification unique fonctionne, vous devez établir une connexion entre Azure AD et Maxient Conduct Manager Software.

Pour configurer et tester l’authentification unique Azure AD avec Maxient Conduct Manager Software, suivez les indications des modules ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs de s’authentifier afin d’utiliser Maxient Conduct Manager Software
    1. **[Affecter tous les utilisateurs pour l’utilisation de Maxient](#assign-all-users-to-be-able-to-authenticate-for-the-maxient-conduct-manager-software)** afin de permettre à tous les membres de votre établissement de s’authentifier.
1. **[Tester la configuration d’Azure AD avec Maxient](#test-with-maxient)** pour vérifier si la configuration fonctionne et si les attributs corrects sont publiés.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [Portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Maxient Conduct Manager Software** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , l’application est préconfigurée en mode Lancement par le **fournisseur d’identité** et les URL nécessaires sont déjà préremplies avec Azure. L’utilisateur doit enregistrer la configuration en cliquant sur le bouton **Enregistrer** .

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://cm.maxient.com/<SCHOOLCODE>`

    > [!NOTE]
    > Cette valeur n’est pas la valeur réelle. Mettez à jour la valeur avec l’URL de connexion réelle. Contactez votre représentant de support/implémentation Maxient pour obtenir la valeur.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur le bouton Copier pour copier l’ **URL des métadonnées de fédération d’application** , puis enregistrez-la sur votre ordinateur.  Vous devez fournir cette URL à votre représentant de support/implémentation Maxient.

    ![Lien Téléchargement de certificat](common/copy-metadataurl.png)

### <a name="assign-all-users-to-be-able-to-authenticate-for-the-maxient-conduct-manager-software"></a>Affecter tous les utilisateurs pour qu’ils puissent s’authentifier auprès de Maxient Conduct Manager Software

Dans cette section, vous allez accorder l’accès pour que tous les comptes s’authentifient à l’aide du système Azure auprès de Maxient Conduct Manager Software.  Il est important de noter que cette étape est **OBLIGATOIRE** pour que Maxient fonctionne correctement.  Maxient utilise votre système Azure AD pour *authentifier* les utilisateurs. L’ *autorisation* des utilisateurs est effectué dans le système Maxient pour la fonction particulière qu’ils essaient d’exécuter. Maxient n’utilise pas les attributs de votre annuaire pour prendre ces décisions.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Maxient Conduct Manager Software** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez tous les utilisateurs (ou les groupes appropriés) et **affectez** -les pour qu’ils puissent s’authentifier auprès de Maxient.

## <a name="test-with-maxient"></a>Tester avec Maxient 

Si vous n’avez pas encore ouvert de ticket de support auprès d’un représentant du support technique/implémentation de Maxient, envoyez un e-mail à [support@maxient.com](mailto:support@maxient.com) avec l’objet « Authentification basée sur le campus/configuration Azure - \<\<School Name\>\> ». Dans le corps de l’e-mail, indiquez l’ **URL des métadonnées de fédération d’application** . Le personnel Maxient répondra avec un lien de test pour vérifier que les attributs appropriés sont publiés.  
    
## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Maxient Conduct Manager Software avec Azure AD](https://aad.portal.azure.com/)
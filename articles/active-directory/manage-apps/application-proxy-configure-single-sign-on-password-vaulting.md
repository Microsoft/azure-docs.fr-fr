---
title: Authentification unique auprès des applications avec le proxy d’application Azure AD | Microsoft Docs
description: Activez l’authentification unique pour vos applications locales publiées avec le proxy d’application Azure AD dans le portail Azure.
services: active-directory
documentationcenter: ''
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: how-to
ms.date: 11/12/2018
ms.author: kenwith
ms.reviewer: japere
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 00b25f1b506228d2d80aecae3845467759a4bc4f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88165037"
---
# <a name="password-vaulting-for-single-sign-on-with-application-proxy"></a>Authentification unique avec mise au coffre des mots de passe par le biais du proxy d’application

Le service Proxy d’application Azure Active Directory vous aide à améliorer la productivité en publiant des applications locales afin que les employés travaillant à distance puissent également y accéder. Dans le portail Azure, vous pouvez également configurer l’authentification unique auprès de ces applications. Vos utilisateurs n’ont qu’à s’authentifier auprès d’Azure AD pour accéder à votre application d’entreprise sans se reconnecter.

Le proxy d’application prend en charge plusieurs [modes d’authentification unique](sso-options.md#choosing-a-single-sign-on-method). L’authentification par mot de passe est destinée aux applications qui utilisent une combinaison nom d’utilisateur/mot de passe pour l’authentification. Lorsque vous configurez l’authentification par mot de passe pour votre application, vos utilisateurs ne doivent se connecter à l’application locale qu’une seule fois. Azure Active Directory stocke ensuite les informations de connexion et les fournit automatiquement à l’application lorsque vos utilisateurs y accèdent à distance.

Vous devez avoir déjà publié et testé votre application avec le proxy d’application. Sinon, suivez les étapes de la rubrique [Publier des applications avec le Proxy d’application Azure AD](application-proxy-add-on-premises-application.md) avant de revenir ici.

## <a name="set-up-password-vaulting-for-your-application"></a>Configurer le stockage de mots de passe pour votre application

1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’administrateur.
1. Sélectionnez **Azure Active Directory** > **Applications d’entreprise** > **Toutes les applications**.
1. Dans la liste, sélectionnez l’application pour laquelle vous voulez configurer l’authentification unique.  
1. Sélectionnez **Proxy d’application**. 
1. Choisissez **Passthrough** comme **Type de pré-authentification**, puis sélectionnez **Enregistrer**. Vous pouvez très bien repasser au type **Azure Active Directory** par la suite. 
1. Sélectionnez **Authentification unique**.

   ![Sélectionner l’authentification unique à partir de la page de présentation de l’application](./media/application-proxy-configure-single-sign-on-password-vaulting/select-sso.png)

1. Pour le mode d’authentification unique, choisissez **Authentification par mot de passe**.
1. Pour l’URL de connexion, entrez l’URL de la page où les utilisateurs entrent leur nom d’utilisateur et leur mot de passe pour se connecter à votre application en dehors du réseau d’entreprise. Il peut s’agir de l’URL externe que vous avez créée quand vous avez publié l’application par le biais du proxy d’application.

   ![Choisissez Authentification par mot de passe et entrez votre URL.](./media/application-proxy-configure-single-sign-on-password-vaulting/password-sso.png)

1. Sélectionnez **Enregistrer**.
1. Sélectionnez **Proxy d’application**. 
1. Choisissez **Azure Active Directory** comme **Type de pré-authentification**, puis sélectionnez **Enregistrer**. 
1. Sélectionnez **Users and Groups**.
1. Affectez des utilisateurs à l’application. Pour cela, sélectionnez **Ajouter un utilisateur**. 
1. Si vous souhaitez prédéfinir des informations d’identification pour un utilisateur, cochez la case devant le nom d’utilisateur et sélectionnez **Mettre à jour les informations d’identification**.
1. Sélectionnez **Azure Active Directory** > **Inscriptions des applications** > **Toutes les applications**.
1. Dans la liste, sélectionnez l’application que vous avez configurée avec l’authentification unique par mot de passe.
1. Sélectionnez **Personnalisation**. 
1. Mettez à jour l’**URL de la page d’accueil** avec l’**URL de connexion** à partir de la page d’authentification unique par mot de passe, puis sélectionnez **Enregistrer**.  



<!-- Need to repro?
7. The page should tell you that a sign-in form was successfully detected at the provided URL. If it doesn't, select **Configure [your app name] Password Single Sign-on Settings** and choose **Manually detect sign-in fields**. Follow the instructions to point out where the sign-in credentials go. 
-->

## <a name="test-your-app"></a>Test de l'application

Accédez au portail Mes applications. Connectez-vous avec vos informations d’identification (ou celles d’un compte de test pour lequel vous avez configuré un accès). Une fois que vous êtes connecté, cliquez sur l’icône de l’application. Cela peut déclencher l’installation de l’extension de navigateur de connexion sécurisée à Mes applications. Si votre utilisateur a des informations d’identification prédéfinies, l’authentification auprès de l’application doit être automatique ; sinon, vous devez spécifier le nom d’utilisateur ou le mot de passe pour la première fois. 

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur les autres façons d’implémenter l’[authentification unique](what-is-single-sign-on.md)
- En savoir plus sur les [considérations de sécurité pour l’accès aux applications à distance avec le proxy d’application Azure AD](application-proxy-security.md)

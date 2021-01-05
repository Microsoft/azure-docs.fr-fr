---
title: Déboguer l'authentification unique SAML - Azure Active Directory
description: Déboguez l’authentification unique SAML aux applications dans Azure Active Directory.
services: active-directory
ms.author: kenwith
author: kenwith
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: troubleshooting
ms.workload: identity
ms.date: 02/18/2019
ms.reviewer: luleon, hirsin, paulgarn
ms.openlocfilehash: f8eb00a2a88c6e26dd5361097f17990469caa8f3
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92325927"
---
# <a name="debug-saml-based-single-sign-on-to-applications-in-azure-active-directory"></a>Déboguer l’authentification unique SAML aux applications dans Azure Active Directory

Apprenez à rechercher et corriger les problèmes d'[authentification unique](what-is-single-sign-on.md) pour les applications qui utilisent l'authentification unique SAML dans Azure Active Directory (Azure AD). 

## <a name="before-you-begin"></a>Avant de commencer

Nous vous recommandons d’installer [l’extension de connexion sécurisée à Mes applications](../user-help/my-apps-portal-end-user-troubleshoot.md#im-having-trouble-installing-the-my-apps-secure-sign-in-extension). Cette extension de navigateur facilite la collecte des informations nécessaires sur la demande et la réponse SAML pour résoudre les problèmes liés à l’authentification unique. Au cas où vous ne parviendriez pas à l’installer, cet article explique comment résoudre les problèmes avec et sans l’extension.

Pour télécharger et installer l’extension de connexion sécurisée à Mes applications, suivez l’un des liens ci-dessous.

- [Chrome](https://go.microsoft.com/fwlink/?linkid=866367)
- [Microsoft Edge](https://go.microsoft.com/fwlink/?linkid=845176)
- [Firefox](https://go.microsoft.com/fwlink/?linkid=866366)

## <a name="test-saml-based-single-sign-on"></a>Tester l’authentification unique SAML

Pour tester l’authentification unique SAML entre Azure AD et une application cible :

1. Connectez-vous au [Portail Azure](https://portal.azure.com) en tant qu’administrateur général (ou autre type d’administrateur autorisé à gérer les applications).
1. Sélectionnez **Azure Active Directory** sous l’onglet gauche, puis **Applications d’entreprise** . 
1. Dans la liste des applications d’entreprise, cliquez sur celle sur laquelle vous souhaitez tester l’authentification unique. Ensuite, dans les options sur la gauche, cliquez sur **Authentification unique** .
1. Pour ouvrir l’expérience de test d’authentification unique reposant sur SAML, accédez à **Tester l’authentification unique sur** (étape 5). Si le bouton **Tester** est grisé, commencez par remplir et enregistrer les attributs requis dans la section **Configuration SAML de base** .
1. Dans le panneau **Tester l’authentification unique** , utilisez vos informations d’identification d’entreprise pour vous connecter à l’application cible. Vous pouvez vous connecter sous l’identité de l’utilisateur actuel ou sous celle d’un autre utilisateur. Dans le deuxième cas, une invite vous demande de vous authentifier.

    ![Capture d’écran montrant la page Tester l’authentification unique SAML](./media/debug-saml-sso-issues/test-single-sign-on.png)

Si la connexion est établie, le test a réussi. Dans ce cas, Azure AD émis un jeton de réponse SAML, que l’application a utilisé pour vous connecter.

Si une erreur s’affiche sur la page de connexion d’entreprise ou sur la page de l’application, utilisez l’une des sections suivantes pour la résoudre.

## <a name="resolve-a-sign-in-error-on-your-company-sign-in-page"></a>Résoudre une erreur de connexion sur votre page de connexion d’entreprise

Il peut arriver qu’une erreur semblable à ce qui suit apparaisse sur la page de connexion d’entreprise lors d’une tentative de connexion.

![Exemple illustrant une erreur dans la page de connexion d’entreprise](./media/debug-saml-sso-issues/error.png)

Pour déboguer cette erreur, vous aurez besoin du message d’erreur et de la demande SAML. L’extension de connexion sécurisée à Mes applications rassemble automatiquement ces informations et affiche une aide à la résolution sur Azure AD.

### <a name="to-resolve-the-sign-in-error-with-the-my-apps-secure-sign-in-extension-installed"></a>Pour résoudre l’erreur de connexion si l’extension de connexion sécurisée à Mes applications est installée

1. Lorsqu’une erreur se produit, l’extension vous redirige vers le panneau **Tester l’authentification unique** Azure AD.
1. Sur le panneau **Tester l’authentification unique** , sélectionnez **Télécharger la demande SAML** .
1. Lisez l’aide à la résolution qui s’affiche. Elle dépend de l’erreur et des valeurs de la
1. Un bouton **Corriger** permettant de mettre à jour automatiquement la configuration dans Azure AD pour résoudre le problème s’affiche. Si vous ne voyez pas ce bouton, le problème de connexion n’est pas dû à une configuration incorrecte sur Azure AD.

Si aucune résolution n’est proposée pour l’erreur de connexion, nous vous suggérons d’utiliser la zone de texte des commentaires pour nous en informer.

### <a name="to-resolve-the-error-without-installing-the-my-apps-secure-sign-in-extension"></a>Pour résoudre l’erreur sans installer l’extension de connexion sécurisée à Mes applications

1. Copiez le message d’erreur dans le coin inférieur droit de la page. Il comporte :
    - Un ID de corrélation et un timestamp. Ces valeurs sont importantes lors de la création d’une demande de support auprès de Microsoft, car elles permettent aux ingénieurs d’identifier le problème et de le résoudre avec précision.
    - Une instruction identifiant la cause du problème.
1. Revenez à Azure AD et localisez le panneau **Tester l’authentification unique** .
1. Dans la zone de texte au-dessus de **Obtenir une aide à la résolution** , collez le message d’erreur.
1. Cliquez sur **Obtenir une aide à la résolution** pour afficher les étapes de résolution du problème. Elles sont susceptibles de demander des informations sur la demande ou la réponse SAML. Si vous n’utilisez pas l’extension de connexion sécurisée à Mes applications, il vous faudra peut-être un outil comme [Fiddler](https://www.telerik.com/fiddler) pour récupérer la demande et la réponse SAML.
1. Vérifiez que la destination de la demande SAML correspond à l’URL du service d’authentification unique SAML obtenue auprès d’Azure AD.
1. Vérifiez que l’émetteur dans la demande SAML est identique à l’identificateur configuré pour l’application dans Azure AD. Azure AD utilise l’émetteur pour rechercher une application dans l’annuaire.
1. Vérifiez qu’AssertionConsumerServiceURL correspond à l’emplacement où l’application doit recevoir le jeton SAML d’Azure AD. Vous pouvez configurer cette valeur dans Azure AD, mais ce n’est pas obligatoire si elle fait partie de la demande SAML.


## <a name="resolve-a-sign-in-error-on-the-application-page"></a>Résoudre une erreur de connexion sur la page d’application

Il peut arriver que la connexion réussisse, puis qu’une erreur s’affiche sur la page de l’application. Cela se produit lorsque Azure AD a envoyé un jeton à l’application, mais que celle-ci n’accepte pas la réponse.

Pour résoudre l’erreur, suivez ces étapes ou regardez cette [brève vidéo sur l’utilisation d’Azure AD pour résoudre les problèmes liés à l’authentification unique SAML](https://www.youtube.com/watch?v=poQCJK0WPUk&list=PLLasX02E8BPBm1xNMRdvP6GtA6otQUqp0&index=8) :

1. Si l’application se trouve dans la galerie Azure AD, vérifiez que vous avez suivi toutes les étapes d’intégration de l’application avec Azure AD. Pour trouver les instructions d’intégration relatives à votre application, voir la [liste des tutoriels d’intégration d’applications SaaS](../saas-apps/tutorial-list.md).
1. Récupérez la réponse SAML.
    - Si l’extension de connexion sécurisée à Mes applications est installée, cliquez sur **Télécharger la réponse SAML** dans le panneau **Tester l’authentification unique** .
    - Si l’extension n’est pas installée, utilisez un outil comme [Fiddler](https://www.telerik.com/fiddler) pour récupérer la réponse SAML.
1. Notez ces éléments dans le jeton de la réponse SAML :
   - Identificateur unique de la valeur NameID et du format
   - Revendications émises dans le jeton
   - Certificat utilisé pour signer le jeton.

     Pour plus d’informations sur la réponse SAML, voir [Protocole SAML d’authentification unique](../develop/single-sign-on-saml-protocol.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json).

1. Maintenant que vous avez vérifié la réponse SAML, consultez l’aide à la résolution du problème dans [Erreur sur la page d’une application après connexion](application-sign-in-problem-application-error.md). 
1. Si vous ne parvenez toujours pas à vous connecter, vous pouvez demander au fournisseur de l’application ce qui manque dans la réponse SAML.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que l’authentification unique fonctionne sur votre application, vous pouvez [Automatiser l’attribution d’utilisateurs et la suppression de privilèges d’accès pour les applications SaaS](../app-provisioning/user-provisioning.md) ou [Bien démarrer avec l’accès conditionnel](../conditional-access/app-based-conditional-access.md).

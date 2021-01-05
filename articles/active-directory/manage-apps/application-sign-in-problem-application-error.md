---
title: Un message d’erreur apparaît sur la page de l’application après la connexion | Microsoft Docs
description: Comment résoudre les problèmes de connexion à Azure AD quand l’application renvoie un message d’erreur.
services: active-directory
documentationcenter: ''
author: kenwith
manager: celestedg
ms.assetid: ''
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 07/11/2017
ms.author: kenwith
ms.reviewer: asteen
ms.collection: M365-identity-device-management
ms.openlocfilehash: 779286a43f8b20ce9a9a528e14eaa930763d82b4
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94651581"
---
# <a name="an-app-page-shows-an-error-message-after-the-user-signs-in"></a>Une page d’application affiche un message d’erreur une fois que l’utilisateur est connecté

Dans ce scénario, Azure Active Directory (Azure AD) connecte l’utilisateur. Toutefois, l’application affiche un message d’erreur et ne permet pas à l’utilisateur de terminer le flux de connexion. Le problème est que l’application n’a pas accepté la réponse émise par Azure AD.

De nombreuses raisons peuvent expliquer pourquoi l’application n’a pas accepté la réponse d’Azure AD. Si le message d’erreur n’identifie pas clairement ce qui manque dans la réponse, essayez ce qui suit :

-   Si l’application en question est la galerie Azure AD, vérifiez que vous avez suivi les étapes de l’article [Comment déboguer une authentification unique SAML pour des applications dans Azure AD](./debug-saml-sso-issues.md).

-   Utilisez un outil tel que [Fiddler](https://www.telerik.com/fiddler) pour capturer la requête, la réponse et le jeton SAML.

-   Envoyez la réponse SAML à l’éditeur de l’application et demandez-lui ce qui manque.

## <a name="attributes-are-missing-from-the-saml-response"></a>Attributs manquants dans la réponse SAML

Pour ajouter un attribut dans la configuration Azure AD qui sera envoyée dans la réponse Azure AD, procédez comme suit :

1. Ouvrez le [**portail Azure**](https://portal.azure.com/) et connectez-vous en tant qu’administrateur général ou coadministrateur.

2. En haut du volet de navigation situé à gauche, sélectionnez **Tous les services** pour ouvrir l’extension Azure AD.

3. Saisissez **Azure Active Directory** dans la zone de recherche de filtre, puis sélectionnez **Azure Active Directory**.

4. Sélectionnez **Applications d’entreprise** dans le volet de navigation Azure AD.

5. Sélectionnez **Toutes les applications** pour afficher la liste complète de vos applications.

   > [!NOTE]
   > Si vous ne voyez pas l’application souhaitée, utilisez le contrôle **Filtrer** en haut de la liste **Toutes les applications**. Définissez l’option **Afficher** sur « Toutes les applications ».

6. Sélectionnez l’application pour laquelle vous souhaitez configurer l’authentification unique.

7. Après le chargement de l’application, sélectionnez **Authentification unique** dans le volet de navigation à gauche.

8. Dans la section **Attributs utilisateur**, sélectionnez **Afficher et modifier tous les autres attributs utilisateur**. Ici, vous pouvez modifier les attributs à envoyer à l’application dans le jeton SAML lorsque les utilisateurs se connectent.

   Pour ajouter un attribut :

   1. Sélectionnez **Ajouter un attribut**. Entrez le **nom**, puis sélectionnez la **valeur** dans la liste déroulante.

   1.  Sélectionnez **Enregistrer**. Le nouvel attribut s’affiche dans le tableau.

9. Enregistrez la configuration.

   La prochaine fois que l’utilisateur se connectera à l’application, Azure AD enverra le nouvel attribut dans la réponse SAML.

## <a name="the-app-doesnt-identify-the-user"></a>L’application n’identifie pas l’utilisateur

La connexion à l’application échoue, car il manque un attribut (par exemple, un rôle) dans la réponse SAML. Elle peut également échouer, car l’application attend un format différent ou une autre valeur pour l’attribut **NameID** (identificateur d’utilisateur).

Si vous utilisez l’[approvisionnement automatique des utilisateurs d’Azure AD](../app-provisioning/user-provisioning.md) pour créer, gérer et supprimer des utilisateurs dans l’application, vérifiez que l’utilisateur a été approvisionné dans l’application SaaS. Pour en savoir plus, voir [Aucun utilisateur n’est approvisionné dans une application de la galerie Azure AD](../app-provisioning/application-provisioning-config-problem-no-users-provisioned.md).

## <a name="add-an-attribute-to-the-azure-ad-app-configuration"></a>Ajouter un attribut dans la configuration de l’application Azure AD

Pour modifier la valeur de l’identificateur d’utilisateur, procédez comme suit :

1. Ouvrez le [**portail Azure**](https://portal.azure.com/) et connectez-vous en tant qu’administrateur général ou coadministrateur.

2. En haut du volet de navigation situé à gauche, sélectionnez **Tous les services** pour ouvrir l’extension Azure AD.

3. Saisissez **Azure Active Directory** dans la zone de recherche de filtre, puis sélectionnez **Azure Active Directory**.

4. Sélectionnez **Applications d’entreprise** dans le volet de navigation Azure AD.

5. Sélectionnez **Toutes les applications** pour afficher la liste complète de vos applications.

   > [!NOTE]
   > Si vous ne voyez pas l’application souhaitée, utilisez le contrôle **Filtrer** en haut de la liste **Toutes les applications**. Définissez l’option **Afficher** sur « Toutes les applications ».

6. Sélectionnez l’application que vous voulez configurer pour l’authentification unique.

7. Après le chargement de l’application, sélectionnez **Authentification unique** dans le volet de navigation à gauche.

8. Sous **Attributs utilisateur**, sélectionnez l’identificateur unique de l’utilisateur dans la liste déroulante **Identificateur de l’utilisateur**.

## <a name="change-the-nameid-format"></a>Modifier le format NameID

Si l’application attend un autre format pour l’attribut **NameID** (identificateur d’utilisateur), consultez la procédure de [modification de nameID](../develop/active-directory-saml-claims-customization.md#editing-nameid) pour modifier le format NameID.

Azure AD sélectionne le format de l’attribut **NameID** (identificateur d’utilisateur) en fonction de la valeur choisie ou du format demandé par l’application dans la requête d’authentification SAML. Pour plus d’informations, consultez la section « NameIDPolicy » du [protocole SAML d’authentification unique](../develop/single-sign-on-saml-protocol.md#nameidpolicy).

## <a name="the-app-expects-a-different-signature-method-for-the-saml-response"></a>L’application attend une méthode de signature différente pour la réponse SAML

Pour modifier les parties du jeton SAML qui sont signées numériquement par Azure AD, procédez comme suit :

1. Ouvrez le [portail Azure](https://portal.azure.com/) et connectez-vous en tant qu’Administrateur général ou que Coadministrateur.

2. En haut du volet de navigation situé à gauche, sélectionnez **Tous les services** pour ouvrir l’extension Azure AD.

3. Saisissez **Azure Active Directory** dans la zone de recherche de filtre, puis sélectionnez **Azure Active Directory**.

4. Sélectionnez **Applications d’entreprise** dans le volet de navigation Azure AD.

5. Sélectionnez **Toutes les applications** pour afficher la liste complète de vos applications.

   > [!NOTE]
   > Si vous ne voyez pas l’application souhaitée, utilisez le contrôle **Filtrer** en haut de la liste **Toutes les applications**. Définissez l’option **Afficher** sur « Toutes les applications ».

6. Sélectionnez l’application pour laquelle vous souhaitez configurer l’authentification unique.

7. Après le chargement de l’application, sélectionnez **Authentification unique** dans le volet de navigation à gauche.

8. Sous **Certificat de signature SAML**, sélectionnez **Afficher les paramètres avancés de signature de certificat**.

9. Sélectionnez l’**option de signature** que l’application attend parmi ces options :

   * **Signer la réponse SAML**
   * **Signer la réponse et l’assertion SAML**
   * **Signer l’assertion SAML**

   La prochaine fois que l’utilisateur se connectera à l’application, Azure AD signera la partie de la réponse SAML sélectionnée.

## <a name="the-app-expects-the-sha-1-signing-algorithm"></a>L’application attend l’algorithme de signature SHA-1

Par défaut, Azure AD signe le jeton SAML à l’aide de l’algorithme le plus sécurisé. Nous vous recommandons de ne pas remplacer l’algorithme de signature par *SHA-1* si l’application ne nécessite pas SHA-1.

Pour modifier l’algorithme de signature, procédez comme suit :

1. Ouvrez le [portail Azure](https://portal.azure.com/) et connectez-vous en tant qu’Administrateur général ou que Coadministrateur.

2. En haut du volet de navigation situé à gauche, sélectionnez **Tous les services** pour ouvrir l’extension Azure AD.

3. Saisissez **Azure Active Directory** dans la zone de recherche de filtre, puis sélectionnez **Azure Active Directory**.

4. Sélectionnez **Applications d’entreprise** dans le volet de navigation Azure AD.

5. Sélectionnez **Toutes les applications** pour afficher la liste de vos applications.

   > [!NOTE]
   > Si vous ne voyez pas l’application souhaitée, utilisez le contrôle **Filtrer** en haut de la liste **Toutes les applications**. Définissez l’option **Afficher** sur « Toutes les applications ».

6. Sélectionnez l’application pour laquelle vous souhaitez configurer l’authentification unique.

7. Après le chargement de l’application, sélectionnez **Authentification unique** dans le volet de navigation à gauche de l’application.

8. Sous **Certificat de signature SAML**, sélectionnez **Afficher les paramètres avancés de signature de certificat**.

9. Sélectionnez **SHA-1** comme **Algorithme de signature**.

   La prochaine fois que l’utilisateur se connectera à l’application, Azure AD signera le jeton SAML à l’aide de l’algorithme SHA-1.

## <a name="next-steps"></a>Étapes suivantes
[Comment déboguer une authentification unique SAML pour des applications dans Azure AD](./debug-saml-sso-issues.md).
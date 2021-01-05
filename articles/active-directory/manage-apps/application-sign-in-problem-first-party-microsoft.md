---
title: Problèmes de connexion à une application Microsoft | Microsoft Docs
description: Résoudre les problèmes courants liés à la connexion aux applications internes Microsoft (comme Microsoft 365) à l’aide d’Azure AD.
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
ms.date: 09/10/2018
ms.author: kenwith
ms.reviewer: asteen
ms.collection: M365-identity-device-management
ms.openlocfilehash: 57deed9d7fb178ba1cdc8d6e954d751752532de4
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94654403"
---
# <a name="problems-signing-in-to-a-microsoft-application"></a>Problèmes de connexion à une application Microsoft

Les applications Microsoft (comme Exchange, SharePoint, Yammer, etc.) sont affectées et gérées un peu différemment des applications SaaS tierces et des autres applications que vous intégrez à Azure AD pour l’authentification unique.

Il existe trois méthodes principales pour se connecter à une application publiée par Microsoft.

-   Pour les applications faisant partie de Microsoft 365 ou d’autres suites payantes, les utilisateurs peuvent obtenir un accès via **l’attribution de licence** directement dans leur compte d’utilisateur ou via un groupe à l’aide de la fonctionnalité d’attribution de licence de groupe.

-   Pour les applications publiées et fournies gratuitement par Microsoft ou par un tiers, les utilisateurs peuvent obtenir un accès via le **consentement de l’utilisateur**. Cela signifie qu’ils peuvent se connecter à l’application avec leur compte professionnel ou scolaire Azure AD et lui permettre d’accéder à certaines données de leur compte.

-   Pour les applications publiées et fournies gratuitement par Microsoft ou par un tiers, les utilisateurs peuvent également obtenir un accès via le **consentement de l’administrateur**. Cela veut dire que l’administrateur a déterminé que l’application peut être utilisée par tous les membres de l’organisation. Dans ce cas, il se connecte à l’application avec un compte d’administrateur général et accorde l’accès à tous les membres de l’organisation.

Pour résoudre votre problème, lisez [Problèmes d'ordre général à prendre en considération concernant l'accès aux applications](#general-problem-areas-with-application-access-to-consider), puis Procédure pas à pas : Étapes de résolution des problèmes d'accès aux applications Microsoft afin d'obtenir plus de détails.

## <a name="general-problem-areas-with-application-access-to-consider"></a>Problèmes d’ordre général à prendre en considération concernant l’accès aux applications

La liste ci-après répertorie les problèmes d’ordre général que vous pouvez étudier si vous savez par où commencer. Toutefois, nous vous conseillons de lire ce guide de démarrage rapide : Procédure pas à pas : Étapes de résolution des problèmes d'accès aux applications Microsoft.

-   [Problèmes avec le compte de l’utilisateur](#problems-with-the-users-account)

-   [Problèmes liés aux groupes](#problems-with-groups)

-   [Problèmes liés aux stratégies d’accès conditionnel](#problems-with-conditional-access-policies)

-   [Problèmes liés au consentement d’application](#problems-with-application-consent)

## <a name="steps-to-troubleshoot-microsoft-application-access"></a>Étapes de résolution des problèmes d’accès aux applications Microsoft

La liste ci-après répertorie les problèmes couramment rencontrés lorsque les utilisateurs ne peuvent pas se connecter à une application Microsoft.

- Problèmes d’ordre général à vérifier en premier

  * Vérifiez que l’utilisateur se connecte à la **bonne URL** et non à une URL d’application locale.

  * Vérifiez que le compte d’utilisateur **n’est pas verrouillé**.

  * Vérifiez que le **compte de l’utilisateur existe bien** dans Azure Active Directory. [Vérifier l’existence d’un compte d’utilisateur dans Azure Active Directory](#problems-with-the-users-account)

  * Vérifiez que les connexions sont **activées** dans le compte de l’utilisateur. [Vérifier l’état du compte d’un utilisateur](#problems-with-the-users-account)

  * Vérifiez que le **mot de passe de l’utilisateur n’a pas expiré ou qu’il n’a pas été oublié**. [Réinitialiser le mot de passe d’un utilisateur](#reset-a-users-password) ou [Activer la réinitialisation du mot de passe libre-service](../authentication/tutorial-enable-sspr.md)

  * Vérifiez que **Multi-Factor Authentication** ne bloque pas l’accès utilisateur. [Vérifier l’état Multi-Factor Authentication d’un utilisateur](#check-a-users-multi-factor-authentication-status) ou [Vérifier les informations de contact de l’authentification d’un utilisateur](#check-a-users-authentication-contact-info)

  * Vérifiez qu’une **stratégie d’accès conditionnel** ou **stratégie de protection d’identité** ne bloque pas l’accès utilisateur. [Vérifier une stratégie d’accès conditionnel](#problems-with-conditional-access-policies) ou [Vérifier la stratégie d’accès conditionnel d’une application spécifique](#check-a-specific-applications-conditional-access-policy) ou [Désactiver une stratégie d’accès conditionnel](#disable-a-specific-conditional-access-policy)

  * Vérifiez que les **informations de contact d’authentification** de l’utilisateur sont à jour et permettent d’appliquer les stratégies d’accès conditionnel ou de Multi-Factor Authentication. [Vérifier l’état Multi-Factor Authentication d’un utilisateur](#check-a-users-multi-factor-authentication-status) ou [Vérifier les informations de contact de l’authentification d’un utilisateur](#check-a-users-authentication-contact-info)

- Pour les applications **Microsoft** **qui nécessitent une licence** (comme Office 365), voici certains problèmes à vérifier une fois que vous avez écarté les problèmes d’ordre général ci-dessus :

  * Vérifiez que l’utilisateur **dispose d’une licence**. [Vérifier les licences affectées à un utilisateur](#check-a-users-assigned-licenses) ou [Vérifier les licences affectées à un groupe](#check-a-groups-assigned-licenses)

  * Si la licence est **affectée à un** **groupe statique**, vérifiez que **l’utilisateur est membre** de ce groupe. [Vérifier les appartenances d’un utilisateur à des groupes](#check-a-users-group-memberships)

  * Si la licence est **affectée à un** **groupe dynamique**, vérifiez que la **règle de groupe dynamique est correctement définie**. [Vérifier les critères d’appartenance d’un groupe dynamique](#check-a-dynamic-groups-membership-criteria)

  * Si la licence est **affectée à un** **groupe dynamique**, vérifiez que le groupe dynamique **a terminé de traiter** son appartenance et que l’**utilisateur est membre du groupe** (cela peut prendre un certain temps). [Vérifier les appartenances d’un utilisateur à des groupes](#check-a-users-group-memberships)

  *  Après avoir vérifié que la licence est affectée, vérifiez que la licence **n’a pas expiré**.

  *  Vérifiez que la licence **correspond à l’application** à laquelle il souhaite accéder.

- Pour les **applications** Microsoft **qui ne nécessitent pas de licence**, voici quelques autres points à vérifier :

  * Si l’application demande des **autorisations de niveau utilisateur** (par exemple « Accéder à la boîte aux lettres de cet utilisateur »), vérifiez que l’utilisateur s’est connecté à l’application et qu’il a effectué une **opération de consentement de niveau utilisateur** pour permettre à l’application d’accéder à ses données.

  * Si l’application demande des **autorisations de niveau administrateur** (par exemple « Accéder à la boîte aux lettres de tous les utilisateurs »), vérifiez qu’un administrateur général a effectué une **opération de consentement de niveau administrateur pour le compte de tous les utilisateurs** au sein de l’organisation.

## <a name="problems-with-the-users-account"></a>Problèmes avec le compte de l’utilisateur

L’accès à l’application peut être bloqué en raison d’un problème avec un utilisateur auquel a été accordé un accès à l’application. La liste ci-après répertorie quelques méthodes pouvant vous aider à résoudre les problèmes liés aux utilisateurs et à leurs paramètres de compte :

-   [Vérifier l’existence d’un compte d’utilisateur dans Azure Active Directory](#check-if-a-user-account-exists-in-azure-active-directory)

-   [Vérifier l’état du compte d’un utilisateur](#check-a-users-account-status)

-   [Réinitialiser le mot de passe d’un utilisateur](#reset-a-users-password)

-   [Activer la réinitialisation du mot de passe libre-service](#enable-self-service-password-reset)

-   [Vérifier l’état Multi-Factor Authentication d’un utilisateur](#check-a-users-multi-factor-authentication-status)

-   [Vérifier les informations de contact de l’authentification d’un utilisateur](#check-a-users-authentication-contact-info)

-   [Vérifier les appartenances d’un utilisateur à des groupes](#check-a-users-group-memberships)

-   [Vérifier les licences affectées à un utilisateur](#check-a-users-assigned-licenses)

-   [Affecter une licence à un utilisateur](#assign-a-user-a-license)

### <a name="check-if-a-user-account-exists-in-azure-active-directory"></a>Vérifier l’existence d’un compte d’utilisateur dans Azure Active Directory

Pour vérifier si le compte d’un utilisateur est présent, procédez comme suit :

1.  Ouvrez le [**portail Azure**](https://portal.azure.com/) et connectez-vous en tant **qu’Administrateur général**.

2.  Ouvrez **l’extension Azure Active Directory** en cliquant sur **Tous les services** en haut du menu de navigation principal de gauche.

3.  Tapez « **Azure Active Directory** » dans la zone de recherche de filtre et sélectionnez l’élément **Azure Active Directory**.

4.  Cliquez sur **Utilisateurs et groupes** dans le menu de navigation.

5.  Cliquez sur **Tous les utilisateurs**.

6.  **Recherchez** l’utilisateur qui vous intéresse et **cliquez sur la ligne** pour le sélectionner.

7.  Vérifiez les propriétés de l’objet utilisateur pour vous assurer qu’elles apparaissent comme prévu et qu’aucune donnée n’est manquante.

### <a name="check-a-users-account-status"></a>Vérifier l’état du compte d’un utilisateur

Pour vérifier l’état du compte d’un utilisateur, procédez comme suit :

1.  Ouvrez le [**portail Azure**](https://portal.azure.com/) et connectez-vous en tant **qu’Administrateur général**.

2.  Ouvrez **l’extension Azure Active Directory** en cliquant sur **Tous les services** en haut du menu de navigation principal de gauche.

3.  Tapez « **Azure Active Directory** » dans la zone de recherche de filtre et sélectionnez l’élément **Azure Active Directory**.

4.  Cliquez sur **Utilisateurs et groupes** dans le menu de navigation.

5.  Cliquez sur **Tous les utilisateurs**.

6.  **Recherchez** l’utilisateur qui vous intéresse et **cliquez sur la ligne** pour le sélectionner.

7.  Cliquez sur **Profil**.

8.  Sous **Paramètres**, assurez-vous que **Bloquer la connexion** est défini sur **Non**.

### <a name="reset-a-users-password"></a>Réinitialiser le mot de passe d’un utilisateur

Pour réinitialiser le mot de passe d’un utilisateur, procédez comme suit :

1.  Ouvrez le [**portail Azure**](https://portal.azure.com/) et connectez-vous en tant **qu’Administrateur général**.

2.  Ouvrez **l’extension Azure Active Directory** en cliquant sur **Tous les services** en haut du menu de navigation principal de gauche.

3.  Tapez « **Azure Active Directory** » dans la zone de recherche de filtre et sélectionnez l’élément **Azure Active Directory**.

4.  Cliquez sur **Utilisateurs et groupes** dans le menu de navigation.

5.  Cliquez sur **Tous les utilisateurs**.

6.  **Recherchez** l’utilisateur qui vous intéresse et **cliquez sur la ligne** pour le sélectionner.

7.  Cliquez sur le bouton **Réinitialiser le mot de passe** en haut du volet Utilisateur.

8.  Cliquez sur le bouton **Réinitialiser le mot de passe** dans le volet **Réinitialiser le mot de passe** qui s’affiche.

9.  Copiez le **mot de passe temporaire** ou **entrez un nouveau mot de passe** pour l’utilisateur.

10. Communiquez ce nouveau mot de passe à l’utilisateur. Ce dernier devra changer ce mot de passe lors de sa prochaine connexion à Azure Active Directory.

### <a name="enable-self-service-password-reset"></a>Activer la réinitialisation du mot de passe libre-service

Pour activer la réinitialisation du mot de passe libre-service, suivez les étapes de déploiement ci-dessous :

-   [Permettre aux utilisateurs de réinitialiser leur mot de passe Azure Active Directory](../authentication/tutorial-enable-sspr.md)

-   [Permettre aux utilisateurs de réinitialiser ou de modifier leur mot de passe Active Directory Azure local](../authentication/tutorial-enable-sspr.md)

### <a name="check-a-users-multi-factor-authentication-status"></a>Vérifier l’état Multi-Factor Authentication d’un utilisateur

Pour vérifier l’état d’authentification multifacteur d’un utilisateur, procédez comme suit :

1. Ouvrez le [**portail Azure**](https://portal.azure.com/) et connectez-vous en tant **qu’Administrateur général**.

2. Ouvrez **l’extension Azure Active Directory** en cliquant sur **Tous les services** en haut du menu de navigation principal de gauche.

3. Tapez « **Azure Active Directory** » dans la zone de recherche de filtre et sélectionnez l’élément **Azure Active Directory**.

4. Cliquez sur **Utilisateurs et groupes** dans le menu de navigation.

5. Cliquez sur **Tous les utilisateurs**.

6. Cliquez sur le bouton **Multi-Factor Authentication** en haut du volet.

7. Une fois le **portail d’administration Multi-Factor Authentication** chargé, vérifiez que vous vous trouvez sur l’onglet **Utilisateurs**.

8. Recherchez l’utilisateur dans la liste des utilisateurs au moyen de la recherche, du filtrage ou du tri.

9. Sélectionnez l’utilisateur dans la liste des utilisateurs et **activez**, **désactivez** ou **appliquez** la Multi-Factor Authentication comme souhaité.

   * **Remarque** : Si un utilisateur est dans un état **Appliqué**, vous pouvez définir le définir sur **Désactivé** de façon temporaire pour lui permettre de revenir à son compte. Une fois qu’il est revenu dans son compte, vous pouvez ensuite modifier son état en le redéfinissant sur **Activé** pour lui demander d’enregistrer à nouveau ses informations de contact à sa prochaine connexion. Sinon, vous pouvez suivre les étapes décrites dans [Vérifier les informations de contact de l’authentification d’un utilisateur](#check-a-users-authentication-contact-info) pour vérifier ou définir ces données à sa place.

### <a name="check-a-users-authentication-contact-info"></a>Vérifier les informations de contact de l’authentification d’un utilisateur

Pour vérifier les informations de contact de l’authentification d’un utilisateur utilisées pour Authentification multifacteur, Accès conditionnel, Identity Protection et Réinitialisation du mot de passe, procédez comme suit :

1.  Ouvrez le [**portail Azure**](https://portal.azure.com/) et connectez-vous en tant **qu’Administrateur général**.

2.  Ouvrez **l’extension Azure Active Directory** en cliquant sur **Tous les services** en haut du menu de navigation principal de gauche.

3.  Tapez « **Azure Active Directory** » dans la zone de recherche de filtre et sélectionnez l’élément **Azure Active Directory**.

4.  Cliquez sur **Utilisateurs et groupes** dans le menu de navigation.

5.  Cliquez sur **Tous les utilisateurs**.

6.  **Recherchez** l’utilisateur qui vous intéresse et **cliquez sur la ligne** pour le sélectionner.

7.  Cliquez sur **Profil**.

8.  Faites défiler la page jusqu’aux **informations de contact d’authentification**.

9.  **Passez en revue** les données enregistrées pour l’utilisateur et mettez-les à jour si besoin.

### <a name="check-a-users-group-memberships"></a>Vérifier les appartenances d’un utilisateur à des groupes

Pour vérifier l’appartenance d’un utilisateur à des groupes, procédez comme suit :

1.  Ouvrez le [**portail Azure**](https://portal.azure.com/) et connectez-vous en tant **qu’Administrateur général**.

2.  Ouvrez **l’extension Azure Active Directory** en cliquant sur **Tous les services** en haut du menu de navigation principal de gauche.

3.  Tapez « **Azure Active Directory** » dans la zone de recherche de filtre et sélectionnez l’élément **Azure Active Directory**.

4.  Cliquez sur **Utilisateurs et groupes** dans le menu de navigation.

5.  Cliquez sur **Tous les utilisateurs**.

6.  **Recherchez** l’utilisateur qui vous intéresse et **cliquez sur la ligne** pour le sélectionner.

7.  Cliquez sur **Groupes** pour afficher les groupes dont l’utilisateur est membre.

### <a name="check-a-users-assigned-licenses"></a>Vérifier les licences affectées à un utilisateur

Pour vérifier les licences affectées à un utilisateur, procédez comme suit :

1.  Ouvrez le [**portail Azure**](https://portal.azure.com/) et connectez-vous en tant **qu’Administrateur général**.

2.  Ouvrez **l’extension Azure Active Directory** en cliquant sur **Tous les services** en haut du menu de navigation principal de gauche.

3.  Tapez « **Azure Active Directory** » dans la zone de recherche de filtre et sélectionnez l’élément **Azure Active Directory**.

4.  Cliquez sur **Utilisateurs et groupes** dans le menu de navigation.

5.  Cliquez sur **Tous les utilisateurs**.

6.  **Recherchez** l’utilisateur qui vous intéresse et **cliquez sur la ligne** pour le sélectionner.

7.  Cliquez sur **Licences** pour voir quelles licences sont actuellement affectées à l’utilisateur.

### <a name="assign-a-user-a-license"></a>Affecter une licence à un utilisateur 

Pour affecter une licence à un utilisateur, procédez comme suit :

1.  Ouvrez le [**portail Azure**](https://portal.azure.com/) et connectez-vous en tant **qu’Administrateur général**.

2.  Ouvrez **l’extension Azure Active Directory** en cliquant sur **Tous les services** en haut du menu de navigation principal de gauche.

3.  Tapez « **Azure Active Directory** » dans la zone de recherche de filtre et sélectionnez l’élément **Azure Active Directory**.

4.  Cliquez sur **Utilisateurs et groupes** dans le menu de navigation.

5.  Cliquez sur **Tous les utilisateurs**.

6.  **Recherchez** l’utilisateur qui vous intéresse et **cliquez sur la ligne** pour le sélectionner.

7.  Cliquez sur **Licences** pour voir quelles licences sont actuellement affectées à l’utilisateur.

8.  Cliquez sur le bouton **Attribuer**.

9.  Sélectionnez **un ou plusieurs produits** dans la liste des produits disponibles.

10. **Facultatif** Cliquez sur l’élément **Options d’affectation** pour affecter les produits de façon granulaire. Cliquez sur **OK** lorsque l’opération est terminée.

11. Cliquez sur le bouton **Attribuer** pour affecter ces licences à cet utilisateur.

## <a name="problems-with-groups"></a>Problèmes liés aux groupes

L’accès à l’application peut être bloqué en raison d’un problème avec un groupe auquel a été accordé un accès à l’application. La liste ci-après répertorie quelques méthodes pouvant vous aider à résoudre les problèmes liés aux groupes et à l’appartenance :

-   [Vérifier l’appartenance à un groupe](#check-a-groups-membership)

-   [Vérifier les critères d’appartenance d’un groupe dynamique](#check-a-dynamic-groups-membership-criteria)

-   [Vérifier les licences affectées à un groupe](#check-a-groups-assigned-licenses)

-   [Retraiter les licences d’un groupe](#reprocess-a-groups-licenses)

-   [Affecter une licence à un groupe](#assign-a-group-a-license)

### <a name="check-a-groups-membership"></a>Vérifier l’appartenance à un groupe

Pour vérifier l’appartenance d’un utilisateur à un groupe, procédez comme suit :

1.  Ouvrez le [**portail Azure**](https://portal.azure.com/) et connectez-vous en tant **qu’Administrateur général**.

2.  Ouvrez **l’extension Azure Active Directory** en cliquant sur **Tous les services** en haut du menu de navigation principal de gauche.

3.  Tapez « **Azure Active Directory** » dans la zone de recherche de filtre et sélectionnez l’élément **Azure Active Directory**.

4.  Cliquez sur **Utilisateurs et groupes** dans le menu de navigation.

5.  Cliquez sur **Tous les groupes**.

6.  **Recherchez** le groupe qui vous intéresse et **cliquez sur la ligne** pour le sélectionner.

7.  Cliquez sur **Membres** pour consulter la liste des utilisateurs affectés à ce groupe.

### <a name="check-a-dynamic-groups-membership-criteria"></a>Vérifier les critères d’appartenance d’un groupe dynamique 

Pour vérifier les critères d’appartenance à un groupe dynamique, procédez comme suit :

1.  Ouvrez le [**portail Azure**](https://portal.azure.com/) et connectez-vous en tant **qu’Administrateur général**.

2.  Ouvrez **l’extension Azure Active Directory** en cliquant sur **Tous les services** en haut du menu de navigation principal de gauche.

3.  Tapez « **Azure Active Directory** » dans la zone de recherche de filtre et sélectionnez l’élément **Azure Active Directory**.

4.  Cliquez sur **Utilisateurs et groupes** dans le menu de navigation.

5.  Cliquez sur **Tous les groupes**.

6.  **Recherchez** le groupe qui vous intéresse et **cliquez sur la ligne** pour le sélectionner.

7.  Cliquez sur **Règles d’appartenance dynamique**.

8.  Examinez la règle **simple** ou **avancée** définie pour ce groupe et vérifiez que l’utilisateur que vous souhaitez ajouter à ce groupe répond à ces critères.

### <a name="check-a-groups-assigned-licenses"></a>Vérifier les licences affectées à un groupe

Pour vérifier les licences affectées à un groupe, procédez comme suit :

1.  Ouvrez le [**portail Azure**](https://portal.azure.com/) et connectez-vous en tant **qu’Administrateur général**.

2.  Ouvrez **l’extension Azure Active Directory** en cliquant sur **Tous les services** en haut du menu de navigation principal de gauche.

3.  Tapez « **Azure Active Directory** » dans la zone de recherche de filtre et sélectionnez l’élément **Azure Active Directory**.

4.  Cliquez sur **Utilisateurs et groupes** dans le menu de navigation.

5.  Cliquez sur **Tous les groupes**.

6.  **Recherchez** le groupe qui vous intéresse et **cliquez sur la ligne** pour le sélectionner.

7.  Cliquez sur **Licences** pour voir quelles licences sont actuellement affectées au groupe.

### <a name="reprocess-a-groups-licenses"></a>Retraiter les licences d’un groupe

Pour retraiter les licences affectées à un groupe, procédez comme suit :

1. Ouvrez le [**portail Azure**](https://portal.azure.com/) et connectez-vous en tant **qu’Administrateur général**.

2. Ouvrez **l’extension Azure Active Directory** en cliquant sur **Tous les services** en haut du menu de navigation principal de gauche.

3. Tapez « **Azure Active Directory** » dans la zone de recherche de filtre et sélectionnez l’élément **Azure Active Directory**.

4. Cliquez sur **Utilisateurs et groupes** dans le menu de navigation.

5. Cliquez sur **Tous les groupes**.

6. **Recherchez** le groupe qui vous intéresse et **cliquez sur la ligne** pour le sélectionner.

7. Cliquez sur **Licences** pour voir quelles licences sont actuellement affectées au groupe.

8. Cliquez sur le bouton **Retraiter** pour vous assurer que les licences attribuées aux membres de ce groupe sont à jour. Cette opération peut être très longue, selon la taille et la complexité du groupe.

   >[!NOTE]
   >Pour accélérer ce processus, vous pouvez affecter temporairement une licence directement à l’utilisateur. [Affecter une licence à un utilisateur](#problems-with-application-consent)
   >
   >

### <a name="assign-a-group-a-license"></a>Affecter une licence à un groupe

Pour affecter une licence à un groupe, procédez comme suit :

1. Ouvrez le [**portail Azure**](https://portal.azure.com/) et connectez-vous en tant **qu’Administrateur général**.

2. Ouvrez **l’extension Azure Active Directory** en cliquant sur **Tous les services** en haut du menu de navigation principal de gauche.

3. Tapez « **Azure Active Directory** » dans la zone de recherche de filtre et sélectionnez l’élément **Azure Active Directory**.

4. Cliquez sur **Utilisateurs et groupes** dans le menu de navigation.

5. Cliquez sur **Tous les groupes**.

6. **Recherchez** le groupe qui vous intéresse et **cliquez sur la ligne** pour le sélectionner.

7. Cliquez sur **Licences** pour voir quelles licences sont actuellement affectées au groupe.

8. Cliquez sur le bouton **Attribuer**.

9. Sélectionnez **un ou plusieurs produits** dans la liste des produits disponibles.

10. **Facultatif** Cliquez sur l’élément **Options d’affectation** pour affecter les produits de façon granulaire. Cliquez sur **OK** lorsque l’opération est terminée.

11. Cliquez sur le bouton **Attribuer** pour affecter ces licences à ce groupe. Cette opération peut être très longue, selon la taille et la complexité du groupe.

    >[!NOTE]
    >Pour accélérer ce processus, vous pouvez affecter temporairement une licence directement à l’utilisateur. [Affecter une licence à un utilisateur](#problems-with-application-consent)
    > 
    >

## <a name="problems-with-conditional-access-policies"></a>Problèmes liés aux stratégies d’accès conditionnel

### <a name="check-a-specific-conditional-access-policy"></a>Vérifier une stratégie d’accès conditionnel

Pour vérifier ou valider une stratégie d’accès conditionnel :

1. Ouvrez le [**portail Azure**](https://portal.azure.com/) et connectez-vous en tant **qu’Administrateur général**.

2. Ouvrez **l’extension Azure Active Directory** en cliquant sur **Tous les services** en haut du menu de navigation principal de gauche.

3. Tapez « **Azure Active Directory** » dans la zone de recherche de filtre et sélectionnez l’élément **Azure Active Directory**.

4. Cliquez sur **Applications d’entreprise** dans le menu de navigation.

5. Cliquez sur l’élément de navigation **Accès conditionnel**.

6. Cliquez sur la stratégie que vous souhaitez examiner.

7. Vérifiez la présence de conditions, d’affectations ou d’autres paramètres pouvant bloquer l’accès de l’utilisateur.

   >[!NOTE]
   >Vous pouvez temporairement désactiver cette stratégie pour vérifier qu’elle n’affecte pas les connexions. Pour ce faire, sélectionnez la valeur **Non** de l’option **Activer une stratégie**, puis cliquez sur le bouton **Enregistrer**.
   >
   >

### <a name="check-a-specific-applications-conditional-access-policy"></a>Vérifier une stratégie d’accès conditionnel d’une application

Pour vérifier ou valider une stratégie d’accès conditionnel configurée dans une application :

1.  Ouvrez le [**portail Azure**](https://portal.azure.com/) et connectez-vous en tant **qu’Administrateur général**.

2.  Ouvrez **l’extension Azure Active Directory** en cliquant sur **Tous les services** en haut du menu de navigation principal de gauche.

3.  Tapez « **Azure Active Directory** » dans la zone de recherche de filtre et sélectionnez l’élément **Azure Active Directory**.

4.  Cliquez sur **Applications d’entreprise** dans le menu de navigation.

5.  Cliquez sur **Toutes les applications**.

6.  Recherchez l’application qui vous intéresse ou à laquelle l’utilisateur tente de se connecter en fournissant son nom d’affichage ou son ID.

     >[!NOTE]
     >Si vous ne voyez pas l’application que vous recherchez, cliquez sur le bouton **Filtrer**, puis développez l’étendue de la liste sur **Toutes les applications**. Si vous souhaitez afficher plus de colonnes, cliquez sur le bouton **Colonnes** pour ajouter des détails à vos applications.
     >
     >

7.  Cliquez sur l’élément de navigation **Accès conditionnel**.

8.  Cliquez sur la stratégie que vous souhaitez examiner.

9.  Vérifiez la présence de conditions, d’affectations ou d’autres paramètres pouvant bloquer l’accès de l’utilisateur.

     >[!NOTE]
     >Vous pouvez temporairement désactiver cette stratégie pour vérifier qu’elle n’affecte pas les connexions. Pour ce faire, sélectionnez la valeur **Non** de l’option **Activer une stratégie**, puis cliquez sur le bouton **Enregistrer**.
     >
     >

### <a name="disable-a-specific-conditional-access-policy"></a>Désactiver une stratégie d’accès conditionnel

Pour vérifier ou valider une stratégie d’accès conditionnel :

1.  Ouvrez le [**portail Azure**](https://portal.azure.com/) et connectez-vous en tant **qu’Administrateur général**.

2.  Ouvrez **l’extension Azure Active Directory** en cliquant sur **Tous les services** en haut du menu de navigation principal de gauche.

3.  Tapez « **Azure Active Directory** » dans la zone de recherche de filtre et sélectionnez l’élément **Azure Active Directory**.

4.  Cliquez sur **Applications d’entreprise** dans le menu de navigation.

5.  Cliquez sur l’élément de navigation **Accès conditionnel**.

6.  Cliquez sur la stratégie que vous souhaitez examiner.

7.  Désactivez la stratégie en sélectionnant la valeur **Non** de l’option **Activer une stratégie**, puis cliquez sur le bouton **Enregistrer**.

## <a name="problems-with-application-consent"></a>Problèmes liés au consentement d’application

L’accès à l’application peut être bloqué, car l’opération de consentement avec les autorisations adéquates n’a pas eu lieu. La liste ci-après répertorie quelques méthodes pouvant vous aider à résoudre les problèmes de consentement d’application :

-   [Effectuer une opération de consentement de niveau utilisateur](#perform-a-user-level-consent-operation)

-   [Effectuer une opération de consentement de niveau administrateur pour n’importe quelle application](#perform-administrator-level-consent-operation-for-any-application)

-   [Effectuer une opération de consentement de niveau administrateur pour une application à locataire unique](#perform-administrator-level-consent-for-a-single-tenant-application)

-   [Effectuer une opération de consentement de niveau administrateur pour une application multilocataire](#perform-administrator-level-consent-for-a-multi-tenant-application)

### <a name="perform-a-user-level-consent-operation"></a>Effectuer une opération de consentement de niveau utilisateur

-   Pour toute application Open ID Connect qui demande des autorisations, la navigation vers l’écran de connexion de l’application permet d’exécuter un consentement au niveau de l’utilisateur de l’application pour l’utilisateur connecté.

-   Si vous souhaitez procéder par programmation, consultez [Demande de consentement d’utilisateur individuel](../develop/v2-permissions-and-consent.md#requesting-individual-user-consent).

### <a name="perform-administrator-level-consent-operation-for-any-application"></a>Effectuer une opération de consentement de niveau administrateur pour n’importe quelle application

-   Pour les **applications développées à l’aide du modèle d’application V1 uniquement**, vous pouvez forcer ce consentement de niveau administrateur en ajoutant « **?prompt=admin\_consent** » à la fin de l’URL de connexion de l’application.

-   Pour **toutes les applications développées à l’aide du modèle d’application V2**, vous pouvez appliquer ce consentement de niveau administrateur en suivant les instructions situées dans la section **Demander les autorisations d’un administrateur Active Directory** de la rubrique [Utilisation du point de terminaison de consentement administrateur](../develop/v2-permissions-and-consent.md#using-the-admin-consent-endpoint).

### <a name="perform-administrator-level-consent-for-a-single-tenant-application"></a>Effectuer une opération de consentement de niveau administrateur pour une application à locataire unique

-   Pour les **applications à locataire unique** qui demandent des autorisations (telles que celles que vous développez ou celles dont vous êtes propriétaire au sein de votre organisation), vous pouvez effectuer une opération de **consentement de niveau administrateur** pour le compte de tous les utilisateurs en vous connectant en tant qu’administrateur général, puis en cliquant sur le bouton **Accorder des autorisations** en haut du volet **Registre d’application -&gt; Toutes les applications -&gt; Sélectionner une application -&gt; Autorisations requises**.

-   Pour **toutes les applications développées à l’aide du modèle d’application V1 ou V2**, vous pouvez appliquer ce consentement de niveau administrateur en suivant les instructions situées dans la section **Demander les autorisations d’un administrateur Active Directory** de la rubrique [Utilisation du point de terminaison de consentement administrateur](../develop/v2-permissions-and-consent.md#using-the-admin-consent-endpoint).

### <a name="perform-administrator-level-consent-for-a-multi-tenant-application"></a>Effectuer une opération de consentement de niveau administrateur pour une application multilocataire

-   Pour les **applications multilocataires** qui demandent des autorisations (comme les applications développées par un tiers ou par Microsoft), vous pouvez effectuer une opération de **consentement de niveau administrateur**. Connectez-vous en tant qu’administrateur général, puis cliquez sur le bouton **Accorder des autorisations** sous le volet **Applications d’entreprise -&gt; Toutes les applications -&gt; Sélectionner une application -&gt; Autorisations** (prochainement disponible).

-   Vous pouvez également appliquer ce consentement de niveau administrateur en suivant les instructions situées dans la section **Demander les autorisations d’un administrateur Active Directory** de la rubrique [Utilisation du point de terminaison de consentement administrateur](../develop/v2-permissions-and-consent.md#using-the-admin-consent-endpoint).

## <a name="next-steps"></a>Étapes suivantes
[Utilisation du point de terminaison de consentement administrateur](../develop/v2-permissions-and-consent.md#using-the-admin-consent-endpoint)
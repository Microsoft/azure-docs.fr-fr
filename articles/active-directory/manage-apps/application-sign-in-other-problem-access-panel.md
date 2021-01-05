---
title: Résoudre les problèmes de connexion à une application à partir d’Azure AD Mes applications
description: Résoudre les problèmes de connexion à une application à partir d’Azure AD Mes applications
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: troubleshooting
ms.date: 07/11/2017
ms.author: kenwith
ms.reviewer: japere
ms.custom: contperf-fy21q2
ms.openlocfilehash: 59d733bfe5580e64d531eeac1db443982a308517
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033627"
---
# <a name="troubleshoot-problems-signing-in-to-an-application-from-azure-ad-my-apps"></a>Résoudre les problèmes de connexion à une application à partir d’Azure AD Mes applications

Mes applications est un portail web qui permet à un utilisateur disposant d’un compte professionnel ou scolaire dans Azure Active Directory (Azure AD) de visualiser et de démarrer des applications informatiques auxquelles l’administrateur Azure AD lui a donné accès. Pour accéder à Mes applications, ouvrez un navigateur et accédez à l’adresse [https://myapps.microsoft.com](https://myapps.microsoft.com).

Pour plus d’informations sur l’utilisation d’Azure AD en tant que fournisseur d’identité pour une application, consultez [Qu’est-ce que la gestion des applications dans Azure AD](what-is-application-management.md). Pour être à jour rapidement, suivez la [série de démarrages rapides sur la gestion des applications](view-applications-portal.md).

Ces applications sont configurées pour le compte de l’utilisateur dans le portail Azure AD. Pour que l’application soit visible dans Mes applications, elle doit être correctement configurée et attribuée à l’utilisateur ou à un groupe dont est membre l’utilisateur. 

Les types d’applications que peut voir l’utilisateur tombent dans les catégories suivantes :
-   Applications Microsoft 365
-   Applications Microsoft et tierces configurées avec l’authentification unique (SSO) basée sur la fédération
-   Applications à authentification unique (SSO) par mot de passe
-   Applications avec solutions d’authentification unique (SSO) existantes

Voici quelques éléments à vérifier si une application s’affiche ou non.
- Assurez-vous que l’application est ajoutée à Azure AD et que l’utilisateur est affecté. Pour en savoir plus, consultez la [série de démarrages rapides sur la gestion des applications](add-application-portal.md).
- Si une application a été récemment ajoutée, demandez à l’utilisateur de se déconnecter, puis de se reconnecter. 
- Si l’application nécessite une licence, comme Office, assurez-vous que la licence appropriée est attribuée à l’utilisateur.
- Le temps nécessaire aux changements de licence peut varier en fonction de la taille et de la complexité du groupe.

## <a name="general-issues-to-check-first"></a>Problèmes d’ordre général à vérifier en premier

-   Assurez-vous que le navigateur web respecte la configuration requise, voir [Navigateurs pris en charge par Mes applications](../user-help/my-apps-portal-end-user-access.md).
-   Vérifiez que l’URL de l’application figure dans la liste des **sites de confiance** du navigateur de l’utilisateur.
-   Vérifiez que l’application est **configurée** correctement.
-   Vérifiez que les connexions sont **activées** dans le compte de l’utilisateur.
-   Vérifiez que le compte d’utilisateur **n’est pas verrouillé**.
-   Vérifiez que le **mot de passe de l’utilisateur n’a pas expiré ou qu’il n’a pas été oublié**.
-   Vérifiez que **Multi-Factor Authentication** ne bloque pas l’accès utilisateur.
-   Vérifiez qu’une **stratégie d’accès conditionnel** ou **stratégie de protection d’identité** ne bloque pas l’accès utilisateur.
-   Vérifiez que les **informations de contact d’authentification** de l’utilisateur sont à jour et permettent d’appliquer les stratégies d’accès conditionnel ou de Multi-Factor Authentication.
-   Essayez également d’effacer les cookies de votre navigateur, puis réessayez de vous connecter.

## <a name="problems-with-the-users-account"></a>Problèmes avec le compte de l’utilisateur
L’accès à Mes applications peut être bloqué en raison d’un problème avec le compte de l’utilisateur. La liste ci-après répertorie quelques méthodes pouvant vous aider à résoudre les problèmes liés aux utilisateurs et à leurs paramètres de compte :
-   [Vérifier l’existence d’un compte d’utilisateur dans Azure Active Directory](#check-if-a-user-account-exists-in-azure-active-directory)
-   [Vérifier l’état du compte d’un utilisateur](#check-a-users-account-status)
-   [Réinitialiser le mot de passe d’un utilisateur](#reset-a-users-password)
-   [Activer la réinitialisation du mot de passe libre-service](#enable-self-service-password-reset)
-   [Vérifier l’état Multi-Factor Authentication d’un utilisateur](#check-a-users-multi-factor-authentication-status)
-   [Vérifier les informations de contact de l’authentification d’un utilisateur](#check-a-users-authentication-contact-info)
-   [Vérifier les appartenances d’un utilisateur à des groupes](#check-a-users-group-memberships)
-   [Vérifier si un utilisateur a plus de 999 attributions de rôle d’application](#check-if-a-user-has-more-than-999-app-role-assignments)
-   [Vérifier les licences affectées à un utilisateur](#check-a-users-assigned-licenses)
-   [Affecter une licence à un utilisateur](#assign-a-user-a-license)

### <a name="check-if-a-user-account-exists-in-azure-active-directory"></a>Vérifier l’existence d’un compte d’utilisateur dans Azure Active Directory
Pour vérifier si le compte d’un utilisateur est présent, procédez comme suit :
1.  Ouvrez le [**portail Azure**](https://portal.azure.com/) et connectez-vous en tant **qu’Administrateur général**.
2.  Ouvrez l’**extension Azure Active Directory** en sélectionnant **Tous les services** en haut du menu de navigation principal de gauche.
3.  Tapez « **Azure Active Directory** » dans la zone de recherche de filtre et sélectionnez l’élément **Azure Active Directory**.
4.  Sélectionnez **Utilisateurs et groupes** dans le menu de navigation.
5.  Sélectionnez **Tous les utilisateurs**.
6.  **Recherchez** l’utilisateur qui vous intéresse et **sélectionnez la ligne** correspondante.
7.  Vérifiez les propriétés de l’objet utilisateur pour vous assurer qu’elles apparaissent comme prévu et qu’aucune donnée n’est manquante.

### <a name="check-a-users-account-status"></a>Vérifier l’état du compte d’un utilisateur
Pour vérifier l’état du compte d’un utilisateur, procédez comme suit :
1.  Ouvrez le [**portail Azure**](https://portal.azure.com/) et connectez-vous en tant **qu’Administrateur général**.
2.  Ouvrez l’**extension Azure Active Directory** en sélectionnant **Tous les services** en haut du menu de navigation principal de gauche.
3.  Tapez « **Azure Active Directory** » dans la zone de recherche de filtre et sélectionnez l’élément **Azure Active Directory**.
4.  Sélectionnez **Utilisateurs et groupes** dans le menu de navigation.
5.  Sélectionnez **Tous les utilisateurs**.
6.  **Recherchez** l’utilisateur qui vous intéresse et **sélectionnez la ligne** correspondante.
7.  Sélectionnez **Profil**.
8.  Sous **Paramètres**, assurez-vous que **Bloquer la connexion** est défini sur **Non**.

### <a name="reset-a-users-password"></a>Réinitialiser le mot de passe d’un utilisateur
Pour réinitialiser le mot de passe d’un utilisateur, procédez comme suit :
1.  Ouvrez le [**portail Azure**](https://portal.azure.com/) et connectez-vous en tant **qu’Administrateur général**.
2.  Ouvrez l’**extension Azure Active Directory** en sélectionnant **Tous les services** en haut du menu de navigation principal de gauche.
3.  Tapez « **Azure Active Directory** » dans la zone de recherche de filtre et sélectionnez l’élément **Azure Active Directory**.
4.  Sélectionnez **Utilisateurs et groupes** dans le menu de navigation.
5.  Sélectionnez **Tous les utilisateurs**.
6.  **Recherchez** l’utilisateur qui vous intéresse et **sélectionnez la ligne** correspondante.
7.  Sélectionnez le bouton **Réinitialiser le mot de passe** en haut du volet Utilisateur.
8.  Sélectionnez le bouton **Réinitialiser le mot de passe** dans le volet **Réinitialiser le mot de passe** qui s’affiche.
9.  Copiez le **mot de passe temporaire** ou **entrez un nouveau mot de passe** pour l’utilisateur.
10. Communiquez ce nouveau mot de passe à l’utilisateur. Ce dernier devra changer ce mot de passe lors de sa prochaine connexion à Azure Active Directory.

### <a name="enable-self-service-password-reset"></a>Activer la réinitialisation du mot de passe libre-service
Pour activer la réinitialisation du mot de passe en libre-service, procédez comme suit :
-   [Permettre aux utilisateurs de réinitialiser leur mot de passe Azure Active Directory](../authentication/tutorial-enable-sspr.md)
-   [Permettre aux utilisateurs de réinitialiser ou de modifier leur mot de passe Active Directory Azure local](../authentication/tutorial-enable-sspr.md)

### <a name="check-a-users-multi-factor-authentication-status"></a>Vérifier l’état Multi-Factor Authentication d’un utilisateur
Pour vérifier l’état d’authentification multifacteur d’un utilisateur, procédez comme suit :
1. Ouvrez le [**portail Azure**](https://portal.azure.com/) et connectez-vous en tant **qu’Administrateur général**.
2. Ouvrez l’**extension Azure Active Directory** en sélectionnant **Tous les services** en haut du menu de navigation principal de gauche.
3. Tapez « **Azure Active Directory** » dans la zone de recherche de filtre et sélectionnez l’élément **Azure Active Directory**.
4. Sélectionnez **Utilisateurs et groupes** dans le menu de navigation.
5. Sélectionnez **Tous les utilisateurs**.
6. Sélectionnez le bouton **Multi-Factor Authentication** en haut du volet.
7. Une fois le **portail d’administration Multi-Factor Authentication** chargé, assurez-vous de vous trouver sur l’onglet **Utilisateurs**.
8. Recherchez l’utilisateur dans la liste des utilisateurs au moyen de la recherche, du filtrage ou du tri.
9. Sélectionnez l’utilisateur dans la liste des utilisateurs et **activez**, **désactivez** ou **appliquez** la Multi-Factor Authentication comme souhaité.
   >[!NOTE]
   >Si un utilisateur est dans un état **Appliqué**, vous pouvez définir le définir sur **Désactivé** de façon temporaire pour lui permettre de revenir à son compte. Une fois qu’il est revenu dans son compte, vous pouvez ensuite redéfinir son état sur **Activé** pour lui demander d’enregistrer à nouveau ses informations de contact à sa prochaine connexion. Sinon, vous pouvez suivre les étapes décrites dans [Vérifier les informations de contact de l’authentification d’un utilisateur](#check-a-users-authentication-contact-info) pour vérifier ou définir ces données à sa place.

### <a name="check-a-users-authentication-contact-info"></a>Vérifier les informations de contact de l’authentification d’un utilisateur
Pour vérifier les informations de contact de l’authentification d’un utilisateur utilisées pour Authentification multifacteur, Accès conditionnel, Identity Protection et Réinitialisation du mot de passe, procédez comme suit :
1.  Ouvrez le [**portail Azure**](https://portal.azure.com/) et connectez-vous en tant **qu’Administrateur général**.
2.  Ouvrez l’**extension Azure Active Directory** en sélectionnant **Tous les services** en haut du menu de navigation principal de gauche.
3.  Tapez « **Azure Active Directory** » dans la zone de recherche de filtre et sélectionnez l’élément **Azure Active Directory**.
4.  Sélectionnez **Utilisateurs et groupes** dans le menu de navigation.
5.  Sélectionnez **Tous les utilisateurs**.
6.  **Recherchez** l’utilisateur qui vous intéresse et **sélectionnez la ligne** correspondante.
7.  Sélectionnez **Profil**.
8.  Faites défiler la page jusqu’aux **informations de contact d’authentification**.
9.  **Passez en revue** les données enregistrées pour l’utilisateur et mettez-les à jour si besoin.

### <a name="check-a-users-group-memberships"></a>Vérifier les appartenances d’un utilisateur à des groupes
Pour vérifier l’appartenance d’un utilisateur à des groupes, procédez comme suit :
1.  Ouvrez le [**portail Azure**](https://portal.azure.com/) et connectez-vous en tant **qu’Administrateur général**.
2.  Ouvrez l’**extension Azure Active Directory** en sélectionnant **Tous les services** en haut du menu de navigation principal de gauche.
3.  Tapez « **Azure Active Directory** » dans la zone de recherche de filtre et sélectionnez l’élément **Azure Active Directory**.
4.  Sélectionnez **Utilisateurs et groupes** dans le menu de navigation.
5.  Sélectionnez **Tous les utilisateurs**.
6.  **Recherchez** l’utilisateur qui vous intéresse et **sélectionnez la ligne** correspondante.
7.  Sélectionnez **Groupes** pour afficher les groupes dont l’utilisateur est membre.

### <a name="check-if-a-user-has-more-than-999-app-role-assignments"></a>Vérifier si un utilisateur a plus de 999 attributions de rôle d’application
Si plus de 999 attributions de rôle d’application sont affectées à l’utilisateur, il se peut qu’il ne puisse pas voir toutes ses applications sur Mes applications.

Cela est dû au fait que Mes applications indique actuellement jusqu’à 999 attributions de rôle d’application pour déterminer les applications auxquelles les utilisateurs sont affectés. Si un utilisateur est affecté à plus de 999 applications, il n’est pas possible de contrôler celles de ces applications qui s’afficheront dans le portail Mes applications.

Pour vérifier le nombre d’attributions de rôle d’application à un utilisateur, procédez comme suit :
1. Installez le module PowerShell [**Microsoft. Graph**](https://github.com/microsoftgraph/msgraph-sdk-powershell).
2. Exécutez `Connect-MgGraph -Scopes "Directory.Read.All"` et authentifiez-vous en tant qu’**administrateur général**.
3. Exécutez `$m = Get-MgUserAppRoleAssignment -UserId "<userId>" | Measure; $m.Count` pour déterminer le nombre actuel d’attributions de rôle d’application à l’utilisateur.

### <a name="check-a-users-assigned-licenses"></a>Vérifier les licences affectées à un utilisateur
Pour vérifier les licences affectées à un utilisateur, procédez comme suit :
1.  Ouvrez le [**portail Azure**](https://portal.azure.com/) et connectez-vous en tant **qu’Administrateur général**.
2.  Ouvrez l’**extension Azure Active Directory** en sélectionnant **Tous les services** en haut du menu de navigation principal de gauche.
3.  Tapez « **Azure Active Directory** » dans la zone de recherche de filtre et sélectionnez l’élément **Azure Active Directory**.
4.  Sélectionnez **Utilisateurs et groupes** dans le menu de navigation.
5.  Sélectionnez **Tous les utilisateurs**.
6.  **Recherchez** l’utilisateur qui vous intéresse et **sélectionnez la ligne** correspondante.
7.  Sélectionnez **Licences** pour voir quelles licences sont actuellement attribuées à l’utilisateur.

### <a name="assign-a-user-a-license"></a>Affecter une licence à un utilisateur 
Pour affecter une licence à un utilisateur, procédez comme suit :
1.  Ouvrez le [**portail Azure**](https://portal.azure.com/) et connectez-vous en tant **qu’Administrateur général**.
2.  Ouvrez l’**extension Azure Active Directory** en sélectionnant **Tous les services** en haut du menu de navigation principal de gauche.
3.  Tapez « **Azure Active Directory** » dans la zone de recherche de filtre et sélectionnez l’élément **Azure Active Directory**.
4.  Sélectionnez **Utilisateurs et groupes** dans le menu de navigation.
5.  Sélectionnez **Tous les utilisateurs**.
6.  **Recherchez** l’utilisateur qui vous intéresse et **sélectionnez la ligne** correspondante.
7.  Sélectionnez **Licences** pour voir quelles licences sont actuellement attribuées à l’utilisateur.
8.  Sélectionnez le bouton **Attribuer**.
9.  Sélectionnez **un ou plusieurs produits** dans la liste des produits disponibles.
10. **Facultatif** Sélectionnez l’élément **Options d’affectation** pour attribuer les produits de façon granulaire. Sélectionnez **OK**.
11. Sélectionnez le bouton **Attribuer** pour attribuer ces licences à cet utilisateur.

## <a name="troubleshooting-deep-links"></a>Résolution des problèmes de liens profonds
Les liens profonds ou URL d’accès utilisateur sont des liens que vos utilisateurs peuvent utiliser pour accéder à leurs applications à authentification unique par mot de passe directement à partir des barres d’URL de leurs navigateurs. En accédant à ce lien, les utilisateurs sont automatiquement connectés à l’application sans devoir accéder à Mes applications d’abord. Le lien est le même que celui dont les utilisateurs se servent pour accéder à ces applications à partir du lanceur d’applications Microsoft 365.

### <a name="checking-the-deep-link"></a>Vérification du lien profond

Pour vérifier si vous disposez du lien profond correct, procédez comme suit :
1. Ouvrez le [**portail Azure**](https://portal.azure.com/) et connectez-vous en tant **qu’Administrateur général** ou que **Coadministrateur**.
2. Ouvrez l’**extension Azure Active Directory** en sélectionnant **Tous les services** en haut du menu de navigation principal de gauche.
3. Tapez « **Azure Active Directory** » dans la zone de recherche de filtre et sélectionnez l’élément **Azure Active Directory**.
4. Sélectionnez **Applications d’entreprise** dans le menu de navigation de gauche d’Azure Active Directory.
5. Sélectionnez **Toutes les applications** pour afficher la liste complète de vos applications.
   * Si l’application que vous recherchez ne figure pas dans la liste, utilisez la commande **Filtre** en haut de la **liste de toutes les applications** et définissez l’option **Afficher** sur **Toutes les applications.**
6. Ouvrez le [**portail Azure**](https://portal.azure.com/) et connectez-vous en tant **qu’Administrateur général** ou que **Coadministrateur**.
7. Ouvrez l’**extension Azure Active Directory** en sélectionnant **Tous les services** en haut du menu de navigation principal de gauche.
8. Tapez « **Azure Active Directory** » dans la zone de recherche de filtre et sélectionnez l’élément **Azure Active Directory**.
9. Sélectionnez **Applications d’entreprise** dans le menu de navigation de gauche d’Azure Active Directory.
10. Sélectionnez **Toutes les applications** pour afficher la liste complète de vos applications.
    * Si l’application que vous recherchez ne figure pas dans la liste, utilisez la commande **Filtre** en haut de la **liste de toutes les applications** et définissez l’option **Afficher** sur **Toutes les applications.**
11. Sélectionnez l’application pour laquelle vous souhaitez vérifier le lien profond.
12. Recherchez l’étiquette **URL d’accès utilisateur**. Votre lien profond doit correspondre à cette URL.

## <a name="contact-support"></a>Contacter le support technique
Ouvrez un ticket de support en fournissant les informations suivantes, dans la mesure du possible :
-   ID d’erreur de corrélation
-   Nom d’utilisateur principal (adresse de messagerie de l’utilisateur)
-   ID de locataire
-   Type de navigateur
-   Fuseau horaire et heure/période à laquelle l’erreur se produit
-   Traces Fiddler

## <a name="next-steps"></a>Étapes suivantes
- [Série de guides de démarrage rapide sur la gestion des applications](view-applications-portal.md)

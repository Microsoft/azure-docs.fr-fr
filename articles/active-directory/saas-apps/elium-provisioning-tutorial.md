---
title: 'Tutoriel : Configurer Elium pour le provisionnement automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer Azure Active Directory pour provisionner et déprovisionner automatiquement des comptes d’utilisateurs dans Elium.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/19/2019
ms.author: Zhchia
ms.openlocfilehash: e8f027ccc577df79e561fca7194c20b6cc7ef2c6
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96005501"
---
# <a name="tutorial-configure-elium-for-automatic-user-provisioning"></a>Tutoriel : Configurer Elium pour le provisionnement automatique d’utilisateurs

Ce tutoriel montre comment configurer Elium et Azure Active Directory (Azure AD) pour provisionner et déprovisionner automatiquement des utilisateurs ou des groupes sur Elium.

> [!NOTE]
> Ce tutoriel décrit un connecteur reposant sur le service de provisionnement d’utilisateurs Azure AD. Pour obtenir des informations importantes sur l’utilisation et le fonctionnement de ce service, ainsi que des réponses aux questions fréquentes, consultez [Automatiser le provisionnement et le déprovisionnement d’utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Ce connecteur est actuellement en préversion. Pour obtenir les conditions d’utilisation générales relatives aux fonctionnalités Azure en préversion, consultez [Conditions d’utilisation supplémentaires des préversions Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

Ce tutoriel part du principe que vous disposez des prérequis suivants :

* un locataire Azure AD ;
* [un locataire Elium](https://www.elium.com/pricing/) ;
* un compte d’utilisateur dans Elium avec des autorisations d’administration.

## <a name="assigning-users-to-elium"></a>Affectation d’utilisateurs à Elium

Azure AD utilise un concept appelé *affectations* pour déterminer les utilisateurs recevant l’accès aux applications sélectionnées. Dans le cadre du provisionnement d’utilisateurs automatique, seuls les utilisateurs et les groupes qui ont été affectés à une application dans Azure AD sont synchronisés.

Avant de configurer et d’activer le provisionnement d’utilisateurs automatique, identifiez les utilisateurs et groupes dans Azure AD qui doivent accéder à Elium. Ensuite, affectez ces utilisateurs et groupes à Elium en suivant les étapes décrites dans [Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md).

## <a name="important-tips-for-assigning-users-to-elium"></a>Conseils importants pour l’affectation d’utilisateurs à Elium 

Nous vous recommandons d’attribuer un seul utilisateur Azure AD à Elium afin de tester la configuration du provisionnement d’utilisateurs automatique. D’autres utilisateurs et groupes peuvent être attribués ultérieurement.

Quand vous affectez un utilisateur à Elium, vous devez sélectionner un rôle propre à l’application valide (si disponible) dans la boîte de dialogue d’affectation. Les utilisateurs dont le rôle est **Accès par défaut** sont exclus du provisionnement.

## <a name="set-up-elium-for-provisioning"></a>Configurer Elium pour le provisionnement

Avant de configurer Elium pour le provisionnement d’utilisateurs automatique avec Azure AD, vous devez activer le provisionnement SCIM (System for Cross-domain Identity Management) dans Elium. Procédez comme suit :

1. Connectez-vous à Elium et accédez à **Mon profil** > **Paramètres**.

    ![Élément de menu Paramètres dans Elium](media/Elium-provisioning-tutorial/setting.png)

1. Dans le coin inférieur gauche, sous **AVANCÉ**, sélectionnez **Sécurité**.

    ![Lien Sécurité dans Elium](media/Elium-provisioning-tutorial/security.png)

1. Copiez les valeurs **URL de locataire** et **Jeton secret**. Vous utiliserez ces valeurs ultérieurement, dans les champs correspondants de l’onglet **Provisioning** (Provisionnement) de votre application Elium dans le portail Azure.

    ![Champs URL de locataire et Jeton secret dans Elium](media/Elium-provisioning-tutorial/token.png)

## <a name="add-elium-from-the-gallery"></a>Ajouter Elium à partir de la galerie

Pour configurer Elium pour le provisionnement d’utilisateurs automatique avec Azure AD, vous devez également ajouter Elium à votre liste d’applications SaaS (Software-as-a-Service) managées à partir de la galerie d’applications Azure AD. Procédez comme suit :

1. Dans le panneau de navigation gauche du [portail Azure](https://portal.azure.com), sélectionnez **Azure Active Directory**.

    ![Élément de menu Azure Active Directory](common/select-azuread.png)

1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.

     ![Panneau Applications d’entreprise Azure AD](common/enterprise-applications.png)

1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** dans la partie supérieure du volet.

    ![Lien Nouvelle application](common/add-new-app.png)

1. Dans la zone de recherche, tapez **Elium**, sélectionnez **Elium** dans la liste de résultats, puis **Ajouter** pour ajouter l’application.

    ![Zone de recherche Ajouter à partir de la galerie](common/search-new-app.png)

## <a name="configure-automatic-user-provisioning-to-elium"></a>Configurer le provisionnement d’utilisateurs automatique sur Elium

Cette section vous guide tout au long des étapes de configuration du service de provisionnement Azure AD pour créer, mettre à jour et désactiver des utilisateurs et des groupes dans Elium en fonction des affectations d’utilisateurs et de groupes dans Azure AD.

> [!TIP]
> Vous pouvez aussi choisir d’activer l’authentification unique pour Elium selon SAML (Security Assertion Markup Language) en suivant les instructions fournies dans le [tutoriel sur l’authentification unique Elium](Elium-tutorial.md). Vous pouvez configurer l’authentification unique indépendamment du provisionnement d’utilisateurs automatique, même si ces deux fonctionnalités se complètent.

Pour configurer le provisionnement d’utilisateurs automatique pour Elium dans Azure AD, procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com), sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.

    ![Panneau Applications d’entreprise Azure AD](common/enterprise-applications.png)

1. Dans la liste des applications, sélectionnez **Elium**.

    ![Liste des applications dans le panneau Applications d’entreprise](common/all-applications.png)

1. Sélectionnez l’onglet **Approvisionnement**.

    ![Onglet Provisioning (Provisionnement) dans le panneau Applications d’entreprise](common/provisioning.png)

1. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Paramètre Automatique pour le mode de provisionnement](common/provisioning-automatic.png)

1. Dans la section **Informations d’identification d’administrateur**, saisissez **\<tenantURL\>/scim/v2** dans le champ **URL de locataire**. (La valeur **tenantURL** est la valeur récupérée précédemment à partir de la console d’administration Elium.) Tapez également la valeur Elium **Jeton secret** dans le champ **Jeton secret**. Enfin, sélectionnez **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à Elium. Si la connexion échoue, vérifiez que votre compte Elium dispose d’autorisations d’administration, puis réessayez.

    ![Champs URL de locataire et Jeton secret dans Informations d’identification d’administrateur](common/provisioning-testconnection-tenanturltoken.png)

1. Dans le champ **E-mail de notification**, entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur de provisionnement. Cochez ensuite la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

1. Cliquez sur **Enregistrer**.

1. Dans la section **Mappages**, sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec Elium**.

    ![Lien Synchroniser pour le mappage des utilisateurs Azure AD à Elium](media/Elium-provisioning-tutorial/usermapping.png)

1. Dans la section **Mappages d’attributs**, passez en revue les attributs utilisateur qui sont synchronisés entre Azure AD et Elium. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés afin d’établir une correspondance avec les comptes d’utilisateurs dans Elium pour les opérations de mise à jour. Sélectionnez **Enregistrer** pour valider les modifications.

    ![Mappages d’attributs entre Azure AD et Elium](media/Elium-provisioning-tutorial/userattribute.png)

1. Pour configurer des filtres d’étendue, suivez les instructions fournies dans le [tutoriel sur les filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

1. Pour activer le service de provisionnement d’Azure AD pour Elium, définissez le paramètre **État d’approvisionnement** sur **Activé** dans la section **Paramètres**.

    ![État de provisionnement activé](common/provisioning-toggle-on.png)

1. Définissez les utilisateurs et groupes que vous voulez provisionner dans Elium en choisissant les valeurs souhaitées dans la zone de liste déroulante **Étendue** de la section **Paramètres**.

    ![Zone de liste Étendue de provisionnement](common/provisioning-scope.png)

1. Quand vous êtes prêt à effectuer l’approvisionnement, sélectionnez **Enregistrer**.

    ![Bouton Enregistrer pour la configuration de provisionnement](common/provisioning-configuration-save.png)

Cette opération démarre la synchronisation initiale de tous les utilisateurs et groupes définis dans **Étendue** dans la section **Paramètres**. Ce processus de synchronisation initial prend plus de temps que les synchronisations ultérieures. Pour plus d’informations sur le temps nécessaire au provisionnement, consultez [Combien de temps faut-il pour provisionner des utilisateurs ?](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md#how-long-will-it-take-to-provision-users).

Utilisez la section **État actuel** pour superviser la progression et suivre les liens vers votre rapport d’activité de provisionnement. Ce rapport décrit toutes les actions effectuées par le service de provisionnement Azure AD sur Elium. Pour plus d’informations, consultez [Vérifier l’état de l’approvisionnement d’utilisateurs](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md). Pour la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md).
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)

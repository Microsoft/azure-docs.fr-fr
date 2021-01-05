---
title: 'Tutoriel : Configurer Oracle Fusion ERP pour le provisionnement automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer Azure Active Directory pour provisionner et déprovisionner automatiquement des comptes d’utilisateurs sur Oracle Fusion ERP.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 07/26/2019
ms.author: Zhchia
ms.openlocfilehash: da6e1a8ba31f8f4991bde4803191598a015a68b3
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94358437"
---
# <a name="tutorial-configure-oracle-fusion-erp-for-automatic-user-provisioning"></a>Tutoriel : Configurer Oracle Fusion ERP pour le provisionnement automatique d’utilisateurs

L’objectif de ce tutoriel est de présenter les étapes à effectuer dans Oracle Fusion ERP et Azure Active Directory (Azure AD) pour configurer Azure AD pour le provisionnement et le déprovisionnement automatiques d’utilisateurs et/ou de groupes sur Oracle Fusion ERP.

> [!NOTE]
>  Ce didacticiel décrit un connecteur reposant sur le service d’attribution d’utilisateurs Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Ce connecteur est en préversion. Pour plus d’informations sur les conditions d’utilisation Microsoft Azure générales relatives aux fonctionnalités d’évaluation, consultez [Conditions d’utilisation supplémentaires des préversions Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* un locataire Azure AD ;
* [un locataire Oracle Fusion ERP](https://www.oracle.com/applications/erp/).
* Un compte d’utilisateur dans Oracle Fusion ERP avec des autorisations d’administrateur

## <a name="assign-users-to-oracle-fusion-erp"></a>Attribuer des utilisateurs à Oracle Fusion ERP 
Azure Active Directory utilise un concept appelé « affectations » pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre d’une attribution automatique d’utilisateurs, seuls les utilisateurs ou les groupes auxquels une application a été attribuée dans Azure AD sont synchronisés.

Avant de configurer et d’activer le provisionnement automatique d’utilisateurs, vous devez décider quels utilisateurs et/ou groupes dans Azure AD ont besoin d’accéder à Oracle Fusion ERP. Une fois que vous avez choisi, vous pouvez attribuer ces utilisateurs et/ou groupes à Oracle Fusion ERP en suivant les instructions fournies ici :
 
* [Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md) 

 ## <a name="important-tips-for-assigning-users-to-oracle-fusion-erp"></a>Conseils importants pour l’attribution d’utilisateurs à Oracle Fusion ERP 

 * Il est recommandé de n’attribuer qu’un seul utilisateur Azure AD à Oracle Fusion ERP pour tester la configuration du provisionnement automatique d’utilisateurs. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

* Quand vous attribuez un utilisateur à Oracle Fusion ERP, vous devez sélectionner un rôle valide propre à l’application (si disponible) dans la boîte de dialogue d’attribution. Les utilisateurs dont le rôle est Accès par défaut sont exclus de l’approvisionnement.

## <a name="set-up-oracle-fusion-erp-for-provisioning"></a>Configurer Oracle Fusion ERP pour le provisionnement

Avant de configurer Oracle Fusion ERP pour le provisionnement automatique d’utilisateurs avec Azure AD, vous devez activer le provisionnement SCIM sur Oracle Fusion ERP.

1. Connectez-vous à la [console d’administration Oracle Fusion ERP](https://cloud.oracle.com/sign-in).

2. Cliquez sur le navigateur dans l’angle supérieur gauche. Sous **Tools** , sélectionnez **Security Console**.

    :::image type="content" source="media/oracle-fusion-erp-provisioning-tutorial/login.png" alt-text="Capture d’écran de la page du navigateur dans la console d’administration Oracle Fusion ERP. Tools et Security Console sont mis en évidence." border="false":::

3. Accédez à **Users**.
    
    :::image type="content" source="media/oracle-fusion-erp-provisioning-tutorial/user.png" alt-text="Capture d’écran d’un volet de la console d’administration Oracle Fusion ERP. L’élément Utilisateurs est mis en évidence." border="false":::

4. Enregistrez le nom d’utilisateur et le mot de passe du compte d’utilisateur administrateur que vous allez utiliser pour vous connecter à la console d’administration Oracle Fusion ERP. Ces valeurs doivent être entrées dans les champs **Nom d’utilisateur de l’administrateur** et **Mot de passe** de l’onglet Provisionnement de votre application Oracle Fusion ERP dans le portail Azure.

## <a name="add-oracle-fusion-erp-from-the-gallery"></a>Ajouter Oracle Fusion ERP à partir de la galerie

Pour configurer Oracle Fusion ERP pour le provisionnement automatique d’utilisateurs avec Azure AD, vous devez ajouter Oracle Fusion ERP à partir de la galerie d’applications Azure AD à votre liste d’applications SaaS managées.

**Pour ajouter Oracle Fusion ERP à partir de la galerie d’applications Azure AD, procédez comme suit :**

1. Dans le panneau de navigation gauche du **[portail Azure](https://portal.azure.com)** , sélectionnez **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une nouvelle application, cliquez sur le bouton **Nouvelle application** en haut du volet.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **Oracle Fusion ERP** , puis sélectionnez **Oracle Fusion ERP** dans le volet de résultats.

    ![Oracle Fusion ERP dans la liste des résultats](common/search-new-app.png)

 ## <a name="configure-automatic-user-provisioning-to-oracle-fusion-erp"></a>Configurer le provisionnement automatique d’utilisateurs sur Oracle Fusion ERP 

Cette section vous guide tout au long des étapes de configuration du service de provisionnement d’Azure AD pour créer, mettre à jour et désactiver des utilisateurs et/ou des groupes dans Oracle Fusion ERP en fonction des affectations d’utilisateurs et/ou de groupes dans Azure AD.

> [!TIP]
> Vous pouvez également choisir d’activer l’authentification unique basée sur SAML pour Oracle Fusion ERP en suivant les instructions fournies dans le [tutoriel sur l’authentification unique Oracle Fusion ERP](oracle-fusion-erp-tutorial.md). L’authentification unique peut être configurée indépendamment de l’attribution automatique d’utilisateurs, bien que ces deux fonctionnalités se complètent.

> [!NOTE]
> Pour en savoir plus sur le point de terminaison SCIM d’Oracle Fusion ERP, reportez-vous à [API REST pour les fonctionnalités courantes dans Oracle Applications Cloud](https://docs.oracle.com/en/cloud/saas/applications-common/18b/farca/index.html).

### <a name="to-configure-automatic-user-provisioning-for-fuze-in-azure-ad"></a>Pour configurer le provisionnement automatique d’utilisateurs pour Fuze dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise** , puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Oracle Fusion ERP**.

    ![Lien Oracle Fusion ERP dans la liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Capture d’écran des options Gérer avec l’option Provisionnement en évidence.](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran de la liste déroulante Mode de provisionnement avec l’option Automatique en évidence.](common/provisioning-automatic.png)

5. Dans la section **Informations d’identification de l’administrateur** , entrez `https://ejlv.fa.em2.oraclecloud.com/hcmRestApi/scim/` dans **URL de locataire**. Entrez le nom d’utilisateur administrateur et le mot de passe récupérés antérieurement dans les champs **Nom d’utilisateur de l’administrateur** et **Mot de passe**. Cliquez sur **Tester la connexion** pour vérifier la connexion entre Azure AD et Oracle Fusion ERP. 

    :::image type="content" source="media/oracle-fusion-erp-provisioning-tutorial/admin.png" alt-text="Capture d’écran de la section Informations d’identification de l’administrateur. Un bouton Tester la connexion et les champs pour URL de locataire, Nom d’utilisateur de l’administrateur et Mot de passe d’administrateur sont visibles." border="false":::

6. Dans le champ **E-mail de notification** , entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement, puis cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

7. Cliquez sur **Enregistrer**.

8. Dans la section **Mappages** , sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec Oracle Fusion ERP**.

    :::image type="content" source="media/oracle-fusion-erp-provisioning-tutorial/user-mapping.png" alt-text="Capture d’écran de la section Mappages. Sous Nom, Synchroniser les utilisateurs Azure Active Directory avec Oracle Fusion ERP est visible." border="false":::

9. Dans la section **Mappages d’attributs** , passez en revue les attributs utilisateur qui sont synchronisés entre Azure AD et Oracle Fusion ERP. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes d’utilisateur dans Oracle Fusion ERP pour les opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    :::image type="content" source="media/oracle-fusion-erp-provisioning-tutorial/user-attribute.png" alt-text="Capture d’écran de la page Mappages d’attributs. Un tableau liste les attributs Azure Active Directory et Oracle Fusion ERP et la priorité de correspondance." border="false":::

10. Dans la section **Mappages** , sélectionnez **Synchroniser les groupes Azure Active Directory avec Oracle Fusion ERP**.

    ![Mappages de groupes ERP Oracle Fusion](media/oracle-fusion-erp-provisioning-tutorial/groupmappings.png)

11. Dans la section **Mappages d’attributs** , passez en revue les attributs de groupe qui sont synchronisés entre Azure AD et Oracle Fusion ERP. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les groupes dans Oracle Fusion ERP pour les opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    ![Attributs de groupes Oracle Fusion ERP](media/oracle-fusion-erp-provisioning-tutorial/groupattributes.png)

12. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Pour activer le service de provisionnement Azure AD pour Oracle Fusion ERP, définissez le paramètre **État de l’approvisionnement** sur **Activé** dans la section **Paramètres**.

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

14. Définissez les utilisateurs et/ou groupes que vous souhaitez provisionner sur Oracle Fusion ERP en choisissant les valeurs souhaitées dans **Étendue** dans la section **Paramètres**.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

15. Lorsque vous êtes prêt à effectuer l’approvisionnement, cliquez sur **Enregistrer**.

    ![Enregistrement de la configuration de l’approvisionnement](common/provisioning-configuration-save.png)

    Cette opération démarre la synchronisation initiale de tous les utilisateurs et/ou groupes définis dans **Étendue** dans la section **Paramètres**. La synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service de provisionnement Azure AD est en cours d’exécution. Vous pouvez utiliser la section **Détails de la synchronisation** pour surveiller la progression et suivre les liens vers les rapports d’activité de provisionnement, qui décrivent toutes les actions effectuées par le service de provisionnement Azure AD sur Oracle Fusion ERP.

    Pour plus d’informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Limitations du connecteur

* Oracle Fusion ERP prend uniquement en charge l’authentification de base pour son point de terminaison SCIM.
* Oracle Fusion ERP ne prend pas en charge l’approvisionnement des groupes.
* Les rôles dans Oracle Fusion ERP sont mappés à des groupes dans Azure AD. Pour attribuer des rôles aux utilisateurs dans la solution Oracle Fusion ERP à partir d’Azure AD, vous devez attribuer les utilisateurs aux groupes Azure AD souhaités qui sont nommés d’après les rôles dans la solution Oracle Fusion ERP.

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)

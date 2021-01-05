---
title: 'Tutoriel : Configurer Zscaler Private Access (ZPA) pour l’approvisionnement automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer Azure Active Directory pour approvisionner et déprovisionner automatiquement des comptes d’utilisateur sur Zscaler Private Access (ZPA).
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/07/2019
ms.author: Zhchia
ms.openlocfilehash: da3ad4f70e0cc41fe95327eb74093c1471c877bd
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96351569"
---
# <a name="tutorial-configure-zscaler-private-access-zpa-for-automatic-user-provisioning"></a>Tutoriel : Configurer Zscaler Private Access (ZPA) pour l’approvisionnement automatique d’utilisateurs

L’objectif de ce tutoriel est de présenter les étapes à effectuer dans Zscaler Private Access (ZPA) et Azure Active Directory (Azure AD) afin de configurer Azure AD pour l’approvisionnement et le déprovisionnement automatiques d’utilisateurs et/ou de groupes sur Zscaler Private Access (ZPA).

> [!NOTE]
> Ce didacticiel décrit un connecteur reposant sur le service d’attribution d’utilisateurs Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Ce connecteur est actuellement en préversion publique. Pour plus d’informations sur les conditions d’utilisation Microsoft Azure générales relatives aux fonctionnalités d’évaluation, consultez [Conditions d’utilisation supplémentaires des préversions Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* un locataire Azure AD ;
* [un locataire Zscaler Private Access (ZPA)](https://www.zscaler.com/pricing-and-plans#contact-us) ;
* un compte d’utilisateur Zscaler Private Access (ZPA) ayant des autorisations d’administrateur.

## <a name="assigning-users-to-zscaler-private-access-zpa"></a>Attribution d’utilisateurs à Zscaler Private Access (ZPA).

Azure Active Directory utilise un concept appelé *affectations* pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre d’une attribution automatique d’utilisateurs, seuls les utilisateurs ou les groupes auxquels une application a été attribuée dans Azure AD sont synchronisés.

Avant de configurer et d’activer l’approvisionnement automatique d’utilisateurs, vous devez décider quels utilisateurs et/ou groupes dans Azure AD ont besoin d’accéder à Zscaler Private Access (ZPA). Une fois que vous avez choisi, vous pouvez attribuer ces utilisateurs et/ou groupes à votre application Zscaler Private Access (ZPA) en suivant les instructions fournies ici :
* [Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-zscaler-private-access-zpa"></a>Conseils importants pour l’attribution d’utilisateurs à Zscaler Private Access (ZPA)

* Il est recommandé de n’attribuer qu’un seul utilisateur Azure AD à Zscaler Private Access (ZPA) afin de tester la configuration de l’approvisionnement automatique d’utilisateurs. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

* Quand vous attribuez un utilisateur à Zscaler Private Access (ZPA), vous devez sélectionner un rôle valide propre à l’application (si disponible) dans la boîte de dialogue d’assignation. Les utilisateurs dont le rôle est **Accès par défaut** sont exclus de l’approvisionnement.

## <a name="set-up-zscaler-private-access-zpa-for-provisioning"></a>Configurer Zscaler Private Access (ZPA) pour l’approvisionnement

1. Connectez-vous à votre [console d’administration Zscaler Private Access (ZPA)](https://admin.private.zscaler.com/). Accédez à **Administration > Configuration IdP**.

    ![Console d’administration Zscaler Private Access (ZPA)](media/zscaler-private-access-provisioning-tutorial/idpconfig.png)

2.  Vérifiez qu’un IdP pour l’**authentification unique** est configuré. Si aucun IdP n’est configuré, ajoutez-en un en cliquant sur l’icône plus dans le coin supérieur droit de l’écran.

    ![Zscaler Private Access (ZPA) – Ajouter SCIM](media/zscaler-private-access-provisioning-tutorial/plusicon.png)

3. Suivez les instructions de l’Assistant **Ajouter une configuration IdP** pour ajouter un fournisseur d’identité. Laissez le champ **Authentification unique** défini sur **Utilisateur**. Indiquez un **Nom** et sélectionnez les **Domaines** dans la liste déroulante. Cliquez sur **Suivant** pour accéder à la fenêtre suivante.

    ![Zscaler Private Access (ZPA) – Ajouter un IdP](media/zscaler-private-access-provisioning-tutorial/addidp.png)

4. Téléchargez le **Certificat du fournisseur de services**. Cliquez sur **Suivant** pour accéder à la fenêtre suivante.

    ![Zscaler Private Access (ZPA) – Certificat du fournisseur de services](media/zscaler-private-access-provisioning-tutorial/spcertificate.png)

5. Dans la fenêtre suivante, téléchargez le **Certificat du fournisseur de services** téléchargé précédemment.

    ![Zscaler Private Access (ZPA) – Charger un certificat](media/zscaler-private-access-provisioning-tutorial/uploadfile.png)

6.  Faites défiler vers le bas pour indiquer l’**URL d’authentification unique** et l’**ID d’entité IdP**.

    ![Zscaler Private Access (ZPA) – ID d’IdP](media/zscaler-private-access-provisioning-tutorial/idpid.png)

7.  Faites défiler vers le bas jusqu’à **Activer la synchronisation SCIM**. Cliquez sur le bouton **Générer un nouveau jeton**. Copiez le **jeton du porteur**. Cette valeur devra être entrée dans le champ Jeton secret dans l’onglet Approvisionnement de votre application Zscaler Private Access (ZPA) dans le Portail Azure.

    ![Zscaler Private Access (ZPA) – Créer un jeton](media/zscaler-private-access-provisioning-tutorial/token.png)

8.  Pour localiser **Tenant URL** (URL de locataire), accédez à **Administration > IdP Configuration (Configuration IdP)** . Cliquez sur le nom de la configuration IdP que vous venez d’ajouter sur la page.

    ![Zscaler Private Access (ZPA) – Nom d’IdP](media/zscaler-private-access-provisioning-tutorial/idpname.png)

9.  Faites défiler jusqu’au bas de la page pour afficher le **Point de terminaison du fournisseur de services SCIM**. Copiez le **Point de terminaison du fournisseur de services SCIM**. Cette valeur devra être entrée dans le champ URL de locataire dans l’onglet Approvisionnement de votre application Zscaler Private Access (ZPA) dans le Portail Azure.

    ![Zscaler Private Access (ZPA) – URL SCIM](media/zscaler-private-access-provisioning-tutorial/tenanturl.png)


## <a name="add-zscaler-private-access-zpa-from-the-gallery"></a>Ajouter Zscaler Private Access (ZPA) à partir de la galerie

Avant de configurer Zscaler Private Access (ZPA) pour l’approvisionnement automatique d’utilisateurs avec Azure AD, vous devez ajouter Zscaler Private Access (ZPA) à partir de la galerie d’applications Azure AD à votre liste d’applications SaaS managées.

**Pour ajouter Zscaler Private Access (ZPA) à partir de la galerie d’applications Azure AD, procédez comme suit :**

1. Dans le panneau de navigation gauche du **[portail Azure](https://portal.azure.com)** , sélectionnez **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une nouvelle application, cliquez sur le bouton **Nouvelle application** en haut du volet.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **Zscaler Private Access (ZPA)** , sélectionnez **Zscaler Private Access (ZPA)** dans le panneau des résultats, puis cliquez sur le bouton **Ajouter** de l’application.

    ![Zscaler Private Access (ZPA) dans la liste des résultats](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-zscaler-private-access-zpa"></a>Configuration de l’approvisionnement automatique d’utilisateurs sur Zscaler Private Access (ZPA) 

Cette section vous guide tout au long des étapes de configuration du service d’approvisionnement d’Azure AD pour créer, mettre à jour et désactiver des utilisateurs et/ou des groupes dans Zscaler Private Access (ZPA) en fonction des attributions d’utilisateurs et/ou de groupes dans Azure AD.

> [!TIP]
> Vous pouvez également choisir d’activer l’authentification unique basée sur SAML pour Zscaler Private Access (ZPA) en suivant les instructions fournies dans le [didacticiel sur l’authentification unique Zscaler Private Access (ZPA)](./zscalerprivateaccess-tutorial.md). La configuration de l’authentification unique est indépendante de celle de l’approvisionnement automatique d’utilisateurs, même si ces deux fonctionnalités se complètent.

> [!NOTE]
> Pour en savoir plus sur le point de terminaison SCIM de Zscaler Private Access, consultez ce [document](https://www.zscaler.com/partners/microsoft).

### <a name="to-configure-automatic-user-provisioning-for-zscaler-private-access-zpa-in-azure-ad"></a>Pour configurer l’approvisionnement automatique d’utilisateurs pour Zscaler Private Access (ZPA) dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Zscaler Private Access (ZPA)** .

    ![Lien Zscaler Private Access (ZPA) dans la liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Capture d’écran des options Gérer avec l’option Provisionnement en évidence.](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran de la liste déroulante Mode de provisionnement avec l’option Automatique en évidence.](common/provisioning-automatic.png)

5. Dans la section **Informations d’identification Administrateur**, entrez la valeur **Point de terminaison du fournisseur de services SCIM** récupérée précédemment dans **URL de locataire**. Entrez la valeur **Jeton du porteur** récupérée précédemment dans **Jeton secret**. Cliquez sur **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à Zscaler Private Access (ZPA). Si la connexion échoue, vérifiez que votre compte Zscaler Private Access (ZPA) dispose des autorisations d’administrateur et réessayez.

    ![URL de locataire + Jeton](common/provisioning-testconnection-tenanturltoken.png)

6. Dans le champ **E-mail de notification**, entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement, puis cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

7. Cliquez sur **Enregistrer**.

8. Dans la section **Mappages**, sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec Zscaler Private Access (ZPA)** .

    ![Zscaler Private Access (ZPA) – Mappages d’utilisateur](media/zscaler-private-access-provisioning-tutorial/usermappings.png)

9. Dans la section **Mappages d’attributs**, passez en revue les attributs utilisateur qui sont synchronisés entre Azure AD et Zscaler Private Access (ZPA). Les attributs sélectionnés comme propriétés de **Correspondance** sont utilisés pour la mise en correspondre des comptes d’utilisateur dans Zscaler Private Access (ZPA) dans le cadre des opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    ![Zscaler Private Access (ZPA) – Attributs utilisateur](media/zscaler-private-access-provisioning-tutorial/userattributes.png)

10. Dans la section **Mappages**, sélectionnez **Synchroniser les groupes Azure Active Directory avec Zscaler Private Access (ZPA)** .

    ![Zscaler Private Access (ZPA) – Mappages de groupes](media/zscaler-private-access-provisioning-tutorial/groupmappings.png)

11. Dans la section **Mappages d’attributs**, passez en revue les attributs de groupe qui sont synchronisés entre Azure AD et Zscaler Private Access (ZPA). Les attributs sélectionnés comme propriétés de **Correspondance** sont utilisés pour la mise en correspondre des groupes dans Zscaler Private Access (ZPA) dans le cadre des opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    ![Zscaler Private Access (ZPA) – Attributs de groupe](media/zscaler-private-access-provisioning-tutorial/groupattributes.png)

12. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Pour activer le service d’approvisionnement Azure AD pour Zscaler Private Access (ZPA), définissez le paramètre **État de l’approvisionnement** sur **Activé** dans la section **Paramètres**.

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

14. Définissez les utilisateurs et/ou groupes que vous souhaitez approvisionner sur Zscaler Private Access (ZPA) en choisissant les valeurs souhaitées dans **Étendue** dans la section **Paramètres**.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

15. Lorsque vous êtes prêt à effectuer l’approvisionnement, cliquez sur **Enregistrer**.

    ![Enregistrement de la configuration de l’approvisionnement](common/provisioning-configuration-save.png)

Cette opération démarre la synchronisation initiale de tous les utilisateurs et/ou groupes définis dans **Étendue** dans la section **Paramètres**. La synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service de provisionnement Azure AD est en cours d’exécution. Vous pouvez utiliser la section **Détails de synchronisation** pour surveiller la progression et les liens vers les rapports d’activité d’approvisionnement, qui décrivent toutes les actions effectuées par le service d’approvisionnement Azure AD sur Zscaler Private Access (ZPA).

Pour plus d’informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)
---
title: 'Tutoriel : Approvisionnement d’utilisateurs pour Linkedin Elevate - Azure AD'
description: Découvrez comment configurer Azure Active Directory pour approvisionner et déprovisionner automatiquement des comptes d’utilisateur sur LinkedIn Elevate.
services: active-directory
author: ArvindHarinder1
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/28/2019
ms.author: arvinh
ms.openlocfilehash: 5e972475530ad36a188f73990bb9eca35748c36c
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94358947"
---
# <a name="tutorial-configure-linkedin-elevate-for-automatic-user-provisioning"></a>Tutoriel : Configurer l’attribution automatique d’utilisateurs dans LinkedIn Elevate

L’objectif de ce didacticiel est de vous montrer les étapes à effectuer dans LinkedIn Elevate et Azure AD pour approvisionner et déprovisionner automatiquement des comptes d’utilisateur d’Azure AD vers LinkedIn Elevate.

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce didacticiel part du principe que vous disposez des éléments suivants :

* Un client Azure Active Directory
* Un locataire LinkedIn Elevate
* Un compte d’administrateur dans LinkedIn Elevate ayant accès au Centre des comptes LinkedIn

> [!NOTE]
> Azure Active Directory s’intègre à LinkedIn Elevate à l’aide du protocole [SCIM](http://www.simplecloud.info/).

## <a name="assigning-users-to-linkedin-elevate"></a>Affectation d’utilisateurs à LinkedIn Elevate

Azure Active Directory utilise un concept appelé « affectations » pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre de l’approvisionnement automatique de comptes d’utilisateur, les utilisateurs et les groupes qui ont été « affectés » à une application dans Azure AD sont synchronisés.

Avant de configurer et d’activer le service d’approvisionnement, vous devez déterminer quels utilisateurs et/ou groupes dans Azure AD représentent les utilisateurs qui ont besoin d’accéder à LinkedIn Elevate. Une fois que vous avez choisi, vous pouvez affecter ces utilisateurs à LinkedIn Elevate en suivant les instructions fournies ici :

[Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-linkedin-elevate"></a>Conseils importants pour l’affectation d’utilisateurs à LinkedIn Elevate

* Il est recommandé d’affecter un seul utilisateur Azure AD à LinkedIn Elevate pour tester la configuration de l’approvisionnement. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

* Quand vous affectez un utilisateur à LinkedIn Elevate, vous devez sélectionner le rôle **Utilisateur** dans la boîte de dialogue d’affectation. Le rôle « Accès par défaut » ne fonctionne pas pour l’approvisionnement.

## <a name="configuring-user-provisioning-to-linkedin-elevate"></a>Configuration de l’approvisionnement des utilisateurs sur LinkedIn Elevate

Cette section vous guide pour connecter votre instance d’Azure AD à l’API d’approvisionnement de comptes d’utilisateur SCIM de LinkedIn Elevate et configurer le service d’approvisionnement pour créer, mettre à jour et désactiver des comptes d’utilisateur affectés dans LinkedIn Elevate en fonction des affectations d’utilisateurs et de groupes dans Azure AD.

**Conseil :** vous pouvez également choisir d’activer l’authentification unique SAML pour LinkedIn Elevate en suivant les instructions indiquées sur le [Portail Azure](https://portal.azure.com). L’authentification unique peut être configurée indépendamment de l’approvisionnement automatique, bien que ces deux fonctionnalités se complètent.

### <a name="to-configure-automatic-user-account-provisioning-to-linkedin-elevate-in-azure-ad"></a>Pour configurer l’approvisionnement automatique de comptes d’utilisateur sur LinkedIn Elevate dans Azure AD :

La première étape consiste à récupérer votre jeton d’accès LinkedIn. Si vous êtes administrateur d’entreprise, vous pouvez approvisionner vous-même un jeton d’accès. Dans le Centre des comptes, accédez à **Paramètres &gt; Paramètres globaux** , puis ouvrez le panneau de configuration **SCIM**.

> [!NOTE]
> Si vous ouvrez le Centre des comptes directement plutôt qu’en passant par un lien, vous pouvez y accéder en effectuant les étapes suivantes.

1. Connectez-vous au Centre des comptes.

2. Sélectionnez **Administrateur &gt; Paramètres d’administration**.

3. Cliquez sur **Intégrations avancées** dans la barre latérale gauche. Vous êtes redirigé vers le Centre des comptes.

4. Cliquez sur **+ Ajouter une nouvelle configuration SCIM** , puis suivez la procédure en remplissant chaque champ.

    > [!NOTE]
    > Si l’affectation automatique de licences n’est pas activée, cela signifie que seules les données utilisateur sont synchronisées.

    ![Capture d’écran montrant les paramètres globaux du Centre des comptes LinkedIn.](./media/linkedinelevate-provisioning-tutorial/linkedin_elevate1.PNG)

    > [!NOTE]
    > Quand l’affectation automatique de licences est activée, vous devez noter l’instance d’application et le type de licence. Les licences sont toutes affectées sur le principe du « premier arrivé, premier servi ».

    ![Capture d’écran montrant la page de configuration de SCIM.](./media/linkedinelevate-provisioning-tutorial/linkedin_elevate2.PNG)

5. Cliquez sur **Générer un jeton**. Un jeton d’accès doit s’afficher sous le champ **Jeton d’accès**.

6. Enregistrez votre jeton d’accès dans votre Presse-papiers ou votre ordinateur avant de quitter la page.

7. Ensuite, connectez-vous au [portail Azure](https://portal.azure.com), puis accédez à la section **Azure Active Directory > Applications d’entreprise > Toutes les applications**.

8. Si vous avez déjà configuré LinkedIn Elevate pour l’authentification unique, recherchez votre instance de LinkedIn Elevate à l’aide du champ de recherche. Sinon, sélectionnez **Ajouter** , puis recherchez **LinkedIn Elevate** dans la galerie d’applications. Sélectionnez LinkedIn Elevate dans les résultats de la recherche, puis ajoutez-le à votre liste d’applications.

9. Sélectionnez votre instance de LinkedIn Elevate, puis sélectionnez l’onglet **Approvisionnement**.

10. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran montrant la page Approvisionnement LinkedIn Elevate.](./media/linkedinelevate-provisioning-tutorial/linkedin_elevate3.PNG)

11. Renseignez les champs suivants sous **Informations d’identification de l’administrateur**  :

    * Dans le champ **URL de locataire** , entrez `https://api.linkedin.com`.

    * Dans le champ **Jeton secret** , entrez le jeton d’accès que vous avez généré à l’étape 1, puis cliquez sur **Tester la connexion**.

    * Une notification de réussite doit s’afficher en haut à droite de votre portail.

12. Entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement dans le champ **E-mail de notification** , puis cochez la case se trouvant en dessous.

13. Cliquez sur **Enregistrer**.

14. Dans la section **Mappages d’attributs** , passez en revue les attributs d’utilisateur et de groupe qui seront synchronisés d’Azure AD vers LinkedIn Elevate. Notez que les attributs sélectionnés comme propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes d’utilisateur et les groupes dans LinkedIn Elevate pour les opérations de mise à jour. Cliquez sur le bouton Enregistrer pour valider les modifications.

    ![Capture d’écran montrant les mappages, y compris les mappages d’attributs.](./media/linkedinelevate-provisioning-tutorial/linkedin_elevate4.PNG)

15. Pour activer le service d’approvisionnement Azure AD pour LinkedIn Elevate, définissez l’ **État d’approvisionnement** sur **Activé** dans la section **Paramètres**.

16. Cliquez sur **Enregistrer**.

Cette commande démarre la synchronisation initiale des utilisateurs et/ou groupes affectés à LinkedIn Elevate dans la section Utilisateurs et groupes. Notez que la synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service est en cours d’exécution. Vous pouvez utiliser la section **Détails de la synchronisation** pour surveiller la progression et suivre les liens vers les journaux d’activité de provisionnement, qui décrivent toutes les actions effectuées par le service de provisionnement dans votre application LinkedIn Elevate.

Pour plus d’informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

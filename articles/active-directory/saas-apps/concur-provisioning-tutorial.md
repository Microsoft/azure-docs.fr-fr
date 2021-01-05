---
title: 'Tutoriel : Configurer Concur pour l’approvisionnement automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Concur.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/26/2018
ms.author: jeedes
ms.openlocfilehash: edb21287b30f8ba77d6312ec6b456e20aa260598
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94358209"
---
# <a name="tutorial-configure-concur-for-automatic-user-provisioning"></a>Tutoriel : Configurer Concur pour l’approvisionnement automatique d’utilisateurs

L’objectif de ce didacticiel est de vous montrer les étapes à effectuer dans Concur et Azure AD pour approvisionner et retirer automatiquement des comptes utilisateur d’Azure AD vers Concur.

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce didacticiel part du principe que vous disposez des éléments suivants :

*   Un locataire Azure Active Directory.
*   Un abonnement Concur pour lequel l’authentification unique est activée
*   Un compte d’utilisateur dans Concur avec les autorisations d’administrateur d’équipe

## <a name="assigning-users-to-concur"></a>Affectation d’utilisateurs à Concur

Azure Active Directory utilise un concept appelé « affectations » pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre de l’approvisionnement automatique des comptes d’utilisateur, seuls les utilisateurs et les groupes qui ont été « affectés » à une application dans Azure AD sont synchronisés.

Avant de configurer et d’activer le service d’approvisionnement, vous devez déterminer quels utilisateurs et/ou groupes dans Azure AD représentent les utilisateurs qui ont besoin d’accéder à votre application Concur. Une fois que vous avez choisi, vous pouvez affecter ces utilisateurs à votre application Concur en suivant les instructions fournies ici :

[Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-concur"></a>Conseils importants pour l’affectation d’utilisateurs à Concur

*   Nous vous recommandons de n’affecter qu’un seul utilisateur Azure AD à Concur pour tester la configuration de l’approvisionnement. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

*   Quand vous affectez un utilisateur à Concur, vous devez sélectionner un rôle d’utilisateur valide. Le rôle « Accès par défaut » ne fonctionne pas pour l’approvisionnement.

## <a name="enable-user-provisioning"></a>Activer l’approvisionnement d’utilisateurs

Cette section explique comment connecter votre annuaire Azure AD à l’API d’approvisionnement de comptes d’utilisateur de Concur pour créer, mettre à jour et désactiver les comptes d’utilisateur affectés dans Concur en fonction des attributions d’utilisateurs et de groupes dans Azure AD.

> [!Tip] 
> Vous pouvez également choisir d’activer l’authentification unique basée sur SAML pour Concur en suivant les instructions fournies dans le [portail Azure](https://portal.azure.com). L’authentification unique peut être configurée indépendamment de l’approvisionnement automatique, bien que chacune de ces deux fonctionnalités compléte l’autre.

### <a name="to-configure-user-account-provisioning"></a>Pour configurer l’approvisionnement de comptes utilisateur :

Cette section décrit comment activer l’approvisionnement des comptes d’utilisateurs Active Directory sur Concur.

Pour activer des applications dans le service Dépenses, vous devez vous assurer que la bonne configuration est définie et qu’un profil d’administrateur des services Web est utilisé. N’ajoutez pas le rôle WS Admin au profil d’administrateur existant que vous utilisez pour les fonctions administratives T&E.

Concur Consultants ou l’administrateur des clients doit créer un profil d’administrateur des services web distinct, et l’administrateur des clients doit utiliser ce profil pour les fonctions d’administrateur des services web (par exemple pour l’activation d’applications). Ces profils doivent être séparés du profil d’administrateur T&E utilisé quotidiennement par l’administrateur des clients (le profil d’administrateur T&E ne doit pas se voir affecter le rôle WS Admin).

Quand vous créez le profil à utiliser pour l’activation de l’application, entrez le nom de l’administrateur des clients dans les champs du profil utilisateur. Cela permet d’affecter la propriété au profil. Une fois qu’un ou plusieurs profils ont été créés, le client doit se connecter avec ce profil pour pouvoir cliquer sur le bouton «  *Activer*  » d’une application partenaire dans le menu Services Web.

Cette action ne doit pas être exécutée avec le profil utilisé pour l’administration T&E normale, et ce pour les raisons suivantes.

* Le client doit être celui qui clique sur *Oui* dans la boîte de dialogue qui s’affiche quand une application est activée. Ce clic signifie que le client accepte que l’application partenaire accède aux données. Par conséquent, vous ou le partenaire ne pouvez pas cliquer sur le bouton Oui.

* Si un administrateur de clients ayant activé une application à l’aide du profil d’administrateur T&E quitte l’entreprise (entraînant la désactivation du profil), toute application activée à l’aide de ce profil cesse de fonctionner jusqu’à son activation au moyen d’un autre profil WS Admin actif. C’est pourquoi vous êtes censé créer des profils WS Admin distincts.

* Si un administrateur quitte l’entreprise, le nom associé au profil WS Admin peut être remplacé par celui du nouvel administrateur sans affecter l’application activée, puisque le profil n’a pas besoin d’être désactivé.

**Pour configurer l'approvisionnement des utilisateurs, procédez comme suit :**

1. Connectez-vous à votre locataire **Concur**.

2. Dans le menu **Administration** , sélectionnez **Web Services**.
   
    ![Locataire Concur](./media/concur-provisioning-tutorial/IC721729.png "Client Concur")

3. Sur le côté gauche, dans le volet **Web Services** , sélectionnez **Enable Partner Application**.
   
    ![Enable Partner Application](./media/concur-provisioning-tutorial/ic721730.png "Enable Partner Application")

4. Dans la liste **Enable Application** , sélectionnez **Azure Active Directory** , puis cliquez sur **Enable**.
   
    ![Microsoft Azure Active Directory](./media/concur-provisioning-tutorial/ic721731.png "Microsoft Azure Active Directory")

5. Cliquez sur **Yes** pour fermer la boîte de dialogue **Confirm Action**.
   
    ![Confirm Action](./media/concur-provisioning-tutorial/ic721732.png "Confirm Action")

6. Sur le [portail Azure](https://portal.azure.com), accédez à la section **Azure Active Directory > Applications d’entreprise > Toutes les applications**.

7. Si vous avez déjà configuré Concur pour l’authentification unique, recherchez votre instance de Concur à l’aide du champ de recherche. Sinon, sélectionnez **Ajouter** et recherchez **Concur** dans la galerie d’applications. Sélectionnez Concur dans les résultats de recherche et ajoutez-la à votre liste d’applications.

8. Sélectionnez votre instance de Concur, puis sélectionnez l’onglet **Approvisionnement**.

9. Définissez le **Mode d’approvisionnement** sur **Automatique**. 
 
    ![Capture d’écran de l’onglet Provisionnement pour Concur dans le portail Azure. Le mode de provisionnement est défini sur Automatique et le bouton Tester la connexion est en évidence.](./media/concur-provisioning-tutorial/provisioning.png)

10. Dans la section **Informations d’identification administrateur** , entrez le **nom d’utilisateur** et le **mot de passe** de l’administrateur Concur.

11. Dans le portail Azure, cliquez sur **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à votre application Concur. Si la connexion échoue, vérifiez que votre compte Concur dispose des autorisations d’administrateur d’équipe.

12. Entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement dans le champ **E-mail de notification** , puis cochez la case.

13. Cliquez sur **Enregistrer.**

14. Dans la section Mappages, sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec Concur**.

15. Dans la section **Mappages des attributs** , passez en revue les attributs utilisateur qui sont synchronisés d’Azure AD vers Concur. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes d’utilisateur dans Concur pour les opérations de mise à jour. Cliquez sur le bouton Enregistrer pour valider les modifications.

16. Pour activer le service d’approvisionnement Azure AD pour Concur, affectez la valeur **Activé** au paramètre **Statut d’approvisionnement** dans la section **Paramètres**.

17. Cliquez sur **Enregistrer.**

Vous pouvez à présent créer un compte de test. Patientez jusqu’à 20 minutes et vérifiez que le compte a bien été synchronisé avec Concur.

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](tutorial-list.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)
* [Configurer l’authentification unique](concur-tutorial.md)
---
title: 'Tutoriel : Configurer Salesforce pour l’approvisionnement automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez les étapes à effectuer dans Salesforce et Azure AD pour approvisionner et déprovisionner automatiquement des comptes d’utilisateur d’Azure AD vers Salesforce.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/01/2019
ms.author: jeedes
ms.openlocfilehash: 2a4d3f3f9465b8813cdf6ee26760d819d73a08c1
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94353102"
---
# <a name="tutorial-configure-salesforce-for-automatic-user-provisioning"></a>Tutoriel : Configurer Salesforce pour l’approvisionnement automatique d’utilisateurs

L’objectif de ce didacticiel est de montrer les étapes à effectuer dans Salesforce et Azure AD pour approvisionner et retirer automatiquement des comptes utilisateur d’Azure AD vers Salesforce.

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce didacticiel part du principe que vous disposez des éléments suivants :

* Un locataire Azure Active Directory
* Un locataire Salesforce.com

> [!IMPORTANT]
> Si vous utilisez un compte d’essai Salesforce.com, vous ne pouvez pas configurer l’approvisionnement automatique des utilisateurs. Les comptes d’essai n’ont pas l’accès d’API requis tant qu’ils ne sont pas achetés. Vous pouvez contourner cette limitation en utilisant un [compte de développeur](https://developer.salesforce.com/signup) gratuit pour suivre ce didacticiel.

Si vous utilisez un environnement Salesforce Sandbox, veuillez consulter la page [Didacticiel : intégration d’Azure Active Directory à Salesforce Sandbox](./salesforce-sandbox-tutorial.md).

## <a name="assigning-users-to-salesforce"></a>Assigner des utilisateurs à Salesforce

Azure Active Directory utilise un concept appelé « affectations » pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre de l’approvisionnement automatique des comptes d’utilisateur, seuls les utilisateurs et les groupes qui ont été « affectés » à une application dans Azure AD sont synchronisés.

Avant de configurer et d’activer le service de provisionnement, vous devez déterminer quels utilisateurs ou groupes dans Azure AD ont besoin d’accéder à votre application Salesforce. Une fois cette décision prise, vous pouvez affecter ces utilisateurs à votre application Salesforce en suivant les instructions fournies dans [Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-salesforce"></a>Conseils importants concernant l’assignation d’utilisateurs à Salesforce

* Il est recommandé de n’assigner qu’un seul utilisateur Azure AD à Salesforce afin de tester la configuration de l’approvisionnement. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

* Lorsque vous assignez un utilisateur à Salesforce, vous devez sélectionner un rôle d’utilisateur valide. Le rôle « Accès par défaut » ne fonctionne pas pour l’approvisionnement

    > [!NOTE]
    > Cette application importe des rôles à partir de Salesforce dans le cadre du processus d’approvisionnement. Il est possible que le client souhaite sélectionner ce processus lors de l’assignation des utilisateurs dans Azure AD. Veuillez noter que les profils qui sont importés à partir de Salesforce apparaissent en tant que rôles dans Azure AD.

## <a name="enable-automated-user-provisioning"></a>Activer l’approvisionnement automatique des utilisateurs

Cette section va vous guider afin de connecter votre instance Azure AD à votre [compte utilisateur Salesforce qui fournit l’API - v40](https://developer.salesforce.com/docs/atlas.en-us.208.0.api.meta/api/implementation_considerations.htm). À l’aide de ce guide, vous pourrez aussi découvrir comment configurer le service d’approvisionnement afin de créer, mettre à jour et désactiver des comptes utilisateur assignés dans Salesforce, en fonction des attributions d’utilisateur et de groupe dans Azure AD.

> [!Tip]
> Vous pouvez également choisir d’activer l’authentification unique basée sur SAML pour Salesforce, grâce aux instructions disponibles dans le [portail Azure](https://portal.azure.com). L’authentification unique peut être configurée indépendamment de l’approvisionnement automatique, bien que chacune de ces deux fonctionnalités compléte l’autre.

### <a name="configure-automatic-user-account-provisioning"></a>Configurer l’approvisionnement automatique d’un compte utilisateur

Cette section décrit comment activer l’approvisionnement des utilisateurs des comptes d’utilisateurs Active Directory sur Salesforce.

1. Sur le [portail Azure](https://portal.azure.com), accédez à la section **Azure Active Directory > Applications d’entreprise > Toutes les applications**.

2. Si vous avez déjà configuré Salesforce pour l’authentification unique, recherchez votre instance de Salesforce à l’aide du champ de recherche. Sinon, sélectionnez **Ajouter** et effectuer une recherche pour **Salesforce** dans la galerie d’applications. Dans les résultats de la recherche, sélectionnez Salesforce, puis ajoutez-le à votre liste d’applications.

3. Sélectionnez votre instance de Salesforce, puis sélectionnez l’onglet **Approvisionnement**.

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran montrant la page d’approvisionnement Salesforce, avec le Mode d’approvisionnement défini sur Automatique et d’autres valeurs que vous pouvez définir.](./media/salesforce-provisioning-tutorial/provisioning.png)

5. Dans la section **Informations d’identification de l’administrateur** , fournissez les paramètres de configuration suivants :

    a. Dans la zone de texte **Nom d’utilisateur administrateur** , saisissez le nom d’un compte Salesforce auquel le profil **Administrateur système** est assigné dans Salesforce.com.

    b. Dans la zone de texte **Mot de passe d’administrateur** , entrez le mot de passe de ce compte.

6. Pour obtenir le jeton de sécurité Salesforce, ouvrez un nouvel onglet et connectez-vous au même compte d’administration Salesforce. Dans le coin supérieur droit de la page, cliquez sur votre nom, puis cliquez sur **Paramètres**.

    ![Capture d’écran montrant le lien Paramètres sélectionné.](./media/salesforce-provisioning-tutorial/sf-my-settings.png "Activer l'approvisionnement de l'utilisateur automatique.")

7. Dans le volet de navigation gauche, cliquez sur **Mes informations personnelles** pour développer la section associée, puis sur **Réinitialiser mon jeton de sécurité**.
  
    ![Capture d’écran montrant Réinitialiser mon jeton de sécurité sélectionné à partir de Mes informations personnelles.](./media/salesforce-provisioning-tutorial/sf-personal-reset.png "Activer l'approvisionnement de l'utilisateur automatique.")

8. Sur la page **Réinitialiser le jeton de sécurité** , cliquez sur **Réinitialiser le jeton de sécurité**.

    ![La capture d’écran montre la page Jeton de sécurité REST, avec du texte explicatif et l’option de réinitialisation du jeton de sécurité](./media/salesforce-provisioning-tutorial/sf-reset-token.png "Activer l'approvisionnement de l'utilisateur automatique.")

9. Contrôlez la boîte de réception associée à ce compte d’administrateur. Recherchez un message électronique provenant de Salesforce.com qui contient le nouveau jeton de sécurité.

10. Copiez le jeton, accédez à votre fenêtre Azure AD et collez-le dans le champ **Jeton secret**.

11. L’ **URL de locataire** doit être entrée si l’instance de Salesforce se trouve sur Salesforce Government Cloud. Dans le cas contraire, elle est facultative. Entrez l’URL de locataire en utilisant le format « https://\<your-instance\>.my.salesforce.com », où vous devez remplacer \<your-instance\> par le nom de votre instance Salesforce.

12. Dans le portail Azure, cliquez sur **Tester la connexion** pour vous assurer qu’Azure AD peut se connecter à votre application Salesforce.

13. Dans le champ **E-mail de notification** , saisissez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement, puis cochez la case.

14. Cliquez sur **Enregistrer.**  

15. Dans la section Mappages, sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec Salesforce.**

16. Dans la section **Mappages d’attributs** , passez en revue les attributs utilisateur qui sont synchronisés à partir d’Azure AD vers Salesforce. Remarque : Les attributs sélectionnés en tant que propriétés de **Correspondance** servent à faire correspondre les comptes utilisateur dans Salesforce, en vue d’opérations de mise à jour. Cliquez sur le bouton Enregistrer pour valider les modifications.

17. Pour activer le service d’approvisionnement Azure AD pour Salesforce, accédez à la section Paramètres et définissez le **Statut de l’approvisionnement** sur **Activé**.

18. Cliquez sur **Enregistrer.**

> [!NOTE]
> Une fois les utilisateurs approvisionnés dans l’application Salesforce, l’administrateur doit configurer les paramètres propres à leur langue. Pour plus d' informations sur la configuration de la langue, consultez [cet](https://help.salesforce.com/articleView?id=setting_your_language.htm&type=5) article.

Cette commande démarre la synchronisation initiale des utilisateurs et/ou des groupes affectés à Salesforce dans la section Utilisateurs et Groupes. Notez que la synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service est en cours d’exécution. Vous pouvez utiliser la section **Détails de la synchronisation** pour surveiller la progression et suivre les liens vers les journaux d’activité de provisionnement, qui décrivent toutes les actions effectuées par le service de provisionnement dans votre application Salesforce.

Pour plus d’informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="common-issues"></a>Problèmes courants
* Si vous rencontrez des problèmes pour autoriser l'accès à Salesforce, vérifiez les points suivants :
    * Les informations d'identification utilisées disposent d’un accès administrateur à Salesforce.
    * La version de Salesforce que vous utilisez prend en charge l'accès Web (par exemple, les versions Developer, Enterprise, Sandbox et Unlimited de Salesforce).
    * L'accès à l'API Web est activé pour l'utilisateur.
* Le service d’approvisionnement Azure AD prend en charge la langue, les paramètres régionaux et le fuseau horaire d’approvisionnement pour un utilisateur. Ces attributs figurent dans les mappages d'attributs par défaut mais n'ont pas d'attribut source par défaut. Assurez-vous que vous sélectionnez l'attribut source par défaut et que l'attribut source est dans le format attendu par SalesForce. Par exemple, localeSidKey pour english(UnitedStates) est en_US. Passez en revue les conseils fournis [ici](https://help.salesforce.com/articleView?id=setting_your_language.htm&type=5) pour déterminer le format localeSidKey approprié. Les formats languageLocaleKey se trouvent [ici](https://help.salesforce.com/articleView?id=faq_getstart_what_languages_does.htm&type=5). En plus de vous assurer que le format est correct, vous devrez peut-être vérifier que la langue est activée pour vos utilisateurs, comme décrit [ici](https://help.salesforce.com/articleView?id=setting_your_language.htm&type=5). 
* **SalesforceLicenseLimitExceeded :** L’utilisateur n’a pas pu être créé dans l’application cible, car il n’existe aucune licence disponible pour cet utilisateur. Procurez-vous des licences supplémentaires pour l’application cible, ou passez en revue vos attributions d’utilisateurs et la configuration de mappage des attributs pour vous assurer que les utilisateurs appropriés sont affectés avec les attributs appropriés.
* **SalesforceDuplicateUserName :** L'utilisateur ne peut pas être approvisionné car il dispose d'un attribut 'Username' Salesforce.com qui est dupliqué dans un autre locataire Salesforce.com.  Dans Salesforce.com, les valeurs de l'attribut 'Username' doivent être uniques pour tous les locataires Salesforce.com.  Par défaut, l’attribut userPrincipalName d'un utilisateur dans Azure Active Directory devient son attribut 'Username' dans Salesforce.com.   Vous avez le choix entre deux options.  Une option consiste à trouver et à renommer l'utilisateur avec le doublon 'Username' dans l'autre locataire Salesforce.com, si vous administrez également cet autre locataire.  L'autre option consiste à supprimer l'accès de l'utilisateur Azure Active Directory au locataire Salesforce.com auquel votre répertoire est intégré. Nous réessayerons cette opération lors de la prochaine tentative de synchronisation. 
* **SalesforceRequiredFieldMissing :** Salesforce exige que certains attributs soient présents sur l'utilisateur pour créer ou mettre à jour avec succès l’utilisateur. Il manque à cet utilisateur un des attributs requis. Assurez-vous que tous les utilisateurs que vous souhaitez approvisionner dans Salesforce possèdent des attributs tels que l'e-mail et l'alias. Vous pouvez définir l’étendue des utilisateurs qui n'ont pas ces attributs à l'aide de [filtres d’étendue basés sur des attributs](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 
* Le mappage d'attributs par défaut pour l’approvisionnement dans Salesforce inclut l'expression SingleAppRoleAssignments permettant de mapper appRoleAssignments dans Azure AD à ProfileName dans Salesforce. Assurez-vous que les utilisateurs n'ont pas plusieurs affectations de rôles d'applications dans Azure AD car le mappage d’attributs ne prend en charge qu'un seul rôle. 
* Salesforce requiert l’approbation manuelle des mises à jour de la messagerie électronique avant leur modification. Par conséquent, vous pouvez voir plusieurs entrées dans les journaux d’approvisionnement pour mettre à jour la messagerie de l’utilisateur (jusqu’à ce que la modification de la messagerie ait été approuvée).


## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](tutorial-list.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)
* [Configurer l’authentification unique](./salesforce-tutorial.md)
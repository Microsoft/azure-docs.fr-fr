---
title: 'Tutoriel : Configurer le provisionnement entrant SuccessFactors dans Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer le provisionnement entrant de SuccessFactors vers Azure AD.
services: active-directory
author: cmmdesai
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.topic: tutorial
ms.workload: identity
ms.date: 08/05/2020
ms.author: chmutali
ms.openlocfilehash: a62943c1a808424ded1a5e46ed115cda332bf7d5
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96020753"
---
# <a name="tutorial-configure-sap-successfactors-to-azure-ad-user-provisioning"></a>Tutoriel : Configurer l’approvisionnement d’utilisateur SAP SuccessFactors vers Azure AD
L’objectif de ce tutoriel est de présenter les étapes à effectuer pour provisionner les données utilisateurs de SuccessFactors Employee Central dans Azure Active Directory, avec réécriture facultative de l’adresse e-mail dans SuccessFactors. 

>[!NOTE]
>Utilisez ce tutoriel si les utilisateurs que vous souhaitez provisionner à partir de SuccessFactors sont des utilisateurs cloud uniquement qui n’ont pas besoin de compte AD local. Si les utilisateurs n’ont besoin que d’un compte AD local ou ont besoin d’un compte AD et d’un compte Azure AD, consultez le tutoriel sur la façon de [configurer le provisionnement d’utilisateurs SAP SuccessFactors vers Active Directory](sap-successfactors-inbound-provisioning-tutorial.md#overview). 

## <a name="overview"></a>Vue d’ensemble

Le [service de provisionnement d’utilisateurs Azure Active Directory](../app-provisioning/user-provisioning.md) s’intègre à [SuccessFactors Central Employee](https://www.successfactors.com/products-services/core-hr-payroll/employee-central.html) afin de gérer le cycle de vie des identités des utilisateurs. 

Les workflows de provisionnement d’utilisateurs SuccessFactors pris en charge par le service de provisionnement d’utilisateurs Azure AD permettent d’automatiser les scénarios de gestion du cycle de vie des identités et des ressources humaines suivants :

* **Nouvelles embauches** : quand un nouvel employé est ajouté à SuccessFactors, un compte d’utilisateur est créé automatiquement dans Azure Active Directory et éventuellement dans Microsoft 365 et d’[autres applications SaaS prises en charge par Azure AD](../app-provisioning/user-provisioning.md), avec réécriture de l’adresse e-mail dans SuccessFactors.

* **Mises à jour du profil et des attributs de l’employé** : quand le dossier d’un employé est mis à jour dans SuccessFactors (par exemple le nom, la fonction ou le responsable), son compte d’utilisateur est mis à jour automatiquement dans Azure Active Directory et éventuellement dans Microsoft 365 et d’[autres applications SaaS prises en charge par Azure AD](../app-provisioning/user-provisioning.md).

* **Résiliations de contrats d’employés** : quand le contrat d’un employé est résilié dans SuccessFactors, son compte d’utilisateur est désactivé automatiquement dans Azure Active Directory et éventuellement dans Microsoft 365 et d’[autres applications SaaS prises en charge par Azure AD](../app-provisioning/user-provisioning.md).

* **Employés réembauchés** : quand un employé est réembauché dans SuccessFactors, son ancien compte peut être réactivé ou reprovisionné automatiquement (en fonction de votre préférence) dans Azure Active Directory et éventuellement dans Microsoft 365 et d’[autres applications SaaS prises en charge par Azure AD](../app-provisioning/user-provisioning.md).

### <a name="who-is-this-user-provisioning-solution-best-suited-for"></a>À qui cette solution d’attribution d’utilisateurs convient-elle le mieux ?

Cette solution de provisionnement d’utilisateurs SuccessFactors vers Azure Active Directory est idéale pour :

* Les organisations qui souhaitent une solution cloud prédéfinie pour le provisionnement d’utilisateurs SuccessFactors.

* Les organisations qui ont besoin du provisionnement d’utilisateurs direct de SuccessFactors vers Azure Active Directory.

* Les organisations qui imposent le provisionnement d’utilisateurs avec des données provenant de [SuccessFactors Employee Central (EC)](https://www.successfactors.com/products-services/core-hr-payroll/employee-central.html).

* Organisations utilisant Microsoft 365 pour la messagerie

## <a name="solution-architecture"></a>Architecture de la solution

Cette section décrit l’architecture de la solution de provisionnement d’utilisateurs de bout en bout pour les utilisateurs du cloud uniquement. Il existe deux flux connexes :

* **Flux de données RH faisant autorité – de SuccessFactors vers Azure Active Directory :** dans ce flux, les événements concernant les employés (comme les nouveaux recrutements, les transferts, les fins de contrat) se produisent d’abord dans SuccessFactors Employee Central dans le cloud, puis les données des événements transitent dans Azure Active Directory. Selon l’événement, cela peut provoquer des opérations de création/mise à jour/activation/désactivation dans Azure AD.
* **Flux de réécriture des e-mails – d’Active Directory local vers SuccessFactors :** Une fois la création du compte terminée dans Azure Active Directory, la valeur de l’attribut email ou de l’UPN générée dans Azure AD peut être réécrite dans SuccessFactors.

  ![Vue d’ensemble](./media/sap-successfactors-inbound-provisioning/sf2aad-overview.png)

### <a name="end-to-end-user-data-flow"></a>Flux de données utilisateur de bout en bout

1. L’équipe RH effectue des transactions sur les employés (entrants/changements de poste/sortants ou recrutements/transferts/arrêts) dans SuccessFactors Employee Central
2. Le service de provisionnement Azure AD effectue des synchronisations planifiées des identités à partir de SuccessFactors EC et identifie les modifications qui doivent être traitées pour la synchronisation avec Active Directory local.
3. Le service de provisionnement Azure AD détermine la modification et appelle l’opération de création/mise à jour/activation/désactivation pour l’utilisateur dans Azure AD.
4. Si l’[application SuccessFactors Writeback](sap-successfactors-writeback-tutorial.md) est configurée, l’adresse e-mail de l’utilisateur est récupérée à partir d’Azure AD. 
5. Le service de provisionnement Azure AD écrit l’attribut email dans SuccessFactors, en fonction de l’attribut correspondant utilisé.

## <a name="planning-your-deployment"></a>Planification de votre déploiement

La configuration du provisionnement d’utilisateurs piloté par les RH dans le cloud de SuccessFactors vers Azure AD nécessite une planification minutieuse couvrant différents aspects tels que :

* Détermination de l’ID correspondant 
* Mappage d’attributs
* Transformation d’attributs 
* Filtres d’étendue

Pour obtenir des instructions complètes sur ces sujets, consultez le [plan de déploiement cloud des RH](../app-provisioning/plan-cloud-hr-provision.md). Reportez-vous à la [référence sur l’intégration de SAP SuccessFactors](../app-provisioning/sap-successfactors-integration-reference.md) pour en savoir plus sur les entités prises en charge, sur le traitement des détails et sur la personnalisation de l’intégration pour différents scénarios de ressources humaines. 

## <a name="configuring-successfactors-for-the-integration"></a>Configuration de SuccessFactors pour l’intégration

L’une des exigences communes de tous les connecteurs de provisionnement SuccessFactors est qu’ils nécessitent les informations d’identification d’un compte SuccessFactors avec les autorisations appropriées pour appeler les API OData SuccessFactors. Cette section décrit les étapes permettant de créer le compte de service dans SuccessFactors et d’accorder les autorisations appropriées. 

* [Créer/identifier un compte d’utilisateur d’API dans SuccessFactors](#createidentify-api-user-account-in-successfactors)
* [Créer un rôle d’autorisations d’API](#create-an-api-permissions-role)
* [Créer un groupe d’autorisations pour l’utilisateur des API](#create-a-permission-group-for-the-api-user)
* [Accorder le rôle d’autorisation au groupe d’autorisations](#grant-permission-role-to-the-permission-group)

### <a name="createidentify-api-user-account-in-successfactors"></a>Créer/identifier un compte d’utilisateur d’API dans SuccessFactors
Collaborez avec votre équipe d’administration SuccessFactors ou votre partenaire d’implémentation pour créer ou identifier un compte d’utilisateur dans SuccessFactors qui sera utilisé pour appeler les API OData. Les informations d’identification de nom d’utilisateur et de mot de passe de ce compte seront requises lors de la configuration des applications de provisionnement dans Azure AD. 

### <a name="create-an-api-permissions-role"></a>Créer un rôle d’autorisations d’API

* Connectez-vous à SAP SuccessFactors avec un compte d’utilisateur qui a accès au centre d’administration.
* Recherchez *Manage Permission Roles* (Gérer les rôles d’autorisations), puis sélectionnez **Manage Permission Roles** dans les résultats de la recherche.
  ![Gérer les rôles d’autorisations](./media/sap-successfactors-inbound-provisioning/manage-permission-roles.png)
* Dans la liste Permission Role (Rôle d’autorisation), cliquez sur **Create New** (Créer).
  > [!div class="mx-imgBorder"]
  > ![Créer un rôle d’autorisation](./media/sap-successfactors-inbound-provisioning/create-new-permission-role-1.png)
* Ajoutez un **nom de rôle** et une **description** pour le nouveau rôle d’autorisation. Le nom et la description doivent indiquer que le rôle est destiné aux autorisations d’utilisation des API.
  > [!div class="mx-imgBorder"]
  > ![Détail du rôle d’autorisation](./media/sap-successfactors-inbound-provisioning/permission-role-detail.png)
* Sous Permission settings (Paramètres d’autorisation), cliquez sur **Permission...** , faites défiler la liste des autorisations, puis cliquez sur **Manage Integration Tools** (Gérer les outils d’intégration). Cochez la case **Allow Admin to Access to OData API through Basic Authentication** (Autoriser l’administrateur à accéder à l’API OData via l’authentification de base).
  > [!div class="mx-imgBorder"]
  > ![Gérer les outils d’intégration](./media/sap-successfactors-inbound-provisioning/manage-integration-tools.png)
* Faites défiler la liste dans la même zone et sélectionnez **Employee Central API**. Ajoutez des autorisations comme indiqué ci-dessous pour lire à l’aide de l’API ODATA et modifier à l’aide de l’API ODATA. Sélectionnez l’option de modification si vous envisagez d’utiliser le même compte pour le scénario de réécriture vers SuccessFactors. 
  > [!div class="mx-imgBorder"]
  > ![Autorisations de lecture-écriture](./media/sap-successfactors-inbound-provisioning/odata-read-write-perm.png)
* Cliquez sur **Done** (Terminé). Cliquez sur **Enregistrer les modifications**.

### <a name="create-a-permission-group-for-the-api-user"></a>Créer un groupe d’autorisations pour l’utilisateur des API

* Dans le centre d’administration SuccessFactors, recherchez *Manage Permission Groups* (Gérer les groupes d’autorisations), puis sélectionnez **Manage Permission Groups** dans les résultats de la recherche.
  > [!div class="mx-imgBorder"]
  > ![Gérer les groupes d’autorisations](./media/sap-successfactors-inbound-provisioning/manage-permission-groups.png)
* Dans la fenêtre Manage Permission Groups, cliquez sur **Create New**.
  > [!div class="mx-imgBorder"]
  > ![Ajouter un nouveau groupe](./media/sap-successfactors-inbound-provisioning/create-new-group.png)
* Ajoutez un nom pour le nouveau groupe. Ce nom doit indiquer que le groupe est destiné aux utilisateurs des API.
  > [!div class="mx-imgBorder"]
  > ![Nom du groupe d’autorisations](./media/sap-successfactors-inbound-provisioning/permission-group-name.png)
* Ajoutez des membres au groupe. Par exemple, vous pouvez sélectionner **Username** dans le menu déroulant People Pool, puis entrer le nom d’utilisateur du compte d’API qui sera utilisé pour l’intégration. 
  > [!div class="mx-imgBorder"]
  > ![Add group members](./media/sap-successfactors-inbound-provisioning/add-group-members.png) (Ajouter des membres à un groupe)
* Cliquez sur **Done** pour terminer la création du groupe d’autorisations.

### <a name="grant-permission-role-to-the-permission-group"></a>Accorder le rôle d’autorisation au groupe d’autorisations

* Dans le centre d’administration SuccessFactors, recherchez *Manage Permission Roles*, puis sélectionnez **Manage Permission Roles** dans les résultats de la recherche.
* Dans la liste **Permission Role List** (Liste des rôles d’autorisations), sélectionnez le rôle que vous avez créé pour les autorisations d’utilisation des API.
* Sous **Grant this role to...** (Attribuer ce rôle à), cliquez sur le bouton **Add...** (Ajouter).
* Sélectionnez **Permission Group...** (Groupe d’autorisations) dans le menu déroulant, puis cliquez sur **Select...** (Sélectionner) pour ouvrir la fenêtre Groups afin de rechercher et sélectionner le groupe créé ci-dessus. 
  > [!div class="mx-imgBorder"]
  > ![Ajouter un groupe d’autorisations](./media/sap-successfactors-inbound-provisioning/add-permission-group.png)
* Passez en revue l’octroi de rôle d’autorisation au groupe d’autorisations. 
  > [!div class="mx-imgBorder"]
  > ![Détails du rôle et du groupe d’autorisations](./media/sap-successfactors-inbound-provisioning/permission-role-group.png)
* Cliquez sur **Enregistrer les modifications**.

## <a name="configuring-user-provisioning-from-successfactors-to-azure-ad"></a>Configuration du provisionnement d’utilisateurs de SuccessFactors vers Azure AD

Cette section décrit les étapes du provisionnement de comptes d’utilisateur de SuccessFactors vers Azure AD.

* [Ajouter l’application du connecteur de provisionnement et configurer la connectivité à SuccessFactors](#part-1-add-the-provisioning-connector-app-and-configure-connectivity-to-successfactors).
* [Configurer les mappages d'attributs](#part-2-configure-attribute-mappings)
* [Activer et lancer l'approvisionnement des utilisateurs](#enable-and-launch-user-provisioning)

### <a name="part-1-add-the-provisioning-connector-app-and-configure-connectivity-to-successfactors"></a>Première partie : Ajouter l’application du connecteur de provisionnement et configurer la connectivité à SuccessFactors

**Pour configurer le provisionnement SuccessFactors vers Azure AD**

1. Accédez à <https://portal.azure.com>

2. Dans la barre de navigation de gauche, sélectionnez **Azure Active Directory**

3. Cliquez sur **Applications d’entreprise**, puis sur **Toutes les applications**.

4. Sélectionnez **Ajouter une application**, puis sélectionnez la catégorie **Tous**.

5. Recherchez **SuccessFactors to Azure Active Directory User Provisioning** et ajoutez cette application à partir de la galerie.

6. Une fois l’application ajoutée et l’écran de détails de l’application affiché, sélectionnez **Approvisionnement**

7. Définissez **Mode** **d'approvisionnement** sur **Automatique**

8. Fermez la section **Informations d’identification de l’administrateur**, comme suit :

   * **Nom de l’utilisateur administrateur** : entrez le nom d’utilisateur du compte d’utilisateur de l’API SuccessFactors, accompagné de l’ID d’entreprise. Son format est le suivant : **nom_utilisateur\@ID_société**

   * **Mot de passe d’administrateur** : entrez le mot de passe du compte d’utilisateur de l’API SuccessFactors. 

   * **URL du locataire** : entrez le nom du point de terminaison des services de l’API OData SuccessFactors. Entrez uniquement le nom d’hôte du serveur, sans http ou https. Cette valeur doit ressembler à ceci : **nom_serveur_API.successfactors.com**.

   * **E-mail de notification :** entrez votre adresse e-mail et activez la case à cocher « Envoyer un e-mail en cas de défaillance ».
    > [!NOTE]
    > Le service Azure AD Provisioning envoie la notification par e-mail si le travail de provisionnement passe à l’état [Mise en quarantaine](../app-provisioning/application-provisioning-quarantine-status.md).

   * Cliquez sur le bouton **Tester la connexion**. Si le test de connexion aboutit, cliquez sur le bouton **Enregistrer**, en haut de l'écran. En cas d’échec, vérifiez que les informations d’identification et l’URL SuccessFactors sont valides.
    >[!div class="mx-imgBorder"]
    >![Azure portal](./media/sap-successfactors-inbound-provisioning/sf2aad-provisioning-creds.png)

   * Une fois les informations d’identification enregistrées, la section **Mappages** affiche le mappage par défaut **Synchroniser les utilisateurs de SuccessFactors vers Azure Active Directory**.

### <a name="part-2-configure-attribute-mappings"></a>Deuxième partie : Configuration des mappages d’attributs

Dans cette section, vous allez configurer le flux des données utilisateur de SuccessFactors vers Active Directory.

1. Sous l’onglet Provisionnement, sous **Mappages**, cliquez sur **Synchroniser les utilisateurs de SuccessFactors vers Azure Active Directory**.

1. Dans le champ **Portée de l’objet source**, vous pouvez sélectionner les ensembles d’utilisateurs de SuccessFactors concernés par le provisionnement dans Azure AD, en définissant des filtres basés sur des attributs. L’étendue par défaut est « tous les utilisateurs de SuccessFactors ». Exemples de filtres :

   * Exemple : Étendue pour les utilisateurs avec un personIdExternal compris entre 1000000 et 2000000 (2000000 exclus)

      * Attribut : personIdExternal

      * Opérateur : REGEX Match

      * Valeur : (1[0-9][0-9][0-9][0-9][0-9][0-9])

   * Exemple : Uniquement les employés et non les employés occasionnels

      * Attribut : EmployeeID

      * Opérateur : IS NOT NULL

   > [!TIP]
   > Lors de la configuration initiale de l'application d'approvisionnement, vous devez tester et vérifier vos mappages d'attributs et expressions pour être sûr d'obtenir le résultat souhaité. Microsoft vous recommande d’utiliser les filtres d’étendue disponibles sous **Portée de l’objet source** pour tester vos mappages dans SuccessFactors avec quelques utilisateurs test. Après avoir vérifié que les mappages fonctionnent, vous pouvez supprimer le filtre ou l'étendre progressivement pour inclure d'autres utilisateurs.

   > [!CAUTION] 
   > Par défaut, le moteur de provisionnement désactive/supprime les utilisateurs qui sortent de l’étendue. Vous pouvez juger ce comportement inopportun dans votre intégration de SuccessFactors à Azure AD. Pour remplacer ce comportement par défaut, consultez l’article [Ignorer la suppression des comptes d’utilisateurs qui sortent de l’étendue](../app-provisioning/skip-out-of-scope-deletions.md).
  
1. Dans le champ **Actions de l'objet cible**, vous pouvez filtrer globalement les actions exécutées sur Active Directory. Les actions **Créer** et **Mettre à jour** sont les plus courantes.

1. Dans la section **Mappages d’attributs**, vous pouvez définir la manière dont les différents attributs SuccessFactors sont mappés aux attributs Active Directory.

  >[!NOTE]
  >Pour obtenir la liste complète des attributs SuccessFactors pris en charge par l’application, consultez la [Référence des attributs SuccessFactors](../app-provisioning/sap-successfactors-attribute-reference.md).


1. Cliquez sur un mappage d’attributs existants à mettre à jour ou cliquez sur **Ajouter un nouveau mappage** en bas de l’écran pour ajouter de nouveaux mappages. Un mappage d’attribut individuel prend en charge les propriétés suivantes :

      * **Type de mappage**

         * **Direct** : inscrit la valeur de l’attribut SuccessFactors dans l’attribut AD, sans aucune modification

         * **Constante** : inscrivez une valeur de chaîne constante et statique dans l'attribut AD

         * **Expression** : vous permet d’écrire une valeur personnalisée dans l’attribut AD, basée sur un ou plusieurs attributs SuccessFactors. [Pour plus d’informations, consultez l’article sur les expressions](../app-provisioning/functions-for-customizing-application-data.md).

      * **Attribut source** : l’attribut utilisateur dans SuccessFactors.

      * **Valeur par défaut** : facultatif. Si l’attribut source a une valeur vide, le mappage écrit cette valeur à la place.
            La configuration la plus courante consiste à laisser ce champ vide.

      * **Attribut cible** : attribut de l'utilisateur dans Active Directory.

      * **Faire correspondre des objets à l’aide de cet attribut** : indique ci ce mappage est utilisé ou pas pour identifier les utilisateurs de manière unique entre SuccessFactors et Active Directory. Cette valeur est communément définie dans le champ ID collaborateur de SuccessFactors, qui est généralement mappé avec l’un des attributs ID employé d’Active Directory.

      * **Priorité des correspondances** : plusieurs attributs de correspondance peuvent être définis. S’il en existe plusieurs, ils sont évalués dans l’ordre défini par ce champ. Dès qu'une correspondance est trouvée, aucun autre attribut de correspondance n'est évalué.

      * **Appliquer ce mappage**

         * **Toujours** : applique ce mappage à la création de l’utilisateur et des actions de mise à jour.

         * **Lors de la création uniquement** : applique ce mappage uniquement aux actions de création d’utilisateur.

1. Pour enregistrer vos mappages, cliquez sur **Enregistrer** en haut de la section Mappage d'attributs.

Une fois vos mappages d'attributs configurés, vous pouvez [activer et lancer le service d'approvisionnement d'utilisateurs](#enable-and-launch-user-provisioning).

## <a name="enable-and-launch-user-provisioning"></a>Activer et lancer l'approvisionnement des utilisateurs

Une fois les configurations d’application de provisionnement SuccessFactors effectuées, vous pouvez activer le service de provisionnement dans le portail Azure.

> [!TIP]
> Par défaut, lorsque vous activez le service d'approvisionnement, il lance les opérations d'approvisionnement pour tous les utilisateurs concernés. En cas d'erreur de mappage ou de problème lié aux données Workday, le travail d'approvisionnement peut échouer et être mis en quarantaine. Pour éviter ce genre de problème, nous vous recommandons de configurer le filtre **Portée de l'objet source** et de tester vos mappages d'attributs sur quelques utilisateurs test avant de lancer la synchronisation complète de tous les utilisateurs. Après avoir vérifié que les mappages fonctionnent et qu'ils vous donnent les résultats souhaités, vous pouvez supprimer le filtre ou l'étendre progressivement pour inclure d'autres utilisateurs.

1. Dans l’onglet **Approvisionnement**, définissez **État d’approvisionnement** sur **Activé**.

2. Cliquez sur **Enregistrer**.

3. Cette opération permet de lancer la synchronisation initiale, dont la durée dépendra du nombre d’utilisateurs du locataire SuccessFactors. Vous pouvez consulter la barre de progression pour suivre la progression du cycle de synchronisation. 

4. À tout moment, consultez l’onglet **Journaux d’audit** dans le portail Azure pour connaître les actions effectuées par le service d’approvisionnement. Les journaux d’audit répertorient tous les événements de synchronisation individuels effectués par le service d’approvisionnement, tels que les utilisateurs lus dans Workday et par la suite ajoutés ou mis à jour dans Active Directory. 

5. Au terme de la synchronisation initiale, un rapport de synthèse d'audit est créé dans l'onglet **Approvisionnement**, comme illustré ci-dessous.

   > [!div class="mx-imgBorder"]
   > ![Barre de progression de provisionnement](./media/sap-successfactors-inbound-provisioning/prov-progress-bar-stats.png)

## <a name="next-steps"></a>Étapes suivantes

* [En savoir plus sur les attributs SuccessFactors pris en charge pour le provisionnement entrant](../app-provisioning/sap-successfactors-attribute-reference.md)
* [Découvrez comment configurer la réécriture d’e-mail vers SuccessFactors](sap-successfactors-writeback-tutorial.md)
* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)
* [Découvrez comment configurer l’authentification unique entre SuccessFactors et Azure Active Directory](successfactors-tutorial.md)
* [Découvrir comment intégrer d’autres applications SaaS à Azure Active Directory](tutorial-list.md)
* [Découvrez comment exporter et importer vos configurations de provisionnement](../app-provisioning/export-import-provisioning-configuration.md)
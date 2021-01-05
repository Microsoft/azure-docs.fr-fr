---
title: 'Tutoriel : Configurer l’approvisionnement entrant SuccessFactors dans AD et Azure AD | Microsoft Docs'
description: Découvrez comment configurer le provisionnement entrant à partir de SuccessFactors
services: active-directory
author: cmmdesai
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.topic: tutorial
ms.workload: identity
ms.date: 08/05/2020
ms.author: chmutali
ms.openlocfilehash: 53707261070e8efbd014614ee700df63a0925ef8
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95999703"
---
# <a name="tutorial-configure-sap-successfactors-to-active-directory-user-provisioning"></a>Tutoriel : Configurer l’approvisionnement d’utilisateurs SAP SuccessFactors vers Active Directory 
L’objectif de ce tutoriel est de présenter les étapes à effectuer pour provisionner les utilisateurs de SuccessFactors Employee Central dans Active Directory (AD) et Azure AD, avec réécriture facultative de l’adresse e-mail dans SuccessFactors. 

>[!NOTE]
>Utilisez ce tutoriel si les utilisateurs que vous souhaitez provisionner à partir de SuccessFactors ont besoin d’un compte AD local et éventuellement d’un compte Azure AD. Si les utilisateurs de SuccessFactors ont seulement besoin d’un compte Azure AD (utilisateurs cloud uniquement), consultez le tutoriel sur la [configuration du provisionnement d’utilisateurs de SAP SuccessFactors vers Azure AD](sap-successfactors-inbound-provisioning-cloud-only-tutorial.md). 


## <a name="overview"></a>Vue d’ensemble

Le [service de provisionnement d’utilisateurs Azure Active Directory](../app-provisioning/user-provisioning.md) s’intègre à [SuccessFactors Central Employee](https://www.successfactors.com/products-services/core-hr-payroll/employee-central.html) afin de gérer le cycle de vie des identités des utilisateurs. 

Les workflows de provisionnement d’utilisateurs SuccessFactors pris en charge par le service de provisionnement d’utilisateurs Azure AD permettent d’automatiser les scénarios de gestion du cycle de vie des identités et des ressources humaines suivants :

* **Nouvelles embauches** : quand un nouvel employé est ajouté à SuccessFactors, un compte d’utilisateur est créé automatiquement dans Active Directory, Azure Active Directory et éventuellement dans Microsoft 365 et d’[autres applications SaaS prises en charge par Azure AD](../app-provisioning/user-provisioning.md), avec réécriture de l’adresse e-mail dans SuccessFactors.

* **Mises à jour du profil et des attributs de l’employé** : quand le dossier d’un employé est mis à jour dans SuccessFactors (par exemple le nom, la fonction ou le responsable), son compte d’utilisateur est mis à jour automatiquement dans Active Directory, Azure Active Directory et éventuellement dans Microsoft 365 et d’[autres applications SaaS prises en charge par Azure AD](../app-provisioning/user-provisioning.md).

* **Résiliations de contrats d’employés** : quand le contrat d’un employé est résilié dans SuccessFactors, son compte d’utilisateur est désactivé automatiquement dans Active Directory, Azure Active Directory et éventuellement dans Microsoft 365 et d [’autres applications SaaS prises en charge par Azure AD](../app-provisioning/user-provisioning.md).

* **Employés réembauchés** : quand un employé est réembauché dans SuccessFactors, son ancien compte peut être réactivé ou reprovisionné automatiquement (en fonction de votre préférence) dans Active Directory, Azure Active Directory et éventuellement dans Microsoft 365 et d’[autres applications SaaS prises en charge par Azure AD](../app-provisioning/user-provisioning.md).

### <a name="who-is-this-user-provisioning-solution-best-suited-for"></a>À qui cette solution d’attribution d’utilisateurs convient-elle le mieux ?

Cette solution de provisionnement d’utilisateurs SuccessFactors vers Active Directory est idéale pour :

* Les organisations qui souhaitent une solution cloud prédéfinie pour le provisionnement d’utilisateurs SuccessFactors.

* Les organisations qui ont besoin du provisionnement d’utilisateurs direct de SuccessFactors vers Active Directory.

* Les organisations qui imposent le provisionnement d’utilisateurs avec des données provenant de [SuccessFactors Employee Central (EC)](https://www.successfactors.com/products-services/core-hr-payroll/employee-central.html).

* Les organisations qui ont besoin de synchroniser les utilisateurs entrants/changeant de poste/sortants avec une ou plusieurs unités d’organisation, domaines ou forêts Active Directory seulement sur la base d’informations de modifications détectées dans [SuccessFactors Employee Central (EC)](https://www.successfactors.com/products-services/core-hr-payroll/employee-central.html).

* Organisations utilisant Microsoft 365 pour la messagerie

## <a name="solution-architecture"></a>Architecture de la solution

Cette section décrit l’architecture de la solution de provisionnement des utilisateurs de bout en bout pour les environnements hybrides courants. Il existe deux flux connexes :

* **Flux de données RH faisant autorité – de SuccessFactors vers Active Directory local :** dans ce flux, les événements concernant les employés (comme les nouveaux recrutements, les transferts, les fins de contrat) se produisent d’abord dans SuccessFactors Employee Central dans le cloud, puis les données des événements transitent dans Active Directory local par le biais d’Azure AD et l’agent de provisionnement. Selon l’événement, cela peut provoquer des opérations de création/mise à jour/activation/désactivation dans AD.
* **Flux de réécriture des e-mails – d’Active Directory local vers SuccessFactors :** une fois la création du compte terminée dans Active Directory, il est synchronisé avec Azure AD par le biais de la synchronisation Azure AD Connect et l’attribut d’e-mail peut être réécrit dans SuccessFactors.

  ![Vue d’ensemble](./media/sap-successfactors-inbound-provisioning/sf2ad-overview.png)

### <a name="end-to-end-user-data-flow"></a>Flux de données utilisateur de bout en bout

1. L’équipe RH effectue des transactions sur les employés (entrants/changements de poste/sortants ou recrutements/transferts/arrêts) dans SuccessFactors Employee Central
2. Le service de provisionnement Azure AD effectue des synchronisations planifiées des identités à partir de SuccessFactors EC et identifie les modifications qui doivent être traitées pour la synchronisation avec Active Directory local.
3. Le service Azure AD Provisioning appelle l’agent d’approvisionnement Azure AD Connect local avec une charge utile de demandes contenant des opérations de création/mise à jour/activation/désactivation de compte AD.
4. L’agent de provisionnement Azure AD Connect utilise un compte de service pour ajouter/mettre à jour des données de compte AD.
5. Le moteur de synchronisation Azure AD Connect exécute la synchronisation delta pour extraire les mises à jour dans AD.
6. Les mises à jour d’Active Directory sont synchronisées avec Azure Active Directory local.
7. Si l’[application SuccessFactors Writeback](sap-successfactors-writeback-tutorial.md) est configurée, elle réécrit l’attribut d’e-mail dans SuccessFactors, en fonction de l’attribut de correspondance utilisé.

## <a name="planning-your-deployment"></a>Planification de votre déploiement

La configuration du provisionnement d’utilisateurs piloté par les RH dans le cloud de SuccessFactors vers AD nécessite une planification minutieuse couvrant différents aspects tels que :
* Configuration l’agent de provisionnement Azure AD Connect 
* Nombre d’applications de provisionnement d’utilisateurs SuccessFactors vers Active Directory à déployer
* ID de correspondance, mappage d’attributs, transformation et filtres d’étendue

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

  >[!NOTE]
  >Pour obtenir la liste complète des attributs récupérés par cette application de provisionnement, consultez [Référence des attributs SuccessFactors](../app-provisioning/sap-successfactors-attribute-reference.md)

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

## <a name="configuring-user-provisioning-from-successfactors-to-active-directory"></a>Configuration du provisionnement d’utilisateurs de SuccessFactors vers Active Directory

Cette section présente les étapes à suivre pour configurer le provisionnement de comptes d’utilisateur de SuccessFactors vers chacun des domaines Active Directory concernés par votre intégration.

* [Ajouter l’application du connecteur d’approvisionnement et télécharger l’agent d’approvisionnement](#part-1-add-the-provisioning-connector-app-and-download-the-provisioning-agent)
* [Installer et configurer les agents d'approvisionnement locaux](#part-2-install-and-configure-on-premises-provisioning-agents)
* [Configurer la connectivité à SuccessFactors et Active Directory](#part-3-in-the-provisioning-app-configure-connectivity-to-successfactors-and-active-directory)
* [Configurer les mappages d'attributs](#part-4-configure-attribute-mappings)
* [Activer et lancer l'approvisionnement des utilisateurs](#enable-and-launch-user-provisioning)

### <a name="part-1-add-the-provisioning-connector-app-and-download-the-provisioning-agent"></a>Première partie : Ajouter l’application du connecteur d’approvisionnement et télécharger l’agent d’approvisionnement

**Pour configurer le provisionnement SuccessFactors vers Active Directory**

1. Accédez à <https://portal.azure.com>

2. Dans la barre de navigation de gauche, sélectionnez **Azure Active Directory**

3. Cliquez sur **Applications d’entreprise**, puis sur **Toutes les applications**.

4. Sélectionnez **Ajouter une application**, puis sélectionnez la catégorie **Tous**.

5. Recherchez **SuccessFactors to Active Directory User Provisioning** et ajoutez cette application à partir de la galerie.

6. Une fois l’application ajoutée et l’écran de détails de l’application affiché, sélectionnez **Approvisionnement**

7. Définissez **Mode** **d’approvisionnement** sur **Automatique**

8. Cliquez sur la bannière d’informations affichée pour télécharger l’agent d’approvisionnement. 
   > [!div class="mx-imgBorder"]
   > ![Télécharger l’agent](./media/sap-successfactors-inbound-provisioning/download-pa-agent.png "Écran Télécharger l’agent")


### <a name="part-2-install-and-configure-on-premises-provisioning-agents"></a>Deuxième partie : Installer et configurer des agents de provisionnement locaux

Pour tout approvisionnement vers une instance locale d’Active Directory, l’agent d’approvisionnement doit être installé sur un serveur doté de .NET Framework 4.7.1+ et d’un accès réseau aux domaines Active Directory souhaités.

> [!TIP]
> Vous pouvez vérifier la version du .NET Framework sur votre serveur en suivant les instructions fournies [ici](/dotnet/framework/migration-guide/how-to-determine-which-versions-are-installed).
> Si .NET 4.7.1 ou version ultérieure n'est pas installé sur le serveur, vous pouvez le télécharger [ici](https://support.microsoft.com/help/4033342/the-net-framework-4-7-1-offline-installer-for-windows).  

Transférez le programme d’installation de l’agent téléchargé sur l’hôte du serveur et suivez les étapes indiquées ci-dessous pour terminer la configuration de l’agent.

1. Connectez-vous au serveur Windows Server sur lequel vous souhaitez installer le nouvel agent.

1. Lancez le programme d’installation de l’agent de provisionnement, acceptez les conditions d’utilisation, puis cliquez sur le bouton **Install** (Installer).

   ![Écran Installation](./media/workday-inbound-tutorial/pa_install_screen_1.png "Écran Installation")
   
1. Une fois l’installation terminée, l’Assistant démarrer et vous voyez l’écran **Connect Azure AD** (Connecter Azure AD). Cliquez sur le bouton **Authenticate** (S’authentifier) pour vous connecter à votre instance Azure AD.

   ![Connecter Azure AD](./media/workday-inbound-tutorial/pa_install_screen_2.png "Connecter Azure AD")
   
1. Authentifiez-vous auprès de votre instance Azure AD avec les informations d’identification de l’administrateur général.

   ![Authentification de l’administrateur](./media/workday-inbound-tutorial/pa_install_screen_3.png "Authentification de l’administrateur")

   > [!NOTE]
   > Les informations d'identification de l'administrateur Azure AD sont uniquement utilisées pour la connexion à votre locataire Azure AD. L'agent ne stocke pas les informations d'identification sur le serveur local.

1. Après une authentification réussie auprès d’Azure AD, vous voyez l’écran **Connect Active Directory** (Connecter Active Directory). Entrez alors votre nom de domaine AD, puis cliquez sur le bouton **Ajouter un annuaire**.

   ![Ajouter un répertoire](./media/workday-inbound-tutorial/pa_install_screen_4.png "Ajouter un répertoire")
  
1. Vous êtes maintenant invité à entrer les informations d’identification nécessaires pour vous connecter au domaine AD. Sur le même écran, vous pouvez utiliser **Select domain controller priority** (Sélectionner la priorité de contrôleur de domaine) pour spécifier les contrôleurs de domaine que l’agent doit utiliser pour l’envoi des demandes de provisionnement.

   ![Informations d'identification du domaine](./media/workday-inbound-tutorial/pa_install_screen_5.png)
   
1. Après avoir configuré le domaine, le programme d’installation affiche une liste des domaines configurés. Dans cet écran, vous pouvez répéter les étapes 5 et 6 pour ajouter d’autres domaines ou cliquer sur **Suivant** pour passer à l’inscription de l’agent.

   ![Domaines configurés](./media/workday-inbound-tutorial/pa_install_screen_6.png "Domaines configurés")

   > [!NOTE]
   > Si vous avez plusieurs domaines AD (par exemple na.contoso.com, emea.contoso.com), ajoutez individuellement chaque domaine à la liste.
   > L'ajout du domaine parent (par exemple, contoso.com) ne suffit pas. Vous devez inscrire chaque domaine enfant auprès de l'agent.
   
1. Passez en revue les détails de la configuration, puis cliquez sur **Confirm** (Confirmer) pour inscrire l’agent.
  
   ![Écran Confirmer](./media/workday-inbound-tutorial/pa_install_screen_7.png "Écran Confirmer")
   
1. L’Assistant Configuration affiche la progression de l’inscription de l’agent.
  
   ![Inscription de l’agent](./media/workday-inbound-tutorial/pa_install_screen_8.png "Inscription de l’agent")
   
1. Une fois l’inscription de l’agent réussie, vous pouvez cliquer sur **Exit** (Quitter) pour quitter l’Assistant.
  
   ![Écran Quitter](./media/workday-inbound-tutorial/pa_install_screen_9.png "Écran Quitter")
   
1. Vérifiez l’installation de l’agent et assurez-vous qu’il fonctionne en ouvrant le composant logiciel enfichable « Services » et en recherchant le service « Agent d’approvisionnement Microsoft Azure AD Connect ».
  
   ![Capture d’écran de l’agent d’approvisionnement Microsoft Azure AD Connect en cours d’exécution dans Services.](./media/workday-inbound-tutorial/services.png)

### <a name="part-3-in-the-provisioning-app-configure-connectivity-to-successfactors-and-active-directory"></a>Troisième partie : Dans l’application de provisionnement, configurer la connectivité à SuccessFactors et Active Directory
Lors de cette étape, nous allons établir la connectivité avec SuccessFactors et Active Directory dans le portail Azure. 

1. Dans le portail Azure, revenez à l’application de provisionnement d’utilisateur SuccessFactors vers Active Directory créée dans la [Partie 1](#part-1-add-the-provisioning-connector-app-and-download-the-provisioning-agent).

1. Fermez la section **Informations d’identification de l’administrateur**, comme suit :

   * **Nom de l’utilisateur administrateur** : entrez le nom d’utilisateur du compte d’utilisateur de l’API SuccessFactors, accompagné de l’ID d’entreprise. Son format est le suivant : **nom_utilisateur\@ID_société**

   * **Mot de passe d’administrateur** : entrez le mot de passe du compte d’utilisateur de l’API SuccessFactors. 

   * **URL du locataire** : entrez le nom du point de terminaison des services de l’API OData SuccessFactors. Entrez uniquement le nom d’hôte du serveur, sans http ou https. Cette valeur doit ressembler à ceci : **<nom_serveur_API>.successfactors.com**.

   * **Forêt Active Directory :** « nom » de votre domaine Active Directory, tel qu'il est inscrit auprès de l'agent. Utilisez le menu déroulant pour sélectionner le domaine cible à approvisionner. Cette valeur correspond généralement à une chaîne de type : *contoso.com*

   * **Conteneur Active Directory :** entrez le nom unique du conteneur où l’agent doit créer des comptes d’utilisateur par défaut.
        Exemple : *OU=Users,DC=contoso,DC=com*
        > [!NOTE]
        > Ce paramètre concerne seulement les créations de comptes d’utilisateur si l’attribut *parentDistinguishedName* attribut n’est pas configuré dans les mappages d’attributs. Ce paramètre n’est pas utilisé pour la recherche d’utilisateurs ni pour les opérations de mise à jour. Toute la sous-arborescence du domaine se trouve dans l’étendue de l’opération de recherche.

   * **E-mail de notification :** entrez votre adresse e-mail et activez la case à cocher « Envoyer un e-mail en cas de défaillance ».
    > [!NOTE]
    > Le service Azure AD Provisioning envoie la notification par e-mail si le travail de provisionnement passe à l’état [Mise en quarantaine](../app-provisioning/application-provisioning-quarantine-status.md).

   * Cliquez sur le bouton **Tester la connexion**. Si le test de connexion aboutit, cliquez sur le bouton **Enregistrer**, en haut de l'écran. En cas d’échec, vérifiez que les informations d’identification de SuccessFactors et celles d’Active Directory configurées dans le programme d’installation de l’agent sont valides.
    >[!div class="mx-imgBorder"]
    >![Azure portal](./media/sap-successfactors-inbound-provisioning/sf2ad-provisioning-creds.png)

   * Une fois les informations d’identification enregistrées, la section **Mappages** affiche le mappage par défaut **Synchroniser les employés SuccessFactors avec l’instance locale d’Active Directory**.

### <a name="part-4-configure-attribute-mappings"></a>Partie 4 : Configuration des mappages d’attributs

Dans cette section, vous allez configurer le flux des données utilisateur de SuccessFactors vers Active Directory.

1. Sous l’onglet Provisionnement, sous **Mappages**, cliquez sur **Synchroniser les utilisateurs SuccessFactors avec l’instance locale d’Active Directory**.

1. Dans le champ **Portée de l’objet source**, vous pouvez sélectionner les ensembles d’utilisateurs de SuccessFactors concernés par le provisionnement vers AD, en définissant des filtres basés sur des attributs. L’étendue par défaut est « tous les utilisateurs de SuccessFactors ». Exemples de filtres :

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
   > Par défaut, le moteur de provisionnement désactive/supprime les utilisateurs qui sortent de l’étendue. Vous pouvez juger ce comportement inopportun dans votre intégration de SuccessFactors à AD. Pour remplacer ce comportement par défaut, consultez l’article [Ignorer la suppression des comptes d’utilisateurs qui sortent de l’étendue](../app-provisioning/skip-out-of-scope-deletions.md).
  
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
> Par défaut, lorsque vous activez le service d'approvisionnement, il lance les opérations d'approvisionnement pour tous les utilisateurs concernés. En cas d’erreur de mappage ou de problème lié aux données SuccessFactors, le travail de provisionnement peut échouer et être mis en quarantaine. Pour éviter ce genre de problème, nous vous recommandons de configurer le filtre **Portée de l'objet source** et de tester vos mappages d'attributs sur quelques utilisateurs test avant de lancer la synchronisation complète de tous les utilisateurs. Après avoir vérifié que les mappages fonctionnent et qu'ils vous donnent les résultats souhaités, vous pouvez supprimer le filtre ou l'étendre progressivement pour inclure d'autres utilisateurs.

1. Dans l’onglet **Approvisionnement**, définissez **État d’approvisionnement** sur **Activé**.

2. Cliquez sur **Enregistrer**.

3. Cette opération permet de lancer la synchronisation initiale, dont la durée dépendra du nombre d’utilisateurs du locataire SuccessFactors. Vous pouvez consulter la barre de progression pour suivre la progression du cycle de synchronisation. 

4. À tout moment, consultez l’onglet **Journaux d’audit** dans le portail Azure pour connaître les actions effectuées par le service d’approvisionnement. Les journaux d’audit listent tous les événements de synchronisation individuels effectués par le service de provisionnement, tels que les utilisateurs lus dans SuccessFactors et par la suite ajoutés ou mis à jour dans Active Directory. 

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
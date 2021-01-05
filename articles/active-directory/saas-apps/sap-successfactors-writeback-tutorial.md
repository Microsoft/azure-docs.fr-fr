---
title: 'Tutoriel : Configurer SAP SuccessFactors Writeback dans Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer l’écriture différée des attributs sur SAP SuccessFactors à partir d’Azure AD.
services: active-directory
author: cmmdesai
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.topic: tutorial
ms.workload: identity
ms.date: 10/14/2020
ms.author: chmutali
ms.openlocfilehash: d39e00a80ab167936a749c73867b4343e6ed9d76
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96006436"
---
# <a name="tutorial-configure-attribute-write-back-from-azure-ad-to-sap-successfactors"></a>Tutoriel : Configurer l’écriture différée des attributs d’Azure AD sur SAP SuccessFactors
L’objectif de ce tutoriel est d’illustrer les étapes de l’écriture différée des attributs d’Azure AD sur SAP SuccessFactors Employee Central. 

## <a name="overview"></a>Vue d’ensemble

Vous pouvez configurer l’application SAP SuccessFactors Writeback pour écrire des attributs spécifiques vers SAP SuccessFactors Employee Central à partir d’Azure Active Directory. L’application d’approvisionnement SuccessFactors Writeback permet d’attribuer des valeurs aux attributs Employee Central suivants :

* Adresse e-mail professionnelle
* Nom d’utilisateur
* Numéro de téléphone professionnel (y compris l’indicatif du pays, l’indicatif régional, le numéro et le poste)
* Indicateur principal du numéro de téléphone professionnel
* Numéro de téléphone portable (y compris l’indicatif du pays, l’indicatif régional et le numéro)
* Indicateur principal du numéro de téléphone portable 
* Attributs utilisateur custom01-custom15
* Attribut loginMethod

> [!NOTE]
> Cette application n’a aucune dépendance vis-à-vis des applications d’intégration d’approvisionnement des utilisateurs entrants de SuccessFactors. Vous pouvez la configurer indépendamment de l’application d’approvisionnement [SuccessFactors vers AD local](sap-successfactors-inbound-provisioning-tutorial.md) ou [SuccessFactors vers Azure AD](sap-successfactors-inbound-provisioning-cloud-only-tutorial.md).

### <a name="who-is-this-user-provisioning-solution-best-suited-for"></a>À qui cette solution d’attribution d’utilisateurs convient-elle le mieux ?

Cette solution de provisionnement d’utilisateurs SuccessFactors Writeback est idéale pour :

* Les organisations qui utilisent Microsoft 365 et qui souhaitent écrire en différé vers SuccessFactors Employee Central des attributs faisant autorité gérés par le service informatique (par exemple, l’adresse e-mail, le numéro de téléphone ou le nom d’utilisateur).

## <a name="configuring-successfactors-for-the-integration"></a>Configuration de SuccessFactors pour l’intégration

Tous les connecteurs d’approvisionnement SuccessFactors nécessitent les informations d’identification d’un compte SuccessFactors avec les autorisations appropriées pour appeler les API OData Employee Central. Cette section décrit les étapes permettant de créer le compte de service dans SuccessFactors et d’accorder les autorisations appropriées. 

* [Créer/identifier un compte d’utilisateur d’API dans SuccessFactors](#createidentify-api-user-account-in-successfactors)
* [Créer un rôle d’autorisations d’API](#create-an-api-permissions-role)
* [Créer un groupe d’autorisations pour l’utilisateur des API](#create-a-permission-group-for-the-api-user)
* [Accorder le rôle d’autorisation au groupe d’autorisations](#grant-permission-role-to-the-permission-group)

### <a name="createidentify-api-user-account-in-successfactors"></a>Créer/identifier un compte d’utilisateur d’API dans SuccessFactors
Collaborez avec votre équipe d’administration SuccessFactors ou votre partenaire d’implémentation pour créer ou identifier un compte d’utilisateur dans SuccessFactors qui sera utilisé pour appeler les API OData. Les informations d’identification de nom d’utilisateur et de mot de passe de ce compte seront requises lors de la configuration des applications de provisionnement dans Azure AD. 

### <a name="create-an-api-permissions-role"></a>Créer un rôle d’autorisations d’API

1. Connectez-vous à SAP SuccessFactors avec un compte d’utilisateur qui a accès au centre d’administration.
1. Recherchez *Manage Permission Roles* (Gérer les rôles d’autorisations), puis sélectionnez **Manage Permission Roles** dans les résultats de la recherche.

   ![Gérer les rôles d’autorisations](./media/sap-successfactors-inbound-provisioning/manage-permission-roles.png)

1. Dans la liste Permission Role (Rôle d’autorisation), cliquez sur **Create New** (Créer).

   > [!div class="mx-imgBorder"]
   > ![Créer un rôle d’autorisation](./media/sap-successfactors-inbound-provisioning/create-new-permission-role-1.png)

1. Ajoutez un **nom de rôle** et une **description** pour le nouveau rôle d’autorisation. Le nom et la description doivent indiquer que le rôle est destiné aux autorisations d’utilisation des API.

   > [!div class="mx-imgBorder"]
   > ![Détail du rôle d’autorisation](./media/sap-successfactors-inbound-provisioning/permission-role-detail.png)

1. Sous Permission settings (Paramètres d’autorisation), cliquez sur **Permission...** , faites défiler la liste des autorisations, puis cliquez sur **Manage Integration Tools** (Gérer les outils d’intégration). Cochez la case **Allow Admin to Access to OData API through Basic Authentication** (Autoriser l’administrateur à accéder à l’API OData via l’authentification de base).

   > [!div class="mx-imgBorder"]
   > ![Gérer les outils d’intégration](./media/sap-successfactors-inbound-provisioning/manage-integration-tools.png)

1. Faites défiler la liste dans la même zone et sélectionnez **Employee Central API**. Ajoutez des autorisations comme indiqué ci-dessous pour lire à l’aide de l’API ODATA et modifier à l’aide de l’API ODATA. Sélectionnez l’option de modification si vous envisagez d’utiliser le même compte pour le scénario d’écriture différée vers SuccessFactors. 

   > [!div class="mx-imgBorder"]
   > ![Autorisations de lecture-écriture](./media/sap-successfactors-inbound-provisioning/odata-read-write-perm.png)

1. Cliquez sur **Done** (Terminé). Cliquez sur **Enregistrer les modifications**.

### <a name="create-a-permission-group-for-the-api-user"></a>Créer un groupe d’autorisations pour l’utilisateur des API

1. Dans le centre d’administration SuccessFactors, recherchez *Manage Permission Groups* (Gérer les groupes d’autorisations), puis sélectionnez **Manage Permission Groups** dans les résultats de la recherche.

   > [!div class="mx-imgBorder"]
   > ![Gérer les groupes d’autorisations](./media/sap-successfactors-inbound-provisioning/manage-permission-groups.png)

1. Dans la fenêtre Manage Permission Groups, cliquez sur **Create New**.

   > [!div class="mx-imgBorder"]
   > ![Ajouter un nouveau groupe](./media/sap-successfactors-inbound-provisioning/create-new-group.png)

1. Ajoutez un nom pour le nouveau groupe. Ce nom doit indiquer que le groupe est destiné aux utilisateurs des API.

   > [!div class="mx-imgBorder"]
   > ![Nom du groupe d’autorisations](./media/sap-successfactors-inbound-provisioning/permission-group-name.png)

1. Ajoutez des membres au groupe. Par exemple, vous pouvez sélectionner **Username** dans le menu déroulant People Pool, puis entrer le nom d’utilisateur du compte d’API qui sera utilisé pour l’intégration. 

   > [!div class="mx-imgBorder"]
   > ![Add group members](./media/sap-successfactors-inbound-provisioning/add-group-members.png) (Ajouter des membres à un groupe)

1. Cliquez sur **Done** pour terminer la création du groupe d’autorisations.

### <a name="grant-permission-role-to-the-permission-group"></a>Accorder le rôle d’autorisation au groupe d’autorisations

1. Dans le centre d’administration SuccessFactors, recherchez *Manage Permission Roles*, puis sélectionnez **Manage Permission Roles** dans les résultats de la recherche.
1. Dans la liste **Permission Role List** (Liste des rôles d’autorisations), sélectionnez le rôle que vous avez créé pour les autorisations d’utilisation des API.
1. Sous **Grant this role to...** (Attribuer ce rôle à), cliquez sur le bouton **Add...** (Ajouter).
1. Sélectionnez **Permission Group...** (Groupe d’autorisations) dans le menu déroulant, puis cliquez sur **Select...** (Sélectionner) pour ouvrir la fenêtre Groups afin de rechercher et sélectionner le groupe créé ci-dessus. 

   > [!div class="mx-imgBorder"]
   > ![Ajouter un groupe d’autorisations](./media/sap-successfactors-inbound-provisioning/add-permission-group.png)

1. Passez en revue l’octroi de rôle d’autorisation au groupe d’autorisations. 
   > [!div class="mx-imgBorder"]
   > ![Détails du rôle et du groupe d’autorisations](./media/sap-successfactors-inbound-provisioning/permission-role-group.png)

1. Cliquez sur **Enregistrer les modifications**.

## <a name="preparing-for-successfactors-writeback"></a>Préparation de SuccessFactors Writeback

L’application d’approvisionnement SuccessFactors Writeback utilise certaines valeurs de *code* pour définir les adresses email et les numéros de téléphone dans Employee Central. Ces valeurs de *code* sont définies comme des valeurs constantes dans la table de mappage d’attributs et sont différentes pour chaque instance SuccessFactors. Cette section décrit les étapes à suivre pour capturer ces valeurs de *code*.

   > [!NOTE]
   > Veuillez impliquer votre administrateur SuccessFactors pour effectuer les étapes décrites dans cette section. 

### <a name="identify-email-and-phone-number-picklist-names"></a>Identifier les noms de liste déroulante d’adresses e-mail et de numéros de téléphone 

Dans SAP SuccessFactors, une *liste déroulante* est un ensemble configurable d’options parmi lesquelles un utilisateur peut effectuer une sélection. Les différents types d’adresse e-mail et de numéro de téléphone (par exemple, professionnel, personnel, autre) sont représentés à l’aide d’une liste déroulante. Dans cette étape, nous allons identifier les listes déroulantes configurées dans votre locataire SuccessFactors pour stocker les valeurs d’adresse e-mail et de numéro de téléphone. 
 
1. Dans le centre d’administration SuccessFactors, recherchez *Gérer la configuration d’entreprise*. 

   > [!div class="mx-imgBorder"]
   > ![Manage business configuration](./media/sap-successfactors-inbound-provisioning/manage-business-config.png)

1. Sous **Éléments HRIS**, sélectionnez **emailInfo**, puis cliquez sur *Détails* pour le champ **email-type**.

   > [!div class="mx-imgBorder"]
   > ![Get email info](./media/sap-successfactors-inbound-provisioning/get-email-info.png)

1. Dans la page de détails d’**email-type**, notez le nom de la liste déroulante associée à ce champ. Par défaut, il s’agit de **ecEmailType**. Toutefois, il peut être différent dans votre locataire. 

   > [!div class="mx-imgBorder"]
   > ![Identify email picklist](./media/sap-successfactors-inbound-provisioning/identify-email-picklist.png)

1. Sous **Éléments HRIS**, sélectionnez **phoneInfo**, puis cliquez sur *Détails* pour le champ **phone-type**.

   > [!div class="mx-imgBorder"]
   > ![Get phone info](./media/sap-successfactors-inbound-provisioning/get-phone-info.png)

1. Dans la page de détails de **phone-type**, notez le nom de la liste déroulante associée à ce champ. Par défaut, il s’agit de **ecPhoneType**. Toutefois, il peut être différent dans votre locataire. 

   > [!div class="mx-imgBorder"]
   > ![Identify phone picklist](./media/sap-successfactors-inbound-provisioning/identify-phone-picklist.png)

### <a name="retrieve-constant-value-for-emailtype"></a>Récupérer la valeur constante pour emailType

1. Dans le centre d’administration SuccessFactors, recherchez et ouvrez *Centre de liste déroulante*. 
1. Utilisez le nom de la liste déroulante des adresses e-mail capturé dans la section précédente (par exemple, ecEmailType) pour trouver la liste déroulante des adresses e-mail. 

   > [!div class="mx-imgBorder"]
   > ![Find email type picklist](./media/sap-successfactors-inbound-provisioning/find-email-type-picklist.png)

1. Ouvrez la liste déroulante des adresses e-mail active. 

   > [!div class="mx-imgBorder"]
   > ![Open active email type picklist](./media/sap-successfactors-inbound-provisioning/open-active-email-type-picklist.png)

1. Sur la page de la liste déroulante des types d’e-mail, sélectionnez le type d’e-mail *Professionnel*.

   > [!div class="mx-imgBorder"]
   > ![Select business email type](./media/sap-successfactors-inbound-provisioning/select-business-email-type.png)

1. Notez l’**ID d’option** associé à l’e-mail *professionnel*. Il s’agit du code que nous allons utiliser avec *emailType* dans la table de mappage des attributs.

   > [!div class="mx-imgBorder"]
   > ![Get email type code](./media/sap-successfactors-inbound-provisioning/get-email-type-code.png)

   > [!NOTE]
   > Le cas échéant, supprimez la virgule (ici, symbole de groupement de chiffres) lorsque vous copiez la valeur. Par exemple, si la valeur **ID d’option** est *8,448*, définissez *emailType* dans Azure AD sur le nombre constant *8448* (sans virgule). 

### <a name="retrieve-constant-value-for-phonetype"></a>Récupérer la valeur constante pour phoneType

1. Dans le centre d’administration SuccessFactors, recherchez et ouvrez *Centre de liste déroulante*. 
1. Utilisez le nom de la liste déroulante des numéros de téléphone capturé dans la section précédente pour trouver la liste déroulante des numéros de téléphone. 

   > [!div class="mx-imgBorder"]
   > ![Find phone type picklist](./media/sap-successfactors-inbound-provisioning/find-phone-type-picklist.png)

1. Ouvrez la liste déroulante des numéros de téléphone active. 

   > [!div class="mx-imgBorder"]
   > ![Open active phone type picklist](./media/sap-successfactors-inbound-provisioning/open-active-phone-type-picklist.png)

1. Sur la page de la liste déroulante des types de numéro de téléphone, passez en revue les différents types de numéros de téléphone listés sous **Valeurs de liste déroulante**.

   > [!div class="mx-imgBorder"]
   > ![Review phone types](./media/sap-successfactors-inbound-provisioning/review-phone-types.png)

1. Notez l’**ID d’option** associé au numéro de téléphone *professionnel*. Il s’agit du code que nous allons utiliser avec *businessPhoneType* dans la table de mappage des attributs.

   > [!div class="mx-imgBorder"]
   > ![Get business phone code](./media/sap-successfactors-inbound-provisioning/get-business-phone-code.png)

1. Notez l’**ID d’option** associé au numéro de téléphone *portable*. Il s’agit du code que nous allons utiliser avec *cellPhoneType* dans la table de mappage des attributs.

   > [!div class="mx-imgBorder"]
   > ![Get cell phone code](./media/sap-successfactors-inbound-provisioning/get-cell-phone-code.png)

   > [!NOTE]
   > Le cas échéant, supprimez la virgule (ici, symbole de groupement de chiffres) lorsque vous copiez la valeur. Par exemple, si la valeur **ID d’option** est *10,606*, définissez *cellPhoneType* dans Azure AD sur le nombre constant *10606* (sans virgule). 


## <a name="configuring-successfactors-writeback-app"></a>Configuration de l’application SuccessFactors Writeback

Cette section décrit les étapes à suivre pour 

* [Ajouter l’application du connecteur de provisionnement et configurer la connectivité à SuccessFactors](#part-1-add-the-provisioning-connector-app-and-configure-connectivity-to-successfactors).
* [Configurer les mappages d'attributs](#part-2-configure-attribute-mappings)
* [Activer et lancer l'approvisionnement des utilisateurs](#enable-and-launch-user-provisioning)

### <a name="part-1-add-the-provisioning-connector-app-and-configure-connectivity-to-successfactors"></a>Première partie : Ajouter l’application du connecteur de provisionnement et configurer la connectivité à SuccessFactors

**Pour configurer SuccessFactors Writeback**

1. Accédez à <https://portal.azure.com>

2. Dans la barre de navigation de gauche, sélectionnez **Azure Active Directory**

3. Cliquez sur **Applications d’entreprise**, puis sur **Toutes les applications**.

4. Sélectionnez **Ajouter une application**, puis sélectionnez la catégorie **Tous**.

5. Recherchez **SuccessFactors Writeback** et ajoutez cette application à partir de la galerie.

6. Une fois l’application ajoutée et l’écran de détails de l’application affiché, sélectionnez **Approvisionnement**

7. Définissez **Mode** **d'approvisionnement** sur **Automatique**

8. Fermez la section **Informations d’identification de l’administrateur**, comme suit :

   * **Nom de l’utilisateur administrateur** : entrez le nom d’utilisateur du compte d’utilisateur de l’API SuccessFactors, accompagné de l’ID d’entreprise. Son format est le suivant : **nom_utilisateur\@ID_société**

   * **Mot de passe d’administrateur** : entrez le mot de passe du compte d’utilisateur de l’API SuccessFactors. 

   * **URL du locataire** : entrez le nom du point de terminaison des services de l’API OData SuccessFactors. Entrez uniquement le nom d’hôte du serveur, sans http ou https. Cette valeur doit ressembler à ceci : `api4.successfactors.com`.

   * **E-mail de notification :** entrez votre adresse e-mail et activez la case à cocher « Envoyer un e-mail en cas de défaillance ».
    > [!NOTE]
    > Le service Azure AD Provisioning envoie la notification par e-mail si le travail de provisionnement passe à l’état [Mise en quarantaine](../app-provisioning/application-provisioning-quarantine-status.md).

   * Cliquez sur le bouton **Tester la connexion**. Si le test de connexion aboutit, cliquez sur le bouton **Enregistrer**, en haut de l'écran. En cas d’échec, vérifiez que les informations d’identification et l’URL SuccessFactors sont valides.
    >[!div class="mx-imgBorder"]
    >![Azure portal](./media/sap-successfactors-inbound-provisioning/sfwb-provisioning-creds.png)

   * Une fois les informations d’identification enregistrées, la section **Mappages** affiche le mappage par défaut. Actualisez la page si les mappages d’attributs ne sont pas visibles.  

### <a name="part-2-configure-attribute-mappings"></a>Deuxième partie : Configuration des mappages d’attributs

Dans cette section, vous allez configurer le flux des données utilisateur de SuccessFactors vers Active Directory.

1. Dans l’onglet Approvisionnement, sous **Mappages**, cliquez sur **Approvisionner les utilisateurs d’Azure Active Directory**.

1. Dans le champ **Portée de l’objet source**, vous pouvez sélectionner les ensembles d’utilisateurs Azure AD qui doivent être pris en compte pour l’écriture différée, en définissant des filtres d’attributs. La portée par défaut est « tous les utilisateurs dans Azure AD ». 
   > [!TIP]
   > Lors de la configuration initiale de l'application d'approvisionnement, vous devez tester et vérifier vos mappages d'attributs et expressions pour être sûr d'obtenir le résultat souhaité. Microsoft vous recommande d’utiliser les filtres d’étendue disponibles sous **Portée de l’objet source** pour tester vos mappages avec quelques utilisateurs test d’Azure AD. Après avoir vérifié que les mappages fonctionnent, vous pouvez supprimer le filtre ou l'étendre progressivement pour inclure d'autres utilisateurs.

1. Le champ **Actions d’objet cible** prend uniquement en charge l’opération **Mise à jour**.

1. Dans la table mappage, sous la section **Mappages d’attributs**, vous pouvez mapper les attributs Azure Active Directory suivants à SuccessFactors. Le tableau ci-dessous fournit des conseils sur la façon de mapper les attributs d’écriture différée. 

   | \# | Attribut Azure AD | Attribut SuccessFactors | Notes |
   |--|--|--|--|
   | 1 | employeeId | personIdExternal | Par défaut, cet attribut est l’identificateur correspondant. Au lieu d’employeeId, vous pouvez utiliser tout autre attribut Azure AD capable de stocker la valeur égale à personIdExternal dans SuccessFactors.    |
   | 2 | mail | email | Mappez la source de l’attribut « email ». À des fins de test, vous pouvez mapper userPrincipalName à email. |
   | 3 | 8448 | emailType | Cette valeur constante est la valeur d’ID SuccessFactors associée à la messagerie professionnelle. Mettez à jour cette valeur pour qu’elle corresponde à votre environnement SuccessFactors. Consultez la section [Récupérer la valeur constante pour emailType](#retrieve-constant-value-for-emailtype) afin de connaître les étapes permettant de définir cette valeur. |
   | 4 | true | emailIsPrimary | Utilisez cet attribut pour définir la messagerie professionnelle comme messagerie principale dans SuccessFactors. Si la messagerie professionnelle n’est pas la messagerie principale, définissez cet indicateur sur « false ». |
   | 5 | userPrincipalName | [custom01 – custom15] | En utilisant **Ajouter un nouveau mappage**, vous pouvez éventuellement écrire userPrincipalName ou tout autre attribut Azure AD dans un attribut personnalisé disponible dans l’objet Utilisateur SuccessFactors.  |
   | 6 | on-prem-samAccountName | username | En utilisant **Ajouter un nouveau mappage**, vous pouvez éventuellement mapper l’attribut samAccountName local à l’attribut username de SuccessFactors. |
   | 7 | SSO | loginMethod | Si le locataire SuccessFactors est configuré pour une [SSO partielle](https://apps.support.sap.com/sap/support/knowledge/en/2320766), en utilisant Ajouter un nouveau mappage, vous pouvez éventuellement définir loginMethod sur une valeur constante « SSO » ou « PWD ». |
   | 8 | telephoneNumber | businessPhoneNumber | Utilisez ce mappage pour transmettre l’attribut *telephoneNumber* d’Azure AD au numéro de téléphone professionnel SuccessFactors. |
   | 9 | 10605 | businessPhoneType | Cette valeur constante est la valeur d’ID SuccessFactors associée au téléphone professionnel. Mettez à jour cette valeur pour qu’elle corresponde à votre environnement SuccessFactors. Consultez la section [Récupérer la valeur constante pour phoneType](#retrieve-constant-value-for-phonetype) afin de connaître les étapes permettant de définir cette valeur. |
   | 10 | true | businessPhoneIsPrimary | Utilisez cet attribut pour définir le numéro de téléphone professionnel comme numéro principal. Les valeurs valides sont « true » ou « false ». |
   | 11 | mobile | cellPhoneNumber | Utilisez ce mappage pour transmettre l’attribut *telephoneNumber* d’Azure AD au numéro de téléphone professionnel SuccessFactors. |
   | 12 | 10606 | cellPhoneType | Cette valeur constante est la valeur d’ID SuccessFactors associée au téléphone portable. Mettez à jour cette valeur pour qu’elle corresponde à votre environnement SuccessFactors. Consultez la section [Récupérer la valeur constante pour phoneType](#retrieve-constant-value-for-phonetype) afin de connaître les étapes permettant de définir cette valeur. |
   | 13 | false | cellPhoneIsPrimary | Utilisez cet attribut pour définir le numéro de téléphone portable comme numéro principal. Les valeurs valides sont « true » ou « false ». |
 
1. Validez et vérifiez vos mappages d’attributs. 
 
    >[!div class="mx-imgBorder"]
    >![Writeback attribute mapping](./media/sap-successfactors-inbound-provisioning/writeback-attribute-mapping.png)

1. Cliquez sur **Enregistrer** pour enregistrer les mappages. Ensuite, nous allons mettre à jour les expressions d’API du chemin JSON pour utiliser les codes phoneType dans votre instance SuccessFactors. 
1. Sélectionnez **Afficher les options avancées**. 

    >[!div class="mx-imgBorder"]
    >![Show advanced options](./media/sap-successfactors-inbound-provisioning/show-advanced-options.png)

1. Cliquez sur **Modifier la liste des attributs de SuccessFactors**. 

   > [!NOTE] 
   > Si l’option **Modifier la liste des attributs de SuccessFactors** ne s’affiche pas dans le portail Azure, utilisez l’URL *https://portal.azure.com/?Microsoft_AAD_IAM_forceSchemaEditorEnabled=true* pour accéder à la page. 

1. La colonne **Expression d’API** de cette vue affiche les expressions de chemin JSON utilisées par le connecteur. 
1. Mettez à jour les expressions de chemin JSON pour le téléphone professionnel et le téléphone portable afin d’utiliser la valeur d’ID (*businessPhoneType* et *cellPhoneType*) correspondant à votre environnement. 

    >[!div class="mx-imgBorder"]
    >![Phone JSON Path change](./media/sap-successfactors-inbound-provisioning/phone-json-path-change.png)

1. Cliquez sur **Enregistrer** pour enregistrer les mappages.

## <a name="enable-and-launch-user-provisioning"></a>Activer et lancer l'approvisionnement des utilisateurs

Une fois les configurations d’application de provisionnement SuccessFactors effectuées, vous pouvez activer le service de provisionnement dans le portail Azure.

> [!TIP]
> Par défaut, lorsque vous activez le service d'approvisionnement, il lance les opérations d'approvisionnement pour tous les utilisateurs concernés. En cas d’erreur de mappage ou de problème lié aux données, le travail d’approvisionnement peut échouer et être mis en quarantaine. Pour éviter ce genre de problème, nous vous recommandons de configurer le filtre **Portée de l'objet source** et de tester vos mappages d'attributs sur quelques utilisateurs test avant de lancer la synchronisation complète de tous les utilisateurs. Après avoir vérifié que les mappages fonctionnent et qu'ils vous donnent les résultats souhaités, vous pouvez supprimer le filtre ou l'étendre progressivement pour inclure d'autres utilisateurs.

1. Dans l’onglet **Approvisionnement**, définissez **État d’approvisionnement** sur **Activé**.

1. Sélectionnez **Étendue**. Vous pouvez sélectionner l’une des options suivantes : 
   * **Synchroniser l’ensemble des utilisateurs et groupes** : Sélectionnez cette option si vous envisagez de réécrire des attributs mappés de tous les utilisateurs d’Azure AD dans SuccessFactors, selon les règles d’étendue définies sous **Mappages** -> **Portée de l’objet source**. 
   * **Synchroniser uniquement les utilisateurs et groupes assignés** : Sélectionnez cette option si vous envisagez de réécrire des attributs mappés des utilisateurs que vous avez assignés à cette application uniquement dans l’option de menu **Application** -> **Gérer** -> **Utilisateurs et groupes**. Ces utilisateurs sont également soumis aux règles d’étendue définies sous **Mappages** -> **Portée de l’objet source**.

   > [!div class="mx-imgBorder"]
   > ![Sélectionner l’étendue d’écriture différée](./media/sap-successfactors-inbound-provisioning/select-writeback-scope.png)

   > [!NOTE]
   > L’application d’approvisionnement de l’écriture différée SuccessFactors ne prend pas en charge l’« affectation de groupe ». Seule l’« attribution d’utilisateur » est prise en charge. 

1. Cliquez sur **Enregistrer**.

1. Cette opération permet de lancer la synchronisation initiale, dont la durée dépendra du nombre d’utilisateurs du locataire Azure AD et de l’étendue définie pour l’opération. Vous pouvez consulter la barre de progression pour suivre la progression du cycle de synchronisation. 

1. À tout moment, consultez l’onglet **Journaux d’approvisionnement** dans le portail Azure pour connaître les actions effectuées par le service d’approvisionnement. Les journaux d’approvisionnement répertorient tous les événements de synchronisation individuels effectués par le service d’approvisionnement. 

1. Au terme de la synchronisation initiale, un rapport de synthèse d'audit est créé dans l'onglet **Approvisionnement**, comme illustré ci-dessous.

   > [!div class="mx-imgBorder"]
   > ![Barre de progression de provisionnement](./media/sap-successfactors-inbound-provisioning/prov-progress-bar-stats.png)

## <a name="supported-scenarios-known-issues-and-limitations"></a>Scénarios pris en charge, problèmes connus et limitations

Reportez-vous à la [section Scénarios d’écriture différée](../app-provisioning/sap-successfactors-integration-reference.md#writeback-scenarios) du guide de référence sur l’intégration de SAP SuccessFactors. 

## <a name="next-steps"></a>Étapes suivantes

* [Approfondissez vos connaissances sur l’intégration d’Azure AD et de SAP SuccessFactors](../app-provisioning/sap-successfactors-integration-reference.md)
* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)
* [Découvrez comment configurer l’authentification unique entre SuccessFactors et Azure Active Directory](successfactors-tutorial.md)
* [Découvrir comment intégrer d’autres applications SaaS à Azure Active Directory](tutorial-list.md)
* [Découvrez comment exporter et importer vos configurations de provisionnement](../app-provisioning/export-import-provisioning-configuration.md)
---
title: "Tutoriel : Configurer l'approvisionnement entrant Workday dans Azure Active Directory | Microsoft Docs"
description: Découvrez comment configurer l'approvisionnement entrant de Workday vers Azure AD.
services: active-directory
author: cmmdesai
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.topic: tutorial
ms.workload: identity
ms.date: 05/26/2020
ms.author: chmutali
ms.openlocfilehash: 197b7ff0a6c613a019007ba507d678b619c9afd4
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94358590"
---
# <a name="tutorial-configure-workday-to-azure-ad-user-provisioning"></a>Tutoriel : Configurer l'attribution d'utilisateurs Workday vers Azure AD
Ce tutoriel a pour objectif d’expliquer les étapes à suivre pour approvisionner des données d’employés de Workday vers Azure Active Directory. 

>[!NOTE]
>Utilisez ce tutoriel si les utilisateurs que vous souhaitez attribuer à partir de Workday sont des utilisateurs cloud uniquement qui n’ont pas besoin de compte AD local. Si les utilisateurs n’ont besoin que d’un compte AD local ou d’un compte AD et Azure AD, consultez le tutoriel sur la façon de [configurer l'attribution d’utilisateurs Workday vers Active Directory](workday-inbound-tutorial.md). 

## <a name="overview"></a>Vue d’ensemble

Le [service d’approvisionnement utilisateur Azure Active Directory](../app-provisioning/user-provisioning.md) s’intègre aux [API de ressources humaines Workday](https://community.workday.com/sites/default/files/file-hosting/productionapi/Human_Resources/v21.1/Get_Workers.html) afin d’approvisionner des comptes d’utilisateur. Les flux de travail d’approvisionnement de l’utilisateur Workday pris en charge par le service d’approvisionnement de l’utilisateur Azure AD autorisent l’automatisation des scénarios de gestion du cycle de vie des identités et des ressources humaines suivants :

* **Nouvelles embauches**  : lorsqu’un nouvel employé est ajouté à Workday, un compte d’utilisateur est automatiquement créé dans Azure Active Directory et, éventuellement, Microsoft 365 et [d’autres applications SaaS prises en charge par Azure AD](../app-provisioning/user-provisioning.md), avec la réécriture de l’adresse e-mail pour Workday.

* **Mises à jour du profil et des attributs de l’employé**  : lorsque le dossier d’un employé est mis à jour dans Workday (par exemple le nom, la fonction ou le responsable), son compte d’utilisateur est mis à jour automatiquement dans Azure Active Directory et éventuellement dans Microsoft 365 et d’ [autres applications SaaS prises en charge par Azure AD](../app-provisioning/user-provisioning.md).

* **Résiliations de contrats d’employés**  : lorsque le contrat d’un employé est résilié dans Workday, son compte d’utilisateur est désactivé automatiquement dans Azure Active Directory et éventuellement dans Microsoft 365 et d’ [autres applications SaaS prises en charge par Azure AD](../app-provisioning/user-provisioning.md).

* **Employés réembauchés**  : lorsqu’un employé est réembauché dans Workday, son ancien compte peut être réactivé ou reprovisionné automatiquement (en fonction de votre préférence) dans Azure Active Directory et éventuellement dans Microsoft 365 et d’ [autres applications SaaS prises en charge par Azure AD](../app-provisioning/user-provisioning.md).

### <a name="who-is-this-user-provisioning-solution-best-suited-for"></a>À qui cette solution d’attribution d’utilisateurs convient-elle le mieux ?

Cette solution d'attribution d’utilisateurs Workday vers Azure Active Directory est idéale pour :

* les organisations qui souhaitent une solution cloud prédéfinie pour l’attribution d’utilisateurs Workday ;

* Les organisations qui ont besoin d'une attribution directe d’utilisateurs de Workday vers Azure Active Directory ;

* Les organisations qui ont besoin d'une attribution d’utilisateurs à l’aide de données provenant de Workday.

* Organisations utilisant Microsoft 365 pour la messagerie

## <a name="solution-architecture"></a>Architecture de solution

Cette section décrit l’architecture de la solution de provisionnement d’utilisateurs de bout en bout pour les utilisateurs du cloud uniquement. Il existe deux flux connexes :

* **Flux de données RH faisant autorité - de Workday vers Azure Active Directory :** dans ce flux, les événements concernant les employés (comme les nouveaux recrutements, les transferts, les fins de contrat) se produisent d’abord dans Workday, puis les données des événements transitent dans Azure Active Directory. Selon l’événement, cela peut provoquer des opérations de création/mise à jour/activation/désactivation dans Azure AD.
* **Flux de réécriture - d’Active Directory en local vers Workday :** Une fois le compte créé dans Active Directory, il est synchronisé avec Azure AD par le biais d’Azure AD Connect et des informations telles que l'e-mail, le nom d’utilisateur et le numéro de téléphone peuvent être réécrites dans Workday.

  ![Vue d’ensemble](./media/workday-inbound-tutorial/workday-cloud-only-provisioning.png)

### <a name="end-to-end-user-data-flow"></a>Flux de données utilisateur de bout en bout

1. L’équipe RH effectue des transactions sur les employés (entrants/changements de poste/sortants ou recrutements/transferts/arrêts) dans Workday Employee Central
2. Le service d'approvisionnement Azure AD effectue des synchronisations planifiées des identités à partir de Workday EC et identifie les modifications qui doivent être traitées pour la synchronisation avec Active Directory local.
3. Le service de provisionnement Azure AD détermine la modification et appelle l’opération de création/mise à jour/activation/désactivation pour l’utilisateur dans Azure AD.
4. Si l'application [Workday Writeback](workday-writeback-tutorial.md) est configurée, elle récupère des attributs tels que la messagerie, le nom d’utilisateur et le numéro de téléphone à partir d’Azure AD. 
5. Le service d'approvisionnement Azure AD définit la messagerie, le nom d’utilisateur et le numéro de téléphone dans Workday.

## <a name="planning-your-deployment"></a>Planification de votre déploiement

La configuration de l'attribution d’utilisateurs pilotée par les RH de Workday vers Azure AD nécessite une planification minutieuse couvrant différents aspects tels que :

* Détermination de l’ID correspondant 
* Mappage d’attributs
* Transformation d’attributs 
* Filtres d’étendue

Pour obtenir des instructions complètes sur ces sujets, consultez le [plan de déploiement cloud des RH](../app-provisioning/plan-cloud-hr-provision.md). 

## <a name="configure-integration-system-user-in-workday"></a>Configurer un utilisateur du système d’intégration dans Workday

Reportez-vous à la section [Configurer un utilisateur du système d’intégration](workday-inbound-tutorial.md#configure-integration-system-user-in-workday) pour créer un compte d'utilisateur du système d’intégration Workday autorisé à récupérer les données d’employés. 

## <a name="configure-user-provisioning-from-workday-to-azure-ad"></a>Configurer l'attribution d'utilisateurs de Workday vers Azure AD

Les sections suivantes décrivent les étapes à suivre afin de configurer l'approvisionnement d'utilisateurs de Workday vers Azure AD pour les déploiements « cloud only ».

* [Ajouter l'application de connecteur d'approvisionnement Azure AD et établir la connexion avec Workday](#part-1-adding-the-azure-ad-provisioning-connector-app-and-creating-the-connection-to-workday)
* [Configurer les mappages d'attributs Workday et Azure AD](#part-2-configure-workday-and-azure-ad-attribute-mappings)
* [Activer et lancer l'approvisionnement des utilisateurs](#enable-and-launch-user-provisioning)

### <a name="part-1-adding-the-azure-ad-provisioning-connector-app-and-creating-the-connection-to-workday"></a>Première partie : Ajout de l’application des connecteurs de provisionnement Azure AD et création de la connexion à Workday

**Pour configurer Workday sur l’approvisionnement Azure Active Directory pour les utilisateurs « cloud only » :**

1. Atteindre <https://portal.azure.com>.

2. Dans le Portail Azure, recherchez et sélectionnez **Azure Active Directory**.

3. Cliquez sur **Applications d’entreprise** , puis sur **Toutes les applications**.

4. Sélectionnez **Ajouter une application** , puis sélectionnez la catégorie **Tous**.

5. Recherchez **Attribution d'utilisateurs de Workday vers Azure AD** et ajoutez cette application à partir de la galerie.

6. Une fois l’application ajoutée et l’écran de détails de l’application affiché, sélectionnez **Provisionnement**.

7. Définissez le **mode** de **provisionnement** sur **Automatique**

8. Fermez la section **Informations d’identification de l’administrateur** , comme suit :

   * **Nom d'utilisateur Workday**  : entrez le nom d'utilisateur du compte du système d'intégration Workday, avec le nom de domaine du locataire. Le résultat doit être le suivant : username@contoso4

   * **Mot de passe Workday :** entrez le mot de passe du compte du système d'intégration Workday.

   * **URL de l'API des services web Workday :** entrez l'URL du point de terminaison des services web Workday de votre locataire. L’URL détermine la version de l’API Workday Web Services utilisée par le connecteur. 
   
     | Format d’URL | Version d'API WWS utilisée | Modifications XPATH requises |
     |------------|----------------------|------------------------|
     | https://####.workday.com/ccx/service/tenantName | Version 21.1 | Non |
     | https://####.workday.com/ccx/service/tenantName/Human_Resources | Version 21.1 | Non |
     | https://####.workday.com/ccx/service/tenantName/Human_Resources/v##.# | v##.# | Oui |

      > [!NOTE]
     > Si aucune information de version n’est spécifiée dans l’URL, l’application utilise Workday Web Services (WWS) version 21.1, et aucune modification n’est requise pour les expressions de l’API XPATH par défaut fournies avec l’application. Pour utiliser une version spécifique de l’API WWS, spécifiez le numéro de version dans l’URL. <br>
     > Exemple : `https://wd3-impl-services1.workday.com/ccx/service/contoso4/Human_Resources/v34.0` <br>
     > <br> Si vous utilisez une API WWS v30.0, avant d’activer le travail d’approvisionnement, mettez à jour les **expressions API XPATH** sous **Mappage d’attributs -> Options avancées -> Modifier la liste d’attributs de Workday** en vous reportant à la section [Gestion de votre configuration](workday-inbound-tutorial.md#managing-your-configuration) et aux [Informations de référence sur l’attribut Workday](../app-provisioning/workday-attribute-reference.md#xpath-values-for-workday-web-services-wws-api-v30).  

   * **E-mail de notification :** entrez votre adresse e-mail et activez la case à cocher « Envoyer un e-mail en cas de défaillance ».

   * Cliquez sur le bouton **Tester la connexion**.

   * Si le test de connexion réussit, cliquez sur le bouton **Enregistrer** en haut. En cas d’échec, vérifiez que l’URL et les informations d’identification Workday sont valides dans Workday.

### <a name="part-2-configure-workday-and-azure-ad-attribute-mappings"></a>Deuxième partie : Configurer les mappages d'attributs Workday et Azure AD

Dans cette section, vous allez configurer le flux des données de Workday vers Azure Active Directory pour l’utilisateur « cloud only ».

1. Dans l’onglet Approvisionnement sous **Mappages** , cliquez sur **Synchroniser les employés sur Azure AD**.

2. Dans le champ **Portée de l'objet source** , vous pouvez sélectionner les ensembles d'utilisateurs de Workday concernés par l'approvisionnement vers Azure AD, en définissant des filtres basés sur des attributs. L’étendue par défaut est « tous les utilisateurs de Workday ». Exemples de filtres :

   * Exemple : Étendue pour les utilisateurs avec des ID d’employés entre 1000000 et 2000000

      * Attribut : WorkerID

      * Opérateur : REGEX Match

      * Valeur : (1[0-9][0-9][0-9][0-9][0-9][0-9])

   * Exemple : Uniquement les employés occasionnels et pas les employés réguliers

      * Attribut : ContingentID

      * Opérateur : IS NOT NULL

3. Dans le champ **Actions de l'objet cible** , vous pouvez filtrer globalement les actions exécutées sur Azure AD. Les actions **Créer** et **Mettre à jour** sont les plus courantes.

4. Dans la section **Mappages d’attributs** , vous pouvez définir comment les attributs Workday sont mappés aux attributs Active Directory.

5. Cliquez sur un mappage d’attributs existants à mettre à jour ou cliquez sur **Ajouter un nouveau mappage** en bas de l’écran pour ajouter de nouveaux mappages. Un mappage d’attribut individuel prend en charge les propriétés suivantes :

   * **Type de mappage**

      * **Direct**  : inscrit la valeur de l'attribut Workday dans l'attribut AD, sans aucune modification

      * **Constante**  : inscrivez une valeur de chaîne constante et statique dans l'attribut AD

      * **Expression** : vous permet d’écrire une valeur personnalisée dans l’attribut AD, basée sur un ou plusieurs attributs Workday. [Pour plus d’informations, consultez l’article sur les expressions](../app-provisioning/functions-for-customizing-application-data.md).

   * **Attribut source** : l’attribut utilisateur dans Workday. Si l’attribut que vous recherchez n’est pas présent, consultez [Personnalisation de la liste des attributs d’utilisateur Workday](workday-inbound-tutorial.md#customizing-the-list-of-workday-user-attributes).

   * **Valeur par défaut** : facultatif. Si l’attribut source a une valeur vide, le mappage écrit cette valeur à la place.
            La configuration la plus courante consiste à laisser ce champ vide.

   * **Attribut cible** : l’attribut utilisateur dans Azure AD.

   * **Faire correspondre des objets à l'aide de cet attribut**  : indique si cet attribut doit être utilisé pour identifier les utilisateurs de manière unique entre Workday et Azure AD. Cette valeur est communément définie dans le champ ID collaborateur de Workday, qui est généralement mappé avec l'attribut ID employé (nouveau) ou un attribut d'extension dans Azure AD.

   * **Priorité des correspondances** : plusieurs attributs de correspondance peuvent être définis. S’il en existe plusieurs, ils sont évalués dans l’ordre défini par ce champ. Dès qu’une correspondance est trouvée, aucun autre attribut correspondant n’est évalué.

   * **Appliquer ce mappage**

     * **Toujours**  : applique ce mappage à la création de l’utilisateur et des actions de mise à jour.

     * **Lors de la création uniquement**  : applique ce mappage uniquement aux actions de création d’utilisateur.

6. Pour enregistrer vos mappages, cliquez sur **Enregistrer** en haut de la section Mappage d’attributs.


## <a name="enable-and-launch-user-provisioning"></a>Activer et lancer l'approvisionnement des utilisateurs

Une fois les configurations d'application d'approvisionnement Workday effectuées, vous pouvez activer le service d'approvisionnement sur le portail Azure.

> [!TIP]
> Par défaut, lorsque vous activez le service d'approvisionnement, il lance les opérations d'approvisionnement pour tous les utilisateurs concernés. En cas d'erreur de mappage ou de problème lié aux données Workday, le travail d'approvisionnement peut échouer et être mis en quarantaine. Pour éviter ce genre de problème, nous vous recommandons de configurer le filtre **Portée de l'objet source** et de tester vos mappages d'attributs sur quelques utilisateurs test avant de lancer la synchronisation complète de tous les utilisateurs. Après avoir vérifié que les mappages fonctionnent et qu'ils vous donnent les résultats souhaités, vous pouvez supprimer le filtre ou l'étendre progressivement pour inclure d'autres utilisateurs.

1. Dans l’onglet **Approvisionnement** , définissez **État d’approvisionnement** sur **Activé**.

2. Cliquez sur **Enregistrer**.

3. Cette opération permet de lancer la synchronisation initiale, dont la durée dépendra du nombre d'utilisateurs du locataire Workday. Vous pouvez consulter la barre de progression pour suivre la progression du cycle de synchronisation. 

4. À tout moment, consultez l’onglet **Journaux d’audit** dans le portail Azure pour connaître les actions effectuées par le service d’approvisionnement. Les journaux d’audit répertorient tous les événements de synchronisation individuels effectués par le service d’approvisionnement, tels que les utilisateurs lus dans Workday et par la suite ajoutés ou mis à jour dans Azure Active Directory. 

5. Au terme de la synchronisation initiale, un rapport de synthèse d'audit est créé dans l'onglet **Approvisionnement** , comme illustré ci-dessous.

   > [!div class="mx-imgBorder"]
   > ![Barre de progression de provisionnement](./media/sap-successfactors-inbound-provisioning/prov-progress-bar-stats.png)

## <a name="next-steps"></a>Étapes suivantes

* [En savoir plus sur les attributs Workday pris en charge pour l'approvisionnement entrant](../app-provisioning/workday-attribute-reference.md)
* [Découvrez comment configurer Workday Writeback](workday-writeback-tutorial.md)
* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)
* [Découvrir comment configurer l’authentification unique entre Workday et Azure Active Directory](workday-tutorial.md)
* [Découvrir comment intégrer d’autres applications SaaS à Azure Active Directory](tutorial-list.md)
* [Découvrez comment exporter et importer vos configurations de provisionnement](../app-provisioning/export-import-provisioning-configuration.md)



---
title: 'Tutoriel : Configurer Workday pour l’attribution automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer Azure Active Directory pour provisionner et retirer automatiquement des comptes d’utilisateur sur Workday.
services: active-directory
author: cmmdesai
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.topic: tutorial
ms.workload: identity
ms.date: 05/26/2020
ms.author: chmutali
ms.openlocfilehash: 5cbfdd57ebd25da013bfb82b761839b1e74ee012
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97609018"
---
# <a name="tutorial-configure-workday-for-automatic-user-provisioning"></a>Tutoriel : Configurer Workday pour l'approvisionnement automatique d'utilisateurs

Ce tutoriel a pour objectif d’expliquer les étapes à suivre pour approvisionner des profils d’employés de Workday vers Azure Active Directory (AD) local.

>[!NOTE]
>Utilisez ce tutoriel si les utilisateurs que vous souhaitez provisionner à partir de Workday ont besoin d’un compte AD local et d’un compte Azure AD. 
>* Si les utilisateurs de Workday ont seulement besoin d’un compte Azure AD (utilisateurs cloud uniquement), consultez le tutoriel sur la [configuration de l’attribution d’utilisateurs de Workday vers Azure AD](workday-inbound-cloud-only-tutorial.md). 
>* Pour configurer l’écriture différée d’attributs tels que l’adresse e-mail, le nom d’utilisateur et le numéro de téléphone d’Azure AD vers Workday, consultez le tutoriel sur la [configuration de l’écriture différée Workday](workday-writeback-tutorial.md).

## <a name="overview"></a>Vue d’ensemble

Le [service d’approvisionnement utilisateur Azure Active Directory](../app-provisioning/user-provisioning.md) s’intègre aux [API de ressources humaines Workday](https://community.workday.com/sites/default/files/file-hosting/productionapi/Human_Resources/v21.1/Get_Workers.html) afin d’approvisionner des comptes d’utilisateur. Les flux de travail d’approvisionnement de l’utilisateur Workday pris en charge par le service d’approvisionnement de l’utilisateur Azure AD autorisent l’automatisation des scénarios de gestion du cycle de vie des identités et des ressources humaines suivants :

* **Nouvelles embauches** : lorsqu’un nouvel employé est ajouté à Workday, un compte d’utilisateur est automatiquement créé dans Active Directory, Azure Active Directory et, éventuellement, Microsoft 365 et [d’autres applications SaaS prises en charge par Azure AD](../app-provisioning/user-provisioning.md), avec l’écriture différée des informations de contact gérées par informatique pour Workday.

* **Mises à jour du profil et des attributs de l’employé** : lorsqu’un enregistrement d’employé est mis à jour dans Workday (par exemple, le nom, le titre ou le responsable), son compte d’utilisateur est automatiquement mis à jour dans Active Directory, Azure Active Directory et, éventuellement, Microsoft 365 et [d’autres applications SaaS prises en charge par Azure AD](../app-provisioning/user-provisioning.md).

* **Résiliations de contrats d’employé** : lorsque le contrat d’un employé est résilié dans Workday, le compte d’utilisateur est automatiquement désactivé dans Active Directory, Azure Active Directory et, éventuellement, Microsoft 365 et [d’autres applications SaaS prises en charge par Azure AD](../app-provisioning/user-provisioning.md).

* **Employés ré-embauchés** : lorsqu'un employé est ré-embauché dans Workday, son ancien compte peut être automatiquement réactivé ou réapprovisionné (selon votre préférence) dans Active Directory, Azure Active Directory et, éventuellement dans Microsoft 365 et d'[autres applications SaaS prises en charge par Azure AD](../app-provisioning/user-provisioning.md).

### <a name="whats-new"></a>Nouveautés
Cette section capture les améliorations récentes en matière d’intégration Workday. Pour obtenir la liste des mises à jour, modifications planifiées et archives, consultez la page [Nouveautés d’Azure Active Directory](../fundamentals/whats-new.md). 

* **Mai 2020 - Possibilité d’écriture différée des numéros de téléphone vers Workday :** En plus de l’adresse e-mail et du nom d’utilisateur, vous pouvez désormais écrire de manière différée le numéro de téléphone professionnel et le numéro de téléphone mobile d’Azure AD vers Workday. Pour plus d’informations, consultez le [didacticiel de l’application d’écriture différée](workday-writeback-tutorial.md).

* **Avril 2020 - Prise en charge de la dernière version de l’API Workday Web Services (WWS) :** Deux fois par an, en mars et septembre, Workday propose des mises à jour riches en fonctionnalités qui vous aident à répondre aux besoins de votre entreprise et aux demandes de vos employés. Pour suivre les nouvelles fonctionnalités fournies par Workday, vous pouvez désormais spécifier directement la version de l’API WWS que vous souhaitez utiliser dans l’URL de connexion. Pour savoir comment spécifier la version de l’API Workday, reportez-vous à la section sur la [configuration de la connectivité Workday](#part-3-in-the-provisioning-app-configure-connectivity-to-workday-and-active-directory). 

* **Janvier 2020 - Possibilité de définir l’attribut AD accountExpires :** À l’aide de la fonction [NumFromDate](../app-provisioning/functions-for-customizing-application-data.md#numfromdate) vous pouvez désormais mapper des champs de date Workday tels que *EndContractDate* ou *StatusTerminationDate*. 

### <a name="who-is-this-user-provisioning-solution-best-suited-for"></a>À qui cette solution d’attribution d’utilisateurs convient-elle le mieux ?

Cette solution d’attribution d’utilisateurs Workday est idéale pour :

* les organisations qui souhaitent une solution cloud prédéfinie pour l’attribution d’utilisateurs Workday ;

* les organisations qui ont besoin d’une attribution d’utilisateurs directe de Workday à Active Directory, ou Azure Active Directory ;

* les organisations qui imposent l’attribution d’utilisateurs à l’aide de données provenant du module HCM Workday (voir [Get_Workers](https://community.workday.com/sites/default/files/file-hosting/productionapi/Human_Resources/v21.1/Get_Workers.html)) ;

* les organisations qui ont besoin de joindre, de déplacer et de laisser les utilisateurs à synchroniser avec une ou plusieurs unités d’organisation, domaines et forêts Active Directory seulement sur la base d’informations de modifications détectées dans le module Workday HCM (voir [Get_Workers](https://community.workday.com/sites/default/files/file-hosting/productionapi/Human_Resources/v21.1/Get_Workers.html)) ;

* Organisations utilisant Microsoft 365 pour la messagerie

## <a name="solution-architecture"></a>Architecture de la solution

Cette section décrit l’architecture de la solution de provisionnement des utilisateurs de bout en bout pour les environnements hybrides courants. Il existe deux flux connexes :

* **Flux de données RH faisant autorité – de Workday vers Active Directory en local :** Dans ce flux, les événements concernant les employés (comme les nouveaux recrutements, les transferts, les fins de contrat) se produisent d’abord dans le locataire Workday HR cloud, puis les données des événements transitent dans Active Directory local par le biais d’Azure AD et l’agent de provisionnement. Selon l’événement, cela peut provoquer des opérations de création/mise à jour/activation/désactivation dans AD.
* **Flux de réécriture - d’Active Directory en local vers Workday :** Une fois le compte créé dans Active Directory, il est synchronisé avec Azure AD par le biais d’Azure AD Connect et des informations telles que l'e-mail, le nom d’utilisateur et le numéro de téléphone peuvent être réécrites dans Workday.

![Vue d’ensemble](./media/workday-inbound-tutorial/wd_overview.png)

### <a name="end-to-end-user-data-flow"></a>Flux de données utilisateur de bout en bout

1. L’équipe RH effectue des transactions sur les employés (entrants/changements de poste/sortants ou recrutements/transferts/arrêts) dans Workday HCM
2. Le service Azure AD Provisioning effectue des synchronisations planifiées des identités à partir de Workday HR et identifie les modifications qui doivent être traitées pour la synchronisation avec Active Directory local.
3. Le service Azure AD Provisioning appelle l’agent d’approvisionnement Azure AD Connect local avec une charge utile de demandes contenant des opérations de création/mise à jour/activation/désactivation de compte AD.
4. L’agent de provisionnement Azure AD Connect utilise un compte de service pour ajouter/mettre à jour des données de compte AD.
5. Le moteur Azure AD Connect / AD Sync effectue exécute la synchronisation différentielle pour extraire les mises à jour dans AD.
6. Les mises à jour d’Active Directory sont synchronisées avec Azure Active Directory local.
7. Si l'application [Workday Writeback](workday-writeback-tutorial.md) est configurée, elle réécrit des attributs tels que la messagerie, le nom d’utilisateur et le numéro de téléphone dans Workday.

## <a name="planning-your-deployment"></a>Planification de votre déploiement

La configuration de l'attribution d’utilisateurs Workday vers Active Directory nécessite une planification minutieuse couvrant différents aspects tels que :
* Configuration l’agent de provisionnement Azure AD Connect 
* Nombre d’applications d’attribution d’utilisateurs Workday vers Active Directory à déployer
* Sélection de l’ID de correspondance approprié, mappage d’attributs, transformation et filtres d’étendue

Pour obtenir des instructions complètes ainsi que les meilleures pratiques recommandées, consultez le [plan de déploiement cloud des RH](../app-provisioning/plan-cloud-hr-provision.md). 

## <a name="configure-integration-system-user-in-workday"></a>Configurer un utilisateur du système d’intégration dans Workday

En général, tous les connecteurs d'approvisionnement Workday ont besoin des informations d'identification d'un utilisateur du système d'intégration Workday pour se connecter à l'API Ressources humaines Workday. Cette section explique comment créer un utilisateur du système d'intégration dans Workday. Elle comprend les sous-sections suivantes :

* [Création d'un utilisateur du système d'intégration](#creating-an-integration-system-user)
* [Création d'un groupe de sécurité pour le système d'intégration](#creating-an-integration-security-group)
* [Configuration des autorisations relatives à la stratégie de sécurité du domaine](#configuring-domain-security-policy-permissions)
* [Configuration des autorisations relatives à la stratégie de sécurité des processus métier](#configuring-business-process-security-policy-permissions)
* [Activation des modifications apportées à la stratégie de sécurité](#activating-security-policy-changes)

> [!NOTE]
> Il est possible d’ignorer cette procédure et d’utiliser à la place un compte d’administrateur général Workday en tant que compte d’intégration système. Cette procédure, bien qu’adaptée aux démonstrations, n’est pas recommandée pour les déploiements de production.

### <a name="creating-an-integration-system-user"></a>Création d’un utilisateur système d’intégration

**Pour créer un utilisateur de système d’intégration :**

1. Connectez-vous à votre locataire Workday à l’aide d’un compte administrateur. Dans **Workday Application** (Application Workday), entrez « create user » dans la zone de recherche, puis cliquez sur **Create Integration System User** (Créer un utilisateur du système d’intégration).

   >[!div class="mx-imgBorder"] 
   >![Créer un utilisateur](./media/workday-inbound-tutorial/wd_isu_01.png "Créer un utilisateur")
2. Exécutez la tâche **Create Integration System User** en fournissant un nom d’utilisateur et un mot de passe pour un nouvel utilisateur du système d’intégration.  

   * Laissez l’option **Require New Password at Next Sign In** désactivée, étant donné que cet utilisateur se connectera par programmation.
   * Conservez la valeur par défaut de 0 pour l’option **Délai d’expiration de session (minutes)** afin d’éviter que les sessions de l’utilisateur expirent prématurément.
   * Sélectionnez l’option **Do Not Allow UI Sessions** (Ne pas autoriser les sessions d’interface utilisateur), car elle fournit une couche de sécurité supplémentaire qui empêche un utilisateur ayant le mot de passe du système d’intégration de se connecter à Workday.

   > [!div class="mx-imgBorder"]
   > ![Créer un utilisateur de système d’intégration](./media/workday-inbound-tutorial/wd_isu_02.png "Create Integration System User")

### <a name="creating-an-integration-security-group"></a>Création d'un groupe de sécurité pour le système d'intégration

Au cours de cette étape, vous allez créer dans Workday un groupe de sécurité avec ou sans contraintes, puis attribuer à ce groupe l'utilisateur du système d'intégration créé à l'étape précédente.

**Pour créer un groupe de sécurité :**

1. Entrez create security group dans la zone de recherche, puis cliquez sur **Create Security Group**.

   > [!div class="mx-imgBorder"]
   > ![Capture d'écran dans laquelle le texte « create security group » et le texte « Create Security Group - Task » apparaissent respectivement dans la zone de recherche et dans les résultats de la recherche.](./media/workday-inbound-tutorial/wd_isu_03.png)
2. Exécutez la tâche **Create Security Group**. 

   * Il existe deux types de groupes de sécurité dans Workday :
     * **Sans contraintes :** les membres du groupe de sécurité ont accès à toutes les instances de données sécurisées par celui-ci.
     * **Avec contraintes :** les membres du groupe de sécurité bénéficient d'un accès contextuel à un sous-ensemble d'instances de données (lignes) auxquelles le groupe de sécurité a accès.
   * Consultez votre partenaire d'intégration Workday afin de sélectionner le type de groupe de sécurité qui convient le mieux à l'intégration.
   * Une fois le type de groupe choisi, sélectionnez **Groupe de sécurité du système d'intégration (sans contraintes)** ou **Groupe de sécurité du système d'intégration (avec contraintes)** dans le menu déroulant **Type de groupe de sécurité avec locataire pris en charge**.

     > [!div class="mx-imgBorder"]
     >![Créer un groupe de sécurité](./media/workday-inbound-tutorial/wd_isu_04.png "Créer un groupe de sécurité")

3. Une fois le groupe de sécurité créé, vous voyez une page où vous pouvez lui affecter des membres. Ajoutez le nouvel utilisateur du système d'intégration créé à l'étape précédente à ce groupe de sécurité. Si vous utilisez un groupe de sécurité *avec contraintes*, vous devez également sélectionner la portée appropriée de l'organisation.

   >[!div class="mx-imgBorder"]
   >![Modifier un groupe de sécurité](./media/workday-inbound-tutorial/wd_isu_05.png "Edit Security Group")

### <a name="configuring-domain-security-policy-permissions"></a>Configuration des autorisations relatives à la stratégie de sécurité du domaine

Dans cette étape, vous accordez au groupe de sécurité des autorisations de stratégie de « sécurité du domaine » pour les données des employés.

**Pour configurer des autorisations de stratégie de sécurité du domaine :**

1. Entrez **Domain Security Configuration** (Configuration de la sécurité du domaine) dans la zone de recherche, puis cliquez sur le lien **Domain Security Configuration Report** (Rapport sur la configuration de la sécurité du domaine).  
   >[!div class="mx-imgBorder"]
   >![Capture d'écran dans laquelle le texte « domain security configuration » et le texte « Domain Security Configuration - Report » apparaissent respectivement dans la zone de recherche et dans les résultats.](./media/workday-inbound-tutorial/wd_isu_06.png "Stratégies de sécurité de domaine")  
2. Dans la zone de texte **Domain** (Domaine), recherchez les domaines suivants et ajoutez-les au filtre un par un.  
   * *External Account Provisioning*
   * *Worker Data: Workers*
   * *Worker Data: Public Worker Reports*
   * *Person Data: Work Contact Information*
   * *Worker Data: All Positions*
   * *Worker Data: Current Staffing Information*
   * *Worker Data: Business Title on Worker Profile*
   * *Comptes Workday*

     >[!div class="mx-imgBorder"]
     >![Capture d'écran représentant le rapport sur la configuration de la sécurité du domaine, dans lequel « External Account » apparaît dans la zone de texte « Domain ».](./media/workday-inbound-tutorial/wd_isu_07.png "Stratégies de sécurité de domaine")  

     >[!div class="mx-imgBorder"]
     >![Capture d'écran représentant le rapport sur la configuration de la sécurité du domaine, dans lequel une liste de domaines est sélectionnée.](./media/workday-inbound-tutorial/wd_isu_08.png "Stratégies de sécurité de domaine") 

     Cliquez sur **OK**.

3. Dans le rapport qui s’affiche, sélectionnez les points de suspension (...) qui apparaissent en regard de **External Account Provisioning** (Provisionnement de compte externe), puis cliquez sur l’option de menu **Domain -> Edit Security Policy Permissions** (Domaine -> Modifier les autorisations de la stratégie de sécurité)
   >[!div class="mx-imgBorder"]
   >![Stratégies de sécurité de domaine](./media/workday-inbound-tutorial/wd_isu_09.png "Stratégies de sécurité de domaine")  

4. Sur la page **Edit Domain Security Policy Permissions** (Modifier les autorisations de la stratégie de sécurité), faites défiler jusqu’à la section **Integration Permissions** (Autorisations de l’intégration). Cliquez sur le signe « + » pour ajouter le groupe du système d’intégration à la liste des groupes de sécurité avec les autorisations d’intégration **Get** (Obtenir) et **Put** (Placer).
   >[!div class="mx-imgBorder"]
   >![Capture d'écran dans laquelle la section « Integration Permissions » est en surbrillance.](./media/workday-inbound-tutorial/wd_isu_10.png "Modifier une autorisation")  

5. Cliquez sur le signe « + » pour ajouter le groupe du système d’intégration à la liste des groupes de sécurité avec les autorisations d’intégration **Get** (Obtenir) et **Put** (Placer).

   >[!div class="mx-imgBorder"]
   >![Modifier une autorisation](./media/workday-inbound-tutorial/wd_isu_11.png "Modifier une autorisation")  

6. Répétez les étapes 3 à 5 ci-dessus pour chacune de ces stratégies de sécurité restantes :

   | Opération | Stratégie de sécurité du domaine |
   | ---------- | ---------- |
   | Get et Put | Worker Data: Public Worker Reports |
   | Get et Put | Person Data: Work Contact Information |
   | Obtenir | Worker Data: Workers |
   | Obtenir | Worker Data: All Positions |
   | Obtenir | Worker Data: Current Staffing Information |
   | Obtenir | Worker Data: Business Title on Worker Profile |
   | Get et Put | Comptes Workday |

### <a name="configuring-business-process-security-policy-permissions"></a>Configuration des autorisations relatives à la stratégie de sécurité des processus métier

Dans cette étape, vous accordez des autorisations de stratégies de sécurité « sécurité des processus métier » pour les données des employés au groupe de sécurité. 

> [!NOTE]
> Cette étape est uniquement nécessaire pour configurer le connecteur de l'application de réécriture Workday.

**Pour configurer des autorisations de stratégie de sécurité de processus métier :**

1. Entrez **Business Process Policy** (Stratégie de processus métier) dans la zone de recherche, puis cliquez sur le lien vers la tâche **Edit Business Process Security Policy** (Modifier la stratégie de sécurité de processus métier).  

   >[!div class="mx-imgBorder"]
   >![Capture d'écran où « Business Process Policy » est entré dans la zone de recherche et où « Edit Business Process Security Policy » est sélectionné.](./media/workday-inbound-tutorial/wd_isu_12.png "Stratégies de sécurité de processus métier")  

2. Dans la zone de texte **Business Process Type** (Type de processus métier), recherchez *Contact*, sélectionnez le processus métier **Work Contact Change** (Modification du contact professionnel) et cliquez sur **OK**.

   >[!div class="mx-imgBorder"]
   >![Capture d'écran représentant la page « Edit Business Process Security Policy » et où l'option « Work Contact Change » est sélectionnée dans le menu « Business Process Type ».](./media/workday-inbound-tutorial/wd_isu_13.png "Stratégies de sécurité de processus métier")  

3. Sur la page **Edit Business Process Security Policy** (Modifier la stratégie de sécurité de processus métier), faites défiler jusqu’à la section **Change Work Contact Information** (Modifier les informations de contact professionnel (service web)).


4. Sélectionnez et ajoutez le nouveau groupe de sécurité du système d’intégration à la liste des groupes de sécurité qui peuvent lancer la demande de services web. 

   >[!div class="mx-imgBorder"]
   >![Stratégies de sécurité de processus métier](./media/workday-inbound-tutorial/wd_isu_15.png "Stratégies de sécurité de processus métier")  

5. Cliquez sur **Done** (Terminé). 

### <a name="activating-security-policy-changes"></a>Activation des modifications de stratégie de sécurité

**Pour activer les modifications de la stratégie de sécurité :**

1. Entrez activate dans la zone de recherche, puis cliquez sur le lien **Activate Pending Security Policy Changes**.
   >[!div class="mx-imgBorder"]
   >![Activer](./media/workday-inbound-tutorial/wd_isu_16.png "Activer")

1. Commencez la tâche Activate Pending Security Policy Changes en entrant un commentaire à des fins d’audit, puis cliquez sur **OK**.
1. Terminez la tâche sur l’écran suivant en cochant la case **Confirmer**, puis cliquez sur **OK**.

   >[!div class="mx-imgBorder"]
   >![Activer la sécurité en attente](./media/workday-inbound-tutorial/wd_isu_18.png "Activer la sécurité en attente")  

## <a name="configure-active-directory-service-account"></a>Configurer le compte de service Active Directory

Cette section décrit les autorisations de compte de service Active Directory nécessaires à l’installation et à la configuration de l’agent d’approvisionnement Azure AD Connect.

### <a name="permissions-required-to-run-the-provisioning-agent-installer"></a>Autorisations requises pour exécuter le programme d’installation de l’agent d’approvisionnement
Après avoir identifié l’instance Windows Server qui hébergera l’agent d’approvisionnement, connectez-vous à l’hôte du serveur à l’aide des informations d’identification de l’administrateur local ou du domaine. Le processus d’installation de l’agent crée des fichiers d’informations d’identification dans un magasin de clés sécurisé et met à jour la configuration du profil de service sur le serveur hôte. Cela nécessite un accès administrateur au serveur qui héberge l’agent. 

### <a name="permissions-required-to-configure-the-provisioning-agent-service"></a>Autorisations requises pour configurer le service de l’agent d’approvisionnement
Utilisez les étapes ci-dessous pour configurer un compte de service qui peut être utilisé pour les opérations de l’agent d’approvisionnement. 
1. Sur le contrôleur de domaine AD, ouvrez le composant logiciel enfichable *Utilisateurs et ordinateurs Active Directory*. 
2. Créer un utilisateur de domaine (exemple: *provAgentAdmin*)  
3. Cliquez avec le bouton droit sur l’unité d’organisation ou le nom de domaine, puis sélectionnez *Déléguer le contrôle* afin d’ouvrir l’*Assistant Délégation de contrôle*. 

> [!NOTE] 
> Si vous souhaitez limiter l’agent d’approvisionnement de manière à créer et lire uniquement des utilisateurs d’une unité d’organisation à des fins de test, nous vous recommandons de déléguer le contrôle au niveau de l’unité d’organisation qui convient lors des séries de tests.

4. Dans l'écran de bienvenue, cliquez sur **Suivant**. 
5. Dans l’écran **Sélectionner des utilisateurs ou des groupes**, ajoutez l’utilisateur de domaine que vous avez créé à l’étape 2. Cliquez sur **Suivant**.
   >[!div class="mx-imgBorder"]
   >![Écran Ajouter](./media/workday-inbound-tutorial/delegation-wizard-01.png "Écran Ajouter")

6. Dans l’écran **Tâches à déléguer**, sélectionnez les tâches suivantes : 
   * Créer, supprimer et gérer des comptes d’utilisateurs
   * Lire toutes les informations d’utilisateur

   >[!div class="mx-imgBorder"]
   >![Écran Tâches](./media/workday-inbound-tutorial/delegation-wizard-02.png "Écran Tâches")

7. Cliquez sur **Suivant** et **Enregistrer** pour enregistrer la configuration.

## <a name="configuring-user-provisioning-from-workday-to-active-directory"></a>Configuration de l’approvisionnement de l’utilisateur de Workday à Active Directory

Cette section présente les étapes à suivre pour configurer l'approvisionnement des comptes d'utilisateur de Workday vers chacun des domaines Active Directory concernés par votre intégration.

* [Ajouter l’application du connecteur d’approvisionnement et télécharger l’agent d’approvisionnement](#part-1-add-the-provisioning-connector-app-and-download-the-provisioning-agent)
* [Installer et configurer les agents d'approvisionnement locaux](#part-2-install-and-configure-on-premises-provisioning-agents)
* [Configurer la connectivité à Workday et Active Directory](#part-3-in-the-provisioning-app-configure-connectivity-to-workday-and-active-directory)
* [Configurer les mappages d'attributs](#part-4-configure-attribute-mappings)
* [Activer et lancer l'approvisionnement des utilisateurs](#enable-and-launch-user-provisioning)

### <a name="part-1-add-the-provisioning-connector-app-and-download-the-provisioning-agent"></a>Première partie : Ajouter l’application du connecteur d’approvisionnement et télécharger l’agent d’approvisionnement

**Pour configurer l’approvisionnement Workday dans Active Directory :**

1. Atteindre <https://portal.azure.com>.

2. Dans le Portail Azure, recherchez et sélectionnez **Azure Active Directory**.

3. Cliquez sur **Applications d’entreprise**, puis sur **Toutes les applications**.

4. Sélectionnez **Ajouter une application**, puis sélectionnez la catégorie **Tous**.

5. Recherchez **Attribution d’utilisateurs de Workday vers Active Directory** et ajoutez cette application à partir de la galerie.

6. Une fois l’application ajoutée et l’écran de détails de l’application affiché, sélectionnez **Provisionnement**.

7. Définissez le **mode** de **provisionnement** sur **Automatique**

8. Cliquez sur la bannière d’informations affichée pour télécharger l’agent d’approvisionnement. 

   >[!div class="mx-imgBorder"]
   >![Télécharger l’agent](./media/workday-inbound-tutorial/pa-download-agent.png "Écran Télécharger l’agent")

### <a name="part-2-install-and-configure-on-premises-provisioning-agents"></a>Deuxième partie : Installer et configurer des agents de provisionnement locaux

Pour tout approvisionnement vers une instance locale d’Active Directory, l’agent d’approvisionnement doit être installé sur un serveur doté de .NET Framework 4.7.1+ et d’un accès réseau aux domaines Active Directory souhaités.

> [!TIP]
> Vous pouvez vérifier la version du .NET Framework sur votre serveur en suivant les instructions fournies [ici](/dotnet/framework/migration-guide/how-to-determine-which-versions-are-installed).
> Si .NET 4.7.1 ou version ultérieure n'est pas installé sur le serveur, vous pouvez le télécharger [ici](https://support.microsoft.com/help/4033342/the-net-framework-4-7-1-offline-installer-for-windows).  

Transférez le programme d’installation de l’agent téléchargé sur l’hôte du serveur et suivez les étapes indiquées ci-dessous pour terminer la configuration de l’agent.

1. Connectez-vous au serveur Windows Server sur lequel vous souhaitez installer le nouvel agent.

1. Lancez le programme d’installation de l’agent de provisionnement, acceptez les conditions d’utilisation, puis cliquez sur le bouton **Install** (Installer).

   >[!div class="mx-imgBorder"]
   >![Écran Installation](./media/workday-inbound-tutorial/pa_install_screen_1.png "Écran Installation")

1. Une fois l’installation terminée, l’Assistant démarrer et vous voyez l’écran **Connect Azure AD** (Connecter Azure AD). Cliquez sur le bouton **Authenticate** (S’authentifier) pour vous connecter à votre instance Azure AD.

   >[!div class="mx-imgBorder"]
   >![Connecter Azure AD](./media/workday-inbound-tutorial/pa_install_screen_2.png "Connecter Azure AD")

1. Authentifiez-vous auprès de votre instance Azure AD avec les informations d’identification de l’administrateur d’identité hybride.

   >[!div class="mx-imgBorder"]
   >![Authentification de l’administrateur](./media/workday-inbound-tutorial/pa_install_screen_3.png "Authentification de l’administrateur")

   > [!NOTE]
   > Les informations d'identification de l'administrateur Azure AD sont uniquement utilisées pour la connexion à votre locataire Azure AD. L'agent ne stocke pas les informations d'identification sur le serveur local.

1. Après une authentification réussie auprès d’Azure AD, vous voyez l’écran **Connect Active Directory** (Connecter Active Directory). Entrez alors votre nom de domaine AD, puis cliquez sur le bouton **Ajouter un annuaire**.

   >[!div class="mx-imgBorder"]
   >![Ajouter un répertoire](./media/workday-inbound-tutorial/pa_install_screen_4.png "Ajouter un répertoire")

1. Vous êtes maintenant invité à entrer les informations d’identification nécessaires pour vous connecter au domaine AD. Sur le même écran, vous pouvez utiliser **Select domain controller priority** (Sélectionner la priorité de contrôleur de domaine) pour spécifier les contrôleurs de domaine que l’agent doit utiliser pour l’envoi des demandes de provisionnement.

   >[!div class="mx-imgBorder"]
   >![Informations d'identification du domaine](./media/workday-inbound-tutorial/pa_install_screen_5.png)

1. Après avoir configuré le domaine, le programme d’installation affiche une liste des domaines configurés. Dans cet écran, vous pouvez répéter les étapes 5 et 6 pour ajouter d’autres domaines ou cliquer sur **Suivant** pour passer à l’inscription de l’agent.

   >[!div class="mx-imgBorder"]
   >![Domaines configurés](./media/workday-inbound-tutorial/pa_install_screen_6.png "Domaines configurés")

   > [!NOTE]
   > Si vous avez plusieurs domaines AD (par exemple na.contoso.com, emea.contoso.com), ajoutez individuellement chaque domaine à la liste.
   > L'ajout du domaine parent (par exemple, contoso.com) ne suffit pas. Vous devez inscrire chaque domaine enfant auprès de l'agent.

1. Passez en revue les détails de la configuration, puis cliquez sur **Confirm** (Confirmer) pour inscrire l’agent.

   >[!div class="mx-imgBorder"]
   >![Écran Confirmer](./media/workday-inbound-tutorial/pa_install_screen_7.png "Écran Confirmer")

1. L’Assistant Configuration affiche la progression de l’inscription de l’agent.

   >[!div class="mx-imgBorder"]
   >![Inscription de l’agent](./media/workday-inbound-tutorial/pa_install_screen_8.png "Inscription de l’agent")

1. Une fois l’inscription de l’agent réussie, vous pouvez cliquer sur **Exit** (Quitter) pour quitter l’Assistant.

   >[!div class="mx-imgBorder"]
   >![Écran Quitter](./media/workday-inbound-tutorial/pa_install_screen_9.png "Écran Quitter")

1. Vérifiez l’installation de l’agent et assurez-vous qu’il fonctionne en ouvrant le composant logiciel enfichable « Services » et en recherchant le service « Agent d’approvisionnement Microsoft Azure AD Connect ».

   >[!div class="mx-imgBorder"]
   >![Capture d’écran de l’agent d’approvisionnement Microsoft Azure AD Connect en cours d’exécution dans Services.](./media/workday-inbound-tutorial/services.png)

### <a name="part-3-in-the-provisioning-app-configure-connectivity-to-workday-and-active-directory"></a>Troisième partie : Dans l’application d’approvisionnement, configurer la connectivité à Workday et Active Directory
Lors de cette étape, nous allons établir la connectivité avec Workday et Active Directory dans le Portail Azure. 

1. Dans le Portail Azure, revenez à l’application Attribution d’utilisateurs Workday vers Active Directory créée dans la [Partie 1](#part-1-add-the-provisioning-connector-app-and-download-the-provisioning-agent).

1. Fermez la section **Informations d’identification de l’administrateur**, comme suit :

   * **Nom d'utilisateur Workday** : entrez le nom d'utilisateur du compte du système d'intégration Workday, avec le nom de domaine du locataire. Le résultat doit ressembler à : **nom d’utilisateur\@nom_locataire**

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
     > <br> Si vous utilisez une API WWS v30.0, avant d’activer le travail d’approvisionnement, mettez à jour les **expressions API XPATH** sous **Mappage d’attributs -> Options avancées -> Modifier la liste d’attributs de Workday** en vous reportant à la section [Gestion de votre configuration](#managing-your-configuration) et aux [Informations de référence sur l’attribut Workday](../app-provisioning/workday-attribute-reference.md#xpath-values-for-workday-web-services-wws-api-v30).  

   * **Forêt Active Directory :** « nom » de votre domaine Active Directory, tel qu'il est inscrit auprès de l'agent. Utilisez le menu déroulant pour sélectionner le domaine cible à approvisionner. Cette valeur correspond généralement à une chaîne de type : *contoso.com*

   * **Conteneur Active Directory :** entrez le nom unique du conteneur où l’agent doit créer des comptes d’utilisateur par défaut.
        Exemple : *OU=Standard Users,OU=Users,DC=contoso,DC=test*

     > [!NOTE]
     > Ce paramètre concerne seulement les créations de comptes d’utilisateur si l’attribut *parentDistinguishedName* attribut n’est pas configuré dans les mappages d’attributs. Ce paramètre n’est pas utilisé pour la recherche d’utilisateurs ni pour les opérations de mise à jour. Toute la sous-arborescence du domaine se trouve dans l’étendue de l’opération de recherche.

   * **E-mail de notification :** entrez votre adresse e-mail et activez la case à cocher « Envoyer un e-mail en cas de défaillance ».

     > [!NOTE]
     > Le service Azure AD Provisioning envoie la notification par e-mail si le travail de provisionnement passe à l’état [Mise en quarantaine](../app-provisioning/application-provisioning-quarantine-status.md).

   * Cliquez sur le bouton **Tester la connexion**. Si le test de connexion aboutit, cliquez sur le bouton **Enregistrer**, en haut de l'écran. En cas d’échec, vérifiez que les informations d’identification de Workday et celles d’Active Directory configurées sur le programme d’installation de l’agent sont valides.

     >[!div class="mx-imgBorder"]
     >![Capture d'écran représentant la page « Approvisionnement », sur laquelle les informations d'identification sont entrées.](./media/workday-inbound-tutorial/wd_1.png)

   * Une fois les informations d'identification enregistrées, la section **Mappages** affiche le mappage par défaut **Synchroniser les employés Workday avec l'instance locale d'Active Directory**.

### <a name="part-4-configure-attribute-mappings"></a>Partie 4 : Configuration des mappages d’attributs

Dans cette section, vous allez configurer le flux des données de l’utilisateur de Workday vers Active Directory.

1. Dans l'onglet Approvisionnement, sous **Mappages**, cliquez sur **Synchroniser les employés Workday avec l'instance locale d'Active Directory**.

1. Dans le champ **Portée de l'objet source**, vous pouvez sélectionner les ensembles d'utilisateurs de Workday concernés par l'approvisionnement vers AD, en définissant des filtres basés sur des attributs. L’étendue par défaut est « tous les utilisateurs de Workday ». Exemples de filtres :

   * Exemple : Étendue pour les utilisateurs avec des ID d’employés entre 1000000 et 2000000 (2000000 exclus)

      * Attribut : WorkerID

      * Opérateur : REGEX Match

      * Valeur : (1[0-9][0-9][0-9][0-9][0-9][0-9])

   * Exemple : Uniquement les employés et non les employés occasionnels

      * Attribut : EmployeeID

      * Opérateur : IS NOT NULL

   > [!TIP]
   > Lors de la configuration initiale de l'application d'approvisionnement, vous devez tester et vérifier vos mappages d'attributs et expressions pour être sûr d'obtenir le résultat souhaité. Microsoft vous recommande d'utiliser les filtres d'étendue disponibles sous **Portée de l'objet source** pour tester vos mappages dans Workday avec quelques utilisateurs test. Après avoir vérifié que les mappages fonctionnent, vous pouvez supprimer le filtre ou l'étendre progressivement pour inclure d'autres utilisateurs.

   > [!CAUTION] 
   > Par défaut, le moteur de provisionnement désactive/supprime les utilisateurs qui sortent de l’étendue. Vous pouvez juger ce comportement inopportun dans votre intégration de Workday à AD. Pour remplacer ce comportement par défaut, consultez l’article [Ignorer la suppression des comptes d’utilisateurs qui sortent de l’étendue](../app-provisioning/skip-out-of-scope-deletions.md).

1. Dans le champ **Actions de l'objet cible**, vous pouvez filtrer globalement les actions exécutées sur Active Directory. Les actions **Créer** et **Mettre à jour** sont les plus courantes.

1. Dans la section **Mappages d’attributs**, vous pouvez définir comment les attributs Workday sont mappés aux attributs Active Directory.

1. Cliquez sur un mappage d’attributs existants à mettre à jour ou cliquez sur **Ajouter un nouveau mappage** en bas de l’écran pour ajouter de nouveaux mappages. Un mappage d’attribut individuel prend en charge les propriétés suivantes :

      * **Type de mappage**

         * **Direct** : inscrit la valeur de l'attribut Workday dans l'attribut AD, sans aucune modification

         * **Constante** : inscrivez une valeur de chaîne constante et statique dans l'attribut AD

         * **Expression** : vous permet d’écrire une valeur personnalisée dans l’attribut AD, basée sur un ou plusieurs attributs Workday. [Pour plus d’informations, consultez l’article sur les expressions](../app-provisioning/functions-for-customizing-application-data.md).

      * **Attribut source** : l’attribut utilisateur dans Workday. Si l’attribut que vous recherchez n’est pas présent, consultez [Personnalisation de la liste des attributs d’utilisateur Workday](#customizing-the-list-of-workday-user-attributes).

      * **Valeur par défaut** : facultatif. Si l’attribut source a une valeur vide, le mappage écrit cette valeur à la place.
            La configuration la plus courante consiste à laisser ce champ vide.

      * **Attribut cible** : attribut de l'utilisateur dans Active Directory.

      * **Faire correspondre des objets à l’aide de cet attribut** : indique ci ce mappage est utilisé ou pas pour identifier les utilisateurs de manière unique entre Workday et Active Directory. Cette valeur est communément définie dans le champ ID collaborateur de Workday, qui est généralement mappé avec l'un des attributs ID employé d'Active Directory.

      * **Priorité des correspondances** : plusieurs attributs de correspondance peuvent être définis. S’il en existe plusieurs, ils sont évalués dans l’ordre défini par ce champ. Dès qu'une correspondance est trouvée, aucun autre attribut de correspondance n'est évalué.

      * **Appliquer ce mappage**

         * **Toujours** : applique ce mappage à la création de l’utilisateur et des actions de mise à jour.

         * **Lors de la création uniquement** : applique ce mappage uniquement aux actions de création d’utilisateur.

1. Pour enregistrer vos mappages, cliquez sur **Enregistrer** en haut de la section Mappage d'attributs.
   >[!div class="mx-imgBorder"]
   >![Capture d'écran représentant la page « Mappages d'attributs », sur laquelle l'action « Enregistrer » est sélectionnée.](./media/workday-inbound-tutorial/wd_2.png)

#### <a name="below-are-some-example-attribute-mappings-between-workday-and-active-directory-with-some-common-expressions"></a>Vous trouverez ci-dessous des exemples de mappages d'attributs entre Workday et Active Directory, avec quelques expressions communes

* L'expression mappée avec l'attribut *parentDistinguishedName* permet d'attribuer un utilisateur à différentes unités d'organisation en fonction d'un ou plusieurs attributs sources Workday. Cet exemple les place en fonction de la ville dans laquelle il se trouve.

* L'attribut *userPrincipalName* d'Active Directory est généré à l'aide de la fonction de déduplication [SelectUniqueValue](../app-provisioning/functions-for-customizing-application-data.md#selectuniquevalue) qui recherche l'existence d'une valeur générée dans le domaine AD cible et ne la définit que si elle est unique.  

* Vous pouvez consulter la [documentation sur l’écriture d’expressions](../app-provisioning/functions-for-customizing-application-data.md), Cette section comprend des exemples de suppression de caractères spéciaux.

| ATTRIBUT WORKDAY | ATTRIBUT ACTIVE DIRECTORY |  CORRESPONDANCE D’ID ? | CRÉER / METTRE À JOUR |
| ---------- | ---------- | ---------- | ---------- |
| **WorkerID**  |  EmployeeID | **Oui** | Écrit lors de la création uniquement |
| **PreferredNameData**    |  cn    |   |   Écrit lors de la création uniquement |
| **SelectUniqueValue( Join("\@", Join(".",  \[FirstName\], \[LastName\]), "contoso.com"), Join("\@", Join(".",  Mid(\[FirstName\], 1, 1), \[LastName\]), "contoso.com"), Join("\@", Join(".",  Mid(\[FirstName\], 1, 2), \[LastName\]), "contoso.com"))**   | userPrincipalName     |     | Écrit lors de la création uniquement 
| `Replace(Mid(Replace(\[UserID\], , "(\[\\\\/\\\\\\\\\\\\\[\\\\\]\\\\:\\\\;\\\\\|\\\\=\\\\,\\\\+\\\\\*\\\\?\\\\&lt;\\\\&gt;\])", , "", , ), 1, 20), , "([\\\\.)\*\$](file:///\\.)*$)", , "", , )`      |    sAMAccountName            |     |         Écrit lors de la création uniquement |
| **Switch(\[Active\], , "0", "True", "1", "False")** |  accountDisabled      |     | Créer + mettre à jour |
| **FirstName**   | givenName       |     |    Créer + mettre à jour |
| **LastName**   |   sn   |     |  Créer + mettre à jour |
| **PreferredNameData**  |  displayName |     |   Créer + mettre à jour |
| **Société**         | société   |     |  Créer + mettre à jour |
| **SupervisoryOrganization**  | department  |     |  Créer + mettre à jour |
| **ManagerReference**   | manager  |     |  Créer + mettre à jour |
| **BusinessTitle**   |  title     |     |  Créer + mettre à jour | 
| **AddressLineData**    |  streetAddress  |     |   Créer + mettre à jour |
| **Municipalité**   |   l   |     | Créer + mettre à jour |
| **CountryReferenceTwoLetter**      |   co |     |   Créer + mettre à jour |
| **CountryReferenceTwoLetter**    |  c  |     |         Créer + mettre à jour |
| **CountryRegionReference** |  st     |     | Créer + mettre à jour |
| **WorkSpaceReference** | physicalDeliveryOfficeName    |     |  Créer + mettre à jour |
| **PostalCode**  |   postalCode  |     | Créer + mettre à jour |
| **PrimaryWorkTelephone**  |  telephoneNumber   |     | Créer + mettre à jour |
| **Fax**      | facsimileTelephoneNumber     |     |    Créer + mettre à jour |
| **Mobile**  |    mobile       |     |       Créer + mettre à jour |
| **LocalReference** |  preferredLanguage  |     |  Créer + mettre à jour |                                               
| **Switch(\[Municipality\], "OU=Default Users,DC=contoso,DC=com", "Dallas", "OU=Dallas,OU=Users,DC=contoso,DC=com", "Austin", "OU=Austin,OU=Users,DC=contoso,DC=com", "Seattle", "OU=Seattle,OU=Users,DC=contoso,DC=com", "London", "OU=London,OU=Users,DC=contoso,DC=com")**  | parentDistinguishedName     |     |  Créer + mettre à jour |

Une fois vos mappages d'attributs configurés, vous pouvez [activer et lancer le service d'approvisionnement d'utilisateurs](#enable-and-launch-user-provisioning).

## <a name="enable-and-launch-user-provisioning"></a>Activer et lancer l'approvisionnement des utilisateurs

Une fois les configurations d'application d'approvisionnement Workday effectuées, vous pouvez activer le service d'approvisionnement sur le portail Azure.

> [!TIP]
> Par défaut, lorsque vous activez le service d'approvisionnement, il lance les opérations d'approvisionnement pour tous les utilisateurs concernés. En cas d'erreur de mappage ou de problème lié aux données Workday, le travail d'approvisionnement peut échouer et être mis en quarantaine. Pour éviter ce genre de problème, nous vous recommandons de configurer le filtre **Portée de l'objet source** et de tester vos mappages d'attributs sur quelques utilisateurs test avant de lancer la synchronisation complète de tous les utilisateurs. Après avoir vérifié que les mappages fonctionnent et qu'ils vous donnent les résultats souhaités, vous pouvez supprimer le filtre ou l'étendre progressivement pour inclure d'autres utilisateurs.

1. Dans l’onglet **Approvisionnement**, définissez **État d’approvisionnement** sur **Activé**.

2. Cliquez sur **Enregistrer**.

3. Cette opération permet de lancer la synchronisation initiale, dont la durée dépendra du nombre d'utilisateurs du locataire Workday. 

4. À tout moment, consultez l’onglet **Journaux d’audit** dans le portail Azure pour connaître les actions effectuées par le service d’approvisionnement. Les journaux d’audit répertorient tous les événements de synchronisation individuels effectués par le service d’approvisionnement, tels que les utilisateurs lus dans Workday et par la suite ajoutés ou mis à jour dans Active Directory. Reportez-vous à la section Résolution des problèmes pour savoir comment consulter les journaux d’audit et corriger les erreurs d’approvisionnement.

5. Au terme de la synchronisation initiale, un rapport de synthèse d'audit est créé dans l'onglet **Approvisionnement**, comme illustré ci-dessous.
   > [!div class="mx-imgBorder"]
   > ![Barre de progression de provisionnement](./media/sap-successfactors-inbound-provisioning/prov-progress-bar-stats.png)

## <a name="frequently-asked-questions-faq"></a>Forum Aux Questions (FAQ)

* **Questions relatives aux fonctionnalités de la solution**
  * [Lors du traitement d'un nouvel employé à partir de Workday, comment la solution définit-elle le mot de passe du nouveau compte d'utilisateur dans Active Directory ?](#when-processing-a-new-hire-from-workday-how-does-the-solution-set-the-password-for-the-new-user-account-in-active-directory)
  * [La solution prend-elle en charge l'envoi de notifications par e-mail au terme des opérations d'approvisionnement ?](#does-the-solution-support-sending-email-notifications-after-provisioning-operations-complete)
  * [Comment gérer la remise des mots de passe aux nouveaux employés et leur fournir un mécanisme sécurisé pour la réinitialisation de leur mot de passe ?](#how-do-i-manage-delivery-of-passwords-for-new-hires-and-securely-provide-a-mechanism-to-reset-their-password)
  * [La solution met-t-elle les profils utilisateur Workday en cache dans le cloud Azure AD ou au niveau de la couche de l'agent d'approvisionnement ?](#does-the-solution-cache-workday-user-profiles-in-the-azure-ad-cloud-or-at-the-provisioning-agent-layer)
  * [La solution prend-elle en charge l'attribution de groupes AD locaux à l'utilisateur ?](#does-the-solution-support-assigning-on-premises-ad-groups-to-the-user)
  * [Quelles API Workday la solution utilise-t-elle pour interroger et mettre à jour les profils d'employé Workday ?](#which-workday-apis-does-the-solution-use-to-query-and-update-workday-worker-profiles)
  * [Puis-je configurer mon locataire Workday HCM avec deux locataires Azure AD ?](#can-i-configure-my-workday-hcm-tenant-with-two-azure-ad-tenants)
  * [Pourquoi l'application d'approvisionnement d'utilisateurs « Workday vers Azure AD » n'est-elle pas prise en charge si Azure AD Connect a été déployé ?](#why-workday-to-azure-ad-user-provisioning-app-is-not-supported-if-we-have-deployed-azure-ad-connect)
  * [Comment suggérer des améliorations ou réclamer de nouvelles fonctionnalités en lien avec l'intégration de Workday et Azure AD ?](#how-do-i-suggest-improvements-or-request-new-features-related-to-workday-and-azure-ad-integration)

* **Questions relatives à l'agent d'approvisionnement**
  * [Quelle est la version mise à la disposition générale de l'agent d'approvisionnement ?](#what-is-the-ga-version-of-the-provisioning-agent)
  * [Comment connaître la version de mon agent d'approvisionnement ?](#how-do-i-know-the-version-of-my-provisioning-agent)
  * [Microsoft fournit-il automatiquement les mises à jour de l'agent d'approvisionnement ?](#does-microsoft-automatically-push-provisioning-agent-updates)
  * [Puis-je installer l’agent d’approvisionnement sur le serveur qui exécute Azure AD Connect ?](#can-i-install-the-provisioning-agent-on-the-same-server-running-azure-ad-connect)
  * [Comment configurer l'agent d'approvisionnement afin qu'il utilise un serveur proxy pour la communication HTTP sortante ?](#how-do-i-configure-the-provisioning-agent-to-use-a-proxy-server-for-outbound-http-communication)
  * [Comment savoir que l'agent d'approvisionnement est en mesure de communiquer avec le locataire Azure AD et qu'aucun pare-feu ne bloque les ports requis par l'agent ?](#how-do-i-ensure-that-the-provisioning-agent-is-able-to-communicate-with-the-azure-ad-tenant-and-no-firewalls-are-blocking-ports-required-by-the-agent)
  * [Comment désinscrire le domaine associé à mon agent d'approvisionnement ?](#how-do-i-de-register-the-domain-associated-with-my-provisioning-agent)
  * [Comment désinstaller l'agent d'approvisionnement ?](#how-do-i-uninstall-the-provisioning-agent)

* **Questions relatives à la configuration et au mappage des attributs de Workday vers AD**
  * [Comment sauvegarder ou exporter une copie de travail de mon schéma et de mon mappage d'attributs d'approvisionnement Workday ?](#how-do-i-back-up-or-export-a-working-copy-of-my-workday-provisioning-attribute-mapping-and-schema)
  * [Je dispose d'attributs personnalisés dans Workday et Active Directory. Comment configurer la solution pour qu'elle fonctionne avec mes attributs personnalisés ?](#i-have-custom-attributes-in-workday-and-active-directory-how-do-i-configure-the-solution-to-work-with-my-custom-attributes)
  * [Puis-je transférer une photo de l'utilisateur de Workday vers Active Directory ?](#can-i-provision-users-photo-from-workday-to-active-directory)
  * [Comment synchroniser des numéros de téléphone mobile à partir de Workday avec consentement des utilisateurs pour un usage public ?](#how-do-i-sync-mobile-numbers-from-workday-based-on-user-consent-for-public-usage)
  * [Comment procéder à la mise en forme des noms d’affichage AD en fonction des attributs service/pays/ville de l’utilisateur et gérer les écarts régionaux ?](#how-do-i-format-display-names-in-ad-based-on-the-users-departmentcountrycity-attributes-and-handle-regional-variances)
  * [Comment utiliser SelectUniqueValue afin de générer des valeurs uniques pour l'attribut samAccountName ?](#how-can-i-use-selectuniquevalue-to-generate-unique-values-for-samaccountname-attribute)
  * [Comment supprimer des caractères comportant des signes diacritiques et les convertir en lettres normales de l'alphabet anglais ?](#how-do-i-remove-characters-with-diacritics-and-convert-them-into-normal-english-alphabets)

### <a name="solution-capability-questions"></a>Questions relatives aux fonctionnalités de la solution

#### <a name="when-processing-a-new-hire-from-workday-how-does-the-solution-set-the-password-for-the-new-user-account-in-active-directory"></a>Lors du traitement d'un nouvel employé à partir de Workday, comment la solution définit-elle le mot de passe du nouveau compte d'utilisateur dans Active Directory ?

Lorsque l'agent d'approvisionnement local reçoit une demande de création d'un compte AD, il génère automatiquement un mot de passe aléatoire complexe, conçu pour répondre aux exigences de complexité du mot de passe définies par le serveur AD, et le définit sur l'objet utilisateur. Ce mot de passe n'est enregistré nulle part.

#### <a name="does-the-solution-support-sending-email-notifications-after-provisioning-operations-complete"></a>La solution prend-elle en charge l'envoi de notifications par e-mail au terme des opérations d'approvisionnement ?

Non, l'envoi de notifications par e-mail au terme des opérations d'approvisionnement n'est pas pris en charge dans la version actuelle.

#### <a name="how-do-i-manage-delivery-of-passwords-for-new-hires-and-securely-provide-a-mechanism-to-reset-their-password"></a>Comment gérer la remise des mots de passe aux nouveaux employés et leur fournir un mécanisme sécurisé pour la réinitialisation de leur mot de passe ?

L’une des dernières étapes du processus d’approvisionnement d’un nouveau compte AD est la remise du mot de passe temporaire attribué au compte AD de l’utilisateur. Beaucoup d’entreprises utilisent encore l’approche traditionnelle qui consiste à remettre le mot de passe temporaire au responsable de l’utilisateur, qui le remet à son tour au nouvel employé ou à l’intérimaire. Mais compte tenu de la faille de sécurité inhérente à ce processus, vous pouvez adopter une approche plus efficace en utilisant les fonctionnalités d'Azure AD.

Dans le cadre du processus de recrutement, les équipes des ressources humaines vérifient généralement les références ainsi que le numéro de téléphone mobile du nouvel employé. Avec l'intégration de l'approvisionnement d'utilisateurs de Workday vers AD, vous pouvez vous appuyer sur cette vérification des RH et déployer une fonctionnalité de réinitialisation de mot de passe en libre-service que l'utilisateur pourra utiliser le jour de sa prise de fonction. Pour ce faire, il convient de propager l’attribut « Numéro de téléphone mobile » du nouvel employé de Workday vers AD, puis d’AD vers Azure AD à l’aide d’Azure AD Connect. Une fois le « Numéro de téléphone mobile » présent dans Azure AD, vous pouvez activer la [Réinitialisation du mot de passe libre-service (SSPR)](../authentication/howto-sspr-authenticationdata.md) sur le compte de l’utilisateur. Ainsi, le jour de sa prise de fonction, le nouvel employé pourra utiliser le numéro enregistré et vérifié pour s’authentifier.

#### <a name="does-the-solution-cache-workday-user-profiles-in-the-azure-ad-cloud-or-at-the-provisioning-agent-layer"></a>La solution met-t-elle les profils utilisateur Workday en cache dans le cloud Azure AD ou au niveau de la couche de l'agent d'approvisionnement ?

Non, la solution ne met pas les profils utilisateur en cache. Le service d'approvisionnement Azure AD assume simplement le rôle de processeur de données, en lisant les données à partir de Workday et en les écrivant sur l'instance cible d'Active Directory ou d'Azure AD. Reportez-vous à la section [Gestion des données personnelles](#managing-personal-data) pour plus d'informations sur la confidentialité des utilisateurs et sur la conservation des données.

#### <a name="does-the-solution-support-assigning-on-premises-ad-groups-to-the-user"></a>La solution prend-elle en charge l'attribution de groupes AD locaux à l'utilisateur ?

Cette fonctionnalité n'est actuellement pas prise en charge. La solution de contournement recommandée consiste à déployer un script PowerShell qui interroge le point de terminaison de l’API Microsoft Graph pour accéder aux [données du journal d’audit](/graph/api/resources/azure-ad-auditlog-overview?view=graph-rest-beta) et utilise celles-ci pour déclencher des scénarios tels que l’attribution de groupes. Ce script PowerShell peut être associé à un planificateur de tâches et déployé sur le boîtier qui exécute l'agent d'approvisionnement.  

#### <a name="which-workday-apis-does-the-solution-use-to-query-and-update-workday-worker-profiles"></a>Quelles API Workday la solution utilise-t-elle pour interroger et mettre à jour les profils d'employé Workday ?

La solution utilise actuellement les API Workday suivantes :

* Le format de l'**URL Workday Web Services API URL** utilisée dans la section **Informations d'identification de l'administrateur**, détermine la version d'API utilisée pour Get_Workers
  * Si le format d’URL est : https://\#\#\#\#\.workday\.com/ccx/service/tenantName, l'API v21.1 est utilisée. 
  * Si le format d’URL est : https://\#\#\#\#\.workday\.com/ccx/service/tenantName/Human\_Resources, l'API v21.1 est utilisée. 
  * Si le format d’URL est : https://\#\#\#\#\.workday\.com/ccx/service/tenantName/Human\_Resources/v\#\#\.\#, la version d'API spécifiée est utilisée. (Exemple : si v34.0 est spécifiée, elle est utilisée.)  

* La fonctionnalité de réécriture des adresses e-mail Workday utilise Change_Work_Contact_Information (v30.0) 
* La fonctionnalité de réécriture des noms d’utilisateurs Workday utilise Update_Workday_Account (v31.2) 

#### <a name="can-i-configure-my-workday-hcm-tenant-with-two-azure-ad-tenants"></a>Puis-je configurer mon locataire Workday HCM avec deux locataires Azure AD ?

Oui, cette configuration est prise en charge. Voici les étapes générales à suivre pour configurer ce scénario :

* Déployez l'agent d'approvisionnement n°1 et enregistrez-le auprès du locataire Azure AD n°1.
* Déployez l'agent d'approvisionnement n°2 et enregistrez-le auprès du locataire Azure AD n°2.
* En fonction des « domaines enfants » gérés par les agents d'approvisionnement, configurez chaque agent avec les domaines qui conviennent. Un agent peut gérer plusieurs domaines.
* Sur le portail Azure, configurez l'application d'approvisionnement d'utilisateurs de Workday vers AD sur chaque locataire et configurez les domaines respectifs.

#### <a name="why-workday-to-azure-ad-user-provisioning-app-is-not-supported-if-we-have-deployed-azure-ad-connect"></a>Pourquoi l'application d'approvisionnement d'utilisateurs « Workday vers Azure AD » n'est-elle pas prise en charge si Azure AD Connect a été déployé ?

Lorsqu'Azure AD est utilisé en mode hybride (avec un mélange d'utilisateurs cloud et locaux), la « source d'autorité » doit être clairement définie. Généralement, les scénarios hybrides nécessitent le déploiement d'Azure AD Connect. Lorsqu'Azure AD Connect est déployé, l'instance locale d'AD devient la source d'autorité. Si le connecteur Workday vers Azure AD est introduit, les valeurs des attributs Workday risquent d'écraser les valeurs définies par Azure AD Connect. Par conséquent, l'utilisation de l'application d'approvisionnement « Workday vers Azure AD » n'est pas prise en charge lorsqu'Azure AD Connect est activé. Dans ce cas, nous vous recommandons d'utiliser l'application d'approvisionnement « Workday vers AD » pour transférer les utilisateurs vers l'instance locale d'AD avant de les synchroniser avec Azure AD à l'aide d'Azure AD Connect.

#### <a name="how-do-i-suggest-improvements-or-request-new-features-related-to-workday-and-azure-ad-integration"></a>Comment suggérer des améliorations ou réclamer de nouvelles fonctionnalités en lien avec l'intégration de Workday et Azure AD ?

Vos commentaires ont beaucoup de valeur pour nous car ils nous aident à définir une orientation pour nos futures versions et améliorations. Nous vous invitons à nous faire part de vos commentaires et vous encourageons à soumettre vos idées ou suggestions d'amélioration sur le [forum de commentaires d'Azure AD](https://feedback.azure.com/forums/169401-azure-active-directory). Pour accéder à des commentaires existants en lien avec l'intégration de Workday, sélectionnez la catégorie *Applications SaaaS* et lancez une recherche avec le mot clé *Workday*.

> [!div class="mx-imgBorder"]
> ![UserVoice - Applications SaaS](media/workday-inbound-tutorial/uservoice_saas_apps.png)

> [!div class="mx-imgBorder"]
> ![UserVoice - Workday](media/workday-inbound-tutorial/uservoice_workday_feedback.png)

Avant de suggérer une idée, vérifiez que personne ne vous a devancé. Si tel est le cas, vous pouvez voter pour la demande de fonctionnalité ou d'amélioration existante. Vous pouvez également laisser un commentaire sur votre cas d'usage spécifique pour montrer que vous soutenez l'idée et que la fonctionnalité vous sera également utile.

### <a name="provisioning-agent-questions"></a>Questions relatives à l'agent d'approvisionnement

#### <a name="what-is-the-ga-version-of-the-provisioning-agent"></a>Quelle est la version mise à la disposition générale de l'agent d'approvisionnement ?

Consultez [Agent d’approvisionnement Azure AD Connect : Historique de publication des versions](../app-provisioning/provisioning-agent-release-version-history.md) pour la dernière version en disponibilité générale de l’agent d’approvisionnement.  

#### <a name="how-do-i-know-the-version-of-my-provisioning-agent"></a>Comment connaître la version de mon agent d'approvisionnement ?

* Connectez-vous au serveur Windows sur lequel l’agent d’approvisionnement est installé.
* Accédez à **Panneau de configuration** -> **Désinstaller ou modifier un programme**.
* Recherchez la version correspondant à l'entrée **Agent d'approvisionnement Microsoft Azure AD Connect**.

  >[!div class="mx-imgBorder"]
  >![Azure portal](./media/workday-inbound-tutorial/pa_version.png)

#### <a name="does-microsoft-automatically-push-provisioning-agent-updates"></a>Microsoft fournit-il automatiquement les mises à jour de l'agent d'approvisionnement ?

Oui, Microsoft met à jour automatiquement l’agent d’approvisionnement si le service Windows **Microsoft Azure AD Connect Agent Updater** est en cours d’exécution.

#### <a name="can-i-install-the-provisioning-agent-on-the-same-server-running-azure-ad-connect"></a>Puis-je installer l’agent d’approvisionnement sur le serveur qui exécute Azure AD Connect ?

Oui, vous pouvez installer l’agent d’approvisionnement sur le serveur qui exécute Azure AD Connect.

#### <a name="at-the-time-of-configuration-the-provisioning-agent-prompts-for-azure-ad-admin-credentials-does-the-agent-store-the-credentials-locally-on-the-server"></a>Au moment de la configuration, l'agent d'approvisionnement demande les informations d'identification de l'administrateur Azure AD. L'agent stocke-t-il les informations d'identification sur le serveur local ?

Lors de la configuration, l'agent d'approvisionnement demande les informations d'identification de l'administrateur Azure AD, mais uniquement pour se connecter à votre locataire Azure AD. Il ne stocke pas les informations d'identification sur le serveur local. Toutefois, il conserve les informations d'identification utilisées pour se connecter au *domaine Active Directory local* dans un coffre de mots de passe Windows local.

#### <a name="how-do-i-configure-the-provisioning-agent-to-use-a-proxy-server-for-outbound-http-communication"></a>Comment configurer l'agent d'approvisionnement afin qu'il utilise un serveur proxy pour la communication HTTP sortante ?

L'agent d'approvisionnement prend en charge l'utilisation du proxy sortant. Vous pouvez le configurer en modifiant le fichier de configuration **C:\Program Files\Microsoft Azure AD Connect Provisioning Agent\AADConnectProvisioningAgent.exe.config** de l'agent. Ajoutez les lignes suivantes à la fin du fichier, juste avant la balise `</configuration>` de fermeture.
Remplacez les variables [proxy-server] et [proxy-port] par le nom de votre serveur proxy et les valeurs de port.

```xml
    <system.net>
          <defaultProxy enabled="true" useDefaultCredentials="true">
             <proxy
                usesystemdefault="true"
                proxyaddress="http://[proxy-server]:[proxy-port]"
                bypassonlocal="true"
             />
         </defaultProxy>
    </system.net>
```

#### <a name="how-do-i-ensure-that-the-provisioning-agent-is-able-to-communicate-with-the-azure-ad-tenant-and-no-firewalls-are-blocking-ports-required-by-the-agent"></a>Comment savoir que l'agent d'approvisionnement est en mesure de communiquer avec le locataire Azure AD et qu'aucun pare-feu ne bloque les ports requis par l'agent ?

Vous pouvez également vérifier si tous les [ports requis](../manage-apps/application-proxy-add-on-premises-application.md#open-ports) sont ouverts.

#### <a name="can-one-provisioning-agent-be-configured-to-provision-multiple-ad-domains"></a>Un agent d'approvisionnement peut-il être configuré pour approvisionner plusieurs domaines AD ?

Oui, un agent d'approvisionnement peut être configuré pour gérer plusieurs domaines AD à condition de disposer d'une visibilité directe sur les contrôleurs de domaine correspondants. Microsoft recommande de configurer un groupe de 3 agents d'approvisionnement desservant le même ensemble de domaines AD afin de garantir la haute disponibilité et d'assurer la prise en charge du basculement.

#### <a name="how-do-i-de-register-the-domain-associated-with-my-provisioning-agent"></a>Comment désinscrire le domaine associé à mon agent d'approvisionnement ?

* Sur le portail Azure, procurez-vous l'*ID de locataire* de votre locataire Azure AD.
* Connectez-vous au serveur Windows qui exécute l’agent d’approvisionnement.
* Ouvrez PowerShell en tant qu’administrateur Windows.
* Accédez au répertoire contenant les scripts d'inscription et exécutez les commandes suivantes en remplaçant le paramètre \[tenant ID\] par l'ID de votre locataire.

  ```powershell
  cd "C:\Program Files\Microsoft Azure AD Connect Provisioning Agent\RegistrationPowershell\Modules\PSModulesFolder"
  Import-Module "C:\Program Files\Microsoft Azure AD Connect Provisioning Agent\RegistrationPowershell\Modules\PSModulesFolder\AppProxyPSModule.psd1"
  Get-PublishedResources -TenantId "[tenant ID]"
  ```

* Dans la liste des agents qui s’affiche, copiez la valeur du champ `id` de la ressource dont l’attribut *resourceName* correspond au nom de votre domaine AD.
* Collez la valeur ID dans cette commande et exécutez la commande dans PowerShell.

  ```powershell
  Remove-PublishedResource -ResourceId "[resource ID]" -TenantId "[tenant ID]"
  ```

* Réexécutez l'Assistant Configuration de l'agent.
* Tous les autres agents précédemment attribués à ce domaine devront être reconfigurés.

#### <a name="how-do-i-uninstall-the-provisioning-agent"></a>Comment désinstaller l'agent d'approvisionnement ?

* Connectez-vous au serveur Windows sur lequel l’agent d’approvisionnement est installé.
* Accédez à **Panneau de configuration** -> **Désinstaller ou modifier un programme**.
* Désinstallez les programmes suivants :
  * Agent d'approvisionnement Microsoft Azure AD Connect
  * Programme de mise à jour de l'agent Microsoft Azure AD Connect
  * Package Agent d'approvisionnement Microsoft Azure AD Connect

### <a name="workday-to-ad-attribute-mapping-and-configuration-questions"></a>Questions relatives à la configuration et au mappage des attributs de Workday vers AD

#### <a name="how-do-i-back-up-or-export-a-working-copy-of-my-workday-provisioning-attribute-mapping-and-schema"></a>Comment sauvegarder ou exporter une copie de travail de mon schéma et de mon mappage d'attributs d'approvisionnement Workday ?

Vous pouvez utiliser l'API Microsoft Graph pour exporter votre configuration d'approvisionnement d'utilisateurs Workday. Pour plus d'informations, reportez-vous à la procédure de la section [Exporter et importer la configuration de vos mappages d'attributs d'approvisionnement d'utilisateurs Workday](#exporting-and-importing-your-configuration).

#### <a name="i-have-custom-attributes-in-workday-and-active-directory-how-do-i-configure-the-solution-to-work-with-my-custom-attributes"></a>Je dispose d'attributs personnalisés dans Workday et Active Directory. Comment configurer la solution pour qu'elle fonctionne avec mes attributs personnalisés ?

La solution prend en charge les attributs Workday et Active Directory personnalisés. Pour ajouter vos attributs personnalisés au schéma de mappage, ouvrez le panneau **Mappage d'attributs** et faites défiler l'écran vers le bas pour développer la section **Afficher les options avancées**. 

![Modifier la liste des attributs](./media/workday-inbound-tutorial/wd_edit_attr_list.png)

Pour ajouter vos attributs Workday personnalisés, sélectionnez l'option *Modifier la liste des attributs de Workday*, et pour ajouter vos attributs AD personnalisés, sélectionnez l'option *Modifier la liste des attributs de l'instance locale d'Active Directory*.

Voir aussi :

* [Personnaliser la liste des attributs d’utilisateurs Workday](#customizing-the-list-of-workday-user-attributes)

#### <a name="how-do-i-configure-the-solution-to-only-update-attributes-in-ad-based-on-workday-changes-and-not-create-any-new-ad-accounts"></a>Comment configurer la solution pour mettre à jour les attributs AD en fonction des modifications apportées à Workday sans créer de nouveaux comptes AD ?

Cette configuration peut être appliquée en définissant les **Actions de l'objet cible** dans le panneau **Mappages d'attributs**, comme illustré ci-dessous :

![Action de mise à jour](./media/workday-inbound-tutorial/wd_target_update_only.png)

Cochez la case « Mettre à jour » pour que seules les opérations de mise à jour soient transférées de Workday vers AD. 

#### <a name="can-i-provision-users-photo-from-workday-to-active-directory"></a>Puis-je transférer une photo de l'utilisateur de Workday vers Active Directory ?

La solution ne prend actuellement pas en charge la définition des attributs binaires tels que *thumbnailPhoto* et *jpegPhoto* dans Active Directory.

#### <a name="how-do-i-sync-mobile-numbers-from-workday-based-on-user-consent-for-public-usage"></a>Comment synchroniser des numéros de téléphone mobile à partir de Workday avec consentement des utilisateurs pour un usage public ?

* Accédez au panneau « Approvisionnement » de votre application d'approvisionnement Workday.
* Cliquez sur Mappages d'attributs. 
* Sous **Mappages**, sélectionnez **Synchroniser les employés Workday avec l'instance locale d'Active Directory** (ou **Synchroniser les employés Workday avec Azure AD**).
* Sur la page Mappages d'attributs, faites défiler la liste et cochez la case « Afficher les options avancées ».  Cliquez sur **Modifier la liste des attributs de Workday**.
* Dans le panneau qui s’ouvre, localisez l’attribut « Mobile » et cliquez sur la ligne afin de modifier l’**Expression de l’API** ![RGPD mobile](./media/workday-inbound-tutorial/mobile_gdpr.png).

* Remplacez l'**Expression de l'API** par la nouvelle expression suivante, qui ne récupère le numéro de téléphone mobile professionnel que si l'indicateur d'utilisation publique est défini sur « True » dans Workday.

    ```
     wd:Worker/wd:Worker_Data/wd:Personal_Data/wd:Contact_Data/wd:Phone_Data[translate(string(wd:Phone_Device_Type_Reference/@wd:Descriptor),'abcdefghijklmnopqrstuvwxyz','ABCDEFGHIJKLMNOPQRSTUVWXYZ')='MOBILE' and translate(string(wd:Usage_Data/wd:Type_Data/wd:Type_Reference/@wd:Descriptor),'abcdefghijklmnopqrstuvwxyz','ABCDEFGHIJKLMNOPQRSTUVWXYZ')='WORK' and string(wd:Usage_Data/@wd:Public)='1']/@wd:Formatted_Phone
    ```

* Enregistrez la liste des attributs.
* Enregistrez le mappage d'attributs.
* Effacez l'état actuel et redémarrez la synchronisation complète.

#### <a name="how-do-i-format-display-names-in-ad-based-on-the-users-departmentcountrycity-attributes-and-handle-regional-variances"></a>Comment procéder à la mise en forme des noms d’affichage AD en fonction des attributs service/pays/ville de l’utilisateur et gérer les écarts régionaux ?

Il est souvent nécessaire de configurer l’attribut *displayName* d’AD afin qu’il fournisse également des informations sur le service et le pays/la région de l’utilisateur. Par exemple, si John Smith travaille au service Marketing aux États-Unis, vous souhaiterez peut-être que son nom d'affichage (*displayName*) se présente sous la forme *Smith, John (Marketing-US)* .

Voici comment gérer ces exigences pour que la valeur *CN* ou *displayName* inclue des attributs tels que la société, l’unité commerciale, la ville ou le pays/la région.

* Chaque attribut Workday est extrait à l'aide d'une expression sous-jacente de l'API XPATH, configurable sous **Mappage d'attributs -> Section avancée -> Modifier la liste des attributs de Workday** . Voici l'expression par défaut de l'API XPATH pour les attributs Workday *PreferredFirstName*, *PreferredLastName*, *Company* et *SupervisoryOrganization*.

     | Attribut Workday | Expression de l'API XPATH |
     | ----------------- | -------------------- |
     | PreferredFirstName | wd:Worker/wd:Worker_Data/wd:Personal_Data/wd:Name_Data/wd:Preferred_Name_Data/wd:Name_Detail_Data/wd:First_Name/text() |
     | PreferredLastName | wd:Worker/wd:Worker_Data/wd:Personal_Data/wd:Name_Data/wd:Preferred_Name_Data/wd:Name_Detail_Data/wd:Last_Name/text() |
     | Company | wd:Worker/wd:Worker_Data/wd:Organization_Data/wd:Worker_Organization_Data[wd:Organization_Data/wd:Organization_Type_Reference/wd:ID[@wd:type='Organization_Type_ID']='Company']/wd:Organization_Reference/@wd:Descriptor |
     | SupervisoryOrganization | wd:Worker/wd:Worker_Data/wd:Organization_Data/wd:Worker_Organization_Data/wd:Organization_Data[wd:Organization_Type_Reference/wd:ID[@wd:type='Organization_Type_ID']='Supervisory']/wd:Organization_Name/text() |

   Vérifiez auprès de votre équipe Workday que l'expression d'API ci-dessus est valide pour la configuration de votre locataire Workday. Si nécessaire, vous pouvez y apporter des modifications comme décrit dans la section [Personnaliser la liste des attributs d'utilisateurs Workday](#customizing-the-list-of-workday-user-attributes).

* De même, les informations relatives au pays ou à la région présentes dans Workday sont récupérées à l’aide de l’expression XPATH suivante : *wd:Worker/wd:Worker_Data/wd:Employment_Data/wd:Position_Data/wd:Business_Site_Summary_Data/wd:Address_Data/wd:Country_Reference*

     Cinq attributs liés au pays ou à la région sont disponibles dans la section Liste des attributs de Workday.

     | Attribut Workday | Expression de l'API XPATH |
     | ----------------- | -------------------- |
     | CountryReference | wd:Worker/wd:Worker_Data/wd:Employment_Data/wd:Position_Data/wd:Business_Site_Summary_Data/wd:Address_Data/wd:Country_Reference/wd:ID[@wd:type='ISO_3166-1_Alpha-3_Code']/text() |
     | CountryReferenceFriendly | wd:Worker/wd:Worker_Data/wd:Employment_Data/wd:Position_Data/wd:Business_Site_Summary_Data/wd:Address_Data/wd:Country_Reference/@wd:Descriptor |
     | CountryReferenceNumeric | wd:Worker/wd:Worker_Data/wd:Employment_Data/wd:Position_Data/wd:Business_Site_Summary_Data/wd:Address_Data/wd:Country_Reference/wd:ID[@wd:type='ISO_3166-1_Numeric-3_Code']/text() |
     | CountryReferenceTwoLetter | wd:Worker/wd:Worker_Data/wd:Employment_Data/wd:Position_Data/wd:Business_Site_Summary_Data/wd:Address_Data/wd:Country_Reference/wd:ID[@wd:type='ISO_3166-1_Alpha-2_Code']/text() |
     | CountryRegionReference | wd:Worker/wd:Worker_Data/wd:Employment_Data/wd:Position_Data/wd:Business_Site_Summary_Data/wd:Address_Data/wd:Country_Region_Reference/@wd:Descriptor |

  Vérifiez auprès de votre équipe Workday que les expressions d'API ci-dessus sont valides pour la configuration de votre locataire Workday. Si nécessaire, vous pouvez y apporter des modifications comme décrit dans la section [Personnaliser la liste des attributs d'utilisateurs Workday](#customizing-the-list-of-workday-user-attributes).

* Pour générer l’expression de mappage d’attributs appropriée, identifiez l’attribut Workday qui « fait autorité » pour représenter le prénom, le nom, le pays/la région et le service de l’utilisateur. Supposons que les attributs soient respectivement : *PreferredFirstName*, *PreferredLastName*, *CountryReferenceTwoLetter* et *SupervisoryOrganization*. Vous pouvez utiliser ce qui suit afin de générer une expression pour l'attribut AD *displayName* ; vous obtiendrez ainsi un nom d'affichage tel que *Smith, John (Marketing-US)* .

    ```
     Append(Join(", ",[PreferredLastName],[PreferredFirstName]), Join(""," (",[SupervisoryOrganization],"-",[CountryReferenceTwoLetter],")"))
    ```
    Dès que vous disposez de l'expression adéquate, modifiez la table Mappages d'attributs et changez le mappage de l'attribut *displayName* comme indiqué ci-dessous :   ![Mappage de DisplayName](./media/workday-inbound-tutorial/wd_displayname_map.png)

* Dans le prolongement de l'exemple ci-dessus, supposons que vous souhaitiez convertir les noms de ville issus de Workday en valeurs abrégées, puis les utiliser pour générer des noms d'affichage tels que *Smith, John (CHI)* ou *Doe, Jane (NYC)* . Pour ce faire, vous pouvez utiliser une expression Switch avec l'attribut Workday *Municipality* comme variable déterminante.

     ```
    Switch
    (
      [Municipality],
      Join(", ", [PreferredLastName], [PreferredFirstName]),  
           "Chicago", Append(Join(", ",[PreferredLastName], [PreferredFirstName]), "(CHI)"),
           "New York", Append(Join(", ",[PreferredLastName], [PreferredFirstName]), "(NYC)"),
           "Phoenix", Append(Join(", ",[PreferredLastName], [PreferredFirstName]), "(PHX)")
    )
     ```
    Voir aussi :
  * [Syntaxe de la fonction Switch](../app-provisioning/functions-for-customizing-application-data.md#switch)
  * [Syntaxe de la fonction Join](../app-provisioning/functions-for-customizing-application-data.md#join)
  * [Syntaxe de la fonction Append](../app-provisioning/functions-for-customizing-application-data.md#append)

#### <a name="how-can-i-use-selectuniquevalue-to-generate-unique-values-for-samaccountname-attribute"></a>Comment utiliser SelectUniqueValue afin de générer des valeurs uniques pour l'attribut samAccountName ?

Supposons que vous souhaitiez générer des valeurs uniques pour l'attribut *samAccountName* en combinant les attributs *FirstName* et *LastName* de Workday. Vous pouvez commencer avec l'expression suivante :

```
SelectUniqueValue(
    Replace(Mid(Replace(NormalizeDiacritics(StripSpaces(Join("",  Mid([FirstName],1,1), [LastName]))), , "([\\/\\\\\\[\\]\\:\\;\\|\\=\\,\\+\\*\\?\\<\\>])", , "", , ), 1, 20), , "(\\.)*$", , "", , ),
    Replace(Mid(Replace(NormalizeDiacritics(StripSpaces(Join("",  Mid([FirstName],1,2), [LastName]))), , "([\\/\\\\\\[\\]\\:\\;\\|\\=\\,\\+\\*\\?\\<\\>])", , "", , ), 1, 20), , "(\\.)*$", , "", , ),
    Replace(Mid(Replace(NormalizeDiacritics(StripSpaces(Join("",  Mid([FirstName],1,3), [LastName]))), , "([\\/\\\\\\[\\]\\:\\;\\|\\=\\,\\+\\*\\?\\<\\>])", , "", , ), 1, 20), , "(\\.)*$", , "", , )
)
```

L'expression ci-dessus fonctionne de la façon suivante : si l'utilisateur s'appelle John Smith, elle essaie d'abord de générer JSmith. Si JSmith existe déjà, elle génère JoSmith. Et si ce dernier existe aussi, elle génère JohSmith. L'expression veille également à ce que la valeur générée respecte les restrictions de longueur et de caractères spéciaux associées à *samAccountName*.

Voir aussi :

* [Syntaxe de la fonction Mid](../app-provisioning/functions-for-customizing-application-data.md#mid)
* [Syntaxe de la fonction Replace](../app-provisioning/functions-for-customizing-application-data.md#replace)
* [Syntaxe de la fonction SelectUniqueValue](../app-provisioning/functions-for-customizing-application-data.md#selectuniquevalue)

#### <a name="how-do-i-remove-characters-with-diacritics-and-convert-them-into-normal-english-alphabets"></a>Comment supprimer des caractères comportant des signes diacritiques et les convertir en lettres normales de l'alphabet anglais ?

Utilisez la fonction [NormalizeDiacritics](../app-provisioning/functions-for-customizing-application-data.md#normalizediacritics) pour supprimer les caractères spéciaux du prénom et du nom de l'utilisateur lors de la création de l'adresse e-mail ou de la valeur CN de celui-ci.

## <a name="troubleshooting-tips"></a>Conseils de dépannage

Cette section fournit des conseils spécifiques pour résoudre les problèmes d’approvisionnement liés à votre intégration Workday à l’aide des journaux d’activité d’audit Azure AD et des journaux d’activité de l’observateur d’événements Windows Server. Elle s'appuie sur les étapes génériques de résolution des problèmes ainsi que sur les concepts présentés dans le [Didacticiel : Création de rapports sur l'approvisionnement automatique de comptes d'utilisateur](../app-provisioning/check-status-user-account-provisioning.md)

Cette section couvre les aspects suivants de la résolution de problèmes :

* [Configuration de l'observateur d'événements Windows pour résoudre les problèmes liés à l'agent](#setting-up-windows-event-viewer-for-agent-troubleshooting)
* [Configuration des journaux d’activité d’audit du portail Azure pour résoudre les problèmes de service](#setting-up-azure-portal-audit-logs-for-service-troubleshooting)
* [Familiarisation avec les journaux d’activité pour les opérations de création de compte d’utilisateur AD](#understanding-logs-for-ad-user-account-create-operations)
* [Familiarisation avec les journaux d’activité pour les opérations de mise à jour de l’attribut manager](#understanding-logs-for-manager-update-operations)
* [Résolution des erreurs les plus fréquentes](#resolving-commonly-encountered-errors)

### <a name="setting-up-windows-event-viewer-for-agent-troubleshooting"></a>Configuration de l'observateur d'événements Windows pour résoudre les problèmes liés à l'agent

* Se connecter à l’ordinateur Windows Server sur lequel l’agent d’approvisionnement est déployé
* Ouvrez l'application de bureau **Observateur d'événements Windows Server**.
* Sélectionnez **Journaux d’activité Windows &gt; Application**.
* Utilisez l'option **Filtrer le journal actuel...** pour afficher tous les événements enregistrés sous la source **AAD.Connect.ProvisioningAgent** et exclure les événements dotés de l'ID d'événement « 5 », en spécifiant le filtre « -5 » comme illustré ci-dessous.

  ![Observateur d'événements Windows](media/workday-inbound-tutorial/wd_event_viewer_01.png))

* Cliquez sur **OK** et classez la vue des résultats par colonne **Date et heure**.

### <a name="setting-up-azure-portal-audit-logs-for-service-troubleshooting"></a>Configuration des journaux d’activité d’audit du portail Azure pour résoudre les problèmes de service

* Lancez le [Portail Azure](https://portal.azure.com) et accédez à la section **Journaux d’audit** de votre application d’approvisionnement Workday.
* Utilisez le bouton **Colonnes** de la page Journaux d’audit pour n’afficher que les colonnes suivantes (Date, Activité, Statut, Raison du statut). Cette configuration vous permet de vous concentrer uniquement sur les données à prendre en compte pour la résolution des problèmes.

  ![Colonnes du journal d'audit](media/workday-inbound-tutorial/wd_audit_logs_00.png)

* Utilisez les paramètres de requête **Cible** et **Plage de dates** pour filtrer la vue. 
  * Définissez le paramètre de requête **Cible** sur l'« ID collaborateur » ou l'« ID employé » de l'objet collaborateur de Workday.
  * Définissez la **Plage de dates** sur une période appropriée à la recherche d'erreurs ou de problèmes liés à l'approvisionnement.

  ![Filtres du journal d'audit](media/workday-inbound-tutorial/wd_audit_logs_01.png)

### <a name="understanding-logs-for-ad-user-account-create-operations"></a>Familiarisation avec les journaux d’activité pour les opérations de création de compte d’utilisateur AD

Lorsqu'un nouvel employé est détecté dans Workday (par exemple, avec l'ID employé *21023*), le service d'approvisionnement d'Azure AD tente de créer un nouveau compte d'utilisateur AD pour l'employé et crée 4 enregistrements de journal d'audit, comme illustré ci-dessous :

  [![Opérations de création dans le journal d’audit](media/workday-inbound-tutorial/wd_audit_logs_02.png)](media/workday-inbound-tutorial/wd_audit_logs_02.png#lightbox)

Lorsque vous cliquez sur l'un des enregistrements du journal d'audit, la page **Détails de l'activité** s'ouvre. Voici ce que la page **Détails de l'activité** affiche pour chaque type d'enregistrement du journal.

* Enregistrement **Workday Import** : cet enregistrement du journal affiche les informations relative à l'employé extraites de Workday. Utilisez les informations de la section *Détails supplémentaires* de l'enregistrement du journal pour résoudre les problèmes liés à l'extraction de données à partir de Workday. Un exemple d'enregistrement est fourni ci-dessous avec des indications permettant d'interpréter chacun des champs.

  ```JSON
  ErrorCode : None  // Use the error code captured here to troubleshoot Workday issues
  EventName : EntryImportAdd // For full sync, value is "EntryImportAdd" and for delta sync, value is "EntryImport"
  JoiningProperty : 21023 // Value of the Workday attribute that serves as the Matching ID (usually the Worker ID or Employee ID field)
  SourceAnchor : a071861412de4c2486eb10e5ae0834c3 // set to the WorkdayID (WID) associated with the record
  ```

* Enregistrement **AD Import** : cet enregistrement du journal affiche les informations relatives au compte extraites d'AD. Comme lors de la création initiale de l'utilisateur, il n'y a pas de compte AD. Le champ *Raison du statut Activité* indique qu'aucun compte doté de la valeur d'attribut ID correspondant n'a été trouvé dans Active Directory. Utilisez les informations de la section *Détails supplémentaires* de l'enregistrement du journal pour résoudre les problèmes liés à l'extraction de données à partir de Workday. Un exemple d'enregistrement est fourni ci-dessous avec des indications permettant d'interpréter chacun des champs.

  ```JSON
  ErrorCode : None // Use the error code captured here to troubleshoot Workday issues
  EventName : EntryImportObjectNotFound // Implies that object was not found in AD
  JoiningProperty : 21023 // Value of the Workday attribute that serves as the Matching ID
  ```

  Pour rechercher les enregistrements du journal de l’agent d’approvisionnement correspondant à cette opération d’importation AD, ouvrez les journaux d’activité de l’observateur d’événements Windows et utilisez l’option de menu **Rechercher...** afin de rechercher les entrées de journal contenant la valeur de l'attribut ID correspondant/Propriété de jonction (dans ce cas, *21023*).

  ![Rechercher](media/workday-inbound-tutorial/wd_event_viewer_02.png)

  Recherchez l'entrée contenant *Event ID = 9*, qui vous fournira le filtre de recherche LDAP utilisé par l'agent pour extraire le compte AD. Vous pouvez vérifier s'il s'agit du bon filtre de recherche pour extraire des entrées d'utilisateur uniques.

  ![Recherche LDAP](media/workday-inbound-tutorial/wd_event_viewer_03.png)

  L'enregistrement suivant, qui contient *Event ID = 2*, capture le résultat de l'opération de recherche et indique si elle a renvoyé des résultats.

  ![Résultats LDAP](media/workday-inbound-tutorial/wd_event_viewer_04.png)

* Enregistrement **Synchronization rule action** : cet enregistrement du journal affiche les résultats des règles de mappage des attributs et des filtres d'étendue configurés, ainsi que l'action d'approvisionnement qui sera entreprise pour traiter l'événement Workday entrant. Utilisez les informations de la section *Détails supplémentaires* de l'enregistrement du journal pour résoudre les problèmes liés à l'action de synchronisation. Un exemple d'enregistrement est fourni ci-dessous avec des indications permettant d'interpréter chacun des champs.

  ```JSON
  ErrorCode : None // Use the error code captured here to troubleshoot sync issues
  EventName : EntrySynchronizationAdd // Implies that the object will be added
  JoiningProperty : 21023 // Value of the Workday attribute that serves as the Matching ID
  SourceAnchor : a071861412de4c2486eb10e5ae0834c3 // set to the WorkdayID (WID) associated with the profile in Workday
  ```

  À ce stade, en cas de problème lié à vos expressions de mappage d'attributs ou aux données Workday entrantes (par exemple : valeur vide ou nulle pour les attributs requis), vous rencontrerez un échec accompagné du code ErrorCode détaillant celui-ci.

* Enregistrement **AD Export** : cet enregistrement du journal affiche le résultat de l'opération de création de compte AD, ainsi que les valeurs d'attribut définies au cours du processus. Utilisez les informations de la section *Détails supplémentaires* de l'enregistrement du journal pour résoudre les problèmes liés à l'opération de création de compte. Un exemple d'enregistrement est fourni ci-dessous avec des indications permettant d'interpréter chacun des champs. Dans la section « Détails supplémentaires », la propriété « EventName » est définie sur « EntryExportAdd », la propriété « JoiningProperty » est définie sur la valeur de l’attribut ID correspondant, la propriété « SourceAnchor » est définie sur l’ID Workday (WID) associé à l’enregistrement et la propriété « TargetAnchor » est définie sur le valeur de l’attribut AD « ObjectGuid » de l’utilisateur nouvellement créé. 

  ```JSON
  ErrorCode : None // Use the error code captured here to troubleshoot AD account creation issues
  EventName : EntryExportAdd // Implies that object will be created
  JoiningProperty : 21023 // Value of the Workday attribute that serves as the Matching ID
  SourceAnchor : a071861412de4c2486eb10e5ae0834c3 // set to the WorkdayID (WID) associated with the profile in Workday
  TargetAnchor : 83f0156c-3222-407e-939c-56677831d525 // set to the value of the AD "objectGuid" attribute of the new user
  ```

  Pour rechercher les enregistrements du journal de l’agent d’approvisionnement correspondant à cette opération d’exportation AD, ouvrez les journaux d’activité de l’observateur d’événements Windows et utilisez l’option de menu **Rechercher...** afin de rechercher les entrées de journal contenant la valeur de l'attribut ID correspondant/Propriété de jonction (dans ce cas, *21023*).  

  Recherchez un enregistrement HTTP POST correspondant à l'horodatage de l'opération d'exportation contenant *Event ID = 2*. Cet enregistrement contiendra les valeurs d'attribut envoyées par le service d'approvisionnement à l'agent d'approvisionnement.

  :::image type="content" source="media/workday-inbound-tutorial/wd_event_viewer_05.png" alt-text="Capture d'écran représentant l'enregistrement « HTTP POST » dans le journal « Agent d'approvisionnement »." lightbox="media/workday-inbound-tutorial/wd_event_viewer_05.png":::

  Juste après l'événement ci-dessus, un autre événement doit capturer la réponse de l'opération de création de compte AD. Cet événement renvoie le nouvel identifiant objectGuid créé dans AD et celui-ci est défini comme attribut TargetAnchor dans le service d'approvisionnement.

  :::image type="content" source="media/workday-inbound-tutorial/wd_event_viewer_06.png" alt-text="Capture d'écran représentant le journal « Agent d'approvisionnement », avec l'objectGuid créé dans Active Directory en surbrillance." lightbox="media/workday-inbound-tutorial/wd_event_viewer_06.png":::

### <a name="understanding-logs-for-manager-update-operations"></a>Familiarisation avec les journaux d’activité pour les opérations de mise à jour de l’attribut manager

L'attribut manager est un attribut de référence dans AD. Le service d'approvisionnement ne définit pas l'attribut manager dans le cadre de l'opération de création de l'utilisateur. L'attribut manager est plutôt défini dans le cadre d'une opération de *mise à jour* après la création du compte AD de l'utilisateur. Dans le prolongement de l’exemple précédent, supposons qu’un nouvel employé doté de l’ID employé « 21451 » soit activé dans Workday et que son manager (*21023*) dispose déjà d’un compte AD. Dans ce scénario, la recherche dans les journaux d’audit de l’utilisateur 21451 affiche 5 entrées.

  [![Mise à jour de l’attribut manager](media/workday-inbound-tutorial/wd_audit_logs_03.png)](media/workday-inbound-tutorial/wd_audit_logs_03.png#lightbox)

Les 4 premiers enregistrements sont semblables à ceux que nous avons explorés dans le cadre de l'opération de création de l'utilisateur. Le cinquième enregistrement correspond à l'exportation associée à la mise à jour de l'attribut manager. L’enregistrement du journal affiche le résultat de l’opération de mise à jour du manager du compte AD, effectuée à l’aide de l’attribut *objectGuid* du manager.

  ```JSON
  // Modified Properties
  Name : manager
  New Value : "83f0156c-3222-407e-939c-56677831d525" // objectGuid of the user 21023

  // Additional Details
  ErrorCode : None // Use the error code captured here to troubleshoot AD account creation issues
  EventName : EntryExportUpdate // Implies that object will be created
  JoiningProperty : 21451 // Value of the Workday attribute that serves as the Matching ID
  SourceAnchor : 9603bf594b9901693f307815bf21870a // WorkdayID of the user
  TargetAnchor : 43b668e7-1d73-401c-a00a-fed14d31a1a8 // objectGuid of the user 21451

  ```

### <a name="resolving-commonly-encountered-errors"></a>Résolution des erreurs les plus fréquentes

Cette section décrit les erreurs fréquemment rencontrées lors de l'approvisionnement d'utilisateurs Workday et explique comment y remédier. Les erreurs sont regroupées comme suit :

* [Erreurs liées à l'agent d'approvisionnement](#provisioning-agent-errors)
* [Erreurs de connectivité](#connectivity-errors)
* [Erreurs liées à la création de comptes d'utilisateur AD](#ad-user-account-creation-errors)
* [Erreurs liées à la mise à jour de comptes d'utilisateur AD](#ad-user-account-update-errors)

#### <a name="provisioning-agent-errors"></a>Erreurs liées à l'agent d'approvisionnement

|#|Scénario d'erreur |Causes probables|Résolution recommandée|
|--|---|---|---|
|1.| Erreur survenue lors de l'installation de l'agent d'approvisionnement et accompagnée du message suivant :  *Le service « Agent d'approvisionnement Microsoft Azure AD Connect » (AADConnectProvisioningAgent) n'a pas pu démarrer. Vérifiez que vous disposez des privilèges suffisants pour démarrer le système.* | Cette erreur apparaît généralement lorsque vous essayez d'installer l'agent d'approvisionnement sur un contrôleur de domaine et que la stratégie de groupe empêche le service de démarrer.  Elle apparaît également lorsqu'une version précédente de l'agent est en cours d'exécution et que vous ne l'avez pas désinstallée avant de procéder à la nouvelle installation.| Installez l'agent d'approvisionnement sur un serveur autre que celui du contrôleur de domaine. Assurez-vous que les versions précédentes de l'agent sont désinstallées avant d'installer le nouvel agent.|
|2.| Le service Windows « Agent d'approvisionnement Microsoft Azure AD Connect » est bloqué sur l'état *Démarrage* et ne bascule pas vers l'état *En cours d'exécution*. | Lors de l’installation, l’Assistant de l’agent crée un compte local (**Service NT\\AADConnectProvisioningAgent**) sur le serveur. Il s’agit du compte de connexion qui est utilisé pour démarrer le service. Vous rencontrerez cette erreur si une stratégie de sécurité de votre serveur Windows empêche les comptes locaux d'exécuter les services. | Ouvrez la *console Services*. Cliquez avec le bouton droit sur le service Windows « Agent de provisionnement Microsoft Azure AD Connect » et, sous l’onglet Connexion, spécifiez le compte d’un administrateur de domaine pour exécuter le service. Redémarrez le service. |
|3.| Lors de la configuration de l'agent d'approvisionnement avec votre domaine AD à l'étape *Connecter Active Directory*, l'Assistant passe beaucoup de temps à essayer de charger le schéma AD et finit par expirer. | Cette erreur se produit généralement lorsque l'Assistant ne parvient pas à contacter le serveur du contrôleur de domaine AD en raison de problèmes de pare-feu. | Sur l'écran de l'Assistant *Connecter Active Directory*, au moment d'entrer les informations d'identification de votre domaine AD, vous disposez d'une option appelée *Sélectionner la priorité du contrôleur de domaine*. Utilisez cette option pour sélectionner un contrôleur de domaine situé sur le même site que le serveur de l'agent et assurez-vous qu'aucune règle de pare-feu ne bloque la communication. |

#### <a name="connectivity-errors"></a>Erreurs de connectivité

Si le service d'approvisionnement ne parvient pas à se connecter à Workday ou à Active Directory, l'approvisionnement est mis en quarantaine. Utilisez le tableau ci-dessous pour résoudre les problèmes de connectivité.

|#|Scénario d'erreur |Causes probables|Résolution recommandée|
|--|---|---|---|
|1.| Lorsque vous cliquez sur **Tester la connexion**, le message d'erreur suivant s'affiche : *Une erreur s'est produite lors de la connexion à Active Directory. Vérifiez que l'agent d'approvisionnement local est en cours d'exécution et qu'il est configuré avec le domaine Active Directory approprié.* | Cette erreur apparaît généralement lorsque l'agent d'approvisionnement ne fonctionne pas ou qu'un pare-feu bloque la communication entre Azure AD et l'agent d'approvisionnement. Cette erreur peut également apparaître lorsque le domaine n'est pas configuré dans l'Assistant Agent. | Ouvrez la console *Services* sur le serveur Windows pour vérifier que l'agent est en cours d'exécution. Ouvrez l'Assistant de l'agent d'approvisionnement et vérifiez que le bon domaine est inscrit auprès de l'agent.  |
|2.| Le travail d'approvisionnement est mis en quarantaine pendant le week-end (vendredi et samedi) et un e-mail nous est adressé pour nous informer qu'une erreur de synchronisation s'est produite. | Cette erreur est souvent due au temps d'arrêt planifié pour Workday. Si vous utilisez un locataire pour l'implémentation de Workday, notez que Workday a programmé des temps d'arrêt pour ses locataires d'implémentation pendant les week-ends (généralement du vendredi soir au samedi matin). En conséquence, pendant cette période, les applications d'approvisionnement Workday peuvent être mises en quarantaine car elles ne peuvent se connecter à Workday. Tout rentre dans l'ordre lorsque le locataire d'implémentation Workday est de nouveau en ligne. Dans de rares cas, vous pouvez également rencontrer cette erreur lorsque le mot de passe de l'utilisateur du système d'intégration a changé suite à l'actualisation du locataire ou encore lorsque le compte est verrouillé ou qu'il est arrivé à expiration. | Contactez votre administrateur Workday ou votre partenaire d'intégration afin de connaître les temps d'arrêt planifiés par Workday. Vous pourrez ainsi ignorer les messages d'alerte survenant aux heures correspondantes et vérifier la disponibilité une fois l'instance de Workday de nouveau en ligne.  |

#### <a name="ad-user-account-creation-errors"></a>Erreurs liées à la création de comptes d'utilisateur AD

|#|Scénario d'erreur |Causes probables|Résolution recommandée|
|--|---|---|---|
|1.| Échec de l'opération d'exportation dans le journal d'audit avec le message *Erreur : OperationsError-SvcErr : Une erreur de fonctionnement s'est produite. Aucun renvoi supérieur n'a été configuré pour le service d'annuaire. Le service d'annuaire n'est donc pas en mesure d'émettre des renvois vers des objets situés en dehors de cette forêt.* | Cette erreur apparaît généralement lorsque l'unité d'organisation *Conteneur Active Directory* n'est pas correctement configurée ou en cas de problèmes liés au mappage d'expression utilisé pour *parentDistinguishedName*. | Vérifiez l'absence de fautes de frappe dans le paramètre de l'unité d'organisation *Conteneur Active Directory*. Si vous utilisez *parentDistinguishedName* dans le mappage d'attributs, assurez-vous qu'il correspond toujours à un conteneur connu du domaine AD. Consultez l’événement *Export* dans les journaux d’audit pour connaître la valeur générée. |
|2.| Échec de l'opération d'exportation dans le journal d'audit avec le code d'erreur : *SystemForCrossDomainIdentityManagementBadResponse* et le message *Erreur : ConstraintViolation-AtrErr : Une valeur de la demande n'est pas valide. Une valeur de l'attribut se trouvait en dehors de la plage acceptable. \Détails de l'erreur : CONSTRAINT_ATT_TYPE - company*. | Bien que cette erreur soit spécifique à l'attribut *company*, vous pouvez également la rencontrer pour d'autres attributs tels que *CN*. Cette erreur apparaît en raison de la contrainte de schéma imposée par AD. Par défaut, les attributs AD tels que *company* et *CN* sont limités à 64 caractères. Si la valeur provenant de Workday comporte plus de 64 caractères, ce message d'erreur s'affiche. | Consultez l’événement *Export* dans les journaux d’audit pour connaître la valeur de l’attribut signalé dans le message d’erreur. Vous pouvez tronquer la valeur provenant de Workday à l'aide de la fonction [Mid](../app-provisioning/functions-for-customizing-application-data.md#mid) ou remplacer les mappages par un attribut AD qui ne comporte pas de contraintes de longueur similaires.  |

#### <a name="ad-user-account-update-errors"></a>Erreurs liées à la mise à jour de comptes d'utilisateur AD

Pendant le processus de mise à jour du compte d'utilisateur AD, le service d'approvisionnement lit les informations de Workday et d'AD, exécute les règles de mappage des attributs et détermine si des modifications doivent être appliquées. En conséquence, un événement de mise à jour est déclenché. Si l’une de ces étapes se solde par un échec, celui-ci est consigné dans les journaux d’audit. Utilisez le tableau ci-dessous pour résoudre les erreurs de mise à jour courantes.

|#|Scénario d'erreur |Causes probables|Résolution recommandée|
|--|---|---|---|
|1.| Échec de l'action de la règle de synchronisation dans le journal d'audit avec le message *EventName = EntrySynchronizationError and ErrorCode = EndpointUnavailable*. | Cette erreur apparaît lorsque le service d'approvisionnement n'est pas en mesure de récupérer les données de profil utilisateur dans Active Directory en raison d'une erreur de traitement rencontrée par l'agent d'approvisionnement local. | Dans les journaux d’activité de l’observateur d’événements de l’agent d’approvisionnement, recherchez les événements d’erreur révélant des problèmes liés à l’opération de lecture (appliquez le filtre suivant : Event ID #2). |
|2.| L'attribut manager d'AD n'est pas mis à jour pour certains utilisateurs d'AD. | La cause la plus probable de cette erreur est que vous utilisez des règles d'étendue et que le manager de l'utilisateur ne fait pas partie de l'étendue. Vous pouvez également rencontrer ce problème si l'attribut ID correspondant du manager (par exemple, EmployeeID) est introuvable dans le domaine AD cible ou s'il n'est pas défini sur la valeur qui convient. | Vérifiez le filtre d'étendue et ajoutez l'utilisateur manager dans l'étendue. Vérifiez le profil du manager dans AD pour vous assurer qu'il existe une valeur pour l'attribut ID correspondant. |

## <a name="managing-your-configuration"></a>Gestion de votre configuration

Cette section explique comment approfondir l'extension, la personnalisation et la gestion de votre configuration d'approvisionnement d'utilisateurs Workday. Elle couvre les rubriques suivantes :

* [Personnaliser la liste des attributs d’utilisateurs Workday](#customizing-the-list-of-workday-user-attributes)  
* [Exporter et importer votre configuration](#exporting-and-importing-your-configuration)

### <a name="customizing-the-list-of-workday-user-attributes"></a>Personnaliser la liste des attributs d’utilisateurs Workday

Les applications d’attribution Workday pour Active Directory et Azure AD contiennent tous les deux une liste par défaut des attributs d’utilisateurs Workday utilisables. Toutefois, ces listes ne sont pas complètes. Workday prend en charge plusieurs centaines d’attributs d’utilisateurs possibles, qui peuvent être standard ou propres au locataire Workday concerné.

Le service d’attribution Azure AD prend en charge la possibilité de personnaliser la liste ou l’attribut Workday de façon à inclure tous les attributs exposés dans l’opération [Get_Workers](https://community.workday.com/sites/default/files/file-hosting/productionapi/Human_Resources/v21.1/Get_Workers.html) de l’API Ressources humaines.

Il vous faut pour cela utiliser [Workday Studio](https://community.workday.com/studio-download) afin d'extraire les expressions XPath qui représentent les attributs à utiliser, puis les ajouter à votre configuration d'approvisionnement à l'aide de l'éditeur d'attributs avancé du portail Azure.

**Pour récupérer une expression XPath pour un attribut d’utilisateur Workday :**

1. Téléchargez et installez [Workday Studio](https://community.workday.com/studio-download). Vous aurez besoin d’un compte Communauté Workday pour accéder au programme d’installation.

2. Téléchargez le fichier WSDL Workday **Human_Resources** spécifique à la version de l’API WWS que vous envisagez d’utiliser dans [l’annuaire Workday Web Services](https://community.workday.com/sites/default/files/file-hosting/productionapi/index.html)

3. Lancez Workday Studio.

4. Dans la barre de commandes, sélectionnez l’option **Workday > Tester le service web dans le testeur**.

5. Sélectionnez **Externe**, puis le fichier WSDL Human_Resources que vous avez téléchargé à l’étape 2.

    ![Capture d'écran représentant le fichier « Human_Resources » ouvert dans Workday Studio.](./media/workday-inbound-tutorial/wdstudio1.png)

6. Définissez le champ **Emplacement** sur `https://IMPL-CC.workday.com/ccx/service/TENANT/Human_Resources`, mais en remplaçant « IMPL-CC » par le type réel de votre instance, et « TENANT » par le nom réel de votre locataire.

7. Définissez **Opération** sur **Get_Workers**.

8.    Cliquez sur le petit lien **Configurer** sous les volets Requête/Réponse pour définir vos informations d’identification Workday. Cochez **Authentification**, puis entrez le nom d’utilisateur et le mot de passe de votre compte système d’intégration Workday. Veillez à mettre le nom d’utilisateur au format nom\@locataire et à laisser l’option **WS-Security UsernameToken** sélectionnée.
   ![Capture d'écran représentant l'onglet « Sécurité », où le « Nom d'utilisateur » et le « Mot de passe » sont entrés, et où l'option « WS-Security UsernameToken » est sélectionnée.](./media/workday-inbound-tutorial/wdstudio2.png)

9. Sélectionnez **OK**.

10. Dans le volet de **Requête**, collez le code XML ci-dessous. Définissez **Employee_ID** sur l’ID d’employé d’un utilisateur réel de votre locataire Workday. Définissez **wd:version** sur la version de WWS que vous prévoyez d’utiliser. Sélectionnez un utilisateur dont l’attribut à extraire est rempli.

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <env:Envelope xmlns:env="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsd="https://www.w3.org/2001/XMLSchema">
      <env:Body>
        <wd:Get_Workers_Request xmlns:wd="urn:com.workday/bsvc" wd:version="v21.1">
          <wd:Request_References wd:Skip_Non_Existing_Instances="true">
            <wd:Worker_Reference>
              <wd:ID wd:type="Employee_ID">21008</wd:ID>
            </wd:Worker_Reference>
          </wd:Request_References>
          <wd:Response_Group>
            <wd:Include_Reference>true</wd:Include_Reference>
            <wd:Include_Personal_Information>true</wd:Include_Personal_Information>
            <wd:Include_Employment_Information>true</wd:Include_Employment_Information>
            <wd:Include_Management_Chain_Data>true</wd:Include_Management_Chain_Data>
            <wd:Include_Organizations>true</wd:Include_Organizations>
            <wd:Include_Reference>true</wd:Include_Reference>
            <wd:Include_Transaction_Log_Data>true</wd:Include_Transaction_Log_Data>
            <wd:Include_Photo>true</wd:Include_Photo>
            <wd:Include_User_Account>true</wd:Include_User_Account>
          <wd:Include_Roles>true</wd:Include_Roles>
          </wd:Response_Group>
        </wd:Get_Workers_Request>
      </env:Body>
    </env:Envelope>
    ```

11. Cliquez sur la flèche verte **Envoyer la requête** pour exécuter la commande. En cas de réussite, la réponse apparaîtra dans le volet **Réponse**. Vérifiez que la réponse comporte les données de l’ID d’utilisateur que vous avez entré et non une erreur.

12. En cas de réussite, copiez le code XML du volet **Réponse** et enregistrez-le dans un fichier XML.

13. Dans la barre de commandes de Workday Studio, sélectionnez **Fichier > Ouvrir un fichier...**  et ouvrez le fichier XML que vous avez enregistré. Cette action ouvrira le fichier dans l'éditeur XML de Workday Studio.

    ![Capture d'écran d'un fichier XML ouvert dans l'éditeur XML de Workday Studio.](./media/workday-inbound-tutorial/wdstudio3.png)

14. Dans l’arborescence des fichiers, accédez à **/env: Envelope > env: Body > wd:Get_Workers_Response > wd:Response_Data > wd: Worker** pour trouver les données de votre utilisateur.

15. Sous **wd: Worker**, recherchez l’attribut à ajouter, puis sélectionnez-le.

16. Copiez l’expression XPath de l’attribut sélectionné hors du champ **Chemin d’accès du document**.

17. Supprimez le préfixe **/env:Envelope/env:Body/wd:Get_Workers_Response/wd:Response_Data/** dans l’expression copiée.

18. Si le dernier élément dans l’expression copiée est un nœud (exemple : « /wd: Birth_Date »), ajoutez **/text()** à la fin de l’expression. Ce n’est pas nécessaire si le dernier élément est un attribut (exemple : « type /@wd: »).

19. Le résultat doit se présenter ainsi : `wd:Worker/wd:Worker_Data/wd:Personal_Data/wd:Birth_Date/text()`. Cette valeur est celle que vous allez copier sur le portail Azure.

**Pour ajouter votre attribut d’utilisateur Workday personnalisé à votre configuration d’attribution :**

1. Lancez le [Portail Azure](https://portal.azure.com), accédez à la section Attribution de votre application d’attribution Workday, selon la procédure décrite plus haut dans ce didacticiel.

2. Définissez **État d’attribution** sur **Désactivé**, puis sélectionnez **Enregistrer**. Cette étape vous aidera à faire en sorte que vos modifications ne prennent effet que lorsque vous le déciderez.

3. Sous **Mappages**, sélectionnez **Synchroniser les employés Workday avec l'instance locale d'Active Directory** (ou **Synchroniser les employés Workday avec Azure AD**).

4. Faites défiler l’écran suivant jusqu’en bas, puis sélectionnez **Afficher les options avancées**.

5. Sélectionnez **Modifier la liste d’attributs de Workday**.

    ![Capture d'écran représentant la page « Workday vers Azure AD - Attribution d'utilisateurs - Approvisionnement », sur laquelle l'action « Modifier la liste d'attributs de Workday » est en surbrillance.](./media/workday-inbound-tutorial/wdstudio_aad1.png)

6. Faites défiler la liste d’attributs jusqu’en bas, où se trouvent les champs d’entrée.

7. Pour **Nom**, entrez le nom d’affichage de votre attribut.

8. Pour **Type**, sélectionnez le type correspondant à votre attribut (le plus courant est **Chaîne**).

9. Pour **Expression API**, entrez l’expression XPath que vous avez copiée dans Workday Studio. Exemple : `wd:Worker/wd:Worker_Data/wd:Personal_Data/wd:Birth_Date/text()`

10. Sélectionnez **Ajouter un attribut**.

    ![Workday Studio](./media/workday-inbound-tutorial/wdstudio_aad2.png)

11. Sélectionnez **Enregistrer** ci-dessus, puis **Oui** dans la boîte de dialogue. Fermez l’écran Mappage des attributs s’il est toujours ouvert.

12. Dans l'onglet **Approvisionnement**, sélectionnez de nouveau **Synchroniser les employés Workday avec l'instance locale d'Active Directory** (ou **Synchroniser les employés avec Azure AD**).

13. Sélectionnez **Ajouter un mappage**.

14. Votre nouvel attribut devrait maintenant apparaître dans la liste **Attribut source**.

15. Ajoutez le mappage de votre choix pour votre nouvel attribut.

16. Ensuite, pensez à redéfinir **État d’attribution** sur **Activé** et à enregistrer.

### <a name="exporting-and-importing-your-configuration"></a>Exporter et importer votre configuration

Consultez l’article [Exportation et importation d’une configuration de provisionnement](../app-provisioning/export-import-provisioning-configuration.md)

## <a name="managing-personal-data"></a>Gestion des données personnelles

La solution d’approvisionnement Workday pour Active Directory nécessite l’installation d’un agent d’approvisionnement sur un serveur Windows local. Cet agent crée dans le journal des événements Windows des journaux d’activité qui peuvent contenir des données personnelles, en fonction de vos mappages d’attributs Workday avec AD. Pour respecter les obligations de confidentialité de l’utilisateur, vous pouvez veiller à ce qu’aucune donnée ne soit conservée plus de 48 heures dans les journaux des événements en configurant une tâche planifiée Windows pour effacer le journal des événements.

Dans la classification du RGPD, le service d'approvisionnement Azure AD appartient à la catégorie **Processeur de données**. En tant que pipeline de processeur de données, le service fournit un traitement des données pour les partenaires et les clients. Le service d'approvisionnement Azure AD ne génère pas de données utilisateur et n'a aucun contrôle indépendant sur les données personnelles qui sont collectées, ni sur la façon dont elles sont utilisées. Dans le service d'approvisionnement Azure AD, l'extraction, l'agrégation et l'analyse de données, ainsi que la création de rapports contenant ces données, sont basées sur des données d'entreprise existantes.

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-hybrid-note.md)]

En ce qui concerne la conservation des données, le service d'approvisionnement Azure AD ne génère pas de rapports, n'effectue pas d'analyses et ne fournit pas d'insights au-delà de 30 jours. Par conséquent, le service d'approvisionnement Azure AD ne stocke pas, ne traite pas et ne conserve pas de données au-delà de 30 jours. Cette conception est conforme aux réglementations RGPD, aux réglementations de Microsoft concernant la conformité, ainsi qu’aux stratégies de conservation des données Azure AD.

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)
* [Découvrir comment configurer l’authentification unique entre Workday et Azure Active Directory](workday-tutorial.md)
* [Découvrir comment intégrer d’autres applications SaaS à Azure Active Directory](tutorial-list.md)
* [Découvrir comment utiliser les API Microsoft Graph pour gérer les configurations d'approvisionnement](/graph/api/resources/synchronization-overview)
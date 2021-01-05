---
title: 'Tutoriel : Configurer SolarWinds Service Desk (anciennement Samanage) pour l’approvisionnement automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment provisionner et déprovisionner automatiquement des comptes d’utilisateur entre Azure AD et SolarWinds Service Desk (anciennement Samanage).
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/13/2020
ms.author: Zhchia
ms.openlocfilehash: cca46d47003a1611c861986f8df839de57500db6
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96181478"
---
# <a name="tutorial-configure-solarwinds-service-desk-previously-samanage-for-automatic-user-provisioning"></a>Tutoriel : Configurer SolarWinds Service Desk (précédemment Samanage) pour l’approvisionnement automatique d’utilisateurs

Ce tutoriel décrit les étapes à effectuer dans SolarWinds Service Desk (anciennement Samanage) et Azure Active Directory (Azure AD) pour configurer le provisionnement automatique d’utilisateurs. Une fois configuré, Azure AD approvisionne et désapprovisionne automatiquement les utilisateurs et les groupes pour [SolarWinds Service Desk](https://www.samanage.com/pricing/) à l’aide du service d’approvisionnement d’Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).

## <a name="migrate-to-the-new-solarwinds-service-desk-application"></a>Migrer vers la nouvelle application SolarWinds Service Desk

Si vous disposez d’une intégration existante avec SolarWinds Service Desk, consultez la section suivante sur les modifications à venir. Si vous configurez SolarWinds Service Desk pour la première fois, vous pouvez ignorer cette section et passer à la section **Fonctionnalités prises en charge**.

#### <a name="whats-changing"></a>Ce qui a changé

* Changements du côté Azure AD : La méthode d’autorisation pour provisionner des utilisateurs dans Samanage a été jusqu’à maintenant une **authentification de base**. Bientôt, la méthode d’autorisation utilisée sera remplacée par le **jeton secret à long terme**.


#### <a name="what-do-i-need-to-do-to-migrate-my-existing-custom-integration-to-the-new-application"></a>Que dois-je faire pour migrer mon intégration personnalisée existante vers la nouvelle application ?

Si vous disposez d’une intégration SolarWinds Service Desk existante avec des informations d’identification d’administrateur valides, **vous n’avez rien à faire**. Nous effectuerons automatiquement la migration des clients vers la nouvelle application. Ce processus aura lieu en arrière-plan. Si les informations d’identification existantes expirent, ou si vous avez besoin d’autoriser l’accès à l’application, vous devez générer un jeton secret à long terme. Pour générer un nouveau jeton, reportez-vous à l’étape 2 de cet article.


#### <a name="how-can-i-tell-if-my-application-has-been-migrated"></a>Comment savoir si mon application a été migrée ? 

Une fois votre application migrée, dans la section **Informations d’identification de l’administrateur**, les champs **Nom d’utilisateur de l’administrateur** et **Mot de passe de l’administrateur** sont remplacés par un seul champ intitulé **Jeton secret**.

## <a name="capabilities-supported"></a>Fonctionnalités prises en charge

> [!div class="checklist"]
> * Créer des utilisateurs dans SolarWinds Service Desk
> * Supprimer les utilisateurs dans SolarWinds Service Desk lorsqu’ils ne nécessitent plus d’accès
> * Conserver les attributs utilisateur synchronisés entre Azure AD et SolarWinds Service Desk
> * Approvisionner des groupes et des appartenances aux groupes dans SolarWinds Service Desk
> * [Authentification unique](./samanage-tutorial.md) à SolarWinds Service Desk (recommandé)

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* [Un locataire Azure AD](../develop/quickstart-create-new-tenant.md) 
* Un compte d’utilisateur dans Azure AD avec l’[autorisation](../roles/permissions-reference.md) de configurer l’approvisionnement (par exemple, administrateur d’application, administrateur d’application Cloud, propriétaire d’application ou administrateur général). 
* Un client [SolarWinds Service Desk](https://www.samanage.com/pricing/) avec le package Professionnel.
* Un compte d’utilisateur dans SolarWinds Service Desk avec des autorisations d’administrateur.

## <a name="step-1-plan-your-provisioning-deployment"></a>Étape 1. Planifier votre déploiement de l’approvisionnement
1. En savoir plus sur le [fonctionnement du service d’approvisionnement](../app-provisioning/user-provisioning.md).
2. Déterminez qui sera dans l’[étendue pour l’approvisionnement](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
3. Déterminez les données à [mapper entre Azure AD et SolarWinds Service Desk](../app-provisioning/customize-application-attributes.md). 

## <a name="step-2-configure-solarwinds-service-desk-to-support-provisioning-with-azure-ad"></a>Étape 2. Configurer SolarWinds Service Desk pour prendre en charge l’approvisionnement avec Azure AD

Pour générer un jeton secret pour l’authentification, consultez [Tutoriel : Authentification à l’aide de jetons pour l’intégration d’API](https://help.samanage.com/s/article/Tutorial-Tokens-Authentication-for-API-Integration-1536721557657).

## <a name="step-3-add-solarwinds-service-desk-from-the-azure-ad-application-gallery"></a>Étape 3. Ajouter SolarWinds Service Desk à partir de la galerie d’applications Azure AD

Ajoutez SolarWinds Service Desk à partir de la galerie d’applications Azure AD pour commencer à gérer l’approvisionnement sur SolarWinds Service Desk. Si vous avez déjà configuré SolarWinds Service Desk pour l’authentification unique, vous pouvez utiliser la même application. Toutefois, il est recommandé de créer une application distincte lors du test initial de l’intégration. En savoir plus sur l’ajout d’une application à partir de la galerie [ici](../manage-apps/add-application-portal.md). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Étape 4. Définir qui sera dans l’étendue pour l’approvisionnement 

Le service d’approvisionnement Azure AD vous permet de définir l’étendue des utilisateurs approvisionnés en fonction de l’affectation à l’application et/ou en fonction des attributs de l’utilisateur/groupe. Si vous choisissez de définir l’étendue de l’approvisionnement pour votre application en fonction de l’attribution, vous pouvez utiliser les étapes de [suivantes](../manage-apps/assign-user-or-group-access-portal.md) pour affecter des utilisateurs et des groupes à l’application. Si vous choisissez de définir l’étendue de l’approvisionnement en fonction uniquement des attributs de l’utilisateur ou du groupe, vous pouvez utiliser un filtre d’étendue comme décrit [ici](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

* Lorsque vous affectez des utilisateurs et des groupes à SolarWinds Service Desk, vous devez sélectionner un autre rôle qu’**Accès par défaut**. Les utilisateurs disposant du rôle Accès par défaut sont exclus de l’approvisionnement et sont marqués comme non autorisés dans les journaux de configuration. Si le seul rôle disponible dans l’application est le rôle d’accès par défaut, vous pouvez [mettre à jour le manifeste de l’application](../develop/howto-add-app-roles-in-azure-ad-apps.md) pour ajouter des rôles supplémentaires. 

* Commencez progressivement. Testez avec un petit ensemble d’utilisateurs et de groupes avant d’effectuer un déploiement général. Lorsque l’étendue de l’approvisionnement est définie sur les utilisateurs et les groupes attribués, vous pouvez contrôler cela en affectant un ou deux utilisateurs ou groupes à l’application. Lorsque l’étendue est définie sur tous les utilisateurs et groupes, vous pouvez spécifier un [filtre d’étendue basé sur l’attribut](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 


## <a name="step-5-configure-automatic-user-provisioning-to-solarwinds-service-desk"></a>Étape 5. Configurer l’approvisionnement automatique d’utilisateurs sur SolarWinds Service Desk 

Cette section vous guide tout au long des étapes de configuration du service d’approvisionnement d’Azure AD pour créer, mettre à jour et désactiver des utilisateurs et/ou des groupes dans TestApp en fonction des assignations d’utilisateurs et/ou de groupes dans Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-solarwinds-service-desk-in-azure-ad"></a>Pour configurer l’approvisionnement automatique d’utilisateurs sur SolarWinds Service Desk dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **SolarWinds Service Desk**.

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Capture d’écran montrant l’onglet Approvisionnement sélectionné.](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran montrant le Mode d’approvisionnement défini sur Automatique.](common/provisioning-automatic.png)

5. Dans la section **Informations d’identification de l’administrateur**, entrez `https://api.samanage.com` dans **URL de locataire**.  Entrez la valeur du jeton secret récupérée précédemment dans **Jeton secret**. Sélectionnez **Tester la connexion** pour vérifier qu'Azure AD peut se connecter à SolarWinds Service Desk. Si la connexion échoue, vérifiez que votre compte SolarWinds Service Desk dispose d’autorisations d’administrateur et réessayez.

    ![Capture d’écran montrant le bouton Tester la connexion sélectionné.](./media/samanage-provisioning-tutorial/provisioning.png)

6. Dans le champ **E-mail de notification**, entrez l’adresse e-mail de la personne ou du groupe qui doit recevoir les notifications d’erreur de provisionnement et sélectionnez la case à cocher **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

7. Sélectionnez **Enregistrer**.

8. Dans la section **Mappages**, sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec SolarWinds Service Desk**.

9. Passez en revue les attributs utilisateur qui sont synchronisés d’Azure AD vers SolarWinds Service Desk dans la section **Mappages des attributs**. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes d’utilisateur dans SolarWinds Service Desk pour les opérations de mise à jour. Si vous choisissez de modifier l’[attribut cible correspondant](../app-provisioning/customize-application-attributes.md), vous devez vous assurer que l’API SolarWinds Service Desk prend en charge le filtrage des utilisateurs en fonction de cet attribut. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

      ![Mappages d’utilisateurs Samanage](./media/samanage-provisioning-tutorial/user-attributes.png)

10. Dans la section **Mappages**, sélectionnez **Synchroniser les groupes Azure Active Directory avec SolarWinds Service Desk**.

11. Passez en revue les attributs de groupe qui sont synchronisés d’Azure AD vers SolarWinds Service Desk dans la section **Mappages des attributs**. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les groupes dans SolarWinds Service Desk pour les opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

      ![Mappages de groupes Samanage](./media/samanage-provisioning-tutorial/group-attributes.png)

12. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Pour activer le service d’approvisionnement Azure AD pour SolarWinds Service Desk, modifiez le paramètre **État d’approvisionnement** sur **Activé** dans la section **Paramètres**.

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

14. Définissez les utilisateurs et/ou groupes que vous aimeriez approvisionner sur SolarWinds Service Desk en choisissant les valeurs souhaitées dans **Étendue** dans la section **Paramètres**.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

15. Quand vous êtes prêt à effectuer l’approvisionnement, sélectionnez **Enregistrer**.

    ![Enregistrement de la configuration de l’approvisionnement](common/provisioning-configuration-save.png)

Cette opération démarre le cycle de synchronisation initiale de tous les utilisateurs et groupes définis dans **Étendue** dans la section **Paramètres**. Le cycle de synchronisation initiale prend plus de temps que les cycles de synchronisation suivants, qui se produisent toutes les 40 minutes environ tant que le service de provisionnement Azure AD est en cours d’exécution. 

## <a name="step-6-monitor-your-deployment"></a>Étape 6. Surveiller votre déploiement
Une fois que vous avez configuré l’approvisionnement, utilisez les ressources suivantes pour surveiller votre déploiement :

1. Utilisez les [journaux d’approvisionnement](../reports-monitoring/concept-provisioning-logs.md) pour déterminer quels utilisateurs ont été configurés avec succès ou échoué.
2. Consultez la [barre de progression](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md) pour afficher l’état du cycle d’approvisionnement et quand il se termine
3. Si la configuration de l’approvisionnement semble se trouver dans un état non sain, l’application passe en quarantaine. Pour en savoir plus sur les états de quarantaine, cliquez [ici](../app-provisioning/application-provisioning-quarantine-status.md).

## <a name="connector-limitations"></a>Limitations du connecteur

Si vous sélectionnez l’option **Synchroniser l’ensemble des utilisateurs et groupes** option et configurez une valeur pour l’attribut **roles** de SolarWinds Service Desk, la valeur dans la zone **Valeur par défaut si Null (facultatif)** doit être exprimée dans le format suivant :

- {"displayName":"role"}, où rôle est la valeur par défaut souhaitée.

## <a name="change-log"></a>Journal des modifications

* 14/09/2020 : modification du nom de la société dans deux didacticiels SaaS de Samanage à SolarWinds Service Desk (anciennement Samanage) par https://github.com/ravitmorales.
* 22/04/2020 - Mise à jour de la méthode d’autorisation : remplacement de l’authentification de base par le jeton secret à long terme.

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)
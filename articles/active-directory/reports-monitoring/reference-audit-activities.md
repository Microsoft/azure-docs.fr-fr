---
title: Référence sur l’activité d’audit Azure Active Directory (Azure AD) | Microsoft Docs
description: Obtenez une vue d’ensemble des activités d’audit pouvant être enregistrées dans vos journaux d’audit dans Azure Active Directory (Azure AD).
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: a1f93126-77d1-4345-ab7d-561066041161
ms.service: active-directory
ms.devlang: na
ms.topic: reference
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 01/24/2019
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: 56f9976faf2a803e747bce8b851a3555e6d746b0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89231144"
---
# <a name="azure-ad-audit-activity-reference"></a>Référence sur l’activité d’audit Azure AD

Avec les rapports Azure Active Directory (Azure AD), vous pouvez obtenir toutes les informations dont vous avez besoin pour déterminer l’état de votre environnement.

L’architecture de création de rapports dans Azure AD comprend les composants suivants :

- **Rapports d’activité** 
    - [Connexions](concept-sign-ins.md) – Fournit des informations sur l’utilisation des applications managées et les activités de connexion des utilisateurs
    - [Journaux d’audit](concept-audit-logs.md) : traçabilité proposée via des journaux d’activité pour toutes les modifications effectuées par diverses fonctionnalités au sein d’Azure AD. 
    
- **Rapports de sécurité** 
    - [Connexions risquées](../identity-protection/overview-identity-protection.md) : une connexion risquée est une tentative de connexion susceptible de provenir d’un utilisateur autre que le propriétaire légitime d’un compte d’utilisateur. 
    - [Utilisateurs avec indicateur de risque](../identity-protection/overview-identity-protection.md) : il s’agit d’un compte d’utilisateur susceptible d’être compromis. 

Cet article répertorie les activités d’audit qui peuvent être enregistrées dans vos journaux d’audit.

## <a name="access-reviews"></a>Révisions d’accès

|Catégorie d’audit|Activité|
|---|---|
|Révisions d’accès|Révision d’accès terminée|
|Révisions d’accès|Ajouter un approbateur pour approuver une demande|
|Révisions d’accès|Ajouter un réviseur pour la révision d’accès|
|Révisions d’accès|Appliquer une révision d’accès|
|Révisions d’accès|Créer une révision d’accès|
|Révisions d’accès|Créer le programme|
|Révisions d’accès|Créer une approbation de demande|
|Révisions d’accès|Supprimer une révision d’accès|
|Révisions d’accès|Supprimer le programme|
|Révisions d’accès|Lier le contrôle du programme|
|Révisions d’accès|Intégrer les révisions d’accès Azure AD|
|Révisions d’accès|Supprimer un réviseur pour la révision d’accès|
|Révisions d’accès|Demander l’arrêt de la révision|
|Révisions d’accès|Demander l’application du résultat de révision|
|Révisions d’accès|Examiner l’appartenance au rôle Rbac|
|Révisions d’accès|Examiner l’affectation d’application|
|Révisions d’accès|Examiner l’appartenance au groupe|
|Révisions d’accès|Examiner la demande d’approbation de demande|
|Révisions d’accès|Supprimer le lien du contrôle du programme|
|Révisions d’accès|Mettre à jour une révision d’accès|
|Révisions d’accès|Mettre à jour l'état d'intégration des révisions d'accès Azure AD|
|Révisions d’accès|Mettre à jour les paramètres de notification par e-mail pour la révision d’accès|
|Révisions d’accès|Mettre à jour le paramètre de nombre de la périodicité de révision de l’accès|
|Révisions d’accès|Mettre à jour le paramètre de durée de la périodicité de révision de l’accès en jours|
|Révisions d’accès|Mettre à jour le paramètre de type de fin de la périodicité de révision de l’accès|
|Révisions d’accès|Mettre à jour le paramètre de type de la périodicité de révision de l’accès|
|Révisions d’accès|Mettre à jour les paramètres de rappel pour la révision d’accès|
|Révisions d’accès|Mettre à jour le programme|
|Révisions d’accès|Mettre à jour une approbation de demande|
|Révisions d’accès|Utilisateur désactivé|

## <a name="account-provisioning"></a>Approvisionnement des comptes

|Catégorie d’audit|Activité|
|---|---|
|Gestion des applications|Récupérer des autorisations de l’application V2|
|Gestion des applications|Récupérer des principaux de service d’application V2 dans le locataire actuel|
|Gestion des applications|Mettre à jour une application V1|
|Gestion des applications|Mettre à jour une application V2|
|Gestion des applications|Mettre à jour une autorisation de l’application V2|
|Gestion des applications|Ajouter OAuth2PermissionGrant|
|Gestion des applications|Ajouter une attribution de rôle d’application à un principal de service|

## <a name="application-proxy"></a>Proxy d’application

|Catégorie d’audit|Activité|
|---|---|
|Gestion des applications|Ajouter l’application|
|Gestion des applications|Ajouter un propriétaire à une application|
|Gestion des applications|Ajouter un propriétaire à un principal de service|
|Gestion des applications|Ajouter une stratégie à un principal de service|
|Gestion de répertoires|Ajouter un principal du service|
|Gestion de répertoires|Ajouter les informations d'identification du principal du service|
|Gestion de répertoires|Consentement pour une application|
|Gestion de répertoires|Supprimer l’application|
|Gestion de répertoires|Supprimer définitivement une application|
|Gestion de répertoires|Supprimer OAuth2PermissionGrant|
|Gestion de répertoires|Supprimer une attribution de rôle d’application d’un principal de service|
|Gestion de répertoires|Supprimer un propriétaire d’une application|
|Ressource|Supprimer un propriétaire d’un principal de service|
|Ressource|Supprimer une stratégie d’un principal de service|
|Ressource|Supprimer le principal du service|


## <a name="automated-password-rollover"></a>Substitution de mot de passe automatique

|Catégorie d’audit|Activité|
|---|---|
|Gestion des applications|Supprimer les informations d'identification du principal du service|


## <a name="b2c"></a>B2C

|Catégorie d’audit|Activité|
|---|---|
|Gestion des applications|Restaurer une application|
|Gestion des applications|Révoquer un consentement|
|Gestion des applications|Mettre à jour l’application|
|Gestion des applications|Mise à jour de clés secrètes externes.|
|Gestion des applications|Mettre à jour un principal de service|
|Gestion des applications|Émettre un jeton d’accès à l’application|
|Gestion des applications|Émettre un code d’autorisation pour l’application|
|Gestion des applications|Émettre un id_token pour l’application|
|Gestion des applications|Valider les informations d’identification de compte local|
|Gestion des applications|Valider l’authentification de l’utilisateur|
|Gestion des applications|Ajouter des autorisations de l’application V2|
|Gestion des applications|Ajouter une clé basée sur un secret ASCII à un conteneur de clés CPIM|
|Gestion des applications|Ajouter une clé à un conteneur de clés CPIM|
|Gestion des applications|AdminPolicyDatas-SetResources|
|Gestion des applications|AdminUserJourneys-GetResources|
|Gestion des applications|AdminUserJourneys-RemoveResources|
|Authentification|AdminUserJourneys-SetResources|
|Authentification|Créer un IdentityProvider|
|Authentification|Créer une application V1|
|Authentification|Créer une application V2|
|Authentification|Créer un domaine personnalisé dans le locataire|
|Autorisation|Créer un AdminUserJourney|
|Autorisation|Créer une ressource json localisée|
|Autorisation|Créer un IDP personnalisé|
|Autorisation|Créer un IDP|
|Autorisation|Créer ou mettre à jour une ressource de répertoire B2C|
|Autorisation|Créer une stratégie|
|Autorisation|Créer une stratégie trustFramework|
|Autorisation|Créer une stratégie trustFramework avec un préfixe configurable|
|Autorisation|Créer un attribut utilisateur|
|Autorisation|CreateTrustFrameworkPolicy|
|Autorisation|Crée ou met à jour un AdminUserJourney|
|Autorisation|Supprimer un IDP|
|Autorisation|Supprimer un IdentityProvider|
|Autorisation|Supprimer une application V1|
|Autorisation|Supprimer une application V2|
|Autorisation|Supprimer une autorisation de l’application V2|
|Autorisation|Supprimer une ressource de répertoire B2C|
|Autorisation|Supprimer un conteneur de clés CPIM|
|Autorisation|Supprimer une stratégie trustFramework|
|Autorisation|Supprimer un attribut utilisateur|
|Autorisation|Activer la fonctionnalité B2C|
|Autorisation|Obtenir des ressources de répertoire B2C dans un abonnement|
|Autorisation|Obtenir un IDP personnalisé|
|Autorisation|Obtenir un IDP|
|Autorisation|Obtenir des applications V1 et V2|
|Autorisation|Obtenir une application V1|
|Autorisation|Obtenir des applications V1|
|Autorisation|Obtenir une application V2|
|Autorisation|Obtenir des applications V2|
|Autorisation|Obtenir une ressource de répertoire B2C|
|Autorisation|Obtenir une liste de domaines personnalisés dans le locataire|
|Autorisation|Obtenir un parcours utilisateur|
|Autorisation|Obtenir des réclamations de l’application autorisées pour le parcours utilisateur|
|Autorisation|Obtenir des réclamations déclarées automatiquement autorisées pour le parcours utilisateur|
|Autorisation|Obtenir des réclamations déclarées automatiquement autorisées pour la stratégie|
|Autorisation|Obtenir la liste des revendications de sortie disponibles|
|Autorisation|Obtenir les définitions de contenu pour le parcours utilisateur|
|Autorisation|Obtenir des IDP pour un flux administrateur spécifique|
|Autorisation|Obtenir les métadonnées de clé active du conteneur de clés au format JWK|
|Autorisation|Obtenir une liste de tous les flux d’administration|
|Autorisation|Obtenir une liste des balises de tous les flux d’administration pour l’ensemble des utilisateurs|
|Autorisation|Obtenir une liste des locataires pour un utilisateur|
|Autorisation|Obtenir des revendications déclarées automatiquement des comptes locaux|
|Autorisation|Obtenir une ressource json localisée|
|Autorisation|Obtenir des opérations du fournisseur de ressources Microsoft.AzureActiveDirectory|
|Autorisation|Obtenir des stratégies|
|Autorisation|Obtenir une stratégie|
|Autorisation|Obtenir des propriétés de ressource d’un locataire|
|Autorisation|Obtenir une liste d’IDP pris en charge|
|Autorisation|Obtenir une liste d’IDP pris en charge du parcours utilisateur|
|Autorisation|Obtenir des informations de locataire|
|Autorisation|Obtenir des fonctionnalités autorisées de locataire|
|Autorisation|Obtenir une liste d’IDP personnalisés définie par le locataire|
|Autorisation|Obtenir une liste d’IDP définie par le locataire|
|Autorisation|Obtenir une liste d’IDP locaux définie par le locataire|
|Autorisation|Obtenir des informations sur un locataire pour un utilisateur afin de créer des ressources|
|Autorisation|Obtenir une liste de locataires|
|Autorisation|Obtenir tenantDomains|
|Autorisation|Obtenir la culture par défaut prise en charge pour CPIM|
|Autorisation|Obtenir les informations d’un flux d’administration|
|Autorisation|Obtenir la liste des UserJourneys pour ce locataire|
|Autorisation|Obtient l’ensemble des cultures disponibles prises en charge pour CPIM|
|Autorisation|Obtenir une stratégie trustFramework|
|Autorisation|Obtenir une stratégie trustFramework au format xml|
|Autorisation|Obtenir un attribut utilisateur|
|Autorisation|Obtenir des attributs utilisateur|
|Autorisation|Obtenir une liste de parcours utilisateur|
|Autorisation|GetIEFPolicies|
|Autorisation|GetIdentityProviders|
|Autorisation|GetTrustFrameworkPolicy|
|Autorisation|Obtient un conteneur de clés CPIM au format jwk|
|Autorisation|Obtient une liste de conteneurs de clés dans le locataire|
|Autorisation|Obtient le type de locataire|
|Autorisation|MigrateTenantMetadata|
|Autorisation|Corriger un IdentityProvider|
|Autorisation|PutTrustFrameworkPolicy|
|Autorisation|PutTrustFrameworkpolicy|
|Autorisation|Supprimer un parcours utilisateur|
|Autorisation|Restaurer une sauvegarde de conteneur de clés CPIM|
|Autorisation|Récupérer des autorisations de l’application V2|
|Autorisation|Récupérer des principaux de service d’application V2 dans le locataire actuel|
|Autorisation|Mettre à jour un IDP personnalisé|
|Autorisation|Mettre à jour un IDP|
|Autorisation|Mettre à jour un IDP local|
|Autorisation|Mettre à jour une application V1|
|Autorisation|Mettre à jour une application V2|
|Autorisation|Mettre à jour une autorisation de l’application V2|
|Autorisation|Mettre à jour la stratégie|
|Autorisation|Mettre à jour un attribut utilisateur|
|Autorisation|Charger une clé chiffrée CPIM|
|Autorisation|Autorisation utilisateur : L'API est désactivée pour l'ensemble de fonctionnalités du locataire|
|Autorisation|Autorisation utilisateur : Accès accordé à l'utilisateur en tant que « Tenant Admin »|
|Autorisation|Autorisation utilisateur : Les droits d'accès « Authenticated Users » ont été accordés à l'utilisateur|
|Autorisation|Vérifier si la fonctionnalité B2C est activée|
|Autorisation|Vérifier si la fonctionnalité est activée|
|Autorisation|Créer le programme|
|Autorisation|Supprimer le programme|
|Autorisation|Lier le contrôle du programme|
|Autorisation|Intégrer les révisions d’accès Azure AD|
|Autorisation|Supprimer le lien du contrôle du programme|
|Autorisation|Mettre à jour le programme|
|Autorisation|Désactiver l’authentification unique Bureau|
|Autorisation|Désactiver l’authentification unique Bureau pour un domaine spécifique|
|Autorisation|Désactiver le proxy d’application|
|Autorisation|Désactiver l’authentification directe|
|Autorisation|Activer l’authentification unique Bureau|
|Gestion de répertoires|Activer l’authentification unique Bureau pour un domaine spécifique|
|Gestion de répertoires|Activer le proxy d’application|
|Gestion de répertoires|Activer l’authentification directe|
|Gestion de répertoires|Créer un domaine personnalisé dans le locataire|
|Gestion de répertoires|Activer la fonctionnalité B2C|
|Gestion de répertoires|Obtenir une liste de domaines personnalisés dans le locataire|
|Gestion de répertoires|Obtenir des propriétés de ressource d’un locataire|
|Gestion de répertoires|Obtenir des informations de locataire|
|Gestion de répertoires|Obtenir des fonctionnalités autorisées de locataire|
|Gestion de répertoires|Obtenir tenantDomains|
|Clé|Obtient le type de locataire|
|Clé|Vérifier si la fonctionnalité B2C est activée|
|Clé|Vérifier si la fonctionnalité est activée|
|Clé|Ajouter un partenaire à l'entreprise|
|Clé|Ajouter un domaine non vérifié|
|Clé|Ajouter un domaine vérifié|
|Clé|Créer une entreprise|
|Clé|Création de paramètres d’entreprise.|
|Clé|Suppression de paramètres d’entreprise.|
|Clé|Rétrograder un partenaire|
|Clé|Répertoire supprimé|
|Autres|Répertoire supprimé définitivement|
|Autres|Suppression du répertoire planifiée|
|Ressource|Promouvoir une entreprise auprès d’un partenaire|
|Ressource|Vider des propriétés de gestion des droits|
|Ressource|Supprimer un partenaire d’une entreprise|
|Ressource|Supprimer un domaine non vérifié|
|Ressource|Supprimer un domaine vérifié|
|Ressource|Définir les informations de l'entreprise|
|Ressource|Définir la fonctionnalité DirSync|
|Ressource|Définir l’indicateur DirSyncEnabled|
|Ressource|Définition d’un partenariat|
|Ressource|Définir un seuil de suppression accidentelle|
|Ressource|Définir un emplacement de données autorisé|
|Ressource|Définir l’activation de la fonctionnalité multinationale|
|Ressource|Définir la fonctionnalité de répertoire sur un locataire|
|Ressource|Définir l'authentification de domaine|
|Ressource|Définir les paramètres de fédération du domaine|
|Ressource|Définir une stratégie de mot de passe|
|Ressource|Définir des propriétés de gestion des droits|
|Ressource|Mettre à jour une entreprise|
|Ressource|Mettre à jour des paramètres d’entreprise|
|Ressource|Mettre à jour le domaine|
|Ressource|Vérifier le domaine|
|Ressource|Vérifier le domaine vérifié par courrier électronique|
|Ressource|Mise en route|
|Ressource|Mettre à jour des paramètres d’alerte|
|Ressource|Mettre à jour des paramètres de synthèse hebdomadaire|
|Ressource|Désactiver la réécriture de mot de passe pour un répertoire|
|Ressource|Activer la réécriture de mot de passe pour un répertoire|
|Ressource|Ajouter une attribution de rôle d’application à un groupe|
|Ressource|Ajouter un groupe|
|Ressource|Ajouter un membre à un groupe|
|Ressource|Ajouter un propriétaire à un groupe|
|Ressource|Créer des paramètres de groupe|
|Ressource|Supprimer un groupe|
|Ressource|Supprimer des paramètres de groupe|
|Ressource|Terminer l’application d’une licence basée sur le groupe à des utilisateurs|
|Ressource|Supprimer définitivement un groupe|
|Ressource|Supprimer une attribution de rôle d’application d’un groupe|
|Ressource|Supprimer un membre d’un groupe|
|Ressource|Supprimer un propriétaire d’un groupe|
|Ressource|Restaurer un groupe|
|Ressource|Définir une licence de groupe|
|Ressource|Définition d’un groupe à gérer par un utilisateur.|
|Ressource|Commencer à appliquer une licence basée sur un groupe à des utilisateurs|
|Ressource|Déclencher un nouveau calcul de licence de groupe|
|Ressource|Mettre à jour un groupe|
|Ressource|Mettre à jour des paramètres d’un groupe|
|Ressource|Ajout d’un membre|
|Ressource|Créer un groupe|
|Ressource|Suppression d’un groupe|
|Ressource|Suppression d’un membre|
|Ressource|Mise à jour d’un groupe|
|Ressource|Approuver une demande en attente pour rejoindre un groupe|
|Ressource|Annuler une demande en attente pour rejoindre un groupe|
|Ressource|Créer une stratégie de gestion du cycle de vie|
|Ressource|Supprimer une demande en attente pour rejoindre un groupe|
|Ressource|Rejeter une demande en attente pour rejoindre un groupe|
|Ressource|Renouveler un groupe|
|Ressource|Demander à rejoindre un groupe|
|Ressource|Définir des propriétés de groupe dynamiques|
|Ressource|Mettre à jour une stratégie de gestion du cycle de vie|
|Ressource|Ajouter une clé basée sur un secret ASCII à un conteneur de clés CPIM|
|Ressource|Ajouter une clé à un conteneur de clés CPIM|
|Ressource|Supprimer un conteneur de clés CPIM|
|Ressource|Supprimer un conteneur de clés|
|Ressource|Obtenir les métadonnées de clé active du conteneur de clés au format JWK|
|Ressource|Obtenir des métadonnées de conteneur de clés|
|Ressource|Obtient un conteneur de clés CPIM au format jwk|
|Ressource|Obtient une liste de conteneurs de clés dans le locataire|
|Ressource|Restaurer une sauvegarde de conteneur de clés CPIM|
|Ressource|Enregistrer un conteneur de clés|
|Ressource|Charger une clé chiffrée CPIM|
|Ressource|Émettre un code d’autorisation pour l’application|
|Ressource|Émettre un id_token pour l’application|


## <a name="core-directory"></a>Répertoire principal

|Catégorie d’audit|Activité|
|---|---|
|Gestion des unités administratives|Télécharger un seul type de détection d’événement à risque|
|Gestion des unités administratives|Télécharger les administrateurs et l’état d’acceptation de synthèse hebdomadaire|
|Gestion des unités administratives|Télécharger tous les types de détections d’événements à risque|
|Gestion des unités administratives|Télécharger les détections d’événements à risque d’utilisateur gratuites|
|Gestion des unités administratives|Télécharger les utilisateurs avec indicateur de risque|
|Gestion des applications|Lot d’invitations traité|
|Gestion des applications|Lot d’invitations chargé|
|Gestion des applications|Ajouter un propriétaire à une stratégie|
|Gestion des applications|Add policy|
|Gestion des applications|Supprimer la stratégie|
|Gestion des applications|Supprimer des informations d’identification de stratégie|
|Gestion des applications|Mettre à jour la stratégie|
|Gestion des applications|Définir la stratégie d’inscription MFA|
|Gestion des applications|Définir une stratégie en matière de risque à la connexion|
|Gestion des applications|Définir une stratégie en matière de risque d’utilisateur|
|Gestion des applications|Accepter des conditions d’utilisation|
|Gestion des applications|Créer des conditions d’utilisation|
|Gestion des applications|Refuser des conditions d’utilisation|
|Gestion des applications|Supprimer des conditions d’utilisation|
|Gestion des applications|Modifier des conditions d’utilisation|
|Gestion des applications|Publier des conditions d’utilisation|
|Gestion des applications|Annuler la publication des conditions d’utilisation|
|Gestion des applications|Ajouter le certificat TLS/SSL de l'application|
|Gestion des applications|Supprimer la liaison TLS|
|Gestion des applications|Inscrire le connecteur|
|Gestion des applications|AdminPolicyDatas-RemoveResources|
|Gestion des applications|AdminPolicyDatas-SetResources|
|Gestion des applications|AdminUserJourneys-GetResources|
|Gestion de répertoires|AdminUserJourneys-RemoveResources|
|Gestion de répertoires|AdminUserJourneys-SetResources|
|Gestion de répertoires|Créer un IdentityProvider|
|Gestion de répertoires|Créer un AdminUserJourney|
|Gestion de répertoires|Créer une ressource json localisée|
|Gestion de répertoires|Créer un IDP personnalisé|
|Gestion de répertoires|Créer un IDP|
|Gestion de répertoires|Créer ou mettre à jour une ressource de répertoire B2C|
|Gestion de répertoires|Créer une stratégie|
|Gestion de répertoires|Créer une stratégie trustFramework|
|Gestion de répertoires|Créer une stratégie trustFramework avec un préfixe configurable|
|Gestion de répertoires|Créer un attribut utilisateur|
|Gestion de répertoires|CreateTrustFrameworkPolicy|
|Gestion de répertoires|Supprimer un IDP|
|Gestion de répertoires|Supprimer un IdentityProvider|
|Gestion de répertoires|Supprimer une ressource de répertoire B2C|
|Gestion de répertoires|Supprimer une stratégie trustFramework|
|Gestion de répertoires|Supprimer un attribut utilisateur|
|Gestion de répertoires|Obtenir des ressources de répertoire B2C dans un groupe de ressources|
|Gestion de répertoires|Obtenir des ressources de répertoire B2C dans un abonnement|
|Gestion de répertoires|Obtenir un IDP personnalisé|
|Gestion de répertoires|Obtenir un IDP|
|Gestion de répertoires|Obtenir une ressource de répertoire B2C|
|Gestion de répertoires|Obtenir un parcours utilisateur|
|Gestion de répertoires|Obtenir des réclamations de l’application autorisées pour le parcours utilisateur|
|Gestion de répertoires|Obtenir des réclamations déclarées automatiquement autorisées pour le parcours utilisateur|
|Gestion de répertoires|Obtenir des réclamations déclarées automatiquement autorisées pour la stratégie|
|Gestion de répertoires|Obtenir la liste des revendications de sortie disponibles|
|Gestion de répertoires|Obtenir les définitions de contenu pour le parcours utilisateur|
|Gestion de répertoires|Obtenir des IDP pour un flux administrateur spécifique|
|Gestion de répertoires|Obtenir une liste de tous les flux d’administration|
|Gestion de répertoires|Obtenir une liste des balises de tous les flux d’administration pour l’ensemble des utilisateurs|
|Gestion des groupes|Téléchargement en bloc des membres du groupe - démarré|
|Gestion des groupes|Téléchargement en bloc des membres du groupe - terminé|
|Gestion des groupes|Importation en bloc des membres du groupe - démarré|
|Gestion des groupes|Importation en bloc des membres du groupe - terminé|
|Gestion des groupes|Suppression en bloc des membres du groupe - démarré|
|Gestion des groupes|Suppression en bloc des membres du groupe - terminé|
|Gestion des groupes|Téléchargement en bloc des groupes - démarré|
|Gestion des groupes|Téléchargement en bloc des groupes - terminé|
|Gestion des groupes|Obtenir une liste des locataires pour un utilisateur|
|Gestion des groupes|Obtenir des revendications déclarées automatiquement des comptes locaux|
|Gestion des groupes|Obtenir une ressource json localisée|
|Gestion des groupes|Obtenir des opérations du fournisseur de ressources Microsoft.AzureActiveDirectory|
|Gestion des groupes|Obtenir des stratégies|
|Gestion des groupes|Obtenir une stratégie|
|Gestion des groupes|Obtenir une liste d’IDP pris en charge|
|Gestion des groupes|Obtenir une liste d’IDP pris en charge du parcours utilisateur|
|Gestion des groupes|Obtenir une liste d’IDP personnalisés définie par le locataire|
|Gestion des groupes|Obtenir une liste d’IDP définie par le locataire|
|Gestion des groupes|Obtenir une liste d’IDP locaux définie par le locataire|
|Gestion des groupes|Obtenir des informations sur un locataire pour un utilisateur afin de créer des ressources|
|Gestion des groupes|Obtenir la culture par défaut prise en charge pour CPIM|
|Gestion des groupes|Obtenir les informations d’un flux d’administration|
|Gestion des groupes|Obtenir la liste des UserJourneys pour ce locataire|
|Gestion des groupes|Obtient l’ensemble des cultures disponibles prises en charge pour CPIM|
|Gestion des groupes|Obtenir une stratégie trustFramework|
|Gestion des groupes|Obtenir une stratégie trustFramework au format xml|
|Gestion des groupes|Obtenir un attribut utilisateur|
|Gestion des stratégies|Obtenir des attributs utilisateur|
|Gestion des stratégies|Obtenir une liste de parcours utilisateur|
|Gestion des stratégies|GetIEFPolicies|
|Gestion des stratégies|GetIdentityProviders|
|Gestion des stratégies|GetTrustFrameworkPolicy|
|Ressource|MigrateTenantMetadata|
|Ressource|Déplacer des ressources|
|Ressource|Corriger un IdentityProvider|
|Ressource|PutTrustFrameworkPolicy|
|Ressource|PutTrustFrameworkpolicy|
|Ressource|Supprimer un parcours utilisateur|
|Ressource|Mettre à jour un IDP personnalisé|
|Ressource|Mettre à jour un IDP|
|Ressource|Mettre à jour un IDP local|
|Ressource|Mettre à jour une ressource de répertoire B2C|
|Ressource|Mettre à jour la stratégie|
|Ressource|Mettre à jour un état d’abonnement|
|Gestion des rôles|Mettre à jour un attribut utilisateur|
|Gestion des rôles|Valider le déplacement des ressources|
|Gestion des rôles|Ajouter un appareil|
|Gestion des rôles|Ajouter une configuration d’appareil|
|Gestion des rôles|Ajouter un propriétaire enregistré à un appareil|
|Gestion des rôles|Ajouter des utilisateurs enregistrés à un appareil|
|Gestion des rôles|Supprimer un appareil|
|Gestion des rôles|Supprimer la configuration de l’appareil|
|Gestion des rôles|Un appareil n’est plus conforme|
|Gestion des rôles|Un appareil n’est plus géré|
|User Management|AccessReview_Review|
|User Management|AccessReview_Update|
|User Management|ActivationAborted|
|User Management|ActivationApproved|
|User Management|ActivationCanceled|
|User Management|ActivationRequested|
|User Management|Ajouter un membre éligible au rôle|
|User Management|Ajouter un membre à un rôle|
|User Management|Ajouter une attribution de rôle à une définition de rôle|
|User Management|Ajouter un rôle à partir d’un modèle|
|User Management|Ajouter un membre inclus dans une étendue à un rôle|
|User Management|Ajouté|
|User Management|Assigner|
|User Management|Création d’utilisateurs en bloc - démarré|
|User Management|Création d’utilisateurs en bloc - terminé|
|User Management|Suppression d’utilisateurs en bloc - démarré|
|User Management|Suppression d’utilisateurs en bloc - terminé|
|User Management|Téléchargement en bloc des utilisateurs - démarré|
|User Management|Téléchargement en bloc des utilisateurs - terminé|
|User Management|Restauration d’utilisateurs supprimés en bloc - démarré|
|User Management|Restauration d’utilisateurs supprimés en bloc - terminé|
|User Management|Invitation en bloc d’utilisateurs - démarré|
|User Management|Invitation en bloc des utilisateurs - terminé|
|User Management|Supprimer un propriétaire enregistré d’un appareil|
|User Management|Supprimer des utilisateurs enregistrés d’un appareil|
|User Management|Supprimer un membre éligible d’un rôle|
|User Management|Supprimer un membre d’un rôle|
|User Management|Supprimer une attribution de rôle d’une définition de rôle|
|User Management|Supprimer un membre inclus dans une étendue d’un rôle|
|User Management|Mettre à jour un appareil|
|User Management|Mettre à jour une configuration d’appareil|
|User Management|Mettre à jour un rôle|






## <a name="identity-protection"></a>Identity Protection

|Catégorie d’audit|Activité|
|---|---|
|Gestion de répertoires|Élever|
|Gestion de répertoires|Supprimé|
|Gestion de répertoires|Modifications des paramètres de rôle|
|Autres|ScanAlertsNow|
|Autres|Connecter|
|Autres|Baisser les privilèges|
|Autres|UpdateAlertSettings|
|Autres|UpdateCurrentState|
|Gestion des stratégies|Révision d’accès terminée|
|Gestion des stratégies|Ajouter un approbateur pour approuver une demande|
|Gestion des stratégies|Ajouter un réviseur pour la révision d’accès|
|User Management|Appliquer une révision d’accès|
|User Management|Créer une révision d’accès|


## <a name="invited-users"></a>Utilisateurs invités

|Catégorie d’audit|Activité|
|---|---|
|Autres|Créer une approbation de demande|
|Autres|Supprimer une révision d’accès|
|User Management|Supprimer un réviseur pour la révision d’accès|
|User Management|Demander l’application du résultat de révision|
|User Management|Demander l’arrêt de la révision|
|User Management|Examiner l’affectation d’application|
|User Management|Examiner l’appartenance au groupe|
|User Management|Examiner l’appartenance au rôle Rbac|


## <a name="microsoft-identity-manager-mim"></a>Microsoft Identity Manager (MIM)

|Catégorie d’audit|Activité|
|---|---|
|Gestion des groupes|Examiner la demande d’approbation de demande|
|Gestion des groupes|Mettre à jour une révision d’accès|
|Gestion des groupes|Mettre à jour les paramètres de notification par e-mail pour la révision d’accès|
|Gestion des groupes|Mettre à jour le paramètre de nombre de la périodicité de révision de l’accès|
|Gestion des groupes|Mettre à jour le paramètre de durée de la périodicité de révision de l’accès en jours|
|User Management|Mettre à jour le paramètre de type de fin de la périodicité de révision de l’accès|
|User Management|Mettre à jour le paramètre de type de la périodicité de révision de l’accès|



## <a name="privileged-identity-management"></a>Privileged Identity Management

|Catégorie d’audit|Activité|
|---|---|
|PIM|ActivationAborted|
|PIM|ActivationApproved|
|PIM|ActivationCanceled|
|PIM|ActivationDenied|
|PIM|ActivationRequested|
|PIM|Ajouté|
|PIM|AddedOutsidePIM|
|PIM|Assigner|
|PIM|DismissAlert|
|PIM|Élever|
|PIM|ReactivateAlert|
|PIM|Supprimé|
|PIM|RemovedOutsidePIM|
|PIM|Demander l’arrêt de la révision|
|PIM|Modifications des paramètres de rôle|
|PIM|ScanAlertsNow|
|PIM|Connecter|
|PIM|Annuler l'assignation|
|PIM|Baisser les privilèges|
|PIM|UpdateAlertSettings|
|PIM|UpdateCurrentState|


## <a name="self-service-group-management"></a>Gestion des groupes en libre service

|Catégorie d’audit|Activité|
|---|---|
|Gestion des groupes|Réinitialiser le mot de passe de l'utilisateur|
|Gestion des groupes|Restaurer un utilisateur|
|Gestion des groupes|Définir le mot de passe utilisateur|
|Gestion des groupes|Définir un gestionnaire d’utilisateurs|
|Gestion des groupes|Définir l’activation des métadonnées de jeton oath d’utilisateurs|
|Gestion des groupes|Mettre à jour un timestamp StsRefreshTokenValidFrom|
|Gestion des groupes|Mise à jour de clés secrètes externes.|
|Gestion des groupes|Mettre à jour l'utilisateur|
|Gestion des groupes|Un administrateur génère un mot de passe temporaire|


## <a name="self-service-password-management"></a>Gestion des mots de passe en libre-service

|Catégorie d’audit|Activité|
|---|---|
|Gestion de répertoires|L’administrateur demande à l’utilisateur de réinitialiser son mot de passe|
|Gestion de répertoires|Assigner un utilisateur externe à une application|
|User Management|E-mail non envoyé, utilisateur désabonné|
|User Management|Inviter un utilisateur externe|
|User Management|Utiliser une invitation d’utilisateur externe|
|User Management|Création d’un locataire viral|
|User Management|Création d’un utilisateur viral|
|User Management|Enregistrement de mot de passe utilisateur|
|User Management|Réinitialisation de mot de passe utilisateur|
|User Management|Blocage suite à une réinitialisation de mot de passe en libre-service|


## <a name="terms-of-use"></a>Conditions d’utilisation

|Catégorie d’audit|Activité|
|---|---|
|Conditions d’utilisation|Accepter des conditions d’utilisation|
|Conditions d’utilisation|Créer des conditions d’utilisation|
|Conditions d’utilisation|Refuser des conditions d’utilisation|
|Conditions d’utilisation|Supprimer le consentement|
|Conditions d’utilisation|Supprimer des conditions d’utilisation|
|Conditions d’utilisation|Modifier des conditions d’utilisation|
|Conditions d’utilisation|Expiration des conditions d’utilisation|
|Conditions d’utilisation|Supprimer définitivement des conditions d’utilisation|
|Conditions d’utilisation|Publier des conditions d’utilisation|
|Conditions d’utilisation|Annuler la publication des conditions d’utilisation|


## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble des rapports Azure AD](overview-reports.md).
- [Rapport de journaux d’audit](concept-audit-logs.md) 
- [Accès par programmation aux rapports Azure AD](concept-reporting-api.md)
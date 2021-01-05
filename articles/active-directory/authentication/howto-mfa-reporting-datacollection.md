---
title: Collecte de données utilisateur via Azure AD MFA - Azure Active Directory
description: Quelles sont les informations utilisées pour authentifier des utilisateurs via Azure AD Multi-Factor Authentication ?
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 11/21/2019
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.openlocfilehash: c2124ca9fae76d87a695a2c453f7b222e7a08794
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96742791"
---
# <a name="azure-ad-multi-factor-authentication-user-data-collection"></a>Collecte de données utilisateur via Azure AD Multi-Factor Authentication

Ce document explique comment rechercher les informations utilisateur collectées par le serveur Azure Multi-Factor Authentication (serveur MFA) et Azure AD MFA (dans le cloud) dans le cas où vous souhaiteriez les supprimer.

[!INCLUDE [gdpr-hybrid-note](../../../includes/gdpr-hybrid-note.md)]

## <a name="information-collected"></a>Informations collectées

Le serveur MFA, l'extension NPS et l'adaptateur Windows Server 2016 Azure AD MFA AD FS collectent et stockent les informations suivantes pendant 90 jours.

Tentatives d’authentification (utilisées pour la création de rapports et la résolution des problèmes) :

- Timestamp
- Nom d’utilisateur
- Prénom
- Nom
- Adresse de messagerie
- Groupe d’utilisateurs
- Méthode d’authentification (appel téléphonique, message texte, application mobile, jeton OATH)
- Mode d’appel téléphonique (standard, PIN)
- Direction du message texte (unidirectionnelle, bidirectionnelle)
- Mode du message texte (OTP, OTP + PIN)
- Mode d’application mobile (standard, PIN)
- Mode de jeton OATH (standard, PIN)
- Type d’authentification
- Nom de l’application
- Indicatif de pays de l’appel principal
- Numéro de téléphone de l’appel principal
- Extension de l’appel principal
- Appel principal authentifié
- Résultat de l’appel principal
- Indicatif de pays de l’appel de sauvegarde
- Numéro de téléphone de l’appel de sauvegarde
- Extension de l’appel de sauvegarde
- Appel de sauvegarde authentifié
- Résultat de l’appel de sauvegarde
- Authentification globale
- Résultat global
- Résultats
- Authentifiée
- Résultats
- Adresse IP d’initialisation
- Appareils
- Jeton d’appareil
- Type d’appareil
- Version de l’application mobile
- Version du SE
- Résultats
- Utilisation de la vérification de la présence de notifications

Activations (tentatives d’activation d’un compte dans l’application mobile Microsoft Authenticator) :
- Nom d’utilisateur
- Nom du compte
- Timestamp
- Obtenir le résultat du code d’activation
- Succès de l’activation
- Erreur d’activation
- Résultat de l’état d’activation
- Nom de l’appareil
- Type d’appareil
- Version de l’application
- Activation du jeton OATH

Blocs (utilisés pour déterminer l’état de blocage pour la création de rapports) :

- Horodatage du blocage
- Blocage par nom d’utilisateur
- Nom d’utilisateur
- Indicatif de pays
- Numéro de téléphone
- Numéro de téléphone formaté
- Extension
- Nettoyer l’extension
- Bloqué
- Raison du blocage
- Horodatage de l’achèvement
- Motif de la saisie semi-automatique
- Verrouillage de compte
- Alerte de fraude
- Alerte de fraude non bloquée
- Langage

Contournements (utilisé pour la création de rapports) :

- Horodatage du contournement
- Nombre de secondes du contournement
- Contournement par nom d’utilisateur
- Nom d’utilisateur
- Indicatif de pays
- Numéro de téléphone
- Numéro de téléphone formaté
- Extension
- Nettoyer l’extension
- Motif du contournement
- Horodatage de l’achèvement
- Motif de la saisie semi-automatique
- Utilisation du contournement

Modifications (utilisé pour synchroniser les modifications utilisateur sur le serveur MFA ou Azure AD) :

- Horodatage de la modification
- Nom d’utilisateur
- Nouvel indicatif de pays
- Nouveau numéro de téléphone
- Nouvelle extension
- Nouvel indicatif de pays de sauvegarde
- Nouveau numéro de téléphone de sauvegarde
- Nouvelle extension de sauvegarde
- Nouveau PIN
- Modification du PIN requise
- Ancien jeton d’appareil
- Nouveau jeton d’appareil

## <a name="gather-data-from-mfa-server"></a>Collecter des données à partir du serveur MFA

Pour MFA Server version 8.0 ou ultérieure le processus suivant permet aux administrateurs d’exporter toutes les données pour les utilisateurs :

- Connectez-vous à votre serveur MFA, accédez à l’onglet **Utilisateurs**, sélectionnez l’utilisateur en question, puis cliquez sur le bouton **Modifier**. Faites des captures d’écran (Alt + Imp. écr.) de chaque onglet pour fournir à l’utilisateur ses paramètres MFA actuels.
- À partir de la ligne de commande du serveur MFA, exécutez la commande suivante en remplaçant le chemin d’accès en fonction de votre installation `C:\Program Files\Multi-Factor Authentication Server\MultiFactorAuthGdpr.exe export <username>` pour générer un fichier au format JSON.
- Les administrateurs peuvent également utiliser l’opération GetUserGdpr du Kit de développement logiciel (SDK) Service Web sous forme d’option pour exporter toutes les informations du service cloud MFA collectées pour un utilisateur donné ou les intégrer dans une solution de création de rapports plus vaste.
- Recherchez `C:\Program Files\Multi-Factor Authentication Server\Logs\MultiFactorAuthSvc.log` et toutes les sauvegardes pour « \<username> » (incluez les guillemets dans la recherche) pour rechercher toutes les instances de l’enregistrement utilisateur ajoutées ou modifiées.
   - Ces enregistrements peuvent être limités (mais pas éliminés) en décochant l'option **Consigner les modifications des utilisateurs** dans Expérience utilisateur du serveur MFA, section Journalisation, onglet Fichiers journaux.
   - Si le journal système est configuré, et que l'option **Consigner les modifications des utilisateurs** est cochée dans Expérience utilisateur du serveur MFA, section Journalisation, onglet Syslog, les entrées de journal peuvent être collectées à partir de syslog.
- Les autres occurrences du nom d’utilisateur dans MultiFactorAuthSvc.log et les autres fichiers journaux du serveur MFA appartenant aux tentatives d’authentification sont considérés comme des données opérationnelles et répétées pour les informations fournies à l’aide de l’exportation de MultiFactorAuthGdpr.exe ou de l’opération GetUserGdpr du Kit de développement logiciel (SDK) Web Service.

## <a name="delete-data-from-mfa-server"></a>Supprimer les données du serveur MFA

À partir de la ligne de commande du serveur MFA, exécutez la commande suivante en remplaçant le chemin d’accès en fonction de votre installation `C:\Program Files\Multi-Factor Authentication Server\MultiFactorAuthGdpr.exe delete <username>` pour supprimer toutes les informations de service cloud MFA collectées pour cet utilisateur.

- Les données incluses dans l’exportation de données sont supprimées en temps réel, cependant la suppression totale des données opérationnelles et répétées peut prendre jusqu’à 30 jours.
- Les administrateurs peuvent également utiliser l’opération DeleteUserGdpr du Kit de développement logiciel (SDK) Web Service sous forme d’option pour supprimer toutes les informations du service cloud MFA collectées pour un utilisateur donné ou les intégrer dans une solution de création de rapports plus vaste.

## <a name="gather-data-from-nps-extension"></a>Collecter des données à partir de l’extension NPS

Utilisez le [Portail de confidentialité de Microsoft](https://portal.azure.com/#blade/Microsoft_Azure_Policy/UserPrivacyMenuBlade/Overview) pour effectuer une demande d’exportation.

- Les informations MFA sont incluses dans l’exportation qui peut donc prendre des heures ou des jours.
- Les occurrences du nom d’utilisateur dans les journaux d’événements AzureMfa/AuthN/AuthNOptCh, AzureMfa/AuthZ/AuthZAdminCh et AzureMfa/AuthZ/AuthZOptCh sont considérées comme des données opérationnelles et répétées pour les informations fournies dans l’exportation.

## <a name="delete-data-from-nps-extension"></a>Supprimer des données de l’Extension NPS

Utilisez le [Portail de confidentialité de Microsoft](https://portal.azure.com/#blade/Microsoft_Azure_Policy/UserPrivacyMenuBlade/Overview) pour effectuer une demande de fermeture de compte afin de supprimer toutes les informations du service cloud MFA collectées pour cet utilisateur.

- La suppression totale de toutes les données peut prendre jusqu’à 30 jours.

## <a name="gather-data-from-windows-server-2016-azure-ad-mfa-ad-fs-adapter"></a>Collecter des données à partir de l'adaptateur Windows Server 2016 Azure AD MFA AD FS

Utilisez le [Portail de confidentialité de Microsoft](https://portal.azure.com/#blade/Microsoft_Azure_Policy/UserPrivacyMenuBlade/Overview) pour effectuer une demande d’exportation. 

- Les informations MFA sont incluses dans l’exportation qui peut donc prendre des heures ou des jours.
- Les occurrences du nom d’utilisateur dans les journaux d’événements AD FS Tracing/Debug (s’ils sont activés) sont considérées comme des données opérationnelles et répétées pour les informations fournies dans l’exportation.

## <a name="delete-data-from-windows-server-2016-azure-ad-mfa-ad-fs-adapter"></a>Supprimer des données à partir de l'adaptateur Windows Server 2016 Azure AD MFA AD FS

Utilisez le [Portail de confidentialité de Microsoft](https://portal.azure.com/#blade/Microsoft_Azure_Policy/UserPrivacyMenuBlade/Overview) pour effectuer une demande de fermeture de compte afin de supprimer toutes les informations du service cloud MFA collectées pour cet utilisateur.

- La suppression totale de toutes les données peut prendre jusqu’à 30 jours.

## <a name="gather-data-for-azure-ad-mfa"></a>Collecter des données pour Azure AD Multi-Factor Authentication

Utilisez le [Portail de confidentialité de Microsoft](https://portal.azure.com/#blade/Microsoft_Azure_Policy/UserPrivacyMenuBlade/Overview) pour effectuer une demande d’exportation.

- Les informations MFA sont incluses dans l’exportation qui peut donc prendre des heures ou des jours.

## <a name="delete-data-for-azure-ad-mfa"></a>Supprimer des données pour Azure AD Multi-Factor Authentication

Utilisez le [Portail de confidentialité de Microsoft](https://portal.azure.com/#blade/Microsoft_Azure_Policy/UserPrivacyMenuBlade/Overview) pour effectuer une demande de fermeture de compte afin de supprimer toutes les informations du service cloud MFA collectées pour cet utilisateur.

- La suppression totale de toutes les données peut prendre jusqu’à 30 jours.

## <a name="next-steps"></a>Étapes suivantes

[Création de rapports du serveur MFA](howto-mfa-reporting.md)

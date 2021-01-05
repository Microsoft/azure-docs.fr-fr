---
title: Trouver les rapports d’activité utilisateur dans le Portail Azure | Microsoft Docs
description: Découvrez où se trouvent les rapports d’activité utilisateur Azure Active Directory sur le portail Azure.
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.service: active-directory
ms.topic: how-to
ms.workload: identity
ms.subservice: report-monitor
ms.date: 11/13/2018
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: 42afa073da9197c12e4cbd316d311a7699d9a95f
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96013021"
---
# <a name="find-activity-reports-in-the-azure-portal"></a>Trouver les rapports d’activité sur le Portail Azure

Dans cet article, vous allez apprendre à rechercher les rapports d’activité utilisateur Azure Active Directory (Azure AD) sur le portail Azure.

## <a name="audit-logs-report"></a>Rapport de journaux d’audit

Le rapport de journaux d’audit regroupe plusieurs rapports en lien avec les activités des applications sur une vue unique pour la création de rapports contextuels. Pour accéder au rapport de journaux d’audit :

1. Accédez au [portail Azure](https://portal.azure.com).
2. Sélectionnez votre annuaire en haut à droite de l'écran, puis sélectionnez le panneau **Azure Active Directory** à partir du volet de navigation de gauche.
3. Sélectionnez **Journaux d’audit** dans la section **Activité** du panneau Azure Active Directory. 

    ![Journaux d’audit](./media/howto-find-activity-reports/482.png "Journaux d’audit")

Le rapport de journaux d’audit regroupe les rapports suivants :

* Rapport d’audit
* Activité de réinitialisation de mot de passe
* Activité de l’enregistrement de la réinitialisation de mot de passe
* Activité de groupes en libre-service
* Changements de nom de groupe d’Office 365
* Activité d’approvisionnement de compte
* État de substitution de mot de passe
* Erreurs de configuration de compte

### <a name="filtering-on-audit-logs"></a>Filtrage sur les journaux d’audit

Vous pouvez appliquer le filtrage avancé au rapport d’audit pour accéder à une catégorie spécifique de données d’audit, en spécifiant celle-ci dans le filtre **Catégorie**. Par exemple, pour afficher toutes les activités liées aux utilisateurs, sélectionnez la catégorie **UserManagement**. 

Les catégories sont les suivantes :

- Tous
- AdministrativeUnit
- ApplicationManagement
- Authentification
- Autorisation
- Contact
- Appareil
- DeviceConfiguration
- DirectoryManagement
- EntitlementManagement
- GroupManagement
- Autres
- Stratégie
- ResourceManagement
- RoleManagement
- UserManagement

Vous pouvez également filtrer sur un service spécifique à l’aide du filtre de liste déroulante **Service**. Par exemple, pour obtenir tous les événements d’audit liés à la gestion de mot de passe en libre-service, sélectionnez le filtre **gestion de mot de passe libre-service**.

Les services sont les suivants :

- Tous
- Révisions d’accès
- Approvisionnement des comptes 
- Authentification unique des applications
- Méthodes d’authentification
- B2C
- Accès conditionnel
- Annuaire principal
- Gestion des droits d’utilisation
- Identity Protection
- Utilisateurs invités
- PIM
- Gestion des groupes en libre-service
- Gestion des mots de passe en libre-service
- Conditions d’utilisation

## <a name="sign-ins-report"></a>Rapport de connexions 

La vue **Connexions** inclut toutes les connexions utilisateur ainsi que le rapport **Utilisation des applications**. Vous pouvez également afficher des informations sur l’utilisation des applications dans la section **Gérer** de la vue d’ensemble **Applications d’entreprise**.

Pour accéder au rapport de connexions :

1. Accédez au [portail Azure](https://portal.azure.com).
2. Sélectionnez votre annuaire en haut à droite de l'écran, puis sélectionnez le panneau **Azure Active Directory** à partir du volet de navigation de gauche.
3. Sélectionnez **Connexions** dans la section **Activité** du panneau Azure Active Directory. 

    ![Affichage des connexions](./media/howto-find-activity-reports/483.png "Affichage des connexions")


### <a name="filtering-on-application-name"></a>Filtrage sur le nom de l’application

Vous pouvez utiliser le rapport de connexions pour afficher des détails sur l’utilisation d'une application, en filtrant sur le nom d’utilisateur ou sur le nom de l’application.

![page Filtrer les événements de connexion](./media/howto-find-activity-reports/07.png "Page Filtrer les événements de connexion")

## <a name="security-reports"></a>Rapports de sécurité

### <a name="anomalous-activity-reports"></a>Rapports d’activités anormales

Les rapports d’activités anormales fournissent des informations sur les détections d’événements à risque qu’Azure AD est capable de signaler.

Le tableau suivant liste les rapports d’activités anormales Azure AD et les types de détections d’événements à risque correspondants dans le portail Azure. Pour plus d’informations, consultez [Détections des risques dans Azure Active Directory](../identity-protection/overview-identity-protection.md).  


| Rapport d’activités anormales Azure AD |  Type de détection d’événement à risque Identity Protection|
| :--- | :--- |
| Utilisateurs avec des informations d’identification volées | Informations d’identification divulguées |
| Activité de connexion anormale | Voyage impossible vers des emplacements inhabituels |
| Connexions à partir d’appareils potentiellement infectés | Connexions depuis des appareils infectés|
| Connexions à partir de sources inconnues | Connexions depuis des adresses IP anonymes |
| Connexions depuis des adresses IP avec des activités suspectes | Connexions depuis des adresses IP avec des activités suspectes |
| - | Connexions depuis des emplacements non connus |

Les rapports d’activités anormales Azure AD suivants ne sont pas inclus en tant que détections d’événements à risque dans le portail Azure :

* Connexions après plusieurs échecs
* Connexions depuis plusieurs zones géographiques


### <a name="detected-risk-detections"></a>Détections d’événements à risque trouvées

Vous pouvez accéder aux rapports sur les détections d’événements à risque dans la section **Sécurité** du panneau **Azure Active Directory** du [portail Azure](https://portal.azure.com). Les détections d’événements à risque sont suivies dans les rapports suivants :   

- [Utilisateurs à risque](../identity-protection/overview-identity-protection.md)
- [Connexions risquées](../identity-protection/overview-identity-protection.md)

    ![Rapports de sécurité](./media/howto-find-activity-reports/04.png "Rapports de sécurité")

## <a name="troubleshoot-issues-with-activity-reports"></a>Résoudre les problèmes liés aux rapports d’activité

### <a name="missing-data-in-the-downloaded-activity-logs"></a>Données manquantes dans les journaux d’activité téléchargés

#### <a name="symptoms"></a>Symptômes 

J’ai téléchargé les journaux d’activité (d’audit ou de connexion) et tous les enregistrements correspondant à la période choisie n’apparaissent pas. Pourquoi ? 

 ![Capture d’écran montrant le bouton Télécharger dans le rapport d’activité.](./media/troubleshoot-missing-data-download/01.png)
 
#### <a name="cause"></a>Cause

Lorsque vous téléchargez des journaux d’activité à partir du portail Azure, nous limitons l’échelle à 250 000 enregistrements, classés du plus récent au moins récent. 

#### <a name="resolution"></a>Résolution

Vous pouvez tirer parti des [API de création de rapports Azure AD](concept-reporting-api.md) pour extraire jusqu’à un million d’enregistrements pour un point donné.

### <a name="missing-audit-data-for-recent-actions-in-the-azure-portal"></a>Données d’audit manquantes pour des actions récentes dans le portail Azure

#### <a name="symptoms"></a>Symptômes

J’ai réalisé certaines actions dans le portail Azure et je pensais pouvoir consulter les journaux d’audit associés dans le panneau `Activity logs > Audit Logs`, mais je ne les trouve pas.

 ![Capture d’écran montrant le rapport d’activité.](./media/troubleshoot-missing-audit-data/01.png)
 
#### <a name="cause"></a>Cause

Les actions n’apparaissent pas immédiatement dans les journaux d’activité. Le tableau ci-dessous énumère nos valeurs de latence pour les journaux d’activité. 

| Rapport | Latence (P95) | Latence (P99) |
|--------|---------------|---------------|
| Audit de répertoire | 2 minutes | 5 minutes |
| Activité de connexion | 2 minutes | 5 minutes |

#### <a name="resolution"></a>Résolution

Attendez entre 15 minutes et deux heures pour voir si les actions apparaissent dans le journal. Si vous ne voyez toujours pas les journaux d’activité après deux heures, [créez un ticket de support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) et nous l’étudierons.

### <a name="missing-logs-for-recent-user-sign-ins-in-the-azure-ad-sign-ins-activity-log"></a>Journaux d’activité manquants pour des connexions utilisateur récentes dans le journal d’activité des connexions à Azure AD

#### <a name="symptoms"></a>Symptômes

Je me suis connecté récemment au portail Azure et je pensais pouvoir consulter les journaux d’activité de connexion associés dans le panneau `Activity logs > Sign-ins`, mais je ne les trouve pas.

 ![Capture d’écran montrant les connexions pour Azure Active Directory.](./media/troubleshoot-missing-audit-data/02.png)
 
#### <a name="cause"></a>Cause

Les actions n’apparaissent pas immédiatement dans les journaux d’activité. Le tableau ci-dessous énumère nos valeurs de latence pour les journaux d’activité. 

| Rapport | Latence (P95) | Latence (P99) |
|--------|---------------|---------------|
| Audit de répertoire | 2 minutes | 5 minutes |
| Activité de connexion | 2 minutes | 5 minutes |

#### <a name="resolution"></a>Résolution

Attendez entre 15 minutes et deux heures pour voir si les actions apparaissent dans le journal. Si vous ne voyez toujours pas les journaux d’activité après deux heures, [créez un ticket de support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) et nous l’étudierons.

### <a name="i-cant-view-more-than-30-days-of-report-data-in-the-azure-portal"></a>Je n’arrive pas à afficher plus de 30 jours de données de rapport dans le portail Azure

#### <a name="symptoms"></a>Symptômes

Je n’arrive pas à afficher plus de 30 jours de données de connexion et d’audit dans le portail Azure. Pourquoi ? 

 ![Capture d’écran montrant le menu Date.](./media/troubleshoot-missing-audit-data/03.png)

#### <a name="cause"></a>Cause

Selon votre licence, les actions Azure Active Directory stockent les rapports d’activité pour les durées suivantes :

| Rapport           | Azure AD Gratuit | Azure AD Premium P1 | Azure AD Premium P2 |
| ---              | ---           | ---                 | ---
| Audit de répertoire  | 7 jours        | 30 jours             | 30 jours             |
| Activité de connexion | Non disponible. Vous ne pouvez pas accéder à vos propres connexions pendant 7 jours depuis le panneau de profil utilisateur individuel | 30 jours | 30 jours             |

Pour plus d’informations, consultez [Stratégies de rétention des rapports Azure Active Directory](reference-reports-data-retention.md).  

#### <a name="resolution"></a>Résolution

Vous avez deux options pour conserver les données pendant plus de 30 jours. Vous pouvez utiliser les [API de création de rapports Azure AD](concept-reporting-api.md) pour récupérer les données par programmation et les stocker dans une base de données. Sinon, vous pouvez intégrer des journaux d’audit dans un système SIEM tiers tel que Splunk ou SumoLogic.

## <a name="next-steps"></a>Étapes suivantes

* [Vue d’ensemble des journaux d’audit](concept-audit-logs.md)
* [Vue d’ensemble des connexions](concept-sign-ins.md)
* [Vue d’ensemble des événements à risque](../identity-protection/overview-identity-protection.md)
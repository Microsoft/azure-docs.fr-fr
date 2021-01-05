---
title: Rapports d’activité de connexion dans le portail Azure Active Directory | Microsoft Docs
description: Présentation des rapports d’activité de connexion dans le portail Azure Active Directory
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: 4b18127b-d1d0-4bdc-8f9c-6a4c991c5f75
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 03/24/2020
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: d61962667953b20f4b542874e902411bb579b9c3
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93122841"
---
# <a name="sign-in-activity-reports-in-the-azure-active-directory-portal"></a>Rapports d’activité de connexion dans le portail Azure Active Directory

L’architecture de création de rapports dans Azure Active Directory (Azure AD) comprend les composants suivants :

- **Activité** 
    - **Connexions**  – Il s’agit d’informations sur l’utilisation des applications managées et les activités de connexion des utilisateurs.
    - **Journaux d’audit** - [Journaux d’audit](concept-audit-logs.md) : fournit des informations sur les activités du système liées aux utilisateurs et à la gestion des groupes, les applications gérées et les activités de répertoire.
    - **Les journaux de provisionnement** - [journaux de provisionnement](./concept-provisioning-logs.md) permettent aux clients de superviser l’activité effectuée par le service de provisionnement, telle que la création d’un groupe dans ServiceNow ou l’importation d’un utilisateur à partir de Workday. 
- **Sécurité** 
    - **Connexions risquées** : une [connexion risquée](../identity-protection/overview-identity-protection.md) correspond à un indicateur de tentative de connexion d’un utilisateur autre que le propriétaire légitime d’un compte d’utilisateur.
    - **Utilisateurs avec indicateur de risque**  : un [utilisateur à risque](../identity-protection/overview-identity-protection.md) correspond à un indicateur de compte d’utilisateur susceptible d’être compromis.

Cet article présente une vue d’ensemble du rapport de connexions.

## <a name="prerequisites"></a>Prérequis

### <a name="who-can-access-the-data"></a>Qui peut accéder aux données ?

* Les utilisateurs ayant les rôles Administrateur de la sécurité, Lecteur Sécurité, Lecteur général et Lecteur de rapports
* Les administrateurs généraux
* Tous les utilisateurs (non administrateurs) peuvent accéder à leurs propres connexions 

### <a name="what-azure-ad-license-do-you-need-to-access-sign-in-activity"></a>De quelle licence Azure AD avez-vous besoin pour accéder à l’activité de connexion ?

Le rapport relatif à l'activité de connexion est disponible dans [toutes les éditions d'Azure AD](reference-reports-data-retention.md#how-long-does-azure-ad-store-the-data) ; il est également accessible via l'API Microsoft Graph.

## <a name="sign-ins-report"></a>Rapport de connexions

Le rapport de connexions des utilisateurs permet de répondre aux questions suivantes :

* Quel est le modèle de connexion d’un utilisateur ?
* Combien d’utilisateurs se sont connectés au cours d’une semaine ?
* Quel est l’état de ces connexions ?

Dans le menu du [portail Azure](https://portal.azure.com), sélectionnez **Azure Active Directory** ou recherchez et sélectionnez **Azure Active Directory** dans n’importe quelle page.

![Sélectionner Azure Active Directory](./media/concept-sign-ins/select-azure-active-directory.png "Azure Active Directory")

Sous **Surveillance** , sélectionnez **Connexions** pour ouvrir le [Rapport de connexions](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/SignIns).

![Capture d’écran montrant des connexions sélectionnées dans le menu Supervision.](./media/concept-sign-ins/monitoring-sign-ins-in-azure-active-directory.png "Activité de connexion")

Il peut s’écouler jusqu’à deux heures avant que certains enregistrements de connexion s’affichent dans le portail.

> [!IMPORTANT]
> Le rapport des connexions montre seulement les connexions **interactives** , c’est-à-dire les connexions où un utilisateur se connecte manuellement avec son nom d’utilisateur et son mot de passe. Les connexions non interactives, comme l’authentification de service à service, n’apparaissent pas dans le rapport des connexions. 

Un journal de connexion inclut un mode Liste par défaut, qui indique :

- La date de connexion
- L’utilisateur associé
- L’application à laquelle l’utilisateur s’est connecté
- L’état de connexion
- L’état de la détection de risque
- L’état de l’exigence de l’authentification multifacteur (MFA)

![Capture d’écran montrant les connexions Office 365 SharePoint Online.](./media/concept-sign-ins/sign-in-activity.png "Activité de connexion")

Vous pouvez personnaliser le mode Liste en cliquant sur **Colonnes** dans la barre d’outils.

![Capture d’écran montrant l’option Colonnes dans la page Connexions.](./media/concept-sign-ins/19.png "Activité de connexion")

La boîte de dialogue **Colonnes** vous permet d’accéder aux attributs sélectionnables. Dans un rapport de connexion, vous ne pouvez pas avoir de champs qui contiennent plusieurs valeurs pour une demande de connexion donnée sous forme de colonne. C’est par exemple le cas pour les détails d’authentification, les données d’accès conditionnel et l’emplacement réseau.   

![Capture d’écran montrant la boîte de dialogue Colonnes dans laquelle vous pouvez sélectionner des attributs.](./media/concept-sign-ins/columns.png "Activité de connexion")

Sélectionnez un élément dans la vue sous forme de liste pour obtenir des informations plus détaillées.

![Capture d’écran montrant une vue d’informations détaillées.](./media/concept-sign-ins/basic-sign-in.png "Activité de connexion")

> [!NOTE]
> Les clients peuvent maintenant résoudre les problèmes de stratégies d’accès conditionnel grâce à tous les rapports de connexion. Les clients peuvent examiner l’état de l’accès conditionnel et consulter en détail les stratégies applicables à la connexion et les résultats de chaque stratégie en cliquant sur l’onglet **Accès conditionnel** pour obtenir un rapport de connexion.
> Pour en savoir plus, consultez le [Forum aux questions sur les informations de l’accès conditionnel dans toutes les connexions](reports-faq.md#conditional-access).



## <a name="filter-sign-in-activities"></a>Filtrer les activités de connexion

Commencez par réduire les données signalées jusqu’au niveau qui vous convient. Ensuite, filtrez les données de connexions en utilisant le champ de date comme filtre par défaut. Azure AD offre une large gamme de filtres supplémentaires que vous pouvez définir :

![Capture d’écran montrant l’option Ajouter des filtres.](./media/concept-sign-ins/04.png "Activité de connexion")

**ID de requête**  : ID de la requête qui vous intéresse.

**Utilisateur**  : nom ou nom d’utilisateur principal (UPN) de l’utilisateur qui vous intéresse.

**Application**  : nom de l’application cible.
 
**État**  : état de connexion qui vous intéresse :

- Succès

- Échec

- Interrompu


**Adresse IP**  : adresse IP de l’appareil utilisé pour se connecter à votre locataire.

**Emplacement**  : emplacement à partir duquel la connexion a été établie :

- City

- Département/Province

- Pays/région


**Ressource**  : nom du service utilisé pour la connexion.


**ID de ressource**  : ID du service utilisé pour la connexion.


**Application cliente**  : type de l’application cliente utilisée pour se connecter à votre locataire :

![Filtre d’application cliente](./media/concept-sign-ins/client-app-filter.png)


|Nom|Authentification moderne|Description|
|---|:-:|---|
|SMTP authentifié| |Utilisé par les clients POP et IMAP pour envoyer des e-mails.|
|Découverte automatique| |Utilisé par les clients Outlook et EAS pour rechercher des boîtes aux lettres dans Exchange Online et s’y connecter.|
|Exchange ActiveSync| |Ce filtre affiche toutes les tentatives de connexion où le protocole EAS a été utilisé.|
|Browser|![Coche bleue](./media/concept-sign-ins/check.png)|Affiche toutes les tentatives de connexion d’utilisateurs à l’aide de navigateurs web|
|Exchange ActiveSync| | Affiche toutes les tentatives de connexion d’utilisateurs avec des applications clientes utilisant Exchange ActiveSync pour se connecter à Exchange Online|
|Exchange Online PowerShell| |Utilisé pour se connecter à Exchange Online à l’aide de PowerShell à distance. Si vous bloquez l’authentification de base pour Exchange Online PowerShell, vous devez utiliser le module Exchange Online PowerShell pour vous connecter. Pour obtenir des instructions, consultez [Se connecter à Exchange Online PowerShell à l’aide de l’authentification multifacteur](/powershell/exchange/exchange-online/connect-to-exchange-online-powershell/mfa-connect-to-exchange-online-powershell).|
|Exchange Web Services| |Interface de programmation utilisée par Outlook, Outlook pour Mac et des applications tierces.|
|IMAP4| |Un client de messagerie hérité qui utilise IMAP pour récupérer le courrier électronique.|
|MAPI sur HTTP| |Utilisé par Outlook 2010 et versions ultérieures.|
|Applications mobiles et clients de bureau|![Coche bleue](./media/concept-sign-ins/check.png)|Affiche toutes les tentatives de connexion d’utilisateurs à l’aide d’applications mobiles et de clients de bureau.|
|Carnet d’adresses en mode hors connexion| |Copie des collections de listes d’adresses téléchargées et utilisées par Outlook.|
|Outlook Anywhere (RPC sur HTTP)| |Utilisé par Outlook 2016 et versions antérieures.|
|Service Outlook| |Utilisé par l’application Courrier et Calendrier pour Windows 10.|
|POP3| |Un client de messagerie hérité qui utilise POP3 pour récupérer le courrier électronique.|
|Reporting Web Services| |Utilisé pour récupérer des données de rapports dans Exchange Online.|
|Autres clients| |Affiche toutes les tentatives de connexion d’utilisateurs où l’application cliente n’est pas incluse ou connue.|



**Système d’exploitation**  : le système d’exploitation s’exécutant sur l’appareil a utilisé l’authentification auprès de votre locataire. 


**Explorateur d’appareils**  : si la connexion a été établie à partir d’un navigateur, ce champ vous permet de filtrer par nom de navigateur.


**ID de corrélation**  : ID de corrélation de l’activité.




**Accès conditionnel**  : état des règles d’accès conditionnel appliquées

- **Non applicable**  : aucune stratégie n’est appliquée à l’utilisateur et à l’application lors de la connexion.

- **Réussite**  : une ou plusieurs stratégies d’accès conditionnel sont appliquées à l’utilisateur et à l’application (mais pas nécessairement les autres conditions) lors de la connexion. 

- **Échec**  : la connexion a satisfait à la condition d’utilisateur et d’application d’au moins une stratégie d’accès conditionnel et les contrôles d’octroi ne sont pas satisfaisants ou ne sont pas configurés pour bloquer l’accès.









## <a name="download-sign-in-activities"></a>Télécharger les activités de connexion

Cliquez sur l’option **Télécharger** pour créer un fichier CSV ou JSON des 250 000 enregistrements les plus récents. Commencez par [télécharger les données de connexions](quickstart-download-sign-in-report.md) si vous souhaitez les utiliser en dehors du portail Azure.  

![Télécharger](./media/concept-sign-ins/71.png "Téléchargement")

> [!IMPORTANT]
> Le nombre d’enregistrements que vous pouvez télécharger est limité par les [stratégies de rétention de rapport Azure Active Directory](reference-reports-data-retention.md).  


## <a name="sign-ins-data-shortcuts"></a>Raccourcis vers les données de connexions

Azure AD et le portail Azure vous offrent d’autres points d’entrée pour accéder aux données de connexions :

- Vue d’ensemble de la protection de la sécurité des identités
- Utilisateurs
- Groupes
- Applications d’entreprise

### <a name="users-sign-ins-data-in-identity-security-protection"></a>Données de connexion des utilisateurs dans la protection de la sécurité des identités

Le graphique des connexions des utilisateurs figurant sur la page de présentation **Protection de la sécurité des identités** affiche les agrégations hebdomadaires des connexions. La valeur par défaut de cette période est de 30 jours.

![Capture d’écran montrant un graphique illustrant les connexions sur un mois.](./media/concept-sign-ins/06.png "Activité de connexion")

Lorsque vous cliquez sur un jour dans le graphique des connexions, vous obtenez une vue d’ensemble des activités de connexion correspondantes.

Chaque ligne de la liste des activités de connexion affiche :

* Qui s’est connecté ?
* Quelle application a été la cible de la connexion ?
* Quel est l’état de la connexion ?
* Quel est l’état MFA de la connexion ?

En cliquant sur un élément, vous obtenez plus d’informations sur l’opération de connexion :

- ID d'utilisateur
- Utilisateur
- Nom d’utilisateur
- ID de l'application
- Application
- Client
- Emplacement
- Adresse IP
- Date
- MFA obligatoire
- État de la connexion

> [!NOTE]
> Les adresses IP sont émises de manière à ce qu’il n’existe aucune connexion définitive entre une adresse IP et l’endroit où se trouve physiquement l’ordinateur avec cette adresse. Le mappage des adresses IP est compliqué par le fait que les fournisseurs mobiles et les VPN qui émettent des adresses IP à partir de pools centraux sont souvent très éloignés de l’endroit où l’appareil client est réellement utilisé. Pour le moment, la conversion de l’adresse IP en un emplacement physique constitue la meilleure solution pour les suivis, les données de registre, les recherches inversées et d’autres informations dans les rapports Azure AD.

Sur la page **Utilisateurs** , vous obtenez une vue d’ensemble complète de toutes les connexions des utilisateurs en cliquant sur **Connexions** dans la section **Activité**.

![Capture d’écran montrant la section Activité dans laquelle vous pouvez sélectionner des connexions.](./media/concept-sign-ins/08.png "Activité de connexion")

## <a name="usage-of-managed-applications"></a>Utilisation des applications gérées

En disposant d’une vue centrée sur les applications de vos données de connexion, vous pouvez répondre aux questions telles que :

* Qui utilise mes applications ?
* Quelles sont les trois principales applications dans votre organisation ?
* Comment fonctionne mon application la plus récente ?

Le point d’entrée de ces données correspond aux trois principales applications de votre organisation. Les données sont contenues dans le rapport sur les 30 derniers jours dans la section **Vue d’ensemble** sous **Applications d’entreprise**.

![Capture d’écran montrant où vous pouvez sélectionner Vue d’ensemble.](./media/concept-sign-ins/10.png "Activité de connexion")

Les graphiques d’utilisation des applications affiche les agrégations hebdomadaires des connexions pour vos trois principales applications au cours d’une période donnée. La valeur par défaut de cette période est de 30 jours.

![Capture d’écran montrant l’utilisation de l’application pendant une période d’un mois.](./media/concept-sign-ins/graph-chart.png "Activité de connexion")

Si vous le souhaitez, vous pouvez définir la focalisation sur une application spécifique.

![Création de rapports](./media/concept-sign-ins/single-app-usage-graph.png "Signalement")

Lorsque vous cliquez sur un jour dans le graphique d’utilisation des applications, vous obtenez une liste détaillée des activités de connexion.

L’option **Connexions** vous fournit une vue d’ensemble complète de tous les événements de connexion à vos applications.

## <a name="microsoft-365-activity-logs"></a>Journaux d’activité Microsoft 365

Vous pouvez consulter les journaux d’activité Microsoft 365 dans le [centre d’administration Microsoft 365](/office365/admin/admin-overview/about-the-admin-center). N’oubliez pas que les journaux d’activité de Microsoft 365 et d’Azure AD partagent un nombre important de ressources de répertoire. Seul le centre d’administration Microsoft 365 fournit une vue complète des journaux d’activité d’Microsoft 365. 

Vous pouvez également accéder par programme aux journaux d’activité de Microsoft 365 en utilisant les [API de gestion Microsoft 365](/office/office-365-management-api/office-365-management-apis-overview).

## <a name="next-steps"></a>Étapes suivantes

* [Codes d’erreur du rapport d’activité de connexion](reference-sign-ins-error-codes.md)
* [Stratégies de conservation des données Azure AD](reference-reports-data-retention.md)
* [Latences de rapport Azure AD](reference-reports-latencies.md)
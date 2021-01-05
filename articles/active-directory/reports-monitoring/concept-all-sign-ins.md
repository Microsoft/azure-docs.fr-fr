---
title: Rapports d’activité de connexion Azure Active Directory – Préversion | Microsoft Docs
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
ms.date: 09/23/2020
ms.author: markvi
ms.reviewer: besiler
ms.collection: M365-identity-device-management
ms.openlocfilehash: 36e6642e230fa809630751e224ff9384ea8524d1
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92319779"
---
# <a name="azure-active-directory-sign-in-activity-reports---preview"></a>Rapports d’activité de connexion Azure Active Directory – Préversion

L’architecture de création de rapports dans Azure Active Directory (Azure AD) comprend les composants suivants :

- **Activité** 
    - **Connexions** : informations concernant les moments auxquels des utilisateurs, des applications et des ressources managées se connectent à Azure AD et accèdent à des ressources.
    - **Journaux d’audit** - [Journaux d’audit](concept-audit-logs.md) : fournit des informations sur les activités du système liées aux utilisateurs et à la gestion des groupes, les applications gérées et les activités de répertoire.
- **Sécurité** 
    - **Connexions risquées** : une [connexion risquée](../identity-protection/overview-identity-protection.md) correspond à un indicateur de tentative de connexion d’un utilisateur autre que le propriétaire légitime d’un compte d’utilisateur.
    - **Utilisateurs avec indicateur de risque** : un [utilisateur à risque](../identity-protection/overview-identity-protection.md) correspond à un indicateur de compte d’utilisateur susceptible d’être compromis.

Le rapport classique sur les connexions dans Azure Active Directory fournit une vue d’ensemble des connexions utilisateur interactives. En plus de celui-ci, vous avez désormais accès à trois rapports de connexion supplémentaires actuellement en préversion :

- Connexions utilisateur non interactives

- Connexions de principal de service

- Identités managées pour les connexions aux ressources Azure

Cet article présente une vue d’ensemble du rapport d’activité de connexion avec la préversion des connexions non interactives, d’applications et d’identités managées pour les ressources Azure. Pour plus d’informations sur le rapport de connexion sans les fonctionnalités en version préliminaire, consultez [Rapports d’activité de connexion dans le portail Azure Active Directory](concept-sign-ins.md).



## <a name="prerequisites"></a>Prérequis

Avant de commencer à utiliser cette fonctionnalité, vous devez connaître les réponses aux questions suivantes :

- Qui peut accéder aux données ?

- De quelle licence Azure AD avez-vous besoin pour accéder à l’activité de connexion ?

### <a name="who-can-access-the-data"></a>Qui peut accéder aux données ?

- Les utilisateurs ayant le rôle Administrateur de sécurité, Lecteur Sécurité et Lecteur de rapports

- Les administrateurs généraux

- Tous les utilisateurs (non administrateurs) peuvent accéder à leurs propres connexions 

### <a name="what-azure-ad-license-do-you-need-to-access-sign-in-activity"></a>De quelle licence Azure AD avez-vous besoin pour accéder à l’activité de connexion ?

Votre client doit avoir une licence Azure AD Premium associée pour afficher les activités de connexion. Consultez [Bien démarrer avec Azure Active Directory Premium](../fundamentals/active-directory-get-started-premium.md) pour mettre à niveau votre édition d’Azure Active Directory. Quelques jours seront nécessaires pour que les données s’affichent dans les rapports après la mise à niveau vers une licence premium s’il n’y a aucune activité de données avant la mise à niveau.



## <a name="sign-ins-report"></a>Rapport de connexions

Le rapport de connexions contient des réponses aux questions suivantes :

- Quel est le modèle de connexion d’un utilisateur, d’une application ou d’un service ?
- Combien d’utilisateurs, d’applications ou de services se sont-ils connectés au cours d’une semaine ?
- Quel est l’état de ces connexions ?


Dans le panneau du rapport de connexions, vous pouvez basculer entre les deux affichages suivants :

- **Connexions utilisateur interactives** – Connexions où un utilisateur fournit un facteur d’authentification, tel qu’un mot de passe, une réponse via une application d’authentification multifacteur, un facteur biométrique ou un code QR.

- **Connexions utilisateur non interactives** – Connexions établies par un client pour le compte d’un utilisateur. Ces connexions ne nécessitent ni interaction, ni facteur d’authentification de la part de l’utilisateur. L’authentification et l’autorisation se font à l’aide de jetons d’accès et d’actualisation ne nécessitent pas que l’utilisateur entre des informations d’identification.

- **Connexions de principal de service** – Connexions établies par des principaux d’applications et de service n’impliquant aucune intervention de l’utilisateur. Dans ces connexions, l’application ou le service fournit des informations d’identification pour son propre compte afin de s’authentifier ou d’accéder à des ressources.

- **Connexions avec identités managées pour ressources Azure** – Connexions établies par des ressources Azure dont les secrets sont gérés par Azure. Pour plus d’informations, consultez [Que sont les identités managées pour les ressources Azure ?](../managed-identities-azure-resources/overview.md) 


![Types de rapports de connexions](./media/concept-all-sign-ins/sign-ins-report-types.png)












## <a name="user-sign-ins"></a>Connexions utilisateur

Chaque onglet du panneau des connexions affiche les colonnes par défaut ci-dessous. Certains onglets contiennent des colonnes supplémentaires :

- Date de la connexion

- ID de la demande

- Nom ou ID de l’utilisateur

- Nom ou ID de l’application

- État de la connexion

- Adresse IP de l’appareil utilisé pour la connexion



### <a name="interactive-user-sign-ins"></a>Connexions utilisateur interactives


Les connexions utilisateur interactives sont des connexions où un utilisateur fournit un facteur d’authentification à Azure AD ou interagit directement avec Azure AD ou une application auxiliaire telle que l’application Microsoft Authenticator. Les facteurs que les utilisateurs fournissent incluent des mots de passe, des réponses à des demandes d’authentification multifacteur, des facteurs biométriques ou des codes QR qu’un utilisateur fournit à Azure AD ou à une application auxiliaire.

Ce rapport comprend également des connexions fédérées de fournisseurs d’identité qui sont fédérés à Azure AD.  


**Taille du rapport :** petite <br> 
**Exemples :**

- Un utilisateur fournit un nom d’utilisateur et un mot de passe dans l’écran de connexion à Azure AD.

- Un utilisateur transmet une réponse reçue par SMS à une demande d’authentification multifacteur.

- Un utilisateur effectue un geste d’identification biométrique pour déverrouiller son PC Windows avec Windows Hello Entreprise.

- Un utilisateur est fédéré à Azure AD avec une assertion SAML AD FS.


En plus des champs par défaut, le rapport de connexions interactives affiche les informations suivantes : 

- Emplacement de la connexion

- Application ou non de l’accès conditionnel



Vous pouvez personnaliser le mode Liste en cliquant sur **Colonnes** dans la barre d’outils.

![Colonnes de connexion utilisateur interactive](./media/concept-all-sign-ins/columns-interactive.png "Colonnes de connexion utilisateur interactive")





La personnalisation de la vue vous permet d’afficher des champs supplémentaires ou de supprimer des champs affichés.

![Toutes les colonnes interactives](./media/concept-all-sign-ins/all-interactive-columns.png)


Sélectionnez un élément dans la liste pour obtenir des informations plus détaillées sur la connexion associée.

![Activité de connexion](./media/concept-all-sign-ins/interactive-user-sign-in-details.png "Connexions utilisateur interactives")



### <a name="non-interactive-user-sign-ins"></a>Connexions utilisateur non interactives

Les connexions utilisateur non interactives sont des connexions établies par une application cliente ou des composants du système d’exploitation pour le compte d’un utilisateur. À l’instar des connexions utilisateur interactives, ces connexions sont établies pour le compte d’un utilisateur. Contrairement aux connexions utilisateur interactives, ces connexions ne nécessitent pas que l’utilisateur fournisse un facteur d’authentification. Au lieu de cela, l’appareil ou l’application cliente utilisent un jeton ou du code pour s’authentifier ou accéder à une ressource pour le compte d’un utilisateur. En général, l’utilisateur perçoit ces connexions comme se produisant en arrière-plan de son activité.


**Taille du rapport :** grand <br>
**Exemples :** 

- Une application cliente utilise un jeton d’actualisation OAuth 2.0 pour obtenir un jeton d’accès.

- Un client utilise un code d’autorisation OAuth 2.0 pour obtenir un jeton d’accès et un jeton d’actualisation.

- Un utilisateur effectue une authentification unique (SSO) auprès d’une application web ou Windows sur un PC joint Azure AD.

- Un utilisateur se connecte à une deuxième application Microsoft Office tandis qu’il a une session ouverte sur un appareil mobile à l’aide d’une FOCI (famille d’ID client).




En plus des champs par défaut, le rapport de connexions non interactives affiche les informations suivantes : 

- ID de ressource

- Nombre de connexions regroupées




Vous ne pouvez pas personnaliser les champs affichés dans ce rapport.


![Colonnes désactivées](./media/concept-all-sign-ins/disabled-columns.png "Colonnes désactivées")

Pour faciliter la synthèse des données, les événements de connexion non interactive sont regroupés. Les clients créent souvent de nombreuses connexions non interactives pour le compte d’un même utilisateur sur une brève période, qui partagent les mêmes caractéristiques, à l’exception de l’heure à laquelle la connexion a été tentée. Par exemple, un client peut obtenir un jeton d’accès une fois par heure pour le compte d’un utilisateur. Si l’utilisateur ou le client ne changent pas d’état, l’adresse IP, la ressource et toutes les autres informations sont les mêmes pour chaque demande de jeton d’accès. Quand Azure AD journalise plusieurs connexions identiques à l’exception de la date et de l’heure, ces connexions en provenance de la même entité sont agrégées dans une seule ligne. Une ligne avec plusieurs connexions identiques (à l’exception de la date et de l’heure) a une valeur supérieure à 1 dans la colonne du nombre de connexions. Vous pouvez développer la ligne pour voir les différentes connexions et leurs horodatages. Les connexions sont agrégées dans les utilisateurs non interactifs lorsque les données suivantes correspondent :


- Application

- Utilisateur

- Adresse IP

- Statut

- ID de ressource


Vous pouvez :

- développer un nœud pour afficher les éléments d’un groupe ;  

- cliquer sur un élément pour afficher tous ses détails. 


![Détails de connexion utilisateur non interactive](./media/concept-all-sign-ins/non-interactive-sign-ins-details.png)




## <a name="service-principal-sign-ins"></a>Connexions de principal de service

Contrairement aux connexions utilisateur interactives et non interactives, les connexions de principal de service n’impliquent pas d’utilisateur. Au lieu de cela, il s’agit de connexions établies par n’importe quel compte non-utilisateur, par exemple, des applications ou des principaux de service (à l’exception des connexions d’identités managées qui sont incluses uniquement dans le rapport homonyme). Dans le cadre de ces connexions, l’application ou le service fournit ses propres informations d’identification, telles qu’un certificat ou un secret d’application, pour s’authentifier ou accéder à des ressources.


**Taille du rapport :** grand <br>
**Exemples :**

- Un principal de service utilise un certificat pour s’authentifier et accéder au Microsoft Graph. 

- Une application utilise une clé secrète client pour s’authentifier dans le flux des informations d’identification de client OAuth. 


Ce rapport contient une liste par défaut qui affiche les éléments suivants :

- Date de la connexion

- ID de la demande

- Nom ou ID de principal de service

- Statut

- Adresse IP

- Nom de la ressource

- ID de ressource

- Nombre de connexions

Vous ne pouvez pas personnaliser les champs affichés dans ce rapport.

![Colonnes désactivées](./media/concept-all-sign-ins/disabled-columns.png "Colonnes désactivées")

Pour faciliter la synthèse des données dans les journaux de connexion du principal de service, les événements de connexion de celui-ci sont regroupés. Les connexions à partir de la même entité dans les mêmes conditions sont agrégées dans une seule ligne. Vous pouvez développer la ligne pour voir les différentes connexions et leurs horodatages. Les connexions sont agrégées dans le rapport du principal de service lorsque les données suivantes correspondent :

- Nom ou ID de principal de service

- Statut

- Adresse IP

- Nom ou ID de la ressource

Vous pouvez :

- développer un nœud pour afficher les éléments d’un groupe ;  

- cliquer sur un élément pour afficher tous ses détails. 


![Détails des colonnes](./media/concept-all-sign-ins/service-principals-sign-ins-view.png "Détails de colonne")




## <a name="managed-identity-for-azure-resources-sign-ins"></a>Connexions avec identité managée pour ressources Azure 

Les connexions avec identité managée pour ressources Azure sont des connexions établies par des ressources dont les secrets sont gérés par Azure pour simplifier la gestion des informations d’identification.

**Taille du rapport :** Petite <br> 
**Exemples :**

Une machine virtuelle avec des informations d’identification managées utilise Azure AD pour obtenir un jeton d’accès.   


Ce rapport contient une liste par défaut qui affiche les éléments suivants :


- ID de l’identité managée

- Nom de l’identité managée

- Ressource

- ID de ressource

- Nombre de connexions regroupées

Vous ne pouvez pas personnaliser les champs affichés dans ce rapport.

Pour faciliter la synthèse des données, les journaux de connexion avec identités managées pour ressources Azure et les événements de connexion non interactifs sont regroupés. Les connexions à partir de la même entité sont agrégées dans une seule ligne. Vous pouvez développer la ligne pour voir les différentes connexions et leurs horodatages. Les connexions sont agrégées dans le rapport d’identités managées quand toutes les données suivantes correspondent :

- Nom ou ID de l’identité managée

- Statut

- Adresse IP

- Nom ou ID de la ressource

Sélectionnez un élément dans la liste pour afficher toutes les connexions regroupées sous un nœud.

Sélectionnez un élément regroupé pour afficher tous les détails de la connexion. 


## <a name="filter-sign-in-activities"></a>Filtrer les activités de connexion

En définissant un filtre, vous pouvez réduire l’étendue des données de connexion retournées. Azure AD vous offre une large gamme de filtres supplémentaires que vous pouvez définir. Lorsque vous définissez votre filtre, vous devez toujours prêter une attention particulière à votre filtre de **plage de dates** de configuré. Un filtre de plage de dates approprié garantit qu’Azure AD retourne uniquement les données qui vous intéressent vraiment.     

Le filtre de **plage de dates** vous permet de définir un intervalle de temps pour les données renvoyées.
Les valeurs possibles sont les suivantes :

- Un mois

- Sept jours

- Vingt-quatre heures

- Custom

![Filtre de plage de dates](./media/concept-all-sign-ins/date-range-filter.png)





### <a name="filter-user-sign-ins"></a>Filtrer les connexions utilisateur

Le filtre est le même pour les des connexions interactives et non interactives. C’est pourquoi le filtre que vous avez configuré pour les connexions interactives est conservé pour les connexions non interactives, et inversement. 






## <a name="access-the-new-sign-in-activity-reports"></a>Accéder aux nouveaux rapports d’activité de connexion 

Le rapport d’activité de connexion dans le Portail Azure offre une méthode simple pour activer et désactiver le rapport en préversion. Si les rapports en préversion sont activés, un nouveau menu s’affiche, qui vous permet d’accéder à tous les types de rapports d’activité de connexion.     


Pour accéder aux nouveaux rapports de connexion avec des connexions d’application non interactives : 

1. Dans le [portail Azure](https://portal.azure.com), sélectionnez **Azure Active Directory**.

    ![Sélectionner Azure AD](./media/concept-all-sign-ins/azure-services.png)

2. Dans la section **Surveillance**, cliquez **Connexions**.

    ![Sélectionner Connexions](./media/concept-all-sign-ins/sign-ins.png)

3. Cliquez sur la barre de **Préversion**.

    ![Activer le nouvel affichage](./media/concept-all-sign-ins/enable-new-preview.png)

4. Pour revenir à l’affichage par défaut, cliquez de nouveau sur la barre **Préversion**. 

    ![Restaurer l’affichage classique](./media/concept-all-sign-ins/switch-back.png)







## <a name="download-sign-in-activity-reports"></a>Télécharger les rapports d’activité de connexion

Lorsque vous téléchargez un rapport d’activité de connexion, les conditions suivantes sont réunies :

- Vous pouvez télécharger le rapport de connexion en tant que fichier CSV ou JSON.

- Vous pouvez télécharger jusqu’à 100 000 enregistrements. Si vous souhaitez télécharger davantage de données, utilisez l’API de création de rapports.

- Votre téléchargement est basé sur la sélection de filtre que vous avez effectuée.

- Le nombre d’enregistrements que vous pouvez télécharger est limité par les [stratégies de rétention de rapport Azure Active Directory](reference-reports-data-retention.md). 


![Télécharger des rapports](./media/concept-all-sign-ins/download-reports.png "Rapports de téléchargement")


Chaque téléchargement au format CSV se compose de six fichiers différents :

- Connexions interactives

- Détails d’authentification des connexions interactives

- Connexions non interactives

- Détails d’authentification des connexions non interactives

- Connexions de principal de service

- Connexions avec identité managée pour ressources Azure

Chaque téléchargement au format JSON se compose de quatre fichiers différents :

- Connexions interactives (inclut des détails d’authentification)

- Connexions non interactives (inclut des détails d’authentification)

- Connexions de principal de service

- Connexions avec identité managée pour ressources Azure

![Télécharger des fichiers](./media/concept-all-sign-ins/download-files.png "Télécharger des fichiers")




## <a name="next-steps"></a>Étapes suivantes

* [Codes d’erreur du rapport d’activité de connexion](reference-sign-ins-error-codes.md)
* [Stratégies de conservation des données Azure AD](reference-reports-data-retention.md)
* [Latences de rapport Azure AD](reference-reports-latencies.md)
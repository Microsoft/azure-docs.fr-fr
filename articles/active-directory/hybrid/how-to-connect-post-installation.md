---
title: 'Azure AD Connect : Étapes suivantes et gestion Azure AD Connect | Microsoft Docs'
description: Apprenez à étendre la configuration par défaut et les tâches opérationnelles pour Azure AD Connect.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: curtand
ms.assetid: c18bee36-aebf-4281-b8fc-3fe14116f1a5
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 04/26/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 7d6689b4b67b8462e983ae9b111e0fbc60c422b7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89657120"
---
# <a name="next-steps-and-how-to-manage-azure-ad-connect"></a>Étapes suivantes et gestion d’Azure AD Connect
Utilisez les procédures opérationnelles dans cet article pour personnaliser Azure Active Directory (Azure AD) Connect pour l’adapter aux besoins et aux spécifications de votre organisation.  

## <a name="add-additional-sync-admins"></a>Ajout d’administrateurs de synchronisation supplémentaires
Par défaut, seul l’utilisateur qui a effectué l’installation et les administrateurs locaux est en mesure de gérer le moteur de synchronisation installé. Pour que d’autres personnes puissent accéder au moteur de synchronisation et en assurer la gestion, localisez le groupe nommé ADSyncAdmins sur le serveur local et ajoutez ces personnes à ce groupe.

## <a name="assign-licenses-to-azure-ad-premium-and-enterprise-mobility-suite-users"></a>Attribution des licences aux utilisateurs Azure AD Premium et Enterprise Mobility Suite
Maintenant que vos utilisateurs ont été synchronisés avec le cloud, vous devez leur attribuer une licence afin qu'ils puissent utiliser des applications cloud telles que Microsoft 365.

### <a name="to-assign-an-azure-ad-premium-or-enterprise-mobility-suite-license"></a>Pour l’attribution d’une licence Azure AD Premium ou Enterprise Mobility Suite

1. Connectez-vous au portail Azure en tant qu’administrateur.
2. Sélectionnez **Active Directory**à gauche.
3. Sur la page **Active Directory**, double-cliquez sur le répertoire qui contient les utilisateurs que vous souhaitez configurer.
4. En haut de la page du répertoire, sélectionnez **Licences**.
5. Sur la page **Licences**, sélectionnez **Active Directory Premium** ou **Enterprise Mobility Suite**, puis cliquez sur **Attribuer**.
6. Dans la boîte de dialogue, sélectionnez les utilisateurs auxquels vous souhaitez attribuer des licences, puis cliquez sur l'icône de coche pour enregistrer les modifications.

## <a name="verify-the-scheduled-synchronization-task"></a>Vérification de la tâche de synchronisation planifiée
Utilisez le portail Azure pour vérifier l’état d’une synchronisation.

### <a name="to-verify-the-scheduled-synchronization-task"></a>Pour la vérification de la tâche de synchronisation planifiée
1. Connectez-vous au portail Azure en tant qu’administrateur.
2. Sélectionnez **Active Directory**à gauche.
3. Sélectionnez **Azure AD Connect** à gauche
4. En haut de la page, notez la dernière synchronisation.

![Heure de synchronisation d’annuaires](./media/how-to-connect-post-installation/verify2.png)

## <a name="start-a-scheduled-synchronization-task"></a>Lancement d’une tâche de synchronisation planifiée
Si vous devez exécuter une tâche de synchronisation, effectuez les opérations suivantes :

1. Double-cliquez sur le raccourci Bureau Azure AD Connect pour démarrer l’Assistant.
2. Cliquez sur **Configurer**.
3. Dans l’écran des tâches, sélectionnez **Personnaliser les options de synchronisation**, puis cliquez sur **Suivant**
4. Entrez vos informations d’identification Azure AD.
5. Cliquez sur **Suivant**. Cliquez sur **Suivant**.  Cliquez sur **Suivant**.
5.  Dans l’écran **Prêt à configurer**, vérifiez que la case **Démarrez le processus de synchronisation une fois la configuration terminée** est cochée.
6.  Cliquez sur **Configurer**.

Pour plus d’informations sur le planificateur de synchronisation Azure AD Connect, consultez [Planificateur Azure AD Connect](how-to-connect-sync-feature-scheduler.md).

## <a name="additional-tasks-available-in-azure-ad-connect"></a>Tâches supplémentaires disponibles dans Azure AD Connect
Après l’installation initiale d’Azure AD Connect, vous pouvez toujours redémarrer l’Assistant depuis la page de démarrage ou le raccourci du bureau Azure AD Connect.  Vous remarquerez qu’en passant à nouveau par l’Assistant de nouvelles options apparaissent sous forme de tâches supplémentaires.  

Le tableau suivant fournit un résumé de ces tâches et une brève description de chacune d’elles.

![Liste des tâches supplémentaires](./media/how-to-connect-post-installation/addtasks2.png)

| Tâche supplémentaire | Description |
| --- | --- |
|**Paramètres de confidentialité**|Afficher les données de télémétrie qui sont partagées avec Microsoft.|
|**Afficher la configuration actuelle**|Afficher votre solution Azure AD Connect actuelle.  Cela inclut les paramètres généraux, les répertoires synchronisés et les paramètres de synchronisation. |
| **Personnalisation des options de synchronisation** |Modifier la configuration actuelle, notamment par l’ajout de forêts Active Directory supplémentaires à la configuration ou l’activation d’options de synchronisation telles que l’utilisateur, le groupe, l’appareil ou le mot de passe en écriture différée. |
|**Tâche Configurer les options de l’appareil**|Options de l’appareil disponibles pour la synchronisation|
|**Actualiser le schéma d’annuaire**|Vous permet d’ajouter de nouveaux objets d’annuaire en local pour la synchronisation|
|**Configurer le mode de préproduction** |Réalisez un déploiement intermédiaire des informations qui ne sont pas immédiatement synchronisées et ne sont pas exportées vers Azure AD ou l’Active Directory local.  Cette fonctionnalité vous permet d’afficher un aperçu des synchronisations avant qu’elles ne s’effectuent. |
|**Modifier la connexion de l’utilisateur**|Modifier la méthode d’authentification que les utilisateurs emploient pour se connecter|
|**Gérer la fédération**|Gérer votre infrastructure AD FS, renouveler les certificats et ajouter des serveurs AD FS|
|**Résolution des problèmes**|Aide pour la résolution des problèmes liés à Azure AD Connect|

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur [l’intégration de vos identités locales avec Azure Active Directory](whatis-hybrid-identity.md).

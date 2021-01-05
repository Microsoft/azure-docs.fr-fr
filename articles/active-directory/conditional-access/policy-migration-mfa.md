---
title: Migrer les stratégies d’accès conditionnel avec l’authentification multifacteur - Azure Active Directory
description: Cet article montre comment migrer une stratégie classique qui nécessite l’authentification multifacteur dans le portail Azure.
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: how-to
ms.date: 05/26/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: nigu
ms.collection: M365-identity-device-management
ms.openlocfilehash: 5cc99241f5fa7c09c69db13dcb3f6c3aaa75e376
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87027391"
---
# <a name="migrate-a-classic-policy-in-the-azure-portal"></a>Migrer une stratégie classique dans le Portail Azure

Cet article montre comment migrer une stratégie classique qui nécessite l’**authentification multifacteur** pour une application cloud. Bien qu’il ne s’agisse pas d’une prérequis, nous vous recommandons de lire [Migrer les stratégies classiques dans le portail Azure](policy-migration.md) avant d’effectuer la migration de vos stratégies classiques.

![Détails de la stratégie classique nécessitant l’authentification multifacteur pour l’application Salesforce](./media/policy-migration/33.png)

Le processus de migration est constitué des étapes suivantes :

1. [Ouvrez la stratégie classique](#open-a-classic-policy) pour obtenir les paramètres de configuration.
1. Créez une nouvelle stratégie d’accès conditionnel Azure AD afin de remplacer votre stratégie classique. 
1. Désactivez la stratégie classique.

## <a name="open-a-classic-policy"></a>Ouvrir une stratégie classique

1. Dans le [Portail Azure](https://portal.azure.com), accédez à **Azure Active Directory** > **Sécurité** > **Accès conditionnel**.
1. Sélectionnez **Stratégies classiques**.

   ![Affichage Stratégies classiques](./media/policy-migration-mfa/12.png)

1. Dans la liste des stratégies classiques, sélectionnez celle que vous souhaitez migrer. Documentez les paramètres de configuration afin de pouvoir les recréer dans une nouvelle stratégie d’accès conditionnel.

## <a name="create-a-new-conditional-access-policy"></a>Créer une nouvelle stratégie d’accès conditionnel

1. Dans le [Portail Azure](https://portal.azure.com), accédez à **Azure Active Directory** > **Sécurité** > **Accès conditionnel**.
1. Pour créer une stratégie d’accès conditionnel, sélectionnez **Nouvelle stratégie**.
1. Dans la page **Nom** du panneau **Nouveau**, indiquez le nom de votre stratégie.
1. Dans la section **Affectations**, cliquez sur **Utilisateurs et groupes**.
   1. Si vous avez tous les utilisateurs sélectionnés dans votre stratégie classique, cliquez sur **Tous les utilisateurs**. 
   1. Si vous avez des groupes sélectionnés dans votre stratégie classique, cliquez sur **Sélectionner des utilisateurs et des groupes**, puis sélectionnez les groupes et utilisateurs requis.
   1. Si vous avez des groupes exclus, cliquez sur l’onglet **Exclure** et sélectionnez les groupes et utilisateurs requis. 
   1. Sélectionnez **Terminé**
1. Dans la section **Affectations**, cliquez sur **Applications ou actions cloud**.
1. Dans la page **Applications ou actions cloud**, procédez comme suit :
   1. Cliquez sur **Sélectionner les applications**.
   1. Cliquez sur **Sélectionner**.
   1. Dans la page **Sélectionner**, sélectionnez votre application cloud, puis cliquez sur **Sélectionner**.
   1. Dans la page **Applications cloud**, cliquez sur **Terminé**.
1. Si vous avez sélectionné **Imposer l’authentification multifacteur** :
   1. Dans la section **Contrôles d’accès**, cliquez sur **Accorder**.
   1. Sur la page **Accorder**, cliquez sur **Accorder l’accès**, puis cliquez sur **Imposer l’authentification multifacteur**.
   1. Cliquez sur **Sélectionner**.
1. Cliquez sur **Activé** pour activer votre stratégie, puis sélectionnez **Enregistrer**.

   ![Création d’une stratégie d’accès conditionnel](./media/policy-migration-mfa/conditional-access-policy-migration.png)

## <a name="disable-the-classic-policy"></a>Désactiver la stratégie classique

Pour désactiver votre stratégie classique, cliquez sur **Désactiver** dans la vue **Détails**.

![Désactiver des stratégies classiques](./media/policy-migration-mfa/14.png)

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur la migration des stratégies classiques, consultez [Migrer les stratégies classiques dans le portail Azure](policy-migration.md).
- [Utilisez le mode Rapport uniquement pour l’accès conditionnel afin de déterminer l’impact des nouvelles décisions de stratégie.](concept-conditional-access-report-only.md)
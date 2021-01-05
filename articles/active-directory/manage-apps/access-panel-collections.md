---
title: Créer des collections pour les portails Mes applications dans Azure Active Directory | Microsoft Docs
description: Utilisez des collections Mes applications pour personnaliser les pages Mes applications afin de simplifier l’expérience Mes applications pour vos utilisateurs finaux. Organisez les applications en groupes avec des onglets distincts.
services: active-directory
documentationcenter: ''
author: kenwith
manager: celestedg
ms.assetid: ''
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: how-to
ms.date: 02/10/2020
ms.author: kenwith
ms.reviewer: kasimpso
ms.collection: M365-identity-device-management
ms.openlocfilehash: 967ac34ddb736f58f66639181cdf10e4aaa8ffab
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94656783"
---
# <a name="create-collections-on-the-my-apps-portal"></a>Créer des collections sur le portail Mes applications

Vos utilisateurs peuvent utiliser le portail Mes applications pour afficher et démarrer les applications cloud auxquelles ils ont accès. Par défaut, toutes les applications auxquelles un utilisateur peut accéder sont répertoriées sur une seule page. Pour mieux organiser cette page pour vos utilisateurs, si vous disposez d’une licence Azure AD Premium P1 ou P2, vous pouvez configurer des collections. Avec une collection, vous pouvez regrouper des applications associées (par exemple, par rôle de travail, tâche ou projet) et les afficher sous un onglet distinct. Une collection applique essentiellement un filtre aux applications auxquelles un utilisateur a déjà accès, de sorte que celui-ci ne voit que les applications de la collection qui lui ont été attribuées.

> [!NOTE]
> Cet article explique comment un administrateur peut activer et créer des collections. Pour plus d’informations sur l’utilisation du portail et des collections Mes applications à destination des utilisateurs finaux, voir [Accéder aux collections et les utiliser](../user-help/my-applications-portal-workspaces.md).

## <a name="enable-the-latest-my-apps-features"></a>Activer les dernières fonctionnalités de Mes applications

1. Ouvrez le [**portail Azure**](https://portal.azure.com/) et connectez-vous en tant qu’administrateur général ou administrateur utilisateur.

2. Accédez à **Azure Active Directory** > **Paramètres utilisateur**.

3. Sous **Préversions des fonctionnalités utilisateur**, sélectionnez **Gérer les paramètres de préversion de la fonctionnalité utilisateur**.

4. Sous **Les utilisateurs peuvent utiliser les fonctionnalités en préversion pour Mes applications**, choisissez une des options suivantes :
   * **Sélectionné** : active les fonctionnalités pour un groupe spécifique. Utilisez l’option **Sélectionner un groupe** pour sélectionner le groupe pour lequel vous souhaitez activer les fonctionnalités.  
   * **Tous** : active les fonctionnalités pour tous les utilisateurs.

> [!NOTE]
> Pour ouvrir le portail Mes applications, les utilisateurs peuvent utiliser le lien `https://myapps.microsoft.com` ou le lien personnalisé pour votre organisation, par exemple `https://myapps.microsoft.com/contoso.com`. Une fois la nouvelle expérience Mes applications activée, la bannière indiquant **Une expérience Mes applications mise à jour est disponible** s’affiche en haut de la page Mes applications, et les utilisateurs peuvent sélectionner **Essayer** pour afficher la nouvelle expérience. Pour ne plus utiliser la nouvelle expérience, les utilisateurs peuvent sélectionner **Oui** dans la bannière **Quitter la nouvelle expérience** en haut de la page.

## <a name="create-a-collection"></a>Création d'une collection

Pour créer une collection, vous devez disposer d’une licence Azure AD Premium P1 ou P2.

1. Ouvrez le [**portail Azure**](https://portal.azure.com/) et connectez-vous en tant qu’administrateur avec une licence Azure AD Premium P1 ou P2.

2. Accédez à **Azure Active Directory** > **Enterprise applications (Applications d’entreprise)** .

3. Sous **Gérer**, sélectionnez **Collections**.

4. Sélectionnez **Nouvelle collection**. Dans la page **nouvelle collection**, entrez un **Nom** pour la collection (nous vous recommandons de ne pas utiliser le mot « collection » dans le nom). Saisissez ensuite une **Description**.

   ![Page Nouvelle collection](media/acces-panel-collections/new-collection.png)

5. Sélectionnez l’onglet **Applications** . Sélectionnez **+ Ajouter une application**, puis, dans la page **Ajouter des applications**, sélectionnez toutes les applications que vous souhaitez ajouter à la collection ou utilisez la zone **Recherche** pour rechercher des applications.

   ![Ajouter une application à la collection](media/acces-panel-collections/add-applications.png)

6. Lorsque vous avez fini d’ajouter des applications, sélectionnez **Ajouter**. La liste des applications sélectionnées s’affiche. Vous pouvez utiliser la flèche Haut pour modifier l’ordre des applications dans la liste. Pour déplacer une application vers le bas ou la supprimer de la collection, sélectionnez le menu **Autres** ( **...** ).

7. Sélectionnez l’onglet **Propriétaires** . Sélectionnez **+ Ajouter des utilisateurs et des groupes**, puis, dans la page **Ajouter des utilisateurs et des groupes**, sélectionnez les utilisateurs ou les groupes auxquels vous souhaitez attribuer la propriété. Une fois que vous avez fini de sélectionner les utilisateurs et groupes, choisissez **Sélectionner**.

9. Sélectionnez l’onglet **Utilisateurs et groupes**. Sélectionnez **+ Ajouter des utilisateurs et des groupes**, puis, dans la page **Ajouter des utilisateurs et des groupes**, sélectionnez les utilisateurs ou les groupes auxquels vous souhaitez attribuer la collection. Ou utilisez la zone **Rechercher** pour rechercher des utilisateurs ou des groupes. Une fois que vous avez fini de sélectionner les utilisateurs et groupes, choisissez **Sélectionner**.

   ![Ajoutez des utilisateurs et des groupes](media/acces-panel-collections/add-users-and-groups.png)

11. Sélectionnez **Vérifier + créer**. Les propriétés de la nouvelle collection s’affichent.


## <a name="view-audit-logs"></a>Afficher les journaux d’audit

Les journaux d’audit enregistrent les opérations de collection de Mes applications, y compris les actions de création de collection effectuées par l’utilisateur final. Les événements suivants sont générés pour ces actions à partir de Mes applications :

* Créer une collection
* Modifier une collection
* Supprimer une collection
* Lancer une application (utilisateur final)
* Ajouter des applications en libre-service (utilisateur final)
* Supprimer une application libre-service (utilisateur final)

Vous pouvez accéder aux journaux d’audit dans le [portail Azure](https://portal.azure.com) en sélectionnant **Azure Active Directory** > **Applications d’entreprise**  > **Journaux d’audit** dans la section Activité. Pour **Service**, sélectionnez **Mes applications**.

## <a name="get-support-for-my-account-pages"></a>Obtenir de l’aide pour les pages Mon compte

À partir de la page Mes applications, un utilisateur peut sélectionner **Mon compte** > **Afficher mon compte** pour ouvrir ses paramètres de compte. Sur la page **Mon compte** Azure AD, les utilisateurs peuvent gérer leurs informations de sécurité, appareils, mots de passe, etc. Ils peuvent également accéder à leurs paramètres de compte Office.

Si vous devez envoyer une demande de support pour un problème lié à la page du compte Azure AD ou à la page du compte Office, procédez comme suit afin de permettre son bon acheminement : 

* Pour les problèmes liés à la page **« Mon compte » Azure AD**, ouvrez une demande de support depuis le portail Azure. Accédez à **Portail Azure** > **Azure Active Directory** > **Nouvelle demande de support**.

* Pour les problèmes liés à la page **« Mon compte » Office**, ouvrez une demande de support depuis le Centre d’administration Microsoft 365. Accédez au **Centre d'administration Microsoft 365** > **Support**. 

## <a name="next-steps"></a>Étapes suivantes
[Expériences d’utilisateurs finaux auprès des applications dans Azure Active Directory](end-user-experiences.md)
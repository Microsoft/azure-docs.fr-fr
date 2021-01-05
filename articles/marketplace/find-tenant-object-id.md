---
title: Rechercher l’ID du locataire, l’ID de l’objet et les détails de l’association de partenaires dans la place de marché Azure
description: Comment rechercher l’ID de locataire, l’ID d’objet et les détails d’association de partenaires d’un ID d’abonnement dans la place de marché Azure.
ms.service: marketplace
ms.topic: article
author: keferna
ms.author: keferna
ms.date: 10/09/2020
ms.openlocfilehash: db09943085cb7934bca5d7f2dc24ba692613ee19
ms.sourcegitcommit: 6906980890a8321dec78dd174e6a7eb5f5fcc029
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92426897"
---
# <a name="find-tenant-id-object-id-and-partner-association-details"></a>Rechercher l’ID du locataire, l’ID de l’objet et les détails de l’association de partenaires

Cet article explique comment rechercher l’ID de locataire, l’ID d’objet et les détails de l’association partenaire, ainsi que leurs ID d’abonnement respectifs.

Si vous avez besoin de captures d’écran de ces éléments dans Azure Cloud Shell pour les utiliser pour l’assistance au débogage, accédez à [Rechercher l’ID du locataire, de l’objet et du partenaire pour le débogage](#find-ids-for-debugging).

>[!Note]
> Seul le propriétaire d’un abonnement dispose des privilèges nécessaires pour effectuer ces étapes.

## <a name="find-tenant-id"></a>Rechercher l’ID de locataire

1. Accédez au [portail Azure](https://ms.portal.azure.com/).
2. Sélectionnez **Azure Active Directory**.

    :::image type="content" source="media/tenant-and-object-id/icon-azure-ad.png" alt-text="Icône d’Azure Active Directory dans le portail Azure.":::

3. Sélectionnez **Groupes**. Votre ID de locataire se trouve dans la boîte **Informations sur le locataire**.

    :::image type="content" source="media/tenant-and-object-id/select-groups-1.png" alt-text="Sélectionnez Groupes dans le portail Azure.":::

## <a name="find-subscriptions-and-roles"></a>Rechercher des abonnements et des rôles

1. Accédez au portail Azure et sélectionnez **Azure Active Directory** comme noté dans les étapes 1 et 2 qui précèdent.
2. Sélectionnez **Abonnements**.

    :::image type="content" source="media/tenant-and-object-id/icon-azure-subscriptions-1.png" alt-text="L’icône Abonnements dans le portail Azure.":::

3. Voir les abonnements et les rôles.

    :::image type="content" source="media/tenant-and-object-id/subscriptions-screen-1.png" alt-text="L’écran Abonnements dans le portail Azure.":::

## <a name="find-partner-id"></a>Rechercher l’ID partenaire

1. Accédez à la page Abonnements comme décrit dans la section précédente.
2. Sélectionnez un abonnement.
3. Sous **Facturation** , sélectionnez **Informations sur les partenaires**.

    :::image type="content" source="media/tenant-and-object-id/menu-partner-information.png" alt-text="Informations sur le partenaire dans le menu de navigation de gauche.":::

## <a name="find-user-object-id"></a>Rechercher un utilisateur (ID d’objet)

1. Connectez-vous au [portail d’administration Office 365](https://portal.office.com/adminportal/home) avec un compte dans le locataire souhaité et avec les droits d’administration appropriés.
2. Dans le menu de gauche, développez la section **Centres d’administration** en bas, puis sélectionnez l’option Azure Active Directory pour lancer la console d’administration dans une nouvelle fenêtre de navigateur.
3. Sélectionnez **Utilisateurs**.
4. Recherchez l’utilisateur souhaité, puis sélectionnez le nom du compte pour voir les informations de profil du compte d’utilisateur.
5. L’ID d’objet se trouve dans la section de l’identité, à droite.

    :::image type="content" source="media/tenant-and-object-id/azure-ad-admin-center.png" alt-text="Centre d’administration Azure Active Directory.":::

6. Recherchez **attributions de rôle** en sélectionnant **Contrôle d’accès (IAM)** dans le menu de gauche, puis **Attributions de rôle**.

    :::image type="content" source="https://docs.microsoft.com/azure/role-based-access-control/media/role-assignments-portal/role-assignments.png" alt-text="Attributions de rôles pour les ressources Azure":::

## <a name="find-ids-for-debugging"></a>Rechercher des ID pour le débogage

Cette section décrit comment rechercher un ID de locataire, d’objet et d’association de partenaire pour le débogage.

1. Accédez au [portail Azure](https://ms.portal.azure.com/).
2. Ouvrez Azure Cloud Shell en sélectionnant l’icône PowerShell en haut à droite.

    :::image type="content" source="media/tenant-and-object-id/icon-azure-cloud-shell-1.png" alt-text="Icône PowerShell en haut à droite de l’écran.":::

3. Sélectionnez **PowerShell**.

    :::image type="content" source="media/tenant-and-object-id/icon-powershell.png" alt-text="Sélectionnez le lien PowerShell.":::

4. Cochez la case **Abonnement** pour choisir celui auquel le partenaire est lié, puis **Créer un stockage**. Il s’agit d’une action à effectuer une seule fois. Si vous avez déjà configuré le stockage, passez à l’étape suivante.

    :::image type="content" source="media/tenant-and-object-id/create-storage-window.png" alt-text="Sélectionnez le bouton Créer le stockage.":::

5. La création (ou la présence) d’un stockage ouvre la fenêtre Azure Cloud Shell.

    :::image type="content" source="media/tenant-and-object-id/cloud-shell-window-1.png" alt-text="Fenêtre Azure Cloud Shell.":::

6. Pour plus d’informations sur les associations de partenaires, installez l’extension avec cette commande :<br>`az extension add --name managementpartner`.<br>Azure Cloud Shell indique si l’extension est déjà installée :

    :::image type="content" source="media/tenant-and-object-id/cloud-shell-window-2.png" alt-text="Fenêtre Azure Cloud Shell indiquant si l’extension est déjà installée.":::

7. Vérifiez les détails du partenaire à l’aide de cette commande :<br>`az managementpartner show --partner-id xxxxxx`<br>Exemple : `az managementpartner show --partner-id 4760962`.<br>faites une capture d’écran des résultats de la commande, qui ressemble à ceci :

    :::image type="content" source="media/tenant-and-object-id/partner-id-sample-screenshot.png" alt-text="Écran d’exemple qui affiche les résultats de la commande précédente pour afficher l’ID du partenaire.":::

>[!NOTE]
>Si plusieurs abonnements requièrent une capture d’écran, utilisez cette commande pour basculer entre les abonnements :<br>`az account set --subscription "My Demos"`

## <a name="next-steps"></a>Étapes suivantes

- [Pays et régions pris en charge](sell-from-countries.md)
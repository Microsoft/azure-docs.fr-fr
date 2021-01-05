---
title: Créer un espace de travail avec la protection contre l’exfiltration de données activée
description: Cet article explique comment créer un espace de travail avec la protection contre l’exfiltration de données dans Azure Synapse Analytics
author: NanditaV
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: security
ms.date: 12/01/2020
ms.author: NanditaV
ms.reviewer: jrasnick
ms.openlocfilehash: 8f11f104e8d3d0a4f3a004ebe9f149cb4132d910
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96501270"
---
# <a name="create-a-workspace-with-data-exfiltration-protection-enabled"></a>Créer un espace de travail avec la protection contre l’exfiltration de données activée
Cet article explique comment créer un espace de travail avec la protection contre l’exfiltration des données activée et comment gérer les locataires Azure AD approuvés pour cet espace de travail.

>[!Note]
>Vous ne pouvez pas modifier la configuration de l’espace de travail pour le réseau virtuel managé et la protection contre l’exfiltration de données après la création de l’espace de travail.

## <a name="prerequisites"></a>Prérequis
- Des autorisations pour créer une ressource d’espace de travail dans Azure.
- Les autorisations de l’espace de travail Synapse pour créer des points de terminaison privés managés.
- Abonnements inscrits pour le fournisseur de ressources de mise en réseau. [En savoir plus.](../../azure-resource-manager/management/resource-providers-and-types.md)

Suivez les étapes figurant dans [Démarrage rapide : Créer un espace de travail Synapse](../quickstart-create-workspace.md) pour commencer à créer votre espace de travail. Avant de créer votre espace de travail, utilisez les informations ci-dessous pour ajouter la protection contre l’exfiltration de données à votre espace de travail.

## <a name="add-data-exfiltration-protection-when-creating-your-workspace"></a>Ajouter une protection contre l’exfiltration de données lors de la création de votre espace de travail
1. Sous l’onglet Mise en réseau, activez la case à cocher « Activer le réseau virtuel managé ».
1. Sélectionnez « Oui » pour l’option « Autoriser le trafic de données sortantes uniquement vers les cibles approuvées ».
1. Choisissez les locataires Azure AD approuvés pour cet espace de travail.
1. Passez en revue la configuration et créez l’espace de travail.
:::image type="content" source="./media/how-to-create-a-workspace-with-data-exfiltration-protection/workspace-creation-data-exfiltration-protection.png" alt-text="Créer un espace de travail avec la protection contre l’exfiltration de données":::

## <a name="manage-approved-azure-active-directory-tenants-for-the-workspace"></a>Gérer les locataires Azure Active Directory approuvés pour l’espace de travail
1. À partir du Portail Azure de l’espace de travail, accédez à « Locataires Azure AD approuvés ». La liste des locataires Azure AD approuvés pour l’espace de travail est répertoriée ici. Le locataire de l’espace de travail est inclus par défaut et n’est pas répertorié.
1. Utilisez « + Ajouter » pour inclure les nouveaux locataires dans la liste approuvée.
1. Pour supprimer un locataire Azure AD dans la liste approuvée, sélectionnez le locataire et sélectionnez « Supprimer », puis « Enregistrer ».
:::image type="content" source="./media/how-to-create-a-workspace-with-data-exfiltration-protection/workspace-manage-aad-tenant-list.png" alt-text="Créer un espace de travail avec la protection contre l’exfiltration de données":::


## <a name="connecting-to-azure-resources-in-approved-azure-ad-tenants"></a>Connexion aux ressources Azure dans les locataires Azure AD approuvés

Vous pouvez créer des points de terminaison privés managés pour vous connecter aux ressources Azure qui résident dans les locataires Azure AD, qui sont approuvés pour un espace de travail. Suivez les étapes indiquées dans le guide pour [créer des points de terminaison privés managés](./how-to-create-managed-private-endpoints.md).

>[!IMPORTANT]
>Les ressources dans les locataires autres que le locataire de l’espace de travail ne doivent pas avoir de règles de pare-feu bloquantes pour que les pools SQL puissent s’y connecter. Les ressources au sein du réseau virtuel managé de l’espace de travail, comme les clusters Spark, peuvent se connecter via des liaisons privées managées aux ressources protégées par un pare-feu.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur la [protection contre l’exfiltration des données dans les espaces de travail Synapse](./workspace-data-exfiltration-protection.md)

En savoir plus sur le [Réseau virtuel d’espace de travail managé](./synapse-workspace-managed-vnet.md)

En savoir plus sur les [Points de terminaison privés managés](./synapse-workspace-managed-private-endpoints.md)

[Créer des points de terminaison privés managés à vos sources de données](./how-to-create-managed-private-endpoints.md)

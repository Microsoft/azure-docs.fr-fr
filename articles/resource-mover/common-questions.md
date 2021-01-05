---
title: Questions courantes sur Azure Resource Mover
description: Découvrez les réponses aux questions courantes sur Azure Resource Mover.
author: rayne-wiselman
manager: evansma
ms.service: resource-move
ms.topic: conceptual
ms.date: 09/14/2020
ms.author: raynew
ms.openlocfilehash: 68e5f937b8ad8367abf488598bda311a39d462c6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90600651"
---
# <a name="common-questions"></a>Questions courantes

Cet article répond à des questions courantes sur [Azure Resource Mover](overview.md).

## <a name="general"></a>Général

### <a name="is-resource-mover-generally-available"></a>Resource Mover est-il mis à la disposition générale ?

Resource Mover est actuellement disponible en préversion publique. Les charges de travail de production sont prises en charge.



## <a name="moving-across-regions"></a>Déplacement entre les régions

### <a name="can-i-move-resources-across-any-regions"></a>Puis-je déplacer des ressources d’une région à une autre ?

Actuellement, vous pouvez déplacer des ressources de n’importe quelle région publique source vers n’importe quelle région publique cible, selon les [types de ressources disponibles dans la région en question](https://azure.microsoft.com/global-infrastructure/services/). Le déplacement des ressources dans les régions Azure Government n’est pas pris en charge actuellement.

### <a name="what-resources-can-i-move-across-regions-using-resource-mover"></a>Quelles ressources puis-je déplacer d’une région à l’autre à l’aide de Resource Mover ?

À l’aide de Resource Mover, vous pouvez actuellement déplacer les ressources suivantes d’une région à l’autre :

- Machines virtuelles Azure et disques associés
- Cartes réseau
- Groupes à haute disponibilité 
- Réseaux virtuels Azure 
- Adresses IP publiques
- Groupes de sécurité réseau (NSG)
- Équilibreurs de charge internes et publics 
- Bases de données Azure SQL et pools élastiques


### <a name="can-i-move-resources-across-subscriptions-when-i-move-them-across-regions"></a>Puis-je déplacer des ressources d’un abonnement à l’autre lorsque je les déplace d’une région à l’autre ?

Vous pouvez modifier l’abonnement après avoir déplacé les ressources vers la région de destination. [En savoir plus](../azure-resource-manager/management/move-resource-group-and-subscription.md) sur le déplacement de ressources vers un autre abonnement. 

### <a name="where-is-the-metadata-for-moving-across-regions-stored"></a>Où sont stockées les métadonnées permettant d’effectuer un déplacement d’une région à l’autre ?

Elles sont stockées dans une base de données [Azure Cosmos](../cosmos-db/database-encryption-at-rest.md) et dans [Stockage Blob Azure](../storage/common/storage-service-encryption.md) au sein d’un abonnement Microsoft. Actuellement, les métadonnées sont stockées dans les régions USA Est 2 et Europe Nord. Nous allons étendre cette couverture à d’autres régions. Cela ne vous empêche pas de déplacer des ressources dans toutes les régions publiques.

### <a name="is-the-collected-metadata-encrypted"></a>Les métadonnées collectées sont-elles chiffrées ?

Oui, aussi bien en transit qu’au repos.
- Pendant le transit, les métadonnées sont envoyées en toute sécurité au service Resource Mover via Internet à l’aide du protocole HTTPS.
- Une fois dans le stockage, les métadonnées sont chiffrées.

### <a name="how-is-managed-identity-used-in-resource-mover"></a>Comment l’identité managée est-elle utilisée dans Resource Mover ?

L’[identité managée](../active-directory/managed-identities-azure-resources/overview.md) (anciennement identité du service géré, MSI) fournit des services Azure avec une identité managée automatiquement dans Azure AD.
- Resource Mover utilise une identité managée afin de pouvoir accéder aux abonnements Azure pour déplacer les ressources d’une région à l’autre.
- Une collection de déplacements a besoin d’une identité affectée par le système, avec un accès à l’abonnement qui contient les ressources que vous déplacez.

- Si vous déplacez des ressources d’une région à l’autre dans le portail, ce processus se produit automatiquement.
- Si vous déplacez des ressources à l’aide de PowerShell, vous exécutez des cmdlets pour attribuer une identité affectée par le système à la collection, puis vous attribuez un rôle avec les autorisations d’abonnement appropriées au principal d’identité. 

### <a name="what-managed-identity-permissions-does-resource-mover-need"></a>De quelles autorisations d’identité managée a besoin Resource Mover ?

L’identité managée Azure Resource Mover nécessite au moins les autorisations suivantes : 

- Autorisation d’écrire/de créer des ressources dans un abonnement utilisateur, disponible avec le rôle *Contributeur*. 
- Autorisation de créer des attributions de rôles. L’autorisation de créer des attributions de rôles existe dans le rôle *Propriétaire*, le rôle *Administrateur des accès utilisateur* ou un rôle personnalisé avec *l’autorisation Microsoft.Authorization/role assignments/write* attribuée. Cette autorisation n’est pas nécessaire si l’identité managée de la ressource de partage de données est déjà autorisée à accéder au magasin de données Azure. 
 
Lorsque vous ajoutez des ressources dans le hub Azure Resource Mover dans le portail, les autorisations sont gérées automatiquement tant que l’utilisateur dispose des autorisations décrites ci-dessus. Si vous ajoutez des ressources avec PowerShell, vous devez attribuer les autorisations manuellement.

> [!IMPORTANT]
> Nous vous recommandons vivement de ne pas modifier ni de supprimer les attributions de rôles d’identité. 

### <a name="what-should-i-do-if-i-dont-have-permissions-to-assign-role-identity"></a>Que dois-je faire si je n’ai pas l’autorisation d’attribuer une identité de rôle ?

**Cause possible** | **Recommandation**
--- | ---
Lorsque vous ajoutez une ressource pour la première fois, vous n’êtes pas *Contributeur* ni *Administrateur de l’accès utilisateur* (ou *Propriétaire*). | Utilisez un compte avec des autorisations *Contributeur* et *Administrateur de l’accès utilisateur* (ou *Propriétaire*) pour l’abonnement.
L’identité managée de Resource Mover n’a pas le rôle requis. | Ajoutez les rôles « Contributeur » et « Administrateur de l’accès utilisateur ».
L’identité managée de Resource Mover a été réinitialisée sur *Aucun*. | Réactivez une identité affectée par le système dans la collection de déplacements > **Identité**. Vous pouvez également rajouter la ressource dans **Ajouter des ressources**, ce qui fait la même chose.  
L’abonnement a été déplacé vers un autre locataire. | Désactivez puis activez l’identité managée pour la collection de déplacements.

### <a name="how-can-i-do-multiple-moves-together"></a>Comment puis-je effectuer plusieurs déplacements en même temps ?

Modifiez les combinaisons source/cible en fonction des besoins à l’aide de l’option de modification dans le portail.

## <a name="next-steps"></a>Étapes suivantes

[En savoir plus](about-move-process.md) sur les composants de Resource Mover et le processus de déplacement.

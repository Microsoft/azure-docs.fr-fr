---
title: Modifier les paramètres cibles à l’occasion du déplacement de machines virtuelles Azure entre différentes régions avec Azure Resource Mover
description: Découvrez comment modifier les paramètres cibles à l’occasion du déplacement de machines virtuelles Azure entre différentes régions avec Azure Resource Mover.
manager: evansma
author: rayne-wiselman
ms.service: resource-move
ms.topic: how-to
ms.date: 09/10/2020
ms.author: raynew
ms.openlocfilehash: 27e7c899f0d22789c10541fc98a0d2c63a7843ec
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95533053"
---
# <a name="modify-target-settings"></a>Modifier les paramètres cibles

Cet article explique comment modifier les paramètres cibles à l’occasion du déplacement de ressources entre différentes régions avec [Azure Resource Mover](overview.md).


## <a name="modify-vm-settings"></a>Modifier les paramètres de machine virtuelle

Au moment de déplacer des machines virtuelles Azure et les ressources associées, vous pouvez modifier les paramètres cibles. 

- Nous vous recommandons de ne modifier les paramètres cibles qu’une fois la collection de déplacement validée.
- Nous vous recommandons de modifier les paramètres avant de préparer les ressources, car certaines propriétés cibles risquent de ne plus pouvoir être modifiées à l’issue de la préparation.

Toutefois :
- Si vous déplacez la ressource source, vous pouvez généralement modifier les paramètres cibles tant que vous n’avez pas lancé le processus de déplacement.
- Si vous affectez une ressource existante dans la région source, vous pouvez modifier les paramètres cibles tant que la validation du déplacement n’est pas terminée.

### <a name="settings-you-can-modify"></a>Paramètres que vous pouvez modifier

Les paramètres de configuration que vous pouvez modifier sont résumés dans le tableau.

**Ressource** | **Options** 
--- | --- | --- 
**Nom de la machine virtuelle** | Options :<br/><br/> - Créer une nouvelle machine virtuelle sous le même nom dans la région cible.<br/><br/> - Créer une nouvelle machine virtuelle sous un nom différent dans la région cible.<br/><br/> - Utiliser une machine virtuelle existante dans la région cible.<br/><br/> Si vous créez une nouvelle machine virtuelle, mis à part les paramètres que vous modifiez, la nouvelle machine virtuelle hérite des mêmes paramètres que la source.
**Zone de disponibilité de la machine virtuelle** | Zone de disponibilité dans laquelle la machine virtuelle cible sera placée. Vous pouvez la marquer **N/A** si vous ne voulez pas modifier les paramètres de la source ou si vous n’avez pas l’intention de placer la machine virtuelle dans une zone de disponibilité.
**Référence de la machine virtuelle** | [Type de machine virtuelle](https://azure.microsoft.com/pricing/details/virtual-machines/series/) (disponible dans la région cible) qui sera utilisé pour la machine virtuelle cible.<br/><br/> La taille de la machine virtuelle cible sélectionnée ne doit pas être inférieure à celle de la machine virtuelle source.
**Ressources réseau** | Options pour les réseaux virtuels (VNets)/groupes de sécurité/interfaces réseau :<br/><br/> - Créer une nouvelle ressource sous le même nom dans la région cible.<br/><br/> - Créer une nouvelle ressource sous un nom différent dans la région cible.<br/><br/> - Utiliser une ressource réseau existante dans la région cible.<br/><br/> Si vous créez une nouvelle ressource cible, mis à part les paramètres que vous modifiez, elle hérite des mêmes paramètres que la ressource source.
**Nom de l’adresse IP publique** | Spécifiez le nom.
**Référence SKU d’adresse IP publique** | Spécifiez la référence [SKU](../virtual-network/public-ip-addresses.md#sku).
**Zone des adresses IP publiques** | Spécifiez la [zone](../virtual-network/public-ip-addresses.md#standard) pour les adresses IP publiques standard.<br/><br/> Si vous voulez qu’elles soient redondantes interzone, définissez-les comme étant **Redondant interzone**.
**Nom de l’équilibreur de charge** | Spécifiez le nom.
**Référence SKU de l’équilibreur de charge** | De base ou Standard. Nous vous recommandons d’utiliser l’option Standard.
**Zone de l’équilibreur de charge** | Spécifiez une zone pour l’équilibreur de charge. <br/><br/> Si vous voulez qu’il soit redondant interzone, définissez-le comme étant **Redondant interzone**.
**Dépendances des ressources** | Options pour chaque dépendance :<br/><br/>- La ressource utilise des ressources dépendantes de la source qui seront déplacées dans la région cible.<br/><br/> - La ressource utilise d’autres ressources dépendantes situées dans la région cible. Dans ce cas, vous pouvez effectuer un choix parmi des ressources similaires dans la région cible.

### <a name="edit-vm-target-settings"></a>Modifier les paramètres cibles de machine virtuelle

Si vous ne voulez déplacer des ressources dépendantes de la région source vers la cible, d’autres possibilités s’offrent à vous :

- Créez une nouvelle ressource dans la région cible. À moins de spécifier des paramètres différents, la nouvelle ressource aura les mêmes paramètres que la ressource source.
- Utilisez une ressource existante dans la région cible.

Le comportement exact dépend du type de ressource. [Apprenez-en davantage](modify-target-settings.md) sur la modification des paramètres cibles.

Pour modifier les paramètres cibles d’une ressource, vous devez utiliser l’entrée **Configuration cible** dans la collection de déplacement de ressources. 

Pour modifier un paramètre : 

1. Dans la page **Entre les régions** > colonne **Configuration cible**, cliquez sur le lien correspondant à l’entrée de ressource.
2. Dans **Paramètres de configuration**, vous pouvez créer une nouvelle machine virtuelle dans la région cible.
3. Affectez une nouvelle zone de disponibilité, un groupe à haute disponibilité ou une référence SKU à la machine virtuelle cible. **Zone de disponibilité** et référence **SKU**.

Les modifications ne sont apportées qu’à la ressource que vous modifiez. Vous devez mettre à jour toute ressource dépendante séparément.


## <a name="modify-sql-settings"></a>Modifier les paramètres SQL

Au moment de déplacer des ressources Azure SQL Database, vous pouvez modifier les paramètres cibles pour le déplacement. 

- Pour la base de données SQL :
    - Nous vous recommandons de modifier les paramètres de configuration cibles avant de préparer le déplacement.
    - Vous pouvez modifier les paramètres de la base de données cible et la redondance de zone pour la base de données.
- Pour les pools élastiques :
    -  Vous pouvez modifier la configuration cible à tout moment avant de lancer le déplacement.
    - Vous pouvez modifier le pool élastique cible et la redondance de zone pour le pool. 

### <a name="sql-settings-you-can-modify"></a>Paramètres SQL que vous pouvez modifier

**Paramètre** | **Base de données SQL** | **Pool élastique**
--- | --- | ---
**Nom** | Créer une nouvelle base de données sous le même nom dans la région cible.<br/><br/> Créer une nouvelle base de données sous un nom différent dans la région cible.<br/><br/> Utiliser une base de données existante dans la région cible. | Créer un nouveau pool élastique sous le même nom dans la région cible.<br/><br/> Créer un nouveau pool élastique sous un nom différent dans la région cible.<br/><br/> Utiliser un pool élastique existant dans la région cible.
**Redondance de zone** | Pour passer d’une région qui prend en charge la redondance de zone à une région qui ne la prend pas en charge, tapez **Désactiver** dans le paramètre de zone.<br/><br/> Pour passer d’une région qui ne prend pas en charge la redondance de zone à une région qui la prend en charge, tapez **Activer** dans le paramètre de zone. | Pour passer d’une région qui prend en charge la redondance de zone à une région qui ne la prend pas en charge, tapez **Désactiver** dans le paramètre de zone.<br/><br/> Pour passer d’une région qui ne prend pas en charge la redondance de zone à une région qui la prend en charge, tapez **Activer** dans le paramètre de zone.

### <a name="edit-sql-target-settings"></a>Modifier les paramètres cibles SQL

Pour modifier les paramètres cibles d’une ressource Azure SQL Database, procédez comme suit : 

1. Dans **Entre les régions**, pour la ressource que vous voulez modifier, cliquez sur l’entrée **Configuration cible**.
2. Dans **Paramètres de configuration**, spécifiez les paramètres cibles résumés dans le tableau ci-dessus.

## <a name="next-steps"></a>Étapes suivantes

[Déplacer une machine virtuelle Azure](tutorial-move-region-virtual-machines.md) dans une autre région.
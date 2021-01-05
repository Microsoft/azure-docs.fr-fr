---
title: Utiliser un tableau de bord pour un laboratoire de classe dans Azure Lab Services | Microsoft Docs
description: Découvrez comment utiliser un tableau de bord pour un laboratoire de classe dans Azure Lab Services.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: a91e1649d913956219ddcf192ab315a245b32eae
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96434598"
---
# <a name="dashboard-for-labs"></a>Tableau de bord des labos
Cet article décrit la vue du tableau de bord d’un laboratoire de classe dans Azure Lab Services. 

![Capture d’écran montrant la vue du tableau de bord d’un laboratoire de classe dans Azure Lab Services](./media/use-dashboard/dashboard.png)

## <a name="costs-and-billing-tile"></a>Vignette Coûts et facturation
Cette vignette fournit les détails d’estimation de coût suivants :

| Paramètre | Valeur | 
| ------- | ----- | 
| Heures de quota | Nombre maximal d’heures pendant lesquelles un utilisateur peut utiliser la machine virtuelle en dehors des heures planifiées. |
| Heures planifiées | Heures qui seront facturées en fonction de la planification définie dans le laboratoire. Cette valeur est disponible uniquement si une date de début/de fin est définie sur tous les événements de planification. |
| Heures/utilisateur | Somme des heures de quota et des heures planifiées. |
| Nombre maximal d’utilisateurs | Nombre maximal d’utilisateurs dans le laboratoire en fonction des machines virtuelles à revendiquer. |
| Heures x utilisateurs | Heures par utilisateur multipliées par le nombre d’utilisateurs. |
| Quota ajusté | Somme des heures de quota ajoutées à des utilisateurs spécifiques. |
| Nombre total d’heures * $/heure | Coût par heure en fonction de la taille de machine virtuelle sélectionnée. Il est basé sur le tarif du paiement à l’utilisation standard. |
| Coût estimé total | Il s’agit du prix maximal pour ce laboratoire basé sur les paramètres actuels. |

## <a name="template-tile"></a>Vignette de modèle
La vignette affiche les informations suivantes :

- Date de création du modèle 
- Date de dernière publication du modèle 

Elle comporte également un lien permettant d’accéder à la page **Modèle** dans laquelle vous pouvez [gérer le modèle de machines virtuelles](how-to-create-manage-template.md) pour la classe. 

## <a name="virtual-machine-pool-tile"></a>Vignette de pool de machine virtuelle

La vignette affiche les informations suivantes :

- Nombre de machines virtuelles affectées aux étudiants (utilisateurs)
- Nombre de machines virtuelles n’ayant pas encore été affectées aux étudiants

Elle comporte également un lien permettant d’accéder à la page **Pool de machine virtuelle** dans laquelle vous pouvez [gérer le pool de machines virtuelles](how-to-set-virtual-machine-passwords.md) dans le laboratoire. 

## <a name="users-tile"></a>Vignette Utilisateurs

La vignette affiche les informations suivantes :

- Nombre d’utilisateurs inscrits à la classe
- Nombre d’utilisateurs ajoutés au laboratoire mais non inscrits à la classe 

Elle comporte également un lien permettant d’accéder à la page **Utilisateurs** dans laquelle vous pouvez [gérer les utilisateurs](how-to-configure-student-usage.md) pour le laboratoire. 

## <a name="schedules-tile"></a>Vignette Planifications
Vous voyez les événements planifiés en cours pour le laboratoire sur la vignette. Elle comporte également un lien permettant d’accéder à la page **Planifications** dans laquelle vous pouvez [créer et gérer des planifications](how-to-create-schedules.md). La vignette affiche les détails de deux événements planifiés uniquement et le nombre d’événements planifiés restants pour le laboratoire. 

![Événements planifiés](./media/use-dashboard/scheduled-events.png)


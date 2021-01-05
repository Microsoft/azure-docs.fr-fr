---
title: Actions interdites pendant l’accès avec élévation de privilèges
description: Le moteur VMware rétablit les modifications pour s’assurer que le service reste ininterrompu lorsque le moteur VMware détecte l’une des actions interdites suivantes.
titleSuffix: Azure VMware Solution by CloudSimple
ms.date: 10/28/2020
ms.topic: article
ms.service: azure-vmware-cloudsimple
author: divka78
ms.author: dikamath
ms.openlocfilehash: 81a09a9cfe749de56f7306a8d5e1ae5db01295fc
ms.sourcegitcommit: d76108b476259fe3f5f20a91ed2c237c1577df14
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92915369"
---
# <a name="forbidden-actions-during-elevated-access"></a>Actions interdites pendant l’accès avec élévation de privilèges

Pendant l’intervalle d’élévation de privilèges, certaines actions sont interdites. Lorsque le moteur VMware détecte l’une des actions interdites suivantes, il rétablit les modifications pour s’assurer que le service reste ininterrompu.

## <a name="cluster-actions"></a>Actions sur le cluster

- Supprimer un cluster de vCenter.
- Modifier la haute disponibilité (HA) vSphere sur un cluster.
- Ajouter un hôte au cluster à partir de vCenter.
- Supprimer un hôte du cluster à partir de vCenter.

## <a name="host-actions"></a>Actions sur l’hôte

- Supprimer des magasins de données sur un hôte ESXi.
- Désinstaller l’agent vCenter de l’hôte.
- Modifier la configuration de l’hôte.
- Apporter des modifications aux profils de l’hôte.
- Placer un hôte en mode de maintenance.

## <a name="network-actions"></a>Actions sur le réseau

- Supprimer le commutateur virtuel distribué (DVS) par défaut dans un cloud privé.
- Supprimer un ordinateur hôte du DVS par défaut.
- Importer un paramètre DVS.
- Reconfigurer un paramètre DVS.
- Mettre DVS à niveau.
- Supprimer le groupe de ports de gestion.
- Modifier le groupe de ports de gestion.

## <a name="roles-and-permissions-actions"></a>Actions sur les rôles et les autorisations

- Créer un rôle global.
- Modifier ou supprimer une autorisation sur un objet de gestion.
- Modifier ou supprimer des rôles par défaut.
- Élever les privilèges à un rôle supérieur à celui du propriétaire du cloud.

## <a name="other-actions"></a>Autres actions

- Supprimer des licences par défaut :
  - Serveur vCenter
  - Nœuds ESXi
  - NSX-T
  - HCX
- Modifier ou supprimer le pool des ressources de gestion.
- Cloner les machines virtuelles de gestion.


## <a name="next-steps"></a>Étapes suivantes
[Maintenance et mises à jour CloudSimple](cloudsimple-maintenance-updates.md) 

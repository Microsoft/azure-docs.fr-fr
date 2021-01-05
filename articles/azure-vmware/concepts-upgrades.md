---
title: Concepts – Mises à jour et mises à niveau des clouds privés
description: Découvrez les processus et fonctionnalités clés de mise à niveau dans Azure VMware Solution.
ms.topic: conceptual
ms.date: 09/22/2020
ms.openlocfilehash: 6fb65a0e192fbf1f054857223b3fdaf58fc6a903
ms.sourcegitcommit: 31d242b611a2887e0af1fc501a7d808c933a6bf6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/29/2020
ms.locfileid: "97809647"
---
# <a name="azure-vmware-solution-private-cloud-updates-and-upgrades"></a>Mises à jour et mises à niveau des clouds privés Azure VMware Solution

L’un des principaux avantages des clouds privés d’Azure VMware Solution réside dans le fait que la plateforme est tenue à jour pour vous. La maintenance de la plateforme comprend les mises à jour automatisées vers une offre groupée de logiciels validée VMware, ce qui aide à garantir que vous utilisez la dernière version en date des logiciels de cloud privé Azure VMware Solution validés.

Plus précisément, un cloud privé Azure VMware Solution inclut :

- Des nœuds de serveurs nus dédiés provisionnés à l’aide de l’hyperviseur VMware ESXi 
- Un serveur vCenter pour la gestion d’ESXi et de vSAN 
- Une mise en réseau SDN (Software Defined Networking) VMware NSX-T pour les machines virtuelles de charge de travail vSphere  
- Un magasin de données VMware vSAN pour les machines virtuelles de charge de travail vSphere  
- VMware HCX pour la mobilité des charges de travail  

En plus de ces composants, un cloud privé Azure VMware Solution comprend des ressources dans le réseau sous-jacent Azure, requises pour la connectivité et pour faire fonctionner le cloud privé. Azure VMware Solution surveille en permanence l’intégrité des composants sous-jacents et VMware. Quand Azure VMware Solution détecte une défaillance, elle entreprend des actions pour réparer les composants défaillants. 

## <a name="what-components-get-updated"></a>Quels composants sont-ils mis à jour ?   

Azure VMware Solution met à jour les composants VMware suivants : 

- vCenter Server et ESXi s’exécutant sur les nœuds de serveurs nus 
- vSAN 
- NSX-T 

Azure VMware Solution met également à jour les logiciels sous-jacents, tels que les pilotes, les logiciels sur les commutateurs réseau et les microprogrammes sur les nœuds nus. 

## <a name="types-of-updates"></a>Types de mise à jour

Azure VMware Solution applique les types suivants de mises à jour aux composants VMware :

- Correctifs : correctifs de sécurité et correctifs de bogues publiés par VMware. 
- Mises à jour : mises à jour de la version mineure d’un ou de plusieurs composants VMware. 
- Mises à niveau : mises à jour de la version majeure d’un ou de plusieurs composants VMware.

Vous serez averti avant et après l’application des correctifs sur vos clouds privés. Nous travaillerons également avec vous pour planifier une fenêtre de maintenance avant d’appliquer des mises à jour ou des mises à niveau à votre cloud privé. 

## <a name="vmware-appliance-backup"></a>Sauvegarde d’une appliance VMware 

En plus d’effectuer des mises à jour, Azure VMware Solution effectue une sauvegarde de la configuration de ces composants VMware :

- Serveur vCenter 
- Gestionnaire NSX-T 

En cas d’échec, Azure VMware Solution peut les restaurer à partir de la sauvegarde de la configuration. 

Pour plus d’informations sur les versions des logiciels VMware, consultez l’[article sur les concepts des clusters et des clouds privés](concepts-private-clouds-clusters.md) ainsi que les [Questions fréquentes (FAQ)](faq.md).

## <a name="next-steps"></a>Étapes suivantes

L’étape suivante consiste à [créer un cloud privé](tutorial-create-private-cloud.md).

<!-- LINKS - external -->

<!-- LINKS - internal -->

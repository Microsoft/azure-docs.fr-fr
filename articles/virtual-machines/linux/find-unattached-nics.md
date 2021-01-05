---
title: Rechercher et supprimer les cartes réseau Azure non attachées
description: Guide pratique pour rechercher et supprimer des cartes réseau Azure non attachées aux machines virtuelles avec l’interface Azure CLI
author: cynthn
ms.service: virtual-machines
ms.subservice: networking
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 04/10/2018
ms.author: cynthn
ms.openlocfilehash: 5b28226cfe6bc51a2619c4dd63e666ddd51dad2a
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96016197"
---
# <a name="how-to-find-and-delete-unattached-network-interface-cards-nics-for-azure-vms"></a>Procédure de recherche et de suppression de carte d’interface réseau non attachées pour les machines virtuelles Azure
Quand vous supprimez une machine virtuelle dans Azure, les cartes d’interface réseau ne sont pas supprimées par défaut. Si vous créez et supprimez plusieurs machines virtuelles, les cartes réseau non utilisées continuent à valoriser les baux d’adresses IP internes. Quand vous créez d’autres cartes réseau de machines virtuelles, il se peut qu’elles soient dans l’incapacité d’obtenir un bail IP dans l’espace d’adressage du sous-réseau. Cet article vous explique comment rechercher et supprimer les cartes réseau non attachées.

## <a name="find-and-delete-unattached-nics"></a>Rechercher et supprimer les cartes réseau non attachées

La propriété *virtualMachine* d’une carte réseau stocke l’ID et le groupe de ressources de la machine virtuelle à laquelle est attachée la carte réseau. Le script suivant effectue une boucle sur l’ensemble des carte réseau d’un abonnement et vérifie si la propriété *virtualMachine* est nulle. Si cette propriété est nulle, la carte réseau n’est pas attachée à une machine virtuelle.

Pour afficher toutes les cartes réseau non attachées, il est fortement recommandé d’exécuter dans un premier temps le script avec la variable *deleteUnattachedNics* définie sur *0*. Pour supprimer toutes les cartes réseau non attachées après la révision de la sortie de liste, exécutez le script avec la variable *deleteUnattachedNics* définie sur *1*.

```azurecli
# Set deleteUnattachedNics=1 if you want to delete unattached NICs
# Set deleteUnattachedNics=0 if you want to see the Id(s) of the unattached NICs
deleteUnattachedNics=0

unattachedNicsIds=$(az network nic list --query '[?virtualMachine==`null`].[id]' -o tsv)
for id in ${unattachedNicsIds[@]}
do
   if (( $deleteUnattachedNics == 1 ))
   then

       echo "Deleting unattached NIC with Id: "$id
       az network nic delete --ids $id
       echo "Deleted unattached NIC with Id: "$id
   else
       echo $id
   fi
done
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la création et la gestion des réseaux virtuels dans Azure, consultez le didacticiel consacré à la [création et à la gestion des réseaux de machines virtuelles](tutorial-virtual-network.md).

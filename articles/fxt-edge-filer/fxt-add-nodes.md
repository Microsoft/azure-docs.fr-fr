---
title: 'Tutoriel : Ajouter des nœuds à un cluster Azure FXT Edge Filer'
description: Découvrez comment ajouter des nœuds de cluster au cache de stockage Azure FXT Edge Filer et activer la fonctionnalité Haute disponibilité.
author: ekpgh
ms.author: rohogue
ms.service: fxt-edge-filer
ms.topic: tutorial
ms.date: 06/20/2019
ms.openlocfilehash: 85ad78eeb095b427b1a6334f57c351e926022dff
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96021875"
---
# <a name="tutorial-add-cluster-nodes-to-an-azure-fxt-edge-filer-cluster"></a>Tutoriel : Ajouter des nœuds de cluster à un cluster Azure FXT Edge Filer

Un nouveau cluster Azure FXT Edge Filer est créé avec un seul nœud. Vous devez ajouter au moins deux nœuds supplémentaires et activer la haute disponibilité avant de procéder à toute autre configuration.

Ce tutoriel explique comment ajouter des nœuds de cluster et activer la fonctionnalité de haute disponibilité (HA).

Ce tutoriel vous apprendra à effectuer les opérations suivantes :

> [!div class="checklist"]
>
> * Ajouter des nœuds au cluster FXT.
> * Activer la haute disponibilité.

Les étapes décrites dans ce tutoriel prennent environ 45 minutes.

Avant de commencer ce tutoriel, mettez sous tension les nœuds que vous souhaitez ajouter et [définissez leurs mots de passe initiaux](fxt-node-password.md).

## <a name="1-load-the-cluster-nodes-page"></a>1. Charger la page de nœuds de cluster

Ouvrez le panneau de configuration du cluster dans un navigateur web et connectez-vous en tant qu’administrateur. (Des instructions détaillées figurent dans l’article de vue d’ensemble, sous [Ouvrir les pages de paramètres](fxt-cluster-create.md#open-the-settings-pages).)

Le panneau de configuration présente la page **Dashboard** (Tableau de bord) lors de son ouverture. 

![Panneau de configuration Dashboard (premier onglet)](media/fxt-cluster-config/dashboard-1-node.png)

Cette illustration montre le tableau de bord d’un cluster qui vient d’être créé, avec un seul nœud.

## <a name="2-locate-the-node-to-add"></a>2. Rechercher le nœud à ajouter

Pour ajouter des nœuds, cliquez sur l’onglet **Settings** et choisissez la page **FXT Nodes** dans la section **Cluster**.

![Onglet Settings du panneau de configuration (deuxième onglet) avec Cluster > FXT Nodes chargé](media/fxt-cluster-config/settings-fxt-nodes.png)

La liste **FXT Nodes - Unjoined** montre tous les nœuds FXT non affectés (la plupart des centres de données n’en ont que quelques-uns). Recherchez les nœuds FXT que vous souhaitez ajouter au cluster.

> [!Tip]
> Si vous ne trouvez pas le nœud souhaité dans la liste **Unjoined**, vérifiez qu’il remplit les conditions suivantes :
>
> * Il est sous tension et un [mot de passe racine a été défini](fxt-node-password.md)
> * Il est connecté à un réseau auquel vous pouvez accéder. Si vous utilisez des réseaux locaux virtuels, il doit être sur le même réseau local virtuel que le cluster
> * Il peut être détecté avec le protocole Bonjour
>
>   Certains paramètres de pare-feu bloquent les ports TCP/UDP utilisés par Bonjour, ce qui empêche le système d’exploitation FXT de détecter automatiquement les nœuds.
>
> Si le nœud que vous souhaitez ajouter ne figure pas dans la liste, essayez les solutions suivantes :
>
> * Cliquez sur le bouton **Manual Discover** pour le rechercher par adresse IP
>
> * Attribuez manuellement des adresses IP temporaires. Cela est rare, mais peut être nécessaire si vous utilisez des réseaux locaux virtuels avec étiquettes et que les nœuds ne se trouvent pas sur le réseau approprié, ou si votre réseau n’autorise pas les adresses IP attribuées automatiquement. Suivez les instructions de l’ancienne version de ce document pour [définir manuellement une adresse IP statique](https://azure.github.io/Avere/legacy/create_cluster/4_8/html/static_ip.html)

Le nom du nœud, l’adresse IP, la version du logiciel et l’état d’éligibilité sont affichés dans la liste. En règle générale, la colonne **Status** indique « Wants to join » ou décrit un problème matériel ou système qui rend le nœud non autorisé à rejoindre le cluster.

La colonne **Actions** comporte des boutons qui vous permettent d’ajouter le nœud au cluster ou de mettre à jour son logiciel. Le bouton de mise à jour installe automatiquement la version du logiciel qui correspond aux nœuds déjà présents dans le cluster.

Tous les nœuds d’un cluster doivent utiliser la même version du système d’exploitation, mais vous n’avez pas besoin de mettre à jour le logiciel avant d’ajouter un nœud. Une fois que vous avez cliqué sur le bouton **Allow To Join**, le processus de jonction au cluster vérifie et installe automatiquement le logiciel du système d’exploitation qui correspond à la version sur le cluster.

Pour en savoir plus sur les options de cette page, consultez [**Cluster** > **FXT Nodes**](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gui_fxt_nodes.html) dans le Guide de configuration de cluster.

## <a name="3-click-the-allow-to-join-button"></a>3. Cliquer sur le bouton « Allow to Join »

Cliquez sur le bouton **Allow to Join** _ (Autoriser à joindre) dans la colonne _ *Actions** correspondant au nœud que vous souhaitez ajouter.

Une fois que vous avez cliqué sur le bouton, l’état du nœud peut changer tandis que son logiciel est mis à jour en vue de son ajout au cluster.

L’image ci-dessous montre un nœud qui est en train de rejoindre le cluster (il reçoit probablement une mise à jour du système d’exploitation avant d’être ajouté). Aucun bouton n’apparaît dans la colonne **Actions** pour les nœuds qui sont en cours d’ajout au cluster.

![ligne du tableau de nœuds montrant le nom d’un nœud, l’adresse IP, la version du logiciel, le message « Allowed to join » et une dernière colonne vide](media/fxt-cluster-config/node-join-in-process.png)

Après quelques instants, le nouveau nœud doit apparaître dans la liste des nœuds du cluster en haut de la page de paramètres **FXT Nodes**.

Répétez ce processus pour ajouter les autres nœuds à votre cluster. Vous n’avez pas besoin d’attendre qu’un nœud ait fini de rejoindre le cluster avant de commencer la jonction d’un autre nœud.

## <a name="enable-high-availability"></a>Activer la haute disponibilité

Une fois que vous avez ajouté un second nœud à votre cluster, un message d’avertissement peut s’afficher sur le panneau de configuration Dashboard pour signaler que la fonctionnalité de haute disponibilité n’est pas configurée.

La haute disponibilité (HA) autorise une compensation entre les nœuds du cluster si l’un d’eux tombe en panne. La haute disponibilité n’est pas activée par défaut.

![Onglet Dashboard avec le message « The cluster has more than one node, but HA is not enabled… » (Le cluster a plusieurs nœuds, mais la haute disponibilité n’est pas activée) dans le tableau Conditions](media/fxt-cluster-config/no-ha-2-nodes.png)

> [!Note]
> N’activez pas la haute disponibilité avant d’avoir au moins trois nœuds dans le cluster.

Suivez cette procédure pour activer la haute disponibilité :

1. Chargez la page **High Availability** dans la section **Cluster** de l’onglet **Settings**.

   ![Page de configuration de haute disponibilité (Cluster > High Availability). La case à cocher « Enable HA » se trouve en haut et le bouton Submit se trouve en bas.](media/fxt-cluster-config/enable-ha.png)

2. Cochez la case **Enable HA** et cliquez sur le bouton **Submit**.

Une alerte s’affiche sur le **tableau de bord** pour confirmer que la haute disponibilité est activée.

![Tableau Dashboard affichant le message « HA is now fully configured »](media/fxt-cluster-config/ha-configured-alert.png)

## <a name="next-steps"></a>Étapes suivantes

Une fois que vous avez ajouté tous les nœuds à votre cluster, poursuivez l’installation en configurant le stockage à long terme de votre cluster.

> [!div class="nextstepaction"]
> [Ajouter du stockage back-end et configurer l’espace de noms virtuel](fxt-add-storage.md)
---
title: Monter des clients sur le cluster Microsoft Azure FXT Edge Filer
description: Comment les ordinateurs clients NFS peuvent monter le cache de stockage hybride d’Azure FXT Edge Filer
author: ekpgh
ms.service: fxt-edge-filer
ms.topic: tutorial
ms.date: 06/20/2019
ms.author: rohogue
ms.openlocfilehash: fa1f9da2c60aaf4c552916d16c266e984bf08892
ms.sourcegitcommit: f88074c00f13bcb52eaa5416c61adc1259826ce7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92340510"
---
# <a name="tutorial-mount-the-cluster"></a>Tutoriel : Monter le cluster

Ce didacticiel vous apprend à monter des clients NFS pour le cluster Azure FXT Edge Filer. Les clients montent les chemins d’accès à l’espace de noms virtuel que vous avez attribués lorsque vous avez ajouté le système de stockage principal.

Ce didacticiel présente :

> [!div class="checklist"]
>
> * Les stratégies d’équilibrage de charge pour les clients de toute la plage d’adresses IP exposées aux clients
> * La méthode de construction d’un chemin de montage à partir d’une adresse IP exposée aux clients et d’une jonction d’espace de noms
> * Les arguments à utiliser dans une commande mount

Ce tutoriel prend environ 45 minutes.

## <a name="steps-to-mount-the-cluster"></a>Étapes de montage du cluster

Suivez ces étapes pour connecter les ordinateurs clients à votre cluster Azure FXT Edge Filer.

1. Décidez comment équilibrer la charge du trafic client entre vos nœuds de cluster. Pour plus d’informations, lisez [Équilibrer la charge du client](#balance-client-load), ci-dessous.
1. Identifiez l’adresse IP du cluster et le chemin de jonction sur lesquels effectuer le montage.
1. Définissez le chemin exposé aux clients pour le montage.
1. Émettez la [commande mount](#use-recommended-mount-command-options) avec les arguments appropriés.

## <a name="balance-client-load"></a>Équilibrer la charge du client

Pour équilibrer les requêtes du client entre tous les nœuds du cluster, vous devez montez les clients sur la plage complète d’adresses IP côté client. Il existe plusieurs façons d’automatiser cette tâche.

Pour découvrir l’équilibrage de charge DNS par tourniquet (round-robin) pour le cluster, veuillez consulter la section [Configurer des serveurs DNS pour le cluster Azure FXT Edge Filer](fxt-configure-network.md#configure-dns-for-load-balancing). Pour utiliser cette méthode, vous devez assurer la maintenance d’un serveur DNS, ce qui n’est pas expliqué dans ces articles.

Une méthode plus simple pour les petites installations consiste à utiliser un script pour attribuer des adresses IP tout au long de la plage au moment du montage du client.

D’autres méthodes d’équilibrage de charge peuvent convenir pour les systèmes complexes ou de grande taille. Consultez votre agent Microsoft ou ouvrez une [demande de support](fxt-support-ticket.md) pour obtenir de l’aide. (Azure Load Balancer n’est *pas pris en charge* pour le moment avec Azure FXT Edge Filer.)

## <a name="create-the-mount-command"></a>Créer la commande mount

À partir de votre client, la commande ``mount`` mappe le cluster Azure FXT Edge Filer sur un chemin sur le système de fichiers local.

Le format est ``mount <FXT cluster path> <local path> {options}``

La commande mount comprend trois éléments :

* chemin du cluster : combinaison d’une adresse IP et d’un chemin de jonction d’espaces de noms décrite ci-dessous
* chemin local : chemin sur le client
* options de la commande mount : répertoriées dans [Utiliser les options de commande mount recommandées](#use-recommended-mount-command-options)

### <a name="create-the-cluster-path"></a>Créer le chemin d’accès du cluster

Le chemin du cluster est une combinaison de son *adresse IP* vserver et du chemin à une *jonction d’espace de noms*. La jonction d’espace de noms est un chemin virtuel qui a été défini lors de l’[ajout du système de stockage](fxt-add-storage.md#create-a-junction).

Par exemple, si vous avez utilisé ``/fxt/files`` comme chemin de votre espace de noms, vos clients montent *adresse_IP*:/fxt/files sur leur point de montage local.

![Boîte de dialogue « Ajouter une jonction » avec /avere/files dans le champ de chemin d’espace de noms](media/fxt-mount/fxt-junction-example.png)

L’adresse IP est l’une des adresses IP côté client définies pour le serveur virtuel (vserver). La plage d’adresses IP côté client est disponible à deux emplacements dans le Panneau de configuration du cluster :

* Table **VServers** (onglet Tableau de bord) :

  ![Onglet Tableau de bord du Panneau de configuration avec l’onglet VServers sélectionné dans la table de données sous le graphe et la section des adresses IP encadrée](media/fxt-mount/fxt-ip-addresses-dashboard.png)

* Page de paramètres **Réseau côté client** :

  ![Page de configuration Paramètres > VServer > Réseau côté client avec un cadre autour de la section Plage d’adresses de la table pour un serveur virtuel (vserver) particulier](media/fxt-mount/fxt-ip-addresses-settings.png)

Combinez l’adresse IP et le chemin d’accès de l’espace de noms pour former le chemin d’accès du cluster pour la commande mount.

Exemple de commande mount client : ``mount 10.0.0.12:/sd-access /mnt/fxt {options}``

### <a name="create-the-local-path"></a>Créer le chemin d’accès local

Il vous revient de créer le chemin d’accès local pour la commande mount. Vous pouvez définir n’importe quelle structure de chemin d’accès en tant que partie de l’espace de noms virtuels. Nous vous recommandons de concevoir un espace de noms et un chemin d’accès local pratique pour le flux de travail de votre client.

Pour plus d’informations sur l’espace de noms utilisé par le client, consultez la section [vue d’ensemble de l’espace de noms](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gns_overview.html) du Guide de configuration du cluster.

Outre les chemins, vous devez inclure les [options de la commande mount](#use-recommended-mount-command-options) décrits ci-dessous lors du montage de chaque client.

### <a name="use-recommended-mount-command-options"></a>Utiliser les options de commande mount recommandées

Pour garantir un montage sans interruption du client, passez les paramètres et arguments suivants dans votre commande mount :

``mount -o hard,nointr,proto=tcp,mountproto=tcp,retry=30 ${VSERVER_IP_ADDRESS}:/${NAMESPACE_PATH} ${LOCAL_FILESYSTEM_MOUNT_POINT}``

| Paramètres obligatoires | Description |
--- | ---
``hard`` | Les montages conditionnels sur le cluster Azure FXT Edge Filer sont associés à des échecs d’application et à des pertes de données possibles.
``proto=netid`` | Cette option prend en charge la gestion appropriée des erreurs réseau NFS.
``mountproto=netid`` | Cette option prend en charge la gestion appropriée des erreurs réseau pour les opérations de montage.
``retry=n`` | Définissez ``retry=30`` pour éviter les échecs de montage temporaires. (Une valeur différente est recommandée dans les montages de premier plan.)

| Paramètres favoris  | Description |
--- | ---
``nointr``            | Si vos clients utilisent des noyaux de systèmes d’exploitation plus anciens (datant d’avant avril 2008) qui prennent en charge cette option, utilisez-la. Notez que l’option « intr » est la valeur par défaut.

## <a name="next-steps"></a>Étapes suivantes

Une fois que vous avez monté des clients, vous pouvez tester votre flux de travail et apprendre à utiliser votre cluster.

Si vous avez besoin de déplacer des données vers un core filer cloud, tirez parti de la structure du cache en utilisant une ingestion des données parallèle. Certaines stratégies sont décrites dans l’article [Déplacement des données vers un cluster vFXT](../avere-vfxt/avere-vfxt-data-ingest.md). (Avere vFXT pour Azure est un produit basé sur le cloud qui utilise une technologie de mise en cache très similaire à celle d’Azure FXT Edge Filer.)

Si vous avez besoin d’aide pour résoudre des problèmes de matériel, veuillez consulter la section [Superviser l’état du matériel Azure FXT Edge Filer](fxt-monitor.md).
---
title: Prérequis pour Azure HPC Cache
description: Prérequis à l’utilisation d’Azure HPC Cache
author: ekpgh
ms.service: hpc-cache
ms.topic: how-to
ms.date: 11/05/2020
ms.author: v-erkel
ms.openlocfilehash: a31aee3f4548d3137fa1241aaa3a0f6171cf6895
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94412508"
---
# <a name="prerequisites-for-azure-hpc-cache"></a>Prérequis pour Azure HPC Cache

Avant d’utiliser le portail Azure pour créer une nouvelle instance d’Azure HPC Cache, vérifiez que votre environnement répond à ces exigences.

## <a name="video-overviews"></a>Vues d’ensemble en vidéo

Regardez ces vidéos pour obtenir une vue d’ensemble rapide des composants du système et de ce dont ils ont besoin pour fonctionner ensemble.

(Cliquez sur l’image vidéo ou sur le lien pour regarder son contenu.)

* [Fonctionnement](https://azure.microsoft.com/resources/videos/how-hpc-cache-works/) - Explique comment Azure HPC Cache interagit avec le stockage et les clients

  [![image miniature de la vidéo : Azure HPC Cache : Fonctionnement (cliquez pour accéder à la page vidéo)](media/video-2-components.png)](https://azure.microsoft.com/resources/videos/how-hpc-cache-works/)  

* [Prérequis](https://azure.microsoft.com/resources/videos/hpc-cache-prerequisites/) - Décrit les conditions requises pour le stockage NAS, Stockage Blob Azure, l’accès réseau et l’accès client

  [![image miniature de la vidéo : Azure HPC Cache : Prérequis (cliquez pour accéder à la page vidéo)](media/video-3-prerequisites.png)](https://azure.microsoft.com/resources/videos/hpc-cache-prerequisites/)

Pour obtenir des suggestions spécifiques, lisez le reste de cet article.

## <a name="azure-subscription"></a>Abonnement Azure

Un abonnement payant est recommandé.

## <a name="network-infrastructure"></a>Infrastructure réseau

Avant de pouvoir utiliser votre cache, vous devez configurer deux prérequis liés au réseau :

* Un sous-réseau dédié pour l’instance Azure HPC Cache
* La prise en charge du DNS afin que le cache puisse accéder au stockage et aux autres ressources

### <a name="cache-subnet"></a>Sous-réseau du cache

Azure HPC Cache a besoin d’un sous-réseau dédié avec les qualités suivantes :

* Le sous-réseau doit disposer d’au moins 64 adresses IP.
* Le sous-réseau ne peut pas héberger d’autres machines virtuelles, même pour les services associés tels que les ordinateurs clients.
* Si vous utilisez plusieurs instances Azure HPC Cache, chacune d’elles doit avoir son propre sous-réseau.

La méthode recommandée consiste à créer un nouveau sous-réseau pour chaque cache. Vous pouvez créer un réseau virtuel et un sous-réseau dans le cadre de la création du cache.

### <a name="dns-access"></a>Accès DNS

Le cache a besoin d’un accès DNS pour accéder aux ressources situées en dehors de son réseau virtuel. Selon les ressources que vous utilisez, vous devrez peut-être configurer un serveur DNS personnalisé et configurer un transfert entre ce serveur et les serveurs Azure DNS :

* Pour accéder aux points de terminaison du stockage Blob Azure et à d’autres ressources internes, vous avez besoin d’un serveur DNS basé sur Azure.
* Pour accéder au stockage local, vous devez configurer un serveur DNS personnalisé capable de résoudre les noms d’hôte de votre stockage. Vous devez effectuer cette opération **avant** de créer le cache.

Si vous avez uniquement besoin d’accéder au stockage Blob, vous pouvez utiliser le serveur DNS par défaut qui est fourni par Azure pour votre cache. Toutefois, si vous avez besoin d’accéder à d’autres ressources, vous devez créer un serveur DNS personnalisé et le configurer dans le but de transférer toutes les requêtes de résolution concernant Azure vers le serveur Azure DNS

Pour utiliser un serveur DNS personnalisé, vous devez effectuer ces étapes de configuration avant de créer votre cache :

* Créez le réseau virtuel qui hébergera Azure HPC Cache.
* Créez le serveur DNS.
* Ajoutez le serveur DNS au réseau virtuel du cache.

  Procédez comme suit pour ajouter le serveur DNS au réseau virtuel dans le portail Azure :

  1. Ouvrez le réseau virtuel dans le portail Azure.
  1. Choisissez **Serveurs DNS** dans le menu **Paramètres** de la barre latérale.
  1. Sélectionnez **Personnalisée**.
  1. Entrez l’adresse IP du serveur DNS dans le champ.

Un simple serveur DNS peut également être utilisé pour équilibrer la charge des connexions clientes entre tous les points de montage du cache disponibles.

Pour plus d’informations sur les réseaux virtuels Azure et les configurations de serveur DNS, consultez [Résolution de noms des ressources dans les réseaux virtuels Azure](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md).

## <a name="permissions"></a>Autorisations

Consultez les prérequis liés aux autorisations avant de créer votre cache.

* L’instance de cache doit pouvoir créer des interfaces réseau virtuelles (NIC). L’utilisateur qui crée le cache doit disposer de privilèges suffisants dans l’abonnement pour créer des cartes réseau.

* Si vous utilisez le stockage Blob, Azure HPC Cache a besoin d’une autorisation pour accéder à votre compte de stockage. Utilisez le contrôle d’accès basé sur le rôle Azure (Azure RBAC) pour permettre au cache d’accéder à votre stockage Blob. Deux rôles sont nécessaires : Contributeur de comptes de stockage et Contributeur aux données Blob du stockage.

  Suivez les instructions fournies dans [Ajouter des cibles de stockage](hpc-cache-add-storage.md#add-the-access-control-roles-to-your-account).

## <a name="storage-infrastructure"></a>Infrastructure du stockage

Le cache prend en charge les conteneurs d’objets blob Azure et les exportations de stockage matériel NFS. Ajoutez des cibles de stockage après avoir créé le cache.

Chaque type de stockage possède des conditions préalables spécifiques.

### <a name="blob-storage-requirements"></a>Exigences relatives au stockage Blob

Si vous souhaitez utiliser le stockage Blob Azure avec votre cache, vous aurez besoin d’un compte de stockage compatible, et soit d’un conteneur d’objets blob vide, soit d’un conteneur comprenant des données au format Azure HPC Cache. Pour plus d’informations, consultez [Déplacer des données vers le stockage Blob Azure](hpc-cache-ingest.md).

Créez le compte avant de tenter d’ajouter une cible de stockage. Vous pouvez créer un conteneur lorsque vous ajoutez la cible.

Pour créer un compte de stockage compatible, utilisez les paramètres suivants :

* Performances : **Standard**
* Type de compte : **StorageV2 (usage général v2)**
* Réplication : **Stockage localement redondant (LRS)**
* Niveau d’accès (par défaut) : **Chaud**

Il est recommandé d’utiliser un compte de stockage se trouvant au même emplacement que votre cache.

Vous devez également autoriser l’application de cache à accéder à votre compte de stockage Azure, tel que mentionné dans [Autorisations](#permissions), ci-dessus. Suivez la procédure indiquée dans [Ajouter des cibles de stockage](hpc-cache-add-storage.md#add-the-access-control-roles-to-your-account) pour accorder au cache les rôles d’accès requis. Si vous n’êtes pas le propriétaire du compte de stockage, demandez au propriétaire d’effectuer cette étape.

### <a name="nfs-storage-requirements"></a>Conditions requises pour le stockage NFS
<!-- linked from configuration.md -->

Si vous utilisez un système de stockage NFS (par exemple, un système NAS matériel local), assurez-vous qu’il répond à ces exigences. Vous devrez peut-être travailler avec les administrateurs réseau ou les managers de pare-feu de votre système de stockage (ou centre de données) pour vérifier ces paramètres.

> [!NOTE]
> La création de la cible de stockage échoue si le cache ne dispose pas d’un accès suffisant au système de stockage NFS.

Pour plus d’informations, consultez [Résoudre les problèmes de configuration NAS et de cible de stockage NFS](troubleshoot-nas.md).

* **Connectivité réseau :** Azure HPC Cache a besoin d’un accès réseau à bande passante élevée entre le sous-réseau du cache et le centre de données du système NFS. Il est recommandé de disposer d’un accès [ExpressRoute](../expressroute/index.yml) ou similaire. Si vous utilisez un VPN, vous devrez peut-être le configurer pour fixer la MSS TCP à 1350 afin de vous assurer que les paquets volumineux ne sont pas bloqués. Lisez les [restrictions de taille des paquets VPN](troubleshoot-nas.md#adjust-vpn-packet-size-restrictions) pour obtenir de l'aide supplémentaire sur le dépannage des paramètres VPN.

* **Accès au port :** Le cache a besoin d’accéder à des ports TCP/UDP spécifiques sur votre système de stockage. Les différents types de stockage ont des exigences de port différentes.

  Pour vérifier les paramètres de votre système de stockage, procédez comme suit.

  * Adressez une commande `rpcinfo` à votre système de stockage pour vérifier les ports nécessaires. La commande ci-dessous répertorie les ports et met en forme les résultats pertinents dans une table. (Utilisez l’adresse IP de votre système à la place du terme *<storage_IP>* .)

    Vous pouvez lancer cette commande à partir de tout client Linux doté d’une infrastructure NFS. Si vous utilisez un client à l’intérieur du sous-réseau du cluster, il peut également vous aider à vérifier la connectivité entre le sous-réseau et le système de stockage.

    ```bash
    rpcinfo -p <storage_IP> |egrep "100000\s+4\s+tcp|100005\s+3\s+tcp|100003\s+3\s+tcp|100024\s+1\s+tcp|100021\s+4\s+tcp"| awk '{print $4 "/" $3 " " $5}'|column -t
    ```

  Vérifiez que tous les ports retournés par la requête ``rpcinfo`` autorisent sans restrictions le trafic provenant du sous-réseau d’Azure HPC Cache.

  * Si vous ne pouvez pas utiliser la commande `rpcinfo`, assurez-vous que ces ports couramment utilisés autorisent le trafic entrant et sortant :

    | Protocol | Port  | Service  |
    |----------|-------|----------|
    | TCP/UDP  | 111   | rpcbind  |
    | TCP/UDP  | 2049  | NFS      |
    | TCP/UDP  | 4045  | nlockmgr |
    | TCP/UDP  | 4046  | mountd   |
    | TCP/UDP  | 4047  | status   |

    Certains systèmes utilisent des numéros de port différents pour ces services : consultez la documentation de votre système de stockage pour vous en assurer.

  * Vérifiez les paramètres du pare-feu pour vous assurer qu’ils autorisent le trafic sur tous ces ports requis. Veillez à vérifier les pare-feux utilisés dans Azure ainsi que ceux de votre centre de données.

* **Accès au répertoire :** Activez la commande `showmount` sur le système de stockage. Azure HPC Cache utilise cette commande pour vérifier que la configuration de votre cible de stockage pointe vers une exportation valide ainsi que pour s’assurer que plusieurs montages n’accèdent pas aux mêmes sous-répertoires (risque de collision de fichiers).

  > [!NOTE]
  > Si votre système de stockage NFS utilise le système d’exploitation ONTAP 9.2 de NetApp, **n’activez pas `showmount`** . [Contactez les services du Support Technique Microsoft](hpc-cache-support-ticket.md) pour obtenir de l’aide.

  Pour en savoir plus sur l'accès aux listes de répertoires, consultez l'[article sur le dépannage de la cible de stockage NFS](troubleshoot-nas.md#enable-export-listing).

* **Accès racine** (lecture/écriture) : Le cache se connecte au système principal en tant qu’identifiant utilisateur 0. Vérifiez ces paramètres sur votre système de stockage :
  
  * Activez `no_root_squash`. Cette option permet de s’assurer que l’utilisateur racine distant peut accéder aux fichiers appartenant à la racine.

  * Vérifiez les stratégies d’exportation pour vous assurer qu’elles n’incluent pas de restrictions sur l’accès à la racine depuis le sous-réseau du cache.

  * Si votre stockage comporte des exportations représentant des sous-répertoires d'une autre exportation, assurez-vous que le cache dispose d’un accès à la racine du segment le plus bas du chemin. Pour plus de détails, consultez l'article sur le dépannage de la cible de stockage NFS [Accès racine aux chemins d’accès aux répertoires](troubleshoot-nas.md#allow-root-access-on-directory-paths).

* Le stockage back-end NFS doit être une plateforme matérielle ou logicielle compatible. Pour plus d’informations, contactez l’équipe Azure HPC Cache.

## <a name="set-up-azure-cli-access-optional"></a>Configuration de l’accès Azure CLI (facultatif)

Si vous souhaitez créer ou gérer Azure HPC Cache à partir de l’interface de ligne de commande Azure (Azure CLI), vous devez installer le logiciel CLI et l’extension hpc-cache. Suivez les instructions dans [Configurer Azure CLI pour Azure HPC Cache](az-cli-prerequisites.md).

## <a name="next-steps"></a>Étapes suivantes

* [Créez une instance d’Azure HPC Cache](hpc-cache-create.md) à partir du portail Azure.
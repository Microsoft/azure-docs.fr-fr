---
title: Monter un cache Azure HPC Cache
description: Comment connecter des clients à un service Azure HPC Cache
author: ekpgh
ms.service: hpc-cache
ms.topic: how-to
ms.date: 09/30/2020
ms.author: v-erkel
ms.openlocfilehash: 7f1d8d34d6351fc344fdb101ac8e9a96678df9d5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91651426"
---
# <a name="mount-the-azure-hpc-cache"></a>Monter le cache Azure HPC Cache

Une fois le cache créé, les clients NFS peuvent y accéder à l’aide d’une simple commande `mount`. La commande connecte un chemin d’accès cible de stockage spécifique sur le service Azure HPC Cache à un répertoire local sur la machine cliente.

La commande Mount est composée des éléments suivants :

* L’une des adresses de montage du cache (figurant sur la page de vue d’ensemble du cache)
* Un chemin d’espace de noms virtuel que vous définissez pour la cible de stockage (indiqué sur la page Espace de noms du cache)
* Chemin d’accès local à utiliser sur le client
* Paramètres de commande qui optimisent la réussite de ce type de montage NFS

La page **Instructions de montage** pour votre cache collecte les informations et les options recommandées pour vous, et crée une commande de montage prototype que vous pouvez copier. Pour plus d’informations, consultez [Utiliser l’utilitaire d’instructions de montage](#use-the-mount-instructions-utility).

## <a name="prepare-clients"></a>Préparer les clients

Assurez-vous que vos clients sont en mesure de monter le service Azure HPC Cache en suivant les instructions de cette section.

### <a name="provide-network-access"></a>Fournir un accès réseau

Les ordinateurs clients doivent disposer d’un accès réseau au réseau virtuel et au sous-réseau privé du cache.

Par exemple, créez des machines virtuelles clientes dans le même réseau virtuel, ou utilisez un point de terminaison, une passerelle ou une autre solution dans le réseau virtuel pour permettre un accès en dehors du réseau. (n’oubliez pas que rien d’autre que le cache proprement dit ne doit être hébergé dans le sous-réseau du cache).

### <a name="install-utilities"></a>Installer les utilitaires

Installez le logiciel de l’utilitaire Linux approprié pour prendre en charge la commande de montage NFS :

* Pour Red Hat Enterprise Linux ou SuSE : `sudo yum install -y nfs-utils`
* Pour Ubuntu ou Debian : `sudo apt-get install nfs-common`

### <a name="create-a-local-path"></a>Créer un chemin d’accès local

Créez un chemin d’accès de répertoire local sur chaque client pour la connexion au cache. Créez un chemin d’accès pour chaque chemin d’espace de noms que vous souhaitez monter.

Exemple : `sudo mkdir -p /mnt/hpc-cache-1/target3`

La page [Instructions de montage](#use-the-mount-instructions-utility) dans le Portail Azure contient un prototype de commande que vous pouvez copier.

Lorsque vous connectez l’ordinateur client au cache, vous associez ce chemin à un chemin d’espace de noms virtuel qui représente une exportation de cible de stockage. Créez des répertoires pour chaque chemin d’espace de noms virtuel que le client utilisera.

## <a name="use-the-mount-instructions-utility"></a>Utiliser l’utilitaire d’instructions de montage

Vous pouvez utiliser la page **Instructions de montage** dans le Portail Azure pour créer une commande de montage pouvant être copiée. Ouvrez la page à partir de la section **Configurer** de l’affichage du cache dans le portail.

Avant d’utiliser la commande sur un client, assurez-vous que le client remplit les conditions préalables et dispose du logiciel nécessaire pour utiliser la commande NFS `mount`, comme décrit ci-dessus dans [Préparer les clients](#prepare-clients).

![Capture d’écran d’une instance Azure HPC Cache dans le portail, avec la page Configurer > Instructions de montage chargée](media/mount-instructions.png)

Procédez comme suit pour créer la commande de montage.

1. Personnalisez le champ **Chemin d’accès client**. Ce champ fournit un exemple de commande que vous pouvez utiliser pour créer un chemin d’accès local sur le client. Le client accède au contenu à partir d’Azure HPC Cache localement dans ce répertoire.

   Cliquez sur le champ et modifiez la commande pour qu’elle contienne le nom du répertoire de votre choix. Le nom apparaît à la fin de la chaîne après `sudo mkdir -p`.

   ![capture d’écran du champ Chemin d’accès client avec le curseur positionné à la fin](media/mount-edit-client.png)

   Une fois que vous avez fini de modifier le champ, la commande de montage en bas de la page est mise à jour avec le nouveau chemin d’accès client.

1. Choisissez l’**adresse de montage du cache** dans la liste. Ce menu répertorie tous les [points de montage du client](#find-mount-command-components) du cache.

   Équilibrez la charge du client sur toutes les adresses de montage disponibles afin d’améliorer les performances du cache.

   ![capture d’écran du champ Adresse de montage du cache avec le sélecteur montrant trois adresses IP parmi lesquelles choisir](media/mount-select-ip.png)

1. Choisissez le **chemin d’espace de noms virtuel** à utiliser pour le client. Ces chemins d’accès sont liés aux exportations sur le système de stockage back-end.

   ![Capture d’écran montrant le champ « Chemin d’espace de noms virtuel » avec le sélecteur ouvert.](media/mount-select-target.png)

   Vous pouvez afficher et modifier les chemins d’espaces de noms virtuels sur la page **Espace de noms** du portail. Pour savoir comment procéder, consultez [Configurer l’espace de noms agrégé](add-namespace-paths.md).

   Pour en savoir plus sur la fonctionnalité d’espace de noms agrégé d’Azure HPC Cache, lisez [Planifier l’espace de noms agrégé](hpc-cache-namespace.md).

1. Le champ **Commande de montage** de l’étape 3 est automatiquement renseigné avec une commande de montage personnalisée qui utilise l’adresse de montage, le chemin d’espace de noms virtuel et le chemin d’accès client que vous avez définis dans les champs précédents.

   Cliquez sur le symbole de copie à droite du champ pour copier automatiquement le contenu de celui-ci dans le Presse-papiers.

   ![Capture d’écran du champ Commande de montage du prototype, montrant le texte du survol du bouton « Copier dans le Presse-papiers »](media/mount-command-copy.png)

1. Utilisez la commande de montage copiée sur l’ordinateur client pour le connecter à Azure HPC Cache. Vous pouvez émettre la commande directement à partir de la ligne de commande du client ou inclure la commande de montage dans un modèle ou un script d’installation de client.

## <a name="understand-mount-command-syntax"></a>Comprendre la syntaxe de la commande de montage

La commande de montage (mount) se présente sous la forme suivante :

> sudo mount {*options*} *cache_mount_address*:/*namespace_path* *local_path*

Exemple :

```bash
root@test-client:/tmp# mkdir hpccache
root@test-client:/tmp# sudo mount -o hard,proto=tcp,mountproto=tcp,retry=30 10.0.0.28:/blob-demo-0722 hpccache
root@test-client:/tmp#
```

Si l’exécution de cette commande réussit, le contenu de l’exportation de stockage doit être visible dans le répertoire ``hpccache`` sur le client.

### <a name="mount-command-options"></a>Options de la commande mount

Pour garantir un montage robuste du client, passez les paramètres et arguments suivants dans votre commande mount :

> mount -o hard,proto=tcp,mountproto=tcp,retry=30 ${CACHE_IP_ADDRESS}:/${NAMESPACE_PATH} ${LOCAL_FILESYSTEM_MOUNT_POINT}

| Paramètres recommandés pour la commande mount | Description |
--- | ---
``hard`` | Les montages conditionnels sur un cache Azure HPC Cache sont associés à des échecs d’application et à une perte possible de données.
``proto=tcp`` | Cette option prend en charge la gestion appropriée des erreurs réseau NFS.
``mountproto=tcp`` | Cette option prend en charge la gestion appropriée des erreurs réseau pour les opérations de montage.
``retry=<value>`` | Définissez ``retry=30`` pour éviter les échecs de montage temporaires. (Une valeur différente est recommandée dans les montages de premier plan.)

### <a name="find-mount-command-components"></a>Rechercher les composants de la commande de montage

Si vous souhaitez créer une commande de montage sans utiliser la page **Instructions de montage**, vous pouvez trouver les adresses de montage dans la page **Vue d’ensemble** du cache, et les chemins d’espaces de noms virtuels sur la page **Espace de noms**.

![Capture d’écran de la page Vue d’ensemble de l’instance Azure HPC Cache, avec la liste des adresses de montage mise en surbrillance en bas à droite](media/hpc-cache-mount-addresses.png)

> [!NOTE]
> Les adresses de montage du cache correspondent aux interfaces réseau du sous-réseau du cache. Dans un groupe de ressources, ces cartes réseau sont répertoriées avec des noms se terminant par `-cluster-nic-` et un nombre. Vous ne devez ni modifier ni supprimer ces interfaces, car cela rendrait le cache indisponible.

Les chemins d’espaces de noms virtuels sont affichés dans la page des paramètres **Espace de noms** du cache.

![Capture d’écran de la page du portail Paramètres > Espace de noms avec un encadré autour de la première colonne du tableau : « Chemin de l’espace de noms »](media/view-namespace-paths.png)

## <a name="next-steps"></a>Étapes suivantes

* Pour déplacer des données vers les cibles de stockage du cache, consultez [Remplir de données un nouveau stockage Blob Azure](hpc-cache-ingest.md).

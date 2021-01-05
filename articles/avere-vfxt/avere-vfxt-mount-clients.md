---
title: Monter le cluster Avere vFXT - Azure
description: Découvrez comment connecter des clients à votre cluster vFXT dans Average vFXT pour Azure et comment équilibrer la charge du trafic client entre les nœuds de votre cluster.
author: ekpgh
ms.service: avere-vfxt
ms.topic: how-to
ms.date: 12/16/2019
ms.author: rohogue
ms.openlocfilehash: 44a4e1293bc4c5a54e1e345d5cf95ba307a7b120
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88272569"
---
# <a name="mount-the-avere-vfxt-cluster"></a>Monter le cluster Avere vFXT

Suivez ces instructions pour connecter des ordinateurs clients à votre cluster vFXT.

1. Décidez comment équilibrer la charge du trafic client entre vos nœuds de cluster. Pour plus d’informations, lisez [Équilibrer la charge du client](#balance-client-load), ci-dessous.
1. Identifiez l'adresse IP et le chemin de jonction sur lesquels effectuer le montage.
1. Émettez la [commande mount](#mount-command-arguments) avec les arguments appropriés.

## <a name="balance-client-load"></a>Équilibrer la charge du client

Pour équilibrer les requêtes du client entre tous les nœuds du cluster, vous devez montez les clients sur la plage complète d’adresses IP côté client. Il existe plusieurs façons simples d’automatiser cette tâche.

> [!TIP]
> D’autres méthodes d’équilibrage de charge peuvent convenir pour les systèmes complexes ou de grande taille. ([Ouvrez un ticket de support](avere-vfxt-open-ticket.md#open-a-support-ticket-for-your-avere-vfxt) pour obtenir de l’aide.)
>
> Si vous préférez utiliser un serveur DNS pour l’équilibrage de charge automatique côté serveur, vous devez configurer et gérer votre propre serveur DNS dans Azure. Dans ce cas, vous pouvez configurer le système DNS de type tourniquet (round-robin) pour le cluster vFXT conformément à ce document : [Configuration DNS de cluster Avere](avere-vfxt-configure-dns.md).

### <a name="sample-balanced-client-mounting-script"></a>Exemple de script de montage de client à charge équilibrée

Cet exemple de code utilise des adresses IP clientes comme élément aléatoire pour distribuer des clients à toutes les adresses IP disponibles du cluster vFXT.

```bash
function mount_round_robin() {
    # to ensure the nodes are spread out somewhat evenly the default
    # mount point is based on this node's IP octet4 % vFXT node count.
    declare -a AVEREVFXT_NODES="($(echo ${NFS_IP_CSV} | sed "s/,/ /g"))"
    OCTET4=$((`hostname -i | sed -e 's/^.*\.\([0-9]*\)/\1/'`))
    DEFAULT_MOUNT_INDEX=$((${OCTET4} % ${#AVEREVFXT_NODES[@]}))
    ROUND_ROBIN_IP=${AVEREVFXT_NODES[${DEFAULT_MOUNT_INDEX}]}

    DEFAULT_MOUNT_POINT="${BASE_DIR}/default"

    # no need to write again if it is already there
    if ! grep --quiet "${DEFAULT_MOUNT_POINT}" /etc/fstab; then
        echo "${ROUND_ROBIN_IP}:${NFS_PATH}    ${DEFAULT_MOUNT_POINT}    nfs hard,proto=tcp,mountproto=tcp,retry=30 0 0" >> /etc/fstab
        mkdir -p "${DEFAULT_MOUNT_POINT}"
        chown nfsnobody:nfsnobody "${DEFAULT_MOUNT_POINT}"
    fi
    if ! grep -qs "${DEFAULT_MOUNT_POINT} " /proc/mounts; then
        retrycmd_if_failure 12 20 mount "${DEFAULT_MOUNT_POINT}" || exit 1
    fi
}
```

La fonction ci-dessus fait partie de l’exemple Batch disponible sur le site [Exemples Avere vFXT](https://github.com/Azure/Avere#tutorials).

## <a name="create-the-mount-command"></a>Créer la commande mount

> [!NOTE]
> Si vous n’avez pas créé de conteneur d’objets blob lors de la création de votre cluster Avere vFXT, avant de tenter de monter des clients, ajoutez des systèmes de stockages en procédant de la manière décrite dans [Configurer le stockage](avere-vfxt-add-storage.md).

À partir de votre client, la commande ``mount`` mappe le serveur virtuel (vserver) sur le cluster vFXT à un chemin sur le système de fichiers local. Le format est ``mount <vFXT path> <local path> {options}``

La commande Mount comporte trois éléments :

* chemin vFXT : combinaison d’une adresse IP et d’un chemin de jonction d’espaces de noms sur le cluster (décrite ci-dessous)
* chemin local : chemin sur le client
* options de la commande mount : répertoriées dans [Arguments de la commande mount](#mount-command-arguments)

### <a name="junction-and-ip"></a>Jonction et adresse IP

Le chemin du serveur virtuel (vserver) est une combinaison de son *adresse IP* et du chemin à une *jonction d’espace de noms*. La jonction d’espace de noms est un chemin virtuel qui a été défini lors de l’ajout du système de stockage.

Si votre cluster a été créé avec un Stockage Blob, le chemin de l’espace de noms vers ce conteneur est `/msazure`

Exemple : ``mount 10.0.0.12:/msazure /mnt/vfxt``

Si vous avez ajouté un stockage après avoir créé le cluster, le chemin de la jonction d’espace de noms est la valeur définie dans **Chemin de l’espace de noms** lors de la création de la jonction. Par exemple, si vous avez utilisé ``/avere/files`` comme chemin de votre espace de noms, vos clients montent *adresse_IP*:/avere/files sur leur point de montage local.

![Boîte de dialogue « Ajouter une jonction » avec /avere/files dans le champ de chemin d’espace de noms](media/avere-vfxt-create-junction-example.png) <!-- to do - change example and screenshot to vfxt/files instead of avere -->

L’adresse IP est l’une des adresses IP côté client définies pour le serveur virtuel (vserver). La plage d’adresses IP côté client est disponible à deux emplacements dans le Panneau de configuration Avere :

* Table **VServers** (onglet Tableau de bord) :

  ![Onglet Tableau de bord du Panneau de configuration Avere avec l’onglet VServers sélectionné dans la table de données sous le graphe et la section des adresses IP encadrée](media/avere-vfxt-ip-addresses-dashboard.png)

* Page de paramètres **Réseau côté client** :

  ![Page de configuration Paramètres > VServer > Réseau côté client avec un cadre autour de la section Plage d’adresses de la table pour un serveur virtuel (vserver) particulier](media/avere-vfxt-ip-addresses-settings.png)

Outre les chemins, vous devez inclure les [arguments de la commande mount](#mount-command-arguments) décrits ci-dessous lors du montage de chaque client.

### <a name="mount-command-arguments"></a>Arguments de la commande mount

Pour garantir un montage sans interruption du client, passez les paramètres et arguments suivants dans votre commande mount :

``mount -o hard,proto=tcp,mountproto=tcp,retry=30 ${VSERVER_IP_ADDRESS}:/${NAMESPACE_PATH} ${LOCAL_FILESYSTEM_MOUNT_POINT}``

| Paramètres obligatoires | Description |
--- | ---
``hard`` | Les montages conditionnels sur le cluster vFXT sont associés à des échecs d’application et à des pertes de données possibles.
``proto=netid`` | Cette option prend en charge la gestion appropriée des erreurs réseau NFS.
``mountproto=netid`` | Cette option prend en charge la gestion appropriée des erreurs réseau pour les opérations de montage.
``retry=n`` | Définissez ``retry=30`` pour éviter les échecs de montage temporaires. (Une valeur différente est recommandée dans les montages de premier plan.)

## <a name="next-steps"></a>Étapes suivantes

Une fois les clients montés, vous pouvez les utiliser pour copier des données vers un nouveau conteneur de stockage d’objets blob sur votre cluster. Si vous n’avez pas besoin de remplir un nouveau stockage, suivez les autres liens pour en savoir plus sur les tâches de configuration supplémentaires :

* [Déplacer des données vers un système de stockage principal du cluster](avere-vfxt-data-ingest.md) : guide pratique pour utiliser plusieurs clients et threads afin de charger efficacement vos données dans un nouveau un système de stockage principal
* [Personnaliser le paramétrage du cluster](avere-vfxt-tuning.md) : personnalisez les paramètres du cluster pour les adapter à votre charge de travail
* [Gérer le cluster](avere-vfxt-manage-cluster.md) : guide pratique pour démarrer ou arrêter le cluster et gérer les nœuds

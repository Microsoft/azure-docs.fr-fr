---
title: Gérer le cluster Avere vFXT - Azure
description: Découvrez comment gérer le cluster Avere vFXT, notamment comment le redémarrer, l’arrêter ou le détruire et comment y ajouter ou supprimer des nœuds
author: ekpgh
ms.service: avere-vfxt
ms.topic: how-to
ms.date: 01/13/2020
ms.author: rohogue
ms.openlocfilehash: 4135bfe528c33a2beaeb21438181deb5b19ad12e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85505492"
---
# <a name="manage-the-avere-vfxt-cluster"></a>Gérer le cluster Avere vFXT

À un moment donné du cycle de vie de votre cluster Avere vFXT pour Azure, il peut être nécessaire d’ajouter des nœuds de cluster, ou de démarrer ou redémarrer le cluster. Une fois votre projet terminé, vous devez savoir comment arrêter le cluster et comment le supprimer de façon définitive.

Cet article explique comment ajouter ou supprimer des nœuds de cluster, et d’autres opérations de base sur le cluster. Si vous avez besoin de changer les paramètres du cluster ou de superviser son fonctionnement, utilisez le [Panneau de configuration Avere](avere-vfxt-cluster-gui.md).

Selon la tâche de gestion à effectuer, vous pouvez être amené à utiliser un des trois outils suivants : le panneau de configuration Avere, le script de gestion de cluster de la ligne de commande vfxt.py et le portail Azure.

Ce tableau récapitule les outils disponibles pour les différentes tâches.

| Action | Panneau de configuration Avere | vfxt.py  | Portail Azure |
| --- | --- | --- | --- |
| Ajouter des nœuds au cluster | non | Oui | non |
| Supprimer des nœuds du cluster | Oui | non | non |
| Arrêter un nœud du cluster | Oui (également pour redémarrer des services ou le cluster entier) | non | La mise hors tension d’une machine virtuelle d’un nœud à partir du portail est interprétée comme une défaillance de nœud |
| Démarrer un nœud arrêté | non | non | Oui |
| Détruire un seul nœud du cluster | non | non | Oui |
| Redémarrer le cluster entier |  |  |  |
| Fermer ou arrêter le cluster en toute sécurité | Oui | Oui | non |
| Détruire le cluster  | non | Oui | Oui, mais sans garantie de l’intégrité des données |

Vous trouverez ci-dessous des instructions détaillées pour chaque outil.

## <a name="about-stopped-instances-in-azure"></a>Informations sur les instances arrêtées dans Azure

Quand vous fermez ou arrêtez une machine virtuelle Azure, les frais de calcul ne vous sont plus facturés, mais vous continuez à payer le stockage pour cette machine virtuelle. Si vous fermez définitivement un nœud vFXT ou le cluster vFXT entier, vous devez supprimer les machines virtuelles associées à l’aide du portail Azure.

Dans le portail Azure, un nœud *arrêté* (qui peut être redémarré) apparaît avec l’état **Arrêté** dans le portail Azure. Un nœud *supprimé* apparaît avec l’état **Arrêté (désalloué)** , et n’entraîne plus de frais de calcul ou de stockage.

Avant de supprimer une machine virtuelle, assurez-vous que toutes les données modifiées du cache ont été écrites dans un stockage back-end. Utilisez le Panneau de configuration Avere ou le script vfxt.py pour arrêter ou fermer le cluster.

## <a name="manage-the-cluster-with-avere-control-panel"></a>Gérer le cluster à l’aide du Panneau de configuration Avere

Vous pouvez utiliser le Panneau de configuration Avere pour effectuer les tâches suivantes :

* Arrêter ou redémarrer des nœuds individuels
* Supprimer un nœud du cluster
* Arrêter ou redémarrer le cluster entier

Le Panneau de configuration Avere donne la priorité à l’intégrité des données : il tente donc d’écrire les données modifiées dans le stockage back-end avant une opération destructive potentielle. Ceci en fait une option plus sûre que le portail Azure.

Accédez au Panneau de configuration Avere à partir d’un navigateur web. Au besoin, aidez-vous des instructions fournies dans [Accéder au cluster vFXT](avere-vfxt-cluster-gui.md).

### <a name="manage-nodes-with-avere-control-panel"></a>Gérer des nœuds à l’aide du Panneau de configuration Avere

La page de paramètres **FXT Nodes** (Nœuds FXT) contient plusieurs commandes permettant de gérer les nœuds de manière individuelle.

Pour fermer, redémarrer ou supprimer un nœud, recherchez le nœud dans la liste de la page **FXT Nodes**, puis cliquez sur le bouton approprié dans la colonne **Actions** du nœud.

> [!NOTE]
> Il est possible que les adresses IP soient déplacées entre les nœuds de cluster lorsque le nombre de nœuds actifs change.

Pour plus d’informations, consultez [Cluster > FXT Nodes](<https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gui_fxt_nodes.html#gui-fxt-nodes>) dans le guide des paramètres du cluster Avere.

### <a name="stop-or-reboot-the-cluster-with-avere-control-panel"></a>Arrêter ou redémarrer le cluster à l’aide du Panneau de configuration Avere

La page de paramètres **System Maintenance** (Maintenance système) comporte des commandes permettant de redémarrer des services du cluster, de redémarrer le cluster entier ou de mettre le cluster hors tension en toute sécurité. Pour plus d’informations, consultez [Administration > System Maintenance](<https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gui_system_maintenance.html#gui-system-maintenance>) dans le guide des paramètres du cluster Avere.

Quand un cluster commence son processus d’arrêt, il publie des messages d’état sous l’onglet **Tableau de bord**. Après quelques instants, les messages cessent et la session du Panneau de configuration Avere cesse finalement de répondre, ce qui signifie que le cluster est fermé.

## <a name="manage-the-cluster-with-vfxtpy"></a>Gérer le cluster à l’aide du script vfxt.py

vfxt.py est un outil en ligne de commande conçu pour la création et la gestion de clusters.

vfxt.py est préinstallé sur la machine virtuelle du contrôleur de cluster. Si vous souhaitez l'installer sur un autre système, reportez-vous à la documentation disponible à l'adresse <https://github.com/Azure/AvereSDK>.

Vous pouvez utiliser le script vfxt.py pour effectuer les tâches de gestion du cluster suivantes :

* Ajouter de nouveaux nœuds au cluster
* Arrêter ou démarrer le cluster  
* Détruire le cluster

Comme le Panneau de configuration Avere, vfxt.py tente de s’assurer que les données modifiées sont enregistrées dans le stockage back-end permanent avant de fermer ou détruire le cluster ou le nœud. Ceci en fait une option plus sûre que le portail Azure.

Un guide d’utilisation de vfxt.py complet est disponible sur GitHub : [Gestion des clusters cloud avec vfxt.py](https://github.com/azure/averesdk/blob/master/docs/README.md)

### <a name="add-cluster-nodes-with-vfxtpy"></a>Ajouter des nœuds au cluster avec vfxt.py

Le contrôleur de cluster fournit un exemple de script de commande pour ajouter des nœuds à un cluster. Recherchez ``./add-nodes`` sur le contrôleur et ouvrez-le dans un éditeur afin de le personnaliser avec les informations propres à votre cluster.

Le cluster doit avoir été démarré pour utiliser cette commande.

Indiquez les valeurs suivantes :

* Nom du groupe de ressources pour le cluster, mais aussi pour les ressources réseau et de stockage si elles ne sont pas dans le même groupe de ressources que le cluster
* Emplacement du cluster
* Réseau et sous-réseau du cluster
* Rôle d'accès aux nœuds du cluster (utiliser le rôle intégré [Opérateur Avere](../role-based-access-control/built-in-roles.md#avere-operator))
* Adresse IP de gestion et mot de passe d’administration du cluster
* Nombre de nœuds à ajouter (1, 2 ou 3)
* Type d’instance et taille du cache pour chaque nœud

Si vous n’utilisez pas le modèle fourni, créez une commande similaire à l’exemple ci-dessous, en incluant toutes les informations nécessaires décrites ci-dessus.

```bash
   vfxt.py --cloud-type azure --from-environment \
   --resource-group GROUP_NAME \
   [--network-resource-group GROUP_NAME --storage-resource-group GROUP_NAME]  \
   --location LOCATION --azure-network NETWORK_NAME --azure-subnet SUBNET_NAME \
   --add-nodes --nodes NODE_COUNT \
   --management-address CLUSTER_IP --admin-password ADMIN_PASSWORD \
   --instance-type TYPE --node-cache-size SIZE \
   --azure-role "Avere Operator" \
   --log ~/vfxt.log
```

Pour plus d’informations, consultez [Ajouter des nœuds à un cluster](https://github.com/Azure/AvereSDK/blob/master/docs/using_vfxt_py.md#add-nodes-to-a-cluster) dans le guide d’utilisation de vfxt.py.

### <a name="stop-a-cluster-with-vfxtpy"></a>Arrêter un cluster avec vfxt.py

```bash
vfxt.py --cloud-type azure --from-environment --stop --resource-group GROUPNAME --admin-password PASSWORD --management-address ADMIN_IP --location LOCATION --azure-network NETWORK --azure-subnet SUBNET
```

### <a name="start-a-stopped-cluster-with-vfxtpy"></a>Démarrer un cluster arrêté avec vfxt.py

```bash
vfxt.py --cloud-type azure --from-environment --start --resource-group GROUPNAME --admin-password PASSWORD --management-address ADMIN_IP --location LOCATION --azure-network NETWORK --azure-subnet SUBNET --instances INSTANCE1_ID INSTANCE2_ID INSTANCE3_ID ...
```

Étant donné que le cluster est arrêté, vous devez passer les identificateurs d’instance pour spécifier les nœuds de cluster. Pour en savoir plus, consultez [Spécifier le cluster à modifier](https://github.com/Azure/AvereSDK/blob/master/docs/using_vfxt_py.md#specifying-which-cluster-to-modify) dans le guide d’utilisation de vfxt.py.

### <a name="destroy-a-cluster-with-vfxtpy"></a>Détruire un cluster avec vfxt.py

```bash
vfxt.py --cloud-type azure --from-environment --destroy --resource-group GROUPNAME --admin-password PASSWORD --management-address ADMIN_IP --location LOCATION --azure-network NETWORK --azure-subnet SUBNET --management-address ADMIN_IP
```

Vous pouvez utiliser l’option ``--quick-destroy`` si vous ne voulez pas enregistrer les données modifiées du cache du cluster.

Pour plus d’informations, consultez le [guide d’utilisation de vfxt.py](<https://github.com/Azure/AvereSDK/blob/master/docs/README.md>).

## <a name="manage-cluster-vms-from-the-azure-portal"></a>Gérer des machines virtuelles du cluster à partir du portail Azure

Il est possible de détruire individuellement des machines virtuelles du cluster par le biais du portail Azure, mais l’intégrité des données n’est pas garantie si le cluster n’est pas préalablement fermé en toute sécurité.

Vous pouvez utiliser le portail Azure pour effectuer les tâches de gestion du cluster suivantes :

* Démarrer un nœud vFXT arrêté
* Arrêter un nœud vFXT individuel (le cluster interprète cette action comme une défaillance de nœud)
* Détruire un cluster vFXT *si* vous n’avez pas besoin de vous assurer que les données modifiées dans le cache du cluster sont écrites dans le système de stockage principal (core filer)
* Supprimer définitivement des nœuds vFXT et d’autres ressources du cluster une fois qu’ils ont été fermés en toute sécurité

### <a name="restart-vfxt-instances-from-the-azure-portal"></a>Redémarrer des instances de vFXT à partir du portail Azure

Si vous devez redémarrer un nœud arrêté, utilisez le portail Azure. Sélectionnez **Machines virtuelles** dans le menu de gauche, puis cliquez sur le nom de la machine virtuelle dans la liste pour ouvrir sa page de présentation.

Cliquez sur le bouton **Démarrer** en haut de la page de présentation pour réactiver la machine virtuelle.

![Capture d’écran du portail Azure montrant l’option pour démarrer une machine virtuelle arrêtée](media/avere-vfxt-start-stopped-incurring-annot.png)

### <a name="delete-cluster-nodes"></a>Supprimer des nœuds du cluster

Si vous souhaitez supprimer un seul nœud du cluster vFXT et conserver les autres composants du cluster, vous devez d’abord [supprimer le nœud du cluster](#manage-nodes-with-avere-control-panel) à l’aide du Panneau de configuration Avere.

> [!CAUTION]
> Si vous supprimez un nœud sans l’avoir préalablement supprimé du cluster vFXT, vous risquez de perdre des données.

Pour détruire définitivement une ou plusieurs instances utilisées en tant que nœud vFXT, utilisez le portail Azure.
Sélectionnez **Machines virtuelles** dans le menu de gauche, puis cliquez sur le nom de la machine virtuelle dans la liste pour ouvrir sa page de présentation.

Cliquez sur le bouton **Supprimer** en haut de la page de présentation pour détruire définitivement la machine virtuelle.

Vous pouvez utiliser cette méthode pour supprimer définitivement des nœuds du cluster qui ont été préalablement fermés en toute sécurité.

### <a name="destroy-the-cluster-from-the-azure-portal"></a>Détruire le cluster à partir du portail Azure

> [!NOTE]
> Si vous souhaitez que les modifications clientes restantes dans le cache soient écrites dans le stockage back-end, utilisez soit l’option `--destroy` de vfxt.py, soit le Panneau de configuration Avere pour fermer le cluster en toute sécurité avant de supprimer les instances de nœud dans le portail Azure.

Pour détruire définitivement des instances de nœud, supprimez-les dans le portail Azure. Vous pouvez supprimer les instances de façon individuelle, comme décrit ci-dessus, ou les supprimer toutes en même temps. Pour effectuer une suppression en bloc, accédez à la page **Machines virtuelles**, recherchez toutes les machines virtuelles du cluster, sélectionnez-les en les cochant et cliquez sur le bouton **Supprimer**.

![Liste de machines virtuelles dans le portail, filtrées sur le terme « cluster », où trois des quatre machines virtuelles sont cochées et mises en surbrillance](media/avere-vfxt-multi-vm-delete.png)

### <a name="delete-additional-cluster-resources-from-the-azure-portal"></a>Supprimer des ressources de cluster supplémentaires à partir du portail Azure

Si vous avez créé des ressources supplémentaires spécifiquement pour le cluster vFXT, vous souhaiterez peut-être les supprimer en même temps que la destruction du cluster. Ne détruisez pas les éléments contenant des données dont vous avez encore besoin, ni les éléments partagés avec d'autres projets.

Outre la suppression des nœuds du cluster, vous pouvez supprimer ces composants :

* La machine virtuelle du contrôleur de cluster
* Les disques de données associés aux nœuds du cluster
* Les interfaces réseau et adresses IP publiques associées aux composants du cluster
* Réseaux virtuels
* Conteneurs de stockage et comptes de stockage (**seulement** s’ils ne contiennent pas de données importantes)
* Groupe à haute disponibilité

![Liste de toutes les ressources dans le portail Azure, répertoriant les ressources créées pour un cluster de test](media/avere-vfxt-all-resources-list.png)

### <a name="delete-a-clusters-resource-group-from-the-azure-portal"></a>Supprimer un groupe de ressources du cluster à partir du portail Azure

Si vous avez créé un groupe de ressources spécifiquement pour héberger le cluster, vous pouvez détruire toutes les ressources associées au cluster en détruisant le groupe de ressources.

> [!Caution]
> Détruisez le groupe de ressources uniquement si vous êtes certain qu’il ne contient pas de données importantes. Par exemple, assurez-vous que vous avez déplacé les données nécessaires se trouvant dans les conteneurs de stockage dans le groupe de ressources.  

Pour supprimer un groupe de ressources, cliquez sur **Groupes de ressources** dans le menu de gauche du portail, puis filtrez la liste des groupes de ressources afin de rechercher celui que vous avez créé pour le cluster vFXT. Sélectionnez le groupe de ressources et cliquez sur les points de suspension à droite du panneau. Choisissez **Supprimer un groupe de ressources**. Le portail vous invite à confirmer la suppression, qui est irréversible.

![Groupe de ressources affichant l’action « Supprimer un groupe de ressources »](media/avere-vfxt-delete-resource-group.png)

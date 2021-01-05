---
title: Planification de votre système Avere vFXT - Azure
description: Planifiez un cluster Avere vFXT pour Azure adapté à vos besoins. Découvrez les questions à poser avant d’accéder à la Place de marché Azure ou de créer des machines virtuelles.
author: ekpgh
ms.service: avere-vfxt
ms.topic: conceptual
ms.date: 01/21/2020
ms.author: rohogue
ms.openlocfilehash: 9f852ec056f3252005bba0bc142c05a9cfbe6c20
ms.sourcegitcommit: f88074c00f13bcb52eaa5416c61adc1259826ce7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92342397"
---
# <a name="plan-your-avere-vfxt-system"></a>Planifier votre système Avere vFXT

Cet article explique comment planifier un nouveau cluster vFXT Avere pour Azure dont la position et la taille répondent à vos besoins.

Avant d’accéder à la Place de marché Azure ou de créer des machines virtuelles, prenez-en compte les aspects suivants :

* Comment le cluster interagit-il avec d’autres ressources Azure ?
* Où les éléments de cluster doivent-ils être situés dans des réseaux privés et des sous-réseaux ?
* Quel type de stockage back-end allez-vous utiliser et comment le cluster y accèdera-t-il ?
* Quelle doit être la puissance de vos nœuds de cluster pour prendre en charge votre workflow ?

Lisez la suite pour en savoir plus.

## <a name="learn-the-components-of-the-system"></a>En savoir plus sur les composants du système

Il peut être utile de comprendre les composants du système Avere vFXT pour Azure lorsque vous commencez à planifier.

* Nœuds de cluster : le cluster est constitué de trois machines virtuelles ou plus configurées en tant que nœuds de cluster. Un plus grand nombre de nœuds donne au système un débit supérieur et un cache plus important.

* Cache : la capacité du cache est répartie de manière égale entre les nœuds du cluster. Définissez la taille du cache par nœud lorsque vous créez le cluster ; les tailles de nœuds sont ajoutées pour former la taille totale du cache.

* Contrôleur de cluster : le contrôleur de cluster est une machine virtuelle supplémentaire située dans le même sous-réseau que les nœuds de cluster. Le contrôleur est nécessaire pour créer le cluster et pour les tâches de gestion courantes.

* Stockage principal : les données que vous souhaitez mettre en cache sont stockées à long terme dans un système de stockage matériel ou conteneur d’objets Blob Azure. Vous pouvez ajouter du stockage après avoir créé le cluster Avere vFXT pour Azure, ou si vous utilisez le stockage d’objets Blob, vous pouvez ajouter et configurer le conteneur lors de la création du cluster.

* Clients : les machines clientes qui utilisent les fichiers mis en cache se connectent au cluster à l’aide d’un chemin d’accès de fichier virtuel au lieu d’accéder directement aux systèmes de stockage. (Pour plus d’informations, consultez [Monter le cluster Avere vFXT](avere-vfxt-mount-clients.md).)

## <a name="subscription-resource-group-and-network-infrastructure"></a>Abonnement, groupe de ressources et infrastructure réseau

Pensez à l’emplacement des éléments de votre déploiement vFXT Avere pour Azure. Le diagramme suivant montre une disposition possible pour les composants Avere vFXT pour Azure :

![Diagramme montrant le contrôleur de cluster et les machines virtuelles de cluster dans un sous-réseau Les limites du sous-réseau sont entourées des limites d’un réseau virtuel. Dans le réseau virtuel, un hexagone représente le point de terminaison du service de stockage. Celui-ci est relié par une flèche pointillée à un stockage Blob situé en dehors du réseau virtuel.](media/avere-vfxt-components-option.png)

Suivez ces instructions pour planifier l’infrastructure réseau de votre cluster Avere vFXT :

* Créez un abonnement par Avere vFXT pour un déploiement Azure. Gérez tous les composants de cet abonnement.

  Les avantages de l’utilisation d’un nouvel abonnement pour chaque déploiement sont les suivants :
  * Simplification du suivi des coûts : visualisez et auditez tous les coûts des ressources, de l’infrastructure et des cycles de calcul dans un seul abonnement.
  * Simplification du nettoyage : vous pouvez supprimer l’ensemble de l’abonnement une fois que vous en avez terminé avec le projet.
  * Partitionnement approprié des quotas de ressources : isolez les clients et clusters Avere vFXT dans un seul abonnement pour protéger d’autres charges de travail critiques d’une possible limitation des ressources. Cela permet d’éviter des conflits lors de l’importation d’un grand nombre de clients pour un workflow de calcul haute performance.

* Recherchez les systèmes de calcul clients qui sont proches du cluster vFXT. Le stockage back-end, quant à lui, peut être plus distant.  

* Localisez le cluster vFXT et la machine virtuelle du contrôleur de cluster. Spécifiquement, ils doivent être :

  * Dans le même réseau virtuel
  * Dans le même groupe de ressources
  * À l’aide du même compte de stockage
  
  Le modèle de création de cluster gère cette configuration dans la plupart des situations.

* Le cluster doit se trouver dans son propre sous-réseau afin d’éviter tout conflit d’adresses IP avec des clients ou d’autres ressources de calcul.

* Utilisez le modèle de création de cluster pour créer la plupart des ressources d'infrastructure nécessaires au cluster, notamment les groupes de ressources, réseaux virtuels, sous-réseaux et comptes de stockage.

  Si vous souhaitez utiliser des ressources déjà existantes, assurez-vous qu’elles répondent aux exigences stipulées dans ce tableau.

  | Ressource | Utiliser l’existant ? | Spécifications |
  |----------|-----------|----------|
  | Resource group | Oui, si vide | Doit être vide|
  | Compte de stockage | **Oui**, en cas de connexion d'un conteneur d'objets blob existant après création du cluster <br/>  **Non**, en cas de création d'un conteneur d’objets blob lors de la création du cluster | Le conteneur d’objets blob existant doit être vide <br/> &nbsp; |
  | Réseau virtuel | Oui | Doit inclure un point de terminaison de service de stockage en cas de création d’un conteneur d’objets blob Azure |
  | Subnet | Oui | Ne peut pas contenir d’autres ressources |

## <a name="ip-address-requirements"></a>Exigences relatives aux adresses IP

Vérifiez que le sous-réseau de votre cluster dispose d’une plage d’adresses IP suffisamment étendue pour prendre en charge le cluster.

Le cluster Avere vFXT utilise les adresses IP suivantes :

* Une adresse IP de gestion de cluster. Cette adresse peut être déplacée de nœud à nœud dans le cluster en fonction des besoins, afin qu’elle soit toujours disponible. Utilisez cette adresse pour vous connecter à l’outil de configuration du panneau de configuration Avere.
* Pour chaque nœud du cluster :
  * Au moins une adresse IP exposée aux clients (toutes les adresses exposées aux clients sont gérées par le *vserver* du cluster, qui peut déplacer les adresses IP entre les nœuds en fonction des besoins).
  * Une adresse IP pour la communication de cluster
  * Une adresse IP d’instance (affectée à la machine virtuelle)

Si vous utilisez le stockage Blob Azure, les adresses IP du réseau virtuel de votre cluster peuvent également être nécessaires :  

* Un compte de stockage Blob Azure nécessite au moins cinq adresses IP. Gardez cette nécessité à l’esprit si vous placez le stockage Blob dans le même réseau virtuel que votre cluster.
* Si vous utilisez le stockage Blob Azure situé en dehors du réseau virtuel du cluster, créez un point de terminaison de service de stockage à l’intérieur du réseau virtuel. Le point de terminaison n’utilise pas d’adresse IP.

Vous avez la possibilité de placer les ressources réseau et le stockage Blob (si utilisé) dans des groupes de ressources différents du cluster.

## <a name="vfxt-node-size"></a>Taille des nœuds vFXT

Les machines virtuelles qui servent de nœuds de cluster déterminent le débit des requêtes et la capacité de stockage de votre cache. <!-- The instance type offered has been chosen for its memory, processor, and local storage characteristics. You can choose from two instance types, with different memory, processor, and local storage characteristics. -->

Chaque nœud vFXT est identique. Autrement dit, si vous créez un cluster à trois nœuds, vous aurez trois machines virtuelles de même type et de même taille.

| Type d’instance | Processeurs virtuels | Mémoire  | Stockage SSD local  | Disques de données max. | Débit du disque non mis en cache | Cartes réseau (nombre) |
| --- | --- | --- | --- | --- | --- | --- |
| Standard_E32s_v3 | 32  | 256 Gio | 512 Go  | 32 | 51 200 IOPS <br/> 768 Mbits/s | 16 000 Mbits/s (8)  |

La taille du cache de disque par nœud est configurable et peut aller de 1 000 Go à 8 000 Go. La taille du cache recommandée est de 4 To par nœud pour les nœuds Standard_E32s_v3.

Pour plus d’informations sur ces machines virtuelles, consultez la documentation Microsoft Azure : [Tailles de machine virtuelle à mémoire optimisée](../virtual-machines/sizes-memory.md)

## <a name="account-quota"></a>Quota des comptes

Vérifiez que votre abonnement a la capacité nécessaire pour exécuter le cluster Avere vFXT, ainsi que tous les systèmes clients et systèmes de calcul utilisés. Pour plus d’informations, lisez [Quota pour le cluster vFXT](avere-vfxt-prereqs.md#quota-for-the-vfxt-cluster).

## <a name="back-end-data-storage"></a>Stockage de données back-end

Les systèmes de stockage principaux fournissent des fichiers dans le cache du cluster et reçoivent également les données modifiées du cache. Décidez si votre plage de travail sera stockée à long terme dans un nouveau conteneur d’objets blob ou dans un système de stockage (cloud ou matériel) existant. Ces systèmes sont appelés *systèmes de stockage principaux*.

### <a name="hardware-core-filers"></a>Systèmes de stockage principaux matériels

Ajoutez des systèmes de stockage matériels au cluster vFXT après avoir créé le cluster. Vous pouvez utiliser divers systèmes matériels courants, dont des systèmes locaux, tant que le système de stockage est accessible à partir du sous-réseau du cluster.

Pour obtenir des instructions détaillées sur l’ajout d’un système de stockage existant au cluster vFXT Avere, lisez [Configurer le stockage](avere-vfxt-add-storage.md).

### <a name="cloud-core-filers"></a>Systèmes de stockage principaux cloud

Le système Avere vFXT pour Azure moyen peut utiliser des conteneurs d’objets Blob vides pour le stockage principal. Les conteneurs doivent être vides lorsqu’ils sont ajoutés au cluster : le système vFXT doit être en mesure de gérer son magasin d’objets sans qu’il soit nécessaire de conserver les données existantes.

> [!TIP]
> Si vous souhaitez utiliser le stockage Blob Azure pour le back-end, créez un conteneur dans le cadre de la création du cluster vFXT. Le modèle de création de cluster peut créer et configurer un nouveau conteneur d’objets BLOB afin qu’il soit prêt à être utilisé dès que le cluster est disponible. L’ajout d’un conteneur plus tard est plus compliqué.
>
> Pour plus d’informations, consultez [Créer le déploiement Avere vFXT pour Azure](avere-vfxt-deploy.md#create-the-avere-vfxt-for-azure).

Après avoir ajouté le conteneur de stockage d’objets Blob vide en tant que serveur de fichiers principal, vous pouvez y copier des données via le cluster. Utilisez un mécanisme de copie parallèle multithread. Pour savoir comment copier des données vers le conteneur du cluster à l’aide des ordinateurs clients et du cache Avere vFXT, lisez [Déplacement de données vers le cluster vFXT](avere-vfxt-data-ingest.md).

## <a name="cluster-access"></a>Accès au cluster

Le cluster Avere vFXT pour Azure se trouve sur un sous-réseau privé et n’a pas d’adresse IP publique. Vous devez disposer d’une méthode d’accès au sous-réseau privé pour l’administration du cluster et les connexions clientes.

Les options d’accès disponibles sont les suivantes :

* Hôte de saut : affectez une adresse IP publique à une autre machine virtuelle sur le réseau privé, puis utilisez-la pour créer un tunnel TLS sur les nœuds de cluster.

  > [!TIP]
  > Si vous définissez une adresse IP publique sur le contrôleur de cluster, vous pouvez l’utiliser en tant qu’hôte de saut. Pour plus d’informations, consultez [Contrôleur de cluster en tant qu’hôte de saut](#cluster-controller-as-jump-host).

* Réseau privé virtuel (VPN) : configurez un VPN point à site ou site à site entre votre réseau privé dans Azure et des réseaux d’entreprise.

* Azure ExpressRoute : configurez une connexion privée par le biais d’un partenaire ExpressRoute.

Pour plus d’informations sur ces options, lisez la [documentation des réseaux virtuels Azure sur la communication Internet](../virtual-network/virtual-networks-overview.md#communicate-with-the-internet).

### <a name="cluster-controller-as-jump-host"></a>Contrôleur de cluster en tant qu’hôte de saut

Si vous définissez une adresse IP publique sur le contrôleur de cluster, vous pouvez l’utiliser en tant qu’hôte de saut (« jump host ») pour contacter le cluster Avere vFXT à partir d’un emplacement en dehors du sous-réseau privé. Toutefois, ce contrôleur disposant de privilèges d’accès pour modifier des nœuds de cluster, cela crée un faible risque de sécurité.

Pour améliorer la sécurité d'un contrôleur avec adresse IP publique, le script de déploiement crée automatiquement un groupe de sécurité réseau qui limite l’accès entrant au port 22. Vous pouvez renforcer la protection du système en verrouillant l'accès à votre plage d'adresses IP sources, c'est-à-dire autoriser uniquement les connexions des machines que vous avez l'intention d'utiliser pour l'accès au cluster.

Quand vous créez le cluster, vous pouvez choisir s’il faut créer une adresse IP publique sur le contrôleur de cluster.

* Si vous créez un **réseau virtuel** ou un **sous-réseau**, une **adresse IP publique** est attribuée au contrôleur de cluster.
* Si vous sélectionnez un réseau virtuel et un sous-réseau existants, le contrôleur de cluster aura uniquement des adresses IP **privées**.

## <a name="vm-access-roles"></a>Rôles d’accès de machine virtuelle

Azure utilise le [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../role-based-access-control/index.yml) pour autoriser les machines virtuelles de cluster à effectuer certaines tâches. Par exemple, le contrôleur de cluster doit être autorisé à créer et configurer les machines virtuelles de nœud de cluster. Les nœuds de cluster doivent pouvoir affecter ou réaffecter des adresses IP à d’autres nœuds de cluster.

Deux rôles Azure intégrés sont utilisés pour les machines virtuelles Avere vFXT :

* Le contrôleur de cluster utilise le rôle intégré [Contributeur Avere](../role-based-access-control/built-in-roles.md#avere-contributor).
* Les nœuds de cluster utilisent le rôle intégré [Opérateur Avere](../role-based-access-control/built-in-roles.md#avere-operator).

S'il vous faut personnaliser les rôles d’accès pour les composants Avere vFXT, définissez votre propre rôle, puis attribuez-le aux machines virtuelles au moment de leur création. Vous ne pouvez pas utiliser le modèle de déploiement dans la Place de marché Azure. Contactez le Support technique et le Service clientèle Microsoft en ouvrant un ticket dans le portail Azure, comme décrit dans [Obtenir de l’aide avec votre système](avere-vfxt-open-ticket.md).

## <a name="next-steps"></a>Étapes suivantes

La [vue d’ensemble du déploiement](avere-vfxt-deploy-overview.md) donne une vision globale des étapes nécessaires pour créer un système Avere vFXT pour Azure, ainsi que pour le préparer à fournir des données.
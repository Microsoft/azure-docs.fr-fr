---
title: Planifier un déploiement de cluster Azure Service Fabric
description: Découvrez-en plus sur la planification et la préparation d'un déploiement de cluster de production Service Fabric sur Azure.
ms.topic: conceptual
ms.date: 03/20/2019
ms.openlocfilehash: 9de59811397eb47809c6d71f608e43beae5bfadb
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97109621"
---
# <a name="plan-and-prepare-for-a-cluster-deployment"></a>Planifier et préparer un déploiement de cluster

Planifier et préparer un déploiement de cluster de production est très important.  De nombreux facteurs sont à prendre en compte.  Cet article vous guide tout au long des étapes de préparation de déploiement de cluster.

## <a name="read-the-best-practices-information"></a>Lire les informations en matière de meilleures pratiques
Pour une bonne gestion des applications et clusters Azure Service Fabric, nous vous recommandons certaines opérations afin de renforcer la fiabilité de votre environnement de production.  Pour plus d'informations, consultez [Meilleures pratiques relatives aux applications et aux clusters Service Fabric](service-fabric-best-practices-overview.md).

## <a name="select-the-os-for-the-cluster"></a>Sélectionner le système d'exploitation pour le cluster
Service Fabric permet la création de clusters Service Fabric sur toute machine virtuelle ou tout ordinateur exécutant Windows Server ou Linux.  Avant de déployer votre cluster, vous devez choisir le système d’exploitation :  Windows ou Linux.  Chaque nœud (machine virtuelle) du cluster exécute le même système d’exploitation et vous ne pouvez pas combiner des machines virtuelles Linux et Windows au sein du même cluster.

## <a name="capacity-planning"></a>planification de la capacité
Pour un déploiement de production, la planification de la capacité est une étape importante. Voici quelques éléments à prendre en compte dans le cadre de ce processus.

* Nombre initial de types de nœuds pour votre cluster 
* Propriétés de chaque type de nœud (taille, nombre d'instances, principal ou non, accessibilité sur Internet, nombre de machines virtuelles, etc.)
* Caractéristiques de fiabilité et de durabilité du cluster

### <a name="select-the-initial-number-of-node-types"></a>Sélectionner le nombre initial de types de nœuds
Vous devez d’abord déterminer l’utilisation du cluster que vous créez. Quels types d’applications planifiez-vous de déployer dans ce cluster ? Votre application inclut-elle plusieurs services ? Si oui, ces services doivent-ils être publics ou accessibles sur Internet ? Vos services (qui composent votre application) ont-ils des besoins d’infrastructure différents tels qu’une RAM plus volumineuse ou des cycles processeur plus élevés ? Un cluster Service Fabric peut être constitué de plusieurs types de nœuds : un type de nœud principal et un ou plusieurs types de nœuds non principaux. Chaque type de nœud est mappé à un groupe de machines virtuelles identiques. Chaque type de nœud peut ensuite faire l’objet d’une montée ou descente en puissance de manière indépendante, avoir différents jeux de ports ouverts et présenter différentes métriques de capacité. Des [propriétés de nœud et contraintes de placement][placementconstraints] peuvent être configurées pour limiter des services spécifiques à des types de nœuds spécifiques.  Pour plus d’informations, consultez la [Planification de la capacité des clusters Service Fabric](service-fabric-cluster-capacity.md).

### <a name="select-node-properties-for-each-node-type"></a>Sélectionner les propriétés de nœud pour chaque type de nœud
Les types de nœuds définissent les références, le nombre et les propriétés des machines virtuelles du groupe identique correspondant.

La taille minimale des machines virtuelles pour chaque type de nœud est déterminée par le [niveau de durabilité][durability] que vous choisissez.

Le nombre minimal de machines virtuelles pour le type de nœud principal est déterminé par le [niveau de fiabilité][reliability] que vous choisissez.

Consultez les recommandations minimales en matière de [types de nœuds principaux](service-fabric-cluster-capacity.md#primary-node-type), [charges de travail avec état sur les types de nœuds non principaux](service-fabric-cluster-capacity.md#stateful-workloads) et [charges de travail sans état sur les types de nœuds non principaux](service-fabric-cluster-capacity.md#stateless-workloads).

Tout nombre supérieur au nombre minimal de nœuds doit dépendre du nombre de réplicas des applications/services que vous souhaitez exécuter dans ce type de nœud.  [Planifier la capacité pour les applications Service Fabric](service-fabric-capacity-planning.md) vous aide à estimer les ressources requises pour exécuter vos applications. Vous pourrez ensuite mettre à l'échelle le cluster pour l'ajuster à l'évolution de la charge de travail. 

#### <a name="use-ephemeral-os-disks-for-virtual-machine-scale-sets"></a>Disques de système d’exploitation éphémères pour groupes de machines virtuelles identiques

Les *disques de système d’exploitation éphémères* sont des dispositifs de stockage créés sur la machine virtuelle locale, qui ne sont pas enregistrés sur le Stockage Azure à distance. Ils sont recommandés pour tous les types de nœuds Service Fabric (principaux et secondaires) car, par rapport aux disques de système d’exploitation permanents traditionnels, les disques de système d’exploitation éphémères :

* réduisent la latence de lecture/écriture sur le disque du système d’exploitation ;
* permettent des opérations plus rapides de gestion de réinitialisation/réimagerie de nœuds ;
* réduisent les coûts globaux (les disques sont gratuits et n’impliquent aucun coût de stockage supplémentaire).

Les disques de système d’exploitation éphémères ne sont pas spécifiques de Service Fabric mais des *groupes de machines virtuelles identiques* Azure mappées à des types de nœuds Service Fabric. Pour les utiliser avec Service Fabric, vous devez disposer des éléments suivants dans votre modèle Azure Resource Manager de cluster :

1. Vérifiez que les types de nœuds spécifient les [tailles de machines virtuelles Azure prises en charge](../virtual-machines/ephemeral-os-disks.md) pour les disques de système d’exploitation éphémères, et que la taille de machine virtuelle offre une taille de cache suffisante pour prendre en charge la taille de son disque de système d’exploitation (voir *Note* ci-dessous). Par exemple :

    ```xml
    "vmNodeType1Size": {
        "type": "string",
        "defaultValue": "Standard_DS3_v2"
    ```

    > [!NOTE]
    > Veillez à sélectionner une taille de machine virtuelle avec une taille de cache égale ou supérieure à la taille du disque de système d’exploitation de la machine virtuelle elle-même. Autrement, votre déploiement Azure risque d’entraîner une erreur (même s’il est initialement accepté).

2. Spécifiez la version de groupe de machines virtuelles identiques (`vmssApiVersion`) `2018-06-01` ou une version ultérieure :

    ```xml
    "variables": {
        "vmssApiVersion": "2018-06-01",
    ```

3. Dans la section Groupe de machines virtuelles identiques de votre modèle de déploiement, spécifiez l’option `Local` pour `diffDiskSettings` :

    ```xml
    "apiVersion": "[variables('vmssApiVersion')]",
    "type": "Microsoft.Compute/virtualMachineScaleSets",
        "virtualMachineProfile": {
            "storageProfile": {
                "osDisk": {
                        "caching": "ReadOnly",
                        "createOption": "FromImage",
                        "diffDiskSettings": {
                            "option": "Local"
                        },
                }
            }
        }
    ```

> [!NOTE]
> Les applications d’utilisateurs ne doivent pas avoir de dépendance/fichier/artefact sur le disque du système d’exploitation, car le disque du système d’exploitation serait perdu en cas de mise à niveau du système d’exploitation.

> [!NOTE]
> Les VMSS non éphémères existants ne peuvent pas être mis à niveau sur place pour utiliser des disques éphémères.
> Pour effectuer une migration, les utilisateurs devront [ajouter](./virtual-machine-scale-set-scale-node-type-scale-out.md) un nouveau nodeType avec des disques éphémères, déplacer les charges de travail vers le nouveau nodeType, puis [supprimer](./service-fabric-how-to-remove-node-type.md) le nodeType existant.
>

Pour plus d’informations et d’autres options de configuration, voir [Disques de système d’exploitation éphémères pour machines virtuelles Azure](../virtual-machines/ephemeral-os-disks.md) 


### <a name="select-the-durability-and-reliability-levels-for-the-cluster"></a>Sélectionner les niveaux de durabilité et de fiabilité du cluster
Le niveau de durabilité est utilisé pour indiquer au système les privilèges dont disposent vos machines virtuelles avec l’infrastructure Azure sous-jacente. Dans le type de nœud principal, ce privilège permet à Service Fabric de suspendre toute demande de l’infrastructure au niveau de la machine virtuelle (par exemple, un redémarrage de la machine virtuelle, une réinitialisation de la machine virtuelle ou une migration de machine virtuelle) qui influe sur la configuration requise du quorum pour les services système et vos services avec état. Dans les types de nœuds non principaux, ce privilège permet à Service Fabric de suspendre toute demande de l’infrastructure au niveau de la machine virtuelle (comme le redémarrage de la machine virtuelle, la réinitialisation de la machine virtuelle, la migration de machine virtuelle, etc.) qui influe sur la configuration requise du quorum pour vos services avec état.  Pour connaître les avantages liés aux différents niveaux et savoir quel niveau utiliser et quand, consultez [Caractéristiques de durabilité du cluster][durability].

Le niveau de fiabilité est utilisé pour définir le nombre de réplicas des services système que vous voulez exécuter dans ce cluster sur le type de nœud principal. Plus le nombre de réplicas est important, plus les services système sont fiables dans votre cluster.  Pour connaître les avantages liés aux différents niveaux et savoir quel niveau utiliser et quand, consultez [Caractéristiques de fiabilité du cluster][reliability]. 

## <a name="enable-reverse-proxy-andor-dns"></a>Activer un proxy inverse et/ou DNS
Les services se connectant entre eux, en général à l’intérieur d’un cluster, peuvent accéder directement aux points de terminaison d’autres services, car les nœuds d’un cluster se trouvent sur le même réseau local. Afin de faciliter la connexion entre les services, Service Fabric fournit des services supplémentaires : un [service DNS](service-fabric-dnsservice.md) et un [service de proxy inverse](service-fabric-reverseproxy.md).  Ces deux services peuvent être activés lors du déploiement d'un cluster.

Dans la mesure où de nombreux services, en particulier les services en conteneur, peuvent avoir un nom d’URL existant, il est pratique de pouvoir utiliser le protocole DNS standard (plutôt que le protocole Service de nommage), notamment dans les scénarios impliquant le développement et le transfert d’applications. C’est là précisément qu’intervient le service DNS. Il vous permet de mapper les noms DNS à un nom de service et, par conséquent, de résoudre les adresses IP des points de terminaison.

Le proxy inverse traite les services dans le cluster qui expose des points de terminaison HTTP (y compris HTTPS). Le proxy inverse simplifie l’appel d’autres services en fournissant un format d'URI spécifique.  Le proxy inverse gère également les étapes de résolution, connexion et nouvelle tentative requises par un service pour communiquer avec un autre.

## <a name="prepare-for-disaster-recovery"></a>Préparation à la récupération d’urgence
Pour fournir une haute disponibilité, il est essentiel que les services puissent survivre à tous les types d’échecs. Ceci est particulièrement important pour les échecs inattendus et hors de votre contrôle. L'article [Préparation à la récupération d’urgence](service-fabric-disaster-recovery.md)décrit certains modes d’échec courants qui peuvent aboutir à une situation critique s’ils ne sont pas modélisés et gérés correctement. Il traite également des atténuations de risques et des actions à entreprendre si un incident se produit.

## <a name="production-readiness-checklist"></a>Liste de vérification de disponibilité de la production
Votre application et le cluster sont prêts à accepter le trafic de production ? Avant de déployer votre cluster en production, passez en revue la [Check-list de préparation à la production](service-fabric-production-readiness-checklist.md). Assurez le bon fonctionnement de votre application et de votre cluster en passant en revue les éléments de cette check-list. Nous recommandons vivement de vérifier tous ces points avant de passer en production.

## <a name="next-steps"></a>Étapes suivantes
* [Créer un cluster Service Fabric exécutant Windows](service-fabric-best-practices-overview.md)
* [Créer un cluster Service Fabric exécutant Linux](service-fabric-tutorial-create-vnet-and-linux-cluster.md)

[placementconstraints]: service-fabric-cluster-resource-manager-cluster-description.md#node-properties-and-placement-constraints
[durability]: service-fabric-cluster-capacity.md#durability-characteristics-of-the-cluster
[reliability]: service-fabric-cluster-capacity.md#reliability-characteristics-of-the-cluster
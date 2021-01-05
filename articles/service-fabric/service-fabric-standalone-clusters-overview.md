---
title: Vue d’ensemble des clusters Service Fabric autonomes
description: Les clusters Service Fabric peuvent être exécutés sous Windows Server et Linux, ce qui vous permet de déployer et d’héberger des applications Service Fabric partout où vous pouvez exécuter Windows Server ou Linux.
ms.topic: conceptual
ms.date: 02/01/2019
ms.openlocfilehash: a3162a3d02510ce8efab6c5bcac0f1fdd2b2539b
ms.sourcegitcommit: c157b830430f9937a7fa7a3a6666dcb66caa338b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94683996"
---
# <a name="overview-of-service-fabric-standalone-clusters"></a>Vue d’ensemble des clusters Service Fabric autonomes

Un cluster Service Fabric est un groupe de machines virtuelles ou physiques connectées au réseau, sur lequel vos microservices sont déployés et gérés. Une machine ou machine virtuelle faisant partie d’un cluster est appelée un nœud de cluster. Les clusters peuvent être mis à l’échelle pour des milliers de nœuds. Si vous ajoutez des nœuds au cluster, Service Fabric rééquilibre les réplicas de partition du service et les instances sur le nombre de nœuds augmenté. Les performances globales de l’application s’améliorent tandis que le conflit d’accès à la mémoire diminue. Si les nœuds du cluster ne sont pas utilisés efficacement, vous pouvez diminuer le nombre de nœuds dans le cluster. Service Fabric rééquilibre à nouveau les réplicas de partition et les instances sur le nombre réduit de nœuds afin de mieux utiliser le matériel sur chaque nœud.

Un type de nœud définit la taille, le nombre et les propriétés d’un ensemble de nœuds du cluster. Chaque type de nœud peut ensuite faire l’objet d’une montée ou descente en puissance de manière indépendante, avoir différents jeux de ports ouverts et présenter différentes métriques de capacité. Les types de nœuds permettent de définir les rôles d’un groupe de nœuds de cluster, par exemple « frontal » ou « back end ». Votre cluster peut avoir plusieurs types de nœuds, mais le type de nœud principal doit avoir au moins cinq machines virtuelles pour les clusters de production (ou au moins trois machines virtuelles pour les clusters de test). Les [services système de Service Fabric](service-fabric-technical-overview.md#system-services) sont placés sur les nœuds du type de nœud principal.

Le processus de création d’un cluster Service Fabric en local est similaire au processus de création d’un cluster sur un cloud de votre choix avec un ensemble de machines virtuelles. Les étapes initiales d’approvisionnement des machines virtuelles sont régies par le fournisseur de cloud ou l’environnement local que vous utilisez. Une fois que vous avez un ensemble de machines virtuelles avec une connectivité réseau activée entre elles, les étapes suivantes pour configurer le package Service Fabric, modifier les paramètres du cluster et exécuter les scripts de création et de gestion du cluster sont identiques. Cela garantit que vos connaissances et votre expérience en matière d’exploitation et de gestion des clusters Service Fabric peuvent être transférées lorsque vous choisissez de cibler de nouveaux environnements d’hébergement.

## <a name="cluster-security"></a>Sécurité des clusters

Un cluster Service Fabric est une ressource que vous possédez.  Il vous incombe la responsabilité de sécuriser vos clusters pour empêcher les utilisateurs non autorisés de s’y connecter. La sécurisation des clusters est particulièrement importante lorsque vous exécutez des charges de travail de production sur le cluster.

> [!NOTE]
> L’authentification Windows est basée sur Kerberos. NTLM n’est pas pris en charge en tant que type d’authentification.
>
> Dans la mesure du possible, utilisez l’authentification par certificat X.509 pour les clusters Service Fabric.

### <a name="node-to-node-security"></a>Sécurité nœud à nœud

La sécurité nœud à nœud sécurise la communication entre les machines virtuelles ou les ordinateurs d’un cluster. Ce scénario de sécurité garantit que seuls les ordinateurs qui sont autorisés à rejoindre le cluster peuvent participer à l’hébergement des applications et des services du cluster. Service Fabric utilise des certificats X.509 pour sécuriser un cluster et fournir des fonctionnalités de sécurité d’applications.  Un certificat de cluster est nécessaire pour sécuriser le trafic de cluster et fournir une authentification de cluster et de serveur.  Les certificats auto-signés peuvent être utilisés pour les clusters de test, mais un certificat émis par une autorité de certification approuvée doit être utilisé pour sécuriser les clusters de production.

La sécurité Windows peut également être activée pour un cluster autonome Windows. Nous vous recommandons d’utiliser la sécurité Windows avec des comptes de service gérés de groupe si vous disposez de Windows Server 2012 R2 et d’Active Directory. Sinon, continuez à utiliser la sécurité Windows avec les comptes Windows.

Pour plus d’informations, consultez [Sécurité nœud à nœud](service-fabric-cluster-security.md#node-to-node-security)

### <a name="client-to-node-security"></a>Sécurité client à nœud

La sécurité client à nœud authentifie les clients et sécurise la communication entre un client et les nœuds du cluster. Ce type de sécurité aide à garantir que seuls les utilisateurs autorisés peuvent accéder au cluster et aux applications déployées sur le cluster. Les clients sont identifiés de manière unique grâce à leurs informations d’identification de sécurité de certificat X.509. Il est possible d’utiliser un nombre quelconque de certificats clients facultatifs pour authentifier les clients d’administration ou d’utilisateur auprès du cluster.

En plus des certificats clients, Azure Active Directory peut également être configuré pour authentifier des clients auprès du cluster.

Pour plus d’informations, consultez [Sécurité client à nœud](service-fabric-cluster-security.md#client-to-node-security)

### <a name="service-fabric-role-based-access-control"></a>Contrôle d’accès en fonction du rôle Service Fabric
Service Fabric prend également en charge le contrôle d’accès pour limiter l’accès à certaines opérations de cluster pour différents groupes d’utilisateurs. Ainsi, vous rendez le cluster plus sécurisé. Deux types de contrôle d’accès sont pris en charge pour les clients qui se connectent à un cluster : le rôle Administrateur et le rôle Utilisateur.  

Pour plus d’informations, consultez [Contrôle d’accès en fonction du rôle Service Fabric](service-fabric-cluster-security.md#service-fabric-role-based-access-control).

## <a name="scaling"></a>Mise à l'échelle

La demande des applications change au fil du temps. Il vous faut éventuellement augmenter les ressources de cluster pour prendre en charge l’augmentation de la charge applicative ou du trafic réseau ou diminuer les ressources du cluster en cas de baisse de la demande. Une fois que vous avez créé un cluster Service Fabric, vous pouvez le mettre à l’échelle horizontalement (modifier le nombre de nœuds) ou verticalement (modifier les ressources des nœuds). Une mise à l’échelle peut s’effectuer à tout moment, même lorsque des charges de travail sont en cours d’exécution sur le cluster. Lorsque vous mettez vos nœuds à l’échelle, vos applications sont automatiquement mises à l’échelle.

Pour plus d’informations, consultez [Mise à l’échelle des clusters autonomes](service-fabric-cluster-scaling-standalone.md).

## <a name="upgrading"></a>Mise à niveau

Un cluster autonome est une ressource que vous possédez entièrement. Vous êtes responsable de la mise à jour corrective du système d’exploitation sous-jacent et des mises à niveau de structure. Vous pouvez définir votre cluster de façon à recevoir des mises à niveau automatiques du runtime quand Microsoft publie une nouvelle version, ou choisir de sélectionner la version de runtime prise en charge de votre choix. En plus des mises à niveau de structure, vous pouvez également corriger le système d’exploitation et mettre à jour la configuration du cluster telle que les certificats ou les ports de l’application. 

Pour plus d’informations, consultez [Mise à niveau des clusters autonomes](service-fabric-cluster-upgrade-standalone.md).

## <a name="supported-operating-systems"></a>Systèmes d’exploitation pris en charge

Vous pouvez créer des clusters sur des machines virtuelles ou des ordinateurs qui exécutent ces systèmes d’exploitation (Linux n’est pas encore pris en charge) :

* Windows Server 2012 R2
* Windows Server 2016 
* Windows Server 2019

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en plus sur la [sécurisation](service-fabric-cluster-security.md), la [mise à l’échelle](service-fabric-cluster-scaling-standalone.md) et la [mise à niveau](service-fabric-cluster-upgrade-standalone.md) des clusters autonomes.

Découvrez les [options de support de Service Fabric](service-fabric-support.md).

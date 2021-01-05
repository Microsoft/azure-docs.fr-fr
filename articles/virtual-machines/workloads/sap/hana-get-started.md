---
title: Installation de SAP HANA sur des machines virtuelles Azure | Microsoft Docs
description: Guide d’installation de SAP HANA sur des machines virtuelles Azure
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: c51a2a06-6e97-429b-a346-b433a785c9f0
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 03/19/2020
ms.author: juergent
ms.openlocfilehash: 83c149e92276265b5b5236ae56c475d7307c18e4
ms.sourcegitcommit: 5db975ced62cd095be587d99da01949222fc69a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97093908"
---
# <a name="installation-of-sap-hana-on-azure-virtual-machines"></a>Installation de SAP HANA sur des machines virtuelles Azure
## <a name="introduction"></a>Introduction
Ce guide vous aide à pointer sur les ressources appropriées pour déployer correctement HANA sur des machines virtuelles Azure. Ce guide va vous orienter vers les ressources de documentation que vous devez consulter avant d’installer SAP HANA sur une machine virtuelle Azure. Ainsi, vous serez en mesure d’effectuer les opérations appropriées pour bénéficier au final d’une configuration prise en charge de SAP HANA sur les machines virtuelles Azure.  

> [!NOTE]
> Ce guide décrit des déploiements de SAP HANA sur des machines virtuelles Azure. Pour obtenir des informations sur la façon de déployer SAP HANA sur de grandes instances HANA, consultez [Guide pratique d’installation et de configuration de SAP HANA (grandes instances) sur Azure](./hana-installation.md).
 
## <a name="prerequisites"></a>Prérequis
Ce guide suppose également que vous êtes familiarisé avec :
* Les systèmes SAP HANA et SAP NetWeaver et comment les installer localement.
* L’installation et l’utilisation d’instances d’application SAP HANA et SAP sur Azure.
* Les concepts et procédures décrits dans :
   * La planification du déploiement de SAP sur Azure, ce qui comprend la planification du réseau virtuel Azure et l’utilisation du stockage Azure. Consultez [SAP NetWeaver sur Azure Virtual Machines – Guide de planification et d’implémentation](./planning-guide.md)
   * Principes de déploiement et modalités de déploiement sur des machines virtuelles dans Azure. Consultez [Déploiement de machines virtuelles Azure pour SAP](./deployment-guide.md)
   * Concepts de haute disponibilité pour SAP HANA, comme indiqués dans [Haute disponibilité de SAP HANA pour les machines virtuelles Azure](./sap-hana-availability-overview.md)

## <a name="step-by-step-before-deploying"></a>Procédure pas à pas avant le déploiement
Cette section liste les différentes étapes que vous devez suivre avant de commencer l’installation de SAP HANA sur une machine virtuelle Azure. L’ordre des opérations est fourni et doit être suivi tel quel :

1. Tous les scénarios de déploiement possibles ne sont pas pris en charge sur Azure. Par conséquent, vous devez rechercher le scénario que vous avez à l’esprit avec votre déploiement de SAP HANA dans le document [Charge de travail SAP sur les machines virtuelles Azure - Scénarios pris en charge](./sap-planning-supported-configurations.md). Si ce scénario ne figure pas dans la liste, vous devez supposer qu’il n’a pas été testé et, par conséquent, qu’il n’est pas pris en charge.
2. En supposant que vous avez une idée approximative de vos besoins en mémoire pour votre déploiement SAP HANA, vous devez trouver une machine virtuelle Azure adaptée. Toutes les machines virtuelles certifiées pour SAP NetWeaver, comme indiqué dans la [note de support SAP n° 1928533](https://launchpad.support.sap.com/#/notes/1928533), sont certifiées SAP HANA. La source de vérité pour les machines virtuelles Azure certifiées SAP HANA est le site web [Répertoire matériel ASAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure). Les unités commençant par la lettre **S** sont des unités de [grandes instances HANA](./hana-overview-architecture.md) et non des machines virtuelles Azure.
3. Les différents types de machines virtuelles Azure ont des versions de système d’exploitation minimales différentes pour SUSE Linux et Red Hat Linux. Sur le site web [Répertoire matériel SAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure), vous devez cliquer sur une entrée dans la liste des unités certifiées SAP HANA pour obtenir des données détaillées de cette unité. Outre la charge de travail HANA prise en charge, les versions de système d’exploitation prises en charge avec ces unités pour SAP HANA sont listées.
4. En ce qui concerne les versions de système d’exploitation, vous devez prendre en compte certaines versions de noyau minimales. Ces versions minimales sont documentées dans les notes de support SAP suivantes :
    - [Note de support SAP n° 2814271 – La sauvegarde SAP HANA échoue sur Azure avec une erreur de somme de contrôle](https://launchpad.support.sap.com/#/notes/2814271)
    - [Note de support SAP n° 2753418 – Dégradation potentielle des performances en raison du repli du minuteur](https://launchpad.support.sap.com/#/notes/2753418)
    - [Note de support SAP n° 2791572 – Dégradation des performances en raison de la prise en charge VDSO manquante pour Hyper-V dans Azure](https://launchpad.support.sap.com/#/notes/2791572)
4. En fonction de la version du système d’exploitation prise en charge pour le type de machine virtuelle de votre choix, vous devez vérifier si la version de SAP HANA que vous souhaitez est prise en charge avec cette version du système d’exploitation. Consultez la [note de support SAP n° 2235581](https://launchpad.support.sap.com/#/notes/2235581) pour obtenir un tableau de prise en charge des versions SAP HANA avec les différentes versions de système d’exploitation.
5. Lorsque vous pensez avoir trouvé une combinaison valide de type de machine virtuelle Azure, de version du système d’exploitation et de version SAP HANA, vérifiez le tableau de disponibilité des produits SAP. Dans le tableau de disponibilité SAP, vous pouvez déterminer si le produit SAP que vous souhaitez exécuter sur votre base de données SAP HANA est pris en charge.


## <a name="step-by-step-vm-deployment-and-guest-os-considerations"></a>Déploiement pas à pas de machines virtuelles et considérations sur les systèmes d’exploitation invités
Au cours de cette phase, vous devez suivre les étapes de déploiement de la ou des machines virtuelles pour installer HANA et optimiser finalement le système d’exploitation choisi après l’installation.

1. Choisissez l’image de base à partir de la galerie Azure. Si vous souhaitez créer votre propre image de système d’exploitation pour SAP HANA, vous devez connaître tous les packages nécessaires à une installation réussie de SAP HANA. Dans le cas contraire, il est recommandé d’utiliser les images SUSE et Red Hat pour SAP ou SAP HANA, disponibles dans la galerie d’images Azure. Ces images incluent les packages nécessaires à une installation HANA réussie. En fonction de votre contrat de support avec le fournisseur de système d’exploitation, vous devez choisir une image où apporter votre propre licence. Vous pouvez également choisir une image de système d’exploitation qui inclut un support.
2. Si vous avez choisi une image de système d’exploitation invité qui requiert que vous apportiez votre propre licence, vous devez inscrire l’image de système d’exploitation à votre abonnement, afin de pouvoir télécharger et appliquer les derniers correctifs. Cette étape nécessite un accès à l’Internet public. À moins que vous ne configuriez votre instance privée avec, par exemple, un serveur SMT dans Azure.
3. Choisissez la configuration réseau de la machine virtuelle. Vous trouverez plus d’informations dans le document [Configurations et opérations de l’infrastructure SAP HANA sur Azure](./hana-vm-operations.md). N’oubliez pas qu’il n’existe aucun quota de débit réseau que vous pouvez attribuer aux cartes réseau virtuelles dans Azure. Par conséquent, le seul objectif qu’il y a à diriger le trafic via différentes cartes réseau virtuelles est basé sur des considérations de sécurité. Nous sommes confiants que vous trouverez un compromis acceptable entre la complexité du routage du trafic via plusieurs cartes réseau virtuelles et les exigences imposées par les aspects sécuritaires.
3. Appliquez les derniers correctifs au système d’exploitation une fois la machine virtuelle déployée et inscrite. Inscrite à votre propre abonnement, ou, si vous avez choisi une image comprenant la prise en charge du système d’exploitation, la machine virtuelle doit déjà avoir accès aux correctifs. 
4. Appliquez les réglages nécessaires pour SAP HANA. Ces réglages sont listés dans les notes de support SAP suivantes :

    - [Note de support SAP n° 2694118 - Module complémentaire de haute disponibilité Red Hat Enterprise Linux sur Azure](https://launchpad.support.sap.com/#/notes/2694118)
    - [Note de support SAP n° 1984787 - SUSE Linux Enterprise Server 12 : notes d’installation](https://launchpad.support.sap.com/#/notes/1984787) 
    - [Note de support SAP n° 2578899 - SUSE Linux Enterprise Server 15 : note d’installation](https://launchpad.support.sap.com/#/notes/2578899)
    - [Note de support SAP #2002167 - Red Hat Enterprise Linux 7.x : installation et mise à niveau](https://launchpad.support.sap.com/#/notes/0002002167)
    - [Note de support SAP #2292690 - SAP HANA DB : Paramètres de système d’exploitation recommandés pour RHEL 7](https://launchpad.support.sap.com/#/notes/0002292690) 
    -  [Note de support SAP n° 2772999 - Red Hat Enterprise Linux 8.x : installation et configuration](https://launchpad.support.sap.com/#/notes/2772999) 
    -  [Note de support SAP n° 2777782 - SAP HANA DB : paramètres de système d’exploitation recommandés pour RHEL 8](https://launchpad.support.sap.com/#/notes/2777782)
    -  [Note de support SAP n° 2455582 – Linux : Exécution d’applications SAP compilées avec GCC 6.x](https://launchpad.support.sap.com/#/notes/0002455582)
    -  [Note de support SAP n° 2382421 - Optimisation de la configuration réseau au niveau HANA et du système d’exploitation](https://launchpad.support.sap.com/#/notes/2382421)

1. Sélectionnez le type de stockage Azure pour SAP HANA. Au cours de cette étape, vous devez choisir la disposition du stockage pour l’installation de SAP HANA. Vous allez utiliser des disques Azure attachés ou des partages Azure NFS natifs. Les types de stockage Azure pris en charge et les combinaisons de différents types de stockage Azure qui peuvent être utilisées, sont documentés dans [Configurations du stockage des machines virtuelles SAP HANA Azure](./hana-vm-operations-storage.md). Prenez les configurations documentées comme point de départ. Pour les systèmes hors production, vous pouvez être en mesure de configurer des valeurs inférieures de débit ou IOPS. À des fins de production, vous devrez peut-être configurer des valeurs légèrement supérieures de débit et IOPS.
2. Assurez-vous d’avoir configuré l’[Accélérateur des écritures Azure](../../how-to-enable-write-accelerator.md) pour vos volumes qui contiennent les journaux de restauration par progression ou les journaux de transactions SGBD lorsque vous utilisez des machines virtuelles de série M ou Mv2. Tenez compte des limites de l’Accélérateur des écritures, telles qu’indiquées.
2. Vérifiez si la [mise en réseau accélérée Azure](https://azure.microsoft.com/blog/maximize-your-vm-s-performance-with-accelerated-networking-now-generally-available-for-both-windows-and-linux/) est activée sur la ou les machines virtuelles déployées.

> [!NOTE]
> Toutes les commandes dans les différents profils de paramétrage SAP ou décrites dans les notes ne peuvent pas s’exécuter correctement sur Azure. Les commandes qui manipulent le mode d’alimentation des machines virtuelles retournent généralement une erreur, car le mode d’alimentation du matériel hôte Azure sous-jacent ne peut pas être manipulé.

## <a name="step-by-step-preparations-specific-to-azure-virtual-machines"></a>Préparations pas à pas spécifiques aux machines virtuelles Azure
L’une des spécificités d’Azure est l’installation d’une extension de machine virtuelle Azure qui fournit des données de supervision pour l’agent hôte SAP. Les détails de l’installation de cette extension de supervision sont documentés dans :

-  La [note SAP 2191498](https://launchpad.support.sap.com/#/notes/2191498/E) aborde la supervision améliorée SAP avec des machines virtuelles Linux sur Azure. 
-  La [note SAP 1102124](https://launchpad.support.sap.com/#/notes/1102124/E) contient des informations relatives à SAPOSCOL sur Linux. 
-  La [note SAP 2178632](https://launchpad.support.sap.com/#/notes/2178632/E) aborde les métriques de supervision clés pour SAP sur Microsoft Azure.
-  [Déploiement de machines virtuelles Azure pour SAP NetWeaver](./deployment-guide.md#d98edcd3-f2a1-49f7-b26a-07448ceb60ca)

## <a name="sap-hana-installation"></a>Installation de SAP HANA
Une fois les machines virtuelles Azure déployées et les systèmes d’exploitation inscrits et configurés, vous pouvez installer SAP HANA conformément à l’installation SAP. Dans le cadre de cette documentation, commencez par utiliser le site web SAP des [ressources HANA](https://www.sap.com/products/s4hana-erp.html?btp=9d3e6f82-d8ab-4122-8d2d-bf4971217afd).

Pour les configurations de scale-out SAP HANA utilisant des disques à connexion directe de stockage Premium Azure ou de stockage Ultra, lisez les spécificités fournies dans le document [Configurations et opérations de l’infrastructure SAP HANA sur Azure](./hana-vm-operations.md#configuring-azure-infrastructure-for-sap-hana-scale-out).


## <a name="additional-resources-for-sap-hana-backup"></a>Ressources supplémentaires pour la sauvegarde SAP HANA
Pour plus d’informations sur la sauvegarde des bases de données SAP HANA sur des machines virtuelles Azure, consultez :
* [Guide de sauvegarde pour SAP HANA sur des machines virtuelles Azure](./sap-hana-backup-guide.md)
* [Sauvegarde SAP HANA sur Azure au niveau fichier](./sap-hana-backup-file-level.md)

## <a name="next-steps"></a>Étapes suivantes
Lisez la documentation :

- [Configurations et opérations de l’infrastructure SAP HANA sur Azure](./hana-vm-operations.md)
- [Configurations du stockage des machines virtuelles SAP HANA Azure](./hana-vm-operations-storage.md)

---
title: Environnement d’intelligence artificielle et d’analyse d’équipe
titleSuffix: Azure Data Science Virtual Machine
description: Modèles de déploiement de la machine virtuelle Data Science dans un environnement en équipes d’entreprise.
keywords: formation approfondie, IA, outils de science des données, machine virtuelle de science des données, analyse géospatiale, processus de science des données en équipe
services: machine-learning
ms.service: machine-learning
ms.subservice: data-science-vm
author: vijetajo
ms.author: vijetaj
ms.topic: overview
ms.date: 05/08/2018
ms.openlocfilehash: 9126dbcfbfe9e3a94514ebf45685bdfc55dd0306
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96462999"
---
# <a name="data-science-virtual-machine-based-team-analytics-and-ai-environment"></a>Environnement analytique d’équipe et d’intelligence artificielle basé sur Data Science Virtual Machine 
[Data Science Virtual Machine](overview.md) (DSVM) fournit un environnement enrichi sur la plateforme Azure avec des logiciels prédéfinis pour l’intelligence artificielle (IA) et l’analytique des données.

En règle générale, la DSVM est utilisée comme un ordinateur de bureau d’analyse individuel. Les scientifiques des données individuels gagnent en productivité avec cet environnement d’analytique prédéfini partagé. Étant donné que les grandes équipes d’analytique planifient les environnements pour leurs scientifiques des données et leurs développeurs en intelligence artificielle, une infrastructure d’analytique partagée pour le développement et l’expérimentation est l’un des thèmes récurrents. Cette infrastructure est gérée en fonction des stratégies informatiques de l’entreprise, qui facilitent également la collaboration et la cohérence entre les équipes de science des données et les équipes d’analytique.

Une infrastructure partagée permet une meilleure utilisation informatique de l’environnement d’analytique. Certaines organisations appellent *bac à sable (sandbox) d’analytique* l’infrastructure de science des données/d’analytique basée sur une équipe. Elle permet aux scientifiques des données d’accéder à différentes ressources de données pour comprendre rapidement les données. Cet environnement de bac à sable leur permet également d’exécuter des expériences, de valider des hypothèses et de générer des modèles prédictifs sans affecter l’environnement de production.

Étant donné que la Data Science Virtual Machine (DSVM) fonctionne au niveau de l’infrastructure Azure, les administrateurs informatiques peuvent la configurer facilement afin qu’elle fonctionne conformément aux stratégies informatiques de l’entreprise. DSVM offre une flexibilité parfaite d’implémentation de différentes architectures de partage tout en permettant également un accès contrôlé aux ressources de données de l’entreprise.

Cette section présente les modèles et recommandations que vous pouvez appliquer pour déployer la machine virtuelle DSVM en tant qu’infrastructure de science des données basée sur une équipe. Comme les composants de ces modèles proviennent de l’infrastructure en tant que service (IaaS) Azure, ils s’appliquent à toutes les machines virtuelles Azure. Cette série d’articles est axée sur l’application de ces fonctionnalités d’infrastructure Azure standard à DSVM.

Les principaux composants d’un environnement d’analytique d’équipe d’entreprise sont les suivants :

* [Pool de machines virtuelles DSVM mis à l’échelle automatiquement](dsvm-pools.md)
* [Identité commune et accès à l’espace de travail à partir des machines virtuelles DSVM du pool](dsvm-common-identity.md)
* [Sécuriser l’accès aux sources de données](dsvm-secure-access-keys.md)


Cette série d’articles contient des conseils et des pointeurs qui couvrent chacune des rubriques précédentes. Elle n’aborde pas toutes les considérations et exigences relatives au déploiement de machines virtuelles DSVM dans des configurations d’entreprise d’envergure. Voici d’autres ressources Azure que vous pouvez utiliser lors de l’implémentation d’instances DSVM dans votre entreprise :

* [Sécurité du réseau](../../security/fundamentals/network-overview.md)
* [Surveillance](../../azure-monitor/insights/monitor-vm-azure.md) et [gestion](../../virtual-machines/maintenance-and-updates.md?bc=%2fazure%2fvirtual-machines%2fwindows%2fbreadcrumb%2ftoc.json%252c%2fazure%2fvirtual-machines%2fwindows%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json%253ftoc%253d%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [Journalisation et audit](../../security/fundamentals/log-audit.md)
* [Contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../role-based-access-control/overview.md)
* [Définition et application de stratégies](../../governance/policy/overview.md)
* [Logiciel anti-programme malveillant](../../security/fundamentals/antimalware.md)
* [Chiffrement](../../virtual-machines/windows/disk-encryption-overview.md)
* [Découverte et gouvernance des données](../../data-catalog/index.yml)

Enfin, le [Centre des architectures Azure](/azure/architecture/) fournit une architecture de bout en bout détaillée et des modèles pour la création et la gestion de votre infrastructure d’analytique basée sur le cloud.
---
title: Disponibilité de SAP HANA sur les machines virtuelles Azure - Vue d’ensemble | Microsoft Docs
description: Décrivez les opérations SAP HANA sur les machines virtuelles Azure natives.
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: msjuergent
manager: patfilot
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 03/05/2018
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: ac74bb4b37535d0c0b095ea300afc15aa0978ed8
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94950123"
---
# <a name="sap-hana-high-availability-for-azure-virtual-machines"></a>Haute disponibilité de SAP HANA pour les machines virtuelles Azure

Vous pouvez utiliser de nombreuses fonctionnalités Azure pour déployer des bases de données stratégiques comme SAP HANA dans des machines virtuelles Azure. Cet article fournit des conseils sur la façon de bénéficier d’une certaine disponibilité pour les instances SAP HANA qui sont hébergées dans des machines virtuelles Azure. L’article décrit plusieurs scénarios que vous pouvez implémenter en utilisant l’infrastructure Azure pour augmenter la disponibilité de SAP HANA sur Azure. 

## <a name="prerequisites"></a>Prérequis

Cet article suppose que vous maîtrisiez certains concepts de base d’IaaS (Infrastructure as a Service) dans Azure, notamment : 

- Comment déployer des machines virtuelles ou des réseaux virtuels en utilisant le portail Azure ou PowerShell.
- L’utilisation de l’interface de ligne de commande multiplateforme (Azure CLI) Azure, y compris la possibilité d’utiliser des modèles JSON (JavaScript Object Notation).

Cet article suppose également que vous êtes familiarisé avec l’installation, l’administration et le fonctionnement des instances SAP HANA. Il est très important de se familiariser avec la configuration et les opérations de réplication du système HANA. Cela inclut des tâches telles que la sauvegarde et la restauration des bases de données SAP HANA.

Ces articles fournissent une bonne vue d’ensemble de l’utilisation de SAP HANA dans Azure :

- [Installation manuelle d’un système SAP HANA à instance unique sur des machines virtuelles Azure](./hana-get-started.md)
- [Installer la réplication de système SAP HANA sur des machines virtuelles Azure](sap-hana-high-availability.md)
- [Sauvegarder SAP HANA sur des machines virtuelles Azure](./sap-hana-backup-guide.md)

Il est également judicieux de se familiariser avec ces articles sur SAP HANA :

- [Haute disponibilité pour SAP HANA](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.02/en-US/6d252db7cdd044d19ad85b46e6c294a4.html)
- [FAQ : Haute disponibilité pour SAP HANA](https://archive.sap.com/documents/docs/DOC-66702)
- [Effectuer une réplication de système pour SAP HANA](https://archive.sap.com/documents/docs/DOC-47702)
- [SAP HANA 2.0 SPS 01 - Nouveautés : Haute disponibilité](https://blogs.sap.com/2017/05/15/sap-hana-2.0-sps-01-whats-new-high-availability-by-the-sap-hana-academy/)
- [Recommandations relatives au réseau pour la réplication de système SAP HANA](https://www.sap.com/documents/2016/06/18079a1c-767c-0010-82c7-eda71af511fa.html)
- [Réplication de système SAP HANA](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.01/en-US/b74e16a9e09541749a745f41246a065e.html)
- [Redémarrage automatique du service SAP HANA](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.01/en-US/cf10efba8bea4e81b1dc1907ecc652d3.html)
- [Configurer la réplication de système SAP HANA](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.01/en-US/676844172c2442f0bf6c8b080db05ae7.html)

En plus de vous familiariser avec le déploiement de machines virtuelles dans Azure, nous vous recommandons également de lire l’article [Gestion de la disponibilité des machines virtuelles Windows dans Azure](../../manage-availability.md) avant de poursuivre la définition de votre architecture de disponibilité dans Azure.

## <a name="service-level-agreements-for-azure-components"></a>Contrats de niveau de service pour des composants Azure

Azure propose différents contrats SLA de disponibilité pour différents composants, comme la mise en réseau, le stockage et les machines virtuelles. Tous les contrats SLA sont documentés. Pour plus d'informations, consultez la page [Contrats de niveau de service Microsoft Azure](https://azure.microsoft.com/support/legal/sla/). 

[Contrat SLA pour les machines virtuelles](https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_8/) décrit trois contrats de niveau de service, pour trois configurations différentes :

- Une seule machine virtuelle qui utilise des [disques SSD Premium Azure](../../managed-disks-overview.md) pour le disque du système d'exploitation et tous les disques de données. Cette option fournit une durée de fonctionnement mensuelle de 99,9 %.
- Plusieurs machines virtuelles (au moins deux) qui sont organisées dans un [groupe à haute disponibilité Azure](../../windows/tutorial-availability-sets.md). Cette option fournit une durée de fonctionnement mensuelle de 99,95 %.
- Plusieurs machines virtuelles (au moins deux) qui sont organisées dans une [zone de disponibilité](../../../availability-zones/az-overview.md). Cette option fournit une durée de fonctionnement mensuelle de 99,99 %.

Mesurez vos exigences de disponibilité par rapport aux contrats SLA que les composants Azure peuvent fournir. Ensuite, choisissez vos scénarios pour SAP HANA pour atteindre le niveau de disponibilité requis.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur [Disponibilité de SAP HANA au sein d’une région Azure](./sap-hana-availability-one-region.md).
- En savoir plus sur [Disponibilité de SAP HANA dans l’ensemble des régions Azure](./sap-hana-availability-across-regions.md). 
















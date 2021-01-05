---
title: Certification de SAP HANA sur Azure (grandes instances) | Microsoft Docs
description: Certification de SAP HANA sur Azure (grandes instances).
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/04/2018
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 7cbd50fae2b53d801f7aa09fae2b4540eef13c9e
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94967888"
---
# <a name="certification"></a>Certification

Outre la certification NetWeaver, SAP requiert une certification spéciale pour SAP HANA, afin de prendre en charge SAP HANA sur certaines infrastructures, telles que Azure IaaS.

La Remarque SAP principale sur NetWeaver, et dans une certaine mesure sur la certification SAP HANA, est [Remarque SAP n° 1928533 - Applications SAP sur Azure : types de machines virtuelles Azure et produits pris en charge](https://launchpad.support.sap.com/#/notes/1928533).

Vous trouvez les enregistrements de certification pour les unités SAP HANA sur Azure (grandes instances) sur le site [Plateformes IaaS certifiées SAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure). 

Les types SAP HANA sur Azure (grandes instances), auxquels il est fait référence sur le site Plateformes IaaS certifiées SAP HANA, fournit aux clients SAP et Microsoft la possibilité de déployer d’importantes ressources SAP Business Suite, SAP BW, S/4 HANA, BW/4HANA ou d’autres charges de travail SAP HANA dans Azure. La solution est basée sur le tampon matériel dédié et certifié SAP-HANA ([SAP HANA Tailored Datacenter Integration – TDI](https://scn.sap.com/docs/DOC-63140)). Si vous exécutez une solution TDI configurée de SAP HANA, toutes les applications SAP HANA (tels que SAP Business Suite sur SAP HANA, SAP BW sur SAP HANA, S4/HANA et BW4/HANA) fonctionne sur l’infrastructure matérielle.

Par rapport à SAP HANA en cours d’exécution dans des machines virtuelles, cette solution offre un avantage. Elle fournit des volumes de grande capacité mémoire. Pour activer cette solution, vous devez comprendre les aspects clés suivants :

- La couche Application SAP et les applications autres que SAP s’exécutent dans des machines virtuelles hébergées dans les tampons matériels Azure habituels.
- Les déploiements d’applications, de centres de données et d’infrastructures locaux du client sont connectés à la plateforme cloud via ExpressRoute (recommandé) ou un réseau privé virtuel (VPN). Le répertoire Active Directory et le DNS sont également étendus dans Azure.
- L’instance de base de données SAP HANA pour les charges de travail HANA s’exécute sur SAP HANA sur Azure (grandes instances). Le tampon de grande instance est connecté au réseau Azure, afin que les logiciels s’exécutant dans des machines virtuelles puissent interagir avec l’instance HANA s’exécutant dans la grande instance HANA.
- Le matériel de SAP HANA sur Azure (grandes instances) est un matériel dédié fourni dans une Infrastructure as a Service (IaaS) avec SUSE Linux Enterprise Server ou Red Hat Enterprise Linux préinstallé. Comme pour les machines virtuelles, les mises à jour et la maintenance supplémentaires du système d’exploitation relèvent de votre responsabilité.
- L’installation de HANA ou tous les composants supplémentaires nécessaires à l’exécution de SAP HANA sur les unités de la grande instance HANA est de votre responsabilité. Toutes les opérations en cours respectives et l’administration de SAP HANA sur Azure sont également de votre responsabilité.
- Outre les solutions décrites ici, vous pouvez installer d’autres composants dans votre abonnement Azure qui se connecte à SAP HANA sur Azure (grandes instances). Par exemple, les composants qui permettent la communication directement avec la base de données SAP HANA, serveurs jump, serveurs RDP, SAP HANA Studio, scénarios SAP Data Services for SAP BI ou des solutions de surveillance réseau.
- Comme dans Azure, la grande instance HANA prend en charge des fonctionnalités de haute disponibilité et de récupération d’urgence.

**Étapes suivantes**
- Voir [Références SKU disponibles pour HLI](hana-available-skus.md) 
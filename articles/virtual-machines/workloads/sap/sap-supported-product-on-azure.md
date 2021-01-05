---
title: 'SAP sur Azure : logiciels SAP sont pris en charge sur Azure'
description: Évoque les logiciels SAP dont le déploiement est pris en charge sur Azure
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
tags: azure-resource-manager
keywords: SAP
ms.assetid: d7c59cc1-b2d0-4d90-9126-628f9c7a5538
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 04/21/2020
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: a02c30423ab3c6de87eac7d34b5f0004e54a8d5d
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94957807"
---
# <a name="what-sap-software-is-supported-for-azure-deployments"></a>Logiciels SAP dont le déploiement est pris en charge sur Azure
Cet article explique comment identifier les logiciels SAP dont le déploiement est pris en charge sur Azure, ainsi que les versions de système d’exploitation ou système de gestion de base de données nécessaires.

Pour évaluer si votre logiciel SAP actuel est pris en charge et identifier les versions de système d’exploitation et de système de gestion de base de données (SGBD) prises en charge avec votre logiciel SAP dans Azure, vous devez consulter les ressources suivantes :

- Notes du support SAP
- Tableau de disponibilité des produits SAP



## <a name="general-restrictions-for-sap-workload"></a>Restrictions générales pour la charge de travail SAP
Les services Azure IaaS qui utilisables pour des charges de travail SAP sont limités aux composants matériels x86-64 ou x64. Aucune offre basée sur un processeur Sparc ou Power ne s’applique aux charges de travail SAP. Les clients qui exécutent leurs applications sur des systèmes d’exploitation appartenant à des architectures matérielles telles que des macroordinateurs ou des mini-ordinateurs AS400 IBM, ou bien où les systèmes d’exploitation HP-UX, Solaris ou AIX sont utilisés, doivent remplacer leurs applications SAP, y compris les systèmes de gestion de base de données (SGBD), par l’un des systèmes d’exploitation suivants :

- Windows Server 64 bits pour la plateforme x86-64
- SUSE Linux 64 bits pour la plateforme x86-64
- Red Hat Linux 64 bits pour la plateforme x86-64
- Oracle Linux 64 bits pour la plateforme x86-64

En association avec des logiciels SAP, les autres versions de système d’exploitation ou de distribution Linux ne sont pas prises en charge. Des détails exacts sur des versions et cas spécifiques sont documentés plus loin dans le document.


## <a name="you-start-here"></a>Vous commencez ici
Votre point de départ est la [Note de support SAP n° 1928533](https://launchpad.support.sap.com/#/notes/1928533). La présente note SAP présente plusieurs logiciels et machines virtuelles pris en charge

La première section indique la configuration minimale requise des systèmes d’exploitation pris en charge avec les logiciels SAP sur des machines virtuelles Azure en général. Si vous ne disposez pas de cette configuration minimale requise et exécutez des versions antérieures de ces systèmes d’exploitation, vous devez mettre à niveau votre système d’exploitation vers cette version minimale, voire vers des versions plus récentes. En règle générale, Azure prend en charge des versions plus anciennes de certains de ces systèmes d’exploitation. Toutefois, les restrictions ou versions minimales répertoriées sont basées sur des tests et procédures de qualification d’ores et déjà exécutés qui ne seront pas répétés. 


> [!NOTE]
>Certains types spécifiques de machines virtuelles, de grandes instances HANA ou de charges de travail SAP nécessitent des versions de système d’exploitation plus récentes. Le document mentionne de tels cas. Ceux-ci sont clairement documentés dans des notes SAP ou d’autres publications SAP.

La section suivante répertorie les plateformes SAP générales compatibles avec les mises en production et surtout les noyaux SAP pris en charge. Elle énumère les piles NetWeaver/ABAP ou Java prises en charge ET nécessitant des versions de noyau minimales. Des piles ABAP plus récentes sont prises en charge sur Azure, qui n’imposent pas de versions de noyau minimales parce que les modifications destinées à Azure ont été implémentées dès le début de leur développement.

Vous devez vérifier les points suivants :

- Si les applications SAP que vous exécutez sont couvertes par les versions minimales indiquées. Si ce n’est pas le cas, vous devez définir une nouvelle version cible, et vérifier dans le Tableau de disponibilité des produits SAP les combinaisons de build de système d’exploitation et de SGBD prises en charge avec la nouvelle version cible. Ainsi, vous pouvez ainsi choisir les versions de système d’exploitation et de SGBD appropriées.
- Si vous devez mettre à jour vos noyaux SAP en opérant une migration vers Azure.
- Si vous devez mettre à jour des packages de support SAP. En particulier, des packages de support de base qui peuvent être nécessaires pour les cas où vous devez impérativement passer à une version de SGBD plus récente.


La section suivante décrit plus en détail d’autres produits SAP et versions de SGBD pris en charge par SAP sur Azure pour Windows et Linux. 

> [!NOTE]
> Les versions minimales des différents SGBD sont soigneusement choisies et peuvent ne pas toujours refléter le spectre entier des versions de SGBD que les différents fournisseurs de SGBD prennent en charge sur Azure en général. De nombreuses considérations relatives à la charge de travail SAP ont été prises en compte pour définir ces versions minimales. Le test et la qualification des anciennes versions de SGBD ne nécessitent aucun effort. 

> [!NOTE]
> Les versions minimales répertoriées représentent des versions antérieures des systèmes d’exploitation et des bases de données. Nous encourageons vivement à utiliser les versions les plus récentes des systèmes d’exploitation et des bases de données. Dans de nombreux cas, des versions plus récentes de système d’exploitation et de bases de données ont pris en compte le cas d’utilisation consistant à exécuter dans le cloud public, et adapté le code pour une exécution dans un cloud public ou plus spécifiquement sur Azure.

## <a name="oracle-dbms-support"></a>Prise en charge de SGBD Oracle
Le système d’exploitation, les versions de SGBD Oracle et les fonctionnalités Oracle pris en charge sur Azure sont spécifiquement répertoriés dans la [Note de support SAP n°2039619](https://launchpad.support.sap.com/#/notes/2039619). L’essence de cette note peut être résumée comme suit :

- La version minimale d’Oracle prise en charge sur les machines virtuelles Azure certifiées pour NetWeaver est Oracle 11g version 2 Patchset 3 (11.2.0.4)
- En tant que systèmes d’exploitation invités, seuls Windows et Oracle Linux sont éligibles. Les versions exactes de système d’exploitation et de SGBD associées sont répertoriées dans la note
- La prise en charge d’Oracle Linux s’étend également au client de SGBD Oracle. Cela signifie que tous les composants SAP, tels que les instances de boîte de dialogue d’ABAP ou de la pile Java, doivent également s’exécuter sur Oracle Linux. Seuls des composants SAP à l’intérieur d’un tel système SAP qui ne se connecterait pas au SGBD Oracle seraient autorisés à exécuter un autre système d’exploitation Linux.
- Un RAC Oracle n’est pas pris en charge. 
- Un ASM Oracle est pris en charge dans certains cas. Des détails sont fournis dans la note
- Les systèmes SAP non-Unicode sont pris en charge uniquement avec des serveurs d’applications qui exécutent le système d’exploitation invité Windows. Le système d’exploitation invité du SGBD peut être Oracle Linux ou Windows. La raison de cette restriction apparaît évidente dans le Tableau de disponibilité des produits (PAM) SAP. Pour Oracle Linux, SAP n’a jamais publié de noyaux SAP non-Unicode

Connaissant les versions de SGBD prises en charge avec l’infrastructure Azure ciblée, vous devez vérifier dans le Tableau de disponibilité des produits SAP si les versions de système d’exploitation et le SGBD requis sont pris en charge avec les versions de votre produit SAP que vous souhaitez exécuter. 


## <a name="sap-hana-support"></a>Pris en charge de SAP HANA
Azure inclut deux services que vous pouvez utiliser pour exécuter une base de données HANA :

- Machines virtuelles Azure
- [Grandes instances HANA](./hana-overview-architecture.md)

Pour l’exécution de SAP HANA, SAP doit remplir des besoins d’infrastructure aux conditions plus nombreuses et plus fortes pour exécuter NetWeaver ou d’autres applications et SGBD SAP. Par conséquent, un plus petit nombre de machines virtuelles Azure sont éligibles pour l’exécution du SGBD SAP HANA. La liste des infrastructures Azure prises en charge pour SAP HANA figure dans le [répertoire de matériel SAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure). 

> [!NOTE]
> Les unités qui commencent par la lettre « S » sont des unités de [grandes instances HANA](./hana-overview-architecture.md). 

> [!NOTE]
> Aucune certification SAP spécifique ne dépend des versions majeures de SAP HANA. Contrairement à une idée reçue au sujet de la colonne **Scénario de certification** des [plateformes IaaS certifiées HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure), la colonne ne comprend **aucune instruction sur la version majeure ou mineure HANA certifiée**. Vous devez partir du principe que toutes les unités répertoriées peuvent être utilisées pour HANA 1.0 et HANA 2.0 tant que les versions certifiées du système d'exploitation des unités spécifiques sont également prises en charge par les versions HANA 1.0. 

Pour l’utilisation de SAP HANA, les versions minimales de système d’exploitation applicables peuvent différer de celles appropriées pour les cas généraux NetWeaver. Vous devez vérifier individuellement les systèmes d’exploitation pris en charge pour chaque unité, car ils peuvent varier. Pour ce faire, cliquez sur chaque unité. Cela a pour effet d’afficher des détails supplémentaires. Parmi ces détails figurent les différents systèmes d’exploitation pris en charge pour cette unité spécifique.

> [!NOTE]
> Les unités de grande instance Azure HANA sont plus restrictives en ce qui concerne les systèmes d’exploitation pris en charge que les machines virtuelles Azure. En revanche, des machines virtuelles Azure peuvent imposer des versions de système d’exploitation minimales plus récentes. C’est particulièrement vrai de certaines unités de machines virtuelles plus volumineuses qui ont nécessité des modifications des noyaux Linux

Connaissant le système d’exploitation pris en charge pour l’infrastructure Azure, vous devez vérifier dans la [Note de support SAP n°2235581](https://launchpad.support.sap.com/#/notes/2235581) les versions de SAP HANA et les niveaux de correctifs précis pris en charge avec les unités Azure que vous ciblez. 

> [!IMPORTANT]
> L’étape de vérification des versions de SAP HANA et des niveaux de correctifs précis pris en charge est très importante. Dans de nombreux cas, la prise en charge d’une certaine version de système d’exploitation dépend d’un niveau de correctif spécifique des exécutables SAP HANA.

Lorsque vous connaissez les versions de HANA spécifiques que vous pouvez exécuter sur l’infrastructure Azure ciblée, vous devez consulter le Tableau de disponibilité des produits SAP pour déterminer s’il existe des restrictions applicables aux versions de produits SAP qui prennent en charge les versions de HANA que vous avez filtrées.


## <a name="certified-azure-vms-and-hana-large-instance-units-and-business-transaction-throughput"></a>Machines virtuelles Azure certifiées, unités de grande instance HANA et débit de transactions
Outre l’évaluation des versions de système d’exploitation prises en charge, des versions de SGBD et des versions de logiciels SAP de support qui en dépendent pour les unités d’infrastructure Azure, vous devez qualifier ces unités par le débit de transactions exprimé dans l’unité « SAP » par SAP. Le dimensionnement SAP dépend intégralement des calculs SAP. En évaluant les systèmes SAP existants, vous pouvez généralement, avec l’aide de votre fournisseur d’infrastructure, calculer les SAP des unités. Pour la couche SGBD, ainsi que pour la couche application. Dans d’autres cas où de nouvelles fonctionnalités sont créées, un exercice de dimensionnement avec SAP peut révéler les chiffres de SAP requis pour la couche application et la couche SGBD. En tant que fournisseur d’infrastructure, Microsoft est tenu de fournir les caractéristiques de débit SAP des différentes unités certifiées NetWeaver et/ou HANA.

Pour des machines virtuelles Azure, ces chiffres de débit SAP figurent dans la [Note de support SAP n°1928533](https://launchpad.support.sap.com/#/notes/1928533). Pour les unités de grande instance Azure HANA, les chiffres de débit SAP figurent dans la [Note de support SAP n°2316233](https://launchpad.support.sap.com/#/notes/2316233)

La [Note de support SAP n°1928533](https://launchpad.support.sap.com/#/notes/1928533) soulève les remarques suivantes :

- **Pour les machines virtuelles Azure des séries M et MV2, les versions minimales de système d’exploitation applicables diffèrent de celles appropriées pour d’autres types de machines virtuelles Azure**. La configuration requise pour des versions de système d’exploitation plus récentes est basée sur les modifications que les différents fournisseurs de systèmes d’exploitation ont dû apporter à leurs versions de système d’exploitation afin de permettre à leurs systèmes d’exploitation de s’exécuter sur des types de machines virtuelles Azure spécifiques ou d’optimiser les performances et le débit de charge de travail SAP sur ces types de machines virtuelles.
- Il existe deux tableaux qui spécifient différents types de machines virtuelles. Le deuxième spécifie un débit SAP pour les types de machines virtuelles Azure qui prennent en charge uniquement le Stockage Standard Azure. Le déploiement de SGBD sur les unités spécifiées dans le deuxième tableau de la note n’est pas pris en charge.


## <a name="other-sap-products-supported-on-azure"></a>Autres produits SAP pris en charge sur Azure
L’hypothèse générale est que, compte tenu de l’état des clouds hyperscale tels qu’Azure, la plupart des logiciels SAP devraient s’exécuter sans problèmes fonctionnels dans Azure. Néanmoins, contrairement à la visualisation de cloud privé, SAP fait toujours explicitement état de la prise en charge des différents produits SAP de manière explicite pour les différents fournisseurs de cloud hyerscale. Il existe par conséquent différentes notes de support SAP évoquant la prise en charge par Azure de différents produits SAP. 

Pour la plateforme Business Objects BI, la [Note de support SAP n°2145537](https://launchpad.support.sap.com/#/notes/2145537) fournit une liste de produits Business Objects SAP pris en charge sur Azure. Si vous avez des questions concernant des composants ou des combinaisons de versions de logiciels et de système d’exploitation qui semblent ne pas être répertoriés ou pris en charge, et qui sont plus récents que les versions minimales énoncées, vous devez ouvrir une demande de support SAP concernant le support du composant sur lequel vous vous interrogez.

Pour les services de données Business Objects, la [Note de support SAP n°22288344](https://launchpad.support.sap.com/#/notes/2288344) explique la prise en charge minimale des SAP Data Services s’exécutant sur Azure. 

> [!NOTE]
> Comme indiqué dans la note de support SAP, vous devez consulter le PAM SAP afin d’identifier le niveau de package de support correct qu’Azure doit prendre en charge.

La prise en charge de SAP DataHub/Vora par Azure Kubernetes Service (AKS) est détaillée dans la [Note de support SAP n°2464722](https://launchpad.support.sap.com/#/notes/2464722)

La prise en charge de SAP BPC 10.1 SP08 est décrite dans la [Note de support SAP n°2451795](https://launchpad.support.sap.com/#/notes/2451795)

La prise en charge de SAP Hybris Commerce Platform sur Azure est détaillée dans la [documentation Hybris](https://help.sap.com/viewer/a74589c3a81a4a95bf51d87258c0ab15/1811/en-US/8c71300f866910149b40c88dfc0de431.html). La prise en charge du système SGBD pour SAP Hybris Commerce Platform se répartit comme suit :

- SQL Server et Oracle sur la plateforme du système d’exploitation Windows. Les versions minimales les plus récentes s’appliquent à SAP NetWeaver. Pour plus d’informations, consultez [la note n°1928533 sur la prise en charge de SAP](https://launchpad.support.sap.com/#/notes/1928533)
- SAP HANA sur Red Hat et SUSE Linux. Les types de machines virtuelles certifiés SAP HANA sont requis, comme indiqué plus haut dans [ce document](#sap-hana-support). SAP Hybris Commerce Platform est considéré comme une charge de travail OLTP
- SQL Azure DB à partir de SAP (Hybris) Commerce Platform version 1811




## <a name="next-steps"></a>Étapes suivantes
Découvrez les étapes suivantes dans [Planification et implémentation de machines virtuelles Azure pour SAP NetWeaver](./planning-guide.md)

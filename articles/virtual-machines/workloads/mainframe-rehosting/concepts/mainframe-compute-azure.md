---
title: Déplacer le calcul mainframe vers les machines virtuelles Azure
description: Comme les ressources de calcul Azure soutiennent largement la comparaison avec la capacité de l'ordinateur mainframe, vous pouvez migrer et moderniser les applications IBM z14.
author: njray
ms.author: larryme
ms.date: 04/02/2019
ms.topic: article
ms.service: multiple
ms.openlocfilehash: 04ec652c6e7ce8de003a464c6427439cf1519eab
ms.sourcegitcommit: 5db975ced62cd095be587d99da01949222fc69a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97092667"
---
# <a name="move-mainframe-compute-to-azure"></a>Déplacer le calcul mainframe vers Azure

Les mainframes sont réputés pour leur fiabilité et leur disponibilité élevées, et continuent d'être la dorsale de confiance de nombreuses entreprises. On pense souvent qu'ils offrent une évolutivité et une puissance de calcul presque illimitées. Mais certaines entreprises ont dépassé les capacités des plus grands mainframes disponibles. Si c’est votre cas, Azure offre des économies en termes d’agilité, d’infrastructure et de portée.

Pour exécuter des charges de travail mainframe sur Microsoft Azure, vous devez pouvoir établir une comparaison entre les fonctionnalités de calcul de votre mainframe et celles d’Azure. Basé sur un mainframe IBM z14 (le modèle le plus récent à ce jour), cet article vous explique comment obtenir des résultats comparables sur Azure.

Pour commencer, prenez en compte les environnements côte à côte. La figure suivante compare un environnement mainframe pour l'exécution d'applications sur un environnement d'hébergement Azure.

![Les services Azure et les environnements d'émulation offrent un support comparable et simplifient la migration](media/mainframe-compute-azure.png)

La puissance des mainframes est souvent utilisée pour les systèmes de traitement des transactions en ligne (OLTP) qui gèrent des millions de mises à jour pour des milliers d'utilisateurs. Ces applications utilisent souvent des logiciels pour le traitement des transactions, la gestion des écrans et la saisie des formulaires. Elles peuvent utiliser un système de type Customer Information Control System (CICS), Information Management System (IMS) ou Transaction Interface Package (TIP).

Comme le montre la figure, un émulateur TPM sur Azure peut gérer des charges de travail CICS et IMS. Un émulateur de système batch sur Azure joue le rôle de Job Control Language (JCL). Les données mainframe sont migrées vers les bases de données Azure, par exemple Azure SQL Database. Les services Azure ou d’autres logiciels hébergés dans les machines virtuelles Azure peuvent être utilisés pour la gestion du système.

## <a name="mainframe-compute-at-a-glance"></a>Aperçu du calcul mainframe

Dans le mainframe z14, les processeurs sont organisés en quatre *tiroirs*. Un *tiroir* représente simplement un cluster de processeurs et de chipsets. Chaque tiroir peut avoir six puces de processeur central (CP) actives, et chaque CP comporte 10 puces de contrôleur système (SC). Dans la terminologie Intel x86, il existe six sockets par tiroir, 10 cœurs par socket et quatre tiroirs. Cette architecture fournit l’équivalent approximatif de 24 sockets et 240 cœurs, maximum, pour un système z14.

Le CP z14 CP rapide affiche une fréquence d’horloge de 5,2 GHz. En règle générale, un système z14 est fourni avec tous les CP. Ils sont activés selon les besoins. Un client est généralement facturé au moins quatre heures de temps de calcul par mois, quelle que soit son utilisation réelle.

Un processeur mainframe peut être configuré avec l’un des types suivants :

- General Purpose (GP)
- Système z Integrated Information Processor (zIIP)
- Integrated Facility for Linux (IFL)
- System Assist Processor (SAP)
- Integrated Coupling Facility (ICF)

## <a name="scaling-mainframe-compute-up-and-out"></a>Montée en puissance et augmentation du calcul mainframe

Les mainframes IBM offrent la possibilité de mettre à l'échelle jusqu'à 240 cœurs (la taille z14 actuelle pour un seul système). De plus, les mainframes IBM peuvent effectuer un scale-out grâce à une fonctionnalité appelée Coupling Facility (CF). La fonctionnalité CF permet à plusieurs systèmes mainframe d'accéder simultanément aux mêmes données. Grâce à CF, la technologie Mainframe Parallel Sysplex regroupe les processeurs mainframe en clusters. Au moment de la rédaction de ce guide, la fonction Parallel Sysplex prenait en charge 32 groupes de 64 processeurs chacun. Jusqu'à 2 048 processeurs peuvent être regroupés de cette manière pour effectuer un scale-out de la capacité de calcul.

CF permet aux clusters de calcul de partager des données avec un accès direct. Elle est utilisé pour verrouiller les informations, les informations en cache et la liste des ressources de données partagées. Un système Parallel Sysplex avec une ou plusieurs CF peut être considéré comme un cluster de calcul « tout partagé » avec montée en puissance. Pour plus d’informations sur ces fonctionnalités, consultez [Parallel Sysplex sur IBM Z](https://www.ibm.com/it-infrastructure/z/technologies/parallel-sysplex-resources) sur le site web d’IBM.

Les applications peuvent utiliser ces fonctionnalités pour fournir à la fois des performances de montée en puissance et une haute disponibilité. Pour plus d’informations sur la façon dont CICS peut utiliser Parallel Sysplex avec CF, téléchargez le livre rouge [IBM CICS and the Coupling Facility: Beyond the Basics](https://www.redbooks.ibm.com/redbooks/pdfs/sg248420.pdf).

## <a name="azure-compute-at-a-glance"></a>Aperçu du calcul Azure

Certains pensent à tort que les serveurs basés sur Intel ne sont pas aussi puissants que les mainframes. Cependant, les nouveaux systèmes multicœurs Intel ont une capacité de calcul égale à celle des ordinateurs mainframe. Cette section décrit les options Azure infrastructure-as-a-service (IaaS) pour le calcul et le stockage. Azure propose également des options platform-as-a-service (PaaS), mais cet article se concentre sur les options IaaaS qui offrent une capacité mainframe comparable.

Les machines virtuelles Azure fournissent une puissance de calcul dans une large gamme de tailles et de types. Dans Azure, un CPU virtuel (vCPU) équivaut à peu près à un cœur sur un mainframe.

Actuellement, l’éventail des tailles de machines virtuelles Azure s’étend de 1 à 128 vCPU. Les types de machines virtuelles (VM) sont optimisés pour des charges de travail spécifiques. Par exemple, la liste suivante montre les types de machine virtuelle (à jour au moment de la rédaction du présent document) et leurs utilisations recommandées :

| Taille     | Type et description                                                                 |
|----------|--------------------------------------------------------------------------------------|
| Série D | Usage général avec 64 vCPU et jusqu'à 3,5 GHz de fréquence d’horloge                           |
| Série E | Optimisé pour la mémoire avec jusqu'à 64 vCPU                                                 |
| Série F | Optimisé pour le calcul avec jusqu'à 64 vCPU et une fréquence d’horloge de 3,7 GHz                       |
| Série H | Optimisé pour les applications de calcul haute performance (HPC)                          |
| Série L | Optimisé pour le stockage pour les applications à haut débit s’appuyant sur des bases de données comme NoSQL |
| Série M | Les plus grosses machines virtuelles optimisées pour le calcul et la mémoire, avec jusqu'à 128 vCPU                        |

Pour en savoir plus sur les machines virtuelles disponibles, consultez [Série de machines virtuelles](https://azure.microsoft.com/pricing/details/virtual-machines/series/).

Un mainframe z14 peut avoir jusqu'à 240 cœurs. Cependant, les mainframes z14 n'utilisent presque jamais tous les cœurs pour une même application ou charge de travail. Au lieu de cela, un mainframe sépare les charges de travail en partitions logiques (LPAR), et les LPAR comportent différents niveaux appelés MIPS (Millions of Instructions Per Second) ou MSU (Million Service Unit). Pour déterminer la taille d’une machine virtuelle comparable nécessaire pour exécuter une charge de travail mainframe sur Azure, prenez en compte le niveau MIPS (ou MSU).

Les chiffres suivants sont des estimations générales :

-   150 MIPS par vCPU

-   1 000 MIPS par processeur

Afin de déterminer la taille correcte de la machine virtuelle pour une charge de travail donnée dans une partition logique, commencez par optimiser la machine virtuelle pour cette charge de travail. Déterminez ensuite le nombre de vCPU nécessaires. Une estimation prudente recommande 150 MIPS par vCPU. Sur la base de cette estimation, par exemple, une machine virtuelle de série F avec 16 vCPU pourrait facilement gérer une charge de travail IBM Db2 provenant d'un LPAR avec 2 400 MIPS.

## <a name="azure-compute-scale-up"></a>Montée en puissance du calcul Azure

Les machines virtuelles de la série M peuvent monter en puissance jusqu'à 128 vCPU (au moment de la rédaction de cet article). En utilisant l'estimation prudente de 150 MIPS par vCPU, la machine virtuelle de la série M équivaut à environ 19 000 MIPS. La règle générale pour l’estimation MIPS d’un mainframe est de compter 1 000 MIPS par processeur. Un mainframe z14 peut avoir jusqu'à 24 processeurs et fournir environ 24 000 MIPS pour un même système mainframe.

Le plus gros mainframe z14 possède environ 5 000 MIPS de plus que la plus grosse machine virtuelle disponible sur Azure. Mais il est important de comparer la façon dont les charges de travail sont déployées. Si un système mainframe possède à la fois une application et une base de données relationnelle, ces éléments sont généralement déployés sur le même mainframe physique, chacun dans leur propre LPAR. La même solution sur Azure est souvent déployée en utilisant une machine virtuelle pour l'application, avec une machine distincte et de taille adaptée pour la base de données.

Par exemple, si un système vCPU M64 prend en charge l'application et qu'un vCPU M96 est utilisé pour la base de données, environ 150 vCPU sont nécessaires, soit à peu près 24 000 MIPS, comme le montre la figure suivante.

![Comparaison de déploiements de charges de travail de 24 000 MIPS](media/mainframe-compute-mips.png)

L'approche consiste à migrer les LPAR vers des machines virtuelles individuelles. Azure peut ensuite facilement monter en puissance à la taille requise pour la plupart des applications déployées sur un même système mainframe.

## <a name="azure-compute-scale-out"></a>Montée en charge du calcul Azure

L'un des avantages d'une solution basée sur Azure est la possibilité d’effectuer un scale-out. Cette montée en charge permet à une application de disposer d'une capacité de calcul presque illimitée. Azure prend en charge plusieurs méthodes pour effectuer un scale-out de la puissance de calcul :

- **Équilibrage de la charge sur un cluster.** Dans ce scénario, une application peut utiliser un [ équilibreur de charge](../../../../load-balancer/load-balancer-overview.md) ou un gestionnaire de ressources pour répartir la charge de travail entre plusieurs machines virtuelles dans un cluster. Si une plus grande capacité de calcul est nécessaire, des machines virtuelles supplémentaires sont ajoutées au cluster.

- **Jeux de mise à l’échelle de machine virtuelle.** Dans ce scénario de rafales, une application peut monter en charge avec d’autres [ressources de calcul](../../../../virtual-machine-scale-sets/overview.md) selon l’utilisation de la machine virtuelle. Lorsque la demande diminue, le nombre de machines virtuelles d'un groupe identique peut également diminuer, ce qui garantit une utilisation efficace de la puissance de calcul.

- **Mise à l’échelle PaaS.** Les offres PaaS d'Azure mettent à l'échelle les ressources de calcul. Par exemple, [Azure Service Fabric](../../../../service-fabric/service-fabric-overview.md) alloue des ressources de calcul pour répondre à l'augmentation du volume des demandes.

- **Clusters Kubernetes.** Les applications sur Azure peuvent utiliser des [clusters Kubernetes](../../../../aks/concepts-clusters-workloads.md) afin de calculer des services pour des ressources spécifiques. Azure Kubernetes Service (AKS) est un service géré qui g_re les nœuds, les pools et les clusters Kubernetes sur Azure.

Pour choisir la bonne méthode de montée en charge des ressources de calcul, il est important de comprendre en quoi Azure et les mainframes diffèrent. L’élément le plus important est comment - ou si - les données sont partagées par les ressources de calcul. Dans Azure, les données (par défaut) ne sont généralement pas partagées par plusieurs machines virtuelles. Si un partage de données est requis par plusieurs machines virtuelles dans un cluster de calcul avec montée en charge, les données partagées doivent résider dans une ressource qui prend en charge cette fonctionnalité. Sur Azure, le partage de données implique un stockage, comme nous le verrons dans la section suivante.

## <a name="azure-compute-optimization"></a>Optimisation du calcul Azure

Vous pouvez optimiser chaque niveau de traitement dans une architecture Azure. Utilisez les machines virtuelles et les fonctionnalités les mieux adaptées à chaque environnement. La figure suivante montre un modèle potentiel de déploiement de machines virtuelles dans Azur pour prendre en charge une application CICS qui utilise Db2. Dans le site principal, les machines virtuelles de test, de préproduction et de production sont déployées avec une haute disponibilité. Le site secondaire est utilisé pour la sauvegarde et pour la reprise d’activité après sinistre.

Chaque niveau peut fournir des services de reprise d’activité après sinistre. Par exemple, les machines virtuelles de base de données et de production peuvent nécessiter une reprise d’activité à chaud, alors que les machines virtuelles de développement et de test prennent en charge une reprise d’activité à froid.

![Déploiement hautement disponible prenant en charge la récupération d'urgence](media/mainframe-compute-dr.png)

## <a name="next-steps"></a>Étapes suivantes

- [Migration de mainframe](/azure/architecture/cloud-adoption/infrastructure/mainframe-migration/overview)
- [Réhébergement du mainframe sur des machines virtuelles Azure](../overview.md)
- [Déplacement du stockage du mainframe vers Azure](mainframe-storage-Azure.md)

### <a name="ibm-resources"></a>Ressources IBM

- [Parallel Sysplex sur IBM Z](https://www.ibm.com/it-infrastructure/z/technologies/parallel-sysplex-resources)
- [IBM CICS and the Coupling Facility: Beyond the Basics](https://www.redbooks.ibm.com/redbooks/pdfs/sg248420.pdf)
- [Création des utilisateurs requis pour l’installation d’une fonctionnalité DB2 pureScale](https://www.ibm.com/support/knowledgecenter/en/SSEPGG_11.1.0/com.ibm.db2.luw.qb.server.doc/doc/t0055374.html?pos=2)
- [DB2icrt - Commande Créer une instance](https://www.ibm.com/support/knowledgecenter/en/SSEPGG_11.1.0/com.ibm.db2.luw.admin.cmd.doc/doc/r0002057.html)
- [Solution de base de données de clusters Db2 pureScale](https://www.ibmbigdatahub.com/blog/db2-purescale-clustered-database-solution-part-1)
- [IBMData Studio](https://www.ibm.com/developerworks/downloads/im/data/index.html/)

### <a name="azure-government"></a>Azure Government

- [Cloud Microsoft Azure Government pour applications mainframe](https://azure.microsoft.com/resources/microsoft-azure-government-cloud-for-mainframe-applications/)
- [Microsoft et FedRAMP](https://www.microsoft.com/TrustCenter/Compliance/FedRAMP)

### <a name="more-migration-resources"></a>Autres ressources de migration

- [Azure Virtual Datacenter: Lift and Shift Guide](https://azure.microsoft.com/resources/azure-virtual-datacenter-lift-and-shift-guide/)
- [iSCSI GlusterFS](https://glusterdocs.readthedocs.io/en/latest/Administrator%20Guide/GlusterFS%20iSCSI/)

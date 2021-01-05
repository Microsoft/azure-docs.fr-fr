---
title: Distributions Linux approuvées sur Azure
description: Découvrez les distributions Linux approuvées sur Azure, notamment des instructions pour Ubuntu, CentOS, Oracle et SUSE.
services: virtual-machines-linux
documentationcenter: ''
author: danielsollondon
manager: gwallace
tags: azure-service-management,azure-resource-manager
ms.assetid: 2777a526-c260-4cb9-a31a-bdfe1a55fffc
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.topic: conceptual
ms.date: 10/09/2020
ms.author: guybo
ms.openlocfilehash: b27b7344d84ce1361d8294fa4f3490c50afbb4c3
ms.sourcegitcommit: b4880683d23f5c91e9901eac22ea31f50a0f116f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94489656"
---
# <a name="endorsed-linux-distributions-on-azure"></a>Distributions Linux approuvées sur Azure

Les partenaires fournissent des images Linux dans la Place de marché Azure. Microsoft collabore avec différentes communautés Linux pour étoffer la liste des distributions approuvée. Pour les distributions qui ne sont pas disponibles sur la Place de marché, vous pouvez toujours apporter votre propre distribution Linux en suivant les instructions fournies dans [Créer et charger un disque dur virtuel contenant le système d’exploitation Linux](./create-upload-generic.md).

## <a name="supported-distributions-and-versions"></a>Distributions et versions prises en charge

Le tableau suivant répertorie les distributions et les version Linux prises en charge sur Microsoft Azure. Pour plus d’informations, consultez [Prise en charge d’images Linux dans Microsoft Azure](https://support.microsoft.com/help/2941892/support-for-linux-and-open-source-technology-in-azure).

Les pilotes LIS (Linux Integration Services) pour Hyper-V et Azure sont des modules noyau que Microsoft distribue directement au noyau Linux en amont. Certains pilotes LIS sont intégrés dans le noyau de la distribution par défaut. D’anciennes distributions basées sur Red Hat Enterprise (RHEL)/CentOS sont disponibles sous la forme de téléchargements séparés à partir de la page [Linux Integration Services Version 4.2 for Hyper-V and Azure](https://www.microsoft.com/download/details.aspx?id=55106) (Services d’intégration Linux version 4.2 pour Hyper-V et Azure). Pour plus d’informations, consultez [Conditions requises pour le noyau Linux](create-upload-generic.md#linux-kernel-requirements).

L’agent Linux Azure est pré-installé sur les images de la Place de marché Azure et est généralement disponible dans le référentiel du package de la distribution. Le code source est disponible sur [GitHub](https://github.com/azure/walinuxagent).

| Distribution | Version | Pilotes | Agent |
| --- | --- | --- | --- |
| CentOS par Rogue Wave Software |CentOS 6.x, 7.x, 8.x |CentOS 6.3 : [Téléchargement de LIS](https://www.microsoft.com/download/details.aspx?id=55106)<p>CentOS 6.4+ : Dans le noyau |Package : dans le [dépôt](http://olcentgbl.trafficmanager.net/openlogic/6/openlogic/x86_64/RPMS/) sous « WALinuxAgent » <br/>Code source : [GitHub](https://github.com/Azure/WALinuxAgent) |
| [CoreOS](https://coreos.com/docs/running-coreos/cloud-providers/azure/)<p> Coreos est désormais en [fin de vie](https://coreos.com/os/eol/) à compter du 26 mai 2020. |Plus disponible | | |
| Debian par Credativ |8.x, 9.x |Dans le noyau |Package : dans le dépôt sous « waagent » <br/>Code source : [GitHub](https://github.com/Azure/WALinuxAgent) |
|Conteneur flatcar Linux par Kinvolk| Pro, Stable, Bêta| Dans le noyau | wa-linux-agent est déjà installé dans/usr/share/oem/bin/waagent |
| Oracle Linux par Oracle |6.x, 7.x, 8.x |Dans le noyau |Package : dans le dépôt sous « WALinuxAgent » <br/>Code source : [GitHub](https://go.microsoft.com/fwlink/p/?LinkID=250998) |
| [Red Hat Enterprise Linux par Red Hat](../workloads/redhat/overview.md) |6.x, 7.x, 8.x |Dans le noyau |Package : dans le dépôt sous « WALinuxAgent » <br/>Code source : [GitHub](https://github.com/Azure/WALinuxAgent) |
| SUSE Linux Enterprise par SUSE |SLES/SLES pour SAP 11.x, 12.x, 15.x <br/> [Cycle de vie d’une image de cloud public SUSE](https://www.suse.com/c/suse-public-cloud-image-life-cycle/) |Dans le noyau |Package :<p> pour 11 dans le référentiel [Cloud:Tools](https://build.opensuse.org/project/show/Cloud:Tools)<br>pour 12 inclus dans le module « Cloud public » sous « python-azure-agent »<br/>Code source : [GitHub](https://go.microsoft.com/fwlink/p/?LinkID=250998) |
| openSUSE par SUSE |openSUSE Leap 15.x |Dans le noyau |Package : dans le dépôt [Cloud:Tools](https://build.opensuse.org/project/show/Cloud:Tools) sous « python-azure-agent » <br/>Code source : [GitHub](https://github.com/Azure/WALinuxAgent) |
| Ubuntu par Canonical |Ubuntu Server et Pro. 16.x, 18.x, 20.x<p>Pour plus d’informations sur la prise en charge étendue pour Ubuntu 12.04 et 14.04, consultez : [Ubuntu Extended Security Maintenance](https://www.ubuntu.com/esm). |Dans le noyau |Package : dans le dépôt sous « walinuxagent » <br/>Code source : [GitHub](https://github.com/Azure/WALinuxAgent) |

## <a name="image-update-cadence"></a>Cadence de mise à jour d’image

Azure nécessite des éditeurs des distributions Linux approuvées qu’ils mettent régulièrement à jour leurs images dans la Place de marché Azure avec les derniers correctifs et résolutions de sécurité, au moins une fois par trimestre. Les images mises à jour dans la Place de marché sont disponibles automatiquement pour les clients en tant que nouvelles versions d’une référence (SKU) d’image. Plus d’informations sur la recherche d’images Linux : [Rechercher des images de machine virtuelle Linux sur la Place de marché Azure](./cli-ps-findimage.md).

## <a name="azure-tuned-kernels"></a>Kernels optimisés pour Azure

Azure travaille en collaboration étroite avec diverses distributions Linux approuvées afin d’optimiser les images qu’elles publient sur la Place de marché Azure. L’un des aspects de cette collaboration se traduit par le développement de kernels Linux « optimisés » pour la plateforme Azure et proposés comme composants entièrement pris en charge de la distribution Linux. Les kernels optimisés pour Azure intègrent de nouvelles améliorations en matière de fonctionnalités et de performances, et de façon plus régulière (une fois par trimestre) comparé à des kernels génériques ou par défaut disponibles.

Dans la plupart des cas, vous trouverez ces kernels préinstallés sur les images par défaut disponibles sur la Place de marché Azure, de sorte que les clients profitent immédiatement des avantages qu’ils procurent. Pour en savoir plus sur ces kernels optimisés pour Azure, suivez ces liens :

- [Kernel optimisé pour Azure CentOS - Disponible via le SIG de virtualisation CentOS](https://wiki.centos.org/SpecialInterestGroup/Virtualization)
- [Kernel de cloud Debian - Disponible avec les images « backports » Debian 10 et Debian 9 sur Azure](https://wiki.debian.org/Cloud/MicrosoftAzure)
- [Kernel SLES optimisé pour Azure](https://www.suse.com/c/a-different-builtin-kernel-for-azure-on-demand-images/)
- [Kernel Ubuntu optimisé pour Azure](https://blog.ubuntu.com/2017/09/21/microsoft-and-canonical-increase-velocity-with-azure-tailored-kernel)
- [Flatcar Container Linux Pro](https://azuremarketplace.microsoft.com/marketplace/apps/kinvolk.flatcar_pro)

## <a name="partners"></a>Partenaires

### <a name="coreos"></a>CoreOS

Coreos est planifié pour être en [fin de vie](https://coreos.com/os/eol/) d’ici le 26 mai 2020.
Microsoft a deux (2) canaux de migration pour les utilisateurs CoreOS.

- Flatcar par Kinvolk (Voir l’entrée « Conteneur Flatcar Linux par Kinvolk ».)
- [Noyau de système d’exploitation Fedora](https://docs.fedoraproject.org/en-US/fedora-coreos/provisioning-azure/) (les clients doivent charger leur propre image. Vous trouverez ici la [documentation relative à la migration](https://docs.fedoraproject.org/en-US/fedora-coreos/migrate-cl/)).

### <a name="credativ"></a>credativ

[https://www.credativ.de/en/portfolio/support/open-source-support-center/](https://www.credativ.de/en/portfolio/support/open-source-support-center/)

credativ est une société de conseil et de services indépendante, spécialisée dans le développement et l’implémentation de solutions professionnelles à l’aide de logiciels libres. En tant que leader spécialiste de l’Open Source, credativ a acquis une reconnaissance internationale et de nombreux services informatiques font appel à ses services. En collaboration avec Microsoft, credativ prépare les images Debian. Les images sont spécialement conçues pour s’exécuter sur Azure et peuvent être facilement gérées via la plateforme. credativ prendra également en charge la maintenance et la mise à jour sur le long terme des images Debian pour Azure grâce à ses centres de support Open Source.

### <a name="kinvolk"></a>Kinvolk
[https://www.kinvolk.io/flatcar-container-linux/](https://www.kinvolk.io/flatcar-container-linux/)

Kinvolk est la société derrière Flatcar Container Linux, qui poursuit la vision de Coreos d’origine pour une base de mise à jour minimale, immuable et automatique pour des applications en conteneurs. En tant que distribution minimale, Flatcar contient uniquement les packages requis pour le déploiement de conteneurs. Son système de fichiers immuable garantit la cohérence et la sécurité, tandis que ses fonctionnalités de mise à jour automatique vous permettent d’être toujours à jour avec les derniers correctifs de sécurité. 

Flatcar Container Linux bénéficie du soutien de l’équipe internationale Kinvolk de Linux, et d’experts en technologie des conteneurs qui proposent un abonnement de support commercial facultatif qui inclut une réponse 24x7, des alertes techniques et de sécurité, ainsi que des images optimisées Azure exclusives, notamment un canal de support à long terme.


### <a name="oracle"></a>Oracle

[https://www.oracle.com/technetwork/topics/cloud/faq-1963009.html](https://www.oracle.com/technetwork/topics/cloud/faq-1963009.html)

La stratégie d’Oracle est d’offrir un vaste portefeuille de solutions pour les clouds privés et publics. La stratégie offre aux clients de choix et la flexibilité leur permettant de décider comment déployer les logiciels Oracle dans des clouds Oracle et autres. À la faveur du partenariat conclu entre Oracle et Microsoft, les clients peuvent déployer les logiciels Oracle dans les clouds publics et privés Microsoft tout en étant assurés de bénéficier de la certification et du support d'Oracle.  L'engagement et l'investissement d'Oracle vis-à-vis des solutions de cloud public et privé Oracle sont intacts.

### <a name="red-hat"></a>Red Hat

[https://www.redhat.com/en/partners/strategic-alliance/microsoft](https://www.redhat.com/en/partners/strategic-alliance/microsoft)

Le leader mondial de solutions open source, Red Hat permet à plus de 90 % des sociétés du classement Fortune 500 de résoudre les défis commerciaux, d’aligner leur stratégies informatique et commerciale, et d’anticiper l’évolution des technologies. Red Hat y parvient en proposant des solutions sécurisées grâce à un modèle commercial ouvert et à un modèle d’abonnement abordable et prévisible.

### <a name="suse"></a>SUSE

[https://www.suse.com/suse-linux-enterprise-server-on-azure](https://www.suse.com/suse-linux-enterprise-server-on-azure)

SUSE Linux Enterprise Server sur Azure est une plateforme éprouvée qui offre une fiabilité et un niveau de sécurité supérieurs pour le cloud computing. Polyvalente, la plateforme Linux de SUSE s'intègre en toute transparence aux services cloud Azure pour fournir un environnement cloud facile à gérer. Avec plus de 9 200 applications certifiées issues de plus de 1 800 éditeurs de logiciels indépendants pour SUSE Linux Enterprise Server, SUSE est l’assurance que les charges de travail prises en charge dans les centres de données peuvent être déployées en toute confiance sur Azure.

### <a name="canonical"></a>Canonical

[https://www.ubuntu.com/cloud/azure](https://www.ubuntu.com/cloud/azure)

L’ingénierie de Canonical et le mode de gouvernance de la communauté Open Source sont les éléments moteurs de la réussite d’Ubuntu dans les environnements client, serveur et cloud, y compris les services cloud personnels à destination du grand public. La vision de Canonical d’une plateforme unifiée gratuite disponible dans Ubuntu, offre une gamme d’interfaces cohérentes pour téléphones, tablettes, TV et ordinateurs de bureau. Cette vision fait d’Ubuntu le premier choix pour divers établissements, des fournisseurs de cloud public aux fabricant de produits électroniques de grande consommation, et l’un des choix de prédilection des technologues individuels.

Avec des développeurs et des centres d’ingénierie présents dans le monde entier, Canonical est particulièrement bien placé pour collaborer avec les fabricants de matériel, les fournisseurs de contenu et les développeurs de logiciels pour mettre sur le marché des solutions Ubuntu destinées tant aux PC, qu’aux serveurs ou aux appareils portables.

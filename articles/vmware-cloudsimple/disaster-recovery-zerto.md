---
title: Azure VMware Solution by CloudSimple - Utiliser le cloud privé comme site de reprise d’activité pour les charges de travail locales
description: Explique comment configurer votre cloud privé CloudSimple en tant que site de reprise d’activité pour les charges de travail VMware locales.
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/20/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 0e019a9229b671be2fb73e758bd39f33657bc2d4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77083135"
---
# <a name="set-up-cloudsimple-private-cloud-as-a-disaster-recovery-site-for-on-premises-vmware-workloads"></a>Configurer votre cloud privé CloudSimple comme site de reprise d’activité pour les charges de travail VMware locales

Vous pouvez configurer votre cloud privé CloudSimple en tant que site de récupération pour les applications locales afin d’assurer la continuité des activités en cas de sinistre. La solution de récupération est basée sur Zerto Virtual Replication comme plateforme de réplication et d’orchestration. Les machines virtuelles d’infrastructure et d’applications critiques peuvent être répliquées en continu de votre vCenter local vers votre cloud privé. Vous pouvez utiliser votre cloud privé pour tester le basculement et garantir la disponibilité de votre application en cas de sinistre. Vous pouvez adopter une approche similaire pour configurer le cloud privé en tant que site principal protégé par un site de récupération à un autre emplacement.

> [!NOTE]
> Pour obtenir des instructions sur le dimensionnement de votre environnement de reprise d’activité après sinistre, reportez-vous au document Zerto intitulé [Sizing Considerations For Zerto Virtual Replication](https://s3.amazonaws.com/zertodownload_docs/5.5U3/Zerto%20Virtual%20Replication%20Sizing.pdf) (Considérations relatives au dimensionnement pour Zerto Virtual Replication).

La solution CloudSimple :

* Élimine la nécessité de configurer un centre de données spécifiquement dédié à la reprise d’activité.
* Vous permet de tirer parti des emplacements Azure où CloudSimple est déployé pour une résilience géographique à l’échelle mondiale.
* Vous permet de réduire les coûts de déploiement et le coût total de possession de la reprise d’activité.

La solution nécessite que vous :

* Installiez, configuriez et gériez Zerto dans votre cloud privé.
* Fournissiez vos propres licences pour Zerto quand le cloud privé est le site protégé. Vous pouvez associer Zerto en cours d’exécution sur le site CloudSimple à votre site local pour la gestion des licences.

La figure suivante illustre l’architecture de la solution Zerto.

![Architecture](media/cloudsimple-zerto-architecture.png)

## <a name="how-to-deploy-the-solution"></a>Procédure : déployer la solution

Les sections suivantes expliquent comment déployer une solution de reprise d’activité à l’aide de Zerto Virtual Replication dans votre cloud privé.

1. [Prérequis](#prerequisites)
2. [Configuration facultative sur le cloud privé CloudSimple](#optional-configuration-on-your-private-cloud)
3. [Configurer ZVM et VRA sur le cloud privé CloudSimple](#set-up-zvm-and-vra-on-your-private-cloud)
4. [Configurer le groupe de protection virtuel Zerto](#set-up-zerto-virtual-protection-group)

### <a name="prerequisites"></a>Conditions préalables requises

Pour activer Zerto Virtual Replication de votre environnement local vers votre cloud privé, effectuez les étapes requises suivantes.

1. [Configurer une connexion VPN de site à site entre votre réseau local et votre cloud privé CloudSimple](set-up-vpn.md).
2. [Configurer la recherche DNS afin que vos composants de gestion de cloud privé soient transférés vers les serveurs DNS du cloud privé](on-premises-dns-setup.md).  Pour activer le transfert de la recherche DNS, créez une entrée de zone de transfert sur votre serveur DNS local pour `*.cloudsimple.io` vers les serveurs DNS CloudSimple.
3. Configurez la recherche DNS afin que les composants vCenter locaux soient transférés vers des serveurs DNS locaux.  Les serveurs DNS doivent être accessibles à partir de votre cloud privé CloudSimple par le biais d’un VPN de site à site. Pour obtenir de l’aide, envoyez une [demande de support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) et fournissez les informations suivantes.  

    * Nom de domaine DNS local
    * Adresses IP des serveurs DNS locaux

4. Installez un serveur Windows Server sur votre cloud privé. Le serveur est utilisé pour installer Zerto Virtual Manager.
5. [Augmentez vos privilèges CloudSimple](escalate-private-cloud-privileges.md).
6. Créez un utilisateur sur votre cloud privé vCenter avec le rôle d’administrateur, que vous utiliserez comme compte de service pour Zerto Virtual Manager.

### <a name="optional-configuration-on-your-private-cloud"></a>Configuration facultative sur votre cloud privé

1. Créez un ou plusieurs pools de ressources sur votre cloud privé vCenter à utiliser comme pools de ressources cibles pour les machines virtuelles de votre environnement local.
2. Créez un ou plusieurs dossiers sur votre cloud privé vCenter à utiliser comme dossiers cibles pour les machines virtuelles de votre environnement local.
3. Créez des réseaux locaux virtuels pour le réseau de basculement et configurez des règles de pare-feu. Ouvrez une [demande de support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) pour obtenir de l’aide.
4. Créez des groupes de ports distribués pour le réseau de basculement et le réseau de test, afin de tester le basculement des machines virtuelles.
5. Installez des [serveurs DHCP et DNS](dns-dhcp-setup.md) ou utilisez un contrôleur de domaine Active Directory dans votre environnement de cloud privé.

### <a name="set-up-zvm-and-vra-on-your-private-cloud"></a>Configurer ZVM et VRA sur votre cloud privé

1. Installez Zerto Virtual Manager (ZVM) sur le serveur Windows Server dans votre cloud privé.
2. Connectez-vous à ZVM à l’aide du compte de service créé lors des étapes précédentes.
3. Configurez la gestion des licences pour Zerto Virtual Manager.
4. Installez Zerto Virtual Replication Appliance (VRA) sur les hôtes ESXi de votre cloud privé.
5. Couplez votre ZVM de cloud privé à votre ZVM local.

### <a name="set-up-zerto-virtual-protection-group"></a>Configurer le groupe de protection virtuel Zerto

1. Créez un groupe de protection virtuel (VPG) et spécifiez sa priorité.
2. Sélectionnez les machines virtuelles qui nécessitent une protection pour la continuité des activités, et personnalisez l’ordre de démarrage si nécessaire.
3. Sélectionnez votre cloud privé comme site de récupération, et le cluster de clouds privés ou le groupe de ressources que vous avez créé comme serveur de récupération par défaut. Sélectionnez **vsanDatastore** comme banque de données de récupération sur votre cloud privé.

    ![VPG](media/cloudsimple-zerto-vpg.png)

    > [!NOTE]
    > Vous pouvez personnaliser l’option d’hôte pour des machines virtuelles spécifiques sous l’option VM Settings (Paramètres de machines virtuelles).

4. Personnalisez les options de stockage en fonction de vos besoins.
5. Spécifiez les groupes de ports distribués créés plus tôt comme réseaux de récupération à utiliser pour le réseau de basculement et le réseau de test de basculement, et personnalisez les scripts de récupération en fonction des besoins.
6. Personnalisez les paramètres réseau pour des machines virtuelles spécifiques si nécessaire, puis créez le VPG.
7. Testez le basculement une fois la réplication terminée.

## <a name="reference"></a>Informations de référence

[Documentation Zerto](https://www.zerto.com/myzerto/technical-documentation/)

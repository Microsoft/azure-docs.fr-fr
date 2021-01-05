---
title: Réhébergement de mainframe sur des machines virtuelles Azure
description: Réhéberger vos charges de travail mainframe comme les systèmes IBM Z en utilisant des machines virtuelles sur Microsoft Azure.
services: virtual-machines-linux
documentationcenter: ''
author: njray
manager: edprice
editor: edprice
ms.author: larryme
ms.date: 04/02/2019
ms.topic: article
ms.service: multiple
ms.openlocfilehash: d754f27a1510d9db0837eabb96b3a7cf75f76fc1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87835525"
---
# <a name="mainframe-rehosting-on-azure-virtual-machines"></a>Réhébergement de mainframe sur des machines virtuelles Azure

La migration de charges de travail à partir d’environnements mainframe vers le cloud vous permet de moderniser votre infrastructure et souvent de réduire les coûts. De nombreuses charges de travail peuvent être transférées vers Azure suite à quelques modifications mineures du code (mettre à jour le nom des bases de données, par exemple).

En règle générale, le terme *mainframe* désigne un système informatique de grande envergure. Plus précisément, la grande majorité des ordinateurs mainframe actuellement utilisés sont des serveurs IBM System Z ou des systèmes compatibles IBM exécutant MVS, DOS, VSE, OS/390 ou z/OS.

Une machine virtuelle Azure est utilisée pour isoler et gérer les ressources pour une application spécifique sur une seule instance. Les ordinateurs mainframe tels qu’IBM z/OS utilisent des partitions logiques à cette fin. Un mainframe peut utiliser une partition logique pour une région CICS et les programmes COBOL associés et une partition LPAR séparée pour une base de données IBM Db2. Une [application multiniveau classique sur Azure](/azure/architecture/reference-architectures/n-tier/n-tier-sql-server) déploie des machines virtuelles Azure dans un réseau virtuel qui peut être subdivisé en sous-réseaux pour chaque niveau.

Les machines virtuelles Azure peuvent exécuter des environnements d’émulation de mainframe et des compilateurs qui prennent en charge les scénarios lift-and-shift. Le développement et les tests font souvent partie des premières charges de travail à migrer d’un mainframe vers un environnement de développement/test Azure. Les composants serveur communs que vous pouvez émuler incluent les systèmes de traitement transactionnel en ligne (OLTP), de traitement par lots et d’ingestion des données, comme illustré dans la figure suivante.

![Les environnements d’émulation sur Azure vous permettent d’exécuter des systèmes basés sur z/OS.](media/01-overview.png)

Certaines charges de travail mainframe peuvent être migrées vers Azure assez facilement. D’autres peuvent être réhébergées sur Azure à l’aide d’une solution partenaire. Pour obtenir des instructions détaillées sur le choix d’une solution partenaire, vous pouvez vous appuyer sur le [centre dédié aux migrations de mainframe Azure](https://azure.microsoft.com/migration/mainframe/).

## <a name="mainframe-migration"></a>Migration de mainframe

Réhéberger, régénérer, remplacer ou mettre hors service ? IaaS ou PaaS ? Pour déterminer la stratégie de migration appropriée pour votre application de mainframe, consultez le guide sur la [migration de mainframe](/azure/architecture/cloud-adoption/infrastructure/mainframe-migration/overview) dans le Centre des architectures Azure.

## <a name="micro-focus-rehosting-platform"></a>Plateforme de réhébergement Micro Focus

Micro Focus Enterprise Server est l’une des plus grandes plateformes de réhébergement de mainframe disponibles. Vous pouvez l’utiliser pour exécuter vos charges de travail z/OS sur une plateforme x86 moins coûteuse sur Azure.

Pour commencer :

- [Install Enterprise Server and Enterprise Developer on Azure](./microfocus/set-up-micro-focus-azure.md) (Installer Enterprise Server et Enterprise Developer sur Azure)
- [Set up CICS BankDemo for Enterprise Developer on Azure](./microfocus/demo.md) (Configurer CICS BankDemo pour Enterprise Developer sur Azure)
- [Run Enterprise Server in a Docker Container on Azure](./microfocus/run-enterprise-server-container.md) (Exécuter Enterprise Server dans un conteneur Docker sur Azure)


## <a name="tmaxsoft-openframe-on-azure"></a>TmaxSoft OpenFrame sur Azure

TmaxSoft OpenFrame est une solution de réhébergement de mainframe populaire utilisée pour les scénarios lift-and-shift. Un environnement OpenFrame dans Azure convient aux charges de travail de développement, de démonstration, de test et de production.

Pour commencer :

- [Get started with TmaxSoft OpenFrame](./tmaxsoft/get-started.md) (Bien démarrer avec TmaxSoft OpenFrame)
- [Télécharger le livre électronique](https://azure.microsoft.com/resources/install-tmaxsoft-openframe-on-azure/)

## <a name="ibm-zdt-120"></a>IBM zD&T 12.0

IBM zD&T (IBM Z Development and Test Environment) configure un environnement hors production sur Azure, que vous pouvez utiliser pour le développement, les tests et les démonstrations d’applications basées sur z/OS.

L’environnement d’émulation sur Azure peut héberger différents types d’instances de Z par le biais de distributions contrôlées de développeurs d’applications (ADCD, Application Developers Controlled Distribution). Vous pouvez exécuter zD&T Personal Edition, zD&T Parallel Sysplex et zD&T Enterprise Edition sur Azure et Azure Stack.

Pour commencer :

- [Configurez IBM zD&T 12.0 sur Azure](./ibm/install-ibm-z-environment.md).
- [Configurez ADCD sur zD&T](./ibm/demo.md).

## <a name="ibm-db2-purescale-on-azure"></a>IBM DB2 pureScale sur Azure

L’environnement IBM DB2 pureScale fournit un cluster de base de données pour Azure. Il n’est pas identique à l’environnement d’origine, mais il offre une disponibilité et une mise à l’échelle similaires à celles d’IBM DB2 pour z/OS exécuté dans une configuration Parallel Sysplex.

Pour commencer, consultez [IBM DB2 pureScale sur Azure](../../linux/ibm-db2-purescale-azure.md).

## <a name="considerations"></a>Considérations

Lorsque vous migrez des charges de travail mainframe vers l’infrastructure as a service (IaaS) Azure, vous pouvez choisir parmi plusieurs types de ressources informatiques à la demande et évolutives, y compris des machines virtuelles Azure. Azure propose toute une gamme de machines virtuelles [Linux](../../linux/overview.md) et [Windows](../../windows/overview.md).

### <a name="compute"></a>Calcul

Azure affiche une puissance de calcul élevée, comparable à la capacité d’un mainframe. Si vous envisagez de migrer une charge de travail mainframe vers Azure, comparez la métrique d’un mainframe s’élevant à un million d’instructions par seconde à celle de processeurs virtuels. 

Découvrez comment [déplacer le calcul mainframe vers Azure](./concepts/mainframe-compute-azure.md).

### <a name="high-availability-and-failover"></a>Haute disponibilité et basculement

Azure propose des contrats de niveau de service basés sur des engagements. La disponibilité à plusieurs 9 est proposée par défaut, et les contrats de niveau de service peuvent être optimisés avec la réplication locale ou géographique des services. La version complète du [contrat SLA Azure](https://azure.microsoft.com/support/legal/sla/virtual-machines/) explique la disponibilité garantie d’Azure dans son ensemble.

Avec Azure IaaS (une machine virtuelle, par exemple), des fonctions système spécifiques assurent la prise en charge du basculement, par exemple les instances de clustering de basculement et les groupes à haute disponibilité. Lorsque vous utilisez les ressources PaaS (platform-as-a-service) d’Azure, la plateforme gère automatiquement le basculement. Il peut s’agir, par exemple, [d’Azure SQL Database](../../../azure-sql/database/sql-database-paas-overview.md) et [d’Azure Cosmos DB](../../../cosmos-db/introduction.md).

### <a name="scalability"></a>Extensibilité

En règle générale, les ordinateurs mainframe procèdent à un scale-up, tandis que les environnements cloud procèdent à un scale-out. Pour répondre à vos besoins, Azure propose toute une gamme de dimensions sous [Linux](../../sizes.md) et [Windows](../../sizes.md). Le cloud s’adapte également aux spécifications exactes de l’utilisateur (scale up et scale down). La puissance de calcul, le stockage et les services sont [mis à l’échelle](/azure/architecture/best-practices/auto-scaling) à la demande selon un modèle de facturation à l’utilisation.

### <a name="storage"></a>Stockage

Dans le cloud, vous disposez d’un éventail d’options de stockage flexibles et évolutives. Vous payez uniquement en fonction de vos besoins. [Stockage Azure](../../../storage/common/storage-introduction.md) offre un magasin d’objets hautement scalable pour les objets de données, un service de système de fichiers pour le cloud, un magasin de messagerie fiable et un magasin NoSQL. Les machines virtuelles bénéficient d’un stockage sur disque sécurisé et persistant grâce à des disques managés et non managés.

Découvrez comment [déplacer le stockage du mainframe vers Azure](./concepts/mainframe-storage-azure.md).

### <a name="backup-and-recovery"></a>Sauvegarde et récupération

La gestion de votre propre site de récupération d’urgence peut représenter une option coûteuse. Azure propose des options de [sauvegarde](../../../backup/backup-overview.md), de [récupération](../../../site-recovery/site-recovery-overview.md) et de [redondance](../../../storage/common/storage-redundancy.md) économiques et faciles à implémenter au niveau local ou régional ou via la géoredondance.

## <a name="azure-government-for-mainframe-migrations"></a>Azure Government pour les migrations de mainframe

Plusieurs entités du secteur public aimeraient déplacer leurs applications de mainframe vers une plateforme plus flexible et plus moderne. Microsoft Azure Government est une instance physiquement séparée de la plateforme Microsoft Azure globale, préparée pour les systèmes gouvernementaux fédéraux, nationaux et locaux. Elle fournit des services de sécurité, de protection et de conformité de haut niveau, dédiés aux agences gouvernementales américaines et à leurs partenaires.

Azure Government s’est vu accorder une autorisation provisoire d’opérer (P-ATO) à un niveau d’impact élevé par le programme FedRAMP, pour les systèmes requérant ce type d’environnement.

Pour commencer, téléchargez le document [Microsoft Azure Government cloud for mainframe applications](https://azure.microsoft.com/resources/microsoft-azure-government-cloud-for-mainframe-applications/en-us/) (Cloud Microsoft Azure Government pour les applications de mainframe).

## <a name="next-steps"></a>Étapes suivantes

Demandez à nos [partenaires](partner-workloads.md) pour vous aider à migrer ou réhéberger vos applications de mainframe. 

Voir aussi :

- [White papers about mainframe topics](mainframe-white-papers.md) (Livres blancs relatifs aux mainframes)
- [Migration de mainframe](/azure/architecture/cloud-adoption/infrastructure/mainframe-migration/overview)
- [Dépannage](../../troubleshooting/index.yml)
- [Démystifier la migration d’un mainframe vers Azure](https://azure.microsoft.com/resources/demystifying-mainframe-to-azure-migration/)

<!-- INTERNAL LINKS -->
[microfocus-get-started]: /microfocus/get-started.md
[microfocus-setup]: /microfocus/set-up-micro-focus-azure.md
[microfocus-demo]: /microfocus/demo.md
[ibm-get-started]: /ibm/get-started.md
[ibm-install-z]: /ibm/install-ibm-z-environment.md
[ibm-demo]: /ibm/demo.md
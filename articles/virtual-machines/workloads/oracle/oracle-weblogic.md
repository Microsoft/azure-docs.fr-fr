---
title: Quelles sont les solutions permettant d’exécuter Oracle WebLogic Server sur Machines Virtuelles Azure
description: Découvrez comment exécuter Oracle WebLogic Server sur Machines virtuelles Microsoft Azure.
author: rezar
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.date: 09/23/2020
ms.author: rezar
ms.reviewer: cynthn
ms.openlocfilehash: 277e9049d2069190a68dd98c22acd60212c1b99d
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96486689"
---
# <a name="what-are-solutions-for-running-oracle-weblogic-server-on-azure-virtual-machines"></a>Quelles sont les solutions permettant d’exécuter Oracle WebLogic Server sur des machines virtuelles Azure ?

Cette page décrit les solutions permettant d’exécuter Oracle WebLogic Server (WLS) sur des machines virtuelles Azure. Ces solutions sont le fruit d’un développement et bénéficient d’un support conjoints d’Oracle et de Microsoft.

Il est également possible d’exécuter WLS sur Azure Kubernetes Service. Pour ce faire, les solutions sont décrites dans [cet article de Microsoft](./weblogic-aks.md).

WLS est un des principaux serveurs d’applications Java qui exécutent certaines des applications d’entreprise Java les plus critiques dans le monde entier. WLS forme la base de l’intergiciel (middleware) pour la suite de logiciels Oracle. Oracle et Microsoft s’engagent à donner aux clients WLS les choix et la flexibilité nécessaires pour exécuter des charges de travail sur Azure en tant que plateforme cloud de premier plan.

Les solutions WLS d’Azure visent à faciliter autant que possible le lift-and-shift de vos applications Java EE vers des machines virtuelles Azure en automatisant la plupart des opérations standard. Les solutions provisionnent automatiquement des ressources de réseau virtuel, de stockage, Java et Linux. Avec un travail minimal, WebLogic Server est installé. Les solutions peuvent configurer la sécurité avec un groupe de sécurité réseau, l’équilibrage de charge avec Azure Application Gateway et l’authentification avec Azure Active Directory. Vous pouvez également vous connecter automatiquement à votre base de données existante, y compris Azure PostgreSQL, Azure SQL et Oracle DB sur le cloud Oracle ou Azure. Il est prévu que les solutions incluent la possibilité d’activer la journalisation distribuée et la mise en cache distribuée via Oracle Coherence.  

:::image type="content" source="media/oracle-weblogic/wls-on-azure.gif" alt-text="Vous pouvez utiliser le portail Azure pour déployer WebLogic Server sur Azure":::

Quatre offres sont disponibles pour répondre à différents scénarios : [nœud unique sans serveur d’administration](https://portal.azure.com/#create/oracle.20191001-arm-oraclelinux-wls20191001-arm-oraclelinux-wls), [nœud unique avec un serveur d’administration](https://portal.azure.com/#create/oracle.20191009-arm-oraclelinux-wls-admin20191009-arm-oraclelinux-wls-admin), [cluster](https://portal.azure.com/#create/oracle.20191007-arm-oraclelinux-wls-cluster20191007-arm-oraclelinux-wls-cluster) et [cluster dynamique](https://portal.azure.com/#create/oracle.20191021-arm-oraclelinux-wls-dynamic-cluster20191021-arm-oraclelinux-wls-dynamic-cluster). Les offres sont disponibles gratuitement. Ces offres sont décrites ci-dessous avec les liens nécessaires.

_Ces offres sont des BYOL (apportez votre propre licence)_ . Elles supposent que vous avez déjà obtenu les licences appropriées auprès d’Oracle et que vous disposez d’une licence adéquate pour exécuter des offres dans Azure.

Les offres prennent en charge une variété de versions des systèmes d’exploitation, de Java et de WLS via des images de base (comme WebLogic Server 14 et JDK 11 sur Oracle Linux 7.6). Ces images de base sont également disponibles sur Azure. Les images de base sont adaptées aux clients qui ont besoin de déploiements Azure complexes et personnalisés. L’ensemble actuel d’images de base est disponible [ici](https://azuremarketplace.microsoft.com/en-us/marketplace/apps?search=WebLogic%20Server%20Base%20Image&page=1).

_Si vous souhaitez travailler étroitement sur vos scénarios de migration avec l’équipe d’ingénierie développant ces offres, sélectionnez le bouton [ME CONTACTER](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/oracle.oraclelinux-wls-cluster?tab=Overview)_ sur la [page de vue d’ensemble des offres de la Place de marché](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/oracle.oraclelinux-wls-cluster?tab=Overview). Des responsables, des architectes et des ingénieurs du programme reviendront rapidement vers vous pour démarrer une étroite collaboration. L’opportunité de collaborer sur un scénario de migration est gratuite pendant que les offres sont en cours de développement initial.

## <a name="oracle-weblogic-server-single-node"></a>Oracle WebLogic Server avec nœud unique

[Cette offre](https://portal.azure.com/#create/oracle.20191001-arm-oraclelinux-wls20191001-arm-oraclelinux-wls) approvisionne une machine virtuelle et installe WebLogic Server sur celle-ci. Elle ne crée pas de domaine ni ne démarre le serveur d’administration. L’offre avec un nœud unique convient pour les scénarios ayant une configuration de domaine hautement personnalisée.

## <a name="oracle-weblogic-server-with-admin-server"></a>Oracle WebLogic Server avec serveur d’administration

[Cette offre](https://portal.azure.com/#create/oracle.20191009-arm-oraclelinux-wls-admin20191009-arm-oraclelinux-wls-admin) approvisionne une machine virtuelle et installe WebLogic Server sur celle-ci. Elle crée un domaine et démarre le serveur d’administration. Vous pouvez gérer le domaine et vous familiariser immédiatement avec les déploiements d’applications.

## <a name="oracle-weblogic-server-cluster"></a>Cluster Oracle WebLogic Server

[Cette offre](https://portal.azure.com/#create/oracle.20191007-arm-oraclelinux-wls-cluster20191007-arm-oraclelinux-wls-cluster) crée un cluster hautement disponible de machines virtuelles WLS. Le serveur d’administration et tous les serveurs managés sont démarrés par défaut. Vous pouvez gérer le cluster et commencer immédiatement avec des applications à haute disponibilité.

## <a name="oracle-weblogic-server-dynamic-cluster"></a>Cluster dynamique Oracle WebLogic Server

[Cette offre](https://portal.azure.com/#create/oracle.20191021-arm-oraclelinux-wls-dynamic-cluster20191021-arm-oraclelinux-wls-dynamic-cluster) crée un cluster dynamique évolutif et hautement disponible de machines virtuelles WLS. Le serveur d’administration et tous les serveurs managés sont démarrés par défaut.

Les solutions permettront un large éventail d’architectures de déploiement prêtes pour la production et d’une facilité relative. Vous pouvez faire face à la majorité des cas de migration de la façon la plus productive possible en vous concentrant sur le développement d’applications métier.

:::image type="content" source="media/oracle-weblogic/weblogic-architecture-vms.png" alt-text="Les déploiements complexes de WebLogic Server sont activés sur Azure":::

Au-delà de ce qui est provisionné automatiquement par les solutions, les clients ont une flexibilité totale pour personnaliser davantage leurs déploiements. Il est probable qu’en plus de déployer des applications, les clients vont intégrer d’autres ressources Azure à leurs déploiements. Les clients sont encouragés à fournir un feedback visant à l’amélioration des solutions.

## <a name="next-steps"></a>Étapes suivantes

Explorez les offres sur Azure.

> [!div class="nextstepaction"]
> [Oracle WebLogic Server avec nœud unique](https://portal.azure.com/#create/oracle.20191001-arm-oraclelinux-wls20191001-arm-oraclelinux-wls)

> [!div class="nextstepaction"]
> [Oracle WebLogic Server avec serveur d’administration](https://portal.azure.com/#create/oracle.20191009-arm-oraclelinux-wls-admin20191009-arm-oraclelinux-wls-admin)

> [!div class="nextstepaction"]
> [Cluster Oracle WebLogic Server](https://portal.azure.com/#create/oracle.20191007-arm-oraclelinux-wls-cluster20191007-arm-oraclelinux-wls-cluster)

> [!div class="nextstepaction"]
> [Cluster dynamique Oracle WebLogic Server](https://portal.azure.com/#create/oracle.20191021-arm-oraclelinux-wls-dynamic-cluster20191021-arm-oraclelinux-wls-dynamic-cluster)
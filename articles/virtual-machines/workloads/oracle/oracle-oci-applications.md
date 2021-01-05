---
title: Architectures pour déployer des applications Oracle sur des machines virtuelles Azure | Microsoft Docs
description: Architectures d'applications pour déployer des applications Oracle, notamment E-Business Suite, JD Edwards EnterpriseOne et PeopleSoft sur des machines virtuelles Microsoft Azure avec des bases de données dans Azure ou dans Oracle Cloud Infrastructure (OCI).
author: dbakevlar
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.date: 07/18/2019
ms.author: kegorman
ms.reviewer: cynthn
ms.openlocfilehash: aa481090e3483e58f6a88304e3e9d8c1a16df3c7
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94965916"
---
# <a name="architectures-to-deploy-oracle-applications-on-azure"></a>Architectures pour déployer des applications Oracle sur Azure

Microsoft et Oracle ont travaillé ensemble pour permettre à leurs clients de déployer des applications Oracle comme Oracle E-Business Suite, JD Edwards EnterpriseOne et PeopleSoft dans le cloud. Avec l'introduction de la préversion de [l'interconnectivité des réseaux privés](configure-azure-oci-networking.md) entre Microsoft Azure et Oracle Cloud Infrastructure (OCI), les applications Oracle peuvent maintenant être déployées sur Azure avec leurs bases de données back-end dans Azure ou OCI. Les applications Oracle peuvent également être intégrées à Azure Active Directory, ce qui vous permet de configurer une authentification unique afin que les utilisateurs se connectent à l'application Oracle à l’aide de leurs identifiants Azure Active Directory (Azure AD).

OCI offre plusieurs options de base de données Oracle pour les applications Oracle, notamment DBaaS, Exadata Cloud Service, Oracle RAC et Infrastructure-as-a-Service (IaaS). Actuellement, Autonomous Database n'est pas un back-end supporté pour les applications Oracle.

Il existe [plusieurs options](oracle-overview.md) pour déployer des applications Oracle dans Azure, y compris d'une manière hautement disponible et sécurisée. Azure offre également des [images de machine virtuelle Oracle Database](oracle-vm-solutions.md) que vous pouvez déployer si vous choisissez d'exécuter vos applications Oracle entièrement sur Azure.

Les sections suivantes décrivent les recommandations d'architecture Microsoft et Oracle pour déployer Oracle E-Business Suite, JD Edwards EnterpriseOne et PeopleSoft dans une configuration multicloud ou entièrement dans Azure. Microsoft et Oracle ont testé ces applications et confirmé que les performances sont conformes aux normes établies par Oracle pour ces applications.

## <a name="architecture-considerations"></a>Considérations relatives à l’architecture

Les applications Oracle sont composées de plusieurs services, qui peuvent être hébergés sur la même machine virtuelle ou sur plusieurs machines virtuelles dans Azure et éventuellement dans OCI. 

Les instances d'application peuvent être configurées avec des points de terminaison privés ou publics. Microsoft et Oracle recommandent de configurer une *machine virtuelle bastion* avec une adresse IP publique dans un sous-réseau séparé pour la gestion de l'application. N'attribuez ensuite que des adresses IP privées aux autres machines, y compris au niveau de la base de données. 

Lors de la configuration d'une application dans une architecture multicloud, une planification est nécessaire pour s'assurer que l'espace d'adressage IP dans le réseau virtuel Azure ne chevauche pas l'espace d'adresse IP privé dans le réseau cloud privé OCI.

Pour plus de sécurité, configurez des groupes de sécurité réseau au niveau du sous-réseau pour vous assurer que seul le trafic sur des ports et adresses IP spécifiques est autorisé. Par exemple, les machines du niveau intermédiaire ne devraient recevoir que du trafic provenant du réseau virtuel. Aucun trafic externe ne doit atteindre directement les machines de niveau intermédiaire.

Pour garantir une haute disponibilité, vous pouvez configurer des instances redondantes des différents serveurs dans le même groupe à haute disponibilité ou dans des zones de disponibilité différentes. Les zones de disponibilité vous permettent d'obtenir un contrat SLA à 99,99 % de durée de fonctionnement, tandis que les groupes à haute disponibilité offrent un contrat SLA à 99,95 % dans une région. Les exemples d'architectures présentés dans cet article sont déployés sur deux zones de disponibilité.

Lors du déploiement d'une application à l’aide d’une interconnexion multicloud, vous pouvez continuer à utiliser un circuit ExpressRoute existant pour connecter votre environnement Azure à votre réseau local. Cependant, vous avez besoin d'un circuit ExpressRoute pour l'interconnexion à OCI autre que celui qui se connecte à votre réseau local.

## <a name="e-business-suite"></a>E-Business Suite

Oracle E-Business Suite (EBS) est une suite d'applications comprenant la gestion de la chaîne d’approvisionnement (SCM) et la gestion de la relation client (CRM). Pour tirer parti du portefeuille de bases de données gérées d'OCI, EBS peut être déployé en utilisant l'interconnexion multicloud entre Microsoft Azure et OCI. Dans cette configuration, les niveaux de présentation et d'application s'exécutent dans Azure et le niveau de base de données dans OCI, comme le montre le diagramme d'architecture suivant (Figure 1).

![Architecture multicloud E-Business Suite](media/oracle-oci-applications/ebs-arch-cross-cloud.png)

*Figure 1 : Architecture multicloud E-Business Suite* 

Dans cette architecture, le réseau virtuel Azure est connecté à un réseau cloud virtuel dans OCI en utilisant l'interconnexion multicloud. Le niveau d'application est configuré dans Azure, tandis que la base de données est configurée dans OCI. Il est recommandé de déployer chaque composant sur son propre sous-réseau avec des groupes de sécurité réseau pour autoriser le trafic uniquement à partir de sous-réseaux spécifiques sur des ports spécifiques.

L'architecture peut également être adaptée à un déploiement entièrement sur Azure avec des bases de données Oracle hautement disponibles configurées avec Oracle Data Guard dans deux zones de disponibilité d’une même région. Le diagramme suivant (Figure 2) est un exemple de ce modèle architectural :

![Architecture Azure E-Business Suite](media/oracle-oci-applications/ebs-arch-azure.png)

*Figure 2 : Architecture Azure E-Business Suite*

Les sections suivantes décrivent les différents composants à un niveau élevé.

[!INCLUDE [virtual-machines-oracle-applications-bastion](../../../../includes/virtual-machines-oracle-applications-bastion.md)]

### <a name="application-middle-tier"></a>Niveau d’application (intermédiaire)

Le niveau d'application est isolé dans son propre sous-réseau. Il existe plusieurs machines virtuelles configurées pour une tolérance aux pannes et une gestion facile des correctifs. Ces machines virtuelles peuvent être sauvegardées à l’aide d’un stockage partagé, fourni par Azure NetApp Files et des disques Ultra SSD. Cette configuration facilite le déploiement des correctifs sans temps d'arrêt. Les machines du niveau d'application doivent être précédées d'un équilibreur de charge public afin de traiter les requêtes envoyées au niveau d'application EBS même si une machine du niveau est hors ligne en raison d'une défaillance.

### <a name="load-balancer"></a>Équilibrage de charge

Un équilibreur de charge Azure vous permet de répartir le trafic sur plusieurs instances de votre charge de travail pour assurer une haute disponibilité. Dans ce cas, un équilibreur de charge public est mis en place, car les utilisateurs sont autorisés à accéder à l'application EBS sur le Web. L'équilibreur de charge répartit la charge sur les deux machines du niveau intermédiaire. Pour plus de sécurité, autorisez uniquement le trafic des utilisateurs accédant au système depuis votre réseau d'entreprise en utilisant un VPN site à site ou ExpressRoute et des groupes de sécurité réseau.

### <a name="database-tier"></a>Couche base de données

Ce niveau héberge la base de données Oracle et se divise avec son propre sous-réseau. Il est recommandé d'ajouter des groupes de sécurité réseau qui n'autorisent que le trafic du niveau d’application vers le niveau de base de données sur le port 1521 de la base de données Oracle spécifique.

Microsoft et Oracle recommandent une configuration à haute disponibilité. La haute disponibilité dans Azure peut être obtenue en installant deux bases de données Oracle dans deux zones de disponibilité avec Oracle Data Guard, ou en utilisant Oracle Database Exadata Cloud Service dans OCI. Lorsque vous utilisez Oracle Database Exadata Cloud Service, votre base de données est déployée en deux sous-réseaux. Vous pouvez également configurer Oracle Database avec des machines virtuelles dans OCI et dans deux domaines de disponibilité avec Oracle Data Guard.


### <a name="identity-tier"></a>Niveau d'identité

Le niveau d'identité contient la machine virtuelle EBS Asserter. EBS Asserter vous permet de synchroniser les identités depuis Oracle Identity Cloud Service (IDCS) et Azure AD. EBS Asserter est requis car EBS ne prend pas en charge les protocoles d’authentification unique comme SAML 2.0 ou OpenID Connect. EBS Asserter consomme le jeton de connexion OpenID (généré par IDCS), le valide, puis crée une session pour l'utilisateur dans EBS. 

Bien que cette architecture montre l'intégration IDCS, l'accès unifié et l'authentification unique Azure AD peuvent également être activés avec Oracle Access Manager en utilisant Oracle Internet Directory ou Oracle Unified Directory. Pour plus d'informations, consultez les livres blancs sur le [déploiement d’Oracle EBS avec intégration IDCS](https://cloud.oracle.com/iaas/whitepapers/deploy_ebusiness_suite_across_oci_azure_sso_idcs.pdf) ou le [déploiement d’Oracle EBS avec intégration OAM](https://cloud.oracle.com/iaas/whitepapers/deploy_ebusiness_suite_across_oci_azure_sso_oam.pdf).

Pour garantir une haute disponibilité, il est recommandé de déployer des serveurs redondants EBS Asserter sur plusieurs zones de disponibilité, en les précédant d’un équilibreur de charge.

Une fois votre infrastructure configurée, vous pouvez installer E-Business Suite en suivant le guide d'installation fourni par Oracle.

## <a name="jd-edwards-enterpriseone"></a>JD Edwards EnterpriseOne

JD Edwards EnterpriseOne d'Oracle est une suite d'applications intégrées de logiciels complets de planification des ressources d'entreprise. Il s'agit d'une application à plusieurs niveaux qui peut être configurée avec un back-end de base de données Oracle ou SQL Server. Cette section traite en détail du déploiement de JD Edwards EnterpriseOne avec un back-end de base de données Oracle dans OCI ou Azure.

Dans l'architecture recommandée suivante (Figure 3), l'administration, la présentation et les niveaux intermédiaires sont déployés sur le réseau virtuel dans Azure. La base de données est déployée dans un réseau cloud virtuel au sein d'OCI.

Comme avec E-Business Suite, vous pouvez configurer un niveau bastion facultatif à des fins administratives sécurisées. Utilisez l'hôte de machine virtuelle bastion comme serveur de rebond pour accéder aux instances de l'application et de la base de données.

![Architecture multicloud JD Edwards EnterpriseOne](media/oracle-oci-applications/jdedwards-arch-cross-cloud.png)

*Figure 3 : Architecture multicloud JD Edwards EnterpriseOne*

Dans cette architecture, le réseau virtuel Azure est connecté au réseau cloud virtuel dans OCI en utilisant l'interconnexion multicloud. Le niveau d'application est configuré dans Azure, tandis que la base de données est configurée dans OCI. Il est recommandé de déployer chaque composant sur son propre sous-réseau avec des groupes de sécurité réseau pour autoriser le trafic uniquement à partir de sous-réseaux spécifiques sur des ports spécifiques.

L'architecture peut également être adaptée à un déploiement entièrement sur Azure avec des bases de données Oracle hautement disponibles configurées avec Oracle Data Guard dans deux zones de disponibilité d’une même région. Le diagramme suivant (Figure 4) est un exemple de ce modèle architectural :

![Architecture Azure JD Edwards EnterpriseOne](media/oracle-oci-applications/jdedwards-arch-azure.png)

*Figure 4 : Architecture Azure JD Edwards EnterpriseOne*

Les sections suivantes décrivent les différents composants à un niveau élevé.

[!INCLUDE [virtual-machines-oracle-applications-bastion](../../../../includes/virtual-machines-oracle-applications-bastion.md)]

### <a name="administrative-tier"></a>Niveau administratif

Comme son nom l'indique, ce niveau est utilisé pour les tâches administratives. Vous pouvez créer un sous-réseau distinct pour le niveau administratif. Les services et serveurs de ce niveau sont principalement utilisés pour l'installation et l'administration de l'application. Par conséquent, des instances uniques de ces serveurs sont suffisantes. Les instances redondantes ne sont pas nécessaires pour la haute disponibilité de votre application.

Voici les composants de ce niveau :
    
 - **Serveur d'approvisionnement** : ce serveur est utilisé pour le déploiement de bout en bout des différents composants de l'application. Il communique avec les instances des autres niveaux, y compris les instances du niveau base de données, via le port 22. Il héberge la console Gestionnaire de serveur pour JD Edwards EnterpriseOne.
 - **Serveur de déploiement** : ce serveur est principalement requis pour l'installation de JD Edwards EnterpriseOne. Pendant le processus d'installation, ce serveur sert de référentiel central pour les fichiers et paquets d'installation requis. Le logiciel est distribué ou déployé sur d'autres serveurs et clients depuis ce serveur.
 - **Client de développement** : ce serveur contient des composants qui s'exécutent dans un navigateur web ainsi que des applications natives.

### <a name="presentation-tier"></a>Niveau de présentation

Ce niveau contient divers composants comme les services d’interface d’application (Application Interface Services ou AIS), l’infrastructure de développement d’application (Development Framework ou ADF) et les serveurs d’application Java (Java Application Servers ou JAS). Les serveurs de ce niveau communiquent avec les serveurs du niveau intermédiaire. Ils sont gérés par un équilibreur de charge qui achemine le trafic vers le serveur nécessaire selon le numéro de port et l'URL sur lesquels le trafic est reçu. Il est recommandé de déployer plusieurs instances de chaque type de serveur pour garantir une haute disponibilité.

Voici les composants de ce niveau :
    
- **Application Interface Services (AIS)**  : le serveur AIS fournit l'interface de communication entre les applications mobiles JD Edwards EnterpriseOne et JD Edwards EnterpriseOne.
- **Java Application Server (JAS)**  : le serveur JAS reçoit les requêtes de l'équilibreur de charge et les transmet au niveau intermédiaire pour exécuter des tâches complexes. Le JAS a la capacité d'exécuter une logique métier simple.
- **BI Publisher Server (BIP)**  : ce serveur présente des rapports basés sur les données collectées par l'application JD Edwards EnterpriseOne. Vous pouvez concevoir et contrôler la façon dont le rapport présente les données en fonction de différents modèles.
- **Business Services Server (BSS)**  : le serveur BSS permet l'échange d'informations et l'interopérabilité avec les autres applications Oracle.
- **Real-Time Events Server (RTE)**  : le serveur RTE vous permet de configurer des notifications envoyées vers les systèmes externes concernant les transactions effectuées dans le système JDE EnterpriseOne. Il utilise un modèle d'abonné et permet aux systèmes tiers de s'abonner à des événements. Pour charger les requêtes d'équilibrage sur les deux serveurs RTE, vérifiez que les serveurs figurent dans un cluster.
- **Application Development Framework (ADF) Server** : le serveur ADF est utilisé pour exécuter les applications JD Edwards EnterpriseOne développées avec Oracle ADF. Il est déployé sur un serveur Oracle WebLogic avec runtime ADF.

### <a name="middle-tier"></a>Niveau intermédiaire

Le niveau intermédiaire contient le serveur logique et le serveur par lots. Dans ce cas, les deux serveurs sont installés sur la même machine virtuelle. Mais pour les scénarios de production, il est recommandé de déployer le serveur logique et le serveur par lots sur des serveurs distincts. Plusieurs serveurs sont déployés au niveau intermédiaire sur deux zones de disponibilité pour une plus grande disponibilité. Un équilibreur de charge Azure devrait être créé, et ces serveurs devraient être placés dans son pool principal pour s'assurer que les deux serveurs sont actifs et traitent les requêtes.

Les serveurs du niveau intermédiaire reçoivent uniquement les requêtes des serveurs du niveau de présentation et de l'équilibreur de charge public. Des règles de groupe de sécurité réseau doivent être configurées de manière à refuser le trafic provenant de toute adresse autre que le sous-réseau du niveau de présentation et de l'équilibreur de charge. Une règle NSG peut également être configurée pour autoriser le trafic sur le port 22 depuis l'hôte bastion à des fins de gestion. Vous pouvez utiliser l'équilibreur de charge public pour charger les requêtes d'équilibrage de charge entre les machines virtuelles du niveau intermédiaire.

Voici les deux composants du niveau intermédiaire :

- **Serveur logique** : contient la logique métier ou les fonctions métier.
- **Serveur par lots** : utilisé pour le traitement par lots

[!INCLUDE [virtual-machines-oracle-applications-database](../../../../includes/virtual-machines-oracle-applications-database.md)]

[!INCLUDE [virtual-machines-oracle-applications-identity](../../../../includes/virtual-machines-oracle-applications-identity.md)]

## <a name="peoplesoft"></a>PeopleSoft

La suite d'applications PeopleSoft d'Oracle contient un logiciel de gestion des ressources humaines et financières. La suite d'applications comporte plusieurs niveaux et les applications incluent des systèmes de gestion des ressources humaines (SGRH), de gestion de la relation client (GRC), de gestion financière et de la chaîne d'approvisionnement (GCAA) et de gestion des performances de l'entreprise (GPE).

Il est recommandé de déployer chaque niveau de la suite logicielle dans son propre sous-réseau. Une base de données Oracle ou Microsoft SQL Server est requise comme base de données principale pour l'application. Cette section traite en détail du déploiement de PeopleSoft avec une base de données principale Oracle.

Voici une architecture canonique de déploiement de la suite d'applications PeopleSoft dans une architecture multicloud (Figure 5).

![Architecture PeopleSoft multicloud](media/oracle-oci-applications/peoplesoft-arch-cross-cloud.png)

*Figure 5 : Architecture PeopleSoft multicloud*

Dans cet exemple d’architecture, le réseau virtuel Azure est connecté au réseau cloud virtuel dans OCI en utilisant l'interconnexion multicloud. Le niveau d'application est configuré dans Azure, tandis que la base de données est configurée dans OCI. Il est recommandé de déployer chaque composant sur son propre sous-réseau avec des groupes de sécurité réseau pour autoriser le trafic uniquement à partir de sous-réseaux spécifiques sur des ports spécifiques.

L'architecture peut également être adaptée à un déploiement entièrement sur Azure avec des bases de données Oracle hautement disponibles configurées avec Oracle Data Guard dans deux zones de disponibilité d’une même région. Le diagramme suivant (Figure 6) est un exemple de ce modèle architectural :

![Architecture Azure PeopleSoft](media/oracle-oci-applications/peoplesoft-arch-azure.png)

*Figure 6 : Architecture Azure PeopleSoft*

Les sections suivantes décrivent les différents composants à un niveau élevé.

[!INCLUDE [virtual-machines-oracle-applications-bastion](../../../../includes/virtual-machines-oracle-applications-bastion.md)]

### <a name="application-tier"></a>Couche Application

Le niveau d'application contient des instances de serveurs d'application PeopleSoft, de serveurs web PeopleSoft, une recherche élastique et PeopleSoft Process Scheduler. Un équilibreur de charge Azure est configuré pour accepter les requêtes d’utilisateurs acheminées vers le serveur approprié dans le niveau d'application.

Pour garantir une haute disponibilité, vous pouvez configurer des instances redondantes de chaque serveur du niveau d’application sur différentes zones de disponibilité. L'équilibreur de charge Azure peut être configuré avec plusieurs pools principaux afin de diriger chaque requête vers le serveur adéquat.

### <a name="peopletools-client"></a>PeopleTools Client

PeopleTools Client est utilisé pour effectuer des tâches administratives telles que le développement, la migration et la mise à niveau. PeopleTools Client n’étant pas nécessaire pour obtenir une haute disponibilité de votre application, les serveurs redondants PeopleTools Client sont inutiles.

[!INCLUDE [virtual-machines-oracle-applications-database](../../../../includes/virtual-machines-oracle-applications-database.md)]

[!INCLUDE [virtual-machines-oracle-applications-identity](../../../../includes/virtual-machines-oracle-applications-identity.md)]

## <a name="next-steps"></a>Étapes suivantes

Utilisez des [scripts Terraform](https://github.com/microsoft/azure-oracle) pour configurer des applications Oracle dans Azure et établir une connectivité multicloud avec OCI.

Pour plus d’informations et de livres blancs sur OCI, consultez la documentation [Oracle Cloud](https://docs.cloud.oracle.com/iaas/Content/home.htm).

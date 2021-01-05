---
title: À propos de la reprise d’activité pour les applications locales avec Azure Site Recovery
description: Décrit les charges de travail qui peuvent être protégées à l’aide de la récupération d’urgence avec le service Azure Site Recovery.
ms.topic: conceptual
ms.date: 03/18/2020
ms.openlocfilehash: 2b901425a0020c0ccc7b834ee36d965910028018
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80062834"
---
# <a name="about-disaster-recovery-for-on-premises-apps"></a>À propos de la reprise d’activité pour les applications locales

Cet article décrit les charges de travail et les applications locales que vous pouvez protéger pour la récupération d’urgence avec le service [Azure Site Recovery](site-recovery-overview.md).

## <a name="overview"></a>Vue d’ensemble

Les organisations ont besoin d’une stratégie de continuité d’activité et reprise d’activité (BCDR) pour que les charges de travail et les données demeurent en sécurité et disponibles pendant les temps d’arrêt prévus et imprévus. Et pour rétablir au plus vite des conditions de travail normales.

Site Recovery est un service Azure qui participe à votre stratégie de continuité des activités et de récupération d’urgence. À l’aide de Site Recovery, vous pouvez déployer une réplication compatible avec les applications vers le cloud ou un site secondaire. Vous pouvez utiliser Site Recovery pour gérer la réplication, effectuer des tests de reprise d’activité et exécuter des basculements et des restaurations automatiques. Vos applications peuvent s’exécuter sur des ordinateurs Windows ou Linux, sur des serveurs physiques, sur VMware ou sur Hyper-V.

Site Recovery s’intègre à des applications Microsoft, telles que SharePoint, Exchange, Dynamics, SQL Server et Active Directory. Microsoft travaille en étroite collaboration avec les principaux fournisseurs, notamment Oracle, SAP et Red Hat. Vous pouvez personnaliser les solutions de réplication application par application.

## <a name="why-use-site-recovery-for-application-replication"></a>Pourquoi utiliser Site Recovery pour la réplication des applications ?

Site Recovery contribue à la protection des applications et à la récupération comme suit :

- Service indépendant de l’application qui assure la réplication de n’importe quelle charge de travail exécutée sur une machine prise en charge.
- Réplication quasiment synchrone avec des objectifs de point de récupération (RPO) pouvant descendre jusqu’à 30 secondes pour répondre aux besoins des applications métier les plus critiques.
- Captures instantanées cohérentes de l’application pour les applications uniques ou multiniveau.
- Intégration avec SQL Server AlwaysOn et partenariat avec d’autres technologies de réplication au niveau de l’application. Par exemple, réplication Active Directory, SQL AlwaysOn et groupes de disponibilité de base de données Exchange (DAG).
- Plans de récupération flexibles qui vous permettent de récupérer une pile d’application entière en un seul clic et d’inclure les scripts externes et des actions manuelles.
- Gestion de réseau avancée dans Site Recovery et Azure pour simplifier la configuration réseau requise pour l’application. Gestion réseau telle que la capacité à réserver des adresses IP, la configuration de l’équilibrage de charge et l’intégration avec Azure Traffic Manager pour les commutations réseau à objectifs de délai de récupération (RTO) faibles.
- Une bibliothèque d’automatisation avancée qui fournit des scripts spécifiques d’application prêts pour la production, qui peuvent être téléchargés et intégrés avec des plans de récupération.

## <a name="workload-summary"></a>Synthèse relative aux charges de travail

Site Recovery permet de répliquer n’importe quelle application exécutée sur une machine prise en charge. Nous collaborons avec les équipes produit afin d’effectuer des tests supplémentaires pour les applications spécifiées dans le tableau suivant.

| **Charge de travail** |**Réplication de machines virtuelles Azure dans Azure** |**Réplication de machines virtuelles Hyper-V vers un site secondaire** | **Réplication de machines virtuelles Hyper-V dans Azure** | **Réplication de machines virtuelles VMware vers un site secondaire** | **Réplication de machines virtuelles VMware dans Azure** |
| --- | --- | --- | --- | --- |---|
| Active Directory, DNS |Oui |Oui |Oui |Oui |Oui|
| Applications web (IIS, SQL) |Oui |Oui |Oui |Oui |Oui|
| System Center Operations Manager |Oui |Oui |Oui |Oui |Oui|
| SharePoint |Oui |Oui |Oui |Oui |Oui|
| SAP<br/><br/>Réplication d’un site SAP vers Microsoft Azure (aucun cluster) |Oui (testé par Microsoft) |Oui (testé par Microsoft) |Oui (testé par Microsoft) |Oui (testé par Microsoft) |Oui (testé par Microsoft)|
| Microsoft Exchange (aucun DAG) |Oui |Oui |Oui |Oui |Oui|
| Bureau à distance/VDI |Oui |Oui |Oui |Oui |Oui|
| Linux (système d’exploitation et applications) |Oui (testé par Microsoft) |Oui (testé par Microsoft) |Oui (testé par Microsoft) |Oui (testé par Microsoft) |Oui (testé par Microsoft)|
| Dynamics AX |Oui |Oui |Oui |Oui |Oui|
| Serveur de fichiers Windows |Oui |Oui |Oui |Oui |Oui|
| Citrix XenApp et XenDesktop |Oui|N/A |Oui |N/A |Oui |

## <a name="replicate-active-directory-and-dns"></a>Répliquer Active Directory et DNS

Une infrastructure DNS et Active Directory sont essentiels pour la plupart des applications d’entreprise. Lors d’une reprise d’activité, vous devez protéger et récupérer ces composants d’infrastructure avant de récupérer les charges de travail et les applications.

À l’aide de Site Recovery, vous pouvez créer un plan de récupération d’urgence automatisé complet pour Active Directory et DNS. Par exemple, pour basculer SharePoint et SAP d’un site principal vers un site secondaire, vous pouvez configurer un plan de récupération qui bascule d’abord Active Directory. Utilisez ensuite un plan de récupération supplémentaire, spécifique à l’application, pour basculer les autres applications qui reposent sur Active Directory.

[En savoir plus](site-recovery-active-directory.md) sur la reprise d’activité pour Active Directory et DNS.

## <a name="protect-sql-server"></a>Protéger SQL Server

SQL Server fournit une base pour les services de données pour de nombreuses applications métiers dans un centre de données local. Site Recovery peut être utilisé avec les technologies SQL Server HA/DR pour protéger les applications d’entreprise multiniveau faisant appel à SQL Server.

Site Recovery présente les caractéristiques suivantes :

- Solution de récupération d’urgence simple et économique pour SQL Server. Réplication de plusieurs versions et éditions de serveurs et clusters autonomes SQL Server vers Azure ou un site secondaire.
- Intégration avec les groupes de disponibilité SQL AlwaysOn pour gérer le basculement et la restauration automatique avec les plans de récupération Azure Site Recovery.
- Plans de récupération de bout en bout pour tous les niveaux d’une application, y compris les bases de données SQL Server.
- Mise à l’échelle de SQL Server pour les pics de charge à l’aide de Site Recovery par _éclatement_ en tailles de machines virtuelles IaaS plus volumineuses dans Azure.
- Test facile de récupération d’urgence de SQL Server. Vous pouvez effectuer des basculements tests pour l’analyse et les vérifications de conformité des données, sans impact sur votre environnement de production.

[En savoir plus](site-recovery-sql.md) sur la reprise d’activité pour SQL Server.

## <a name="protect-sharepoint"></a>Protéger SharePoint

Azure Site Recovery vous aide à protéger vos déploiements SharePoint comme suit :

- Élimine le besoin et les coûts de l’infrastructure associée à une batterie de secours pour la récupération d’urgence. Utilisez Site Recovery pour répliquer une batterie entière (niveaux web, application et base de données) sur Azure ou sur un site secondaire.
- Simplifie la gestion et le déploiement de l’application. Les mises à jour déployées sur le site principal sont automatiquement répliquées. Les mises à jour sont disponibles après le basculement et la récupération d’une batterie de serveurs dans un site secondaire. Cela réduit la complexité de gestion et les coûts associés au maintien d’une batterie de secours à jour.
- Simplifie le développement et le test d’applications SharePoint en créant un environnement de réplica de copie à la demande de type production pour le test et le débogage.
- Simplifie la transition vers le cloud en utilisant Site Recovery pour migrer les déploiements SharePoint vers Azure.

[En savoir plus](site-recovery-sharepoint.md) sur la reprise d’activité pour SharePoint.

## <a name="protect-dynamics-ax"></a>Protéger Dynamics AX

Azure Site Recovery vous permet de protéger votre solution Dynamics AX ERP à travers :

- La gestion de la réplication de l’ensemble de votre environnement Dynamics AX (niveaux web et AOS, niveaux de base de données, SharePoint) sur Azure ou sur un site secondaire.
- La simplification de la migration des déploiements de Dynamics AX dans le cloud (Azure).
- La simplification du développement et du test d’applications Dynamics AX en créant une copie de type production à la demande pour le test et le débogage.

[En savoir plus](site-recovery-dynamicsax.md) sur la reprise d’activité pour Dynamic AX.

## <a name="protect-remote-desktop-services"></a>Protéger les services Bureau à distance

Les services Bureau à distance (RDS) activent l’infrastructure de bureau virtuel (VDI), les bureaux basés sur session et les applications, qui permettent aux utilisateurs de travailler n’importe où.

Avec Azure Site Recovery, vous pouvez répliquer les services suivants :

- Répliquez des bureaux virtuels mis en pool managés ou non vers un site secondaire.
- Répliquez des sessions et des applications distantes vers un site secondaire ou Azure.

Le tableau suivant affiche les options de réplication :

| **RDS** |**Réplication de machines virtuelles Azure dans Azure** | **Réplication de machines virtuelles Hyper-V vers un site secondaire** | **Réplication de machines virtuelles Hyper-V dans Azure** | **Réplication de machines virtuelles VMware vers un site secondaire** | **Réplication de machines virtuelles VMware dans Azure** | **Réplication de serveurs physiques vers un site secondaire** | **Répliquer des serveurs physiques dans Azure** |
|---| --- | --- | --- | --- | --- | --- | --- |
| **Bureau virtuel en pool (non géré)** |Non|Oui |Non |Oui |Non |Oui |Non |
| **Bureau virtuel en pool (géré et sans UPD)** |Non|Oui |Non |Oui |Non |Oui |Non |
| **Applications à distance et sessions de bureau (sans UPD)** |Oui|Oui |Oui |Oui |Oui |Oui |Oui |

[En savoir plus](/windows-server/remote/remote-desktop-services/rds-disaster-recovery-with-azure) sur la reprise d’activité pour RDS.

## <a name="protect-exchange"></a>Protéger Exchange

Site Recovery vous permet de protéger Exchange comme suit :

- Site Recovery peut répliquer et basculer les déploiements Exchange de petite taille, par exemple des serveurs uniques ou autonomes, vers Azure ou un site secondaire.
- Pour les déploiements plus importants, Site Recovery s’intègre avec les groupes de disponibilité de base de données (DAG) Exchange.
- Les groupes de disponibilité de base de données (DAG) Exchange sont la solution recommandée pour la récupération d’urgence Exchange dans une entreprise.  Les plans de récupération Site Recovery peuvent inclure des groupes de disponibilité de base de données (DAG) pour orchestrer le basculement de ceux-ci entre les sites.

Pour en savoir plus sur la reprise d’activité pour Exchange, consultez [Groupes de disponibilité de base de données Exchange](/Exchange/high-availability/database-availability-groups/database-availability-groups) et [Reprise d’activité Exchange](/Exchange/high-availability/disaster-recovery/disaster-recovery).

## <a name="protect-sap"></a>Protéger SAP

Utilisez Site Recovery pour protéger votre déploiement SAP comme suit :

- Activez la protection des applications de production SAP NetWeaver et non-NetWeaver exécutées localement, par la réplication des composants sur Azure.
- Activez la protection des applications de production SAP NetWeaver et non-NetWeaver exécutant Azure, par la réplication des composants sur un autre centre de données Azure.
- Simplifiez la migration vers le cloud en utilisant Site Recovery pour migrer votre déploiement SAP vers Azure.
- Simplifiez les mises à niveau, les tests et la création de prototypes du projet SAP, par la création d’un clone de production à la demande pour tester les applications SAP.

[En savoir plus](site-recovery-sap.md) sur la reprise d’activité pour SAP.

## <a name="protect-internet-information-services"></a>Protéger Internet Information Services

Utilisez Site Recovery pour protéger votre déploiement Internet Information Services (IIS), comme suit :

Azure Site Recovery assure la récupération d’urgence en répliquant les composants critiques de votre environnement vers un site distant à froid ou un cloud public comme Microsoft Azure. Étant donné que les machines virtuelles avec le serveur web et la base de données sont répliquées vers le site de récupération, il n’est pas nécessaire de sauvegarder séparément les certificats et les fichiers de configuration. Les mappages et les liaisons d’application dépendant des variables d’environnement modifiées post-basculement peuvent être mis à jour via des scripts intégrés dans les plans de récupération d’urgence. Les machines virtuelles sont montées sur le site de récupération uniquement pendant un basculement. Azure Site Recovery vous aide également à orchestrer le basculement de bout en bout en mettant à disposition les fonctionnalités suivantes :

- Séquencement de l’arrêt et du démarrage des machines virtuelles dans les différents niveaux.
- Ajout de scripts pour autoriser les mises à jour des liaisons et des dépendances d’applications sur les machines virtuelles après leur démarrage. Les scripts peuvent également être utilisés pour mettre à jour le serveur DNS pour pointer vers le site de récupération.
- Allouez des adresses IP aux machines virtuelles pré-basculement en mappant les réseaux principal et de récupération, et utilisez des scripts qui ne nécessitent pas de mise à jour post-basculement.
- Possibilité de basculement en un clic pour plusieurs applications web, qui limite ainsi la confusion en cas de sinistre.
- Possibilité de tester les plans de récupération dans un environnement isolé pour les exercices de récupération d’urgence.

[En savoir plus](site-recovery-iis.md) sur la reprise d’activité pour IIS.

## <a name="protect-citrix-xenapp-and-xendesktop"></a>Protéger Citrix XenApp et XenDesktop

Site Recovery permet de protéger vos déploiements Citrix XenApp XenDesktop, comme suit :

- Activez la protection du déploiement de Citrix XenApp et XenDesktop. Répliquez les différentes couches de déploiement dans Azure : Active Directory, serveur DNS, serveur SQL Database, Citrix Delivery Controller, serveur StoreFront, XenApp Master (VDA) et serveur de licences Citrix XenApp.
- Simplifiez la migration vers le cloud en utilisant Site Recovery pour migrer votre déploiement Citrix XenApp et XenDesktop sur Azure.
- Simplifiez le test de Citrix XenApp/XenDesktop en créant une copie de type production à la demande pour le test et le débogage.
- Cette solution s’applique uniquement aux bureaux virtuels Windows Server et non aux bureaux virtuels clients. Les bureaux virtuels clients ne sont pas encore pris en charge pour la gestion des licences dans Azure. [Apprenez-en plus](https://azure.microsoft.com/pricing/licensing-faq/) sur les licences pour les bureaux client/serveur dans Azure.

[En savoir plus](site-recovery-citrix-xenapp-and-xendesktop.md) sur la reprise d’activité pour les déploiements Citrix XenApp et XenDesktop. Vous pouvez également consulter le [livre blanc Citrix](https://aka.ms/citrix-xenapp-xendesktop-with-asr).

## <a name="next-steps"></a>Étapes suivantes

[En savoir plus](azure-to-azure-quickstart.md) sur la reprise d’activité pour une machine virtuelle Azure.

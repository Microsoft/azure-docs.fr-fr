---
title: Services et technologies de sécurité Azure | Microsoft Docs
description: Cet article fournit une liste des services et technologies de sécurité Azure.
services: security
documentationcenter: na
author: terrylanfear
manager: barbkess
editor: TomSh
ms.assetid: a5a7f60a-97e2-49b4-a8c5-7c010ff27ef8
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 1/29/2019
ms.author: terrylan
ms.openlocfilehash: 8a33e9d462f3105bfee295ac82384a38325f7c7c
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96498793"
---
# <a name="security-services-and-technologies-available-on-azure"></a>Services et technologies de sécurité disponibles sur Azure

Dans nos discussions avec les clients Azure actuels et futurs, une question revient souvent : « Avez-vous une liste de tous les services et technologies de sécurité proposés par Azure ? ».

Quand vous évaluez les options d’un fournisseur de services cloud, il est utile de disposer de ces informations. Nous avons donc confectionné cette liste pour vous aider à démarrer.

Au fil du temps, cette liste sera modifiée et développée, tout comme Azure. Veillez à consulter cette page régulièrement pour vous tenir au courant de l’évolution de nos technologies et services liés à la sécurité.

## <a name="general-azure-security"></a>Sécurité générale d’Azure
|Service|Description|
|--------|--------|
|[Azure&nbsp;Security&nbsp;Center](../../security-center/security-center-introduction.md)| Solution de protection des charges de travail cloud qui fournit une gestion de la sécurité et une protection avancée contre les menaces sur l’ensemble des charges de travail cloud hybrides.|
|[Azure Key Vault](../../key-vault/general/overview.md)| Magasin de secrets sécurisé pour les mots de passe, les chaînes de connexion et d’autres informations dont vous avez besoin pour que vos applications continuent de fonctionner. |
|[Journaux d’activité Azure Monitor](../../azure-monitor/log-query/log-query-overview.md)|Service de surveillance qui collecte des données de télémétrie et d’autres données, et qui fournit un langage de requête et un moteur analytique permettant d’obtenir des insights sur le fonctionnement de vos applications et de vos ressources. Peut être utilisé seul ou avec d’autres services, comme Security Center. |
|[Dev/Test Labs Azure](../../devtest-labs/devtest-lab-overview.md)|Service permettant aux développeurs et aux testeurs de créer rapidement des environnements dans Azure, tout en réduisant le temps perdu et les coûts.  |

<!---|[Azure&nbsp;Disk&nbsp;Encryption](/azure/azure-security-disk-encryption-overview)| THIS WILL GO TO THE NEW OVERVIEW TOPIC MEGHAN STEWART IS WRITING|--->

## <a name="storage-security"></a>Sécurité du stockage
|Service|Description|
|------|--------|
| [Azure&nbsp;Storage&nbsp;Service&nbsp;Encryption](../../storage/common/storage-service-encryption.md)|Fonctionnalité de sécurité qui chiffre automatiquement vos données dans le stockage Azure.   |
|[Stockage hybride chiffré StorSimple](../../storsimple/storsimple-ova-overview.md)| Solution de stockage intégrée qui gère les tâches de stockage entre les appareils locaux et le stockage cloud Azure.|
|[Chiffrement côté client Azure](../../storage/common/storage-client-side-encryption.md)| Solution de chiffrement côté client qui chiffre les données dans les applications clientes avant de les charger sur Stockage Azure ; elle déchiffre aussi les données lors du téléchargement. |
| [Signatures d’accès partagé Azure Storage](../../storage/common/storage-sas-overview.md)|Une signature d'accès partagé fournit un accès délégué aux ressources de votre compte de stockage.  |
|[Clés de compte de stockage Azure](../../storage/common/storage-account-create.md)| Méthode de contrôle d’accès pour le stockage Azure, qui est utilisée pour l’authentification lors de l’accès au compte de stockage. |
|[Partages de fichiers Azure avec chiffrement SMB 3.0](../../storage/files/storage-files-introduction.md)|Technologie de sécurité réseau qui permet le chiffrement automatique du réseau pour le protocole de partage de fichiers Server Message Block (SMB). |
|[Azure Storage Analytics](/rest/api/storageservices/Storage-Analytics)| Technologie de journalisation et de génération de métriques pour les données dans votre compte de stockage. |

<!------>

## <a name="database-security"></a>Sécurité de la base de données
|Service|Description|
|------|--------|
| [Pare-feu&nbsp;SQL&nbsp;Azure](../../azure-sql/database/firewall-configure.md)|Fonctionnalité de contrôle d’accès réseau qui protège la base de données contre les attaques réseau. |
|[Chiffrement&nbsp;au niveau des cellules&nbsp;SQL&nbsp;Azure](/archive/blogs/sqlsecurity/recommendations-for-using-cell-level-encryption-in-azure-sql-database)| Technologie de sécurité des bases de données qui fournit un chiffrement à un niveau granulaire.  |
| [Chiffrement de connexion&nbsp;SQL&nbsp;Azure](../../azure-sql/database/logins-create-manage.md)|Pour assurer la sécurité, SQL Database contrôle l’accès avec des règles de pare-feu qui limitent la connectivité par adresse IP, des mécanismes d’authentification qui obligent les utilisateurs à prouver leur identité, et des mécanismes d’autorisation qui les restreignent à certaines actions et données. |
| [Chiffrement systématique SQL Azure](/sql/relational-databases/security/encryption/always-encrypted-database-engine?view=sql-server-2017)|Protège les données sensibles, comme les numéros de carte bancaire ou les numéros d’identification (par exemple les numéros de sécurité sociale), stockées dans Azure SQL Database ou dans des bases de données SQL Server.  |
| [Azure&nbsp;SQL&nbsp;Transparent Data Encryption](/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql?view=azuresqldb-current)| Fonctionnalité de sécurité de base de données qui chiffre le stockage de toute une base de données. |
| [Audit Azure SQL Database](../../azure-sql/database/auditing-overview.md)|Fonctionnalité d’audit de base de données qui suit les événements de base de données et les consigne dans un journal d’audit, dans votre compte de stockage Azure.  |


## <a name="identity-and-access-management"></a>Gestion de l’identité et de l’accès
|Service|Description|
|------|--------|
| [Contrôle d’accès&nbsp;en fonction du rôle&nbsp;Azure](../../role-based-access-control/role-assignments-portal.md)|Fonctionnalité de contrôle d’accès conçue pour autoriser les utilisateurs à accéder seulement aux ressources auxquelles ils doivent accéder en fonction de leur rôle au sein de l’organisation.  |
| [Azure Active Directory](../../active-directory/fundamentals/active-directory-whatis.md)|Référentiel d’authentification cloud qui prend en charge un annuaire multilocataire dans le cloud et plusieurs services de gestion des identités dans Azure.  |
| [Azure Active Directory B2C](../../active-directory-b2c/overview.md)|Service de gestion des identités qui permet de contrôler la façon dont les clients s’inscrivent, se connectent et gèrent leurs profils quand ils utilisent vos applications basées sur Azure.   |
| [Services de domaine Azure Active Directory](../../active-directory-domain-services/overview.md)| Version cloud et managée d’Active Directory Domain Services. |
| [Azure AD Multi-Factor Authentication](../../active-directory/authentication/concept-mfa-howitworks.md)| Provision de sécurité qui utilise plusieurs formulaires d’authentification et de vérification avant d’autoriser l’accès à des informations sécurisées. |

## <a name="backup-and-disaster-recovery"></a>Sauvegarde et récupération d'urgence
|Service|Description|
|------|--------|
| [Sauvegarde&nbsp;Azure](../../backup/backup-overview.md)| Service basé sur Azure utilisé pour sauvegarder et restaurer des données dans le cloud Azure. |
| [Azure&nbsp;Site&nbsp;Recovery](../../site-recovery/site-recovery-overview.md)|Service en ligne qui réplique les charges de travail s’exécutant sur des machines virtuelles et physiques depuis un site principal vers un emplacement secondaire pour permettre la récupération des services après une défaillance. |

## <a name="networking"></a>Mise en réseau
|Service|Description|
|------|--------|
| [Groupes&nbsp;de sécurité&nbsp;réseau](../../virtual-network/virtual-network-vnet-plan-design-arm.md)| Fonctionnalité de contrôle d’accès basée sur le réseau utilisant un tuple de 5 éléments pour prendre les décisions d’autoriser ou de refuser.  |
| [Passerelle VPN Azure](../../vpn-gateway/vpn-gateway-about-vpngateways.md)| Appareil réseau utilisé comme un point de terminaison de réseau privé virtuel pour autoriser les accès locaux à des réseaux virtuels Azure.  |
| [Application Gateway Azure](../../application-gateway/overview.md)|Équilibreur de charge pour les applications web avancées qui peut router en fonction des URL et effectuer un déchargement SSL. |
|[Pare-feu d’applications web](../../web-application-firewall/afds/afds-overview.md) (WAF)|Fonctionnalité d’Application Gateway qui permet de protéger de manière centralisée vos applications web contre les vulnérabilités courantes et le code malveillant exploitant.|
| [Équilibrage de charge Azure](../../load-balancer/load-balancer-overview.md)|Équilibreur de charge réseau pour les applications TCP/UDP. |
| [Azure ExpressRoute](../../expressroute/expressroute-introduction.md)| Lien WAN dédié entre des réseaux locaux et des réseaux virtuels Azure. |
| [Azure Traffic Manager](../../traffic-manager/traffic-manager-overview.md)| Équilibreur de charge DNS global.|
| [Proxy d’application Azure](../../active-directory/manage-apps/application-proxy.md)| Frontend d’authentification utilisé pour sécuriser l’accès à distance pour des applications web hébergées en local. |
|[Pare-feu Azure](../../firewall/overview.md)|Service de sécurité réseau cloud managé qui protège vos ressources Réseau virtuel Azure.|
|[Protection DDoS Azure](../../ddos-protection/ddos-protection-overview.md)|Associé aux bonnes pratiques de conception d’application, assure une excellente protection contre les attaques DDoS.|
|[Points de terminaison de service de réseau virtuel](../../virtual-network/virtual-network-service-endpoints-overview.md)|Étend votre espace d’adressage privé de réseau virtuel et l’identité de votre réseau virtuel aux services Azure, par le biais d’une connexion directe.|
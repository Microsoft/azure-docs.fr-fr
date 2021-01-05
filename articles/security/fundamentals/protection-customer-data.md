---
title: Protection des données des clients dans Azure
description: Découvrez comment Azure protège les données client par le biais de la séparation des données, de la redondance des données et de la destruction des données.
services: security
documentationcenter: na
author: TerryLanfear
manager: barbkess
editor: TomSh
ms.assetid: 61e95a87-39c5-48f5-aee6-6f90ddcd336e
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/10/2020
ms.author: terrylan
ms.openlocfilehash: 14589e4efe22d89468b069bf6ff7e3d9babcc714
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87543790"
---
# <a name="azure-customer-data-protection"></a>Protection des données client Azure   
L’accès aux données clientes par le personnel des opérations et du support Microsoft est refusé par défaut. Lorsque l'accès aux données liées à un cas de support est accordé, il n'est accordé qu'avec un modèle juste-à-temps utilisant des stratégies qui sont auditées et vérifiées par rapport à nos stratégies de conformité et de confidentialité.  Les conditions de contrôle d’accès sont établies par la stratégie de sécurité Azure suivante :

- Aucun accès aux données client par défaut.
- Aucun compte d’utilisateur ou d’administrateur sur les machines virtuelles des clients.
- Donner le minimum de privilèges obligatoire pour effectuer une tâche ; audit et journalisation des demandes d’accès.

Le personnel du support Azure se voit attribuer des comptes Active Directory d’entreprise uniques par Microsoft. Microsoft Azure s’appuie sur Microsoft Corporate Active Directory, géré par MSIT (Microsoft Information Technologies), pour contrôler l’accès aux principaux systèmes d’informations. L’authentification multifacteur est obligatoire et l’accès est accordé uniquement à partir de consoles sécurisées.

Toutes les tentatives d’accès sont surveillées et peuvent être affichées dans un ensemble de rapports simple.

## <a name="data-protection"></a>Protection de données
Azure fournit aux clients une sécurité renforcée des données, à la fois par défaut et comme options pour les clients.

**Ségrégation des données :** Azure est un service multilocataire, ce qui signifie que plusieurs machines virtuelles et déploiements clients sont stockés sur le même matériel physique. Azure utilise une isolation logique pour séparer les données de chaque client les unes des autres. La séparation offre les avantages financiers et d’échelle des services multilocataires, tout en empêchant rigoureusement les clients d’accéder aux données des autres.

**Protection des données au repos** : les clients sont tenus de veiller à ce que les données stockées dans Azure soient chiffrées conformément aux normes en vigueur. Azure propose toute une série de fonctionnalités de chiffrement qui permettent aux clients de choisir la solution correspondant le mieux à leurs besoins. Azure Key Vault aide les clients à assurer facilement le contrôle des clés utilisées par les applications et les services cloud pour chiffrer les données. Azure Disk Encryption permet aux clients de chiffrer les machines virtuelles. Azure Storage Service Encryption offre la possibilité de chiffrer toutes les données placées dans le compte de stockage du client.

**Protection des données en transit** : Microsoft propose un certain nombre d'options qui peuvent être utilisées par les clients pour sécuriser les données en transit en interne au sein du réseau Azure, et en externe sur Internet, pour l'utilisateur final.  Il s'agit notamment des communications par le biais de réseaux privés virtuels (utilisant le chiffrement IPsec/IKE), de Transport Layer Security (TLS) 1.2 ou version ultérieure (via des composants Azure tels qu'Application Gateway ou Azure Front Door), de protocoles utilisés directement sur les machines virtuelles Azure (comme Windows IPsec ou SMB), etc. 

En outre, le « chiffrement par défaut » à l'aide de MACsec (norme IEEE au niveau de la couche de liaison de données) est activé pour tout le trafic Azure qui transite entre les centres de données Azure afin de garantir la confidentialité et l'intégrité des données client. 

**Redondance des données** : Microsoft contribue à assurer la protection des données en cas de cyberattaque ou de dommages physiques sur un centre de données. Les clients peuvent choisir :

- Un stockage dans leur pays/région pour des raisons de conformité ou de latence.
- Un stockage hors de leur pays/région pour des raisons de sécurité ou de reprise d’activité après sinistre.

Les données peuvent être répliquées au sein d’une zone géographique sélectionnée pour la redondance, mais ne sont pas transmises en dehors. Les clients ont plusieurs options pour la réplication des données, notamment le nombre de copies et le nombre d’emplacements des centres de données de réplication.

Quand vous créez votre compte de stockage, vous devez sélectionner une des options de réplication suivantes :

- **Stockage localement redondant (LRS)**  : Le stockage localement redondant conserve trois copies de vos données. Le stockage LRS est répliqué trois fois dans une même installation d’une même région. Le stockage LRS protège vos données contre les défaillances matérielles normales, mais pas des pannes susceptibles de se produire dans une installation donnée.
- **Stockage redondant interzone (ZRS)**  : Le stockage redondant interzone effectue trois copies de vos données. Le stockage ZRS est répliqué trois fois sur deux à trois installations pour fournir une plus grande durabilité que celle du stockage LRS. La réplication a lieu dans une même région ou dans deux régions. Le stockage ZRS garantit la durabilité de vos données dans une même région.
- **Stockage géo-redondant (GRS)**  : Le stockage géoredondant est activé par défaut sur votre compte de stockage lors de la création de celui-ci. Le stockage GRS conserve six copies de vos données. Avec le stockage GRS, vos données sont répliquées trois fois dans la région principale. Vos données sont aussi répliquées trois fois dans une région secondaire située à des centaines de kilomètres de la région principale pour offrir le plus haut niveau de durabilité. En cas de défaillance dans la région principale, le stockage Azure bascule sur la région secondaire. Le stockage GRS assure la durabilité de vos données dans deux régions distinctes.

**Destruction des données** : lorsque les clients suppriment des données ou quittent Azure, Microsoft suit des normes strictes pour écraser les ressources de stockage avant leur réutilisation, ainsi que pour la destruction physique du matériel mis hors service. Microsoft supprime la totalité des données à la demande du client et à la fin du contrat.

## <a name="customer-data-ownership"></a>Propriété des données clientes
Microsoft ne procède à aucune inspection, approbation ou surveillance des applications que les clients déploient dans Azure. Microsoft ne sait pas non plus quel type de données les clients choisissent de stocker dans Azure. Microsoft ne revendique pas la propriété des informations des clients entrées dans Azure.

## <a name="records-management"></a>Gestion des enregistrements
Azure a établi des conditions de conservation d’enregistrements internes pour les données backend. Les clients sont chargés d’identifier leurs propres conditions de conservation d’enregistrements. Pour les enregistrements stockés dans Azure, le client est responsable de l’extraction de ses données et de la conservation du contenu en dehors d’Azure pendant la période de conservation qu’il aura spécifiée.

Azure offre au client la possibilité d’exporter des données et d’auditer des rapports à partir du produit. Les exportations sont enregistrées localement pour garder les informations pendant la période de conservation définie par le client.

## <a name="electronic-discovery-e-discovery"></a>Découverte électronique (e-discovery)
Les clients Azure son tenus de se conformer aux conditions d’e-discovery quand ils utilisent les services Azure. Si les clients Azure doivent conserver leurs données, ils peuvent les exporter et les enregistrer localement. Les clients peuvent aussi demander des exportations de leurs données auprès du service de support client Azure. En plus de permettre aux clients d’exporter leurs données, Azure procède à une journalisation et une surveillance complètes en interne.

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur ce que Microsoft fait pour sécuriser l’infrastructure Azure, consultez :

- [Sécurité locale et physique des centres de données Azure](physical-security.md)
- [Disponibilité de l’infrastructure Azure](infrastructure-availability.md)
- [Composants et limites du système d’informations Azure](infrastructure-components.md)
- [Architecture réseau Azure](infrastructure-network.md)
- [Réseau de production Azure](production-network.md)
- [Fonctionnalités de sécurité d’Azure SQL Database](infrastructure-sql.md)
- [Opérations de production et administration Azure](infrastructure-operations.md)
- [Surveillance de l’infrastructure Azure](infrastructure-monitoring.md)
- [Intégrité de l’infrastructure Azure](infrastructure-integrity.md)

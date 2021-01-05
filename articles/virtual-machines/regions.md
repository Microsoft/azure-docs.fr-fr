---
title: Régions Azure
description: Découvrez les régions pour l’exécution de machines virtuelles dans Azure.
author: cynthn
ms.author: cynthn
ms.service: virtual-machines
ms.topic: conceptual
ms.workload: infrastructure-services
ms.date: 05/28/2019
ms.openlocfilehash: 094b78fa35da451101f4f5664cca46292d78d146
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88650551"
---
# <a name="regions-for-virtual-machines-in-azure"></a>Régions pour les machines virtuelles dans Azure

Il est important de comprendre le fonctionnement des machines virtuelles et de connaître leur emplacement dans Azure, ainsi que les différentes options disponibles pour optimiser les performances, la disponibilité et la redondance. Cet article fournit une vue d’ensemble des fonctionnalités d’optimisation de la disponibilité et de redondance d’Azure.


## <a name="what-are-azure-regions"></a>Que représentent les régions Azure ?
Azure fonctionne dans plusieurs centres de données à travers le monde. Ces centres de données sont regroupés en régions géographiques, ce qui vous donne une certaine flexibilité dans le choix de l’emplacement où vous créez vos applications. 

Vous créez des ressources Azure dans des régions géographiques définies comme « USA Ouest », « Europe Nord » ou « Asie Sud-Est ». Vous pouvez consulter la [liste de ces régions et leurs emplacements](https://azure.microsoft.com/regions/). Chaque région comporte plusieurs centres de données, afin de garantir la redondance et la disponibilité. Lorsque vous concevez vos applications, vous bénéficiez ainsi d’une certaine flexibilité pour créer des machines virtuelles plus proches de vos utilisateurs et pour répondre à n’importe quel besoin juridique, de conformité ou fiscal.

## <a name="special-azure-regions"></a>Régions Azure spéciales
Lors de la création de vos applications, vous pouvez utiliser certaines régions Azure spéciales pour répondre à vos besoins de conformité ou juridiques. Ces régions spéciales sont les suivantes :

* **US Gov Virginie** et **US Gov Iowa**
  * Instance physique et logique d’Azure isolée du réseau pour les administrations et les partenaires des États-Unis, gérée par un personnel autorisé aux États-Unis. Inclut des certifications de conformité supplémentaires comme [FedRAMP](https://www.microsoft.com/en-us/TrustCenter/Compliance/FedRAMP) et [DISA](https://www.microsoft.com/en-us/TrustCenter/Compliance/DISA). En savoir plus sur [Azure Government](https://azure.microsoft.com/features/gov/).
* **Est de la Chine** et **Nord de la Chine**
  * Ces régions sont disponibles grâce à un partenariat unique conclu entre Microsoft et 21Vianet, qui stipule que Microsoft ne gère pas directement les centres de données. Découvrez plus d’informations sur [Azure Chine géré par 21Vianet](https://www.windowsazure.cn/).
* **Centre de l’Allemagne** et **Nord-Est de l’Allemagne**
  * Ces régions sont disponibles par le biais d’un modèle d’administration des données, dans lequel les données client sont conservées en Allemagne sous le contrôle de T-Systems, société Deutsche Telekom responsable de l’administration des données allemandes.

## <a name="region-pairs"></a>Paires de régions
Chaque région Azure est associée à une autre région au sein de la même région géographie (par exemple, États-Unis, Europe ou Asie). Cette approche permet la réplication des ressources, par exemple le stockage des machines virtuelles, sur l’ensemble d’une région, réduisant ainsi la probabilité de catastrophes naturelles, de troubles civils, de coupures de courant ou de pannes de réseau physique affectant les deux régions en même temps. Les paires de régions offrent les avantages supplémentaires suivants :

* En cas d’une panne Azure plus importante, une région est hiérarchisée dans chaque paire afin de réduire le temps de restauration des applications. 
* Les mises à jour Azure planifiées sont déployées vers les régions jumelées une par une pour limiter les interruptions de service et les risques de panne de l’application.
* Les données continuent de résider dans la même zone géographique que la région avec laquelle elle est jumelée (sauf Brésil Sud) pour répondre aux exigences en termes d’impôts et d’application de la loi.

Voici quelques exemples de paires de régions :

| Principal | Secondary |
|:--- |:--- |
| USA Ouest |USA Est |
| Europe Nord |Europe Ouest |
| Asie Sud-Est |Asie Est |

Vous pouvez afficher la [liste des paires de régions ici](../best-practices-availability-paired-regions.md#what-are-paired-regions).

## <a name="feature-availability"></a>Disponibilité des fonctionnalités
Certains services ou fonctionnalités des machines virtuelles ne sont disponibles que dans certaines régions, par exemple des tailles de machines virtuelles ou des types de stockage spécifiques. De même, certains services Azure globaux ne vous obligent pas à sélectionner une région particulière, par exemple [Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md), [Traffic Manager](../traffic-manager/traffic-manager-overview.md), ou [Azure DNS](../dns/dns-overview.md). Pour vous aider à concevoir votre environnement d’application, vous pouvez vérifier la [disponibilité des services Azure dans chaque région](https://azure.microsoft.com/regions/#services). Vous pouvez également [interroger par programmation les tailles de machine virtuelle et les restrictions prises en charge dans chacune des régions](../azure-resource-manager/templates/error-sku-not-available.md).

## <a name="storage-availability"></a>Disponibilité du stockage
Il est important de bien comprendre les régions géographiques Azure lorsque vous considérez les options de réplication de stockage disponibles. Selon le type de stockage, vous disposez d’options de réplication différentes.

**Azure Disques managés**
* Stockage localement redondant (LRS)
  * Réplique trois fois vos données dans la région dans laquelle vous avez créé votre compte de stockage.

**Disques basés sur un compte de stockage**
* Stockage localement redondant (LRS)
  * Réplique trois fois vos données dans la région dans laquelle vous avez créé votre compte de stockage.
* Stockage redondant dans une zone (ZRS)
  * Réplique trois fois vos données dans deux ou trois installations au sein d’une même région ou dans deux régions distinctes.
* Stockage géo-redondant (GRS)
  * Réplique vos données vers une région secondaire à des centaines de kilomètres de la région primaire.
* Stockage géo-redondant avec accès en lecture (RA-GRS)
  * Réplique vos données dans une région secondaire, comme avec GRS, mais fournit également un accès en lecture seule aux données dans l’emplacement secondaire.

Le tableau suivant fournit une rapide vue d’ensemble des différences entre les types de réplications de stockage :

| Stratégie de réplication | LRS | ZRS | GRS | RA-GRS |
|:--- |:--- |:--- |:--- |:--- |
| Les données sont répliquées entre plusieurs installations. |Non |Oui |Oui |Oui |
| Les données peuvent être lues à partir de l’emplacement secondaire comme de l’emplacement principal. |Non |Non |Non |Oui |
| Nombre de copies de données conservées sur des nœuds distincts. |3 |3 |6 |6 |

Vous pouvez en savoir plus sur les [options de réplication de stockage Azure ici](../storage/common/storage-redundancy.md). Pour plus d’informations sur les disques managés, consultez [Vue d’ensemble d’Azure Disques managés](./managed-disks-overview.md).

### <a name="storage-costs"></a>Coûts de stockage
Les prix varient selon le type de stockage et la disponibilité que vous sélectionnez.

**Azure Disques managés**
* Les disques managés Premium s’appuient sur des disques SSD et les disques managés Standard sur des disques à rotation classique. Les fonctionnalités Disques managés Premium et Standard sont facturées en fonction de la capacité déployée pour le disque.

**Disques non managés**
* Le stockage Premium s’appuie sur des disques SSD et est facturé en fonction de la capacité du disque.
* Le stockage standard s’appuie sur des disques rotatifs ordinaires et est facturé en fonction de la capacité en cours d’utilisation et la disponibilité de stockage souhaitée.
  * Pour RA-GRS, les frais de transfert de données de géo-réplication supplémentaires correspondent au coût de la bande passante nécessaire à la réplication de ces données vers une autre région Azure.

Consultez [Tarification Azure Storage](https://azure.microsoft.com/pricing/details/storage/) pour connaître les informations de tarification des différents types de stockage et options de disponibilité.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez [Régions Azure](https://azure.microsoft.com/global-infrastructure/regions/).
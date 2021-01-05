---
title: Gestion centrale de Pare-feu Azure
description: En savoir plus sur la gestion centrale d’Azure Firewall Manager
author: vhorne
ms.service: firewall
services: firewall
ms.topic: conceptual
ms.date: 07/13/2020
ms.author: victorh
ms.openlocfilehash: 23a7682d8a64de57db4ff9ae785ada90d4a06944
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87084668"
---
# <a name="azure-firewall-central-management"></a>Gestion centrale de Pare-feu Azure

Si vous gérez plusieurs pare-feu, vous savez que les règles de pare-feu qui changent continuellement compliquent leur synchronisation. Les équipes informatiques centrales doivent disposer d’un moyen de définir des stratégies de pare-feu de base et de les appliquer à plusieurs unités commerciales. En même temps, les équipes DevOps veulent créer leurs propres stratégies de pare-feu dérivées locales pour une meilleure agilité.

Azure Firewall Manager peut aider à résoudre ces problèmes.


## <a name="azure-firewall-manager"></a>Azure Firewall Manager

Azure Firewall Manager est un service de gestion de la sécurité réseau qui fournit une stratégie de sécurité centralisée et la gestion des itinéraires pour les périmètres de sécurité informatiques. Il permet aux équipes informatiques des entreprises de définir de manière centralisée des règles de niveau réseau et application pour filtrer le trafic sur plusieurs instances Pare-feu Azure. Vous pouvez couvrir différentes régions et différents abonnements Azure dans les architectures Hub-and-spoke pour la gouvernance et la protection du trafic. Il offre également à DevOps une meilleure agilité avec des stratégies de sécurité de pare-feu locales dérivées qui sont implémentées dans les organisations.

### <a name="firewall-policy"></a>Stratégie de pare-feu

Une stratégie de pare-feu est une ressource Azure qui contient des collections de règles NAT, de réseau et d’application ainsi que des paramètres de renseignement sur les menaces. Cette ressource globale peut s’utiliser sur plusieurs instances Pare-feu Azure dans des *hubs virtuels sécurisés* et des *réseaux virtuels hubs*. Les stratégies peuvent être créées à partir de zéro ou être héritées de stratégies existantes. Avec l’héritage, DevOps crée des stratégies de pare-feu locales en plus de la stratégie de base mise en place dans l’organisation. Les stratégies sont applicables sur plusieurs régions et abonnements.
 
Vous pouvez créer une stratégie de pare-feu et des associations à l’aide d’Azure Firewall Manager. Toutefois, vous pouvez également créer et gérer une stratégie à l’aide de l’API REST, de modèles, d’Azure PowerShell et de l’interface CLI. Une fois que vous avez créé une stratégie, vous pouvez l’associer à un pare-feu dans un hub WAN virtuel, ce qui en fait un *hub virtuel sécurisé*, et/ou à un pare-feu dans un réseau virtuel, ce qui en fait un *réseau virtuel hub*.

### <a name="pricing"></a>Tarifs

Les stratégies sont facturées en fonction des associations de pare-feu. Une stratégie avec zéro ou une seule association de pare-feu est gratuite. Une stratégie avec plusieurs associations de pare-feu est facturée à un tarif fixe. Pour plus d’informations, consultez [Tarification Azure Firewall Manager](https://azure.microsoft.com/pricing/details/firewall-manager/).

## <a name="azure-firewall-management-partners"></a>Partenaires de gestion de Pare-feu Azure

Les principales solutions tierces suivantes prennent en charge la gestion centrale de Pare-feu Azure à l’aide des API REST Azure standard. Chacune de ces solutions a ses propres caractéristiques et fonctionnalités uniques :

- [AlgoSec CloudFlow](https://www.algosec.com/azure/) 
- [Barracuda Cloud Security Guardian](https://www.barracuda.com/products/cloudsecurityguardian/for_azure)
- [Tufin Orca](https://www.tufin.com/products/tufin-orca)


## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur Azure Firewall Manager, consultez [Qu’est-ce qu’Azure Firewall Manager ?](../firewall-manager/overview.md)
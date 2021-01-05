---
author: dlepow
ms.service: container-instances
ms.topic: include
ms.date: 07/22/2020
ms.author: danlep
ms.openlocfilehash: 6878180ffedfaa53f25d2bdc6db72dcd7dd8b38b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87384823"
---
| Ressource | Limite |
| --- | :--- |
| Groupes de conteneurs de la référence SKU Standard par région et par abonnement | 100<sup>1</sup> |
| Groupes de conteneurs de la référence SKU dédiée par région et par abonnement | 0<sup>1</sup> |
| Nombre de conteneurs par groupe de conteneurs | 60 |
| Nombre de volumes par groupe de conteneurs | 20 |
| Cœurs de la référence SKU Standard (CPU) par région par abonnement | 10<sup>1,2</sup> | 
| Cœurs de la référence SKU Standard (CPU) pour GPU K80 par région par abonnement | 18<sup>1,2</sup> |
| Cœurs de la référence SKU Standard (CPU) pour GPU P100 ou V100 par région par abonnement | 0<sup>1,2</sup> |
| Ports par adresse IP | 5 |
| Taille de journal d’instance de conteneur - instance en cours d’exécution | 4 Mo |
| Taille de journal d’instance de conteneur - instance arrêtée | 16 Ko ou 1 000 lignes |
| Création de conteneurs par heure |300<sup>1</sup> |
| Création de conteneurs par tranche de 5 minutes | 100<sup>1</sup> |
| Suppression de conteneurs par heure | 300<sup>1</sup> |
| Suppression de conteneurs par tranche de 5 minutes | 100<sup>1</sup> |


<sup>1</sup>Pour réclamer un relèvement de la limite, créez une [demande de support Azure][azure-support]. Les abonnements gratuits, notamment le [compte gratuit Azure](https://azure.microsoft.com/offers/ms-azr-0044p/) et [Azure for Students](https://azure.microsoft.com/offers/ms-azr-0170p/) ne sont pas éligibles à des augmentations de limite ou de quota. Si vous avez un abonnement gratuit, vous pouvez [passer](../articles/cost-management-billing/manage/upgrade-azure-subscription.md) à un abonnement Paiement à l’utilisation.<br />
<sup>2</sup>Limite par défaut pour l’abonnement [Paiement à l’utilisation](https://azure.microsoft.com/offers/ms-azr-0003p/). La limite peut être différente pour les autres types de catégorie.<br/>

<!-- LINKS - External -->
[azure-support]: https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest

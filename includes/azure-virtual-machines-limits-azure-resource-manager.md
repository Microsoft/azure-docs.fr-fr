---
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 02/10/2020
ms.author: cynthn
ms.openlocfilehash: a9c830025b9ce6c93bf106a8348b0f9dc6521582
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95555183"
---
| Ressource | Limite |
| --- | --- |
| Machines virtuelles par [abonnement](https://azure.microsoft.com/pricing/) |25 000<sup>1</sup> par région. |
| Nombre total de cœurs de machine virtuelle par [abonnement](https://azure.microsoft.com/pricing/) |20<sup>1</sup> par région. Contactez le support pour augmenter la limite. |
| Nombre total de cœurs de machine virtuelle Azure Spot par [abonnement](https://azure.microsoft.com/pricing/) |20<sup>1</sup> par région. Contactez le support pour augmenter la limite. |
| Machine virtuelle par série, telle que Dv2, et F, cœurs par [abonnement](https://azure.microsoft.com/pricing/) |20<sup>1</sup> par région. Contactez le support pour augmenter la limite. |
| [Groupes à haute disponibilité](../articles/virtual-machines/manage-availability.md#configure-multiple-virtual-machines-in-an-availability-set-for-redundancy) par abonnement |2 500 par région. |
| Machines virtuelles par groupe à haute disponibilité | 200 |
| Certificats par abonnement |Illimité<sup>2</sup> |

<sup>1</sup>Les limites par défaut varient selon le type de catégorie d’offre, comme la version d’évaluation gratuite et le paiement à l’utilisation, et selon la série, par exemple dv2, F et G. Par exemple, la valeur par défaut pour les abonnements Accord Entreprise est 350.

<sup>2</sup> Avec Azure Resource Manager, les certificats sont stockés dans Azure Key Vault. Le nombre de certificats est illimité pour un abonnement. Un déploiement est limité à 1 Mo de certificat, dont une machine virtuelle ou un groupe à haute disponibilité.

> [!NOTE]
> Les cœurs de machines virtuelles sont soumis à une limite totale régionale. Ils ont également une limite pour les séries par taille régionales, telles que Dv2 et F. Ces limites sont appliquées séparément. Par exemple, considérons un abonnement dont le nombre total limite de cœurs de machine virtuelle est de 30 pour la région USA Est, de 30 pour la gamme A et de 30 pour la gamme D. Cet abonnement peut déployer 30 machines virtuelles A1, ou 30 machines virtuelles D1, ou encore une combinaison de ces deux types de machines dans la limite de 30 cœurs au total. Par exemple, 10 machines virtuelles A1 et 20 machines virtuelles D1.  
> <!-- -->
>
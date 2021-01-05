---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 08/03/2020
ms.author: alkohli
ms.openlocfilehash: 7ce49873b4e59bcf474deaea4420f56a72c9c589
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89081530"
---
La taille de machine virtuelle détermine la quantité de ressources de calcul, comme le processeur, le processeur graphique (GPU) et la mémoire, qui sont mises à la disposition de la machine virtuelle. Les machines virtuelles doivent être créées avec une taille adaptée à la charge de travail. Même si toutes les machines vont s’exécuter sur le même matériel, leur taille ont des limites différentes pour l’accès au disque, ce qui peut vous aider à gérer l’accès global au disque sur toutes vos machines virtuelles. Si une charge de travail augmente, une machine virtuelle existante peut également être redimensionnée.

La création des machines virtuelles de la série Standard Dv2 suivantes est prise en charge sur un appareil Azure Stack Edge.

### <a name="dv2-series"></a>Série Dv2
|Taille     |Processeurs virtuels     |Mémoire (Gio) | Stockage temporaire (Gio)  | Débit de disque du système d’exploitation max. (E/S par seconde) | Débit de stockage temporaire max. (E/S par seconde) | Disques de données max. / débit (E/S par seconde) | Nombre max de cartes réseau |
|-------------------|----|----|-----|----|------|------------|---------|
|**Standard_D1_v2** |1   |3,5 |50   |500 |3000  |4 / 4 x 500   |2 |
|**Standard_D2_v2** |2   |7   |100  |500 |6000  |8 / 8 x 500   |2 |
|**Standard_D3_v2** |4   |14  |200  |500 |12 000 |16 / 16 x 500 |4 |
|**Standard_D4_v2** |8   |28  |400  |500 |24 000 |32 / 32 x 500 |8 |
|**Standard_D5_v2** |16  |56  |800  |500 |48 000 |64 / 64 x 500 |8 |

### <a name="dsv2-series"></a>Séries DSv2
|Taille     |Processeurs virtuels     |Mémoire (Gio) | Stockage temporaire (Gio)  | Débit de disque du système d’exploitation max. (E/S par seconde) | Débit de stockage temporaire max. (E/S par seconde) | Disques de données max. / débit (E/S par seconde) | Nombre max de cartes réseau |
|--------------------|----|----|----|-----|------|-------------|---------|
|**Standard_DS1_v2** |1   |3,5 |7   |1 000 |4000  |4 / 4x2 300   |2 |
|**Standard_DS2_v2** |2   |7   |14  |1 000 |8000  |8 / 8x2 300   |2 |
|**Standard_DS3_v2** |4   |14  |28  |1 000 |16000 |16 / 16x2 300 |4 |
|**Standard_DS4_v2** |8   |28  |56  |1 000 |32000 |32 / 32x2 300 |8 |
|**Standard_DS5_v2** |16  |56  |112 |1 000 |64 000 |64 / 64x2 300 |8 |

### <a name="dv2-series"></a>Série Dv2
|Taille     |Processeurs virtuels     |Mémoire (Gio) | Stockage temporaire (Gio)  | Débit de disque du système d’exploitation max. (E/S par seconde) | Débit de stockage temporaire max. (E/S par seconde) | Disques de données max. / débit (E/S par seconde) | Nombre max de cartes réseau |
|--------------------|----|----|-----|----|-------|-------------|---------|
|**Standard_D11_v2** |2   |14  |100  |500 |6000   |8 / 8 x 500    |2 |
|**Standard_D12_v2** |4   |28  |200  |500 |12 000  |16 / 16 x 500  |4 |
|**Standard_D13_v2** |8   |56  |400  |500 |24 000  |32 / 32 x 500  |8 |
|**Standard_D14_v2** |16  |112 |800  |500 |48 000  |64 / 64 x 500  |8 |


### <a name="dsv2-series"></a>Séries DSv2
|Taille     |Processeurs virtuels     |Mémoire (Gio) | Stockage temporaire (Gio)  | Débit de disque du système d’exploitation max. (E/S par seconde) | Débit de stockage temporaire max. (E/S par seconde) | Disques de données max. / débit (E/S par seconde) | Nombre max de cartes réseau |
|---------------------|----|----|-----|-----|-------|--------------|---------|
|**Standard_DS11_v2** |2   |14  |28   |1 000 |8000   |4 / 4x2 300    |2 |
|**Standard_DS12_v2** |4   |28  |56   |1 000 |16000  |8 / 8x2 300    |4 |
|**Standard_DS13_v2** |8   |56  |112  |1 000 |32000  |16 / 16x2 300  |8 |
|**Standard_DS14_v2** |16  |112 |224  |1 000 |64 000  |32 / 32x2 300  |8 |

Pour plus d’informations, consultez [Série Dv2 sur des tailles de machine virtuelle à usage général](../articles/virtual-machines/dv2-dsv2-series.md#dv2-series).

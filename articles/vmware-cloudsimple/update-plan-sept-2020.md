---
title: Azure VMware Solution by CloudSimple – Mise à jour de septembre 2020
description: Dans cet article, découvrez ce qui se passe au cours de cette opération de maintenance, ainsi que les modifications apportées à votre cloud privé.
author: dikamath
ms.author: dikamath
ms.date: 09/3/2020
ms.topic: article
ms.service: azure-vmware-cloudsimple
manager: dikamath
ms.openlocfilehash: ae9c1ba5259e95ed030d7099e5dafe2d4f7935b4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89447628"
---
# <a name="azure-vmware-solution-by-cloudsimple-september-2020-update"></a>Azure VMware Solution by CloudSimple – Mise à jour de septembre 2020

Une mise à jour importante du service Azure VMware Solution sera effectuée en septembre. Un e-mail de notification, envoyé dans le cadre de la maintenance, inclura la chronologie de la maintenance. Cet article fournit des informations sur ce qui se passe au cours de cette opération de maintenance, ainsi que sur les modifications apportées à votre cloud privé.

> [!NOTE]
> Il s’agit d’une mise à niveau sans interruption de service. Au cours de cette mise à niveau, vous pouvez voir l’un des composants redondants devenir indisponible.

## <a name="vmware-infrastructure-upgrade"></a>Mise à niveau de l’infrastructure VMware

L’infrastructure VMware de votre cloud privé sera mise à jour vers une version plus récente. Cela comprend des mises à jour vers les composants vCenter, ESXi, NSX-T et HCX (Hybrid Cloud Extension) (s’ils sont déployés) de votre cloud privé.

Pendant la mise à niveau, un nouveau nœud est ajouté à votre cloud privé avant qu’un nœud soit placé en mode de maintenance pour l’opération de mise à niveau. Cela garantit la conservation de la capacité de votre cloud privé et de sa disponibilité pendant le processus de mise à niveau. Pendant la mise à niveau des composants VMware, vous pouvez voir des alarmes affichées dans votre interface utilisateur web vCenter. Les alarmes font partie des opérations de maintenance effectuées par l’équipe des opérations de service.

**Versions des composants**

- ESXi 6.7U3
- vCenter 6.7U3
- vSAN 6.7
- Centre de données NSX 2.5.1
- HCX 3.5.2

## <a name="datacenter-updates"></a>Mises à jour des centres de données

Cette mise à jour comprend des mises à jour de l’infrastructure des centres de données. Les mises à jour sans interruption de service seront effectuées au cours de la période de maintenance. Vous remarquerez une redondance réduite pendant le processus de mise à jour. Des alertes peuvent être générées dans votre infrastructure VMware de cloud privé, vos circuits ExpressRoute, vos connexions GlobalReach et tous les périphériques VPN de site à site associés à l’une des disponibilités de lien. Cela est normal pendant la mise à jour, car les composants sont redémarrés à cette occasion.

-   Si un réseau VPN de site à site est déployé en tant qu’instance unique (sans haute disponibilité), vous devrez peut-être rétablir la connexion VPN.

-   Si vous utilisez une connexion VPN de point à site, vous devrez rétablir la connexion VPN.

## <a name="post-update"></a>Après la mise à jour

Une fois les mises à jour terminées, vous devez voir les nouvelles versions des composants VMware. Si vous rencontrez des problèmes ou si vous avez des questions, contactez notre équipe de support technique en ouvrant un [ticket de support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

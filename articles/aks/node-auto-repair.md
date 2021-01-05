---
title: Réparation automatique des nœuds Azure Kubernetes Service
description: Découvrez la fonctionnalité de réparation automatique des nœuds et la façon dont AKS corrige les nœuds Worker endommagés.
services: container-service
ms.topic: conceptual
ms.date: 08/24/2020
ms.openlocfilehash: 781a1ffebb40b0cce9f18699d308db90633e8626
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89490103"
---
# <a name="azure-kubernetes-service-aks-node-auto-repair"></a>Réparation automatique des nœuds AKS

AKS vérifie en permanence l’état d’intégrité des nœuds Worker et effectue une réparation automatique des nœuds s’ils ne sont pas sains. Ce document informe les opérateurs quant au comportement de la fonctionnalité de réparation automatique de nœud pour les nœuds Windows et Linux. En plus des réparations AKS, la plateforme de machine virtuelle Azure [effectue une maintenance sur les machines virtuelles][vm-updates] qui rencontrent des problèmes. AKS et les machines virtuelles Azure opèrent ensemble pour réduire les interruptions de service pour vos clusters.

## <a name="how-aks-checks-for-unhealthy-nodes"></a>Comment AKS vérifie les nœuds non sains

AKS utilise des règles pour déterminer si un nœud est dans un état non sain et nécessite une réparation. AKS utilise les règles suivantes pour déterminer si une réparation automatique est nécessaire.

* Le nœud signale l’état de **NotReady** lors de vérifications consécutives sur une période de 10 minutes
* Le nœud ne signale pas d’état pendant 10 minutes

Vous pouvez vérifier manuellement l’état d’intégrité de vos nœuds avec kubectl.

```
kubectl get nodes
```

## <a name="how-automatic-repair-works"></a>Fonctionnement de la réparation automatique

> [!Note]
> AKS lance des opérations de réparation avec le compte d’utilisateur **aks-remediator**.

Si un nœud est défectueux conformément aux règles ci-dessus, et reste défectueux pendant 10 minutes consécutives, les actions suivantes sont effectuées.

1. Redémarrage du nœud
1. Si le redémarrage échoue, réinitialisation du nœud
1. Si la réinitialisation échoue, création et réinitialisation d’un nouveau nœud

Si aucune des actions ne réussit, des corrections supplémentaires sont étudiées par les ingénieurs AKS. Si plusieurs nœuds sont en état non sain pendant un contrôle d’intégrité, chaque nœud est réparé avant le début de la réparation d’un autre.

## <a name="next-steps"></a>Étapes suivantes

Utilisez [Zones de disponibilité][availability-zones] pour augmenter la haute disponibilité avec les charges de travail de votre cluster AKS.

<!-- LINKS - External -->

<!-- LINKS - Internal -->
[availability-zones]: ./availability-zones.md
[vm-updates]: ../virtual-machines/maintenance-and-updates.md

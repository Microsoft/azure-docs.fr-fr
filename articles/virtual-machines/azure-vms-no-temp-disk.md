---
title: FAQ sur les tailles de machine virtuelle Azure sans disque temporaire local
description: Cet article fournit des réponses aux questions fréquentes (FAQ) sur les tailles de machine virtuelle Microsoft Azure qui n’ont pas de disque temporaire local.
author: brbell
ms.service: virtual-machines
ms.topic: conceptual
ms.author: brbell
ms.reviewer: mimckitt
ms.date: 06/15/2020
ms.openlocfilehash: 30587fac7d7be37d7595a78502b7999adee9a30f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91665308"
---
# <a name="azure-vm-sizes-with-no-local-temporary-disk"></a>Tailles de machine virtuelle Azure sans disque temporaire local 
Cet article fournit des réponses aux questions fréquentes (FAQ) sur les tailles de machine virtuelle Azure qui n’ont pas de disque temporaire local. Pour plus d’informations sur ces tailles de machine virtuelle, consultez les [spécifications des séries Dv4 et  Dsv4 (charges de travail universelles)](dv4-dsv4-series.md) ou les [spécifications des séries Ev4 et Esv4 (charges de travail à mémoire optimisée)](ev4-esv4-series.md).

## <a name="what-does-no-local-temp-disk-mean"></a>Que signifie l’absence de disque temporaire local ? 
Traditionnellement, nous avons des tailles de machine virtuelle (par exemple Standard_D2s_v3, Standard_E48_v3) qui incluent un petit disque local (c’est-à-dire un lecteur D: ). Désormais, avec ces nouvelles tailles de machine virtuelle, ce petit disque local n’existe plus. Toutefois, vous pouvez toujours joindre un disque HDD Standard, SSD Premium ou SSD Ultra.

## <a name="what-if-i-still-want-a-local-temp-disk"></a>Que se passe-t-il si je souhaite conserver un disque temporaire local ?
Si votre charge de travail nécessite un disque temporaire local, de nouvelles tailles de machine virtuelle [Ddv4 et Ddsv4](ddv4-ddsv4-series.md) ou [Edv4 et Edsv4](edv4-edsv4-series.md) sont également disponibles. Ces tailles offrent un disque temporaire dont la taille est supérieure de 50 % à celle des tailles v3 précédentes.

> [!NOTE]
> Le disque local temporaire n’est pas persistant ; pour garantir la persistance de vos données, utilisez les options HDD Standard, SSD Premium ou SSD Ultra. 

## <a name="what-are-the-differences-between-these-new-vm-sizes-and-the-general-purpose-dv3dsv3-or-the-memory-optimized-ev3esv3-vm-sizes-that-i-am-used-to"></a>Quelles sont les différences entre ces nouvelles tailles de machine virtuelle et les tailles de machine virtuelle Dv3/Dsv3 universelles ou Ev3/Esv3 à mémoire optimisée que j’utilise ? 
| Familles de machines virtuelles v3 avec disque temporaire local   | Nouvelles familles v4 avec disque temporaire local | Nouvelles familles v4 sans disque temporaire local |
|---|---|---|
| Dv3   | Ddv4 | Dv4 |
| Dsv3 | Ddsv4  | Dsv4 |
| Ev3   | Edv4  | Ev4 |
| Esv3 | Edsv4 |    Esv4 | 

## <a name="can-i-resize-a-vm-size-that-has-a-local-temp-disk-to-a-vm-size-with-no-local-temp-disk"></a>Puis-je redimensionner une taille de machine virtuelle ayant un disque temporaire local en une taille de machine virtuelle sans disque temporaire local ?  
Non. Les seules combinaisons autorisées pour le redimensionnement sont les suivantes : 

1. Machine virtuelle (avec disque temporaire local) -> machine virtuelle (avec disque temporaire local) ; et 
2. Machine virtuelle (sans disque temporaire local) -> machine virtuelle (sans disque temporaire local). 

> [!NOTE]
> Si une image dépend du disque de ressources, ou s’il existe un fichier d’échange sur le disque temporaire local, les images sans disque ne fonctionneront pas. Utilisez plutôt l’alternative « avec disque ». 

## <a name="do-these-vm-sizes-support-both-linux-and-windows-operating-systems-os"></a>Ces tailles de machine virtuelle prennent-elles en charge les systèmes d’exploitation Linux et Windows ?
Oui.

## <a name="will-this-break-my-custom-scripts-custom-images-or-os-images-that-have-scratch-files-or-page-files-on-a-local-temp-disk"></a>Cela va-t-il rompre mes scripts personnalisés, images personnalisées ou images de système d’exploitation qui contiennent des fichiers Scratch ou des fichiers d’échange sur un disque temporaire local ?
Si l’image du système d’exploitation personnalisée pointe vers le disque local temporaire, l’image risque de ne pas fonctionner correctement avec cette taille sans disque.

## <a name="have-questions-or-feedback"></a>Avez-vous des questions ou des commentaires ?
Remplissez le [formulaire de commentaires]( https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR_Y3toRKxchLjARedqtguBRUMzdCQkw0OVVRTldFUUtXSTlLQVBPUkVHSy4u). 

## <a name="next-steps"></a>Étapes suivantes 
Ce document vous a fourni les réponses aux questions les plus courantes concernant les machines virtuelles Azure sans disque temporaire local. Pour plus d’informations sur ces tailles de machine virtuelle, consultez les articles suivants :

- [Spécifications des séries Dv4 et Dsv4 (charges de travail universelles)](dv4-dsv4-series.md)
- [Spécifications des séries Ev4 et Esv4 (charges de travail à mémoire optimisée)](ev4-esv4-series.md)

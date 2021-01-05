---
author: cynthn
ms.service: virtual-machines-linux
ms.topic: include
ms.date: 10/26/2018
ms.author: cynthn
ms.openlocfilehash: 09fa612e7e5c681da16bf19e94c626ae14a3b8a1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77590800"
---
Lorsque vous ajoutez des disques de données à une machine virtuelle Linux, vous pouvez rencontrer des erreurs si un disque n’existe pas sur le LUN 0. Si vous ajoutez un disque manuellement à l’aide de la commande `az vm disk attach -new` et que vous spécifiez un LUN (`--lun`) au lieu de laisser la plateforme Azure déterminer le LUN approprié, vérifiez qu’un disque existe déjà ou existera sur le LUN 0. 

Prenons l’exemple suivant montrant un extrait de code de sortie de `lsscsi`:

```bash
[5:0:0:0]    disk    Msft     Virtual Disk     1.0   /dev/sdc 
[5:0:0:1]    disk    Msft     Virtual Disk     1.0   /dev/sdd 
```

Les deux disques de données existent sur le LUN 0 et sur le LUN 1 (la première colonne dans les détails de sortie `lsscsi``[host:channel:target:lun]`). Les deux disques doivent être accessibles à partir de la machine virtuelle. Si vous aviez manuellement spécifié le premier disque à ajouter au LUN 1 et le deuxième disque au LUN 2, les disques peuvent ne pas s’afficher correctement à partir de votre machine virtuelle.

> [!NOTE]
> La valeur Azure `host` est égale à 5 dans ces exemples, mais elle peut varier selon le type de stockage que vous sélectionnez.
> 
> 

Ce comportement de disque n’est pas dû à un problème d’Azure, mais dépend de la façon dont le noyau Linux suit les spécifications SCSI. Lorsque le noyau Linux analyse le bus SCSI à la recherche des périphériques connectés, un périphérique doit être présent sur le LUN 0 pour que le système puisse continuer l’analyse des autres périphériques. Par conséquent :

* Examinez la sortie de `lsscsi` après l’ajout d’un disque de données pour vérifier que vous disposez d’un disque sur le LUN 0.
* Si le disque ne s’affiche pas correctement dans votre machine virtuelle, vérifiez qu'un disque existe sur le LUN 0.


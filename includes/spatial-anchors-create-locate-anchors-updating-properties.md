---
ms.openlocfilehash: 8ebb10f955be8f3004fdbdc595ea0fefc0d2b7ea
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "67176776"
---
## <a name="update-properties"></a>Mettre à jour des propriétés

Pour mettre à jour les propriétés d’une ancre, vous utilisez la méthode `UpdateAnchorProperties()`. Si deux ou plusieurs appareils essaient de mettre à jour simultanément des propriétés pour la même ancre, nous utilisons un modèle d’accès concurrentiel optimiste. Ce qui signifie que la première écriture gagne.  Toutes les autres écritures obtiendront une erreur « Accès concurrentiel » indiquant qu’une mise à jour des propriétés est nécessaire avant de réessayer.

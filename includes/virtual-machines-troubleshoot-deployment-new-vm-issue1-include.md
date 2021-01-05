---
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 10/26/2018
ms.author: cynthn
ms.openlocfilehash: 2ae72045ae18d84eac2a6d619d94e3a9e49415ae
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "67177061"
---
## <a name="issue-custom-image-provisioning-errors"></a>Problème : Image personnalisée ; erreurs de configuration
Les erreurs de configuration se produisent lorsque vous téléchargez ou capturez une image de machine virtuelle généralisée en tant qu’image de machine virtuelle spécialisée, ou inversement. Une erreur de délai d’attente de configuration survient dans le premier cas, tandis que c’est un échec de configuration dans le second. Pour déployer votre image personnalisée sans erreur, vous devez vous assurer que le type de l’image ne change pas pendant le processus de capture.

Le tableau suivant répertorie les combinaisons possibles d’images généralisées et spécialisés, le type d’erreur que vous pouvez rencontrer et ce qu’il faut faire pour corriger ces erreurs.


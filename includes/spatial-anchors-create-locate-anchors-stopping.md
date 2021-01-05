---
ms.openlocfilehash: a165061b2f927d6302504819ef27c846d76d0f30
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83006410"
---
## <a name="pause-reset-or-stop-the-session"></a>Suspendre, réinitialiser ou arrêter la session

Pour arrêter la session temporairement, vous pouvez appeler `Stop()`. Ainsi, le traitement de tous les observateurs et environnements s’arrête, même si vous appelez `ProcessFrame()`. Vous pouvez ensuite appeler `Start()` pour reprendre le traitement. Lors de la reprise, les données d’environnement déjà capturées dans la session sont conservées.

---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 06/11/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 05dbde2f8fc7b348a52d139a835d06f5a8b084e4
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/11/2021
ms.locfileid: "112040566"
---
|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[La sauvegarde géoredondante doit être activée pour Azure Database pour PostgreSQL](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F48af4db5-9b8b-401c-8e74-076be876a430) |Azure Database pour PostgreSQL vous permet de choisir l’option de redondance pour votre serveur de base de données. Vous pouvez choisir un stockage de sauvegarde géoredondant. Dans ce cas, les données sont non seulement stockées dans la région qui héberge votre serveur, mais elles sont aussi répliquées dans une région jumelée pour offrir une possibilité de récupération en cas de défaillance régionale. La configuration du stockage géoredondant pour la sauvegarde est uniquement possible lors de la création du serveur. |Audit, Désactivé |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/GeoRedundant_DBForPostgreSQL_Audit.json) |

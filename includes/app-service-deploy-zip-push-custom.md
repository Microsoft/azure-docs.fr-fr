---
author: cephalin
ms.service: app-service
ms.topic: include
ms.date: 11/03/2016
ms.author: cephalin
ms.openlocfilehash: 0019e50615f3e66778709ad8cb28f92967c66e2e
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96018424"
---
## <a name="deployment-customization"></a>Personnalisation du déploiement

Le processus de déploiement suppose que le fichier .zip que vous envoyez (push) contienne une application prête à l’exécution. Par défaut, aucune personnalisation n’est exécutée. Pour activer les même processus de génération que ceux obtenus avec l’intégration continue, ajoutez ceci aux paramètres de votre application :

`SCM_DO_BUILD_DURING_DEPLOYMENT=true`

Quand vous utilisez le déploiement par envoi (push) de fichier .zip, ce paramètre est **false** par défaut. La valeur par défaut est **true** pour les déploiements par intégration continue. Quand la valeur est **true**, vos paramètres de déploiement sont utilisés pendant le déploiement. Vous pouvez configurer ces paramètres comme paramètres de l’application ou dans un fichier de configuration .deployment qui se trouve dans la racine de votre fichier .zip. Pour plus d’informations, consultez [Repository and deployment-related settings](https://github.com/projectkudu/kudu/wiki/Configurable-settings#repository-and-deployment-related-settings) dans les informations de référence sur le déploiement.
---
author: msftradford
ms.service: azure-spatial-anchors
ms.topic: include
ms.date: 11/20/2020
ms.author: parkerra
ms.openlocfilehash: 7210b05566f5cd6f3c56792bce0904b3c9b645da
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95993044"
---
## <a name="set-up-authentication"></a>Configurer l’authentification

Pour accéder au service, vous devez fournir une clé de compte, un jeton d’accès ou un jeton d’authentification Azure Active Directory. Pour plus d’informations, consultez la page sur les [concepts relatifs à l’authentification](../articles/spatial-anchors/concepts/authentication.md).

### <a name="account-keys"></a>Clés de compte

Les clés de compte sont une information d’identification qui permet à votre application de s’authentifier auprès du service Azure Spatial Anchors. La finalité des clés de compte est de vous aider à démarrer rapidement, notamment lors de la phase de développement de l’intégration de votre application avec Azure Spatial Anchors. Ainsi, vous pouvez utiliser des clés de compte en les incorporant dans vos applications clientes lors du développement. Une fois le développement terminé, nous vous recommandons vivement de passer à une mécanisme d’authentification appliqué au niveau de la production et pris en charge par les jetons d’accès ou l’authentification utilisateur de Microsoft Azure Active Directory. Pour obtenir une clé de compte pour le développement, consultez votre compte Azure Spatial Anchors et accédez à l’onglet relatif aux clés.
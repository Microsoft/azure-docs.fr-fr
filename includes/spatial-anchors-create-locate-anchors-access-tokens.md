---
author: msftradford
ms.service: azure-spatial-anchors
ms.topic: include
ms.date: 11/20/2020
ms.author: parkerra
ms.openlocfilehash: c9b4e0fdf95c2bb4888a3a11820f1785d8545bd4
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95993039"
---
### <a name="access-tokens"></a>Jetons d’accès

Les jetons d’accès sont une méthode plus robuste pour vous authentifier à Azure Spatial Anchors. Plus particulièrement lorsque vous préparez votre application à un déploiement de production. En résumé, cette approche consiste à configurer un service back-end auquel votre application cliente peut s’authentifier en toute sécurité. Vos interfaces de service back-end avec AAD lors de l’exécution et le service de jeton de sécurité Azure Spatial Anchors demande un jeton d’accès. Ce jeton est ensuite remis à l’application cliente et utilisé dans le Kit de développement logiciel pour s’authentifier auprès Azure Spatial Anchors.

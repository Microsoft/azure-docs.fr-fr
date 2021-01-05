---
author: msftradford
ms.author: parkerra
ms.date: 11/20/2020
ms.service: azure-spatial-anchors
ms.topic: include
ms.openlocfilehash: 41c0a6292bf7369a30fbb1e4bd474e70ffaedcb6
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95999736"
---
## <a name="configure-the-cloud-spatial-anchor-session"></a>Configurer la session d’ancre spatiale cloud

Nous nous occuperons ensuite de la configuration de la session d’ancre spatiale cloud. Sur la première ligne, nous définissons le fournisseur de lecteur de la session. À partir de maintenant, toutes les ancres créées pendant la session seront associées à un ensemble de relevés de lecteur. Ensuite, nous instancions des critères de proximité de l’appareil et nous les initialisons en fonction des spécifications de l’application. Enfin, nous indiquons à la session d’utiliser les données du lecteur lors de la localisation des ancres en créant un observateur à partir de nos critères de proximité de l’appareil.
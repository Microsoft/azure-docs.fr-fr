---
author: mikeparker104
ms.author: miparker
ms.date: 06/02/2020
ms.service: notification-hubs
ms.topic: include
ms.openlocfilehash: 5c3277394350036b8863185e83b7cfcd4975be1c
ms.sourcegitcommit: 971a3a63cf7da95f19808964ea9a2ccb60990f64
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/19/2020
ms.locfileid: "85095285"
---
Elle vous permet de vous inscrire auprès d’un hub de notification et de vous en désinscrire par le biais du service back-end que vous avez créé. 

Une alerte s’affiche quand une action est spécifiée et que l’application est au premier plan. Sinon, les notifications s’affichent dans le centre de notifications.

> [!NOTE]
> En général, vous effectuez les actions d’inscription (et de désinscription) au moment approprié durant le cycle de vie de l’application (ou éventuellement dans le cadre de votre première expérience d’exécution) sans entrées d’inscription ou de désinscription explicites de la part de l’utilisateur. Toutefois, cet exemple demande une entrée utilisateur explicite pour faciliter l’exploration et le test de cette fonctionnalité.

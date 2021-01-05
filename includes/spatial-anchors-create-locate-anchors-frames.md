---
ms.openlocfilehash: 849777ac3ec11915c602861fea0c42a49fb08bcb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83006425"
---
## <a name="provide-frames-to-the-session"></a>Fournir des images à la session

La session d’ancrage spatial fonctionne en mappant l’espace autour de l’utilisateur. Cela aide à déterminer l’emplacement des points d’ancrage. Les plateformes mobiles (iOS et Android) nécessitent un appel natif à l’appareil photo pour obtenir des images à partir de la bibliothèque AR de votre plateforme. En revanche, HoloLens analyse constamment l’environnement. Il n’est donc pas nécessaire d’effectuer un appel, comme avec les plateformes mobiles.
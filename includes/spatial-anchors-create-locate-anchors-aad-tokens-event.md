---
ms.openlocfilehash: 7219a457a2631f9ff6beee06eff34bce0ff5a23f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "67176765"
---
Comme pour les jetons d’accès, si un jeton Azure AD n’est pas défini, vous devez gérer l’événement TokenRequired ou implémenter la méthode tokenRequired sur le protocole délégué.

Vous pouvez gérer l’événement de façon synchrone en définissant la propriété sur les arguments d’événement.

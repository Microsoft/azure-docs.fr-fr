---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 10/15/2020
ms.author: alkohli
ms.openlocfilehash: 160f85b34b3730b14ed96854d5f60ad81f4eb63b
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96580973"
---
Votre appareil est associé à un compte de stockage qui est utilisé en tant que destination pour vos données dans Azure. L’accès au compte de stockage est contrôlé par l’abonnement et deux clés d’accès au stockage de 512 bits associées à ce compte de stockage.

L’une des clés est utilisée pour l’authentification quand l’appareil Azure Stack Edge accède au compte de stockage. L’autre clé est gardée en réserve, ce qui permet une rotation régulière des clés.

De nombreux centres de données ont recours à la rotation des clés pour des raisons de sécurité. Nous vous recommandons de suivre ces méthodes recommandées pour la rotation des clés :

- Votre clé de compte de stockage est similaire au mot de passe racine pour votre compte de stockage. Protégez soigneusement votre clé de compte. Ne communiquez pas le mot de passe à d’autres utilisateurs, ne le codez pas en dur et ne l’enregistrez pas en texte brut, où que ce soit.
- Régénérez votre clé de compte via le Portail Azure si vous pensez qu’elle a été compromise. Pour plus d’informations, consultez [Gérer les clés d’accès au compte de stockage](../articles/storage/common/storage-account-keys-manage.md).
- Votre administrateur Azure doit changer ou régénérer régulièrement la clé primaire ou secondaire à l’aide de la section Stockage du portail Azure pour accéder directement au compte de stockage.
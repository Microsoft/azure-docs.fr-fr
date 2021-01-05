---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 05/06/2020
ms.author: glenga
ms.openlocfilehash: 0bc5977a581006dc760c0435f9d68371ced7e4cd
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83648783"
---
Le stockage Azure chiffre toutes les données dans un compte de stockage au repos. Pour plus d'informations, consultez [Fonctionnalité de chiffrement du service Stockage Azure pour les données au repos](../articles/storage/common/storage-service-encryption.md).

Par défaut, les données sont chiffrées avec des clés managées par Microsoft Pour plus de contrôle sur les clés de chiffrement, vous pouvez fournir des clés gérées par le client à utiliser pour le chiffrement des données de fichiers et d’objets blob. Ces clés doivent être présentes dans Azure Key Vault pour que Functions puisse accéder au compte de stockage. Pour en savoir plus, consultez [Chiffrement au repos à l’aide de clés gérées par le client](../articles/azure-functions/configure-encrypt-at-rest-using-cmk.md).  
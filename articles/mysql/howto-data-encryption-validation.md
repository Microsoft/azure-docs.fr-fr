---
title: Guide pratique pour garantir la validation du chiffrement des données Azure Database pour MySQL
description: Découvrez comment valider le chiffrement des données Azure Database pour MySQL à l’aide de la clé gérée par le client.
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: how-to
ms.date: 04/28/2020
ms.openlocfilehash: 1f47b3d5679b70461eaba64b5815770162cb6fda
ms.sourcegitcommit: 80034a1819072f45c1772940953fef06d92fefc8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93241070"
---
# <a name="validating-data-encryption-for-azure-database-for-mysql"></a>Validation du chiffrement des données pour Azure Database pour MySQL

Cet article vous permet de vérifier que le chiffrement des données à l’aide de la clé gérée par le client pour Azure Database pour MySQL fonctionne comme prévu.

## <a name="check-the-encryption-status"></a>Vérifier l’état du chiffrement

### <a name="from-portal"></a>À partir du portail

1. Si vous souhaitez vérifier que la clé du client est utilisée pour le chiffrement, procédez comme suit :

    * Dans le portail Azure, accédez à **Azure Key Vault** -> **Clés**.
    * Sélectionnez la clé utilisée pour le chiffrement du serveur.
    * Définissez l’état de la clé **Activé** sur **Non**.
  
       Après un certain temps ( **environ 15 minutes** ), l’ **État** du serveur Azure Database pour MySQL doit être **Inaccessible**. Toutes les opérations d’E/S effectuées sur le serveur échouent, ce qui confirme que le serveur est effectivement chiffré avec la clé du client et que la clé n’est pas valide.
    
       Pour que le serveur soit **Disponible** , vous pouvez revalider la clé. 
    
    * Définissez l’état de la clé dans le coffre de clés sur **Oui**.
    * Dans la section **Chiffrement des données** du serveur, sélectionnez **Revalider la clé**.
    * Une fois la clé revalidée, l’ **État** du serveur passe à **Disponible**.

2. Sur le portail Azure, si vous pouvez vous assurer que la clé de chiffrement est définie, les données sont chiffrées à l’aide de la clé du client utilisée dans le portail Azure.

  :::image type="content" source="media/concepts-data-access-and-security-data-encryption/byok-validate.png" alt-text="Vue d’ensemble de la stratégie d’accès":::

### <a name="from-cli"></a>À partir de l’interface CLI

1. Nous pouvons utiliser la commande *az CLI* afin de valider les ressources clés utilisées pour le serveur Azure Database pour MySQL.

    ```azurecli-interactive
   az mysql server key list --name  '<server_name>'  -g '<resource_group_name>'
    ```

    Pour un serveur sans chiffrement des données, cette commande génère un ensemble vide [].

### <a name="azure-audit-reports"></a>Rapports d’audit Azure

Vous pouvez également consulter les [rapports d’audit](https://servicetrust.microsoft.com) qui fournissent des informations sur la conformité aux normes de protection des données et à la réglementation.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur le chiffrement des données, consultez [Chiffrement des données Azure Database pour MySQL avec une clé gérée par le client](concepts-data-encryption-mysql.md).
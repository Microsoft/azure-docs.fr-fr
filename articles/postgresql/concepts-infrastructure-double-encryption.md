---
title: Chiffrement double d’infrastructure - Azure Database pour PostgreSQL
description: Découvrez comment utiliser le chiffrement double d’infrastructure pour ajouter une deuxième couche de chiffrement avec des clés gérées par le service.
author: mksuni
ms.author: sumuth
ms.service: postgresql
ms.topic: conceptual
ms.date: 6/30/2020
ms.openlocfilehash: 03d52611c89012d1132b4a27820f7f47e6cdd512
ms.sourcegitcommit: 80034a1819072f45c1772940953fef06d92fefc8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93242022"
---
# <a name="azure-database-for-postgresql-infrastructure-double-encryption"></a>Azure Database pour PostgreSQL - Chiffrement double d’infrastructure

Azure Database pour PostgreSQL utilise le stockage [chiffrement des données au repos](concepts-security.md#at-rest) pour les données utilisant des clés gérées par Microsoft. Les données, y compris les sauvegardes, sont chiffrées sur le disque et ce chiffrement est toujours activé et ne peut pas être désactivé. Le chiffrement utilise le module de chiffrement validé FIPS 140-2 et un chiffrement AES 256 bits pour le chiffrement de stockage Azure.

Le chiffrement double d’infrastructure ajoute une deuxième couche de chiffrement avec des clés gérées par le service. Il utilise le module de cryptage validé FIPS 140-2, mais avec un algorithme de chiffrement différent. Cela fournit une couche supplémentaire de protection pour vos données au repos. La clé utilisée dans le chiffrement double d’infrastructure est également gérée par le service Azure Database pour PostgreSQL. Le chiffrement double d’infrastructure n’est pas activé par défaut, car la couche supplémentaire de chiffrement peut avoir un impact sur les performances.

> [!NOTE]
> Cette fonctionnalité est uniquement prise en charge pour les niveaux tarifaires « Usage général » et « À mémoire optimisée » dans Azure Database pour PostgreSQL.

Le chiffrement de couche d’infrastructure présente l’avantage d’être implémenté au niveau de la couche la plus proche du dispositif de stockage ou des câbles réseau. Azure Database pour PostgreSQL implémente les deux couches de chiffrement à l’aide de clés gérées par le service. Bien que toujours techniquement dans la couche de service, il est très proche du matériel qui stocke les données au repos. Vous pouvez également activer le chiffrement des données au repos à l’aide de [clé gérée par le client](concepts-data-encryption-postgresql.md) pour le serveur PostgreSQL approvisionné.  

L’implémentation aux couches d’infrastructure prend également en charge des clés différentes. L’infrastructure doit être consciente des différents clusters de machines et de réseaux. Par conséquent, différentes clés sont utilisées pour réduire le rayon d’explosion des attaques d’infrastructure et une grande variété de défaillances matérielles et réseau. 

> [!NOTE]
> L’utilisation du chiffrement double d’infrastructure aura un impact sur les performances sur le serveur Azure Database pour PostgreSQL en raison du processus de chiffrement supplémentaire.

## <a name="benefits"></a>Avantages

Le chiffrement double d’infrastructure pour Azure Database pour PostgreSQL offre les avantages suivants :

1. **Diversité supplémentaire de l’implémentation du chiffrement**  - Le passage planifié au chiffrement basé sur le matériel permet de diversifier davantage les implémentations en fournissant une implémentation basée sur le matériel en plus de celle basée sur le logiciel.
2. **Erreurs d’implémentation** - Deux couches de chiffrement au niveau de l’infrastructure protègent des erreurs de mise en cache ou de gestion de la mémoire dans des couches supérieures qui exposent des données en texte clair. En outre, les deux couches protègent également des erreurs dans l’implémentation du chiffrement en général.

Cette combinaison offre une protection renforcée contre les menaces courantes et les faiblesses exploitées pour attaquer le chiffrement.

## <a name="supported-scenarios-with-infrastructure-double-encryption"></a>Scénarios pris en charge avec le chiffrement double d’infrastructure

Les fonctionnalités de chiffrement fournies par Azure Database pour PostgreSQL peuvent être utilisées ensemble. Vous trouverez ci-dessous un résumé des différents scénarios que vous pouvez utiliser :

|  ##   | Chiffrement par défaut | Chiffrement double d’infrastructure | Chiffrement à l’aide de clés gérées par le client  |
|:------|:------------------:|:--------------------------------:|:--------------------------------------------:|
| 1     | *Oui*              | *Non*                             | *Non*                                         |
| 2     | *Oui*              | *Oui*                            | *Non*                                         |
| 3     | *Oui*              | *Non*                             | *Oui*                                        |
| 4     | *Oui*              | *Oui*                            | *Oui*                                        |
|       |                    |                                  |                                              |

> [!Important]
> - Les scénarios 2 et 4 auront un impact sur les performances du serveur Azure Database pour PostgreSQL en raison de la couche supplémentaire de chiffrement de l’infrastructure.
> - La configuration du double chiffrement de l’infrastructure pour Azure Database pour PostgreSQL est uniquement autorisée pendant la création du serveur. Une fois que le serveur est approvisionné, vous ne pouvez pas modifier le chiffrement de stockage. Toutefois, vous pouvez toujours activer le chiffrement des données à l’aide de clés gérées par le client pour le serveur créé avec/sans le chiffrement double de l’infrastructure.

## <a name="limitations"></a>Limites

Pour Azure Database pour PostgreSQL, la prise en charge du double chiffrement de l’infrastructure à l’aide de la clé gérée par le service présente les limitations suivantes :

* La prise en charge de cette fonctionnalité est limitée aux niveaux tarifaires **Usage général** et **À mémoire optimisée**.
* Vous pouvez créer un Azure Database pour PostgreSQL avec un double chiffrement d’infrastructure activé dans les régions suivantes :

   * USA Est
   * États-Unis - partie centrale méridionale
   * USA Ouest 2
   
* Cette fonctionnalité est uniquement prise en charge dans les régions et les serveurs qui prennent en charge jusqu’à 16 To de stockage. Pour obtenir la liste des régions Azure qui prennent en charge le stockage jusqu’à 16 To, reportez-vous à la [documentation du stockage](concepts-pricing-tiers.md#storage).

    > [!NOTE]
    > - Tous les **nouveaux** serveurs PostgreSQL créés dans les régions mentionnées ci-dessus prennent également en charge le chiffrement des données avec les clés CMK. Dans ce cas, les serveurs créés via la restauration à un point dans le temps (PITR) ou les réplicas en lecture ne sont pas qualifiés comme « nouveaux ».
    > - Pour confirmer que votre serveur approvisionné prend en charge jusqu’à 16 To, vous pouvez accéder au tableau de bord de niveau tarifaire dans le portail et voir si le curseur du stockage peut-être déplacé jusqu’à 16 To. Si vous ne pouvez déplacer le curseur que jusqu’à 4 to, il se peut que votre serveur ne prenne pas en charge le chiffrement avec les clés gérées par le client. Toutefois, les données sont chiffrées à l’aide de clés gérées par le service à tout moment. Posez vos questions en contactant AskAzureDBforPostgreSQL@service.microsoft.com.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment [configurer le double chiffrement de l’infrastructure pour Azure Database pour PostgreSQL](howto-double-encryption.md).

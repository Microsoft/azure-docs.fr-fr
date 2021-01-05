---
title: Comment identifier des adresses IP publiques sortantes dans Azure Spring Cloud
description: Comment afficher les adresses IP publiques sortantes statiques pour communiquer avec les ressources externes, telles que la base de données, le stockage, Key Vault, etc.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: how-to
ms.date: 09/17/2020
ms.custom: devx-track-java
ms.openlocfilehash: 04174b9cffb7e853dee235a4141ccda74a7847c6
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94663238"
---
# <a name="how-to-identify-outbound-public-ip-addresses-in-azure-spring-cloud"></a>Comment identifier des adresses IP publiques sortantes dans Azure Spring Cloud

Cette page explique comment afficher les adresses IP publiques statiques sortantes des applications Azure Spring Cloud.  Les adresses IP publiques sont utilisées pour communiquer avec des ressources externes, telles que des bases de données, le stockage et des coffres de clés.

## <a name="how-ip-addresses-work-in-azure-spring-cloud"></a>Fonctionnement des adresses IP dans Azure Spring Cloud

Un service Azure Spring Cloud possède une ou plusieurs adresses IP publiques sortantes. Le nombre d’adresses IP publiques sortantes peut varier en fonction des niveaux et d’autres facteurs. 

Les adresses IP publiques sortantes sont généralement constantes et restent identiques, mais il existe des exceptions.

## <a name="when-outbound-ips-change"></a>Lorsque les adresses IP sortantes changent

Chaque instance Azure Spring Cloud possède un nombre défini d’adresses IP publiques sortantes à tout moment. Toute connexion sortante depuis les applications, par exemple à une base de données back-end, utilise l’une des adresses IP publiques sortantes comme adresse IP d’origine. L’adresse IP est sélectionnée de façon aléatoire au moment de l’exécution. Par conséquent, votre service back-end doit ouvrir son pare-feu à toutes les adresses IP sortantes.

Le nombre d’adresses IP publiques sortantes change lorsque vous effectuez l’une des actions suivantes :

- Mise à jour de votre instance Azure Spring Cloud d’un niveau à l’autre.
- Envoi d’un ticket de support pour demander d’autres adresses IP publiques sortantes pour les besoins de l’entreprise.

## <a name="find-outbound-ips"></a>Trouver des adresses IP sortantes

Pour rechercher les adresses IP publiques sortantes actuellement utilisées par votre instance de service dans le portail Azure, cliquez sur **Mise en réseau** dans le volet de navigation de gauche de votre instance. Elles sont répertoriées dans le champ **Adresses IP sortantes**.

Vous pouvez obtenir les mêmes informations en exécutant la commande suivante dans Cloud Shell

```Azure CLI
az spring-cloud show --resource-group <group_name> --name <service_name> --query properties.networkProfile.outboundIPs.publicIPs --output tsv
```

## <a name="next-steps"></a>Étapes suivantes
> [!div class="nextstepaction"]
* [En savoir plus sur les identités managées pour les ressources Azure](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/active-directory/managed-identities-azure-resources/overview.md)
* [En savoir plus sur le coffre de clés dans Azure Spring Cloud](spring-cloud-tutorial-managed-identities-key-vault.md)

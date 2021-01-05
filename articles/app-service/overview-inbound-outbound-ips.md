---
title: Adresses IP entrantes/sortantes
description: Découvrez comment les adresses IP entrantes et sortantes sont utilisées dans Azure App Service, à quel moment elles changent et comment trouver les adresses pour votre application.
ms.topic: article
ms.date: 08/25/2020
ms.custom: seodec18, devx-track-azurecli
ms.openlocfilehash: e5b271cc5cd8cb52267b6ee44bc3965d0e4b0aab
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92746150"
---
# <a name="inbound-and-outbound-ip-addresses-in-azure-app-service"></a>Adresses IP entrantes et sortantes dans Azure App Service

[Azure App Service](overview.md) est un service mutualisé, à l’exception des [environnements App Service](environment/intro.md). Les applications qui ne sont pas dans un environnement App Service (pas dans le [niveau Isolé](https://azure.microsoft.com/pricing/details/app-service/)) partagent l’infrastructure réseau avec d’autres applications. Par conséquent, les adresses IP entrantes et sortantes d’une application peuvent être différentes et peuvent même changer dans certaines situations.

Les [environnements App Service](environment/intro.md) utilisent des infrastructures réseau dédiées, afin que les applications s’exécutant dans un environnement App Service obtiennent des adresses IP statiques dédiées pour les connexions entrantes et sortantes.

## <a name="how-ip-addresses-work-in-app-service"></a>Fonctionnement des adresses IP dans App Service

Une application App Service fonctionne dans un plan App Service, et les plans App Service sont déployés dans l’une des unités de déploiement de l’infrastructure Azure (appelée en interne un espace web). Chaque unité de déploiement se voit attribuer jusqu’à cinq adresses IP virtuelles, dont une adresse IP entrante publique et quatre adresses IP sortantes. Tous les plans App Service dans la même unité de déploiement, et les instances d’applications qui y sont exécutées, partagent le même ensemble d’adresses IP virtuelles. Pour un environnement App Service Environment (plan App Service de [niveau Isolé](https://azure.microsoft.com/pricing/details/app-service/)), le plan App Service est l’unité de déploiement proprement dite, de sorte que les adresses IP virtuelles lui sont dédiées en conséquence.

Comme vous n’êtes pas autorisé à déplacer un plan App Service entre des unités de déploiement, les adresses IP virtuelles attribuées à votre application restent généralement les mêmes, mais il existe des exceptions.

## <a name="when-inbound-ip-changes"></a>Lorsque l’adresse IP entrante change

Quel que soit le nombre d’instances scale-out, chaque application a une seule adresse IP entrante. L’adresse IP entrante peut changer lorsque vous effectuez l’une des actions suivantes :

- Supprimer une application, puis la recréer dans un autre groupe de ressources (l’unité de déploiement peut changer).
- Supprimer la dernière application dans une combinaison de groupe de ressources _et_ de région, puis la recréer (l’unité de déploiement peut changer).
- Supprimer une liaison TLS/SSL sur IP existante, comme pendant un renouvellement de certificat (voir [Renouveler un certificat](configure-ssl-certificate.md#renew-certificate)).

## <a name="find-the-inbound-ip"></a>Rechercher l’IP entrante

Il vous suffit d’exécuter la commande suivante dans un terminal local :

```bash
nslookup <app-name>.azurewebsites.net
```

## <a name="get-a-static-inbound-ip"></a>Obtenir une adresse IP entrante statique

Vous pouvez avoir besoin d’une adresse IP statique dédiée pour votre application. Pour obtenir une adresse IP entrante statique, vous devez [sécuriser un domaine personnalisé](configure-ssl-bindings.md#secure-a-custom-domain). Si vous n’avez pas besoin de la fonctionnalité TLS pour sécuriser votre application, vous pouvez même charger un certificat autosigné pour cette liaison. Dans une liaison TLS basée sur l’adresse IP, le certificat est lié à l’adresse IP elle-même. App Service fournit donc une adresse IP statique pour que cela se produise. 

## <a name="when-outbound-ips-change"></a>Lorsque les adresses IP sortantes changent

Quel que soit le nombre d’instances scale-out, chaque application a un nombre défini d’adresses IP sortantes à un moment donné. Toute connexion sortante depuis l’application App Service, par exemple la connexion à une base de données principale, utilise l’une des adresses IP sortantes comme adresse IP d’origine. L’adresse IP à utiliser est sélectionnée de façon aléatoire au moment de l’exécution. Par conséquent, votre service principal doit ouvrir son pare-feu à toutes les adresses IP sortantes de votre application.

L’ensemble des adresses IP sortantes de votre application change lorsque vous effectuez l’une des actions suivantes :

- Supprimer une application, puis la recréer dans un autre groupe de ressources (l’unité de déploiement peut changer).
- Supprimer la dernière application dans une combinaison de groupe de ressources _et_ de région, puis la recréer (l’unité de déploiement peut changer).
- Mettez à l’échelle votre application entre les niveaux inférieurs ( **De base** , **Standard** et **Premium** ) et le niveau **Premium v2** (les adresses IP peuvent être ajoutées ou soustraites de l’ensemble).

Vous pouvez trouver toutes les adresses IP sortantes que votre application est susceptible d’utiliser, indépendamment des niveaux de tarification, en recherchant la propriété `possibleOutboundIpAddresses`, ou à l’aide du champ **Adresses IP sortantes supplémentaires** du panneau **Propriétés** du portail Azure. Consultez [Trouver des adresses IP sortantes](#find-outbound-ips).

## <a name="find-outbound-ips"></a>Trouver des adresses IP sortantes

Pour trouver les adresses IP sortantes actuellement utilisées par votre application dans le portail Azure, cliquez sur **Propriétés** dans le volet de navigation gauche de votre application. Elles sont répertoriées dans le champ **Adresses IP sortantes** .

Vous pouvez obtenir les mêmes informations en exécutant la commande suivante dans [Cloud Shell](../cloud-shell/quickstart.md).

```azurecli-interactive
az webapp show --resource-group <group_name> --name <app_name> --query outboundIpAddresses --output tsv
```

```azurepowershell
(Get-AzWebApp -ResourceGroup <group_name> -name <app_name>).OutboundIpAddresses
```

Pour rechercher _toutes_ les adresses IP sortantes possibles pour votre application, indépendamment des niveaux de tarification, cliquez sur **Propriétés** dans la navigation à gauche de votre application. Elles sont répertoriées dans le champ **Adresses IP sortantes supplémentaires** .

Vous pouvez obtenir les mêmes informations en exécutant la commande suivante dans [Cloud Shell](../cloud-shell/quickstart.md).

```azurecli-interactive
az webapp show --resource-group <group_name> --name <app_name> --query possibleOutboundIpAddresses --output tsv
```

```azurepowershell
(Get-AzWebApp -ResourceGroup <group_name> -name <app_name>).PossibleOutboundIpAddresses
```

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment limiter le trafic entrant par adresses IP sources.

> [!div class="nextstepaction"]
> [Restrictions d’adresse IP statique](app-service-ip-restrictions.md)

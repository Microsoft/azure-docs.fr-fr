---
title: Mettre sur liste fiable les URL du portail Microsoft Azure sur votre pare-feu ou serveur proxy
description: Ajouter ces URL au contournement du serveur proxy pour communiquer avec le portail Microsoft Azure et ses services
ms.date: 04/10/2020
ms.topic: conceptual
ms.openlocfilehash: 7d9c8222ee85c0c16ec1e1926335ac06e0389797
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96745874"
---
# <a name="safelist-the-azure-portal-urls-on-your-firewall-or-proxy-server"></a>Mettre sur liste fiable les URL du portail Microsoft Azure sur votre pare-feu ou serveur proxy

Vous pouvez configurer des appareils de sécurité en local afin de contourner les restrictions de sécurité pour les URL du portail Microsoft Azure. Cette configuration permet d'améliorer les performances et la connectivité entre votre réseau local ou étendu et le cloud Azure.

Les administrateurs réseau déploient souvent des serveurs proxy, des pare-feu ou autres périphériques. Ces périphériques contribuent à sécuriser et à contrôler la manière dont les utilisateurs accèdent à Internet. Les règles conçues pour protéger les utilisateurs peuvent parfois bloquer ou ralentir le trafic Internet lié aux activités professionnelles. Ce trafic comprend les communications entre vous et Azure. Pour optimiser la connectivité entre votre réseau et le portail Microsoft Azure et ses services, nous vous recommandons de mettre sur liste fiable les URL du portail Microsoft Azure.

## <a name="azure-portal-urls-for-proxy-bypass"></a>URL du portail Microsoft Azure pour le contournement du proxy

Les points de terminaison d’URL à mettre sur liste fiable pour le portail Azure sont spécifiques au cloud Azure sur lequel votre organisation est déployée. Pour autoriser le trafic réseau vers ces points de terminaison à contourner les restrictions, sélectionnez votre cloud. Ajoutez ensuite la liste des URL à votre serveur proxy ou pare-feu.

#### <a name="public-cloud"></a>[Cloud public](#tab/public-cloud)

```
*.aadcdn.microsoftonline-p.com
*.aka.ms
*.applicationinsights.io
*.azure.com
*.azure.net
*.azurefd.net
*.azure-api.net
*.azuredatalakestore.net
*.azureedge.net
*.loganalytics.io
*.microsoft.com
*.microsoftonline.com
*.microsoftonline-p.com
*.msauth.net
*.msftauth.net
*.trafficmanager.net
*.visualstudio.com
*.windows.net
*.windows-int.net
```

#### <a name="us-government-cloud"></a>[U.S. U.S. Government](#tab/us-government-cloud)

```
*.applicationinsights.us
*.azure.us
*.loganalytics.us
*.microsoft.us
*.microsoftonline.us
*.msauth.net
*.usgovcloudapi.net
*.usgovtrafficmanager.net
*.windowsazure.us
```

#### <a name="china-government-cloud"></a>[Cloud China Government](#tab/china-government-cloud)

```
*.azure.cn
*.microsoft.cn
*.microsoftonline.cn
*.chinacloudapi.cn
*.trafficmanager.cn
*.chinacloudsites.cn
*.windowsazure.cn
```
---

> [!NOTE]
> Le trafic vers ces points de terminaison utilise les ports TCP standard pour HTTP (80) et HTTPS (443).
>
>

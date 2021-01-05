---
title: Passerelle de données locale pour Azure Analysis Services | Microsoft Docs
description: Une passerelle locale est nécessaire si votre serveur Analysis Services dans Azure se connecte à des sources de données locales.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 07/29/2020
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 6ca96f76287482a445d8a9a1cdc441333b36efbd
ms.sourcegitcommit: 44844a49afe8ed824a6812346f5bad8bc5455030
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/23/2020
ms.locfileid: "97739601"
---
# <a name="connecting-to-on-premises-data-sources-with-on-premises-data-gateway"></a>Connexion aux sources de données locales avec la passerelle de données locale

La passerelle de données locale assure un transfert de données sécurisé entre des sources de données locales et vos serveurs Azure Analysis Services dans le cloud. Pouvant être utilisée avec plusieurs serveurs Azure Analysis Services dans la même région, la dernière version de la passerelle fonctionne également avec Azure Logic Apps, Power BI, Power Apps et Power Automate. Bien que la passerelle que vous installez est la même pour tous ces services, Azure Analysis Services et Logic Apps présentent des étapes supplémentaires.

Les informations fournies ici sont spécifiques à la façon dont Azure Analysis Services fonctionne avec la passerelle de données locale. Pour plus d’informations sur la passerelle en général et sur son fonctionnement avec d’autres services, consultez [Qu’est-ce qu’une passerelle de données locale ?](/data-integration/gateway/service-gateway-onprem).

Pour Azure Analysis Services, la première configuration de la passerelle se déroule en quatre étapes :

- **Télécharger et exécuter le programme d’installation** - Cette étape installe un service de passerelle sur un ordinateur de votre organisation. Vous vous connectez également à Azure à l’aide d’un compte du Azure AD de votre [locataire](/previous-versions/azure/azure-services/jj573650(v=azure.100)#what-is-an-azure-ad-tenant). Les comptes Azure B2B (invité) ne sont pas pris en charge.

- **Inscrire votre passerelle** - Lors de cette étape, vous spécifiez un nom et une clé de récupération pour votre passerelle et vous sélectionnez une région, pour l’inscription de votre passerelle auprès du service cloud de passerelle. Votre ressource de passerelle peut être inscrite dans n’importe quelle région, mais il est recommandé de l’inscrire dans la même région que vos serveurs Analysis Services. 

- **Créer une ressource de passerelle dans Azure** - Dans cette étape, vous créez une ressource de passerelle dans Azure.

- **Connecter la ressource de passerelle aux serveurs** - Quand vous disposez d’une ressource de passerelle, vous pouvez commencer à y connecter des serveurs. Vous pouvez y connecter plusieurs serveurs et d’autres ressources, pourvu qu’ils soient dans la même région.

## <a name="installing"></a>Installation

Lors de l’installation d’un environnement Azure Analysis Services, il est important de suivre les étapes décrites dans [Installer et configurer la passerelle de données locale pour Azure Analysis Services](analysis-services-gateway-install.md). Cet article est spécifique à Azure Analysis Services. Il comprend des étapes supplémentaires requises pour configurer une ressource de passerelle de données locale dans Azure et connecter votre serveur Azure Analysis Services à la ressource.

## <a name="connecting-to-a-gateway-resource-in-a-different-subscription"></a>Connexion à une ressource de passerelle dans un autre abonnement

Il est recommandé de créer une ressource de passerelle Azure dans le même abonnement que votre serveur. Toutefois, vous pouvez configurer vos serveurs pour qu’ils se connectent à une ressource de passerelle dans un autre abonnement. La connexion à une ressource de passerelle dans un autre abonnement n’est pas prise en charge quand vous configurez les paramètres d’un serveur existant ou créez un serveur dans le portail, mais peut être configurée à l’aide de PowerShell. Pour en savoir plus, consultez [Connecter la ressource de passerelle au serveur](analysis-services-gateway-install.md#connect-gateway-resource-to-server).

## <a name="ports-and-communication-settings"></a>Paramètres de ports et de communication

La passerelle crée une connexion sortante vers Azure Service Bus. Elle communique sur les ports sortants : TCP 443 (par défaut), 5671, 5672 et 9350 à 9354.  La passerelle ne nécessite pas de ports entrants.

Il vous faudra peut-être inclure les adresses IP pour votre région de données dans votre pare-feu. Vous pouvez télécharger la [liste d’adresses IP de centre de données Microsoft Azure](https://www.microsoft.com/download/details.aspx?id=56519). Cette liste est actualisée chaque semaine. Les adresses IP répertoriées dans la liste d’adresses IP de centre de données Azure sont en notation CIDR. Pour plus d’informations, voir [Routage interdomaine sans classe](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing).

Voici les noms de domaine complets utilisés par la passerelle.

| Noms de domaine | Ports sortants | Description |
| --- | --- | --- |
| *. powerbi.com |80 |HTTP utilisé pour télécharger le programme d’installation. |
| *. powerbi.com |443 |HTTPS |
| *.analysis.windows.net |443 |HTTPS |
| *.login.windows.net, login.live.com, aadcdn.msauth.net |443 |HTTPS |
| *.servicebus.windows.net |5671-5672 |Advanced Message Queuing Protocol (AMQP) |
| *.servicebus.windows.net |443, 9350-9354 |Récepteurs du Service Bus Relay sur TCP (nécessite le port 443 pour l’acquisition du jeton Access Control) |
| *.frontend.clouddatahub.net |443 |HTTPS |
| *.core.windows.net |443 |HTTPS |
| login.microsoftonline.com |443 |HTTPS |
| *.msftncsi.com |80 |Permet de tester la connectivité internet si la passerelle est inaccessible par le service Power BI. |
| *.microsoftonline-p.com |443 |Utilisé pour l’authentification en fonction de la configuration. |
| dc.services.visualstudio.com    |443 |Utilisé par AppInsights pour collecter les données de télémétrie. |

## <a name="next-steps"></a>Étapes suivantes 

Les articles suivants sont inclus dans le contenu général de la passerelle de données locale qui s’applique à tous les services pris en charge par la passerelle :

* [Questions fréquentes (FAQ) sur la passerelle de données locale](/data-integration/gateway/service-gateway-onprem-faq)   
* [Utiliser l’application de passerelle de données locale](/data-integration/gateway/service-gateway-app)   
* [Administration au niveau du locataire](/data-integration/gateway/service-gateway-tenant-level-admin)
* [Configurer les paramètres de proxy](/data-integration/gateway/service-gateway-proxy)   
* [Ajuster les paramètres de communication](/data-integration/gateway/service-gateway-communication)   
* [Configurer des fichiers journaux](/data-integration/gateway/service-gateway-log-files)   
* [Résolution des problèmes](/data-integration/gateway/service-gateway-tshoot)
* [Surveiller et optimiser les performances de la passerelle](/data-integration/gateway/service-gateway-performance)

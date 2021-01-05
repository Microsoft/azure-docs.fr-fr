---
title: Vue d’ensemble
description: Découvrez comment Azure App Service vous aide à développer et héberger des applications web.
ms.assetid: 94af2caf-a2ec-4415-a097-f60694b860b3
ms.topic: overview
ms.date: 07/06/2020
ms.custom: devx-track-dotnet, mvc, seodec18
ms.openlocfilehash: e9c49d472f6cf0b9f0e5b3e2f7728417034059b1
ms.sourcegitcommit: 273c04022b0145aeab68eb6695b99944ac923465
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97007091"
---
# <a name="app-service-overview"></a>Vue d'ensemble d'App Service

*Azure App Service* est un service HTTP pour l’hébergement d’applications web, d’API REST et de backends mobiles. Vous pouvez développer dans votre langage préféré, par exemple .NET, .NET Core, Java, Ruby, Node.js, PHP ou Python. Les applications s’exécutent et sont mises à l’échelle facilement dans les environnements Windows et [Linux](#app-service-on-linux).

App Service ajoute la puissance de Microsoft Azure à votre application, notamment la sécurité, l’équilibrage de charge, la mise à l’échelle automatique et la gestion automatisée. Vous pouvez également bénéficier de ses fonctionnalités DevOps, notamment le déploiement continu depuis Azure DevOps, GitHub, Docker Hub et d’autres sources, la gestion des packages, les environnements intermédiaires, le domaine personnalisé et les certificats TLS/SSL. 

Avec App Service, vous payez pour les ressources de calcul Azure que vous utilisez. Les ressources de calcul que vous utilisez sont déterminées par le _plan App Service_ sur lequel vous exécutez vos applications. Pour plus d’informations, consultez [Vue d’ensemble des plans Azure App Service](overview-hosting-plans.md).

## <a name="why-use-app-service"></a>Pourquoi utiliser App Service ?

Voici quelques-unes des principales fonctionnalités d’App Service :

* **Plusieurs langages et frameworks** : App Service offre une prise en charge de première classe pour ASP.NET, ASP.NET Core, Java, Ruby, Node.js, PHP ou Python. Vous pouvez également exécuter [PowerShell et d’autres scripts ou exécutables](webjobs-create.md) comme services en arrière-plan.
* **Environnement de production managé** - App Service [corrige et tient à jour les frameworks de système d’exploitation et de langage](overview-patch-os-runtime.md) automatiquement. Consacrez votre temps à écrire des applications et laissez Azure se charger de la plateforme.
* **Conteneurisation et Docker** : dockerisez votre application et hébergez un conteneur Windows ou Linux personnalisé dans App Service. Exécutez des applications à plusieurs conteneurs avec Docker Compose. Migrez vos compétences Docker directement dans App Service.
* **Optimisation DevOps** : configurez [l’intégration et le déploiement continus](deploy-continuous-deployment.md) avec Azure DevOps, GitHub, BitBucket, Docker Hub ou Azure Container Registry. Assurez la promotion des mises à jour par le biais des [environnements de test et intermédiaires](deploy-staging-slots.md). Gérez vos applications dans App Service à l’aide d’[Azure PowerShell](/powershell/azure/) ou de la [CLI interplateforme](/cli/azure/install-azure-cli).
* **Mise à l’échelle globale avec une haute disponibilité** : effectuez des [montées en puissance](manage-scale-up.md) ou [augmentez la taille des instances](../azure-monitor/platform/autoscale-get-started.md) manuellement ou automatiquement. Hébergez vos applications n’importe où dans l’infrastructure mondiale des centres de données de Microsoft, et bénéficiez de garanties sur la haute disponibilité du  [SLA](https://azure.microsoft.com/support/legal/sla/app-service/) App Service.
* **Connexion aux plateformes SaaS et données locales** : choisissez parmi plus de 50 [connecteurs](../connectors/apis-list.md) pour des systèmes d’entreprise tels que SAP, des services SaaS tels que Salesforce et des services Internet tels que Facebook. Accédez aux données locales à l’aide de [connexions hybrides](app-service-hybrid-connections.md) et de [réseaux virtuels Azure](web-sites-integrate-with-vnet.md).
* **Sécurité et conformité** : App Service est [conforme aux normes ISO, SOC et PCI](https://www.microsoft.com/en-us/trustcenter). Authentifiez les utilisateurs avec [Azure Active Directory](configure-authentication-provider-aad.md), [Google](configure-authentication-provider-google.md), [Facebook](configure-authentication-provider-facebook.md), [Twitter](configure-authentication-provider-twitter.md) ou un [compte Microsoft](configure-authentication-provider-microsoft.md). Créez des [restrictions par adresse IP](app-service-ip-restrictions.md) et [gérez les identités de service](overview-managed-identity.md).
* **Modèles d’application** : faites votre choix parmi une liste complète de modèles d’application dans [Place de marché Azure](https://azure.microsoft.com/marketplace/), tels que WordPress, Joomla et Drupal.
* **Intégration Visual Studio et Visual Studio Code** : les outils dédiés de Visual Studio et Visual Studio Code rationalisent le travail de création, de déploiement et de débogage.
* **API et fonctionnalités mobiles** : App Service fournit une prise en charge CORS clés en main pour les scénarios API RESTful et simplifie les scénarios d’application mobile grâce à l’authentification, la synchronisation des données hors connexion, des notifications push et bien plus encore.
* **Code sans serveur** : exécutez un extrait de code ou un script à la demande sans avoir à configurer ou gérer explicitement l’infrastructure, et ne payez que pour le temps de calcul que votre code utilise réellement (voir [Azure Functions](../azure-functions/index.yml)).

En plus d’App Service, Azure offre d’autres services qui peuvent être utilisés pour l’hébergement de sites et d’applications web. Pour la plupart des scénarios, App Service est le meilleur choix.  Pour l’architecture microservice, utilisez le [service Azure Spring Cloud](../spring-cloud/index.yml) ou [Service Fabric](https://azure.microsoft.com/documentation/services/service-fabric).  Si vous avez besoin de contrôler davantage les machines virtuelles sur lesquelles votre code s’exécute, utilisez plutôt [Machines virtuelles Microsoft Azure](https://azure.microsoft.com/documentation/services/virtual-machines/). Pour plus d’informations sur le choix entre ces services Azure, consultez [Comparaison entre Azure App Service, Virtual Machines, Service Fabric et Cloud Services](/azure/architecture/guide/technology-choices/compute-decision-tree).

## <a name="app-service-on-linux"></a>App Service sur Linux

App Service sur Linux peut aussi héberger des applications web en mode natif sur Linux pour les piles d’applications prises en charge. En outre, il peut exécuter des conteneurs Linux personnalisés (Web App pour conteneurs).

### <a name="built-in-languages-and-frameworks"></a>Langages et infrastructures intégrés

App Service sur Linux prend en charge un certain nombre d’images intégrées spécifiques à au langage. Déployez simplement votre code. Les langages pris en charge incluent : Node.js, Java (JRE 8 et JRE 11), PHP, Python, .NET Core et Ruby. Exécutez [`az webapp list-runtimes --linux`](/cli/azure/webapp#az-webapp-list-runtimes) pour afficher les derniers langages et versions pris en charge. Si le runtime requis par votre application n’est pas pris en charge dans les images intégrées, vous pouvez la déployer avec un conteneur personnalisé.

### <a name="limitations"></a>Limites

- App Service sur Linux n’est pas pris en charge au niveau de tarification [Partagé](https://azure.microsoft.com/pricing/details/app-service/plans/). 
- Vous ne pouvez pas mélanger les applications Windows et Linux dans le même plan App Service.  
- Au sein du même groupe de ressources, vous ne pouvez pas mélanger les applications Windows et Linux dans la même région.
- Le portail Azure affiche uniquement les fonctionnalités actuellement compatibles avec les applications Linux. À mesure que des fonctionnalités sont compatibles, elles sont activées sur le portail.
- En cas de déploiement sur des images intégrées, votre code et votre contenu sont alloués à un volume de stockage pour le contenu web et sauvegardé par le Stockage Azure. La latence du disque de ce volume est supérieure et plus variable que la latence du système de fichiers du conteneur. L’option de conteneur peut être bénéfique aux applications qui nécessitent un accès en lecture seule intensif aux fichiers de contenu, car les fichiers sont placés dans le système de fichiers du conteneur plutôt que sur le volume de contenu.

## <a name="next-steps"></a>Étapes suivantes

Créez votre première application web.

> [!div class="nextstepaction"]
> [ASP.NET Core (sous Windows ou Linux)](quickstart-dotnetcore.md)

> [!div class="nextstepaction"]
> [ASP.NET (sous Windows)](quickstart-dotnet-framework.md)

> [!div class="nextstepaction"]
> [PHP (sous Windows ou Linux)](quickstart-php.md)

> [!div class="nextstepaction"]
> [Ruby (sous Linux)](quickstart-ruby.md)

> [!div class="nextstepaction"]
> [Node.js (sous Windows ou Linux)](quickstart-nodejs.md)

> [!div class="nextstepaction"]
> [Java (sous Windows ou Linux)](quickstart-java.md)

> [!div class="nextstepaction"]
> [Python (sous Linux)](quickstart-python.md)

> [!div class="nextstepaction"]
> [HTML (sous Windows ou Linux)](quickstart-html.md)

> [!div class="nextstepaction"]
> [Conteneur personnalisé (Windows ou Linux)](tutorial-custom-container.md)

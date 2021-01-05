---
title: Intégration d’Application Gateway par des points de terminaison de service – Azure App Service | Microsoft Docs
description: Décrit comment Application Gateway s’intègre à Azure App Service sécurisé par des points de terminaison de service.
services: app-service
documentationcenter: ''
author: madsd
manager: ccompy
editor: ''
ms.assetid: 073eb49c-efa1-4760-9f0c-1fecd5c251cc
ms.service: app-service
ms.workload: web
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 12/09/2019
ms.author: madsd
ms.custom: seodec18, devx-track-azurecli
ms.openlocfilehash: 954e94063ec91cd2a6d67d154dfd7da553e0935a
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94560891"
---
# <a name="application-gateway-integration-with-service-endpoints"></a>Intégration d’Application Gateway par des points de terminaison de service
Il existe trois variantes d’App Service qui nécessitent une configuration légèrement différente de l’intégration avec Azure Application Gateway. Les variantes incluent la version normale d’App Service, également appelée multilocataire, App Service Environment (ASE) Load Balancer interne (ILB) et ASE externe. Cet article explique comment le configurer avec App Service (multilocataire) et aborde les considérations relatives aux ASE ILB et externe.

## <a name="integration-with-app-service-multi-tenant"></a>Intégration à App Service (multilocataire)
App Service (multilocataire) dispose d’un point de terminaison public accessible sur Internet. À l’aide des [points de terminaison de service](../../virtual-network/virtual-network-service-endpoints-overview.md), vous pouvez autoriser le trafic uniquement à partir d’un sous-réseau spécifique au sein d’un réseau virtuel Azure et bloquer tout le reste. Dans le scénario suivant, nous allons utiliser cette fonctionnalité pour nous assurer qu’une instance App Service peut uniquement recevoir le trafic d’une instance Application Gateway spécifique.

![Diagramme montrant le flux Internet qui se dirige vers une passerelle applicative dans un réseau virtuel Azure et, de là, traverse une icône de pare-feu pour continuer vers des instances d’applications dans App Service.](./media/app-gateway-with-service-endpoints/service-endpoints-appgw.png)

Cette configuration comprend deux parties, en plus de la création des instances App Service et Application Gateway. La première partie consiste à activer des points de terminaison de service dans le sous-réseau du réseau virtuel sur lequel Application Gateway est déployé. Les points de terminaison de service garantissent que tout le trafic qui quitte le sous-réseau vers App Service sera marqué avec l’ID spécifique du sous-réseau. La deuxième partie consiste à définir une restriction d’accès de l’application web spécifique pour garantir que seul le trafic marqué avec cet ID de sous-réseau spécifique est autorisé. Vous pouvez le configurer à l’aide de différents outils, selon vos préférences.

## <a name="using-azure-portal"></a>En passant par le portail Azure
Avec Portail Azure, vous suivez quatre étapes pour approvisionner et configurer l’installation. Si vous disposez de ressources existantes, vous pouvez ignorer les premières étapes.
1. Créez une instance App Service à l’aide de l’un des guides de démarrage rapide dans la documentation d’App Service, par exemple [Guide de démarrage rapide .Net Core](../quickstart-dotnetcore.md).
2. Créez une instance Application Gateway à l’aide du [portail Démarrage rapide](../../application-gateway/quick-create-portal.md), mais ignorez la section Ajouter des cibles de serveur principal.
3. Configurez [App Service en tant que serveur principal dans Application Gateway](../../application-gateway/configure-web-app-portal.md), mais ignorez la section Restreindre l’accès.
4. Enfin, créez la [restriction d’accès à l’aide de points de terminaison de service](../../app-service/app-service-ip-restrictions.md#use-service-endpoints).

Vous pouvez maintenant accéder au App Service via Application Gateway, mais si vous essayez d’y accéder directement, vous devriez recevoir une erreur HTTP 403 indiquant que le site web est arrêté.

![Capture d’écran montrant le texte d’une erreur 403 : Cette application web est arrêtée.](./media/app-gateway-with-service-endpoints/web-site-stopped.png)

## <a name="using-azure-resource-manager-template"></a>Utilisation d’un modèle Azure Resource Manager
Le [modèle de déploiement Resource Manager][template-app-gateway-app-service-complete] fournira un scénario complet. Le scénario se compose d’une instance App Service verrouillée par des points de terminaison de service et une restriction d’accès pour recevoir uniquement le trafic provenant d’Application Gateway. Le modèle comprend de nombreuses valeurs Smart Defaults et des suffixes uniques ajoutés aux noms des ressources pour qu’ils soient simples. Pour les remplacer, vous devez cloner le référentiel ou télécharger le modèle pour le modifier. 

Pour appliquer le modèle, vous pouvez utiliser le bouton Déployer sur Azure figurant dans la description du modèle, ou vous pouvez utiliser l’outil PowerShell/CLI approprié.

## <a name="using-azure-command-line-interface"></a>Utilisation de l’interface de ligne de commande Azure
L’[exemple de l’interface de ligne de commande Azure](../../app-service/scripts/cli-integrate-app-service-with-application-gateway.md) configurera une instance App Service verrouillée par des points de terminaison de service et une restriction d’accès pour recevoir uniquement le trafic provenant d’Application Gateway. Si vous devez uniquement isoler le trafic vers une instance App Service existante et provenant d’une instance Application Gateway existante, la commande suivante suffit.

```azurecli-interactive
az webapp config access-restriction add --resource-group myRG --name myWebApp --rule-name AppGwSubnet --priority 200 --subnet mySubNetName --vnet-name myVnetName
```

Dans la configuration par défaut, la commande vérifie à la fois le paramétrage de la configuration du point de terminaison de service dans le sous-réseau et la restriction d’accès dans l’App Service.

## <a name="considerations-for-ilb-ase"></a>Considérations relatives à ASE ILB
ASE ILB n’est pas exposé à Internet et le trafic entre l’instance et une Application Gateway est donc déjà isolé dans le réseau virtuel. Le [guide pratique](../environment/integrate-with-application-gateway.md) suivant configure un ASE ILB et l’intègre à une Application Gateway à l’aide du Portail Azure. 

Si vous souhaitez vous assurer que seul le trafic provenant du sous-réseau Application Gateway atteint l’ASE, vous pouvez configurer un groupe de sécurité réseau (NSG) qui affecte toutes les applications web dans l’ASE. Pour le NSG, vous pouvez spécifier la plage d’adresses IP du sous-réseau et éventuellement les ports (80/443). Assurez-vous que vous ne remplacez pas les [règles NSG requises](../environment/network-info.md#network-security-groups) pour qu’ASE fonctionne correctement.

Pour isoler le trafic à une application web individuelle, vous devez utiliser des restrictions d’accès basées sur l’adresse IP, car les points de terminaison de service ne fonctionneront pas pour ASE. L’adresse IP doit être l’adresse IP privée de l’instance Application Gateway.

## <a name="considerations-for-external-ase"></a>Considérations relatives à ASE externe
ASE externe dispose d’un équilibreur de charge public, comme App Service multilocataire. Les points de terminaison de service ne fonctionnent pas pour ASE, et c’est pourquoi vous devrez utiliser des restrictions d’accès basées sur l’adresse IP en utilisant l’IP publique de l’instance Application Gateway. Pour créer un ASE externe à l’aide du Portail Azure, vous pouvez suivre ce [guide de démarrage rapide](../environment/create-external-ase.md)

[template-app-gateway-app-service-complete]: https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-with-app-gateway-v2/ "Modèle Azure Resource Manager pour un scénario complet"

## <a name="considerations-for-kuduscm-site"></a>Considérations relatives au site kudu/scm
Le site scm, également appelé kudu, est un site d’administration qui existe pour chaque application web. Il n’est pas possible d’inverser le proxy du site scm et il est fort probable que vous souhaitiez également le verrouiller à des adresses IP individuelles ou à un sous-réseau spécifique.

Si vous souhaitez utiliser les mêmes restrictions d’accès que le site principal, vous pouvez hériter des paramètres à l’aide de la commande suivante.

```azurecli-interactive
az webapp config access-restriction set --resource-group myRG --name myWebApp --use-same-restrictions-for-scm-site
```

Si vous souhaitez définir des restrictions d’accès individuelles pour le site scm, vous pouvez ajouter des restrictions d’accès à l’aide de l’indicateur --scm-site comme indiqué ci-dessous.

```azurecli-interactive
az webapp config access-restriction add --resource-group myRG --name myWebApp --scm-site --rule-name KudoAccess --priority 200 --ip-address 208.130.0.0/16
```

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur App Service Environment, consultez [Documentation sur App Service Environment](/azure/app-service/environment).

Pour renforcer la sécurité de votre application web, vous trouverez des informations relatives au pare-feu d’applications web sur Application Gateway dans la [documentation du pare-feu d’applications web Azure](../../web-application-firewall/ag/ag-overview.md).

---
title: Forum aux questions sur la connectivité réseau Analysis Services | Microsoft Docs
description: Cet article fournit des réponses à certaines des questions les plus courantes sur la connectivité réseau Analysis Services.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 05/05/2020
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: b60cf34e8efed2ed63b6e35cfaf7445edb701610
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "82838499"
---
# <a name="frequently-asked-questions-about-analysis-services-network-connectivity"></a>Forum aux questions sur la connectivité réseau Analysis Services

Cet article fournit des réponses aux questions courantes sur la connexion aux comptes de stockage, aux sources de données, aux pare-feu et aux adresses IP.

## <a name="backup-and-restore"></a>Sauvegarde et restauration

**Question** : Comment sauvegarder et restaurer à l’aide d’un compte de stockage qui se trouve derrière un pare-feu ?   
**Réponse** : Azure Analysis Services n’utilise pas d’adresses IP fixes ni de balises de service. La plage d’adresses IP utilisée par vos serveurs Analysis Services peut être n’importe quelle valeur dans la plage d’adresses IP de la *région Azure*. Étant donné que les adresses IP de votre serveur sont variables et peuvent changer au fil du temps, vos règles de pare-feu doivent autoriser l’intégralité de la plage d’adresses IP de la région Azure dans laquelle se trouve votre serveur.

**Question** : Mon compte de stockage Azure se trouve dans une région différente de celle de mon serveur Analysis Services. Comment configurer les paramètres de pare-feu du compte de stockage ?   
**Réponse** : si le compte de stockage se trouve dans une autre région, les paramètres du pare-feu du compte de stockage doivent être configurés de façon à autoriser l’accès à partir des **Réseaux sélectionnés**. Dans la **plage d’adresses** du pare-feu, spécifiez la plage d’adresses IP pour la région dans laquelle se trouve le serveur Analysis Services. Pour obtenir les plages d’adresses IP pour les régions Azure, consultez [des plages d’adresses IP Azure et des balises de service – cloud public](https://www.microsoft.com/download/details.aspx?id=56519). La configuration des paramètres de pare-feu de compte de stockage pour autoriser l’accès à partir de tous les réseaux est prise en charge, mais il est préférable de choisir des réseaux sélectionnés et de spécifier une plage d’adresses IP. 

**Question** : mon compte de stockage Azure se trouve dans une région différente de celle de mon serveur Analysis Services. Comment configurer les paramètres de pare-feu du compte de stockage ?   
**Réponse** : étant donné que votre compte de stockage et de serveur Analysis Services se trouvent dans la même région, les communications entre eux utilisent des plages d’adresses IP internes. Par conséquent, la configuration d’un pare-feu pour utiliser des réseaux sélectionnés et la spécification d’une plage d’adresses IP ne sont pas prises en charge. Si les stratégies de l’organisation requièrent un pare-feu, elles doivent être configurées pour autoriser l’accès à partir de tous les réseaux.


## <a name="data-source-connections"></a>Connexions à la source de données

**Question** : Je dispose d’un réseau virtuel pour mon système de source de données. Comment autoriser mes serveurs Analysis Services à accéder à la base de données à partir du réseau virtuel ?   
**Réponse** : Azure Analysis Services n’est pas en mesure de rejoindre un réseau virtuel. La meilleure solution ici consiste à installer et à configurer une passerelle de données locale sur le réseau virtuel, puis à configurer vos serveurs Analysis Services avec la propriété de serveur **AlwaysUseGateway**. Pour en savoir plus, consultez [Utiliser la passerelle pour les sources de données sur un Réseau virtuel Microsoft Azure](analysis-services-vnet-gateway.md).

**Question** : Je dispose d’une base de données source derrière un pare-feu. Comment configurer le pare-feu pour autoriser mon serveur Analysis Services à y accéder ?   
**Réponse** : Azure Analysis Services n’utilise pas d’adresses IP fixes ni de balises de service. La plage d’adresses IP utilisée par vos serveurs Analysis Services peut être n’importe quelle valeur dans la plage d’adresses IP de la *région Azure*. Vous devez indiquer la *plage complète* d’adresses IP pour la région Azure de votre serveur dans les règles de pare-feu de la base de données source. Une autre solution, peut-être plus sûre, consiste à configurer une passerelle de données locale. Vous pouvez ensuite configurer vos serveurs Analysis Services avec la [propriété de serveur AlwaysUseGateway](analysis-services-vnet-gateway.md#configure-alwaysusegateway-property), puis vérifier que la passerelle de données locale a une adresse IP autorisée par les règles de pare-feu de la source de données.

## <a name="azure-apps-with-ip-address"></a>Applications Azure avec adresse IP

**Question** : J’utilise une application basée sur Azure (par exemple, Azure Functions ou Azure Data Factory) avec une adresse IP qui change à la volée. Comment gérer les règles de pare-feu d’Azure Analysis Services pour autoriser dynamiquement l’adresse IP sur laquelle mon application s’exécute ?   
**Réponse** : Azure Analysis Services ne prend pas en charge les liaisons privées, les réseaux virtuels ni les balises de service. Il existe des solutions open source (par exemple, https://github.com/mathwro/Scripts/blob/master/Azure/AllowAzure-AnalysisServer.ps1) qui détectent l’adresse IP de l’application cliente et mettent à jour automatiquement et temporairement les règles de pare-feu.


## <a name="next-steps"></a>Étapes suivantes

[Installer et configurer une passerelle de données locale](analysis-services-gateway-install.md)   
[Connexion aux sources de données locales avec la passerelle de données locale](analysis-services-gateway.md)   
[Utiliser la passerelle pour les sources de données sur un Réseau virtuel Azure](analysis-services-vnet-gateway.md)

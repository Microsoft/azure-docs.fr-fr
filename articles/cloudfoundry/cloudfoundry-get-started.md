---
title: Prise en main de Cloud Foundry sur Microsoft Azure
description: Exécuter OSS ou Pivotal Cloud Foundry sur Microsoft Azure
author: seanmck
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.date: 01/19/2017
ms.author: seanmck
ms.openlocfilehash: 8645afac5734cdbffb8df6f341529e688ca526d7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84754449"
---
# <a name="cloud-foundry-on-azure"></a>Cloud Foundry sur Azure

Cloud Foundry est une solution de type plate-forme en tant que service (PaaS) open source pour la génération, le déploiement et l’exploitation d’applications 12 facteurs développées dans plusieurs langages et infrastructures. Ce document décrit les options dont vous disposez pour exécuter Cloud Foundry sur Azure et vous explique comment commencer.

## <a name="cloud-foundry-offerings"></a>Offres Cloud Foundry

Cloud Foundry peut s’exécuter sur Azure selon deux formes : Cloud Foundry open source (OSS CF) et Pivotal Cloud Foundry (PCF). OSS CF est une version entièrement [open source](https://github.com/cloudfoundry) de Cloud Foundry, gérée par Cloud Foundry Foundation. Pivotal Cloud Foundry est une version de Cloud Foundry pour les entreprises fournie par Pivotal Software Inc. Nous allons aborder certaines différences entre les deux offres.

### <a name="open-source-cloud-foundry"></a>Cloud Foundry open source

Vous pouvez déployer OSS Cloud Foundry sur Azure en déployant d’abord un directeur BOSH, puis en déployant Cloud Foundry en suivant les [instructions fournies sur GitHub](https://github.com/cloudfoundry-incubator/bosh-azure-cpi-release/blob/master/docs/guidance.md). Pour en savoir plus sur l’utilisation d’OSS CF, consultez la [documentation](https://docs.cloudfoundry.org/) fournie par Cloud Foundry Foundation.

Microsoft offre un support de type « Meilleur effort » pour OSS CF, par le biais des canaux communautaires suivants :

- #<a name="bosh-azure-cpi-channel-on-cloud-foundry-slack"></a>Canal bosh-azure-cpi sur [Cloud Foundry Slack](https://slack.cloudfoundry.org/)
- [Liste de diffusion cf-bosh](https://lists.cloudfoundry.org/pipermail/cf-bosh)
- Problèmes GitHub pour le [CPI](https://github.com/cloudfoundry-incubator/bosh-azure-cpi-release/issues) et [Service Broker](https://github.com/Azure/meta-azure-service-broker/issues)

>[!NOTE]
> Le niveau de support assuré pour vos ressources Azure, notamment les machines virtuelles sur lesquelles vous exécutez Cloud Foundry, est basé sur votre contrat de support Azure. Le support de la communauté de type « Meilleur effort » s’applique uniquement aux composants spécifiques à Cloud Foundry.

### <a name="pivotal-cloud-foundry"></a>Pivotal Cloud Foundry

Pivotal Cloud Foundry inclut la même plateforme principale que la version OSS, ainsi qu’un ensemble d’outils de gestion propriétaires et un support pour entreprise. Pour exécuter PCF sur Azure, vous devez acquérir une licence auprès de Pivotal. L’offre PCF sur la Place de marché Microsoft Azure inclut une licence d’évaluation de 90 jours.

Les outils incluent [Pivotal Operations Manager](https://docs.pivotal.io/pivotalcf/customizing/), une application web qui simplifie le déploiement et la gestion d’une solution Cloud Foundry Foundation, et [Pivotal Apps Manager](https://docs.pivotal.io/pivotalcf/console/), une application web pour la gestion des utilisateurs et des applications.

Outre les canaux de support répertoriés ci-avant pour OSS CF, une licence PCF vous autorise à contacter Pivotal pour vos besoins en support. Microsoft et Pivotal ont également mis en place des flux de travail de support qui vous permettent de les contacter et qui acheminent votre demande de façon appropriée selon la nature du problème.

## <a name="azure-service-broker"></a>Azure Service Broker

Cloud Foundry recommande la méthodologie basée sur les [« applications 12 facteurs »](https://12factor.net/), qui encourage une séparation claire entre les processus d’application sans état et les services de soutien avec état. Les [Service Brokers](https://docs.cloudfoundry.org/services/api.html) offrent un moyen cohérent de configurer et de lier les services de soutien aux applications. [Azure Service Broker](https://github.com/Azure/meta-azure-service-broker) propose certains services Azure clés par le biais de ce canal, notamment Stockage Azure et Azure SQL.

Si vous utilisez Pivotal Cloud Foundry, le Service Broker est également [disponible sous forme de mosaïque](https://docs.pivotal.io/azure-sb/installing.html) à partir du réseau Pivotal.

## <a name="related-resources"></a>Ressources associées

### <a name="azure-devops-services-plugin"></a>Plug-in Azure DevOps Services

Cloud Foundry convient idéalement au développement agile de logiciels et permet notamment de recourir à l’intégration continue et à la livraison continue. Si vous utilisez Azure DevOps Services pour gérer vos projets et souhaitez configurer un pipeline avec intégration continue/livraison continue ciblant Cloud Foundry, vous pouvez utiliser l’[extension de build Cloud Foundry pour Azure DevOps Services](https://marketplace.visualstudio.com/items?itemName=ms-vsts.cloud-foundry-build-extension). Le plug-in simplifie la configuration et l’automatisation des déploiements sur Cloud Foundry, que l’exécution s’effectue dans Azure ou dans un autre environnement.

## <a name="next-steps"></a>Étapes suivantes

- [Pivotal Cloud Foundry on Microsoft Azure](https://azuremarketplace.microsoft.com/marketplace/apps/pivotal.pivotal-cloud-foundry) (Pivotal Cloud Foundry sur Microsoft Azure)
- [Déployer une application vers Cloud Foundry dans Azure](./cloudfoundry-deploy-your-first-app.md)

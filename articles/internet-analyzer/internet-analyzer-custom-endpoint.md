---
title: Créer un point de terminaison personnalisé | Microsoft Docs
description: Dans cet article, découvrez comment configurer un point de terminaison personnalisé à mesurer avec votre ressource Internet Analyzer.
services: internet-analyzer
author: mattcalder
ms.service: internet-analyzer
ms.topic: how-to
ms.date: 10/16/2019
ms.author: mebeatty
ms.openlocfilehash: 6ca71bce726f16eeacc96e10eb654bb3e21c5924
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84744083"
---
# <a name="measure-custom-endpoints-to-evaluate-in-your-internet-analyzer-tests"></a>Mesurer les points de terminaison personnalisés à évaluer dans vos tests Internet Analyzer 

Cet article explique comment configurer un point de terminaison personnalisé à mesurer dans le cadre de vos tests Internet Analyzer. Les points de terminaison personnalisés permettent d’évaluer les charges de travail locales, les charges de travail exécutées sur d’autres fournisseurs de cloud et les configurations Azure personnalisées.  La comparaison de deux points de terminaison personnalisés dans un test est possible si un point de terminaison est une ressource Azure. Pour plus d’informations sur Internet Analyzer, consultez la [vue d’ensemble](internet-analyzer-overview.md). 

> [!IMPORTANT]
> Cette préversion publique est fournie sans contrat de niveau de service et ne doit pas être utilisée pour les charges de travail de production. Certaines fonctionnalités peuvent ne pas être prises en charge, disposer de capacités limitées ou ne pas être disponibles dans tous les emplacements Azure. Consultez les [Conditions d’utilisation supplémentaires des préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).
>

## <a name="before-you-begin"></a>Avant de commencer

Veillez à configurer une ressource Internet Analyzer et sélectionnez l’option « point de terminaison personnalisé ». Internet Analyzer part du principe que votre point de terminaison personnalisé est accessible via Internet. Pour en savoir plus, consultez [Create an Internet Analyzer Resource](internet-analyzer-create-test-portal.md) (Créer une ressource Internet Analyzer).


## <a name="create-custom-endpoint"></a>Créer un point de terminaison personnalisé

1. Téléchargez une image de test transparente d’un pixel [ici](https://fpc.msedge.net/apc/trans.gif). Cette image d’un pixel est la ressource que le code JavaScript du client va extraire pour mesurer les performances.
2. Dans votre application web personnalisée, déployez l’image de test dans un chemin accessible publiquement. Le chemin d’accès doit fonctionner sur HTTPS. 
3. Copiez l’intégralité de l’URL du point de terminaison personnalisé (par exemple, `https://contoso.com/test/trans.gif`) dans le champ de point de terminaison personnalisé lors de la création de votre test.

## <a name="next-steps"></a>Étapes suivantes

Consultez la [FAQ sur Internet Analyzer](internet-analyzer-faq.md)


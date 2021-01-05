---
title: Prise en charge du gestionnaire de rendus
description: Utilisation de l'intégration du gestionnaire de rendu Azure Batch Découvrez-en plus sur la prise en charge intégrée ou les modules complémentaires relatifs aux gestionnaires de rendus courants.
author: mscurrell
ms.author: markscu
ms.date: 08/02/2018
ms.topic: how-to
ms.openlocfilehash: c44cbf86d8bf2fe83a6dc91dee1c4f58eec156c0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83726449"
---
# <a name="using-azure-batch-with-render-farm-managers"></a>Utilisation d’Azure Batch avec des gestionnaires de groupe de rendus

Si vous utilisez un groupe de rendus local existant, il est fort probable qu’un gestionnaire de rendus contrôle la capacité du groupe de rendus et les travaux de rendu.

Azure fournit la prise en charge intégrée ou des modules complémentaires pour les gestionnaires de rendus courants. Vous pouvez ensuite ajouter et supprimer des machines virtuelles Azure, y compris les machines virtuelles avec une licence d’application de paiement à l’utilisation et les machines virtuelles de faible priorité.

Les gestionnaires de rendus suivants sont pris en charge :

* [PipelineFX Qube!](https://www.pipelinefx.com/)
* [Royal Render](https://www.royalrender.de/)
* [Thinkbox Deadline](https://deadline.thinkboxsoftware.com/)

## <a name="azure-render-hub"></a>Azure Render Hub

Azure Render Hub simplifie la création et la gestion des groupes de rendus Azure.  Render Hub assure une prise en charge native de PipelineFx Qube et Deadline 10.  Pour obtenir plus d’informations et des instructions détaillées, consultez le [dépôt GitHub](https://github.com/Azure/azure-render-hub).

## <a name="using-azure-with-pipelinefx-qube"></a>Utilisation d’Azure avec PipelineFX Qube

Azure Render Hub prend en charge les gestionnaires de rendus populaires, dont Deadline.  Pour obtenir des instructions sur le déploiement et l’utilisation de Render Hub, consultez le [dépôt GitHub](https://github.com/Azure/azure-render-hub).

Les scripts et instructions permettant d’utiliser des machines virtuelles du pool Azure Batch comme workers Qube sont également disponibles dans le [dépôt GitHub](https://github.com/Azure/azure-qube).

## <a name="using-azure-with-royal-render"></a>Utilisation d’Azure avec Royal Render

Azure et Azure Batch sont intégrés dans Royal Render, ce qui vous permet d’étendre un groupe de rendus avec des machines virtuelles Azure. Pour obtenir un résumé, consultez les [fichiers d’aide](https://www.royalrender.de/help8/index.html?Cloudrendering.html).

Pour obtenir un exemple d’un client Royal Render utilisant l’intégration d’Azure, consultez le [témoignage client de Jellyfish Pictures](https://customers.microsoft.com/story/jellyfishpictures).

## <a name="using-azure-with-thinkbox-deadline"></a>Utilisation d’Azure avec Thinkbox Deadline

Azure Render Hub prend en charge les gestionnaires de rendus populaires, dont Deadline.  Pour obtenir des instructions sur le déploiement et l’utilisation de Render Hub, consultez le [dépôt GitHub](https://github.com/Azure/azure-render-hub).

## <a name="next-steps"></a>Étapes suivantes

Essayez l’intégration Azure Batch de votre gestionnaire de rendus, en utilisant le plug-in et les instructions appropriés sur GitHub, le cas échéant.

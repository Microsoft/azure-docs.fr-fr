---
title: Présentation du rendu
description: Présentation de l’utilisation d’Azure pour le rendu et vue d’ensemble des fonctionnalités de rendu Azure Batch
author: mscurrell
ms.author: markscu
ms.date: 08/02/2018
ms.topic: how-to
ms.openlocfilehash: 9fac5d3efabc5d9f796c91d688f35e01aeefdca3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87092760"
---
# <a name="rendering-using-azure"></a>Rendu à l’aide d’Azure

Le rendu correspond au processus visant à récupérer des modèles 3D et à les convertir en images 2D. Les fichiers de scène 3D sont créés dans des applications telles qu’Autodesk 3ds Max, Autodesk Maya et Blender.  Les applications de rendu telles qu’Autodesk Maya, Autodesk Arnold, Chaos Group V-Ray et Blender Cycles produisent des images 2D.  Des images sont parfois créées à partir de fichiers de scène. Toutefois, il est courant de modéliser et d’effectuer le rendu de plusieurs images, puis de les combiner dans une animation.

La charge de travail de rendu est principalement utilisée pour les effets spéciaux (VFX) dans l’industrie du multimédia et du divertissement. Le rendu est également utilisé dans de nombreux autres secteurs comme la publicité, la vente au détail, le pétrole et le gaz et la fabrication.

Le processus de rendu est très gourmand en ressources ; le nombre d’images à produire peut être très important et le rendu de chaque image peut prendre plusieurs heures.  Par conséquent, le rendu est une charge de travail de traitement par lots idéale qui peut utiliser Azure et Azure Batch pour exécuter plusieurs rendus en parallèle.

## <a name="why-use-azure-for-rendering"></a>Pourquoi utiliser Azure pour le rendu ?

Pour de nombreuses raisons, le rendu est une charge de travail parfaitement adaptée pour Azure et Azure Batch :

* Les travaux de rendu peuvent être fractionnés en plusieurs éléments qui peuvent être exécutés en parallèle à l’aide de plusieurs machines virtuelles :
  * Les animations sont constituées de nombreuses images, et chaque image peut être rendue en parallèle.  Plus le nombre de machines virtuelles disponibles pour traiter que image est important, et plus la production de toutes les images et de l’animation sera rapide.
  * Certains logiciels de rendu permettent de diviser des images uniques en plusieurs éléments, comme des vignettes ou tranches.  Chaque élément peut être rendu séparément, puis combiné dans l’image finale lorsque tous les éléments sont terminés.  Plus le nombre de machines virtuelles disponibles est important, et plus le rendu d’une image sera rapide.
* Les projets de rendu peuvent nécessiter une mise à l’échelle importante :
  * Les images individuelles peuvent être complexes et leur rendu peut prendre plusieurs heures, même sur du matériel haut de gamme ; les animations peuvent contenir des centaines de milliers d’images.  Une quantité de calcul importante est nécessaire pour effectuer le rendu d’animations de haute qualité dans un délai raisonnable.  Dans certains cas, plus de 100 000 cœurs ont été utilisés pour effectuer le rendu de milliers d’images en parallèle.
* Les projets de rendu sont basés sur le projet et nécessitent différentes quantités de calcul :
  * Allouez de la capacité de calcul et de stockage si nécessaire, augmentez-la ou réduisez-la en fonction de la charge pendant le projet, puis supprimez-la lorsque le projet est terminé.
  * Payez la capacité lorsqu’elle est allouée, mais ne la payez pas lorsqu’il n’y a pas de charge, entre les projets par exemple.
  * Répondez aux pics d’activité dus à des modifications non prévues ; augmentez en cas de modifications inattendues et tardives dans un projet, ces modifications doivent être traitées dans un délai serré.
* Choisissez parmi un large choix de matériels en fonction de l’application, de la charge de travail et du délai d’exécution :
  * Un large choix de matériels est disponible dans Azure, qui peut être alloué et géré avec Batch.
  * En fonction du projet, la configuration requise peut être le meilleur rapport prix/performances ou les meilleures performances globales.  Différentes applications de scènes et/ou de rendu auront des exigences de mémoire différentes.  Certaines applications de rendu peuvent utiliser des GPU pour des performances optimales ou certaines fonctionnalités. 
* Les machines virtuelles de faible priorité réduisent les coûts :
  * Les machines virtuelles de faible priorité sont disponibles à un prix bien moindre que les machines virtuelles à la demande classiques et conviennent pour certains types de travaux.
  * Les machines virtuelles de faible priorité peuvent être allouées par Azure Batch, Batch offrant la flexibilité sur leur utilisation pour répondre à un large éventail d’exigences.  Les pools batch peuvent comprendre des machines virtuelles dédiées et de faible priorité, permettant ainsi de modifier la combinaison de types de machine virtuelle à tout moment.

## <a name="options-for-rendering-on-azure"></a>Options de rendu sur Azure

Toute une gamme de fonctionnalités Azure peut être utilisée pour les charges de travail de rendu.  Les fonctionnalités à utiliser dépendent de l’environnement et des exigences.

### <a name="existing-on-premises-rendering-environment-using-a-render-management-application"></a>Environnement de rendu local existant utilisant une application de gestion du rendu

Le cas le plus courant est un groupe de rendus local existant géré par une application de gestion du rendu comme PipelineFX Qube, Royal Rendu ou Thinkbox Deadline.  L’exigence consiste à étendre la capacité du groupe de rendus local à l’aide de machines virtuelles Azure.

Le logiciel de gestion du rendu intègre la prise en charge Azure ou nous mettons à disposition des plug-ins incluant la prise en charge Azure. Pour plus d’informations sur les gestionnaire de rendu pris en charge et la fonctionnalité activée, consultez l’article sur l’[utilisation des gestionnaires de rendu](./batch-rendering-render-managers.md).

### <a name="custom-rendering-workflow"></a>Flux de travail de rendu personnalisé

L’exigence porte sur les machines virtuelles afin d’étendre un groupe de rendus existant.  Les pools Azure Batch peuvent allouer de grands nombres de machines virtuelles, autoriser l’utilisation de machines virtuelles de faible priorité et leur mise à l’échelle automatique avec des machines virtuelles à prix intégral, et proposer une licence de paiement à l’utilisation pour les applications de rendu populaires.

### <a name="no-existing-render-farm"></a>Aucun groupe de rendus existant

Les stations de travail clientes peuvent effectuer le rendu, mais la charge de travail de rendu augmente et prend trop de temps pour n’utiliser que la capacité de la station de travail.  Azure Batch peut être utilisé pour allouer le calcul du groupe de rendus à la demande et pour planifier les travaux de rendu au groupe de rendus Azure.

## <a name="azure-batch-rendering-capabilities"></a>Fonctionnalités de rendu Azure Batch

Azure Batch permet d’exécuter des charges de travail parallèles dans Azure.  Il permet de créer et de gérer de grands nombres de machines virtuelles sur lesquelles les applications sont installées et exécutées.  Il fournit également des fonctionnalités de planification des travaux complètes pour exécuter des instances de ces applications, en permettant d’affecter des tâches aux machines virtuelles, de mettre en file d’attente, de contrôler les applications, et bien d’autres encore.

Azure Batch est utilisé pour nombreuses charges de travail, mais les fonctionnalités suivantes sont disponibles pour simplifier et accélérer plus particulièrement l’exécution des charges de travail de rendu.

* Images de machine virtuelle avec des applications graphiques et de rendu préinstallées :
  * Des images de machine virtuelle de la Place de marché Azure contenant des applications graphiques et de rendu populaires sont disponibles, vous évitant ainsi de les applications vous-même ou de créer vos propres images personnalisées avec les applications installées. 
* Licence avec paiement à l’utilisation pour les applications de rendu :
  * Vous pouvez choisir de payer pour les applications à la minute, en plus de payer pour les machines virtuelles de calcul, vous évitant ainsi d’acheter des licences et d’éventuellement configurer un serveur de licences pour les applications.  Le paiement à l’utilisation implique également qu’il est possible de répondre à une charge changeante et inattendue car il n’existe pas un nombre déterminé de licences.
  * Il est également possible d’utiliser les applications préinstallées avec vos propres licences et de ne pas utiliser la licence avec paiement à l’utilisation. Pour cela, vous installez généralement un serveur de licences local ou dans Azure, et vous utilisez un réseau virtuel Azure pour connecter le pool de rendu au serveur de licences.
* Plug-ins pour les applications de conception de modélisation clientes :
  * Les plug-ins permettent aux utilisateurs finaux d’utiliser Azure Batch directement à partir de l’application cliente, telle qu’Autodesk Maya, leur permettant ainsi de créer des pools, de soumettre des travaux et d’utiliser plus de capacité de calcul pour des rendus plus rapides.
* Intégration du gestionnaire de rendu :
  * Azure Batch est intégré dans des applications de gestion du rendu ou des plug-ins sont disponibles pour l’intégration d’Azure Batch.

Plusieurs méthodes permettent d’utiliser Azure Batch, toutes s’appliquant également au rendu Azure Batch.

* API :
  * Écrivez du code à l’aide de l’API [REST](/rest/api/batchservice), [.NET](/dotnet/api/overview/azure/batch), [Python](/python/api/overview/azure/batch), [Java](/java/api/overview/azure/batch) ou d’autres API prises en charge.  Les développeurs peuvent intégrer les fonctionnalités Azure Batch dans leurs applications ou workflows existants, dans le cloud ou en local.  Par exemple, le [plug-in Autodesk Maya](https://github.com/Azure/azure-batch-maya) utilise l’API Python Batch pour appeler Batch, la création et la gestion de pools, l’envoi de travaux et de tâches, et la surveillance de l’état.
* Outils de ligne de commande :
  * Vous pouvez utiliser la [ligne de commande Azure](/cli/azure/) ou [Azure PowerShell](/powershell/azure/) pour écrire un script d’utilisation de Batch.
  * En particulier, la prise en charge des modèles CLI Batch facilite considérablement la création de pools et l’envoi de travaux.
* Interfaces utilisateur :
  * [Batch Explorer](https://github.com/Azure/BatchExplorer) est un outil client multiplateforme qui permet également de gérer et de surveiller des comptes Batch, mais qui fournit des fonctionnalités plus riches que l’interface utilisateur du portail Azure.  Un ensemble de modèles de pool et de travail personnalisés pour chaque application prise en charge est fourni et peut être utilisé pour créer facilement des pools et envoyer des travaux.
  * Le portail Azure peut être utilisé pour gérer et surveiller Azure Batch.
* Plug-ins d’application cliente :
  * Les plug-ins disponibles permettent d’utiliser le rendu Batch directement dans les applications clientes de conception et de modélisation. Les plug-ins appellent principalement l’application Batch Explorer avec des informations contextuelles sur le modèle 3D actif.
  * Les plug-ins suivants sont disponibles :
    * [Azure Batch pour Maya](https://github.com/Azure/azure-batch-maya)
    * [3ds Max](https://github.com/Azure/azure-batch-rendering/tree/master/plugins/3ds-max)
    * [Blender](https://github.com/Azure/azure-batch-rendering/tree/master/plugins/blender)

## <a name="getting-started-with-azure-batch-rendering"></a>Prise en main du rendu Azure Batch

Consultez les didacticiels d’introduction suivants pour essayer le rendu Azure Batch :

* [Utiliser Batch Explorer pour effectuer le rendu d’une scène de Blender](./tutorial-rendering-batchexplorer-blender.md)
* [Utiliser l’interface de ligne de commande Batch pour effectuer le rendu d’une scène d’Autodesk 3ds Max](./tutorial-rendering-cli.md)

## <a name="next-steps"></a>Étapes suivantes

Déterminez la liste des applications de rendu et les versions incluses dans les images de machine virtuelle de la Place de marché Azure dans [cet article](./batch-rendering-applications.md).

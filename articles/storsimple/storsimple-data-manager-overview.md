---
title: Vue d’ensemble de Microsoft Azure StorSimple Data Manager | Microsoft Docs
description: Découvrez la solution StorSimple Data Manager et comment utiliser ce service pour écrire des applications s’appuyant sur les données de StorSimple et d’autres services Azure.
services: storsimple
documentationcenter: NA
author: vidarmsft
manager: syadav
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 05/21/2018
ms.author: alkohli
ms.openlocfilehash: d683f49cadb384ef59d3bae819156733691813cd
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88183494"
---
# <a name="storsimple-data-manager-solution-overview"></a>Vue d’ensemble de la solution StorSimple Data Manager

## <a name="overview"></a>Vue d’ensemble

Microsoft Azure StorSimple utilise le stockage cloud pour étendre la solution locale, et hiérarchise automatiquement les données sur le stockage local et le cloud. Les données sont stockées dans le cloud dans un format dédupliqué et compressé, de façon à maximiser l’efficacité et à réduire les coûts. Étant au format StorSimple, elles ne sont pas directement utilisables par d’autres applications cloud.

StorSimple Data Manager permet d’y accéder et de les utiliser en toute simplicité dans le cloud. Pour cela, il transforme le format StorSimple en fichiers et en objets blob natifs utilisables avec d’autres services, par exemple, Azure Media Services, Azure HDInsights et Azure Machine Learning.

Cet article offre une vue d’ensemble de la solution StorSimple Data Manager. Il explique également comment utiliser ce service pour écrire des applications qui utilisent des données de StorSimple et les autres services Azure dans le cloud.

## <a name="how-it-works"></a>Fonctionnement

Le service StorSimple Data Manager identifie les données StorSimple dans le cloud sur un appareil local de la série StorSimple 8000. Les données StorSimple dans le cloud sont dédupliquées et compressées au format StorSimple. Le service Data Manager fournit des API pour extraire les données dans ce format et les convertir dans d’autres formats, par exemple des objets blob Azure et des fichiers Azure Files. Ces données transformées sont alors facilement consommées par Azure HDInsight et Azure Media Services. La transformation de données permet donc à ces services d’exploiter les données StorSimple converties sur l’appareil local de la série StorSimple 8000. Ce flux est illustré par le diagramme suivant :

![Diagramme général](./media/storsimple-data-manager-overview/storsimple-data-manager-overview2.png)


## <a name="data-manager-use-cases"></a>Cas d’usage de Data Manager

Vous pouvez utiliser Data Manager avec Azure Functions, Azure Automation et Azure Data Factory pour exécuter des workflows sur vos données au fur et à mesure de leur ajout à StorSimple. Vous avez la possibilité de traiter le contenu multimédia stocké sur StorSimple avec Azure Media Services, d’exécuter un algorithme de Machine Learning sur ces données ou de monter un cluster Hadoop pour analyser les données stockées sur StorSimple. Grâce au large éventail des services disponibles sur Azure et aux données présentes sur StorSimple, vous pourrez exploiter toute la puissance de vos données.


## <a name="region-availability"></a>Disponibilité des régions

StorSimple Data Manager est disponible dans les sept régions suivantes :

 - Asie Sud-Est
 - USA Est
 - USA Ouest
 - USA Ouest 2
 - Centre-USA Ouest
 - Europe Nord
 - Europe Ouest

Toutefois, StorSimple Data Manager peut permettre de transformer les données dans les régions suivantes : 

![Régions disponibles pour les données](./media/storsimple-data-manager-overview/data-manager-job-definition-different-regions-m.png)

La seconde liste est plus longue, car le déploiement de ressources dans l’une des régions de la première est capable d’élargir le processus de transformation aux régions de la seconde. Par conséquent, tant que vos données se trouvent dans l’une des 19 régions, vous pouvez les transformer à l’aide de ce service.


## <a name="choosing-a-region"></a>Choisir une région

Nous vous recommandons :
 - que votre compte de stockage source (associé à votre appareil StorSimple) se trouve dans la même région Azure que votre compte de stockage cible (où vous souhaitez placer les données au format natif) ;
 - de placer votre service Data Manager et la définition de la tâche dans la région qui contient le compte de stockage StorSimple. Si ce n’est pas possible, placez Data Manager dans la région Azure le plus proche, puis créez la définition de la tâche dans la même région que votre compte de stockage StorSimple. 

    Si votre compte de stockage StorSimple ne se trouve pas dans l’une des 26 régions qui prennent en charge la création de définitions de tâches, nous vous recommandons de ne pas exécuter StorSimple Data Manager, car vous observerez des latences importantes et d’éventuels frais de sortie.
    
Microsoft s’efforce de faire en sorte que les services Azure soient toujours disponibles dans toutes les régions. Cependant, de brèves interruptions de service inopinées peuvent se produire dans une région donnée. En pareil cas, vous pouvez établir un gestionnaire de données et une définition de tâche dans une région qui n’est pas affectée par l’interruption et exécuter la tâche de transformation. Si ce scénario vous expose à une latence supplémentaire, il peut s’agir néanmoins de votre stratégie de récupération dans les rares cas où une interruption régionale se produit.

## <a name="security-considerations"></a>Considérations relatives à la sécurité

StorSimple Data Manager a besoin de la clé de chiffrement des données de service pour transformer le format StorSimple en format natif. Elle est générée lors de l’inscription du premier appareil auprès du service StorSimple. Pour plus d’informations sur cette clé, consultez la page [Sécurité StorSimple](storsimple-8000-security.md).

La clé de chiffrement de données de service fournie en entrée est stockée dans un coffre de clés généré à la création d’un service Data Manager. Le coffre se trouve dans la même région Azure que votre service StorSimple Data Manager. Cette clé est supprimée lors de la suppression du service Data Manager.

Cette clé est utilisée par les ressources de calcul pour effectuer la transformation. Ces ressources se trouvent dans la même région Azure que la définition de la tâche. Cette région n’est pas nécessairement identique à celle du service Data Manager.

Si la région de votre service Data Manager est différente de celle de votre définition de tâche, il est important que vous sachiez quelles données/métadonnées se trouvent dans chacune d’entre elles. Le diagramme suivant illustre l’effet produit lorsque la région de Data Manager est différente de celle de la définition de la tâche.

![Service et définition de la tâche dans des régions différentes](./media/storsimple-data-manager-overview/data-manager-job-different-regions.png)

## <a name="managing-personal-information"></a>Gestion des informations personnelles

StorSimple Data Manager ne collecte ni ne montre aucune information personnelle. Pour plus d’informations, consultez la [Politique de confidentialité Microsoft sur le Centre de gestion de la confidentialité](https://www.microsoft.com/trustcenter).

## <a name="known-limitations"></a>Limites connues

Le service présente actuellement les limites suivantes :
- Actuellement, StorSimple Data Manager ne fonctionne pas avec des volumes chiffrés BitLocker. Si vous tentez d’exécuter le service avec un lecteur chiffré, vous constaterez des échecs des travaux.
- Certaines métadonnées de fichiers (dont les ACL) ne sont pas conservées dans les données transformées.
- Ce service fonctionne uniquement avec des volumes NTFS.
- Les chemins d’accès des fichiers doivent compter moins de 256 caractères, sans quoi le travail échoue.

## <a name="next-steps"></a>Étapes suivantes

[Utilisez l’interface utilisateur de StorSimple Data Manager pour transformer vos données](storsimple-data-manager-ui.md).

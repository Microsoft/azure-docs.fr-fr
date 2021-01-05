---
title: Fonctionnalités de rendu
description: Les fonctionnalités Standard d’Azure Batch sont utilisées pour exécuter des applications et des charges de travail de rendu. Batch inclut des fonctionnalités spécifiques qui prennent en charge les charges de travail de rendu.
author: mscurrell
ms.author: markscu
ms.date: 08/02/2018
ms.topic: how-to
ms.openlocfilehash: 77a6ec54495b394c597f6d6b4ddb5f5fe3285550
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92107468"
---
# <a name="azure-batch-rendering-capabilities"></a>Fonctionnalités de rendu Azure Batch

Les fonctionnalités standard d’Azure Batch sont utilisées pour exécuter des applications et des charges de travail de rendu. Batch comprend également des fonctionnalités qui prennent en charge les charges de travail de rendu.

Pour une présentation des concepts Batch, tels que les pools, les travaux et les tâches, consultez [cet article](./batch-service-workflow-features.md).

## <a name="batch-pools"></a>Pools Batch

### <a name="rendering-application-installation"></a>Installation de l’application de rendu

Pour spécifier une image de machine virtuelle de rendu issue de la Place de marché Azure dans la configuration du pool, seules les applications préinstallées doivent être utilisées.

Il existe une image Windows 2016 et une image CentOS.  Dans la [Place de marché Azure](https://azuremarketplace.microsoft.com), vous pouvez rechercher les images de machine virtuelle à l’aide de la requête « batch rendering ».

Pour un exemple de configuration de pool, consultez le [tutoriel sur le rendu Azure CLI](./tutorial-rendering-cli.md).  Le portail Azure et Batch Explorer fournissent des outils GUI permettant de sélectionner une image de machine virtuelle de rendu lorsque vous créez un pool.  Si vous utilisez une API Batch, spécifiez les valeurs de propriété suivantes pour [ImageReference](/rest/api/batchservice/pool/add#imagereference) quand vous créez un pool :

| Serveur de publication | Offre | Sku | Version |
|---------|---------|---------|--------|
| lot | rendering-centos73 | rendu | latest |
| lot | rendering-windows2016 | rendu | latest |

D’autres options sont disponibles si des applications supplémentaires sont nécessaires sur les machines virtuelles du pool :

* Image personnalisée de Shared Image Gallery :
  * À l’aide de cette option, vous pouvez configurer votre machine virtuelle avec les applications et versions spécifiques dont vous avez besoin. Pour plus d’informations, voir [Créer un pool avec Shared Image Gallery](batch-sig-images.md). Autodesk et Chaos Group ont modifié les logiciels Arnold et V-Ray de manière à effectuer la validation par rapport à un service de gestion des licences Azure Batch. Veillez à avoir les versions de ces applications comprenant cette prise en charge, sinon le service de licence avec paiement à l’utilisation ne fonctionnera pas. Les versions actuelles de Maya et de 3ds Max ne nécessitent pas de serveur de licences lorsqu’elles sont exécutées sans périphérique de contrôle (en mode batch ou ligne de commande). Contactez le support Azure si vous ne savez pas comment utiliser cette option.
* [Packages d’applications](./batch-application-packages.md) :
  * Empaquetez les fichiers d’application dans un ou plusieurs fichiers ZIP, chargez-les via le portail Azure et spécifiez le package dans la configuration du pool. Lorsque des machines virtuelles de pool sont créées, les fichiers ZIP sont téléchargés et les fichiers extraits.
* Les fichiers de ressources :
  * Les fichiers d’application sont chargés dans le Stockage Blob Azure, et vous spécifiez des références de fichier dans la [tâche de démarrage du pool](/rest/api/batchservice/pool/add#starttask). Lorsque des machines virtuelles de pool sont créées, les fichiers de ressources sont téléchargés sur chaque machine virtuelle.

### <a name="pay-for-use-licensing-for-pre-installed-applications"></a>Licences avec paiement à l’utilisation pour les applications préinstallées

Les applications qui seront utilisées et pour lesquelles il existe des frais de licence doivent être spécifiées dans la configuration du pool.

* Spécifiez la propriété `applicationLicenses` lors de la [création d’un pool](/rest/api/batchservice/pool/add#request-body).  Les valeurs suivantes peuvent être spécifiées dans le tableau de chaînes : « vray », « arnold », « 3dsmax », « maya ».
* Lorsque vous spécifiez une ou plusieurs applications, leur coût est ajouté à celui des machines virtuelles.  Les prix des applications sont listés dans la [page des prix Azure Batch](https://azure.microsoft.com/pricing/details/batch/#graphic-rendering).

> [!NOTE]
> Si, au lieu de cela, vous vous connectez à un serveur de licences pour utiliser les applications de rendu, ne spécifiez pas la propriété `applicationLicenses`.

Vous pouvez utiliser le portail Azure ou Batch Explorer pour sélectionner des applications et afficher leur prix.

Si vous tentez d’utiliser une application alors que celle-ci n’a pas été spécifiée dans la propriété `applicationLicenses` de la configuration du pool ou qu’elle n’atteint pas un serveur de licences, l’exécution de l’application échoue avec une erreur de licence et un code de sortie différent de zéro.

### <a name="environment-variables-for-pre-installed-applications"></a>Variables d’environnement pour applications préinstallées

Si vous souhaitez créer la ligne de commande pour les tâches de rendu, vous devez spécifier l’emplacement d’installation des fichiers exécutables des applications de rendu.  Les variables d’environnement système ont été créées dans les images de machine virtuelle Place de marché Azure, qui peuvent être utilisées pour ne pas avoir à spécifier des chemins.  Ces variables d’environnement viennent s’ajouter aux [variables d’environnement standard de Batch](./batch-compute-node-environment-variables.md), qui sont créées pour chaque tâche.

|Application|Exécutable de l’application|Variable d’environnement|
|---------|---------|---------|
|Autodesk 3ds Max 2018|3dsmaxcmdio.exe|3DSMAX_2018_EXEC|
|Autodesk 3ds Max 2019|3dsmaxcmdio.exe|3DSMAX_2019_EXEC|
|Autodesk Maya 2017|render.exe|MAYA_2017_EXEC|
|Autodesk Maya 2018|render.exe|MAYA_2018_EXEC|
|Chaos Group V-Ray (version autonome)|vray.exe|VRAY_3.60.4_EXEC|
Ligne de commande Arnold 2017|kick.exe|ARNOLD_2017_EXEC|
|Ligne de commande Arnold 2018|kick.exe|ARNOLD_2018_EXEC|
|Blender|blender.exe|BLENDER_2018_EXEC|

### <a name="azure-vm-families"></a>Familles de machines virtuelles Azure

Comme pour les autres charges de travail, les exigences système des applications de rendu varient, et les exigences de performances varient selon les travaux et les projets.  De nombreuses familles de machines virtuelles sont disponibles dans Azure, selon vos besoins : faible coût, meilleur rapport prix/performances, meilleures performances, etc.
Certaines applications de rendu, telles qu’Arnold, sont basées sur le processeur. D’autres, telles que V-Ray et Blender Cycles, peuvent utiliser des UC et/ou des GPU.
Pour obtenir la description des familles de machines virtuelles disponibles et de la taille des machines virtuelles, consultez [Types et tailles des machines virtuelles](../virtual-machines/sizes.md).

### <a name="low-priority-vms"></a>Machines virtuelles de faible priorité

Comme avec les autres charges de travail, les machines virtuelles de faible priorité peuvent être utilisées dans des pools Batch pour le rendu.  Les machines virtuelles de faible priorité fonctionnent comme des machines virtuelles dédiées normales, à la différence qu’elles utilisent la capacité Azure excédentaire et peuvent faire l’objet d’une remise importante.  La contrepartie à l’utilisation de machines virtuelles de faible priorité est que ces machines virtuelles risquent de ne pas pouvoir être réaffectées ou d’être reportées à tout moment, selon la capacité disponible. Pour cette raison, les machines virtuelles de faible priorité ne sont pas adaptées à tous les travaux de rendu. Par exemple, si des images mettent plusieurs heures à s’afficher, il est évidemment inacceptable d’interrompre et de redémarrer le rendu de ces images en raison de machines virtuelles anticipées.

Pour plus d’informations sur les caractéristiques des machines virtuelles de faible priorité et les différentes façons de les configurer à l’aide de Batch, consultez [Utiliser des machines virtuelles de faible priorité avec Batch](./batch-low-pri-vms.md).

## <a name="jobs-and-tasks"></a>Travaux et tâches

Aucune prise en charge de rendu n’est nécessaire pour les travaux et les tâches.  Le principal élément de configuration est la ligne de commande de tâche, qui doit référencer l’application nécessaire.
Lorsque des images de machine virtuelle issues de la Place de marché Azure sont utilisées, une bonne pratique consiste à utiliser les variables d’environnement pour spécifier le chemin et l’exécutable de l’application.

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir des exemples de rendu Batch, suivez les deux tutoriels suivants :

* [Rendu à l’aide d’Azure CLI](./tutorial-rendering-cli.md)
* [Rendu à l’aide de Batch Explorer](./tutorial-rendering-batchexplorer-blender.md)
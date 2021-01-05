---
title: Applications de rendu
description: Vous pouvez utiliser n’importe quelle application de rendu avec Azure Batch. Toutefois, les images de machine virtuelle de la Place de marché Azure sont disponibles avec des applications courantes préinstallées.
ms.date: 09/19/2019
ms.topic: how-to
ms.openlocfilehash: f876671c6ac16137c2c1cf8d40bd70860c06975b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85960484"
---
# <a name="pre-installed-applications-on-rendering-vm-images"></a>Applications de rendu d’images de machine virtuelle préinstallées

Vous pouvez utiliser n’importe quelle application de rendu avec Azure Batch. Toutefois, les images de machine virtuelle de la Place de marché Azure sont disponibles avec des applications courantes préinstallées.

Le cas échéant, une licence de paiement à l’utilisation est disponible pour les applications de rendu préinstallées. Quand un pool Batch est créé, les applications requises peuvent être spécifiées et les frais de la machine virtuelle et des applications sont facturés à la minute. Les prix des applications sont listés dans la [page des prix Azure Batch](https://azure.microsoft.com/pricing/details/batch/#graphic-rendering).

Certaines applications prennent en charge uniquement Windows, mais la plupart sont prises en charge sur Windows et Linux.

## <a name="applications-on-centos-7-rendering-images"></a>Applications sur les images de rendu CentOS 7

La liste suivante s’applique aux images de rendu CentOS 7.6, version 1.1.6.

* Autodesk Maya I/O 2017 Update 5 (cut 201708032230)
* Autodesk Maya I/O 2018 Update 2 (cut 201711281015)
* Autodesk Maya I/O 2019 Update 1
* Autodesk Arnold pour Maya 2017 (Arnold version 5.3.1.1) MtoA-3.2.1.1-2017
* Autodesk Arnold pour Maya 2018 (Arnold version 5.3.1.1) MtoA-3.2.1.1-2018
* Autodesk Arnold pour Maya 2019 (Arnold version 5.3.1.1) MtoA-3.2.1.1-2019
* Chaos Group V-Ray for Maya 2017 (version 3.60.04)
* Chaos Group V-Ray for Maya 2018 (version 3.60.04)
* Blender (2.68)
* Blender (2.8)

## <a name="applications-on-latest-windows-server-2016-rendering-images"></a>Applications sur les dernières images de rendu Windows Server 2016

La liste suivante s’applique aux images de rendu Windows Server 2016 version 1.3.8.

* Autodesk Maya I/O 2017 Update 5 (version 17.4.5459)
* Autodesk Maya I/O 2018 Update 6 (version 18.4.0.7622)
* Autodesk Maya I/O 2019
* Autodesk 3ds Max I/O 2018 Update 4 (version 20.4.0.4254)
* Autodesk 3ds Max I/O 2019 Update 1 (version 21.2.0.2219)
* Autodesk 3ds Max I/O 2020 Update 2
* Autodesk Arnold pour Maya 2017 (Arnold version 5.3.0.2) MtoA-3.2.0.2-2017
* Autodesk Arnold pour Maya 2018 (Arnold version 5.3.0.2) MtoA-3.2.0.2-2018
* Autodesk Arnold pour Maya 2019 (Arnold version 5.3.0.2) MtoA-3.2.0.2-2019
* Autodesk Arnold pour 3ds Max 2018 (Arnold version 5.3.0.2) (version 1.2.926)
* Autodesk Arnold pour 3ds Max 2019 (Arnold version 5.3.0.2) (version 1.2.926)
* Autodesk Arnold pour 3ds Max 2020 (Arnold version 5.3.0.2) (version 1.2.926)
* Chaos Group V-Ray for Maya 2017 (version 4.12.01)
* Chaos Group V-Ray for Maya 2018 (version 4.12.01)
* Chaos Group V-Ray for Maya 2019 (version 4.04.03)
* Chaos Group V-Ray for 3ds Max 2018 (version 4.20.01)
* Chaos Group V-Ray for 3ds Max 2019 (version 4.20.01)
* Chaos Group V-Ray for 3ds Max 2020 (version 4.20.01)
* Blender (2.79)
* Blender (2.80)
* AZ 10

> [!IMPORTANT]
> Pour exécuter V-Ray avec Maya en dehors des [modèles d’extension Azure batch](https://github.com/Azure/batch-extension-templates), démarrez `vrayses.exe` avant d’exécuter le rendu. Pour démarrer vrayses.exe en dehors des modèles, vous pouvez utiliser la commande suivante `%MAYA_2017%\vray\bin\vrayses.exe"`.
>
> Pour obtenir un exemple, voir la tâche de démarrage du [modèle Maya et V-Ray](https://github.com/Azure/batch-extension-templates/blob/master/templates/maya/render-vray-windows/pool.template.json) sur GitHub.

## <a name="applications-on-previous-windows-server-2016-rendering-images"></a>Applications sur les images de rendu Windows Server 2016 précédentes

La liste suivante s’applique aux images de rendu Windows Server 2016 version 1.3.7.

* Autodesk Maya I/O 2017 Update 5 (version 17.4.5459)
* Autodesk Maya I/O 2018 Update 4 (version 18.4.0.7622)
* Autodesk 3ds Max I/O 2019 Update 1 (version 21.2.0.2219)
* Autodesk 3ds Max I/O 2018 Update 4 (version 20.4.0.4254)
* Autodesk Arnold for Maya 2017 (Arnold version 5.2.0.1) MtoA-3.1.0.1-2017
* Autodesk Arnold for Maya 2018 (Arnold version 5.2.0.1) MtoA-3.1.0.1-2018
* Autodesk Arnold pour 3ds Max 2018 (Arnold version 5.0.2.4) (version 1.2.926)
* Autodesk Arnold pour 3ds Max 2019 (Arnold version 5.0.2.4) (version 1.2.926)
* Chaos Group V-Ray for Maya 2018 (version 3.52.03)
* Chaos Group V-Ray for 3ds Max 2018 (version 3.60.02)
* Chaos Group V-Ray for Maya 2019 (version 3.52.03)
* Chaos Group V-Ray for 3ds Max 2019 (version 4.10.01)
* Blender (2.79)

> [!NOTE]
> Chaos Group V-Ray for 3ds Max 2019 (version 4.10.01) introduit des changements majeurs de V-ray. Pour utiliser la version précédente (version 3.60.02), utilisez des nœuds de rendu Windows Server 2016 version 1.3.2.

## <a name="next-steps"></a>Étapes suivantes

Pour utiliser les images de machine virtuelle de rendu, celles-ci doivent être spécifiées dans la configuration du pool au moment de la création du pool. Consultez les [capacités du pool Batch pour le rendu](./batch-rendering-functionality.md#batch-pools).

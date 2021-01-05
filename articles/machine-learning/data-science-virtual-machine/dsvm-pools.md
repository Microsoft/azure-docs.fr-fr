---
title: Pools partagés
titleSuffix: Azure Data Science Virtual Machine
description: Découvrez comment créer et déployer un pool partagé de DSVM (Data Science Virtual Machine) en tant que ressource partagée pour une équipe.
keywords: formation approfondie, IA, outils de science des données, machine virtuelle de science des données, analyse géospatiale, processus de science des données en équipe
services: machine-learning
ms.service: machine-learning
ms.subservice: data-science-vm
author: vijetajo
ms.author: vijetaj
ms.topic: conceptual
ms.date: 12/10/2018
ms.openlocfilehash: eb1242189f4c5a38421a7f44e8f5e738c44970b6
ms.sourcegitcommit: e7152996ee917505c7aba707d214b2b520348302
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/20/2020
ms.locfileid: "97705521"
---
# <a name="create-a-shared-pool-of-data-science-virtual-machines"></a>Création d’un pool partagé de Data Science Virtual Machines

Dans cet article, vous allez apprendre à créer un pool partagé de DSVM (Data Science Virtual Machine) pour une équipe. L’utilisation d’un pool partagé comporte certains avantages : meilleure utilisation des ressources, partage et collaboration facilités, et gestion plus efficace des ressources des machines virtuelles DSVM.

Vous pouvez utiliser plusieurs méthodes et technologies pour créer un pool de DSVM. Cet article se concentre sur les pools de machines virtuelles interactives. La capacité de calcul Azure Machine Learning est un autre exemple d'infrastructure de calcul managée. Pour plus d’informations, consultez [Créer un cluster de calcul](../how-to-create-attach-compute-cluster.md).

## <a name="interactive-vm-pool"></a>Pool de machines virtuelles interactives

Un pool de machines virtuelles interactives qui sont partagées par l’équipe complète AI/science des données permet aux utilisateurs de se connecter à une instance disponible de la DSVM au lieu de disposer d’une instance dédiée pour chaque ensemble d’utilisateurs. Cette configuration permet une meilleure disponibilité et une utilisation plus efficace des ressources.

Vous utilisez la technologie des You use [groupe de machines virtuelles identiques Azure](../../virtual-machine-scale-sets/index.yml) pour créer un pool de machines virtuelles interactives. Les groupes identiques vous permettent de créer et de gérer un groupe de machines virtuelles identiques, disposant d’une charge équilibrée et d’une mise à l’échelle automatique.

L’utilisateur se connecte à l’adresse IP ou DNS du pool principal. La session est automatiquement acheminée vers une DSVM disponible dans le groupe identique. Comme les utilisateurs souhaitent un environnement cohérent et familier quelle que soit la machine virtuelle à laquelle ils se connectent, toutes les instances de la machine virtuelle dans le groupe identique montent un lecteur réseau partagé, comme un partage Azure Files ou NFS (Network File System). L’espace de travail partagé de l’utilisateur est normalement conservé dans le magasin de fichiers partagé qui est monté sur chacune des instances.

Un exemple de modèle Azure Resource Manager qui permet de créer un groupe identique comptant des instances de DSVM Ubuntu se trouve dans [GitHub](https://raw.githubusercontent.com/Azure/DataScienceVM/master/Scripts/CreateDSVM/Ubuntu/dsvm-vmss-cluster.json). Vous trouverez un exemple de [fichier de paramètres](https://raw.githubusercontent.com/Azure/DataScienceVM/master/Scripts/CreateDSVM/Ubuntu/dsvm-vmss-cluster.parameters.json) pour le modèle Azure Resource Manager au même emplacement.

Vous pouvez créer le groupe identique à partir du modèle Azure Resource Manager en spécifiant des valeurs appropriées pour le fichier de paramètres dans l’interface Azure CLI :

```azurecli-interactive
az group create --name [[NAME OF RESOURCE GROUP]] --location [[ Data center. For eg: "West US 2"]
az deployment group create --resource-group  [[NAME OF RESOURCE GROUP ABOVE]]  --template-uri https://raw.githubusercontent.com/Azure/DataScienceVM/master/Scripts/CreateDSVM/Ubuntu/dsvm-vmss-cluster.json --parameters @[[PARAMETER JSON FILE]]
```

Les commandes précédentes supposent que vous disposez :

* D’une copie du fichier de paramètres avec les valeurs spécifiées pour votre instance de groupe identique.
* Du nombre d’instances de machine virtuelle.
* De pointeurs vers le partage de fichiers Azure.
* Des informations d’identification pour le compte de stockage qui sera monté sur chaque machine virtuelle.

Le fichier de paramètres est référencé en local dans les commandes. Vous pouvez également transmettre des paramètres inclus ou inviter à les entrer dans votre script.  

Le modèle présenté ci-dessus active le protocole SSH et le port JupyterHub du groupe identique front-end au pool de machines virtuelles DSVM Ubuntu back-end. En tant qu’utilisateur, vous vous connectez normalement à la machine virtuelle sur SSH (Secure Shell) ou JupyterHub. Comme la taille des instances de machine virtuelle peut être augmentée ou diminuée de façon dynamique, tout état doit être enregistré dans le partage Azure Files monté. La même approche peut être utilisée pour créer un pool de DSVM Windows.

Le [script permettant de monter le partage Azure Files](https://raw.githubusercontent.com/Azure/DataScienceVM/master/Extensions/General/mountazurefiles.sh) est également disponible dans le référentiel GitHub des machines virtuelles DSVM Azure. Le script monte le partage de fichiers Azure au niveau du point de montage spécifié dans le fichier de paramètres. Le script crée également des liens symboliques pour le lecteur monté dans le répertoire de base initial de l’utilisateur. Un répertoire de notebooks propre à l’utilisateur figurant dans le partage Azure Files est associé par un lien symbolique au répertoire `$HOME/notebooks/remote` ce qui permet aux utilisateurs d’accéder aux notebooks Jupyter, de les exécuter et de les enregistrer. Vous pouvez utiliser la même convention lorsque vous créez des utilisateurs supplémentaires sur la machine virtuelle pour pointer l’espace de travail Jupyter de chaque utilisateur vers le répertoire Azure Files partagé.

Le groupe de machines virtuelles identiques prend en charge la mise à l’échelle automatique. Vous pouvez définir des règles concernant ce qui déclenche la création d’instances supplémentaires ou la diminution du nombre d’instances. Par exemple, vous pouvez descendre en puissance jusqu’à ne plus avoir d’instances afin d’économiser sur les coûts d’utilisation de matériel cloud lorsque les machines virtuelles ne sont pas du tout utilisées. Les pages de documentation des groupes identiques de machines virtuelles fournissent des instructions détaillées de [mise à l’échelle automatique](../../virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview.md).

## <a name="next-steps"></a>Étapes suivantes

* [Configurer une identité commune](dsvm-common-identity.md)
* [Stocker en toute sécurité les informations d’identification pour accéder aux ressources cloud](dsvm-secure-access-keys.md)
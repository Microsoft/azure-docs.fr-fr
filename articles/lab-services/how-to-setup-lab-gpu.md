---
title: Configurer un laboratoire avec GPU dans Azure Lab Services | Microsoft Docs
description: Découvrez comment configurer un laboratoire avec des machines virtuelles de processeur graphique (GPU).
author: nicolela
ms.topic: article
ms.date: 06/26/2020
ms.author: nicolela
ms.openlocfilehash: 8293ed1bfb53895b9631d9730fb75a2364457180
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96452373"
---
# <a name="set-up-a-lab-with-gpu-virtual-machines"></a>Configurer un laboratoire avec des machines virtuelles GPU

Cet article explique comment effectuer les tâches suivantes :

- Choisir entre les processeurs graphiques (GPU) de *visualisation* et de *calcul*.
- Vérifier que les pilotes GPU appropriés sont installés.

## <a name="choose-between-visualization-and-compute-gpu-sizes"></a>Choisir entre les tailles de GPU pour la visualisation et le calcul
Sur la première page de l’Assistant Création de laboratoire, dans la liste déroulante **Quelle taille de machine virtuelle vous faut-il ?** , sélectionnez la taille des machines virtuelles nécessaires pour votre classe.  

![Capture d’écran du volet « Nouvel atelier » où sélectionner une taille de machine virtuelle](./media/how-to-setup-gpu/lab-gpu-selection.png)

Au cours de ce processus, vous avez la possibilité de sélectionner des GPU de **Visualisation** ou **Calcul**.  Il est important de choisir le type de GPU en fonction du logiciel que vos étudiants vont utiliser.  

Comme décrit dans le tableau suivant, la taille de GPU pour le *calcul* est destinée aux applications nécessitant beaucoup de ressources de calcul.  Par exemple, le [Deep Learning dans le type de classe de traitement en langage naturel](./class-type-deep-learning-natural-language-processing.md) utilise la **petite taille de GPU (calcul)** .  Le GPU de calcul convient pour ce type de classe, car les étudiants utilisent des infrastructures et des outils de Deep Learning fournis par l’image de [Data Science Virtual Machine](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-dsvm.ubuntu-1804) pour former des modèles de Deep Learning avec de grands ensembles de données.

| Taille | Cœurs | RAM | Description | 
| ---- | ----- | --- | ----------- | 
| GPU de petite taille (calcul) | -&nbsp;6&nbsp;cœurs<br>-&nbsp;56&nbsp;Go de&nbsp;RAM  | [Standard_NC6](../virtual-machines/nc-series.md) |Cette taille convient tout particulièrement aux applications qui nécessitent beaucoup de ressources informatiques comme l’intelligence artificielle (IA) et le Deep Learning. |

Les tailles de GPU pour la *visualisation* sont destinées aux applications nécessitant beaucoup de ressources graphiques.  Par exemple, le [type de classe d’ingénierie SOLIDWORKS](./class-type-solidworks.md) montre l’utilisation de la taille **GPU de petite taille (visualisation)** .  Le GPU de visualisation convient à ce type de classe, car les étudiants interagissent avec l’environnement de conception 3D assistée par ordinateur (CAO) de SOLIDWORKS pour la modélisation et la visualisation d’objets solides.

| Taille | Cœurs | RAM | Description | 
| ---- | ----- | --- | ----------- | 
| GPU de petite taille (visualisation) | -&nbsp;6&nbsp;cœurs<br>-&nbsp;56&nbsp;Go de&nbsp;RAM  | [Standard_NV6](../virtual-machines/nv-series.md) | Cette taille est plus appropriée pour la visualisation, le streaming, les jeux et l'encodage à distance à l'aide d’infrastructures tels qu'OpenGL et DirectX. |
| GPU de taille moyenne (visualisation) | -&nbsp;12&nbsp;cœurs<br>-&nbsp;112&nbsp;Go de&nbsp;RAM  | [Standard_NV12](../virtual-machines/nv-series.md?bc=%2fazure%2fvirtual-machines%2flinux%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Cette taille est plus appropriée pour la visualisation, le streaming, les jeux et l'encodage à distance à l'aide d’infrastructures tels qu'OpenGL et DirectX. |

> [!NOTE]
> Il se peut que certaines de ces tailles de machine virtuelle ne s’affichent pas dans la liste lors de la création d’un laboratoire de classe. La liste est remplie en fonction de la capacité actuelle de l’emplacement du laboratoire. Si le créateur du compte lab [permet aux créateurs de laboratoire de choisir un emplacement pour le laboratoire](allow-lab-creator-pick-lab-location.md), vous pouvez essayer de choisir un autre emplacement pour le laboratoire et voir si la taille de la machine virtuelle est disponible. Pour connaître la disponibilité des machines virtuelles, consultez [Disponibilité des produits par région](https://azure.microsoft.com/regions/services/?products=virtual-machines).

## <a name="ensure-that-the-appropriate-gpu-drivers-are-installed"></a>Vérifier que les pilotes GPU appropriés sont installés
Pour tirer parti des fonctionnalités GPU de vos machines virtuelles de labo, assurez-vous que les pilotes GPU appropriés sont installés.  Dans l’Assistant de création de laboratoire, lorsque vous sélectionnez une taille de machine virtuelle GPU, vous pouvez choisir l’option **Installer les pilotes GPU**.  

![Capture d’écran du « Nouveau labo » montrant l’option « Installer les pilotes GPU »](./media/how-to-setup-gpu/lab-gpu-drivers.png)

Comme indiqué dans l’image ci-dessus, cette option est activée par défaut, ce qui garantit que les *derniers* pilotes sont installés pour le type de GPU et d’image que vous avez sélectionnés.
- Lorsque vous sélectionnez une taille de GPU pour le *calcul*, les machines virtuelles de votre laboratoire sont alimentées par la GPU [NVIDIA Tesla K80](https://www.nvidia.com/content/dam/en-zz/Solutions/Data-Center/tesla-product-literature/Tesla-K80-BoardSpec-07317-001-v05.pdf).  Dans ce cas, les [pilotes d’architecture de calcul unifiée (CUDA)](http://developer.download.nvidia.com/compute/cuda/2_0/docs/CudaReferenceManual_2.0.pdf) les plus récents sont installés, ce qui permet un calcul hautes performances.
- Lorsque vous sélectionnez une taille de GPU pour la *visualisation*, les machines virtuelles de votre laboratoire sont alimentées par la GPU [NVIDIA Tesla M60](https://images.nvidia.com/content/tesla/pdf/188417-Tesla-M60-DS-A4-fnl-Web.pdf) et la technologie [GRID](https://www.nvidia.com/content/dam/en-zz/Solutions/design-visualization/solutions/resources/documents1/NVIDIA_GRID_vPC_Solution_Overview.pdf).  Dans ce cas, les derniers pilotes GRID sont installés, ce qui permet l’utilisation d’applications gourmandes en ressources graphiques.

### <a name="install-the-drivers-manually"></a>Pour installer manuellement les pilotes
Vous devrez peut-être installer une version de pilote autre que la plus récente.  Cette section montre comment installer manuellement les pilotes appropriés, selon que vous utilisez un GPU pour le *calcul* ou la *visualisation*.

#### <a name="install-the-compute-gpu-drivers"></a>Installer les pilotes GPU de calcul

Pour installer manuellement les pilotes pour la taille du GPU de calcul, procédez comme suit :

1. Dans l’Assistant Création de laboratoire, lorsque vous [créez votre labo](./how-to-manage-classroom-labs.md), désactivez le paramètre **Installer les pilotes GPU**.

1. Une fois votre laboratoire créé, connectez-vous au modèle de machine virtuelle pour installer les pilotes appropriés.

   ![Capture d’écran de la page de téléchargement des pilotes NVIDIA](./media/how-to-setup-gpu/nvidia-driver-download.png) 

   a. Dans votre navigateur, accédez à la [page de téléchargement des pilotes NVIDIA](https://www.nvidia.com/Download/index.aspx).  
   b. Définissez le **Type de produit** sur **Tesla**.  
   c. Définissez la **Série de produits** sur **K-series**.  
   d. Définissez le **Système d’exploitation** sur le type d’image de base que vous avez sélectionné lors de la création de votre laboratoire.  
   e. Définissez le **Kit d’outils CUDA** sur la version du pilote CUDA dont vous avez besoin.  
   f. Cliquez sur **Rechercher** pour chercher vos pilotes.  
   g. Sélectionnez **Télécharger** pour télécharger le programme d'installation.  
   h. Exécutez le programme d’installation afin que les pilotes soient installés sur le modèle de machine virtuelle.  
1. Vérifiez que les pilotes sont installés correctement en suivant les instructions de la section [Vérifier les pilotes installés](how-to-setup-lab-gpu.md#validate-the-installed-drivers). 
1. Une fois que vous avez installé les pilotes et les autres logiciels requis pour votre classe, sélectionnez **Publier** pour créer les machines virtuelles de vos étudiants.

> [!NOTE]
> Si vous utilisez une image Linux, après avoir téléchargé le programme d’installation, installez les pilotes en suivant les instructions fournies dans [Installer des pilotes CUDA sur Linux](../virtual-machines/linux/n-series-driver-setup.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json#install-cuda-drivers-on-n-series-vms).

#### <a name="install-the-visualization-gpu-drivers"></a>Installer les pilotes GPU de visualisation

Pour installer manuellement les pilotes pour la taille de GPU de visualisation, procédez comme suit :

1. Dans l’Assistant Création de laboratoire, lorsque vous [créez votre labo](./how-to-manage-classroom-labs.md), désactivez le paramètre **Installer les pilotes GPU**.
1. Une fois votre laboratoire créé, connectez-vous au modèle de machine virtuelle pour installer les pilotes appropriés.
1. Installez les pilotes GRID fournis par Microsoft sur le modèle de machine virtuelle en suivant les instructions selon votre système d’exploitation :
   -  [Pilotes NVIDIA GRID pour Windows](../virtual-machines/windows/n-series-driver-setup.md#nvidia-grid-drivers)
   -  [Pilotes NVIDIA GRID pour Linux](../virtual-machines/linux/n-series-driver-setup.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json#nvidia-grid-drivers)
  
1. Redémarrez le modèle de machine virtuelle.
1. Vérifiez que les pilotes sont installés correctement en suivant les instructions de la section [Vérifier les pilotes installés](how-to-setup-lab-gpu.md#validate-the-installed-drivers).
1. Une fois que vous avez installé les pilotes et les autres logiciels requis pour votre classe, sélectionnez **Publier** pour créer les machines virtuelles de vos étudiants.

### <a name="validate-the-installed-drivers"></a>Vérifier les pilotes installés
Cette section décrit comment vérifier que vos pilotes GPU ont bien été installés.

#### <a name="windows-images"></a>Images Windows
1.  Suivez les instructions de la section « Vérifier l’installation des pilotes » dans [installer les pilotes GPU NVIDIA sur les machines virtuelles de la série N exécutant Windows](../virtual-machines/windows/n-series-driver-setup.md#verify-driver-installation).
1.  Si vous utilisez un GPU de *visualisation*, vous pouvez également :
    - Afficher et ajuster les paramètres de votre GPU dans le panneau de configuration NVIDIA. Pour ce faire, dans le **Panneau de configuration Windows**, sélectionnez **Matériel et audio**, puis **Panneau de configuration NVIDIA**.

      ![Capture d’écran du panneau de configuration Windows montrant le lien du panneau de configuration NVIDIA](./media/how-to-setup-gpu/control-panel-nvidia-settings.png) 

     - Affichez les performances de votre GPU à l’aide du **Gestionnaire des tâches**.  Sélectionnez l’onglet **Performances**, puis sélectionnez l’option **GPU**.

       ![Capture d’écran montrant l’onglet performances du GPU dans le Gestionnaire des tâches](./media/how-to-setup-gpu/task-manager-gpu.png) 

      > [!IMPORTANT]
      > Les paramètres du panneau de configuration NVIDIA sont accessibles uniquement pour les GPU de *visualisation*.  Si vous tentez d’ouvrir le panneau de configuration NVIDIA pour un GPU de calcul, vous obtiendrez l’erreur suivante : « Les paramètres d’affichage NVIDIA ne sont pas disponibles.  Vous n'utilisez actuellement pas d'écran connecté à un processeur graphique NVIDA ».  De même, les informations de performances du GPU dans le gestionnaire des tâches sont fournies uniquement pour les GPU de visualisation.

#### <a name="linux-images"></a>Images Linux
Suivez les instructions de la section « Vérifier l’installation des pilotes » dans [installer les pilotes GPU NVIDIA sur les machines virtuelles de la série N exécutant Linux](../virtual-machines/linux/n-series-driver-setup.md#verify-driver-installation).

## <a name="next-steps"></a>Étapes suivantes
Voir les articles suivants :

- [Créer et gérer des labos](how-to-manage-classroom-labs.md)
- [Type de classe pour la conception assistée par ordinateur (CAO) SOLIDWORKS](class-type-solidworks.md)
- [Type de classe MATLAB (laboratoire de matrice)](class-type-matlab.md)
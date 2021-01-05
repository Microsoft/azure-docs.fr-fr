---
title: Tutoriel - Mettre à l’échelle un cloud privé
description: Dans ce tutoriel, vous utilisez le portail Azure pour mettre à l’échelle un cloud privé Azure VMware Solution.
ms.topic: tutorial
ms.date: 09/21/2020
ms.openlocfilehash: d49d973cc6d97280dc0c7ea6681f2602b871e1ba
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92791237"
---
# <a name="tutorial-scale-an-azure-vmware-solution-private-cloud"></a>Tutoriel : Mettre à l’échelle un cloud privé Azure VMware Solution

Pour tirer le meilleur parti de votre expérience de cloud privé Azure VMware Solution, mettez à l’échelle les clusters et les hôtes selon vos besoins pour les charges de travail planifiées. Vous pouvez mettre à l’échelle les clusters et les hôtes d’un cloud privé en fonction des besoins de la charge de travail de vos applications. Les limitations de performances et de disponibilité pour des services spécifiques doivent être traitées au cas par cas. Les limites de clusters et d’hôtes sont indiquées dans l’[article sur le concept de cloud privé](concepts-private-clouds-clusters.md).

Dans ce tutoriel, nous allons utiliser le portail Azure pour :

> [!div class="checklist"]
> * Ajouter un cluster à un cloud privé existant
> * Ajoutez des hôtes à un cluster existant

## <a name="prerequisites"></a>Prérequis

Un cloud privé pour suivre ce tutoriel. Si vous n’avez pas créé de cloud privé, utilisez le [tutoriel Créer un cloud privé](tutorial-create-private-cloud.md) pour en créer un. Configurez le réseau pour votre cloud privé VMware dans Azure afin de mettre en place le réseau virtuel requis.

## <a name="add-a-new-cluster"></a>Ajouter un nouveau cluster

1. Dans la page Vue d’ensemble d’un cloud privé existant, sous **Gérer** , sélectionnez **Mettre à l’échelle le cloud privé**. Ensuite, sélectionnez **+ Ajouter un cluster**.

   :::image type="content" source="./media/tutorial-scale-private-cloud/ss2-select-add-cluster.png" alt-text="Sélectionner Ajouter un cluster" border="true":::

1. Dans la page **Ajouter un cluster** , utilisez le curseur pour sélectionner le nombre d’hôtes. Sélectionnez **Enregistrer**.

   :::image type="content" source="./media/tutorial-scale-private-cloud/ss3-configure-new-cluster.png" alt-text="Dans la page Ajouter un cluster, utilisez le curseur pour sélectionner le nombre d’hôtes. Sélectionnez Enregistrer." border="true":::

   Le déploiement du nouveau cluster va commencer.

## <a name="scale-a-cluster"></a>Mettre à l’échelle un cluster 

1. Dans la page Vue d’ensemble d’un cloud privé existant, sélectionnez **Mettre à l’échelle le cloud privé** , puis sélectionnez l’icône de crayon pour modifier le cluster.

   :::image type="content" source="./media/tutorial-scale-private-cloud/ss4-select-scale-private-cloud-2.png" alt-text="Sélectionner Mettre à l’échelle le cloud privé dans Vue d’ensemble" border="true":::

1. Dans la page **Modifier le cluster** , utilisez le curseur pour sélectionner le nombre d’hôtes. Sélectionnez **Enregistrer**.

   :::image type="content" source="./media/tutorial-scale-private-cloud/ss5-scale-cluster.png" alt-text="Dans la page Modifier le cluster, utilisez le curseur pour sélectionner le nombre d’hôtes. Sélectionnez Enregistrer." border="true":::

   L’ajout des hôtes au cluster va commencer.

## <a name="next-steps"></a>Étapes suivantes

Si vous avez besoin d’un autre cloud privé Azure VMware Solution, [créez un autre cloud privé](tutorial-create-private-cloud.md) en respectant les mêmes prérequis réseau et les mêmes limites de clusters et d’hôtes.

<!-- LINKS - external-->

<!-- LINKS - internal -->

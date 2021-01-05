---
title: Créer une bibliothèque de contenu pour déployer des machines virtuelles dans Azure VMware Solution
description: Créez une bibliothèque de contenu pour déployer une machine virtuelle dans un cloud privé Azure VMware Solution.
ms.topic: how-to
ms.date: 09/21/2020
ms.openlocfilehash: 57db8ac4da8eef826d85feb11680c9c87823ee36
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92461683"
---
# <a name="create-a-content-library-to-deploy-vms-in-azure-vmware-solution"></a>Créer une bibliothèque de contenu pour déployer des machines virtuelles dans Azure VMware Solution

Une bibliothèque de contenu stocke et gère du contenu sous la forme d’éléments de bibliothèque. Un élément de bibliothèque unique se compose d’un ou de plusieurs fichiers que vous utilisez pour déployer des machines virtuelles. 

Dans cet article, nous allons parcourir la procédure de création d’une bibliothèque de contenu.  Ensuite, nous allons examiner le déploiement d’une machine virtuelle à l’aide d’une image ISO provenant de cette bibliothèque de contenu.

## <a name="prerequisites"></a>Prérequis

Pour suivre ce tutoriel, un segment de commutateur logique NSX-T et un service DHCP managé.  Pour plus d’informations, consultez l’article [Comment gérer le protocole DHCP dans Azure VMWare Solution](manage-dhcp.md).

## <a name="create-a-content-library"></a>Créer une bibliothèque de contenu

1. Dans le client vSphere local, sélectionnez **Menu > Bibliothèques de contenu**.

   ![Sélectionner Menu -> Bibliothèques de contenu](./media/content-library/vsphere-menu-content-libraries.png)

1. Sélectionnez le bouton **Ajouter** pour créer une bibliothèque de contenu.

   ![Sélectionnez le bouton Ajouter pour créer une bibliothèque de contenu.](./media/content-library/create-new-content-library.png)

1. Spécifiez un nom et confirmez l’adresse IP du serveur vCenter, puis sélectionnez **Suivant**.

   ![Spécifiez le nom et les notes de votre choix, puis sélectionnez Suivant.](./media/content-library/new-content-library-step1.png)

1. Sélectionnez la **Bibliothèque de contenu locale** , puis **Suivant**.

   ![Pour cet exemple, nous allons créer une bibliothèque de contenu locale, puis sélectionner Suivant.](./media/content-library/new-content-library-step2.png)

1. Sélectionnez le magasin de données qui stockera votre bibliothèque de contenu, puis sélectionnez **Suivant**.

   ![Sélectionnez le magasin de données dans lequel vous voulez héberger votre bibliothèque de contenu, puis sélectionnez Suivant.](./media/content-library/new-content-library-step3.png)

1. Examinez et vérifiez les paramètres de la bibliothèque de contenu, puis sélectionnez **Terminer**.

   ![Vérifiez vos paramètres, puis sélectionnez Terminer.](./media/content-library/new-content-library-step4.png)

## <a name="upload-an-iso-image-to-the-content-library"></a>Charger une image ISO dans la bibliothèque de contenu

Maintenant que la bibliothèque de contenu a été créée, vous pouvez ajouter une image ISO pour déployer une machine virtuelle sur un cluster de cloud privé. 

1. Dans le client vSphere, sélectionnez **Menu > Bibliothèques de contenu**.

1. Cliquez avec le bouton droit sur la bibliothèque de contenu que vous voulez utiliser pour la nouvelle image ISO, puis sélectionnez **Importer un élément**.

1. Importez un élément de bibliothèque pour la source en procédant de l’une des façons suivantes, puis sélectionnez **Importer**  :
   1. Sélectionnez « URL », puis fournissez une URL pour télécharger une image ISO.

   1. Sélectionnez **Fichier local** pour effectuer le chargement à partir de votre système local.

   > [!TIP]
   > Si vous le souhaitez, vous pouvez définir un nom et des notes d’élément personnalisés pour la Destination.

1. Ouvrez la bibliothèque, puis sélectionnez l’onglet **Autres types** pour vérifier que votre image ISO a été correctement chargée.


## <a name="deploy-a-vm-to-a-private-cloud-cluster"></a>Déployer une machine virtuelle sur un cluster de cloud privé

1. Dans le client vSphere, sélectionnez **Menu > Hôtes et clusters**.

1. Dans le volet gauche, développez l’arborescence, puis sélectionnez un cluster.

1. Sélectionnez **Actions > Nouvelle machine virtuelle**.

1. Suivez les étapes de l’Assistant, puis modifiez les paramètres de votre choix.

1. Sélectionnez **New CD/DVD Drive > Client Device > Content Library ISO File** (Nouveau lecteur CD/DVD > Appareil client > Fichier ISO de bibliothèque de contenu).

1. Sélectionnez le fichier ISO chargé à la section précédente, puis sélectionnez **OK**.

1. Cochez la case **Se connecter** afin que l’image ISO soit montée au moment de la mise sous tension.

1. Sélectionnez **Nouveau réseau > Sélectionner dans la liste déroulante > Parcourir**.

1. Sélectionnez le **commutateur logique (segment)** , puis sélectionnez **OK**.

1. Modifiez tout autre paramètre matériel, puis sélectionnez **Suivant**.

1. Vérifiez les paramètres, puis sélectionnez **Terminer**.


## <a name="next-steps"></a>Étapes suivantes

Si vous envisagez d’utiliser VMware HCX pour migrer des charges de travail de machine virtuelle vers votre cloud privé, utilisez la procédure [Déployer et configurer VMware HCX](tutorial-deploy-vmware-hcx.md).

<!-- LINKS - external-->

<!-- LINKS - internal -->

---
title: 'Démarrage rapide : Créer une instance Windows de Data Science Virtual Machine'
titleSuffix: Azure Data Science Virtual Machine
description: Configurez et créez une machine virtuelle pour la science des données sur Microsoft Azure pour vos besoins d’analyse et d’apprentissage automatique.
ms.service: machine-learning
ms.subservice: data-science-vm
author: gvashishtha
ms.author: gopalv
ms.topic: quickstart
ms.date: 12/31/2019
ms.openlocfilehash: e876ddc9b4df32cad5bbf15215da5ce2f912bba1
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97109723"
---
# <a name="quickstart-set-up-the-data-science-virtual-machine-for-windows"></a>Démarrage rapide : Configurer Data Science Virtual Machine pour Windows

Devenir opérationnel avec une machine virtuelle Windows Server 2019 Data Science Virtual Machine.

## <a name="prerequisite"></a>Configuration requise

Pour créer une machine virtuelle Windows Data Science Virtual Machine, vous devez disposer d’un abonnement Azure. [Essayez Azure gratuitement](https://azure.com/free).
Notez que les comptes gratuits Azure ne prennent pas en charge les références SKU de machines virtuelles où le GPU est activé.

## <a name="create-your-dsvm"></a>Créer votre DSVM

Pour créer une instance de DSVM

1. Accédez au [portail Azure](https://portal.azure.com). Si vous n’êtes pas encore connecté, vous pouvez être invité à vous connecter à votre compte Azure.
1. Recherchez la liste des machines virtuelles en tapant « data science virtual machine », puis en sélectionnant « Data Science Virtual Machine - Windows 2019 ».

1. Sélectionnez le bouton **Créer** en bas.

1. Vous devez être redirigé vers le panneau « Créer une machine virtuelle ».

1. Remplissez l’onglet **Informations de base** :
      * **Abonnement**: Si vous disposez de plusieurs abonnements, sélectionnez celui qui sera associé à la création et à la facturation de la machine. Vous devez disposer des privilèges de création de ressources pour cet abonnement.
      * **Groupe de ressources** : Créez un groupe ou sélectionnez-en un.
      * **Nom de la machine virtuelle** : Entrez le nom de la machine virtuelle. Voici comment il s’affichera dans votre portail Azure.
      * **Emplacement** : Sélectionnez le centre de données qui convient le mieux. Pour un accès réseau plus rapide, il s’agit du centre de données qui héberge la plupart de vos données ou du centre de données le plus proche de votre emplacement physique. Apprenez-en davantage sur les [régions Azure](https://azure.microsoft.com/global-infrastructure/regions/).
      * **Image** : Conservez la valeur par défaut.
      * **Size** : Cette valeur doit être renseignée automatiquement avec une taille appropriée pour les charges de travail générales. Découvrez-en plus sur les [tailles des machines virtuelles Windows dans Azure](../../virtual-machines/sizes.md).
      * **Nom d’utilisateur** : Entrez le nom d’utilisateur de l’administrateur. Il s’agit du nom d’utilisateur que vous utiliserez pour vous connecter à votre machine virtuelle. Il ne doit pas nécessairement être identique à votre nom d’utilisateur Azure.
      * **Mot de passe** : Entrez le mot de passe que vous utiliserez pour vous connecter à votre machine virtuelle.    
1. Sélectionnez **Revoir + créer**.
1. **Vérifier+créer**
   * Vérifiez que toutes les informations que vous avez saisies sont correctes. 
   * Sélectionnez **Create** (Créer).


> [!NOTE]
> * Vous ne payez pas de frais de licence pour le logiciel qui est préchargé sur la machine virtuelle. Vous payez le coût de calcul de la taille de serveur que vous avez choisie à l’étape de définition du paramètre **Taille**.
> * Le provisionnement prend 10 à 20 minutes. Vous pouvez voir l’état de votre machine virtuelle sur le portail Azure.

## <a name="access-the-dsvm"></a>Accéder à la DSVM

Une fois la machine virtuelle créée et provisionnée, suivez les étapes indiquées pour [vous connecter à votre machine virtuelle Azure](../../marketplace/azure-vm-create-using-approved-base.md). Utilisez les informations d’identification du compte administrateur que vous avez configurées à l’étape **de base** de la création d’une machine virtuelle. 

Vous êtes maintenant prêt à utiliser les outils qui sont installés et configurés sur la machine virtuelle. La plupart des outils sont accessibles par le biais des icônes du Bureau et des vignettes du menu **Démarrer**.

<a name="tools"></a>


## <a name="next-steps"></a>Étapes suivantes

* Explorez les outils sur la DSVM en ouvrant le menu **Démarrer**.
* Apprenez-en plus sur Azure Machine Learning en lisant [Qu’est-ce qu’Azure Machine Learning ?](../overview-what-is-azure-ml.md) et en suivant ces [tutoriels](../index.yml).
* Lisez l’article [Science des données avec une image Data Science Virtual Machine Windows dans Azure](./vm-do-ten-things.md)

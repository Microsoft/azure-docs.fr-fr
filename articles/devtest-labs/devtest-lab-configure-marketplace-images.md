---
title: Configuration des paramètres d’image Azure Marketplace dans Azure DevTest Labs
description: Configurer quelles images Azure Marketplace peuvent être utilisées lors de la création d’une machine virtuelle dans Azure DevTest Labs
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: a5158ed33bf253db1dbe0eb3232bc43d27ce15e7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86512433"
---
# <a name="configure-azure-marketplace-image-settings-in-azure-devtest-labs"></a>Configuration des paramètres d’image Azure Marketplace dans Azure DevTest Labs
DevTest Labs prend en charge la création de machines virtuelles basées sur des images Azure Marketplace en fonction de la manière dont vous avez configuré des images Azure Marketplace à utiliser dans votre laboratoire. Cet article montre comment spécifier, le cas échéant, les images Azure Marketplace qui peuvent être utilisées lors de la création de machines virtuelles dans un laboratoire. Cela garantit que votre équipe a accès uniquement aux images de la Place de marché dont elle a besoin. 

## <a name="specify-allowed-images-for-creating-vms"></a>Spécifier les images autorisées pour la création de machines virtuelles
Pour spécifier les images de la Place de marché Azure autorisées lors de la création d’une machine virtuelle, procédez comme suit. 

1. Connectez-vous au [portail Azure](https://go.microsoft.com/fwlink/p/?LinkID=525040).
2. Sélectionnez **Tous les services**, puis **DevTest Labs** dans la liste.
3. Dans la liste de laboratoires, sélectionnez votre laboratoire. 
4. Sur la page d’accueil de la page du laboratoire, sélectionnez **Configuration et stratégies**.
5. Sur la page **Configuration et stratégies** du laboratoire, sous **Bases de machine virtuelle**, sélectionnez **Images de la Place de marché**.
6. Spécifiez si vous souhaitez que toutes les images Azure Marketplace qualifiées puissent être utilisées comme base d’une nouvelle machine virtuelle. Si vous sélectionnez **Oui**, toutes les images Azure Marketplace répondant à tous les critères suivants sont autorisées dans le laboratoire :
   
   * L’image crée une seule machine virtuelle, **et**
   * L’image utilise Azure Resource Manager pour configurer des machines virtuelles, **et**
   * L’image ne nécessite pas l’achat d’un plan de licences supplémentaire
     
     Si vous ne souhaitez autoriser aucune image, ou vous souhaitez spécifier les images pouvant être utilisées, sélectionnez **Non**.
     
     ![Option permettant d’autoriser l’utilisation de toutes les images Marketplace en tant qu’images de base pour des machines virtuelles](./media/devtest-lab-configure-marketplace-images/allow-all-marketplace-images.png)
7. Si vous sélectionnez **Non** à l’étape précédente, la case **Images autorisées/Sélectionner tout** est cochée. 
   Vous pouvez utiliser cette option avec la zone de recherche pour rapidement sélectionner ou désélectionner tous les éléments affichés dans la liste.
   * Sélectionnez les images Azure Marketplace que vous souhaitez autoriser pour la création de machines virtuelles individuellement en activant la case à cocher correspondante de chaque image.
   * N’effectuez aucune sélection dans la liste si vous ne souhaitez autoriser aucune image Azure Marketplace dans le laboratoire.
   
     ![Vous pouvez spécifier les images Azure Marketplace pouvant être utilisées en tant qu’images de base pour des machines virtuelles](./media/devtest-lab-configure-marketplace-images/select-marketplace-images.png)


## <a name="troubleshoot"></a>Dépanner
Si vous ne trouvez pas une image spécifique à activer pour le laboratoire, procédez comme suit : 

- Vérifiez si vous pouvez afficher l’image lors de la création d’une machine virtuelle de calcul.
- L’image peut ne pas être disponible pour le type d’abonnement que vous utilisez. Vérifiez auprès de l’administrateur de l’abonnement le type d’abonnement (par exemple : MSDN, gratuit, paiement à l’utilisation, etc.). 
- La prise en charge des images de la génération 2 dans DevTest Labs est limitée. Si les versions de génération 1 et 2 sont toutes deux disponibles pour une image, DevTest Labs affiche uniquement la version de génération 1 de l’image au moment de créer une machine virtuelle. La solution de contournement consiste à créer une image personnalisée de la génération 2 à l’extérieur du laboratoire et à l’utiliser pour la création d’une machine virtuelle. Si seule une version de la génération 2 de l’image est disponible, elle est prise en charge et affichée dans la liste par DevTest Labs. 
      


## <a name="next-steps"></a>Étapes suivantes
Une fois que vous avez configuré la manière dont les images de la Place de marché Azure sont autorisées lors de la création d’une machine virtuelle, l’étape suivante consiste à [ajouter une machine virtuelle à votre laboratoire](devtest-lab-add-vm.md).


---
title: Utiliser Azure DevTest Labs à des fins de formation | Microsoft Docs
description: Cet article fournit des instructions détaillées que vous pouvez suivre pour configurer un lab de formation dans Azure DevTest Labs.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: b2de9550c62f04286a4f9ad42238bfefb9846477
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89294847"
---
# <a name="use-azure-devtest-labs-for-training"></a>Utiliser Azure DevTest Labs à des fins de formation
Azure DevTest Labs peut être utilisé pour implémenter de nombreux scénarios clés en plus du développement et du test. Un de ces scénarios consiste à mettre en place un laboratoire à des fins de formation. Azure DevTest Labs vous permet de créer un laboratoire où vous pouvez fournir des modèles personnalisés utilisables par chaque participant pour créer des environnements de formation identiques et isolés. Vous pouvez appliquer des stratégies pour vous assurer que les environnements de formation sont à la disposition des participants uniquement lorsque ces derniers en ont besoin et qu’ils contiennent suffisamment de ressources (telles que des machines virtuelles) pour la formation. Enfin, vous pouvez facilement partager le laboratoire avec les participants, qui peuvent y accéder en un seul clic.

![Utiliser DevTest Labs à des fins de formation](./media/devtest-lab-training-lab/devtest-lab-training.png)

Azure DevTest Labs remplit les exigences suivantes, indispensables pour diriger une formation dans n’importe quel environnement virtuel : 

* Les participants ne voient pas les machines virtuelles créées par les autres participants.
* Toutes les machines de formation sont identiques.
* Les participants peuvent configurer rapidement leur environnement de formation.
* Contrôlez les coûts en vous assurant que les participants ne peuvent pas obtenir plus de machines virtuelles que nécessaire et qu’ils arrêtent les machines virtuelles lorsqu’ils ne les utilisent pas.
* Partagez facilement le laboratoire de formation avec chaque participant.
* Réutilisez le laboratoire de formation à l’infini.

Cet article présente les différentes fonctionnalités d’Azure DevTest Labs qui peuvent être utilisées pour satisfaire les exigences de formation décrites précédemment et détaille la procédure à suivre pour mettre en place un laboratoire à des fins de formation.  

## <a name="implementing-training-with-azure-devtest-labs"></a>Mise en œuvre de formations avec Azure DevTest Labs
1. **Créer le laboratoire** 
   
    Les laboratoires sont le point de départ dans Azure DevTest Labs. Une fois que vous avez créé un laboratoire, vous pouvez ajouter des utilisateurs (participants) au laboratoire, mettre en place des stratégies pour contrôler les coûts, définir des images de machine virtuelle qui peuvent être créées rapidement, etc.   
   
    Pour en savoir plus, cliquez sur les liens du tableau suivant :
   
   | Tâche | Contenu |
   | --- | --- |
   | [Créer un laboratoire dans Azure DevTest Labs](devtest-lab-create-lab.md) |Découvrez comment créer un laboratoire dans Azure DevTest Labs à l’aide du portail Azure. |
2. **Créer des machines virtuelles de formation en quelques minutes à l’aide d’images Marketplace prêtes à l’emploi et d’images personnalisées** 
   
    Vous pouvez choisir des images prêtes à l’emploi parmi le large éventail d’images disponible dans Azure Marketplace et les mettre à disposition des participants dans le laboratoire. Si les images prêtes à l’emploi ne répondent pas à vos besoins, vous pouvez créer une image personnalisée de machine virtuelle de laboratoire en utilisant une image prête à l’emploi d’Azure Marketplace, en installant tous les logiciels dont vous avez besoin pour la formation et en enregistrant la machine virtuelle en tant qu’image personnalisée dans le laboratoire. 
   
    Pour en savoir plus, cliquez sur les liens du tableau suivant :
   
   | Tâche | Contenu |
   | --- | --- |
   | [Configurer des images Place de marché Azure](devtest-lab-configure-marketplace-images.md) |Découvrez comment autoriser des images Place de marché Azure, afin que seules les images souhaitées pour la formation soient sélectionnables. |
   | [Créer une image personnalisée](devtest-lab-create-template.md) |Créez une image personnalisée en préinstallant les logiciels dont vous avez besoin pour la formation afin que les participants puissent créer rapidement une machine virtuelle à l’aide de cette image. |
3. **Créer des modèles réutilisables pour les machines de formation** 
   
    Dans Azure DevTest Labs, une formule est une liste de valeurs de propriétés par défaut utilisée pour créer une machine virtuelle. Vous pouvez créer une formule dans le laboratoire en choisissant une image, une taille de machine virtuelle (une combinaison de puissance processeur et de RAM) et un réseau virtuel. Chaque candidat peut accéder à la formule dans le laboratoire et l’utiliser pour créer une machine virtuelle. 
   
    Pour en savoir plus, cliquez sur les liens du tableau suivant :
   
   | Tâche | Contenu |
   | --- | --- |
   | [Gérer les formules DevTest Labs pour créer des machines virtuelles](devtest-lab-manage-formulas.md) |Découvrez comment créer une formule en choisissant une image, une taille de machine virtuelle (une combinaison de puissance processeur et de RAM) et un réseau virtuel. |
4. **Contrôle des coûts**
   
    Azure DevTest Labs vous permet de mettre en place une stratégie dans le laboratoire pour spécifier le nombre maximal de machines virtuelles qui peuvent être créées par un participant. 
   
    Si vous dirigez une formation de plusieurs jours et souhaitez arrêter toutes les machines virtuelles à un moment précis de la journée, puis les redémarrer automatiquement le lendemain, vous pouvez facilement le faire en définissant des stratégies d’arrêt et de démarrage automatiques dans le laboratoire. 
   
    Enfin, une fois la formation terminée, vous pouvez supprimer toutes les machines virtuelles d’un coup en exécutant un simple script PowerShell. 
   
    Pour en savoir plus, cliquez sur les liens du tableau suivant :
   
   | Tâche | Contenu |
   | --- | --- |
   | [Définir des stratégies de laboratoire](devtest-lab-set-lab-policy.md) |Contrôlez les coûts en mettant en place des stratégies dans le laboratoire. |
   | [Supprimer toutes les machines virtuelles de laboratoire à l’aide d’un script PowerShell](devtest-lab-faq.md#how-do-i-automate-the-process-of-deleting-all-the-vms-in-my-lab) |Supprimez tous les laboratoires en une seule opération une fois la formation terminée. |
5. **Partager le laboratoire avec chaque participant**
   
    Les laboratoires sont directement accessibles à l’aide d’un lien que vous partagez avec les participants. Les participants n’ont même pas besoin d’avoir de compte Azure, à condition qu’ils aient un [compte Microsoft](devtest-lab-faq.md#what-is-a-microsoft-account). Les participants ne voient pas les machines virtuelles créées par les autres participants.  
   
    Pour en savoir plus, cliquez sur les liens du tableau suivant :
   
   | Tâche | Contenu |
   | --- | --- |
   | [Ajouter un participant à un laboratoire dans Azure DevTest Labs](devtest-lab-add-devtest-user.md) |Utilisez le portail Azure pour ajouter des participants à votre laboratoire de formation. |
   | [Ajouter des participants au laboratoire à l’aide d’un script PowerShell](devtest-lab-add-devtest-user.md#add-an-external-user-to-a-lab-using-powershell) |Utilisez PowerShell pour automatiser l’ajout de participants à votre laboratoire de formation. |
   | [Obtenir un lien vers le laboratoire](devtest-lab-faq.md#how-do-i-share-a-direct-link-to-my-lab) |Découvrez comment rendre un laboratoire directement accessible via un lien hypertexte. |
6. **Réutiliser le laboratoire à l’infini** 
   
    Vous pouvez automatiser la création de laboratoires, y compris les paramètres personnalisés, en créant un modèle Resource Manager qui vous permettra de mettre en place des laboratoires identiques à l’infini. 
   
    Pour en savoir plus, cliquez sur les liens du tableau suivant :
   
   | Tâche | Contenu |
   | --- | --- |
   | [Créer un laboratoire à l’aide d’un modèle Resource Manager](devtest-lab-faq.md#how-do-i-create-a-lab-from-a-resource-manager-template) |Créez des laboratoires dans Azure DevTest Labs à l’aide de modèles Resource Manager. |

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

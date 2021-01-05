---
title: Utiliser Azure DevTest Labs pour développeurs | Microsoft Docs
description: Découvrez les fonctionnalités d’Azure DevTest Labs qui peuvent être utilisées pour satisfaire les exigences du développeur et la procédure détaillée que vous pouvez suivre pour mettre en place un lab.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: b4f195f843e30ad1a56bad7d5b9ca9e73888783b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89296241"
---
# <a name="use-azure-devtest-labs-for-developers"></a>Utiliser Azure DevTest Labs pour développeurs
Azure Labs DevTest peut être utilisé pour implémenter de nombreux scénarios clés, dont celui qui implique l’utilisation de DevTest Labs dans l’hébergement d’ordinateurs de développement pour les développeurs. Dans ce scénario, DevTest Labs offre les avantages suivants :

- Les développeurs peuvent rapidement mettre en service leurs ordinateurs de développement à la demande.
- Si besoin, les développeurs peuvent facilement personnaliser leurs ordinateurs de développement.
- Les administrateurs peuvent contrôler les coûts en s’assurant que :
  - Les développeurs ne peuvent pas obtenir plus de machines virtuelles que nécessaire pour le développement.
  - Les machines virtuelles sont éteintes lorsqu’elles ne sont pas utilisées. 

![Utiliser DevTest Labs à des fins de formation](./media/devtest-lab-developer-lab/devtest-lab-developer-lab.png)

Cet article présente les différentes fonctionnalités d’Azure DevTest Labs qui peuvent être utilisées pour satisfaire les exigences du développeur et détaille la procédure à suivre pour mettre en place un laboratoire.

## <a name="implementing-developer-environments-with-azure-devtest-labs"></a>Implémentation d’environnements pour développeurs avec Azure DevTest Labs
1. **Créer le laboratoire** 
   
    Les laboratoires sont le point de départ dans Azure DevTest Labs. Une fois que vous avez créé un laboratoire, vous pouvez ajouter des utilisateurs (développeurs) au laboratoire, mettre en place des stratégies pour contrôler les coûts, définir des images de machine virtuelle qui peuvent être créées rapidement, etc.  
   
    Pour en savoir plus, cliquez sur les liens du tableau suivant :
   
   | Tâche | Contenu |
   | --- | --- |
   | [Créer un laboratoire dans Azure DevTest Labs](devtest-lab-create-lab.md) |Découvrez comment créer un laboratoire dans Azure DevTest Labs à l’aide du portail Azure. |
2. **Créer des machines virtuelles en quelques minutes à l’aide d’images Marketplace prêtes à l’emploi et d’images personnalisées** 
   
    Vous pouvez choisir des images prêtes à l’emploi parmi le large éventail d’images disponible dans Azure Marketplace et les mettre à disposition dans le laboratoire. Si les images prêtes à l’emploi ne répondent pas à vos besoins, vous pouvez créer une image personnalisée de machine virtuelle de laboratoire en utilisant une image prête à l’emploi de la Place de marché Azure, en installant tous les logiciels dont vous avez besoin et en enregistrant la machine virtuelle en tant qu’image personnalisée dans le laboratoire.

    Si vous utilisez des images personnalisées, n’hésitez pas à utiliser une fabrique d’images pour créer et distribuer vos images. Une fabrique d’images est une solution en tant que code de configuration qui crée et distribue vos images configurées automatiquement. C’est un gain de temps sur la configuration manuelle du système après la création d’une machine virtuelle avec le système d’exploitation de base.
  
    Pour en savoir plus, cliquez sur les liens du tableau suivant :
   
   | Tâche | Contenu |
   | --- | --- |
   | [Configurer des images Place de marché Azure](devtest-lab-configure-marketplace-images.md) |Découvrez comment autoriser des images Place de marché Azure, afin que seules les images destinées aux développeurs soient sélectionnables.|
   | [Créer une image personnalisée](devtest-lab-create-template.md) |Créez une image personnalisée en préinstallant les logiciels dont vous avez besoin afin que les développeurs puissent créer rapidement une machine virtuelle à l’aide de cette image.|
   | [Découvrir la fabrique d’images](./devtest-lab-faq.md#blog-post) |Regardez une vidéo expliquant comment configurer et utiliser une fabrique d’images.|

3. **Créer des modèles réutilisables pour ordinateurs de développement** 
   
    Dans Azure DevTest Labs, une formule est une liste de valeurs de propriétés par défaut utilisée pour créer une machine virtuelle. Vous pouvez créer une formule dans le laboratoire en choisissant une image, une taille de machine virtuelle (une combinaison de puissance processeur et de RAM) et un réseau virtuel. Chaque développeur peut accéder à la formule dans le laboratoire et l’utiliser pour créer une machine virtuelle. 
   
    Pour en savoir plus, cliquez sur les liens du tableau suivant :
   
   | Tâche | Contenu |
   | --- | --- |
   | [Gérer les formules DevTest Labs pour créer des machines virtuelles](devtest-lab-manage-formulas.md) |Découvrez comment créer une formule en choisissant une image, une taille de machine virtuelle (une combinaison de puissance processeur et de RAM) et un réseau virtuel.|

4. **Créer des artefacts pour activer la personnalisation flexible de machine virtuelle**

   Les artefacts sont utilisés pour déployer et configurer votre application après l’approvisionnement d’une machine virtuelle. Les artefacts peuvent être :

   - des outils que vous voulez installer sur la machine virtuelle, tels que des agents, Fiddler, Visual Studio ;
   - des actions que vous souhaitez exécuter sur la machine virtuelle, telles que le clonage d’un dépôt ;
   - des applications que vous voulez tester.

   De nombreux artefacts prêts à l’emploi sont disponibles. Vous pouvez créer vos propres artefacts personnalisés, si vos besoins spécifiques requièrent davantage de personnalisation.

   Pour en savoir plus, cliquez sur les liens du tableau suivant :
   
   | Tâche | Contenu |
   | --- | --- |
   | [Créer des artefacts personnalisés pour vos machines virtuelles DevTest Labs](devtest-lab-artifact-author.md) |Créer vos propres artefacts personnalisés pour les machines virtuelles de votre laboratoire.|
   | [Ajouter un référentiel Git pour stocker des artefacts personnalisés et des modèles Azure Resource Manager pour une utilisation dans Azure DevTest Labs](devtest-lab-add-artifact-repo.md) |Apprenez à stocker vos artefacts personnalisés dans votre propre référentiel Git privé.|

5. **Contrôle des coûts**
   
    Azure DevTest Labs vous permet de mettre en place une stratégie dans le laboratoire pour spécifier le nombre maximal de machines virtuelles qui peuvent être créées par un développeur. 
   
    Si votre équipe de développement a un planning de travail défini et que vous souhaitez arrêter toutes les machines virtuelles à un moment précis de la journée, puis les redémarrer automatiquement le lendemain, vous pouvez facilement le faire en définissant des stratégies d’arrêt et de démarrage automatiques dans le laboratoire. 
   
    Enfin, une fois le développement de l’application terminé, vous pouvez supprimer toutes les machines virtuelles d’un coup en exécutant un simple script PowerShell. 
   
    Pour en savoir plus, cliquez sur les liens du tableau suivant :
   
   | Tâche | Contenu |
   | --- | --- |
   | [Définir des stratégies de laboratoire](devtest-lab-set-lab-policy.md) |Contrôlez les coûts en mettant en place des stratégies dans le laboratoire. |
   | [Supprimer toutes les machines virtuelles de laboratoire à l’aide d’un script PowerShell](devtest-lab-faq.md#how-do-i-automate-the-process-of-deleting-all-the-vms-in-my-lab) |Supprimez tous les laboratoires en une seule opération une fois le développement terminé.|

1. **Ajouter un réseau virtuel à une machine virtuelle** 
   
    DevTest Labs crée un réseau virtuel (VNET) à chaque création de laboratoire. Si vous avez configuré votre propre réseau virtuel (par exemple, en utilisant ExpressRoute ou un VPN de site à site), vous pouvez l’ajouter aux paramètres de réseau virtuel de votre laboratoire afin qu’il soit disponible lors de la création de machines virtuelles.

    De plus, un artefact de jonction de domaine Azure Active Directory est disponible et liera une machine virtuelle à un domaine où la machine virtuelle est créée. 
   
    Pour en savoir plus, cliquez sur les liens du tableau suivant :
   
   | Tâche | Contenu |
   | --- | --- |
   | [Configuration d’un réseau virtuel dans Azure DevTest Labs](devtest-lab-configure-vnet.md) |Apprenez à configurer un réseau virtuel dans Azure DevTest Labs à l’aide du portail Azure.|

6. **Partager le laboratoire avec chaque développeur**
   
    Les laboratoires sont directement accessibles à l’aide d’un lien que vous partagez avec les développeurs. Ceux-ci n’ont même pas besoin d’un compte Azure pour autant qu’ils aient un [compte Microsoft](devtest-lab-faq.md#what-is-a-microsoft-account). Les développeurs ne voient pas les machines virtuelles créées par les autres développeurs.  
   
    Pour en savoir plus, cliquez sur les liens du tableau suivant :
   
   | Tâche | Contenu |
   | --- | --- |
   | [Ajouter un développeur à un laboratoire dans Azure DevTest Labs](devtest-lab-add-devtest-user.md) |Utilisez le portail Azure pour ajouter des développeurs à votre laboratoire.|
   | [Ajouter des développeurs au laboratoire à l’aide d’un script PowerShell](devtest-lab-add-devtest-user.md#add-an-external-user-to-a-lab-using-powershell) |Utilisez PowerShell pour automatiser l’ajout de développeurs à votre laboratoire. |
   | [Obtenir un lien vers le laboratoire](devtest-lab-faq.md#how-do-i-share-a-direct-link-to-my-lab) |Découvrez comment les développeurs peuvent accéder directement à un laboratoire via un lien hypertexte.|

7. **Automatiser la création de laboratoire pour d’autres équipes** 
   
    Vous pouvez automatiser la création de laboratoires, y compris les paramètres personnalisés, en créant un modèle Resource Manager qui vous permettra de mettre en place des laboratoires identiques à l’infini. 
   
    Pour en savoir plus, cliquez sur les liens du tableau suivant :
   
   | Tâche | Contenu |
   | --- | --- |
   | [Créer un laboratoire à l’aide d’un modèle Resource Manager](devtest-lab-faq.md#how-do-i-create-a-lab-from-a-resource-manager-template) |Créez des laboratoires dans Azure DevTest Labs à l’aide de modèles Resource Manager. |

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

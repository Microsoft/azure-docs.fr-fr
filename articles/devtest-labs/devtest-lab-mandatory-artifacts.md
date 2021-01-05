---
title: Spécifier des artefacts obligatoires pour une instance Azure DevTest Labs | Microsoft Docs
description: Découvrez comment spécifier les artefacts à installer obligatoirement avant d’installer les artefacts sélectionnés par l’utilisateur sur les machines virtuelles (VM) de l’instance lab.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 0677f8bea35cb34735fdcf34e717eea349fad8bf
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85480301"
---
# <a name="specify-mandatory-artifacts-for-your-lab-in-azure-devtest-labs"></a>Spécifier des artefacts obligatoires pour une instance Azure DevTest Labs
En tant que propriétaire d’un labo, vous pouvez spécifier des artefacts obligatoires qui sont appliqués à chaque machine créée dans le labo. Supposons que vous souhaitez qu’elles soient toutes connectées à votre réseau d’entreprise. Dans ce cas, chaque utilisateur lab devra ajouter un artefact de jonction de domaine lors de la création de la machine virtuelle pour respecter cette exigence. En d’autres termes, il devra pour ainsi dire recréer une machine si jamais il oublie d’y appliquer les artefacts obligatoires. En tant que propriétaire d’une instance lab, c’est vous qui rendez obligatoire l’artefact de jonction de domaine. L’objectif est de faire en sorte que chaque machine soit connectée au réseau d’entreprise et de faire gagner du temps aux utilisateurs.
 
On peut citer d’autres artefacts obligatoires possibles, comme un outil commun utilisé par l’équipe, un pack de sécurité lié à la plateforme que chaque machine doit posséder par défaut, etc. En somme, tout logiciel devant être commun à toutes les machines de l’instance lab devient un artefact obligatoire. Si vous créez une image personnalisée à partir d’une machine à laquelle des artefacts obligatoires sont appliqués, puis que vous créez une nouvelle machine à partir de cette image, les artefacts obligatoires sont réappliqués à la machine lors de sa création. Ce comportement a une autre conséquence : chaque fois qu’une image personnalisée, même ancienne, est utilisée pour créer une machine, la dernière version des artefacts obligatoires est appliquée à cette dernière au cours du flux de création. 
 
Seuls les artefacts sans paramètres sont pris en charge comme artefacts obligatoires. L’utilisateur de l’instance lab n’a pas besoin d’entrer des paramètres supplémentaires lors de la création de l’instance, ce qui simplifie le processus de création de machines virtuelles. 

## <a name="specify-mandatory-artifacts"></a>Spécifier des artefacts obligatoires
Les artefacts obligatoires des machines Windows et Linux peuvent être sélectionnés séparément. Vous pouvez également réorganiser ces artefacts selon l’ordre d’application de votre choix. 

1. Sur la page d’accueil de votre instance lab, sélectionnez **Configuration et stratégies** sous **PARAMÈTRES**. 
3. Sélectionnez **Artefacts obligatoires** sous **RESSOURCES EXTERNES**. 
4. Sélectionnez **Modifier** dans la section **Windows** ou **Linux**. L’option **Windows** est sélectionnée dans cet exemple. 

    ![Page d’artefacts obligatoire – Bouton Modifier](media/devtest-lab-mandatory-artifacts/mandatory-artifacts-edit-button.png)
4. Sélectionnez un artefact. L’option **7-Zip** est sélectionnée dans cet exemple. 
5. Sur la page **Ajouter un artefact**, sélectionnez **Ajouter**. 

    ![Pages des artefacts obligatoires – Ajouter 7-Zip](media/devtest-lab-mandatory-artifacts/add-seven-zip.png)
6. Pour ajouter un autre artefact, sélectionnez l’article, puis **Ajouter**. Cet exemple ajoute **Chrome** comme deuxième artefact obligatoire.

    ![Pages des artefacts obligatoires – Ajouter Chrome](media/devtest-lab-mandatory-artifacts/add-chrome.png)
7. Sur la page **Artefacts obligatoires**, un message indique le nombre d’artefacts sélectionnés. Cliquez dessus pour voir les artefacts sélectionnés. Sélectionnez **Enregistrer** pour enregistrer. 

    ![Pages des artefacts obligatoires – Enregistrer les artefacts](media/devtest-lab-mandatory-artifacts/save-artifacts.png)
8. Répétez les étapes pour spécifier les artefacts obligatoires des machines virtuelles Linux. 
    
    ![Page des artefacts obligatoires – Artefacts Windows et Linux](media/devtest-lab-mandatory-artifacts/windows-linux-artifacts.png)
9. Pour **supprimer** un artefact dans la liste, sélectionnez **… (points de suspension)**  à la fin de la ligne, puis **Supprimer**. 
10. Pour **réorganiser** les artefacts dans la liste, placez le curseur de la souris sur l’artefact, sélectionnez les **… (points de suspension)**  qui apparaissent au début de la ligne et faites glisser l’élément vers la nouvelle position. 
11. Pour enregistrer les artefacts obligatoires dans l’instance lab, sélectionnez **Enregistrer**. 

    ![Pages des artefacts obligatoires – Enregistrer les artefacts dans l’instance lab](media/devtest-lab-mandatory-artifacts/save-to-lab.png)
12. Fermez la page **Configuration et stratégies** (sélectionnez **X** dans le coin supérieur droit) pour revenir à la page d’accueil de votre instance lab.  

## <a name="delete-a-mandatory-artifact"></a>Supprimer un artefact obligatoire
Pour supprimer un artefact obligatoire d’une instance lab, effectuez les actions suivantes : 

1. Sélectionnez **Configuration et stratégies** sous **PARAMÈTRES**. 
2. Sélectionnez **Artefacts obligatoires** sous **RESSOURCES EXTERNES**. 
3. Sélectionnez **Modifier** dans la section **Windows** ou **Linux**. L’option **Windows** est sélectionnée dans cet exemple. 
4. Sélectionnez le message indiquant le nombre d’artefacts obligatoires en haut. 

    ![Page des artefacts obligatoires – Sélectionner le message](media/devtest-lab-mandatory-artifacts/select-message-artifacts.png)
5. Sur la page **Artefacts sélectionnés**, sélectionnez **… (points de suspension)**  pour l’artefact à supprimer, puis **supprimer**. 
    
    ![Pages des artefacts obligatoires – Supprimer un artefact](media/devtest-lab-mandatory-artifacts/remove-artifact.png)
6. Cliquez sur **OK** pour fermer la page **Artefacts sélectionnés**. 
7. Sélectionnez **Enregistrer** sur la page **Artefacts obligatoires**.
8. Répétez les étapes pour les images **Linux** si nécessaire. 
9. Sélectionnez **Enregistrer** pour enregistrer toutes les modifications apportées à l’instance lab. 

## <a name="view-mandatory-artifacts-when-creating-a-vm"></a>Afficher les artefacts obligatoires lors de la création d’une machine virtuelle
Les utilisateurs de l’instance lab peuvent maintenant afficher la liste des artefacts obligatoires lors de la création d’une machine virtuelle. Vous ne pouvez pas modifier ou supprimer des artefacts obligatoires définis dans l’instance lab par son propriétaire.

1. Sur la page d’accueil de votre instance lab, sélectionnez **Vue d’ensemble** dans le menu.
2. Pour ajouter une machine virtuelle au laboratoire, sélectionnez **+ Ajouter**. 
3. Sélectionnez une **image de base**. Cet exemple utilise **Windows Server version 1709**.
4. Un message indique pour **Artefacts** le nombre d’artefacts obligatoires sélectionnés. 
5. Sélectionnez **Artefacts**. 
6. Vérifiez que vous voyez les **artefacts obligatoires** que vous avez spécifiés dans la configuration et les stratégies de l’instance lab. 

    ![Créer une machine virtuelle – Artefacts obligatoires](media/devtest-lab-mandatory-artifacts/create-vm-artifacts.png)

## <a name="next-steps"></a>Étapes suivantes
* Découvrez comment [ajouter un dépôt d’artefacts Git à un laboratoire](devtest-lab-add-artifact-repo.md).


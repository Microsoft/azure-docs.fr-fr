---
title: Stocker des secrets dans un coffre de clés dans Azure DevTest Labs | Microsoft Docs
description: Découvrez comment stocker des secrets dans un coffre Azure Key Vault et les utiliser pour créer une machine virtuelle, une formule ou un environnement.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 5714279ef183cb930d643575466dae3d6cb69bba
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85481644"
---
# <a name="store-secrets-in-a-key-vault-in-azure-devtest-labs"></a>Stocker des secrets dans un coffre de clés dans Azure DevTest Labs
Vous pouvez avoir besoin d’entrer un secret complexe lorsque vous utilisez Azure DevTest Labs : un mot de passe pour votre machine virtuelle Windows, une clé SSH publique pour votre machine virtuelle Linux ou un jeton d’accès personnel pour cloner votre dépôt Git via un artefact. Les secrets sont généralement longs et ont des caractères aléatoires. Par conséquent, la saisie d’un secret peut être difficile et peu pratique, surtout si vous l’utilisez plusieurs fois.

Pour résoudre ce problème tout en conservant vos secrets dans un emplacement sûr, DevTest Labs prend en charge le stockage des secrets dans un [coffre de clés Azure](../key-vault/general/overview.md). Quand un utilisateur enregistre un secret pour la première fois, le service DevTest Labs crée automatiquement un coffre de clés dans le groupe de ressources qui contient le laboratoire et stocke le secret dans le coffre de clés. DevTest Labs crée un coffre de clés distinct pour chaque utilisateur. 

Veuillez noter que l’utilisateur du laboratoire devra d’abord créer une machine virtuelle de laboratoire avant de pouvoir créer un secret dans le coffre de clés. Cela est dû au fait que le service DevTest Lab doit associer l’utilisateur du laboratoire à un document utilisateur valide avant qu’il ne soit autorisé à créer et à stocker des secrets dans son coffre de clés. 


## <a name="save-a-secret-in-azure-key-vault"></a>Enregistrer un secret dans Azure Key Vault
Pour enregistrer votre secret dans Azure Key Vault, procédez comme suit :

1. Sélectionnez **Mes secrets** dans le menu de gauche.
2. Entrez un **nom** pour le secret. Vous voyez ce nom dans la liste déroulante lorsque vous créez une machine virtuelle, une formule ou un environnement. 
3. Entrez le secret en tant que **valeur**.

    ![Stockage d’un secret](media/devtest-lab-store-secrets-in-key-vault/store-secret.png)

## <a name="use-a-secret-from-azure-key-vault"></a>Utiliser un secret à partir d’Azure Key Vault
Lorsque vous avez besoin d’entrer un secret pour créer une machine virtuelle, une formule ou un environnement, vous pouvez entrer un secret manuellement ou sélectionner un secret enregistré dans le coffre de clés. Pour utiliser un secret stocké dans votre coffre de clés, effectuez les actions suivantes :

1. Sélectionnez **Utiliser un secret enregistré**. 
2. Sélectionnez votre secret dans la liste déroulante pour **Choisir un secret**. 

    ![Utilisation d’un secret dans la machine virtuelle](media/devtest-lab-store-secrets-in-key-vault/secret-store-pick-a-secret.png)

## <a name="use-a-secret-in-an-azure-resource-manager-template"></a>Utiliser un secret dans un modèle Azure Resource Manager
Vous pouvez spécifier le nom de votre secret dans un modèle Azure Resource Manager qui est utilisé pour créer une machine virtuelle, comme illustré dans l’exemple suivant :

![Utilisation d’un secret dans une formule ou un environnement](media/devtest-lab-store-secrets-in-key-vault/secret-store-arm-template.png)

## <a name="next-steps"></a>Étapes suivantes

- [Créer une machine virtuelle à l’aide du secret](devtest-lab-add-vm.md) 
- [Créer une formule à l’aide du secret](devtest-lab-manage-formulas.md)
- [Créer un environnement à l’aide du secret](devtest-lab-create-environment-from-arm.md)

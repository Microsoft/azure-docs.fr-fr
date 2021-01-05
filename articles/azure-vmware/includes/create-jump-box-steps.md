---
title: Créer la jumpbox Azure VMware Solution
description: Procédure pour créer la jumpbox Azure VMware Solution.
ms.topic: include
ms.date: 09/28/2020
ms.openlocfilehash: 5d5a5d82cf6e70e7ad55bcfa10c7f85372131c87
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91578413"
---
<!-- Used in deploy-azure-vmware-solution.md and tutorial-access-private-cloud.md -->

1. Dans le groupe de ressources, sélectionnez **+ Ajouter**, recherchez et sélectionnez **Microsoft Windows 10**, puis **Créer**.

   :::image type="content" source="../media/tutorial-access-private-cloud/ss8-azure-w10vm-create.png" alt-text="Ajoutez une nouvelle machine virtuelle Windows 10 pour une jumpbox." border="true":::

1. Entrez les valeurs nécessaires dans les champs, puis sélectionnez **Vérifier + créer**. 

   Pour plus d’informations sur les champs, consultez le tableau suivant.

   | Champ | Valeur |
   | --- | --- |
   | **Abonnement** | Cette valeur est prérenseignée avec l’abonnement appartenant au groupe de ressources. |
   | **Groupe de ressources** | La valeur est préremplie pour le groupe de ressources actuel que vous avez créé dans le tutoriel précédent.  |
   | **Nom de la machine virtuelle** | Entrez un nom unique pour la machine virtuelle. |
   | **Région** | Sélectionnez le lieu géographique de la machine virtuelle. |
   | **Options de disponibilité** | Laissez la valeur par défaut sélectionnée. |
   | **Image** | Sélectionnez l’image de machine virtuelle. |
   | **Taille** | Laissez la valeur de la taille par défaut. |
   | **Type d’authentification**  | Sélectionnez **Mot de passe**. |
   | **Nom d’utilisateur** | Entrez le nom d’utilisateur pour la connexion à la machine virtuelle. |
   | **Mot de passe** | Entrez le mot de passe pour la connexion à la machine virtuelle. |
   | **Confirmer le mot de passe** | Entrez le mot de passe pour la connexion à la machine virtuelle. |
   | **Ports d’entrée publics** | Sélectionnez **Aucun**. Si vous sélectionnez Aucun, vous pouvez utiliser l’[accès JIT](../../security-center/security-center-just-in-time.md#jit-configure) pour contrôler l’accès à la machine virtuelle seulement quand vous voulez y accéder.  |


1. Une fois la validation réussie, sélectionnez **Créer** pour démarrer le processus de création de la machine virtuelle.

   :::image type="content" source="../media/tutorial-access-private-cloud/ss11-review-create-wjb01.png" alt-text="Ajoutez une nouvelle machine virtuelle Windows 10 pour une jumpbox." border="true":::
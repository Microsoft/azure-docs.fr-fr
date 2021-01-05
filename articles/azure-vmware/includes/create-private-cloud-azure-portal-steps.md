---
title: Déployer Azure VMware Solution
description: Étapes de déploiement d’Azure VMware Solution à l’aide du portail Azure.
ms.topic: include
ms.date: 09/28/2020
ms.openlocfilehash: 4d0881721cd8c13d1b6c9fb3a29e4cdb6d6a753f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91578311"
---
<!-- Used in deploy-azure-vmware-solution.md and tutorial-create-private-cloud.md -->

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Sélectionnez **Créer une ressource**. Dans la zone de texte **Rechercher dans la Place de marché**, tapez `Azure VMware Solution`, puis sélectionnez **Azure VMware Solution** dans la liste. Dans la fenêtre **Azure VMware Solution**, sélectionnez **Créer**.

1. Sous l’onglet **De base**, entrez des valeurs pour les champs. Le tableau suivant liste les propriétés des champs.

   | Champ   | Valeur  |
   | ---| --- |
   | **Abonnement** | Abonnement que vous prévoyez d’utiliser pour le déploiement.|
   | **Groupe de ressources** | Groupe de ressources pour vos ressources de cloud privé. |
   | **Lieu** | Sélectionnez un emplacement, comme **USA Est**.|
   | **Nom de la ressource** | Nom de votre cloud privé Azure VMware Solution. |
   | **Référence (SKU)** | Sélectionnez la valeur de référence SKU suivante : AV36 |
   | **Hôtes** | Nombre d’hôtes à ajouter au cluster du cloud privé. La valeur par défaut est 3. Elle peut être augmentée ou diminuée après le déploiement.  |
   | **Mot de passe de l’administrateur du vCenter** | Entrez en mot de passe d’administrateur du cloud. |
   | **Mot de passe de NSX-T Manager** | Entrez un mot de passe d’administrateur NSX-T. |
   | **Bloc d’adresses** | Entrez un bloc d’adresses IP pour le réseau CIDR pour le cloud privé, par exemple 10.175.0.0/22. |
   | **Réseau virtuel** | Sélectionnez un réseau virtuel ou créez-en un pour le cloud privé Azure VMware Solution.  |

   :::image type="content" source="../media/tutorial-create-private-cloud/create-private-cloud.png" alt-text="Sous l’onglet De base, entrez des valeurs pour les champs." border="true":::

1. Quand vous avez terminé, sélectionnez **Vérifier + créer**. Dans l’écran suivant, vérifiez les informations entrées. Si elles sont toutes correctes, sélectionnez **Créer**.

   > [!NOTE]
   > Cette étape prend environ 2 heures. 

1. Vérifiez que déploiement a réussi. Accédez au groupe de ressources que vous avez créé et sélectionnez votre cloud privé.  Quand le déploiement est terminé, vous voyez l’état **Réussi**. 

   :::image type="content" source="../media/tutorial-create-private-cloud/validate-deployment.png" alt-text="Sous l’onglet De base, entrez des valeurs pour les champs." border="true":::
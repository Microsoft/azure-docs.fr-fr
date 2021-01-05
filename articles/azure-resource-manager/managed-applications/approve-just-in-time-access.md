---
title: Approuver l’accès juste-à-temps
description: Décrit comment les consommateurs d’applications managées Azure approuvent les demandes d’accès juste à temps à une application managée.
author: MSEvanhi
ms.topic: conceptual
ms.date: 06/03/2019
ms.author: evanhi
ms.openlocfilehash: 4a3604a3256fb2d3f4253891bbc28c7685748cd2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75649369"
---
# <a name="configure-and-approve-just-in-time-access-for-azure-managed-applications"></a>Configurer et approuver l’accès juste à temps pour les applications managées Azure

En tant que consommateur d’une application managée, vous n’êtes peut-être pas d’accord avec le fait d’octroyer à l’éditeur un accès permanent au groupe de ressources managées. Pour renforcer votre contrôle sur l’accès aux ressources managées, les applications managées Azure offrent une fonction appelée Accès juste à temps (JAT), qui est actuellement en cours préversion. Il vous permet d’approuver quand et pendant combien de temps l’éditeur a accès au groupe de ressources. L’éditeur peut effectuer les mises à jour nécessaires pendant cette période, mais lorsque cette période est terminée, son accès expire.

Voici la procédure à suivre pour accorder l’accès :

1. L’éditeur ajoute une application managée à la Place de marché et précise que l’accès JAT est disponible.

1. Pendant le déploiement, vous activez l’accès JAT pour votre instance de l’application managée.

1. Après le déploiement, vous pouvez modifier les paramètres d’accès JAT.

1. L’éditeur envoie une demande d’accès.

1. Vous approuvez la demande.

Le présent article porte sur les mesures prises par les consommateurs pour permettre l’accès JAT et approuver les demandes. Pour en savoir plus sur la publication d’une application managée avec accès JAT, voir [Demander l’accès juste à temps dans les applications managées Azure](request-just-in-time-access.md).

> [!NOTE]
> Pour utiliser l’accès juste à temps, vous devez disposer d’une licence [Azure Active Directory P2](../../active-directory/privileged-identity-management/subscription-requirements.md).

## <a name="enable-during-deployment"></a>Activer pendant le déploiement

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Recherchez une entrée de Place de marché pour une application managée avec JAT activé. Sélectionnez **Create** (Créer).

1. Tout en fournissant des valeurs pour la nouvelle application managée, l’étape **Configuration JAT**vous permet d’activer ou de désactiver l’accès JAT pour l’application managée. Sélectionnez **Oui** pour **Activer l’accès JAT**. Cette option est sélectionnée par défaut pour les applications managées qui sont définies avec JAT activé sur la Place de marché.

   ![Configurer l’accès](./media/approve-just-in-time-access/configure-jit-access.png)

   Vous pouvez activer l’accès JAT uniquement pendant le déploiement. Si vous sélectionnez **Non**, l’éditeur obtient un accès permanent au groupe de ressources managées. Vous ne pouvez pas activer l’accès JAT ultérieurement.

1. Pour modifier les paramètres d’approbation par défaut, sélectionnez **Personnaliser la configuration JAT**.

   ![Personnaliser l’accès](./media/approve-just-in-time-access/customize-jit-access.png)

   Par défaut, une application managée avec le JAT activé a les paramètres suivants :

   * Mode d’approbation : automatique
   * Durée maximale d’accès : 8 heures
   * Approbateurs : aucun

   Quand le mode d’approbation est défini sur **Automatique**, les approbateurs reçoivent une notification pour chaque demande, mais la demande est approuvée automatiquement. Lorsque le mode d’approbation est défini sur **Manuelle**, les approbateurs reçoivent une notification pour chaque demande, et l’un d’eux doit l’approuver.

   La durée maximale d’activation spécifie le délai maximal qu’un éditeur peut demander pour l’accès au groupe de ressources managées.

   La liste des approbateurs contient les utilisateurs Azure Active Directory qui peuvent approuver des demandes d’accès JAT. Pour ajouter un approbateur, sélectionnez **Ajouter un approbateur** et recherchez l’utilisateur de votre choix.

   Après la mise à jour du paramètre, sélectionnez **Enregistrer**.

## <a name="update-after-deployment"></a>Mettre à jour après le déploiement

Vous pouvez modifier les valeurs d’approbation des demandes. Toutefois, si vous n’avez pas activé l’accès JAT au cours du déploiement, vous ne pouvez pas l’activer ultérieurement.

Pour modifier les paramètres d’une application managée déployée :

1. Dans le portail, sélectionnez Gérer une application.

1. Sélectionnez **Configuration JAT**, puis modifiez les paramètres selon vos besoins.

   ![Modifier les paramètres d’accès](./media/approve-just-in-time-access/change-settings.png)

1. Lorsque vous avez terminé, sélectionnez **Enregistrer**.

## <a name="approve-requests"></a>Approuver les demandes

Lorsque l’éditeur demande l’accès, vous en êtes averti. Vous pouvez approuver les demandes d’accès JAT directement par le biais de l’application managée, ou pour toutes les applications managées via le service Azure Active Directory Privileged Identity Management. Pour utiliser l’accès juste à temps, vous devez disposer d’une licence [Azure Active Directory P2](../../active-directory/privileged-identity-management/subscription-requirements.md).

Pour approuver les demandes via l’application managée :

1. Sélectionnez **Accès JAT** pour l’application managée, puis sélectionnez **Approuver les demandes**.

   ![Approuver les demandes](./media/approve-just-in-time-access/approve-requests.png)
 
1. Sélectionnez la demande à approuver.

   ![Sélection de la demande](./media/approve-just-in-time-access/select-request.png)

1. Dans le formulaire, indiquez la raison de l’approbation, puis sélectionnez **Approuver**.

Pour approuver les demandes via Azure AD Privileged Identity Management :

1. Sélectionnez **Tous les services**, puis filtrez la liste pour rechercher **Azure AD Privileged Identity Management**. Sélectionnez-le parmi les options disponibles.

   ![Recherche du service](./media/approve-just-in-time-access/search.png)

1. Sélectionnez **Approuver les demandes**.

   ![Sélection de l’option Approuver les demandes](./media/approve-just-in-time-access/select-approve-requests.png)

1. Sélectionnez **Applications managées Azure**, puis sélectionnez la demande à approuver.

   ![Sélection des demandes](./media/approve-just-in-time-access/view-requests.png)

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur la publication d’une application managée avec accès JAT, voir [Demander l’accès juste à temps dans les applications managées Azure](request-just-in-time-access.md).

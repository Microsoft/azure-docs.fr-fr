---
title: Configurer Microsoft Endpoint Configuration Manager - Azure
description: Configuration de Microsoft Endpoint Configuration Manager de manière à déployer des mises à jour logicielles pour Windows 10 Entreprise multisession sur Windows Virtual Desktop.
author: Heidilohr
ms.topic: how-to
ms.date: 06/12/2020
ms.author: helohr
ms.reviewer: v-cawood; clemr
manager: lizross
ms.openlocfilehash: bfc797e0c11a1804d27c6c97fea8cb2ba62dca7a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88010121"
---
# <a name="configure-microsoft-endpoint-configuration-manager"></a>Configurer Microsoft Endpoint Configuration Manager

Cet article explique comment configurer Microsoft Endpoint Configuration Manager de manière à appliquer automatiquement les mises à jour à l’hôte Windows Virtual Desktop en exécutant Windows 10 Entreprise multisession.

## <a name="prerequisites"></a>Prérequis

Pour configurer ce paramètre, vous aurez besoin des éléments suivants :

   - Assurez-vous que vous avez installé l’agent Microsoft Endpoint Configuration Manager sur vos machines virtuelles.
   - Assurez-vous que votre version de Microsoft Endpoint Configuration Manager est au moins au niveau de branche 1906. Pour optimiser les résultats, utilisez le niveau de branche 1910 ou un niveau supérieur.

## <a name="configure-the-software-update-point"></a>Configurer le point de mise à jour logicielle

Pour recevoir des mises à jour pour Windows 10 Entreprise multisession, vous devez activer le produit Windows Server version 1903 et ultérieure dans Microsoft Endpoint Configuration Manager. Ce paramètre du produit s’applique également si vous utilisez Windows Server Update Service pour déployer des mises à jour sur vos systèmes.

Pour recevoir des mises à jour :

1. Ouvrez Microsoft Endpoint Configuration Manager et sélectionnez **Sites**.
2. Sélectionnez **Configurer les composants de site**.
3. Dans le menu déroulant, sélectionnez **Point de mise à jour logicielle**.
4. Sélectionnez l’onglet **Produits**.
5. Cochez la case **Windows Server, version 1903 et ultérieure**.
6. Accédez à **Bibliothèque de logiciels** > **Vue d’ensemble** > **Mises à jour logicielles** > **Toutes les mises à jour logicielles**, puis sélectionnez **Synchroniser les mises à jour logicielles**.
7. Consultez le fichier wsyncmgr.log dans **Program Files** > **Microsoft Configuration Manager** > **Journaux** pour vérifier que vos modifications ont été enregistrées. La synchronisation des mises à jour peut prendre quelques minutes.

## <a name="create-a-query-based-collection"></a>Créer une collection basée sur une requête

Pour créer une collection de machines virtuelles Windows 10 Entreprise multisession, une collection basée sur une requête peut être utilisée pour identifier le système d’exploitation SKU spécifique.

Pour créer une collection :

1. Sélectionnez **Ressources et conformité**.
2. Accédez à **Vue d’ensemble** > **Collection d’appareils**, cliquez avec le bouton droit sur **Collections d’appareils**, puis sélectionnez **Créer une collection d’appareils** dans le menu déroulant.
3. Sous l’onglet **Général** du menu qui s’ouvre, entrez un nom qui décrit votre collection dans le champ **Nom**. Dans le champ **Commentaire**, vous pouvez renseigner des informations supplémentaires décrivant la collection. Dans **Limitation au regroupement**, définissez les machines que vous souhaitez inclure dans la requête de collection.
4. Sous l’onglet **Règles d’adhésion**, ajoutez une règle pour votre requête en sélectionnant **Ajouter une règle**, puis **Règle de requête**.
5. Dans **Propriétés de la règle de requête**, entrez un nom pour votre règle, puis définissez les paramètres de la règle en sélectionnant **Modifier l’instruction de requête**.
6. Sélectionnez **Afficher l’instruction de la requête**.
7. Dans l’instruction, entrez la chaîne suivante :

    ```syntax
    select
    SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client
    from SMS_R_System inner join SMS_G_System_OPERATING_SYSTEM on
    SMS_G_System_OPERATING_SYSTEM.ResourceId = SMS_R_System.ResourceId where
    SMS_G_System_OPERATING_SYSTEM.OperatingSystemSKU = 175
    ```

8. Sélectionnez **OK** pour créer la collection.
9. Pour vérifier que vous avez créé une collection avec succès, accédez à **Ressources et conformité** > **Vue d’ensemble** > **Collections d’appareils**.

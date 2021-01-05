---
title: Obtenir une imagerie par drone
description: Cet article décrit comment obtenir une imagerie par drone auprès des partenaires.
author: uhabiba04
ms.topic: article
ms.date: 11/04/2019
ms.author: v-umha
ms.openlocfilehash: 04f0c3c63d00ea49bf43f00f256266599a73d6c0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88508802"
---
# <a name="get-drone-imagery-from-drone-partners"></a>Obtenir une imagerie par drone auprès des partenaires de drone

Cet article explique comment récupérer, dans Azure FarmBeats Datahub, les données orthophotographiques de partenaires d’imagerie par drone. Une orthophotographie est une illustration ou une image aérienne qui est corrigée géométriquement et dont un panorama est créé à partir de données collectées par un drone.

À l’heure actuelle, les partenaires d’imagerie suivants sont pris en charge.

  ![Partenaires d’imagerie par drone de FarmBeats](./media/get-drone-imagery-from-drone-partner/drone-partner-1.png)

L’intégration de données d’imagerie par drone à Azure FarmBeats vous permet de récupérer, dans le hub de données, des données orthophotographiques à partir des vols de drone réalisés sur votre ferme. Une fois les données disponibles, vous pouvez les voir dans l’accélérateur FarmBeats. Vous pouvez utiliser les données pour la fusion de données et pour la création de modèles d’intelligence artificielle et de machine learning.

## <a name="before-you-begin"></a>Avant de commencer

  - Vérifiez que vous avez installé Azure FarmBeats. Pour plus d’informations sur l’installation de FarmBeats, consultez [Installer Azure FarmBeats](install-azure-farmbeats.md).
  - Vérifiez que la ferme pour laquelle vous souhaitez disposer de l’imagerie par drone est définie dans votre système FarmBeats.

## <a name="enable-drone-imagery-integration-with-farmbeats"></a>Activer l’intégration de l’imagerie par drone à FarmBeats

Indiquez les informations suivantes au fournisseur d’appareils pour activer l’intégration à FarmBeats :
 - Point de terminaison d’API
 - ID client
 - ID client
 - Clé secrète client

Effectuez les opérations suivantes.

1. Téléchargez ce [script](https://aka.ms/farmbeatspartnerscript) et extrayez-le sur votre lecteur local. Le fichier zip contient deux fichiers.
2. Connectez-vous au [portail Azure](https://portal.azure.com/) et ouvrez Azure Cloud Shell. Cette option est disponible dans la barre d’outils située en haut à droite du portail.

    ![Ouvrir Azure Cloud Shell dans la barre située en haut à droite du portail](./media/get-drone-imagery-from-drone-partner/navigation-bar-1.png)

3. Veillez à ce que **PowerShell** soit défini comme environnement.

    ![Paramètre PowerShell](./media/get-drone-imagery-from-drone-partner/power-shell-new-1.png)

4. Chargez les deux fichiers que vous avez téléchargés à l’étape 1 dans votre instance Cloud Shell.

    ![Charger des fichiers](./media/get-drone-imagery-from-drone-partner/power-shell-two-1.png)

5. Accédez au répertoire où les fichiers ont été chargés. Par défaut, ils sont chargés dans le répertoire de base sous le nom d’utilisateur.
6. Exécutez le script suivant :

    ```azurepowershell-interactive

    ./generateCredentials.ps1

    ```

7. Suivez les instructions à l’écran pour capturer les valeurs suivantes : point de terminaison d’API, ID de locataire, ID client, secret client et chaîne de connexion du hub d’événements.

    Après avoir entré les informations d’identification requises dans le système logiciel du drone du partenaire, vous pouvez importer toutes les fermes à partir du système FarmBeats. Vous pouvez ensuite utiliser les détails de la ferme pour planifier votre trajectoire de vol et collecter des images du drone.

    Une fois les images brutes traitées par le logiciel du fournisseur de drone, le système logiciel du drone charge les orthophotographies assemblées et d’autres images traitées dans le hub de données.

## <a name="view-drone-imagery"></a>Voir l’imagerie par drone

Une fois les données envoyées au hub de données FarmBeats, vous pouvez interroger le magasin de scènes à l’aide des API FarmBeats Datahub.

Vous pouvez également voir la dernière image de drone dans la page **Farm Details** (Détails de la ferme). Pour voir l’image, effectuez les étapes suivantes.

1. Sélectionnez la ferme sur laquelle l’imagerie a été chargée. La page **Farm Details** s’affiche.
2. Faites défiler la page vers le bas jusqu’à la dernière section **Precision Maps** (Cartes de précision).
3. Examinez l’image dans la section **Drone Imagery** (Imagerie par drone).

    ![Section Drone Imagery (Imagerie par drone)](./media/get-drone-imagery-from-drone-partner/drone-imagery-1.png)

## <a name="download-drone-imagery"></a>Télécharger l’imagerie par drone

Quand vous sélectionnez la section Drone Imagery, une fenêtre indépendante s’ouvre avec une image haute résolution de l’orthophotographie du drone.

![Orthophotographie haute résolution](./media/get-drone-imagery-from-drone-partner/download-drone-imagery-1.png)

## <a name="view-all-drone-maps"></a>Voir toutes les cartes de drone

Les fichiers et images chargés par le fournisseur de drone s’affichent dans la section **Maps**. Sélectionnez la section **Maps**, filtrez par **Farm**, puis sélectionnez les fichiers à afficher et à télécharger.

  ![Section Maps](./media/get-drone-imagery-from-drone-partner/view-drone-maps-1.png)

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment utiliser les [API](rest-api-in-azure-farmbeats.md) FarmBeats Datahub pour obtenir vos données d’imagerie par drone.

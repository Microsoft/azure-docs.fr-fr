---
title: Création et gestion de runtimes d’intégration
description: Cet article décrit la procédure à suivre pour créer et gérer des runtimes d’intégration dans Azure Purview.
author: viseshag
ms.author: viseshag
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 11/16/2020
ms.openlocfilehash: 0d365787ea3603ef0adb8ad0b48bef9792ffb003
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96550109"
---
# <a name="create-and-manage-a-self-hosted-integration-runtime"></a>Création et gestion d’un runtime d’intégration auto-hébergé

Cet article explique comment créer et gérer un runtime d’intégration auto-hébergé pour faciliter l’analyse des sources de données.

## <a name="create-a-self-hosted-integration-runtime"></a>Créer un runtime d’intégration auto-hébergé

1. Sur la page d’accueil de Purview Studio, sélectionnez **Centre de gestion** dans le volet de navigation gauche.

2. Sous **Sources et analyse** dans le volet gauche, sélectionnez **Runtimes d’intégration**, puis **+ Nouveau**.

    :::image type="content" source="media/manage-integration-runtimes/select-integration-runtimes.png" alt-text="Clic sur Runtimes d’intégration":::

3. Sur la page **Configuration des runtimes d’intégration**, sélectionnez **Auto-hébergé** pour créer un runtime d’intégration auto-hébergé, puis **Continuer**.

    :::image type="content" source="media/manage-integration-runtimes/select-self-hosted-ir.png" alt-text="Création d’un runtime d’intégration auto-hébergé":::

4. Entrez un nom pour votre runtime d’intégration, puis sélectionnez Créer.

5. Sur la page **Paramètres des runtime d’intégration**, suivez la procédure décrite dans la section **Configuration manuelle**. Vous devrez télécharger le runtime d’intégration sur une machine virtuelle ou l’ordinateur sur lequel vous envisagez de l’exécuter à partir du site de téléchargement.

    :::image type="content" source="media/manage-integration-runtimes/integration-runtime-settings.png" alt-text="Récupération de la clé":::

    a. Copiez et collez la clé d’authentification.
        
    b. Téléchargez le runtime intégration auto-hébergé sur un ordinateur Windows local à partir [d’Azure Data Factory Integration Runtime](https://www.microsoft.com/download/details.aspx?id=39717). Exécutez le programme d’installation.
        
    c. Sur la page **Inscrire le runtime d’intégration (auto-hébergé)** , collez l’une des deux clés que vous avez enregistrées, puis cliquez sur **Inscrire**.

    :::image type="content" source="media/manage-integration-runtimes/register-integration-runtime.png" alt-text="Clé d’entrée":::

    d. Dans la page **Nouveau runtime d’intégration (auto-hébergé)** , sélectionnez **Terminer**.

6. Une fois le runtime d’intégration auto-hébergé inscrit, la fenêtre suivante s’affiche :

    :::image type="content" source="media/manage-integration-runtimes/successfully-registered.png" alt-text="Inscription réussie":::

## <a name="manage-a-self-hosted-integration-runtime"></a>Gestion d’un runtime d’intégration auto-hébergé

Pour modifier un runtime d’intégration auto-hébergé, accédez à **Runtimes d’intégration** dans le **Centre de gestion**, sélectionnez le runtime d’intégration, puis cliquez sur Modifier. Vous pouvez alors mettre à jour la description, copier la clé ou en régénérer de nouvelles.

:::image type="content" source="media/manage-integration-runtimes/edit-integration-runtime.png" alt-text="Modification du runtime d’intégration":::

:::image type="content" source="media/manage-integration-runtimes/edit-integration-runtime-settings.png" alt-text="Modification des détails du runtime d’intégration":::

Pour supprimer un runtime d’intégration auto-hébergé, accédez à **Runtimes d’intégration** dans le Centre de gestion, sélectionnez le runtime d’intégration, puis cliquez sur **Supprimer**. Une fois le runtime d’intégration supprimé, toutes les analyses en cours qui en dépendent échouent.

## <a name="next-steps"></a>Étapes suivantes

* [Comment les analyses détectent les éléments supprimés](concept-detect-deleted-assets.md)

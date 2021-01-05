---
title: Informations détaillées sur les documents – Custom Translator
titleSuffix: Azure Cognitive Services
description: La page de liste des documents affiche les 10 premiers documents dans votre espace de travail. Le nom de chaque document, ainsi que les informations relatives à son appariement, son type, sa langue, l’heure de son chargement, ainsi que l’adresse de messagerie de l’utilisateur ayant chargé ce document.
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.date: 08/17/2020
ms.author: swmachan
ms.topic: conceptual
ms.openlocfilehash: 87b999069ef9088a731a4e972c5d548cac0b917c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88509560"
---
# <a name="view-document-details"></a>Afficher les détails du document

La page de liste des documents affiche les 10 premiers documents dans votre espace de travail. Le nom de chaque document, ainsi que les informations relatives à son appariement, son type, sa langue, l’heure de son chargement, ainsi que l’adresse de messagerie de l’utilisateur ayant chargé ce document.

Cliquez sur un document pour afficher la page d’informations détaillées à son sujet. La page d’informations détaillées d’un document contient une liste des phrases extraites de ce document.

- Par défaut, l’affichage « côte à côte » et les langues source et cible sont sélectionnés dans le champ de liste déroulante, mais vous pouvez choisir de voir les phrases dans la langue source ou la langue cible.
- Par défaut, chaque page affiche 20 phrases. Vous pouvez utiliser le contrôle de pagination pour naviguer entre les pages.

![Informations détaillées sur le document](media/how-to/how-to-view-document-details.png)

## <a name="delete-a-document"></a>Supprimer un document

Pour pouvoir supprimer un document, l’utilisateur doit être propriétaire de l’espace de travail dans lequel il se trouve. De plus, si un document est utilisé par un modèle dans n’importe quelle partie du processus de formation ou de déploiement, le document ne peut pas être supprimé.

1. Accédez à page du document.
2. Placez le pointeur sur n’importe quel enregistrement de document, puis cliquez sur l’icône Corbeille.

    ![Supprimer un document](media/how-to/how-to-delete-document-1.png)

3. Confirmez la suppression.

    ![Confirmer la suppression](media/how-to/how-to-delete-document-confirm.png)

## <a name="next-steps"></a>Étapes suivantes

- Découvrez [comment effectuer l’apprentissage d’un modèle](how-to-train-model.md).

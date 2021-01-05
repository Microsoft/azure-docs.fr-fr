---
title: Utiliser une vignette Markdown personnalisée sur les tableaux de bord Azure
description: Découvrez comment ajouter une vignette Markdown à un tableau de bord Azure pour afficher un contenu statique
ms.date: 01/08/2020
ms.topic: how-to
ms.custom: devx-track-js
ms.openlocfilehash: 5121142ecf568aa1ac9a7ec19f7211c6f9a6253f
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96745755"
---
# <a name="use-a-markdown-tile-on-azure-dashboards-to-show-custom-content"></a>Utiliser une vignette Markdown sur les tableaux de bord Azure pour afficher un contenu personnalisé

Vous pouvez ajouter une vignette Markdown à vos tableaux de bord Azure pour afficher un contenu statique personnalisé. Par exemple, vous pouvez afficher des instructions de base, une image ou un ensemble de liens hypertexte dans une vignette Markdown.

## <a name="add-a-markdown-tile-to-your-dashboard"></a>Ajouter une vignette Markdown à votre tableau de bord

1. Sélectionnez **Tableau de bord** à partir de la barre latérale du portail Azure.

   ![Capture d’écran montrant la barre latérale du portail](./media/azure-portal-markdown-tile/azure-portal-nav.png)

1. Si vous avez créé des tableaux de bord personnalisés, dans la vue de tableau de bord, utilisez la liste déroulante pour sélectionner le tableau de bord où doit apparaître la vignette Markdown personnalisée. Sélectionnez l’icône Modifier pour ouvrir la **Galerie de vignettes**.

   ![Capture d’écran montrant la vue de tableau de bord en mode modification](./media/azure-portal-markdown-tile/azure-portal-dashboard-edit.png)

1. Dans la **Galerie de vignettes**, recherchez la vignette appelée **Markdown**, puis sélectionnez **Ajouter**. La vignette est ajoutée au tableau de bord et le volet **Modifier le balisage Markdown** s’ouvre.

1. Entrez des valeurs pour le **Titre** et le **Sous-titre**, qui s’afficheront sur la vignette une fois que vous serez passé à un autre champ.

   ![Capture d’écran montrant les résultats de la saisie du titre et du sous-titre](./media/azure-portal-markdown-tile/azure-portal-dashboard-enter-title.png)

1. Sélectionnez l’une des options permettant d’inclure le contenu Markdown : **Modification intraligne** ou **Insérer du contenu à l’aide d’une URL**.

   - Sélectionnez **Modification intraligne** si vous souhaitez entrer le contenu Markdown directement.

      ![Capture d’écran montrant la saisie du contenu intraligne](./media/azure-portal-markdown-tile/azure-portal-dashboard-markdown-inline-content.png)

   - Sélectionnez **Insérer du contenu à l’aide d’une URL** si vous souhaitez utiliser le contenu Markdown existant qui est hébergé en ligne.

      ![Capture d’écran montrant la saisie de l’URL](./media/azure-portal-markdown-tile/azure-portal-dashboard-markdown-url.png)

      > [!NOTE]
      > Pour renforcer la sécurité, vous pouvez créer un fichier Markdown et le stocker dans un [objet blob de compte de stockage Azure où le chiffrement est activé](../storage/common/storage-service-encryption.md), puis référencer le fichier à l’aide de l’option URL. Le contenu Markdown est chiffré à l’aide des options de chiffrement du compte de stockage. Seuls les utilisateurs disposant d’autorisations d’accès au fichier peuvent voir le contenu Markdown dans le tableau de bord. Il se peut que vous deviez définir une règle de [partage des ressources cross-origin (CORS)](/rest/api/storageservices/cross-origin-resource-sharing--cors--support-for-the-azure-storage-services) sur le compte de stockage afin que le portail Azure ( _https://portal.azure.com/_ ) puisse accéder au fichier Markdown dans le blob.

1. Sélectionnez **Terminé** pour faire disparaître le volet **Modifier le balisage Markdown**. Votre contenu s’affiche sur la vignette Markdown, que vous pouvez ensuite redimensionner en faisant glisser la poignée vers le coin inférieur droit.

   ![Capture d’écran montrant la vignette Markdown personnalisée](./media/azure-portal-markdown-tile/azure-portal-custom-markdown-tile.png)

## <a name="markdown-content-capabilities-and-limitations"></a>Limitations et fonctionnalités du contenu Markdown

Vous pouvez utiliser n’importe quelle combinaison de texte brut, syntaxe Markdown et contenu HTML sur la vignette Markdown. Le portail Azure utilise une bibliothèque open source appelée _marked_ pour transformer votre contenu en syntaxe HTML qui est affichée sur la vignette. La syntaxe HTML générée par _marked_ est prétraitée par le portail avant d’être affichée. Ainsi, votre personnalisation n’affecte pas la sécurité ou la disposition du portail. Pendant ce prétraitement, toute partie de la syntaxe HTML qui représente une menace potentielle est supprimée. Les types de contenu suivants ne sont pas autorisés par le portail :

* JavaScript : les balises `<script>` et les évaluations JavaScript inline sont supprimées.
* IFrames : les balises `<iframe>` sont supprimées.
* Style : les balises `<style>` sont supprimées. Les attributs de style intraligne sur les éléments HTML ne sont pas officiellement pris en charge. Certains éléments de style intraligne peuvent sembler fonctionner, mais s’ils interfèrent avec la disposition du portail, ils peuvent cesser de fonctionner à tout moment. La vignette Markdown est destinée à du contenu statique de base qui utilise les styles par défaut du portail.

## <a name="next-steps"></a>Étapes suivantes

* Pour créer un tableau de bord personnalisé, consultez [Créer et partager des tableaux de bord dans le portail Azure](../azure-portal/azure-portal-dashboards.md).

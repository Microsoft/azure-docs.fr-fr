---
title: Disponibilité générale de Microsoft Threat Modeling Tool version du 12/09/2018
titleSuffix: Azure
description: Lisez les notes de publication pour Microsoft Threat Modeling Tool éditées le 12/09/2018. Les notes incluent des modifications de fonctionnalités et des correctifs de bogues.
author: jegeib
ms.author: jegeib
ms.service: security
ms.subservice: security-develop
ms.topic: article
ms.date: 01/15/2019
ms.openlocfilehash: 7b0afdd90f2a1413c7f8364fc4518c4d1116454a
ms.sourcegitcommit: d76108b476259fe3f5f20a91ed2c237c1577df14
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92913583"
---
# <a name="threat-modeling-tool-ga-release-71509112---9122018"></a>Version en disponibilité générale de Threat Modeling Tool v 7.1.50911.2 - 12/09/2018

Nous sommes heureux d’annoncer que Microsoft Threat Modeling Tool est maintenant disponible au téléchargement en tant que version en disponibilité générale (GA) prise en charge. Cette version contient des mises à jour importantes relatives à la confidentialité et à la sécurité, ainsi que des correctifs de bogues, des mises à jour de fonctionnalités et des améliorations de la stabilité. Les utilisateurs existants de la version Preview 2017 seront invités à se mettre à jour avec la version la plus récente via la technologie ClickOnce à l’ouverture du client. Pour les nouveaux utilisateurs de l’outil, vous pouvez [télécharger le client](https://aka.ms/threatmodelingtool).

Avec cette version, nous mettons fin au support de la version Preview 2017 et recommandons à tous les utilisateurs de la préversion de procéder à la mise à jour vers la version en disponibilité générale. À compter du 15 octobre 2018, nous allons définir la version minimale requise de ClickOnce pour Threat Modeling Tool ; tous les clients de la préversion devront se mettre à niveau.

Microsoft Threat Modeling Tool 2016, qui est disponible à partir du [Centre de téléchargement Microsoft](https://www.microsoft.com/en-us/download/details.aspx?id=49168), continue d’être pris en charge jusqu’au 1er octobre 2019 pour les correctifs de sécurité critiques uniquement.

## <a name="feature-changes"></a>Évolution des fonctionnalités

### <a name="azure-stencil-updates"></a>Mises à jour des stencils Azure

Des stencils Azure supplémentaires ainsi que leurs menaces et atténuations de risques associées ont été ajoutés à l’ensemble des stencils fournis avec cette version. Des modifications significatives ont été apportées en priorité à « Azure App Services », « Offres Azure Database » et « Stockage Azure ».

![Mises à jour des stencils Azure](./media/threat-modeling-tool-releases-71509112/tmt_azure_stencil_update-300x70.png)

### <a name="onedrive-integration-feature-removed"></a>Fonctionnalités d’intégration à OneDrive supprimées

Les fonctionnalités « Save To OneDrive » (Enregistrer dans OneDrive), « Open From OneDrive » (Ouvrir à partir de OneDrive) et « Share a Link » (Partager un lien) de la préversion ont été supprimées. Nous conseillons aux utilisateurs de OneDrive de se servir du client [OneDrive pour Windows](https://onedrive.live.com/about/en-us/download/) pour accéder à leurs fichiers stockés sur OneDrive, en utilisant les boîtes de dialogue classiques d’ouverture et d’enregistrement de fichier.

## <a name="notable-fixed-bugs-reported-by-customers"></a>Correction de bogues notables signalés par des clients

### <a name="in-tmt-preview-the-tool-crashes-when-using-the-standard-template"></a>Dans la préversion de TMT, l’outil plante lors de l’utilisation du modèle standard

- Quand un stencil générique, par exemple « Generic Data Flow » (flux de données génériques), est ajouté à la surface de dessin et génère des menaces, l’outil peut planter. Ce problème est à présent résolu.

### <a name="in-tmt-preview-when-i-save-a-report-or-copy-the-threats-the-risk-levels-are-incorrect"></a>Dans la préversion de TMT, lorsque j’enregistre un rapport ou que je copie les menaces, les niveaux de risque sont incorrects

- Si un utilisateur modifie le niveau Risk (risque) de menaces particulières, puis enregistre un rapport ou copie les risques, le niveau de risque peut revenir à « High » (élevé). Ce problème est à présent résolu.

## <a name="known-issues-and-faq"></a>Problèmes connus et FAQ

### <a name="users-of-high-resolution-screens-may-experience-small-text-in-the-threat-properties"></a>Les utilisateurs d’écrans haute résolution peuvent voir du texte affiché en petit dans les propriétés des menaces

#### <a name="issue"></a>Problème

Dans la vue d’analyse de l’outil, si l’utilisateur est équipé d’un écran haute résolution qui est défini par défaut sur l’utilisation de la Loupe pour une meilleure lisibilité dans Windows, la section « Possible Mitigation(s) » (Atténuation(s) des risques possible(s)) d’une menace peut apparaître en petits caractères.

![Problème connu avec les écrans haute résolution](./media/threat-modeling-tool-releases-71509112/tmt_screen_resolution-300x153.png)

#### <a name="workaround"></a>Solution de contournement

L’utilisateur peut cliquer sur le texte d’atténuation et utiliser le contrôle de zoom Windows standard (Ctrl+rotation de la roulette de la souris vers le haut) pour augmenter l’agrandissement de cette section.

### <a name="files-in-the-recently-opened-models-section-of-the-main-window-may-fail-to-open"></a>Les fichiers de la section « Recently Opened Models » (Modèles récemment ouverts) dans la fenêtre principale peuvent ne pas s’ouvrir

#### <a name="issue"></a>Problème

La fonctionnalité « Open From OneDrive » (Ouvrir à partir de OneDrive) de la préversion a été supprimée. Les utilisateurs disposant de « modèles récemment ouverts » enregistrés dans OneDrive recevront l’erreur suivante.

![Capture d’écran d’une erreur : Erreur à l’ouverture du fichier – Référence d’objet non définie comme une instance d’un objet](./media/threat-modeling-tool-releases-71509112/tmt_save_error-300x131.png)

#### <a name="workaround"></a>Solution de contournement

Nous conseillons aux utilisateurs de OneDrive de se servir du client [OneDrive pour Windows](https://onedrive.live.com/about/en-us/download/) pour accéder à leurs fichiers stockés sur OneDrive, en utilisant la boîte de dialogue classique « d’ouverture d’un modèle ».

![Capture d’écran de OneDrive sélectionné dans la boîte de dialogue Ouvrir un modèle](./media/threat-modeling-tool-releases-71509112/tmt_save_onedrive-300x149.png)

### <a name="my-organization-uses-the-2016-version-of-the-tool-can-i-use-the-azure-stencil-set"></a>Mon organisation utilise la version 2016 de l’outil, puis-je utiliser le jeu de stencils Azure ?

Oui, vous pouvez ! L’[ensemble de stencils Azure est disponible sur GitHub](https://github.com/Microsoft/threat-modeling-templates/) et peut être chargé dans la version 2016 de l’outil. Pour créer un modèle avec le jeu de stencils Azure, utilisez la boîte de dialogue « Template For New Models » (Modèle pour les nouveaux modèles) dans l’écran du menu principal. TMT 2016 ne peut pas restituer les liens trouvés dans les champs « Possible Mitigations » (Atténuations des risques possibles) du jeu de stencils Azure, par conséquent vous pouvez voir des liens s’afficher sous forme de balises HTML.

![Mises à jour des stencils Azure dans le client 2016](./media/threat-modeling-tool-releases-71509112/tmt_azure_stencils-300x212.png)

## <a name="system-requirements"></a>Configuration système requise

- Systèmes d’exploitation pris en charge
  - Microsoft Windows 10
- Version .NET obligatoire
  - .Net 3.5.2
- Autres conditions requises
  - Une connexion Internet est nécessaire pour recevoir les mises à jour de l’outil ainsi que les modèles.

## <a name="documentation-and-feedback"></a>Documentation et commentaires

- La documentation de Threat Modeling Tool se trouve sur [docs.microsoft.com](threat-modeling-tool.md) et contient des informations [sur l’utilisation de l’outil](threat-modeling-tool-getting-started.md).

## <a name="next-steps"></a>Étapes suivantes

Télécharger la version la plus récente de [Microsoft Threat Modeling Tool](https://aka.ms/threatmodelingtool).

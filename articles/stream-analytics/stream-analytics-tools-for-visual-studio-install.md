---
title: Configurer Azure Stream Analytics Tools pour Visual Studio
description: Cet article décrit les conditions requises à l’installation des outils Azure Stream Analytics pour Visual Studio et comment configurer ces outils.
author: su-jie
ms.author: sujie
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 05/22/2018
ms.openlocfilehash: 464e8fbf624dfa84ec54bf6a7e0ce202d047e15d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86045108"
---
# <a name="install-azure-stream-analytics-tools-for-visual-studio"></a>Installer Azure Stream Analytics Tools pour Visual Studio

Visual Studio 2019 et Visual Studio 2017 prennent en charge les Outils Azure Data Lake et Stream Analytics. Cet article décrit comment installer et désinstaller ces outils.

Pour plus d’informations sur l’utilisation des outils, consultez [Démarrage rapide : Créer une tâche Azure Stream Analytics à l’aide de Visual Studio](stream-analytics-quick-create-vs.md).

## <a name="install"></a>Installer

Les éditions Visual Studio Enterprise (Ultimate/Premium), Professional et Community prennent en charge les outils. L’édition Express et Visual Studio pour Mac ne les prennent pas en charge.

Nous vous recommandons d’utiliser Visual Studio 2019.

### <a name="install-for-visual-studio-2019-and-2017"></a>Installer pour Visual Studio 2019 et 2017<a name="recommended-visual-studio-2019-and-2017"></a>

Les Outils Azure Data Lake et Stream Analytics font partie des charges de travail **Développement Azure** et **Stockage et traitement des données**. Activez l’une de ces deux charges de travail au cours de l’installation. Si Visual Studio est déjà installé, sélectionnez **Outils** > **Obtenir les outils et fonctionnalités** pour ajouter des charges de travail.

Téléchargez [Visual Studio 2019 (Preview 2 ou version ultérieure) ou Visual Studio 2017 (15.3 ou version ultérieure)](https://www.visualstudio.com/) et suivez les instructions d’installation.

Sélectionnez la charge de travail **Stockage et traitement des données** comme indiqué :

![La charge de travail Stockage et traitement des données est sélectionnée.](./media/stream-analytics-tools-for-visual-studio-install/stream-analytics-tools-for-vs-2019-install-01.png)

Sélectionnez la charge de travail **Développement Azure** comme indiqué :

![La charge de travail Développement Azure est sélectionnée.](./media/stream-analytics-tools-for-visual-studio-install/stream-analytics-tools-for-vs-2019-install-02.png)

Après avoir ajouté la charge de travail, mettez à jour les outils. Cette procédure fait référence à Visual Studio 2019 :

1. Sélectionnez **Extensions** > **Gérer les extensions**.

1. Dans **Gérer les extensions**, sélectionnez **Mises à jour** et choisissez **Outils Azure Data Lake et Stream Analytics**.

1. Sélectionnez **Mettre à jour** pour installer l’extension la plus récente.

![Extensions et mises à jour Visual Studio](./media/stream-analytics-tools-for-visual-studio-install/stream-analytics-tools-vs2019-extensions-updates.png)

### <a name="install-for-visual-studio-2015-and-2013"></a>Installer pour Visual Studio 2015 et 2013<a name="visual-studio-2015-2013"></a>

Les éditions Visual Studio Enterprise (Ultimate/Premium), Professional et Community prennent en charge les outils. L’édition Express ne les prend pas en charge.

* Installez Visual Studio 2015 ou Visual Studio 2013 Update 4.
* Installez le kit de développement logiciel Microsoft Azure pour .NET version 2.7.1 ou ultérieure à l’aide de [Web Platform Installer](https://www.microsoft.com/web/downloads/platform.aspx).
* Installez [Microsoft Azure Data Lake and Stream Analytics Tools pour Visual Studio](https://www.microsoft.com/en-us/download/details.aspx?id=49504).

## <a name="update"></a>Mettre à jour<a name="visual-studio-2019-and-2017"></a><a name="visual-studio-2015-and-2013"></a>

Pour Visual Studio 2019 et Visual Studio 2017, un nouveau rappel de version est affiché sous forme de notification Visual Studio.

Pour Visual Studio 2015 et Visual Studio 2013, les outils vérifient automatiquement l’existence de nouvelles versions. Suivez les instructions pour installer la dernière version.

## <a name="uninstall"></a>Désinstaller l’interface

Vous pouvez désinstaller les Outils Azure Data Lake et Stream Analytics. Pour Visual Studio 2019 ou Visual Studio 2017, sélectionnez **Outils** > **Obtenir les outils et fonctionnalités**. Dans **Modification**, désélectionnez **Outils Azure Data Lake et Stream Analytics**. Les outils apparaissent sous la charge de travail **Stockage et traitement des données** ou **Développement Azure**.

Pour désinstaller les outils de Visual Studio 2015 ou Visual Studio 2013, accédez à **Panneau de configuration** > **Programmes et fonctionnalités**. Désinstallez **Microsoft Azure Data Lake and Stream Analytics Tools pour Visual Studio**.

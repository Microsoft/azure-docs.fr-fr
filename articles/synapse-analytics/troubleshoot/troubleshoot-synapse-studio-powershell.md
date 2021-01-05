---
title: Résoudre les problèmes de connectivité de Synapse Studio
description: Résoudre les problèmes de connectivité Azure Synapse Studio à l’aide de PowerShell
author: saveenr
ms.service: synapse-analytics
ms.topic: conceptual
ms.date: 10/30/2020
ms.author: saveenr
ms.reviewer: jrasnick
ms.openlocfilehash: 5f0dc73877d7742d4fc6a0c5b9bcf0529d475e0a
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96445259"
---
# <a name="troubleshoot-synapse-studio-connectivity-with-powershell"></a>Résoudre les problèmes de connectivité de Synapse Studio à l'aide de PowerShell

Le bon fonctionnement d'Azure Synapse Studio s'appuie sur un ensemble de points de terminaison d'API web. Ce guide vous aidera à identifier les causes des problèmes de connectivité lorsque vous :
- configurez votre réseau local (réseau derrière un pare-feu d’entreprise, par exemple) pour accéder à Azure Synapse Studio.
- rencontrez des problèmes de connectivité avec Azure Synapse Studio.

## <a name="prerequisite"></a>Configuration requise

* PowerShell 5.0 ou version ultérieure sur Windows, ou
* PowerShell Core 6.0 ou version ultérieure sur Windows ou Linux.

## <a name="troubleshooting-steps"></a>Étapes de dépannage

Cliquez avec le bouton droit sur le lien suivant, puis sélectionnez « Enregistrer la cible sous » :

- [Test-AzureSynapse.ps1](https://go.microsoft.com/fwlink/?linkid=2119734)

Vous pouvez aussi ouvrir le lien directement, puis enregistrer le fichier de script ouvert. N’enregistrez pas l’adresse du lien ci-dessus car celle-ci peut être amenée à changer.

Dans l’Explorateur de fichiers, cliquez avec le bouton droit sur le fichier de script téléchargé, puis sélectionnez « Exécuter avec PowerShell ».

![Exécuter le fichier de script téléchargé avec PowerShell](media/troubleshooting-synapse-studio-powershell/run-with-powershell.png)

Lorsque vous y êtes invité, entrez le nom de l’espace de travail Azure Synapse qui rencontre un problème ou dont vous souhaitez tester la connectivité, puis appuyez sur Entrée.

![Entrer le nom de l’espace de travail](media/troubleshooting-synapse-studio-powershell/enter-workspace-name.png)

La session de diagnostic démarre. Patientez jusqu’à la fin de son exécution.

![Attendre la fin du diagnostic](media/troubleshooting-synapse-studio-powershell/wait-for-diagnosis.png)

Une fois le diagnostic terminé, un résumé s'affiche. Si votre ordinateur n'est pas en mesure de se connecter à un ou plusieurs points de terminaison, des suggestions s’affichent dans la section « Résumé ».

![Examiner le résumé de diagnostic](media/troubleshooting-synapse-studio-powershell/diagnosis-summary.png)

En outre, un fichier journal de diagnostic correspondant à cette session est généré dans le même dossier que le script de résolution des problèmes. Son emplacement est indiqué dans la section « Conseils généraux » (`D:\TestAzureSynapse_2020....log`). Si nécessaire, vous pouvez envoyer ce fichier au support technique.

En tant qu'administrateur réseau, lorsque vous ajustez la configuration de votre pare-feu pour Azure Synapse Studio, les détails techniques qui s'affichent au-dessus de la section « Résumé » peuvent vous être utiles.

* Tous les éléments de test (requêtes) marqués avec « Réussi » indique que les tests de connectivité ont abouti, quel que soit le code d’état HTTP.
 En cas d'échec, la raison est indiquée en jaune, par exemple `NamedResolutionFailure` ou `ConnectFailure`. Ces raisons peuvent vous aider à déterminer d'éventuelles erreurs de configuration liées à votre environnement réseau.


## <a name="next-steps"></a>Étapes suivantes
Si les étapes précédentes ne permettent pas de résoudre le problème, [créez un ticket de support](../../sql-data-warehouse/sql-data-warehouse-get-started-create-support-ticket.md).

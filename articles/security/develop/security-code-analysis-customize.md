---
title: Personnaliser les tâches Microsoft Security Code Analysis
titleSuffix: Azure
description: Cet article décrit la personnalisation des tâches dans l’extension Microsoft Security Code Analysis
author: sukhans
manager: sukhans
ms.author: terrylan
ms.date: 07/31/2019
ms.topic: article
ms.service: security
services: azure
ms.assetid: 521180dc-2cc9-43f1-ae87-2701de7ca6b8
ms.devlang: na
ms.tgt_pltfrm: na
ms.workload: na
ms.openlocfilehash: 4016e1dd055b45f9cd59a172d0e71ef95fec1c40
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96008682"
---
# <a name="configure-and-customize-the-build-tasks"></a>Configurer et personnaliser les tâches de build

Cet article décrit en détail les options de configuration disponibles dans chacune des tâches de build. Il commence par les tâches correspondant aux outils d’analyse du code de sécurité. Il se termine par les tâches post-traitement.

## <a name="anti-malware-scanner-task"></a>Tâche Analyseur anti-programme malveillant

>[!NOTE]
> La tâche de build Analyseur anti-programme malveillant nécessite un agent de build avec Windows Defender activé. Visual Studio 2017 hébergé et versions ultérieures fournissent un tel agent. La tâche de build ne s’exécute pas sur l’agent hébergé Visual Studio 2015.
>
> Bien qu'il ne soit pas possible de mettre à jour les signatures sur ces agents, elles doivent toujours être antérieures à trois heures.

Les détails de configuration des tâches sont affichés dans la capture d’écran et le texte suivants.

![Configuration de la tâche de build Analyseur anti-programme malveillant](./media/security-tools/5-add-anti-malware-task600.png)

Dans la zone de liste **Type** de la capture d'écran, l'option **De base** est sélectionnée. Sélectionnez **Personnaliser** pour fournir des arguments de ligne de commande qui personnalisent l’analyse.

Windows Defender utilise le client Windows Update pour télécharger et installer les signatures. Si la mise à jour de la signature échoue sur votre agent de build, le code d’erreur **HRESULT** provient probablement de Windows Update.

Pour plus d’informations sur les erreurs de Windows Update et leur atténuation, consultez [, Codes d'erreur de Windows Update par composant](/windows/deployment/update/windows-update-error-reference) et l'article TechNet [Agent Windows Update - Codes d'erreur](https://social.technet.microsoft.com/wiki/contents/articles/15260.windows-update-agent-error-codes.aspx).

Pour plus d’informations sur la configuration YAML de cette tâche, consultez nos [Options YAML pour les logiciels anti-programme malveillant](yaml-configuration.md#anti-malware-scanner-task)

## <a name="binskim-task"></a>Tâche BinSkim

> [!NOTE]
> Avant d'exécuter la tâche BinSkim, votre build doit respecter l’une des conditions suivantes :
>  - Votre build produit des artefacts binaires à partir de code managé.
>  - Vous avez des artefacts binaires validés que vous souhaitez analyser avec BinSkim.

Les détails de configuration des tâches sont affichés dans la capture d’écran et la liste suivantes.

![Configuration de la tâche de build BinSkim](./media/security-tools/7-binskim-task-details.png)

- Définissez la configuration de build sur Debug pour produire des fichiers de débogage .pdb. BinSkim utilise ces fichiers pour mapper les problèmes des fichiers binaires de sortie vers le code source.
- Pour éviter de rechercher et de créer votre propre ligne de commande :
     - Dans la liste **Type**, sélectionnez **De base**.
     - Dans la liste **Fonction**, sélectionnez **Analyser**.
- Dans **Cible**, entrez un ou plusieurs spécificateurs pour un modèle de fichier, de répertoire ou de filtre. Ces spécificateurs se résolvent en un ou plusieurs fichiers binaires à analyser :
    - Les cibles multiples spécifiées doivent être séparées par un point-virgule (;).
    - Un spécificateur peut être un seul fichier ou contenir des caractères génériques.
    - Les spécifications de répertoire doivent toujours se terminer par \\*.
    - Exemples :

```binskim-targets
           *.dll;*.exe
           $(BUILD_STAGINGDIRECTORY)\*
           $(BUILD_STAGINGDIRECTORY)\*.dll;$(BUILD_STAGINGDIRECTORY)\*.exe;
```

- Si vous sélectionnez **Ligne de commande** dans la liste **Type**, vous devez exécuter binskim.exe :
     - Vérifiez que les premiers arguments de binskim.exe correspondent au verbe **analyser** suivi d’une ou plusieurs spécifications de chemin d’accès. Chaque chemin d’accès peut correspondre à un chemin d’accès complet ou à un chemin d’accès relatif au répertoire source.
     - Plusieurs chemins d’accès cibles doivent être séparés par un espace.
     - Vous pouvez omettre l'option **/o** ou **/output**. La valeur de sortie est ajoutée pour vous ou remplacée.
     - Les configurations de ligne de commande standard sont indiquées comme suit.

```binskim-line-args
           analyze $(Build.StagingDirectory)\* --recurse --verbose
           analyze *.dll *.exe --recurse --verbose
```

> [!NOTE]
> La fin \\* est importante si vous spécifiez des répertoires pour la cible.

Pour plus d’informations sur les arguments de ligne de commande, les règles par ID ou les codes de sortie pour BinSkim, consultez le [Guide de l’utilisateur BinSkim](https://github.com/Microsoft/binskim/blob/master/docs/UserGuide.md).

Pour plus d’informations sur la configuration YAML de cette tâche, consultez nos [Options YAML pour BinSkim](yaml-configuration.md#binskim-task)

## <a name="credential-scanner-task"></a>Tâche Credential Scanner

Les détails de configuration des tâches sont affichés dans la capture d’écran et la liste suivantes.

![Configuration de la tâche de build Credential Scanner](./media/security-tools/3-taskdetails.png)

Les options disponibles sont les suivantes :
  - **Nom d’affichage** : Nom de la tâche Azure DevOps. La valeur par défaut est Exécuter Credential Scanner
  - **Version principale de l'outil** : Les valeurs disponibles sont **CredScan v2**, **CredScan v1**. Nous recommandons aux clients d’utiliser la version **CredScan v2**.
  - **Format de sortie** : Les valeurs disponibles incluent **TSV**, **CSV**, **SARIF** et **PREfast**.
  - **Version de l'outil** : Nous vous recommandons de sélectionner **La plus récente**.
  - **Dossier d'analyse** : Dossier de référentiel à analyser.
  - **Type de fichier des chercheurs** : Options pour localiser le fichier des chercheurs utilisé pour l’analyse.
  - **Fichier des suppressions** : Un fichier [JSON](https://json.org/) peut supprimer des problèmes dans le journal de sortie. Pour plus d’informations sur les scénarios de suppression, consultez la section FAQ de cet article.
  - **Sortie détaillée** : Explicite.
  - **Taille du lot** : Nombre de threads simultanés utilisés pour exécuter Credential Scanner. La valeur par défaut est 20. Les valeurs possibles sont comprises entre 1 et 2,147,483,647.
  - **Délai d’attente d’une correspondance** : Quantité de temps à passer à essayer une mise en correspondance d’un chercheur avant d’abandonner la vérification.
  - **Taille de la mémoire tampon de lecture de l’analyse des fichiers** : Taille en octets de la mémoire tampon pour la lecture du contenu. La valeur par défaut est 524 288.  
  - **Nombre maximal d’octets lus pour l’analyse des fichiers** : Nombre maximal d’octets à lire dans un fichier donné lors de l’analyse du contenu. La valeur par défaut 104,857,600.
  - **Options de contrôle** > **Exécuter cette tâche** : Spécifie le moment où la tâche s'exécutera. Sélectionnez **Conditions personnalisées** pour spécifier des conditions plus complexes.
  - **Version** : Version de la tâche de build dans Azure DevOps. Cette option n’est pas fréquemment utilisée.

Pour plus d’informations sur la configuration YAML de cette tâche, consultez nos [Options YAML pour Credential Scanner](yaml-configuration.md#credential-scanner-task)

## <a name="roslyn-analyzers-task"></a>Tâche Analyseurs Roslyn

> [!NOTE]
> Avant d'exécuter la tâche Analyseurs Roslyn, votre build doit respecter les conditions suivantes :
>
> - Votre définition de build comprend la tâche de build intégrée MSBuild ou VSBuild pour compiler du code C# ou code Visual Basic. La tâche des analyseurs s’appuie sur l’entrée et la sortie de la tâche intégrée pour exécuter la compilation MSBuild avec les analyseurs Roslyn activés.
> - Visual Studio 2017 version 15.15.5 ou ultérieure est installée sur l’agent de build qui exécute cette tâche de build de manière à ce qu'il utilise le compilateur version 2.6 ou ultérieure.

Les détails de configuration des tâches sont affichés dans la liste et la remarque suivantes.

Les options disponibles sont les suivantes :

- **Ruleset** : Les valeurs correspondent à **SDL obligatoire**, **SDL recommandé** ou votre propre ensemble de règles personnalisées.
- **Version des analyseurs** : Nous vous recommandons de sélectionner **La plus récente**.
- **Fichier des suppressions des avertissements du compilateur** : Fichier texte avec la liste des ID des avertissements supprimés.
- **Options de contrôle** > **Exécuter cette tâche** : Spécifie le moment où la tâche s'exécutera. Sélectionnez **Conditions personnalisées** pour spécifier des conditions plus complexes.

> [!NOTE]
>
> - Les analyseurs Roslyn sont intégrés au compilateur et peuvent être exécutés seulement dans le cadre de la compilation csc.exe. Par conséquent, cette tâche nécessite la relecture/réexécution de la commande du compilateur exécutée précédemment dans la build. Cette relecture ou exécution s’effectue en interrogeant Azure DevOps (anciennement Visual Studio Team Services) pour les journaux des tâches de build MSBuild.
>
>   Il n’existe aucun autre moyen pour la tâche d’obtenir de façon fiable la ligne de commande de compilation MSBuild à partir de la définition de build. Nous avons envisagé d’ajouter une zone de texte de forme libre pour permettre aux utilisateurs d’entrer leurs lignes de commande. Mais il serait difficile de les maintenir à jour et de les synchroniser avec la build principale.
>
>   Les builds personnalisées nécessitent la relecture de l’ensemble des commandes, et pas seulement celle des commandes du compilateur. Et dans ce cas, il n’est ni évident ni fiable d’activer les analyseurs Roslyn.
>
> - Les analyseurs Roslyn sont intégrés au compilateur. Pour être appelés, les analyseurs Roslyn nécessitent une compilation.
>
>   Cette nouvelle tâche de build est implémentée par recompilation des projets C# qui ont déjà été générés. La nouvelle tâche utilise uniquement les tâches de build MSBuild et VSBuild dans la même build ou définition de build que la tâche d’origine. Toutefois, dans ce cas, la nouvelle tâche les utilise avec les analyseurs Roslyn activés.
>
>   Si la nouvelle tâche s’exécute sur le même agent que la tâche d’origine, la sortie de la nouvelle tâche remplace la sortie de la tâche d'origine dans le dossier de sources *s*. Bien que la sortie de la build soit la même, nous vous conseillons d’exécuter MSBuild, de copier la sortie dans le répertoire intermédiaire des artefacts, puis d’exécuter les analyseurs Roslyn.

Pour obtenir des ressources supplémentaires pour la tâche Analyseurs Roslyn, consultez [Analyseurs Roslyn](/dotnet/standard/analyzers/api-analyzer) sur Microsoft Docs.

Pour connaître le package de l’analyseur installé et utilisé par cette tâche de build, consultez la page NuGet [Microsoft.CodeAnalysis.FxCopAnalyzers](https://www.nuget.org/packages/Microsoft.CodeAnalysis.FxCopAnalyzers).

Pour plus d’informations sur la configuration YAML de cette tâche, consultez nos [Options YAML pour analyseurs Roslyn](yaml-configuration.md#roslyn-analyzers-task)

## <a name="tslint-task"></a>Tâche TSLint

Pour plus d’informations sur TSLint, accédez au [référentiel GitHub TSLint](https://github.com/palantir/tslint).

>[!NOTE] 
>Comme vous devez le savoir, la page d'accueil du [référentiel GitHub TSLint](https://github.com/palantir/tslint) indique que TSLint sera déconseillé en 2019. Microsoft examine la possibilité d'utiliser [ESLint](https://github.com/eslint/eslint) en guise d'alternative.

Pour plus d’informations sur la configuration YAML de cette tâche, consultez nos [Options YAML pour TSLint](yaml-configuration.md#tslint-task)

## <a name="publish-security-analysis-logs-task"></a>Tâche Publier les journaux d’analyse de la sécurité

Les détails de configuration des tâches sont affichés dans la capture d’écran et la liste suivantes.

![Configuration de la tâche de build Publier les journaux d’analyse de la sécurité](./media/security-tools/9-publish-security-analsis-logs600.png)  

- **Nom de l’artefact** : N’importe quel identificateur de chaîne.
- **Type d’artefact** : Selon votre sélection, vous pouvez publier des journaux sur votre Azure DevOps Server ou sur un fichier partagé accessible à l’agent de build.
- **Outils** : Vous pouvez choisir de conserver les journaux pour des outils spécifiques ou sélectionner **Tous les outils** pour conserver tous les journaux.

Pour plus d’informations sur la configuration YAML de cette tâche, consultez nos [Options YAML pour la tâche Publier des journaux de sécurité](yaml-configuration.md#publish-security-analysis-logs-task)

## <a name="security-report-task"></a>Tâche Rapport de sécurité

Les détails de configuration du rapport de sécurité sont affichés dans la capture d’écran et la liste suivantes.

![Configuration de la tâche de build Rapport de sécurité](./media/security-tools/4-createsecurityanalysisreport600.png)

- **Rapports** : Sélectionnez un des formats suivants : **Console pipeline**, **Fichier TSV** et **Fichier Html**. Un fichier de rapport est créé pour chaque format sélectionné.
- **Outils** : Sélectionnez les outils de votre définition de build pour lesquels vous souhaitez obtenir un résumé des problèmes détectés. Pour chaque outil sélectionné, il peut y avoir une option pour choisir si vous souhaitez voir dans le rapport seulement les erreurs, ou bien les erreurs et les avertissements.
- **Options avancées** : S'il n’existe pas de journal pour un des outils sélectionnés, vous pouvez choisir de consigner un avertissement ou une erreur. Si vous consignez une erreur, la tâche échoue.
- **Dossier des journaux de base** : Vous pouvez personnaliser le dossier des journaux de base où les journaux doivent se trouver. Cette option n'est cependant pas courante.

Pour plus d’informations sur la configuration YAML de cette tâche, consultez nos [Options YAML pour les rapports de sécurité](yaml-configuration.md#security-report-task)

## <a name="post-analysis-task"></a>Tâche Post-analyse

Les détails de configuration des tâches sont affichés dans la capture d’écran et la liste suivantes.

![Configuration de la tâche de build Post-analyse](./media/security-tools/a-post-analysis600.png)

- **Outils** : Sélectionnez les outils de votre définition de build pour lesquels vous souhaitez injecter sous condition un arrêt de la build. Pour chaque outil sélectionné, il peut y avoir une option pour choisir si vous souhaitez interrompre seulement pour les erreurs, ou bien pour les erreurs et les avertissements.
- **Rapport** : Vous pouvez écrire les résultats qui provoquent l’arrêt de la build. Les résultats sont écrits dans la fenêtre de la console Azure DevOps et dans le fichier journal.
- **Options avancées** : S'il n’existe pas de journal pour un des outils sélectionnés, vous pouvez choisir de consigner un avertissement ou une erreur. Si vous consignez une erreur, la tâche échoue.

Pour plus d’informations sur la configuration YAML de cette tâche, consultez nos [Options YAML pour l’analyse postérieure](yaml-configuration.md#post-analysis-task)

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la configuration basée sur YAML, reportez-vous à notre [guide de configuration YAML](yaml-configuration.md).

Si vous avez d'autres questions sur l'extension Microsoft Security Code Analysis et les outils proposés, consultez notre [page Forum aux questions](security-code-analysis-faq.md).
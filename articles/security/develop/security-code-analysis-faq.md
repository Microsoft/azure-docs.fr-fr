---
title: Forum aux questions de la documentation de Microsoft Security Code Analysis
description: Pour en savoir plus sur l’extension Analyse du code de sécurité Microsoft, consultez le Forum aux questions (FAQ).
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
ms.openlocfilehash: f872159e538dc85121a7a6d4d6503fd18a263628
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87543042"
---
# <a name="frequently-asked-questions"></a>Forum aux questions
Vous avez des questions ? Pour plus d’informations, consultez le forum aux questions suivant.

## <a name="general-faq"></a>FAQ général

### <a name="can-i-install-the-extension-on-my-visual-studio-team-foundation-server-instance-instead-of-on-an-azure-devops-instance"></a>Puis-je installer l’extension sur mon instance Visual Studio Team Foundation Server plutôt que sur une instance Azure DevOps ?

Non. L’extension n’est pas disponible pour le téléchargement et l’installation pour Visual Studio Team Foundation Server.

### <a name="do-i-have-to-run-microsoft-security-code-analysis-with-my-build"></a>Dois-je exécuter Microsoft Security Code Analysis avec ma build ? 

Peut-être. Tout dépend du type d’outil d’analyse. Le code source peut être l'unique élément requis, ou la sortie de build peut être également requise.

Par exemple, Credential Scanner (CredScan) analyse les fichiers au sein de la structure de dossiers du référentiel de code. En raison de cette analyse, vous pouvez exécuter les tâches de build CredScan et Publier les journaux d’analyse de la sécurité dans une build autonome pour obtenir des résultats.

Pour les autres outils qui analysent les artefacts, comme BinSkim, la build est d’abord nécessaire.

### <a name="can-i-break-my-build-when-results-are-found"></a>Puis-je interrompre ma build quand des résultats sont trouvés ?

Oui. Vous pouvez introduire un arrêt de la build quand un outil signale un problème dans son fichier journal. Ajoutez simplement la tâche de build Post-Analyse et cochez la case correspondant aux outils pour lesquels vous souhaitez interrompre la build.

Dans l’interface utilisateur de la tâche Post-Analysis, vous pouvez choisir d’interrompre la build lorsqu'un outil signale des erreurs uniquement ou des erreurs et des avertissements.

### <a name="how-do-the-command-line-arguments-in-azure-devops-differ-from-those-arguments-in-the-standalone-desktop-tools"></a>En quoi les arguments de ligne de commande dans Azure DevOps sont-ils différents des arguments dans les outils de poste de travail autonomes ? 

Pour l’essentiel, les tâches de build Azure DevOps sont des wrappers directs autour des arguments de ligne de commande des outils de sécurité. Vous pouvez transmettre sous forme d'arguments à une tâche de build tout ce que vous transmettez habituellement à un outil en ligne de commande.

Différences notables :

- Les outils s'exécutent à partir du dossier source de l’agent $(Build.SourcesDirectory) ou %BUILD_SOURCESDIRECTORY%. Exemple : C:\agent\_work\1\s.
- Les chemins d’accès des arguments peuvent être relatifs à la racine du répertoire source précédemment répertorié. Les chemins d’accès peuvent également être absolus. Vous obtenez les chemins d’accès absolus à l’aide des variables de build Azure DevOps ou en exécutant un agent local avec des emplacements de déploiement connus des ressources locales.
- Les outils fournissent automatiquement un chemin d'accès ou un dossier de fichier de sortie. Si vous fournissez un emplacement de sortie pour une tâche de build, cet emplacement est remplacé par le chemin d’accès à un emplacement connu des journaux sur l’agent de build
- D'autres arguments de ligne de commande sont modifiés pour certains outils. À titre d'exemple, citons l'ajout ou la suppression d'options pour veiller à ce qu’aucune interface graphique utilisateur ne soit lancée.

### <a name="can-i-run-a-build-task-like-credential-scanner-across-multiple-repositories-in-an-azure-devops-build"></a>Puis-je exécuter une tâche de build (comme Credential Scanner) sur plusieurs dépôts dans une build Azure DevOps ?

Non. L’exécution des outils de développement sécurisé sur plusieurs dépôts dans un même pipeline n’est pas prise en charge.

### <a name="the-output-file-i-specified-isnt-being-created-or-i-cant-find-the-output-file-i-specified"></a>Le fichier de sortie que j’ai spécifié n’est pas créé ou le fichier de sortie que j’ai spécifié est introuvable

Les tâches de build filtrent certaines entrées utilisateur. Pour ce faire, elles mettent à jour l’emplacement du fichier de sortie généré de sorte qu'il corresponde à un emplacement commun sur l’agent de build. Pour plus d’informations sur cet emplacement, consultez les questions suivantes.

### <a name="where-are-the-output-files-generated-by-the-tools-saved"></a>Où les fichiers de sortie générés par les outils sont-ils enregistrés ? 

Les tâches de build ajoutent automatiquement des chemins de sortie à l’emplacement connu sur l’agent de build : $(Agent.BuildDirectory)\_sdt\log. Cet emplacement étant normalisé, toutes les équipes qui produisent ou utilisent des journaux d’analyse de code ont accès à la sortie.

### <a name="can-i-queue-a-build-to-run-these-tasks-on-a-hosted-build-agent"></a>Puis-je mettre en file d’attente une build pour exécuter ces tâches sur un agent de build hébergé ? 

Oui. Toutes les tâches et tous les outils de l’extension peuvent être exécutés sur un agent de build hébergé.

>[!NOTE]
> La tâche de build Analyseur anti-programme malveillant nécessite un agent de build avec Windows Defender activé. Visual Studio 2017 hébergé et versions ultérieures fournissent un tel agent. La tâche de build ne s’exécute pas sur l’agent hébergé Visual Studio 2015.
>
> Bien qu'il ne soit pas possible de mettre à jour les signatures sur ces agents, elles doivent toujours être antérieures à trois heures.

### <a name="can-i-run-these-build-tasks-as-part-of-a-release-pipeline-as-opposed-to-a-build-pipeline"></a>Puis-je exécuter ces tâches de build dans le cadre d’un pipeline de mise en production (par opposition à un pipeline de build) ?

Dans la plupart des cas, oui.

Toutefois, Azure DevOps ne prend pas en charge l’exécution de tâches dans les pipelines de mise en production lorsque ces tâches publient des artefacts. Cette absence de prise en charge empêche l’exécution correcte de la tâche « Publier les journaux d’analyse de la sécurité » à partir d’un pipeline de mise en production. La tâche échoue avec un message d’erreur descriptif.

### <a name="from-where-do-the-build-tasks-download-the-tools"></a>Où les tâches de build téléchargent les outils ?

Les tâches de build peuvent télécharger les packages NuGet des outils à partir du [flux de gestion des packages Azure DevOps](https://securitytools.pkgs.visualstudio.com/_packaging/SecureDevelopmentTools/nuget/v3/index.json). Les tâches de build peuvent également utiliser le gestionnaire de package de nœud, qui doit être préinstallé sur l’agent de build. La commande **npm install tslint** illustre une telle installation.

### <a name="what-effect-does-installing-the-extension-have-on-my-azure-devops-organization"></a>Quel est l’effet de l’installation de l’extension sur mon organisation Azure DevOps ? 

Après installation, les tâches de build de sécurité fournies par l’extension peuvent être utilisées par tous les utilisateurs de votre organisation. Lorsque vous créez ou modifiez un pipeline Azure, ces tâches sont disponibles à partir de la liste des collections de tâches de génération. À part cela, l’installation de l’extension dans votre organisation Azure DevOps n’a aucun effet. L’installation ne modifie aucun paramètre de compte, paramètre de projet ou pipeline.

### <a name="does-installing-the-extension-modify-my-existing-azure-pipelines"></a>L’installation de l’extension modifie-t-elle mes pipelines Azure existants ? 

Non. L’installation de l’extension permet l'ajout des tâches de build de sécurité disponibles à vos pipelines Azure. Vous devez toujours ajouter ou mettre à jour les définitions de build pour permettre le bon fonctionnement des outils avec votre processus de build.

## <a name="task-specific-faq"></a>FAQ spécifique aux tâches

Les questions spécifiques aux tâches de build sont répertoriées dans cette section.

### <a name="credential-scanner"></a>Credential Scanner

#### <a name="what-are-common-suppression-scenarios-and-examples"></a>Quels sont les scénarios et les exemples de suppression courants ?

Vous trouverez ci-dessous les détails des scénarios de suppression les plus courants.

##### <a name="to-suppress-all-occurrences-of-a-given-secret-within-the-specified-path"></a>Pour supprimer toutes les occurrences d’un secret donné dans le chemin spécifié

La clé de hachage du secret provenant du fichier de sortie de CredScan est nécessaire, comme le montre l’exemple suivant.

```js
{
    "tool": "Credential Scanner",
    "suppressions": [
    {
        "hash": "CLgYxl2FcQE8XZgha9/UbKLTkJkUh3Vakkxh2CAdhtY=",
        "_justification": "Secret used by MSDN sample, it is fake."
    }
  ]
}
```

>[!WARNING]
> La clé de hachage est générée par une partie de la valeur correspondante ou du contenu correspondant du fichier. Toute révision du code source peut modifier la clé de hachage et désactiver la règle de suppression.

##### <a name="to-suppress-all-secrets-in-a-specified-file-or-to-suppress-the-secrets-file-itself"></a>Pour supprimer tous les secrets d’un fichier spécifié ou supprimer le fichier de secrets lui-même

L’expression du fichier peut correspondre à un nom de fichier. Il peut également s’agir de la partie basename d’un chemin d’accès complet ou d'un nom de fichier. Les caractères génériques ne sont pas pris en charge.

Les exemples suivants montrent comment supprimer le fichier \<InputPath>\src\JS\lib\angular.js.

Exemples de règles de suppression valides :

- \<InputPath>\src\JS\lib\angular.js : supprime le fichier dans le chemin spécifié
- \src\JS\lib\angular.js
- \JS\lib\angular.js
- \lib\angular.js
- angular.js - supprime les fichiers portant le même nom

```js
{
    "tool": "Credential Scanner",
    "suppressions": [
    {
        "file": "\\files\\AdditonalSearcher.xml", 
        "_justification": "Additional CredScan searcher specific to my team"
    },
    {
        "file": "\\files\\unittest.pfx", 
        "_justification": "Legitimate UT certificate file with private key"
    }
  ]
}
```

>[!WARNING] 
> Tous les secrets futurs ajoutés au fichier seront également supprimés automatiquement.

#### <a name="what-are-recommended-guidelines-for-managing-secrets"></a>Quelles sont les recommandations en matière de gestion des secrets ?

Les ressources suivantes vous aident à gérer les secrets et à accéder à des informations sensibles à partir de vos applications :

 - [Azure Key Vault](../../key-vault/index.yml)
 - [Azure Active Directory (Azure AD)](../../azure-sql/database/authentication-aad-overview.md)
 - [Azure AD Managed Service Identity (MSI)](https://azure.microsoft.com/blog/keep-credentials-out-of-code-introducing-azure-ad-managed-service-identity/)
 - [Identités managées pour les ressources Azure](../../active-directory/managed-identities-azure-resources/overview.md)
 - [Identités managées dans App Service et Azure Functions](../../app-service/overview-managed-identity.md)
 - [Bibliothèque AppAuthentication](../../key-vault/general/service-to-service-authentication.md)


Pour plus d’informations, consultez le billet de blog sur la [gestion sécurisée des secrets dans le cloud](https://devblogs.microsoft.com/visualstudio/managing-secrets-securely-in-the-cloud/).

#### <a name="can-i-write-my-own-custom-searchers"></a>Puis-je écrire mes propres chercheurs personnalisés ?

Credential Scanner s’appuie sur un ensemble de chercheurs de contenu généralement définis dans le fichier buildsearchers.xml. Le fichier contient un tableau d’objets sérialisés XML qui représentent un objet **ContentSearcher**. Le programme est distribué avec un ensemble de chercheurs testés. Mais vous pouvez également implémenter vos propres chercheurs personnalisés.

Un chercheur de contenu est défini comme suit :

- **Name** : Nom descriptif du chercheur à utiliser dans les fichiers de sortie de Credential Scanner. Nous vous recommandons d’utiliser la convention de nommage avec une casse mixte pour les noms des chercheurs.
- **RuleId** : ID opaque stable du chercheur :
    - Un chercheur Credential Scanner par défaut se voit attribuer une valeur **RuleId** telle que CSCAN0010, CSCAN0020 ou CSCAN0030. Le dernier chiffre est réservé à la fusion ou à la division des groupes de chercheurs par le biais d’expressions régulières.
    - La valeur **RuleId** d'un chercheur personnalisé doit disposer de son propre espace de noms. Par exemple : CSCAN-\<Namespace\>0010, CSCAN-\<Namespace\>0020 et CSCAN-\<Namespace\>0030.
    - Le nom complet de chercheur correspond à une combinaison composée d'une valeur **RuleId** et du nom du chercheur. Par exemple : CSCAN0010.KeyStoreFiles and CSCAN0020.Base64EncodedCertificate.
- **ResourceMatchPattern** : Expression régulière des extensions de fichier à vérifier pour le chercheur.
- **ContentSearchPatterns** : Tableau de chaînes contenant les déclarations d’expressions régulières à mettre en correspondance. Si aucun modèle de recherche n’est défini, tous les fichiers correspondant à la valeur **ResourceMatchPattern** sont retournés.
- **ContentSearchFilters** : Tableau de chaînes contenant des déclarations d’expressions régulières pour filtrer les faux positifs spécifiques au chercheur.
- **MatchDetails** : Message descriptif, instructions d’atténuation, voire les deux, à ajouter pour chaque correspondance du chercheur.
- **Recommandation** : Fournit le contenu du champ des suggestions pour une correspondance avec le format de rapport PREfast.
- **Gravité** : Entier qui reflète le niveau de gravité d’un problème. Le niveau de gravité le plus élevé correspond à la valeur 1.

  ![XML affichant la configuration de Credential Scanner](./media/security-tools/6-credscan-customsearchers.png)

### <a name="roslyn-analyzers"></a>Analyseurs Roslyn

#### <a name="what-are-common-errors-when-using-the-roslyn-analyzers-task"></a>Quelles sont les erreurs courantes lors de l’utilisation de la tâche Analyseurs Roslyn ?

##### <a name="the-project-was-restored-using-a-wrong-microsoftnetcoreapp-version"></a>Le projet a été restauré à l’aide d’une version incorrecte de Microsoft.NETCore.App

Message d’erreur complet :

« Erreur : Le projet a été restauré en utilisant Microsoft.NETCore.App version *x.x.x*, mais avec les paramètres actuels, la version *y.y.y* est utilisée à la place. Pour résoudre ce problème, vérifiez que les mêmes paramètres sont utilisés pour la restauration et pour les opérations qui suivent, comme la génération ou la publication. En général, ce problème peut se produire si la propriété RuntimeIdentifier est définie lors de la génération ou de la publication, mais pas lors de la restauration.

Les tâches des analyseurs Roslyn s’exécutant dans le cadre de la compilation, l’arborescence source sur la machine de build doit être dans un état permettant une build.

Une étape entre votre build principale et les analyseurs Roslyn peut avoir mis l’arborescence source dans un état ne permettant pas une build. Cette étape supplémentaire est probablement **dotnet.exe publish**. Essayez de dupliquer l’étape qui effectue la restauration NuGet juste avant l’étape Analyseurs Roslyn. Cette étape dupliquée peut replacer l’arborescence source dans un état permettant une build.

##### <a name="cscexe-cant-create-an-analyzer-instance"></a>csc. exe ne peut pas créer d’instance d’analyseur

Message d’erreur complet :

« csc. exe » s’est terminé avec le code d’erreur 1 : impossible de créer une instance de l’analyseur *AAAA* à partir de C:\\*BBBB*.dll : Impossible de charger le fichier ou l’assembly ' Microsoft.CodeAnalysis, Version=*X.X.X.X*, Culture=neutral, PublicKeyToken=31bf3856ad364e35 ' ou une de ses dépendances. Le système ne peut pas trouver le fichier spécifié. »

Vérifiez que votre compilateur prend en charge les analyseurs Roslyn. L’exécution de la commande **csc.exe /version** doit signaler une valeur de version de 2.6 ou ultérieure.

Parfois, un fichier .csproj individuels peut remplacer l’installation de Visual Studio de l’ordinateur de build, en référençant un package à partir de Microsoft.Net.Compilers. Si vous n'envisagez pas d'utiliser une version spécifique du compilateur, supprimez les références à Microsoft.Net.Compilers. Sinon, vérifiez que la version du package référencé correspond aussi à la version 2.6 ou ultérieure.

Essayez d’accéder au chemin d’accès du journal des erreurs spécifié dans l'option **csc.exe /errorlog**. L’option et le chemin d’accès apparaissent dans le journal de la tâche de build Analyseurs Roslyn. Ils peuvent se présenter comme suit : **/errorlog:F:\ts-services-123\_work\456\s\Some\Project\Code\Code.csproj.sarif**

##### <a name="the-c-compiler-version-isnt-recent-enough"></a>La version du compilateur C# n’est pas assez récente

Pour obtenir les dernières versions du compilateur C#, accédez à [Microsoft.Net.Compilers](https://www.nuget.org/packages/Microsoft.Net.Compilers). Pour connaître la version installée, exécutez **csc.exe /version** à l’invite de commandes. Vérifiez que vous utilisez un package NuGet Microsoft.Net.Compilers correspondant à la version 2.6 ou ultérieure.

##### <a name="msbuild-and-vsbuild-logs-arent-found"></a>Les journaux MSBuild et VSBuild sont introuvables

La tâche de build Analyseurs Roslyn doit interroger Azure DevOps pour le journal MSBuild à partir de la tâche de build MSBuild. Si la tâche de l’analyseur s’exécute immédiatement après la tâche MSBuild, le journal ne sera pas disponible de suite. Placez d’autres tâches entre la tâche MSBuild et la tâche Analyseurs Roslyn. À titre d'exemple, il peut s'agir de BinSkim et Analyseur anti-programme malveillant.

## <a name="next-steps"></a>Étapes suivantes

Si vous avez besoin d’une aide supplémentaire, notez que le support de Microsoft Security Code Analysis est disponible du lundi au vendredi, de 9h00 à 17h00 (Heure standard du Pacifique).

- Intégration : Consultez notre [documentation sur l’intégration](security-code-analysis-onboard.md)
  
- Support : Envoyez un e-mail à notre équipe au [Support de Microsoft Security Code Analysis](mailto:mscahelp@microsoft.com?Subject=Microsoft%20Security%20Code%20Analysis%20Support%20Request)
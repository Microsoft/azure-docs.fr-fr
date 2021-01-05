---
title: Profilage d’un service cloud local dans l’émulateur de calcul | Microsoft Docs
services: cloud-services
description: Examen des problèmes de performances dans les services cloud à l’aide du profileur Visual Studio
documentationcenter: ''
author: mikejo
manager: jillfra
editor: ''
tags: ''
ms.assetid: 25e40bf3-eea0-4b0b-9f4a-91ffe797f6c3
ms.service: cloud-services
ms.workload: na
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 11/18/2016
ms.author: mikejo
ms.openlocfilehash: 6b5707405879c462a1d919e04730d368332ba68c
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92077153"
---
# <a name="testing-the-performance-of-a-cloud-service-locally-in-the-azure-compute-emulator-using-the-visual-studio-profiler"></a>Test des performances d'un service cloud local dans l'émulateur de calcul Azure avec le profileur Visual Studio
Différents outils et diverses techniques permettent de tester les performances des services cloud.
Quand vous publiez un service cloud sur Azure, vous pouvez demander à ce que Visual Studio collecte des données de profilage, puis les analyse en local, comme décrit dans la page [Analyse du profil d’une application Azure][1].
Vous pouvez également utiliser le diagnostic pour suivre tout un ensemble de compteurs de performances, comme décrit dans la rubrique [Utilisation de compteurs de performances dans Azure][2].
Vous pouvez également profiler votre application en local dans l'émulateur de calcul avant de la déployer dans le cloud.

Cet article présente la méthode de profilage par échantillonnage de l'UC, qui peut se faire en local dans l'émulateur. Cette méthode de profilage est peu intrusive. Selon une fréquence d'échantillonnage définie, le profileur enregistre un instantané de la pile d'appels. Les données sont collectées pendant un certain temps, puis sont présentées dans un rapport. Cette méthode de profilage indique plutôt, dans une application qui effectue beaucoup de calculs, où se fait la plus grande part du travail du processeur.  Ceci vous permet de vous occuper en priorité des « points chauds », là où votre application passe le plus de temps.

## <a name="1-configure-visual-studio-for-profiling"></a>1 : Configurer Visual Studio pour le profilage
Tout d'abord, certaines options de configuration de Visual Studio peuvent s'avérer utiles dans le cadre du profilage. Afin de bien comprendre les rapports de profilage, vous aurez besoin de symboles (fichiers .pdb) pour votre application, ainsi que de symboles pour les bibliothèques système. Assurez-vous que vous faites référence aux serveurs de symboles disponibles. Pour cela, dans le menu **Outils** de Visual Studio, sélectionnez **Options**, puis **Débogage**, et enfin **Symboles**. Assurez-vous que Microsoft Symbol Servers figure bien dans **Emplacements du fichier de symboles (.pdb)** .  Vous pouvez également référencer https://referencesource.microsoft.com/symbols, qui peut avoir des fichiers de symboles supplémentaires.

![Options de symbole][4]

Si vous le souhaitez, vous pouvez simplifier les rapports générés par le profileur en choisissant Uniquement mon code. Lorsque cette option est activée, les piles d'appels de fonction sont simplifiées afin que les appels purement internes aux bibliothèques et à .NET Framework soient masqués dans les rapports. Dans le menu **Outils**, choisissez **Options**. Développez le nœud **Outils de performances** et choisissez **Général**. Activez la case à cocher **Activer Uniquement mon code pour les rapports du profileur**.

![Options Uniquement mon code][17]

Vous pouvez utiliser ces instructions dans un projet existant ou un nouveau projet.  Si vous créez un projet pour tester une des techniques décrites plus bas, choisissez un projet C# **Service cloud Azure** et sélectionnez un **rôle web** et un **rôle de travail**.

![Rôles de projet Azure Cloud Services][5]

Pour l'exemple, ajoutez à votre projet du code qui demande beaucoup de temps et provoque des problèmes de performances évidents. Par exemple, ajoutez le code suivant à un projet de rôle de travail :

```csharp
public class Concatenator
{
    public static string Concatenate(int number)
    {
        int count;
        string s = "";
        for (count = 0; count < number; count++)
        {
            s += "\n" + count.ToString();
        }
        return s;
    }
}
```

Appelez ce code depuis la méthode RunAsync dans la classe RoleEntryPoint-derived du rôle de travail. (Ne tenez pas compte de l'avertissement indiquant l'exécution synchrone de la méthode.)

```csharp
private async Task RunAsync(CancellationToken cancellationToken)
{
    // TODO: Replace the following with your own logic.
    while (!cancellationToken.IsCancellationRequested)
    {
        Trace.TraceInformation("Working");
        Concatenator.Concatenate(10000);
    }
}
```

Générez et exécutez le service cloud en local sans débogage (Ctrl+F5), la configuration de solution étant définie sur **Version finale**. Les fichiers et dossiers sont ainsi créés pour l'exécution de l'application en local et tous les émulateurs sont démarrés. Démarrez l'interface de l'émulateur de calcul à partir de la barre des tâches pour vérifier que votre rôle de travail fonctionne correctement.

## <a name="2-attach-to-a-process"></a>2 : Attacher à un processus
Au lieu de profiler l'application en la démarrant depuis l'interface de développement de Visual Studio 2010, vous devez attacher le profileur à un processus en cours d'exécution. 

Pour ce faire, dans le menu **Analyse**, sélectionnez **Profileur**, puis **Attacher/Détacher**.

![Option Joindre un profil][6]

Pour un rôle de travail, recherchez le processus WaWorkerHost.exe.

![Processus WaWorkerHost][7]

Si votre dossier de projet se trouve sur un disque réseau, le profileur vous demande d'indiquer un autre emplacement pour enregistrer les rapports de profilage.

 Vous pouvez également l'attacher à un rôle web en l'attachant à WaIISHost.exe.
Si votre application comporte plusieurs processus de rôle de travail, utilisez l'ID de processus pour les distinguer les uns des autres. Vous pouvez effectuer une requête par programme sur l'ID de processus en accédant à l'objet Process. Par exemple, si vous ajoutez ce code à la méthode Run de la classe RoleEntryPoint-derived dans un rôle, vous pouvez consulter le journal dans l'interface de l'émulateur de calcul pour savoir à quel processus se connecter.

```csharp
var process = System.Diagnostics.Process.GetCurrentProcess();
var message = String.Format("Process ID: {0}", process.Id);
Trace.WriteLine(message, "Information");
```

Pour consulter le journal, ouvrez l'interface utilisateur de l'émulateur de calcul.

![Démarrage de l’interface de l’émulateur de calcul][8]

Ouvrez la fenêtre de console du journal du rôle de travail dans l'interface utilisateur de l'émulateur de calcul en cliquant sur la barre de titre de la fenêtre de la console. L'ID du processus figure dans le journal.

![Afficher l’ID du processus][9]

Une fois le profileur attaché, suivez les étapes de l'interface utilisateur de votre application (le cas échéant) afin de reproduire le scénario.

Lorsque vous souhaitez arrêter le profilage, cliquez sur le lien **Terminer le profilage** .

![Option Terminer le profilage][10]

## <a name="3-view-performance-reports"></a>3 : Afficher les rapports de performances
Le rapport de performances de votre application s'affiche.

À ce stade, le profileur s'arrête, il enregistre les données dans un fichier .vsp et il affiche un rapport présentant une analyse des données.

![Rapport du profileur][11]

Si vous voyez String.wstrcpy dans le Chemin réactif, cliquez sur Uniquement mon code pour modifier l'affichage et ne voir que le code utilisateur.  Si vous voyez String.Concat, essayez de cliquer sur le bouton Afficher tout le code.

Vous devez voir que la méthode Concatenate et String.Concat occupent une grande partie du temps d'exécution.

![Analyse du rapport][12]

Si vous avez ajouté le code de concaténation de chaîne dans cet article, vous devez obtenir un avertissement correspondant dans la Liste des tâches. Il est possible que vous voyiez également un avertissement lié au nettoyage de la mémoire, ce qui est dû au nombre de chaînes créées et rejetées.

![Avertissements liés aux performances][14]

## <a name="4-make-changes-and-compare-performance"></a>4 : Appliquer les modifications et comparer les performances
Vous pouvez également comparer les performances avant et après la modification du code.  Interrompez le processus en cours d'exécution et modifiez le code de façon à remplacer l'opération de concaténation de chaîne à l'aide de StringBuilder :

```csharp
public static string Concatenate(int number)
{
    int count;
    System.Text.StringBuilder builder = new System.Text.StringBuilder("");
    for (count = 0; count < number; count++)
    {
        builder.Append("\n" + count.ToString());
    }
    return builder.ToString();
}
```

Lancez un nouveau test de performances, et comparez les résultats. Dans l'Explorateur de performances, si les tests font partie de la même session, vous pouvez simplement sélectionner les deux rapports, ouvrir le menu contextuel et sélectionner **Comparer les rapports de performances**. Si vous souhaitez comparer un test avec un test d’une autre session de performances, ouvrez le menu **Analyse**, puis sélectionnez **Comparer les rapports de performances**. Spécifiez les deux fichiers dans la boîte de dialogue qui s'affiche.

![Option Comparer les rapports de performances][15]

Les rapports indiquent les différences entre les deux tests.

![Rapport de comparaison][16]

Félicitations ! Vous avez fait connaissance avec le profileur.

## <a name="troubleshooting"></a>Dépannage
* Assurez-vous que le profilage porte bien sur une build Release et exécutez sans débogage.
* Si l'option Attacher/Détacher n'est pas activée dans le menu Profileur, exécutez l'Assistant Performance.
* Utilisez l'émulateur de calcul pour connaître l'état de votre application. 
* Si vous rencontrez des problèmes pour démarrer les applications dans l'émulateur de calcul ou pour attacher le profileur, arrêtez l'émulateur de calcul et redémarrez-le. Si cela ne résout pas le problème, essayez de redémarrer votre ordinateur. Ce problème peut survenir lorsque vous utilisez l'émulateur de calcul pour suspendre et supprimer des déploiements en cours d'exécution.
* Si vous avez utilisé une des commandes de profilage depuis la ligne de commande, en particulier les paramètres globaux, assurez-vous que VSPerfClrEnv /globaloff a été appelé et que VsPerfMon.exe est arrêté.
* Si lors de l’échantillonnage, le message « PRF0025 : aucune donnée n’a été collectée » s’affiche, vérifiez si le processus que vous avez attaché présente une activité dans le processeur. Les applications qui n'effectuent pas de calculs peuvent ne pas produire de données d'échantillonnage.  Il est également possible que le processus se soit terminé avant l'exécution de l'échantillonnage. Vérifiez si la méthode Run du rôle pour lequel vous établissez le profil ne s'est pas terminée.

## <a name="next-steps"></a>Étapes suivantes
L'instrumentalisation d'exécutables Azure dans l'émulateur de calcul n'est pas prise en charge par le profileur Visual Studio, mais si vous souhaitez tester l'allocation de la mémoire, vous pouvez choisir cette option au moment du profilage. Vous pouvez également choisir le profilage d'accès concurrentiel, qui vous aide à savoir si des threads perdent du temps à se disputer les verrouillages, ou bien le profilage d'interaction de couche, qui permet de détecter les problèmes de performances lors de l'interaction entre différentes couches de l'application, la plupart du temps entre la couche de données et un rôle de travail.  Vous pouvez consulter les requêtes de base de données que votre application génère et utiliser les données de profilage pour optimiser l'utilisation de la base de données. Pour plus d’informations sur le profilage d’interaction de couche, consultez le billet de blog [Walkthrough: Using the Tier Interaction Profiler in Visual Studio Team System 2010][3].

[1]: ../azure-monitor/app/profiler.md
[2]: /previous-versions/azure/hh411542(v=azure.100)
[3]: /archive/blogs/habibh/walkthrough-using-the-tier-interaction-profiler-in-visual-studio-team-system-2010
[4]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally09.png
[5]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally10.png
[6]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally02.png
[7]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally05.png
[8]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally010.png
[9]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally07.png
[10]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally06.png
[11]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally03.png
[12]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally011.png
[14]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally04.png 
[15]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally013.png
[16]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally012.png
[17]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally08.png
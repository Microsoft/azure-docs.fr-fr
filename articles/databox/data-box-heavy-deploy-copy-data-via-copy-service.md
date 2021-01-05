---
title: 'Tutoriel : Copier des données sur Azure Data Box Heavy via le service de copie des données'
description: Dans ce tutoriel, vous allez apprendre à copier des données sur votre appareil Azure Data Box Heavy par le biais du service de copie des données.
services: databox
author: alkohli
ms.service: databox
ms.subservice: heavy
ms.topic: tutorial
ms.date: 07/03/2019
ms.author: alkohli
ms.openlocfilehash: 601e5cf15c47b16c53ff9ca81a56cb613bcfc3f5
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96006557"
---
# <a name="tutorial-use-the-data-copy-service-to-copy-data-into-azure-data-box-heavy-preview"></a>Tutoriel : Utiliser le service de copie des données pour copier des données dans Azure Data Box Heavy (préversion)

Ce tutoriel explique comment ingérer des données par l’intermédiaire du service de copie des données, sans l’aide d’un hôte intermédiaire. Ce service s’exécute localement sur Azure Data Box Heavy, se connecte à votre périphérique NAS par le biais de SMB, puis copie les données sur Data Box Heavy.

Utilisez le service de copie des données :

- Dans les environnements NAS où les hôtes intermédiaires ne sont pas disponibles.
- Avec de petits fichiers dont l’ingestion et le chargement de données prend des semaines. Le service de copie des données améliore considérablement le temps d’ingestion et de chargement des petits fichiers.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Copier des données sur Data Box Heavy

## <a name="prerequisites"></a>Prérequis

Avant de commencer, assurez-vous que :

1. Vous avez terminé le tutoriel [Configurer Azure Data Box Heavy](data-box-heavy-deploy-set-up.md).
2. Vous avez reçu votre Data Box Heavy et l’état de la commande dans le portail est **Livré**.
3. Vous disposez des informations d’identification de l’appareil NAS source auquel vous vous connectez pour la copie des données.
4. Vous êtes connecté à un réseau haut débit. Pour obtenir des vitesses de copie plus rapides, il est possible d’utiliser deux connexions 40 GbE (une par nœud) en parallèle. Si aucune connexion 40-GbE n’est disponible, nous vous recommandons d’avoir au moins deux connexions 10 GbE (une par nœud). 

## <a name="copy-data-to-data-box-heavy"></a>Copier des données sur Data Box Heavy

Une fois que vous êtes connecté à l’appareil NAS, l’étape suivante consiste à copier vos données. Avant de commencer la copie des données, passez en revue les considérations suivantes :

- Quand vous copiez des données, vérifiez que leur taille est conforme aux limites de taille spécifiées dans l’article [Limitations relatives au Stockage Azure et à Data Box Heavy](data-box-heavy-limits.md).
- Si les données chargées par Data Box Heavy le sont aussi simultanément par d’autres applications en dehors de Data Box Heavy, cela peut entraîner l’échec des tâches de chargement et l’altération des données.
- Si les données sont modifiées au moment où elles sont lues par le service de copie des données, vous pouvez constater des échecs ou une altération des données.

Pour copier des données par l’intermédiaire du service de copie des données, vous devez créer une tâche :

1. Dans l’interface utilisateur web locale de votre appareil Data Box Heavy, accédez à **Gérer** > **Copier des données**.
2. Dans la page **Copier des données**, sélectionnez **Créer**.

    ![Sélectionner Créer dans la page Copier des données](media/data-box-deploy-copy-data-via-copy-service/click-create.png)

3. Dans la boîte de dialogue **Configurer le travail et démarrer**, renseignez les champs suivants :
    
    |Champ                          |Valeur    |
    |-------------------------------|---------|
    |**Nom du travail**                       |Nom unique contenant moins de 230 caractères pour le travail. Ces caractères ne sont pas autorisés dans le nom d’un travail : \<, \>, \|, \?, \*, \\, \:, \/ et \\\.         |
    |**Emplacement source**                |Indiquez le chemin SMB vers la source de données au format : `\\<ServerIPAddress>\<ShareName>` ou `\\<ServerName>\<ShareName>`.        |
    |**Nom d’utilisateur**                       |Nom d’utilisateur au format `\\<DomainName><UserName>` pour accéder à la source de données. Si un administrateur local se connecte, il aura besoin d’autorisations de sécurité explicites. Cliquez avec le bouton droit, sélectionnez **Propriétés** , puis **Sécurité**. Cela devrait ajouter l’administrateur local dans l’onglet **Sécurité**.       |
    |**Mot de passe**                       |Mot de passe pour accéder à la source de données.           |
    |**Compte de stockage de destination**    |Sélectionnez le compte de stockage cible sur lequel charger les données à partir de la liste.         |
    |**Type de destination**       |Sélectionnez le type du stockage cible dans la liste : **Objet blob de blocs**, **Objet blob de pages** ou **Azure Files**.        |
    |**Conteneur/partage de destination**    |Entrez le nom du conteneur ou du partage de votre compte de stockage de destination dans lequel vous souhaitez charger les données. Le nom peut être un nom de partage ou un nom de conteneur. Par exemple, utilisez `myshare` ou `mycontainer`. Vous pouvez également entrer le nom au format `sharename\directory_name` ou `containername\virtual_directory_name`.        |
    |**Copier les fichiers correspondant à un modèle**    | Vous pouvez entrer le modèle de correspondance des noms de fichiers de deux façons :<ul><li>**Utiliser des caractères génériques :** seuls `*` et `?` sont pris en charge dans les expressions génériques. Par exemple, l’expression `*.vhd` correspond à tous les fichiers dotés de l’extension `.vhd`. De même, `*.dl?` correspond à tous les fichiers ayant l’extension `.dl` ou qui commencent par `.dl`, tels que `.dll`. De la même façon, `*foo` correspond à tous les fichiers dont le nom se termine par `foo`.<br>Vous pouvez entrer l’expression générique directement dans le champ. Par défaut, la valeur que vous entrez dans le champ est traitée comme une expression générique.</li><li>**Utiliser des expressions régulières :** les expressions régulières POSIX sont prises en charge. Par exemple, l’expression régulière `.*\.vhd` correspond à tous les fichiers dotés de l’extension `.vhd`. Pour les expressions régulières, fournissez le modèle (`<pattern>`) directement au format `regex(<pattern>)`. Pour plus d’informations sur les expressions régulières, consultez [Langage des expressions régulières - Aide-mémoire](/dotnet/standard/base-types/regular-expression-language-quick-reference).</li><ul>|
    |**Optimisation de fichier**              |Lorsque cette fonctionnalité est activée, les fichiers dont la taille est inférieure à 1 Mo sont compressés au cours de l’ingestion. Cette compression accélère la transmission des petits fichiers. Elle permet également un gain de temps significatif lorsque le nombre de fichiers dépasse largement le nombre de répertoires.        |
 
4. Sélectionnez **Démarrer**. Les entrées sont validées, et si la validation aboutit, le travail démarre. Le démarrage du travail peut prendre quelques minutes.

    ![Démarrage d’un travail à partir de la boîte de dialogue Configurer le travail et démarrer](media/data-box-deploy-copy-data-via-copy-service/configure-and-start.png)

5. Une tâche est créée avec les paramètres spécifiés. Vous pouvez suspendre, reprendre, annuler ou redémarrer un travail. Cochez la case en regard du nom du travail, puis sélectionnez le bouton approprié.

    ![Gérer un travail dans la page « Copier des données »](media/data-box-deploy-copy-data-via-copy-service/select-job.png)
    
    - Vous pouvez suspendre un travail si celui-ci affecte les ressources de l’appareil NAS pendant les heures de pointe :

        ![Suspendre un travail dans la page « Copier des données »](media/data-box-deploy-copy-data-via-copy-service/pause-job.png)

        Vous pouvez reprendre le travail plus tard, en dehors des heures de pointe :

        ![Reprendre un travail dans la page « Copier des données »](media/data-box-deploy-copy-data-via-copy-service/resume-job.png)

    - Vous pouvez annuler un travail à tout moment :

        ![Annuler un travail dans la page « Copier des données »](media/data-box-deploy-copy-data-via-copy-service/cancel-job.png)
        
        Lorsque vous annulez une tâche, une confirmation est demandée :

        ![Confirmation de l’annulation d’une tâche](media/data-box-deploy-copy-data-via-copy-service/confirm-cancel-job.png)

        Si vous décidez d’annuler un travail, les données déjà copiées ne sont pas supprimées. Pour supprimer toutes les données que vous avez copiées sur votre appareil Data Box, réinitialisez l’appareil.

        ![Réinitialiser un appareil](media/data-box-deploy-copy-data-via-copy-service/reset-device.png)

        >[!NOTE]
        > Lorsque vous annulez ou suspendez un travail, les fichiers volumineux peuvent n’être copiés que partiellement. Ces fichiers seront chargés dans Azure dans le même état. Lorsque vous annulez ou suspendez un travail, vérifiez que vos fichiers ont été correctement copiés. Pour valider les fichiers, examinez les partages SMB ou téléchargez le fichier de marque d'ordre d'octet.

    - Lorsqu’un travail échoue en raison d’une erreur temporaire, telle qu’un problème réseau, vous pouvez redémarrer ce travail. Toutefois, vous ne pouvez pas redémarrer un travail si celui-ci a atteint un état terminal, tel que **Opération réussie** ou **Opération terminée avec des erreurs**. Les échecs de travaux peuvent être dus à des problèmes liés au nom ou à la taille des fichiers. Ces erreurs sont journalisées, toutefois, le travail ne peut pas être redémarré une fois terminé.

        ![Redémarrage d’une tâche ayant échoué](media/data-box-deploy-copy-data-via-copy-service/restart-failed-job.png)

        Si vous ne parvenez pas à redémarrer le travail après un échec, téléchargez les journaux d’activité d’erreurs pour comprendre les raisons de cet échec. Une fois le problème corrigé, créez un travail pour copier les fichiers. Vous pouvez également [copier les fichiers sur SMB](data-box-deploy-copy-data.md).
    
    - Dans cette version, vous ne pouvez pas supprimer de tâche.
    
    - Vous pouvez créer un nombre illimité de travaux, toutefois, vous ne pouvez en exécuter que 10 simultanément.
    - Si l’option **Optimisation de fichier** est activée, les fichiers de petite taille sont compressés à l’ingestion pour améliorer les performances de copie. Dans ce cas, vous verrez un fichier compressé dont le nom correspond à un GUID. Ne supprimez pas ce fichier. Il sera décompressé lors du chargement.

6. Pendant que la tâche est en cours d’exécution, dans la page **Copier des données** :

    - Dans la colonne **État**, vous pouvez voir l’état du travail de copie. L'état peut prendre l'une des valeurs suivantes :
        - **Exécution**
        - **Échec**
        - **Réussi**
        - **Suspension**
        - **En pause**
        - **Annulation**
        - **Canceled**
        - **Terminé avec des erreurs**
    - Dans la colonne **Fichiers**, vous pouvez voir le nombre ainsi que la taille totale des fichiers en cours de copie.
    - Dans la colonne **Traités**, vous pouvez voir le nombre ainsi que la taille totale des fichiers qui sont traités.
    - Dans la colonne **Détails du travail**, sélectionnez **Afficher** pour voir les informations concernant le travail.
    - Si des erreurs se produisent pendant le processus de copie, comme indiqué dans la colonne **Nombre d’erreurs**, accédez à la colonne **Journal des erreurs**, puis téléchargez les journaux d’activité d’erreurs pour résoudre les problèmes.

Attendez la fin du travail de copie. Comme certaines erreurs ne sont journalisées que dans la page **Connexion et copie**, vérifiez que le travail de copie s’est terminé sans erreur avant de passer à l’étape suivante.

![Aucune erreur dans la page « Connexion et copie »](media/data-box-deploy-copy-data-via-copy-service/verify-no-errors-on-connect-and-copy.png)

Pour garantir l’intégrité des données, la somme de contrôle est calculée par le biais d’une fonction inline lors de la copie des données. Une fois la copie terminée, sélectionnez **Afficher le tableau de bord** pour vérifier l’espace utilisé et l’espace libre sur votre appareil.
    
![Vérifier l’espace libre et l’espace utilisé sur le tableau de bord](media/data-box-deploy-copy-data-via-copy-service/verify-used-space-dashboard.png)

> [!IMPORTANT]
> Répétez les mêmes instructions pour copier des données sur le deuxième nœud de Data Box Heavy.

Une fois le travail de copie terminé, vous pouvez sélectionner **Préparer l’expédition**.

>[!NOTE]
> La **préparation de l’expédition** ne peut pas s’effectuer si les travaux de copie sont en cours.

## <a name="next-steps"></a>Étapes suivantes

Passez au tutoriel suivant pour découvrir comment renvoyer votre appareil Data Box Heavy à Microsoft.

> [!div class="nextstepaction"]
> [Envoyer votre appareil Azure Data Box Heavy à Microsoft](./data-box-heavy-deploy-picked-up.md)
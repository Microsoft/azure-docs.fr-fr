---
title: Créer un runbook graphique dans Azure Automation
description: Cet article vous apprend à créer, tester et publier un runbook graphique simple dans Azure Automation.
keywords: runbook, modèle de runbook, automatisation des runbooks, runbook azure
services: automation
ms.subservice: process-automation
ms.date: 09/15/2020
ms.topic: tutorial
ms.openlocfilehash: 1b6c02778b0ee790d81c713283e653058c29c153
ms.sourcegitcommit: 693df7d78dfd5393a28bf1508e3e7487e2132293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92899793"
---
# <a name="tutorial-create-a-graphical-runbook"></a>Tutoriel : Créer un runbook graphique

Ce didacticiel vous familiarise avec la création d’un [Runbook graphique](../automation-runbook-types.md#graphical-runbooks) dans Azure Automation. Vous pouvez créer et modifier des runbooks graphiques de workflow PowerShell avec l’éditeur graphique du portail Azure. 

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer un runbook graphique simple
> * Tester et publier le runbook
> * Exécuter le travail du runbook et en suivre l’état
> * Mettre à jour le runbook pour démarrer une machine virtuelle Azure avec des paramètres de runbook et des liens conditionnels

## <a name="prerequisites"></a>Prérequis

Pour réaliser ce didacticiel, vous avez besoin des éléments suivants :

* Abonnement Azure. Si vous n’avez pas encore d’abonnement, vous pouvez [activer vos avantages abonnés MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) ou créer [un compte gratuit](https://azure.microsoft.com/free).
* [compte Automation](../index.yml) pour le stockage du Runbook et l’authentification auprès des ressources Azure. Ce compte doit avoir l’autorisation de démarrer et d’arrêter la machine virtuelle.
* Une machine virtuelle Azure. Comme vous allez devoir l’arrêter et la démarrer, il ne doit pas s’agir d’une machine virtuelle de production.
* Si nécessaire, [importez des modules Azure](../shared-resources/modules.md) ou [mettez à jour les modules](../automation-update-azure-modules.md) en fonction des applets de commande que vous utilisez.

## <a name="step-1---create-runbook"></a>Étape 1 - Création d’un Runbook

Commencez par créer un runbook simple qui renvoie le texte `Hello World`.

1. Dans le portail Azure, ouvrez votre compte Automation.

    La page du compte Automation vous offre un aperçu rapide des ressources de ce compte. Vous devriez déjà posséder certains éléments. La plupart de ces ressources sont les modules automatiquement inclus dans un nouveau compte Automation. Vous devez également disposer de la ressource Informations d’identification associée à votre abonnement.

2. Sélectionnez **Runbooks** sous **Automatisation de processus** pour ouvrir la liste des runbooks.

3. Créez un runbook en sélectionnant **Créer un runbook** .

4. Nommez le Runbook **MyFirstRunbook-Graphical** .

5. Dans ce cas, vous allez créer un [graphique runbook](../automation-graphical-authoring-intro.md). Pour **Type de Runbook** , sélectionnez **Graphique** .

    ![Nouveau Runbook](../media/automation-tutorial-runbook-graphical/create-new-runbook.png)

6. Cliquez sur **Créer** pour créer le Runbook et ouvrez l’éditeur graphique.

## <a name="step-2---add-activities"></a>Étape 2 - Ajouter des activités

Le contrôle Bibliothèque à gauche de l’éditeur vous permet de sélectionner des activités à ajouter à votre Runbook. Nous allons ajouter une cmdlet `Write-Output` pour extraire du texte du runbook.

1. Dans le contrôle Bibliothèque, cliquez dans la zone de recherche, puis saisissez `write-output`. Les résultats de la recherche sont présentés dans l’image suivante.

    ![Microsoft.PowerShell.Utility](../media/automation-tutorial-runbook-graphical/search-powershell-cmdlet-writeoutput.png)

2. Faites défiler jusqu'au bas de la liste. Cliquez avec le bouton droit sur **Write-Output** , puis sélectionnez **Ajouter au canevas** . Vous pouvez également cliquer sur les points de suspension (...) en regard du nom de la cmdlet, puis sélectionner **Ajouter au canevas** .

3. Cliquez sur l’activité **Write-Output** sur le canevas. La page du contrôle Configuration qui vous permet de configurer l’activité s’ouvre.

4. Par défaut, le champ **Étiquette** affiche le nom de la cmdlet, mais vous pouvez le modifier pour le rendre plus convivial. Remplacez-le par `Write Hello World to output`.

5. Cliquez sur **Paramètres** pour renseigner les paramètres de l’applet de commande.

   Certaines cmdlets comportent plusieurs jeux de paramètres, et vous devez choisir lequel utiliser. Dans ce cas, `Write-Output` ne possède qu'un seul jeu de paramètres.

6. Sélectionnez le paramètre `InputObject`. Il s’agit du paramètre utilisé pour spécifier le texte à envoyer au flux de sortie.

7. Le menu déroulant **Source de données** affiche des sources que vous utilisez pour renseigner un paramètre. Dans ce menu, sélectionnez **Expression PowerShell** .

   Vous pouvez utiliser une sortie de ces sources, par exemple, une autre activité, une ressource Automation ou une expression PowerShell. Dans ce cas, la sortie est simplement `Hello World`. Vous pouvez utiliser une expression PowerShell et spécifier une chaîne.

8. Dans le champ **Expression** , saisissez `"Hello World"`, puis cliquez deux fois sur **OK** pour revenir au canevas.

9. Enregistrez le Runbook en cliquant sur **Enregistrer** .

## <a name="step-3---test-the-runbook"></a>Étape 3 : Test du Runbook

Avant de publier le runbook pour le rendre disponible en production, vous devez le tester pour vous assurer qu’il fonctionne correctement. Le test d’un runbook exécute sa version Brouillon et vous permet d’afficher sa sortie de manière interactive.

1. Sélectionnez l’option **Volet de test** .

2. Cliquez sur **Démarrer** pour démarrer le test. Ce doit être la seule option activée.

3. Un [travail de runbook](../automation-runbook-execution.md) est créé, dont l’état apparaît dans le volet.

   L’état du travail commence par `Queued`, ce qui indique qu’il attend qu’un Runbook Worker soit disponible dans le cloud. L’état passe ensuite à `Starting` lorsqu’un Worker revendique le travail. Enfin, l’état passe à `Running` lorsque le runbook commence à s’exécuter.

4. Une fois le travail du runbook terminé, le volet Test affiche sa sortie. Dans ce cas, vous voyez `Hello World`.

    ![Sortie de runbook Hello World](../media/automation-tutorial-runbook-graphical/runbook-test-results.png)

5. Fermez le volet de test pour revenir au canevas.

## <a name="step-4---publish-and-start-the-runbook"></a>Étape 4 : Publication et démarrage du Runbook

Le runbook que vous avez créé est toujours en mode Brouillon. Il doit être publié avant de pouvoir l’exécuter en production. Lorsque vous publiez un Runbook, vous écrasez la version publiée existante par la version brouillon. Dans ce cas, vous n’avez pas encore de version publiée car vous venez de créer le runbook.

1. Sélectionnez l’option **Publier** pour publier le runbook, puis cliquez sur **Oui** quand vous y êtes invité.

2. Faites défiler la page vers la gauche pour afficher le runbook sur la page Runbooks, et notez que la valeur **État de création** indique **Publié** .

3. Faites défiler la page vers la droite pour visualiser la page **MyFirstRunbook-Graphical** .

   Les options en haut vous permettent de démarrer le runbook, de planifier une heure de démarrage ou de créer un [webhook](../automation-webhooks.md) pour démarrer le runbook via un appel HTTP.

4. Cliquez sur **Démarrer** , puis sur **Oui** quand vous y êtes invité pour démarrer le runbook.

5. Un volet Travail s’ouvre pour le travail du runbook que vous avez créé. Vérifiez que le champ **État du travail** indique **Terminé** .

6. Cliquez sur **Sortie** pour ouvrir la page Sortie qui affiche `Hello World`.

7. Fermez la page Sortie.

8. Cliquez sur **Tous les journaux d’activité** pour ouvrir le volet Flux de la tâche du Runbook. Vous ne devriez voir que `Hello World` dans le flux de sortie.

    Notez que le volet Flux peut afficher d’autres flux pour un travail de runbook, tels que les flux de détails et d’erreurs, si le runbook écrit dans ceux-ci.

9. Fermez les volets Flux et Travail pour revenir à la page MyFirstRunbook-Graphical.

10. Pour afficher toutes les tâches du runbook, sélectionnez **Travaux** sous **Ressources** . La page Travaux répertorie toutes les tâches créées par votre runbook. Vous ne devriez voir qu’une seule tâche, car vous n’avez exécuté le travail qu’une seule fois.

11. Cliquez sur le nom du travail pour ouvrir le volet Travail que vous avez vu au démarrage du runbook. Utilisez ce volet pour voir les détails de tout travail créé pour le runbook.

## <a name="step-5---create-variable-assets"></a>Étape 5 - Créer des ressources de variables

Vous avez testé et publié votre runbook mais, jusqu’à présent, il ne fait rien d’utile pour gérer des ressources Azure. Avant de configurer le runbook pour l’authentification, vous devez créer une variable destinée à contenir l’ID d’abonnement, configurer une activité à authentifier, puis référencer la variable. L’ajout d’une référence au contexte de l’abonnement vous permet d’utiliser facilement plusieurs abonnements.

1. Copiez votre ID d’abonnement à partir de l’option **Abonnements** du panneau de navigation.

2. Dans la page Comptes Automation, sélectionnez **Variables** sous **Ressources partagées** .

3. Sélectionnez **Ajouter une variable** .

4. Dans les champs de la page Nouvelle variable, définissez les paramètres suivants.

    * **Nom**  : entrez `AzureSubscriptionId`.
    * **Valeur**  : entrez votre ID d’abonnement.
    * **Type**  : conservez la chaîne sélectionnée.
    * **Chiffrement**  : utilisez la valeur par défaut.

5. Cliquez sur **Créer** pour créer la variable.

## <a name="step-6---add-authentication"></a>Étape 6 - Ajouter une authentification

Maintenant que vous disposez d’une variable pouvant contenir l’ID d’abonnement, vous pouvez configurer le runbook pour l’authentification à l’aide des informations d’identification Exécuter en tant que de votre abonnement. Pour ce faire, ajoutez la connexion Azure Exécuter en tant que comme ressource. Vous devez également ajouter les cmdlets [Connect-AzAccount](/powershell/module/az.accounts/Connect-AzAccount) et la cmdlet [Set-AzContext](/powershell/module/az.accounts/Set-AzContext) au canevas.

>[!NOTE]
>Pour les runbooks PowerShell, `Add-AzAccount` et `Add-AzureRMAccount` sont des alias de `Connect-AzAccount`. Notez que ces alias ne sont pas disponibles pour vos runbooks graphiques. Un runbook graphique peut uniquement utiliser `Connect-AzAccount`.

1. Accédez à votre runbook, puis sélectionnez **Modifier** dans la page MyFirstRunbook-Graphical.

2. Vous n'avez plus besoin de l'entrée `Write Hello World to output`. Il vous suffit de cliquer sur les points de suspension, puis de sélectionner **Supprimer** .

3. Dans le contrôle Bibliothèque, développez **RESSOURCES** , puis **Connexions** . Ajoutez `AzureRunAsConnection` au canevas en sélectionnant **Ajouter au canevas** .

4. Renommez `AzureRunAsConnection` en `Get Run As Connection`.

5. Dans la zone de recherche du contrôle Bibliothèque, saisissez `Connect-AzAccount`.

6. Ajoutez `Connect-AzAccount` au canevas.

7. Pointez sur `Get Run As Connection` jusqu'à ce qu'un cercle apparaisse au bas de la forme. Cliquez sur le cercle et faites glisser la flèche vers `Connect-AzAccount` pour former un lien. Le runbook commence par `Get Run As Connection`, puis exécute `Connect-AzAccount`.

    ![Créer un lien entre des activités](../media/automation-tutorial-runbook-graphical/runbook-link-auth-activities.png)

8. Sur le canevas, sélectionnez `Connect-AzAccount`. Dans le volet Contrôle de configuration, saisissez **Login to Azure** dans le champ **Étiquette** .

9. Cliquez sur **Paramètres** pour afficher la page Configuration des paramètres d’activité.

10. La cmdlet `Connect-AzAccount` possède plusieurs jeux de paramètres. Vous devez en sélectionner un avant de fournir des valeurs de paramètre. Cliquez sur **Jeu de paramètres** , puis sélectionnez **ServicePrincipalCertificateWithSubscriptionId** .

11. Les paramètres de ce jeu de paramètres apparaissent sur la page Configuration des paramètres d’activité. Cliquez sur **APPLICATIONID** .

    ![Ajouter des paramètres de compte Azure](../media/automation-tutorial-runbook-graphical/Add-AzureRmAccount-params.png)

12. Dans la page Valeur du paramètre, définissez les paramètres suivants, puis cliquez sur **OK** .

   * **Source de données**  : sélectionnez **Sortie d’activité** .
   * Liste des sources de données : sélectionnez **Obtenir une connexion Automation** .
   * **Chemin du champ**  : saisissez `ApplicationId`. Vous spécifiez le nom de la propriété du chemin du champ, car l’activité génère un objet avec plusieurs propriétés.

13. Cliquez sur **CERTIFICATETHUMBPRINT** , puis, dans la page Valeur du paramètre, définissez les paramètres suivants et cliquez sur **OK** .

    * **Source de données**  : sélectionnez **Sortie d’activité** .
    * Liste des sources de données : sélectionnez **Obtenir une connexion Automation** .
    * **Chemin du champ**  : saisissez `CertificateThumbprint`.

14. Cliquez sur **SERVICEPRINCIPAL** , puis, dans la page Valeur du paramètre, sélectionnez **ConstantValue** pour le champ **Source de données** . Cliquez sur l’option **True** , puis sur **OK** .

15. Cliquez sur **TENANTID** , puis définissez les paramètres suivants sur la page Valeur du paramètre. Lorsque vous avez terminé, cliquez sur **OK** à deux reprises.

    * **Source de données**  : sélectionnez **Sortie d’activité** .
    * Liste des sources de données : sélectionnez **Obtenir une connexion Automation** .
    * **Chemin du champ**  : saisissez `TenantId`.

16. Dans la zone de recherche du contrôle Bibliothèque, saisissez `Set-AzContext`.

17. Ajoutez `Set-AzContext` au canevas.

18. Sur le canevas, sélectionnez `Set-AzContext`. Dans le volet Contrôle de configuration, saisissez `Specify Subscription Id` dans le champ **Étiquette** .

19. Cliquez sur **Paramètres** pour afficher la page Configuration des paramètres d’activité.

20. La cmdlet `Set-AzContext` possède plusieurs jeux de paramètres. Vous devez en sélectionner un avant de fournir des valeurs de paramètre. Cliquez sur **Jeu de paramètres** , puis sélectionnez **SubscriptionId** .

21. Les paramètres de ce jeu de paramètres apparaissent sur la page Configuration des paramètres d’activité. Cliquez sur **SubscriptionID** .

22. Dans la page Valeur du paramètre, sélectionnez **Ressource de variable** comme **Source de données** et sélectionnez **AzureSubscriptionId** dans la liste de sources. Lorsque vous avez terminé, cliquez sur **OK** à deux reprises.

23. Pointez sur `Login to Azure` jusqu'à ce qu'un cercle apparaisse au bas de la forme. Cliquez sur le cercle et faites glisser la flèche vers `Specify Subscription Id`. À ce stade, votre runbook doit être semblable au suivant.

    :::image type="content" source="../media/automation-tutorial-runbook-graphical/runbook-auth-config.png" alt-text="Capture d’écran du runbook après avoir fait glisser la flèche vers « Specify Subscription ID » (Spécifier un ID d’abonnement).":::

## <a name="step-7---add-activity-to-start-a-virtual-machine"></a>Étape 7 - Ajouter une activité pour démarrer une machine virtuelle

Vous devez maintenant ajouter une activité `Start-AzVM` pour démarrer une machine virtuelle. Vous pouvez choisir tout machine virtuelle dans votre abonnement Azure. Pour l’instant, vous allez coder son nom en dur dans la cmdlet [Start-AzVM](/powershell/module/az.compute/start-azvm).

1. Dans la zone de recherche du contrôle Bibliothèque, saisissez `Start-Az`.

2. Ajoutez la cmdlet `Start-AzVM` au canevas, puis cliquez dessus et faites-la glisser sous `Specify Subscription Id`.

3. Pointez sur `Specify Subscription Id` jusqu'à ce qu'un cercle apparaisse au bas de la forme. Cliquez sur le cercle et faites glisser la flèche vers `Start-AzVM`.

4. Sélectionnez `Start-AzVM`. Cliquez sur **Paramètres** , puis sur **Jeu de paramètres** afin d’afficher les jeux de paramètres disponibles pour l’activité.

5. Sélectionnez le jeu de paramètres **ResourceGroupNameParameterSetName** . Les champs **ResourceGroupName** et **Name** contiennent des points d’exclamation pour indiquer qu’il s’agit de paramètres obligatoires. Notez que les deux champs attendent des valeurs de chaîne.

6. Sélectionnez **Name** . Pour le champ **Source de données** , choisissez **Expression PowerShell** . Pour la machine virtuelle que vous utilisez pour démarrer ce runbook, tapez le nom de la machine entouré de guillemets doubles. Cliquez sur **OK** .

7. Sélectionnez **ResourceGroupName** . Pour le champ **Source de données** , utilisez la valeur **Expression PowerShell** , puis saisissez le nom du groupe de ressources entre guillemets. Cliquez sur **OK** .

8. Cliquez sur le volet de **Test** afin de tester le runbook.

9. Cliquez sur **Démarrer** pour commencer le test. Une fois le test terminé, assurez-vous que la machine virtuelle a démarré. À ce stade, votre runbook doit être semblable au suivant.

    ![Runbook - Sortie de la commande Start-AzVM](../media/automation-tutorial-runbook-graphical/runbook-startvm.png)

## <a name="step-8---add-additional-input-parameters"></a>Étape 8 - Ajouter des paramètres d’entrée supplémentaires

Votre runbook démarre actuellement la machine virtuelle dans le groupe de ressources que vous avez spécifié pour la cmdlet `Start-AzVM`. Le runbook est plus utile si vous spécifiez le nom et le groupe de ressources lors du démarrage du runbook. Ajoutons des paramètres d’entrée au runbook pour fournir cette fonctionnalité.

1. Ouvrez l’éditeur graphique en cliquant sur **Modifier** dans la page MyFirstRunbook-Graphical.

2. Cliquez sur **Entrée et sortie** , puis sur **Ajouter une entrée** pour ouvrir le volet Paramètre d’entrée de Runbook.

3. Définissez les paramètres suivants dans les champs fournis, puis cliquez sur **OK** .
   * **Nom**  : spécifiez `VMName`.
   * **Type**  : conservez le paramètre de chaîne.
   * **Obligatoire**  : remplacez la valeur par **Oui** .

4. Créez un second paramètre d'entrée obligatoire appelé `ResourceGroupName`, puis cliquez sur **OK** pour fermer le panneau Entrée et sortie.

    ![Paramètres d'entrée de Runbook](../media/automation-tutorial-runbook-graphical/start-azurermvm-params-outputs.png)

5. Sélectionnez l'activité `Start-AzVM`, puis cliquez sur **Paramètres** .

6. Modifiez le champ **Source de données** pour **Nom** en **Entrée de Runbook** . Sélectionnez ensuite **VMName** .

7. Modifiez le champ **Source de données** de **ResourceGroupName** en **Entrée de Runbook** , puis sélectionnez **ResourceGroupName** .

    ![Paramètres Start-AzVM](../media/automation-tutorial-runbook-graphical/start-azurermvm-params-runbookinput.png)

8. Enregistrez le Runbook et ouvrez le volet de test. Vous pouvez désormais fournir des valeurs pour les deux variables d’entrée que vous utilisez dans le test.

9. Fermez le volet de test.

10. Cliquez sur **Publier** pour publier la nouvelle version du Runbook.

11. Arrêtez la machine virtuelle que vous avez démarrée précédemment.

12. Cliquez sur **Démarrer** pour démarrer le Runbook. Tapez les valeurs de `VMName` et `ResourceGroupName` pour la machine virtuelle que vous allez démarrer.

13. Une fois l’exécution du runbook terminée, assurez-vous que la machine virtuelle a été démarrée.

## <a name="step-9---create-a-conditional-link"></a>Étape 9 - Création d’un lien conditionnel

Vous pouvez maintenant modifier le runbook afin qu’il tente de démarrer la machine virtuelle uniquement si elle n’est pas déjà démarrée. Pour ce faire, ajoutez une cmdlet [Get-AzVM](/powershell/module/Az.Compute/Get-AzVM) qui récupère l’état de niveau d’instance de la machine virtuelle. Vous pouvez ensuite ajouter un module de code Workflow PowerShell appelé `Get Status` avec un extrait de code PowerShell pour déterminer l'état de la machine virtuelle (En cours d'exécution ou Arrêté). Un lien conditionnel à partir du module `Get Status` exécute `Start-AzVM` uniquement si l'état d'exécution en cours est Arrêté. À la fin de cette procédure, votre runbook utilise la cmdlet `Write-Output` pour générer un message vous indiquant si la machine virtuelle a été démarrée avec succès.

1. Ouvrez **MyFirstRunbook-Graphical** dans l’éditeur graphique.

2. Supprimez le lien entre `Specify Subscription Id` et `Start-AzVM` en cliquant dessus, puis en appuyant sur **Supprimer** .

3. Dans la zone de recherche du contrôle Bibliothèque, saisissez `Get-Az`.

4. Ajoutez `Get-AzVM` au canevas.

5. Sélectionnez `Get-AzVM`, puis cliquez sur **Jeu de paramètres** afin d'afficher les jeux pour la cmdlet.

6. Sélectionnez le jeu de paramètres **GetVirtualMachineInResourceGroupNameParamSet** . Les champs **ResourceGroupName** et **Name** sont assortis de points d’exclamation pour indiquer qu’il s’agit de paramètres obligatoires. Notez que les deux champs attendent des valeurs de chaîne.

7. Pour le champ **Name** sous **Source de données** , sélectionnez **Entrée du Runbook** puis **VMName** . Cliquez sur **OK** .

8. Pour le champ **ResourceGroupName** sous **Source de données** , sélectionnez **Entrée de Runbook** , puis **ResourceGroupName** . Cliquez sur **OK** .

9. Pour le paramètre **Status** sous **Source de données** , sélectionnez **Valeur constante** , puis **True** . Cliquez sur **OK** .

10. Créez un lien entre `Specify Subscription Id` et `Get-AzVM`.

11. Dans le contrôle Bibliothèque, développez **Contrôle de Runbook** , puis ajoutez **Code** au canevas.  

12. Créez un lien entre `Get-AzVM` et `Code`.  

13. Cliquez sur `Code`, puis, dans le panneau Configuration, remplacez l'étiquette par **Obtenir l'état** .

14. Sélectionnez `Code` pour afficher la page Éditeur de code.  

15. Collez l’extrait de code suivant dans la page de l’éditeur.

    ```powershell
    $Statuses = $ActivityOutput['Get-AzVM'].Statuses
    $StatusOut = ""
    foreach ($Status in $Statuses) {
      if($Status.Code -eq "Powerstate/running")
        {$StatusOut = "running"}
      elseif ($Status.Code -eq "Powerstate/deallocated")
        {$StatusOut = "stopped"}
    }
    $StatusOut
    ```

16. Créez un lien entre `Get Status` et `Start-AzVM`.

    ![Runbook avec module de code](../media/automation-tutorial-runbook-graphical/runbook-startvm-get-status.png)  

17. Sélectionnez le lien puis, dans le volet Configuration, changez **Appliquer la condition** en **Oui** . Notez que le lien se transforme en ligne pointillée, indiquant que l’activité cible s’exécute uniquement si la condition produit la valeur true.  

18. Pour **Expression de condition** , tapez `$ActivityOutput['Get Status'] -eq "Stopped"`. Désormais, `Start-AzVM` ne s'exécute que si la machine virtuelle est arrêtée.

19. Dans le contrôle Bibliothèque, développez **Applets de commande** , puis **Microsoft.PowerShell.Utility** .

20. Ajoutez deux fois `Write-Output` au canevas.

21. Pour le premier contrôle `Write-Output`, cliquez sur **Paramètres** , puis remplacez la valeur du champ **Étiquette** par **Notify VM Started** .

22. Pour **InputObject** , remplacez **Source de données** par **Expression PowerShell** , puis saisissez l'expression `$VMName successfully started.`.

23. Sur le second contrôle `Write-Output`, cliquez sur **Paramètres** , puis remplacez la valeur du champ **Étiquette** par **Notify VM Start Failed** .

24. Pour **InputObject** , remplacez **Source de données** par **Expression PowerShell** , puis saisissez l'expression `$VMName could not start`.

25. Créez des liens entre `Start-AzVM`, `Notify VM Started` et `Notify VM Start Failed`.

26. Sélectionnez le lien vers `Notify VM Started`, puis remplacez **Appliquer la condition** par True.

27. Pour **Expression de condition** , tapez `$ActivityOutput['Start-AzVM'].IsSuccessStatusCode -eq $true`. Ce contrôle `Write-Output` ne s'exécute désormais que si la machine virtuelle démarre correctement.

28. Sélectionnez le lien vers `Notify VM Start Failed`, puis remplacez **Appliquer la condition** par True.

29. Pour le champ **Expression de condition** , tapez `$ActivityOutput['Start-AzVM'].IsSuccessStatusCode -ne $true`. Ce contrôle `Write-Output` ne s'exécute désormais que si la machine virtuelle ne démarre pas correctement. À ce stade, votre runbook doit être semblable à l'image suivante.

    ![Runbook avec Write-Output](../media/automation-tutorial-runbook-graphical/runbook-startazurermvm-complete.png)

30. Enregistrez le Runbook et ouvrez le volet de test.

31. Démarrez le runbook avec la machine virtuelle arrêtée. La machine doit démarrer.

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur la création graphique, consultez [Créer un runbook graphique dans Azure Automation](../automation-graphical-authoring-intro.md).
* Pour démarrer avec les runbooks PowerShell, consultez [Créer un runbook PowerShell](automation-tutorial-runbook-textual-powershell.md).
* Pour démarrer avec les runbooks de workflow PowerShell, consultez [Créer un runbook de workflow PowerShell](automation-tutorial-runbook-textual.md).
* Pour obtenir des informations de référence sur les cmdlets PowerShell, consultez [Az.Automation](/powershell/module/az.automation).

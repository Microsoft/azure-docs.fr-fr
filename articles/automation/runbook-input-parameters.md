---
title: Configurer les paramètres d’entrée de runbook dans Azure Automation
description: Cet article explique comment configurer des paramètres d’entrée de runbook, qui permettent de transmettre des données à un runbook au démarrage.
services: automation
ms.subservice: process-automation
ms.date: 02/14/2019
ms.topic: conceptual
ms.openlocfilehash: 84e2eaf71326f59102800428479768aeba9ef9ab
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87042149"
---
# <a name="configure-runbook-input-parameters"></a>Configurer les paramètres d’entrée de runbook

Les paramètres d’entrée de runbook augmentent la flexibilité d’un runbook en vous permettant de lui transmettre des données lors de son démarrage. Ces paramètres permettent de cibler les actions des runbooks pour des scénarios et environnements spécifiques. Cet article décrit la configuration et l’utilisation des paramètres d’entrée dans vos runbooks.

Vous pouvez configurer des paramètres d’entrée pour des runbooks PowerShell, PowerShell Workflow, graphiques et Python. Un Runbook peut avoir plusieurs paramètres avec différents types de données, ou aucun paramètre. Les paramètres d’entrée peuvent être obligatoires ou facultatifs, et vous pouvez utiliser des valeurs par défaut pour les paramètres facultatifs.

Vous affectez des valeurs aux paramètres d’entrée d’un runbook quand vous le démarrez. Vous pouvez démarrer un runbook à partir du portail Azure, d’un service web ou de PowerShell. Vous pouvez également démarrer un Runbook en tant que Runbook enfant appelé en ligne dans un autre Runbook.

### <a name="configure-input-parameters-in-powershell-runbooks"></a>Configurer des paramètres d'entrée dans des Runbooks PowerShell

Les runbooks PowerShell et PowerShell Workflow dans Azure Automation prennent en charge les paramètres d’entrée définis par le biais des propriétés suivantes. 

| **Propriété** | **Description** |
|:--- |:--- |
| Type |Obligatoire. Type de données attendu pour la valeur du paramètre. Tout type .NET est valide. |
| Nom |Obligatoire. Le nom du paramètre. Ce nom doit être unique au sein du runbook, doit commencer par une lettre et ne peut contenir que des lettres, des chiffres ou des caractères de soulignement. |
| Obligatoire |facultatif. Valeur booléenne indiquant si le paramètre exige une valeur. Si vous définissez ce paramètre sur True, une valeur doit être fournie au démarrage du runbook. Si vous le définissez sur False, la valeur est facultative. Si vous ne spécifiez pas de valeur pour la propriété `Mandatory`, PowerShell considère que le paramètre d’entrée est facultatif par défaut. |
| Valeur par défaut |facultatif. Valeur utilisée pour le paramètre si aucune valeur d’entrée n’est passée au démarrage du runbook. Le runbook peut définir une valeur par défaut pour n’importe quel paramètre. |

Windows PowerShell prend en charge d’autres attributs de paramètres d’entrée que ceux listés ci-dessus, comme la validation, les alias et les jeux de paramètres. Toutefois, Azure Automation ne prend actuellement en charge que les propriétés de paramètres d’entrée listées ci-dessus.

À titre d’exemple, examinons une définition de paramètre dans un runbook PowerShell Workflow. Cette définition se présente sous la forme générale suivante, où plusieurs paramètres sont séparés par des virgules.

```powershell
Param
(
  [Parameter (Mandatory= $true/$false)]
  [Type] $Name1 = <Default value>,

  [Parameter (Mandatory= $true/$false)]
  [Type] $Name2 = <Default value>
)
```

À présent, configurons les paramètres d’entrée pour un runbook PowerShell Workflow qui génère des détails sur des machines virtuelles, soit une machine virtuelle unique, soit toutes les machines virtuelles au sein d’un groupe de ressources. Ce runbook comporte deux paramètres, comme le montre la capture d’écran suivante : le nom de la machine virtuelle (`VMName`) et le nom du groupe de ressources (`resourceGroupName`).

![PowerShell Workflow d’Automation](media/automation-runbook-input-parameters/automation-01-powershellworkflow.png)

Dans cette définition de paramètre, les paramètres d’entrée sont des paramètres simples de type chaîne.

Notez que les runbooks PowerShell et PowerShell Workflow prennent en charge tous les types, simples et complexes, comme `Object` ou `PSCredential` pour les paramètres d’entrée. Si votre runbook comporte un paramètre d’entrée objet, vous devez utiliser une table de hachage PowerShell contenant des paires nom-valeur pour transmettre une valeur. Par exemple, vous avez le paramètre suivant dans un runbook.

```powershell
[Parameter (Mandatory = $true)]
[object] $FullName
```

Dans ce cas, vous pouvez transmettre la valeur suivante au paramètre.

```powershell
@{"FirstName"="Joe";"MiddleName"="Bob";"LastName"="Smith"}
```

> [!NOTE]
> Quand vous ne transmettez pas de valeur à un paramètre de type String facultatif dont la valeur par défaut est null, la valeur du paramètre est une chaîne vide et non la valeur null.

### <a name="configure-input-parameters-in-graphical-runbooks"></a>Configurer des paramètres d’entrée dans des Runbooks graphiques

Pour illustrer la configuration des paramètres d’entrée d’un runbook graphique, créons un runbook qui génère des détails sur des machines virtuelles, soit une machine virtuelle unique, soit toutes les machines virtuelles au sein d’un groupe de ressources. Pour plus d’informations, consultez [Mon premier runbook graphique](./learn/automation-tutorial-runbook-graphical.md).

Un runbook graphique utilise ces activités principales :

* Configuration du compte d’identification Azure pour l’authentification auprès d’Azure. 
* Définition d’une applet de commande [Get-AzVM](/powershell/module/az.compute/get-azvm?view=azps-3.5.0) pour obtenir les propriétés d’une machine virtuelle.
* Utilisation de l’activité [Write-Output](/powershell/module/microsoft.powershell.utility/write-output) pour générer les noms des machines virtuelles. 

L’activité `Get-AzVM` définit deux entrées : le nom de la machine virtuelle et le nom du groupe de ressources. Étant donné que ces noms peuvent être différents à chaque démarrage du runbook, vous devez ajouter des paramètres d’entrée à votre runbook pour accepter ces entrées. Reportez-vous à [Création de graphiques dans Azure Automation](automation-graphical-authoring-intro.md).

Effectuez ces étapes pour configurer les paramètres d’entrée.

1. Sélectionnez le runbook graphique dans la page Runbooks, puis cliquez sur **Modifier**.
2. Dans l’éditeur graphique, cliquez sur le bouton **Entrée et sortie**, puis le bouton **Ajouter une entrée** pour ouvrir le volet Paramètre d’entrée de runbook.

   ![Runbook graphique d’Automation](media/automation-runbook-input-parameters/automation-02-graphical-runbok-editor.png)

3. Le contrôle Entrée et sortie présente la liste des paramètres d’entrée définis pour le runbook. Vous pouvez alors ajouter un nouveau paramètre d’entrée ou modifier la configuration d’un paramètre d’entrée existant. Pour ajouter un nouveau paramètre pour le runbook, cliquez sur **Ajouter une entrée** pour ouvrir le panneau **Paramètre d’entrée de runbook**, où vous pouvez configurer des paramètres à partir des propriétés définies dans [Création de graphiques dans Azure Automation](automation-graphical-authoring-intro.md).

    ![Ajouter une nouvelle entrée](media/automation-runbook-input-parameters/automation-runbook-input-parameter-new.png)
4. Créez deux paramètres avec les propriétés suivantes qui seront utilisées par l’activité `Get-AzVM`, puis cliquez sur **OK**.

   * Paramètre 1 :
        * **Nom** -- **VMName**
        * **Type** -- Chaîne
        * **Obligatoire** -- **Non**

   * Paramètre 2 :
        * **Nom** -- **resourceGroupName**
        * **Type** -- Chaîne
        * **Obligatoire** -- **Non**
        * **Valeur par défaut** -- **Personnalisé**
        * Valeur par défaut personnalisée -- Nom du groupe de ressources qui contient les machines virtuelles

5. Affichez les paramètres dans le contrôle Entrée et sortie. 
6. Recliquez sur **OK**, puis sur **Enregistrer**.
7. Cliquez sur **Publier** pour publier votre runbook.

### <a name="configure-input-parameters-in-python-runbooks"></a>Configurer des paramètres d’entrée dans des Runbooks Python

Contrairement aux runbooks PowerShell, PowerShell Workflow et graphiques, les runbooks Python ne prennent pas de paramètres nommés. L’éditeur de runbook analyse tous les paramètres d’entrée sous la forme d’un tableau de valeurs d’arguments. Vous pouvez accéder au tableau en important le module `sys` dans votre script Python, puis en utilisant le tableau `sys.argv`. Il est important de noter que le premier élément du tableau, `sys.argv[0]`, est le nom du script. Par conséquent, le premier paramètre d’entrée réel est `sys.argv[1]`.

Pour obtenir un exemple montrant comment utiliser les paramètres d’entrée dans un runbook Python, consultez [My first Python runbook in Azure Automation](./learn/automation-tutorial-runbook-textual-python2.md) (Mon premier runbook Python dans Azure Automation).

## <a name="assign-values-to-input-parameters-in-runbooks"></a>Affecter des valeurs aux paramètres d’entrée dans des Runbooks

Cette section décrit plusieurs façons de passer des valeurs aux paramètres d’entrée dans des runbooks. Vous pouvez affecter des valeurs de paramètre lors des opérations suivantes :

* [Démarrer un runbook](#start-a-runbook-and-assign-parameters)
* [Tester un runbook](#test-a-runbook-and-assign-parameters)
* [Lier une planification pour le runbook](#link-a-schedule-to-a-runbook-and-assign-parameters)
* [Créer un webhook pour le runbook](#create-a-webhook-for-a-runbook-and-assign-parameters)

### <a name="start-a-runbook-and-assign-parameters"></a>Démarrer un Runbook et affecter des paramètres

Un runbook peut être démarré de plusieurs façons : par le biais du portail Azure, avec un webhook, avec des applets de commande PowerShell, avec l’API REST ou avec le SDK. 

#### <a name="start-a-published-runbook-using-the-azure-portal-and-assign-parameters"></a>Démarrer un runbook publié à l’aide du portail Azure et affecter des paramètres

Au [démarrage du runbook](start-runbooks.md#start-a-runbook-with-the-azure-portal) dans le portail Azure, le panneau **Démarrer le runbook** s’ouvre. Vous pouvez y entrer les valeurs des paramètres que avez créés.

![Démarrer à l’aide du portail](media/automation-runbook-input-parameters/automation-04-startrunbookusingportal.png)

Dans l’étiquette située sous la zone d’entrée, vous pouvez voir les propriétés qui ont été définies pour les attributs de paramètre, par exemple un caractère obligatoire ou facultatif, un type, une valeur par défaut. La bulle d’aide située en regard du nom de paramètre définit aussi les informations clés dont vous avez besoin pour prendre des décisions sur les valeurs d’entrée du paramètre. 

> [!NOTE]
> Les paramètres de chaîne prennent en charge des valeurs vides de type chaîne. Le fait d’entrer `[EmptyString]` dans la zone de paramètre d’entrée a pour effet de transmettre une chaîne vide au paramètre. De plus, les paramètres de chaîne ne prennent pas en charge la valeur Null. Si vous ne transmettez pas de valeur à un paramètre de chaîne, PowerShell interprète sa valeur comme étant Null.

#### <a name="start-a-published-runbook-using-powershell-cmdlets-and-assign-parameters"></a>Démarrer un runbook publié à l’aide d’applets de commande PowerShell et affecter des paramètres

* **Applets de commande Azure Resource Manager :** vous pouvez démarrer un runbook Automation créé dans un groupe de ressources en utilisant [Start-AzAutomationRunbook](/powershell/module/Az.Automation/Start-AzAutomationRunbook?view=azps-3.5.0).

   ```powershell
     $params = @{"VMName"="WSVMClassic";"resourceGroupeName"="WSVMClassicSG"}
  
     Start-AzAutomationRunbook -AutomationAccountName "TestAutomation" -Name "Get-AzureVMGraphical" –ResourceGroupName $resourceGroupName -Parameters $params
   ```

* **Applets de commande du modèle de déploiement Azure Classic :** vous pouvez démarrer un runbook Automation créé dans un groupe de ressources par défaut en utilisant [Start-AzureAutomationRunbook](/powershell/module/servicemanagement/azure.service/start-azureautomationrunbook).
  
   ```powershell
     $params = @{"VMName"="WSVMClassic"; "ServiceName"="WSVMClassicSG"}
  
     Start-AzureAutomationRunbook -AutomationAccountName "TestAutomation" -Name "Get-AzureVMGraphical" -Parameters $params
   ```

> [!NOTE]
> Quand vous démarrez un runbook à l’aide d’applets de commande PowerShell, un paramètre par défaut, `MicrosoftApplicationManagementStartedBy`, est créé avec la valeur `PowerShell`. Vous pouvez voir ce paramètre dans le volet Détails du travail.  

#### <a name="start-a-runbook-using-an-sdk-and-assign-parameters"></a>Démarrer un runbook à l’aide d’un SDK et affecter des paramètres

* **Méthode d’Azure Resource Manager :** Vous pouvez démarrer un runbook en utilisant le SDK d’un langage de programmation. Voici un extrait de code C# permettant de démarrer un Runbook dans votre compte Automation. Vous pouvez voir le code complet dans notre [dépôt GitHub](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/automation/Microsoft.Azure.Management.Automation/tests/TestSupport/AutomationTestBase.cs).

   ```csharp
   public Job StartRunbook(string runbookName, IDictionary<string, string> parameters = null)
      {
        var response = AutomationClient.Jobs.Create(resourceGroupName, automationAccount, new JobCreateParameters
         {
            Properties = new JobCreateProperties
             {
                Runbook = new RunbookAssociationProperty
                 {
                   Name = runbookName
                 },
                   Parameters = parameters
             }
         });
      return response.Job;
      }
   ```

* **Méthode du modèle de déploiement Azure Classic :** vous pouvez démarrer un runbook en utilisant le kit SDK d’un langage de programmation. Voici un extrait de code C# permettant de démarrer un Runbook dans votre compte Automation. Vous pouvez voir le code complet dans notre [dépôt GitHub](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/automation/Microsoft.Azure.Management.Automation/tests/TestSupport/AutomationTestBase.cs).

   ```csharp
  public Job StartRunbook(string runbookName, IDictionary<string, string> parameters = null)
    {
      var response = AutomationClient.Jobs.Create(automationAccount, new JobCreateParameters
    {
      Properties = new JobCreateProperties
         {
           Runbook = new RunbookAssociationProperty
         {
           Name = runbookName
              },
                Parameters = parameters
              }
       });
      return response.Job;
    }
   ```

   Pour lancer cette méthode, créez un dictionnaire pour stocker les paramètres de runbook `VMName` et `resourceGroupName` ainsi que leurs valeurs. Démarrez ensuite le Runbook. Voici l’extrait de code C# permettant d’appeler la méthode définie ci-dessus.

   ```csharp
   IDictionary<string, string> RunbookParameters = new Dictionary<string, string>();
  
   // Add parameters to the dictionary.
  RunbookParameters.Add("VMName", "WSVMClassic");
   RunbookParameters.Add("resourceGroupName", "WSSC1");
  
   //Call the StartRunbook method with parameters
   StartRunbook("Get-AzureVMGraphical", RunbookParameters);
   ```

#### <a name="start-a-runbook-using-the-rest-api-and-assign-parameters"></a>Démarrer un runbook à l’aide de l’API REST et affecter des paramètres

Vous pouvez créer et démarrer une tâche de runbook avec l’API REST d’Azure Automation en utilisant la méthode `PUT` avec l’URI de demande suivant : `https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Automation/automationAccounts/{automationAccountName}/jobs/{jobName}?api-version=2017-05-15-preview`

Dans l’URI de demande, remplacez les paramètres suivants :

* `subscriptionId`: ID de votre abonnement Azure.  
* `resourceGroupName`: Nom du groupe de ressources du compte Automation.
* `automationAccountName`: Nom de votre compte Automation hébergé dans le service cloud spécifié.  
* `jobName`: GUID du travail. Vous pouvez créer des GUID dans PowerShell à l’aide de `[GUID]::NewGuid().ToString()*`.

Pour transmettre des paramètres au travail du runbook, utilisez le corps de la demande. Ce dernier prend les informations suivantes, fournies au format JSON :

* Nom du runbook : Obligatoire. Nom du Runbook pour le démarrage de la tâche.  
* Paramètres du runbook : facultatif. Dictionnaire de la liste de paramètres au format (nom, valeur) où nom doit être un nom de type String, et valeur peut être toute valeur JSON valide.

Si vous voulez démarrer le runbook **Get-AzureVMTextual** créé précédemment avec `VMName` et `resourceGroupName` comme paramètres, utilisez le format JSON suivant pour le corps de la demande.

```json
    {
      "properties":{
        "runbook":{
        "name":"Get-AzureVMTextual"},
      "parameters":{
         "VMName":"WindowsVM",
         "resourceGroupName":"ContosoSales"}
        }
    }
```

Un code d’état HTTP 201 est retourné si la tâche est correctement créée. Pour plus d’informations sur les en-têtes et le corps de la réponse, consultez [Créer un travail de runbook à l’aide de l’API REST](/rest/api/automation/job/create).

### <a name="test-a-runbook-and-assign-parameters"></a>Tester un Runbook et affecter des paramètres

Quand vous [testez la version brouillon de votre runbook](./manage-runbooks.md) à l’aide de l’option de test, la page Test s’ouvre. Utilisez cette page pour configurer des valeurs pour les paramètres que vous avez créés.

![Tester et affecter des paramètres](media/automation-runbook-input-parameters/automation-06-testandassignparameters.png)

### <a name="link-a-schedule-to-a-runbook-and-assign-parameters"></a>Lier une planification à un Runbook et affecter des paramètres

Vous pouvez [lier une planification](./shared-resources/schedules.md) à votre Runbook afin de démarrer celui-ci à un moment spécifique. Lors de la création de la planification, vous affectez des paramètres d’entrée, que le runbook utilise quand la planification le démarre. Vous ne pouvez pas enregistrer la planification tant que toutes les valeurs de paramètres obligatoires n’ont pas été fournies.

![Planifier et affecter des paramètres](media/automation-runbook-input-parameters/automation-07-scheduleandassignparameters.png)

### <a name="create-a-webhook-for-a-runbook-and-assign-parameters"></a>Créer un WebHook pour un Runbook et affecter des paramètres

Vous pouvez créer un [WebHook](automation-webhooks.md) pour votre Runbook, puis configurer les paramètres d’entrée de celui-ci. Vous ne pouvez pas enregistrer le webhook tant que toutes les valeurs de paramètres obligatoires n’ont pas été fournies.

![Créer un WebHook et affecter des paramètres](media/automation-runbook-input-parameters/automation-08-createwebhookandassignparameters.png)

Quand vous exécutez un runbook à l’aide d’un webhook, le paramètre d’entrée prédéfini `[WebhookData](automation-webhooks.md)` est envoyé avec les paramètres d’entrée que vous définissez. 

![Paramètre WebhookData](media/automation-runbook-input-parameters/automation-09-webhook-data-parameters.png)

## <a name="pass-a-json-object-to-a-runbook"></a>Passer un objet JSON à un runbook

Il peut être utile stocker les données à passer dans un runbook dans un fichier JSON. Par exemple, vous pouvez créer un fichier JSON qui contient tous les paramètres que vous voulez passer à un runbook. Pour ce faire, vous devez convertir le code JSON en chaîne, puis convertir la chaîne en objet PowerShell avant de la passer au runbook.

Cette section utilise un exemple dans lequel un script PowerShell appelle [Start-AzAutomationRunbook](/powershell/module/az.automation/start-azautomationrunbook?view=azps-3.5.0) pour démarrer un runbook PowerShell, en passant le contenu du fichier JSON au runbook. Le runbook PowerShell démarre une machine virtuelle Azure, en récupérant les paramètres de la machine virtuelle à partir de l’objet JSON.

### <a name="create-the-json-file"></a>Créer le fichier JSON

Tapez le code suivant dans un fichier texte et enregistrez-le sous le nom **test-json** quelque part sur votre ordinateur local.

```json
{
   "VmName" : "TestVM",
   "ResourceGroup" : "AzureAutomationTest"
}
```

### <a name="create-the-runbook"></a>Créer le runbook

Créez un runbook PowerShell nommé **Test-Json** dans Azure Automation. Consultez [Mon premier runbook PowerShell](./learn/automation-tutorial-runbook-textual-powershell.md).

Pour accepter les données JSON, le runbook doit prendre un objet comme paramètre d’entrée. Le runbook peut alors utiliser les propriétés définies dans le fichier JSON.

```powershell
Param(
     [parameter(Mandatory=$true)]
     [object]$json
)

# Connect to Azure account
$Conn = Get-AutomationConnection -Name AzureRunAsConnection
Connect-AzAccount -ServicePrincipal -Tenant $Conn.TenantID `
    -ApplicationID $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint

# Convert object to actual JSON
$json = $json | ConvertFrom-Json

# Use the values from the JSON object as the parameters for your command
Start-AzVM -Name $json.VMName -ResourceGroupName $json.ResourceGroup
```

Enregistrez et publiez ce runbook dans votre compte Automation.

### <a name="call-the-runbook-from-powershell"></a>Appeler le runbook à partir de PowerShell

Vous pouvez maintenant appeler le runbook à partir de votre ordinateur local à l’aide d’Azure PowerShell. 

1. Connectez-vous à Azure comme indiqué. Par la suite, vous êtes invité à entrer vos informations d’identification Azure.

   ```powershell
   Connect-AzAccount
   ```

    >[!NOTE]
    >Pour les runbooks PowerShell, `Add-AzAccount` et `Add-AzureRMAccount` sont des alias de `Connect-AzAccount`. Notez que ces alias ne sont pas disponibles pour les runbooks graphiques. Un runbook graphique peut uniquement utiliser `Connect-AzAccount`.

1. Obtenez le contenu du fichier JSON enregistré et les convertissez-le en chaîne. `JsonPath` indique le chemin où vous avez enregistré le fichier JSON.

   ```powershell
   $json =  (Get-content -path 'JsonPath\test.json' -Raw) | Out-string
   ```

1. Convertissez le contenu de la chaîne de `$json` en objet PowerShell.

   ```powershell
   $JsonParams = @{"json"=$json}
   ```

1. Créez une table de hachage pour les paramètres de `Start-AzAutomationRunbook`. 

   ```powershell
   $RBParams = @{
        AutomationAccountName = 'AATest'
        ResourceGroupName = 'RGTest'
        Name = 'Test-Json'
        Parameters = $JsonParams
   }
   ```

   Notez que vous définissez la valeur `Parameters` sur l'objet PowerShell contenant les valeurs du fichier JSON.
1. Démarrer le runbook.

   ```powershell
   $job = Start-AzAutomationRunbook @RBParams
   ```

## <a name="next-steps"></a>Étapes suivantes

* Pour préparer un runbook textuel, consultez [Modifier le runbooks textuel dans Azure Automation](automation-edit-textual-runbook.md).
* Pour préparer un runbook graphique, consultez [Création de graphiques dans Azure Automation](automation-graphical-authoring-intro.md).

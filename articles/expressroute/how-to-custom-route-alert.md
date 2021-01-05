---
title: 'ExpressRoute : Comment configurer des alertes personnalisées pour les itinéraires publiés'
description: Cet article explique comment utiliser Azure Automation et Logic Apps pour surveiller le nombre d’itinéraires publiés de la passerelle ExpressRoute jusqu’à des réseaux locaux afin de ne pas atteindre la limite des 200 itinéraires.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 05/29/2020
ms.author: duau
ms.openlocfilehash: a960150d68cf2f939e206321a20d98b0e4080313
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92371492"
---
# <a name="configure-custom-alerts-to-monitor-advertised-routes"></a>Configurer des alertes personnalisées pour superviser les routes publiées

Cet article vous aide à utiliser Azure Automation et Logic Apps pour surveiller en permanence le nombre d’itinéraires publiés de la passerelle ExpressRoute jusqu’à des réseaux locaux. La surveillance peut aider à ne pas atteindre la limite des [200 itinéraires](expressroute-faqs.md#how-many-prefixes-can-be-advertised-from-a-vnet-to-on-premises-on-expressroute-private-peering).

**Azure Automation** vous permet d’automatiser l’exécution du script PowerShell personnalisé stocké dans un *runbook* . Lorsque vous utilisez la configuration de cet article, le runbook contient un script PowerShell qui interroge une ou plusieurs passerelles ExpressRoute. Il collecte un jeu de données contenant le groupe de ressources, le nom de la passerelle ExpressRoute et le nombre de préfixes réseau publiés localement.

**Azure Logic Apps** planifie un workflow personnalisé qui appelle le runbook Azure Automation. L’exécution du runbook s’effectue à l’aide d’un travail. Après l’exécution de la collecte de données, le workflow d’Azure Logic Apps classe les données et envoie des informations à une adresse e-mail de destination, en fonction des critères de correspondance sur le nombre de préfixes réseau au-dessus ou en dessous d’un seuil prédéfini.

### <a name="workflow"></a><a name="workflow"></a>Flux de travail

La configuration d’une alerte personnalisée est basée sur trois grandes étapes :

1. Créer un compte Automation avec un compte d’identification et des autorisations.

2. Créer et configurer des runbooks.

3. Créer une application logique qui déclenche le compte Automation et envoie un e-mail d’alerte si le nombre est supérieur à un seuil (par exemple, 160).

## <a name="before-you-begin"></a><a name="before"></a>Avant de commencer

Vérifiez que vous disposez des éléments ci-dessous avant de commencer votre configuration :

* Vous avez au moins une passerelle ExpressRoute dans votre déploiement.

* Vous avez une connaissance de base des [comptes d’identification](../automation/manage-runas-account.md) dans Azure Automation.

* Vous avez une bonne connaissance d’[Azure Logic Apps](../logic-apps/logic-apps-overview.md).

* Vous maîtrisez l’utilisation d’Azure PowerShell. Azure PowerShell est nécessaire pour collecter les préfixes réseau dans la passerelle ExpressRoute. Pour plus d’informations sur Azure PowerShell en général, consultez la [documentation Azure PowerShell](/powershell/azure/?view=azps-4.1.0).

### <a name="notes-and-limitations"></a><a name="limitations"></a>Remarques et limitations

* L’alerte personnalisée abordée dans cet article est un module complémentaire permettant d’optimiser le fonctionnement et le contrôle. Elle ne remplace pas les alertes natives dans ExpressRoute.
* La collecte de données pour les passerelles ExpressRoute s’exécute en arrière-plan. La durée d’exécution peut être plus longue que prévu. Pour éviter la mise en file d’attente des travaux, la périodicité du workflow doit être correctement configurée.
* Les déploiements par scripts ou modèles Resource Manager pourraient se produire plus rapidement que le déclenchement d’une alarme personnalisée. Cela peut entraîner une augmentation du nombre de préfixes réseau dans la passerelle ExpressRoute au-delà de la limite des 200 itinéraires.

## <a name="create-and-configure-accounts"></a><a name="accounts"></a>Créer et configurer des comptes

Lorsque vous créez un compte Automation sur le portail Azure, un [compte d’identification](../automation/automation-security-overview.md#run-as-accounts) est automatiquement créé. Ce compte effectue les actions suivantes :

* Crée une application Azure Active Directory (Azure AD) avec un certificat autosigné. Le compte d’identification possède lui-même un certificat qui doit être renouvelé par défaut chaque année.

* Crée un compte de principal du service pour l’application dans Azure AD.

* S'attribue le rôle de Contributeur (Azure RBAC) sur l'abonnement Azure en cours d'utilisation. Ce rôle gère les ressources Azure Resource Manager à l’aide de runbooks.

Pour créer un compte Automation, vous avez besoin de privilèges et d’autorisations. Pour plus d’informations, consultez [Autorisations requises pour créer un compte Automation](../automation/automation-create-standalone-account.md#permissions-required-to-create-an-automation-account).

### <a name="1-create-an-automation-account"></a><a name="about"></a>1. Créer un compte Automation

Créez un compte Automation avec des autorisations d’exécution. Pour obtenir des instructions, consultez [Créer un compte Azure Automation](../automation/automation-quickstart-create-account.md).

:::image type="content" source="./media/custom-route-alert-portal/create-account.png" alt-text="Ajouter un compte Automation" lightbox="./media/custom-route-alert-portal/create-account-expand.png":::

### <a name="2-assign-the-run-as-account-a-role"></a><a name="about"></a>2. Attribuer un rôle au compte d’identification

Par défaut, le rôle **Contributeur** est attribué au principal de service utilisé par votre **compte d’identification** . Vous pouvez conserver le rôle par défaut attribué au principal du service, ou vous pouvez restreindre les autorisations en attribuant un [rôle intégré](../role-based-access-control/built-in-roles.md) (par exemple, Lecteur) ou un [rôle personnalisé](../active-directory/roles/custom-create.md).

 Procédez comme suit pour déterminer le rôle attribué au principal de service utilisé par votre compte d’identification :

1. Accédez à votre compte Automation. Accédez à **Paramètres du compte** , puis sélectionnez **Comptes d’identification** .

2. Sélectionnez **Rôle** pour afficher la définition de rôle en cours d’utilisation.

   :::image type="content" source="./media/custom-route-alert-portal/run-as-account-permissions.png" alt-text="Ajouter un compte Automation":::

## <a name="create-and-configure-runbooks"></a><a name="runbooks"></a>Créer et configurer des runbooks

### <a name="1-install-modules"></a><a name="install-modules"></a>1. Installer des modules

Pour exécuter des cmdlets PowerShell dans des runbooks Azure Automation, vous devez installer quelques modules Az Azure PowerShell supplémentaires. Pour installer les modules, procédez comme suit :

1. Ouvrez votre compte Azure Automation et accédez à **Modules** .

   :::image type="content" source="./media/custom-route-alert-portal/navigate-modules.png" alt-text="Ajouter un compte Automation":::

2. Recherchez les modules suivants dans la galerie et importez-les : **Az.Accounts** , **Az.Network** , **Az.Automation** et **Az.Profile** .

   :::image type="content" source="./media/custom-route-alert-portal/import-modules.png" alt-text="Ajouter un compte Automation" lightbox="./media/custom-route-alert-portal/import-modules-expand.png":::
  
### <a name="2-create-a-runbook"></a><a name="create"></a>2. Créer un runbook

1. Pour créer votre runbook PowerShell, accédez à votre compte Automation. Sous **Automatisation de processus** , sélectionnez la vignette **Runbooks** , puis sélectionnez **Créer un runbook** .

   :::image type="content" source="./media/custom-route-alert-portal/create-runbook.png" alt-text="Ajouter un compte Automation":::

2. Sélectionnez **Créer** pour créer le runbook.

   :::image type="content" source="./media/custom-route-alert-portal/create-runbook-2.png" alt-text="Ajouter un compte Automation":::

3. Sélectionnez le runbook que vous venez de créer, puis sélectionnez **Modifier** .

   :::image type="content" source="./media/custom-route-alert-portal/edit-runbook.png" alt-text="Ajouter un compte Automation":::

4. Dans **Modifier** , collez le script PowerShell. L’[exemple de script](#script) peut être modifié et utilisé pour surveiller les passerelles ExpressRoute dans un ou plusieurs groupes de ressources.

   Dans l’exemple de script, prêtez attention aux paramètres suivants :

    * Le tableau **$rgList** contient la liste des groupes de ressources ayant des passerelles ExpressRoute. Vous pouvez personnaliser les passerelles ExpressRoute basées sur une liste.
    * La variable **$thresholdNumRoutes** définit le seuil du nombre de préfixes réseau publiés d’une passerelle ExpressRoute aux réseaux locaux.

#### <a name="example-script"></a><a name="script"></a>Exemple de script

```powershell
  ################# Input parameters #################
# Resource Group Name where the ExR GWs resides in
$rgList= @('ASH-Cust10-02','ASH-Cust30')  
$thresholdNumRoutes = 160
###################################################

# Ensures you do not inherit an AzContext in your runbook
Disable-AzContextAutosave -Scope Process | Out-Null

Try {

   $conn = Get-AutomationConnection -Name 'AzureRunAsConnection'
   while(!($connectionResult) -And ($logonAttempt -le 5))
   {
        $LogonAttempt++
        # Logging in to Azure...
        $connectionResult =  Connect-AzAccount `
                               -ServicePrincipal `
                               -ApplicationId $conn.ApplicationId `
                               -Tenant $conn.TenantId `
                               -CertificateThumbprint $conn.CertificateThumbprint `
                               -Subscription $conn.SubscriptionId `
                               -Environment AzureCloud 
                               
        Start-Sleep -Seconds 10
    }
} Catch {
    if (!$conn)
    {
        $ErrorMessage = "Service principal not found."
        throw $ErrorMessage
    } 
    else
    {
        Write-Error -Message $_.Exception
        throw $_.Exception
    }
}

# Get the name of the Azure subscription
$subscriptionName=(Get-AzSubscription -SubscriptionId $conn.SubscriptionId).Name

#write-Output "<br>$(Get-Date) - selection of the Azure subscription: $subscriptionName" 
Select-AzSubscription -SubscriptionId $conn.SubscriptionId | Out-Null


$GtwList = @()
$results= @()

foreach ($rgName in $rgList)
{
## Collect all the ExpressRoute gateways in a Resource Group
$GtwList=Get-AzVirtualNetworkGateway -ResourceGroupName $rgName 

## For each ExpressRoute gateway, get the IP addresses of the BGP peers and collect the number of routes advertised 
foreach ($gw in $GtwList) {
  
  $peers = Get-AzVirtualNetworkGatewayBGPPeerStatus -VirtualNetworkGatewayName $gw.Name -ResourceGroupName $rgName


 if ($peers[0].State -eq 'Connected') {
   $routes1=$null
   $routes1 = Get-AzVirtualNetworkGatewayAdvertisedRoute -VirtualNetworkGatewayName $gw.Name -ResourceGroupName $rgName -Peer $peers[0].Neighbor
 }
  if ($peers[1].State -eq 'Connected') {
  
   $routes2=$null
   $routes2 = Get-AzVirtualNetworkGatewayAdvertisedRoute -VirtualNetworkGatewayName $gw.Name -ResourceGroupName $rgName -Peer $peers[1].Neighbor
 }
 
  $sampleTime=(Get-Date).ToString("dd-MM-yyyy HH:mm:ss")
  if ($routes1.Count -eq $routes2.Count)
  {
     
     if ($routes1.Count -lt $thresholdNumRoutes){
       $status='OK'
       $alertMsg='number of routes below threshold'
     } 
     else {
       $status='ALERT'
       $alertMsg='number of routes above threshold'
     }
  } 
  else
  {
     $status='WARNING'
     $alertMsg='check ER Gateway'
  }
  
  $obj = [psCustomObject]@{
            resourceGroup =$rgName
            nameGtw  = $gw.Name
            peer1 = $peers[0].Neighbor
            peer2 = $peers[1].Neighbor
            numRoutesPeer1=  $routes1.Count
            numRoutesPeer2=  $routes2.Count
            time=$sampleTime
            status=$status
            alertMessage = $alertMsg
        }
  $results += $obj
} ### end foreach gateways in each resource group
} ### end foreach resource group
$jsonResults= ConvertTo-Json $results -Depth 100
Write-Output  $jsonResults
 
 ```

### <a name="3-save-and-publish-the-runbook"></a><a name="publish"></a>3. Enregistrer et publier le runbook

1. Sélectionnez **Enregistrer** pour enregistrer un brouillon du runbook.
2. Sélectionnez **Publier** pour publier le runbook en tant que version officielle du runbook dans le compte Automation.

   :::image type="content" source="./media/custom-route-alert-portal/save-publish-runbook.png" alt-text="Ajouter un compte Automation":::

Lorsque vous exécutez le script PowerShell, une liste de valeurs est collectée :
 
* Resource group

* Nom de la passerelle ExpressRoute

* Adresse IP du premier homologue BGP (peer1)

* Adresse IP du deuxième homologue BGP (peer2)

* Nombre de préfixes réseau publiés de la passerelle ExpressRoute au premier homologue BGP (peer1)

* Nombre de préfixes réseau publiés de la passerelle ExpressRoute au deuxième homologue BGP (peer2)

* Timestamp

* État, classé comme :

  * « OK » si le nombre d’itinéraires est inférieur à une valeur seuil
  * « ALERT » si le nombre d’itinéraires est supérieur à une valeur seuil
  * « WARNING » si le nombre de préfixes réseau publiés sur les deux homologues BGP est différent

* Message d’alerte, pour une description détaillée de l’état (OK, ALERT, WARNING)

Le script PowerShell convertit les informations collectées en sortie JSON. Le runbook utilise la cmdlet PowerShell [Write-Output](/powershell/module/Microsoft.PowerShell.Utility/Write-Output) comme flux de sortie pour communiquer des informations au client.

### <a name="4-validate-the-runbook"></a><a name="validate"></a>4. Valider le runbook

Une fois le runbook créé, il doit être validé. Sélectionnez **Démarrer** et vérifiez la sortie et les erreurs pour les différents flux de travail.

:::image type="content" source="./media/custom-route-alert-portal/validate-runbook.png" alt-text="Ajouter un compte Automation" lightbox="./media/custom-route-alert-portal/validate-runbook-expand.png":::

## <a name="create-and-configure-a-logic-app"></a><a name="logic"></a>Créer et configurer une application logique

Azure Logic Apps est l’orchestrateur de tous les processus de collecte et d’actions. Dans les sections suivantes, vous allez créer un workflow à l’aide d’une application logique.

### <a name="workflow"></a>Workflow

Dans ce workflow, vous créez une application logique qui surveille régulièrement les passerelles ExpressRoute. Si de nouveaux éléments existent, l’application logique envoie un e-mail pour chacun d’eux. Lorsque vous avez terminé, votre application logique ressemble au flux de travail suivant à un niveau élevé :

:::image type="content" source="./media/custom-route-alert-portal/logic-apps-workflow.png" alt-text="Ajouter un compte Automation":::

### <a name="1-create-a-logic-app"></a>1. Créer une application logique

Dans **Concepteur d’application logique** , créez une application logique à l’aide du modèle **Application logique vide** . Pour connaître les étapes à suivre, consultez [Créer des applications logiques](../logic-apps/quickstart-create-first-logic-app-workflow.md#create-your-logic-app).

:::image type="content" source="./media/custom-route-alert-portal/blank-template.png" alt-text="Ajouter un compte Automation":::

### <a name="2-add-a-trigger"></a>2. Ajouter un déclencheur

Chaque application logique est démarrée par un déclencheur. Un déclencheur s’active lorsqu’un événement spécifique se produit ou lorsqu’une condition particulière est remplie. Chaque fois que le déclencheur s’active, le moteur Azure Logic Apps crée une instance d’application logique qui démarre et exécute votre flux de travail.

Pour exécuter régulièrement une application logique basée sur un calendrier prédéfini, ajoutez la **Périodicité : Planification** intégrée à votre workflow. Dans la zone de recherche, saisissez **Planification** . Sélectionnez **Déclencheurs** . Dans la liste Déclencheurs, sélectionnez **Planification de la périodicité** .

:::image type="content" source="./media/custom-route-alert-portal/schedule.png" alt-text="Ajouter un compte Automation":::

Dans le déclencheur Planification de la périodicité, vous pouvez définir le fuseau horaire et une périodicité pour répéter ce workflow. Ensemble, l’intervalle et la fréquence définissent la planification du déclencheur de votre application logique. Pour établir une fréquence de périodicité minimale raisonnable, tenez compte des facteurs suivants :

* Le script PowerShell du runbook Automation prend du temps pour se terminer. La durée d’exécution dépend du nombre de passerelles ExpressRoute à surveiller. Une fréquence de périodicité trop courte entraîne la mise en file d’attente des travaux.

* Le script PowerShell s’exécute en tant que travail en arrière-plan. Il ne démarre pas immédiatement, mais s’exécute après un retard variable.

* Une fréquence de périodicité trop courte génère une charge inutile sur vos passerelles Azure ExpressRoute.

À la fin de la configuration du workflow, vous pouvez vérifier la cohérence de la fréquence de périodicité en exécutant plusieurs fois le workflow, puis en vérifiant le résultat dans l’ **historique des exécutions** .

:::image type="content" source="./media/custom-route-alert-portal/recurrence.png" alt-text="Ajouter un compte Automation" lightbox="./media/custom-route-alert-portal/recurrence-expand.png":::

### <a name="3-create-a-job"></a><a name="job"></a>3. Créer un travail

Une application logique accède à d’autres applications, à des services et à la plateforme via des connecteurs. L’étape suivante de ce workflow consiste à sélectionner un connecteur pour accéder au compte Azure Automation qui a été défini précédemment.

1. Dans **Concepteur d’application logique** , sous **Périodicité** , sélectionnez **Nouvelle étape** . Sous **Choisir une action** et la zone de recherche, sélectionnez **Tous** .
2. Dans la zone de recherche, saisissez **Azure Automation** et lancez la recherche. Sélectionnez **Créer un travail** . **Créer un travail** sera utilisé pour déclencher le runbook Automation créé précédemment.

   :::image type="content" source="./media/custom-route-alert-portal/create-job.png" alt-text="Ajouter un compte Automation":::

3. Connectez-vous à l’aide d’un principal de service. Vous pouvez utiliser un principal de service existant ou en créer un. Pour créer un principal de service, consultez [Comment utiliser le portail pour créer un principal de service Azure AD pouvant accéder aux ressources](../active-directory/develop/howto-create-service-principal-portal.md). Sélectionnez **Se connecter avec le principal de service** .

   :::image type="content" source="./media/custom-route-alert-portal/sign-in.png" alt-text="Ajouter un compte Automation":::

4. Saisissez un **nom de connexion** , ajoutez votre **ID client** (ID d’application), votre **clé secrète client** et votre **ID de locataire** . Sélectionnez ensuite **Create** (Créer).

   :::image type="content" source="./media/custom-route-alert-portal/connect-service-principal.png" alt-text="Ajouter un compte Automation":::

5. Sur la page **Créer un travail** , le principal de service doit avoir le rôle « Lecteur » sur le **groupe de ressources** hébergeant le compte Automation et « Opérateur de travaux Automation » sur le **compte Automation** . En outre, vérifiez que vous avez ajouté le **nom du runbook** comme nouveau paramètre.

   :::image type="content" source="./media/custom-route-alert-portal/roles.png" alt-text="Ajouter un compte Automation" lightbox="./media/custom-route-alert-portal/roles-expand.png":::

### <a name="4-get-the-job-output"></a><a name="output"></a>4. Obtenir la sortie du travail

1. Sélectionnez **Nouvelle étape** . Recherchez « Azure Automation ». Dans la liste **Actions** , sélectionnez **Obtenir la sortie du travail** .

   :::image type="content" source="./media/custom-route-alert-portal/get-output.png" alt-text="Ajouter un compte Automation":::

2. Sur la page **Obtenir la sortie du travail** , spécifiez les informations requises pour accéder au compte Automation. Sélectionnez **l’abonnement, le groupe de ressources** et **le compte Automation** que vous souhaitez utiliser. Cliquez à l’intérieur de la zone **ID de travail** . Lorsque la liste **Contenu dynamique** s’affiche, sélectionnez **ID de travail** .

   :::image type="content" source="./media/custom-route-alert-portal/job-id.png" alt-text="Ajouter un compte Automation" lightbox="./media/custom-route-alert-portal/job-id-expand.png":::

### <a name="5-parse-the-json"></a><a name="parse"></a>5. Analyser le JSON

Les informations contenues dans la sortie de « l’action Azure Automation Créer un travail » (étapes précédentes) génèrent un objet JSON. L’action Logic Apps **Analyser JSON** est une action intégrée pour créer des jetons conviviaux à partir des propriétés et de leurs valeurs dans le contenu JSON. Vous pouvez ensuite utiliser ces propriétés dans votre workflow.

1. Ajoutez une action. Sous **Obtenir la sortie du travail -> action** , sélectionnez **Nouvelle étape** .
2. Dans la zone de recherche **Choisir une action** , saisissez « analyser json » pour rechercher des connecteurs qui proposent cette action. Dans la liste **Actions** , sélectionnez l’action **Analyser JSON** pour les opérations de données que vous souhaitez utiliser.

   :::image type="content" source="./media/custom-route-alert-portal/parse-json.png" alt-text="Ajouter un compte Automation":::

3. Cliquez dans la zone **Contenu** . Lorsque la liste Contenu dynamique s’affiche, sélectionnez **Contenu** .

   :::image type="content" source="./media/custom-route-alert-portal/content.png" alt-text="Ajouter un compte Automation" lightbox="./media/custom-route-alert-portal/content-expand.png":::

4. L’analyse d’un JSON requiert un schéma. Le schéma peut être généré à l’aide de la sortie du runbook Automation. Ouvrez une nouvelle session de navigateur web, exécutez le runbook Automation et récupérez la sortie. Revenez à l’action **Logic Apps Analyser les opérations de données JSON** . En bas de la page, sélectionnez **Utiliser l’exemple de charge utile pour générer le schéma** .

   :::image type="content" source="./media/custom-route-alert-portal/sample-payload.png" alt-text="Ajouter un compte Automation":::

5. Pour **Entrer ou coller un exemple de charge utile JSON** , collez la sortie du runbook Automation et sélectionnez **Terminé** .

   :::image type="content" source="./media/custom-route-alert-portal/paste-payload.png" alt-text="Ajouter un compte Automation" lightbox="./media/custom-route-alert-portal/paste-payload-expand.png":::

6. Un schéma est généré automatiquement par l’analyse de la charge utile d’entrée JSON.

   :::image type="content" source="./media/custom-route-alert-portal/generate-schema.png" alt-text="Ajouter un compte Automation" lightbox="./media/custom-route-alert-portal/generate-schema-expand.png":::

### <a name="6-define-and-initialize-a-variable"></a><a name="define-variable"></a>6. Définir et initialiser une variable

Dans cette étape du workflow, nous créons une condition pour envoyer une alerte par e-mail. Pour une mise en forme flexible et personnalisée du corps d’un e-mail, une variable auxiliaire est introduite dans le workflow.

1. Sous l’action **Obtenir la sortie du travail** , sélectionnez **Nouvelle étape** . Dans la zone de recherche, recherchez et sélectionnez **Variables** .

   :::image type="content" source="./media/custom-route-alert-portal/variables.png" alt-text="Ajouter un compte Automation":::

2. Dans la liste **Actions** , sélectionnez l’action **Initialiser la variable** .

   :::image type="content" source="./media/custom-route-alert-portal/initialize-variables.png" alt-text="Ajouter un compte Automation":::

3. Spécifiez le nom de la variable. Pour **Type** , sélectionnez **Chaîne** . La **valeur** de la variable sera attribuée plus tard dans le workflow.

   :::image type="content" source="./media/custom-route-alert-portal/string.png" alt-text="Ajouter un compte Automation" lightbox="./media/custom-route-alert-portal/string-expand.png":::

### <a name="7-create-a-for-each-action"></a><a name="cycles-json"></a>7. Créer une action « For each »

Une fois le JSON analysé, l’action **Analyser les opérations de données JSON** stocke le contenu dans la sortie *Corps* . Pour traiter la sortie, vous pouvez créer une boucle « For each » qui répète une ou plusieurs actions sur chaque élément du tableau.

1. Sous **Initialiser la variable** , sélectionnez **Ajouter une action** . Dans la zone de recherche, tapez « for each » comme filtre.

   :::image type="content" source="./media/custom-route-alert-portal/control.png" alt-text="Ajouter un compte Automation":::

2. Dans la liste **Actions** , sélectionnez l’action **For each – Contrôle** .

   :::image type="content" source="./media/custom-route-alert-portal/for-each.png" alt-text="Ajouter un compte Automation":::

3. Cliquez dans la zone de texte **Sélectionnez un résultat à partir des étapes précédentes** . Lorsque la liste **Contenu dynamique** s’affiche, sélectionnez le **corps** , qui est la sortie du JSON analysé.

   :::image type="content" source="./media/custom-route-alert-portal/body.png" alt-text="Ajouter un compte Automation":::

4. Pour chaque élément du corps JSON, nous souhaitons définir une condition. Dans le groupe d’actions, sélectionnez **Contrôle** .

   :::image type="content" source="./media/custom-route-alert-portal/condition-control.png" alt-text="Ajouter un compte Automation":::

5. Dans la liste **Actions** , sélectionnez **Condition-Contrôle** . La Condition-Contrôle est une structure de contrôle qui compare les données de votre workflow à des valeurs ou à des champs spécifiques. Vous pouvez ensuite spécifier différentes actions à exécuter selon que les données remplissent ou non la condition.

   :::image type="content" source="./media/custom-route-alert-portal/condition.png" alt-text="Ajouter un compte Automation":::

6. À la racine de l’action **Condition** , remplacez l’opération logique par **Ou** .

   :::image type="content" source="./media/custom-route-alert-portal/condition-or.png" alt-text="Ajouter un compte Automation" lightbox="./media/custom-route-alert-portal/condition-or-expand.png":::

7. Vérifiez la valeur du nombre de préfixes réseau qu’une passerelle ExpressRoute publie sur les deux homologues BGP. Le nombre d’itinéraires est disponible dans « numRoutePeer1 » et « numRoutePeer2 » dans **Contenu dynamique** . Dans la zone valeur, saisissez la valeur de **numRoutePeer1** .

   :::image type="content" source="./media/custom-route-alert-portal/peer-1.png" alt-text="Ajouter un compte Automation":::

8. Pour ajouter une autre ligne à votre condition, choisissez **Ajouter -> Ajouter une ligne** . Dans la deuxième zone, à partir de **Contenu dynamique** , sélectionnez **numRoutePeer2** .

   :::image type="content" source="./media/custom-route-alert-portal/peer-2.png" alt-text="Ajouter un compte Automation":::

9. La condition logique est vraie lorsque l’une des deux variables dynamiques, numRoute1 ou numRoute2, est supérieure au seuil. Dans cet exemple, le seuil est fixé à 160 (80 % de la valeur maximale des 200 itinéraires). Vous pouvez modifier la valeur seuil en fonction de vos besoins. Par souci de cohérence, la valeur doit être la même que celle utilisée dans le script PowerShell du runbook.

   :::image type="content" source="./media/custom-route-alert-portal/logic-condition.png" alt-text="Ajouter un compte Automation":::

10. Sous **Si vraie** , mettez en forme et créez les actions pour envoyer l’alerte par e-mail. Dans **Choisir une action**, recherchez et sélectionnez **Variables** .

    :::image type="content" source="./media/custom-route-alert-portal/condition-if-true.png" alt-text="Ajouter un compte Automation":::

11. Dans Variables, sélectionnez **Ajouter une action** . Dans la liste **Actions** , sélectionnez **Définir une variable** .

    :::image type="content" source="./media/custom-route-alert-portal/condition-set-variable.png" alt-text="Ajouter un compte Automation":::

12. Dans **Nom** , sélectionnez la variable nommée **EmailBody** que vous avez créée précédemment. Pour **Valeur** , collez le script HTML requis pour mettre en forme l’e-mail d’alerte. Utilisez le **contenu dynamique** pour inclure les valeurs du corps JSON. Une fois ces paramètres configurés, le résultat est que la variable **EmailBody** contient toutes les informations relatives à l’alerte, au format HTML.

    :::image type="content" source="./media/custom-route-alert-portal/paste-script.png" alt-text="Ajouter un compte Automation":::

### <a name="8-add-the-email-connector"></a><a name="email"></a>8. Ajouter le connecteur de messagerie

Logic Apps fournit de nombreux connecteurs de messagerie. Dans cet exemple, nous ajoutons un connecteur Outlook pour envoyer l’alerte par e-mail. Sous **Définir la variable** , sélectionnez **Ajouter une action** . Dans **Choisir une action** , saisissez « envoyer un e-mail » dans la zone de recherche.

1. Sélectionnez **Office 365 Outlook** .

   :::image type="content" source="./media/custom-route-alert-portal/email.png" alt-text="Ajouter un compte Automation":::

2. Dans la liste **Actions** , sélectionnez **Envoyer un e-mail (v2)** .

   :::image type="content" source="./media/custom-route-alert-portal/email-v2.png" alt-text="Ajouter un compte Automation":::

3. Connectez-vous pour créer une connexion à Office 365 Outlook.

   :::image type="content" source="./media/custom-route-alert-portal/office-365.png" alt-text="Ajouter un compte Automation":::

4. Dans le champ **Corps** , cliquez sur **Ajouter du contenu dynamique** . À partir du panneau Contenu dynamique, ajoutez la variable **EmailBody** . Remplissez les champs **Objet** et **À** .

   :::image type="content" source="./media/custom-route-alert-portal/emailbody.png" alt-text="Ajouter un compte Automation":::

5. L’action **Envoyer un e-mail (v2)** termine la configuration du workflow.

   :::image type="content" source="./media/custom-route-alert-portal/send-email-v2.png" alt-text="Ajouter un compte Automation" lightbox="./media/custom-route-alert-portal/send-email-v2-expand.png":::

### <a name="9-workflow-validation"></a><a name="validation"></a>9. Validation de workflow

La dernière étape est la validation du workflow. Dans **Vue d’ensemble de Logic Apps** , sélectionnez **Exécuter le déclencheur** . Sélectionnez **Périodicité** . Le workflow peut être surveillé et vérifié dans l’ **historique des exécutions** .

:::image type="content" source="./media/custom-route-alert-portal/trigger.png" alt-text="Ajouter un compte Automation":::

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur la personnalisation du workflow, consultez [Azure Logic Apps](../logic-apps/logic-apps-overview.md).
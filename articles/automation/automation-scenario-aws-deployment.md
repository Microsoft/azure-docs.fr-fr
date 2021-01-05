---
title: Déployer une machine virtuelle Amazon Web Services avec un runbook Azure Automation
description: Cet article explique comment automatiser la création d’une machine virtuelle Amazon Web Services.
services: automation
ms.subservice: process-automation
ms.date: 03/16/2018
ms.topic: conceptual
ms.openlocfilehash: 456a7e4f07b2416e1d2037205574f2e7149e70e2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86185940"
---
# <a name="deploy-an-amazon-web-services-vm-with-a-runbook"></a>Déployer une machine virtuelle Amazon Web Services avec un runbook

Dans cet article, vous allez découvrir comment provisionner une machine virtuelle de votre abonnement AWS (Amazon Web Services) à l’aide d’Azure Automation et comment lui attribuer un nom spécifique (opération appelée « balisage » dans AWS).

## <a name="prerequisites"></a>Prérequis

Vous devez disposer d’un compte Azure Automation et d’un abonnement Amazon Web Services (AWS). Pour plus d’informations sur la création d’un compte Azure Automation et sur sa configuration avec les informations d’identification de votre abonnement AWS, consultez l’article [Authentification des Runbooks avec Amazon Web Services](automation-config-aws-account.md). Vous devez au préalable créer ou mettre à jour ce compte avec les informations d’identification de votre abonnement AWS, car vous ferez référence à ce compte dans les sections suivantes.

## <a name="deploy-amazon-web-services-powershell-module"></a>Déployer le module PowerShell Amazon Web Services

Votre runbook d'approvisionnement de machine virtuelle utilise le module PowerShell AWS pour exécuter cette tâche. Pour ajouter le module à votre compte Automation qui est configuré avec les informations d’identification de votre abonnement AWS, procédez comme suit.  

1. Ouvrez votre navigateur web et accédez à [PowerShell Gallery](https://www.powershellgallery.com/packages/AWSPowerShell/), puis cliquez sur le bouton **Deploy to Azure Automation** (Déployer sur Azure Automation).<br><br> ![Importation du module PS AWS](./media/automation-scenario-aws-deployment/powershell-gallery-download-awsmodule.png)
2. Vous accédez alors à la page de connexion Azure. Après vous être authentifié, vous êtes dirigé vers le portail Azure dans la page suivante :<br><br> ![Page Importer le module](./media/automation-scenario-aws-deployment/deploy-aws-powershell-module-parameters.png)
3. Sélectionnez le compte Automation à utiliser et cliquez sur **OK** pour lancer le déploiement.

   > [!NOTE]
   > Lorsqu'il importe un module PowerShell, Azure Automation extrait les cmdlets. Les activités apparaissent uniquement quand Automation a terminé l’importation du module et extrait les cmdlets. Ce processus peut prendre plusieurs minutes.  
   > <br>

1. Dans le portail Azure, ouvrez votre compte Automation.
2. Cliquez sur la mosaïque **Ressources** et, dans le volet Ressources, sélectionnez **Modules**.
3. Dans la page Modules, le module **AWSPowerShell** apparaît dans la liste.

## <a name="create-aws-deploy-vm-runbook"></a>Créer un runbook de machine virtuelle de déploiement AWS

Une fois le module PowerShell AWS déployé, vous pouvez créer un runbook pour automatiser le provisionnement d’une machine virtuelle dans AWS au moyen d’un script PowerShell. La procédure ci-dessous montre comment utiliser un script PowerShell natif dans Azure Automation.  

> [!NOTE]
> Pour plus d’options et d’informations concernant ce script, visitez [PowerShell Gallery](https://www.powershellgallery.com/packages/New-AwsVM/).
> 

1. Téléchargez le script PowerShell New-AwsVM à partir de PowerShell Gallery en ouvrant une session PowerShell et en entrant la commande suivante :<br>
   ```powershell
   Save-Script -Name New-AwsVM -Path <path>
   ```
   <br>
2. Dans le portail Azure, ouvrez votre compte Automation et sélectionnez **Runbooks** sous **Automatisation de processus**.  
3. Dans la page Runbooks, sélectionnez **Ajouter un runbook**.
4. Dans le volet Ajouter un runbook, sélectionnez **Création rapide** pour créer un runbook.
5. Dans le volet Propriétés du runbook, entrez un nom pour votre runbook.
6. Dans la liste déroulante **Type de runbook**, sélectionnez **PowerShell**, puis cliquez sur **Créer**.<br><br> ![Volet Créer un Runbook](./media/automation-scenario-aws-deployment/runbook-quickcreate-properties.png)
7. Lorsque la page Modifier le Runbook PowerShell apparaît, copiez et collez le script PowerShell dans le canevas de création de runbook.<br><br> ![Script PowerShell de runbook](./media/automation-scenario-aws-deployment/runbook-powershell-script.png)<br>
   
    Notez les points suivants au moment d’utiliser l’exemple de script PowerShell :

    * Le runbook contient un certain nombre de valeurs de paramètres par défaut. Évaluez toutes les valeurs par défaut et procédez aux mises à jour nécessaires.
    * Si vous avez stocké vos informations d’identification AWS comme ressource d’informations d’identification sous un autre nom que `AWScred`, vous devez mettre à jour la ligne 57 du script en conséquence.  
    * Quand vous utilisez les commandes de l’interface de ligne de commande AWS dans PowerShell, plus particulièrement avec cet exemple de runbook, vous devez spécifier la région AWS. Sinon, les applets de commande échouent. Pour plus d’informations, consultez la rubrique AWS [Specify AWS Region](https://docs.aws.amazon.com/powershell/latest/userguide/pstools-installing-specifying-region.html) (Spécifier la région AWS) dans le document AWS Tools for PowerShell (Outils AWS pour PowerShell).  

8. Pour récupérer une liste de noms d’images à partir de votre abonnement AWS, lancez PowerShell ISE et importez le module PowerShell AWS. Authentifiez-vous auprès de AWS en remplaçant `Get-AutomationPSCredential` dans votre environnement ISE par `AWScred = Get-Credential`. Cette instruction vous invite à entrer vos informations d’identification. Vous pouvez fournir votre ID de clé d’accès comme nom d’utilisateur et votre clé d’accès secrète comme mot de passe. 

      ```powershell
        #Sample to get the AWS VM available images
        #Please provide the path where you have downloaded the AWS PowerShell module
        Import-Module AWSPowerShell
        $AwsRegion = "us-west-2"
        $AwsCred = Get-Credential
        $AwsAccessKeyId = $AwsCred.UserName
        $AwsSecretKey = $AwsCred.GetNetworkCredential().Password
   
        # Set up the environment to access AWS
        Set-AwsCredentials -AccessKey $AwsAccessKeyId -SecretKey $AwsSecretKey -StoreAs AWSProfile
        Set-DefaultAWSRegion -Region $AwsRegion
   
        Get-EC2ImageByName -ProfileName AWSProfile
      ```
        
    Voici les résultats qui sont retournés :<br><br>
   ![Obtenir des images AWS](./media/automation-scenario-aws-deployment/powershell-ise-output.png)<br>  
9. Copiez et collez celui relatif aux noms d’images dans une variable Automation référencée dans le runbook par `$InstanceType`. Sachant que cet exemple s’appuie sur l’abonnement hiérarchisé AWS gratuit, utilisez **t2.micro** dans votre exemple de runbook.  
10. Enregistrez le runbook, puis cliquez sur **Publier** pour publier le runbook, puis sur **Oui** quand vous y êtes invité.

### <a name="test-the-aws-vm-runbook"></a>Tester le runbook de machine virtuelle AWS

1. Vérifiez que le runbook crée une ressource nommée `AWScred`permettant de s’authentifier auprès d’AWS ou mettez à jour le script pour référencer le nom de votre ressource d’informations d’identification.    
2. Vérifiez votre nouvelle runbook et assurez-vous que toutes les valeurs de paramètre ont été mises à jour.
Assurez-vous que le module PowerShell AWS a été importé dans Azure Automation.  
3. Dans Azure Automation, les options **Journaliser les enregistrements détaillés** et éventuellement **Journaliser les informations de progression** sous l'opération de runbook **Journalisation et suivi** ont été définies sur **Activé**.<br><br> ![Journalisation et suivi de runbook](./media/automation-scenario-aws-deployment/runbook-settings-logging-and-tracing.png)  
4. Cliquez sur **Démarrer** pour démarrer le runbook, puis sur **OK** à l’ouverture du volet Démarrer le runbook.
5. Dans le volet Démarrer le runbook, entrez un nom de machine virtuelle. Acceptez les valeurs par défaut des autres paramètres que vous avez préconfigurés dans le script. Cliquez sur **OK** pour démarrer la tâche du runbook.<br><br> ![Démarrer un runbook New-AwsVM](./media/automation-scenario-aws-deployment/runbook-start-job-parameters.png)
6. Un volet s’ouvre pour la tâche du runbook qui vient d’être créée. Fermez ce panneau.
7. Vous pouvez examiner la progression de la tâche et consulter les flux de sortie en sélectionnant la vignette **Tous les journaux d’activité** dans la page de tâche du runbook.<br><br> ![Sortie de flux](./media/automation-scenario-aws-deployment/runbook-job-streams-output.png)
8. Pour vérifier que la machine virtuelle est en cours d’approvisionnement, connectez-vous à la console de gestion AWS si ce n’est pas déjà fait.<br><br> ![Machine virtuelle déployée par la console AWS](./media/automation-scenario-aws-deployment/aws-instances-status.png)

## <a name="next-steps"></a>Étapes suivantes
 
* Pour découvrir les runbooks pris en charge, consultez[Types de runbook Azure Automation](automation-runbook-types.md).
* Pour savoir comment utiliser des runbooks, voir [Gérer les runbooks dans Azure Automation](manage-runbooks.md).
* Pour plus d’informations sur PowerShell, consultez la [documentation PowerShell](/powershell/scripting/overview).
* Pour la prise en charge des scripts, consultez [PowerShell natifs dans Azure Automation](https://azure.microsoft.com/blog/announcing-powershell-script-support-azure-automation-2/).
* Pour obtenir des informations de référence sur les applets de commande PowerShell, consultez [Az.Automation](/powershell/module/az.automation/?view=azps-3.7.0#automation).

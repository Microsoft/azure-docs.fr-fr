---
title: "Prise en main d’Azure Key Vault | Microsoft Docs"
description: "Ce didacticiel va vous aider à démarrer avec Azure Key Vault pour créer un conteneur renforcé dans Azure afin de stocker et gérer des clés de chiffrement et les secrets dans Azure."
services: key-vault
documentationcenter: 
author: cabailey
manager: mbaldwin
tags: azure-resource-manager
ms.assetid: 36721e1d-38b8-4a15-ba6f-14ed5be4de79
ms.service: key-vault
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: hero-article
ms.date: 10/24/2016
ms.author: cabailey
translationtype: Human Translation
ms.sourcegitcommit: dcda8b30adde930ab373a087d6955b900365c4cc
ms.openlocfilehash: 115a718977906b3e647b628017f7ab46a588d7f2


---
# <a name="get-started-with-azure-key-vault"></a>Prise en main du coffre de clés Azure
Azure Key Vault est disponible dans la plupart des régions. Pour plus d’informations, consultez la [page de tarification de Key Vault](https://azure.microsoft.com/pricing/details/key-vault/).

## <a name="introduction"></a>Introduction
Ce didacticiel vous aide à démarrer avec Azure Key Vault pour créer un conteneur renforcé (un coffre) dans Azure afin de stocker et gérer les clés de chiffrement et les secrets dans Azure. Il vous guide tout au long du processus d’utilisation d’Azure PowerShell pour créer un coffre qui contient une clé ou un mot de passe que vous pouvez ensuite utiliser avec une application Azure. Il vous montre également comment une application peut utiliser cette clé ou ce mot de passe.

**Durée estimée :** 20 minutes

> [!NOTE]
> Ce didacticiel n'inclut pas d'instructions sur l'écriture de l'application Azure abordée dans une des étapes, en d'autres termes, comment autoriser une application à utiliser une clé ou un secret dans le coffre de clés.
> 
> Ce didacticiel utilise Azure PowerShell. Pour connaître la marche à suivre avec l’interface de ligne de commande interplateforme, consultez [ce didacticiel équivalent](key-vault-manage-with-cli.md).
> 
> 

Pour plus d’informations générales sur Azure Key Vault, consultez la page [Présentation d’Azure Key Vault](key-vault-whatis.md)

## <a name="prerequisites"></a>Composants requis
Pour suivre ce didacticiel, vous avez besoin des éléments suivants :

* Un abonnement à Microsoft Azure. Si vous n’en possédez pas, vous pouvez vous inscrire pour créer dès aujourd’hui un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/).
* Azure PowerShell, **version 1.1.0 minimale**. Pour installer Azure PowerShell et l’associer à votre abonnement Azure, consultez l’article [Installation et configuration d’Azure PowerShell](/powershell/azureps-cmdlets-docs). Si vous avez déjà installé Azure PowerShell et que vous ne connaissez pas la version que vous utilisez, à partir de la console Azure PowerShell, entrez `(Get-Module azure -ListAvailable).Version`. Si vous utilisez Azure PowerShell version 0.9.1 à 0.9.8, vous pouvez toujours utiliser ce didacticiel en y apportant quelques changements mineurs. Par exemple, vous devez utiliser la commande `Switch-AzureMode AzureResourceManager` ; certaines commandes Azure Key Vault ont également changé. Pour obtenir la liste des applets de commande Azure Key Vault pour les versions 0.9.1 à 0.9.8, consultez la page [Applets de commande Azure Key Vault](https://msdn.microsoft.com/library/azure/dn868052\(v=azure.98\).aspx). 
* Une application configurée pour utiliser la clé ou le mot de passe que vous créez dans ce didacticiel. Un exemple d’application est disponible dans le [Centre de téléchargement Microsoft](http://www.microsoft.com/en-us/download/details.aspx?id=45343). Pour obtenir des instructions, consultez le fichier Lisez-moi fourni.

Ce didacticiel s’adresse aux utilisateurs novices d’Azure PowerShell, mais il repose sur l’hypothèse que vous connaissez les concepts fondamentaux (modules, applets de commande et sessions). Pour plus d’informations, consultez la section [Prise en main de Windows PowerShell](https://technet.microsoft.com/library/hh857337.aspx).

Pour accéder à l'aide détaillée de toute applet de commande présentée dans ce didacticiel, utilisez l'applet de commande **Get-Help** .

    Get-Help <cmdlet-name> -Detailed

Par exemple, pour obtenir de l’aide pour l’applet de commande **Login-AzureRmAccount** , tapez :

    Get-Help Login-AzureRmAccount -Detailed

Vous pouvez également consulter les didacticiels suivants afin de vous familiariser avec Azure Resource Manager dans Azure PowerShell :

* [Installation et configuration d’Azure PowerShell](/powershell/azureps-cmdlets-docs)
* [Utilisation d’Azure PowerShell avec Azure Resource Manager](../powershell-azure-resource-manager.md)

## <a name="a-idconnectaconnect-to-your-subscriptions"></a><a id="connect"></a>Se connecter à vos abonnements
Démarrez une session Azure PowerShell et connectez-vous à votre compte Azure avec la commande suivante :  

    Login-AzureRmAccount 

Notez que si vous utilisez une instance spécifique d’Azure, par exemple, Azure Government, utilisez le paramètre -Environment avec cette commande. Par exemple : `Login-AzureRmAccount –Environment (Get-AzureRmEnvironment –Name AzureUSGovernment)`

Dans la fenêtre contextuelle de votre navigateur, entrez votre nom d’utilisateur et votre mot de passe Azure. Azure PowerShell obtient alors tous les abonnements associés à ce compte et utilise par défaut le premier.

Si vous disposez de plusieurs abonnements et souhaitez spécifier un abonnement spécifique à utiliser pour Azure Key Vault, tapez la commande suivante pour afficher les abonnements de votre compte :

    Get-AzureRmSubscription

Ensuite, pour indiquer l’abonnement, tapez ce qui suit :

    Set-AzureRmContext -SubscriptionId <subscription ID>

Pour plus d’informations sur la configuration d’Azure PowerShell, consultez la page [Installation et configuration d’Azure PowerShell](/powershell/azureps-cmdlets-docs).

## <a name="a-idresourceacreate-a-new-resource-group"></a><a id="resource"></a>Création d’un groupe de ressources
Lorsque vous utilisez Azure Resource Manager, toutes les ressources associées sont créées au sein d’un groupe de ressources. Nous allons créer un groupe de ressources nommé **ContosoResourceGroup** pour ce didacticiel :

    New-AzureRmResourceGroup –Name 'ContosoResourceGroup' –Location 'East Asia'


## <a name="a-idvaultacreate-a-key-vault"></a><a id="vault"></a>Création d’un coffre de clés
Utilisez l’applet de commande [New-AzureRmKeyVault](https://msdn.microsoft.com/library/azure/mt603736\(v=azure.300\).aspx) pour créer un coffre de clés. Cette applet de commande a trois paramètres obligatoires : un **nom de groupe de ressources**, un **nom de coffre de clés** et l’**emplacement géographique**.

Par exemple, si vous utilisez le nom de coffre **ContosoKeyVault**, le nom de groupe de ressources **ContosoResourceGroup** et l’emplacement **Asie du Sud-Est**, tapez :

    New-AzureRmKeyVault -VaultName 'ContosoKeyVault' -ResourceGroupName 'ContosoResourceGroup' -Location 'East Asia'

La sortie de cette applet de commande affiche les propriétés du coffre de clés que vous venez de créer. Les deux propriétés les plus importantes sont :

* **Nom du coffre** : dans l’exemple, il s’agit de **ContosoKeyVault**. Vous allez utiliser ce nom pour les autres applets de commande Key Vault.
* **URI du coffre** : dans l’exemple, il s’agit de https://contosokeyvault.vault.azure.net/. Les applications qui utilisent votre coffre via son API REST doivent utiliser cet URI.

Votre compte Azure est pour l’instant le seul autorisé à effectuer des opérations sur ce coffre de clés.

> [!NOTE]
> Si vous rencontrez le message d’erreur **L’abonnement n’est pas inscrit pour utiliser l’espace de noms « Microsoft.KeyVault »** lorsque vous tentez de créer votre coffre de clés, exécutez la commande `Register-AzureRmResourceProvider -ProviderNamespace "Microsoft.KeyVault"` avant d’exécuter de nouveau la commande New-AzureRmKeyVault. Pour plus d’informations, consultez [Register-AzureRmResourceProvider](https://msdn.microsoft.com/library/azure/mt759831\(v=azure.300\).aspx).
> 
> 

## <a name="a-idaddaadd-a-key-or-secret-to-the-key-vault"></a><a id="add"></a>Ajout d’une clé ou d’un secret au coffre de clés
Si vous souhaitez qu’Azure Key Vault crée une clé protégée par un logiciel pour vous, utilisez l’applet de commande [Add-AzureKeyVaultKey](https://msdn.microsoft.com/library/azure/dn868048\(v=azure.300\).aspx) et tapez ce qui suit :

    $key = Add-AzureKeyVaultKey -VaultName 'ContosoKeyVault' -Name 'ContosoFirstKey' -Destination 'Software'

Toutefois, si vous disposez d’une clé existante protégée par un logiciel dans un fichier PFX enregistré sur votre lecteur C:\ et nommé softkey.pfx que vous souhaitez télécharger vers Azure Key Vault, tapez ce qui suit pour définir la variable **securepfxpwd** pour le mot de passe **123** du fichier PFX :

    $securepfxpwd = ConvertTo-SecureString –String '123' –AsPlainText –Force

Tapez ensuite la commande suivante pour importer la clé à partir du fichier PFX, ce qui protège la clé par logiciel dans le service Key Vault :

    $key = Add-AzureKeyVaultKey -VaultName 'ContosoKeyVault' -Name 'ContosoFirstKey' -KeyFilePath 'c:\softkey.pfx' -KeyFilePassword $securepfxpwd


Vous pouvez maintenant référencer cette clé que vous avez créée ou téléchargée dans Azure Key Vault à l’aide de son URI. Utilisez **https://ContosoKeyVault.vault.azure.net/keys/ContosoFirstKey** pour toujours obtenir la version actuelle, et **https://ContosoKeyVault.vault.azure.net/keys/ContosoFirstKey/cgacf4f763ar42ffb0a1gca546aygd87** pour obtenir cette version spécifique.  

Pour afficher l’URI de cette clé, tapez :

    $key.kid

Pour ajouter un secret dans le coffre, c'est-à-dire un mot de passe nommé SQLPassword avec la valeur Pa$$w0rd dans Azure Key Vault, commencez par convertir la valeur Pa$$w0rd en chaîne sécurisée en tapant ce qui suit :

    $secretvalue = ConvertTo-SecureString 'Pa$$w0rd' -AsPlainText -Force

Tapez ensuite la commande suivante :

    $secret = Set-AzureKeyVaultSecret -VaultName 'ContosoKeyVault' -Name 'SQLPassword' -SecretValue $secretvalue

Vous pouvez maintenant référencer ce mot de passe que vous avez ajouté dans Azure Key Vault à l’aide de son URI. Utilisez **https://ContosoVault.vault.azure.net/secrets/SQLPassword** pour toujours obtenir la version actuelle, et **https://ContosoVault.vault.azure.net/secrets/SQLPassword/90018dbb96a84117a0d2847ef8e7189d** pour obtenir cette version spécifique.

Pour afficher l’URI de ce secret, tapez :

    $secret.Id

Examinons la clé ou le mot de passe que vous venez de créer :

* Pour afficher votre clé, tapez : `Get-AzureKeyVaultKey –VaultName 'ContosoKeyVault'`
* Pour afficher votre secret, tapez : `Get-AzureKeyVaultSecret –VaultName 'ContosoKeyVault'`

À présent, votre coffre de clés et la clé/le secret sont prêts à être utilisés par les applications qui doivent recevoir les autorisations adéquates.  

## <a name="a-idregisteraregister-an-application-with-azure-active-directory"></a><a id="register"></a>Inscription d’une application auprès d’Azure Active Directory
Cette étape est généralement effectuée par un développeur et sur un ordinateur distinct. Bien que non spécifique d'Azure Key Vault, elle est incluse ici par souci d'exhaustivité.

> [!IMPORTANT]
> Pour suivre le didacticiel, le compte, le coffre et l’application que vous inscrivez dans cette étape doivent tous se trouver dans le même répertoire Azure.
> 
> 

Les applications qui utilisent un coffre de clés doivent s’authentifier à l’aide d’un jeton à partir d’Azure Active Directory. Pour ce faire, le propriétaire de l’application doit d’abord inscrire l’application dans Azure Active Directory. À la fin de l’inscription, le propriétaire de l’application obtient les valeurs suivantes :

* Un **ID d’application** (également appelé ID de client) et une **clé d’authentification** (également appelée secret partagé). L’application doit présenter ces deux valeurs à Azure Active Directory afin d’obtenir un jeton. La manière dont l’application est configurée pour cela dépend de l’application en question. Pour l’exemple d’application de coffre de clés, le propriétaire de l’application définit ces valeurs dans le fichier app.config.

Pour inscrire votre application auprès d’Azure Active Directory :

1. Connectez-vous à la version classique du portail Azure.
2. Dans le volet gauche, cliquez sur **Active Directory**, puis sélectionnez le répertoire dans lequel vous allez inscrire votre application. <br> <br> **Remarque :** Vous devez sélectionner le répertoire qui contient l’abonnement Azure avec lequel vous avez créé votre coffre de clés. Si vous ne savez pas de quel répertoire il s’agit, cliquez sur **Paramètres**, identifiez l’abonnement avec lequel vous avez créé votre coffre de clés et notez le nom du répertoire affiché dans la dernière colonne.
3. Cliquez sur **APPLICATIONS**. Si aucune application n’a été ajoutée à votre répertoire, cette page affiche uniquement le lien **Ajouter une application**. Cliquez sur le lien. Vous pouvez également cliquer sur **AJOUTER** dans la barre de commandes.
4. Dans l’Assistant **AJOUTER UNE APPLICATION**, dans la page **Que voulez-vous faire ?**, cliquez sur **Ajouter une application développée par mon organisation**.
5. Dans la page **Parlez-nous de votre application**, spécifiez un nom pour votre application et sélectionnez **APPLICATION WEB ET/OU API WEB** (par défaut). Cliquez sur l'icône **Suivant** .
6. Dans la page **Propriétés de l’application**, spécifiez l’**URL DE CONNEXION** et l’**URI ID D’APPLICATION** pour votre application web. Si votre application n’a pas ces valeurs, vous pouvez les créer pour cette étape (par exemple, vous pouvez spécifier http://test1.contoso.com pour les deux zones). Peu importe si ces sites existent. L’important est que l’URI d’ID d’application est différent pour chaque application de votre répertoire. Le répertoire utilise cette chaîne pour identifier votre application.
7. Cliquez sur l'icône **Terminé** pour enregistrer vos modifications dans l'Assistant.
8. Dans la page **Démarrage rapide**, cliquez sur **CONFIGURER**.
9. Accédez à la section **clés** , sélectionnez la durée, puis cliquez sur **ENREGISTRER**. La page est actualisée et affiche à présent une valeur de clé. Vous devez configurer votre application avec cette valeur de clé et la valeur d’ **ID CLIENT** . (Les instructions relatives à cette configuration sont propres à l’application.)
10. Copiez la valeur d’ID client à partir de cette page. Vous l’utiliserez à l’étape suivante pour définir des autorisations sur votre coffre.

## <a name="a-idauthorizeaauthorize-the-application-to-use-the-key-or-secret"></a><a id="authorize"></a>Autorisation de l’application à utiliser la clé ou le secret
Pour autoriser l’application à accéder à la clé ou au secret dans le coffre, utilisez l’applet de commande  [Set-AzureRmKeyVaultAccessPolicy](https://msdn.microsoft.com/library/azure/mt603625\(v=azure.300\).aspx).

Par exemple, si le nom de votre coffre est **ContosoKeyVault** , que l'application que vous souhaitez autoriser a l'ID client 8f8c4bbd-485b-45fd-98f7-ec6300b7b4ed et que vous souhaitez autoriser l'application à déchiffrer et à signer avec des clés dans le coffre, exécutez la commande suivante :

    Set-AzureRmKeyVaultAccessPolicy -VaultName 'ContosoKeyVault' -ServicePrincipalName 8f8c4bbd-485b-45fd-98f7-ec6300b7b4ed -PermissionsToKeys decrypt,sign

Si vous souhaitez autoriser cette même application à lire les éléments secrets de votre coffre, exécutez la commande suivante :

    Set-AzureRmKeyVaultAccessPolicy -VaultName 'ContosoKeyVault' -ServicePrincipalName 8f8c4bbd-485b-45fd-98f7-ec6300b7b4ed -PermissionsToSecrets Get

## <a name="a-idhsmaif-you-want-to-use-a-hardware-security-module-hsm"></a><a id="HSM"></a>Si vous souhaitez utiliser un module de sécurité matériel (HSM)
Pour une meilleure garantie, vous pouvez importer ou générer des clés dans des modules de sécurité matériels (HSM) qui ne franchissent jamais les limites HSM. Les modules HSM bénéficient d’une validation FIPS 140-2 de niveau 2. Si cette exigence ne s’applique pas à vous, ignorez cette section et accédez à [Supprimer le coffre de clés et les clés et secrets associés](#delete).

Pour créer les clés protégées par HSM, vous devez utiliser le [niveau de service Azure Key Vault Premium qui prend en charge les clés protégées par HSM](https://azure.microsoft.com/pricing/free-trial/). En outre, notez que cette fonctionnalité n’est pas disponible pour Azure en Chine.

Lorsque vous créez le coffre de clés, ajoutez le paramètre **-SKU** :

    New-AzureRmKeyVault -VaultName 'ContosoKeyVaultHSM' -ResourceGroupName 'ContosoResourceGroup' -Location 'East Asia' -SKU 'Premium'



Vous pouvez ajouter des clés protégées par logiciel (comme indiqué plus haut) et des clés protégées par HSM dans ce coffre de clés. Pour créer une clé protégée par HSM, définissez le paramètre **-Destination** sur « HSM » :

    $key = Add-AzureKeyVaultKey -VaultName 'ContosoKeyVaultHSM' -Name 'ContosoFirstHSMKey' -Destination 'HSM'

Vous pouvez utiliser la commande suivante pour importer une clé à partir d’un fichier PFX sur votre ordinateur. Cette commande importe la clé dans les modules de sécurité matériels dans le service Key Vault :

    $key = Add-AzureKeyVaultKey -VaultName 'ContosoKeyVaultHSM' -Name 'ContosoFirstHSMKey' -KeyFilePath 'c:\softkey.pfx' -KeyFilePassword $securepfxpwd -Destination 'HSM'


La commande suivante importe un package BYOK (Bring Your Own Key, Apporter votre propre clé). Ce scénario vous permet de générer votre clé dans votre module de sécurité matériel local et de la transférer vers les modules de sécurité matériels du service Key Vault, sans que la clé quitte la limite HSM :

    $key = Add-AzureKeyVaultKey -VaultName 'ContosoKeyVaultHSM' -Name 'ContosoFirstHSMKey' -KeyFilePath 'c:\ITByok.byok' -Destination 'HSM'

Pour plus d'instructions sur la génération de ce package BYOK, consultez [Génération et transfert de clés protégées par HSM pour Azure Key Vault](key-vault-hsm-protected-keys.md).

## <a name="a-iddeleteadelete-the-key-vault-and-associated-keys-and-secrets"></a><a id="delete"></a>Suppression du coffre de clés et des clés/secrets associés
Si vous n’avez plus besoin du coffre de clés ni de la clé ou du secret qu’il contient, vous pouvez supprimer le coffre de clés à l’aide de l’applet de commande [Remove-AzureRmKeyVault](https://msdn.microsoft.com/library/azure/mt619485\(v=azure.300\).aspx) :

    Remove-AzureRmKeyVault -VaultName 'ContosoKeyVault'

Vous pouvez également supprimer un groupe de ressources Azure, qui inclut le coffre de clés et les autres ressources incluses dans ce groupe :

    Remove-AzureRmResourceGroup -ResourceGroupName 'ContosoResourceGroup'


## <a name="a-idotheraother-azure-powershell-cmdlets"></a><a id="other"></a>Autres applets de commande Azure PowerShell
Autres commandes pouvant être utiles pour la gestion du coffre de clés Azure.

* `$Keys = Get-AzureKeyVaultKey -VaultName 'ContosoKeyVault'` : cette commande permet d’obtenir un affichage sous forme de tableau de l’ensemble des clés et propriétés sélectionnées.
* `$Keys[0]` : cette commande affiche la liste complète des propriétés pour la clé spécifiée
* `Get-AzureKeyVaultSecret` : cette commande permet d’obtenir un affichage sous forme de tableau de l’ensemble des secrets et des propriétés sélectionnés.
* `Remove-AzureKeyVaultKey -VaultName 'ContosoKeyVault' -Name 'ContosoFirstKey'` : exemple montrant comment supprimer une clé spécifique.
* `Remove-AzureKeyVaultSecret -VaultName 'ContosoKeyVault' -Name 'SQLPassword'` : exemple montrant comment supprimer un secret spécifique.

## <a name="a-idnextanext-steps"></a><a id="next"></a>Étapes suivantes
Pour assurer le suivi d'un didacticiel sur l'utilisation d'Azure Key Vault dans une application web, consultez la page [Utilisation d'Azure Key Vault à partir d'une application web](key-vault-use-from-web-application.md).

Pour voir comment votre coffre de clés est utilisé, consultez l’article [Journalisation d’Azure Key Vault](key-vault-logging.md).

Pour obtenir la liste des dernières applets de commande Azure PowerShell pour le coffre de clés Azure, consultez la page relative aux [applets de commande Azure Key Vault](https://msdn.microsoft.com/library/azure/dn868052\(v=azure.300\).aspx). 

Pour les références de programmation, consultez le [guide du développeur de coffre de clés Azure](key-vault-developers-guide.md).




<!--HONumber=Dec16_HO2-->



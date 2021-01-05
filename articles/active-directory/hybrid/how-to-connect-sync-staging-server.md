---
title: 'Synchronisation d’Azure AD Connect : Tâches opérationnelles et examen | Microsoft Docs'
description: Cette rubrique décrit les tâches opérationnelles de la synchronisation d’Azure AD Connect et explique comment se préparer pour utiliser ce composant.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: b29c1790-37a3-470f-ab69-3cee824d220d
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 02/27/2018
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 48584fa4042cf53fa1084e519dca0e64f530ca59
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90090123"
---
# <a name="azure-ad-connect-staging-server-and-disaster-recovery"></a>Azure AD Connect : Serveur de préproduction et reprise d’activité après sinistre
Avec un serveur en mode intermédiaire, vous pouvez apporter des modifications à la configuration et visualiser les modifications avant de rendre le serveur actif. Il permet également d’exécuter une importation et la synchronisation complètes afin de vérifier que toutes les modifications sont attendues avant de les appliquer dans un environnement de production.

## <a name="staging-mode"></a>Mode intermédiaire
Le mode intermédiaire peut être utilisé dans le cadre de plusieurs scénarios, notamment :

* Haute disponibilité :
* Tester et déployer de nouvelles modifications de configuration.
* Introduire un nouveau serveur et retirer l’ancien.

Lors de l’installation, vous pouvez sélectionner le serveur en **mode intermédiaire**. Cette action rend le serveur actif pour l’importation et la synchronisation, mais n’exécute aucune exportation. Un serveur en mode intermédiaire n’exécute pas la synchronisation de mot de passe et l’écriture différée de mot de passe même si vous avez sélectionné ces fonctions au cours de l’installation. Lorsque vous désactivez le mode intermédiaire, le serveur lance l’exportation et active la synchronisation de mot de passe et l’écriture différée de mot de passe.

> [!NOTE]
> Supposons que vous avez activé la fonctionnalité Synchronisation de hachage de mot de passe dans Azure AD Connect. Lorsque vous activez le mode de préproduction, le serveur interrompt la synchroniser des modifications du mot de passe à partir de l'instance AD sur site. Lorsque vous désactivez le mode de préproduction, le serveur reprend la synchronisation des modifications du mot de passe à partir du point où il s'est arrêté. Si le serveur reste en mode de préproduction pendant une période prolongée, il risque de mettre un certain temps pour synchroniser toutes les modifications de mot de passe survenues dans l'intervalle.
>
>

Vous pouvez toujours forcer une exportation en utilisant le gestionnaire de services de synchronisation.

Un serveur en mode intermédiaire continue de recevoir des modifications d’Active Directory et d’Azure AD et peut très rapidement reprendre les responsabilités d’un autre serveur en cas de défaillance. Si vous apportez des modifications de configuration à votre serveur principal, la responsabilité d’apporter les mêmes modifications au serveur en mode intermédiaire vous incombe.

Pour ceux qui connaissant les technologies de synchronisation plus anciennes, le mode intermédiaire est différent, dans la mesure où le serveur a sa propre base de données SQL. Cette architecture permet au serveur en mode intermédiaire d’être situé dans un autre centre de données.

### <a name="verify-the-configuration-of-a-server"></a>Vérifiez la configuration d’un serveur
Pour appliquer cette méthode, procédez comme suit :

1. [Préparation](#prepare)
2. [Configuration](#configuration)
3. [Importer et synchroniser](#import-and-synchronize)
4. [Vérifier](#verify)
5. [Changer de serveur actif](#switch-active-server)

#### <a name="prepare"></a>Préparation
1. Installez Azure AD Connect, sélectionnez **mode intermédiaire**, puis désélectionnez **Démarrer la synchronisation** sur la dernière page de l’Assistant Installation. Ce mode vous permet d’exécuter manuellement le moteur de synchronisation.
   ![Capture d’écran qui montre la page Prêt à configurer dans la boîte de dialogue Azure AD Connect.](./media/how-to-connect-sync-staging-server/readytoconfigure.png)
2. Déconnectez-vous puis connectez-vous de nouveau et, dans le menu Démarrer, sélectionnez **Service de synchronisation**.

#### <a name="configuration"></a>Configuration
Si vous avez apporté des modifications personnalisées au serveur principal et que vous souhaitez comparer la configuration avec le serveur intermédiaire, utilisez la [Documentation de configuration d’Azure AD Connect](https://github.com/Microsoft/AADConnectConfigDocumenter).

#### <a name="import-and-synchronize"></a>Importer et synchroniser
1. Sélectionnez **Connecteurs**, puis sélectionnez le premier connecteur de type **Services de domaine Active Directory**. Cliquez sur **Exécuter**, sélectionnez **Importation intégrale**, puis **OK**. Répétez cette procédure pour tous les connecteurs de ce type.
2. Sélectionnez le connecteur de type **Azure Active Directory (Microsoft)** . Cliquez sur **Exécuter**, sélectionnez **Importation intégrale**, puis **OK**.
3. Vérifiez que l’onglet Connecteurs est toujours sélectionné. Pour chaque connecteur de type **Services de domaine Active Directory**, cliquez sur **Exécuter**, sélectionnez **Synchronisation Delta**, puis **OK**.
4. Sélectionnez le connecteur de type **Azure Active Directory (Microsoft)** . Cliquez sur **Exécuter**, sélectionnez **Synchronisation Delta**, puis **OK**.

Vous avez maintenant effectué une exportation intermédiaire vers Azure AD et Active Directory local (si vous utilisez un déploiement Exchange hybride). Les prochaines étapes vous permettront d’inspecter les changements avant de commencer effectivement l’exportation vers les répertoires.

#### <a name="verify"></a>Vérifier
1. Démarrez une invite de commande et accédez à `%ProgramFiles%\Microsoft Azure AD Sync\bin`
2. Exécutez : `csexport "Name of Connector" %temp%\export.xml /f:x` le nom du connecteur se trouve dans le service de synchronisation. Le nom est similaire à « contoso.com – Azure AD » pour Azure AD.
3. Exécutez : `CSExportAnalyzer %temp%\export.xml > %temp%\export.csv` Vous disposez d’un fichier dans %temp% nommé export.csv qui peut être examiné dans Microsoft Excel. Ce fichier contient toutes les modifications sur le point d’être exportées.
4. Apportez les modifications nécessaires aux données ou à la configuration et réexécutez ces opérations (importer, synchroniser et vérifier) jusqu’à ce que les modifications sur le point d’être exportées soient attendues.

**Comprendre le fichier export.csv** La majeure partie du fichier est explicite. Certaines abréviations permettant de comprendre le contenu :
* OMODT – Type de modification d’objet. Indique si l’opération au niveau de l’objet est un ajout, une mise à jour ou une suppression.
* AMODT – Type de modification d’attribut. Indique si l’opération au niveau de l’attribut est un ajout, une mise à jour ou une suppression.

**Récupérer les identificateurs courants** Le fichier export.csv contient toutes les modifications qui sont sur le point d’être exportées. Chaque ligne correspond à une modification d’un objet dans l’espace connecteur, objet qui est identifié par l’attribut de nom unique (DN). L’attribut DN est un identificateur unique assigné à un objet dans l’espace connecteur. Quand le fichier export.csv à analyser contient de nombreuses lignes/modifications, il peut s’avérer difficile de déterminer quels objets sont concernés par les modifications à l’aide uniquement de l’attribut DN. Pour simplifier le processus d’analyse des modifications, utilisez le script PowerShell csanalyzer.ps1. Le script récupère les identificateurs courants (par exemple, displayName, userPrincipalName) des objets. Pour utiliser le script :
1. Copiez le script PowerShell à partir de la section [CSAnalyzer](#appendix-csanalyzer) dans un fichier nommé `csanalyzer.ps1`.
2. Ouvrez une fenêtre PowerShell et accédez au dossier où vous avez créé le script PowerShell.
3. Exécutez : `.\csanalyzer.ps1 -xmltoimport %temp%\export.xml`.
4. Vous disposez maintenant d’un fichier nommé **processedusers1.csv** qui peut être examiné dans Microsoft Excel. Notez que le fichier fournit un mappage de l’attribut DN aux identificateurs courants (par exemple, displayName et userPrincipalName). Il n’inclut pas les modifications d’attribut réelles qui sont sur le point d’être exportées.

#### <a name="switch-active-server"></a>Changer de serveur actif
1. Sur le serveur actif, éteignez le serveur (DirSync/FIM/Azure AD Sync) pour qu’il ne soit pas exporté vers Azure AD ou définissez-le en mode intermédiaire (Azure AD Connect).
2. Exécutez l’Assistant Installation sur le serveur en **mode intermédiaire**, puis désactivez le **mode intermédiaire**.
   ![ReadyToConfigure](./media/how-to-connect-sync-staging-server/additionaltasks.png)

## <a name="disaster-recovery"></a>Récupération d'urgence
Une partie de la conception de l’implémentation consiste à planifier les procédures à suivre si un sinistre occasionne la perte du serveur de synchronisation. Il existe différents modèles et le choix de celui que vous devez utiliser dépend de plusieurs facteurs, notamment :

* Dans quelle mesure pouvez-vous tolérer de ne pas pouvoir apporter des modifications aux objets dans Azure AD pendant les temps d’indisponibilité ?
* Si vous utilisez la synchronisation de mot de passe, les utilisateurs acceptent-ils de devoir utiliser l’ancien mot de passe dans Azure AD dans le cas où il serait modifié en local ?
* Avez-vous une dépendance par rapport aux opérations en temps réel, notamment l’écriture différée de mot de passe ?

Selon les réponses à ces questions et la stratégie de votre organisation, une des stratégies suivantes peut être mise en œuvre :

* Régénérer si nécessaire.
* Disposer d'un serveur de secours en attente, appelé **mode intermédiaire**.
* Utiliser les machines virtuelles.

Si vous n’utilisez pas la base de données SQL Express intégrée, vous devez vous reporter à la section [Haute disponibilité SQL](#sql-high-availability) .

### <a name="rebuild-when-needed"></a>Régénérer lorsque nécessaire
Une stratégie viable consiste à planifier une régénération du serveur si nécessaire. Généralement, l’installation du moteur de synchronisation et l’exécution de l’importation et de la synchronisation initiales peuvent être effectuées en quelques heures. Si aucun serveur n’est libre, il est possible d’utiliser provisoirement un contrôleur de domaine pour héberger le moteur de synchronisation.

Le serveur de moteur de synchronisation ne stocke aucun état relatif aux objets de sorte que la base de données peut être recréée à partir des données présentes dans Active Directory et Azure AD. L’attribut **sourceAnchor** est utilisé pour associer les objets à partir du site et du cloud. Si vous régénérez le serveur avec les objets existants en local et sur le cloud, le moteur de synchronisation les remet en correspondance de nouveau au cours de la réinstallation. Vous devez documenter et enregistrer les modifications de configuration apportées au serveur, notamment aux règles de filtrage et de synchronisation. Ces configurations personnalisées doivent être réappliquées avant de commencer la synchronisation.

### <a name="have-a-spare-standby-server---staging-mode"></a>Disposer d’un serveur de secours en attente, connu sous le nom de mode intermédiaire.
Si vous disposez d’un environnement plus complexe, il est recommandé d’avoir un ou plusieurs serveurs de secours. Lors de l’installation, vous pouvez activer un serveur en **mode intermédiaire**.

Pour en savoir plus, voir [Mode intermédiaire](#staging-mode).

### <a name="use-virtual-machines"></a>Utiliser les machines virtuelles
Une méthode courante et prise en charge consiste à exécuter le moteur de synchronisation sur une machine virtuelle. Si l’hôte rencontre un problème, l’image contenant le serveur de moteur de synchronisation peut être migrée vers un autre serveur.

### <a name="sql-high-availability"></a>Haute disponibilité SQL
Si vous n’utilisez pas SQL Server Express livré avec Azure AD Connect, la haute disponibilité pour SQL Server doit alors être prise en compte. Les solutions de haute disponibilité prises en charge incluent la mise en clusters SQL et AOA (Groupes de disponibilité AlwaysOn). Les solutions non prises en charge incluent la mise en miroir.

La prise en charge de SQL AOA a été ajoutée à Azure AD Connect version 1.1.524.0. Vous devez activer SQL AOA avant d’installer Azure AD Connect. Pendant l’installation, Azure AD Connect détecte si l’instance SQL spécifiée est activée ou non pour SQL AOA. Si SQL AOA est activé, Azure AD Connect détermine si SQL AOA est configuré pour utiliser une réplication synchrone ou asynchrone. Lorsque vous configurez l’écouteur de groupe de disponibilité, il est recommandé de définir la propriété RegisterAllProvidersIP sur 0. En effet, Azure AD Connect utilise actuellement SQL Native Client pour se connecter à SQL, et SQL Native Client ne prend pas en charge l’utilisation de la propriété MultiSubNetFailover.

## <a name="appendix-csanalyzer"></a>Annexe CSAnalyzer
Consultez la section [vérifier](#verify) pour découvrir comment utiliser ce script.

```
Param(
    [Parameter(Mandatory=$true, HelpMessage="Must be a file generated using csexport 'Name of Connector' export.xml /f:x)")]
    [string]$xmltoimport="%temp%\exportedStage1a.xml",
    [Parameter(Mandatory=$false, HelpMessage="Maximum number of users per output file")][int]$batchsize=1000,
    [Parameter(Mandatory=$false, HelpMessage="Show console output")][bool]$showOutput=$false
)

#LINQ isn't loaded automatically, so force it
[Reflection.Assembly]::Load("System.Xml.Linq, Version=3.5.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089") | Out-Null

[int]$count=1
[int]$outputfilecount=1
[array]$objOutputUsers=@()

#XML must be generated using "csexport "Name of Connector" export.xml /f:x"
write-host "Importing XML" -ForegroundColor Yellow

#XmlReader.Create won't properly resolve the file location,
#so expand and then resolve it
$resolvedXMLtoimport=Resolve-Path -Path ([Environment]::ExpandEnvironmentVariables($xmltoimport))

#use an XmlReader to deal with even large files
$result=$reader = [System.Xml.XmlReader]::Create($resolvedXMLtoimport) 
$result=$reader.ReadToDescendant('cs-object')
do 
{
    #create the object placeholder
    #adding them up here means we can enforce consistency
    $objOutputUser=New-Object psobject
    Add-Member -InputObject $objOutputUser -MemberType NoteProperty -Name ID -Value ""
    Add-Member -InputObject $objOutputUser -MemberType NoteProperty -Name Type -Value ""
    Add-Member -inputobject $objOutputUser -MemberType NoteProperty -Name DN -Value ""
    Add-Member -inputobject $objOutputUser -MemberType NoteProperty -Name operation -Value ""
    Add-Member -inputobject $objOutputUser -MemberType NoteProperty -Name UPN -Value ""
    Add-Member -inputobject $objOutputUser -MemberType NoteProperty -Name displayName -Value ""
    Add-Member -inputobject $objOutputUser -MemberType NoteProperty -Name sourceAnchor -Value ""
    Add-Member -inputobject $objOutputUser -MemberType NoteProperty -Name alias -Value ""
    Add-Member -inputobject $objOutputUser -MemberType NoteProperty -Name primarySMTP -Value ""
    Add-Member -inputobject $objOutputUser -MemberType NoteProperty -Name onPremisesSamAccountName -Value ""
    Add-Member -inputobject $objOutputUser -MemberType NoteProperty -Name mail -Value ""

    $user = [System.Xml.Linq.XElement]::ReadFrom($reader)
    if ($showOutput) {Write-Host Found an exported object... -ForegroundColor Green}

    #object id
    $outID=$user.Attribute('id').Value
    if ($showOutput) {Write-Host ID: $outID}
    $objOutputUser.ID=$outID

    #object type
    $outType=$user.Attribute('object-type').Value
    if ($showOutput) {Write-Host Type: $outType}
    $objOutputUser.Type=$outType

    #dn
    $outDN= $user.Element('unapplied-export').Element('delta').Attribute('dn').Value
    if ($showOutput) {Write-Host DN: $outDN}
    $objOutputUser.DN=$outDN

    #operation
    $outOperation= $user.Element('unapplied-export').Element('delta').Attribute('operation').Value
    if ($showOutput) {Write-Host Operation: $outOperation}
    $objOutputUser.operation=$outOperation

    #now that we have the basics, go get the details

    foreach ($attr in $user.Element('unapplied-export-hologram').Element('entry').Elements("attr"))
    {
        $attrvalue=$attr.Attribute('name').Value
        $internalvalue= $attr.Element('value').Value

        switch ($attrvalue)
        {
            "userPrincipalName"
            {
                if ($showOutput) {Write-Host UPN: $internalvalue}
                $objOutputUser.UPN=$internalvalue
            }
            "displayName"
            {
                if ($showOutput) {Write-Host displayName: $internalvalue}
                $objOutputUser.displayName=$internalvalue
            }
            "sourceAnchor"
            {
                if ($showOutput) {Write-Host sourceAnchor: $internalvalue}
                $objOutputUser.sourceAnchor=$internalvalue
            }
            "alias"
            {
                if ($showOutput) {Write-Host alias: $internalvalue}
                $objOutputUser.alias=$internalvalue
            }
            "proxyAddresses"
            {
                if ($showOutput) {Write-Host primarySMTP: ($internalvalue -replace "SMTP:","")}
                $objOutputUser.primarySMTP=$internalvalue -replace "SMTP:",""
            }
        }
    }

    $objOutputUsers += $objOutputUser

    Write-Progress -activity "Processing ${xmltoimport} in batches of ${batchsize}" -status "Batch ${outputfilecount}: " -percentComplete (($objOutputUsers.Count / $batchsize) * 100)

    #every so often, dump the processed users in case we blow up somewhere
    if ($count % $batchsize -eq 0)
    {
        Write-Host Hit the maximum users processed without completion... -ForegroundColor Yellow

        #export the collection of users as a CSV
        Write-Host Writing processedusers${outputfilecount}.csv -ForegroundColor Yellow
        $objOutputUsers | Export-Csv -path processedusers${outputfilecount}.csv -NoTypeInformation

        #increment the output file counter
        $outputfilecount+=1

        #reset the collection and the user counter
        $objOutputUsers = $null
        $count=0
    }

    $count+=1

    #need to bail out of the loop if no more users to process
    if ($reader.NodeType -eq [System.Xml.XmlNodeType]::EndElement)
    {
        break
    }

} while ($reader.Read)

#need to write out any users that didn't get picked up in a batch of 1000
#export the collection of users as CSV
Write-Host Writing processedusers${outputfilecount}.csv -ForegroundColor Yellow
$objOutputUsers | Export-Csv -path processedusers${outputfilecount}.csv -NoTypeInformation
```

## <a name="next-steps"></a>Étapes suivantes
**Rubriques de présentation**  

* [Synchronisation Azure AD Connect : Comprendre et personnaliser la synchronisation](how-to-connect-sync-whatis.md)  
* [Intégration des identités locales dans Azure Active Directory](whatis-hybrid-identity.md)  

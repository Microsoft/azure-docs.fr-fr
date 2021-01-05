---
title: Créer des certificats à l’aide de l’outil Microsoft Azure Stack Hub Readiness Checker | Microsoft Docs
description: Explique comment créer des demandes de certificat, puis récupérer et installer les certificats sur votre appareil Azure Stack Edge Pro avec GPU au moyen de l’outil Azure Stack Hub Readiness Checker.
services: Azure Stack Edge Pro
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 11/24/2020
ms.author: alkohli
ms.openlocfilehash: ab9559e1e8265b3adf08b36d1a8097a00297c61a
ms.sourcegitcommit: 4c89d9ea4b834d1963c4818a965eaaaa288194eb
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96606988"
---
# <a name="create-certificates-for-your-azure-stack-edge-pro-using-azure-stack-hub-readiness-checker-tool"></a>Créer des certificats pour votre instance Azure Stack Edge Pro avec l’outil Azure Stack Hub Readiness Checker 

<!--[!INCLUDE [applies-to-skus](../../includes/azure-stack-edge-applies-to-all-sku.md)]-->

Cet article explique comment créer des certificats pour votre instance Azure Stack Edge Pro avec l’outil Azure Stack Hub Readiness Checker. 

## <a name="using-azure-stack-hub-readiness-checker-tool"></a>Utilisation de l’outil Azure Stack Hub Readiness Checker

Utilisez l’outil Azure Stack Hub Readiness Checker pour créer des demandes de signature de certificat (CSR) destinées au déploiement d’un appareil Azure Stack Edge Pro. Vous pouvez créer ces demandes après avoir passé commande de l’appareil Azure Stack Edge Pro, en attendant son arrivée.

> [!NOTE]
> Utilisez cet outil uniquement à des fins de test ou de développement, mais pas pour des appareils de production. 

L’outil Azure Stack Hub Readiness Checker (AzsReadinessChecker) permet de demander les certificats suivants :

- Certificat Azure Resource Manager
- Certificat de l’interface utilisateur locale
- Certificat de nœud
- Certificat d’objet blob
- Certificat VPN


## <a name="prerequisites"></a>Prérequis

Pour créer des demandes de signature de certificat destinées au déploiement d’un appareil Azure Stack Edge Pro, assurez-vous que : 

- Vous disposez d’un client exécutant Windows 10 ou Windows Server 2016, ou une version ultérieure. 
- Vous avez téléchargé l'outil Microsoft Azure Stack Hub Readiness Checker sur ce système [depuis PowerShell Gallery](https://aka.ms/AzsReadinessChecker).
- Vous disposez des informations suivantes pour les certificats :
  - Nom du périphérique
  - Numéro de série du nœud
  - Nom de domaine pleinement qualifié externe (FQDN)

## <a name="generate-certificate-signing-requests"></a>Générer les demandes de signature de certificat

Suivez ces étapes pour préparer les certificats de l’appareil Azure Stack Edge Pro :

1. Exécutez PowerShell en tant qu’administrateur (5.1 ou version ultérieure).
2. Installez l’outil Azure Stack Hub Readiness Checker. À l’invite PowerShell, tapez : 

    ```azurepowershell
    Install-Module -Name Microsoft.AzureStack.ReadinessChecker
    ```

    Pour obtenir la version installée, saisissez :  

    ```azurepowershell
    Get-InstalledModule -Name Microsoft.AzureStack.ReadinessChecker  | ft Name, Version 
    ```

3. Créez un répertoire pour tous les certificats si vous n’en avez pas déjà un. Tapez : 
    
    ```azurepowershell
    New-Item "C:\certrequest" -ItemType Directory
    ``` 
    
4. Pour créer une demande de certificat, fournissez les informations suivantes. Si vous générez un certificat VPN, certaines de ces entrées ne s’appliquent pas.
    
    |Entrée |Description  |
    |---------|---------|
    |`OutputRequestPath`|Chemin du fichier sur votre client local, où vous souhaitez que les demandes de certificat soient créées.        |
    |`DeviceName`|Nom de votre appareil dans la page **Appareils** de l’interface utilisateur web locale de votre appareil. <br> Ce champ n’est pas obligatoire pour un certificat VPN.         |
    |`NodeSerialNumber`|Numéro de série du nœud de l’appareil dans la page **Réseau** de l’interface utilisateur web locale de votre appareil. <br> Ce champ n’est pas obligatoire pour un certificat VPN.       |
    |`ExternalFQDN`|Valeur DNSDomain dans la page **Appareils** de l’interface utilisateur web locale de votre appareil.         |
    |`RequestType`|Le type de demande peut être pour `MultipleCSR`, soit différents certificats destinés à divers points de terminaison, ou `SingleCSR`, soit un seul certificat pour tous les points de terminaison. <br> Ce champ n’est pas obligatoire pour un certificat VPN.     |

    Pour tous les certificats, à l’exception du certificat VPN, tapez : 
    
    ```azurepowershell
    $edgeCSRparams = @{
        CertificateType = 'AzureStackEdgeDEVICE'
        DeviceName = 'myTEA1'
        NodeSerialNumber = 'VM1500-00025'
        externalFQDN = 'azurestackedge.contoso.com'
        requestType = 'MultipleCSR'
        OutputRequestPath = "C:\certrequest"
    }
    New-AzsCertificateSigningRequest @edgeCSRparams
    ```

    Si vous créez un certificat VPN, tapez : 

    ```azurepowershell
    $edgeCSRparams = @{
        CertificateType = 'AzureStackEdgeVPN'
        externalFQDN = 'azurestackedge.contoso.com'
        OutputRequestPath = "C:\certrequest"
    }
    New-AzsCertificateSigningRequest @edgeCSRparams
    ```

    
5. Les fichiers de demande de certificat se trouvent dans le répertoire que vous avez spécifié dans le paramètre OutputRequestPath ci-dessus. Quand vous utilisez le paramètre `MultipleCSR`, vous devez compter les quatre fichiers suivants avec l’extension `.req` :

    
    |Noms de fichiers  |Type de demande de certificat  |
    |---------|---------|
    |Commençant par votre `DeviceName`     |Demande de certificat de l’interface utilisateur web locale      |
    |Commençant par votre `NodeSerialNumber`     |Demande de certificat de nœud         |
    |À compter de `login`     |Demande de certificat de point de terminaison Azure Resource Manager       |
    |À compter de `wildcard`     |Demande de certificat de stockage de blobs Elle contient un caractère générique, car elle couvre tous les comptes de stockage que vous pouvez créer sur l’appareil.          |
    |À compter de `AzureStackEdgeVPNCertificate`     |Demande de certificat client VPN.         |

    Vous verrez également un dossier INF. Celui-ci contient un fichier d’informations management.<nomdel’appareil-edge> en texte clair expliquant les détails du certificat.  


6. Envoyez ces fichiers à votre autorité de certification (interne ou publique). Assurez-vous que votre autorité de certification crée des certificats en utilisant votre demande générée qui est conforme aux exigences de certificat Azure Stack Edge Pro pour les [certificats de nœud](azure-stack-edge-j-series-manage-certificates.md#node-certificates), les [certificats de point de terminaison](azure-stack-edge-j-series-manage-certificates.md#endpoint-certificates) et les [certificats d’interface utilisateur locale](azure-stack-edge-j-series-manage-certificates.md#local-ui-certificates).

## <a name="prepare-certificates-for-deployment"></a>Préparer les certificats pour le déploiement

Les fichiers de certificat que vous obtenez auprès de l’autorité de certification doivent être importés et exportés avec des propriétés qui correspondent aux exigences de certificat de l’appareil Azure Stack Edge Pro. Effectuez les étapes suivantes sur le système avec lequel vous avez généré les demandes de signature de certificat.

- Pour importer les certificats, suivez les étapes décrites dans [Importer des certificats sur les clients accédant à votre appareil Azure Stack Edge Pro](azure-stack-edge-j-series-manage-certificates.md#import-certificates-on-the-client-accessing-the-device).

- Pour exporter les certificats, suivez les étapes décrites dans [Exporter des certificats du client accédant à l’appareil Azure Stack Edge Pro](azure-stack-edge-j-series-manage-certificates.md#import-certificates-on-the-client-accessing-the-device).


## <a name="validate-certificates"></a>Valider les certificats

Tout d’abord, vous allez générer une structure de dossiers appropriée, puis placer les certificats dans les dossiers correspondants. Vous ne pourrez valider les certificats à l’aide de l’outil qu’après cette opération.

1. Démarrez PowerShell en tant qu'administrateur.

2. Pour générer la structure de dossiers appropriée, à l’invite, tapez :

    `New-AzsCertificateFolder -CertificateType AzureStackEdge -OutputPath "$ENV:USERPROFILE\Documents\AzureStackCSR"`

3. Convertissez le mot de passe PFX en chaîne sécurisée. Tapez :       

    `$pfxPassword = Read-Host -Prompt "Enter PFX Password" -AsSecureString` 

4. Ensuite, validez les certificats. Tapez :

    `Invoke-AzsCertificateValidation -CertificateType AzureStackEdge -DeviceName mytea1 -NodeSerialNumber VM1500-00025 -externalFQDN azurestackedge.contoso.com -CertificatePath $ENV:USERPROFILE\Documents\AzureStackCSR\AzureStackEdge -pfxPassword $pfxPassword`

## <a name="next-steps"></a>Étapes suivantes

[Déployer votre appareil Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-prep.md)

---
title: Créer un environnement ASE ILB v1
description: Créez un environnement App Service avec un équilibreur de charge (ASE ILB). Ce document s’adresse uniquement aux clients qui utilisent l’environnement ASE v1 hérité.
author: stefsch
ms.assetid: 091decb6-b0de-42a1-9f2f-c18d9b2e67df
ms.topic: article
ms.date: 07/11/2017
ms.author: stefsch
ms.custom: seodec18
ms.openlocfilehash: 2a03b791f37868010e107214ddcb7cf42174e4e1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85833551"
---
# <a name="how-to-create-an-ilb-ase-using-azure-resource-manager-templates"></a>Comment créer un ILB ASE à l’aide des modèles Azure Resource Manager

> [!NOTE] 
> Cet article traite de l’environnement App Service Environment v1. Il existe une version plus récente de l’environnement App Service Environment, plus facile à utiliser et qui s’exécute sur des infrastructures plus puissantes. Pour en savoir plus sur la nouvelle version, commencez par la section [Présentation de l’environnement App Service Environment](intro.md).
>

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="overview"></a>Vue d’ensemble
Les environnements App Service peuvent être créés avec une adresse interne de réseau virtuel au lieu d’une adresse IP virtuelle publique.  Cette adresse interne est fournie par un composant Azure appelé « équilibreur de charge interne » (ILB).  Vous pouvez créer un ILB ASE à l’aide du portail Azure.  Il peut également être créé de manière automatisée par le biais des modèles Azure Resource Manager.  Cet article décrit les étapes et la syntaxe nécessaires à la création d’un ILB ASE à l’aide des modèles Azure Resource Manager.

La création automatisée d’un ILB ASE comporte trois étapes :

1. Tout d’abord, l’ASE de base est créé dans un réseau virtuel à l’aide d’une adresse d’équilibreur de charge interne au lieu d’une adresse IP virtuelle publique.  Dans le cadre de cette étape, un nom de domaine racine est affecté à l’ILB ASE.
2. Une fois l’ILB ASE créé, un certificat TLS/SSL est chargé.  
3. Le certificat TLS/SSL chargé est explicitement affecté à l’ILB ASE en tant que certificat TLS/SSL « par défaut ».  Ce certificat TLS/SSL est utilisé pour le trafic TLS à destination des applications de l’ILB ASE quand celles-ci ont pour adresse le domaine racine commun affecté à l’ASE (par exemple, `https://someapp.mycustomrootcomain.com`).

## <a name="creating-the-base-ilb-ase"></a>Création de l’ILB ASE de base
Un exemple de modèle Azure Resource Manager et son fichier de paramètres associés sont disponibles sur GitHub, [ici][quickstartilbasecreate].

La plupart des paramètres du fichier *azuredeploy.parameters.json* sont communs à la création d’ASE ILB, ainsi que d’ASE liés à une adresse IP virtuelle publique.  La liste ci-dessous répertorie les paramètres spécifiques, ou qui sont uniques, lors de la création d’un ILB ASE :

* *internalLoadBalancingMode* :  dans la plupart des cas, définissez ce paramètre sur 3, ce qui signifie que le trafic HTTP/HTTPS sur les ports 80/443 ainsi que sur les ports des canaux de contrôle/données écoutés par le service FTP dans l'environnement ASE sera lié à une adresse interne du réseau virtuel allouée à l'ILB.  Si cette propriété a la valeur 2, seuls les ports relatifs au service FTP (canaux de contrôle et de données) seront liés à une adresse d’ILB, tandis que le trafic HTTP/HTTPS restera sur l’adresse IP virtuelle publique.
* *dnsSuffix* :  ce paramètre définit le domaine racine par défaut qui sera affecté à l'environnement ASE.  Dans la version publique d’Azure App Service, le domaine racine par défaut pour toutes les applications web est *azurewebsites.net*.  Cependant, étant donné qu’un ILB ASE est interne au réseau virtuel d’un client, il n’est pas pertinent d’utiliser le domaine racine par défaut du service public.  Au lieu de cela, un ILB ASE doit avoir un domaine racine par défaut approprié pour une utilisation au sein du réseau virtuel interne d’une société.  Par exemple, une société fictive Contoso Corporation peut utiliser le domaine racine par défaut *internal-contoso.com* pour les applications qui sont destinées à être résolues et accessibles uniquement au sein du réseau virtuel de Contoso. 
* *ipSslAddressCount* :  par défaut, ce paramètre est automatiquement défini sur la valeur 0 dans le fichier *azuredeploy.json* car les ILB ASE disposent d'une seule adresse d'ILB.  Il n’existe aucune adresse IP SSL explicite pour un ILB ASE. Par conséquent, le pool d’adresses IP SSL pour un ILB ASE doit être défini sur zéro, sinon une erreur d’approvisionnement se produit. 

Une fois le fichier *azuredeploy.parameters.json* renseigné pour un ILB ASE, ce dernier peut ensuite être créé à l’aide de l’extrait de code PowerShell suivant.  Changez les chemins de fichiers pour qu’ils correspondent aux emplacements où se trouvent les fichiers de modèle Azure Resource Manager sur votre machine.  Pensez également à indiquer vos propres valeurs pour le nom de déploiement d’Azure Resource Manager et le nom de groupe de ressources.

```azurepowershell-interactive
$templatePath="PATH\azuredeploy.json"
$parameterPath="PATH\azuredeploy.parameters.json"

New-AzResourceGroupDeployment -Name "CHANGEME" -ResourceGroupName "YOUR-RG-NAME-HERE" -TemplateFile $templatePath -TemplateParameterFile $parameterPath
```

Une fois le modèle Azure Resource Manager soumis, la création de l’ILB ASE prendra quelques heures.  Une fois la création terminée, l’ASE ILB s’affichera dans le portail dans la liste des environnements App Service pour l’abonnement qui a déclenché le déploiement.

## <a name="uploading-and-configuring-the-default-tlsssl-certificate"></a>Chargement et configuration du certificat TLS/SSL « par défaut »
Une fois l’ILB ASE créé, un certificat TLS/SSL doit être associé à l’ASE en tant que certificat TLS/SSL « par défaut » utilisé pour établir des connexions TLS/SSL avec les applications.  Poursuivons avec l’exemple de la société fictive Contoso Corporation. Si le suffixe DNS par défaut de l’ASE est *internal-contoso.com*, une connexion à *`https://some-random-app.internal-contoso.com`* nécessite un certificat TLS/SSL valide pour * *.internal-contoso.com*. 

Il existe plusieurs façons de se procurer un certificat TLS/SSL valide, notamment les autorités de certification internes, l’achat d’un certificat auprès d’un émetteur externe ou l’utilisation d’un certificat auto-signé.  Quelle que soit la source du certificat TLS/SSL, les attributs de certificat suivants doivent être configurés correctement :

* *Objet* :  cet attribut doit être défini sur * *.votre-domaine-racine-ici.com*.
* *Autre nom de l'objet* :  cet attribut doit inclure à la fois * *.votre-domaine-racine-ici.com* et * *.scm.votre-domaine-racine-ici.com*.  La deuxième entrée est nécessaire, car les connexions TLS au site SCM/Kudu associé à chaque application sont établies à l’aide d’une adresse au format *votre-nom-application.scm.votre-domaine-racine-ici.com*.

Une fois le certificat TLS/SSL valide à disposition, deux étapes préparatoires supplémentaires sont nécessaires.  Le certificat TLS/SSL doit être converti/enregistré sous forme de fichier .pfx.  N’oubliez pas que le fichier .pfx doit inclure tous les certificats intermédiaires et racine et être sécurisé avec un mot de passe.

Le fichier .pfx obtenu doit ensuite être converti en chaîne au format base64, car le certificat TLS/SSL est chargé à l’aide d’un modèle Azure Resource Manager.  Étant donné que les modèles Azure Resource Manager sont des fichiers texte, le fichier .pfx doit être converti en une chaîne au format Base64 afin d’être inclus en tant que paramètre du modèle.

L’extrait de code PowerShell ci-dessous montre un exemple de génération d’un certificat auto-signé, d’exportation du certificat en fichier .pfx, de conversion du fichier .pfx au format Base64 de la chaîne codée, puis d’enregistrement de la chaîne codée en Base64 dans un fichier distinct.  Le code PowerShell pour l’encodage en Base64 est une adaptation issue du [blog des scripts PowerShell][examplebase64encoding].

```azurepowershell-interactive
$certificate = New-SelfSignedCertificate -certstorelocation cert:\localmachine\my -dnsname "*.internal-contoso.com","*.scm.internal-contoso.com"

$certThumbprint = "cert:\localMachine\my\" + $certificate.Thumbprint
$password = ConvertTo-SecureString -String "CHANGETHISPASSWORD" -Force -AsPlainText

$fileName = "exportedcert.pfx"
Export-PfxCertificate -cert $certThumbprint -FilePath $fileName -Password $password     

$fileContentBytes = get-content -encoding byte $fileName
$fileContentEncoded = [System.Convert]::ToBase64String($fileContentBytes)
$fileContentEncoded | set-content ($fileName + ".b64")
```

Une fois le certificat TLS/SSL généré et converti en chaîne encodée en base64, vous pouvez utiliser l’exemple de modèle Azure Resource Manager sur GitHub pour [configurer le certificat TLS/SSL par défaut][configuringDefaultSSLCertificate].

Les paramètres du fichier *azuredeploy.parameters.json* sont répertoriés ci-dessous :

* *appServiceEnvironmentName* :  nom de l'ILB ASE configuré.
* *existingAseLocation* :  chaîne de texte contenant la région Azure où l'ILB ASE a été déployé.  Par exemple :  « USA Centre Sud ».
* *pfxBlobString* :  représentation sous forme de chaîne codée en Base64 du fichier .pfx.  À l’aide de l’extrait de code ci-dessus, copiez la chaîne contenue dans « exportedcert.pfx.b64 » et collez-la en tant que valeur de l’attribut *pfxBlobString* .
* *mot de passe* :  mot de passe utilisé pour sécuriser le fichier .pfx.
* *certificateThumbprint* :  empreinte numérique du certificat.  Si vous récupérez cette valeur à partir de PowerShell (par exemple, *$certificate.Thumbprint* dans l’extrait de code précédent), vous pouvez utiliser la valeur telle quelle.  Toutefois, si vous copiez la valeur à partir de la boîte de dialogue de certificat Windows, n’oubliez pas de retirer les espaces superflus.  Le *certificateThumbprint* se présente comme suit :  AF3143EB61D43F6727842115BB7F17BBCECAECAE
* *certificateName* :  identificateur de chaîne convivial de votre choix permettant d'identifier le certificat.  Ce nom fait partie de l’identificateur Azure Resource Manager unique pour l’entité *Microsoft.Web/certificates* représentant le certificat TLS/SSL.  Le nom **doit** se terminer par le suffixe suivant : \_nomdevotreASE_InternalLoadBalancingASE.  Ce suffixe permet au portail d’indiquer que le certificat est utilisé pour sécuriser un ASE avec équilibrage de charge interne.

Un exemple abrégé de fichier *azuredeploy.parameters.json* est présenté ci-dessous :

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "appServiceEnvironmentName": {
            "value": "yourASENameHere"
        },
        "existingAseLocation": {
            "value": "East US 2"
        },
        "pfxBlobString": {
            "value": "MIIKcAIBAz...snip...snip...pkCAgfQ"
        },
        "password": {
            "value": "PASSWORDGOESHERE"
        },
        "certificateThumbprint": {
            "value": "AF3143EB61D43F6727842115BB7F17BBCECAECAE"
        },
        "certificateName": {
            "value": "DefaultCertificateFor_yourASENameHere_InternalLoadBalancingASE"
        }
    }
}
```

Une fois le fichier *azuredeploy.parameters.json* complété, le certificat TLS/SSL par défaut peut être configuré à l’aide de l’extrait de code PowerShell suivant.  Changez les chemins de fichiers pour qu’ils correspondent aux emplacements où se trouvent les fichiers de modèle Azure Resource Manager sur votre machine.  Pensez également à indiquer vos propres valeurs pour le nom de déploiement d’Azure Resource Manager et le nom de groupe de ressources.

```azurepowershell-interactive
$templatePath="PATH\azuredeploy.json"
$parameterPath="PATH\azuredeploy.parameters.json"

New-AzResourceGroupDeployment -Name "CHANGEME" -ResourceGroupName "YOUR-RG-NAME-HERE" -TemplateFile $templatePath -TemplateParameterFile $parameterPath
```

Une fois le modèle Azure Resource Manager soumis, cela prendra environ quarante minutes par serveur frontal de l’ASE pour que la modification soit appliquée.  Par exemple, avec un ASE par défaut utilisant deux serveurs frontaux, l’application du modèle prendra environ une heure et vingt minutes.  Lorsque le modèle est en cours d’exécution, l’ASE ne peut pas être mis à l’échelle.  

Une fois le modèle terminé, les applications de l’ILB ASE sont accessibles via le protocole HTTPS et les connexions sont sécurisées à l’aide du certificat TLS/SSL par défaut.  Le certificat TLS/SSL par défaut est utilisé quand l’adresse des applications de l’ILB ASE se compose du nom de l’application et du nom d’hôte par défaut.  Par exemple, *`https://mycustomapp.internal-contoso.com`* utilise le certificat TLS/SSL par défaut pour * *.internal-contoso.com*.

Cependant, à l’instar des applications qui s’exécutent sur le service multilocataire public, les développeurs peuvent aussi configurer des noms d’hôte personnalisés pour des applications individuelles, puis configurer des liaisons de certificat SNI TLS/SSL uniques pour ces applications individuelles.  

## <a name="getting-started"></a>Prise en main
Pour prendre en main les environnements App Service, consultez [Présentation de l'environnement App Service](app-service-app-service-environment-intro.md)

[!INCLUDE [app-service-web-try-app-service](../../../includes/app-service-web-try-app-service.md)]

<!-- LINKS -->
[quickstartilbasecreate]: https://azure.microsoft.com/documentation/templates/201-web-app-ase-ilb-create/
[examplebase64encoding]: https://powershellscripts.blogspot.com/2007/02/base64-encode-file.html 
[configuringDefaultSSLCertificate]: https://azure.microsoft.com/documentation/templates/201-web-app-ase-ilb-configure-default-ssl/ 


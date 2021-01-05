---
title: Créer un environnement ASE avec ARM
description: Découvrez comment créer un environnement App Service externe ou ILB à l’aide d’un modèle Azure Resource Manager.
author: ccompy
ms.assetid: 6eb7d43d-e820-4a47-818c-80ff7d3b6f8e
ms.topic: article
ms.date: 06/13/2017
ms.author: ccompy
ms.custom: seodec18
ms.openlocfilehash: 15cd0979fdc2468ab50451042cd99a8442470139
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92148166"
---
# <a name="create-an-ase-by-using-an-azure-resource-manager-template"></a>Créer un ASE à l’aide d’un modèle Azure Resource Manager

## <a name="overview"></a>Vue d’ensemble

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Les environnements Azure App Service (ASE, App Service Environment) peuvent être créés avec un point de terminaison accessible via Internet ou un point de terminaison sur une adresse interne d’un réseau virtuel Azure. S’il est créé avec un point de terminaison interne, ce point de terminaison est fourni par un composant Azure appelé équilibreur de charge interne (ILB, Internal Load Balancer). Un ASE sur une adresse IP interne est appelé ASE ILB. Un ASE avec un point de terminaison public est appelé ASE externe. 

Un ASE peut être créé à l’aide du portail Azure ou d’un modèle Azure Resource Manager. Cet article décrit les étapes et la syntaxe nécessaires pour créer un ASE externe ou un ASE ILB à l’aide de modèles Resource Manager. Pour découvrir comment créer un ASE dans le portail Azure, voir [Créer un environnement App Service Environment externe][MakeExternalASE] ou [Créer et utiliser un équilibreur de charge interne avec un environnement Azure App Service Environment][MakeILBASE].

Lorsque vous créez un ASE dans le portail Azure, vous pouvez créer votre réseau virtuel en même temps, ou choisir un réseau virtuel préexistant pour le déploiement. Lorsque vous créez un ASE à partir d’un modèle, vous devez commencer avec : 

* Un réseau virtuel Resource Manager.
* Un sous-réseau de ce réseau virtuel. Pour le sous-réseau ASE, nous recommandons une taille de `/24` avec 256 adresses pour s’adapter à une croissance et une mise à l’échelle futures. Une fois l’ASE créé, vous ne pouvez plus en modifier la taille.
* L’ID de ressource de votre réseau virtuel. Vous pouvez le trouver sur portail Azure sous les propriétés de votre réseau virtuel Azure.
* L’abonnement vers lequel vous souhaitez procéder au déploiement.
* L’emplacement dans lequel vous souhaitez procéder au déploiement.

Pour automatiser la création de votre environnement ASE :

1. Créez l’environnement ASE à partir d’un modèle. Si vous créez un ASE externe, vous avez terminé après cette étape. Si vous créez un ASE ILB, il reste quelques étapes à accomplir.

2. Une fois votre ASE ILB créé, un certificat TLS/SSL correspondant à votre domaine ASE ILB est chargé.

3. Le certificat TLS/SSL chargé est explicitement attribué à l’ASE ILB en tant que certificat TLS/SSL « par défaut ».  Ce certificat est utilisé pour le trafic TLS/SSL vers les applications de l’ASE ILB quand celles-ci utilisent le domaine racine commun assigné à l’ASE (par exemple, `https://someapp.mycustomrootdomain.com`).


## <a name="create-the-ase"></a>Créer l’ASE
Un modèle Azure Resource Manager permettant de créer un ASE et son fichier de paramètres associé est disponible sous forme d’[exemple][quickstartasev2create] sur GitHub.

Si vous souhaitez créer un ASE ILB, utilisez ces [exemples][quickstartilbasecreate] de modèle Resource Manager. Ils sont destinés à ce cas d’usage. La plupart des paramètres définis dans le fichier *azuredeploy.parameters.json* sont communs à la création des ASE ILB et des ASE externes. La liste suivante énonce les paramètres appelant un commentaire particulier ou qui sont uniques, en lien avec la création d’un ASE ILB :

* *internalLoadBalancingMode* : dans la plupart des cas, définissez ce paramètre sur 3, ce qui signifie que le trafic HTTP/HTTPS sur les ports 80/443 ainsi que les ports de canaux de contrôle/données écoutés par le service FTP sur l’ASE seront liés à une adresse interne du réseau virtuel allouée à l’ILB. Si ce paramètre est défini sur 2, seuls les ports associés au service FTP (canaux de contrôle et de données) sont liés à une adresse d’ILB. Le trafic HTTP/HTTPS reste sur l’adresse IP virtuelle publique.
* *dnsSuffix* : ce paramètre définit le domaine racine par défaut affecté à l’ASE. Dans la version publique d’Azure App Service, le domaine racine par défaut pour toutes les applications web est *azurewebsites.net*. Étant donné qu’un ASE ILB est interne au réseau virtuel d’un client, il n’est pas pertinent d’utiliser le domaine racine par défaut du service public. Au lieu de cela, un ILB ASE doit avoir un domaine racine par défaut approprié pour une utilisation au sein du réseau virtuel interne d’une société. Par exemple, une société nommée Contoso Corporation peut utiliser le domaine racine par défaut *internal-contoso.com* pour les applications qui sont destinées à être résolues et accessibles uniquement au sein du réseau virtuel de Contoso. 
* *ipSslAddressCount* : ce paramètre est automatiquement défini par défaut sur la valeur 0 dans le fichier *azuredeploy.json*, car les ASE ILB disposent d’une seule adresse d’ILB. Il n’existe pas d’adresse IP SSL explicite pour un ASE ILB. Par conséquent, le pool d’adresses IP SSL pour un ASE ILB doit être défini sur zéro. Autrement, une erreur d’approvisionnement se produit. 

Une fois le fichier *azuredeploy.parameters.json* complété, créez l’ASE à l’aide de l’extrait de code PowerShell. Modifiez les chemins d’accès des fichiers de façon à ce qu’ils correspondent aux emplacements des fichiers du modèle Resource Manager sur votre ordinateur. Songez à indiquer vos propres valeurs pour les noms de déploiement Resource Manager et de groupe de ressources :

```powershell
$templatePath="PATH\azuredeploy.json"
$parameterPath="PATH\azuredeploy.parameters.json"

New-AzResourceGroupDeployment -Name "CHANGEME" -ResourceGroupName "YOUR-RG-NAME-HERE" -TemplateFile $templatePath -TemplateParameterFile $parameterPath
```

La création de l’ASE prend environ une heure. Ensuite, l’ASE apparaît sur le portail dans la liste des ASE pour l’abonnement qui a déclenché le déploiement.

## <a name="upload-and-configure-the-default-tlsssl-certificate"></a>Charger et configurer le certificat TLS/SSL « par défaut »
Un certificat TLS/SSL doit être associé à l’ASE en tant que certificat TLS/SSL « par défaut » utilisé pour établir les connexions TLS aux applications. Si le suffixe DNS par défaut de l’ASE est *internal-contoso.com*, une connexion à `https://some-random-app.internal-contoso.com` nécessite un certificat TLS/SSL valide pour * *.internal-contoso.com*. 

Vous pouvez vous procurer un certificat TLS/SSL valide auprès d’autorités de certification internes, en achetant un certificat à un émetteur externe ou en utilisant un certificat autosigné. Quelle que soit la source du certificat TLS/SSL, les attributs de certificat suivants doivent être configurés correctement :

* **Objet** : cet attribut doit être défini sur * *.votre-domaine-racine-ici.com*.
* **Autre nom de l'objet** : cet attribut doit inclure à la fois * *.votre-domaine-racine-ici.com* et * *.scm.votre-domaine-racine-ici.com*. Les connexions TLS au site GCL/Kudu associé à chaque application utilisent une adresse au format *nom-de-votre-application.scm.votre-domaine-racine-ici.com*.

Une fois le certificat TLS/SSL valide à disposition, deux étapes préparatoires supplémentaires sont nécessaires. Convertissez/enregistrez le certificat TLS/SSL en tant que fichier .pfx. N’oubliez pas que le fichier .pfx doit inclure tous les certificats racines et intermédiaires. Sécurisez-le avec un mot de passe.

Le fichier .pfx doit être converti en chaîne base64, car le certificat TLS/SSL est chargé à l’aide d’un modèle Resource Manager. Étant donné que les modèles Resource Manager sont des fichiers texte, le fichier .pfx doit être converti en chaîne base64. Ainsi, il peut être inclus en tant que paramètre du modèle.

Utilisez l’extrait de code PowerShell ci-dessous pour effectuer les opérations suivantes :

* générer un certificat auto-signé ;
* exporter le certificat dans un fichier .pfx ;
* convertir le fichier .pfx en une chaîne codée en base64 ;
* enregistrer la chaîne codée en base64 dans un fichier distinct. 

Le code PowerShell pour l’encodage en base64 à été adapté à partir du [Blog relatif aux scripts PowerShell][examplebase64encoding] :

```powershell
$certificate = New-SelfSignedCertificate -certstorelocation cert:\localmachine\my -dnsname "*.internal-contoso.com","*.scm.internal-contoso.com"

$certThumbprint = "cert:\localMachine\my\" + $certificate.Thumbprint
$password = ConvertTo-SecureString -String "CHANGETHISPASSWORD" -Force -AsPlainText

$fileName = "exportedcert.pfx"
Export-PfxCertificate -cert $certThumbprint -FilePath $fileName -Password $password     

$fileContentBytes = get-content -encoding byte $fileName
$fileContentEncoded = [System.Convert]::ToBase64String($fileContentBytes)
$fileContentEncoded | set-content ($fileName + ".b64")
```

Une fois le certificat TLS/SSL généré et converti en chaîne codée en base64, utilisez l’exemple de modèle Resource Manager [Configurer le certificat SSL par défaut][quickstartconfiguressl] disponible sur GitHub. 

Les paramètres figurant dans le fichier *azuredeploy.parameters.json* sont répertoriés ci-dessous :

* *appServiceEnvironmentName* : nom de l'ILB ASE configuré.
* *existingAseLocation* : chaîne de texte contenant la région Azure où l'ILB ASE a été déployé.  Par exemple : « USA Centre Sud ».
* *pfxBlobString* : représentation sous forme de chaîne codée en Base64 du fichier .pfx. Utilisez l’extrait de code présenté précédemment, et copiez la chaîne contenue dans « exportedcert.pfx.b64 ». Collez celle-ci en tant que valeur de l’attribut *pfxBlobString*.
* *mot de passe* : mot de passe utilisé pour sécuriser le fichier .pfx.
* *certificateThumbprint* : empreinte numérique du certificat. Si vous récupérez cette valeur à partir de Powershell (par exemple, *$certificate.Thumbprint* dans l’extrait de code précédent), vous pouvez utiliser la valeur telle quelle. Si vous copiez la valeur à partir de la boîte de dialogue du certificat Windows, n’oubliez pas de retirer les espaces superflus. La valeur *certificateThumbprint* doit se présenter sous la forme suivante : AF3143EB61D43F6727842115BB7F17BBCECAECAE.
* *certificateName* : identificateur de chaîne convivial de votre choix permettant d'identifier le certificat. Ce nom fait partie de l’identificateur Resource Manager unique pour l’entité *Microsoft.Web/certificates* qui représente le certificat TLS/SSL. Le nom *doit* se terminer par le suffixe suivant : \_nomdevotreASE_ÉquilibrageChareInterneASE. Le portail Azure utilise ce suffixe en tant qu’indicateur signalant que le certificat est utilisé pour sécuriser ASE avec ILB.

Un exemple abrégé du fichier *azuredeploy.parameters.json* est présenté ici :

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

Une fois le fichier *azuredeploy.parameters.json* rempli, configurez le certificat TLS/SSL par défaut à l’aide de l’extrait de code PowerShell. Modifiez les chemins d’accès aux fichiers pour qu’ils correspondent aux emplacements où se trouvent les fichiers du modèle Azure Resource Manager sur votre ordinateur. Songez à indiquer vos propres valeurs pour les noms de déploiement Resource Manager et de groupe de ressources :

```powershell
$templatePath="PATH\azuredeploy.json"
$parameterPath="PATH\azuredeploy.parameters.json"

New-AzResourceGroupDeployment -Name "CHANGEME" -ResourceGroupName "YOUR-RG-NAME-HERE" -TemplateFile $templatePath -TemplateParameterFile $parameterPath
```

L’application de la modification prend environ 40 minutes par serveur frontal ASE. Par exemple, pour un ASE dimensionné par défaut utilisant deux serveurs frontaux, l’application du modèle prend environ une heure et vingt minutes. Lorsque le modèle est en cours d’exécution, l’ASE ne peut pas mettre à l’échelle.  

Une fois l’exécution du modèle terminé, les applications sur l’ILB ASE est accessible via le protocole HTTPS. Les connexions sont sécurisées à l’aide du certificat TLS/SSL par défaut. Le certificat TLS/SSL par défaut est utilisé lorsque des applications sur l’ASE ILB sont adressée à l’aide d’une combinaison de leur nom et du nom d’hôte par défaut. Par exemple, `https://mycustomapp.internal-contoso.com` utilise le certificat TLS/SSL par défaut pour * *.internal-contoso.com*.

Cependant, comme pour les applications qui s’exécutent sur le service mutualisé public, les développeurs peuvent configurer des noms d’hôtes personnalisés pour des applications individuelles. Ils peuvent également configurer des liaisons de certificat TLS/SSL SNI uniques pour des applications individuelles.

## <a name="app-service-environment-v1"></a>Environnement App Service v1 ##
App Service Environment est disponible en deux versions : ASEv1 et ASEv2. Les informations précédentes sont basées sur ASEv2. Cette section montre les différences entre ASEv1 et ASEv2.

Dans ASEv1, vous gérez toutes les ressources manuellement. Celles-ci incluent les frontends, les workers et les adresses IP utilisées pour le protocole SSL basé sur IP. Pour pouvoir effectuer un scale-out de votre plan App Service, vous devez commencer par effectuer un scale-out du pool de workers dans lequel vous voulez héberger le plan.

Les versions ASEv1 et ASEv2 utilisent un modèle tarifaire différent. Dans ASEv1, vous payez pour chaque processeur virtuel alloué. Cela inclut les processeurs virtuels utilisés pour les serveurs frontaux ou les workers qui n’hébergent pas de charge de travail. Dans la version ASEv1, la taille d’échelle maximale par défaut d’un environnement App Service correspond à un total de 55 hôtes, dont les workers et les frontends. L’un des avantages d’un ASEv1 est qu’il peut être déployé sur un réseau virtuel classique et sur un réseau virtuel Resource Manager. Pour plus d’informations sur ASEv1, consultez [Présentation de l’environnement App Service v1][ASEv1Intro].

Pour créer un ASEv1 à l’aide d’un modèle Resource Manager, voir [Comment créer un ILB ASE à l’aide des modèles Azure Resource Manager][ILBASEv1Template].


<!--Links-->
[quickstartilbasecreate]: https://azure.microsoft.com/documentation/templates/201-web-app-asev2-ilb-create
[quickstartasev2create]: https://azure.microsoft.com/documentation/templates/201-web-app-asev2-create
[quickstartconfiguressl]: https://azure.microsoft.com/documentation/templates/201-web-app-ase-ilb-configure-default-ssl
[quickstartwebapponasev2create]: https://azure.microsoft.com/documentation/templates/201-web-app-asp-app-on-asev2-create
[examplebase64encoding]: https://powershellscripts.blogspot.com/2007/02/base64-encode-file.html 
[configuringDefaultSSLCertificate]: https://azure.microsoft.com/documentation/templates/201-web-app-ase-ilb-configure-default-ssl/
[Intro]: ./intro.md
[MakeExternalASE]: ./create-external-ase.md
[MakeASEfromTemplate]: ./create-from-template.md
[MakeILBASE]: ./create-ilb-ase.md
[ASENetwork]: ./network-info.md
[UsingASE]: ./using-an-ase.md
[UDRs]: ../../virtual-network/virtual-networks-udr-overview.md
[NSGs]: ../../virtual-network/network-security-groups-overview.md
[ConfigureASEv1]: app-service-web-configure-an-app-service-environment.md
[ASEv1Intro]: app-service-app-service-environment-intro.md
[mobileapps]: /previous-versions/azure/app-service-mobile/app-service-mobile-value-prop
[Functions]: ../../azure-functions/index.yml
[Pricing]: https://azure.microsoft.com/pricing/details/app-service/
[ARMOverview]: ../../azure-resource-manager/management/overview.md
[ConfigureSSL]: ../../app-service/configure-ssl-certificate.md
[Kudu]: https://azure.microsoft.com/resources/videos/super-secret-kudu-debug-console-for-azure-web-sites/
[ASEWAF]: app-service-app-service-environment-web-application-firewall.md
[AppGW]: ../../web-application-firewall/ag/ag-overview.md
[ILBASEv1Template]: app-service-app-service-environment-create-ilb-ase-resourcemanager.md
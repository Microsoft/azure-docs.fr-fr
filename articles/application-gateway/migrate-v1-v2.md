---
title: Effectuer la migration de la version 1 à la version 2 - Azure Application Gateway
description: Cet article montre comment migrer la passerelle Azure Application Gateway et le pare-feu d’applications web de v1 à v2
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: how-to
ms.date: 03/31/2020
ms.author: victorh
ms.openlocfilehash: 3dd46f4033a568a278d7006c0d5aab451496ff47
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93397221"
---
# <a name="migrate-azure-application-gateway-and-web-application-firewall-from-v1-to-v2"></a>Migrer la passerelle Azure Application Gateway et le pare-feu d’applications web de v1 à v2

[La passerelle Azure Application Gateway et le pare-feu d’applications web (WAF) v2](application-gateway-autoscaling-zone-redundant.md) sont maintenant disponibles et offrent des fonctionnalités supplémentaires telles que la mise à l’échelle automatique et la redondance de zone de disponibilité. Toutefois, les passerelles v1 existantes ne sont pas automatiquement mises à niveau vers la v2. Si vous souhaitez migrer de v1 à v2, suivez les étapes décrites dans cet article.

Il existe deux phases dans une migration :

1. Migrer la configuration
2. Migrer le trafic client

Cet article couvre la migration de la configuration. La migration du trafic client varie en fonction de votre environnement spécifique. Toutefois, certaines suggestions générales de haut niveau [sont fournies](#migrate-client-traffic).

## <a name="migration-overview"></a>Vue d’ensemble de la migration

Un script Azure PowerShell est disponible qui effectue les opérations suivantes :

* Il crée une nouvelle passerelle Standard_v2 ou WAF_v2 dans un sous-réseau de réseau virtuel que vous spécifiez.
* Il copie en toute transparence la configuration associée à la passerelle WAF ou Standard v1 vers la passerelle Standard_V2 ou WAF_V2 nouvellement créée.

### <a name="caveatslimitations"></a>Mises en garde/Limitations

* La nouvelle passerelle v2 a de nouvelles adresses IP publiques et privées. Il n’est pas possible de déplacer de façon transparente les adresses IP associées à la passerelle v1 existante vers la version 2. Toutefois, vous pouvez allouer une adresse IP publique ou privée existante (non allouée) à la nouvelle passerelle v2.
* Vous devez fournir un espace d’adressage IP pour un autre sous-réseau au sein de votre réseau virtuel où se trouve votre passerelle v1. Le script ne peut pas créer la passerelle v2 dans les sous-réseaux existants qui ont déjà une passerelle v1. Toutefois, si le sous-réseau existant a déjà une passerelle v2, celle-ci peut encore fonctionner sous réserve d’un espace d’adressage IP suffisant.
* Si vous avez un groupe de sécurité réseau ou des routes définies par l’utilisateur associés au sous-réseau de passerelle v2, assurez-vous qu’ils respectent les [exigences NSG](../application-gateway/configuration-infrastructure.md#network-security-groups) et les [exigences UDR](../application-gateway/configuration-infrastructure.md#supported-user-defined-routes) pour une migration réussie.
* Les [stratégies de points de terminaison de service de réseau virtuel](../virtual-network/virtual-network-service-endpoint-policies-overview.md) ne sont actuellement pas prises en charge dans un sous-réseau Application Gateway.
* Pour migrer une configuration TLS/SSL, vous devez spécifier tous les certificats TLS/SSL utilisés sur votre passerelle v1.
* Si le mode FIPS est activé pour votre passerelle V1, il ne sera pas migré vers votre nouvelle passerelle v2. Le mode FIPS n’est pas pris en charge dans v2.
* La version 2 ne prend pas en charge IPv6, si bien que les passerelles v1 compatibles IPv6 ne sont pas migrées. Si vous exécutez le script, il peut ne pas se terminer.
* Si la passerelle v1 a uniquement une adresse IP privée, le script crée une adresse IP publique et une adresse IP privée pour la nouvelle passerelle v2. Les passerelles v2 ne prennent actuellement pas en charge uniquement les adresses IP privées.
* Les en-têtes dont les noms contiennent autre chose que des lettres, des chiffres, des traits d’union et des traits de soulignement ne sont pas transmis à votre application. Cela s’applique uniquement aux noms des en-têtes, et non aux valeurs des en-têtes. Il s’agit d’un changement cassant par rapport à v1.

## <a name="download-the-script"></a>Télécharger le script

Téléchargez le script de migration à partir de [PowerShell Gallery](https://www.powershellgallery.com/packages/AzureAppGWMigration).

## <a name="use-the-script"></a>Utiliser le script

Vous disposez de deux options selon vos préférences et votre configuration de l’environnement PowerShell local :

* Si vous n’avez pas installé les modules Azure Az ou si vous êtes prêt à désinstaller les modules Azure Az, la meilleure option consiste à utiliser l’option `Install-Script` pour exécuter le script.
* Si vous devez conserver les modules Azure Az, la meilleure solution qui s’offre à vous consiste à télécharger le script et à l’exécuter directement.

Pour déterminer si vous avez installé les modules Azure Az, exécutez `Get-InstalledModule -Name az`. Si vous ne voyez aucun module Az installé, vous pouvez utiliser la méthode `Install-Script`.

### <a name="install-using-the-install-script-method"></a>Installer à l’aide de la méthode Install-Script

Pour pouvoir utiliser cette option, les modules Azure Az ne doivent pas être installés sur votre ordinateur. S’ils sont installés, la commande suivante affiche une erreur. Vous pouvez désinstaller les modules Azure Az ou utiliser l’autre option pour télécharger le script manuellement et l’exécuter.
  
Exécutez le script avec la commande suivante pour récupérer la version la plus récente :

`Install-Script -Name AzureAppGWMigration -Force`

Cette commande installe également les modules Az requis.  

### <a name="install-using-the-script-directly"></a>Installer en utilisant directement le script

Si certains modules Azure Az sont installés et ne peuvent pas être désinstallés (ou si vous ne souhaitez pas les désinstaller), vous pouvez télécharger manuellement le script en utilisant l’onglet **Téléchargement manuel** dans le lien de téléchargement du script. Le script est téléchargé sous forme de fichier nupkg brut. Pour installer le script à partir de ce fichier nupkg, consultez [Téléchargement manuel de package](/powershell/scripting/gallery/how-to/working-with-packages/manual-download).

Pour exécuter le script :

1. Utilisez `Connect-AzAccount` pour vous connecter à Azure.

1. Utilisez `Import-Module Az` pour importer les modules Az.

1. Exécutez `Get-Help AzureAppGWMigration.ps1` pour examiner les paramètres requis :

   ```
   AzureAppGwMigration.ps1
    -resourceId <v1 application gateway Resource ID>
    -subnetAddressRange <subnet space you want to use>
    -appgwName <string to use to append>
    -sslCertificates <comma-separated SSLCert objects as above>
    -trustedRootCertificates <comma-separated Trusted Root Cert objects as above>
    -privateIpAddress <private IP string>
    -publicIpResourceId <public IP name string>
    -validateMigration -enableAutoScale
   ```

   Paramètres liés au script :
   * **resourceId : [chaîne] : Requis** - Il s’agit de l’ID de ressource Azure de votre passerelle WAF v1 ou Standard v1 existante. Pour rechercher cette valeur de chaîne, accédez au portail Azure, sélectionnez votre passerelle d’application ou ressource WAF, puis cliquez sur le lien **Propriétés** pour la passerelle. L’ID de ressource figure dans cette page.

     Vous pouvez également exécuter les commandes Azure PowerShell suivantes pour obtenir l’ID de ressource :

     ```azurepowershell
     $appgw = Get-AzApplicationGateway -Name <v1 gateway name> -ResourceGroupName <resource group Name> 
     $appgw.Id
     ```

   * **subnetAddressRange : [chaîne] :  Requis** - Il s’agit de l’espace d’adressage IP que vous avez alloué (ou que vous souhaitez allouer) pour un nouveau sous-réseau qui contient votre nouvelle passerelle v2. Il doit être spécifié dans la notation CIDR. Par exemple : 10.0.0.0/24. Vous n’avez pas besoin de créer ce sous-réseau à l’avance. Le script le crée pour vous s’il n’existe pas.
   * **appgwName : [chaîne] : Facultatif**. Il s’agit d’une chaîne que vous spécifiez comme nom de la nouvelle passerelle Standard_v2 ou WAF_v2. Si ce paramètre n’est pas fourni, le nom de votre passerelle v1 existante est utilisé avec le suffixe *_v2* ajouté.
   * **sslCertificates : [PSApplicationGatewaySslCertificate] : Facultatif**.  Une liste séparée par des virgules des objets PSApplicationGatewaySslCertificate que vous créez pour représenter les certificats TLS/SSL à partir de votre passerelle v1 doit être chargée sur la nouvelle passerelle v2. Pour chacun des certificats TLS/SSL configurés pour votre passerelle Standard v1 ou WAF v1, vous pouvez créer un nouvel objet PSApplicationGatewaySslCertificate via la commande `New-AzApplicationGatewaySslCertificate` illustrée ici. Vous avez besoin du mot de passe et du chemin d'accès à votre fichier de certificat TLS/SSL.

     Ce paramètre n’est facultatif que si vous n’avez pas d’écouteurs HTTPS configurés pour votre passerelle v1 ou pare-feu d’applications web. Si vous avez au moins une configuration d’écouteur HTTPS, vous devez spécifier ce paramètre.

      ```azurepowershell  
      $password = ConvertTo-SecureString <cert-password> -AsPlainText -Force
      $mySslCert1 = New-AzApplicationGatewaySslCertificate -Name "Cert01" `
        -CertificateFile <Cert-File-Path-1> `
        -Password $password 
      $mySslCert2 = New-AzApplicationGatewaySslCertificate -Name "Cert02" `
        -CertificateFile <Cert-File-Path-2> `
        -Password $password
      ```

     Vous pouvez transmettre `$mySslCert1, $mySslCert2` (séparés par des virgules) dans l’exemple précédent en tant que valeurs pour ce paramètre dans le script.
   * **trustedRootCertificates : [PSApplicationGatewayTrustedRootCertificate] : Facultatif**. Liste séparée par des virgules d’objets PSApplicationGatewayTrustedRootCertificate que vous créez pour représenter les [certificats racines approuvés](ssl-overview.md) pour l’authentification de vos instances back-end à partir de votre passerelle v2.
   
      ```azurepowershell
      $certFilePath = ".\rootCA.cer"
      $trustedCert = New-AzApplicationGatewayTrustedRootCertificate -Name "trustedCert1" -CertificateFile $certFilePath
      ```

      Pour créer une liste d’objets PSApplicationGatewayTrustedRootCertificate, consultez [New-AzApplicationGatewayTrustedRootCertificate](/powershell/module/Az.Network/New-AzApplicationGatewayTrustedRootCertificate?view=azps-2.1.0&viewFallbackFrom=azps-2.0.0).
   * **privateIpAddress : [chaîne] : Facultatif**. Adresse IP privée spécifique que vous souhaitez associer à votre nouvelle passerelle v2.  Elle doit provenir du même réseau virtuel que vous allouez pour votre nouvelle passerelle v2. Si elle n’est pas spécifiée, le script alloue une adresse IP privée pour votre passerelle v2.
   * **publicIpResourceId : [chaîne] : Facultatif**. L’ID de ressource de la ressource d’IP publique (référence SKU standard) existante dans votre abonnement que vous souhaitez allouer à la nouvelle passerelle v2. S’il n’est pas spécifié, le script alloue une nouvelle adresse IP publique dans le même groupe de ressources. Ce nom correspond au nom de la passerelle v2 avec *-IP* ajouté.
   * **validateMigration : [commutateur] : Facultatif**. Utilisez ce paramètre si vous souhaitez que le script effectue des validations de comparaison de configuration de base après la création de la passerelle v2 et la copie de la configuration. Par défaut, aucune validation n’est effectuée.
   * **enableAutoScale : [commutateur] : Facultatif**. Utilisez ce paramètre si vous souhaitez que le script active la mise à l’échelle automatique sur la nouvelle passerelle v2 après sa création. Par défaut, la mise à l’échelle automatique est désactivée. Vous pouvez toujours l’activer manuellement par la suite sur la passerelle v2 nouvellement créée.

1. Exécutez le script en utilisant les paramètres appropriés. Cette opération peut prendre entre cinq et sept minutes.

    **Exemple**

   ```azurepowershell
   AzureAppGWMigration.ps1 `
      -resourceId /subscriptions/8b1d0fea-8d57-4975-adfb-308f1f4d12aa/resourceGroups/MyResourceGroup/providers/Microsoft.Network/applicationGateways/myv1appgateway `
      -subnetAddressRange 10.0.0.0/24 `
      -appgwname "MynewV2gw" `
      -sslCertificates $mySslCert1,$mySslCert2 `
      -trustedRootCertificates $trustedCert `
      -privateIpAddress "10.0.0.1" `
      -publicIpResourceId "/subscriptions/8b1d0fea-8d57-4975-adfb-308f1f4d12aa/resourceGroups/MyResourceGroup/providers/Microsoft.Network/publicIPAddresses/MyPublicIP" `
      -validateMigration -enableAutoScale
   ```

## <a name="migrate-client-traffic"></a>Migrer le trafic client

Tout d’abord, vérifiez soigneusement que le script a réussi à créer une nouvelle passerelle v2 en migrant la configuration exacte à partir de votre passerelle v1. Vous pouvez vérifier cela à partir du portail Azure.

De plus, envoyez une petite quantité de trafic via la passerelle v2 en tant que test manuel.
  
Voici quelques scénarios où votre passerelle d’application actuelle (Standard) peut recevoir le trafic client et nos suggestions pour chacun d’eux :

* **Zone DNS personnalisée (par exemple, contoso.com) qui pointe vers l’adresse IP du front-end (à l’aide d’un enregistrement A) associée à votre passerelle Standard v1 ou WAF v1**.

    Vous pouvez mettre à jour votre enregistrement DNS pour pointer vers l’étiquette DNS ou l’adresse IP de front-end associée à votre passerelle d’application Standard_v2. Selon la durée de vie configurée sur votre enregistrement DNS, la migration de tout votre trafic client vers votre nouvelle passerelle v2 peut prendre un certain temps.
* **Zone DNS personnalisée (par exemple, contoso.com) qui pointe vers l’étiquette DNS (par exemple : *myappgw.eastus.cloudapp.azure.com* à l’aide d’un enregistrement CNAME) associé à votre passerelle v1**.

   Deux options s'offrent à vous :

  * Si vous utilisez des adresses IP publiques sur votre passerelle d’application, vous pouvez effectuer une migration granulaire contrôlée en utilisant un profil Traffic Manager pour acheminer de façon incrémentielle le trafic (méthode de routage du trafic par pondération) vers la nouvelle passerelle v2.

    Vous pouvez procéder en ajoutant les étiquettes DNS des deux passerelles d’application v1 et v2 au [profil Traffic Manager](../traffic-manager/traffic-manager-routing-methods.md#weighted-traffic-routing-method) et en liant via CNAME votre enregistrement DNS personnalisé (par exemple, `www.contoso.com`) au domaine Traffic Manager (par exemple, contoso.trafficmanager.net).
  * Ou, vous pouvez mettre à jour votre enregistrement DNS de domaine personnalisé pour pointer vers l’étiquette DNS de la nouvelle passerelle d’application v2. Selon la durée de vie configurée sur votre enregistrement DNS, la migration de tout votre trafic client vers votre nouvelle passerelle v2 peut prendre un certain temps.
* **Vos clients se connectent à l’adresse IP de front-end de votre passerelle d’application**.

   Mettez à jour vos clients pour utiliser la ou les adresses IP associées à la passerelle d’application v2 nouvellement créée. Nous vous recommandons de ne pas utiliser directement des adresses IP. Envisagez d’utiliser l’étiquette de nom DNS (par exemple, yourgateway.eastus.cloudapp.azure.com) associée à votre passerelle d’application que vous pouvez lier via CNAME à votre propre zone DNS personnalisée (par exemple, contoso.com).

## <a name="common-questions"></a>Questions courantes

### <a name="are-there-any-limitations-with-the-azure-powershell-script-to-migrate-the-configuration-from-v1-to-v2"></a>Existe-t-il des restrictions avec le script Azure PowerShell pour migrer la configuration de la version v1 à v2 ?

Oui. Consultez [Mises en garde/Limitations](#caveatslimitations).

### <a name="is-this-article-and-the-azure-powershell-script-applicable-for-application-gateway-waf-product-as-well"></a>Cet article et le script Azure PowerShell sont-ils également applicables pour le produit Application Gateway WAF ? 

Oui.

### <a name="does-the-azure-powershell-script-also-switch-over-the-traffic-from-my-v1-gateway-to-the-newly-created-v2-gateway"></a>Le script Azure PowerShell bascule-t-il également le trafic de ma passerelle v1 à la passerelle v2 nouvellement créée ?

Non. Le script Azure PowerShell migre uniquement la configuration. La migration de trafic réelle est de votre responsabilité et sous votre contrôle.

### <a name="is-the-new-v2-gateway-created-by-the-azure-powershell-script-sized-appropriately-to-handle-all-of-the-traffic-that-is-currently-served-by-my-v1-gateway"></a>La nouvelle passerelle v2 créée par le script Azure PowerShell est-elle correctement dimensionnée pour traiter tout le trafic qui est actuellement desservi par ma passerelle v1 ?

Le script Azure PowerShell crée une nouvelle passerelle v2 avec une taille appropriée pour gérer le trafic sur votre passerelle v1 existante. La mise à l’échelle automatique est désactivée par défaut, mais vous pouvez l’activer lorsque vous exécutez le script.

### <a name="i-configured-my-v1-gateway--to-send-logs-to-azure-storage-does-the-script-replicate-this-configuration-for-v2-as-well"></a>J’ai configuré ma passerelle v1 pour envoyer les journaux vers le stockage Azure. Le script réplique-t-il également cette configuration pour v2 ?

Non. Le script ne réplique pas cette configuration pour v2. Vous devez ajouter séparément la configuration des journaux à la passerelle v2 migrée.

### <a name="does-this-script-support-certificates-uploaded-to-azure-keyvault-"></a>Ce script prend-il en charge les certificats chargés dans Azure KeyVault ?

Non. Actuellement, le script ne prend pas en charge les certificats dans Key Vault. Toutefois, nous réfléchissons à l’intégrer dans une version future.

### <a name="i-ran-into-some-issues-with-using-this-script-how-can-i-get-help"></a>J’ai rencontré des problèmes en utilisant ce script. Comment obtenir de l’aide ?
  
Vous pouvez contacter le support Azure sous la rubrique « Configuration et installation/migration vers la référence SKU V2 ». En savoir plus sur le [support Azure ici](https://azure.microsoft.com/support/options/).

## <a name="next-steps"></a>Étapes suivantes

[En savoir plus sur Application Gateway v2](application-gateway-autoscaling-zone-redundant.md)
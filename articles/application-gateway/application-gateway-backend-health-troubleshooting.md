---
title: Résoudre les problèmes d’intégrité des back-ends dans Azure Application Gateway
description: Décrit comment résoudre les problèmes d’intégrité des back-ends dans Azure Application Gateway
services: application-gateway
author: surajmb
ms.service: application-gateway
ms.topic: troubleshooting
ms.date: 06/09/2020
ms.author: surmb
ms.openlocfilehash: b8acf1b025a5943773821c8ab78de6288eb6bec2
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93397896"
---
<a name="troubleshoot-backend-health-issues-in-application-gateway"></a>Résoudre les problèmes d’intégrité des back-ends dans Application Gateway
==================================================

<a name="overview"></a>Vue d’ensemble
--------

Par défaut, Azure Application Gateway sonde les serveurs back-end afin de vérifier leur état d’intégrité et leur disponibilité pour traiter les requêtes. Les utilisateurs peuvent également créer des sondes personnalisées pour indiquer le nom d’hôte, le chemin à sonder et les codes d’état à considérer comme sains. Dans les deux cas, si le serveur back-end ne répond pas, Application Gateway marque le serveur comme Non sain et arrête la transmission des requêtes au serveur. Quand le serveur répond de nouveau, Application Gateway reprend la transmission des requêtes.

### <a name="how-to-check-backend-health"></a>Comment vérifier l’intégrité des back-ends

Vous pouvez vérifier l’intégrité de votre pool de back-ends par le biais de la page **Intégrité principale** du portail Azure. Vous pouvez également utiliser [Azure PowerShell](/powershell/module/az.network/get-azapplicationgatewaybackendhealth?view=azps-2.6.0), l’[interface CLI](/cli/azure/network/application-gateway?view=azure-cli-latest#az-network-application-gateway-show-backend-health) ou l’[API REST](/rest/api/application-gateway/applicationgateways/backendhealth).

Toutes ces méthodes font apparaître l’un des états suivants :

- Healthy

- Unhealthy

- Unknown

Si l’état d’intégrité d’un serveur back-end est Sain, Application Gateway transmet les requêtes à ce serveur. Toutefois, si tous les serveurs back-end d’un pool de back-ends sont dans un état Non sain ou Inconnu, vous risquez de rencontrer des problèmes lors de l’accès aux applications. Cet article décrit les symptômes, causes et modes de résolution de chaque erreur présentée.

<a name="backend-health-status-unhealthy"></a>État d’intégrité des back-ends : Unhealthy
-------------------------------

Si l’état d’intégrité des back-ends est Non sain, la vue du portail se présente comme sur la capture suivante :

![Intégrité des back-ends dans Application Gateway - Non sain](./media/application-gateway-backend-health-troubleshooting/appgwunhealthy.png)

Si vous utilisez une requête Azure PowerShell, d’interface CLI ou d’API REST Azure, vous obtenez une réponse qui se présente comme suit :
```azurepowershell
PS C:\Users\testuser\> Get-AzApplicationGatewayBackendHealth -Name "appgw1" -ResourceGroupName "rgOne"
BackendAddressPools :
{Microsoft.Azure.Commands.Network.Models.PSApplicationGatewayBackendHealthPool}
BackendAddressPoolsText : [
{
                              "BackendAddressPool": {
                                "Id": "/subscriptions/536d30b8-665b-40fc-bd7e-68c65f816365/resourceGroups/rgOne/providers/Microsoft.Network/applicationGateways/appgw1/b
                          ackendAddressPools/appGatewayBackendPool"
                              },
                              "BackendHttpSettingsCollection": [
                                {
                                  "BackendHttpSettings": {
                                    "TrustedRootCertificates": [],
                                    "Id": "/subscriptions/536d30b8-665b-40fc-bd7e-68c65f816365/resourceGroups/rgOne/providers/Microsoft.Network/applicationGateways/appg
                          w1/backendHttpSettingsCollection/appGatewayBackendHttpSettings"
                                  },
                                  "Servers": [
                                    {
                                      "Address": "10.0.0.5",
                                      "Health": "Healthy"
                                    },
                                    {
                                      "Address": "10.0.0.6",
                                      "Health": "Unhealthy"
                                    }
                                  ]
                                }
                              ]
                            }
                        ]
```
Quand vous recevez un état d’intégrité Non sain pour tous les serveurs back-end d’un pool de back-ends, les requêtes ne sont pas transmises à ces serveurs, et Application Gateway retourne l’erreur « Passerelle incorrecte - 502 » au client à l’origine des requêtes. Pour résoudre ce problème, examinez la colonne **Détails** de l’onglet **Intégrité principale**.

Le message affiché dans la colonne **Détails** fournit des insights plus détaillés, qui vous aident à entreprendre la résolution du problème.

> [!NOTE]
> La demande de sonde par défaut est envoyée au format \<protocol\>://127.0.0.1:\<port\>/. Par exemple, http://127.0.0.1:80 pour une sonde HTTP sur le port 80. Seuls les codes d’état HTTP de 200 à 399 sont considérés comme sains. Le protocole et le port de destination sont hérités des paramètres HTTP. Si vous souhaitez qu’Application Gateway sonde sur un autre protocole, nom d’hôte ou chemin et reconnaisse un autre code d’état comme sain, configurez une sonde personnalisée et associez-la aux paramètres HTTP.

<a name="error-messages"></a>Messages d’erreur
------------------------
#### <a name="backend-server-timeout"></a>Délai d’attente dépassé pour le serveur back-end

**Message :** Le temps pris par le serveur back-end pour répondre à la sonde d’intégrité d\'Application Gateway est supérieur au seuil de délai d’attente défini dans les paramètres de la sonde.

**Cause :** Après avoir envoyé une demande de sondage HTTP(S) au serveur back-end, Application Gateway attend la réponse de ce serveur pendant une période définie. Si le serveur back-end ne répond pas au cours de la période configurée (valeur de délai), il est marqué comme Non sain jusqu’à ce qu’il recommence à répondre (dans le délai imparti).

**Résolution :** Vérifiez la raison pour laquelle l’application ou le serveur back-end ne répond pas dans le délai configuré, et vérifiez les dépendances de l’application. Par exemple, vérifiez si la base de données présente des problèmes susceptibles d’entraîner un retard dans la réponse. Si l’application a le comportement attendu et qu’elle doit répondre seulement après le délai défini, augmentez la valeur du délai dans les paramètres de la sonde personnalisée. Vous devez disposer d’une sonde personnalisée pour modifier la valeur du délai. Pour plus d’informations sur la configuration d’une sonde personnalisée, consultez la [page de documentation](./application-gateway-create-probe-portal.md).

Pour augmenter la valeur du délai, effectuez les étapes suivantes :

1.  Accédez directement au serveur back-end et regardez le temps que le serveur a pris pour répondre sur cette page. Vous pouvez pour cela utiliser n’importe quel outil, y compris les outils de développement d’un navigateur.

1.  Après avoir déterminé le temps de réponse de l’application, sélectionnez l’onglet **Sondes d’intégrité** , puis sélectionnez la sonde associée à vos paramètres HTTP.

1.  Entrez une valeur de délai supérieure au délai de réponse de l’application, en secondes.

1.  Enregistrez les paramètres de la sonde personnalisée et vérifiez si l’intégrité du back-end a désormais l’état Sain.

#### <a name="dns-resolution-error"></a>Erreur de résolution du DNS

**Message :** Application Gateway n’a pas pu créer de sonde pour ce back-end. Cela se produit généralement quand le nom de domaine complet du back-end n’a pas été entré correctement. 

**Cause :** Si le pool de back-ends est de type Adresse IP/Nom de domaine complet ou App Service, Application Gateway résout l’adresse IP du nom de domaine complet entré à l’aide du système DNS (Domain Name System) (personnalisé ou Azure par défaut) et tente de se connecter au serveur sur le port TCP indiqué dans les paramètres HTTP. Toutefois, si ce message s’affiche, cela peut indiquer qu’Application Gateway n’a pas pu résoudre correctement l’adresse IP du nom de domaine complet entré.

**Résolution :**

1.  Vérifiez que le nom de domaine complet entré dans le pool de back-ends est correct et qu’il s’agit d’un domaine public, puis essayez de le résoudre à partir de votre machine locale.

1.  Si vous réussissez à résoudre l’adresse IP, il y a peut-être un problème avec la configuration DNS dans le réseau virtuel.

1.  Vérifiez si le réseau virtuel est configuré avec un serveur DNS personnalisé. Si c’est le cas, examinez le serveur DNS pour comprendre pourquoi il ne parvient pas à résoudre l’adresse IP du nom de domaine complet spécifié.

1.  Si vous utilisez le DNS par défaut d’Azure, vérifiez dans votre registre d’inscription des noms de domaine que le mappage approprié avec un enregistrement A ou un enregistrement CNAME a été effectué.

1.  Si le domaine est privé ou interne, essayez de le résoudre à partir d’une machine virtuelle dans le même réseau virtuel. Si vous pouvez le résoudre, redémarrez Application Gateway et vérifiez de nouveau. Pour redémarrer Application Gateway, vous devez l’[arrêter](/powershell/module/azurerm.network/stop-azurermapplicationgateway?view=azurermps-6.13.0) et le [démarrer](/powershell/module/azurerm.network/start-azurermapplicationgateway?view=azurermps-6.13.0) à l’aide des commandes PowerShell correspondantes (cliquez sur les liens fournis ici pour plus d’informations).

#### <a name="tcp-connect-error"></a>Erreur de connexion TCP

**Message :** Application Gateway n’a pas pu se connecter au serveur back-end.
Vérifiez que le serveur back-end répond sur le port utilisé pour la sonde.
Vérifiez également qu’aucun NSG, UDR ou pare-feu ne bloque l’accès à l’adresse IP et au port de ce serveur back-end

**Cause :** Après la phase de résolution du DNS, Application Gateway tente de se connecter au serveur back-end sur le port TCP configuré dans les paramètres HTTP. Si Application Gateway ne parvient pas à établir une session TCP sur le port spécifié, la sonde est marquée avec l’état Non sain, avec ce message.

**Solution :** Si cette erreur se produit, effectuez les étapes suivantes :

1.  Essayez de vous connecter au serveur back-end sur le port indiqué dans les paramètres HTTP à l’aide d’un navigateur ou de PowerShell. Par exemple, exécutez la commande suivante : `Test-NetConnection -ComputerName
    www.bing.com -Port 443`

1.  Si le port indiqué n’est pas le port souhaité, entrez le numéro de port approprié à utiliser par Application Gateway pour se connecter au serveur back-end.

1.  Si vous ne réussissez pas non plus à vous connecter sur le port à partir de votre machine locale :

    a.  Vérifiez les paramètres de groupe de sécurité réseau (NSG, network security group) de la carte réseau et du sous-réseau du serveur back-end, et vérifiez si les connexions entrantes sont autorisées sur le port configuré. Si ces connexions ne sont pas autorisées, créez une règle pour les autoriser. Pour savoir comment créer des règles NSG, consultez cette [page de documentation](../virtual-network/tutorial-filter-network-traffic.md#create-security-rules).

    b.  Vérifiez si les paramètres NSG du sous-réseau d’Application Gateway autorisent le trafic public et privé sortant, pour que la connexion puisse être établie. Consultez la page de documentation indiquée à l’étape 3a pour en savoir plus sur la création de règles NSG.
    ```azurepowershell
            $vnet = Get-AzVirtualNetwork -Name "vnetName" -ResourceGroupName "rgName"
            Get-AzVirtualNetworkSubnetConfig -Name appGwSubnet -VirtualNetwork $vnet
    ```

    c.  Vérifiez les paramètres d’Application Gateway relatifs aux routes définies par l’utilisateur (UDR, user-defined route) ainsi que le sous-réseau du serveur back-end pour identifier d’éventuelles anomalies de routage. Assurez-vous que l’UDR ne détourne pas le trafic du sous-réseau du back-end. Vérifiez, par exemple, les routes vers les appliances virtuelles du réseau ou les routes par défaut annoncées sur le sous-réseau d’Application Gateway par le biais d’une connexion Azure ExpressRoute et/ou VPN.

    d.  Vous pouvez vérifier les règles et routes effectives d’une carte réseau à l’aide des commandes PowerShell suivantes :
    ```azurepowershell
            Get-AzEffectiveNetworkSecurityGroup -NetworkInterfaceName "nic1" -ResourceGroupName "testrg"
            Get-AzEffectiveRouteTable -NetworkInterfaceName "nic1" -ResourceGroupName "testrg"
    ```
1.  Si vous ne constatez pas d’erreur au niveau du NSG ou de l’UDR, examinez votre serveur back-end pour identifier d’éventuels problèmes d’application empêchant les clients d’établir une session TCP sur les ports configurés. Voici quelques points à vérifier :

    a.  Ouvrez une invite de commandes (Win+R-\> cmd), entrez `netstat`, puis appuyez sur Entrée.

    b.  Vérifiez que le serveur écoute sur le port configuré. Par exemple :
    ```
            Proto Local Address Foreign Address State PID
            TCP 0.0.0.0:80 0.0.0.0:0 LISTENING 4
    ```
    c.  Si le serveur n’écoute pas sur le port configuré, vérifiez les paramètres de votre serveur web. Vérifiez, par exemple, les liaisons de site dans IIS, le bloc serveur dans NGINX et l’hôte virtuel dans Apache.

    d.  Vérifiez les paramètres de pare-feu du système d’exploitation pour vous assurer que le trafic entrant vers le port est autorisé.

#### <a name="http-status-code-mismatch"></a>Non-correspondance du code d’état HTTP

**Message :** Le code d\'état de la réponse HTTP du back-end ne correspond pas au paramètre de la sonde. Attendu :{HTTPStatusCode0} Reçu :{HTTPStatusCode1}.

**Cause :** Une fois que la connexion TCP a été établie et que la négociation TLS est terminée (si TLS est activé), Application Gateway envoie la sonde sous forme de requête HTTP GET au serveur back-end. Comme décrit plus haut, la sonde par défaut est envoyée à \<protocol\>://127.0.0.1:\<port\>/ et considère comme intègre les codes d’état de la réponse compris entre 200 et 399. Si le serveur retourne un autre code d’état, il est marqué comme Non sain, avec ce message.

**Solution :** Selon le code de réponse du serveur back-end, effectuez les étapes appropriées parmi les suivantes. Quelques codes d’état courants sont décrits ici :

| **Error** | **Actions** |
| --- | --- |
| Non-correspondance du code d’état de la sonde : Réponse 401 reçue | Vérifiez si le serveur back-end nécessite une authentification. Les sondes Application Gateway ne peuvent pas transmettre d’informations d’identification pour l’authentification. Autorisez la réponse \"HTTP 401\" dans une correspondance de code d’état de la sonde ou configurez la sonde sur un chemin où le serveur ne nécessite pas d’authentification. | |
| Non-correspondance du code d’état de la sonde : Réponse 403 reçue | Accès interdit. Vérifiez que l’accès au chemin est autorisé sur le serveur back-end. | |
| Non-correspondance du code d’état de la sonde : Réponse 404 reçue | Page introuvable. Vérifiez si le chemin du nom d’hôte est accessible sur le serveur back-end. Affectez au paramètre du chemin ou du nom d’hôte une valeur accessible. | |
| Non-correspondance du code d’état de la sonde : Réponse 405 reçue | Les demandes de sondage d’Application Gateway utilisent la méthode HTTP GET. Vérifiez que votre serveur autorise cette méthode. | |
| Non-correspondance du code d’état de la sonde : Réponse 500 reçue | Erreur interne du serveur. Vérifiez l’intégrité du serveur back-end et vérifiez si les services sont en cours d’exécution. | |
| Non-correspondance du code d’état de la sonde : Réponse 503 reçue | Service indisponible. Vérifiez l’intégrité du serveur back-end et vérifiez si les services sont en cours d’exécution. | |

Si vous pensez que la réponse est légitime et que vous souhaitez qu’Application Gateway considère d’autres codes d’état comme sains, vous pouvez créer une sonde personnalisée. Cette approche est utile dans les situations où le site web back-end nécessite une authentification. Les demandes de sondage échouent, car elles ne contiennent pas d’informations d’identification d’utilisateur. Un code d’état HTTP 401 est alors retourné par le serveur back-end.

Pour créer une sonde personnalisée, effectuez les étapes décrites [ici](./application-gateway-create-probe-portal.md).

#### <a name="http-response-body-mismatch"></a>Non-correspondance du corps de la réponse HTTP

**Message :** Le corps de la réponse HTTP du back-end ne correspond pas au paramètre de la sonde. Le corps de la réponse reçue ne contient pas l’élément {string}.

**Cause :** Quand vous créez une sonde personnalisée, vous pouvez marquer un serveur back-end comme Sain en mettant en correspondance une chaîne du corps de la réponse. Par exemple, configurez Application Gateway pour accepter la mise en correspondance de la chaîne « non autorisé ». Si la réponse du serveur back-end à la demande de sondage contient la chaîne **non autorisé** , le serveur est marqué comme Sain. Sinon, il est marqué comme Non sain, avec ce message.

**Solution :** Pour résoudre ce problème, effectuez les étapes suivantes :

1.  Accédez au serveur back-end localement ou à partir d’une machine cliente sur le chemin de la sonde et examinez le corps de la réponse.

1.  Vérifiez que le corps de la réponse dans la configuration de la sonde personnalisée d’Application Gateway correspond à ce qui a été configuré.

1.  Si ce n’est pas le cas, modifiez la configuration de la sonde en indiquant la valeur de chaîne correcte à accepter.

Pour en savoir plus sur la correspondance des sondes d’Application Gateway, [consultez cette section](./application-gateway-probe-overview.md#probe-matching).

>[!NOTE]
> Pour tous les messages d’erreur liés au protocole TLS, consultez la page de [présentation de TLS](ssl-overview.md) afin d’en savoir plus sur le comportement SNI et les différences entre les références SKU v1 et v2.


#### <a name="backend-server-certificate-invalid-ca"></a>Autorité de certification non valide pour le certificat du serveur back-end

**Message :** Le certificat de serveur utilisé par le serveur back-end n’est pas signé par une autorité de certification reconnue. Autorisez le back-end sur l’instance Application Gateway en chargeant le certificat racine du certificat de serveur utilisé par le back-end.

**Cause :** Le chiffrement SSL de bout en bout avec Application Gateway v2 implique la vérification du certificat du serveur back-end pour garantir que le serveur est sain.
Pour qu’un certificat TLS/SSL soit approuvé, ce certificat utilisé par le serveur back-end doit être émis par l’une des autorités de certification figurant dans le magasin de certificats approuvés d’Application Gateway. Si le certificat n’a pas été émis par une autorité de certification approuvée (ce qui est le cas des certificats auto-signés, par exemple), les utilisateurs doivent charger le certificat de l’émetteur dans Application Gateway.

**Solution :** Effectuez les étapes suivantes pour exporter et charger le certificat racine approuvé dans Application Gateway. (Ces étapes sont destinées aux clients Windows.)

1.  Connectez-vous à la machine qui héberge votre application.

1.  Appuyez sur les touches Win+R ou cliquez avec le bouton droit sur le bouton **Démarrer** , puis sélectionnez **Exécuter**.

1.  Entrez `certmgr.msc`, puis appuyez sur Entrée. Vous pouvez également ouvrir le gestionnaire de certificats à partir du menu **Démarrer**.

1.  Localisez le certificat (généralement sous `\Certificates - Current User\\Personal\\Certificates\`), et ouvrez-le.

1.  Sélectionnez le certificat racine, puis sélectionnez **Afficher le certificat**.

1.  Dans les propriétés du certificat, sélectionnez l’onglet **Détails**.

1.  Dans l’onglet **Détails** , sélectionnez l’option **Copier dans un fichier** , puis enregistrez le fichier au format X.509 encodé en base 64 (.CER).

1.  Ouvrez la page des **paramètres** HTTP d’Application Gateway dans le portail Azure.

1. Ouvrez les paramètres HTTP, sélectionnez **Ajouter un certificat** et recherchez le fichier de certificat que vous venez d’enregistrer.

1. Sélectionnez **Enregistrer** pour enregistrer les paramètres HTTP.

Vous pouvez aussi exporter le certificat racine à partir d’une machine cliente en accédant directement au serveur (sans passer par Application Gateway) à l’aide d’un navigateur et en exportant le certificat racine à partir du navigateur.

Pour plus d’informations sur l’extraction et le chargement de certificats racines approuvés dans Application Gateway, consultez [Exporter le certificat racine approuvé (pour le SKU v2)](./certificates-for-backend-authentication.md#export-trusted-root-certificate-for-v2-sku).

#### <a name="trusted-root-certificate-mismatch"></a>Non-correspondance du certificat racine approuvé

**Message :** Le certificat racine du certificat de serveur utilisé par le serveur back-end ne correspond pas au certificat racine approuvé qui a été ajouté dans Application Gateway. Assurez-vous d’ajouter le certificat racine approprié pour faire figurer le serveur back-end dans la liste approuvée

**Cause :** Le chiffrement SSL de bout en bout avec Application Gateway v2 implique la vérification du certificat du serveur back-end pour garantir que le serveur est sain.
Pour qu’un certificat TLS/SSL soit approuvé, le certificat du serveur back-end doit être émis par l’une des autorités de certification figurant dans le magasin de certificats approuvés d’Application Gateway. Si le certificat n’a pas été émis par une autorité de certification approuvée (ce qui est le cas des certificats auto-signés, par exemple), les utilisateurs doivent charger le certificat de l’émetteur dans Application Gateway.

Le certificat qui a été chargé dans les paramètres HTTP d’Application Gateway doit correspondre au certificat racine du certificat du serveur back-end.

**Solution :** Si vous voyez ce message d’erreur, cela signifie que le certificat chargé dans Application Gateway ne correspond pas à celui qui a été chargé sur le serveur back-end.

Effectuez les étapes 1-11 de la méthode précédente pour charger le certificat racine approuvé correct dans Application Gateway.

Pour plus d’informations sur l’extraction et le chargement de certificats racines approuvés dans Application Gateway, consultez [Exporter le certificat racine approuvé (pour le SKU v2)](./certificates-for-backend-authentication.md#export-trusted-root-certificate-for-v2-sku).
> [!NOTE]
> Cette erreur peut également se produire si le serveur back-end ne passe pas la chaîne complète du certificat, y compris les éléments Root > Intermediate (le cas échéant) > Leaf pendant la négociation TLS. Pour vérifier ce point, utilisez les commandes OpenSSL à partir de n’importe quel client et connectez-vous au serveur back-end avec les paramètres configurés dans la sonde d’Application Gateway.

Par exemple :
```
OpenSSL> s_client -connect 10.0.0.4:443 -servername www.example.com -showcerts
```
Si la sortie n’affiche pas la chaîne complète du certificat retourné, réexportez le certificat avec la chaîne complète incluant le certificat racine. Configurez ce certificat sur votre serveur back-end. 

```
  CONNECTED(00000188)\
  depth=0 OU = Domain Control Validated, CN = \*.example.com\
  verify error:num=20:unable to get local issuer certificate\
  verify return:1\
  depth=0 OU = Domain Control Validated, CN = \*.example.com\
  verify error:num=21:unable to verify the first certificate\
  verify return:1\
  \-\-\-\
  Certificate chain\
   0 s:/OU=Domain Control Validated/CN=*.example.com\
     i:/C=US/ST=Arizona/L=Scottsdale/O=GoDaddy.com, Inc./OU=http://certs.godaddy.com/repository//CN=Go Daddy Secure Certificate Authority - G2\
  \-----BEGIN CERTIFICATE-----\
  xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx\
  \-----END CERTIFICATE-----
```

#### <a name="backend-certificate-invalid-common-name-cn"></a>Nom commun (CN) non valide pour le certificat du back-end

**Message :** Le nom commun (CN) du certificat du back-end ne correspond pas à l’en-tête d’hôte de la sonde.

**Cause :** Application Gateway vérifie si le nom d’hôte spécifié dans les paramètres HTTP du back-end correspond au nom commun (CN) présenté par le certificat TLS/SSL du serveur back-end. Ce comportement (v2) s’applique aux références SKU Standard_v2 et WAF_v2. L’indication du nom du serveur (SNI, Server Name Indication) (v1) des références SKU Standard et WAF est définie comme le nom de domaine complet dans le pool d’adresses back-end. Pour plus d’informations sur le comportement SNI et les différences entre les références SKU v1 et v2, consultez [Présentation de la terminaison TLS et du chiffrement TLS de bout en bout avec Application Gateway](ssl-overview.md).

Dans la référence SKU v2, en présence d’une sonde par défaut (aucune sonde personnalisée n’a été configurée et associée), le SNI est défini à partir du nom d’hôte mentionné dans les paramètres HTTP. Si l’option « Choisir un nom d’hôte à partir d’une adresse back-end » est activée dans les paramètres HTTP et que le pool d’adresses back-end contient un nom de domaine complet valide, ce paramètre est appliqué.

Si une sonde personnalisée est associée aux paramètres HTTP, le SNI est défini à partir du nom d’hôte mentionné dans la configuration correspondante. Si l’option **Choisir le nom d’hôte dans les paramètres HTTP de back-end** est sélectionnée dans la sonde personnalisée, le SNI est défini à partir du nom d’hôte mentionné dans les paramètres HTTP.

Si l’option **Choisir un nom d’hôte à partir d’une adresse back-end** est activée dans les paramètres HTTP, le pool d’adresses back-end doit contenir un nom de domaine complet valide.

Si vous voyez ce message d’erreur, cela signifie que le nom commun (CN) du certificat du back-end ne correspond pas au nom d’hôte configuré dans la sonde personnalisée ou dans les paramètres HTTP (quand l’option **Choisir le nom d’hôte dans les paramètres HTTP de back-end** est activée). Si vous utilisez une sonde par défaut, le nom d’hôte est **127.0.0.1**. Si ce n’est pas la valeur souhaitée, vous devez créer une sonde personnalisée et l’associer aux paramètres HTTP.

**Solution :**

Pour résoudre ce problème, procédez comme suit.

Pour Windows :

1.  Connectez-vous à la machine qui héberge votre application.

1.  Appuyez sur les touches Win+R ou cliquez avec le bouton droit sur le bouton **Démarrer** , puis sélectionnez **Exécuter**.

1.  Entrez **certmgr.msc** et appuyez sur Entrée. Vous pouvez également ouvrir le gestionnaire de certificats à partir du menu **Démarrer**.

1.  Localisez le certificat (généralement sous `\Certificates - Current User\\Personal\\Certificates`), et ouvrez-le.

1.  Dans l’onglet **Détails** , vérifiez l’objet du certificat sous **Objet**.

1.  Vérifiez le nom commun du certificat indiqué dans les détails et entrez le même nom dans le champ du nom d’hôte de la sonde personnalisée ou des paramètres HTTP (quand l’option **Choisir le nom d’hôte dans les paramètres HTTP de back-end** est activée). Si ce n’est pas le nom d’hôte souhaité pour votre site web, vous devez obtenir un certificat pour ce domaine ou entrer le nom d’hôte correct dans la configuration de la sonde personnalisée ou des paramètres HTTP.

Pour Linux avec OpenSSL :

1.  Exécutez cette commande dans OpenSSL :
    ```
    openssl x509 -in certificate.crt -text -noout
    ```

2.  Dans les propriétés affichées, recherchez le nom commun du certificat et entrez le même nom dans le champ du nom d’hôte des paramètres HTTP. Si ce n’est pas le nom d’hôte souhaité pour votre site web, vous devez obtenir un certificat pour ce domaine ou entrer le nom d’hôte correct dans la configuration de la sonde personnalisée ou des paramètres HTTP.

#### <a name="backend-certificate-is-invalid"></a>Le certificat du back-end n’est pas valide

**Message :** Le certificat du back-end n’est pas valide. La date actuelle ne s’inscrit pas dans la plage de dates définie par les options \"Valide à partir du\" et \"Valide jusqu’au\" pour le certificat.

**Cause :** Chaque certificat a une période de validité, et la connexion HTTPS est sécurisée uniquement si le certificat TLS/SSL du serveur est valide. La date actuelle doit être comprise dans la plage de dates délimitée par **Valide à partir du** et **Valide jusqu’au**. Si ce n’est pas le cas, le certificat est considéré comme non valide, ce qui entraîne un problème de sécurité. Application Gateway marque alors le serveur back-end comme Non sain.

**Solution :** Si votre certificat TLS/SSL est arrivé à expiration, renouvelez le certificat auprès de votre fournisseur et mettez à jour les paramètres du serveur avec le nouveau certificat. S’il s’agit d’un certificat auto-signé, vous devez générer un certificat valide et charger le certificat racine dans les paramètres HTTP d’Application Gateway. Pour ce faire, procédez comme suit :

1.  Ouvrez les paramètres HTTP d’Application Gateway dans le portail.

1.  Sélectionnez le paramètre associé au certificat arrivé à expiration, sélectionnez **Ajouter un certificat** et ouvrez le nouveau fichier de certificat.

1.  Supprimez l’ancien certificat en cliquant sur l’icône **Supprimer** à côté du certificat, puis sélectionnez **Enregistrer**.

#### <a name="certificate-verification-failed"></a>Échec de la vérification du certificat

**Message :** La validité du certificat du back-end n’a pas pu être vérifiée. Pour en déterminer la raison, examinez le message associé au code d’erreur {errorCode} dans les diagnostics OpenSSL

**Cause :** Cette erreur se produit quand Application Gateway n’est pas en mesure de vérifier la validité du certificat.

**Solution :** Pour résoudre ce problème, vérifiez que le certificat sur votre serveur a été créé correctement. Par exemple, utilisez [OpenSSL](https://www.openssl.org/docs/man1.0.2/man1/verify.html) pour vérifier le certificat et ses propriétés, puis réessayez de charger le certificat dans les paramètres HTTP d’Application Gateway.

<a name="backend-health-status-unknown"></a>État d’intégrité des back-ends : Inconnu
-------------------------------
Si l’état d’intégrité des back-ends est présenté comme Inconnu, la vue du portail est semblable à la capture suivante :

![Intégrité des back-ends dans Application Gateway - Inconnu](./media/application-gateway-backend-health-troubleshooting/appgwunknown.png)

Ce comportement peut être dû à différentes raisons :

1.  Le NSG sur le sous-réseau d’Application Gateway bloque l’accès entrant sur les ports 65503-65534 (SKU v1) ou 65200-65535 (SKU v2) à partir d’« Internet ».
1.  L’UDR sur le sous-réseau d’Application Gateway est défini sur la route par défaut (0.0.0.0/0) et le tronçon suivant n’est pas spécifié comme « Internet ».
1.  La route par défaut est annoncée par une connexion ExpressRoute/VPN à un réseau virtuel avec le protocole BGP.
1.  Le serveur DNS personnalisé est configuré sur un réseau virtuel qui ne peut pas résoudre les noms de domaine publics.
1.  Application Gateway se trouve dans un état Non sain.

**Solution :**

1.  Vérifiez si votre NSG bloque l’accès aux ports 65503-65534 (SKU v1) ou 65200-65535 (SKU v2) à partir d’ **Internet**  :

    a.  Dans Application Gateway, sous l’onglet **Vue d’ensemble** , sélectionnez le lien **Réseau/sous-réseau virtuel**.

    b.  Dans l’onglet **Sous-réseaux** de votre réseau virtuel, sélectionnez le sous-réseau où Application Gateway a été déployé.

    c.  Vérifiez si un NSG est configuré.

    d.  Si un NSG est configuré, recherchez cette ressource NSG dans l’onglet **Recherche** ou sous **Toutes les ressources**.

    e.  Dans la section **Règles de trafic entrant** , ajoutez une règle de trafic entrant pour autoriser la plage de ports de destination 65503-65534 pour la référence SKU v1 ou 65200-65535 pour la référence SKU v2, avec le paramètre **Source** défini sur **N’importe laquelle** ou **Internet**.

    f.  Sélectionnez **Enregistrer** et vérifiez que le back-end apparaît comme Sain. Vous pouvez également faire cette vérification à l’aide de [PowerShell/CLI](../virtual-network/manage-network-security-group.md).

1.  Vérifiez si votre UDR a une route par défaut (0.0.0.0/0) avec le tronçon suivant non défini comme **Internet**  :
    
    a.  Effectuez les étapes 1a et 1b pour identifier votre sous-réseau.

    b.  Vérifiez si des UDR sont configurés. Si c’est le cas, recherchez la ressource dans la barre de recherche ou sous **Toutes les ressources**.

    c.  Vérifiez la présence de routes par défaut (0.0.0.0/0) avec le tronçon suivant non défini comme **Internet**. Si le paramètre est **Appliance virtuelle** ou **Passerelle de réseau virtuel** , assurez-vous que l’appliance virtuelle ou l’appareil local est en mesure de rediriger correctement le paquet vers la destination Internet sans le modifier.

    d.  Si ce n’est pas le cas, définissez le tronçon suivant sur **Internet** , sélectionnez **Enregistrer** , puis vérifiez l’intégrité du serveur back-end.

1.  Une route par défaut annoncée par la connexion ExpressRoute/VPN au réseau virtuel avec le protocole BGP :

    a.  Si vous avez une connexion ExpressRoute/VPN au réseau virtuel via le protocole BGP et que vous annoncez une route par défaut, assurez-vous que le paquet est redirigé correctement vers la destination Internet sans être modifié. Vous pouvez vérifier ce point à l’aide de l’option **Résoudre les problèmes de connexion** disponible dans le portail Application Gateway.

    b.  Choisissez la destination manuellement (n’importe quelle adresse IP routable sur Internet), par exemple 1.1.1.1. Définissez le port de destination de votre choix, puis vérifiez la connectivité.

    c.  Si le tronçon suivant est une passerelle de réseau virtuel, il y a peut-être une route par défaut annoncée par ExpressRoute ou VPN.

1.  Si un serveur DNS personnalisé est configuré sur le réseau virtuel, vérifiez que le ou les serveurs peuvent résoudre les domaines publics. La résolution des noms de domaines publics peut être nécessaire dans les scénarios où Application Gateway doit accéder à des domaines externes tels que des serveurs OCSP ou pour vérifier l’état de révocation du certificat.

1.  Pour vérifier si Application Gateway est sain et en cours d’exécution, accédez à l’option **Intégrité des ressources** dans le portail et vérifiez que l’état est **Sain**. Si l’état est **Non sain** ou **Dégradé** , [contactez le support](https://azure.microsoft.com/support/options/).

<a name="next-steps"></a>Étapes suivantes
----------

Apprenez-en davantage sur [les diagnostics et la journalisation dans Application Gateway](./application-gateway-diagnostics.md).
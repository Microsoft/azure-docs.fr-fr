---
title: Utiliser le portail Azure pour résoudre des problèmes liés à Azure Stack Edge Pro avec GPU | Microsoft Docs
description: Décrit comment résoudre les problèmes liés à Azure Stack Edge Pro avec GPU.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: troubleshooting
ms.date: 10/07/2020
ms.author: alkohli
ms.openlocfilehash: d07d9dccb0aa273f79b251f2ffb4a920f3cac2e7
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96447613"
---
# <a name="troubleshoot-issues-on-your-azure-stack-edge-pro-gpu-device"></a>Résoudre les problèmes survenant sur votre appareil Azure Stack Edge Pro avec GPU 

<!--[!INCLUDE [applies-to-skus](../../includes/azure-stack-edge-applies-to-all-sku.md)]-->

Cet article explique comment résoudre des problèmes survenant sur votre appareil Azure Stack Edge Pro avec GPU. 


## <a name="run-diagnostics"></a>Exécuter les diagnostics

Pour diagnostiquer et corriger les erreurs d’appareil, vous pouvez exécuter les tests de diagnostics. Pour exécuter ces tests, procédez comme suit dans l’interface utilisateur web locale de votre appareil.

1. Dans l’interface utilisateur web locale, accédez à **Dépannage > Tests de diagnostic**. Sélectionnez le test que vous souhaitez exécuter, puis cliquez sur **Exécuter le test**. Cette opération exécute les tests pour diagnostiquer les éventuels problèmes sur votre réseau, appareil, proxy web, horodatage ou paramètres cloud. Vous êtes averti que des tests sont en cours d’exécution sur l’appareil.

    ![Sélectionner les tests ](media/azure-stack-edge-gpu-troubleshoot/run-diag-1.png)
 
2. Une fois les tests terminés, les résultats correspondants s’affichent. 

    ![Afficher les résultats des tests](media/azure-stack-edge-gpu-troubleshoot/run-diag-2.png)

    Si un test échoue, l’URL de l’action recommandée vous est présentée. Sélectionnez cette URL pour visualiser l’action recommandée.
 
    ![Passer en revue les avertissements des tests ayant échoué](media/azure-stack-edge-j-series-troubleshoot/run-diag-3.png)


## <a name="collect-support-package"></a>Collecter un package de support

Un package de journaux d’activité contient tous les journaux d’activité pertinents qui peuvent aider l’équipe du Support Microsoft à résoudre les problèmes des appareils. Vous pouvez générer un package de journaux par le biais de l’interface utilisateur web locale.

Pour collecter un package de support, procédez comme suit. 

1. Dans l’interface utilisateur web locale, accédez à **Dépannage > Support**. Sélectionnez **Créer un package de support**. Le système commence à collecter le package de support. La collection du package peut nécessiter plusieurs minutes.

    ![Sélectionner Ajouter un utilisateur](media/azure-stack-edge-gpu-troubleshoot/collect-logs-1.png)
 
2. Une fois le package de support créé, sélectionnez **Télécharger le package de support**. Un package compressé est téléchargé dans le chemin d’accès que vous avez choisi. Vous pouvez décompresser le package et visualiser les fichiers journaux système.

    ![Sélectionner Ajouter un utilisateur 2](media/azure-stack-edge-gpu-troubleshoot/collect-logs-2.png)

## <a name="gather-advanced-security-logs"></a>Collecter les journaux de sécurité avancée

Les journaux de sécurité avancée peuvent être des journaux d’intrusion logicielle ou d’intrusion matérielle pour votre appareil Azure Stack Edge Pro.

### <a name="software-intrusion-logs"></a>Journaux d’intrusion logicielle

L’intrusion logicielle ou les journaux de pare-feu par défaut sont collectés pour le trafic entrant et sortant. 

- Lorsque l’appareil est mis en image à l’usine, la journalisation par défaut du pare-feu est activée. Ces journaux sont regroupés par défaut dans le package de support lorsque vous créez ce type de package via l’interface utilisateur locale ou l’interface Windows PowerShell de l’appareil.

- Si seuls les journaux de pare-feu sont demandés dans le package de support pour vérifier une intrusion logicielle (NW) dans l’appareil, utilisez l’option `-Include FirewallLog` lors de la création du package de support. 

- Si aucune option d’inclusion particulière n’est fournie, le journal du pare-feu est inclus par défaut dans le package de support.

- Dans le package de support, le journal du pare-feu est le fichier `pfirewall.log`, il se trouve dans le dossier racine. Voici un exemple du journal d’intrusion logicielle de l’appareil Azure Stack Edge Pro. 

    ```
    #Version: 1.5
    #Software: Microsoft Windows Firewall
    #Time Format: Local
    #Fields: date time action protocol src-ip dst-ip src-port dst-port size tcpflags tcpsyn tcpack tcpwin icmptype icmpcode info path
    
    2019-11-06 12:35:19 DROP UDP 5.5.3.197 224.0.0.251 5353 5353 59 - - - - - - - RECEIVE
    2019-11-06 12:35:19 DROP UDP fe80::3680:dff:fe01:9e88 ff02::fb 5353 5353 89 - - - - - - - RECEIVE
    2019-11-06 12:35:19 DROP UDP fe80::3680:dff:fe01:9e88 ff02::fb 5353 5353 89 - - - - - - - RECEIVE
    2019-11-06 12:35:19 DROP UDP fe80::3680:dff:fe01:9e88 ff02::fb 5353 5353 89 - - - - - - 
    2019-11-06 12:35:19 DROP UDP fe80::3680:dff:fe01:9d87 ff02::fb 5353 5353 79 - - - - - - - RECEIVE
    2019-11-06 12:35:19 DROP UDP 5.5.3.193 224.0.0.251 5353 5353 59 - - - - - - - RECEIVE
    2019-11-06 12:35:19 DROP UDP fe80::3680:dff:fe08:20d5 ff02::fb 5353 5353 89 - - - - - - - RECEIVE
    2019-11-06 12:35:19 DROP UDP fe80::3680:dff:fe08:20d5 ff02::fb 5353 5353 89 - - - - - - - RECEIVE
    2019-11-06 12:35:19 DROP UDP fe80::3680:dff:fe01:9e8b ff02::fb 5353 5353 89 - - - - - - - RECEIVE
    2019-11-06 12:35:19 DROP UDP fe80::3680:dff:fe01:9e8b ff02::fb 5353 5353 89 - - - - - - - RECEIVE
    2019-11-06 12:35:19 DROP UDP 5.5.3.33 224.0.0.251 5353 5353 59 - - - - - - - RECEIVE
    2019-11-06 12:35:19 DROP UDP fe80::3680:dff:fe01:9e8b ff02::fb 5353 5353 89 - - - - - - - RECEIVE
    2019-11-06 12:35:19 DROP UDP fe80::3680:dff:fe01:9e8a ff02::fb 5353 5353 89 - - - - - - - RECEIVE
    2019-11-06 12:35:19 DROP UDP fe80::3680:dff:fe01:9e8b ff02::fb 5353 5353 89 - - - - - - - RECEIVE
    ```

### <a name="hardware-intrusion-logs"></a>Journaux d’intrusion matérielle

Pour détecter toute intrusion matérielle dans l’appareil, les événements de châssis, tels que l’ouverture ou la fermeture du châssis, sont consignés. 

- Le journal des événements système de l’appareil est lu à l’aide de l’applet de commande `racadm`. Ces événements sont ensuite filtrés pour les événements liés au châssis, dans un fichier `HWIntrusion.txt`.

- Pour récupérer uniquement le journal d’intrusion matérielle dans le package de support, utilisez l’option `-Include HWSelLog` lors de la création du package de support. 

- Si aucune option d’inclusion particulière n’est fournie, le journal d’intrusion matérielle est inclus par défaut dans le package de support.

- Dans le package de support, le journal d’intrusion matérielle est le fichier `HWIntrusion.txt`, il se trouve dans le dossier racine. Voici un exemple du journal d’intrusion matérielle de l’appareil Azure Stack Edge Pro. 

    ```
    09/04/2019 15:51:23 system Critical The chassis is open while the power is off.
    09/04/2019 15:51:30 system Ok The chassis is closed while the power is off.
    ```

## <a name="use-logs-to-troubleshoot"></a>Utiliser les journaux d’activité pour résoudre les problèmes

Toutes les erreurs rencontrées lors des processus de chargement et d’actualisation figurent dans les fichiers d’erreurs correspondants.

1. Pour visualiser les fichiers d’erreurs, accédez à votre partage, puis sélectionnez-le pour afficher le contenu. 


2. Sélectionnez le _dossier Microsoft Data Box Edge_. Ce dossier comporte deux sous-dossiers :

    - dossier Upload contenant les fichiers journaux des erreurs de chargement ;
    - dossier Refresh stockant les fichiers journaux des erreurs d’actualisation.

    Voici un exemple de fichier journal d’actualisation.

    ```
    <root container="test1" machine="VM15BS020663" timestamp="03/18/2019 00:11:10" />
    <file item="test.txt" local="False" remote="True" error="16001" />
    <summary runtime="00:00:00.0945320" errors="1" creates="2" deletes="0" insync="3" replaces="0" pending="9" />
    ``` 

3. Lorsque ce fichier signale une erreur (présentée dans cet exemple), notez le code de l’erreur, 16001 dans le cas présent. Recherchez ci-après la description de ce code d’erreur en fonction de la référence de l’erreur.

    [!INCLUDE [data-box-edge-edge-upload-error-reference](../../includes/data-box-edge-gateway-upload-error-reference.md)]

## <a name="use-error-lists-to-troubleshoot"></a>Utiliser les listes d’erreurs pour résoudre des problèmes

Les listes d’erreurs sont compilées à partir de scénarios identifiés, elles peuvent être utilisées pour le diagnostic automatique et la résolution des problèmes. 

## <a name="azure-resource-manager"></a>Azure Resource Manager

Voici les erreurs pouvant s’afficher lors de la configuration de l’accès d’Azure Resource Manager à votre appareil. 

| **Problème / Erreurs** |  **Résolution :** | 
|------------|-----------------|
|Problèmes d’ordre général|<li>[Vérifiez que l’appareil Edge est correctement configuré](#verify-the-device-is-configured-properly).<li> [Vérifiez que le client est correctement configuré](#verify-the-client-is-configured-properly)|
|Add-AzureRmEnvironment : Une erreur s’est produite lors de l’envoi de la requête.<br>À la ligne : 1 caractère : 1<br>+ Add-AzureRmEnvironment -Name Az3 -ARMEndpoint "https://management.dbe ...|Cette erreur signifie que votre appareil Azure Stack Edge Pro n’est pas accessible, ou qu’il n’est pas configuré correctement. Vérifiez que l’appareil Edge et le client sont configurés correctement. Pour obtenir de l’aide, consultez la ligne **Problèmes d’ordre général** dans ce tableau.|
|Le service a retourné une erreur. Pour plus d’informations, consultez InnerException : Le serveur a clos la connexion sous-jacente : Impossible d’établir une relation de confiance pour le canal sécurisé SSL/TLS. |   Cette erreur est probablement due à l’exécution incorrecte d’une ou de plusieurs étapes du scénario d’apport de votre propre certificat. Vous trouverez des conseils [ici](./azure-stack-edge-j-series-connect-resource-manager.md#step-2-create-and-install-certificates). |
|L’opération a retourné un code d’état non valide : « ServiceUnavailable ». <br> Response status code does not indicate success: 503 (Service indisponible). | Cette erreur peut être due à l’une de ces conditions.<li>ArmStsPool est à l’état d’arrêt.</li><li>L’un ou l’autre de ces sites web, Azure Resource Manager ou Services d’émission de jeton de sécurité, est hors service.</li><li>La ressource de cluster Azure Resource Manager est inactive.</li><br><strong>Remarque :</strong> Le redémarrage de l’appliance peut résoudre le problème, mais vous devez collecter le package de support afin de pouvoir le déboguer davantage.|
|AADSTS50126 : Nom d’utilisateur ou mot de passe non valide.<br>ID de suivi : 29317da9-52fc-4ba0-9778-446ae5625e5a<br>ID de corrélation : 1b9752c4-8cbf-4304-a714-8a16527410f4<br>Timestamp : 2019-11-15 09:21:57Z : Le serveur distant a retourné une erreur : (400) Requête incorrecte.<br>À la ligne : 1 caractère : 1 |Cette erreur peut être due à l’une de ces conditions.<li>Pour un nom d’utilisateur et un mot de passe non valides, vérifiez que le client a modifié le mot de passe depuis le portail Azure en suivant les étapes [ici](./azure-stack-edge-j-series-set-azure-resource-manager-password.md), et en utilisant le mot de passe correct.<li>Pour un ID de locataire non valide, l’ID de locataire est un identificateur unique fixe qui doit avoir la valeur `c0257de7-538f-415c-993a-1b87a031879d`</li>|
|connect-AzureRmAccount : AADSTS90056 : La ressource est désactivée ou n’existe pas. Vérifiez le code de votre application pour vous assurer que vous avez spécifié l’URL de ressource exacte pour la ressource à laquelle vous essayez d’accéder.<br>ID de trace : e19bdbc9-5dc8-4a74-85c3-ac6abdfda115<br>ID de corrélation : 75c8ef5a-830e-48b5-b039-595a96488ff9 Timestamp : 2019-11-18 07:00:51Z : Le serveur distant a retourné une erreur : (400) Incorrect |Les points de terminaison de la ressource utilisés dans la commande `Add-AzureRmEnvironment` sont incorrects.|
|Impossible d’accéder aux points de terminaison à partir du cloud.<br>Vérifiez que vous disposez d’une connexion réseau. Détail de l’erreur : HTTPSConnectionPool(host='management.dbg-of4k6suvm.microsoftdatabox.com', port=30005) : Nombre maximal de nouvelles tentatives dépassé avec l’URL :/metadata/endpoints?api-version=2015-01-01 (Provoquée par SSLError(SSLError("bad handshake: Error([('SSL routines', 'tls_process_server_certificate', 'certificate verify failed')],)",),)) |Cette erreur apparaît principalement dans un environnement Mac/Linux et est due aux problèmes suivants :<li>Un certificat au format PEM n’a pas été ajouté au magasin de certificats Python.</li> |

### <a name="verify-the-device-is-configured-properly"></a>Vérifier que l’appareil est configuré correctement

1. À partir de l’interface utilisateur locale, vérifiez que le réseau de l’appareil est correctement configuré.

2. Vérifiez que les certificats sont mis à jour pour tous les points de terminaison, comme indiqué [ici](azure-stack-edge-j-series-connect-resource-manager.md#step-2-create-and-install-certificates).

3. Récupérez le point de terminaison de gestion et de connexion d’Azure Resource Manager dans la page **Appareil** de l’interface utilisateur locale.

4. Vérifiez que l’appareil est activé et inscrit dans Azure.


### <a name="verify-the-client-is-configured-properly"></a>Vérifier que le client est configuré correctement

1. Vérifiez que la version PowerShell appropriée est installée, comme indiqué [ici](azure-stack-edge-j-series-connect-resource-manager.md#step-3-install-powershell-on-the-client).

2. Vérifiez que les modules PowerShell adéquats sont installés, comme indiqué [ici](azure-stack-edge-j-series-connect-resource-manager.md#step-4-set-up-azure-powershell-on-the-client).

3. Vérifiez que les points de terminaison d’Azure Resource Manager et ceux de connexion sont accessibles. Vous pouvez essayer d’effectuer un test ping sur les points de terminaison. Par exemple :

   `ping management.28bmdw2-bb9.microsoftdatabox.com`
   `ping login.28bmdw2-bb9.microsoftdatabox.com`
   
   S’ils ne sont pas joignables, ajoutez des entrées de fichier DNS / hôte, comme indiqué [ici](azure-stack-edge-j-series-connect-resource-manager.md#step-5-modify-host-file-for-endpoint-name-resolution).
   
4. Vérifiez que les certificats clients sont installés, comme indiqué [ici](azure-stack-edge-j-series-connect-resource-manager.md#import-certificates-on-the-client-running-azure-powershell).

5. Si le client utilise PowerShell, vous devez activer la préférence de débogage pour afficher les messages détaillés en exécutant cette commande PowerShell. 

    `$debugpreference = "continue"`

## <a name="blob-storage-on-device"></a>Stockage d’objets blob sur un appareil 

Voici les erreurs liées au stockage d’objets blob sur un appareil Azure Stack Edge Pro/Data Box Gateway.

| **Problème / Erreurs** |  **Résolution :** | 
|--------------------|-----------------|
|Unable to retrieve child resources. (Impossible de récupérer des ressources enfants.) The value for one of the HTTP headers is not in the correct format. (Le format de la valeur d’un des en-têtes HTTP est incorrect.)| Dans le menu **Modifier**, sélectionnez **Target Azure Stack APIs** (Cibler des API Azure Stack). Ensuite, redémarrez l’Explorateur Stockage Azure.|
|getaddrinfo ENOTFOUND <accountname>.blob.<serialnumber>.microsoftdatabox.com|Vérifiez que le nom de point de terminaison `<accountname>.blob.<serialnumber>.microsoftdatabox.com` est ajouté au fichier hosts sur ce chemin : `C:\Windows\System32\drivers\etc\hosts` sur Windows, ou `/etc/hosts` sur Linux.|
|Unable to retrieve child resources. (Impossible de récupérer des ressources enfants.)<br> Détails : certificat auto-signé |Importez le certificat SSL pour votre appareil dans l’Explorateur Stockage Azure : <ol><li>Téléchargez le certificat à partir du Portail Azure. Pour plus d’informations, consultez [Télécharger le certificat](../databox/data-box-deploy-copy-data-via-rest.md#download-certificate).</li><li>Dans le menu **Modifier**, sélectionnez Certificats SSL, puis sélectionnez **Importer les certificats**.</li></ol>|
|La commande AzCopy semble ne plus répondre pendant une minute avant d’afficher cette erreur :<br>`Failed to enumerate directory https://… The remote name could not be resolved <accountname>.blob.<serialnumber>.microsoftdatabox.com`|Vérifiez que le nom de point de terminaison `<accountname>.blob.<serialnumber>.microsoftdatabox.com` est ajouté au fichier hosts sur : `C:\Windows\System32\drivers\etc\hosts`.|
|La commande AzCopy semble ne plus répondre pendant une minute avant d’afficher cette erreur :<br>`Error parsing source location. The underlying connection was closed: Could not establish trust relationship for the SSL/TLS secure channel`. |Importez le certificat SSL pour votre appareil dans le magasin de certificats du système. Pour plus d’informations, consultez [Télécharger le certificat](../databox/data-box-deploy-copy-data-via-rest.md#download-certificate).|
|La commande AzCopy semble ne plus répondre pendant 20 minutes avant d’afficher cette erreur :<br>`Error parsing source location https://<accountname>.blob.<serialnumber>.microsoftdatabox.com/<cntnr>. No such device or address`. |Vérifiez que le nom de point de terminaison `<accountname>.blob.<serialnumber>.microsoftdatabox.com` est ajouté au fichier hosts sur : `/etc/hosts`.|
|La commande AzCopy semble ne plus répondre pendant 20 minutes avant d’afficher cette erreur :<br>`Error parsing source location… The SSL connection could not be established`. |Importez le certificat SSL pour votre appareil dans le magasin de certificats du système. Pour plus d’informations, consultez [Télécharger le certificat](../databox/data-box-deploy-copy-data-via-rest.md#download-certificate).|
|La commande AzCopy semble ne plus répondre pendant 20 minutes avant d’afficher cette erreur :<br>`Error parsing source location https://<accountname>.blob.<serialnumber>.microsoftdatabox.com/<cntnr>. No such device or address`|Vérifiez que le nom de point de terminaison `<accountname>.blob.<serialnumber>.microsoftdatabox.com` est ajouté au fichier hosts sur : `/etc/hosts`.|
|La commande AzCopy semble ne plus répondre pendant 20 minutes avant d’afficher cette erreur : `Error parsing source location… The SSL connection could not be established`.|Importez le certificat SSL pour votre appareil dans le magasin de certificats du système. Pour plus d’informations, consultez [Télécharger le certificat](../databox/data-box-deploy-copy-data-via-rest.md#download-certificate).|
|The value for one of the HTTP headers is not in the correct format. (Le format de la valeur d’un des en-têtes HTTP est incorrect.)|La version installée de la bibliothèque Stockage Microsoft Azure pour Python n’est pas prise en charge par Data Box. Consultez les exigences relatives au Stockage Blob d’Azure Data Box pour connaître les versions prises en charge.|
|… [SSL: CERTIFICATE_VERIFY_FAILED] …| Avant d’exécuter Python, définissez la variable d’environnement REQUESTS_CA_BUNDLE sur le chemin du fichier de certificat SSL codé en Base64 (voir comment [Télécharger le certificat](../databox/data-box-deploy-copy-data-via-rest.md#download-certificate)). Par exemple :<br>`export REQUESTS_CA_BUNDLE=/tmp/mycert.cer`<br>`python`<br>Vous pouvez également ajouter le certificat au magasin de certificats du système, puis définir cette variable d’environnement sur le chemin de ce magasin. Par exemple, sur Ubuntu :<br>`export REQUESTS_CA_BUNDLE=/etc/ssl/certs/ca-certificates.crt`<br>`python`.|
|The connection times out. (La connexion arrive à expiration.)|Connectez-vous à l’appareil Azure Stack Edge Pro, puis vérifiez qu’il est déverrouillé. Chaque fois que l’appareil redémarre, il reste verrouillé jusqu’à ce qu’un utilisateur se connecte.|



## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur la façon de [Résoudre les problèmes d’activation d’appareil](azure-stack-edge-gpu-troubleshoot-activation.md).
---
title: Suivi et journalisation des événements Azure Data Box, Azure Data Box Heavy pour les ordres d’exportation | Microsoft Docs
description: Décrit comment suivre et journaliser les événements à divers stades de votre ordre d’exportation Azure Data Box et Azure Data Box Heavy.
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: article
ms.date: 07/10/2020
ms.author: alkohli
ms.openlocfilehash: 3a915ac8de83a5e183660ec4a3d05044eafff4a9
ms.sourcegitcommit: 2a8a53e5438596f99537f7279619258e9ecb357a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "94337506"
---
# <a name="tracking-and-event-logging-for-your-azure-data-box-and-azure-data-box-heavy-export-orders"></a>Suivi et journalisation des événements de vos ordres d’exportation Azure Data Box et Azure Data Box Heavy

Un ordre d’exportation Data Box ou Data Box Heavy passe par les étapes suivantes : ordre, configuration, copie des données, retour, et effacement des données. Lors de chaque étape de la commande, vous pouvez effectuer plusieurs actions pour contrôler l’accès à la commande, auditer les événements, suivre la commande et interpréter les différents journaux générés.

Cet article décrit en détail les différents mécanismes ou outils disponibles pour effectuer le suivi et l’audit d’un ordre d’exportation Data Box ou Data Box Heavy. Les informations de cet article s’appliquent à Data Box et à Data Box Heavy. Dans les sections suivantes, toutes les références à Data Box s’appliquent également à Data Box Heavy.

Le tableau suivant récapitule les étapes de l’ordre d’exportation Data Box ainsi que les outils disponibles pour effectuer le suivi et l’audit de l’ordre à chaque étape.

| Étape de l’ordre d’exportation Data Box       | Outil de suivi et d’audit                                                                        |
|----------------------------|------------------------------------------------------------------------------------------------|
| Créer la commande               | [Configurer le contrôle d’accès sur la commande via Azure RBAC](#set-up-access-control-on-the-order) <br> [Activer le journal détaillé dans l’ordre](#enable-verbose-log-in-the-order)                                                    |
| Commande traitée            | [Suivre la commande](#track-the-order) via <ul><li> Portail Azure </li><li> Site web du transporteur </li><li>Notifications par e-mail</ul> |
| Configurer l’appareil              | Accès aux informations d’identification de l’appareil dans les [journaux d’activité](#query-activity-logs-during-setup)              |
| Copie de données à partir de l’appareil        | [Consulter les journaux de copie](#copy-log) <br> [Consulter les journaux détaillés](#verbose-log) avant de copier des données            |
| Effacement des données de l’appareil   | [Afficher les journaux d’activité de chaîne de responsabilité](#get-chain-of-custody-logs-after-data-erasure) qui incluent les journaux d’audit et l’historique des commandes                |


## <a name="set-up-access-control-on-the-order"></a>Configurer le contrôle d’accès sur la commande

Vous pouvez contrôler qui peut accéder à votre commande lors de sa création. Configurez des rôles Azure d’étendues différentes pour contrôler l’accès à la commande Data Box. Un rôle Azure détermine le type d’accès accordé : lecture-écriture, lecture seule, lecture-écriture sur un sous-ensemble d’opérations.

Les deux rôles qui peuvent être définis pour le service Azure Data Box sont les suivants :

- **Lecteur Data Box** : bénéficie d’un accès en lecture seule aux commandes, tel que défini par l’étendue. Il peut uniquement afficher les détails d’une commande. Il ne peut pas accéder aux autres informations associées aux comptes de stockage ni modifier les détails de la commande comme l’adresse, etc.
- **Contributeur Data Box** : peut uniquement créer une commande pour transférer des données vers un compte de stockage donné *s’il dispose déjà d’un accès en écriture à un compte de stockage*. S’il n’a pas accès à un compte de stockage, il ne peut même pas créer de commande Data Box pour copier des données sur ce compte. Ce rôle ne définit aucune autorisation liée aux comptes de stockage et n’octroie pas d’accès à ces derniers.  

Pour restreindre l’accès à une commande, vous pouvez :

- Affecter un rôle au niveau d’une commande. L’utilisateur dispose uniquement des autorisations définies par son rôle pour interagir avec cette commande Data Box spécifique, et rien d’autre.
- Affectez un rôle au niveau du groupe de ressources. L’utilisateur a accès à toutes les commandes Data Box au sein d’un groupe de ressources.

Pour plus d’informations sur l’utilisation suggérée d’Azure RBAC, consultez [Meilleures pratiques pour Azure RBAC](../role-based-access-control/best-practices.md).

## <a name="enable-verbose-log-in-the-order"></a>Activer le journal détaillé dans l’ordre

Lorsque vous placez un ordre d’exportation pour Data Box, vous avez la possibilité d’activer la collecte d’un journal détaillé. Voici l’écran de commande dans lequel vous pouvez activer le journal détaillé :

![Sélectionner l’option d’exportation](media/data-box-deploy-export-ordered/azure-data-box-export-order-export-option.png)

Lorsque vous sélectionnez l’option **Inclure le journal détaillé**, un fichier journal détaillé est généré lors de la copie des données à partir de votre compte de stockage Azure. Ce journal contient une liste de tous les fichiers qui ont été exportés avec succès.

Pour plus d’informations sur l’ordre d’exportation, consultez [Créer un ordre d’exportation pour Data Box](data-box-deploy-export-ordered.md)

## <a name="track-the-order"></a>Suivre la commande

Vous pouvez suivre votre commande via le Portail Azure et via le site web du transporteur. Les mécanismes suivants permettent de suivre la commande Data Box à tout moment :

- Pour suivre la commande lorsque l’appareil se trouve dans le centre de données Azure ou sur votre site, accédez à votre **Commande Data Box > Vue d’ensemble** dans le Portail Azure.

    ![Afficher l’état de la commande et son numéro de suivi](media/data-box-logs/overview-view-status-1.png)

- Pour suivre la commande pendant que l’appareil est en transit, rendez-vous sur le site web du transporteur régional, par exemple UPS aux États-Unis. Indiquez le numéro de suivi associé à votre commande.
- Data Box envoie également des notifications électronique à chaque changement d’état de la commande aux adresses e-mail fournies lors de sa création. Pour obtenir la liste de tous les états de commande Data Box, consultez [Afficher l’état de la commande](data-box-portal-admin.md#view-order-status). Pour modifier les paramètres de notification associés à la commande, consultez [Modifier les détails de notification](data-box-portal-admin.md#edit-notification-details).

## <a name="query-activity-logs-during-setup"></a>Interroger les journaux d’activité pendant la configuration

- Votre Data Box arrive dans vos locaux dans un état verrouillé. Vous pouvez utiliser les informations d’identification de l’appareil disponibles dans la section du Portail Azure concernant votre commande.  

    Lors de la configuration de Data Box, vous aurez peut-être besoin de savoir qui a eu accès aux informations d’identification de l’appareil. Pour déterminer qui a accédé au panneau **Informations d’identification de l’appareil**, vous pouvez interroger les journaux d’activité.  Toute action qui implique l’accès au panneau **Détails sur l’appareil > Informations d’identification** est enregistrée dans les journaux d’activité en tant qu’action `ListCredentials`.

    ![Interroger les journaux d’activité](media/data-box-logs/query-activity-log-1.png)

- Chaque connexion à Data Box est enregistrée en temps réel. Toutefois, ces informations sont uniquement disponibles dans la [Chaîne des journaux d’audit de garde](#chain-of-custody-audit-logs) une fois que l’ordre est terminée avec succès.

## <a name="view-logs-during-data-copy"></a>Afficher les journaux pendant la copie des données

Avant de copier des données à partir de votre Data Box, vous pouvez télécharger et consulter les *copies de journaux* et *les journaux détaillés* pour les données qui ont été copiées dans Data Box. Ces journaux sont générés lorsque les données sont copiées à partir de votre compte de stockage dans Azure vers votre Data Box. 

### <a name="copy-log"></a>Journal de copie

Avant de copier les données à partir de votre Data Box, téléchargez le journal de copie à partir de la page **Connecter et copier**.

Voici un exemple de sortie du *journal de copie* lorsqu’il n’y a pas d’erreurs et que tous les fichiers ont été copiés entre Azure et l’appareil Data Box.

```output
<CopyLog Summary="Summary">
  <Status>Succeeded</Status>
  <TotalFiles_Blobs>5521</TotalFiles_Blobs>
  <FilesErrored>0</FilesErrored>
</CopyLog>
``` 
    
Voici un exemple de sortie du *journal de copie* lorsqu’il y a des erreurs et que certains fichiers n’ont pas pu être copiés à partir d’Azure.

```output
<ErroredEntity CloudFormat="AppendBlob" Path="export-ut-appendblob/wastorage.v140.3.0.2.nupkg">
  <Category>UploadErrorCloudHttp</Category>
  <ErrorCode>400</ErrorCode>
  <ErrorMessage>UnsupportBlobType</ErrorMessage>
  <Type>File</Type>
</ErroredEntity><ErroredEntity CloudFormat="AppendBlob" Path="export-ut-appendblob/xunit.console.Primary_2020-05-07_03-54-42-PM_27444.hcsml">
  <Category>UploadErrorCloudHttp</Category>
  <ErrorCode>400</ErrorCode>
  <ErrorMessage>UnsupportBlobType</ErrorMessage>
  <Type>File</Type>
</ErroredEntity><ErroredEntity CloudFormat="AppendBlob" Path="export-ut-appendblob/xunit.console.Primary_2020-05-07_03-54-42-PM_27444 (1).hcsml">
  <Category>UploadErrorCloudHttp</Category>
  <ErrorCode>400</ErrorCode>
  <ErrorMessage>UnsupportBlobType</ErrorMessage>
  <Type>File</Type>
</ErroredEntity><CopyLog Summary="Summary">
  <Status>Failed</Status>
  <TotalFiles_Blobs>4</TotalFiles_Blobs>
  <FilesErrored>3</FilesErrored>
</CopyLog>    
```

Vous disposez des options suivantes pour exporter ces fichiers : 

- Vous pouvez transférer les fichiers qui n’ont pas pu être copiés sur le réseau. 
- Si la taille de vos données est supérieure à la capacité utilisable de l’appareil, une copie partielle est effectuée et tous les fichiers qui n’ont pas été copiés sont listés dans ce journal. Vous pouvez utiliser ce journal comme un fichier XML d’entrée pour créer une nouvelle commande Data Box, puis copier ces fichiers.

### <a name="verbose-log"></a>Journaux d’activité détaillés

Le *journal détaillé* contient la liste de tous les fichiers qui ont été exportés à partir du compte de stockage Azure. Le journal contient également la taille du fichier et le calcul de la somme de contrôle.

Le journal détaillé contient les informations au format suivant :

`<file size = "file-size-in-bytes" crc64="cyclic-redundancy-check-string">\folder-path-on-data-box\name-of-file-copied.md</file>`

Voici un échantillon de sortie pour le journal détaillé. 

```powershell
  <File CloudFormat="BlockBlob" Path="validblobdata/test1.2.3.4" Size="1024" crc64="7573843669953104266">
  </File><File CloudFormat="BlockBlob" Path="validblobdata/helloEndWithDot..txt" Size="11" crc64="7320094093915972193">
  </File><File CloudFormat="BlockBlob" Path="validblobdata/test..txt" Size="12" crc64="17906086011702236012">
  </File><File CloudFormat="BlockBlob" Path="validblobdata/test1" Size="1024" crc64="7573843669953104266">
  </File><File CloudFormat="BlockBlob" Path="validblobdata/test1.2.3" Size="1024" crc64="7573843669953104266">
  </File><File CloudFormat="BlockBlob" Path="validblobdata/.......txt" Size="11" crc64="7320094093915972193">
  </File><File CloudFormat="BlockBlob" Path="validblobdata/copylogb08fa3095564421bb550d775fff143ed====..txt" Size="53638" crc64="1147139997367113454">
  </File><File CloudFormat="BlockBlob" Path="validblobdata/testmaxChars-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-12345679" Size="1024" crc64="7573843669953104266">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/file0" Size="0" crc64="0">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/file1" Size="0" crc64="0">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/file4096_000001" Size="4096" crc64="16969371397892565512">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/file4096_000000" Size="4096" crc64="16969371397892565512">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/64KB-Seed10.dat" Size="65536" crc64="10746682179555216785">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/LiveSiteReport_Oct.xlsx" Size="7028" crc64="6103506546789189963">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/NE_Oct_GeoReport.xlsx" Size="103197" crc64="13305485882546035852">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/64KB-Seed1.dat" Size="65536" crc64="3140622834011462581">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/1mbfiles-0-0" Size="1048576" crc64="16086591317856295272">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/file524288_000001" Size="524288" crc64="8908547729214703832">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/4mbfiles-0-0" Size="4194304" crc64="1339017920798612765">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/file524288_000000" Size="524288" crc64="8908547729214703832">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/8mbfiles-0-1" Size="8388608" crc64="3963298606737216548">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/1mbfiles-0-1" Size="1048576" crc64="11061759121415905887">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/XLS-10MB.xls" Size="1199104" crc64="2218419493992437463">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/8mbfiles-0-0" Size="8388608" crc64="1072783424245035917">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/4mbfiles-0-1" Size="4194304" crc64="9991307204216370812">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/VL_Piracy_Negtive10_TPNameAndGCS.xlsx" Size="12398699" crc64="13526033021067702820">
  </File>
  ``````

Les journaux détaillés sont également copiés vers le compte de stockage Azure. Par défaut, les journaux d’activité sont écrits dans un conteneur nommé `copylog`. Les journaux sont stockés avec la convention d’affectation de noms suivante :

`storage-account-name/databoxcopylog/ordername_device-serial-number_CopyLog_guid.xml`.

Le chemin du journal de copie s’affiche également dans le panneau **Vue d’ensemble** du portail.

<!-- add a screenshot-->

Vous pouvez utiliser ces journaux pour vérifier que les fichiers copiés à partir d’Azure correspondent aux données qui ont été copiées sur votre serveur local. 

Utilisez votre fichier journal détaillé :

- Pour vérifier le nom et le nombre de fichiers réels qui ont été copiés vers Data Box.
- Pour vérifier la taille réelle des fichiers.
- Pour vérifier que la valeur *crc64* correspond à une chaîne non nulle. Un calcul de redondance cyclique (CRC) est effectué pendant le chargement vers Azure. Les CRC de l’exportation et celui effectué une fois les données copiées à partir de Data Box vers le serveur local peuvent être comparés. Une différence entre les CRC indique que les fichiers correspondants n’ont pas été copiés.


## <a name="get-chain-of-custody-logs-after-data-erasure"></a>Obtenir les journaux d’activité de chaîne de responsabilité après l’effacement des données

Une fois les données effacées des disques Data Box conformément à la norme NIST SP 800-88 Revision 1, les journaux de chaîne de responsabilité sont disponibles. Ces journaux incluent les journaux d’audit de la chaîne et l’historique des ordres. Les fichiers de nomenclature ou manifeste sont également copiés avec les journaux d’audit.

### <a name="chain-of-custody-audit-logs"></a>Journaux d’audit de la chaîne de responsabilité

Les journaux d’audit e la chaîne de responsabilité contiennent des informations sur la façon de mettre sous tension des partages sur la Data Box ou Data Box Heavy et d’y accéder quand elle est hors du centre de données Azure. Ces journaux d’activité se trouvent ici : `storage-account/azuredatabox-chainofcustodylogs`.

Voici un exemple de journal d’audit de Data Box :

```output
9/10/2018 8:23:01 PM : The operating system started at system time ‎2018‎-‎09‎-‎10T20:23:01.497758400Z.
9/10/2018 8:23:42 PM : An account was successfully logged on.
Subject:
    Security ID:        S-1-5-18
    Account Name:       WIN-DATABOXADMIN
    Account Domain: Workgroup
    Logon ID:       0x3E7
Logon Information:
    Logon Type:     3
    Restricted Admin Mode:  -
    Virtual Account:        No
    Elevated Token:     No
Impersonation Level:        Impersonation
New Logon:
    Security ID:        S-1-5-7
    Account Name:       ANONYMOUS LOGON
    Account Domain: NT AUTHORITY
    Logon ID:       0x775D5
    Linked Logon ID:    0x0
    Network Account Name:   -
    Network Account Domain: -
    Logon GUID:     {00000000-0000-0000-0000-000000000000}
Process Information:
    Process ID:     0x4
    Process Name:       
Network Information:
    Workstation Name:   -
    Source Network Address: -
    Source Port:        -
Detailed Authentication Information:
    Logon Process:      NfsSvr
    Authentication Package:MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
    Transited Services: -
    Package Name (NTLM only):   -
    Key Length:     0
This event is generated when a logon session is created. It is generated on the computer that was accessed. 
The subject fields indicate the account on the local system which requested the logon. This is most commonly a service such as the Server service, or a local process such as Winlogon.exe or Services.exe. 
The logon type field indicates the kind of logon that occurred. The most common types are 2 (interactive) and 3 (network).
The New Logon fields indicate the account for whom the new logon was created, i.e. the account that was logged on.
The network fields indicate where a remote logon request originated. Workstation name is not always available and may be left blank in some cases.
The impersonation level field indicates the extent to which a process in the logon session can impersonate.
The authentication information fields provide detailed information about this specific logon request.
    - Logon GUID is a unique identifier that can be used to correlate this event with a KDC event.
    - Transited services indicate which intermediate services have participated in this logon request.
    - Package name indicates which sub-protocol was used among the NTLM protocols.
    - Key length indicates the length of the generated session key. This will be 0 if no session key was requested.
9/10/2018 8:25:58 PM : An account was successfully logged on.
```

## <a name="download-order-history"></a>Télécharger l’historique des commandes

L’historique des commandes est disponible dans le Portail Azure. Si la commande est terminée et que le nettoyage de l’appareil (l’effacement des données des disques) est terminé lui aussi, accédez à votre commande d’appareil, puis à **Détails de la commande**. L’option **Download order history** (Télécharger l’historique des commandes) est disponible. Pour plus d’informations, consultez [Télécharger l’historique des commandes](data-box-portal-admin.md#download-order-history).

En faisant défiler l’historique des commandes, vous verrez les éléments suivants :

- Informations de suivi du transporteur pour votre appareil.
- Événements avec activité *SecureErase*. Ces événements correspondent à l’effacement des données sur le disque.
- Liens vers les journaux Data Box. Les chemins des *journaux d’audit*, des *journaux de copie* et des fichiers de *nomenclature* sont présentés.

Voici un exemple du journal d’historique des commandes disponible depuis le Portail Azure :

```
-------------------------------
Microsoft Data Box Order Report
-------------------------------
Name                                               : gus-poland                              
StartTime(UTC)                              : 9/19/2018 8:49:23 AM +00:00                       
DeviceType                                     : DataBox                                           
-------------------
Data Box Activities
-------------------
Time(UTC)                 | Activity                       | Status          | Description

9/19/2018 8:49:26 AM      | OrderCreated                   | Completed       |
10/2/2018 7:32:53 AM      | DevicePrepared                 | Completed       |
10/3/2018 1:36:43 PM      | ShippingToCustomer             | InProgress      | Shipment picked up. Local Time : 10/3/2018 1:36:43 PM at AMSTERDAM-NLD                                                                                
10/4/2018 8:23:30 PM      | ShippingToCustomer             | InProgress      | Processed at AMSTERDAM-NLD. Local Time : 10/4/2018 8:23:30 PM at AMSTERDAM-NLD                                                                        
10/4/2018 11:43:34 PM     | ShippingToCustomer             | InProgress      | Departed Facility in AMSTERDAM-NLD. Local Time : 10/4/2018 11:43:34 PM at AMSTERDAM-NLD
10/5/2018 8:13:49 AM      | ShippingToCustomer             | InProgress      | Arrived at Delivery Facility in BRIGHTON-GBR. Local Time : 10/5/2018 8:13:49 AM at LAMBETH-GBR                                                         
10/5/2018 9:13:24 AM      | ShippingToCustomer             | InProgress      | With delivery courier. Local Time : 10/5/2018 9:13:24 AM at BRIGHTON-GBR                                                                               
10/5/2018 12:03:04 PM     | ShippingToCustomer             | Completed       | Delivered - Signed for by. Local Time : 10/5/2018 12:03:04 PM at BRIGHTON-GBR                                                                          
1/25/2019 3:19:25 PM      | ShippingToDataCenter           | InProgress      | Shipment picked up. Local Time : 1/25/2019 3:19:25 PM at BRIGHTON-GBR                                                                                       
1/25/2019 8:03:55 PM      | ShippingToDataCenter           | InProgress      | Processed at BRIGHTON-GBR. Local Time : 1/25/2019 8:03:55 PM at LAMBETH-GBR                                                                            
1/25/2019 8:04:58 PM      | ShippingToDataCenter           | InProgress      | Departed Facility in BRIGHTON-GBR. Local Time : 1/25/2019 8:04:58 PM at BRIGHTON-GBR                                                                    
1/25/2019 9:06:09 PM      | ShippingToDataCenter           | InProgress      | Arrived at Sort Facility LONDON-HEATHROW-GBR. Local Time : 1/25/2019 9:06:09 PM at LONDON-HEATHROW-GBR                                                
1/25/2019 9:48:54 PM      | ShippingToDataCenter           | InProgress      | Processed at LONDON-HEATHROW-GBR. Local Time : 1/25/2019 9:48:54 PM at LONDON-HEATHROW-GBR                                                            
1/25/2019 10:30:20 PM     | ShippingToDataCenter           | InProgress      | Departed Facility in LONDON-HEATHROW-GBR. Local Time : 1/25/2019 10:30:20 PM at LONDON-HEATHROW-GBR
1/28/2019 7:11:35 AM      | ShippingToDataCenter           | InProgress      | Arrived at Delivery Facility in AMSTERDAM-NLD. Local Time : 1/28/2019 7:11:35 AM at AMSTERDAM-NLD                                                     
1/28/2019 9:07:57 AM      | ShippingToDataCenter           | InProgress      | With delivery courier. Local Time : 1/28/2019 9:07:57 AM at AMSTERDAM-NLD                                                                             
1/28/2019 1:35:56 PM      | ShippingToDataCenter           | InProgress      | Scheduled for delivery. Local Time : 1/28/2019 1:35:56 PM at AMSTERDAM-NLD                                                                            
1/28/2019 2:57:48 PM      | ShippingToDataCenter           | Completed       | Delivered - Signed for by. Local Time : 1/28/2019 2:57:48 PM at AMSTERDAM-NLD
1/29/2019 2:18:43 PM      | PhysicalVerification           | Completed       |
1/29/2019 3:49:50 PM      | DeviceBoot                     | Completed       | Appliance booted up successfully.
1/29/2019 3:49:51 PM      | AnomalyDetection               | Completed       | No anomaly detected.
2/12/2019 10:37:03 PM     | DataCopy                       | Resumed         |
2/13/2019 12:05:15 AM     | DataCopy                       | Resumed         |
2/15/2019 7:07:34 PM      | DataCopy                       | Completed       | Copy Completed.
2/15/2019 7:47:32 PM      | SecureErase                    | Started         |
2/15/2019 8:01:10 PM      | SecureErase                    | Completed       | Azure Data Box:<Device-serial-no> has been sanitized according to NIST 800-88 Rev 1.
------------------
Data Box Log Links
------------------
Account Name         : gusacct
Copy Logs Path       : databoxcopylog/gus-poland_<Device-serial-no>_CopyLog_<GUID>.xml
Audit Logs Path      : azuredatabox-chainofcustodylogs\<GUID>\<Device-serial-no>
BOM Files Path       : azuredatabox-chainofcustodylogs\<GUID>\<Device-serial-no>
```

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [résoudre les problèmes sur votre Data Box et votre Data Box Heavy](data-box-troubleshoot.md).

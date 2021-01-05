---
title: Suivi et journalisation des événements Azure Data Box, Azure Data Box Heavy pour les ordres d’importation | Microsoft Docs
description: Décrit comment suivre et journaliser les événements à divers stades de votre ordre d’importation Azure Data Box et Azure Data Box Heavy.
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: article
ms.date: 07/10/2020
ms.author: alkohli
ms.openlocfilehash: a9304936f746b82b59550d62e8b60a9e0035d188
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92147928"
---
# <a name="tracking-and-event-logging-for-your-azure-data-box-and-azure-data-box-heavy-import-order"></a>Suivi et journalisation des événements de vos ordres d’importation Azure Data Box et Azure Data Box Heavy

Un ordre d’importation Data Box ou Data Box Heavy passe par les étapes suivantes : ordre, configuration, copie des données, retour, chargement sur Azure et vérification, puis effacement des données. Lors de chaque étape de la commande, vous pouvez effectuer plusieurs actions pour contrôler l’accès à la commande, auditer les événements, suivre la commande et interpréter les différents journaux générés.

Le tableau suivant récapitule les étapes de l’ordre d’importation Data Box ou Data Box Heavy ainsi que les outils disponibles pour effectuer le suivi et l’audit de l’ordre à chaque étape.

| Étape de l’ordre d’importation Data Box       | Outil de suivi et d’audit                                                                        |
|----------------------------|------------------------------------------------------------------------------------------------|
| Créer la commande               | [Configurer le contrôle d’accès sur la commande via Azure RBAC](#set-up-access-control-on-the-order)                                                    |
| Commande traitée            | [Suivre la commande](#track-the-order) via <ul><li> Portail Azure </li><li> Site web du transporteur </li><li>Notifications par e-mail</ul> |
| Configurer l’appareil              | Accès aux informations d’identification de l’appareil dans les [journaux d’activité](#query-activity-logs-during-setup)                                              |
| Copie des données vers l’appareil        | [Consulter les fichiers *error.xml*](#view-error-log-during-data-copy) pour la copie des données                                                             |
| Préparer l’expédition            | [Inspecter les fichiers de nomenclature](#inspect-bom-during-prepare-to-ship) ou les fichiers manifeste sur l’appareil                                      |
| Chargement des données dans Azure       | [Passer en revue les journaux de copie](#review-copy-log-during-upload-to-azure) à la recherche des erreurs survenues pendant le chargement de données dans le centre de données Azure                         |
| Effacement des données de l’appareil   | [Afficher les journaux d’activité de chaîne de responsabilité](#get-chain-of-custody-logs-after-data-erasure) qui incluent les journaux d’audit et l’historique des commandes                |

Cet article décrit en détail les différents mécanismes ou outils disponibles pour effectuer le suivi et l’audit d’un ordre d’importation Data Box ou Data Box Heavy. Les informations de cet article s’appliquent aux ordres d’importation à la fois de Data Box et de Data Box Heavy. Dans les sections suivantes, toutes les références à Data Box s’appliquent également à Data Box Heavy.

## <a name="set-up-access-control-on-the-order"></a>Configurer le contrôle d’accès sur la commande

Vous pouvez contrôler qui peut accéder à votre commande lors de sa création. Configurez des rôles Azure d’étendues différentes pour contrôler l’accès à la commande Data Box. Un rôle Azure détermine le type d’accès accordé : lecture-écriture, lecture seule, lecture-écriture sur une partie des opérations.

Les deux rôles qui peuvent être définis pour le service Azure Data Box sont les suivants :

- **Lecteur Data Box** : bénéficie d’un accès en lecture seule aux commandes, tel que défini par l’étendue. Il peut uniquement afficher les détails d’une commande. Il ne peut pas accéder aux autres informations associées aux comptes de stockage ni modifier les détails de la commande comme l’adresse, etc.
- **Contributeur Data Box** : peut uniquement créer une commande pour transférer des données vers un compte de stockage donné *s’il dispose déjà d’un accès en écriture à un compte de stockage* . S’il n’a pas accès à un compte de stockage, il ne peut même pas créer de commande Data Box pour copier des données sur ce compte. Ce rôle ne définit aucune autorisation liée aux comptes de stockage et n’octroie pas d’accès à ces derniers.  

Pour restreindre l’accès à une commande, vous pouvez :

- Affecter un rôle au niveau d’une commande. L’utilisateur dispose uniquement des autorisations définies par son rôle pour interagir avec cette commande Data Box spécifique, et rien d’autre.
- Affectez un rôle au niveau du groupe de ressources. L’utilisateur a accès à toutes les commandes Data Box au sein d’un groupe de ressources.

Pour plus d’informations sur l’utilisation suggérée d’Azure RBAC, consultez [Meilleures pratiques pour Azure RBAC](../role-based-access-control/best-practices.md).

## <a name="track-the-order"></a>Suivre la commande

Vous pouvez suivre votre commande via le Portail Azure et via le site web du transporteur. Les mécanismes suivants permettent de suivre la commande Data Box à tout moment :

- Pour suivre la commande lorsque l’appareil se trouve dans le centre de données Azure ou sur votre site, accédez à votre **Commande Data Box > Vue d’ensemble** dans le Portail Azure.

    ![Afficher l’état de la commande et son numéro de suivi](media/data-box-logs/overview-view-status-1.png)

- Pour suivre la commande pendant que l’appareil est en transit, rendez-vous sur le site web du transporteur régional, par exemple UPS aux États-Unis. Indiquez le numéro de suivi associé à votre commande.
- Data Box envoie également des notifications électronique à chaque changement d’état de la commande aux adresses e-mail fournies lors de sa création. Pour obtenir la liste de tous les états de commande Data Box, consultez [Afficher l’état de la commande](data-box-portal-admin.md#view-order-status). Pour modifier les paramètres de notification associés à la commande, consultez [Modifier les détails de notification](data-box-portal-admin.md#edit-notification-details).

## <a name="query-activity-logs-during-setup"></a>Interroger les journaux d’activité pendant la configuration

- Votre Data Box arrive dans vos locaux dans un état verrouillé. Vous pouvez utiliser les informations d’identification de l’appareil disponibles dans la section du Portail Azure concernant votre commande.  

    Lors de la configuration de Data Box, vous aurez peut-être besoin de savoir qui a eu accès aux informations d’identification de l’appareil. Pour déterminer qui a accédé au panneau **Informations d’identification de l’appareil** , vous pouvez interroger les journaux d’activité.  Toute action qui implique l’accès au panneau **Détails sur l’appareil > Informations d’identification** est enregistrée dans les journaux d’activité en tant qu’action `ListCredentials`.

    ![Interroger les journaux d’activité](media/data-box-logs/query-activity-log-1.png)

- Chaque connexion à Data Box est enregistrée en temps réel. Toutefois, ces informations sont uniquement disponibles dans les [Journaux d’audit](#audit-logs) une fois que la commande est terminée avec succès.

## <a name="view-error-log-during-data-copy"></a>Afficher le journal d’erreur pendant la copie des données

Lors de la copie des données dans Data Box ou Data Box Heavy, un fichier d’erreur est généré en cas de problème avec les données copiées.

### <a name="errorxml-file"></a>Fichier Error.xml

Assurez-vous que les travaux de copie ont été accomplis sans erreurs. Si des erreurs se sont produites durant le processus de copie, téléchargez les journaux à partir de la page **Connexion et copie** .

- Si vous avez copié un fichier qui n’est pas de 512 octets alignés dans un dossier de disque managé sur votre Data Box, celui-ci n’est pas chargé en tant qu’objet blob de pages sur votre compte de stockage intermédiaire. Vous verrez une erreur dans les journaux. Supprimez le fichier et copiez un fichier de 512 octets alignés.
- Si vous avez copié un fichier de disque dur virtuel (VHDX), VHD dynamique ou VHD de différenciation (ces fichiers ne sont pas pris en charge), vous verrez une erreur dans les journaux.

Voici un exemple de fichier *error.xml* contenant différentes erreurs pouvant survenir lors de la copie vers des disques managés.

```xml
<file error="ERROR_BLOB_OR_FILE_TYPE_UNSUPPORTED">\StandardHDD\testvhds\differencing-vhd-022019.vhd</file>
<file error="ERROR_BLOB_OR_FILE_TYPE_UNSUPPORTED">\StandardHDD\testvhds\dynamic-vhd-022019.vhd</file>
<file error="ERROR_BLOB_OR_FILE_TYPE_UNSUPPORTED">\StandardHDD\testvhds\insidefixedvhdx-022019.vhdx</file>
<file error="ERROR_BLOB_OR_FILE_TYPE_UNSUPPORTED">\StandardHDD\testvhds\insidediffvhd-022019.vhd</file>
```

Voici un exemple de fichier *error.xml* contenant différentes erreurs pouvant survenir lors de la copie vers des objets blob de pages.

```xml
<file error="ERROR_BLOB_OR_FILE_SIZE_ALIGNMENT">\PageBlob512NotAligned\File100Bytes</file>
<file error="ERROR_BLOB_OR_FILE_SIZE_ALIGNMENT">\PageBlob512NotAligned\File786Bytes</file>
<file error="ERROR_BLOB_OR_FILE_SIZE_ALIGNMENT">\PageBlob512NotAligned\File513Bytes</file>
<file error="ERROR_BLOB_OR_FILE_SIZE_ALIGNMENT">\PageBlob512NotAligned\File10Bytes</file>
<file error="ERROR_BLOB_OR_FILE_SIZE_ALIGNMENT">\PageBlob512NotAligned\File500Bytes</file>
```


Voici un exemple de fichier *error.xml* contenant différentes erreurs pouvant survenir lors de la copie vers des objets blob de blocs.

```xml
<file error="ERROR_CONTAINER_OR_SHARE_NAME_LENGTH">\ab</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\invalid dns name</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_LENGTH">\morethan63charactersfortestingmorethan63charactersfortestingmorethan63charactersfortestingmorethan63charactersfortestingmorethan63charactersfortestingmorethan63charactersfortesting</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\testdirectory-~!@#$%^&amp;()_+{}</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\test__doubleunderscore</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_IMPROPER_DASH">\-startingwith-hyphen</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\Starting with Capital</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\_startingwith_underscore</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_IMPROPER_DASH">\55555555--4444--3333--2222--111111111111</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_LENGTH">\1</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\11111111-_2222-_3333-_4444-_555555555555</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_IMPROPER_DASH">\test--doublehyphen</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTI5Ni3vv70=</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTMwMS3vv70=</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTMwMy3vv70=</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_CONTROL">\InvalidUnicodeFiles\Ã.txt</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTMwNS3vv70=</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTI5OS3vv70=</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTMwMi3vv70=</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTMwNC3vv70=</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTI5OC3vv70=</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTMwMC3vv70=</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTI5Ny3vv70=</file>
```

Voici un exemple de fichier *error.xml* contenant différentes erreurs pouvant survenir lors de la copie vers Azure Files.

```xml
<file error="ERROR_BLOB_OR_FILE_SIZE_LIMIT">\AzFileMorethan1TB\AzFile1.2TB</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\testdirectory-~!@#$%^&amp;()_+{}</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_IMPROPER_DASH">\55555555--4444--3333--2222--111111111111</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_IMPROPER_DASH">\-startingwith-hyphen</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\11111111-_2222-_3333-_4444-_555555555555</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_IMPROPER_DASH">\test--doublehyphen</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_LENGTH">\ab</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\invalid dns name</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\test__doubleunderscore</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_LENGTH">\morethan63charactersfortestingmorethan63charactersfortestingmorethan63charactersfortestingmorethan63charactersfortestingmorethan63charactersfortestingmorethan63charactersfortesting</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\_startingwith_underscore</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_LENGTH">\1</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\Starting with Capital</file>
```

Dans chacun des cas ci-dessus, résolvez les erreurs avant de passer à l’étape suivante. Pour plus d’informations sur les erreurs reçues pendant la copie des données dans Data Box via les protocoles SMB ou NFS, accédez à la page [Résoudre les problèmes de Data Box et Data Box Heavy](data-box-troubleshoot.md). Pour plus d’informations sur les erreurs reçues pendant la copie des données dans Data Box via une API REST, accédez à la page [Résoudre les problèmes liés au stockage Blob Data Box](data-box-troubleshoot-rest.md).

## <a name="inspect-bom-during-prepare-to-ship"></a>Inspecter la nomenclature pendant la préparation de l’expédition

Pendant la préparation de l’expédition, une liste de fichiers connus sous le nom de nomenclature ou fichier manifeste est créée.

- Utilisez ce fichier pour vérifier le nom et le nombre de fichiers réels qui ont été copiés vers Data Box.
- Ce fichier permet de vérifier la taille réelle des fichiers.
- Vérifiez que la valeur *crc64* correspond à une chaîne non nulle. <!--A null value for crc64 indicates that there was a reparse point error)-->

Pour plus d’informations sur les erreurs reçues pendant la préparation de l’expédition, accédez à la page [Résoudre les problèmes de Data Box et Data Box Heavy](data-box-troubleshoot.md).

### <a name="bom-or-manifest-file"></a>Nomenclature ou fichier manifeste

La nomenclature ou le fichier manifeste contient la liste de tous les fichiers copiés sur l’appareil Data Box. Le fichier de nomenclature contient le nom des fichiers et la taille correspondante, ainsi que la somme de contrôle. Un fichier de nomenclature distinct est créé pour les objets blob de blocs, les objets blob de pages, Azure Files, pour la copie via les API REST et pour la copie vers des disques managés sur Data Box. Vous pouvez télécharger les fichiers de nomenclature à partir de l’interface utilisateur web locale de l’appareil pendant la préparation de l’expédition.

Ces fichiers se trouvent également sur l’appareil Data Box et sont chargés sur le compte de stockage associé dans le centre de données Azure.

### <a name="bom-file-format"></a>Format du fichier de nomenclature

Le fichier de nomenclature ou manifeste utilise le format général suivant :

`<file size = "file-size-in-bytes" crc64="cyclic-redundancy-check-string">\folder-path-on-data-box\name-of-file-copied.md</file>`

Voici un exemple de fichier manifeste généré lors de la copie les données vers le partage d’objet blob de blocs sur Data Box.

```
<file size="10923" crc64="0x51c78833c90e4e3f">\databox\media\data-box-deploy-copy-data\connect-shares-file-explorer1.png</file>
<file size="15308" crc64="0x091a8b2c7a3bcf0a">\databox\media\data-box-deploy-copy-data\get-share-credentials2.png</file>
<file size="53486" crc64="0x053da912fb45675f">\databox\media\data-box-deploy-copy-data\nfs-client-access.png</file>
<file size="6093" crc64="0xadb61d0d7c6d4deb">\databox\data-box-cable-options.md</file>
<file size="6499" crc64="0x080add29add367d9">\databox\data-box-deploy-copy-data-via-nfs.md</file>
<file size="11089" crc64="0xc3ce6b13a4fe3001">\databox\data-box-deploy-copy-data-via-rest.md</file>
<file size="7749" crc64="0xd2e346a4588e307a">\databox\data-box-deploy-ordered.md</file>
<file size="14275" crc64="0x260296e5d1b1608a">\databox\data-box-deploy-copy-data.md</file>
<file size="4077" crc64="0x2bb0a170225bceec">\databox\data-box-deploy-picked-up.md</file>
<file size="15447" crc64="0xcec0ca8527720b3c">\databox\data-box-portal-admin.md</file>
<file size="9126" crc64="0x820856b5a54321ad">\databox\data-box-overview.md</file>
<file size="10963" crc64="0x5e9a14f9f4784fd8">\databox\data-box-safety.md</file>
<file size="5941" crc64="0x8631d62fbc038760">\databox\data-box-security.md</file>
<file size="12536" crc64="0x8c8ff93e73d665ec">\databox\data-box-system-requirements-rest.md</file>
<file size="3220" crc64="0x7257a263c434839a">\databox\data-box-system-requirements.md</file>
```

Les fichiers de nomenclature ou manifeste sont également copiés vers le compte de stockage Azure. Vous pouvez utiliser les fichiers de nomenclature ou manifeste pour vérifier que les fichiers chargés sur Azure correspondent aux données copiées dans Data Box.

## <a name="review-copy-log-during-upload-to-azure"></a>Consulter le journal de copie lors du chargement vers Azure

Lors du chargement de données dans Azure, un journal de copie est créé.

### <a name="copy-log"></a>Journal de copie

Pour chaque commande traitée, le service Data Box crée un journal de copie dans le compte de stockage associé. Le journal de copie contient le nombre total de fichiers chargés et le nombre de fichiers ayant rencontré des erreurs pendant la copie des données de Data Box vers votre compte de stockage Azure.

Un contrôle de redondance cyclique (CRC) est effectué pendant le chargement vers Azure. Les CRC de la copie des données et post-chargement sont comparés. Une différence entre les CRC indique que les fichiers correspondants n’ont pas été chargés.

Par défaut, les journaux d’activité sont écrits dans un conteneur nommé `copylog`. Les journaux sont stockés avec la convention d’affectation de noms suivante :

`storage-account-name/databoxcopylog/ordername_device-serial-number_CopyLog_guid.xml`.

Le chemin du journal de copie s’affiche également dans le panneau **Vue d’ensemble** du portail.

![Chemin du journal de copie dans le panneau Vue d’ensemble une fois l’opération terminée](media/data-box-logs/copy-log-path-1.png)

### <a name="upload-completed-successfully"></a>Chargement réussi 

L’exemple suivant décrit le format général d’un journal de copie pour un chargement Data Box terminé correctement :

```
<?xml version="1.0"?>
-<CopyLog Summary="Summary">
<Status>Succeeded</Status>
<TotalFiles>45</TotalFiles>
<FilesErrored>0</FilesErrored>
</CopyLog>
```

### <a name="upload-completed-with-errors"></a>Chargement terminé avec des erreurs 

Le chargement vers Azure peut également se terminer avec des erreurs.

![Chemin du journal de copie dans le panneau Vue d’ensemble avec des erreurs](media/data-box-logs/copy-log-path-2.png)

Voici un exemple de journal de copie pour un chargement terminé avec des erreurs :

```xml
<ErroredEntity Path="iso\samsungssd.iso">
  <Category>UploadErrorCloudHttp</Category>
  <ErrorCode>409</ErrorCode>
  <ErrorMessage>The blob type is invalid for this operation.</ErrorMessage>
  <Type>File</Type>
</ErroredEntity><ErroredEntity Path="iso\iSCSI_Software_Target_33.iso">
  <Category>UploadErrorCloudHttp</Category>
  <ErrorCode>409</ErrorCode>
  <ErrorMessage>The blob type is invalid for this operation.</ErrorMessage>
  <Type>File</Type>
</ErroredEntity><CopyLog Summary="Summary">
  <Status>Failed</Status>
  <TotalFiles_Blobs>72</TotalFiles_Blobs>
  <FilesErrored>2</FilesErrored>
</CopyLog>
```
### <a name="upload-completed-with-warnings"></a>Chargement terminé avec des avertissements

Le chargement dans Azure s’effectue avec des avertissements si vos données ont des noms de conteneur/d’objet blob/de fichier qui ne sont pas conformes aux conventions de nommage Azure et que les noms ont été modifiés pour charger les données dans Azure.

![Chemin du journal de copie dans le panneau Vue d’ensemble avec des avertissements](media/data-box-logs/copy-log-path-3.png)

Voici un exemple de journal de copie où les conteneurs non conformes aux conventions de nommage Azure ont été renommés lors du chargement des données dans Azure.

Les nouveaux noms uniques des conteneurs sont au format `DataBox-GUID` et les données du conteneur sont placées dans le nouveau conteneur renommé. Le journal de copie spécifie l’ancien et le nouveau nom du conteneur.

```xml
<ErroredEntity Path="New Folder">
   <Category>ContainerRenamed</Category>
   <ErrorCode>1</ErrorCode>
   <ErrorMessage>The original container/share/blob has been renamed to: DataBox-3fcd02de-bee6-471e-ac62-33d60317c576 :from: New Folder :because either the name has invalid character(s) or length is not supported</ErrorMessage>
  <Type>Container</Type>
</ErroredEntity>
```

Voici un exemple de journal de copie où les objets blob ou fichiers non conformes aux conventions de nommage Azure ont été renommés lors du chargement des données dans Azure. Les nouveaux noms des objets blob ou fichiers sont convertis avec le code de hachage SHA256 du chemin d’accès relatif au conteneur et sont chargés vers le chemin d’accès, en fonction du type de destination. Il peut s’agir d’objets blob de blocs, d’objets blob de pages ou fichiers Azure Files.

Le `copylog` spécifie l’ancien et le nouveau nom de l’objet blob ou du fichier et son chemin d’accès dans Azure.

```xml
<ErroredEntity Path="TesDir028b4ba9-2426-4e50-9ed1-8e89bf30d285\Ã">
  <Category>BlobRenamed</Category>
  <ErrorCode>1</ErrorCode>
  <ErrorMessage>The original container/share/blob has been renamed to: PageBlob/DataBox-0xcdc5c61692e5d63af53a3cb5473e5200915e17b294683968a286c0228054f10e :from: Ã :because either name has invalid character(s) or length is not supported</ErrorMessage>
  <Type>File</Type>
</ErroredEntity><ErroredEntity Path="TesDir9856b9ab-6acb-4bc3-8717-9a898bdb1f8c\Ã">
  <Category>BlobRenamed</Category>
  <ErrorCode>1</ErrorCode>
  <ErrorMessage>The original container/share/blob has been renamed to: AzureFile/DataBox-0xcdc5c61692e5d63af53a3cb5473e5200915e17b294683968a286c0228054f10e :from: Ã :because either name has invalid character(s) or length is not supported</ErrorMessage>
  <Type>File</Type>
</ErroredEntity><ErroredEntity Path="TesDirf92f6ca4-3828-4338-840b-398b967d810b\Ã">
  <Category>BlobRenamed</Category>
  <ErrorCode>1</ErrorCode>
  <ErrorMessage>The original container/share/blob has been renamed to: BlockBlob/DataBox-0xcdc5c61692e5d63af53a3cb5473e5200915e17b294683968a286c0228054f10e :from: Ã :because either name has invalid character(s) or length is not supported</ErrorMessage>
  <Type>File</Type>
</ErroredEntity>
```

## <a name="get-chain-of-custody-logs-after-data-erasure"></a>Obtenir les journaux d’activité de chaîne de responsabilité après l’effacement des données

Une fois les données effacées des disques Data Box conformément à la norme NIST SP 800-88 Revision 1, les journaux de chaîne de responsabilité sont disponibles. Ils incluent les journaux d’audit et l’historique des commandes. Les fichiers de nomenclature ou manifeste sont également copiés avec les journaux d’audit.

### <a name="audit-logs"></a>Journaux d’audit

Les journaux d’audit contiennent des informations sur la façon de mettre sous tension des partages sur la Data Box ou Data Box Heavy et d’y accéder quand elle est hors du centre de données Azure. Ces journaux d’activité se trouvent ici : `storage-account/azuredatabox-chainofcustodylogs`.

Voici un exemple de journal d’audit de Data Box :

```
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

L’historique des commandes est disponible dans le Portail Azure. Si la commande est terminée et que le nettoyage de l’appareil (l’effacement des données des disques) est terminé lui aussi, accédez à votre commande d’appareil, puis à **Détails de la commande** . L’option **Download order history** (Télécharger l’historique des commandes) est disponible. Pour plus d’informations, consultez [Télécharger l’historique des commandes](data-box-portal-admin.md#download-order-history).

En faisant défiler l’historique des commandes, vous verrez les éléments suivants :

- Informations de suivi du transporteur pour votre appareil.
- Événements avec activité *SecureErase* . Ces événements correspondent à l’effacement des données sur le disque.
- Liens vers les journaux Data Box. Les chemins des *journaux d’audit* , des *journaux de copie* et des fichiers de *nomenclature* sont présentés.

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

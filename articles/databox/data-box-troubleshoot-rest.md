---
title: Résolution des problèmes d’Azure Data Box liés à l’utilisation de l’interface REST | Microsoft Docs
description: Décrit la résolution des problèmes rencontrés dans Azure Data Box lorsqu’une copie de données est effectuée via l’interface REST.
services: databox
author: alkohli
ms.service: databox
ms.subservice: disk
ms.topic: troubleshooting
ms.date: 04/19/2019
ms.author: alkohli
ms.openlocfilehash: b950f80ba8c2bdbaf7a515dc1ce127b934723177
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85558565"
---
# <a name="troubleshoot-issues-related-to-azure-data-box-blob-storage"></a>Résoudre les problèmes liés au Stockage Blob Azure Data Box

Cet article fournit des informations détaillées sur la résolution de problèmes que vous pouvez rencontrer lors de l’utilisation du stockage Blob Data Box via l’interface REST dans votre Data Box pour copier des données. Ces problèmes surviennent lorsque vous utilisez le stockage Blob Data Box avec d’autres applications ou bibliothèques clientes telles que l’Explorateur Stockage Azure, AzCopy ou la bibliothèque Stockage Azure pour Python.

## <a name="errors-seen-in-azure-storage-explorer"></a>Erreurs rencontrées dans l’Explorateur Stockage Azure

Cette section détaille certains des problèmes rencontrés lors de l’utilisation de l’Explorateur Stockage Azure avec Stockage Blob Data Box.

|Message d’erreur  |Action recommandée |
|---------|---------|
|Unable to retrieve child resources. (Impossible de récupérer des ressources enfants.) The value for one of the HTTP headers is not in the correct format. (Le format de la valeur d’un des en-têtes HTTP est incorrect.)|Dans le menu **Modifier**, sélectionnez **Target Azure Stack APIs** (Cibler des API Azure Stack). <br>Redémarrez l’Explorateur Stockage Azure.|
|`getaddrinfo ENOTFOUND <accountname>.blob.<serialnumber>.microsoftdatabox.com` |Vérifiez que le nom de point de terminaison `<accountname>.blob.<serialnumber>.microsoftdatabox.com` est ajouté au fichier hosts sur ce chemin d’accès : <li>`C:\Windows\System32\drivers\etc\hosts` sur Windows, ou </li><li> `/etc/hosts` sur Linux.</li>|
|Unable to retrieve child resources. (Impossible de récupérer des ressources enfants.) <br>Détails : certificat auto-signé |Importez le certificat TLS/SSL pour votre appareil dans l’Explorateur Stockage Azure : <li>Téléchargez le certificat à partir du Portail Azure. Pour plus d’informations, consultez [Télécharger le certificat](data-box-deploy-copy-data-via-rest.md#download-certificate).</li><li>Dans le menu **Modifier**, sélectionnez **Certificats SSL**, puis sélectionnez **Importer les certificats**.</li>|

## <a name="errors-seen-in-azcopy-for-windows"></a>Erreurs rencontrées dans AzCopy pour Windows

Cette section détaille certains des problèmes rencontrés lors de l’utilisation d’AzCopy pour Windows avec Stockage Blob Data Box.

|Message d’erreur  |Action recommandée |
|---------|---------|
|La commande AzCopy semble ne plus répondre pendant une minute avant d’afficher cette erreur : <br>Failed to enumerate directory https://… (Échec de l’énumération du répertoire https://...) Impossible de résoudre le nom distant `<accountname>.blob.<serialnumber>.microsoftdatabox.com`|Vérifiez que le nom de point de terminaison `<accountname>.blob.<serialnumber>.microsoftdatabox.com` est ajouté au fichier hosts sur : `C:\Windows\System32\drivers\etc\hosts`.|
|La commande AzCopy semble ne plus répondre pendant une minute avant d’afficher cette erreur : <br>Error parsing source location. (Erreur d’analyse de l’emplacement source.) Le serveur a clos la connexion sous-jacente : Impossible d’établir une relation de confiance pour le canal sécurisé SSL/TLS.|Importez le certificat TLS/SSL pour votre appareil dans le magasin de certificats du système. Pour plus d’informations, consultez [Télécharger le certificat](data-box-deploy-copy-data-via-rest.md#download-certificate).|


## <a name="errors-seen-in-azcopy-for-linux"></a>Erreurs rencontrées dans AzCopy pour Linux

Cette section détaille certains des problèmes rencontrés lors de l’utilisation d’AzCopy pour Linux avec Stockage Blob Data Box.

|Message d’erreur  |Action recommandée |
|---------|---------|
|La commande AzCopy semble ne plus répondre pendant 20 minutes avant d’afficher cette erreur : <br>Error parsing source location `https://<accountname>.blob.<serialnumber>.microsoftdatabox.com/<cntnr>`. (Erreur d’analyse de l’emplacement source.) Aucun périphérique ou adresse correspondant|Vérifiez que le nom de point de terminaison `<accountname>.blob.<serialnumber>.microsoftdatabox.com` est ajouté au fichier hosts sur : `/etc/hosts`.|
|La commande AzCopy semble ne plus répondre pendant 20 minutes avant d’afficher cette erreur : <br>Error parsing source location... (Erreur d’analyse de l’emplacement source...) The SSL connection could not be established (La connexion SSL n’a pas pu être établie.)|Importez le certificat TLS/SSL pour votre appareil dans le magasin de certificats du système. Pour plus d’informations, consultez [Télécharger le certificat](data-box-deploy-copy-data-via-rest.md#download-certificate).|

## <a name="errors-seen-in-azure-storage-library-for-python"></a>Erreurs rencontrées dans la bibliothèque de Stockage Azure pour Python

Cette section détaille certains des principaux problèmes rencontrés lors du déploiement de Data Box Disk en cas d’utilisation d’un client Linux pour la copie de données.

|Message d’erreur  |Action recommandée |
|---------|---------|
|The value for one of the HTTP headers is not in the correct format. (Le format de la valeur d’un des en-têtes HTTP est incorrect.) |La version installée de la bibliothèque Stockage Microsoft Azure pour Python n’est pas prise en charge par Data Box. Consultez les exigences relatives au Stockage Blob d’Azure Data Box pour connaître les versions prises en charge.|
|… [SSL: CERTIFICATE_VERIFY_FAILED] …|Avant d’exécuter Python, définissez la variable d’environnement REQUESTS_CA_BUNDLE sur le chemin d’accès du fichier de certificat TLS codé en Base64 (consultez comment [Télécharger le certificat](data-box-deploy-copy-data-via-rest.md#download-certificate)). <br>Par exemple :<br>`export REQUESTS_CA_BUNDLE=/tmp/mycert.cer` <br>`python` <br>Vous pouvez également ajouter le certificat au magasin de certificats du système puis définir cette variable d’environnement sur le chemin d’accès de ce magasin. <br> Par exemple, sur Ubuntu : <br>`export REQUESTS_CA_BUNDLE=/etc/ssl/certs/ca-certificates.crt` <br>`python`|


## <a name="common-errors"></a>Erreurs courantes

Ces erreurs ne sont pas spécifiques à une application.

|Message d’erreur  |Action recommandée |
|---------|---------|
|The connection times out. (La connexion arrive à expiration.) |Connectez-vous à l’appareil Data Box et vérifiez qu’il est déverrouillé. Chaque fois que l’appareil redémarre, il reste verrouillé jusqu’à ce qu’un utilisateur se connecte.|

## <a name="next-steps"></a>Étapes suivantes

- Découvrez la [configuration système requise du Stockage Blob Data Box](data-box-system-requirements-rest.md).

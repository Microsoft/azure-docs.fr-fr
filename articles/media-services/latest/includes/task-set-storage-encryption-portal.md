---
author: IngridAtMicrosoft
ms.service: media-services
ms.topic: include
ms.date: 11/04/2020
ms.author: inhenkel
ms.custom: portal
ms.openlocfilehash: c3a27218f03980e9b42a4a56fa739203c43754a0
ms.sourcegitcommit: c157b830430f9937a7fa7a3a6666dcb66caa338b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "96013229"
---
<!--Set the encryption on storage account in the portal-->

## <a name="set-the-encryption-on-a-storage-account"></a>Définir le chiffrement sur un compte de stockage

1. Dans le portail Azure, entrez le nom du compte de stockage à chiffrer dans le champ **Rechercher** en haut de l’écran.  Les correspondances apparaissent sous le champ de recherche.
1. Sélectionnez le compte de stockage que vous recherchez. L’écran du compte de stockage s’affiche.
1. Sélectionnez **Chiffrement**.
1. Sélectionnez des clés gérées par Microsoft ou par le client.

### <a name="use-microsoft-managed-keys"></a>Utiliser des clés gérées par Microsoft

Par défaut, les données incluses dans le compte de stockage sont chiffrées à l’aide de clés gérées par Microsoft.

### <a name="use-customer-managed-keys"></a>Utiliser des clés gérées par le client

1. Sélectionnez **Clés gérées par le client**.
1. Sélectionnez **Entrer l’URI de la clé** ou **Sélectionner dans le coffre de clés**.
    1. Si vous sélectionnez **Entrer l’URI de la clé**, entrez-le dans le champ idoine et sélectionnez l’abonnement. (Il est peut-être déjà sélectionné automatiquement.)
    1. Si vous sélectionnez **Sélectionner dans le coffre de clés**, vous sélectionnez alors **Sélectionner un coffre de clés et une clé**. L’écran Sélectionner une clé dans Azure Key Vault apparaît.
1. Sélectionnez le **coffre de clés** à utiliser, puis sélectionnez une clé que vous avez déjà dans votre coffre de clés ou **créer-en une**.
    1. Si vous choisissez de créer une clé, sélectionnez **Générer** ou **Importer** dans la liste déroulante **Options**. Vous pouvez importer uniquement des clés RSA.
    1. Pour générer une nouvelle clé, donnez un nom à la clé dans le champ **Nom**, puis sélectionnez le type de clé :
        1. RSA - Tailles de clé :  2 048, 3 072 ou 4 096. Il s’agit du choix de la plupart des clients.
        1. EC - Noms de courbe elliptique : P-256, P-384, P-521 ou P-256K.
        1. Vous pouvez éventuellement définir les dates d’activation et d’expiration de la clé.
        1. Sélectionnez **Oui** pour activer la rotation automatique des clés.
        1. Sélectionnez **Créer**.
    1. Pour importer une clé, sélectionnez le fichier à charger en cliquant n’importe où dans le **champ Sélectionner un fichier**.
        1. Donnez un nom à la clé dans le champ **Nom**.
        1. Vous pouvez éventuellement définir les dates d’activation et d’expiration de la clé.
        1. Sélectionnez **Oui** pour activer la rotation automatique des clés.
        1. Sélectionnez **Créer**.
    1. Choisissez **Sélectionner** pour sélectionner cette clé afin de chiffrer votre compte de stockage. Vous êtes redirigé vers l’écran Chiffrement.
1. **IMPORTANT !** Sélectionnez **Enregistrer** pour enregistrer vos paramètres de chiffrement, sans quoi tout ce que vous venez de faire sera perdu.

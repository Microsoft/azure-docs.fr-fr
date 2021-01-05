---
title: Chiffrement dans Azure Data Lake Storage Gen1 | Microsoft Docs
description: Le chiffrement dans Azure Data Lake Storage Gen1 vous permet de protéger vos données, de mettre en œuvre des stratégies de sécurité d’entreprise et de répondre aux exigences de conformité réglementaire. Cet article fournit une vue d’ensemble de la conception et présente certains aspects techniques de la mise en œuvre.
services: data-lake-store
documentationcenter: ''
author: esung22
ms.service: data-lake-store
ms.topic: conceptual
ms.date: 03/26/2018
ms.author: yagupta
ms.openlocfilehash: f924cb7462f7f8c9939ec261b7ef200ceb8ea70b
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92109151"
---
# <a name="encryption-of-data-in-azure-data-lake-storage-gen1"></a>Chiffrement des données dans Azure Data Lake Storage Gen1

Le chiffrement dans Azure Data Lake Storage Gen1 vous permet de protéger vos données, de mettre en œuvre des stratégies de sécurité d’entreprise et de répondre aux exigences de conformité réglementaire. Cet article fournit une vue d’ensemble de la conception et présente certains aspects techniques de la mise en œuvre.

Data Lake Storage Gen1 prend en charge le chiffrement des données au repos et en transit. Pour les données au repos, Data Lake Storage Gen1 s’appuie sur un chiffrement transparent « activé par défaut ». Voici, plus en détails, ce que ces termes signifient :

* **Activé par défaut** : lorsque vous créez un compte Data Lake Storage Gen1, le paramètre par défaut active le chiffrement. Ainsi, les données stockées dans Data Lake Storage Gen1 sont toujours chiffrées avant d’être stockées sur un support permanent. Ce comportement s’applique à toutes les données et ne peut pas être modifié après la création d’un compte.
* **Transparent** : Data Lake Storage Gen1 chiffre automatiquement les données avant leur stockage permanent et les déchiffre avant leur récupération. Ce chiffrement est configuré et géré par un administrateur au niveau du compte Data Lake Storage Gen1. Aucune modification n’est apportée aux API d’accès aux données. En raison de ce chiffrement, aucune modification n’est donc requise dans les applications et services qui interagissent avec Data Lake Storage Gen1.

De même, les données en transit (ou données en mouvement) sont toujours chiffrées dans Data Lake Storage Gen1. Outre le chiffrement des données avant le stockage sur un support permanent, les données sont également toujours sécurisées en transit à l’aide du protocole HTTPS. HTTPS est le seul protocole pris en charge pour les interfaces REST Data Lake Storage Gen1. Le diagramme suivant illustre le mode de chiffrement des données dans Data Lake Storage Gen1 :

![Diagramme du chiffrement des données dans Data Lake Storage Gen1](./media/data-lake-store-encryption/fig1.png)


## <a name="set-up-encryption-with-data-lake-storage-gen1"></a>Configurer le chiffrement avec Data Lake Storage Gen1

Le chiffrement pour Data Lake Storage Gen1 est configuré au moment de la création du compte et est toujours activé par défaut. Vous pouvez gérer les clés vous-même ou autoriser Data Lake Storage Gen1 à les gérer pour vous (option par défaut).

Pour plus d’informations, consultez [Prise en main](./data-lake-store-get-started-portal.md).

## <a name="how-encryption-works-in-data-lake-storage-gen1"></a>Fonctionnement du chiffrement dans Data Lake Storage Gen1

La section suivante aborde la gestion des clés de chiffrement principales, puis décrit les trois types de clés que vous pouvez utiliser dans le chiffrement des données pour Data Lake Storage Gen1.

### <a name="master-encryption-keys"></a>Clés de chiffrement principales

Data Lake Storage Gen1 propose deux modes de gestion des clés de chiffrement principales (MEK). À ce stade, supposons que la clé de chiffrement principale est la clé de niveau supérieur. Un accès à la clé de chiffrement principale est requis pour déchiffrer les données stockées dans Data Lake Storage Gen1.

Les deux modes de gestion de la clé de chiffrement principale sont les suivants :

*   Clés gérées par le service
*   Clés gérées par le client

Dans ces deux modes, la clé de chiffrement principale est sécurisée en la stockant dans Azure Key Vault. Key Vault est un service hautement sécurisé entièrement géré sur Azure qui peut être utilisé pour protéger les clés cryptographiques. Pour plus d’informations, consultez [Key Vault](https://azure.microsoft.com/services/key-vault).

Voici une brève comparaison des fonctionnalités fournies par les deux modes de gestion des MEK.

| Question | Clés gérées par le service | Clés gérées par le client |
| -------- | -------------------- | --------------------- |
|Comment les données sont stockées ?|Toujours chiffrées avant d’être stockées.|Toujours chiffrées avant d’être stockées.|
|Où est stockée la clé de chiffrement principale ?|Key Vault|Key Vault|
|Des clés de chiffrement sont-elles stockées en dehors de Key Vault ? |Non|Non|
|La MEK peut-elle être récupérée dans Key Vault ?|Non. Une fois la MEK stockée dans Key Vault, elle peut uniquement être utilisée pour le chiffrement et le déchiffrement.|Non. Une fois la MEK stockée dans Key Vault, elle peut uniquement être utilisée pour le chiffrement et le déchiffrement.|
|Qui possède l’instance Key Vault et la MEK ?|Le service Data Lake Storage Gen1|Vous possédez l’instance Key Vault, qui appartient à votre propre abonnement Azure. La MEK dans Key Vault peut être gérée par logiciel ou matériel.|
|Pouvez-vous révoquer l’accès à la MEK pour le service Data Lake Storage Gen1 ?|Non|Oui. Vous pouvez gérer des listes de contrôle d’accès sur Key Vault et supprimer des entrées de contrôle d’accès au niveau de l’identité de service pour le service Data Lake Storage Gen1.|
|Pouvez-vous supprimer définitivement la MEK ?|Non|Oui. Si le client supprime la MEK de Key Vault, les données du compte Data Lake Storage Gen1 ne peuvent être déchiffrées par personne, ni même par le service Data Lake Storage Gen1. <br><br> Si vous avez explicitement sauvegardé la MEK avant de la supprimer de Key Vault, elle peut être restaurée et les données peuvent ensuite être récupérées. Toutefois, si vous n’avez pas sauvegardé la MEK avant de la supprimer de Key Vault, les données du compte Data Lake Storage Gen1 ne pourront plus jamais être déchiffrées par la suite.|


Hormis la différence entre l’utilisateur qui gère la MEK et l’instance Key Vault dans lequel elle se trouve, les autres éléments de conception sont les mêmes pour les deux modes.

Il est important de se rappeler ce qui suit lors du choix du mode des clés de chiffrement principales :

*   Vous pouvez choisir d’utiliser des clés gérées par le client ou des clés gérées par le service au moment de la configuration d’un compte Data Lake Storage Gen1.
*   Une fois qu’un compte Data Lake Storage Gen1 a été configuré, il est impossible de modifier le mode.

### <a name="encryption-and-decryption-of-data"></a>Chiffrement et déchiffrement des données

Trois types de clés sont utilisées dans la conception du chiffrement des données. Le tableau suivant présente une synthèse :

| Clé                   | Abréviation | Associée à | Emplacement de stockage                             | Type       | Notes                                                                                                   |
|-----------------------|--------------|-----------------|----------------------------------------------|------------|---------------------------------------------------------------------------------------------------------|
| Clé de chiffrement principale | MEK          | Un compte Data Lake Storage Gen1 | Key Vault                              | Asymétrique | Peut être gérée par Data Lake Storage Gen1 ou par vous.                                                              |
| Clé de chiffrement des données   | DEK          | Un compte Data Lake Storage Gen1 | Stockage permanent, géré par le service Data Lake Storage Gen1 | Symétrique  | La DEK est chiffrée par la MEK. La DEK chiffrée est stockée sur un support permanent. |
| Clé de chiffrement de bloc  | BEK          | Un bloc de données | None                                         | Symétrique  | La BEK est dérivée de la DEK et du bloc de données.                                                      |

Le schéma suivant illustre ces concepts :

![Clés dans le chiffrement des données](./media/data-lake-store-encryption/fig2.png)

#### <a name="pseudo-algorithm-when-a-file-is-to-be-decrypted"></a>Pseudo-algorithme lorsqu’un fichier doit être déchiffré :
1.  Vérifiez si la DEK du compte Data Lake Storage Gen1 est mise en cache et prête à être utilisée.
    - Si ce n’est pas le cas, lisez alors la DEK chiffrée à partir d’un stockage permanent et envoyez-la vers Key Vault pour être déchiffrée. Mettez la DEK déchiffrée en mémoire cache. Elle est maintenant prête à utiliser.
2.  Pour chaque bloc de données du fichier :
    - Lire le bloc de données chiffré sur un stockage permanent.
    - Générer la BEK à partir de la DEK et du bloc de données chiffré.
    - Utiliser la BEK pour déchiffrer des données.


#### <a name="pseudo-algorithm-when-a-block-of-data-is-to-be-encrypted"></a>Pseudo-algorithme lorsqu’un bloc de données doit être chiffré :
1.  Vérifiez si la DEK du compte Data Lake Storage Gen1 est mise en cache et prête à être utilisée.
    - Si ce n’est pas le cas, lisez alors la DEK chiffrée à partir d’un stockage permanent et envoyez-la vers Key Vault pour être déchiffrée. Mettez la DEK déchiffrée en mémoire cache. Elle est maintenant prête à utiliser.
2.  Générez une BEK unique pour le bloc de données à partir de la DEK.
3.  Chiffrez le bloc de données avec la BEK à l’aide du chiffrement AES-256.
4.  Stockez le bloc de données chiffré sur un stockage permanent.

> [!NOTE] 
> La clé de chiffrement est toujours stockée chiffrée par la MEK, soit sur un média persistant ou via une mise en cache en mémoire.

## <a name="key-rotation"></a>Rotation des clés

Lorsque vous utilisez des clés gérées par le client, vous pouvez effectuer une rotation de la MEK. Pour savoir comment configurer un compte Data Lake Storage Gen1 avec des clés gérées par le client, voir [Prise en main](./data-lake-store-get-started-portal.md).

### <a name="prerequisites"></a>Prérequis

Au moment où vous avez configuré le compte Data Lake Storage Gen1, vous avez choisi d’utiliser vos propres clés. Cette option ne peut pas être modifiée une fois le compte créé. Les étapes ci-dessous supposent que vous utilisez des clés gérées par le client (c’est-à-dire que vous avez choisi vos propres clés dans Key Vault).

Notez que si vous utilisez les options par défaut pour le chiffrement, vos données seront toujours chiffrées à l’aide des clés gérées par Data Lake Storage Gen1. Dans cette option, vous ne pouvez pas effectuer une rotation des clés, car elles sont gérées par Data Lake Storage Gen1.

### <a name="how-to-rotate-the-mek-in-data-lake-storage-gen1"></a>Comment effectuer une rotation de la MEK dans Data Lake Storage Gen1

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Accédez à l’instance Key Vault dans laquelle sont stockées les clés associées à votre compte Data Lake Storage Gen1. Sélectionnez **Clés**.

    ![Capture d’écran de Key Vault](./media/data-lake-store-encryption/keyvault.png)

3. Sélectionnez la clé associée à votre compte Data Lake Storage Gen1, puis créez une nouvelle version de cette clé. Notez que pour le moment, Data Lake Storage Gen1 permet une rotation des clés uniquement vers une nouvelle version de clé. Il ne prend en charge la rotation vers une autre clé.

   ![Capture d’écran de la fenêtre Clés, avec Nouvelle version en surbrillance](./media/data-lake-store-encryption/keynewversion.png)

4. Accédez au compte Data Lake Storage Gen1 et sélectionnez **Chiffrement**.

   ![Capture d’écran de la fenêtre du compte Data Lake Storage Gen1 avec Chiffrement en surbrillance](./media/data-lake-store-encryption/select-encryption.png)

5. Un message vous informe qu’une nouvelle version de clé de la clé est disponible. Cliquez sur **Rotation de clé** pour mettre à jour la clé vers la nouvelle version.

   ![Capture d’écran de la fenêtre Data Lake Storage Gen1 avec message et Rotation de clé en surbrillance](./media/data-lake-store-encryption/rotatekey.png)

Cette opération doit prendre moins de deux minutes et aucune interruption de service due à la rotation des clés n’est attendue. Une fois l’opération terminée, la nouvelle version de la clé est utilisée.

> [!IMPORTANT]
> Une fois l’opération de rotation des clés terminée, l’ancienne version de la clé n’est plus utilisée activement pour chiffrer vos données.  Toutefois, dans de rares cas d’échec inattendu dans lesquels des copies redondantes de vos données sont affectées, les données peuvent être restaurées à partir d’une sauvegarde qui utilise toujours l’ancienne clé. Pour garantir que vos données sont accessibles dans ces rares cas, conservez une copie de la version précédente de votre clé de chiffrement. Pour obtenir de meilleures pratiques sur la planification de votre récupération d’urgence, voir [Conseils sur la récupération d’urgence des données dans Data Lake Storage Gen1](data-lake-store-disaster-recovery-guidance.md).
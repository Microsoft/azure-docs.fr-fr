---
title: Génération et transfert de clés protégées par HSM pour Azure Key Vault - Azure Key Vault
description: Utilisez les informations présentes dans cette rubrique pour planifier, générer, puis transférer vos propres clés protégées par le module de sécurité matériel à utiliser avec le coffre de clés Azure. Cette méthode est également appelée BYOK (Bring Your Own Key, ou apportez votre propre clé).
services: key-vault
author: amitbapat
manager: devtiw
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: keys
ms.topic: tutorial
ms.date: 05/29/2020
ms.author: ambapat
ms.openlocfilehash: 8a1f3b5e80152fb0fb9458aef0d3524dd2d6f5eb
ms.sourcegitcommit: 5db975ced62cd095be587d99da01949222fc69a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97092327"
---
# <a name="import-hsm-protected-keys-for-key-vault-ncipher"></a>Importer des clés protégées par HSM pour Key Vault (nCipher)

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Pour une meilleure garantie, lorsque vous utilisez le coffre de clés Azure, vous pouvez importer ou générer des clés dans des modules de sécurité matériels (HSM) qui ne franchissent jamais les limites HSM. Ce scénario est souvent appelé *Apportez votre propre clé* ou désigné par l’acronyme BYOK. Azure Key Vault utilise la famille nShield de modules HSM (FIPS 140-2 niveau 2 validé) de nCipher pour protéger vos clés.

> [!NOTE]
> La méthode d’importation de clé HSM décrite dans ce document ne fonctionne qu’avec la famille nCipher nShield des modules HSM. Pour importer des clés HSM à partir d’autres modules HSM, [voir ici](hsm-protected-keys-byok.md).

Les informations de cette rubrique vous aident à planifier, à générer puis transférer vos propres clés protégées par HSM à utiliser avec Azure Key Vault. 

Cette fonctionnalité n’est pas disponible pour Azure Chine 21Vianet.

> [!NOTE]
> Pour plus d’informations sur le coffre de clés Azure, consultez la page [Présentation du coffre de clés Azure](../general/overview.md)  
> Pour voir un didacticiel de mise en route incluant un coffre de clés pour les clés protégées par HSM, consultez la section [ Présentation d’Azure Key Vault](../general/overview.md).

Plus d’informations sur la génération et le transfert d’une clé protégée par HSM sur Internet :

* Il est possible de générer la clé depuis une station de travail hors connexion, ce qui réduit la surface d’attaque.
* La clé est chiffrée à l’aide d’une clé Key Exchange Key (KEK), qui reste chiffrée jusqu’à ce qu’elle soit transférée vers le module de sécurité matériel du coffre de clés Azure. Seule une version chiffrée de la clé quitte la station de travail d’origine.
* Le jeu d’outils définit les propriétés sur votre clé de locataire qui lie votre clé au monde de sécurité du coffre de clés Azure. Par conséquent, votre clé ne peut être utilisée que par les modules de sécurité matériels du coffre de clés Azure qui l’ont réceptionnée et déchiffrée. Il est impossible d’exporter votre clé. Cette liaison est appliquée par les modules HSM de nCipher.
* La clé KEK qui est utilisée pour chiffrer votre clé est générée dans les modules de sécurité matériels du coffre de clés Azure et n’est pas exportable. Les HSM garantissent qu’aucune version en clair de la clé KEK ne se trouve hors des HSM. En outre, le jeu d’outils inclut une attestation de nCipher certifiant que la clé KEK n’est pas exportable et qu’elle a été générée à l’intérieur d’un HSM authentique conçu par nCipher.
* Le jeu d’outils inclut une attestation de nCipher certifiant que le monde de sécurité d’Azure Key Vault a été généré sur un module HSM authentique fabriqué par nCipher. Cette attestation prouve que Microsoft utilise du matériel authentique.
* Microsoft utilise des KEK et des mondes de sécurité séparés dans chaque région géographique. Cette séparation garantit que votre clé peut être utilisée uniquement dans des centres de données de la région où vous l’avez chiffrée. Par exemple, la clé d’un client européen ne peut pas être utilisée dans des centres de données d’Amérique du Nord ou d’ Asie.

## <a name="more-information-about-ncipher-hsms-and-microsoft-services"></a>Plus d’informations sur les HSM nCipher et Microsoft

nCipher Security, entreprise Entrust Datacard, est un leader du marché des HSM à usage général, qui renforce les organisations du monde entier en apportant à leurs applications et informations stratégiques un niveau de confiance, d’intégrité et de contrôle. Les solutions de chiffrement de nCipher sécurisent les technologies émergentes (Cloud, IoT, blockchain, paiements numériques) et aident à répondre aux nouvelles obligations de conformité, en utilisant la même technologie éprouvée que celle dont dépendent aujourd’hui les organisations mondiales pour se protéger contre les menaces pesant sur leurs données sensibles, communications réseau et infrastructure d’entreprise. nCipher contribue à la fiabilité des applications vitales pour l’entreprise, garantissant l’intégrité des données et le contrôle total par les clients, aujourd’hui, demain et à tout moment.

Microsoft a collaboré avec nCipher Security pour améliorer encore les performances HSM. Ces améliorations vous permettent de bénéficier des avantages des services hébergés sans avoir à renoncer au contrôle de vos clés. Ces améliorations permettent en particulier à Microsoft de gérer les modules de sécurité matériels pour vous. Comme un service cloud, Azure Key Vault effectue un scale-up très rapidement pour répondre aux pics d’activité de votre organisation. Dans le même temps, votre clé est protégée à l’intérieur des HSM de Microsoft : vous gardez le contrôle sur le cycle de vie de la clé, car vous générez la clé, puis vous la transférez vers les modules de sécurité matérielle (HSM) de Microsoft.

## <a name="implementing-bring-your-own-key-byok-for-azure-key-vault"></a>Mise en œuvre du principe BYOK (Apportez votre propre clé, pour le coffre de clés Azure

Utilisez les informations et les procédures qui suivent pour générer votre propre clé protégée HSM, puis la transférer vers le coffre de clés Azure. Le scénario BYOK.

## <a name="prerequisites-for-byok"></a>Configuration requise pour BYOK

Consultez le tableau qui suit pour connaître les conditions requises pour apporter votre propre clé (BYOK) dans le coffre de clés Azure.

| Condition requise | Informations complémentaires |
| --- | --- |
| Abonnement à Azure |Pour créer un coffre de clés Azure, vous avez besoin d’un abonnement Azure : [Inscrivez-vous pour un essai gratuit](https://azure.microsoft.com/pricing/free-trial/) |
| Niveau de service Premium d’Azure Key Vault pour prendre en charge les clés protégées par HSM |Pour plus d’informations sur les niveaux de service et les capacités du coffre de clés Azure, consultez le site web [Tarifs Azure Key Vault](https://azure.microsoft.com/pricing/details/key-vault/). |
| Modules HSM nShield, cartes à puces et logiciel de support de nCipher |Vous devez disposer d’un accès au module de sécurité matérielle nCipher et de quelques connaissances de bases concernant les modules de sécurité matérielle nShield de nCipher. Consultez [Modules de sécurité matérielle nShield de nCipher](https://go.ncipher.com/rs/104-QOX-775/images/nCipher_nShield_Family_Brochure.pdf?_ga=2.106120835.1607422418.1590478092-577009923.1587131206) pour obtenir une liste des modèles compatibles ou pour acheter un module de sécurité matérielle si vous n’en avez pas déjà. |
| Les matériels et le logiciel suivants :<ol><li>Une station de travail x64 hors connexion avec le système d’exploitation Windows 7 ou version ultérieure, et le logiciel nCipher nShield version 11.50 ou ultérieure.<br/><br/>Si cette station de travail exécute Windows 7, vous devez [installer Microsoft .NET Framework 4.5](https://download.microsoft.com/download/b/a/4/ba4a7e71-2906-4b2d-a0e1-80cf16844f5f/dotnetfx45_full_x86_x64.exe).</li><li>Une station de travail connectée à Internet et dotée du système d’exploitation minimal Windows 7 et d’[Azure PowerShell](/powershell/azure/?view=azps-1.2.0)**version 1.1.0 ou ultérieure**.</li><li>Un lecteur USB ou tout autre appareil de stockage portable offrant au moins 16 Mo d’espace libre.</li></ol> |Pour des raisons sécurité, nous conseillons de faire en sorte que la première station de travail ne soit pas connectée à un réseau. Toutefois, cette recommandation n’est pas appliquée par programmation.<br/><br/>Dans les instructions qui suivent, cette station de travail est désignée en tant que station de travail déconnectée.</p></blockquote><br/>De plus, si votre clé de locataire est destinée à un réseau de production, nous vous recommandons d’utiliser un poste de travail distinct pour télécharger l’ensemble d’outils et charger la clé de locataire. À des fins de test, vous pouvez utiliser la même station de travail que la précédente.<br/><br/>Dans les instructions qui suivent, cette deuxième station de travail est désignée en tant que station de travail connectée à Internet.</p></blockquote><br/> |

## <a name="generate-and-transfer-your-key-to-azure-key-vault-hsm"></a>Générez et transférez votre clé sur le module de sécurité matériel du coffre de clés Azure

Pour générer votre clé et la transférer vers un module de sécurité matériel d’Azure Key Vault, vous allez suivre les cinq étapes suivantes :

* [Étape 1 : Préparer votre station de travail connectée à Internet](#step-1-prepare-your-internet-connected-workstation)
* [Étape 2 : Préparer votre station de travail déconnectée](#step-2-prepare-your-disconnected-workstation)
* [Étape 3 : Générer votre clé](#step-3-generate-your-key)
* [Étape 4 : Préparer votre clé pour le transfert](#step-4-prepare-your-key-for-transfer)
* [Étape 5 : Transférer votre clé vers Azure Key Vault](#step-5-transfer-your-key-to-azure-key-vault)

## <a name="step-1-prepare-your-internet-connected-workstation"></a>Étape 1 : Préparer votre station de travail connectée à Internet

Pour cette première étape, exécutez les procédures qui suivent sur la station de travail connectée à Internet.

### <a name="step-11-install-azure-powershell"></a>Étape 1.1 : Installation d’Azure PowerShell

Depuis la station de travail connectée à Internet, téléchargez et installez le module Azure PowerShell qui inclut les applets de commande servant à gérer le coffre de clés Azure. Pour connaître la procédure d’installation, consultez l’article [Installation et configuration d’Azure PowerShell](/powershell/azure/).

### <a name="step-12-get-your-azure-subscription-id"></a>Étape 1.2 : Obtenir votre ID d’abonnement Azure

Démarrez une session Azure PowerShell et connectez-vous à votre compte Azure en utilisant la commande suivante :

```Powershell
   Connect-AzAccount
```
Dans la fenêtre contextuelle de votre navigateur, entrez votre nom d’utilisateur et votre mot de passe Azure. Utilisez ensuite la commande [Get-AzSubscription](/powershell/module/az.accounts/get-azsubscription) :

```powershell
   Get-AzSubscription
```
Dans le résultat, recherchez l’ID de l’abonnement que vous utiliserez pour le coffre de clés Azure. Vous aurez besoin cet ID d’abonnement ultérieurement.

Ne fermez pas la fenêtre Azure PowerShell.

### <a name="step-13-download-the-byok-toolset-for-azure-key-vault"></a>Étape 1.3 : Télécharger l’ensemble d’outils BYOK pour Azure Key Vault

Accédez au centre de téléchargement Microsoft et [téléchargez l’ensemble d’outils BYOK d’Azure Key Vault](https://www.microsoft.com/download/details.aspx?id=45345) correspondant à votre région géographique ou à votre instance d’Azure. Utilisez les informations suivantes pour identifier le nom du package à télécharger et son package de hachage SHA-256 correspondant :

---
**États-Unis**

KeyVault-BYOK-Tools-UnitedStates.zip

2E8C00320400430106366A4E8C67B79015524E4EC24A2D3A6DC513CA1823B0D4

---
**Europe :**

KeyVault-BYOK-outils-Europe.zip

9AAA63E2E7F20CF9BB62485868754203721D2F88D300910634A32DFA1FB19E4A

---
**Asie :**

KeyVault-BYOK-Tools-AsiaPacific.zip

4BC14059BF0FEC562CA927AF621DF665328F8A13616F44C977388EC7121EF6B5

---
**Amérique latine :**

KeyVault-BYOK-Tools-LatinAmerica.zip

E7DFAFF579AFE1B9732C30D6FD80C4D03756642F25A538922DD1B01A4FACB619

---
**Japon :**

KeyVault-BYOK-Tools-Japan.zip

3933C13CC6DC06651295ADC482B027AF923A76F1F6BF98B4D4B8E94632DEC7DF

---
**Corée :**

KeyVault-BYOK-Tools-Korea.zip

71AB6BCFE06950097C8C18D532A9184BEF52A74BB944B8610DDDA05344ED136F

---
**Afrique du Sud :**

KeyVault-BYOK-Tools-SouthAfrica.zip

C41060C5C0170AAAAD896DA732E31433D14CB9FC83AC3C67766F46D98620784A

---
**Émirats Arabes Unis :**

KeyVault-BYOK-Tools-UAE.zip

FADE80210B06962AA0913EA411DAB977929248C65F365FD953BB9F241D5FC0D3

---
**Australie :**

KeyVault-BYOK-Tools-Australia.zip

CD0FB7365053DEF8C35116D7C92D203C64A3D3EE2452A025223EEB166901C40A

---
[**Azure Government :**](https://azure.microsoft.com/features/gov/)

KeyVault-BYOK-Tools-USGovCloud.zip

F8DB2FC914A7360650922391D9AA79FF030FD3048B5795EC83ADC59DB018621A

---
**Département de la Défense des États-Unis :**

KeyVault-BYOK-Tools-USGovernmentDoD.zip

A79DD8C6DFFF1B00B91D1812280207A205442B3DDF861B79B8B991BB55C35263

---
**Canada :**

KeyVault-BYOK-Tools-Canada.zip

61BE1A1F80AC79912A42DEBBCC42CF87C88C2CE249E271934630885799717C7B

---
**Allemagne :**

KeyVault-BYOK-Tools-Germany.zip

5385E615880AAFC02AFD9841F7BADD025D7EE819894AA29ED3C71C3F844C45D6

---
**Allemagne (version publique) :**

KeyVault-BYOK-Tools-Germany-Public.zip

54534936D0A0C99C8117DB724C34A5E50FD204CFCBD75C78972B785865364A29

---
**Inde :**

KeyVault-BYOK-Tools-India.zip

49EDCEB3091CF1DF7B156D5B495A4ADE1CFBA77641134F61B0E0940121C436C8

---
**France :**

KeyVault-BYOK-Tools-France.zip

5C9D1F3E4125B0C09E9F60897C9AE3A8B4CB0E7D13A14F3EDBD280128F8FE7DF

---
**Royaume-Uni :**

KeyVault-BYOK-Tools-UnitedKingdom.zip

432746BD0D3176B708672CCFF19D6144FCAA9E5EB29BB056489D3782B3B80849

---
**Suisse :**

KeyVault-BYOK-Tools-Switzerland.zip

88CF8D39899E26D456D4E0BC57E5C94913ABF1D73A89013FCE3BBD9599AD2FE9

---


Pour valider l’intégrité de votre jeux d’outils BYOK, dans votre session Azure PowerShell, utilisez l’applet de commande [Get-FileHash](/powershell/module/microsoft.powershell.utility/get-filehash) .

   ```powershell
   Get-FileHash KeyVault-BYOK-Tools-*.zip
   ```

L’ensemble d’outils inclut :

* Un package Key Exchange Key (KEK) dont le nom commence par **BYOK-KEK-pkg-**
* Un package Security World dont le nom commence par **BYOK-SecurityWorld-pkg-**
* Un script python nommé **verifykeypackage.py**.
* Un fichier exécutable de ligne de commande nommé **KeyTransferRemote.exe** et les DLL associées.
* Un package redistribuable Visual C++ nommé **vcredist_x64.exe**.

Copiez le package sur un lecteur USB ou autre support de stockage portable.

## <a name="step-2-prepare-your-disconnected-workstation"></a>Étape 2 : Préparer votre station de travail déconnectée

Pour cette deuxième étape, procédez comme suit sur la station de travail non connectée à un réseau (Internet ou votre réseau interne).

### <a name="step-21-prepare-the-disconnected-workstation-with-ncipher-nshield-hsm"></a>Étape 2.1 : Préparer la station de travail déconnectée avec un HSM nShield de nCipher

Installer le logiciel de support nCipher sur un ordinateur Windows, puis rattachez un module de sécurité matériel nShield de nCipher à cet ordinateur.

Assurez-vous que les outils nCipher se trouvent dans votre chemin d’accès ( **%nfast_home%\bin**). Tapez ensuite la commande suivante :

  ```cmd
  set PATH=%PATH%;"%nfast_home%\bin"
  ```

Pour plus d’informations, consultez le guide de l’utilisateur inclus avec le module de sécurité matériel nShield.

### <a name="step-22-install-the-byok-toolset-on-the-disconnected-workstation"></a>Étape 2.2 : Installer l’ensemble d’outils BYOK sur la station de travail déconnectée

Copiez le package d’outils BYOK de la clé USB ou de l’autre support de stockage portable, puis procédez comme suit :

1. Extrayez les fichiers du package téléchargé dans un dossier.
2. Depuis ce dossier, exécutez vcredist_x64.exe.
3. Suivez les instructions pour installer les composants d’exécution Visual C++ pour Visual Studio 2013.

## <a name="step-3-generate-your-key"></a>Étape 3 : Générer votre clé

Pour cette troisième étape, procédez comme suit sur la station de travail déconnectée. Pour que vous puissiez mener à bien cette étape, votre HSM doit être en mode d’initialisation. 

### <a name="step-31-change-the-hsm-mode-to-i"></a>Étape 3.1 : Définir le mode du HSM sur « I »

Si vous utilisez nCipher nShield Edge, pour changer le mode, procédez comme suit : 1. Utilisez le bouton Mode pour mettre le mode requis en surbrillance. 2. Dans un délai de quelques secondes, maintenez le bouton Clear (Effacer) enfoncé pendant deux à trois secondes. Si le mode est modifié, la LED du nouveau mode cesse de clignoter et reste allumée. La LED d’état peut clignoter de façon irrégulière pendant quelques secondes, puis clignoter régulièrement lorsque l’appareil est prêt. Sinon, l’appareil reste dans le mode actuel, avec la LED du mode approprié allumée.

### <a name="step-32-create-a-security-world"></a>Étape 3.2 : Créer un monde de sécurité

Démarrez une invite de commande et exécutez le programme de nouveau monde nCipher.

   ```cmd
    new-world.exe --initialize --cipher-suite=DLf3072s256mRijndael --module=1 --acs-quorum=2/3
   ```

Ce programme crée un fichier **Security World** à l’emplacement %NFAST_KMDATA%\local\world, qui correspond au dossier C:\ProgramData\nCipher\Key Management Data\local. Vous pouvez utiliser des valeurs différentes pour le quorum, mais dans notre exemple, vous êtes invité à entrer trois cartes vierges et codes confidentiels pour chacune d’elles. Par la suite, chacune des deux cartes offre un accès total au monde de sécurité. Ces cartes constituent l’ **ensemble de cartes administrateur** pour le nouveau monde de sécurité.

> [!NOTE]
> Si votre HSM ne prend pas en charge la suite de chiffrement DLf3072s256mRijndael la plus récente, vous pouvez remplacer --cipher-suite=DLf3072s256mRijndael par --cipher-suite=DLf1024s160mRijndael
> 
> Le monde de sécurité créé avec new-world.exe qui est fourni avec le logiciel nCipher version 12.50 n’est pas compatible avec cette procédure BYOK. Deux options sont disponibles :
> 1) Passez à la version 12.40.2 du logiciel nCipher pour créer un monde de sécurité.
> 2) Contactez le support nCipher afin d’obtenir un correctif pour la version 12.50 du logiciel, qui vous permet d’utiliser la version 12.40.2 de new-world.exe, qui est compatible avec cette procédure BYOK.

Faites ensuite ce qui suit :

* Sauvegardez le fichier de monde. Sécurisez et protégez le fichier de monde, les cartes administrateur et leurs codes confidentiels et assurez-vous que personne n’a accès à plusieurs cartes.

### <a name="step-33-change-the-hsm-mode-to-o"></a>Étape 3.3 : Définir le mode du HSM sur « O »

Si vous utilisez nCipher nShield Edge, pour changer le mode, procédez comme suit : 1. Utilisez le bouton Mode pour mettre le mode requis en surbrillance. 2. Dans un délai de quelques secondes, maintenez le bouton Clear (Effacer) enfoncé pendant deux à trois secondes. Si le mode est modifié, la LED du nouveau mode cesse de clignoter et reste allumée. La LED d’état peut clignoter de façon irrégulière pendant quelques secondes, puis clignoter régulièrement lorsque l’appareil est prêt. Sinon, l’appareil reste dans le mode actuel, avec la LED du mode approprié allumée.

### <a name="step-34-validate-the-downloaded-package"></a>Étape 3.4 : Valider le package téléchargé

Cette étape est facultative, mais recommandée afin que vous puissiez valider les éléments suivants :

* La clé KEK incluse dans le jeu d’outils a été générée à partir d’un HSM nShield de nCipher authentique.
* Le hachage du monde de sécurité inclus dans le jeu d’outils a été généré à partir d’un HSM nShield de nCipher authentique.
* La clé KEK n’est pas exportable.

> [!NOTE]
> Pour valider le package téléchargé, le module de sécurité matérielle doit être connecté, mis sous tension et disposer d’un monde de sécurité (comme celui que vous venez de créer).

Pour valider le package téléchargé :

1. Exécutez le script verifykeypackage.py en tapant l’une des valeurs suivantes, selon votre région géographique ou votre instance d’Azure :

   * Pour l’Amérique du Nord :

      ```azurepowershell
      "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-NA-1 -w BYOK-SecurityWorld-pkg-NA-1
      ```
   * Pour l’Europe :

      ```azurepowershell
      "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-EU-1 -w BYOK-SecurityWorld-pkg-EU-1
      ```
   * Pour l’Asie :

        ```azurepowershell
        "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-AP-1 -w BYOK-SecurityWorld-pkg-AP-1
        ```
   * Pour l’Amérique latine :

        ```azurepowershell
        "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-LATAM-1 -w BYOK-SecurityWorld-pkg-LATAM-1
        ```
   * Pour le Japon :

        ```azurepowershell
        "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-JPN-1 -w BYOK-SecurityWorld-pkg-JPN-1
        ```
   * Pour la Corée :

        ```azurepowershell
        "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-KOREA-1 -w BYOK-SecurityWorld-pkg-KOREA-1
        ```
   * Pour l’Afrique du Sud :

        ```azurepowershell
        "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-SA-1 -w BYOK-SecurityWorld-pkg-SA-1
        ```
   * Pour les Émirats Arabes Unis :

        ```azurepowershell
        "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-UAE-1 -w BYOK-SecurityWorld-pkg-UAE-1
        ```
   * Pour l’Australie :

        ```azurepowershell
        "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-AUS-1 -w BYOK-SecurityWorld-pkg-AUS-1
        ```
   * Pour [Azure Government](https://azure.microsoft.com/features/gov/), qui utilise l’instance du gouvernement américain d’Azure :

        ```azurepowershell
        "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-USGOV-1 -w BYOK-SecurityWorld-pkg-USGOV-1
        ```
   * Pour le Département de la Défense des États-Unis :

        ```azurepowershell
        "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-USDOD-1 -w BYOK-SecurityWorld-pkg-USDOD-1
        ```
   * Pour le Canada :

        ```azurepowershell
        "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-CANADA-1 -w BYOK-SecurityWorld-pkg-CANADA-1
        ```
   * Pour l’Allemagne :

        ```azurepowershell
        "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-GERMANY-1 -w BYOK-SecurityWorld-pkg-GERMANY-1
        ```
   * Pour Allemagne (version publique) :

        ```azurepowershell
        "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-GERMANY-1 -w BYOK-SecurityWorld-pkg-GERMANY-1
        ```
   * Pour l’Inde :

      ```azurepowershell
      "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-INDIA-1 -w BYOK-SecurityWorld-pkg-INDIA-1
      ```
   * Pour la France :

        ```azurepowershell
        "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-FRANCE-1 -w BYOK-SecurityWorld-pkg-FRANCE-1
        ```
   * Pour le Royaume-Uni :

        ```azurepowershell
        "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-UK-1 -w BYOK-SecurityWorld-pkg-UK-1
        ```
   * Pour la Suisse :

        ```azurepowershell
        "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-SUI-1 -w BYOK-SecurityWorld-pkg-SUI-1
        ```

     > [!TIP]
     > Le logiciel nCipher nShield inclut python dans %NFAST_HOME%\python\bin
     >
     >
2. Vérifiez que vous voyez bien ce qui suit, ce qui indique que la validation est réussie : **Résultat : RÉUSSITE**

Ce script valide la chaîne de signataire jusqu’à la clé racine nShield. Le hachage de cette clé racine est incorporé dans le script et sa valeur doit être **59178a47 de508c3f 291277ee 184f46c4 f1d9c639**. Vous pouvez aussi confirmer cette valeur séparément en vous rendant sur le [site web de nCipher](https://www.ncipher.com).

Vous êtes maintenant prêt à créer une clé.

### <a name="step-35-create-a-new-key"></a>Étape 3.5 : Créer une clé

Générez une clé à l’aide du programme **generatekey** de nCipher nShield.

Exécutez la commande suivante pour générer la clé :

```azurepowershell
generatekey --generate simple type=RSA size=2048 protect=module ident=contosokey plainname=contosokey nvram=no pubexp=
```

Lorsque vous exécutez cette commande, utilisez ces instructions :

* Le paramètre *protect* doit être défini sur la valeur **module**, comme indiqué. Ce paramétrage crée une clé protégée par module. Le package d’outils BYOK ne prend pas en charge les clés protégées par OCS.
* Remplacez la valeur *contosokey* d’**ident** et de **plainname** par n’importe quelle valeur de chaîne. Pour réduire la charge administrative et réduire le risque d’erreurs, nous vous recommandons d’utiliser la même valeur pour les deux. La valeur **ident** doit contenir uniquement des chiffres, des tirets et des lettres minuscules.
* Le pubexp est laissé vide (par défaut) dans cet exemple, mais vous pouvez définir des valeurs spécifiques. Pour plus d’informations, consultez la [documentation nCipher](https://www.entrust.com/-/media/documentation/brochures/entrust-nshield-general-purpose-hsms-br-a4.pdf).

Cette commande crée un fichier de clé tokenisée dans le dossier %NFAST_KMDATA%\local avec un nom commençant par **key_simple_** suivi de l’**ident** spécifié dans la commande. Par exemple : **key_simple_contosokey**. Ce fichier contient une clé chiffrée.

Sauvegardez ce fichier de clé à jeton dans un emplacement sûr.

> [!IMPORTANT]
> Lorsque vous transférez par la suite la clé dans le coffre de clés Azure, Microsoft ne peut pas vous la renvoyer, ce qui fait qu’il est extrêmement important de sauvegarder votre clé et votre monde de sécurité. Contactez [nCipher](https://www.ncipher.com/about-us/contact-us) pour connaître les conseils et meilleures pratiques pour sauvegarder votre clé.
>


Vous êtes maintenant prêt à transférer votre clé vers coffre de clés Azure.

## <a name="step-4-prepare-your-key-for-transfer"></a>Étape 4 : Préparer votre clé pour le transfert

Pour cette quatrième étape, procédez comme suit sur la station de travail déconnectée.

### <a name="step-41-create-a-copy-of-your-key-with-reduced-permissions"></a>Étape 4.1 : Créer une copie de votre clé avec des autorisations réduites

Ouvrez une nouvelle invite de commandes et remplacez le répertoire actuel par l’emplacement où vous avez décompressé le fichier zip BYOK. Pour limiter les autorisations sur votre clé, dans l’invite de commande, exécutez l’une des opérations suivantes, en fonction de votre région géographique ou de votre instance d’Azure :

* Pour l’Amérique du Nord :

   ```azurepowershell
   KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-NA-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-NA-
   ```
* Pour l’Europe :

   ```azurepowershell
   KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-EU-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-EU-1
   ```
* Pour l’Asie :

   ```azurepowershell
   KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-AP-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-AP-1
   ```
* Pour l’Amérique latine :

   ```azurepowershell
   KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-LATAM-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-LATAM-1
   ```
* Pour le Japon :

   ```azurepowershell
   KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-JPN-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-JPN-1
   ```
* Pour la Corée :

   ```azurepowershell
   KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-KOREA-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-KOREA-1
   ```
* Pour l’Afrique du Sud :

   ```azurepowershell
   KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-SA-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-SA-1
   ```
* Pour les Émirats Arabes Unis :

   ```azurepowershell
   KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-UAE-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-UAE-1
   ```
* Pour l’Australie :

   ```azurepowershell
   KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-AUS-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-AUS-1
   ```
* Pour [Azure Government](https://azure.microsoft.com/features/gov/), qui utilise l’instance du gouvernement américain d’Azure :

   ```azurepowershell
   KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-USGOV-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-USGOV-1
   ```
* Pour le Département de la Défense des États-Unis :

   ```azurepowershell
   KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-USDOD-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-USDOD-1
   ```
* Pour le Canada :

   ```azurepowershell
   KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-CANADA-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-CANADA-1
   ```
* Pour l’Allemagne :

   ```azurepowershell
   KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-GERMANY-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-GERMANY-1
   ```
* Pour Allemagne (version publique) :

   ```azurepowershell
   KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-GERMANY-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-GERMANY-1
   ```
* Pour l’Inde :

   ```azurepowershell
   KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-INDIA-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-INDIA-1
   ```
* Pour la France :

   ```azurepowershell
   KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-FRANCE-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-FRANCE-1
   ```
* Pour le Royaume-Uni :

   ```azurepowershell
   KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-UK-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-UK-1
   ```
* Pour la Suisse :

   ```azurepowershell
   KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-SUI-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-SUI-1
   ```

Quand vous exécutez cette commande, remplacez *contosokey* par la valeur spécifiée à l’**Étape 3.5 : Créer une clé** de l’étape [Générer votre clé](#step-3-generate-your-key).

Vous êtes invité à connecter vos cartes d’administrateur du monde de sécurité.

Une fois la commande exécutée, vous voyez **Résultat : RÉUSSITE**. La copie de votre clé et des autorisations réduites figurent dans le fichier nommé key_xferacId_\<contosokey>.

Vous pouvez inspecter les listes de contrôle d’accès en utilisant les commandes suivantes dans les services nCipher nShield :

* aclprint.py :

   ```cmd
   "%nfast_home%\bin\preload.exe" -m 1 -A xferacld -K contosokey "%nfast_home%\python\bin\python" "%nfast_home%\python\examples\aclprint.py"
   ```
* kmfile-dump.exe :

   ```cmd
   "%nfast_home%\bin\kmfile-dump.exe" "%NFAST_KMDATA%\local\key_xferacld_contosokey"
   ```
  Quand vous exécutez ces commandes, remplacez contosokey par la valeur spécifiée à l’**Étape 3.5 : Créer une clé** de l’étape [Générer votre clé](#step-3-generate-your-key).

### <a name="step-42-encrypt-your-key-by-using-microsofts-key-exchange-key"></a>Étape 4.2 : Chiffrer votre clé à l’aide d’une clé KEK (Key Exchange Key) de Microsoft

Exécutez l’un des commandes suivantes, en fonction de votre région géographique ou de votre instance d’Azure :

* Pour l’Amérique du Nord :

   ```azurepowershell
   KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-NA-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-NA-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
   ```
* Pour l’Europe :

   ```azurepowershell
   KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-EU-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-EU-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
   ```
* Pour l’Asie :

   ```azurepowershell
   KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-AP-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-AP-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
   ```
* Pour l’Amérique latine :

   ```azurepowershell
   KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-LATAM-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-LATAM-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
   ```
* Pour le Japon :

   ```azurepowershell
   KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-JPN-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-JPN-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
   ```
* Pour la Corée :

   ```azurepowershell
   KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-KOREA-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-KOREA-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
   ```
* Pour l’Afrique du Sud :

   ```azurepowershell
   KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-SA-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-SA-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
   ```
* Pour les Émirats Arabes Unis :

   ```azurepowershell
   KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-UAE-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-UAE-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
   ```
* Pour l’Australie :

   ```azurepowershell
   KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-AUS-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-AUS-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
   ```
* Pour [Azure Government](https://azure.microsoft.com/features/gov/), qui utilise l’instance du gouvernement américain d’Azure :

   ```azurepowershell
   KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-USGOV-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-USGOV-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
   ```
* Pour le Département de la Défense des États-Unis :

   ```azurepowershell
   KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-USDOD-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-USDOD-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
   ```
* Pour le Canada :

   ```azurepowershell
   KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-CANADA-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-CANADA-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
   ```
* Pour l’Allemagne :

   ```azurepowershell
   KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-GERMANY-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-GERMANY-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
   ```
* Pour Allemagne (version publique) :

   ```azurepowershell
   KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-GERMANY-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-GERMANY-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
   ```
* Pour l’Inde :

   ```azurepowershell
   KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-INDIA-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-INDIA-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
   ```
* Pour la France :

   ```azurepowershell
   KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-France-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-France-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
   ```
* Pour le Royaume-Uni :

   ```azurepowershell
   KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-UK-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-UK-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
   ```
* Pour la Suisse :

  ```azurepowershell
  KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-SUI-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-SUI-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
  ```


Lorsque vous exécutez cette commande, utilisez ces instructions :

* Remplacez *contosokey* par l’identificateur utilisé pour générer la clé à l’**Étape 3.5 : Créer une clé** de l’étape [Générer votre clé](#step-3-generate-your-key).
* Remplacezv *SubscriptionID* par l’ID d’abonnement Azure qui contient votre coffre de clés. Vous avez récupéré cette valeur précédemment, à l’**Étape 1.2 : Obtenir votre ID d’abonnement Azure** de l’étape [Préparer votre station de travail connectée à Internet](#step-1-prepare-your-internet-connected-workstation).
* Remplacez *ContosoFirstHSMKey* par une étiquette à utiliser pour votre nom de fichier de sortie.

Une fois cette opération accomplie, le message **Résultat : RÉUSSITE**. s’affiche et un nouveau fichier est ajouté au dossier actuel, dont le nom est : KeyTransferPackage-*ContosoFirstHSMkey*.byok

### <a name="step-43-copy-your-key-transfer-package-to-the-internet-connected-workstation"></a>Étape 4.3 : Copier votre package de transfert de clé sur la station de travail connectée à Internet

Utilisez une clé USB ou un autre dispositif de stockage portable pour copier le fichier de sortie issu de la première étape (KeyTransferPackage-ContosoFirstHSMkey.byok) pour votre station de travail connectée à Internet.

## <a name="step-5-transfer-your-key-to-azure-key-vault"></a>Étape 5 : Transférer votre clé vers Azure Key Vault

Pour l’opération finale, sur la station de travail connectée à Internet, utilisez l’applet de commande [Add-AzKeyVaultKey](/powershell/module/az.keyvault/add-azkeyvaultkey) pour télécharger le package de transfert de clé copié à partir de la station de travail non connectée vers le module de sécurité matériel (HSM) d’Azure Key Vault :

   ```powershell
        Add-AzKeyVaultKey -VaultName 'ContosoKeyVaultHSM' -Name 'ContosoFirstHSMkey' -KeyFilePath 'c:\KeyTransferPackage-ContosoFirstHSMkey.byok' -Destination 'HSM'
   ```

Si le téléchargement réussit, les propriétés de la clé que vous venez de créer s’afficheront.

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez maintenant utiliser cette clé protégée HSM dans votre coffre de clés. Pour plus d’informations, rendez-vous sur le prix et utilisez la fonctionnalité [comparaison](https://azure.microsoft.com/pricing/details/key-vault/).
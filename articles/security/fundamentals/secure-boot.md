---
title: Démarrage sécurisé des microprogrammes – Sécurité Azure
description: Présentation technique du démarrage sécurisé des microprogrammes Azure.
author: yosharm
ms.service: security
ms.subservice: security-fundamentals
ms.topic: article
ms.author: terrylan
manager: rkarlin
ms.date: 11/10/2020
ms.openlocfilehash: feb28b1d448d0146046ed789d1389a3a42f344de
ms.sourcegitcommit: dc342bef86e822358efe2d363958f6075bcfc22a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94557349"
---
# <a name="secure-boot"></a>Démarrage sécurisé

Le démarrage sécurisé est une fonctionnalité d’[Unified Extensible Firmware Interface](https://en.wikipedia.org/wiki/Unified_Extensible_Firmware_Interface) (UEFI) qui requiert la vérification de tous les composants de bas niveau des microprogrammes et des logiciels avant le chargement. Lors du démarrage, le démarrage sécurisé UEFI vérifie la signature de chaque logiciel de démarrage, y compris les pilotes de microprogramme UEFI (également appelés ROM optionnels), les applications Extensible Firmware Interface (EFI), ainsi que les pilotes et fichiers binaires du système d’exploitation. Si les signatures sont valides ou approuvées par le fabricant d’équipement d’origine (OEM, Original Equipment Manufacturer), la machine démarre et les microprogrammes donnent le contrôle au système d’exploitation.

## <a name="components-and-process"></a>Composants et concepts

Le démarrage sécurisé s’appuie sur ces composants essentiels :

- Clé de plateforme (PK) : établit une relation de confiance entre le propriétaire de la plateforme (Microsoft) et les microprogrammes. La moitié publique est PKpub et la moitié privée est PKpriv.
- Base de données des clés d’inscription de clé (KEK) : établit une relation de confiance entre le système d’exploitation et les microprogrammes de la plateforme. La moitié publique est KEKpub et la moitié privée est KEKpriv.
- Base de données de signatures (db) : contient les codes de hachage des signataires approuvés (clés publiques et certificats) des modules de code des microprogrammes et des logiciels autorisés à interagir avec le microprogramme de la plateforme.
- Base de données de signatures révoquées (dbx) : contient les codes de hachage révoqués de modules de code qui ont été identifiés comme étant malveillants, vulnérables, compromis ou non fiables. Si un code de hachage se trouve à la fois dans la base de données des signatures et dans la base de données des signatures révoquées, cette dernière prévaut.

La figure et le processus ci-dessous expliquent la façon dont ces composants sont mis à jour :

![Diagramme illustrant les composants du démarrage sécurisé.](./media/secure-boot/secure-boot.png)

L’OEM stocke les codes de hachage du démarrage sécurisé sur la RAM non volatile (NV-RAM) de l’ordinateur au moment de la fabrication.

1. La base de données de signatures (db) est alimentée par les codes de hachage de signataires ou d’image des applications UEFI, des chargeurs de système d’exploitation (par exemple, le chargeur du système d’exploitation Microsoft ou le gestionnaire de démarrage) et des pilotes UEFI approuvés.
2. La base de données de signatures révoquées (dbx) est alimentée par les codes de hachage des modules qui ne sont plus approuvés.
3. La base de données de clés d’inscription de clé (KEK) est alimentée par les clés de signature qui peuvent être utilisées pour mettre à jour la base de données de signatures et la base de données de signatures révoquées. Les bases de données peuvent être modifiées par le biais de mises à jour signées avec la clé appropriée ou par le biais de mises à jour par un utilisateur autorisé présent physiquement à l’aide des menus des microprogrammes.
4. Une fois que les bases de données db, dbx et KEK ont été ajoutées et que la validation et le test final des microprogrammes sont terminés, l’OEM verrouille les microprogrammes pour les empêcher d’être modifiés et génère une clé de plateforme (PK). La PK peut être utilisée pour signer des mises à jour de la KEK ou pour désactiver le démarrage sécurisé.

À chaque étape du processus de démarrage, les code de hachage des microprogrammes, du chargeur d’amorçage, du système d’exploitation, des pilotes de noyau et d’autres artefacts de la chaîne de démarrage sont calculés et comparés aux valeurs acceptables. Les microprogrammes et les logiciels détectés comme étant non fiables ne sont pas autorisés à être chargés. Ainsi, l’injection de programmes malveillants de bas niveau ou les attaques de programmes malveillants avant le démarrage peuvent être bloquées.

## <a name="secure-boot-on-the-azure-fleet"></a>Démarrage sécurisé sur la flotte Azure
Aujourd’hui, chaque machine intégrée à la flotte de calcul Azure et déployée sur celle-ci pour héberger les charges de travail des clients sortent des ateliers avec le démarrage sécurisé activé. Des outils et des processus ciblés sont en place à chaque étape de la construction et de l’intégration du matériel pour garantir que l’activation de Secure Boot n’est pas annulée par accident ou par malveillance.

Valider que les codes de hachage db et dbx sont corrects garantit que :

- Le chargeur d’amorçage est présent dans l’une des entrées db
- La signature du chargeur d’amorçage est valide
- L’hôte démarre avec un logiciel de confiance

 En validant les signatures de KEKpub et PKpub, nous pouvons vérifier que seules les parties approuvées sont autorisées à modifier les définitions des logiciels considérés comme étant dignes de confiance. Enfin, en veillant à ce que le démarrage sécurisé soit actif, nous pouvons vérifier que ces définitions sont appliquées.

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur ce que nous faisons pour assurer l’intégrité et la sécurité de la plateforme, consultez :

- [Sécurité de microprogramme](firmware.md)
- [Démarrage mesuré et attestation de l’hôte](measured-boot-host-attestation.md)
- [Project Cerberus](project-cerberus.md)
- [Chiffrement au repos](encryption-atrest.md)
- [Sécurité de l’hyperviseur](hypervisor.md)
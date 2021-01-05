---
title: À propos du domaine de sécurité Azure Managed HSM
description: Vue d’ensemble du domaine de sécurité Managed HSM, un ensemble d’informations d’identification de base nécessaires à la reprise d’activité d’un HSM managé
ms.service: key-vault
ms.subservice: managed-hsm
ms.topic: conceptual
author: amitbapat
ms.author: ambapat
ms.date: 09/15/2020
ms.openlocfilehash: 37e2541d0b53c96fd3f85da31b2c0ce5b68b551a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90992735"
---
# <a name="about-the-managed-hsm-security-domain"></a>À propos du domaine de sécurité Managed HSM

Le domaine de sécurité Managed HSM est un ensemble d’informations d’identification de base nécessaires à la reprise d’activité d’un HSM managé en cas de sinistre. Le domaine de sécurité est généré dans le matériel HSM managé et le logiciel du service enclave et représente la « propriété » du HSM.

Chaque HSM managé doit avoir un domaine de sécurité pour fonctionner. Quand vous demandez un nouveau HSM managé, il est provisionné mais pas activé tant que vous n’avez pas téléchargé le domaine de sécurité. Quand un HSM managé est provisionné, mais pas activé, il existe deux façons de l’activer :
- Le téléchargement de votre domaine de sécurité est la méthode par défaut. Elle vous permet de stocker de façon sécurisée le domaine de sécurité pour l’utiliser avec un autre HSM managé ou pour une reprise d’activité après un sinistre total.
- Chargez un domaine de sécurité existant que vous avez déjà, ce qui vous permet de créer plusieurs instances de HSM managé qui partagent le même domaine de sécurité.

## <a name="download-your-security-domain"></a>Télécharger votre domaine de sécurité

Quand un HSM managé est provisionné mais pas activé, le téléchargement du domaine de sécurité permet de capturer les informations d’identification de base nécessaires à une reprise d’activité après une perte complète de tout le matériel. Pour télécharger le domaine de sécurité, vous devez créer au moins 3 paires de clés RSA (maximum 10) et envoyer ces clés publiques au service lors de la demande de téléchargement du domaine de sécurité. Vous devez également spécifier le nombre minimal de clés nécessaires (quorum) pour déchiffrer le domaine de sécurité à l’avenir. Le HSM managé va initialiser le domaine de sécurité et le chiffrer avec les clés publiques que vous fournissez à l’aide de l’[algorithme de partage de clé secrète de Shamir](https://dl.acm.org/doi/10.1145/359168.359176). L’objet blob du domaine de sécurité que vous téléchargez peut uniquement être déchiffré quand au moins un quorum des clés privées est disponible. Vous devez sécuriser les clés privées pour garantir la sécurité du domaine de sécurité. Une fois le téléchargement terminé, le HSM managé est activé et prêt à être utilisé.  

> [!IMPORTANT]
> Pour une reprise d’activité complète, vous devez disposer du domaine de sécurité et du quorum des clés privées qui ont été utilisées pour le protéger, ainsi que d’une sauvegarde HSM complète. Si vous perdez le domaine de sécurité ou si vous n’avez pas suffisamment de clés RSA (clé privée) pour constituer le quorum et qu’aucune instance en cours d’exécution du HSM managé n’est présente, la reprise d’activité n’est pas possible.

## <a name="upload-a-security-domain"></a>Charger un domaine de sécurité

Quand un HSM managé est provisionné mais pas activé, vous pouvez lancer un processus de reprise d’activité du domaine de sécurité. Le HSM managé va générer une paire de clés RSA et retourner la clé publique. Ensuite, vous pouvez charger le domaine de sécurité dans le HSM managé. Avant le chargement, le client (Azure CLI ou PowerShell) a besoin de connaître le nombre minimal de clés privées (quorum) que vous avez utilisées lors du téléchargement du domaine de sécurité. Le client va déchiffrer le domaine de sécurité à l’aide de ce quorum et le rechiffrer à l’aide de la clé publique que vous avez téléchargée quand vous avez demandé la reprise d’activité. Une fois le chargement terminé, le HSM managé est dans l’état activé.

## <a name="backup-and-restore-behavior"></a>Comportement de sauvegarde et de restauration

Les sauvegardes (sauvegarde complète ou sauvegarde d’une clé unique) peuvent être restaurées correctement uniquement si le HSM managé source (où la sauvegarde a été créée) et le HSM managé de destination (où la sauvegarde va être restaurée) partagent le même domaine de sécurité. Ainsi, un domaine de sécurité définit également une limite de chiffrement pour chaque HSM managé.

## <a name="next-steps"></a>Étapes suivantes

- Lire une [présentation d’un HSM managé](overview.md)
- En savoir plus sur la [gestion d’un HSM managé avec Azure CLI](key-management.md)
- Passer en revue les [bonnes pratiques liées à un HSM managé](best-practices.md)

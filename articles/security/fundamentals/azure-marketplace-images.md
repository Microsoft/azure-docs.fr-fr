---
title: Recommandations de sécurité pour les images Place de marché Microsoft Azure | Microsoft Docs
description: Cet article fournit des recommandations pour les images incluses sur la Place de marché
services: security
documentationcenter: na
author: terrylanfear
manager: barbkess
ms.assetid: ''
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.date: 01/11/2019
ms.author: terrylan
ms.openlocfilehash: 7c317a0b4fea0c981b227bace00c1b8924fd582c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89536380"
---
# <a name="security-recommendations-for-azure-marketplace-images"></a>Recommandations de sécurité pour les images Place de marché Microsoft Azure

Votre image doit répondre à ces recommandations en matière de configuration de la sécurité. Cela permet de maintenir un niveau élevé de sécurité pour les images de solutions partenaires dans la Place de marché.

Exécutez toujours une détection de vulnérabilité de sécurité sur votre image avant de la soumettre. Si vous détectez une vulnérabilité de sécurité dans votre propre image publiée, vous devez informer vos clients en temps opportun de la vulnérabilité et de la manière de la corriger.

## <a name="open-source-based-images"></a>Images open source

| Category | Vérification |
| -------- | ----- |
| Sécurité                                                     | Installez tous les derniers correctifs de sécurité pour la distribution de Linux.                                                                                                                                                                                                              |
| Sécurité                                                     | Suivez les bonnes pratiques du secteur pour sécuriser l’image de machine virtuelle pour votre distribution de Linux spécifique.                                                                                                                                                                                     |
| Sécurité                                                     | Limitez la surface d’attaque en conservant un encombrement minimal avec seulement les rôles Windows Server, fonctionnalités, services et ports réseau strictement nécessaires.                                                                                                                                               |
| Sécurité                                                     | Analysez le code source et l’image de machine virtuelle qui en résulte à la recherche de programmes malveillants.                                                                                                                                                                                                                                   |
| Sécurité                                                     | L’image de disque dur virtuel inclut uniquement les comptes verrouillés nécessaires qui n’ont pas de mots de passe par défaut et qui permettent une connexion interactive ; pas de portes dérobées.                                                                                                                                           |
| Sécurité                                                     | Désactivez les règles de pare-feu, sauf si l’application compte fonctionnellement sur elles, par exemple dans le cas d’un appareil de pare-feu.                                                                                                                                                                             |
| Sécurité                                                     | Supprimez toutes les informations sensibles de l’image de disque dur virtuel, comme les clés SSH de test, le fichier d’hôtes connus, les fichiers journaux et les certificats inutiles.                                                                                                                                       |
| Sécurité                                                     | Évitez d’utiliser LVM.                                                                                                                                                                                                                                            |
| Sécurité                                                     | Incluez les versions les plus récentes des bibliothèques requises : </br> - OpenSSL v1.0 ou version ultérieure </br> - Python 2.5 ou version ultérieure (Python 2.6+ est vivement recommandé) </br> - Package Python pyasn1, le cas échéant </br> - d.OpenSSL v 1.0 ou version ultérieure                                                                |
| Sécurité                                                     | Effacez les entrées de l’historique de Bash/l’interpréteur de commandes.                                                                                                                                                                                                                                             |
| Mise en réseau                                                   | Incluez le serveur SSH par défaut. Réglez l’option de maintien de la connexion au fichier de configuration du serveur SSH avec l’option suivante : ClientAliveInterval 180.                                                                                                                                                        |
| Mise en réseau                                                   | Supprimez toute configuration réseau personnalisée de l’image. Supprimez le fichier resolv.conf :`rm /etc/resolv.conf`.                                                                                                                                                                                |
| Déploiement                                                   | Installez l’agent Linux Azure le plus récent.</br> - Réalisez l’installation à l’aide du package RPM ou Deb.  </br> - Vous pouvez également utiliser le processus d’installation manuelle, mais les packages d’installation sont recommandés et préférés. </br> - Si vous installez l’agent manuellement à partir du référentiel GitHub, vous devez tout d’abord copier le fichier `waagent` sur `/usr/sbin` et exécuter (en tant que root) : </br>`# chmod 755 /usr/sbin/waagent` </br>`# /usr/sbin/waagent -install` </br>Le fichier de configuration de l’agent sera placé dans `/etc/waagent.conf`. |
| Déploiement                                                   | Assurez-vous que le support Azure peut fournir à nos partenaires une sortie de console série lorsque cela est nécessaire et indiquez un délai d’expiration adapté au montage du disque de système d’exploitation à partir du stockage cloud. Ajoutez les paramètres suivants à la ligne de démarrage du noyau de l’image :`console=ttyS0 earlyprintk=ttyS0 rootdelay=300`. |
| Déploiement                                                   | Aucune partition d’échange n’est présente sur le disque du système d’exploitation. L’échange peut être demandé pour la création sur le disque de la ressource locale par l’agent Linux.         |
| Déploiement                                                   | Créez une partition racine unique pour le disque de système d’exploitation.      |
| Déploiement                                                   | Système d’exploitation 64 bits uniquement.                                                                                                                                                                                                                                                          |

## <a name="windows-server-based-images"></a>Images Windows Server

| Category | Vérification |
|--------- | ----- |
| Sécurité                                                         | Utilisez une image de base de système d’exploitation sécurisée. Le disque dur virtuel utilisé pour la source de n’importe quelle image basée sur Windows Server doit venir d’images de système d’exploitation Windows Server obtenues via Microsoft Azure. |
| Sécurité                                                         | Installez toutes les dernières mises à jour de sécurité.                                                                                                                                     |
| Sécurité                                                         | Les applications ne doivent pas dépendre de noms d’utilisateur restreints tels que « administrator », « root » et « admin ».                                                                |
| Sécurité                                                         | Activez le Chiffrement de lecteur BitLocker pour les disques durs du système d’exploitation et les disques durs de données.                                                             |
| Sécurité                                                         | Limitez la surface d’attaque en conservant un encombrement minimal avec seulement les rôles Windows Server, fonctionnalités, services et ports réseau strictement nécessaires activés.                         |
| Sécurité                                                         | Analysez le code source et l’image de machine virtuelle qui en résulte à la recherche de programmes malveillants.                                                                                                                     |
| Sécurité                                                         | Réglez les mises à jour de sécurité pour les images Windows Server pour qu’elles se fassent automatiquement.                                                                                                                |
| Sécurité                                                         | L’image de disque dur virtuel inclut uniquement les comptes verrouillés nécessaires qui n’ont pas de mots de passe par défaut et qui permettent une connexion interactive ; pas de portes dérobées.                             |
| Sécurité                                                         | Désactivez les règles de pare-feu, sauf si l’application compte fonctionnellement sur elles, par exemple dans le cas d’un appareil de pare-feu.                                                               |
| Sécurité                                                         | Supprimez toutes les informations sensibles de l’image de disque dur virtuel, notamment les fichiers HOSTS, les fichiers journaux et les certificats inutiles.                                              |
| Déploiement                                                       | Système d’exploitation 64 bits uniquement.                            |

Même si votre organisation ne dispose pas d’images dans la Place de marché Azure, pensez à vérifier les configurations de vos images Windows et Linux par rapport à ces recommandations.


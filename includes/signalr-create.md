---
title: Fichier Include
description: Fichier Include
services: signalr
author: wesmc7777
ms.service: signalr
ms.topic: include
ms.date: 04/17/2018
ms.author: wesmc
ms.custom: include file
ms.openlocfilehash: e5cfc9beb5473917a76f822862ce3d61675d6493
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93406653"
---
1. Pour créer une ressource SignalR Service Azure, commencez par vous connecter au [portail Azure](https://portal.azure.com). En haut à gauche de la page, sélectionnez **+ Créer une ressource**. Dans la zone de texte **Rechercher dans la Place de marché**, entrez **SignalR Service**.

2. Sélectionnez **SignalR Service** dans les résultats, puis choisissez **Créer**.

3. Sur la nouvelle page de paramètres **SignalR**, ajoutez les paramètres suivants pour votre nouvelle ressource SignalR :

    | Name | Valeur recommandée | Description |
    | ---- | ----------------- | ----------- |
    | Nom de la ressource | *testsignalr* | Entrez un nom de ressource unique à utiliser pour la ressource SignalR. Le nom doit être une chaîne de 1 à 63 caractères, et il ne peut contenir que des chiffres, des lettres et le caractère `-` (trait d'union). Le nom ne peut ni commencer ni se terminer par un trait d'union, et il n'accepte pas de traits d'union consécutifs.|
    | Subscription | Choisir votre abonnement |  Sélectionnez l’abonnement Azure que vous souhaitez utiliser pour tester SignalR. Si votre compte a un seul abonnement, il est automatiquement sélectionné et la liste déroulante **Abonnement** ne s’affiche pas.|
    | Resource group | Créer un groupe de ressources nommé *SignalRTestResources*| Sélectionnez ou créez un groupe de ressources pour votre ressource SignalR. Ce groupe est utile pour organiser plusieurs ressources que vous souhaitez supprimer en même temps que vous supprimez ce groupe de ressources. Pour plus d’informations, consultez [Utilisation des groupes de ressources pour gérer vos ressources Azure](../articles/azure-resource-manager/management/overview.md). |
    | Location | *USA Est* | Utilisez **Emplacement** pour indiquer l’emplacement géographique de l’hébergement de votre ressource SignalR. Pour des performances optimales, nous vous recommandons de créer la ressource dans la même région que les autres composants de votre application. |
    | Niveau tarifaire | *Gratuit* | Actuellement, les options **Gratuit** et **Standard** sont disponibles. |
    | Épingler au tableau de bord | ✔ | Cochez cette case pour épingler la ressource au tableau de bord afin de pouvoir la retrouver plus facilement. |

4. Sélectionnez **Revoir + créer**. Attendez que la validation se termine. 

5. Sélectionnez **Create** (Créer). Le déploiement peut prendre quelques minutes.

6. Au terme du déploiement, sélectionnez **Clés** sous **PARAMÈTRES**. Copiez votre chaîne de connexion pour la clé primaire. Vous utiliserez cette chaîne ultérieurement pour configurer votre application de manière à utiliser la ressource Azure SignalR Service.

    La chaîne de connexion suit ce format :
    
    `Endpoint=<service_endpoint>;AccessKey=<access_key>;`

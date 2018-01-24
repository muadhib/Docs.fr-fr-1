---
title: "Immuabilité clée et la modification des paramètres"
author: rick-anderson
description: "Ce document décrit les détails d’implémentation de l’ASP.NET Core protection clée immuabilité des données API."
ms.author: riande
manager: wpickett
ms.date: 10/14/2016
ms.topic: article
ms.technology: aspnet
ms.prod: asp.net-core
uid: security/data-protection/implementation/key-immutability
ms.openlocfilehash: 8e46e634266fa5f082c47f3be306009eb54bcbcc
ms.sourcegitcommit: 3e303620a125325bb9abd4b2d315c106fb8c47fd
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/19/2018
---
# <a name="key-immutability-and-changing-settings"></a><span data-ttu-id="8525f-103">Immuabilité et la modification des paramètres de clé</span><span class="sxs-lookup"><span data-stu-id="8525f-103">Key Immutability and changing settings</span></span>

<span data-ttu-id="8525f-104">Une fois qu’un objet est persistant dans le magasin de sauvegarde, sa représentation est toujours fixe.</span><span class="sxs-lookup"><span data-stu-id="8525f-104">Once an object is persisted to the backing store, its representation is forever fixed.</span></span> <span data-ttu-id="8525f-105">Nouvelles données peuvent être ajoutées au magasin de stockage, mais les données existantes ne peuvent jamais être mutées.</span><span class="sxs-lookup"><span data-stu-id="8525f-105">New data can be added to the backing store, but existing data can never be mutated.</span></span> <span data-ttu-id="8525f-106">L’objectif principal de ce comportement est afin d’éviter une altération des données.</span><span class="sxs-lookup"><span data-stu-id="8525f-106">The primary purpose of this behavior is to prevent data corruption.</span></span>

<span data-ttu-id="8525f-107">Une conséquence de ce comportement est qu’une fois qu’une clé est écrite dans le magasin de stockage, il est immuable.</span><span class="sxs-lookup"><span data-stu-id="8525f-107">One consequence of this behavior is that once a key is written to the backing store, it is immutable.</span></span> <span data-ttu-id="8525f-108">Ses dates de création, d’activation et d’expiration ne sont pas modifiables, si elle peut révoquées à l’aide de `IKeyManager`.</span><span class="sxs-lookup"><span data-stu-id="8525f-108">Its creation, activation, and expiration dates can never be changed, though it can revoked by using `IKeyManager`.</span></span> <span data-ttu-id="8525f-109">En outre, ses informations algorithmiques sous-jacentes, support de gestion et le chiffrement sur les propriétés rest sont également immuables.</span><span class="sxs-lookup"><span data-stu-id="8525f-109">Additionally, its underlying algorithmic information, master keying material, and encryption at rest properties are also immutable.</span></span>

<span data-ttu-id="8525f-110">Si le développeur modifie un paramètre qui affecte la persistance des clés, ces modifications ne prendront effet jusqu'à la prochaine génération d’une clé, soit via un appel explicite à `IKeyManager.CreateNewKey` ou via de la protection système de données propre [clé automatique génération](key-management.md#data-protection-implementation-key-management) comportement.</span><span class="sxs-lookup"><span data-stu-id="8525f-110">If the developer changes any setting that affects key persistence, those changes will not go into effect until the next time a key is generated, either via an explicit call to `IKeyManager.CreateNewKey` or via the data protection system's own [automatic key generation](key-management.md#data-protection-implementation-key-management) behavior.</span></span> <span data-ttu-id="8525f-111">Les paramètres qui affectent la persistance des clés sont les suivantes :</span><span class="sxs-lookup"><span data-stu-id="8525f-111">The settings that affect key persistence are as follows:</span></span>

* [<span data-ttu-id="8525f-112">La durée de vie de clé par défaut</span><span class="sxs-lookup"><span data-stu-id="8525f-112">The default key lifetime</span></span>](key-management.md#data-protection-implementation-key-management)

* [<span data-ttu-id="8525f-113">Le chiffrement à clé au mécanisme de rest</span><span class="sxs-lookup"><span data-stu-id="8525f-113">The key encryption at rest mechanism</span></span>](key-encryption-at-rest.md#data-protection-implementation-key-encryption-at-rest)

* [<span data-ttu-id="8525f-114">Les informations algorithmiques contenues dans la clé</span><span class="sxs-lookup"><span data-stu-id="8525f-114">The algorithmic information contained within the key</span></span>](xref:security/data-protection/configuration/overview#changing-algorithms-with-usecryptographicalgorithms)

<span data-ttu-id="8525f-115">Si vous avez besoin de ces paramètres à bannir antérieures à la clé automatique suivante propagée temps, envisagez de faire un appel explicite à `IKeyManager.CreateNewKey` pour forcer la création d’une nouvelle clé.</span><span class="sxs-lookup"><span data-stu-id="8525f-115">If you need these settings to kick in earlier than the next automatic key rolling time, consider making an explicit call to `IKeyManager.CreateNewKey` to force the creation of a new key.</span></span> <span data-ttu-id="8525f-116">N’oubliez pas de fournir une date de l’activation explicite ({maintenant + 2 jours} est une règle empirique prévoir un délai pour propager les modifications) et la date d’expiration dans l’appel.</span><span class="sxs-lookup"><span data-stu-id="8525f-116">Remember to provide an explicit activation date ({ now + 2 days } is a good rule of thumb to allow time for the change to propagate) and expiration date in the call.</span></span>

>[!TIP]
> <span data-ttu-id="8525f-117">Toutes les applications toucher le référentiel doivent spécifier les mêmes paramètres avec le `IDataProtectionBuilder` les méthodes d’extension.</span><span class="sxs-lookup"><span data-stu-id="8525f-117">All applications touching the repository should specify the same settings with the `IDataProtectionBuilder` extension methods.</span></span> <span data-ttu-id="8525f-118">Dans le cas contraire, les propriétés de la clé persistante seront dépendantes de l’application spécifique qui a appelé les routines de la génération de clés.</span><span class="sxs-lookup"><span data-stu-id="8525f-118">Otherwise, the properties of the persisted key will be dependent on the particular application that invoked the key generation routines.</span></span>
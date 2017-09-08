---
title: "Extensibilidade da criptografia de núcleo"
author: rick-anderson
description: "Explica IAuthenticatedEncryptor, IAuthenticatedEncryptorDescriptor, IAuthenticatedEncryptorDescriptorDeserializer e a fábrica de nível superior."
keywords: "Núcleo do ASP.NET, IAuthenticatedEncryptor, IAuthenticatedEncryptorDescriptor, IAuthenticatedEncryptorDescriptorDeserializer"
ms.author: riande
manager: wpickett
ms.date: 8/11/2017
ms.topic: article
ms.technology: aspnet
ms.prod: asp.net-core
uid: security/data-protection/extensibility/core-crypto
ms.openlocfilehash: 8ee4e380b154db7f1736edc793b56258655ddd52
ms.sourcegitcommit: bd05f7ea8f87ad076ef6e8b704698ebcba5ca80c
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 08/25/2017
---
# <a name="core-cryptography-extensibility"></a><span data-ttu-id="76976-104">Extensibilidade da criptografia de núcleo</span><span class="sxs-lookup"><span data-stu-id="76976-104">Core cryptography extensibility</span></span>

<a name=data-protection-extensibility-core-crypto></a>

>[!WARNING]
> <span data-ttu-id="76976-105">Tipos que implementam qualquer uma das seguintes interfaces devem ser thread-safe para chamadores vários.</span><span class="sxs-lookup"><span data-stu-id="76976-105">Types that implement any of the following interfaces should be thread-safe for multiple callers.</span></span>

<a name=data-protection-extensibility-core-crypto-iauthenticatedencryptor></a>

## <a name="iauthenticatedencryptor"></a><span data-ttu-id="76976-106">IAuthenticatedEncryptor</span><span class="sxs-lookup"><span data-stu-id="76976-106">IAuthenticatedEncryptor</span></span>

<span data-ttu-id="76976-107">O **IAuthenticatedEncryptor** interface é o bloco de construção básico do subsistema de criptografia.</span><span class="sxs-lookup"><span data-stu-id="76976-107">The **IAuthenticatedEncryptor** interface is the basic building block of the cryptographic subsystem.</span></span> <span data-ttu-id="76976-108">Em geral, há um IAuthenticatedEncryptor por chave e a instância IAuthenticatedEncryptor encapsula todas as material de chave de criptografia e algoritmos informações necessárias para executar operações criptográficas.</span><span class="sxs-lookup"><span data-stu-id="76976-108">There is generally one IAuthenticatedEncryptor per key, and the IAuthenticatedEncryptor instance wraps all cryptographic key material and algorithmic information necessary to perform cryptographic operations.</span></span>

<span data-ttu-id="76976-109">Como o nome sugere, o tipo é responsável por fornecer serviços de criptografia e descriptografia autenticados.</span><span class="sxs-lookup"><span data-stu-id="76976-109">As its name suggests, the type is responsible for providing authenticated encryption and decryption services.</span></span> <span data-ttu-id="76976-110">Expõe as APIs a seguir.</span><span class="sxs-lookup"><span data-stu-id="76976-110">It exposes the following two APIs.</span></span>

* <span data-ttu-id="76976-111">Descriptografar (ArraySegment<byte> texto cifrado, ArraySegment<byte> additionalAuthenticatedData): byte]</span><span class="sxs-lookup"><span data-stu-id="76976-111">Decrypt(ArraySegment<byte> ciphertext, ArraySegment<byte> additionalAuthenticatedData) : byte[]</span></span>

* <span data-ttu-id="76976-112">Criptografar (ArraySegment<byte> texto sem formatação, ArraySegment<byte> additionalAuthenticatedData): byte]</span><span class="sxs-lookup"><span data-stu-id="76976-112">Encrypt(ArraySegment<byte> plaintext, ArraySegment<byte> additionalAuthenticatedData) : byte[]</span></span>

<span data-ttu-id="76976-113">O método Encrypt retorna um blob que inclui o texto não criptografado enciphered e uma marca de autenticação.</span><span class="sxs-lookup"><span data-stu-id="76976-113">The Encrypt method returns a blob that includes the enciphered plaintext and an authentication tag.</span></span> <span data-ttu-id="76976-114">A marca de autenticação deve abranger os dados adicionais autenticados (AAD), embora o AAD em si não precisa ser recuperável da carga final.</span><span class="sxs-lookup"><span data-stu-id="76976-114">The authentication tag must encompass the additional authenticated data (AAD), though the AAD itself need not be recoverable from the final payload.</span></span> <span data-ttu-id="76976-115">O método Decrypt valida a marca de autenticação e retorna a carga deciphered.</span><span class="sxs-lookup"><span data-stu-id="76976-115">The Decrypt method validates the authentication tag and returns the deciphered payload.</span></span> <span data-ttu-id="76976-116">Todas as falhas (exceto ArgumentNullException e semelhante) devem ser homogenized para CryptographicException.</span><span class="sxs-lookup"><span data-stu-id="76976-116">All failures (except ArgumentNullException and similar) should be homogenized to CryptographicException.</span></span>

> [!NOTE]
> <span data-ttu-id="76976-117">A própria instância IAuthenticatedEncryptor, na verdade, não precisa conter o material da chave.</span><span class="sxs-lookup"><span data-stu-id="76976-117">The IAuthenticatedEncryptor instance itself doesn't actually need to contain the key material.</span></span> <span data-ttu-id="76976-118">Por exemplo, a implementação pudesse ser delegado a um HSM para todas as operações.</span><span class="sxs-lookup"><span data-stu-id="76976-118">For example, the implementation could delegate to an HSM for all operations.</span></span>

<a name=data-protection-extensibility-core-crypto-iauthenticatedencryptorfactory></a>
<a name=data-protection-extensibility-core-crypto-iauthenticatedencryptordescriptor></a>

## <a name="how-to-create-an-iauthenticatedencryptor"></a><span data-ttu-id="76976-119">Como criar um IAuthenticatedEncryptor</span><span class="sxs-lookup"><span data-stu-id="76976-119">How to create an IAuthenticatedEncryptor</span></span>

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="76976-120">ASP.NET Core 2. x</span><span class="sxs-lookup"><span data-stu-id="76976-120">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)

<span data-ttu-id="76976-121">O **IAuthenticatedEncryptorFactory** interface representa um tipo que sabe como criar um [IAuthenticatedEncryptor](xref:security/data-protection/extensibility/core-crypto#data-protection-extensibility-core-crypto-iauthenticatedencryptor) instância.</span><span class="sxs-lookup"><span data-stu-id="76976-121">The **IAuthenticatedEncryptorFactory** interface represents a type that knows how to create an [IAuthenticatedEncryptor](xref:security/data-protection/extensibility/core-crypto#data-protection-extensibility-core-crypto-iauthenticatedencryptor) instance.</span></span> <span data-ttu-id="76976-122">Sua API é o seguinte.</span><span class="sxs-lookup"><span data-stu-id="76976-122">Its API is as follows.</span></span>

* <span data-ttu-id="76976-123">CreateEncryptorInstance (chave IKey): IAuthenticatedEncryptor</span><span class="sxs-lookup"><span data-stu-id="76976-123">CreateEncryptorInstance(IKey key) : IAuthenticatedEncryptor</span></span>

<span data-ttu-id="76976-124">Para qualquer instância específica de IKey os qualquer intermediária autenticada criada por seu método CreateEncryptorInstance deve ser consideradas equivalentes, como o exemplo de código abaixo.</span><span class="sxs-lookup"><span data-stu-id="76976-124">For any given IKey instance, any authenticated encryptors created by its CreateEncryptorInstance method should be considered equivalent, as in the below code sample.</span></span>

```csharp
// we have an IAuthenticatedEncryptorFactory instance and an IKey instance
IAuthenticatedEncryptorFactory factory = ...;
IKey key = ...;

// get an encryptor instance and perform an authenticated encryption operation
ArraySegment<byte> plaintext = new ArraySegment<byte>(Encoding.UTF8.GetBytes("plaintext"));
ArraySegment<byte> aad = new ArraySegment<byte>(Encoding.UTF8.GetBytes("AAD"));
var encryptor1 = factory.CreateEncryptorInstance(key);
byte[] ciphertext = encryptor1.Encrypt(plaintext, aad);

// get another encryptor instance and perform an authenticated decryption operation
var encryptor2 = factory.CreateEncryptorInstance(key);
byte[] roundTripped = encryptor2.Decrypt(new ArraySegment<byte>(ciphertext), aad);


// the 'roundTripped' and 'plaintext' buffers should be equivalent
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="76976-125">ASP.NET Core 1. x</span><span class="sxs-lookup"><span data-stu-id="76976-125">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)

<span data-ttu-id="76976-126">O **IAuthenticatedEncryptorDescriptor** interface representa um tipo que sabe como criar um [IAuthenticatedEncryptor](xref:security/data-protection/extensibility/core-crypto#data-protection-extensibility-core-crypto-iauthenticatedencryptor) instância.</span><span class="sxs-lookup"><span data-stu-id="76976-126">The **IAuthenticatedEncryptorDescriptor** interface represents a type that knows how to create an [IAuthenticatedEncryptor](xref:security/data-protection/extensibility/core-crypto#data-protection-extensibility-core-crypto-iauthenticatedencryptor) instance.</span></span> <span data-ttu-id="76976-127">Sua API é o seguinte.</span><span class="sxs-lookup"><span data-stu-id="76976-127">Its API is as follows.</span></span>

* <span data-ttu-id="76976-128">CreateEncryptorInstance(): IAuthenticatedEncryptor</span><span class="sxs-lookup"><span data-stu-id="76976-128">CreateEncryptorInstance() : IAuthenticatedEncryptor</span></span>

* <span data-ttu-id="76976-129">ExportToXml(): XmlSerializedDescriptorInfo</span><span class="sxs-lookup"><span data-stu-id="76976-129">ExportToXml() : XmlSerializedDescriptorInfo</span></span>

<span data-ttu-id="76976-130">Como IAuthenticatedEncryptor, supõe-se que uma instância de IAuthenticatedEncryptorDescriptor para encapsular uma chave específica.</span><span class="sxs-lookup"><span data-stu-id="76976-130">Like IAuthenticatedEncryptor, an instance of IAuthenticatedEncryptorDescriptor is assumed to wrap one specific key.</span></span> <span data-ttu-id="76976-131">Isso significa que para qualquer instância específica de IAuthenticatedEncryptorDescriptor qualquer intermediária autenticada criada por seu método CreateEncryptorInstance deve ser considerada equivalentes, como o exemplo de código abaixo.</span><span class="sxs-lookup"><span data-stu-id="76976-131">This means that for any given IAuthenticatedEncryptorDescriptor instance, any authenticated encryptors created by its CreateEncryptorInstance method should be considered equivalent, as in the below code sample.</span></span>

```csharp
// we have an IAuthenticatedEncryptorDescriptor instance
IAuthenticatedEncryptorDescriptor descriptor = ...;

// get an encryptor instance and perform an authenticated encryption operation
ArraySegment<byte> plaintext = new ArraySegment<byte>(Encoding.UTF8.GetBytes("plaintext"));
ArraySegment<byte> aad = new ArraySegment<byte>(Encoding.UTF8.GetBytes("AAD"));
var encryptor1 = descriptor.CreateEncryptorInstance();
byte[] ciphertext = encryptor1.Encrypt(plaintext, aad);

// get another encryptor instance and perform an authenticated decryption operation
var encryptor2 = descriptor.CreateEncryptorInstance();
byte[] roundTripped = encryptor2.Decrypt(new ArraySegment<byte>(ciphertext), aad);


// the 'roundTripped' and 'plaintext' buffers should be equivalent
```

---

<a name=data-protection-extensibility-core-crypto-iauthenticatedencryptordescriptor></a>

## <a name="iauthenticatedencryptordescriptor-aspnet-core-2x-only"></a><span data-ttu-id="76976-132">IAuthenticatedEncryptorDescriptor (ASP.NET Core 2. x somente)</span><span class="sxs-lookup"><span data-stu-id="76976-132">IAuthenticatedEncryptorDescriptor (ASP.NET Core 2.x only)</span></span>

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="76976-133">ASP.NET Core 2. x</span><span class="sxs-lookup"><span data-stu-id="76976-133">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)

<span data-ttu-id="76976-134">O **IAuthenticatedEncryptorDescriptor** interface representa um tipo que sabe como exportar em si para XML.</span><span class="sxs-lookup"><span data-stu-id="76976-134">The **IAuthenticatedEncryptorDescriptor** interface represents a type that knows how to export itself to XML.</span></span> <span data-ttu-id="76976-135">Sua API é o seguinte.</span><span class="sxs-lookup"><span data-stu-id="76976-135">Its API is as follows.</span></span>

* <span data-ttu-id="76976-136">ExportToXml(): XmlSerializedDescriptorInfo</span><span class="sxs-lookup"><span data-stu-id="76976-136">ExportToXml() : XmlSerializedDescriptorInfo</span></span>

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="76976-137">ASP.NET Core 1. x</span><span class="sxs-lookup"><span data-stu-id="76976-137">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)

---

## <a name="xml-serialization"></a><span data-ttu-id="76976-138">Serialização XML</span><span class="sxs-lookup"><span data-stu-id="76976-138">XML Serialization</span></span>

<span data-ttu-id="76976-139">A principal diferença entre IAuthenticatedEncryptor e IAuthenticatedEncryptorDescriptor é que o descritor sabe como criar o Criptografador e fornecê-lo com os argumentos válidos.</span><span class="sxs-lookup"><span data-stu-id="76976-139">The primary difference between IAuthenticatedEncryptor and IAuthenticatedEncryptorDescriptor is that the descriptor knows how to create the encryptor and supply it with valid arguments.</span></span> <span data-ttu-id="76976-140">Considere um IAuthenticatedEncryptor cuja implementação depende do SymmetricAlgorithm e KeyedHashAlgorithm.</span><span class="sxs-lookup"><span data-stu-id="76976-140">Consider an IAuthenticatedEncryptor whose implementation relies on SymmetricAlgorithm and KeyedHashAlgorithm.</span></span> <span data-ttu-id="76976-141">Trabalho do Criptografador é consumir esses tipos, mas não necessariamente souber onde esses tipos de origem, para que ele realmente não é possível gravar uma descrição adequada do como recriar a mesmo se o aplicativo for reiniciado.</span><span class="sxs-lookup"><span data-stu-id="76976-141">The encryptor's job is to consume these types, but it doesn't necessarily know where these types came from, so it can't really write out a proper description of how to recreate itself if the application restarts.</span></span> <span data-ttu-id="76976-142">O descritor de atua como um nível superior sobre isso.</span><span class="sxs-lookup"><span data-stu-id="76976-142">The descriptor acts as a higher level on top of this.</span></span> <span data-ttu-id="76976-143">Desde que o descritor sabe como criar a instância do Criptografador (por exemplo, sabe como criar os algoritmos necessários), ele pode serializar esse conhecimento em formato XML para que a instância do Criptografador pode ser recriada após a redefinição de um aplicativo.</span><span class="sxs-lookup"><span data-stu-id="76976-143">Since the descriptor knows how to create the encryptor instance (e.g., it knows how to create the required algorithms), it can serialize that knowledge in XML form so that the encryptor instance can be recreated after an application reset.</span></span>

<a name=data-protection-extensibility-core-crypto-exporttoxml></a>

<span data-ttu-id="76976-144">O descritor de pode ser serializado por meio de sua rotina de ExportToXml.</span><span class="sxs-lookup"><span data-stu-id="76976-144">The descriptor can be serialized via its ExportToXml routine.</span></span> <span data-ttu-id="76976-145">Esta rotina retorna um XmlSerializedDescriptorInfo que contém duas propriedades: a representação de XElement do descritor e o tipo que representa um [IAuthenticatedEncryptorDescriptorDeserializer](xref:security/data-protection/extensibility/core-crypto#data-protection-extensibility-core-crypto-iauthenticatedencryptordescriptordeserializer) que pode ser usado para lembrar Esse descritor fornecido o XElement correspondente.</span><span class="sxs-lookup"><span data-stu-id="76976-145">This routine returns an XmlSerializedDescriptorInfo which contains two properties: the XElement representation of the descriptor and the Type which represents an [IAuthenticatedEncryptorDescriptorDeserializer](xref:security/data-protection/extensibility/core-crypto#data-protection-extensibility-core-crypto-iauthenticatedencryptordescriptordeserializer) which can be used to resurrect this descriptor given the corresponding XElement.</span></span>

<span data-ttu-id="76976-146">O descritor de serializado pode conter informações confidenciais, como o material de chave de criptografia.</span><span class="sxs-lookup"><span data-stu-id="76976-146">The serialized descriptor may contain sensitive information such as cryptographic key material.</span></span> <span data-ttu-id="76976-147">O sistema de proteção de dados tem suporte interno para criptografar informações antes de ele tem persistidos para armazenamento.</span><span class="sxs-lookup"><span data-stu-id="76976-147">The data protection system has built-in support for encrypting information before it's persisted to storage.</span></span> <span data-ttu-id="76976-148">Para tirar proveito disso, o descritor deve marcar o elemento que contém informações confidenciais com o nome do atributo "requiresEncryption" (xmlns "http://schemas.asp.net/2015/03/dataProtection"), valor "true".</span><span class="sxs-lookup"><span data-stu-id="76976-148">To take advantage of this, the descriptor should mark the element which contains sensitive information with the attribute name "requiresEncryption" (xmlns "http://schemas.asp.net/2015/03/dataProtection"), value "true".</span></span>

>[!TIP]
> <span data-ttu-id="76976-149">Há um auxiliar de API para a configuração deste atributo.</span><span class="sxs-lookup"><span data-stu-id="76976-149">There's a helper API for setting this attribute.</span></span> <span data-ttu-id="76976-150">Chame o método de extensão que XElement.markasrequiresencryption() localizado no namespace Microsoft.AspNetCore.DataProtection.AuthenticatedEncryption.ConfigurationModel.</span><span class="sxs-lookup"><span data-stu-id="76976-150">Call the extension method XElement.MarkAsRequiresEncryption() located in namespace Microsoft.AspNetCore.DataProtection.AuthenticatedEncryption.ConfigurationModel.</span></span>

<span data-ttu-id="76976-151">Também pode haver casos onde o descritor serializado não contêm informações confidenciais.</span><span class="sxs-lookup"><span data-stu-id="76976-151">There can also be cases where the serialized descriptor doesn't contain sensitive information.</span></span> <span data-ttu-id="76976-152">Considere novamente o caso de uma chave de criptografia armazenada em um HSM.</span><span class="sxs-lookup"><span data-stu-id="76976-152">Consider again the case of a cryptographic key stored in an HSM.</span></span> <span data-ttu-id="76976-153">Não é possível gravar o descritor de material de chave ao serializar em si, pois o HSM não irá expor o material em forma de texto sem formatação.</span><span class="sxs-lookup"><span data-stu-id="76976-153">The descriptor cannot write out the key material when serializing itself since the HSM will not expose the material in plaintext form.</span></span> <span data-ttu-id="76976-154">Em vez disso, o descritor pode gravar a versão de chave de linha de chave (se o HSM permite exportar dessa forma) ou o identificador exclusivo do HSM para a chave.</span><span class="sxs-lookup"><span data-stu-id="76976-154">Instead, the descriptor might write out the key-wrapped version of the key (if the HSM allows export in this fashion) or the HSM's own unique identifier for the key.</span></span>

<a name=data-protection-extensibility-core-crypto-iauthenticatedencryptordescriptordeserializer></a>

## <a name="iauthenticatedencryptordescriptordeserializer"></a><span data-ttu-id="76976-155">IAuthenticatedEncryptorDescriptorDeserializer</span><span class="sxs-lookup"><span data-stu-id="76976-155">IAuthenticatedEncryptorDescriptorDeserializer</span></span>

<span data-ttu-id="76976-156">O **IAuthenticatedEncryptorDescriptorDeserializer** interface representa um tipo que sabe como desserializar uma instância IAuthenticatedEncryptorDescriptor de um XElement.</span><span class="sxs-lookup"><span data-stu-id="76976-156">The **IAuthenticatedEncryptorDescriptorDeserializer** interface represents a type that knows how to deserialize an IAuthenticatedEncryptorDescriptor instance from an XElement.</span></span> <span data-ttu-id="76976-157">Ela expõe um único método:</span><span class="sxs-lookup"><span data-stu-id="76976-157">It exposes a single method:</span></span>

* <span data-ttu-id="76976-158">ImportFromXml (elemento de XElement): IAuthenticatedEncryptorDescriptor</span><span class="sxs-lookup"><span data-stu-id="76976-158">ImportFromXml(XElement element) : IAuthenticatedEncryptorDescriptor</span></span>

<span data-ttu-id="76976-159">O método ImportFromXml usa o XElement foi retornado por [IAuthenticatedEncryptorDescriptor.ExportToXml](xref:security/data-protection/extensibility/core-crypto#data-protection-extensibility-core-crypto-exporttoxml) e cria um equivalente a IAuthenticatedEncryptorDescriptor original.</span><span class="sxs-lookup"><span data-stu-id="76976-159">The ImportFromXml method takes the XElement that was returned by [IAuthenticatedEncryptorDescriptor.ExportToXml](xref:security/data-protection/extensibility/core-crypto#data-protection-extensibility-core-crypto-exporttoxml) and creates an equivalent of the original IAuthenticatedEncryptorDescriptor.</span></span>

<span data-ttu-id="76976-160">Tipos que implementam IAuthenticatedEncryptorDescriptorDeserializer devem ter um dos dois construtores públicos a seguir:</span><span class="sxs-lookup"><span data-stu-id="76976-160">Types which implement IAuthenticatedEncryptorDescriptorDeserializer should have one of the following two public constructors:</span></span>

* <span data-ttu-id="76976-161">.ctor(IServiceProvider)</span><span class="sxs-lookup"><span data-stu-id="76976-161">.ctor(IServiceProvider)</span></span>

* <span data-ttu-id="76976-162">.ctor()</span><span class="sxs-lookup"><span data-stu-id="76976-162">.ctor()</span></span>

> [!NOTE]
> <span data-ttu-id="76976-163">O IServiceProvider transmitido ao construtor pode ser nulo.</span><span class="sxs-lookup"><span data-stu-id="76976-163">The IServiceProvider passed to the constructor may be null.</span></span>

## <a name="the-top-level-factory"></a><span data-ttu-id="76976-164">A fábrica de nível superior</span><span class="sxs-lookup"><span data-stu-id="76976-164">The top-level factory</span></span>

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="76976-165">ASP.NET Core 2. x</span><span class="sxs-lookup"><span data-stu-id="76976-165">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)

<span data-ttu-id="76976-166">O **AlgorithmConfiguration** classe representa um tipo que sabe como criar [IAuthenticatedEncryptorDescriptor](xref:security/data-protection/extensibility/core-crypto#data-protection-extensibility-core-crypto-iauthenticatedencryptordescriptor) instâncias.</span><span class="sxs-lookup"><span data-stu-id="76976-166">The **AlgorithmConfiguration** class represents a type which knows how to create [IAuthenticatedEncryptorDescriptor](xref:security/data-protection/extensibility/core-crypto#data-protection-extensibility-core-crypto-iauthenticatedencryptordescriptor) instances.</span></span> <span data-ttu-id="76976-167">Ela expõe uma única API.</span><span class="sxs-lookup"><span data-stu-id="76976-167">It exposes a single API.</span></span>

* <span data-ttu-id="76976-168">CreateNewDescriptor(): IAuthenticatedEncryptorDescriptor</span><span class="sxs-lookup"><span data-stu-id="76976-168">CreateNewDescriptor() : IAuthenticatedEncryptorDescriptor</span></span>

<span data-ttu-id="76976-169">Pense AlgorithmConfiguration como a fábrica de nível superior.</span><span class="sxs-lookup"><span data-stu-id="76976-169">Think of AlgorithmConfiguration as the top-level factory.</span></span> <span data-ttu-id="76976-170">A configuração funciona como um modelo.</span><span class="sxs-lookup"><span data-stu-id="76976-170">The configuration serves as a template.</span></span> <span data-ttu-id="76976-171">Ela inclui informações de algoritmos (por exemplo, essa configuração produz descritores com uma chave mestra de AES-128-GCM), mas ainda não está associado com uma chave específica.</span><span class="sxs-lookup"><span data-stu-id="76976-171">It wraps algorithmic information (e.g., this configuration produces descriptors with an AES-128-GCM master key), but it is not yet associated with a specific key.</span></span>

<span data-ttu-id="76976-172">Quando CreateNewDescriptor é chamado, novo material de chave é criada somente para essa chamada e um novo IAuthenticatedEncryptorDescriptor é produzido que encapsula o material de chave e as informações de algoritmos necessária para consumir o material.</span><span class="sxs-lookup"><span data-stu-id="76976-172">When CreateNewDescriptor is called, fresh key material is created solely for this call, and a new IAuthenticatedEncryptorDescriptor is produced which wraps this key material and the algorithmic information required to consume the material.</span></span> <span data-ttu-id="76976-173">O material da chave pode ser criado no software (e mantido na memória), ele pode ser criado e mantido em um HSM e assim por diante.</span><span class="sxs-lookup"><span data-stu-id="76976-173">The key material could be created in software (and held in memory), it could be created and held within an HSM, and so on.</span></span> <span data-ttu-id="76976-174">O ponto fundamental é que as duas chamadas para CreateNewDescriptor nunca devem criar instâncias de IAuthenticatedEncryptorDescriptor equivalentes.</span><span class="sxs-lookup"><span data-stu-id="76976-174">The crucial point is that any two calls to CreateNewDescriptor should never create equivalent IAuthenticatedEncryptorDescriptor instances.</span></span>

<span data-ttu-id="76976-175">O tipo de AlgorithmConfiguration serve como ponto de entrada para rotinas de criação da chave como [chave automática](../implementation/key-management.md#data-protection-implementation-key-management).</span><span class="sxs-lookup"><span data-stu-id="76976-175">The AlgorithmConfiguration type serves as the entry point for key creation routines such as [automatic key rolling](../implementation/key-management.md#data-protection-implementation-key-management).</span></span> <span data-ttu-id="76976-176">Para alterar a implementação para todas as chaves futuras, defina a propriedade de AuthenticatedEncryptorConfiguration em KeyManagementOptions.</span><span class="sxs-lookup"><span data-stu-id="76976-176">To change the implementation for all future keys, set the AuthenticatedEncryptorConfiguration property in KeyManagementOptions.</span></span>

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="76976-177">ASP.NET Core 1. x</span><span class="sxs-lookup"><span data-stu-id="76976-177">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)

<span data-ttu-id="76976-178">O **IAuthenticatedEncryptorConfiguration** interface representa um tipo que sabe como criar [IAuthenticatedEncryptorDescriptor](xref:security/data-protection/extensibility/core-crypto#data-protection-extensibility-core-crypto-iauthenticatedencryptordescriptor) instâncias.</span><span class="sxs-lookup"><span data-stu-id="76976-178">The **IAuthenticatedEncryptorConfiguration** interface represents a type which knows how to create [IAuthenticatedEncryptorDescriptor](xref:security/data-protection/extensibility/core-crypto#data-protection-extensibility-core-crypto-iauthenticatedencryptordescriptor) instances.</span></span> <span data-ttu-id="76976-179">Ela expõe uma única API.</span><span class="sxs-lookup"><span data-stu-id="76976-179">It exposes a single API.</span></span>

* <span data-ttu-id="76976-180">CreateNewDescriptor(): IAuthenticatedEncryptorDescriptor</span><span class="sxs-lookup"><span data-stu-id="76976-180">CreateNewDescriptor() : IAuthenticatedEncryptorDescriptor</span></span>

<span data-ttu-id="76976-181">Pense IAuthenticatedEncryptorConfiguration como a fábrica de nível superior.</span><span class="sxs-lookup"><span data-stu-id="76976-181">Think of IAuthenticatedEncryptorConfiguration as the top-level factory.</span></span> <span data-ttu-id="76976-182">A configuração funciona como um modelo.</span><span class="sxs-lookup"><span data-stu-id="76976-182">The configuration serves as a template.</span></span> <span data-ttu-id="76976-183">Ela inclui informações de algoritmos (por exemplo, essa configuração produz descritores com uma chave mestra de AES-128-GCM), mas ainda não está associado com uma chave específica.</span><span class="sxs-lookup"><span data-stu-id="76976-183">It wraps algorithmic information (e.g., this configuration produces descriptors with an AES-128-GCM master key), but it is not yet associated with a specific key.</span></span>

<span data-ttu-id="76976-184">Quando CreateNewDescriptor é chamado, novo material de chave é criada somente para essa chamada e um novo IAuthenticatedEncryptorDescriptor é produzido que encapsula o material de chave e as informações de algoritmos necessária para consumir o material.</span><span class="sxs-lookup"><span data-stu-id="76976-184">When CreateNewDescriptor is called, fresh key material is created solely for this call, and a new IAuthenticatedEncryptorDescriptor is produced which wraps this key material and the algorithmic information required to consume the material.</span></span> <span data-ttu-id="76976-185">O material da chave pode ser criado no software (e mantido na memória), ele pode ser criado e mantido em um HSM e assim por diante.</span><span class="sxs-lookup"><span data-stu-id="76976-185">The key material could be created in software (and held in memory), it could be created and held within an HSM, and so on.</span></span> <span data-ttu-id="76976-186">O ponto fundamental é que as duas chamadas para CreateNewDescriptor nunca devem criar instâncias de IAuthenticatedEncryptorDescriptor equivalentes.</span><span class="sxs-lookup"><span data-stu-id="76976-186">The crucial point is that any two calls to CreateNewDescriptor should never create equivalent IAuthenticatedEncryptorDescriptor instances.</span></span>

<span data-ttu-id="76976-187">O tipo de IAuthenticatedEncryptorConfiguration serve como ponto de entrada para rotinas de criação da chave como [chave automática](../implementation/key-management.md#data-protection-implementation-key-management).</span><span class="sxs-lookup"><span data-stu-id="76976-187">The IAuthenticatedEncryptorConfiguration type serves as the entry point for key creation routines such as [automatic key rolling](../implementation/key-management.md#data-protection-implementation-key-management).</span></span> <span data-ttu-id="76976-188">Para alterar a implementação para todas as chaves futuras, registre um singleton IAuthenticatedEncryptorConfiguration no contêiner de serviço.</span><span class="sxs-lookup"><span data-stu-id="76976-188">To change the implementation for all future keys, register a singleton IAuthenticatedEncryptorConfiguration in the service container.</span></span>

---
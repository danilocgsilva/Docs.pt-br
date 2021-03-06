---
title: "Configuração no ASP.NET Core"
author: rick-anderson
description: "Use a API de configuração para configurar um aplicativo do ASP.NET Core por vários métodos."
manager: wpickett
ms.author: riande
ms.custom: mvc
ms.date: 11/01/2017
ms.topic: article
ms.technology: aspnet
ms.prod: asp.net-core
uid: fundamentals/configuration/index
ms.openlocfilehash: 6281d6ba254670b111964715410fc0694ae4d149
ms.sourcegitcommit: 216dfac27542f10a79274a9ce60dc449e888ed20
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/29/2017
---
# <a name="configure-an-aspnet-core-app"></a>Configurar um aplicativo do ASP.NET Core

Por [Rick Anderson](https://twitter.com/RickAndMSFT), [Mark Michaelis](http://intellitect.com/author/mark-michaelis/), [Steve Smith](https://ardalis.com/), [Daniel Roth](https://github.com/danroth27) e [Luke Latham](https://github.com/guardrex)

A API de configuração fornece uma maneira de configurar um aplicativo Web do ASP.NET Core com base em uma lista de pares nome-valor. A configuração é lida em tempo de execução por meio de várias fontes. Você pode agrupar esses pares de nome-valor em uma hierarquia de vários níveis. 

Há provedores de configuração para:

* Formatos de arquivo (INI, JSON e XML)
* Argumentos de linha de comando
* Variáveis de ambiente
* Objetos do .NET na memória
* Um repositório de usuário criptografado
* [Azure Key Vault](xref:security/key-vault-configuration)
* Provedores personalizados (instalados ou criados)

Cada valor de configuração é mapeado para uma chave de cadeia de caracteres. Há suporte interno de associação para desserializar configurações em um objeto [POCO](https://wikipedia.org/wiki/Plain_Old_CLR_Object) personalizado (uma classe simples de .NET com propriedades).

O padrão de opções usa classes de opções para representar grupos de configurações relacionadas. Para obter mais informações sobre como usar o padrão de opções, consulte o tópico [Opções](xref:fundamentals/configuration/options).

[Exibir ou baixar código de exemplo](https://github.com/aspnet/docs/tree/master/aspnetcore/fundamentals/configuration/index/sample) ([como baixar](xref:tutorials/index#how-to-download-a-sample))

## <a name="json-configuration"></a>Configuração de JSON

O aplicativo de console a seguir usa o provedor de configuração JSON:

[!code-csharp[Main](index/sample/ConfigJson/Program.cs)]

O aplicativo lê e exibe as seguintes definições de configuração:

[!code-json[Main](index/sample/ConfigJson/appsettings.json)]

A configuração consiste em uma lista hierárquica de pares nome-valor na qual os nós são separados por dois pontos. Para recuperar um valor, acesse o indexador `Configuration` com a chave do item correspondente:

```csharp
Console.WriteLine($"option1 = {Configuration["subsection:suboption1"]}");
```

Para trabalhar com matrizes em fontes de configuração formatadas em JSON, use um índice de matriz como parte da cadeia de caracteres separada por dois pontos. O exemplo a seguir obtém o nome do primeiro item na matriz `wizards` anterior:

```csharp
Console.Write($"{Configuration["wizards:0:Name"]}, ");
```

Os pares nome-valor gravados nos provedores `Configuration` internos **não** são mantidos. No entanto, você pode criar um provedor personalizado que salva valores. Consulte [provedor de configuração personalizado](xref:fundamentals/configuration/index#custom-config-providers).

O exemplo anterior usa o indexador de configuração para ler valores. Para acessar a configuração fora de `Startup`, use o *padrão de opções*. Para obter mais informações, consulte o tópico [Opções](xref:fundamentals/configuration/options).

É comum ter configurações diferentes para ambientes diferentes, por exemplo, desenvolvimento, teste e produção. O método de extensão `CreateDefaultBuilder` em um aplicativo do ASP.NET Core 2.x (ou usando `AddJsonFile` e `AddEnvironmentVariables` diretamente em um aplicativo do ASP.NET Core 1.x) adiciona provedores de configuração para ler arquivos JSON e fontes de configuração do sistema:

* *appsettings.json*
* *appsettings.\<EnvironmentName>.json*
* Variáveis de ambiente

Consulte [AddJsonFile](/dotnet/api/microsoft.extensions.configuration.jsonconfigurationextensions) para uma explicação sobre os parâmetros. `reloadOnChange` só é compatível no ASP.NET Core 1.1 ou posterior. 

As fontes de configuração são lidas na ordem em que são especificadas. No código acima, as variáveis de ambiente são lidas por último. Os valores de configuração definidos por meio do ambiente substituem aqueles definidos nos dois provedores anteriores.

O ambiente normalmente é definido como `Development`, `Staging` ou `Production`. Consulte [Trabalhando com vários ambientes](xref:fundamentals/environments) para obter mais informações.

Considerações de configuração:

* `IOptionsSnapshot` pode recarregar dados de configuração quando é alterado. Consulte [IOptionsSnapshot](xref:fundamentals/configuration/options#reload-configuration-data-with-ioptionssnapshot) para obter mais informações.
* As chaves de configuração não diferenciam maiúsculas de minúsculas.
* Especifique as variáveis de ambiente por último para que o ambiente local possa substituir as configurações nos arquivos de configuração implantados.
* **Nunca** armazene senhas ou outros dados confidenciais no código do provedor de configuração ou nos arquivos de configuração de texto sem formatação. Não use segredos de produção em seus ambientes de teste ou de desenvolvimento. Em vez disso, especifique segredos fora do projeto para que eles não sejam acidentalmente comprometidos com seu repositório. Saiba mais sobre [trabalhando com vários ambientes](xref:fundamentals/environments) e gerenciamento de [armazenamento seguro de segredos de aplicativo durante o desenvolvimento](xref:security/app-secrets).
* Se dois pontos (`:`) não puder ser usado em variáveis de ambiente em seu sistema, substitua os dois pontos (`:`) por um sublinhado duplo (`__`).

## <a name="in-memory-provider-and-binding-to-a-poco-class"></a>Provedor na memória e associação a uma classe POCO

O exemplo a seguir mostra como usar o provedor na memória e associar a uma classe:

[!code-csharp[Main](index/sample/InMemory/Program.cs)]

Os valores de configuração são retornados como cadeias de caracteres, mas a associação permite a construção de objetos. A associação permite que você recupere objetos POCO ou até mesmo gráficos de objetos inteiros.

### <a name="getvalue"></a>GetValue

O exemplo a seguir demonstra o método de extensão [GetValue&lt;T&gt;](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.configuration.configurationbinder#Microsoft_Extensions_Configuration_ConfigurationBinder_GetValue_Microsoft_Extensions_Configuration_IConfiguration_System_Type_System_String_System_Object_):

[!code-csharp[Main](index/sample/InMemoryGetValue/Program.cs?highlight=27-29)]

O método `GetValue<T>` do ConfigurationBinder permite que você especifique um valor padrão (80 no exemplo). `GetValue<T>` é para cenários simples e não se associa a seções inteiras. `GetValue<T>` obtém valores escalares de `GetSection(key).Value` convertidos em um tipo específico.

## <a name="bind-to-an-object-graph"></a>Associar a um gráfico de objeto

Você pode associar recursivamente a cada objeto em uma classe. Considere a classe `AppSettings` a seguir:

[!code-csharp[Main](index/sample/ObjectGraph/AppSettings.cs)]

O exemplo a seguir associa à classe `AppSettings`:

[!code-csharp[Main](index/sample/ObjectGraph/Program.cs?highlight=15-16)]

O **ASP.NET Core 1.1** e superior pode usar `Get<T>`, que trabalha com seções inteiras. O `Get<T>` pode ser mais conveniente do que usar `Bind`. O código a seguir mostra como usar `Get<T>` com o exemplo acima:

```csharp
var appConfig = config.GetSection("App").Get<AppSettings>();
```

Usando o seguinte arquivo *appsettings.json*:

[!code-json[Main](index/sample/ObjectGraph/appsettings.json)]

O programa exibe `Height 11`.

O código a seguir pode ser usado para o teste de unidade da configuração:

```csharp
[Fact]
public void CanBindObjectTree()
{
    var dict = new Dictionary<string, string>
        {
            {"App:Profile:Machine", "Rick"},
            {"App:Connection:Value", "connectionstring"},
            {"App:Window:Height", "11"},
            {"App:Window:Width", "11"}
        };
    var builder = new ConfigurationBuilder();
    builder.AddInMemoryCollection(dict);
    var config = builder.Build();

    var settings = new AppSettings();
    config.GetSection("App").Bind(settings);

    Assert.Equal("Rick", settings.Profile.Machine);
    Assert.Equal(11, settings.Window.Height);
    Assert.Equal(11, settings.Window.Width);
    Assert.Equal("connectionstring", settings.Connection.Value);
}
```

<a name="custom-config-providers"></a>

## <a name="create-an-entity-framework-custom-provider"></a>Criar um provedor personalizado do Entity Framework

Nesta seção, é criado um provedor de configuração básico, que lê os pares nome-valor de um banco de dados, usando o EF. 

Defina uma entidade `ConfigurationValue` para armazenar valores de configuração no banco de dados:

[!code-csharp[Main](index/sample/CustomConfigurationProvider/ConfigurationValue.cs)]

Adicione um `ConfigurationContext` para armazenar e acessar os valores configurados:

[!code-csharp[Main](index/sample/CustomConfigurationProvider/ConfigurationContext.cs?name=snippet1)]

Crie uma classe que implementa [IConfigurationSource](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.configuration.iconfigurationsource):

[!code-csharp[Main](index/sample/CustomConfigurationProvider/EntityFrameworkConfigurationSource.cs?highlight=7)]

Crie o provedor de configuração personalizado através da herança do [ConfigurationProvider](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.configuration.configurationprovider).  O provedor de configuração inicializa o banco de dados quando ele está vazio:

[!code-csharp[Main](index/sample/CustomConfigurationProvider/EntityFrameworkConfigurationProvider.cs?highlight=9,18-31,38-39)]

Os valores realçados do banco de dados ("value_from_ef_1" e "value_from_ef_2") são exibidos quando o exemplo é executado.

Você pode adicionar um método de extensão `EFConfigSource` para adicionar a fonte de configuração:

[!code-csharp[Main](index/sample/CustomConfigurationProvider/EntityFrameworkExtensions.cs?highlight=12)]

O código a seguir mostra como usar o `EFConfigProvider` personalizado:

[!code-csharp[Main](index/sample/CustomConfigurationProvider/Program.cs?highlight=21-26)]

Observe que o exemplo adiciona o `EFConfigProvider` personalizado depois do provedor JSON, de maneira que todas as configurações do banco de dados substituam as configurações do arquivo *appsettings.json*.

Usando o seguinte arquivo *appsettings.json*:

[!code-json[Main](index/sample/CustomConfigurationProvider/appsettings.json)]

O seguinte é exibido:

```console
key1=value_from_ef_1
key2=value_from_ef_2
key3=value_from_json_3
```

## <a name="commandline-configuration-provider"></a>Provedor de configuração CommandLine

O [Provedor de configuração CommandLine](/aspnet/core/api/microsoft.extensions.configuration.commandline.commandlineconfigurationprovider) recebe pares chave-valor de argumento de linha de comando para a configuração em tempo de execução.

[Exibir ou baixar o exemplo de configuração do CommandLine](https://github.com/aspnet/docs/tree/master/aspnetcore/fundamentals/index/sample/CommandLine)

### <a name="setup-and-use-the-commandline-configuration-provider"></a>Configurar e usar o provedor de configuração CommandLine

# <a name="basic-configurationtabbasicconfiguration"></a>[Configuração básica](#tab/basicconfiguration)

Para ativar a configuração de linha de comando, chame o método de extensão `AddCommandLine` em uma instância do [ConfigurationBuilder](/api/microsoft.extensions.configuration.configurationbuilder):

[!code-csharp[Main](index/sample_snapshot//CommandLine/Program.cs?highlight=18,21)]

Ao executar o código, a seguinte saída será exibida:

```console
MachineName: MairaPC
Left: 1980
```

Passar pares chave-valor de argumento na linha de comando altera os valores de `Profile:MachineName` e `App:MainWindow:Left`:

```console
dotnet run Profile:MachineName=BartPC App:MainWindow:Left=1979
```

A janela do console exibe:

```console
MachineName: BartPC
Left: 1979
```

Para substituir a configuração fornecida por outros provedores de configuração pela configuração de linha de comando, chame `AddCommandLine` por último em `ConfigurationBuilder`:

[!code-csharp[Main](index/sample_snapshot//CommandLine/Program2.cs?range=11-16&highlight=1,5)]

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)

Os aplicativos típicos de ASP.NET Core 2.x usam o método de conveniência estático `CreateDefaultBuilder` para criar o host:

[!code-csharp[Main](index/sample_snapshot//Program.cs?highlight=12)]

O `CreateDefaultBuilder` carrega a configuração opcional de *appsettings.json*, de *appsettings.{Environment}.json*, dos [segredos do usuário](xref:security/app-secrets) (no ambiente `Development`), das variáveis de ambiente e dos argumentos de linha de comando. O provedor CommandLine é chamado por último. Chamar o provedor por último permite que os argumentos de linha de comando passados em tempo de execução substituam a configuração definida por outros provedores de configuração chamados anteriormente.

Observe que para arquivos *appsettings* esse `reloadOnChange` está habilitado. Os argumentos de linha de comando são substituídos se um valor de configuração correspondente em um arquivo *appsettings* é alterado depois que o aplicativo é iniciado.

> [!NOTE]
> Como uma alternativa ao uso do método `CreateDefaultBuilder`, a criação de um host usando [WebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder) e a criação manual da configuração com [ConfigurationBuilder](/api/microsoft.extensions.configuration.configurationbuilder) são compatíveis no ASP.NET Core 2.x. Consulte a guia do ASP.NET Core 1.x para obter mais informações.

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)

Crie um [ConfigurationBuilder](/api/microsoft.extensions.configuration.configurationbuilder) e chame o método `AddCommandLine` para usar o provedor de configuração CommandLine. Chamar o provedor por último permite que os argumentos de linha de comando passados em tempo de execução substituam a configuração definida por outros provedores de configuração chamados anteriormente. Aplique a configuração ao [WebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder) com o método `UseConfiguration`:

[!code-csharp[Main](index/sample_snapshot//CommandLine/Program2.cs?highlight=11,15,19)]

---

### <a name="arguments"></a>Arguments

Os argumentos passados na linha de comando devem estar em conformidade com um dos dois formatos mostrados na tabela a seguir.

| Formato de argumento                                                     | Exemplo        |
| ------------------------------------------------------------------- | :------------: |
| Argumento único: um par chave-valor separado por um sinal de igual (`=`) | `key1=value`   |
| Sequência de dois argumentos: um par chave-valor separado por um espaço    | `/key1 value1` |

**Argumento único**

O valor deve seguir um sinal de igual (`=`). O valor pode ser nulo (por exemplo, `mykey=`).

A chave pode ter um prefixo.

| Prefixo da chave               | Exemplo         |
| ------------------------ | :-------------: |
| Nenhum prefixo                | `key1=value1`   |
| Traço único (`-`)&#8224; | `-key2=value2`  |
| Dois traços (`--`)        | `--key3=value3` |
| Barra (`/`)      | `/key4=value4`  |

&#8224;Uma chave com um prefixo de traço único (`-`) deve ser fornecida em [mapeamentos de comutador](#switch-mappings), como descrito abaixo.

Comando de exemplo:

```console
dotnet run key1=value1 -key2=value2 --key3=value3 /key4=value4
```

Observação: se `-key1` não estiver presente nos [mapeamentos de comutador](#switch-mappings) fornecidos para o provedor de configuração, uma `FormatException` será gerada.

**Sequência de dois argumentos**

O valor não pode ser nulo e deve seguir a chave separada por um espaço.

A chave deve ter um prefixo.

| Prefixo da chave               | Exemplo         |
| ------------------------ | :-------------: |
| Traço único (`-`)&#8224; | `-key1 value1`  |
| Dois traços (`--`)        | `--key2 value2` |
| Barra (`/`)      | `/key3 value3`  |

&#8224;Uma chave com um prefixo de traço único (`-`) deve ser fornecida em [mapeamentos de comutador](#switch-mappings), como descrito abaixo.

Comando de exemplo:

```console
dotnet run -key1 value1 --key2 value2 /key3 value3
```

Observação: se `-key1` não estiver presente nos [mapeamentos de comutador](#switch-mappings) fornecidos para o provedor de configuração, uma `FormatException` será gerada.

### <a name="duplicate-keys"></a>Chaves duplicadas

Se chaves duplicadas forem fornecidas, o último par chave-valor será usado.

### <a name="switch-mappings"></a>Mapeamentos de comutador

Ao criar manualmente a configuração com `ConfigurationBuilder`, você pode fornecer opcionalmente um dicionário de mapeamentos de comutador para o método `AddCommandLine`. Os mapeamentos de comutador permitem que você forneça a lógica de substituição do nome da chave.

Ao ser usado, o dicionário de mapeamentos de comutador é verificado para oferecer uma chave que corresponda à chave fornecida por um argumento de linha de comando. Se a chave de linha de comando for encontrada no dicionário, o valor do dicionário (a substituição da chave) será passado de volta para definir a configuração. Um mapeamento de comutador é necessário para qualquer chave de linha de comando prefixada com um traço único (`-`).

Regras de chave do dicionário de mapeamentos de comutador:

* Os comutadores devem começar com um traço (`-`) ou traço duplo (`--`).
* O dicionário de mapeamentos de comutador chave não deve conter chaves duplicadas.

No exemplo a seguir, o método `GetSwitchMappings` permite que seus argumentos de linha de comando usem um prefixo de chave de traço único (`-`) e evitem prefixos de subchaves à esquerda.

[!code-csharp[Main](index/sample/CommandLine/Program.cs?highlight=10-19,32)]

Sem fornecer argumentos de linha de comando, o dicionário fornecido para `AddInMemoryCollection` definirá os valores de configuração. Execute o aplicativo com o seguinte comando:

```console
dotnet run
```

A janela do console exibe:

```console
MachineName: RickPC
Left: 1980
```

Use o seguinte para passar parâmetros de configuração:

```console
dotnet run /Profile:MachineName=DahliaPC /App:MainWindow:Left=1984
```

A janela do console exibe:

```console
MachineName: DahliaPC
Left: 1984
```

Depois que o dicionário de mapeamentos de comutador for criado, ele conterá os dados mostrados na tabela a seguir.

| Chave            | Valor                 |
| -------------- | --------------------- |
| `-MachineName` | `Profile:MachineName` |
| `-Left`        | `App:MainWindow:Left` |

Para demonstrar a troca de chaves usando o dicionário, execute o seguinte comando:

```console
dotnet run -MachineName=ChadPC -Left=1988
```

As chaves da linha de comando são trocadas. A janela de console exibe os valores de configuração de `Profile:MachineName` e `App:MainWindow:Left`:

```console
MachineName: ChadPC
Left: 1988
```

## <a name="the-webconfig-file"></a>O arquivo Web.config

Um arquivo *Web.config* é necessário quando você hospeda o aplicativo no IIS ou IIS Express. O *Web.config* ativa o AspNetCoreModule no IIS para iniciar seu aplicativo. As configurações no *Web.config* habilitam o AspNetCoreModule no IIS para iniciar seu aplicativo e definir outras configurações e módulos do IIS. Se você estiver usando o Visual Studio e excluir o *Web.config*, o Visual Studio criará um novo.

## <a name="additional-notes"></a>Observações adicionais

* A DI (injeção de dependência) não é configurada até que `ConfigureServices` seja invocado.
* O sistema de configuração não tem reconhecimento de DI.
* O `IConfiguration` tem duas especializações:
  * `IConfigurationRoot` Usada para o nó raiz. Pode disparar um recarregamento.
  * `IConfigurationSection` Representa uma seção de valores de configuração. O métodos `GetSection` e `GetChildren` retornam um `IConfigurationSection`.

## <a name="additional-resources"></a>Recursos adicionais

* [Opções](xref:fundamentals/configuration/options)
* [Trabalhando com vários ambientes](xref:fundamentals/environments)
* [Armazenamento seguro dos segredos do aplicativo durante o desenvolvimento](xref:security/app-secrets)
* [Hospedagem no ASP.NET Core](xref:fundamentals/hosting)
* [Injeção de dependência](xref:fundamentals/dependency-injection)
* [Provedor de configuração do Azure Key Vault](xref:security/key-vault-configuration)

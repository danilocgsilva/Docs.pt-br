---
title: "Introdução a Páginas do Razor no ASP.NET Core"
author: Rick-Anderson
description: "Este documento fornece uma visão geral do uso das páginas Razor no ASP.NET Core para facilitar o desenvolvimento de cenários focados em página."
keywords: "ASP.NET Core, Páginas Razor"
ms.author: riande
manager: wpickett
ms.date: 09/12/2017
ms.topic: get-started-article
ms.technology: aspnet
ms.prod: asp.net-core
uid: mvc/razor-pages/index
ms.openlocfilehash: 36dd2ad01f93ab1093bad84a58504a150c70ea16
ms.sourcegitcommit: 4925a91ef4130ddb333f187ab13defe66f2c6cef
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/03/2017
---
# <a name="introduction-to-razor-pages-in-aspnet-core"></a>Introdução a Páginas do Razor no ASP.NET Core

Por [Rick Anderson](https://twitter.com/RickAndMSFT) e [Ryan Nowak](https://github.com/rynowak)

Páginas do Razor é um novo recurso do ASP.NET Core MVC que torna a codificação de cenários focados em página mais fácil e produtiva.

Se você estiver procurando um tutorial que usa a abordagem Modelo-Exibição-Controlador, consulte a [Introdução ao ASP.NET Core MVC](xref:tutorials/first-mvc-app/start-mvc).

Este documento proporciona uma introdução a páginas do Razor. Este não é um tutorial passo a passo. Se você achar que algumas das seções são difíceis de entender, consulte [Introdução a Páginas do Razor](xref:tutorials/razor-pages/razor-pages-start).

<a name="prerequisites"></a>

## <a name="aspnet-core-20-prerequisites"></a>Pré-requisitos do ASP.NET Core 2.0

Instale o [.NET Core](https://www.microsoft.com/net/core) 2.0.0 ou posterior.

Se você estiver usando o Visual Studio, instale o [Visual Studio](https://www.visualstudio.com/vs/) 2017 versão 15.3 ou posterior com as cargas de trabalho a seguir:

* **ASP.NET e desenvolvimento para a Web**
* **Desenvolvimento entre plataformas do .NET Core**

<a name="rpvs17"></a>

## <a name="creating-a-razor-pages-project"></a>Criando um projeto de Páginas do Razor

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio) 

Consulte a [Introdução a Páginas do Razor](xref:tutorials/razor-pages/razor-pages-start) para obter instruções detalhadas sobre como criar um projeto de Páginas do Razor usando o Visual Studio.

#   <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio para Mac](#tab/visual-studio-mac)

Da linha de comando, execute `dotnet new razor`.

Abra o arquivo *.csproj* gerado do Visual Studio para Mac.

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code) 

Da linha de comando, execute `dotnet new razor`.

#   <a name="net-core-clitabnetcore-cli"></a>[CLI do .NET Core](#tab/netcore-cli) 

Da linha de comando, execute `dotnet new razor`.

---

## <a name="razor-pages"></a>Páginas do Razor

O Páginas do Razor está habilitado em *Startup.cs*:

[!code-cs[main](index/sample/RazorPagesIntro/Startup.cs?name=snippet_Startup)]

Considere uma página básica: <a name="OnGet"></a>

[!code-cshtml[main](index/sample/RazorPagesIntro/Pages/Index.cshtml)]

O código anterior é muito parecido com um arquivo de exibição do Razor. O que o torna diferentes é a diretiva `@page`. `@page` transforma o arquivo em uma ação do MVC – o que significa que ele trata solicitações diretamente, sem passar por um controlador. `@page` deve ser a primeira diretiva do Razor em uma página. `@page` afeta o comportamento de outros constructos do Razor.

Uma página semelhante, usando uma classe `PageModel`, é mostrada nos dois arquivos a seguir. O arquivo *Pages/Index2.cshtml*:

[!code-cshtml[main](index/sample/RazorPagesIntro/Pages/Index2.cshtml)]

O arquivo “code-behind” *Pages/Index2.cshtml.cs*:

[!code-cs[main](index/sample/RazorPagesIntro/Pages/Index2.cshtml.cs)]

Por convenção, o arquivo de classe `PageModel` tem o mesmo nome que o arquivo na Página do Razor com *.cs* acrescentado. Por exemplo, a Página do Razor anterior é *Pages/Index2.cshtml*. O arquivo que contém a classe `PageModel` é chamado *Pages/Index2.cshtml.cs*.

As associações de caminhos de URL para páginas são determinadas pelo local da página no sistema de arquivos. A tabela a seguir mostra um caminho de Página do Razor e a URL correspondente:

| Caminho e nome do arquivo               | URL correspondente |
| ----------------- | ------------ |
| */Pages/Index.cshtml* | `/` ou `/Index` |
| */Pages/Contact.cshtml* | `/Contact` |
| */Pages/Store/Contact.cshtml* | `/Store/Contact` |
| */Pages/Store/Index.cshtml* | `/Store` ou `/Store/Index` |

Notas:

* O tempo de execução procura arquivos de Páginas do Razor na pasta *Pages* por padrão.
* `Index` é a página padrão quando uma URL não inclui uma página.

## <a name="writing-a-basic-form"></a>Escrevendo um formulário básico

Os recursos de Páginas do Razor são projetados para tornar fáceis os padrões comuns usados com navegadores da Web. [Associação de modelos](xref:mvc/models/model-binding), [auxiliares de marcas](xref:mvc/views/tag-helpers/intro) e auxiliares HTML *funcionam todos apenas* com as propriedades definidas em uma classe de Página do Razor. Considere uma página que implementa um formulário básico "Fale conosco" para o modelo `Contact`:

Para as amostras neste documento, o `DbContext` é inicializado no arquivo [Startup.cs](https://github.com/aspnet/Docs/blob/master/aspnetcore/mvc/razor-pages/index/sample/RazorPagesContacts/Startup.cs#L15-L16).

[!code-cs[main](index/sample/RazorPagesContacts/Startup.cs?highlight=15-16)]

O modelo de dados:

[!code-cs[main](index/sample/RazorPagesContacts/Data/Customer.cs)]

O contexto do banco de dados:

[!code-cs[main](index/sample/RazorPagesContacts/Data/AppDbContext.cs)]

O arquivo de exibição *Pages/Create.cshtml*:

[!code-cshtml[main](index/sample/RazorPagesContacts/Pages/Create.cshtml)]

O arquivo code-behind *Pages/Create.cshtml.cs* para a exibição:

[!code-cs[main](index/sample/RazorPagesContacts/Pages/Create.cshtml.cs?name=snippet_ALL)]

Por convenção, a classe `PageModel` é chamada de `<PageName>Model` e está no mesmo namespace que a página.

O uso de um arquivo code-behind `PageModel` dá suporte a teste de unidade, mas exige que você grave uma classe e um construtor explícitos. Páginas sem arquivos code-behind `PageModel` dão suporte a compilação de tempo de execução, o que pode ser uma vantagem no desenvolvimento.  <!-- review: advantage because you can make changes and refresh the browser without explicitly compiling the app -->

A página tem um *método de manipulador* `OnPostAsync`, que é executado em solicitações `POST` (quando um usuário posta o formulário). Você pode adicionar métodos de manipulador para qualquer verbo HTTP. Os manipuladores mais comuns são:

* `OnGet` para inicializar o estado necessário para a página. Amostra de [OnGet](#OnGet).
* `OnPost` para manipular envios de formulário.

O sufixo de nomenclatura `Async` é opcional, mas geralmente é usado por convenção para funções assíncronas. O código `OnPostAsync` no exemplo anterior tem aparência semelhante ao que você normalmente escreve em um controlador. O código anterior é comum para as Páginas do Razor. A maioria dos primitivos MVC como [associação de modelos](xref:mvc/models/model-binding), [validação](xref:mvc/models/validation) e resultados da ação são compartilhados.  <!-- Review: Ryan, can we get a list of what is shared and what isn't? -->

O método `OnPostAsync` anterior:

[!code-cs[main](index/sample/RazorPagesContacts/Pages/Create.cshtml.cs?name=snippet_OnPostAsync)]

O fluxo básico de `OnPostAsync`:

Verifique se há erros de validação.

*  Se não houver nenhum erro, salve os dados e redirecione.
*  Se houver erros, mostre a página novamente com as mensagens de validação. A validação do lado do cliente é idêntica para aplicativos ASP.NET Core MVC tradicionais. Em muitos casos, erros de validação seriam detectados no cliente e nunca enviados ao servidor.

Quando os dados são inseridos com êxito, o método de manipulador `OnPostAsync` chama o método auxiliar `RedirectToPage` para retornar uma instância de `RedirectToPageResult`. `RedirectToPage` é um novo resultado de ação, semelhante a `RedirectToAction` ou `RedirectToRoute`, mas personalizado para páginas. Na amostra anterior, ele redireciona para a página de Índice raiz (`/Index`). `RedirectToPage` é descrito em detalhes na seção [Geração de URLs para páginas](#url_gen).

Quando o formulário enviado tem erros de validação (que são passados para o servidor), o método de manipulador `OnPostAsync` chama o método auxiliar `Page`. `Page` retorna uma instância de `PageResult`. Retornar `Page` é semelhante a como as ações em controladores retornam `View`. `PageResult` é o tipo de retorno <!-- Review  --> padrão para um método de manipulador. Um método de manipulador que retorna `void` renderiza a página.

A propriedade `Customer` usa o atributo `[BindProperty]` para aceitar a associação de modelos.

[!code-cs[main](index/sample/RazorPagesContacts/Pages/Create.cshtml.cs?name=snippet_PageModel&highlight=10-11)]

Páginas do Razor, por padrão, associam as propriedades somente com verbos não GET. A associação de propriedades pode reduzir a quantidade de código que você precisa escrever. A associação reduz o código usando a mesma propriedade para renderizar os campos de formulário (`<input asp-for="Customer.Name" />`) e aceitar a entrada.

A home page (*Index.cshtml*):

[!code-cshtml[main](index/sample/RazorPagesContacts/Pages/Index.cshtml)]

O arquivo code-behind *Index.cshtml.cs*:

[!code-cs[main](index/sample/RazorPagesContacts/Pages/Index.cshtml.cs)]

O arquivo *cshtml* contém a marcação a seguir para criar um link de edição para cada contato:

[!code-cshtml[main](index/sample/RazorPagesContacts/Pages/Index.cshtml?range=21)]

O [auxiliar de marcas de âncora](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper) usou o atributo [asp-route-{valor}](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper#route) para gerar um link para a página Edit. O link contém dados de rota com a ID de contato. Por exemplo, `http://localhost:5000/Edit/1`.

O arquivo *Pages/Edit.cshtml*:

[!code-cshtml[main](index/sample/RazorPagesContacts/Pages/Edit.cshtml?highlight=1)]

A primeira linha contém a diretiva `@page "{id:int}"`. A restrição de roteamento `"{id:int}"` informa à página para aceitar solicitações para a página que contêm dados da rota `int`. Se uma solicitação para a página não contém dados de rota que podem ser convertidos em um `int`, o tempo de execução retorna um erro HTTP 404 (não encontrado).

O arquivo *Pages/Edit.cshtml.cs*:

[!code-cs[main](index/sample/RazorPagesContacts/Pages/Edit.cshtml.cs)]

O arquivo *Index.cshtml* também contém a marcação para criar um botão de exclusão para cada contato de cliente:

[!code-cshtml[main](index/sample/RazorPagesContacts/Pages/Index.cshtml?range=22-23)]

Quando o botão de exclusão é renderizado em HTML, seu `formaction` inclui parâmetros para:

* A ID de contato do cliente especificada pelo atributo `asp-route-id`.
* O `handler` especificado pelo atributo `asp-page-handler`.

Este é um exemplo de um botão de exclusão renderizado com uma ID de contato do cliente de `1`:

```html
<button type="submit" formaction="/?id=1&amp;handler=delete">delete</button>
```

Quando o botão é selecionado, uma solicitação de formulário `POST` é enviada para o servidor. Por convenção, o nome do método do manipulador é selecionado com base no valor do parâmetro `handler` de acordo com o esquema `OnPost[handler]Async`.

Como o `handler` é `delete` neste exemplo, o método do manipulador `OnPostDeleteAsync` é usado para processar a solicitação `POST`. Se o `asp-page-handler` for definido como um valor diferente, como `remove`, um método de manipulador de página com o nome `OnPostRemoveAsync` será selecionado.

[!code-cs[main](index/sample/RazorPagesContacts/Pages/Index.cshtml.cs?range=26-37)]

O método `OnPostDeleteAsync`:

* Aceita o `id` da cadeia de caracteres de consulta.
* Consulta o banco de dados para o contato de cliente com `FindAsync`.
* Se o contato do cliente for encontrado, eles serão removidos da lista de contatos do cliente. O banco de dados é atualizado.
* Chama `RedirectToPage` para redirecionar para a página de índice de raiz (`/Index`).

<a name="xsrf"></a>

## <a name="xsrfcsrf-and-razor-pages"></a>XSRF/CSRF e Páginas do Razor

Você não precisa escrever nenhum código para [validação antifalsificação](xref:security/anti-request-forgery). Validação e geração de token antifalsificação são automaticamente incluídas nas Páginas do Razor.

<a name="layout"></a>
## <a name="using-layouts-partials-templates-and-tag-helpers-with-razor-pages"></a>Usando Layouts, parciais, modelos e auxiliares de marcas com Páginas do Razor

As Páginas funcionam com todos os recursos do mecanismo de exibição do Razor. Layouts, parciais, modelos, auxiliares de marcas, *_ViewStart.cshtml* e *_ViewImports.cshtml* funcionam da mesma forma que funcionam exibições convencionais do Razor.

Organizaremos essa página aproveitando alguns desses recursos.

Adicione uma [página de layout](xref:mvc/views/layout) a *Pages/_Layout.cshtml*:

[!code-cshtml[main](index/sample/RazorPagesContacts2/Pages/_LayoutSimple.cshtml)]

O [Layout](xref:mvc/views/layout):

* Controla o layout de cada página (a menos que a página opte por não usar o layout).
* Importa estruturas HTML como JavaScript e folhas de estilo.

Veja [página de layout](xref:mvc/views/layout) para obter mais informações.

A propriedade [Layout](xref:mvc/views/layout#specifying-a-layout) é definida em *Pages/_ViewStart.cshtml*:

[!code-cshtml[main](index/sample/RazorPagesContacts2/Pages/_ViewStart.cshtml)]

**Observação**: o layout está na pasta *Pages*. As páginas buscam outras exibições (layouts, modelos, parciais) hierarquicamente, iniciando na mesma pasta que a página atual. Um layout na pasta *Pages* pode ser usado em qualquer Página do Razor na pasta *Pages*.

Recomendamos que você **não** coloque o arquivo de layout na pasta *Views/Shared*. *Views/Shared* é um padrão de exibições do MVC. As Páginas do Razor devem confiar na hierarquia de pasta e não nas convenções de caminho.

A pesquisa de modo de exibição de uma Página do Razor inclui a pasta *Pages*. Os layouts, modelos e parciais que você está usando com controladores MVC e exibições do Razor convencionais *apenas funcionam*.

Adicione um arquivo *Pages/_ViewImports.cshtml*:

[!code-cshtml[main](index/sample/RazorPagesContacts2/Pages/_ViewImports.cshtml)]

`@namespace` é explicado posteriormente no tutorial. A diretiva `@addTagHelper` coloca os [auxiliares de marcas internos](xref:mvc/views/tag-helpers/builtin-th/Index) em todas as páginas na pasta *Pages*.

<a name="namespace"></a>

Quando a diretiva `@namespace` é usada explicitamente em uma página:

[!code-cshtml[main](index/sample/RazorPagesIntro/Pages/Customers/Namespace2.cshtml?highlight=2)]

A diretiva define o namespace da página. A diretiva `@model` não precisa incluir o namespace.

Quando a diretiva `@namespace` está contida em *_ViewImports.cshtml*, o namespace especificado fornece o prefixo do namespace gerado na página que importa a diretiva `@namespace`. O restante do namespace gerado (a parte do sufixo) é o caminho relativo separado por ponto entre a pasta que contém *_ViewImports.cshtml* e a pasta que contém a página.

Por exemplo, o arquivo code-behind *Pages/Customers/Edit.cshtml.cs* define explicitamente o namespace:

[!code-cs[main](index/sample/RazorPagesContacts2/Pages/Customers/Edit.cshtml.cs?name=snippet_namespace)]

O arquivo *Pages/_ViewImports.cshtml* define o namespace a seguir:

[!code-cshtml[main](index/sample/RazorPagesContacts2/Pages/_ViewImports.cshtml?highlight=1)]

O namespace gerado para a Página do Razor *Pages/Customers/Edit.cshtml* é o mesmo que o do arquivo code-behind. A diretiva `@namespace` foi projetada de modo que as classes C# adicionadas a um projeto e o código gerado pelas páginas *funcione* sem a necessidade de adicionar uma diretiva `@using` para o arquivo code-behind.

**Observação:** `@namespace` também funciona com exibições do Razor convencionais.

O arquivo de exibição *Pages/Create.cshtml* original:

[!code-cshtml[main](index/sample/RazorPagesContacts/Pages/Create.cshtml?highlight=2)]

O arquivo de exibição *Pages/Create.cshtml* atualizado:

[!code-cshtml[main](index/sample/RazorPagesContacts2/Pages/Customers/Create.cshtml?highlight=2)]

O [projeto inicial de Páginas do Razor](#rpvs17) contém o *Pages/_ValidationScriptsPartial.cshtml*, que conecta a validação do lado do cliente.

<a name="url_gen"></a>

## <a name="url-generation-for-pages"></a>Geração de URL para Páginas

A página `Create`, exibida anteriormente, usa `RedirectToPage`:

[!code-cs[main](index/sample/RazorPagesContacts/Pages/Create.cshtml.cs?name=snippet_OnPostAsync&highlight=10)]

O aplicativo tem a estrutura de arquivos/pastas a seguir:

* */Pages*

  * *Index.cshtml*
  * */Customer*

    * *Create.cshtml*
    * *Edit.cshtml*
    * *Index.cshtml*

As páginas *Pages/Customers/Create.cshtml* e *Pages/Customers/Edit.cshtml* redirecionam para o *Pages/Index.cshtml* após êxito. A cadeia de caracteres `/Index` faz parte do URI para acessar a página anterior. A cadeia de caracteres `/Index` pode ser usada para gerar URIs para a página *Pages/Index.cshtml*. Por exemplo:

* `Url.Page("/Index", ...)`
* `<a asp-page="/Index">My Index Page</a>`
* `RedirectToPage("/Index")`

O nome da página é o caminho para a página da pasta raiz */Pages* (incluindo um `/` à direita, por exemplo, `/Index`). As amostras anteriores de geração de URL são muito mais ricas em recursos do que apenas codificar uma URL. A geração de URL usa [roteamento](xref:mvc/controllers/routing) e pode gerar e codificar parâmetros de acordo com o modo como a rota é definida no caminho de destino.

A Geração de URL para páginas dá suporte a nomes relativos. A tabela a seguir mostra qual página de Índice é selecionada com diferentes parâmetros `RedirectToPage` de *Pages/Customers/Create.cshtml*:

| RedirectToPage(x)| Página |
| ----------------- | ------------ |
| RedirectToPage("/Index") | *Pages/Index* |
| RedirectToPage("./Index"); | *Pages/Customers/Index* |
| RedirectToPage("../Index") | *Pages/Index* |
| RedirectToPage("Index")  | *Pages/Customers/Index* |

`RedirectToPage("Index")`, `RedirectToPage("./Index")` e `RedirectToPage("../Index")` são *nomes relativos*. O parâmetro `RedirectToPage` é *combinado* com o caminho da página atual para calcular o nome da página de destino.  <!-- Review: Original had The provided string is combined with the page name of the current page to compute the name of the destination page. -- page name, not page path -->

Vinculação de nome relativo é útil ao criar sites com uma estrutura complexa. Se você usar nomes relativos para vincular entre páginas em uma pasta, você poderá renomear essa pasta. Todos os links ainda funcionarão (porque eles não incluirão o nome da pasta).

## <a name="tempdata"></a>TempData

O ASP.NET Core expõe a propriedade [TempData](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.controller.tempdata?view=aspnetcore-2.0#Microsoft_AspNetCore_Mvc_Controller_TempData) em um [controlador](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.controller). Essa propriedade armazena dados até eles serem lidos. Os métodos `Keep` e `Peek` podem ser usados para examinar os dados sem exclusão. `TempData` é útil para redirecionamento nos casos em que os dados são necessários para mais de uma única solicitação.

O atributo `[TempData]` é novo no ASP.NET Core 2.0 e tem suporte em controladores e páginas.

Os conjuntos de código a seguir definem o valor de `Message` usando `TempData`:

[!code-cs[main](index/sample/RazorPagesContacts2/Pages/Customers/CreateDot.cshtml.cs?highlight=10-11,25&name=snippet_Temp)]

A marcação a seguir no arquivo *Pages/Customers/Index.cshtml* exibe o valor de `Message` usando `TempData`.

```cshtml
<h3>Msg: @Model.Message</h3>
```

O arquivo code-behind *Pages/Customers/Index.cshtml.cs* aplica o atributo `[TempData]` à propriedade `Message`.

```cs
[TempData]
public string Message { get; set; }
```

Consulte [TempData](xref:fundamentals/app-state#temp) para obter mais informações.

<a name="mhpp"></a>
## <a name="multiple-handlers-per-page"></a>Vários manipuladores por página

A página a seguir gera marcação para dois manipuladores de página usando o auxiliar de marcas `asp-page-handler`:

[!code-cshtml[main](index/sample/RazorPagesContacts2/Pages/Customers/CreateFATH.cshtml?highlight=12-13)]

<!-- Review: the FormActionTagHelper applies to all <form /> elements on a Razor page, even when there is no `asp-` attribute   -->

O formulário no exemplo anterior tem dois botões de envio, cada um usando o `FormActionTagHelper` para enviar para uma URL diferente. O atributo `asp-page-handler` é um complemento para `asp-page`. `asp-page-handler` gera URLs que enviam para cada um dos métodos de manipulador definidos por uma página. `asp-page` não foi especificado porque a amostra está vinculando à página atual.

O arquivo code-behind:

[!code-cs[main](index/sample/RazorPagesContacts2/Pages/Customers/CreateFATH.cshtml.cs?highlight=20,32)]

O código anterior usa *métodos de manipulador nomeados*. Métodos de manipulador nomeados são criados colocando o texto no nome após `On<HTTP Verb>` e antes de `Async` (se houver). No exemplo anterior, os métodos de página são OnPost**JoinList**Async e OnPost**JoinListUC**Async. Com *OnPost* e *Async* removidos, os nomes de manipulador são `JoinList` e `JoinListUC`.

[!code-cshtml[main](index/sample/RazorPagesContacts2/Pages/Customers/CreateFATH.cshtml?range=12-13)]

Usando o código anterior, o caminho da URL que envia a `OnPostJoinListAsync` é `http://localhost:5000/Customers/CreateFATH?handler=JoinList`. O caminho da URL que envia a `OnPostJoinListUCAsync` é `http://localhost:5000/Customers/CreateFATH?handler=JoinListUC`.

## <a name="customizing-routing"></a>Personalizando o roteamento

Se você não deseja a cadeia de consulta `?handler=JoinList` na URL, você pode alterar a rota para colocar o nome do manipulador na parte do caminho da URL. Você pode personalizar a rota adicionando um modelo de rota entre aspas duplas após a diretiva `@page`.

[!code-cshtml[main](index/sample/RazorPagesContacts2/Pages/Customers/CreateRoute.cshtml?highlight=1)]

A rota anterior coloca o nome do manipulador no caminho da URL em vez da cadeia de consulta. O `?` após `handler` significa que o parâmetro de rota é opcional.

Você pode usar `@page` para adicionar parâmetros e segmentos adicionais a uma rota de página. Tudo que está lá está **acrescentado** à rota padrão da página. Não há suporte para o uso de um caminho absoluto ou virtual para alterar a rota da página (como `"~/Some/Other/Path"`).

## <a name="configuration-and-settings"></a>Configuração e definições

Para configurar opções avançadas, use o método de extensão `AddRazorPagesOptions` no construtor de MVC:

[!code-cs[main](index/sample/RazorPagesContacts/StartupAdvanced.cs?name=snippet_1)]

No momento, você pode usar o `RazorPagesOptions` para definir o diretório raiz para páginas ou adicionar as convenções de modelo de aplicativo para páginas. Permitiremos mais extensibilidade dessa maneira no futuro.

Para pré-compilar exibições, consulte [Compilação de exibição do Razor](xref:mvc/views/view-compilation).

[Baixar ou exibir código de exemplo](https://github.com/aspnet/Docs/tree/master/aspnetcore/mvc/razor-pages/index/sample).

Consulte [Introdução a Páginas do Razor no ASP.NET Core](xref:tutorials/razor-pages/razor-pages-start), que se baseia nesta introdução.

### <a name="specify-that-razor-pages-are-at-the-content-root"></a>Especificar que as Páginas Razor estão na raiz do conteúdo

Por padrão, as Páginas Razor estão na raiz do diretório */Pages*. Adicione [WithRazorPagesAtContentRoot](/dotnet/api/microsoft.extensions.dependencyinjection.mvcrazorpagesmvcbuilderextensions.withrazorpagesatcontentroot) em [AddMvc](/dotnet/api/microsoft.extensions.dependencyinjection.mvcservicecollectionextensions.addmvc#Microsoft_Extensions_DependencyInjection_MvcServiceCollectionExtensions_AddMvc_Microsoft_Extensions_DependencyInjection_IServiceCollection_) para especificar que as Páginas Razor estão na raiz do conteúdo ([ContentRootPath](/dotnet/api/microsoft.aspnetcore.hosting.ihostingenvironment.contentrootpath)) do aplicativo:

```csharp
services.AddMvc()
    .AddRazorPagesOptions(options =>
    {
        ...
    })
    .WithRazorPagesAtContentRoot();
```

### <a name="specify-that-razor-pages-are-at-a-custom-root-directory"></a>Especificar que as Páginas Razor estão em um diretório raiz personalizado

Adicione [WithRazorPagesRoot](/dotnet/api/microsoft.extensions.dependencyinjection.mvcrazorpagesmvccorebuilderextensions.withrazorpagesroot) em [AddMvc](/dotnet/api/microsoft.extensions.dependencyinjection.mvcservicecollectionextensions.addmvc#Microsoft_Extensions_DependencyInjection_MvcServiceCollectionExtensions_AddMvc_Microsoft_Extensions_DependencyInjection_IServiceCollection_) para especificar que as Páginas Razor estão em um diretório raiz personalizado no aplicativo (forneça um caminho relativo):

```csharp
services.AddMvc()
    .AddRazorPagesOptions(options =>
    {
        ...
    })
    .WithRazorPagesRoot("/path/to/razor/pages");
```

## <a name="see-also"></a>Consulte também

* [Introdução a Páginas do Razor](xref:tutorials/razor-pages/razor-pages-start)
* [Convenções de autorização de Páginas Razor](xref:security/authorization/razor-pages-authorization)
* [Provedores de modelo personalizado de página e rota de Páginas Razor](xref:mvc/razor-pages/razor-pages-convention-features)
* [Testes de integração e unidade de Páginas Razor](xref:testing/razor-pages-testing)

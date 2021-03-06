---
uid: aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/distributed-caching
title: "(Criando aplicativos de nuvem do mundo Real com o Azure) de cache distribuído | Microsoft Docs"
author: MikeWasson
description: "Os aplicativos de nuvem criando Real World com livro eletrônico do Azure baseia-se em uma apresentação desenvolvida por Scott Guthrie. Ele explica 13 padrões e práticas recomendadas que ele..."
ms.author: aspnetcontent
manager: wpickett
ms.date: 07/20/2015
ms.topic: article
ms.assetid: 406518e9-3817-49ce-8b90-e82bc461e2c0
ms.technology: 
ms.prod: .net-framework
msc.legacyurl: /aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/distributed-caching
msc.type: authoredcontent
ms.openlocfilehash: 923a8257376e98e6cae10d905f1cb18f7fdb28e7
ms.sourcegitcommit: 9a9483aceb34591c97451997036a9120c3fe2baf
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/10/2017
---
<a name="distributed-caching-building-real-world-cloud-apps-with-azure"></a>Cache (Criando nuvem do mundo Real aplicativos distribuídos com o Azure)
====================
por [Mike Wasson](https://github.com/MikeWasson), [Rick Anderson](https://github.com/Rick-Anderson), [Tom Dykstra](https://github.com/tdykstra)

[Download corrigi-lo projeto](http://code.msdn.microsoft.com/Fix-It-app-for-Building-cdd80df4) ou [baixar livro eletrônico](http://blogs.msdn.com/b/microsoft_press/archive/2014/07/23/free-ebook-building-cloud-apps-with-microsoft-azure.aspx)

> O **nuvem aplicativos do mundo Real criando com o Azure** livro eletrônico baseia-se em uma apresentação desenvolvida por Scott Guthrie. Explica as 13 padrões e práticas que podem ajudá-lo a ser bem-sucedido de desenvolvimento de aplicativos da web para a nuvem. Para obter informações sobre o livro eletrônico, consulte [primeiro capítulo](introduction.md).


O capítulo anterior examinou o tratamento de falhas transitórias e mencionado como uma estratégia de disjuntor de cache. Este capítulo fornece mais informações sobre o cache, incluindo quando usá-lo, padrões comuns de uso e como implementá-la no Azure.

## <a name="what-is-distributed-caching"></a>O que é cache distribuído

Um cache fornece alta taxa de transferência, baixa latência de acesso aos dados do aplicativo comumente acessadas, armazenando os dados na memória. Para um aplicativo de nuvem o tipo mais úteis de cache é cache distribuído, o que significa que os dados não são armazenados na memória do servidor web individuais, mas em outros recursos de nuvem e os dados em cache são disponibilizados para todos os servidores de web do aplicativo (ou outra nuvem VMs que ar e usado pelo aplicativo).

![Diagrama mostrando vários servidores web ao acessar os mesmos servidores de cache](distributed-caching/_static/image1.png)

Quando o aplicativo é dimensionado adicionando ou removendo servidores, ou quando os servidores são substituídos devido a atualizações ou falhas, os dados em cache permanecem acessíveis para cada servidor que executa o aplicativo.

Ao evitar o acesso de dados de alta latência de um repositório de dados persistentes, o cache pode melhorar drasticamente capacidade de resposta do aplicativo. Por exemplo, recuperando dados do cache é muito mais rápido que recuperá-los de um banco de dados relacional.

Um benefício adicional de armazenamento em cache é reduzido encargos de tráfego para o repositório de dados persistentes, que pode resultar em redução dos custos quando houver saída de dados do repositório de dados persistentes.

## <a name="when-to-use-distributed-caching"></a>Quando usar o cache distribuído

Armazenamento em cache funciona melhor para cargas de trabalho de aplicativos que não mais lendo de gravação de dados, e quando o modelo de dados oferece suporte a organização de chave/valor que você pode usar para armazenar e recuperar dados em cache. Também é mais útil quando os usuários do aplicativo compartilham muitos dados comuns; Por exemplo, cache não fornecem benefícios como muitos se cada usuário normalmente recupera dados exclusivos para esse usuário. Um exemplo em que o cache pode ser muito útil é um catálogo de produtos, porque os dados não são alterados com frequência, e todos os clientes estão observando os mesmos dados.

O benefício de cache se torna cada vez mais mensurável mais um aplicativo pode ser dimensionado, como os limites de taxa de transferência e os atrasos de latência de armazenamento de dados persistentes tornam-se mais de um limite de desempenho geral do aplicativo. No entanto, você pode implementar o armazenamento em cache por outros motivos de desempenho também. Para dados não precisam ser perfeitamente atualizado quando mostrado ao usuário, acesso ao cache pode servir como um disjuntor para quando o repositório de dados persistente está respondendo ou não está disponível.

## <a name="popular-cache-population-strategies"></a>Estratégias de população de cache populares

Para poder recuperar dados do cache, você precisa armazená-la lá primeiro. Há várias estratégias para obtenção de dados necessários em um cache:

- Sob demanda / Cache Aside

    O aplicativo tenta recuperar dados do cache e quando o cache não tiver os dados (um "erro"), o aplicativo armazena os dados em cache para que ele esteja disponível na próxima vez. Na próxima vez que o aplicativo tenta obter os mesmos dados, ele localiza o que está procurando no cache (um "ocorrências"). Para evitar a busca de dados armazenados em cache que foram alterados no banco de dados, você invalida o cache ao fazer alterações ao repositório de dados.
- Envio de dados do plano de fundo

    Serviços em segundo plano enviar dados por push para o cache em um agendamento regular e o aplicativo sempre recebe do cache. Essa abordagem funciona bem com fontes de dados de alta latência que não exigem que você sempre retorna os dados mais recentes.
- Disjuntor

    O aplicativo normalmente se comunica diretamente com o repositório de dados persistentes, mas quando o repositório de dados persistentes tem problemas de disponibilidade, o aplicativo recupera dados do cache. Dados podem ter sido colocados em cache usando cache separadamente ou estratégia de envio de dados em segundo plano. Esta é a estratégia em vez de uma estratégia de melhoria de desempenho de controle de falhas.

Para manter os dados no cache atual, você pode excluir as entradas de cache relacionados quando seu aplicativo cria, atualizações, ou exclui dados. Se ele estiver muito bem para seu aplicativo para muitas vezes obter dados é um pouco desatualizados, você pode contar com um tempo de expiração configuráveis para definir um limite no cache de quanto tempo os dados podem ser.

Você pode configurar a expiração absoluta (quantidade de tempo desde que o item de cache foi criado) ou deslizando validade (período de tempo desde a última vez em que um item de cache foi acessado). Expiração absoluta é usada quando você dependem do mecanismo de expiração de cache para impedir que os dados se tornam muito desatualizados. No aplicativo corrigi-lo, podemos irá remover manualmente itens de cache obsoleto e usaremos a expiração deslizante para manter os dados mais atuais no cache. Independentemente da política de expiração que você escolher, o cache removerá automaticamente os itens mais antigos (menos usado recentemente ou LRU) quando é atingido o limite de memória do cache.

## <a name="sample-cache-aside-code-for-fix-it-app"></a>Exemplo de código cache-aside corrigir o aplicativo

No código de exemplo a seguir, verificamos o cache primeiro ao recuperar uma tarefa corrigir. Se a tarefa for encontrada no cache, retornamos Se não for encontrado, podemos obtê-lo do banco de dados e armazená-lo no cache. As alterações que você faria ao adicionar o cache para o `FindTaskByIdAsync` método são realçados.

[!code-csharp[Main](distributed-caching/samples/sample1.cs?highlight=5,9-11,13-15,19)]

Quando você atualiza ou excluir uma tarefa corrigi-lo, você precisa invalidar tarefa (remover) o cache. Caso contrário, futuras tentativas de tarefa continuarão a obter os dados antigos do cache de leitura.

[!code-csharp[Main](distributed-caching/samples/sample2.cs?highlight=7)]

Estes são exemplos para ilustrar o código de cache simple. o cache não foi implementado no download corrigir projeto.

## <a name="azure-caching-services"></a>Serviços de cache do Azure

O Azure oferece os seguintes serviços de cache: [Cache Redis do Azure](https://msdn.microsoft.com/en-us/library/dn690523.aspx) e [Cache gerenciado do Azure](https://msdn.microsoft.com/en-us/library/dn386094.aspx). Cache Redis do Azure baseia-se no conhecido [Abrir fonte de Cache Redis](http://redis.io/) e é a primeira opção para a maioria dos cenários de cache.

<a id="sessionstate"></a>
## <a name="aspnet-session-state-using-a-cache-provider"></a>Estado da sessão ASP.NET usando um provedor de cache

Conforme mencionado no [capítulo de práticas recomendado do web desenvolvimento](web-development-best-practices.md), é uma prática recomendada evitar o uso de estado de sessão. Se seu aplicativo requer o estado de sessão, a próxima melhor prática é evitar o provedor de memória padrão porque que não permitem a expansão (várias instâncias do servidor web). O provedor de estado de sessão do SQL Server ASP.NET permite que um site que é executado em vários servidores web para usar o estado de sessão, mas ela incorre em um custo de alta latência, em comparação comparado um provedor na memória. A melhor solução se você tiver que usar o estado de sessão é usar um provedor de cache, como o [provedor de estado de sessão para o Cache do Azure](https://msdn.microsoft.com/en-us/library/windowsazure/gg185668.aspx).

## <a name="summary"></a>Resumo

Você viu como o aplicativo corrigir pode implementar cache para melhorar a escalabilidade e o tempo de resposta e para permitir que o aplicativo continue responsivo para operações de leitura quando o banco de dados não está disponível. No [próximo capítulo](queue-centric-work-pattern.md) vamos mostrar como melhorar a escalabilidade e fazer com que o aplicativo continue a resposta para operações de gravação.

## <a name="resources"></a>Recursos

Para obter mais informações sobre armazenamento em cache, consulte os seguintes recursos.

Documentação

- [Cache do Azure](https://msdn.microsoft.com/en-us/library/gg278356.aspx). Documentação oficial do MSDN em cache no Azure.
- [Padrões e práticas - diretrizes do Azure Microsoft](https://msdn.microsoft.com/en-us/library/dn568099.aspx). Consulte as diretrizes do cache e padrão Cache-Aside.
- [À prova de falhas: Orientação para arquiteturas resilientes na nuvem](https://msdn.microsoft.com/en-us/library/windowsazure/jj853352.aspx). White paper Marc Mercuri, Ulrich Homann e Andrew Townhill. Consulte a seção em cache.
- [Práticas recomendadas para o Design de serviços em grande escala em serviços de nuvem do Azure](https://msdn.microsoft.com/en-us/library/windowsazure/jj717232.aspx). W. White paper, Mark Simms e Michael Thomassy. Consulte a seção sobre armazenamento em cache distribuído.
- [Caching no caminho para escalabilidade distribuído](https://msdn.microsoft.com/en-us/magazine/dd942840.aspx). Um artigo de revista MSDN (2009) mais antigo, mas uma introdução clareza ao cache distribuído em geral; entra em mais detalhes do que o cache seções os white papers à prova de falhas e práticas recomendadas.

Vídeos

- [À prova de falhas: Criação de serviços de nuvem escalonáveis e resilientes](https://channel9.msdn.com/Series/FailSafe). Série de nove partes por Marc Mercuri, Ulrich Homann e Mark Simms. Apresenta uma exibição de nível 400 de como projetar aplicativos de nuvem. Essa série se concentra em teoria e motivos por quê; Para obter mais detalhes instruções, consulte a série de construção grande por Mark Simms. Consulte a discussão de cache no episódio 3 começando em 1:24:14.
- [Criando Big: Lições aprendidas de clientes do Azure - parte I](https://channel9.msdn.com/Events/Build/2012/3-029). Simon Davies discute começando de cache distribuído em 46:00. Semelhante ao entrar em mais detalhes instruções mas série à prova de falhas. A apresentação foi atribuída a 31 de outubro de 2012, portanto ele não aborda o serviço de cache de aplicativos Web no serviço de aplicativo do Azure que foi introduzido em 2013.

Exemplo de código

- [Conceitos fundamentais do serviço no Azure na nuvem](https://code.msdn.microsoft.com/Cloud-Service-Fundamentals-4ca72649). Aplicativo de exemplo que implementa o cache distribuído. Consulte a postagem do blog que acompanha [conceitos básicos de serviço de nuvem – Noções básicas de cache](https://blogs.msdn.com/b/windowsazure/archive/2013/10/03/cloud-service-fundamentals-caching-basics.aspx).

>[!div class="step-by-step"]
[Anterior](transient-fault-handling.md)
[Próximo](queue-centric-work-pattern.md)

---
layout: post
title: Protocolo HTTP - Além do que vemos
date: 2018-07-09 18:02:11 +0300
description: Protocolo HTTP Além do que vemos
img: http-behind-the-eyes.jpg
img-credits: Caspar Rubin on Unsplash
fig-caption: 
tags: [HTTP, Web]
---

Quando estamos aprendendo sobre desenvolvimento web é bem comum iniciarmos criando páginas com html, css e javascript e ficarmos bastante empolgados com o que estamos criando. Costumo dizer que desenvolver páginas com formulários ou listagem de elementos é bem legal pois já vemos o resultado instantaneamente e isso tende a ser bastante motivador para quem está começando. Entretanto, tão importante quanto ver o resultado é entender o que acontece "além do que vemos".  
  
Por trás de uma página carregada em um navegador (browser), seja em um computador convencional ou smartphone, existem tecnologias e padrões importantes que garantem seu funcionamento. O protocolo HTTP (Hypertext Transfer Protocol) é um desses padrões que desenvolvedores web devem conhecer. O protocolo http funciona sobre a camada mais alta do modelo de redes, a camada de aplicação.  
  
### Protocolo  
Um protocolo é um padrão ou norma estabelecidos para a realizar um procedimento ou processo. Por exemplo, na língua portuguesa e nas mais comuns que conhecemos, escrevemos em uma página seguindo um fluxo de cima para baixo, da esquerda para direita e quem lê a página deve seguir o mesmo fluxo para a compreensão do conteúdo, isso é um padrão. Na comunicação entre sistemas web o protocolo http possibilita a comunicação entre uma ponta e outra pois estabelece padrões universais que tanto o criador quanto o consumidor da mensagem conhecem.    
  
### Cliente e Servidor  
A maioria dos sistemas web se baseiam no modelo cliente e servidor representada pela imagem abaixo:  

![Exemplo http response 200]({{ "/assets/img/client-server.jpg" | absolute_url }}){:class="img-responsive"}  
  
Neste exemplo do modelo cliente e servidor temos três dispositivos (cliente) que se conectam pela internet (meio de comunicação) a um computador centralizador de recursos (servidor). Essa é a arquitetura mais utilizada por sites e sistemas web.   
  
### Recursos  
Recursos são conteúdos como páginas, imagens, arquivos, vídeos, etc. que normalmente acessamos na web. Para um cliente acessar um recurso que está em um servidor ambos estabelecem uma conexão (que explicarei mais adiante) e realizam trocas de mensagens padronizadas pelo protocolo http por meio de **requisição** e **resposta**.   
  
### Como funciona  
Fazendo uma analogia, imagine que você foi em uma hamburgueria, pediu um hambúrguer para o atendente e minutos depois o pedido esta em suas mãos. Agora vamos extrair alguns elementos deste cenário e comparar com o que aprendemos até aqui.  
  
 - **Cliente**: você, feliz e contente à beira do balcão;  
 - **Servidor**: o atendente;  
 - **Recurso**: o hambúrguer;  
 - **Meio de comunicação**: o balcão (ou local);  
 - **Mensagem**: o que você e o atendente falam um para o outro.  
 - **Requisição**: sua mensagem para o atendente pedindo o hambúrguer;  
 - **Resposta**: o atendente te entregando te informando que o pedido está pronto e o entregando.  
 - **Protocolo**: o idioma que você e o atendente utilizaram para se comunicar.  
  
Agora utilizando um exemplo real, quando você digitou **http**s://evertoncastro.com/http-alem-do-que-vemos no navegador do seu dispositivo, ele agiu como um **cliente** que utilizou a internet como **meio de comunicação** para **requisitar** esta página (**recurso**) em um computador da *netlify* (onde eu hospedo este site) que tem a função de **servidor**. O servidor, por sua vez, **respondeu** a página no formato html por meio do mesmo caminho e o navegador do seu dispositivo finalmente mostrou o conteúdo para você.   
  
Em todo esse processo o protocolo http foi utilizado, garantindo que o servidor entendesse qual recurso foi solicitado e qual deveria ser o formato da resposta de modo que o navegador pudesse entende-la.  
  
Um exemplo bem legal sobre essa capacidade do navegador entender os padrões contidos no protocolo são os ***status codes***. O http tem como definição que as mensagens de resposta devem conter um código de status informando como o servidor processou a mensagem da requisição.  
  
Na imagem abaixo segue um print de uma requisição que fiz com o navegador Chrome para este mesmo site:  
![Exemplo http response 200]({{ "/assets/img/http-200.png" | absolute_url }}){:class="img-responsive"}  
Assim que a página carregou cliquei com o botão direito do mouse e acessei a opção inspecionar (inspect) para acessar o devtools e executei os passos descritos. Como você pode notar, a seção "cabeçalho" ou "headers" mostra o cabeçalho da resposta com informações importantes e podemos perceber que o *status code* é igual a 200. De acordo com o protocolo http o intervalo 20(X) informa que o servidor conseguiu processar a requisição e nos devolveu a resposta com sucesso. Para conhecer outros status [clique aqui:](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html){:target="_blank"}  
  
Agora vejamos qual será o status quando eu tentar acessar uma página ou recurso que não existe neste site:  
![Exemplo http response 400]({{ "/assets/img/http-400.png" | absolute_url }}){:class="img-responsive"}  
Ao acessar https://evertoncastro.com/qualquercoisa o servidor enviou uma mensagem no cabeçalho da resposta com o código 404. O código 404 no protocolo http informa que o servidor não conhece ou não encontrou o localizador **/qualquercoisa**  
  
### Mensagens e cabeçalhos  
As mensagens são textos padronizados, de acordo com as especificações do http, enviados na requisição pelo cliente e na resposta pelo servidor.   
As mensagens possuem informações como: host, método, versão do http e cabeçalhos quando estamos fazendo uma requisição e quando recebemos a resposta contém a versão http, código de status, cabeçalho e o corpo da resposta.  
  
Os cabeçalhos possuem diversas informações que variam na requisição e resposta. Não detalharei todas, mas por exemplo, no cabeçalho da requisição podemos ver nas imagens acima uma propriedade chamada  
*accept-encoding* informando para o servidor os tipos de encoding que o navegador suporta. Já no cabeçalho da resposta temos o *content-type* que especifica para o servidor o tipo de resposta que o cliente espera. No nosso caso esperávamos uma página do tipo *html*.   
  
### Conexão  
Como eu citei anteriormente, para que o cliente e o servidor troquem mensagens por meio de requisições e respostas é necessário que estabeleçam uma conexão. A conexão é como um túnel, pelo qual as mensagens irão trafegar. Contudo, essa conexão acontece em uma camada mais baixa na arquitetura de redes, a camada de transporte, através do protocolo TCP (Transmission Control Protocol).   
  
Explicando de forma geral o cliente envia uma solicitação para o servidor informando que quer abrir uma conexão, o servidor por sua vez informa que aceita a conexão e o cliente então informa que vai enviar requisições. Esse processo é conhecido como Three-way handshake ou "aperto de mão de três vias".  
  
Em outro post posso explicar melhor este processo como mais detalhes, mas quando digitamos https://evertoncastro.com o navegador tenta abrir uma conexão informando o host: evertoncastro.com e a porta 443 pois esse site utiliza o protocolo seguro, do contrário utilizaria a porta padrão 80. Após realizar o *handshake* o navegador através de camada de aplicação escreve as mensagens explicadas anteriormente em um *socket* que o protocolo TCP na camada de transporte tem a responsabilidade de entregar até o destinatário. Vale ressaltar que esse processo ainda passa por outras camadas mais baixas para ir e voltar do servidor.  
  
O protocolo TCP é um protocolo seguro pois garante que entregará a mensagem correta em todo o processo.  
  
### Versões  
O http existe desde 1991 com sua primeira verão 0.9. A maioria das versões implementadas hoje são 1.0 e 1.1. Mas ja temos a verão 2 com muitas melhorais que proporcionam melhor performance nas aplicações. [Ver](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Evolution_of_HTTP){:target="_blank"}  
  
### Como utilizar no dia a dia  
As linguagens de programação web já implementam o protocolo http abstraindo essa parte de conexão e troca de mensagens, tanto do lado do servidor quanto no cliente de modo que o desenvolvedor consiga pensar apenas no processamento e exibição das informações solicitadas e respondidas.  
  
Porém, ter essa base de conhecimento é muito importante pois pode ajudar o desenvolvedor a resolver problemas inesperados nas aplicações.  
  
Me lembro de uma vez em que desenvolvi uma página do lado servidor e o consumo da mesma no navegador, testei e subi em produção. Dias depois percebemos que no navegador internet explorer a página não estava sendo carregada como em outros navegadores. No momento não fez sentido para mim pois nos testes tudo estava funcionando. Foi então que um colega de equipe, mais experiente, me ajudou a identificar que o servidor não estava especificando o *content-type* como *html* no cabeçalho da resposta. Os outros navegadores possuíam um *fallback* (tratativa) por padrão e conseguiram identificar que o conteúdo da resposta era uma página html mas o Internet Explorer naquela versão específica não. A partir disso entendi que precisava entender melhor o protocolo mesmo com todas as facilidades que a linguagem e *framework* me forneciam.

### Sugestões
Abaixo segue algumas sugestões de estudo para fortalecer o conhecimento adquirido aqui.

 - [HTTP Succintly](https://www.syncfusion.com/ebooks/http){:target="_blank"}
 - [HTTP: The Definitive Guide](http://www.staroceans.org/e-book/O%27Reilly%20-%20HTTP%20-%20The%20Definitive%20Guide.pdf){:target="_blank"}
 - [Curso Online Http fundamentos](https://www.alura.com.br/curso-online-http-fundamentos){:target="_blank"}
 -  [Conhecendo a Web por debaixo dos panos](https://imasters.com.br/desenvolvimento/conhecendo-web-por-debaixo-dos-panos/){:target="_blank"}


Espero que este essa breve explicação possa te ajudar em seu estudo sobre desenvolvimento web.

Abraços!!
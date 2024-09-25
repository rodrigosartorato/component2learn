# Projeto Final

# Estrutura de Arquivos e Pastas

~~~
├── README.md  <- arquivo com o relatório do projeto
│
├── images     <- arquivos de imagens usadas no documento
│
└── app        <- aplicativo feito no MIT Inventor
~~~

# Relatório do Projeto

# Projeto `Brechó Online - Equipe 7`

# Equipe
* `Rafael Gonçalves Vastag`
* `Guilherme Cavassan`
* `Lucas Lopes Moreira`
* `Fabiano Louzada Cesário`
# Nível 1

## Diagrama Geral do Nível 1

![Modelo de diagrama no nível 1](images/projeto.png)

### Detalhamento da interação de componentes

* O componente `Login` recebe os atributos `Username` e `Password` para realizar a autenticação do usuário com sua conta no `BrechoApp`.

* O componente `BrechoApp` interage com `Login` através da interface `ILogin` para a autenticação do usuário. Esse é o componente central do sistema e é responsável por iniciar o fluxo de busca publicando uma mensagem no tópico `buscar/{userId}/request` através da interface `BuscarItem` e recebendo como resposta a mensagem publicada com `items/{userId}` por meio da interface `Itens`, nessa mensagem constará a lista de itens referentes a busca feita pelo usuário agregado com as ofertas disponíveis. Caso o usuário realize a compra de um desses itens será enviada uma mensagem no tópico  `pedido/{userId}/request` com os dados da compra. Este componente também é responsável por solicitar as opções de logistica disponíveis para o cliente publicando uma mensagem no tópico `entrega/{userId}/request` através da interface `BuscarLogistica` e recebendo como resposta a mensagem publicada com `entrega/{userId}/opcoes` por meio da interface `OpcoesLogistica`. O componente também solicita ao barramento através da interface `AcompanharRequest` e mensagem `status/{userId}/{pedidoId}` os status do pedido, recebendo esses dados como mensagem do tópico `status/{userId}/{pedidoId}/status/response`.
  
* O componente `Buscador` recebe uma mensagem `buscar/{userId}/request` da interface `PedidoBusca`, iniciando o processo de busca dos produtos. Esse componente interage com o componente `Ofertas` através da interface `IOfertas`, recebendo deste as ofertas relacionadas com a busca feita, em seguida posta no barramento uma mensagem em `items/{userId}` por meio da interface `ResultadoBusca` com os itens resultantes desta busca. 

* O componente `Logistica` recebe uma mensagem `entrega/{userId}/request` da interface `BuscarLogistica`, iniciando o processo de busca dos produtos. Esse componente interage com o componente `LogisticaInsights` através da interface `ILogistica`, recebendo deste as melhores opções de logistica para este cliente, em seguida posta no barramento uma mensagem em `entrega/{userId}/opcoes` por meio da interface `OpcoesLogistica` com os itens resultantes desta busca. Este componente também responde a solicitação do status da logistica recebendo uma mensagem postada no tópico `logistica/{userId}/{pedidoId}/status/request` através da interface `StatusEntregaRequest` e postando o status no tópico `logistica/{userId}/{pedidoId}/status` usando a interface `statusEntrega`. Esse componente possui dois parâmetros de entrada, modos de logistica e entregadores que alimentarão a inteligência responsável por selecionar as melhores opções de entrega e retornar ao solicitante. 

* O componente `Pedido` é responsável por processar o pedido e fazer a devolução do status do mesmo quando solicitado. O pedido é recebido através do tópico `pedido/{userId}/request` e interface de entrada `ReceberPedido` e status devolvido no tópico `pedido/{pedidoId}/{userId}/status/response` através da interface `StatusPedido`, o componente possui outra interface de saída para solicitação do pagamento, essa interface `SolicitaPagamento` posta uma mensagem no tópico `pagamento/{pedidoId}` e recebe como resposta na interface de entrada `RecebeStatusPagamento` os dados publicados no tópico `pagamento/{pedidoId}/status`. Além disso o pedido também recebe uma solicitação de indicação de status no tópico `pedido/{pedidoId}/{userId}/status` com a interface SolicitacaoStatus

* O componente `Pagamento` recebe dois tipos de mensagens de entrada através das interfaces `StatusPagamento` e `StatusPagamento`  e sempre retorna uma mensagem no tópico `pagamento/{pedidoId}/status` usando a interface de saída `StatusPagamento`. As mensagens de entrada são publicadas em `pagamento/{pedidoId}` e `pagamento/{pedidoId}/status` respectivamente.

* O componente `Acompanhamento` realiza o processo de agregação e consolidação dos status de entrega, pagamento e do pedido como um todo. Este componente possui as saídas que solicitam os dados de status para as respectivas estruturas postando as mensagens no tópicos de entrada que respondem o status, citados anteriormente. O componente de acompanhamento possui uma interface de entrada chamada `SolicitacaoStatus` que é responsável por disparar o processo de recuperação dos status, respondendo à uma mensagem publicada no tópico `status/{userId}/{pedidoId}` e publicando a resposta com os status consolidados por meio da interface `Status` no tópico  `status/{userId}/{pedidoId}/status/response`


## Componente `Login`

> Este componente responsável por realizar a autenticação do usuário.


![Componente](images/LoginComponent.png)

**Interfaces**
> ILogin

**Parâmetros**
> Username

> Password

## Componente `BrechoApp`

> Este componente é responsável pelos fluxos principais do Brechó Online.


![Componente](images/BrechoComponent.png)

**Interfaces**
> BuscarItem

> Itens

> Comprar

> BuscarLogistica

> OpcoesLogistica

> AcompanharRequest

> ResultadoBusca


## Detalhamento das Interfaces

### Interface `BuscarItem`

> É responsável por enviar a solicitação de busca.

Dados da interface:
* Type: `source`
* Topic: `buscar/{userId}/request`
* Message type: `FindProductRequest`

Esquema das mensagens JSON:
~~~json
{
    userId: string,
    categoryId: string,
    key: string,
    typeId: number
}
~~~

### Interface `Itens`

> É responsável por receber a lista de itens referentes a busca feita

Dados da interface:
* Type: `sink`
* Topic: `items/{userId}`
* Message type: `FindProductResponse`

Esquema das mensagens JSON:
~~~json
{
    userId: string,
    key: string,
    categoryId: string,
    items: [
        {
            categoryId: string,
            productId: string,
            preco: number,
            quantidade: number,
            descricao: string,
            typeId: string
        }
    ]
}
~~~

### Interface `Comprar`

> É responsável por enviar os dados do pedido.

Dados da interface:
* Type: `source`
* Topic: `pedido/{userId}/request`
* Message type: `CompraRequest`

Esquema das mensagens JSON:
~~~json
{
    userId: string,
    pedido: [
        {
            categoryId: string,
            productId: string,
            preco: number,
            quantidade: number
        }
    ]
}
~~~

### Interface `BuscarLogistica`

> É responsável solicitar as opções de logística indicadas.

Dados da interface:
* Type: `source`
* Topic: `entrega/{userId}/request`
* Message type: `LogisticaResquest`

Esquema das mensagens JSON:
~~~json
{
    userId: string,
}
~~~


### Interface `OpcoesLogistica`

> É responsável por receber a lista de produtos recomendados.

Dados da interface:
* Type: `sink`
* Topic: `entrega/{userId}/opcoes`
* Message type: `LogisticaOpcoesResponse`

Esquema das mensagens JSON:
~~~json
{
    userId: string,
    logistica: [
        {
            logisticaId: string,
            descricao: string,
            price: number
        }
    ]
}
~~~


### Interface `AcompanharRequest`

> É responsável solicitar o consolidade dos status do pedido.

Dados da interface:
* Type: `source`
* Topic: `status/{userId}/{pedidoId}`
* Message type: `StatusRequest`

Esquema das mensagens JSON:
~~~json
{
    userId: string,
    pedidoId: string,
    paymentId: string
}
~~~


### Interface `ResultadoBusca`

> É responsável por receber a lista de produtos recomendados.

Dados da interface:
* Type: `sink`
* Topic: `status/{userId}/{pedidoId}/status/response`
* Message type: `StatusResponse`

Esquema das mensagens JSON:
~~~json
{
    userId: string,
    categoryId: string,
    key: string,
    typeId: number
}
~~~

## Componente `Buscador`

> Este componente é responsável pela busca de itens

![Componente](images/BuscadorComponent.png)

**Interfaces**
> PedidoBusca

> ResultadoBusca

### Interface `PedidoBusca`

> É responsável por receber a mensagem com os dados para realizar a busca.

Dados da interface:
* Type: `sink`
* Topic: `buscar/{userId}/request`
* Message type: `FindProductRequest`

Esquema das mensagens JSON:
~~~json
{
    userId: string,
    categoryId: string,
    key: string,
    typeId: number
}
~~~

### Interface `ResultadoBusca`

> É responsável por retornar os itens buscados.

Dados da interface:
* Type: `source`
* Topic: `items/{userId}`
* Message type: `FindProductResponse`

Esquema das mensagens JSON:
~~~json
{
    userId: string,
    key: string,
    categoryId: string,
    items: [
        {
            categoryId: string,
            productId: string,
            preco: number,
            quantidade: number,
            descricao: string,
            typeId: string
        }
    ]
}
~~~

## Componente `Ofertas`

> Este componente responsável por realizar a busca de ofertas para o usuário.


![Componente](images/OfertasComponent.png)

**Interfaces**
> IOfertas


## Componente `Logística`

> Este componente é responsável por realizar a busca de opções de logistica.

![Componente](images/LogComponent.png)

**Interfaces**
> OpcoesLogistica

> BuscarLogistica

> StatusEntregaRequest

> StatusEntrega

**Parâmetros**
> modosLogistica

> entregadores

### Interface `BuscarLogistica`

> É responsável por receber a solicitação das opções de logística disponíveis.

Dados da interface:
* Type: `sink`
* Topic: `entrega/{userId}/request`
* Message type: `LogisticaResquest`

Esquema das mensagens JSON:
~~~json
{
    userId: string,
}
~~~

### Interface `OpçoesLogistica`

> É responsável por enviar o resultado das opções de logística disponíveis.

Dados da interface:
* Type: `source`
* Topic: `entrega/{userId}/opcoes`
* Message type: `LogisticaOpcoesResponse`

Esquema das mensagens JSON:
~~~json
{
    userId: string,
    logistica: [
        {
            logisticaId: string,
            descricao: string,
            price: number
        }
    ]
}
~~~

### Interface `StatusEntregaRequest`

> É responsável por receber a solicitação do status da entrega.

Dados da interface:
* Type: `sink`
* Topic: `logistica/{userId}/{pedidoId}/status/request`
* Message type: `StatusEntregaRequest`

Esquema das mensagens JSON:
~~~json
{

    userId: string
    pedidoId: string
}
~~~

### Interface `StatusEntrega`

> É responsável por retornar o status da entrega.

Dados da interface:
* Type: `source`
* Topic: `logistica/{userId}/{pedidoId}/status`
* Message type: `StatusEntregaResponse`

Esquema das mensagens JSON:
~~~json
{

    userId: string,
    pedidoId: string,
    status: number
}
~~~

## Componente `Logistica Insights`

> Este componente responsável por realizar a busca inteligente da melhor opção de logística.


![Componente](images/InsightsComponent.png)

**Interfaces**
> ILogistica


## Componente `Pedido`

> Este componente é responsável por registrar a compra/pedido efetuado.


![Componente](images/PedidoComponent.png)

**Interfaces**
> StatusPedido

> RecebePedido

> SolicitaPagamento

> RecebeStatusPagamento

> SolicitacaoStatus

### Interface `StatusPedido`

> É responsável por retornar o status do pedido.

Dados da interface:
* Type: `source`
* Topic: `pedido/{pedidoId}/{userId}/status/response`
* Message type: `StatusPedidoResponse`

Esquema das mensagens JSON:
~~~json
{
    userId: string,
    pedidoId: string,
    status: number
}
~~~

### Interface `SolicitacaoStatus`

> É responsável por receber o pedido de retorno do status dos pedidos.

Dados da interface:
* Type: `sink`
* Topic: `pedido/{pedidoId}/{userId}/status`
* Message type: `StatusPedidoRequest`

Esquema das mensagens JSON:
~~~json
{
    userId: string,
    pedidoId: string,
}
~~~

### Interface `RecebePedido`

> É responsável por receber o pedido de compra.

Dados da interface:
* Type: `sink`
* Topic: `pedido/{userId}/request`
* Message type: `FazerPedidoResquest`

Esquema das mensagens JSON:
~~~json
{
    userId: string,
    pedido: [
        {
            categoryId: string,
            productId: string,
            preco: number,
            quantidade: number
        }
    ]
}
~~~

### Interface `SolicitaPagamento`

> É responsável por enviar a requisição de pagamento.

Dados da interface:
* Type: `source`
* Topic: `pagamento/{pedidoId}`
* Message type: `PayRequest`

Esquema das mensagens JSON:
~~~json
{
    userId: string,
    value: number,
    card: number,
    code: number,
    securityCode: number
}
~~~

### Interface `RecebeStatusPagamento`

> É responsável por receber o status de pagamento.

Dados da interface:
* Type: `sink`
* Topic:`pagamento/{pedidoId}/status`
* Message type: `PayStatusResponse`

Esquema das mensagens JSON:
~~~json
{
    userId: string,
    paymentId: string,
    pedidoId: string
    status: number
}
~~~

## Componente `Pagamento`

> Este componente é responsável pelo pagamento do pedido.


![Componente](images/PayComponent.png)

**Interfaces**
> StatusPagamento

> RecebePedido

> StatusPagamento

### Interface `StatusPagamento`

> É responsável por informar o status do pagamento.

Dados da interface:
* Type: `source`
* Topic: `pagamento/{pedidoId}/status`
* Message type: `PayStatusResponse`

Esquema das mensagens JSON:
~~~json
{
    userId: string,
    paymentId: string,
    pedidoId: string
    status: number
}
~~~

### Interface `RecebePedido`

> É responsável por receber a solicitação de pagamento para o pedido.

Dados da interface:
* Type: `source`
* Topic: `pagamento/{pedidoId}`
* Message type: `PayRequest`

Esquema das mensagens JSON:
~~~json
{
    userId: string,
    value: number,
    card: number,
    code: number,
    securityCode: number
}
~~~

### Interface `StatusPagamento`

> É responsável por receber a solicitação de indicação do status do pagamento.

Dados da interface:
* Type: `sink`
* Topic: `pagamento/{pedidoId}/status`
* Message type: `PayStatusResponse`

Esquema das mensagens JSON:
~~~json
{
    userId: string,
    paymentId: string,
    pedidoId: string
}
~~~

## Componente `Acompanhamento`

> Este componente é responsável por consolidar os status de pedido, pagamento e entrega e retornar ao solicitante.

![Componente](images/TrackingComponent.png)

**Interfaces**
> StatusPagamentoResponse

> StatusEntregaResponse

> StatusPedidoResponse

> SolicitaçãoStatus

> StatusPedidoRequest

> StatusPagamentoRequest

> StatusEntregaRequest

> StatusRequest

### Interface `StatusPagamentoRequest`

> É responsável por receber a solicitação de indicação do status do pagamento.

Dados da interface:
* Type: `source`
* Topic: `pagamento/{pedidoId}/status`
* Message type: `PayStatusResponse`

Esquema das mensagens JSON:
~~~json
{
    userId: string,
    paymentId: string,
    pedidoId: string
}
~~~

### Interface `StatusPagamentoResponse`

> É responsável por receber o status do pagamento.

Dados da interface:
* Type: `sink`
* Topic: `pagamento/{pedidoId}/status`
* Message type: `PayStatusResponse`

Esquema das mensagens JSON:
~~~json
{
    userId: string,
    paymentId: string,
    pedidoId: string
    status: number
}
~~~

### Interface `StatusEntregaResponse`

> É responsável por receber o do status da entrega.

Dados da interface:
* Type: `sink`
* Topic: `logistica/{userId}/{pedidoId}/status/request`
* Message type: `StatusEntregaRequest`

Esquema das mensagens JSON:
~~~json
{
    userId: string,
    pedidoId: string,
    status: number
}
~~~

### Interface `StatusEntregaRequest`

> É responsável por solicitar o status da entrega.

Dados da interface:
* Type: `source`
* Topic: `logistica/{userId}/{pedidoId}/status`
* Message type: `StatusEntregaResponse`

Esquema das mensagens JSON:
~~~json
{
    userId: string,
    pedidoId: string
}
~~~

### Interface `StatusPedidoRequest`

> É responsável por solicitar o status do pedido.

Dados da interface:
* Type: `source`
* Topic: `pedido/{userId}/status`
* Message type: `StatusPedidoRequest`

Esquema das mensagens JSON:
~~~json
{
    userId: string,
    pedidoId: string,
}
~~~

### Interface `StatusPedidoResponse`

> É responsável por receber o status do pedido.

Dados da interface:
* Type: `sink`
* Topic: `pedido/{pedidoId}/{userId}/status/response`
* Message type: `StatusPedidoResponse`

Esquema das mensagens JSON:
~~~json
{
    userId: string,
    pedidoId: string,
    status: number
}
~~~

### Interface `SolicitaçãoStatus`

> É responsável por receber a solicitação de status do pedido.

Dados da interface:
* Type: `sink`
* Topic: `status/{userId}/{pedidoId}`
* Message type: `StatusRequest`

Esquema das mensagens JSON:
~~~json
{
    userId: string,
    pedidoId: string,
    paymentId: string
}
~~~

### Interface `Status`

> É indicar o status do pedido.

Dados da interface:
* Type: `source`
* Topic: `status/{userId}/{pedidoId}/status/response`
* Message type: `StatusResponse`

Esquema das mensagens JSON:
~~~json
{
    userId: string,
    pedidoId: string,
    paymentId: string
    pedidoStatus: number,
    pagamentoStatus: number
    entregaStatus: number
}
~~~

# Nível 2
## Diagramas do Nível 2 `LogisticaComponent`

> ![Modelo de diagrama no nível 2](images/LogisticaComponenteDetails.png)

### Detalhamento da interação de componentes

* O componente `PedidoLogistica` assina no barramento mensagens de tópico "`entrega/{userId}/request`" através da interface `BuscarLogistica`.
  * Ao receber uma mensagem de tópico "`entrega/{userId}/request`", dispara o início do processo de busca de transportadoras para um conjunto de produtos.
  * Para que o processo de precificação e datas de entrega sejam precisos, são obtidas informações com os componentes internos `VendedorLogistica` e `ClienteLogistica`.
  * Apos obter informações do cliente e vendedor, as informações são transmitidas para o componente `LogisticaInsightsConnector`, que por sua vez se conecta com um compoente externo, responsável pela busca de transportadoras, em um sistema regionalizado que faz uso de IA.
  * O retorno do componente `LogisticaInsightsConnector` faz uma assinatura no barramento com o tópico "`entrega/{userId}/opcoes`" e interface `OpçoesLogistica`

* O componente `TransportadoraLogistica` assina no barramento mensagens de tópico "`logistica/{userId}/{pedidoId}/status/request`" através da interface `StatusEntregaRequest`.
  * Ao receber uma mensagem de tópico "`logistica/{userId}/{pedidoId}/status/request`", realiza o processo de busca do pedido para obtenção do status, junto a transportadora responsável.
  * O proprio componente também é responsável por fazer uma assinatura no barramento com o tópico "`logistica/{userId}/{pedidoId}/status`" e interface `StatusEntrega`, para retorno da solicitação.

* O componente `LogisticaInsightsConnector` ainda é responsável por coletar os dados de `VendedorLogistica`, `ClienteLogistica` e  `TransportadoraLogistica` para alimentar o processo de inteligência artificial de escolha de transportadoras, levando em conta a região, prazo, últimos pedidos entregues, devolução e taxa de sucesso.


## Componente `PedidoLogistica`
Este componente assina o barramento para recepção do pedido, orquestra a busca de informações do pedido em `VendedorLogistica` e `ClienteLogistica` e realiza uma chamada para o componente `LogisticaInsightsConnector`.

**Interfaces**
> IBuscaLogistica

## Detalhamento das Interfaces
### Interface `IBuscaLogistica`
Interface para busca de um parceiro logístico do sistema de brechó online.

Método | Objetivo
-------| --------
`setUserId` | Define o userId que pedido está aberto.
`getUserId` | Retorna o userId que pedido está aberto.



## Componente `VendedorLogistica`
Este componente fornece informações do vendedor para os componentes `PedidoLogistica` e `LogisticaInsightsConnector`.

**Interfaces**
> IVendedorLogistica

> IVendedorLogisticaInsights

## Detalhamento das Interfaces
### Interface `IVendedorLogistica`
Interface para busca de informações de um vendedor no componente de logística.

Método | Objetivo
-------| --------
`setVendedorId` | Define o vendedorId.
`getVendedorId` | Retorna o vendedorId.

### Interface `IVendedorLogisticaInsights`
Interface para busca de informações de vendedores no componente de logística, para treinamento da IA.

Método | Objetivo
-------| --------
`setDataInicio` | Define a data de início que as informações de vendedor serão selecionada para o treinamento.
`getDataInicio` | Retorna a data de início que as informações de vendedor serão selecionada para o treinamento.
`setCompleto` | Define se será pesquisado dados para um treinamento completo.
`isComplete` | Retorna se será pesquisado dados para um treinamento completo.


## Componente `ClienteLogistica`
Este componente fornece informações do cliente para os componentes `PedidoLogistica` e `LogisticaInsightsConnector`.

**Interfaces**
> IClienteLogistica

> IClienteLogisticaInsights

## Detalhamento das Interfaces
### Interface `IClienteLogistica`
Interface para busca de informações de um cliente no componente de logística.

Método | Objetivo
-------| --------
`setClienteId` | Define o clienteId.
`getClienteId` | Retorna o clienteId.

### Interface `IClienteLogisticaInsights`
Interface para busca de informações de clientes no componente de logística, para treinamento da IA.

Método | Objetivo
-------| --------
`setDataInicio` | Define a data início que as informações de cliente serão selecionada para o treinamento.
`getDataInicio` | Retorna a data início que as informações de cliente serão selecionada para o treinamento.
`setCompleto` | Define se será pesquisado dados para um treinamento completo.
`isComplete` | Retorna se será pesquisado dados para um treinamento completo.



## Componente `TransportadoraLogistica`
Este componente assina o barramento para requisição de status de entrega e fornece a assinatura com o respectivo status. Ele também é responsável por fornecer informações de transportes para o componente `LogisticaInsightsConnector`.

**Interfaces**
> IStatusEntregaRequest

> IStatusEntrega

> ITransportadoraLogisticaInsights

## Detalhamento das Interfaces
### Interface `IStatusEntregaRequest`
Interface para busca de informações de status de entrega de um pedido.

Método | Objetivo
-------| --------
`setUserId` | Define o userId.
`getUserId` | Retorna o userId.
`setPedidoId` | Define o pedidoId.
`getPedidoId` | Retorna o pedidoId.

### Interface `IStatusEntrega`
Interface para retorno de informações de status de entrega de um pedido.

Método | Objetivo
-------| --------
`setUserId` | Define o userId.
`getUserId` | Retorna o userId.
`setPedidoId` | Define o pedidoId.
`getPedidoId` | Retorna o pedidoId.
`setStatus` | Define o status.
`getStatus` | Retorna o status.


### Interface `ITransportadoraLogisticaInsights`
Interface para busca de informações de transportadoras no componente de logística, para treinamento da IA.

Método | Objetivo
-------| --------
`setDataInicio` | Define a data início que as informações de cliente serão selecionada para o treinamento.
`getDataInicio` | Retorna a data início que as informações de cliente serão selecionada para o treinamento.
`setCompleto` | Define se será pesquisado dados para um treinamento completo.
`isComplete` | Retorna se será pesquisado dados para um treinamento completo.



## Componente `LogisticaInsightsConnector`
Este componente se comunica com outro componente externo, para realização da busca de parceiros logísticos e fornece assinatura no barramento para opções de parceiros logísticos para um pedido. O componente ainda é responsável por coletar os dados de `VendedorLogistica`, `ClienteLogistica` e  `TransportadoraLogistica` para alimentar o processo de inteligência artificial de escolha de transportadoras, levando em conta a região, prazo, últimos pedidos entregues, devolução e taxa de sucesso.

**Interfaces**
> IOpcoesLogistica

> IBuscarLogisticaInsights

> ILogistica

> ILogisticaTraining

## Detalhamento das Interfaces
### Interface `IOpcoesLogistica`
Interface para retorno da busca de opções de entrega de um pedido.

Método | Objetivo
-------| --------
`setUserId` | Define o userId.
`getUserId` | Retorna o userId.
`setLogisticas` | Define uma coleção de parceiros logísticos.
`getLogisticas` | Retorna uma coleção de parceiros logísticos.

### Interface `IBuscarLogisticaInsights`
Interface para  busca de opções de entrega de um pedido.

Método | Objetivo
-------| --------
`setVendedorDto` | Define o vendedor.
`getVendedorDto` | Retorna o vendedor.
`setClienteDto` | Define o cliente.
`getClienteDto` | Retorna o cliente.
`setPedidoDto` | Define o pedido.
`getPedidoDto` | Retorna o pedido.


### Interface `ILogistica`
Interface para comunicação com o componente de IA responsável pela busca de opções de parceiros logísticos.

Método | Objetivo
-------| --------
`setVendedorDto` | Define o vendedor.
`getVendedorDto` | Retorna o vendedor.
`setClienteDto` | Define o cliente.
`getClienteDto` | Retorna o cliente.
`setPedidoDto` | Define o pedido.
`getPedidoDto` | Retorna o pedido.

### Interface `ILogisticaTraining`
Interface para comunicação com o componente de IA responsável pelo treinamento de opções de parceiros logísticos.

Método | Objetivo
-------| --------
`setVendedoresDto` | Define uma lista de vendedores.
`getVendedoresDto` | Retorna uma lista de vendedores.
`setClientesDto` | Define uma lista de clientes.
`getClientesDto` | Retorna uma lista de clientes.
`setPedidosDto` | Define uma lista de pedidos.
`getPedidosDto` | Retorna uma lista de pedidos.



## Diagramas do Nível 2 `PedidoComponent`

> ![Modelo de diagrama no nível 2](images/PedidoComponenteDetails.png)

### Detalhamento da interação de componentes

* O componente `InformacaoPedido` assina no barramento mensagens de tópico "`pedido/{userId}/request`" através da interface `RecebePedido`.
  * Ao receber uma mensagem de tópico "`pedido/{userId}/request`", realiza a busca das informações necessárias e inicia do processo de pagamento, passando as informações para o componente `PagamentoPedido` atraves da interface `IPagamentoPedido` .
  * O componente `PagamentoPedido` reserva os produtos solicitados atraves do componente `ReservaProduto`, utilizando a interface `IReservaProduto`
  * Para que o processo de pagamento seja concluído o componente `PagamentoPedido` disponibiliza ao barramento o tópico "`pagamento/{pedidoId}`" e aguarda a notificação de pagamento assinando o tópico "`pagamento/{pedidoId}/status`".

* O componente `InformacaoPedido` ainda é responsável por assinar no barramento mensagens de tópico "`pedido/{pedidoId}/{userId}/status`" através da interface `ISolicitacaoStatus`.
  * Ao receber uma mensagem de tópico "`pedido/{pedidoId}/{userId}/status`", realiza o processo de busca do pedido para obtenção do status.
  * O proprio componente também é responsável por disponibilizar uma assinatura no barramento com o tópico "`pedido/{pedidoId}/{userId}/status/response`" e interface `IStatusPedido`, para retorno da solicitação.



## Componente `InformacaoPedido`
Este componente assina o barramento para recebimento de pedido. Ele tambèm é responsável pela requisição de status do pedido e fornece a assinatura com o respectivo status.

**Interfaces**
> IRecebePedido

> ISolicitacaoStatus

> IStatusPedido

## Detalhamento das Interfaces
### Interface `IRecebePedido`
Interface para solicitação de um pedido.

Método | Objetivo
-------| --------
`setUserId` | Define o userId.
`getUserId` | Retorna o userId.
`setPedidos` | Define uma lista de produtos que compõem um pedido.
`getPedidos` | Retorna uma lista de produtos que compõem um pedido.

### Interface `ISolicitacaoStatus`
Interface para solicitação de status de um pedido.

Método | Objetivo
-------| --------
`setUserId` | Define o userId.
`getUserId` | Retorna o userId.
`setPedidoId` | Define o pedidoId.
`getPedidoId` | Retorna o pedidoId.


### Interface `IStatusPedido`
Interface para retornar o status de um pedido.

Método | Objetivo
-------| --------
`setUserId` | Define o userId.
`getUserId` | Retorna o userId.
`setPedidoId` | Define o pedidoId.
`getPedidoId` | Retorna o pedidoId.
`setStatus` | Define o status.
`getStatus` | Retorna o status.



## Componente `PagamentoPedido`
Este componente recebe um pedido e solicita ao barramento via assinatura um pagamento. Ele também recebe através do barramento o status de pagamento de um pedido.

**Interfaces**
> ISolicitaPagamento

> IStatusPagamento

> IPagamentoPedido

## Detalhamento das Interfaces
### Interface `ISolicitaPagamento`
Interface para realização de pagamento de um pedido.

Método | Objetivo
-------| --------
`setUserId` | Define o userId.
`getUserId` | Retorna o userId.
`setValue` | Define o valor a ser cobrado.
`getValue` | Retorna o valor a ser cobrado.
`setCard` | Define o cartão de crédito a ser usado.
`getCard` | Retorna o cartão de crédito a ser usado.
`setCode` | Define o número do cartão.
`getCode` | Retorna o número do cartão.
`setSecurityCode` | Define o código de segurança do cartão.
`getSecurityCode` | Retorna o código de segurança do cartão.

### Interface `IStatusPagamento`
Interface para retorno do status de pagamento de um pedido.

Método | Objetivo
-------| --------
`setUserId` | Define o userId.
`getUserId` | Retorna o userId.
`setPaymentId` | Retorna o paymentId.
`getPaymentId` | Retorna o paymentId.
`setPedidoId` | Define o pedidoId.
`getPedidoId` | Retorna o pedidoId.
`setStatus` | Define o status.
`getStatus` | Retorna o status.

### Interface `IPagamentoPedido`
Interface para solicitar o pagamento de um pedido.

Método | Objetivo
-------| --------
`setUserId` | Define o userId.
`getUserId` | Retorna o userId.
`setPedidoId` | Define o pedidoId.
`getPedidoId` | Retorna o pedidoId.



## Componente `ReservaProduto`
Este componente é responsável por validar os produtos de um pedido e reservá-los para realização do pagamento.

**Interfaces**
> IReservaProduto

### Interface `IReservaProduto`
Interface para reserva de produto durante a realização da operação de pagamento.

Método | Objetivo
-------| --------
`setPedidoId` | Define o ID do pedido.
`getPedidoId` | Retorna o ID do pedido.



## Componente `NotificaStatus`
Este componente é responsável por gerenciar as notificações do componente de pedido.

**Interfaces**
> INotificaStatus

### Interface `INotificaStatus`
Interface para realização de notificações relacionadas a um pedido.

Método | Objetivo
-------| --------
`setPedidoId` | Define o ID do pedido.
`getPedidoId` | Retorna o ID do pedido.



## Componente `NotificaVendedor`
Este componente é responsável por notificar o vendedor para as movimentações do pedido.

**Interfaces**
> INotificaVendedor

### Interface `INotificaVendedor`
Interface para realização de notificações relacionadas a um vendedor.

Método | Objetivo
-------| --------
`setVendedorId` | Define o ID do vendedor.
`getVendedorId` | Retorna o ID do vendedor.
`setPedidoId` | Define o ID do pedido.
`getPedidoId` | Retorna o ID do pedido.



## Componente `NotificaComprador`
Este componente é responsável por notificar o comprador para movimentações do pedido.

**Interfaces**
> INotificaComprador

### Interface `INotificaComprador`
Interface para realização de notificações relacionadas a um comprador.

Método | Objetivo
-------| --------
`setClienteId` | Define o ID do comprador.
`getClienteId` | Retorna a ID do comprador.
`setPedidoId` | Define o ID do pedido.
`getPedidoId` | Retorna o ID do pedido.



## Diagramas do Nível 2 MVC `PedidoComponent`
> ![Modelo de diagrama no nível 2](images/PedidoComponenteDetailsMVC.png)


# Nível 3
> ![View](images/appview.PNG)
> ![Componentes](images/Componentes.PNG)
> ![Diagrama](images/Diagrama.PNG)



  O `Controle de Interface` recebe as informações do produto,como Descrição,Preço e Quantidade, atravez da interface Consulta Produto Info que se comunica com os componentes Externos no barramento com o topico "`items/userId}`".

  Tendo selecionado Quantidade, Forma de Pagamento e clicado em `Comprar Agora!` será enviado as informações da compra para `BrechoApp`que atravez da interface `Compra` postara os dados do pedido no barramento assim como ilustrado anteriormente usando o topico "`pedido/{userId}/request`".
 
  Caso receba uma mensagem com o topico "`buscar/{userId}/request`" dispara a busca por determinado item.

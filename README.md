# Projeto de componentização: Marketplace
## Equipe 1
> Bernardo Fonseca Andrade de Carvalho <br>
> Hélio De Rosa Junior <br>
> José Octavio Vitoriano Martines Penna <br>
> Pablo Gabriel Rodrigues Neves Bedoya <br>
> Rodrigo Leonel Sartorato <br>
# Nível 1

## Diagramas Gerais do Nível 1

## Diagrama do Processo de Compra
![DiagramaNivel1Compra](images/n2-components/DiagramaNivel1Compra.png)
## Diagrama do Processo de Distribuição de Ofertas
![DiagramaNivel1Ofertas](images/n2-components/DiagramaNivel1Ofertas.png)

> O diagrama escolhido para análise foi o **Diagrama do Processo de Distribuição de Ofertas**.<br>

# Nível 2

## Diagrama do Nível 2
![DiagramaNivel2](images/n2-components/DiagramaNivel2.png)

## Detalhamento de interação de componentes
* O componente `OfferDistribution` assina o barramento de mensagens de tópico `products/mostwanted` através da interface `ReceiveProcessedRequests`
	* Através da mensagem de tópico `products/mostwanted`, que chega na interface requerida `IRequest` do componente interno `ExtractProductList`, o componente `OfferDistribution` recebe uma lista que contém o array produtos mais desejados pelos clientes do Marketplace, juntamente com um array de clientes que marcaram como desejados pelo menos ⅓ dos produtos presentes na lista. Tal lista é filtrada para a última quinzena
* Então, tal evento é atendido por uma interface provida do componente `ExtractProductList`.
* O componente `ExtractProductList` separa os arrays de produtos e clientes, os fornece em duas interfaces providas `IRequestProductList` e `IRequestCustomerList`
	* A lista de produtos fornecida pela interface `IRequestProductList` é enviada para o componente `RankOffersBasedOnReference`, e neste componente, é usada como lista de referência para o ranqueamento de ofertas de lojas para operações em IA. 
	* A lista de clientes fornecida pela interface `IRequestCustomerList` é enviada para o componente `AssembleOnSales` que montará mensagens do tipo `OnSale`, que serão enviadas para o barramento com o tópico `campaign/{campaignId}/provideoffers`.
* O componente `OfferDistribution` também assina o barramento de mensagens de tópico `campaign/+/makeoffer/+` através da interface `ReceiveOffer`
	* Através da mensagem de tópico `campaign/+/makeoffer/+`, que chega na interface requerida `IOffer` do componente interno `ProcessStoreOffer`, o componente `OfferDistribution` recebe cada uma das ofertas, no tipo de mensagem `Offer`, feitas pelas lojas que escrevem no barramento através do tópico `campaign/{campaignId}/makeoffer/{offerId}`.
* Esse evento faz com o componente `ProcessStoreOffer`, monte/agrupe um array produtos em oferta, monte/agrupe um array que contém dados das lojas ofertantes, e separe os dados da campanha, e os forneça respectivamente nas interfaces: `IOfferProductList`, `IOfferStoreDataList` e `IOfferCapaignDataList`.
	* O array de produtos em oferta é enviado para o componente `RankOffersBasedOnReference`, através da interface provida `IOfferProductList` para que alimente as operações de IA mencionadas anteriormente.
* O array de informação das lojas ofertantes e os dados da campanha são enviados para o componente `AssembleOnSales`, para que sejam incorporados nas mensagens do tipo `OnSales` como dito anteriormente
* A lista de produtos separados/rankeados pelo componente `RankOfferBasedOnReference` também é enviado para o componente `AssembleOnSales`, para que também seja acrescentado à mensagem do tipo `OnSale`
* Por fim, o componente `AssembleOnSales`, finaliza a montagem da mensagem do tipo `OnSale` e a escreve no barramento através do tópico `campaign/+/provideoffers`, na interface `ProvideRankedOffers`.

## Detalhamento dos Componentes:

**Componente ExtractProductsList**<br>
> Papel: é responsável por receber a lista de produtos pré-processada pelo componente `ProcessCustomerRequests` e separar tal lista em um array de produtos pré-selecionados e em um array de clientes que assinalaram interesse em pelo menos ⅓ da lista de produtos previamente mencionada. 

![ExtractProductsList](images/n2-components/ExtractProductsList.png)

## Interfaces:
> IRequest<br>
> IRequestCustomerList<br>
> IRequestProductList<br>

## Detalhamento das Interfaces

**Inteface IRequest**<br>
![IRequest](images/n2-components/IRequest.png)
> Papel: Interface requerida que pede por dados de produtos e clientes na forma de lista.

Método | Objetivo
-------| --------
`getProcessedRequests` | `pede por uma lista de dados pré-processados`.

**Inteface IRequestProductList**<br>
![IRequestProductList](images/n2-components/IRequestProductList.png)
> Papel: Interface provida que fornece um array de produtos, para que seja usado como array de referência.

Método | Objetivo
-------| --------
`getProductList` | `fornece um array de produtos`.


**Inteface IRequestCustomerList**<br>
![IRequestCustomerList](images/n2-components/IRequestCustomerList.png)
> Papel: Interface provida que fornece um array de customeId, para que seja usado na montagem de mensagens de campanhas de desconto. 

Método | Objetivo
-------| --------
`getCustomerList` | `fornece um array de produtos`.


**Componente ProcessStoreOffer**<br>
> Papel: é responsável por processar cada uma das ofertas feitas por lojas, e separar e agrupar/destacar os produtos, dados da loja e dados da campanha contidos nas ofertas.

![ProcessStoreOffer](images/n2-components/ProcessStoreOffer.png)

## Interfaces:
> IOffer<br>
> IOfferProductList<br>
> IOfferStoreDataList<br>
> IOfferCampaignDataList<br>

## Detalhamento das Interfaces
**Inteface IOffer**<br>
![IOffer](images/n2-components/IOffer.png)
> Papel: Interface requerida que pede por ofertas de cada loja.

Método | Objetivo
-------| --------
`requestOffer` | `pede por uma oferta de um lojista`.

**Inteface IOfferProductList**<br>
![IOfferProductList](images/n2-components/IOfferProductList.png)
> Papel: Interface provida que fornece a lista de produtos ofertados por `n` lojas.

Método | Objetivo
-------| --------
`getOfferProductList` | `fornece uma lista de produtos ofertados pelas lojas`.

**Inteface IOfferStoreDataList**<br>
![IOfferStoreDataList](images/n2-components/IOfferStoreDataList.png)
> Papel: Interface provida que fornece a lista de dados referente às lojas.

Método | Objetivo
-------| --------
`getOfferStoreIdList` | `fornece uma lista que contém strings compostas por id da loja storeId, número de unidades inStock do produto e valor com desconto ofertado salesPrice`

**Inteface IOfferCampaignDataList**<br>
![IOfferCampaignDataList](images/n2-components/IOfferCampaignDataList.png)
> Papel: Interface provida que fornece a data final da campanha.

Método | Objetivo
-------| --------
`getOfferCampaignEndDate` | `fornece a data de término endDate da campanha`.

**Componente RankOffersBasedOnReference**<br>
> Papel: é responsável fornecer uma lista de ofertas ranqueadas, de acordo com operações de IA .

![ProcessStoreOffer](images/n2-components/RankOffersBasedOnReference.png)

## Interfaces:
> IRankedProducts<br>

## Detalhamento das Interfaces

**Inteface IRankedProducts**<br>
![IRankedProducts](images/n2-components/IRankedProducts.png)
> Papel: Interface provida que fornece a lista final de produtos rankeados por IA.

Método | Objetivo
-------| --------
`getRankedProductsList` | `fornece uma lista de produtos ranqueados por operações de IA (e.g. opereções com base em Support-vector Machine ou Logistic Regression)`.

**Componente AssembleOnSales**<br>
> Papel: é responsável fornecer a mensagens de campanha de ofertas que será recebida por clientes.

![ProcessStoreOffer](images/n2-components/AssembleOnSales.png)

## Interfaces:
> ProvideRankedOffers<br>

## Detalhamento das Interfaces

**Inteface ProvideRankedOffers**<br>
![ProvideRankedOffers](images/n2-components/ProvideRankedOffers.png)
> Papel: Interface provida que fornece a mensagem final que será enviada aos clientes.

Método | Objetivo
-------| --------
`getRankedOffers` | `fornece a mensagem final que será escrita no barramento para que os clientes as possam consumir`.


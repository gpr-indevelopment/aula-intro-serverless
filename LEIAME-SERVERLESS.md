# Serverless

Serverless é um paradigma de computação em nuvem que permite a execução de código para aplicações ou serviços de backend sem provisionar ou gerenciar servidores. Esse paradigma cria uma camada de abstração de infraestrutura para que desenvolvedores possam se preocupar apenas com o código das aplicações e não com aspectos de plataforma como escalabilidade automática, balanceamento de carga, tolerância a falhas, suporte a observabilidade, entre outros. Serviços disponibilizados em plataformas serverless são comumente chamados de funções dentro do contexto de FaaS (Function as a Service). Amazon Web Services (AWS), Google Cloud Platform (GCP) e Microsoft Azure são as principais provedoras de nuvem do mercado e oferecem plataformas serverless como produto. Esse capítulo aborda os seguintes aspectos de serverless:

* Ciclo de vida de uma função
* Configurando uma função serverless
* "Cold" e "Warm" starts
* Modelo de cobrança de serviço
* Principais desafios 

## Ciclo de vida de funções Serverless

Por natureza, funções serverless são efêmeras e sem estado. Efêmeras pois são instanciadas mediante um gatilho para depois serem desprovisionadas ao final do seu processamento. As plataformas serverless podem reaproveitar o ambiente de execução de uma função para gatilhos subsequentes, no entanto, a frequência e tempo de reaproveitamento não é configurável pelos desenvolvedores das funções. Por conta disso, não há garantias que os dados de estado armazenados na memória da função durante uma execução ainda sejam acessíveis em execuções subsequentes.

## Modelo de cobrança de serviço

Conforme o marketing das provedoras, uma das principais vantagens do paradigma Serverless é o modelo de cobrança baseado na utilização do serviço:

* **AWS Lambda**: “Economize custos pagando apenas pelo tempo de computação usado (por milissegundo) em vez de provisionar a infraestrutura antecipadamente para obter capacidade máxima.”​
* **Google Cloud Functions**: “Só será cobrado o período de execução da função, que é medido até o valor mais próximo de 100 milissegundos. Você não paga nada quando a função está ociosa. O Cloud Functions adapta automaticamente seus recursos em função dos eventos.”
* **Microsoft Azure Functions**: “Pague pela capacidade de computação por segundo, sem compromisso de longo prazo ou pagamentos adiantados. Aumente ou diminua o consumo sob demanda.”

As plataformas serverless​ seguem o modelo "pay-per-use" ou "pay-as-you-go" de forma que o tempo ocioso dos servidores que hospedam as aplicações não é incluído no custo. Ao invés disso, os usuários da plataforma serverless são cobrados apenas pelo tempo útil dos serviços, ou seja, o tempo em que esses serviços estão ativamente processando requisições. Além disso, o preço do tempo de execução escala proporcionalmente à alocação de recursos de memória e CPU das funções, de forma que o milissegundo de execução é mais caro para funções com mais recursos.

Em geral, as plataformas serverless também incluem um custo fixo por disparo de função serverless, além de cobranças eventuais decorrentes de integrações com outros serviços de nuvem como armazenamento, bancos de dados e filas.
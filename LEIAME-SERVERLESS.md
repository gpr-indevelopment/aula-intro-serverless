# Serverless

Serverless é um paradigma de computação em nuvem que permite a execução de código para aplicações ou serviços de backend sem provisionar ou gerenciar servidores. Esse paradigma cria uma camada de abstração de infraestrutura para que desenvolvedores possam se preocupar apenas com o código das aplicações e não com aspectos de plataforma como escalabilidade automática, balanceamento de carga, tolerância a falhas, suporte a observabilidade, entre outros. Serviços disponibilizados em plataformas serverless são comumente chamados de funções dentro do contexto de FaaS (Function as a Service). Amazon Web Services (AWS), Google Cloud Platform (GCP) e Microsoft Azure são as principais provedoras de nuvem do mercado e oferecem plataformas serverless como serviços. Esse capítulo aborda os seguintes aspectos de serverless:

* Configuração básica de uma função Lambda
* Ciclo de vida de uma função
* "Cold" e "Warm" starts
* Modelo de cobrança de serviço
* Principais desafios 

## Configuração básica de uma função Lambda

O AWS Lambda é atualmente a plataforma serverless de nuvem mais relevante do mercado e será usada como exemplo. Funções Lambda requerem uma conta válida da AWS e podem ser criadas pelo console web ou pelo terminal por meio do AWS Serverless Application Model (SAM). O console web da AWS é acessível por um navegador e é a alternativa mais simples para demonstração:

1. Acesse o [console da AWS](https://aws.amazon.com/pt/console/)
2. Faça o login usando suas credenciais. Crie uma nova conta caso ainda não tenha credenciais. A criação de uma nova conta exige um cartão de crédito, no entanto, diversas funcionalidades estão disponíveis dentro do [nível gratuito](https://aws.amazon.com/pt/free/) da AWS e só serão cobradas se utilizadas fora dos limites de gratuidade. Os passos documentados nessa demonstração não implicam em custos adicionais para uma conta nova na AWS que ainda não utilizou funções Lambda.
3. Visite a página do serviço Lambda. Você pode usar a barra de busca do console AWS para encontrar esse serviço caso ele não seja mostrado na página inicial.
4. Clique em "Create a function".
5. Escolha um nome para a função e o runtime desejado. O runtime NodeJS 18 é recomendado pois dispõe de um editor de código Javascript pela interface web do console AWS.
6. Depois, confirme a criação da função pelo botão "Create function", no canto inferior direito.

![Criando função Lambda](/assets/create-lambda-function.gif)

A função criada retorna um JSON com o texto `Hello from Lambda` conforme podemos observar no editor de código Javascript. A seguir, podemos configurar um gatilho para a execução da função. Gatilhos podem ser representados por requisições HTTP, mensagens em uma fila, eventos de serviço, entre outros. Nessa demonstração vamos criar um gatilho de requisição HTTP:

1. Clique na aba "Configuration"
2. Na direita, clique no menu "Function URL".
3. Depois, clique em "Create function URL" já que ainda não temos uma URL para a função.
4. Por simplicidade, clique em "NONE" como método de autenticação. Inicialmente essa função retorna apenas um texto sem informações sensíveis, e torná-la pública não apresenta nenhum risco de segurança.
5. Clique em "Save". A URL da sua função será então apresentada na tela. Abrir a URL da função no navegador ou fazer uma requisição GET para o mesmo endereço retorna `Hello from Lambda`. O corpo da requisição HTTP é recebido pelo parâmetro `event` da função.

![Disparando uma função Lambda](/assets/fire-lambda-function.gif)

Qualquer alteração no código da função deve ser publicada para que seja refletida nas chamadas HTTP para a URL configurada. Para publicar uma alteração de código:

1. Acesse a aba "Code" do ambiente Lambda.
2. Faça uma alteração de código.
3. Salve a alteração (Ctrl + S).
4. Clique no botão "Deploy".
5. Acesse a URL novamente.

![Atualizando código Lambda](/assets/lambda-update-function-code.gif)

A alocação de recursos de memória e CPU para as funções deve ser configurada no ambiente Lambda. O menu de configurações também permite a configuração de um armazenamento efêmero de arquivos temporários e a alteração do tempo limite de execução de funções antes de um erro de "timeout":

1. Acesse o menu "Configuration".
2. Clique em "General configuration".
3. Configure a alocação desejada de memória para a função. A alocação de CPU deriva da memória, de forma que cada 1792 MB de memória corresponde a 1 vCPU.
4. Configure o tamanho desejado do armazenamento efêmero.
5. Configure o tempo limite desejado.
6. Clique em "Save".

![Atualizando configuração Lambda](/assets/update-function-settings.gif)

Funções Lambda podem importar qualquer biblioteca disponível no gerenciador de pacotes do runtime escolhido (NPM no caso de NodeJS 18). Além disso, elas podem também integrar com bancos de dados e serviços de armazenamento além de publicar mensagens para filas e outras funções.
## Ciclo de vida de funções Serverless

Por natureza, funções serverless são efêmeras e sem estado. Efêmeras pois são instanciadas mediante um gatilho, processam uma requisição, e por fim são desprovisionadas. As plataformas serverless podem reaproveitar o ambiente de execução de uma função para gatilhos subsequentes, no entanto, a frequência e tempo de reaproveitamento não é configurável pelos desenvolvedores das funções. Por conta disso, não há garantias que dados de estado armazenados em memória durante uma execução ainda estejam acessíveis em execuções subsequentes. A imagem abaixo descreve o ciclo de vida de uma função AWS Lambda:

![Ciclo de vida Lambda](/assets/lambda-lifecycle.png)

O ambiente de execução da função é provisionado em um container durante a etapa "INIT". Adiante, a etapa "INVOKE" acontece para cada vez que a função foi executada por um gatilho. Uma função terá pelo menos uma etapa "INVOKE" antes de ser desligada. A etapa de "SHUTDOWN" acontece quando a função e seus recursos são desprovisionados pela plataforma.

Funções em plataformas serverless na nuvem dispõe de escalabilidade horizontal automática. Novas funções são instanciadas para acomodar uma nova demanda de serviço sem necessidade de intervenção manual de um administrador ou desenvolvedor. No entanto, a escalabilidade vertical não é automática. Ou seja, funções são instanciadas com a alocação de recursos de memória e CPU pré-definidos na configuração. A imagem abaixo apresenta uma linha do tempo em que um total de 6 requisições concorrentes chegaram em um serviço, que por sua vez precisou de 10 instâncias de função Lambda executando em paralelo para acomodar a demanda.

![Concorrência Lambda](/assets/lambda-concurrency.gif)

## Modelo de cobrança de serviço

Conforme o marketing das provedoras, uma das principais vantagens do paradigma Serverless é o modelo de cobrança baseado na utilização do serviço:

* **AWS Lambda**: “Economize custos pagando apenas pelo tempo de computação usado (por milissegundo) em vez de provisionar a infraestrutura antecipadamente para obter capacidade máxima.”​
* **Google Cloud Functions**: “Só será cobrado o período de execução da função, que é medido até o valor mais próximo de 100 milissegundos. Você não paga nada quando a função está ociosa. O Cloud Functions adapta automaticamente seus recursos em função dos eventos.”
* **Microsoft Azure Functions**: “Pague pela capacidade de computação por segundo, sem compromisso de longo prazo ou pagamentos adiantados. Aumente ou diminua o consumo sob demanda.”

As plataformas serverless​ seguem o modelo "pay-per-use" ou "pay-as-you-go" de forma que o tempo ocioso dos servidores que hospedam as aplicações não é incluído no custo. Ao invés disso, os usuários da plataforma serverless são cobrados apenas pelo tempo útil dos serviços, ou seja, o tempo em que esses serviços estão ativamente processando requisições. Além disso, o preço do tempo de execução escala proporcionalmente à alocação de recursos de memória e CPU das funções, de forma que o milissegundo de execução é mais caro para funções com mais recursos.

Em geral, as plataformas serverless também incluem um custo fixo por disparo de função serverless, além de cobranças eventuais decorrentes de integrações com outros serviços de nuvem como armazenamento, bancos de dados e filas.
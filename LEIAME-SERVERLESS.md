# Serverless

Serverless é um paradigma de computação em nuvem que permite a execução de código para aplicações ou serviços de backend sem provisionar ou gerenciar servidores. Esse paradigma cria uma camada de abstração de infraestrutura para que desenvolvedores possam se preocupar apenas com o código das aplicações e não com aspectos de plataforma como escalabilidade automática, balanceamento de carga, tolerância a falhas, suporte a observabilidade, entre outros. Serviços disponibilizados em plataformas serverless são comumente chamados de funções dentro do contexto de FaaS (Function as a Service). Amazon Web Services (AWS), Google Cloud Platform (GCP) e Microsoft Azure são as principais provedoras de nuvem do mercado e oferecem plataformas serverless como serviços. Esse documento aborda os seguintes aspectos de serverless:

* [Configuração básica de uma função Lambda](#configuração-básica-de-uma-função-lambda)
* [Ciclo de vida de funções Serverless](#ciclo-de-vida-de-funções-serverless)
* [Requisições "Frias" e "Quentes"](#requisições-frias-e-quentes)
* [Modelo de cobrança de serviço](#modelo-de-cobrança-de-serviço)
* [Principais desafios](#principais-desafios)

O AWS Lambda é atualmente a plataforma serverless em nuvem mais relevante do mercado e será usada como referência ao longo de todas as seções. As características gerais atribuídas ao AWS Lambda são também aplicáveis para outras plataformas. Eventuais características específicas do AWS Lambda que não sejam aplicáveis para outras plataformas serão pontualmente destacadas quando relevantes.
## Configuração básica de uma função Lambda

. Funções Lambda requerem uma conta válida da AWS e podem ser criadas pelo console web ou pelo terminal por meio do AWS Serverless Application Model (SAM). O console web da AWS é acessível por um navegador e é a alternativa mais simples para demonstração:

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
3. Configure a alocação desejada de memória para a função. A alocação de CPU deriva da memória, de forma que cada 1792 MB de memória corresponde a 1 vCPU. O valor pago por execução de função Lambda escala com a alocação de recursos, conforme detalhado [adiante](#modelo-de-cobrança-de-serviço).
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

## Requisições "Frias" e "Quentes"

O tempo gasto durante a etapa "INIT" do ciclo de vida das funções serverless varia significativamente conforme o runtime selecionado para a execução. Por exemplo, funções baseadas em Java tendem a demorar mais para inicializar quando comparadas a funções escritas para a plataforma NodeJS. Isso resulta do longo tempo de inicialização da Java Virtual Machine (JVM) e demais dependências. Essa diferença de performance não é documentada oficialmente pelas plataformas Serverless mas é [amplamente discutida pela comunidade](https://www.pluralsight.com/resources/blog/cloud/does-coding-language-memory-or-package-size-affect-cold-starts-of-aws-lambda).

A demora na inicialização do ambiente de execução (etapa "INIT") pode se tornar um problema de escalabilidade em situações onde precisamos que o sistema reaja rapidamente a um aumento de demanda. Além disso, é um problema de custo já que as plataformas Serverless em nuvem cobram pelo tempo que as funções estão em execução incluindo o tempo de inicialização do ambiente. Dentro desse contexto surgem os conceitos de inicialização "Fria" e "Quente" das funções.

Uma requisição "fria" acontece quando um ambiente de execução novo precisa ser provisionado para que uma função atenda a uma requisição. Esse provisionamento vai passar pelo custo de tempo "INIT", que pode ser alto dependendo das dependências da função e seu runtime. Por outro lado, uma inicialização "quente" é mais rápida pois acontece quando a plataforma Serverless reaproveita um ambiente de execução para atender a uma nova requisição, executando a etapa "INVOKE" sem a necessidade de repetir a etapa "INIT. Portanto, runtimes com baixo custo de inicialização como NodeJS ou Python são preferíveis para funções Serverless por minimizarem o tempo gasto com requisições "frias".

## Modelo de cobrança de serviço

Conforme o marketing das provedoras, uma das principais vantagens do paradigma Serverless é o modelo de cobrança baseado na utilização do serviço:

* **AWS Lambda**: “Economize custos pagando apenas pelo tempo de computação usado (por milissegundo) em vez de provisionar a infraestrutura antecipadamente para obter capacidade máxima.”​
* **Google Cloud Functions**: “Só será cobrado o período de execução da função, que é medido até o valor mais próximo de 100 milissegundos. Você não paga nada quando a função está ociosa. O Cloud Functions adapta automaticamente seus recursos em função dos eventos.”
* **Microsoft Azure Functions**: “Pague pela capacidade de computação por segundo, sem compromisso de longo prazo ou pagamentos adiantados. Aumente ou diminua o consumo sob demanda.”

As plataformas serverless​ seguem o modelo "pay-per-use" ou "pay-as-you-go" de forma que o tempo ocioso dos servidores que hospedam as aplicações não é incluído no custo. Ao invés disso, os usuários da plataforma serverless são cobrados apenas pelo tempo útil dos serviços, ou seja, o tempo em que esses serviços estão ativamente processando requisições. Além disso, o preço do tempo de execução escala proporcionalmente à alocação de recursos de memória e CPU das funções, de forma que o milissegundo de execução é mais caro para funções com mais recursos. Consulte a página de [definição de preço do AWS Lambda](https://aws.amazon.com/pt/lambda/pricing/) para exemplos práticos de precificação de serviços serverless. 

Especificamente no AWS Lambda é possível provisionar um armazenamento efêmero de arquivos para ser acessado pela função. Esse armazenamento é cobrado conforme espaço de armazenamento provisionado e tempo em uso. Ainda, especificamente no AWS Lambda é cobrada uma taxa pela transferência de dados para fora da nuvem de forma equivalente a [política do Amazon EC2](https://aws.amazon.com/pt/ec2/pricing/on-demand/#Data_Transfer).

A equação abaixo representa um modelo geral de custo $C$ para funções Lambda onde:

* $T$ é o tempo total em processamento
* $M$ é a alocação de memória da função
* $P_m$ é o preço atrelado à alocação de memória. Em um exemplo, atualmente o AWS Lambda cobra $2,1 \times 10^{-9}$ USD por milissegundo de execução em funções com 128 MB de memória. Essa taxa é maior para funções com mais memória, e pode mudar conforme política da AWS.
* $S$ é o espaço de armazenamento efêmero provisionado para arquivos.
* $P_s$ é o preço atrelado ao espaço de armazenamento efêmero. Atualmente o AWS Lambda cobra $3,09 \times 10^{-8}$ USD por cada GB/segundo de utilização.
* $N$ é o numero de requisições processadas pela função durante $T$.
* $P_r$ é o preço por requisição processada. Atualmente é cobrado 0,2 USD para cada 1 milhão de requisições.
* $D$ é o volume de dados transferidos para fora da AWS.
* $P_{dt}$ é o preço atribuído a transferência de dados para fora da nuvem. Atualmente os primeiros 10 TB/mês custam 0,09 USD por GB.

$$
C = T \cdot (M \cdot P_{m} + S \cdot P_{s}) + N \cdot P_{r} + D \cdot P_{dt}
$$

Especificamente no AWS Lambda $P_m$ diminui conforme o aumento do uso do serviço no mês. Dessa forma, $P_m$ nos primeiros 6 bilhões de GB-segundos/mês é maior do que $P_m$ para funções com mais de 15 bilhões de GB-segundos/mês. Essa relação de escala é apresentada na [documentação da AWS](https://aws.amazon.com/pt/lambda/pricing/#AWS_Lambda_Pricing). Similarmente, $P_{dt}$ também decai conforme aumento das transferências de dados em um mês conforme [política de transferência de dados do Amazon EC2]($P_{dt}$).

Em geral, as plataformas serverless também incluem cobranças eventuais decorrentes de integrações com outros serviços de nuvem externos ao ambiente serverless como armazenamento permanente, bancos de dados e filas.

## Principais desafios

Apesar dos benefícios de abstração de infraestrutura e modelo de cobrança "pay-per-use" o paradigma Serverless atualmente tem pontos negativos e desafios para o futuro. Essa seção se dedica a listar e explicar alguns dos principais aspectos.

**Requisições "frias" e "quentes"**: Esse aspecto foi detalhado em [uma seção dedicada](#requisições-frias-e-quentes). Embora runtimes com baixo tempo de inicialização sejam preferíveis para Serverless, nem sempre essa escolha é possível. Em migrações de sistemas, por exemplo, é comum que o sistema seja migrado com a mesma linguagem ou runtime do legado. Por isso, a comunidade e plataformas Serverless estão constantemente inovando em maneiras de mitigar as requisições "frias".

**Falta de controle da infraestrutura**: O benefício de abstração impede que desenvolvedores controlem ou alterem a infraestrutura que suporta as funções Serverless. Essa falta de controle impede customizações específicas por contexto de aplicação. Ainda, apenas as plataformas tem controle sobre o reaproveitamento de ambiente de execução e o escalonamento de tarefas entre os servidores da nuvem. Essa é uma característica inerente do paradigma Serverless que o difere de outros paradigmas mais tradicionais como on-premises.

**"Vendor lock-in"**: No contexto de Serverless, esse termo se refere ao alto acoplamento entre uma aplicação e a plataforma Serverless, dificultando a interoperabilidade entre plataformas diferentes. Isso acontece pois o código da função adere especificamente à interface ou API da plataforma onde ele executa, além de potenciais dependências de aspectos específicos de plataforma como armazenamento efêmero de arquivos e outros serviços de nuvem. Esse continua sendo um desafio do contexto de Serverless: Enquanto a comunidade inova em maneiras de criar mais interoperabilidade entre provedores de nuvem, eles por sua vez não contribuem criando interfaces comuns ou outras padronizações.

Além desses, existem outros desafios relacionados a previsibilidade de performance, testes automatizados, arquitetura distribuída e assíncrona baseada em funções, entre outros.
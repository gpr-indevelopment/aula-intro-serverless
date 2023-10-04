# Lambda hands on

Esse repositório contém instruções e código base para uma aula prática e introdutória de tecnologias serverless com AWS Lambda. A prática proposta aqui aborda tópicos como:

* AWS Toolkit for Visual Studio Code.
* Funções Javascript com NodeJS 18 no AWS Lambda.
* Testando funções Lambda em ambiente local.
* Testes unitários para código serverless.
* Executando funções Lambda por gatilho HTTP.

# Pré-requisitos

A instalação de alguns programas é necessária para execução de todos os passos da prática. Esse capítulo lista os pré-requisitos e informações adicionais para a instalação e configuração desses programas. Todos os passos da prática assumem uma máquina com arquitetura x86_64 e Windows 10 ou superior.

O ambiente de desenvolvimento local é integrado com a nuvem AWS. Portanto, uma conta AWS valida é necessária para a prática. A criação da conta exige um cartão de crédito, no entanto, o [nível gratuito da AWS](https://aws.amazon.com/pt/free/) é mais do que suficiente para todas as atividades dessa prática.

* [Docker Desktop](https://docs.docker.com/desktop/install/windows-install/): Necessário para teste local das funções Lambda. Esse produto é gratuito para fins educacionais.
* [AWS Command Line Interface (CLI)](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html): É necessário para funcionalidades básicas de integração com a nuvem AWS. Após a instalação, executar o comando `aws configure` e inserir as chaves de acesso (access key e secret key) da sua conta AWS com a região `sa-east-1` que representa São Paulo. A [documentação oficial da AWS](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html#Using_CreateAccessKey) explora como criar uma access key pelo console web.
* [AWS Serverless Application Model (SAM) CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html#install-sam-cli-instructions): Toolkit de desenvolvimento de aplicações Lambda. Tem o AWS CLI como pré-requisito. Reúne funcionalidades muito úteis de deployment e execução local das funções Lambda.
* [Visual Studio Code (VS Code)](https://code.visualstudio.com/download): Editor de código que usaremos como ambiente de desenvolvimento de aplicações Lambda.
* [Extensão AWS Toolkit do Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=AmazonWebServices.aws-toolkit-vscode): Instalar pelo link ou pela loja de extensões disponível dentro do Visual Studio Code no menu do lado esquerdo (buscar pelo nome "AWS Toolkit").
* [NodeJS 18](https://nodejs.org/dist/v18.17.1/node-v18.17.1-x64.msi): Nessa prática vamos desenvolver funções NodeJS para AWS Lambda. Portanto, uma instalação do NodeJS 18 (LTS) se faz necessária juntamente com o seu gerenciador de pacotes NPM.

# Ambiente de desenvolvimento

O VS Code vai ser usado como ambiente de desenvolvimento durante toda a prática em conjunto com a extensão AWS Toolkit. Primeiro, precisamos configurar o AWS Toolkit para que se comunique com o ambiente em nuvem da AWS.

## Configurando AWS Toolkit

Ao abrir o AWS Toolkit, será solicitado que você crie uma conexão. Entre os tipos de conexão disponíveis, selecione o tipo `AWS Explorer`. Dentro do menu, expanda a opção `add IAM User Credentials`, e insira as suas chaves de acesso da AWS juntamente com um nome de perfil. **O nome de perfil e credenciais ficarão salvos na sua máquina.**

![Credenciais AWS Toolkit](/assets/aws-toolkit-credenciais.gif)

A chave de acesso é necessária para que seja possível a comunicação do ambiente de desenvolvimento com a nuvem AWS da sua conta. As chaves de acesso (access key e secret key) podem ser as mesmas utilizadas na configuração do AWS CLI, [geradas conforme documentação oficial](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html#Using_CreateAccessKey).

Se configurado com sucesso, o menu do AWS Toolkit mostrará uma lista de recursos agrupados por serviço da AWS, incluindo o Lambda.

![Sucesso AWS Toolkit](/assets/aws-toolkit-success.JPG)

## Criando sua primeira função Lambda

O AWS Toolkit pode trabalhar em conjunto com o AWS SAM para criar um projeto base de função Lambda com diversas abstrações que facilitam o setup de um ambiente de desenvolvimento serverless integrado com a nuvem.

O primeiro passo é abrir o menu de comandos do VS Code (`F1` ou `Ctrl + Shift + P` no Windows), e buscar por "AWS: Create Lambda SAM Application". Depois, selecione `nodejs18.x` como o runtime da função que vamos desenvolver com a arquitetura x86_64 no template "AWS SAM Hello World". Você pode escolher uma pasta para o projeto e seu nome em seguida. Após isso, uma estrutura básica de projeto será criada com alguns arquivos.

![Sucesso AWS Toolkit](/assets/sam-create-basic-lambda.gif)

## A função "hello world"

A pasta `./hello-world` contém os arquivos fontes da função Lambda criada. O arquivo `package.json` do NPM contém metadados da função assim como declarações de dependências e scripts de execução. Primeiro, execute `npm install` nessa pasta para que o NPM faça o download das dependências do projeto. Depois, o comando `npm run test` pode ser usado para executar os testes unitários salvos na subpasta `/tests` juntamente com o framework [Mocha](https://mochajs.org/).

O código fonte da sua função Lambda se encontra no caminho `./hello-world/app.mjs` relativo a pasta de projeto criada. A função handler é executada toda vez que a função Lambda for invocada. O evento de entrada da função pode ser lido pelo objeto `event`. Esse evento de entrada pode tomar a forma do corpo de uma requisição HTTP ou uma mensagem em uma fila, dependendo do gatilho configurado para a função; Inicialmente o código gerado retorna um JSON com a mensagem "hello world".

```js
export const lambdaHandler = async (event, context) => {
    try {
        return {
            'statusCode': 200,
            'body': JSON.stringify({
                message: 'hello world',
            })
        }
    } catch (err) {
        console.log(err);
        return err;
    }
};
```

## Utilizando o ambiente local de desenvolvimento

Os arquivos `README.md` e `README.TOOLKIT.md` contém toda a documentação de ambiente local gerada pelo Toolkit para o projeto básico de função Lambda. Em resumo, as funcionalidades mais importantes do ambiente estão listadas abaixo. Todos os comandos devem ser executados a partir da raiz do projeto Lambda gerado:

* `sam build`: Reúne todos os artefatos e dependências necessárias para a função Lambda em uma pasta nomeada `.aws-sam`. Esse comando é um pré-requisito para a execução dos demais comandos do SAM CLI. O SAM sugere outros comandos que você pode experimentar na saída do comando `build`. Exemplo:

```
Commands you can use next
=========================
[*] Validate SAM template: sam validate
[*] Invoke Function: sam local invoke
[*] Test Function in the Cloud: sam sync --stack-name {{stack-name}} --watch
[*] Deploy: sam deploy --guided
```
* `sam local invoke`: Dispara a função construída pelo comando `build` em um container Docker, simulando o ambiente de execução da plataforma Lambda. Um evento pode também ser utilizado como entrada da função pelo parâmetro `-e`. Consulte `sam local invoke --help` para mais informações. O código básico de "hello world" não depende de parâmetros, portanto, pode ser disparado pelo comando `sam local invoke` sem parâmetros adicionais. O container é desligado após o término da execução da função.

* `sam local start-api`: Inicializa um container que expõe a função Lambda a um gatilho de requisição HTTP por uma API. Essa API é inicializada por padrão na porta 3000, e pode ser acessada em `http://127.0.0.1:3000`. A função "hello world" pode ser invocada sem parâmetros pela chamada de API `curl http://localhost:3000/hello`, mas também é possível a passagem de dados para a função pelo corpo da requisição HTTP em um POST, por exemplo. Encerrar o terminal que está executando o comando `sam local start-api` também desligará o container Docker.

* `sam deploy --guided`: Dá inicio a um deployment guiado da função para o ambiente AWS Lambda na nuvem. O SAM apresentará prompts com diversos dados necessários para o deployment utilizando suas credenciais configuradas para o AWS CLI. O mesmo deployment também é possível pela tela do AWS Toolkit no VS Code, e será a alternativa abordada nesse guia.

## Realizando um deployment da função para a nuvem

Uma vez testada localmente, a função Lambda pode ser disponibilizada na nuvem AWS. Esse deployment pode ser realizado pelo AWS Toolkit no VS Code de forma interativa. O menu principal do Toolkit lista todas as funções presentes na sua conta AWS por região. O deployment pode ser iniciado pela opção "Sync SAM Application" disponível ao clicar-se com o botão direito no agrupamento Lambda do Toolkit na região desejada.

![Deployment da função Lambda](/assets/lambda-deployment.JPG)

Você pode configurar a visibilidade das regiões da AWS no Toolkit pelo menu "Show or Hide Regions", conforme abaixo:

![Visibilidade de regiões](/assets/show-or-hide-regions.JPG)

1. Primeiro, selecione o arquivo `template.yaml` gerado no projeto base da função. Ele contém as instruções padrões de deployment.
2. Depois, você precisa nomear uma nova "stack" do CloudFormation. Não precisamos entrar em detalhes do que é o CloudFormation, mas nesse contexto ele agrupa os recursos de nuvem criados a partir do deployment dessa função. Você pode ler mais sobre o CloudFormation [aqui](https://aws.amazon.com/pt/cloudformation/). A mesma "stack" pode ser reutilizada em deployments subsequentes da mesma função.
3. O código fonte e bibliotecas da função precisam ser armazenados em um local que a AWS possa usar para construir o ambiente de execução quando a função for invocada. O local padrão é um "bucket" S3, que é o serviço de armazenamento de objetos AWS. O próximo passo consiste em nomear um "bucket" para o armazenamento dos arquivos desse deployment. O mesmo "bucket" S3 pode ser reutilizado em deployments subsequentes da função.

Por fim, o processo de deployment vai ser executado pelo SAM em uma nova instância de terminal usando as mesmas credenciais configuradas no AWS Toolkit do VS Code. Você pode acessar sua nova função pelo console da AWS ou pela listagem de recursos do AWS Toolkit.

## Executando sua função na nuvem

A função "hello world" pode ser executada na nuvem pelo AWS Toolkit. Para isso, basta clicar com o botão direito na função pela listagem de recursos e selecionar a opção "Invoke on AWS":

![Visibilidade de regiões](/assets/invoke-on-aws.JPG)

Você pode configurar um evento como entrada para a invocação da função, ou apenas dispará-la pelo botão "Invoke", no canto superior direito. Clicar nesse botão invocará a função no ambiente em nuvem comas as [regras de cobrança da AWS para funções Lambda](https://aws.amazon.com/pt/lambda/pricing/).

![Visibilidade de regiões](/assets/invoke-on-aws.gif)


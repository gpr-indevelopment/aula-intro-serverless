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
* [AWS Serverless Application Model (SAM)](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html#install-sam-cli-instructions): Toolkit de desenvolvimento de aplicações Lambda. Tem o AWS CLI como pré-requisito. 
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

O primeiro passo é abrir o menu de comandos do VS Code (`F1` ou `Ctrl + Shift + P` no Windows), e buscar por "AWS: Create Lambda SAM Application". Depois, selecione `nodejs18.x` como o runtime da função que vamos desenvolver com a arquitetura x86_64 no template "AWS SAM Hello World". Você pode escolher uma pasta para o projeto e seu nome em seguida. Após isso, uma estrutura básica de projeto será criada com diversos arquivos.

![Sucesso AWS Toolkit](/assets/sam-create-basic-lambda.gif)

A pasta `./hello-world` contém os arquivos fontes da função Lambda criada. O arquivo `package.json` do NPM contém metadados da função assim como declarações de dependências e scripts de execução. Primeiro, execute `npm install` nessa pasta para que o NPM faça o download das dependências do projeto. Depois, o comando `npm run test` pode ser usado para executar os testes unitários salvos na subpasta `/tests` juntamente com o framework [Mocha](https://mochajs.org/).

O código fonte da sua função Lambda se encontra no caminho `./hello-world/app.mjs` relativo a pasta de projeto criada. A função handler é executada toda vez que a função Lambda for invocada. Inicialmente o código gerado retorna um JSON com uma mensagem de "hello world".

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
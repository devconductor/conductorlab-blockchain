# GETTING STARTED WITH BLOCKCHAIN WITH HYPERLEDGER TOOLS

> Certifique-se de instalar todos os requisitos necessários para iniciar o desenvolvimento:

* Sistemas operacionais: Ubuntu 14.04 **+**, ou Mac OS 10.12
* Ao menos 4GB de RAM
* Docker Engine: versão 17.03 **+**
* Docker-Compose: versão 1.8 **+**
* Node: 8.9 **+**
* npm: v5.x **+**
* git: 2.9.x **+**
* Python: 2.7.x **+**
* Um editor de texto de sua escolha, nós recomendamos o VSCode

> Caso esteja utilizando o ubuntu 16.04, utilize o seguinte script para instalar os pre-requisitos:

```
curl -O https://hyperledger.github.io/composer/latest/prereqs-ubuntu.sh
```


```
chmod u+x prereqs-ubuntu.sh
```

> Em seguida rode o script. O mesmo fara o uso do sudo em sua execução e você precisará autorizar com sua senha:
```
./prereqs-ubuntu.sh
```

## Passo 1: Instalando as ferramentas de linha de comando:

1. Ferramentas essenciais CLI:
```
npm install -g composer-cli@0.20
```

2. Utilitário para gerar um servidor REST na sua máquina e tornar a rede uma API REST.
```
npm install -g composer-rest-server@0.20
```

3. Útil para gerar ativos da aplicação.
```
npm install -g generator-hyperledger-composer@0.20
```

4. Yeoman é uma ferramenta para gerar aplicações. A mesma utiliza o `generator-hyperledger-composer`
```
npm install -g yo
```


## Passo 2: Instalando o Playground

1. Aplicação de navegador para editar e testar uma rede de negócios.
```
npm install -g composer-rest-server@0.20
```

## Passo 3: Configurando sua IDE

#### Enquanto podemos utilizar o navegador para trabalhar na aplicação, muitos usuários preferem utilizar uma IDE. Nós recomendamos o uso do VSCode pois o mesmo tem uma extensão do Composer disponível.

1. Instale o VSCode por este link: [VSCode DOWNLOAD](https://code.visualstudio.com/download "VSCode")
2. Abra-o, vá em extensões e procure por `Hyperledger Composer` e em seguida instale-o.

## Passo 4: Instalando o Hyperledger Fabric
#### Este passo fará com que tenhas um runtime do Hyperledger Fabric em sua máquina para dar deploy em suas redes.

1. Em um diretório de sua escolha, baixe e faça a extração do arquivo `.tar.gz` que contem os arquivos necessários para a instalação:
```
mkdir ~/fabric-dev-servers && cd ~/fabric-dev-servers
```

```
curl -O https://raw.githubusercontent.com/hyperledger/composer-tools/master/packages/fabric-dev-servers/fabric-dev-servers.tar.gz
```

```
tar -xvf fabric-dev-servers.tar.gz
```

2. Use os scripts que você acabou de baixar e extrair para adquirir uma versão local do Hyperledger Fabric 1.2:
```
cd ~/fabric-dev-servers
```

```
export FABRIC_VERSION=hlfv12
```

```
./downloadFabric.sh
```

> Parabéns, você agora instalou tudo o que precisa para começar a desenvolver.

# INICIANDO O DESENVOLVIMENTO

O Hyperledger Composer fornece uma opção mais amigável para o desenvolvimento de aplicações blockchain. Nesse tutorial iremos definir todos os elementos da nossa rede e, por último, iremos gerar uma API REST que pode servir de ponte entre a rede blockchain e aplicações externas.
Para definir todos os elementos dessa rede, iremos seguir os passos fornecidos pela documentação oficial do Hyperledger Composer, que são:
1. Criar a estrutura da rede;
1. Definir os elementos da rede;
1. Gerar um arquivo de rede;
    * Arquivo ao qual são empacotados todas as informações e todos os elementos da rede, necessários para alocar nossa rede blockchain em uma instância de rede Hyperledger Fabric.
1. Implantar a rede em uma instância do Hyperledger Fabric;
1. Gerar um Servidor REST;

No primeiro passo utilizaremos uma ferramenta chamada Yeoman, que basicamente vai gerar um esqueleto ou estrutura básica de diretórios contendo uma “Rede de Negócios”. Essa estrutura conterá alguns arquivos de definições de uma Rede de Negócios básica como por exemplo:
1. O arquivo de modelos (models.cto) que conterá as informações pertinentes aos participantes, ativos e transações da rede.
1. O arquivo de controle de acessos (permissions.acl) que conterá regras para definir a quais ativos de uma rede os participantes terão acesso e sob quais condições.

Para isso, executamos o seguinte comando no terminal:
```
yo hyperledger-composer:businessnetwork
```

Ao executar esse comando, serão pedidas algumas informações para criar a estrutura de rede. A primeira será o nome da rede que definiremos como `blockchain-teste`. Depois colocaremos a descrição da rede, o nome e e-mail do autor. O campo “Licence” podemos deixar com a opção default (Apache-2.0). No campo “Namespace”, colocaremos `org.teste.blockchain`. E por último, a ferramenta pergunta se queremos gerar uma rede vazia ou não. Selecionaremos “Sim”.
Se a geração for bem sucedida, deverão ser exibidos no terminal os arquivos e diretórios criados pela ferramenta:

    create package.json
    create README.md
    create models/org.test.blockchain.cto
    create permissions.acl
    create .eslitrc.yml

Depois de executado esse comando devemos ter uma pasta criada com o mesmo nome da rede. Vá até o diretório do projeto.
Como selecionamos a opção “Sim” para gerar uma rede vazia, o gerador Yo gerou o mínimo de arquivos e diretórios necessários para uma rede de negócios Hyperledger Composer. Para o nosso tutorial precisaremos de mais que isso, portanto devemos criar na raiz do projeto um arquivo chamado “logic.js”.

No segundo passo nós iremos definir os elementos de nossa rede seguindo como base os templates de cada arquivo que foi gerado no passo 1. Para isso utilizaremos a VSCode que foi sugerida como um dos pré-requisitos.

Basta navegar até o diretório do projeto `/blockchain-teste` e digitar o seguinte comando:
```
code .
```

Nesse tutorial definiremos uma rede blockchain bem simples, que simulará um mercado imobiliário, onde teremos como ativos fazendas, e teremos como participantes os proprietários das fazendas.

Na IDE, abra o arquivo `org.teste.blockchain.cto` contido no diretório models e substitua o conteúdo do arquivo pelo seguinte:

```javascript
namespace org.test.blockchain

participant Proprietario identified by cpf {
    o String cpf
    o String nomeProprietario
}

asset Fazenda identified by nomeFazenda {
    o String nomeFazenda
    o Double valor
    --> Proprietario proprietario
}

transaction Transferencia {
    --> Fazenda fazenda
    --> Proprietario novoProprietario
}
```

Aqui temos definido um modelo de participante cujas instâncias serão identificadas pelo `cpf` e possuirão um `nomeProprietario` como atributo, um modelo de ativo cujas instâncias são identificadas pelo `nomeFazenda` e possuem um atributo `valor` e um relacionamento com um participante da rede. 

Por último um modelo de transação que possui um relacionamento com um ativo e um relacionamento para uma instância que representa o novo proprietário da fazenda.
O código desse arquivo é escrito na linguagem orientada a objetos: [Hyperledger Composer Modeling Language](https://hyperledger.github.io/composer/latest/reference/cto_language).


Agora que temos os modelos definidos, devemos implementar a lógica que irá de fato processar a transação e modificar a blockchain, utilizando as definições contidas no arquivo de modelos.
Na IDE, abra o arquivo `logic.js` que criamos e substitua o conteúdo do arquivo pelo seguinte:

```javascript
/**
* @param {org.teste.blockchain.Transferencia} tr
* @transcation
*/
async function transferenciaDeProprietario(tr) {
    tr.fazenda.proprietario = tr.novoProprietario;
    let assetRegistros = await getAssetRegistry('org.blockchain.Fazenda');
    await assetRegistros.update(tr.fazenda)
}
```

> Considerando as linhas do arquivo `logic.js`:

* 2: nós criamos um objeto do tipo parâmetro, que representa qual dos modelos de transação iremos implementar;

* 5: passamos esse objeto como parâmetro para a função JavaScript em questão. Na linha 6 substituímos o relacionamento do asset fazenda contido do objeto parâmetro diretamente pelo relacionamento também contido no objeto parâmetro;

* 7: recuperamos todos os “registros” do asset “Fazenda” contido na blockchain e salvamos na variável “assetRegistros”;

* 8: nós atualizamos esses registros diretamente na blockchain passando como parâmetro para a função “update” o próprio objeto que modificamos nessa transação.

O código desse arquivo é escrito em `Javascript`. 

Nesse tutorial não iremos especificar como funciona o permissionamento numa rede de negócios Hyperledger Composer, portanto não iremos modificar o arquivo `permissions.acl`.

<h1 align="center">
  <img width="60" src="https://www.svgrepo.com/show/354202/postman-icon.svg"><br>
  Rest API Test with Postman
</h1>
<h1>Índice</h1>

[Sobre o Projeto](#-sobre-o-projeto)<br>
[Preparação do Ambiente](%EF%B8%8F-preparação-do-ambiente-)<br>
[Cenário de Teste](#-cenário-de-teste)<br>
[Primeiros Passos no Postman](#-primeiros-passos-no-postman-)<br>
ㅤ[Pre-request Script](#-pre-request-script)<br>
ㅤ[Realizando Requisições do tipo GET](#realizando-requisições-do-tipo-get)<br>
ㅤ[Passando os Requisitos para o Código](#passando-os-requisitos-para-o-código)<br>
ㅤ[Criando Massa de Dados (Objetos)](#criando-massa-de-dados-objeto)<br>
ㅤ[Passando a Massa de Dados para o Body](#passando-a-massa-de-dados-objetos-para-o-body)<br>
[Body](#-body)<br>
[Tests](#-tests)<br>
ㅤ[Declaração de Variáveis](#declaração-de-variáveis)<br>
ㅤ[Função para criação da Base dos Testes](#função-para-criação-da-base-dos-testes)<br>
ㅤ[Criando os Testes](#criando-os-testes)<br>
[Notas Finais](#contribuições)<br>
[Autores](#comentários-da-autora)


<h2>🚀 Sobre o Projeto</h2>
Esse projeto visa instruir sobre o modelo de testes de API usado pela autora do projeto em questão. Além de disponibilizar o código para download.
O objeto técnico do projeto é criar validação para requisições do tipo POST/PUT utilizando como estretégia:

* Criação de massa de dados na aba pre-request script do postman para atingir a expectativa do teste.
* Enviar dados de forma dinâmica para o corpo da requisição na aba body.
* Validar a saída das requisições na aba tests.

<h2>🛠️ Preparação do Ambiente </h2>

1. Criar uma conta no Postman.
2. Instalar Postman.
3. Baixar a collection desse repositório.
4. Importar a collection no Postman.

<!-- CENÁRIO DE TESTE -->
 <h2>📋 Cenário de Teste</h2>
<table border="0", align="center">
    <tr>
        <td>
          <p>O formulário ao lado possui as seguintes regras de negócio:</p>
          <ul>
            <li><b>Nome</b>, <b>Senha</b>, <b>Email</b> e <b>Cargo</b> não podem ser vazios;</li>
            <li><b>Nome</b> deve ter entre 3 a 80 caracteres;</li>
            <li><b>Senha</b> deve ter ao menos 6 caracteres e deve conter letra(a) e número(s);</li>
            <li><b>Email</b> deve ser válido e não pode ser repetido;</li>
            <li><b>Telefone</b> deve ter 11 dígitos numéricos;</li>
            <li><b>Cargo</b> deve existir (QA, PO ou DEV.)</li>
          </ul>
          <p>Os requisitos em questão serão usados como base para a criação dos scripts de teste na ferramenta <b>Postman</b>.</p>
        </td>
        <td><img src="https://github.com/andreinaoliveira/RestAPITest/assets/51168329/b4edcb08-c793-4e9a-9f1e-060bdc913e52"></td>
    </tr>
</table>

<h2>💻 Primeiros passos no Postman </h2>

<h3>➞ Pre-request script</h3>
<h4>Realizando requisições do tipo GET</h4>

O trecho abaixo fará uma requisição para obter a lista de usuários cadastrados no servidor. Dessa lista, será salvo em _existingEmails_ apenas o e-mail desses usuários. Isso auxiliará em um teste futuro como base para checar se o email do usuário é duplicado.

```javascript
pm.sendRequest({
    url: 'https://andreina-qa-test.onrender.com/api/users',
    method: 'GET',
    header: {
        'Authorization': `${pm.collectionVariables.get("token")}`
    }
}, function (err, response) {
    if(err){
        console.error('Erro ao tentar acessar lista de usuários', err)
    }
    pm.variables.set('existingEmails', response.json().map(item => (item.email)))
})
```

O Processo de vida do código quando acontece uma requisição é de executar as demais linhas do código sem aguardar o resulltado da requisição. Portanto, será necessário adicionar um timer (também pode ser um assync await) fazendo com que as demais linhas do código sejam executadas apenas após receber os dados da requisição anterior.

```javascript
setTimeout(function () {

}, 2000)
```
<h4>Passando os requisitos para o código</h4>

Dentro do _setTimeout_ será criado a variável _validation_ onde conterá as regras de negócios definidas em [#1](#1), além disso, contará com uma lista de e-mails inválidos para validações futuras

```javascript
    var validation = {
        minNome: 3,
        maxNome: 80,
        minSenha: 6,
        lenTelefone: 11,
        validCargo: ['QA', 'DEV', 'PO'],
        emailDuplicated: pm.variables.get('existingEmails')
    }; pm.variables.set('validation', validation)

    var ListOfInvalidEmails = [
        "invalid_email",                   // Does not have '@'
        "invalid@.com",                    // Dot after '@'
        "invalid@domain",                  // Without extension (.com, .org, etc.)
        "@invalid.com",                    // Begins with '@'
        "invalid.email.com",               // Colon in a domain
        "invalid@domain@.com",             // More than one '@'
        "invalid email@domain.com",        // Space in an email
        "invalid.email@domain_com",        // Domain underscore
        "invalid.email@domain..com",       // Dot duplicate in domain
        "invalid@domain.-com",             // Hyphen after dot in domain
        "invalid.email@domain.com."        // Dot at end of email
        // Add more invalid emails if needed
    ]
```
<h4>Criando Massa de Dados (Objeto)</h4>

Logo após o código acima, ainda dentro do _setTimeout_, será criado o objeto _validData_ que contem os campos Nome, Senha, Email, Telefone e Cargo com dados válidos.

```javascript
var validData = {                                             // Dados Válidos
        nome: pm.variables.replaceIn('{{$randomFirstName}}'), // Nome Válido
        senha: 'abc123',                                      // Senha Válida
        email: pm.variables.replaceIn('{{$randomEmail}}'),    // Email Válido
        telefone: _.random(10000000000, 99999999999),         // Telefone Válido
        cargo: _.sample(validation.validCargo)                // Cargo Válido
    }
```

Com base no objeto _validData_, será criado uma cópia dele para os objetos abaixo.
- emptyName
- minName
- maxName
- emptyPassword
- minPassword
- emptyEmail
- duplicatedEmail
- invalidEmail
- invalidTelefone
- lenTelefone
- emptyCargo
- invalidCargo

E em seguida o objeto será modificado para conter a falha desejada. Segue exemplo de como isso ocorre:

```javascript
var minName = _.cloneDeep(validData); minName ={                          // Mínimo Nome
    ...minName, nome: 'a'.repeat(validation.minNome - 1)
}

var emptyPassword = _.cloneDeep(validData); emptyPassword ={              // Senha Vazia
    ..emptyPassword, senha: null
}

var duplicatedEmail = _.cloneDeep(validData); duplicatedEmail ={          // Email Duplicado
    ...duplicatedEmail, email: _.sample(validation.emailDuplicated)
}

var invalidEmail = _.cloneDeep(validData); invalidEmail ={                // Email Duplicad.
    ...invalidEmail, email: _.sample(ListOfInvalidEmails)
}
```

Inclusive é possível agrupar falhas dependendo de como o back-end funciona. Por exemplo, no caso de deixar todos os dados vazios e o servidor retornar mensagem de dados vazios para cada campo, é possível criar apenas um objeto contendo todos os campos recebendo null.

<h4>Passando a Massa de Dados (Objetos) para o body</h4>

Ao final da massa de testes, dentro do _setTimeout_ está sendo declarado _testData_ que é a variável que receberá a massa de dados dos testes. Tendo recebido esses dados, será percorrido cada um deles e criado uma variávevl visível no contexto da requisição (ou seja, para todas as abas). Isso servirá para enviar esses dados para o corpo da requisição (body).

```javascript
setTimeout(function () {
const testData = TESTE_AQUI
    for (const key in testData) {
        pm.variables.set(`qa_${key}`, testData[key] === null 
            ? null
            : `"${testData[key]}"`)
    }
}, 2000)
```
---
<h3>➞ Body</h3>
No body será inserido a variável correspondente ao campo, conforme criado no pre-request. Dessa forma será possível enviar de forma automática os dados da requisição, sem a necessidade de escrever manualmente. 

```json
{
    "nome": {{qa_nome}},
    "senha": {{qa_senha}},
    "email": {{qa_email}},
    "telefone": {{qa_telefone}},
    "cargo": {{qa_cargo}}
}
```
---
<h3>➞ Tests</h3>

<h4>Declaração de Variáveis</h4>

Logo no topo da aba testes haverá as variáveis abaixo sendo _message_ responsável pela mensagem retornada ao enviar a requisição, _req_ o copo da requisição enviada e _validation_ o objeto previamente criado no pre-request. Ainda declarando as variáveis que ajudaram na validação, foi declarado o emailRegex com objetivo de validar e-mails.

```javascript
const message = pm.response.text()
const req = JSON.parse(pm.request.body)
const validation = pm.variables.get("validation")
const emailRegex = /^[\w\.-]+@[\w\.-]+\.\w+$/
```

<h4>Função para criação da Base dos Testes</h4>


Abaixo temos a base que dará luz aos testes que serão realizados. A função recebe:
- _testName_: O nome do teste. Ex.: "Bloqueio de telefone diferente de 11 dígitos"
- _condition_: A condição para que o teste seja executado. Ex.: O teste só será executado se o resultado a seguir for verdadeiro ```req.telefone?.length !== 11```
- _expectedResult_: O resultado esperado quando a condição for atingida. Ex.: Dado que _condition_ é verdadeiro então o resultado do código a seguir também deve ser verdadeiro ``` pm.response.code === 400 && message.includes(`Campo telefone deve ter 11 dígitos`)```

Podemos concluir que _condition_ e _expectedResult_ recebem booleanos por parâmentro.

```javascript
function test(testName, condition, expectedResult) {
    if (condition) {
        pm.test(testName, function () {
            pm.expect(expectedResult).to.be.true
        })
        return true
    }
    else {
        return false
    }
}
```

<h4>Criando os Testes</h4>

Muitos testes foram criados e todos os testes estão no arquivo [Hands On.postman_collection](https://github.com/andreinaoliveira/RestAPITest/blob/master/Hands%20On.postman_collection) para documentação não ficar extensa, estarei dando apenas alguns exemplos. Mas existe uma gama de exemplos no arquivo, alguns com técnicas diferentes para deixar o código mais clean.

Nos exemplos abaixo, temos a variável _isTelephoneInvalidLength_ sendo declarada como _false_ o que significa: "O tamanho de telefone é inválido? MENTIRA". Logo abaixo a variável chama a função anterior referente a base de teste, nela, passando por parâmetro o nome do teste, a condição para que o teste ocorra e o resultado esperado para o caso de o teste ocorrer. O teste ocorrendo retorna _true_ no caso de o telefone ser inválido ou _false_ no caso de válido.

Abaixo, a função teste é chamada com o objetivo de validar se os dados passados no body são válidos, a condição é que todos os testes realizados incluindo o teste anterior, _isTelephoneInvalidLength_, retorne _false_ que, como explicado anteriormente, significa que não é inválido, sendo _true_ uma afirmação de inválido. Atingindo a condição, o resultado do teste precisa atingir o código e mensagem de confirmação.

```javascript
let isTelephoneInvalidLength = false
isTelephoneInvalidLength = test(
    `Bloqueio de telefone diferente de 11 dígitos | telefone tem ${req.telefone?.length} dígitos`,
    req.telefone !== null && req.telefone?.length !== 11,
    pm.response.code === 400 && message.includes(`Campo telefone deve ter 11 dígitos`)
)

test(
    `Dados Válidos 
        | nome: ${req.nome}; 
        senha: ${req.senha}; 
        email: ${req.email}; 
        telefone: ${req.telefone}; 
        cargo: ${req.cargo}`,
    !isRequiredFieldsEmpty
    && !isFieldsOutOfLimit
    && !isTelephoneInvalid
    && !isTelephoneInvalidLength
    && !isEmailDuplicated
    && !isEmailInvalid
    && !isCargoInvalid,
    pm.response.code == 201 && message.includes("Usuário registrado com sucesso")
)
```

# Contribuições
<table border="0", align="center">
    <tr>
        <td>
          <img src="https://github.com/andreinaoliveira/RestAPITest/assets/51168329/209eddcc-5963-4e55-9ccb-5e79182085a0" width=90><br>
          <a href="https://github.com/EriikSilva">Erik Felipe</a>
        </td>
        <td>
          <p>Gostaria de agradecer ao Erik Felipe pelo backend que ele criou para a nossa API no projeto. A habilidade dele é fora de série e fez toda a diferença. Valeu demais, Erik, você arrebenta! 👏</p>
        </td>
    </tr>
</table>

# Comentários da Autora
<table border="0", align="center">
    <tr>
        <td>
          <img src="https://github.com/andreinaoliveira/RestAPITest/assets/51168329/d1e532f9-e1f0-45a6-a9a1-85f78281c563" width=500><br>
          <a href="https://github.com/andreinaoliveira">Andreina Oliveira</a>
          <p></p>
        </td>
        <td>
          <p>O conhecimento em validação de REST API Test é essencial para cobertura de testes no backend. Hoje temos várias formas de garantir esse teste. O conteúdo apresentado nesse repositório apresenta como realizar essas validações utilizando a ferramenta Postman, mas considere que nem sempre essa ferramenta será a mais indicada para o seu contexto. A técnicas apresentada pode ser adaptada e inclusive tranformada em um único teste que valida todo os cenários, visto que a forma em que hoje o código foi implemantado a validação é feita por requisição. Sinta-se livre para usar o código como base para seus testes. Em caso de dúvidas e sugestões estou disponível para contato.</p>
        </td>
    </tr>
</table>

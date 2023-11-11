<h1 align="center">
  <img width="60" src="https://www.svgrepo.com/show/354202/postman-icon.svg"><br>
  Rest API Test with Postman
</h1>
<h2>🚀 Sobre o Projeto</h2>
Esse projeto visa instruir sobre o modelo de testes de API usado pela autora do projeto em questão. Além de disponibilizar o código para download.
O objeto técnico do projeto é criar validação para requisições do tipo POST/PUT utilizándo como estretégia:

* Criação de massa de dados na aba pre-request script do postman para atingir a expectativa do teste.
* Enviar dados de forma dinâmica para o corpo da requisição na aba body.
* Validar a saída das requisições na aba tests.

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

<h2>🛠️ Preparação do Ambiente </h2>

1. Criar uma conta no Postman.
2. Instalar Postman.
3. Baixar a collection desse repositório.
4. Importar a collection no Postman.

<h2>💻 Primeiros passos no Postman </h2>

<h3>Pre-request script</h3>
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
Dentro do _setTimeout_ está sendo declarado _testData_ que é a variável que receberá a massa de dados dos testes. Tendo recebido esses dados, será percorrido cada um deles e criado uma variávevl visível no contexto da requisição (ou seja, para todas as abas). Isso servirá para enviar esses dados para o corpo da requisição (body).

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

<h3>Body</h3>
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

<h3>Tests</h3>
Logo no topo da aba testes haverá as variáveis abaixo sendo _message_ responsável pela mensagem retornada ao enviar a requisição, _req_ o copo da requisição enviada e _validation_ o objeto previamente criado no pre-request

```javascript
const message = pm.response.text()
const req = JSON.parse(pm.request.body)
const validation = pm.variables.get("validation")=
```

Abaixo temos a base que dará luz aos testes que serão realizados. A função recebe o nome do teste, a condição que deve atender para garantir um determinado resultado. Atingindo a condição o teste nativo do postman é executado e a função retornará _true_, do contrário _false_. O resultado booleano retornado da função servirá para sabermos se essa condição condiz com o requisito. Por exemplo, em um test _isEmailDuplicated_ quando essa função retornar true, significa que há um emaill duplicado no formulário.

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

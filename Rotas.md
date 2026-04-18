# Rotas e requisições
Rotas são os caminhos definidos por um sistema de roteamento para direcionar as requisições do usuário para os recursos corretos de uma aplicação. 
Elas associam um endereço (URL) a uma função específica, como a exibição de uma página HTML, o processamento de um formulário ou a chamada de uma API. 
Isso permite que a aplicação responda a diferentes URLs sem que cada uma corresponda diretamente a um arquivo estático no servidor. 

## 1. Criando rotas
Para criar rotas no express usamos o objeto `app` para gerar a rota, que conta como parâmetros 
o nome da rota e um função inline que trata e gerencia as requsições (`req`) e respostas (`res`) à rota.
```JavaScript
import express from "express";
const app = express();

// PORT armazena o número da porta que vai rodar o servidor.
const PORT = process.env.PORT || 3000;

/* Rota principal usando um index.ejs. */
app.get('/', function(req, res, next) {
  return res.render('index', { title: 'Express' });
}); // A mensagem será renderizada no arquivo .ejs

/* Rota “sobre” sem usar uma view. */
app.get('/sobre', function(req, res) {
  let msg = '<h2>Sobre Rotas...</h2>';
  return res.send(msg);
});

app.listen(PORT, () => {
    console.log(`Rodando na porta ${PORT}.`);
});
```

## 2. Usando parâmetros em Rotas
No Express.js usamos parâmetros nas rotas para funcionalidades específicas, como buscar um item pelo ID.
Como isso, acessamos os parâmetro por meiuo da requisição (`req`) e utilizamos os **Parâmetros de Rota** (`req.params`)
e os **Parâmetros de Consulta** (`req.query`).

### Parâmetros de Rota
São usados quando a informação é obrigatória para a rota funcionar (ex: buscar um item pelo ID). 
Eles são definidos com dois pontos (:) na definição da rota.
```JavaScript
import express from "express";
const app = express();

const PORT = process.env.PORT || 3000;

/* Outras rotas definidas anteriormente... */

/* Rota usando 1 parâmetro enviado na URL. */
app.get('/ola/:nome', function(req, res) {
  let msg = '<h2>Olá, ' + req.params.nome + '!</h2>';
  return res.send(msg);
});

/*Podemos adicionar mais de um parâmetro na rota, mas é necessario separar os parâmetros por barras.*/
app.get('/ola/:nome/:sobrenome', function(req, res) {
  let msg = '<h2>Olá, ' + req.params.nome + " " + req.params.sobrenome + '!</h2>';
  return res.send(msg);
});

app.listen(PORT, () => {
    console.log(`Rodando na porta ${PORT}.`);
});
```

### Parâmetros de Consulta
São usados para informações opcionais ou para filtrar resultados. Não precisam ser definidos na estrutura da rota e começam após um ?.
```JavaScript
import express from "express";
const app = express();

const PORT = process.env.PORT || 3000;

// Rota: GET /busca?q=node&pag=1
app.get('/busca', (req, res) => {
    const termo = req.query.q; // "node"
    const pagina = req.query.pag; // "1"
    return res.send(`Resultados para: ${termo} na página ${pagina}`);
});

app.listen(PORT, () => {
    console.log(`Rodando na porta ${PORT}.`);
});
```

## 3. Enviando resposta de requisições
No Express, o objeto de resposta (`res`) oferece diversos métodos para enviar dados de volta ao cliente e finalizar o ciclo de requisição-resposta.
Entre os principais métodos de resposta estão:

- `res.send([body])`: Envia uma resposta HTTP de diversos tipos. O corpo (body) pode ser uma string, um objeto, um Array, um Buffer ou um Boolean. O Express define automaticamente o `Content-Type` (ex: `text/html` para strings, `application/json` para objetos).
- `res.json([body])`: Envia uma resposta JSON. É o método ideal para APIs REST. Ele converte objetos ou arrays JavaScript para uma string JSON (usando `JSON.stringify`) e define o `Content-Type` como `application/json`.
- `res.type(type)`: Define o campo `Content-Type` do cabeçalho HTTP para o tipo MIME fornecido. 
- `res.sendFile(path [, options] [, fn])`: Transfere um arquivo como um octet stream, definindo o Content-Type com base na extensão do arquivo.
- `res.download()`: Solicita que um arquivo seja baixado pelo navegador do cliente. 
- `res.status(code)`: Define o código de status HTTP da resposta (ex: 200, 404, 500). É um método encadeável, o que significa que pode ser usado antes de outro método de resposta.
*Ex:* `res.status(404).send('Not Found')`.
- `res.sendStatus(statusCode)`: Define o status code HTTP e envia a string correspondente ao código como corpo da resposta (ex: `res.sendStatus(200)` envia "OK").
- `res.render(view [, locals] [, callback])`: Renderiza uma view (template) e envia o HTML gerado para o cliente. Geralmente usado com motores de template como EJS, Pug ou Handlebars.
- `res.redirect([status,] path)`: Redireciona a requisição para uma URL específica. Por padrão, o status é 302 (Found).
- `res.end()`: Finaliza o processo de resposta manualmente sem enviar nenhum dado no corpo, útil para respostas vazias. 

```js
import express from "express";
const app = express();

const PORT = process.env.PORT || 3000;

app.get('/api/user', (req, res) => {
  // Define status 200 e envia JSON
  return res.status(200).json({ id: 1, name: 'João' });
});

app.get('/error', (req, res) => {
  // Define status 404 e envia mensagem
  return res.status(404).send('Página não encontrada');
});

app.get('/error', (req, res) => {
  // Define status 500 e envia "Internal Server Error"
  return res.sendStatus(500);
});

app.get('/perfil', (req, res) => {
  // Renderiza template engine (EJS, Pug, etc)
  return res.render('perfil', { usuario: 'Maria' });
});

app.get('/imagem', (req, res) => {
  // Envia arquivo
  return res.sendFile(__dirname + '/foto.png');
});

app.get('/antiga-rota', (req, res) => {
  // Redireciona para uma noba rota
  return res.redirect('/nova-rota');
});

app.listen(PORT, () => {
  console.log(`Rodando na porta ${PORT}.`);
});
```




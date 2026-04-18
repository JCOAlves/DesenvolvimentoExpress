# Criando conexão com o Banco de Dados MySQL
O banco de dados MySQL é onde estão armazenados os dados de uma aplicação. Para realizar a conexão do banco no Express é preciso ter o MySQL instalado na máquina. Além disso, é necessario instalar o módulo MySQL.

## 1. Instalação do módulo MySQL
1. Inicialize o servidor MySQL.
2. Na pasta do projeto, instale o pacote MySQL:
   ```bash
   npm install mysql2
   ```
   O `mysql2` garante que sua aplicação seja resiliente e escalável. Para isso, usa o **Connection Pool** (Piscina de Conexões), que diferente de uma conexão única (que pode cair ou ficar ocupada), mantém várias conexões abertas e as "empresta" conforme as requisições chegam, devolvendo-as automaticamente depois.
3. Na pasta do projeto, crie uma subpasta chamada ```utils```.
4. Por motivos de sergurança nós não colocamos os dados de conexão do banco direto no código. Com isso, criamos um arquivo `.env` para armazenar dados da conexão, a qual deve ser ignorado no commit para o GitHub. Para usar o `.env`, instalamos a a depedência `dotenv` para acessar os dados:
   ```bash
   npm install dotenv
   ```
   Criamos o arquivo `.env` na raiz do projeto:
   ```bash
   DB_HOST=127.0.0.1
   DB_USER=root
   DB_PASS=sua_senha_secreta
   DB_NAME=restaurante_db
   DB_PORT=3306
   ```
5. Na sequência, dentro da subpasta ```utils```, crie um arquivo chamado ```db.js```:
   ```javascript
   import mysql from 'mysql2/promise';
   import dotenv from 'dotenv';

   dotenv.config(); // Acessa as credenciais do .env

   // Criamos o pool de conexões
   const pool = mysql.createPool({
      host: process.env.DB_HOST,
      user: process.env.DB_USER,
      password: process.env.DB_PASS,
      database: process.env.DB_NAME,
      port: process.env.DB_PORT,
      waitForConnections: true,
      connectionLimit: 10, // Máximo de 10 conexões simultâneas
      queueLimit: 0
   });

   // Teste rápido de conexão ao iniciar
   try {
      const connection = await pool.getConnection();
      console.log("Banco de dados MySQL conectado com sucesso!");
      connection.release(); // Importante: devolve a conexão para o pool
   } catch (error) {
      console.error("Erro ao conectar ao banco de dados:", error.message);
   }

   export default pool;
   ```

## 2. Retornando dados do banco
Para retornamos os dados do banco utilizamos o objeto da conexão `db`.
```javascript
import express from "express";
let app = express();
import db from '../utils/db.js';

/* Outras rotas definidas anteriormente... */
app.get('/autores/listar', async function(req, res) {
   await db.query('SELECT * FROM TbAutor', [], async function(erro, listagem){
      if (erro){
         return res.send(erro);
      }
      return res.send(listagem);
   });
});
```
- **`let db = require('../utils/db');`**: Importação do objeto da conexão MySQL
- **`router.get`**: Método GET HTTP
- **`'/autores/listar'`**: Rota GET do Express.
- **`db.query`**: Função Express de consulta ao banco. Você pode usar o comando `.execute` ao invez do `.query`, caso tenha que fazer consultas repetitivas.
- **`'SELECT * FROM TbAutor'`**: Script MySQL para seleção dos dados do banco.
- **`[]`**: Lista com os parâmetros para a consulta, não sendo necessario para funções do método GET.
- **`function(erro, listagem)`**: Função com os parâmetros `erro`, a qual guarda e retorna um erro na consulta, se tiver, e `listagem`, onde guarda uma lista JSON  com os dados do banco, podendo ser nomeada da forma que quiser.
- **`res.send(listagem)`**: Retorna uma lista com os dados do banco MySQL em formato JSON.

## 3. Exibição dos dados
Para exibir os dados do banco, podemos exibi-los no terminal ou em uma página HTML, enviando por `JSON` ou renderizando em um arquivo `ejs`, que são arquivos de template que permitem incorporar código JavaScript diretamente em HTML para criar páginas web dinâmicas no lado do servidor`.
```javascript
/* Lembrar das declarações: let express ... let db... let app... */
app.get('/autores/listar', async function(req, res) {
   let cmd = 'SELECT IdAutor, NoAutor, NoNacionalidade ';
   cmd += ' FROM TbAutor AS a INNER JOIN TbNacionalidade AS n';
   cmd += ' ON a.IdNacionalidade = n.IdNacionalidade';
   cmd += ' ORDER BY NoAutor';
   await db.query(cmd, [], function(erro, listagem){
      if (erro){
         return res.send(erro);
      }

      /* Renderiza os dados de listagem no ejs*/
      return res.render('autores-lista', {resultado: listagem});
   });
});
```
```ejs
<!DOCTYPE html>
<html>
   <head>
      <title>Listagem de Autores</title>
      <link rel='stylesheet' href='/stylesheets/style.css' />
   </head>
   <body>
      <h1>Listagem de Autores</h1>
      <p>
         <% for (item of resultado) {%>
            <%=item.IdAutor%> | <%=item.NoAutor%> | <%=item.NoNacionalidade%> <br>
         <%}%>
      </p>
   </body>
</html>
```

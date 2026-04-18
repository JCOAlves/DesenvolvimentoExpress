# Agrupando rotas no Express
Vimos que as rotas do Express podem ser passadas em um único arquivo, mas não é uma pratica interessante. 
Visto isso, é uma boa pratica dividir e agrupa rotas em arquivos separados, segundo as suas funcionalidades.

## 1. Agrupando rotas espeficas
Uma boa forma de agrupar rotas é de acordo com sua funcionalidade e os tipos de dados que manipulam. 
Um exemplo disso são rotas sobre autores de livros, em que listam e registram autores, manipulando assim os métodos
`GET` e `POST`. Assim, as rotas são arquivadas em um único arquivo. E esses arquivos são guardados em uma subpasta no projeto, 
podendo ser chamada de `Routes`. Em aplicações maiores, é comum modularizar ainda mais usando o objeto Router do Express.

```javascript
//Arquivo autoresRouter.js
import express from 'express';
import db from '../utils/db.js';

let router = express.Router(); //É usado como um "mini-aplicativo". Serve exclusivamente para organizar rotas.

// Aqui definimos apenas os "sub-caminhos"

//Lista autores
router.get('/listar', async function(req, res) {
  let cmd = 'SELECT IdAutor, NoAutor, NoNacionalidade';
  cmd += ' FROM TbAutor AS a INNER JOIN TbNacionalidade AS n';
  cmd += ' ON a.IdNacionalidade = n.IdNacionalidade ORDER BY NoAutor';
  await db.query(cmd, [], async function(erro, listagem){
    if (erro){
      return res.send(erro);
    }
    return res.render('autores-lista', {resultado: listagem});
  });
});

//Adicionar autores
router.get('/add', function(req, res) {
  return res.render('autores-add');
});

export default router;
```

## 2. Configurando rotas no arquivo `App.js`
Para configurar as rotas no arquivo principal `App.js` nos importamos as rotas do arquivos com as rotas.
```javascript
import express from 'express'
let app = express();
import autoresRouter from './routes/autores.js';

// Chama e aplica todas as rotas do arquivo autoresRouter sob o prefixo '/autores'
app.use('/autores', autoresRouter);

app.listen(3000, () => {
  console.log('Servidor rodando na porta 3000');
});
```
Chamamos as rotas a partir da rota principal, qual é a raiz para as outras rotas criadas.

## 3. Modulerizando funções nas rotas
Também é uma boa pratica colocar as funções das rotas em arquivos separados. 
A exportação e importação de funções para rotas no Express pode ser feita usando 
a sintaxe de módulos ES (`import` e `export`). 

1. Crie o arquivo de funções usando export (ex: controladores/usuarioController.js):
   ```javascript
   // controladores/usuarioController.js

   // Exemplo de função de controle para listar usuários
   export const listarUsuarios = (req, res) => {
     return res.status(200).send('Lista de usuários');
   };

   // Exemplo de função de controle para obter um único usuário
   export const obterUsuario = (req, res) => {
     const userId = req.params.id;
     return res.status(200).send(`Detalhes do usuário ${userId}`);
   };
   ```

2. Import as funções no arquivo das rotas usuariosRouter.js:
    ```js
    import express from 'express';
    let router = express.Router();
    import { listarUsuarios, obterUsuario } from './controladores/usuarioController.js';
  
    // Define as rotas usando as funções importadas
    router.get('/', listarUsuarios);
    router.get('/:id', obterUsuario);
    
    export default router;
    ```

3. Importe e use as funções no seu arquivo de rotas usando import (ex: app.js):
   ```javascript
   // app.js

   import express from 'express';
   import usuariosRouter from './routers/usuariosRouter.js';

   const app = express();

   // Use as rotas definidas
   app.get('/usuarios', usuariosRouter);

   app.listen(3000, () => {
      console.log('Servidor rodando na porta 3000');
   });
   ```

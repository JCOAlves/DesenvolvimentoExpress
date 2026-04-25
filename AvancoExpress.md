# Frameworks e ferramentas de desenvolvimento em projetos Espress
Agora que compreendemos os conceitos básicos para o desenvolvimento de projetos `Express`, 
podemos avança e utilizar ferramentas e frameworks que nós permitem criar projetos mais complexos e modernos.

## 1. Utilizando React JS
React JS (ou React) é uma biblioteca JavaScript de código aberto usada para construir interfaces de usuário (UIs) interativas e reativas, dividindo-as em componentes reutilizáveis que gerenciam seu próprio estado, facilitando a criação de aplicações web complexas e de página única (SPAs) de forma eficiente. Assm, usar Express.js com React.js permite criar aplicações full-stack robustas, onde o Express atua como backend (API RESTful) gerenciando dados e regras de negócio, enquanto o React gerencia a interface do usuário no frontend. 

Atualmente, o para criar uma interface de usuário se usa `Vite`, uma ferramenta de construção (build tool), 
extremamente rápida, criada para acelerar o desenvolvimento de aplicações `frontend`.

1. No terminal digite o comando:
    ```bash
    npm create vite@latest
    ```
2. No terminal aparecerá opções de ferramentas Frontend, incluindo o React.js, e linguagens de programação. Selecione a linguagem que preferir.
Se você quiser pular as perguntas e criar tudo em um único comando, use: 
    ```bash
    npm create vite@latest meu-app -- --template react-swc
    ```
3. Depois de terminal de configurar, digite no terminal:
    ```bash
    npm install react-router-dom
    ```
    Esse comando instala a bibliotec React responsavel pelo roteamento.

Ao abrir o projeto Frontend, você verá uma estrutura organizada:
- `index.html`: Fica na raiz (não na pasta public), o que permite ao Vite tratá-lo como parte do grafo de dependências.
- `src/main.jsx`: O ponto de entrada onde o React é montado no DOM.
- `vite.config.js`: Onde você adiciona plugins (como suporte para Tailwind ou aliases de pastas).

Para rodar o frontend digite no terminal: `npm run dev`. E assim o frontend será exercutado no endereço `localhost:5137`, onde `5137` é a porta padrão a aplicação.

## 2. Utilizando o **CORS**
Como você deve ter visto, o frontend React está rodando na porta `5137` enquando o backend na `3000`. Por padrão
os navegadores utilizam o **CORS** (Cross-Origin Resource Sharing), que é um mecanismo de segurança dos navegadores que bloqueia requisições entre domínios diferentes, como no caso do frontend e backend. No Express, usa-se o pacote `cors` como middleware para configurar cabeçalhos HTTP (`Access-Control-Allow-Origin`) que autorizam essas origens, permitindo o compartilhamento de recursos. Para isso:

1. No terminal, instale o pacote com o comando:
    ```bash
    npm install cors
    ```

2. Uso básico (Permitir todas as origens - Geralmente para desenvolvimento):
    ```js
    import express from "express";
    import cors from "cors";
    const app = express();

    app.use(cors()); // Habilita CORS para todas as rotas e origens
    ```

3. Uso configurado (Permitir origens específicas - Produção):
    ```js
    const corsOptions = {
        origin: 'http://localhost:5173', // URL do seu frontend (Vite/React)
        methods: ['GET', 'POST', 'PUT', 'DELETE'], // Métodos permitidos
        allowedHeaders: ['Content-Type', 'Authorization'], // Necessário para o JWT
        optionsSuccessStatus: 200
    };
    app.use(cors(corsOptions));
    ```
4. **CORS** em uma única rota:
    ```js
    app.get('/api/data', cors(corsOptions), (req, res) => {
        return res.json({ msg: 'Esta rota tem CORS específico' });
    });
    ```
## 3. Utilizando a sessão (`session`).
Uma sessão é um mecanismo para manter o estado de um usuário entre múltiplas requisições HTTP, armazenando dados temporários no
servidor e usando um cookie com ID de sessão no navegador para identificá-lo, permitindo recursos como login, carrinhos de
compras e preferências personalizadas, usando o middleware `express-session` para gerenciar isso de forma segura e eficiente. 

1. No terminal do seu projeto, instale o pacote: 
    ```bash
    npm install express-session
    ```
2. Importe o módulo e configure o middleware antes das suas rotas no arquivo principal (`App.js`):
    ```javascript
    import express from "express";
    import session from "express-session"; // Importa o módulo
    const app = express();

    // Configuração do middleware de sessão
    app.use(session({
        name: 'connect.sid', // Nome do cookie
        secret: 'uma-chave-secreta-muito-segura', // Essencial para assinar o cookie. Substitua por uma string segura
        resave: true, // Salva a sessão mesmo se não modificada
        saveUninitialized: true, // Salva sessão para usuários não logados, mas o ideal é deixar em false para cumprir leis de privacidade, pois evita a criação de cookies antes que o usuário faça login.
        cookie: {
            httpOnly: true, // Bloqueia o acesso ao cookie via JavaScript do navegador
            secure: true, // Em produção, use secure: true e HTTPOnly. Defina como true se usar HTTPS
            maxAge: 60 * 60 * 1000  // Tempo de vida do cookie em milissegundos (ex: 1 hora)
        } 
    }));

    // 1. Exemplo de rota para usar a sessão
    app.get('/', (req, res) => {
        // Se a sessão 'views' não existir, inicia com 0, senão, incrementa
        req.session.views = (req.session.views || 0) + 1;
        return res.send(`Você visitou esta página ${req.session.views} vezes.`);
    });

    // 2. Rota de Login
    app.post('/login', (req, res) => {
      const { username, password } = req.body;
      // Valide usuário/senha aqui
      if (username === 'usuario' && password === '123') {
        req.session.user = username; // Cria a sessão [2]
        return res.send('Login bem-sucedido');
      } else {
        return res.status(401).send('Credenciais inválidas');
      }
    });

    // 3. Rota Protegida (Exemplo)
    app.get('/dashboard', (req, res) => {
      if (req.session.user) {
        return res.send(`Olá ${req.session.user}, bem-vindo!`);
      } else {
        return res.status(401).send('Não autorizado');
      }
    });

    // 4. Rota de Logout
    app.post('/logout', (req, res) => {
      req.session.destroy((err) => { // Destrói a sessão [5]
        if (err) {
          return res.send('Erro ao sair');
        }
        res.clearCookie('connect.sid'); // Limpa o cookie da sessão
        return res.send('Logout realizado com sucesso');
      });
    });

    app.listen(3000, () => console.log('Servidor rodando na porta 3000'));
    ```

Como funciona
- Cookie: O express-session gera um ID único e o envia para o navegador do usuário em um cookie.
- Requisições subsequentes: O navegador envia o cookie de volta com cada requisição.
- req.session: O middleware usa o ID para carregar os dados da sessão (armazenados no servidor) no objeto req.session para você usar nas suas rotas.
- Segurança: Em produção, configure `cookie: { secure: true, httpOnly: true, sameSite: 'strict' }` para proteger contra ataques. 

Além disso, você precisa entender que o React não "guarda" a sessão,
quem faz isso é o **Navegador** através de um Cookie, e o servidor Express apenas reconhece esse cookie.
Para utilizar o session você precisa configurar o backend e o frontend:

1. Configuração no Backend (Express)
    ```js
    // No app.js
    app.use(cors({
        origin: 'http://localhost:5173', // URL do seu React
        credentials: true                // PERMITE enviar cookies/sessão
    }));

    app.use(session({
        name: 'sistemaWeb', 
        secret: 'sua_chave_secreta',
        resave: false,
        saveUninitialized: false,
        cookie: { 
            secure: false,    // Mantenha FALSE no localhost
            httpOnly: true,   // Segurança: impede que o JS do front leia o cookie
            sameSite: 'lax'   // Necessário para navegadores modernos
        }
    }));
    ```

2. Configuração no Frontend (React)
    Se você usar o `fetch` comum, o navegador vai bloquear o cookie por segurança.
    Você precisa avisar ao fecth que ele deve incluir as "credenciais" em todas as chamadas.
    ```js
    //Metodo GET
    async function GET(rota){ 
        try { 
            let resposta = await fetch(rota, {credentials: 'include'}); // Permite que o navegador pegue cookies de sessão
            const dados = await resposta.json();
            if("mensagemServidor" in dados){
                return dados["mensagemServidor"];
            }else{
                return dados;
            }
    
        } catch (error) {
            console.error(`Erro na busca de dados: ${error.message || error}`);
            return `Erro na busca de dados: ${error.message || error}`;
        }
    };
    
    //Metodo POST
    async function POST(rota, objeto) {
        try {
            const objetoJSON = JSON.stringify(objeto);
            let resposta = await fetch(rota, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: objetoJSON,
                credentials: 'include' // Permite que o navegador envie cookies de sessão
            });
            resposta = await resposta.json();
            return resposta;
        }
        catch (error) {
            console.error(`Erro no envio de dados: ${error.message || error}`);
            return `Erro no envio de dados: ${error.message || error}`;
        }
    };
    
    //Metodo PUT
    async function PUT(rotaEspecifica, objeto) {
        try{
            const objetoJSON = JSON.stringify(objeto);
            let resposta = await fetch(rotaEspecifica, {
                method: 'PUT',
                headers: { 'Content-Type': 'application/json' },
                body: objetoJSON,
                credentials: 'include' // Permite que o navegador envie cookies de sessão
            });
            resposta = await resposta.json();
            return resposta;
        }
        catch(error){
            console.error(`Erro na atualização de dados: ${error.message || error}`);
            return `Erro na atualização de dados: ${error.message || error}`;
        }
    };
    
    //Metedo DELETE
    async function DELETE(rotaEspecifica) {
        try{
            let resposta = await fetch(rotaEspecifica, {
                method: 'DELETE',
                headers: { 'Content-Type': 'application/json' },
                credentials: 'include' // Permite que o navegador envie cookies de sessão
            });
            resposta = await resposta.json();
            return resposta;
        }
        catch(error){
            console.error(`Erro na excluição de dados: ${error.message || error}`);
            return `Erro na excluição de dados: ${error.message || error}`;
        }
    };
    ```



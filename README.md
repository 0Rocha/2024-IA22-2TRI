### 2024-IA22-2TRI

# Iniciando um projeto Node.js com TypeScript

###  abra o CodeSpace no Github e abra o *`Terminal`* (Ctrl+shift+U) e insira  os passos abaixo.

```ts
npm init -y
npm install express cors sqlite3 sqlite
npm install --save-dev typescript nodemon ts-node @types/express @types/cors
npx tsc --init
mkdir src
touch src/app.ts
```
<div style="display: incline_block"><br/>
  <img align="center> alt="c" src="ignore/imagens/IMG-20240808-WA0012.jpg" />
</div>



# Configurando o tsconfig.json

### Mude a linha *`"outDir": "./"`* (localizada entre as linhas 50/60) , para *`"outDir": "./dist"`*, seu arquivo de configuração do compilador do TypeScript ficará mais ou menos assim.

### dicas:
#### Ctrl+F procura palavras
#### Ctrl+G procura linhas


```ts
{
  "compilerOptions": {
    "target": "ES2017",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  }
}
```
# Configurando o package.json

### Adicione o seguinte script: ("dev": "npx nodemon src/app.ts" ) ao seu *`package.json`* na variavel  *`"scripts":`*  ficará mais ou menos assim

```ts
scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
      "dev": "npx nodemon src/app.ts"
  },
```

# Criando arquivo inicial do servidor

### Adicione o seguinte código ao arquivo *`src/app.ts`*

```ts
import express from 'express';
import cors from 'cors';

const port = 3333;
const app = express();

app.use(cors());
app.use(express.json());

app.get('/', (req, res) => {
  res.send('Hello World');
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```

<div style="display: incline_block"><br/>
  <img align="center> alt="c" src="/workspaces/2024-IA22-2TRI/ignore/imagens/IMG-20240808-WA0015.jpg" />
</div>


# Inicializando o servidor

### digite o seguinte codigo no *`terminal`*: 

``` ts
npm run dev
```
<div style="display: incline_block"><br/>
  <img align="center> alt="c" src="/workspaces/2024-IA22-2TRI/ignore/imagens/IMG-20240808-WA0014.jpg" />
</div>
#### Se tudo ocorrer bem, você verá a mensagem Server running on port 3333 no terminal.

# Testando o servidor

### Abra o navegador e acesse http://localhost:3333, você verá a mensagem Hello World.
### insira */users* ao final do link
# Configurando o banco de dados

### Crie um arquivo *`database.ts`* dentro da pasta src e adicione o seguinte código.

``` ts
import { Database, open } from 'sqlite';
import sqlite3 from 'sqlite3';

let instance: Database | null = null;

export async function connect() {
  if (instance) return instance;

  const db = await open({
     filename: './src/database.sqlite',
     driver: sqlite3.Database
   });
  
  await db.exec(`
    CREATE TABLE IF NOT EXISTS users (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      name TEXT,
      email TEXT
    )
  `);

  instance = db;
  return db;
}
```

# Adicionando o banco de dados ao servidor

### no arquivo _`App.ts`_ insira o seguinte código:

``` ts
import express from 'express';
import cors from 'cors';
import { connect } from './database';

const port = 3333;
const app = express();

app.use(cors());
app.use(express.json());

app.get('/', (req, res) => {
  res.send('Hello World');
});

app.post('/users', async (req, res) => {
  const db = await connect();
  const { name, email } = req.body;

  const result = await db.run('INSERT INTO users (name, email) VALUES (?, ?)', [name, email]);
  const user = await db.get('SELECT * FROM users WHERE id = ?', [result.lastID]);

  res.json(user);
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```
# Testando a inserção de dados
### clique "Ctrl+Shift+X" e instale extenção "Rest Client", após isso pressione "Ctrl+Shift+E". Crie um novo arquivo chamado `Teste.http` dentro desse arquivo insira o código:
<div style="display: incline_block"><br/>
  <img align="center> alt="c" src="/workspaces/2024-IA22-2TRI/ignore/imagens/IMG-20240808-WA0013.jpg" />
</div>
### insira um *_`Insira um nome e email`_*

``` ts
POST http://localhost:3333/users HTTP/1.1
Content-Type: application/json



{
  "name": " John Doe ",
  "email": " "
}
```
### Se tudo ocorrer bem, você verá a resposta com o usuário inserido.

``` ts
{
  "id": 1,
  "name": " John Doe",
  "email": "
}
```

# Listando os usuários
## Adicione a rota `/users` ao *`app.ts`*

``` ts
app.get('/users', async (req, res) => {
  const db = await connect();
  const users = await db.all('SELECT * FROM users');

  res.json(users);
}
);

```

# Editando um usuário
## Adicione a rota `/users/:id` ao *`app.ts`*.

``` ts 
app.put('/users/:id', async (req, res) => {
  const db = await connect();
  const { name, email } = req.body;
  const { id } = req.params;

  await db.run('UPDATE users SET name = ?, email = ? WHERE id = ?', [name, email, id]);
  const user = await db.get('SELECT * FROM users WHERE id = ?', [id]);

  res.json(user);
});

```

# Deletando um usuário
## Adicione a rota `/users/:id` ao *`app.ts`*

``` ts

app.delete('/users/:id', async (req, res) => {
  const db = await connect();
  const { id } = req.params;

  await db.run('DELETE FROM users WHERE id = ?', [id]);

  res.json({ message: 'User deleted' });
});

```

# crie um novo arquivo chamado *`test.http`*
## insira o codigo a seguir:

``` ts
POST http://localhost:3333/users HTTP/1.1
content-type: application/json

{
  "name": "John Doe",
  "email": "johndoe@mail.com"
}

####

PUT http://localhost:3333/users/2 HTTP/1.1
content-type: application/json

{
  "name": "John Doe Updated",
  "email": "johndoe@mail.com"
}

####

DELETE http://localhost:3333/users/1 HTTP/1.1

```

### crie uma nova pasta chamada  *`public`*

### agora dentro da posta crie o *index.html* e insira o código:

``` ts
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>

<body>
  <form>
    <input type="text" name="name" placeholder="Nome">
    <input type="email" name="email" placeholder="Email">
    <button type="submit">Cadastrar</button>
  </form>

  <table>
    <thead>
      <tr>
        <th>Id</th>
        <th>Name</th>
        <th>Email</th>
        <th>Ações</th>
      </tr>
    </thead>
    <tbody>
      <!--  -->
    </tbody>
  </table>

  <script>
    // 
    const form = document.querySelector('form')

    form.addEventListener('submit', async (event) => {
      event.preventDefault()

      const name = form.name.value
      const email = form.email.value

      await fetch('/users', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ name, email })
      })

      form.reset()
      fetchData()
    })

    // 
    const tbody = document.querySelector('tbody')

    async function fetchData() {
      const resp = await fetch('/users')
      const data = await resp.json()

      tbody.innerHTML = ''

      data.forEach(user => {
        const tr = document.createElement('tr')
        tr.innerHTML = `
          <td>${user.id}</td>
          <td>${user.name}</td>
          <td>${user.email}</td>
          <td>
            <button class="excluir">excluir</button>
            <button class="editar">editar</button>
          </td>
        `

        const btExcluir = tr.querySelector('button.excluir')
        const btEditar = tr.querySelector('button.editar')

        btExcluir.addEventListener('click', async () => {
          await fetch(`/users/${user.id}`, { method: 'DELETE' })
          tr.remove()
        })

        btEditar.addEventListener('click', async () => {
          const name = prompt('Novo nome:', user.name)
          const email = prompt('Novo email:', user.email)

          await fetch(`/users/${user.id}`, {
            method: 'PUT',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ name, email })
          })

          fetchData()
        })

        tbody.appendChild(tr)
      })
    }

    fetchData()
  </script>
</body>

</html>

```
# na pasta *Src* em *`app.ts`*insira o código a seguir
### "app.use(express.static(__dirname + '/../public'))"
 ## ficará assim: 
``` ts 
import express from 'express';
import cors from 'cors';
import { connect } from './database';

const port = 3333;
const app = express();

app.use(cors())
app.use(express.json())
app.use(express.static(__dirname + '/../public'))

 ```

 # agora como ultimo passo salve e comite o código no terminal
 
 ``` ts
git add .
git commit -m ...
<<<<<<< HEAD
git push origin maingit add .
```
=======
git push origin maingit add 

```
>>>>>>> 019cee3 (...)

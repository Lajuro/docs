# Documentação da API de Pokémon

Esta documentação descreve os passos para criar uma API Node.js que busca dados de Pokémon da PokeAPI.

## Passo 1: Configuração do Projeto

1. Crie uma nova pasta para o projeto e navegue até ela:
   ```
   mkdir pokemon-api
   cd pokemon-api
   ```

2. Inicialize um novo projeto Node.js:
   ```
   npm init -y
   ```

3. Instale as dependências necessárias:
   ```
   npm install express axios
   npm install --save-dev jest supertest
   ```

4. Crie a seguinte estrutura de pastas e arquivos:
   ```
   pokemon-api/
   ├── src/
   │   ├── routes/
   │   │   └── pokemon.js
   │   └── server.js
   ├── tests/
   │   └── pokemon.test.js
   └── package.json
   ```

## Passo 2: Configuração do package.json

Atualize o arquivo `package.json` para incluir os scripts de início e teste:

```json
{
  "name": "pokemon-api",
  "version": "1.0.0",
  "description": "API para buscar dados de Pokémon",
  "main": "src/server.js",
  "scripts": {
    "start": "node src/server.js",
    "test": "jest --detectOpenHandles"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "axios": "^1.6.0",
    "express": "^4.18.2"
  },
  "devDependencies": {
    "jest": "^29.7.0",
    "supertest": "^6.3.3"
  }
}
```

## Passo 3: Criação da Rota

No arquivo `src/routes/pokemon.js`, implemente a rota para buscar dados de Pokémon:

```javascript
const express = require('express');
const axios = require('axios');

const router = express.Router();

router.get('/:name', async (req, res) => {
  try {
    const pokemonName = req.params.name.toLowerCase();
    const response = await axios.get(`https://pokeapi.co/api/v2/pokemon/${pokemonName}`);
    
    const pokemonData = {
      name: response.data.name,
      id: response.data.id,
      height: response.data.height,
      weight: response.data.weight,
      types: response.data.types.map(type => type.type.name)
    };
    
    res.json(pokemonData);
  } catch (error) {
    if (error.response && error.response.status === 404) {
      res.status(404).json({ error: 'Pokémon não encontrado' });
    } else {
      res.status(500).json({ error: 'Erro ao buscar dados do Pokémon' });
    }
  }
});

module.exports = router;
```

## Passo 4: Configuração do Servidor

No arquivo `src/server.js`, configure o servidor Express:

```javascript
const express = require('express');
const pokemonRoutes = require('./routes/pokemon');

const app = express();
const PORT = process.env.PORT || 3000;

app.use('/pokemon', pokemonRoutes);

let server;

function startServer() {
  server = app.listen(PORT, () => {
    console.log(`Servidor rodando na porta ${PORT}`);
  });
  return server;
}

function closeServer() {
  return new Promise((resolve) => {
    server.close(() => {
      resolve();
    });
  });
}

if (require.main === module) {
  startServer();
}

module.exports = { app, startServer, closeServer };
```

## Passo 5: Implementação dos Testes

No arquivo `tests/pokemon.test.js`, implemente os testes unitários:

```javascript
const request = require('supertest');
const axios = require('axios');
const { app, startServer, closeServer } = require('../src/server');

jest.mock('axios');

describe('Testes da rota Pokémon', () => {
  let server;

  beforeAll(() => {
    server = startServer();
  });

  afterAll(async () => {
    await closeServer();
  });

  afterEach(() => {
    jest.resetAllMocks();
  });

  it('deve retornar os dados de um Pokémon válido', async () => {
    const mockPokemonData = {
      data: {
        name: 'pikachu',
        id: 25,
        height: 4,
        weight: 60,
        types: [{ type: { name: 'electric' } }]
      }
    };

    axios.get.mockResolvedValueOnce(mockPokemonData);

    const response = await request(app).get('/pokemon/pikachu');

    expect(response.status).toBe(200);
    expect(response.body).toEqual({
      name: 'pikachu',
      id: 25,
      height: 4,
      weight: 60,
      types: ['electric']
    });

    expect(axios.get).toHaveBeenCalledWith('https://pokeapi.co/api/v2/pokemon/pikachu');
  });

  it('deve retornar 404 para um Pokémon inexistente', async () => {
    axios.get.mockRejectedValueOnce({ 
      response: { status: 404 } 
    });

    const response = await request(app).get('/pokemon/naoexiste');

    expect(response.status).toBe(404);
    expect(response.body).toEqual({ error: 'Pokémon não encontrado' });

    expect(axios.get).toHaveBeenCalledWith('https://pokeapi.co/api/v2/pokemon/naoexiste');
  });

  it('deve retornar 500 para erro na API', async () => {
    axios.get.mockRejectedValueOnce(new Error('API Error'));

    const response = await request(app).get('/pokemon/pikachu');

    expect(response.status).toBe(500);
    expect(response.body).toEqual({ error: 'Erro ao buscar dados do Pokémon' });

    expect(axios.get).toHaveBeenCalledWith('https://pokeapi.co/api/v2/pokemon/pikachu');
  });
});
```

## Passo 6: Executando a API

Para iniciar o servidor:
```
npm start
```

O servidor estará rodando em `http://localhost:3000`.

## Passo 7: Executando os Testes

Para executar os testes:
```
npm test
```

## Uso da API

Para buscar dados de um Pokémon, faça uma requisição GET para:
```
http://localhost:3000/pokemon/{nome-do-pokemon}
```

Exemplo:
```
http://localhost:3000/pokemon/pikachu
```

A resposta será um JSON com os dados básicos do Pokémon.

## Conclusão

Esta API fornece uma interface simples para buscar dados básicos de Pokémon da PokeAPI. Ela inclui tratamento de erros e testes unitários para garantir seu funcionamento correto.

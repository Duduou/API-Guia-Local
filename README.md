O documento do projeto pode ser encontrado em: **Guia Local, Uma Aplicação para Catalogação de Estabelecimentos Locais.pdf**

# 📘 Especificações da API Guia\_Local

Para criação do usuário admin, crie o arquivo `seed.ts` na pasta do prisma, seu contedo deve ser:

<pre><code>
import { PrismaClient } from '@prisma/client';
import bcrypt from 'bcryptjs';

const prisma = new PrismaClient();

async function main() {
  const hashed = await bcrypt.hash('(TROCAR POR SENHA DO ADMIN)', 10);
  const admin = await prisma.usuario.create({
    data: {
      nome: '(TROCAR POR NOME DO ADMIN)',
      email: '(TROCAR POR EMAIL DO ADMIN)',
      senha: hashed,
      admin: true,
    }
  });
  console.log('Usuário admin criado:', admin);
}

main()
  .catch((e) => {
    console.error(e);
    process.exit(1);
  })
  .finally(() => {
    prisma.$disconnect();
  });
</code></pre>

E o arquivo `.env` deve ser adicionado na raiz do projeto e deve definir os seguintes atributos:

- `DATABASE_URL="mysql://USUARIO:SENHA@localhost:3306/ephonebook"`
Url para acesso ao banco de dados MySQL.

- `JWT_SECRET="chaveSuperSecreta"`
Chave para criptografar e descriptografar os JWT.

- `PORT=3000`
Porta do pc onde rodará a API.

- `HOST_PUBLIC="localhost"`
Endreço ip onde rodará a API, localhost caso seja local.

Após isso rodar o comando `npm intall` no prompt e quando pronto rodar `npx prisma migrate dev --name init`.

Então após realizada a instalação, basta rodar `npm run dev` para subir o servidor;

---

## 🔐 Autenticação

### POST /register

* Cria um novo usuário comum.
* **Body:**

```json
{
  "nome": "João",
  "email": "joao@email.com",
  "senha": "senha123"
}
```

* **Retorno:**

```json
{
  "id": 1,
  "nome": "João",
  "email": "joao@email.com"
}
```

---

### POST /login

* Autentica e retorna um token JWT.
* **Body:**

```json
{
  "email": "joao@email.com",
  "senha": "senha123"
}
```

* **Retorno:**

```json
{
  "token": "<jwt-token>"
}
```

---

## 👤 Usuário

### POST /usuarios/foto

* Envia a foto de perfil do usuário.
* **Header:** Authorization: Bearer <token>
* **Form-Data:**

  * **arquivo**: (imagem)
* **Retorno:**

```json
{
  "id": 1,
  "foto": "http://localhost:3000/uploads/abc.jpg"
}
```

### GET /usuarios/foto

* Retorna a foto do usuário autenticado.
* **Header:** Authorization: Bearer <token>
* **Retorno:**

```json
{
  "foto": "http://localhost:3000/uploads/abc.jpg"
}
```

---

## 🏢 Estabelecimento

### POST /estabelecimentos

* Cria um novo estabelecimento (autenticado).
* **Header:** Authorization: Bearer <token>
* **Body:**

```json
{
  "nome": "Sorveteria A",
  "descricao": "O maior buffet de sorvete da região , taças decoradas e café com tortas e salgados.",
  "endereco": "Av. B, 1495 - Vila R",
  "longitude": "-29.900000000",
  "latitude": "-51.600000000",
  "whatsapp": "5551999999999",
  "facebook": "https://www.facebook.com/sorveteria",
  "instagram": "sorveteria"
}
```

* **Retorno:** dados do estabelecimento criado

### GET /estabelecimentos

* Lista todos os estabelecimentos (autenticado).
* **Header:** Authorization: Bearer <token>

### POST /estabelecimentos/\:id/foto-perfil

* Atualiza a foto de perfil do estabelecimento (autenticado).
* **Header:** Authorization: Bearer <token>
* **Form-Data:**

  * **arquivo**: (imagem)
* **Retorno:** dados do estabelecimento com `fotoPerfil`

### POST /estabelecimentos/\:id/fotos/upload

* Faz upload de imagem para a galeria (autenticado).
* **Header:** Authorization: Bearer <token>
* **Form-Data:**

  * **arquivo**: (imagem)
* **Retorno:** dados da foto associada

## Subrotas: Telefones

### **POST /estabelecimentos/\:id/telefones** – Adiciona telefone (autenticado)
* **Header:** Authorization: Bearer <token>

### **GET /estabelecimentos/\:id/telefones** – Lista telefones


### **DELETE /telefones/\:id** – Remove telefone (autenticado)
* **Header:** Authorization: Bearer <token>

## Subrotas: Emails

### **POST /estabelecimentos/\:id/emails** – Adiciona email (autenticado)
* **Header:** Authorization: Bearer <token>

### **GET /estabelecimentos/\:id/emails** – Lista emails

### **DELETE /emails/\:id** – Remove email (autenticado)
* **Header:** Authorization: Bearer <token>

## Subrotas: Horário

### **POST /estabelecimentos/\:id/horario** – Cria horário de funcionamento (autenticado)
* **Header:** Authorization: Bearer <token>

### **GET /estabelecimentos/\:id/horario** – Consulta horário

### **PUT /estabelecimentos/\:id/horario** – Atualiza horário (autenticado)
* **Header:** Authorization: Bearer <token>

### **DELETE /estabelecimentos/\:id/horario** – Remove horário (autenticado)
* **Header:** Authorization: Bearer <token>

---

## 🗂️ Categoria

### POST /categorias

* Cria nova categoria (admin).
* **Body:**

```json
{
  "nome": "Saúde"
}
```

### GET /categorias

* Lista todas as categorias.

### POST /categorias/\:id/imagem

* Atualiza a imagem da categoria.
* **Header:** Authorization: Bearer <admin-token>
* **Form-Data:**

  * **arquivo**: (imagem)
* **Retorno:** dados da categoria com imagem atualizada

### POST /categorias/\:id/tags

* Associa uma ou mais tags a uma categoria.
* **Body:**

```json
{
  "tagIds": [1, 2]
}
```

### DELETE /categorias/\:id/tags/\:tagId

* Remove a associação de uma tag da categoria.

### GET /categorias/\:id/tags

* Lista todas as tags associadas à categoria.

---

## 🏷️ Tag

### POST /tags

* Cria nova tag (admin).
* **Body:**

```json
{
  "nome": "24h"
}
```

### GET /tags

* Lista todas as tags.

---

## ⭐ Favoritos

### POST /favoritos/\:id
* **Header:** Authorization: Bearer <token>

* Adiciona o estabelecimento aos favoritos do usuário autenticado.

### DELETE /favoritos/\:id
* **Header:** Authorization: Bearer <token>

* Remove o estabelecimento dos favoritos.

### GET /favoritos
* **Header:** Authorization: Bearer <token>

* Lista os estabelecimentos favoritos do usuário autenticado.

---

## 🧷 Associação

### POST /estabelecimentos/\:id/categorias
* **Header:** Authorization: Bearer <token>

* Associa uma ou mais categorias (autenticado).
* **Body:**

```json
{
  "categoriaIds": [1, 2]
}
```

### DELETE /estabelecimentos/\:id/categorias/\:categoriaId

* Remove a associação com a categoria (autenticado).
* **Header:** Authorization: Bearer <token>

### POST /estabelecimentos/\:id/tags

* Associa uma ou mais tags (autenticado).
* **Header:** Authorization: Bearer <token>
* **Body:**

```json
{
  "tagIds": [3, 4]
}
```

### DELETE /estabelecimentos/\:id/tags/\:tagId

* Remove a associação com a tag (autenticado)
* **Header:** Authorization: Bearer <token>

---

## 🔍 Busca pública

### GET /buscar?term=padaria

* Busca geral (parcial, sem autenticação)

### GET /buscarNome?nome=padaria

* Busca por nome (parcial, sem autenticação)

### GET /buscar/categorias/\:id

* Lista estabelecimentos de uma categoria

### GET /buscar/tags/\:id

* Lista estabelecimentos de uma tag

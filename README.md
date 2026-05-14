# DemandFlow — Gestão de Demandas de Suporte

Sistema para organizar demandas do time de suporte com três níveis de visibilidade: **Interna**, **Privada** e **Restrita**.

---

## Funcionalidades

| Funcionalidade | Colaborador | Administrador |
|---|---|---|
| Ver demandas internas | ✅ | ✅ |
| Criar demandas internas | ✅ | ✅ |
| Ver demandas privadas (próprias) | ✅ | ✅ |
| Ver demandas restritas | ❌ | ✅ |
| Criar demandas restritas | ❌ | ✅ |
| Alterar status de qualquer demanda | ❌ | ✅ |
| Alterar status das próprias demandas | ✅ | ✅ |
| Comentar em demandas | ✅ | ✅ |
| Gerenciar usuários | ❌ | ✅ |

---

## Pré-requisitos

- Node.js 18+
- Conta no [Firebase](https://firebase.google.com)
- [Firebase CLI](https://firebase.google.com/docs/cli): `npm install -g firebase-tools`
- Conta no [GitHub](https://github.com)

---

## Passo a Passo: Configurar o Firebase

### 1. Criar projeto no Firebase

1. Acesse [console.firebase.google.com](https://console.firebase.google.com)
2. Clique em **Adicionar projeto**
3. Dê um nome (ex: `demandflow-empresa`)
4. Desative o Google Analytics (opcional) e crie o projeto

### 2. Ativar autenticação por e-mail/senha

1. No menu esquerdo, clique em **Authentication**
2. Aba **Sign-in method**
3. Clique em **E-mail/senha** → Ative → Salvar

### 3. Criar banco de dados Firestore

1. No menu esquerdo, clique em **Firestore Database**
2. Clique em **Criar banco de dados**
3. Escolha **Começar no modo de produção** (as regras de segurança já estão no projeto)
4. Escolha a região mais próxima (ex: `us-east1` ou `southamerica-east1` para Brasil)

### 4. Obter as credenciais do app

1. Vá em **Configurações do projeto** (ícone de engrenagem)
2. Role até **Seus apps** → clique em **</>** (Web)
3. Registre o app com um apelido (ex: `demandflow-web`)
4. Copie o objeto `firebaseConfig`

### 5. Configurar o arquivo `src/firebase.js`

Abra o arquivo `src/firebase.js` e substitua com suas credenciais:

```js
const firebaseConfig = {
  apiKey: "AIzaSy...",
  authDomain: "demandflow-empresa.firebaseapp.com",
  projectId: "demandflow-empresa",
  storageBucket: "demandflow-empresa.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc123"
}
```

---

## Passo a Passo: Configurar o GitHub e rodar o projeto

### 1. Clonar / inicializar o repositório

```bash
# Clone este projeto ou inicialize um novo
git init
git add .
git commit -m "feat: projeto inicial DemandFlow"

# Crie um repositório no GitHub e conecte
git remote add origin https://github.com/SEU_USUARIO/demandflow.git
git push -u origin main
```

### 2. Instalar dependências

```bash
npm install
```

### 3. Rodar localmente

```bash
npm run dev
# Acesse: http://localhost:5173
```

### 4. Criar o primeiro usuário administrador

Como o sistema não tem tela de cadastro público (segurança), o primeiro admin precisa ser criado manualmente:

1. Rode o projeto localmente: `npm run dev`
2. No Firebase Console → Authentication → Usuários → **Adicionar usuário**
3. Informe o e-mail e senha do primeiro admin
4. No Firebase Console → Firestore → **Criar documento** na coleção `users`:
   - ID do documento: UID do usuário criado (copie do Authentication)
   - Campos:
     - `name` (string): Nome Completo do Admin
     - `email` (string): email@empresa.com
     - `role` (string): `admin`
     - `createdAt` (string): data atual

5. Agora faça login com esse admin e use a interface para criar outros usuários.

---

## Deploy no Firebase Hosting

```bash
# 1. Autentique no Firebase
firebase login

# 2. Associe o projeto
firebase use --add
# Selecione seu projeto e dê um alias (ex: default)

# 3. Deploy completo (build + hosting + firestore rules)
npm run deploy

# Ou separadamente:
npm run build        # Gera a pasta dist/
firebase deploy      # Faz o deploy de tudo
```

Após o deploy, seu app estará disponível em:
`https://SEU-PROJETO.web.app`

---

## Regras de Segurança do Firestore

As regras estão no arquivo `firestore.rules` e garantem:

- **Demandas internas**: qualquer usuário autenticado pode ler e criar
- **Demandas privadas**: somente o criador pode ler/editar
- **Demandas restritas**: somente usuários com `role == 'admin'`
- **Comentários**: seguem a mesma regra da demanda pai
- **Usuários**: somente admins criam; cada um edita o próprio perfil

---

## Estrutura do Firestore

```
users/
  {uid}
    name: "João Silva"
    email: "joao@empresa.com"
    role: "collaborator" | "admin"
    createdAt: "2024-01-15T10:00:00Z"

demands/
  {demandId}
    title: "Cliente ABC — Erro NF-e"
    description: "Detalhes do problema..."
    type: "internal" | "private" | "restricted"
    priority: "high" | "medium" | "low"
    status: "open" | "in_progress" | "resolved"
    createdBy: {uid}
    createdByName: "João Silva"
    createdAt: Timestamp
    updatedAt: Timestamp

    comments/
      {commentId}
        text: "Verificando o servidor..."
        authorName: "Carlos Admin"
        authorId: {uid}
        createdAt: Timestamp
```

---

## Dúvidas Frequentes

**P: Como adicionar mais colaboradores?**
R: O admin faz login, clica em "Gerenciar usuários" no topo e preenche o formulário.

**P: Posso hospedar em outro lugar além do Firebase?**
R: Sim. Rode `npm run build` e hospede a pasta `dist/` em Netlify, Vercel, etc. O Firebase (Auth + Firestore) continua funcionando normalmente.

**P: O sistema funciona em tempo real?**
R: Sim. Usando Firestore com `onSnapshot`, todos os usuários veem as atualizações instantaneamente sem precisar recarregar a página.

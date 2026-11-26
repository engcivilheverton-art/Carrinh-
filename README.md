# Carrinhô

Planner de compras com login, listas por título e dia, cadastro de produtos com histórico de preços e **listas compartilhadas entre usuários** (ideia para casais).
Tecnologias: um único `index.html` + Firebase (Authentication e Firestore) + Netlify. Não tem etapa de build.

## Arquivos

| Arquivo | Para que serve |
|---|---|
| `index.html` | O app inteiro |
| `firebase-config.js` | Você cola aqui a configuração do seu projeto Firebase |
| `firestore.rules` | Regras de segurança do banco (obrigatório publicar) |
| `firebase.json`, `.firebaserc` | Usados só se publicar as regras pela Firebase CLI |
| `netlify.toml` | Configuração de hospedagem do Netlify |
| `package.json` | Atalhos: `npm start` (teste local) e `npm run deploy:rules` |

## 1. Firebase

1. Em https://console.firebase.google.com crie um projeto (o Google Analytics pode ficar desligado).
2. **Build > Authentication > Começar** e ative **E-mail/senha** e **Google**.
3. **Build > Firestore Database > Criar banco de dados** (modo de produção; região sugerida: `southamerica-east1`).
4. **Configurações do projeto > Seus apps > ícone `</>`** (Web): registre o app e copie o objeto `firebaseConfig`.
5. Cole os valores em `firebase-config.js`.
6. Publique as regras (obrigatório; sem elas o banco fica fechado):
   - **Console:** Firestore Database > aba **Regras** > cole o conteúdo de `firestore.rules` > **Publicar**; ou
   - **CLI:** `npm i -g firebase-tools`, depois `firebase login`, edite o `.firebaserc` com o ID do projeto e rode `npm run deploy:rules`.

## 2. GitHub

```bash
git init
git add .
git commit -m "Carrinhô: versão inicial"
git branch -M main
git remote add origin https://github.com/SEU-USUARIO/carrinho.git
git push -u origin main
```

O `firebase-config.js` pode ir para o repositório: ele não é secreto. Não suba chaves de conta de serviço nem arquivos `.env`.

## 3. Netlify

1. Em https://app.netlify.com: **Add new site > Import an existing project > GitHub** e escolha o repositório.
2. Deixe o comando de build **em branco** e o diretório de publicação como **`.`** (o `netlify.toml` já informa isso).
3. A cada `git push` na `main` o site é publicado de novo.
   (Para um teste rápido sem GitHub, arraste a pasta em https://app.netlify.com/drop.)

## 4. Autorizar o domínio no Firebase

Em **Authentication > Configurações > Domínios autorizados**, adicione o endereço do site (por exemplo `seu-app.netlify.app` e o domínio próprio, se houver). Sem isso, o login com Google dá erro.

## 5. Testar o compartilhamento

1. Crie duas contas (por exemplo, a sua e a de outra pessoa). Quem entra com **e-mail e senha** recebe um link de confirmação; quem entra com **Google** já vem confirmado.
2. Na conta A: **Planejar lista > Criar lista de exemplo** (ou crie uma lista).
3. Aba **Colaboração**: escolha a lista, digite o e-mail da conta B e clique em **Compartilhar**.
   O app só aceita e-mails que já têm conta (e e-mail confirmado).
4. Na conta B: aba **Colaboração > Convites recebidos > Aceitar**.
5. A lista passa a aparecer para os dois, marcada como **Compartilhada**, e os itens e preços sincronizam em tempo real.

## Como os dados são organizados

- `lists/{lista}`: título, dia, dono, participantes (`members`, `memberEmails`) e convites pendentes (`invitedEmails`).
- `lists/{lista}/items/{item}`: um documento por item. Dois celulares editando itens diferentes ao mesmo tempo não se sobrescrevem.
- `userDirectory/{e-mail}`: registra quem tem conta confirmada, para validar o convite sem expor a lista de usuários.
- **Produtos:** o cadastro é montado a partir dos itens das suas listas (o identificador do produto é o nome sem acento e em minúsculas). O mesmo item em listas diferentes forma o histórico de preços.
- Funciona com sinal fraco: as alterações ficam guardadas no aparelho e sincronizam quando a conexão volta.

## Limites e cuidados

- **Quem edita:** qualquer participante pode alterar título, dia, convidar, remover participantes e editar itens. Em listas de casal isso é intencional.
- **Dono:** só o dono exclui a lista (para todos). Quem recebeu a lista sai dela pela opção "Sair desta lista".
- **Mesmo campo ao mesmo tempo:** se duas pessoas editarem o mesmo campo do mesmo item juntas, vale a última alteração.
- **Consulta de e-mail:** qualquer usuário confirmado consegue perguntar se um e-mail específico está cadastrado (necessário para o convite). Não existe como listar todos os usuários.
- **Convites exigem e-mail confirmado**, para ninguém aceitar convite com um e-mail que não é seu.
- **Produto renomeado:** itens com nomes diferentes (por exemplo "Leite" e "Leite integral") são produtos diferentes e têm históricos separados.
- Para testar localmente, não abra o `index.html` com duplo clique (módulos exigem servidor): use `npm start`.

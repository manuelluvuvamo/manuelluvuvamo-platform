# manuelluvuvamo.com

Portfólio pessoal de **Manuel Luvuvamo** e o mini CMS que o alimenta.

Este é o **repositório principal**. Ele não tem código próprio: junta dois
repositórios independentes como submódulos de Git e guarda o que só faz
sentido no conjunto — o `docker-compose.yml` e esta documentação.

```
manuelluvuvamo.com/          repositório principal (este)
├── README.md
├── docker-compose.yml        Mongo + API, para desenvolvimento local
├── .gitmodules              aponta para os dois repositórios abaixo
├── frontend/                submódulo → portfólio e dashboard (Next.js)
└── backend/                 submódulo → API e CMS (Spring Boot + MongoDB)
```

| Parte | Stack | Repositório |
|---|---|---|
| `frontend` | Next.js 13 (App Router), TypeScript, Tailwind | `manuelluvuvamo/manuelluvuvamo.com` |
| `backend` | Java 21, Spring Boot 3.3, MongoDB | `manuelluvuvamo/manuelluvuvamo-api` |

Precisas de **Node 24** (fixado em `frontend/package.json`), **JDK 21** e Docker.
O pacote Java é `com.manuelluvuvamo.portfolio`.

---

## Arranque rápido

```bash
git clone --recurse-submodules https://github.com/manuelluvuvamo/manuelluvuvamo-platform.git
cd manuelluvuvamo-platform
docker compose up --build -d
cd frontend && npm install && npm run dev
```

Fica com o site em `http://localhost:3000`, o painel em
`http://localhost:3000/admin` e a API em `http://localhost:8080`.

---

## Como gerir o Git

A ideia central dos submódulos: **o repositório principal não guarda o código
dos filhos, guarda apenas um ponteiro para um commit exacto de cada um.** É
por isso que dá para fixar o site numa versão da API que se sabe que funciona.

Na prática, isso obriga a uma regra: **cada alteração precisa de dois commits**
— um dentro do submódulo e outro no principal, a registar o novo ponteiro.

### Clonar

```bash
git clone --recurse-submodules <url-do-principal>
```

Se te esqueceste do `--recurse-submodules`, as pastas `frontend/` e `backend/`
vêm vazias. Resolve-se com:

```bash
git submodule update --init --recursive
```

### Trabalhar no código

O trabalho do dia-a-dia acontece **dentro** de `frontend/` ou `backend/`. Cada
uma é um repositório normal, com os seus próprios ramos e histórico.

```bash
cd frontend
git checkout -b feat/nova-seccao
# … editar …
git add . && git commit -m "Adiciona secção X"
git push origin feat/nova-seccao
```

> **Atenção ao HEAD solto.** Por omissão, o Git deixa os submódulos num commit
> específico, fora de qualquer ramo (*detached HEAD*). Se fizeres commits assim,
> ficam órfãos. **Faz sempre `git checkout main` (ou o teu ramo) dentro do
> submódulo antes de começares a trabalhar.**

### Registar a alteração no principal

Depois de dares push no submódulo, o principal vê que o ponteiro mudou:

```bash
cd ..
git status
#   modified: frontend (new commits)

git add frontend
git commit -m "frontend: aponta para a secção X"
git push
```

Enquanto não fizeres isto, quem clonar o principal continua a receber a versão
antiga do frontend.

### Trazer o trabalho de outra máquina

```bash
git pull                                   # actualiza o principal
git submodule update --init --recursive    # põe os submódulos no commit apontado
```

Ou, num só passo:

```bash
git pull --recurse-submodules
```

### Actualizar um submódulo para o topo do ramo dele

`git submodule update` **não** traz commits novos do remoto; só move para o
commit que o principal aponta. Para saltar para o mais recente:

```bash
git submodule update --remote frontend
git add frontend && git commit -m "frontend: actualiza para o último main"
```

### Configurações que evitam surpresas

Corre isto uma vez, dentro do repositório principal:

```bash
git config --global submodule.recurse true
git config --global diff.submodule log
git config --global status.submodulesummary 1
```

- `submodule.recurse` faz `pull`, `checkout` e afins entrarem nos submódulos.
- `diff.submodule log` mostra *que commits* mudaram, em vez de dois hashes.
- `status.submodulesummary` avisa-te quando há trabalho por registar.

### Erros comuns

| Sintoma | Causa | Solução |
|---|---|---|
| `frontend/` e `backend/` vazias | clone sem `--recurse-submodules` | `git submodule update --init --recursive` |
| Commits desaparecidos no submódulo | trabalhaste em *detached HEAD* | `git checkout -b recupera <hash>` antes de mudar de commit |
| `modified: backend (new commits)` que não pediste | fizeste checkout de outro ramo no submódulo | `git submodule update` para voltar ao ponteiro, ou regista a mudança |
| Alterei o submódulo mas o deploy não mudou | falta o commit do ponteiro no principal | `git add <submódulo> && git commit` |

### Mudar o endereço de um submódulo

Os endereços vivem no `.gitmodules`. Depois de editares:

```bash
git submodule sync --recursive
```

---

## Variáveis de ambiente

### `backend/` — API Spring Boot

Copia `backend/.env.example` para `backend/.env` (ou define-as no ambiente de
produção). Todas têm valor por omissão para desenvolvimento local; as duas
marcadas com ⚠️ **têm de mudar antes de publicares**.

| Variável | Por omissão | Para quê |
|---|---|---|
| `MONGODB_URI` | `mongodb://localhost:27017/portfolio` | Ligação ao MongoDB. No compose é `mongodb://mongo:27017/portfolio`. |
| `SERVER_PORT` | `8080` | Porta da API. |
| `CORS_ALLOWED_ORIGINS` | `http://localhost:3000` | Origens autorizadas, separadas por vírgula. Em produção, só o domínio do site. |
| ⚠️ `JWT_SECRET` | valor de exemplo | Chave de assinatura dos tokens. **Mínimo 32 bytes.** Gera com `openssl rand -base64 48`. |
| `JWT_ACCESS_MINUTES` | `30` | Validade do access token. |
| `JWT_REFRESH_DAYS` | `14` | Validade do refresh token. |
| `ADMIN_EMAIL` | `manuelluvuvamo337@gmail.com` | Utilizador do painel, criado no primeiro arranque. |
| ⚠️ `ADMIN_PASSWORD` | `admin12345` | Password inicial. Troca-a no painel depois de entrares — a partir daí esta variável deixa de ser usada. |
| `ADMIN_NAME` | `Manuel Luvuvamo` | Nome mostrado no painel. |
| `SEED_ENABLED` | `true` | Popula colecções **vazias** a partir de `seed/portfolio-seed.json`. Nunca sobrepõe o que já editaste. |

### `frontend/` — Next.js

Copia `frontend/.env.example` para `frontend/.env.local`.

| Variável | Por omissão | Para quê |
|---|---|---|
| `NEXT_PUBLIC_API_URL` | `http://localhost:8080/api/v1` | Endereço da API. Em produção, o domínio público da API. |
| `NEXT_PUBLIC_SITE_URL` | `https://manuelluvuvamo.vercel.app` | Usado nos metadados, canónicos e sitemap. |
| `WEBHOOK_URL` | — | Webhook do Discord. Opcional: se estiver definido, cada mensagem de contacto também chega ao Discord. |

### Raiz — só para o `docker-compose`

| Variável | Por omissão | Para quê |
|---|---|---|
| `MONGO_HOST_PORT` | `27017` | Porta do Windows por onde chegas ao Mongo do container. Muda para `27018` se já tiveres um MongoDB local. |

> Se a API estiver em baixo, o site **não** parte: cai para o conteúdo estático
> de `frontend/src/data/seed.json`. O painel, esse, precisa mesmo da API.

---

## Docker

### Comandos

```bash
docker compose up --build -d      # constrói e arranca Mongo + API
docker compose logs -f api        # segue os registos da API
docker compose ps                 # estado dos containers
docker compose down               # pára e remove os containers
docker compose down -v            # o mesmo, e apaga a base de dados
```

O frontend **não** está no compose: corre-se com `npm run dev`, para teres
recarregamento imediato enquanto trabalhas.

### Preciso de criar uma rede para a API falar com o Mongo?

**Não.** O `docker compose` cria automaticamente uma rede *bridge* para o
projecto (vais vê-la nos logs como `manuelluvuvamocom_default`) e liga lá todos
os serviços do ficheiro. Dentro dessa rede, cada serviço é resolúvel pelo
**nome do serviço** — é por isso que a API usa:

```
MONGODB_URI=mongodb://mongo:27017/portfolio
```

e não `localhost`. Dentro do container da API, `localhost` é o próprio
container, não a máquina nem o Mongo.

Só precisarias de declarar `networks:` à mão se quisesses isolar serviços entre
si, ligar containers de **projectos compose diferentes**, ou fixar o nome da
rede. Para este caso, o comportamento por omissão chega.

O `ports:` do Mongo existe apenas para tu lhe acederes a partir do Windows
(Compass, `mongosh`). Se o removeres, a API continua a falar com ele
exactamente na mesma — e a base de dados deixa de estar exposta, o que é o que
queres em produção.

### E se eu já tiver o MongoDB instalado no PC, na mesma porta?

Não há prioridade entre os dois: **há colisão, e o compose falha** a arrancar o
container com `Bind for 0.0.0.0:27017 failed: port is already allocated`. O
processo do Windows já tem a porta, e o Docker não a consegue tomar.

Nada disto afecta a API. Ela não passa pelo `ports:` — liga-se pela rede do
compose a `mongo:27017`, dentro do Docker. **A API usa sempre o Mongo do
container**, tenhas ou não um instalado no Windows. São duas bases de dados
separadas, que nunca se misturam.

Para resolver a colisão, escolhe uma das três:

```bash
# a) publicar noutra porta do Windows (a API continua igual)
MONGO_HOST_PORT=27018 docker compose up -d

# b) não publicar de todo — apaga o bloco "ports:" do serviço mongo

# c) parar o serviço local enquanto trabalhas neste projecto
net stop MongoDB
```

Com a opção (a), ligas-te ao Mongo do container em `localhost:27018` e ao teu
Mongo local em `localhost:27017`, sem ambiguidade.

---

## Imagens

Todos os campos de imagem do painel — capa de projecto, capa de artigo,
miniatura de vlog, fotografia do perfil — aceitam **duas coisas**: um endereço
escrito à mão, ou um ficheiro carregado do computador.

O que fica guardado é sempre um endereço. Carregar um ficheiro apenas o
preenche por ti:

1. o ficheiro sobe para a API (`POST /api/v1/admin/files`);
2. é guardado no **GridFS do próprio MongoDB** — sem serviço de armazenamento
   a mais para manter, e os ficheiros viajam nos backups da base de dados;
3. o campo fica com `/media/<id>`, um caminho **relativo**.

Ser relativo é o que importa: a mesma base de dados serve local, homologação e
produção sem o endereço da API colado ao conteúdo. O `/media/<id>` é servido
pelo Next, que vai buscar o ficheiro à API e o entrega com cache permanente
(o identificador nunca muda de conteúdo).

Aceita JPEG, PNG, WebP, GIF, AVIF e SVG, até 5 MB.

---

## Contagem de leitura

Cada artigo do blog conta **duas coisas diferentes**:

| | Quando conta |
|---|---|
| **Aberturas** | o artigo foi aberto |
| **Leituras completas** | o leitor chegou ao fim do texto |

A separação é o que torna o número útil: cem aberturas com três leituras dizem
algo que "cem visitas" nunca diria. Vês os dois na listagem do blog, em
`/admin/posts`.

Cada um conta no máximo uma vez por separador, por isso recarregar a página não
inflaciona nada. O incremento acontece dentro do Mongo, sem ler o documento
primeiro — duas visitas ao mesmo tempo contam as duas, e a data de alteração do
artigo não é tocada.

O browser fala com o próprio site (`/api/metrics/…`) e não com a API. É de
propósito: cada preview do Vercel tem um domínio diferente, e uma chamada
directa à API falharia por CORS em todos eles, em silêncio.

---

## Porque é que o `npm run dev` parece lento

Não é a API nem a base de dados. É o Next a compilar cada rota **na primeira
vez que a visitas**. Medido nesta máquina:

| | Primeira visita | Depois |
|---|---|---|
| `npm run dev` | 7 a 10 s | ~1 s |
| `npm run build && npm start` | 40 a 70 ms | 30 a 70 ms |

Em produção as páginas são geradas estaticamente e servidas em dezenas de
milissegundos. Se quiseres avaliar a velocidade real do site, mede com
`npm run build && npm start` — o `dev` mede o compilador, não o site.

Se o `dev` estiver insuportável, o costume em Windows é excluir a pasta do
projecto (sobretudo `node_modules` e `.next`) da análise em tempo real do
antivírus.

---

## Endpoints

| | |
|---|---|
| Site | `http://localhost:3000` |
| Painel | `http://localhost:3000/admin` |
| API pública | `http://localhost:8080/api/v1/public/**` |
| API do painel | `http://localhost:8080/api/v1/admin/**` (exige JWT) |
| Imagens | `http://localhost:3000/media/<id>` |
| Documentação | `http://localhost:8080/docs` |
| Saúde | `http://localhost:8080/actuator/health` |

`GET /api/v1/public/bootstrap` devolve todo o conteúdo publicado num só pedido
— é o que o Next.js usa para gerar as páginas.

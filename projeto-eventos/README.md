# Showline — Gerenciamento de Shows

Front-end + back-end para o seu projeto original do repositório `M-todos-` (IF264).
O back-end (FastAPI + SQLite) foi refinado e ganhou um front-end inspirado em
plataformas como Sympla / Eventbrite / Ticketmaster, com cards em formato de
"canhoto de ingresso", barra de lotação, fila de espera e busca.

```
projeto-eventos/
├── backend/
│   ├── main.py            # API (refinada a partir do seu main.py original)
│   └── requirements.txt
└── frontend/
    ├── index.html
    ├── style.css
    └── script.js
```

## O que mudou em relação ao seu `main.py` original

- **CORS habilitado** — sem isso o navegador bloqueia as chamadas do front (porta diferente da API).
- **Validação com Pydantic** (`EventoCreate`, `InscricaoCreate`) em vez de query params soltos.
- **Novos campos no evento**: `descricao`, `local`, `categoria`, `imagem_url`, `preco`, `criado_em`.
- **Novos endpoints**: `GET /eventos/` (listar tudo), `GET /eventos/{id}` (detalhe), `DELETE /eventos/{id}` (remover evento), `GET /categorias`.
- **Cascade delete**: ao apagar um evento, fila de espera e vínculos de inscrição são removidos junto (`ondelete="CASCADE"` + `cascade="all, delete-orphan"`).
- **Bug corrigido** na busca por data (a versão original comparava `DateTime` completo com uma data sem hora, então quase nunca encontrava nada).

### Sistema de usuários (login/cadastro)

- Tabela `usuarios` (username + senha) e `sessoes` (tokens de login).
- Senhas nunca ficam em texto puro: usamos **PBKDF2-HMAC-SHA256** com salt aleatório,
  só com a biblioteca padrão do Python (`hashlib`/`secrets`) — escolhi essa abordagem
  de propósito para não depender de pacotes como `bcrypt`/`passlib` que às vezes
  exigem compilação no Windows (o mesmo problema que você teve com o `pydantic-core`).
- Login gera um **token opaco** (string aleatória) salvo em `sessoes`. O front guarda
  esse token no `localStorage` do navegador e manda em todo pedido protegido como
  `Authorization: Bearer <token>`.
- Cada evento agora tem um `criador_id`. **Só quem criou o evento pode editá-lo ou excluí-lo**
  (`PUT /eventos/{id}` e `DELETE /eventos/{id}` verificam isso e retornam `403` se não for o dono).
- Inscrição/cancelamento de participantes continuam públicos — qualquer visitante pode se
  inscrever num evento, só a gestão do evento em si é restrita ao criador.
---

## 1. Rodando na sua máquina

### Pré-requisitos
- Python 3.10+ instalado (`python --version` no terminal)
- Não precisa de Node nem de nada além de um navegador para o front-end

### 1.1. Backend (API)

```powershell
# entre na pasta do backend
cd projeto-eventos/backend

# crie um ambiente virtual (recomendado)
python -m venv venv

# ative o ambiente virtual
# Windows (PowerShell):
venv\Scripts\Activate.ps1
# Windows (cmd):
venv\Scripts\activate.bat
# Linux/Mac:
source venv/bin/activate

# instale as dependências
pip install -r requirements.txt

# rode o servidor
uvicorn main:app --reload
```

Se tudo der certo, vai aparecer algo como:
```
Uvicorn running on http://127.0.0.1:8000
```

Deixe esse terminal aberto. Você pode testar a API sozinha abrindo
`http://127.0.0.1:8000/docs` no navegador (documentação automática do FastAPI).

O banco `eventos.db` (SQLite) é criado automaticamente na primeira execução,
na própria pasta `backend/`.

### 1.2. Frontend

Abra um **segundo terminal** (deixe o backend rodando no primeiro):

```powershell
cd projeto-eventos/frontend

# qualquer servidor estático funciona. Com Python já instalado:
python -m http.server 5500
```

Agora abra `http://127.0.0.1:5500` no navegador.

> Por que não abrir o `index.html` direto com duplo clique?
> Alguns navegadores bloqueiam `fetch()` em arquivos abertos via `file://`.
> Servir por `http.server` (ou a extensão "Live Server" do VS Code) evita esse problema.

Se o indicador no topo da página mostrar **"API conectada"** (ponto verde), está tudo certo.
Se mostrar **"API offline"**, confirme que o terminal do backend ainda está rodando.

---
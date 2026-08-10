# AiLM — distribuição

Este repositório **distribui** o AiLM. Ele não contém código versionado: o que vale aqui são as
[Releases](../../releases).

O AiLM é uma plataforma AI-First com método SPDD, agentes, governança e observabilidade.

## Antes de começar

Instale **tudo isto antes** de começar. O `doctor` reprova o que falta, mas descobrir no fim é pior.

| requisito | por quê | como instalar / conferir |
|---|---|---|
| **Node.js ≥ 22.6** | o motor roda em Node | `node --version` · <https://nodejs.org> |
| **Python ≥ 3.11** | gates e projeção de telemetria | `python --version` |
| **git** | o AiLM opera sobre o seu repositório | `git --version` |
| **bun** | **dependência dura** de abrir PR, publicar artefatos e gravar BCP | `bun --version` · <https://bun.sh> |
| **Claude Code CLI**, autenticado | executa os nós de IA | `npm i -g @anthropic-ai/claude-code` e depois `claude login` |
| **`psycopg2`** | acesso do Python ao Postgres | `pip install psycopg2-binary` |
| **PostgreSQL** alcançável | estado autoritativo das demandas | local, container ou serviço gerenciado — você decide |
| **`psql`** no PATH | o setup aplica o schema por ele | `psql --version` · Debian/Ubuntu: `apt install postgresql-client` · Windows: vem com o instalador do PostgreSQL |

### Máquina nova? Instale tudo de uma vez

Os comandos abaixo cobrem a lista inteira. Depois deles, **abra um terminal novo** — instaladores
mexem no `PATH` e a sessão atual não enxerga o que acabou de entrar.

**Windows** (winget já vem no Windows 11; os IDs abaixo foram verificados):

```powershell
winget install OpenJS.NodeJS.LTS
winget install Python.Python.3.12
winget install Git.Git
winget install Oven-sh.Bun
winget install PostgreSQL.PostgreSQL.17   # traz o psql; se já tem um Postgres, veja a nota abaixo
```

**macOS** (Homebrew):

```bash
brew install node python git bun libpq
brew link --force libpq          # sem isto o psql não entra no PATH
```

**Linux** (Debian/Ubuntu):

```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs python3 python3-pip git postgresql-client
curl -fsSL https://bun.sh/install | bash
```

> ⚠ **Windows: o instalador do PostgreSQL NÃO coloca o `psql` no PATH.** Medido em host limpo
> (2026-08-10): tudo o mais funciona e só o `psql` fica invisível. Corrija com uma linha no
> PowerShell — ela descobre o caminho, então serve para qualquer versão:
>
> ```powershell
> $bin = (Get-ChildItem 'C:\Program Files\PostgreSQL\*\bin\psql.exe' | Select-Object -First 1).Directory.FullName
> [Environment]::SetEnvironmentVariable('Path', "$([Environment]::GetEnvironmentVariable('Path','User'));$bin", 'User')
> ```
>
> E abra **outro** terminal — este PATH novo só vale em sessões criadas depois.

Depois, em **qualquer** sistema — terminal novo:

```bash
npm install -g @anthropic-ai/claude-code
claude login                                    # autentica; o AiLM usa sua assinatura

pip install pyyaml jsonschema psycopg2-binary   # no macOS/Linux pode ser pip3
```

> `pyyaml` e `jsonschema` não são opcionais: o `validate.py` — que é gate do `ailm run` — importa os
> dois. Sem eles a primeira demanda falha.

> Só precisa do **PostgreSQL completo** se for hospedar o banco na própria máquina. Se o seu Postgres
> é remoto (RDS, Cloud SQL, Neon, Supabase), basta o **cliente**: no Windows o instalador permite
> marcar apenas *Command Line Tools*; no macOS/Linux os comandos acima já instalam só o cliente.

Confira tudo de uma vez — nenhuma linha pode sair vazia:

```bash
node --version
python --version
git --version
bun --version
psql --version
claude --version
```

Tenha em mãos, para o `setup`:

- a **URL de conexão completa** do seu Postgres (`postgres://usuario:senha@host:5432/banco`);
- um **token do GitHub** com escopo `project` (ver [abaixo](#o-token-do-github-precisa-de-permissão-de-projects));
- opcionalmente, um **token de bot do Slack** e o canal.

Você **não** precisa de `gh`, de `npm install`, nem de conta no LangSmith.

O AiLM se instala **dentro de um repositório git seu**, que é o projeto que ele vai operar.

# Escolha o seu caminho

Os passos são os mesmos, mas os **comandos diferem**. Siga UM dos dois — não misture.

| você usa | vá para |
|---|---|
| **PowerShell** ou **cmd.exe** (Windows padrão) | [Windows sem Git Bash](#instalar-no-windows-sem-git-bash) — logo abaixo |
| **Linux**, **macOS** ou **Git Bash** | [Passo 1](#1-baixar), depois desta seção |

Por que não dá para usar os mesmos comandos: no PowerShell não existe `sha256sum`, `mkdir` não aceita
`-p`, `&&` não é separador válido (5.1), e `curl` é apelido de `Invoke-WebRequest`, que rejeita
`-sL -O`.

## Instalar no Windows sem Git Bash

Mesmas etapas, em PowerShell. Todos os comandos abaixo foram executados num Windows 11 limpo antes de
entrarem aqui.

Use **`curl.exe`** com o `.exe` explícito: sem ele o PowerShell resolve o apelido `curl` para
`Invoke-WebRequest`, que não aceita `-sL -O`.

```powershell
$tmp = "$env:TEMP\ailm"
New-Item -ItemType Directory -Force $tmp | Out-Null
Set-Location $tmp

curl.exe -sL -O https://github.com/danielrmarques/ailm-dist/releases/latest/download/ailm-bundle.tgz
curl.exe -sL -O https://github.com/danielrmarques/ailm-dist/releases/latest/download/ailm-bundle.tgz.sha256
```

Confirme o download (o PowerShell compara sem diferenciar maiúsculas, então não normalize nada):

```powershell
$esperado = (Get-Content ailm-bundle.tgz.sha256).Split()[0]
$obtido   = (Get-FileHash ailm-bundle.tgz -Algorithm SHA256).Hash
if ($esperado -eq $obtido) { "checksum OK" } else { "PARE — checksum divergente" }
```

Extraia (o `tar` do Windows 10+ serve) e instale no **seu** repositório:

```powershell
tar -xzf ailm-bundle.tgz

node "$tmp\.ailm\runtime\scripts\ailm-install.mjs" install $tmp C:\caminho\do\seu\repo
```

Configure — no PowerShell o wrapper é `.\ailm.cmd`, sem a barra do estilo Unix:

```powershell
Set-Location C:\caminho\do\seu\repo
.\ailm.cmd setup
.\ailm.cmd doctor
```

> Uma peculiaridade do PowerShell, não do AiLM: ele trata a saída de erro de programas externos como
> se fossem exceções, então as mensagens de aviso do AiLM aparecem em vermelho e `$LASTEXITCODE` pode
> não refletir o resultado real. Confie no que o `doctor` imprime, linha por linha.

## 1. Baixar

```bash
mkdir -p /tmp/ailm && cd /tmp/ailm

curl -sL -O https://github.com/danielrmarques/ailm-dist/releases/latest/download/ailm-bundle.tgz
curl -sL -O https://github.com/danielrmarques/ailm-dist/releases/latest/download/ailm-bundle.tgz.sha256
```

**Confirme o que baixou antes de aplicar** — precisa imprimir `ailm-bundle.tgz: OK`:

```bash
sha256sum -c ailm-bundle.tgz.sha256
```

Se não imprimir `OK`, pare: o arquivo chegou corrompido ou alterado. Baixe de novo.

```bash
tar -xzf ailm-bundle.tgz
```

## 2. Instalar no seu repositório

Troque `~/meus-projetos/meu-repo` pelo caminho real do **seu** repositório:

```bash
node /tmp/ailm/.ailm/runtime/scripts/ailm-install.mjs install /tmp/ailm ~/meus-projetos/meu-repo
```

É `node .../ailm-install.mjs` e não `ailm install`: o comando `ailm` só passa a existir **depois**
desta etapa.

A instalação é **não-destrutiva**. Seu `package.json`, seu `README.md` e seu código não são tocados.
Arquivos que podem colidir — `.gitignore`, `CLAUDE.md`, `.claude/settings.json` — recebem um **bloco
demarcado** e o resto do conteúdo é preservado. Um ledger registra tudo, e `./ailm uninstall` reverte
exatamente o que foi feito.

## 3. Configurar

```bash
cd ~/meus-projetos/meu-repo

./ailm setup     # perguntas do .env, testa o banco, aplica o schema e cria o Project no GitHub
./ailm doctor    # confirma que o ambiente está pronto
```

No `cmd.exe` ou PowerShell é **`.\ailm.cmd setup`** — o PowerShell não executa comandos do diretório
atual sem o `.\`, e `ailm` puro devolve `CommandNotFoundException`.

O `setup` faz **7 perguntas**, todas coisas que você sabe responder: token, dono e repo do GitHub, a
URL do banco, o nome da sua squad, e — opcionais — token e canal do Slack.

As demais ~25 variáveis são ajuste fino e ficam com valores padrão no `.env`, **documentadas uma a
uma**. Precisa mudar alguma? Edite o `.env` direto — é o caminho enquanto essas configurações não
estiverem no AiLM Console. Para revê-las uma a uma no wizard: `./ailm setup --all`.

Ele **testa a conexão e aplica o schema do banco** na mesma passada, em vez de deixar para depois.

### O token do GitHub precisa de permissão de Projects

O AiLM cria e mantém um Project V2 (chamado `AiLM`) com os campos tipados que a operação usa —
Status, Type, Priority, BCP, PQS, CEI, Active Run. O escopo `project` é o que costuma faltar num token
criado às pressas. Escolha **uma** destas formas:

1. **`gh` CLI**, se você já usa: `gh auth login -s project` — e deixe `GITHUB_TOKEN` vazio no `.env`;
2. **PAT clássico** em <https://github.com/settings/tokens>: marque `repo` **e** `project`;
3. **PAT fine-grained**: em *Account/Organization permissions*, `Projects: Read and write`; em
   *Repository permissions*, `Issues`, `Contents` e `Pull requests` com *Read and write*.

Sem esse escopo o `setup` avisa e diz o que fazer — ele não falha em silêncio.

## 4. Rodar a primeira demanda

```bash
./ailm run <owner>/<repo>#<número-da-issue>
```

O AiLM escolhe o trilho pela complexidade da issue e conduz o ciclo. Demandas maiores pausam num
**gate humano**: você aprova com `./ailm approve <run-id>` ou recusa com
`./ailm reject <run-id> --reason "<motivo>"`.

## Atualizar

```bash
./ailm update --check   # só informa se há versão nova
./ailm update           # baixa, valida o checksum e aplica
```

No PowerShell: `.\ailm.cmd update --check` e `.\ailm.cmd update`.

O `update` **recusa** bundle sem checksum ou com checksum divergente, e **bloqueia com instrução**
quando o layout ou o schema da versão nova diferem do install atual — migrar às cegas corromperia o
que está no disco.

## Se algo der errado

| sintoma | causa provável |
|---|---|
| `bash: gh: command not found` | você está numa doc antiga; use os `curl` acima — `gh` não é necessário |
| `sha256sum: ailm-bundle.tgz: FAILED` | download corrompido; apague e baixe de novo |
| `Cannot find module ...ailm-install.mjs` | o `tar -xzf` não rodou, ou você está fora de `/tmp/ailm` |
| `./ailm: Permission denied` | `chmod +x ailm` |
| `psql` não é reconhecido (Windows) | o instalador do PostgreSQL **não** põe o `bin` no PATH — veja o aviso na seção de pré-requisitos |
| `psql: command not found` (macOS/Linux) | instale o cliente (`postgresql-client`); no macOS falta o `brew link --force libpq` |
| o `doctor` reprova em `critical` | ele diz o que fazer em cada linha; resolva de cima para baixo |

Para desfazer tudo: `./ailm uninstall`.

## Versionamento

Versão `0.x`: o método é usado em produção no desenvolvimento do próprio AiLM, mas a adoção externa
está em piloto. Enquanto isso durar, interfaces podem mudar entre versões menores. `Z` e `Y` nunca
exigem trabalho seu no update; mudança que exija migração vem sinalizada e o `update` bloqueia antes
de aplicar.

## Licença

BUSL-1.1 — ver `LICENSE` e `NOTICE` no bundle. Código legível não é código livre: uso em produção
requer licença comercial.

## Suporte

Este repositório recebe apenas releases. Para dúvidas e relatos, procure o time do AiLM.

# AiLM — distribuição

Este repositório **distribui** o AiLM. Ele não contém código versionado: o que vale aqui são as
[Releases](../../releases).

O AiLM é uma plataforma AI-First com método SPDD, agentes, governança e observabilidade.

## Antes de começar

| requisito | por quê | como conferir |
|---|---|---|
| **Node.js ≥ 22.6** | o motor roda em Node | `node --version` |
| **Python ≥ 3.11** | gates e projeção de telemetria | `python --version` |
| **git** | o AiLM opera sobre o seu repositório | `git --version` |
| **Claude Code CLI**, autenticado | executa os nós de IA | `claude auth status` |
| **PostgreSQL** alcançável | estado autoritativo das demandas | pode ser local, container ou serviço gerenciado |
| **`psql`** no PATH | o setup aplica o schema | `psql --version` — no Debian/Ubuntu: `apt install postgresql-client` |

Você **não** precisa de `gh`, de `npm install`, nem de conta no LangSmith.

O AiLM se instala **dentro de um repositório git seu**, que é o projeto que ele vai operar.

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

No `cmd.exe` ou PowerShell, use `ailm` sem o `./`.

O `setup` pergunta o mínimo: token e dono/repo do GitHub, a URL do banco, e opcionalmente o Slack.
Ele **testa a conexão e aplica o schema** — não deixa isso para depois.

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
| `psql: command not found` no setup | instale o cliente do Postgres (`postgresql-client`) |
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

# AiLM — distribuição

Este repositório existe para **distribuir** o AiLM. Ele não contém código versionado: o que vale
aqui são as [Releases](../../releases).

O AiLM é uma plataforma AI-First com método SPDD, agentes, governança e observabilidade.

## Instalar

```bash
# baixe o bundle da última versão
gh release download --repo danielrmarques/ailm-dist --pattern 'ailm-bundle.tgz*' --dir /tmp/ailm
tar -xzf /tmp/ailm/ailm-bundle.tgz -C /tmp/ailm

# instale no SEU repositório
node /tmp/ailm/.ailm/runtime/scripts/ailm-install.mjs install /tmp/ailm /caminho/do/seu/repo
```

A instalação é **não-destrutiva**: arquivos de raiz que podem colidir com os seus — `.gitignore`,
`CLAUDE.md`, `.claude/settings.json` — são **mesclados por bloco demarcado**, nunca sobrescritos.
Um ledger registra o que foi feito, e `ailm uninstall` reverte exatamente isso.

Antes de aplicar, valide o que baixou:

```bash
sha256sum -c /tmp/ailm/ailm-bundle.tgz.sha256
```

## Atualizar

```bash
ailm update --check   # só informa se há versão nova
ailm update           # baixa, valida o checksum e aplica
```

O `update` **recusa** bundle sem checksum ou com checksum divergente, e **bloqueia com instrução**
quando o layout ou o schema da versão nova diferem do install atual — migrar às cegas corromperia
o footprint.

## Configurar

Depois de instalar:

```bash
ailm setup     # configura as variáveis de ambiente
ailm doctor    # verifica se está pronto para rodar
```

## Versionamento

Versão `0.x`: o método é usado em produção no desenvolvimento do próprio AiLM, mas a adoção externa
está em piloto. Interfaces podem mudar entre versões menores enquanto isso durar.

## Suporte

Este repositório recebe apenas releases. Para dúvidas e relatos, procure o time do AiLM.

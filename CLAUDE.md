# CLAUDE.md

Este repositório documenta a instalação e a configuração do Debian 13 (GNOME) em um
ThinkPad E14 Gen 1. Contém **documentação** (`docs/`) e, no futuro, **scripts de shell** (`scripts/`).
O repositório é **público**.

## Princípios

- **A Escada:** pare no primeiro degrau que resolve.
  1. Não precisa? Não faça. 2. Pacote oficial do Debian. 3. Recurso nativo do GNOME.
  4. Flatpak. 5. Repositório externo. 6. Script próprio.
- Faça o mínimo que funciona. Não instale o que não será usado. Não refatore sem pedido.
  Não deixe placeholder sem avisar.
- **Entender antes de automatizar:** faça à mão, documente, e só então vire script.

## Ao executar comandos no sistema

- Antes de rodar, explique o que o comando faz, por que e como desfazer.
- Comando com `sudo` só roda depois de confirmação do usuário.
- Antes de editar um arquivo em `/etc`, faça backup dele.
- Depois de cada passo, rode um comando de verificação e confira a saída.
- Nunca formate, particione ou monte um disco sem confirmação explícita. Confira o alvo com `lsblk` antes.
- Nunca use `curl ... | bash` sem ler o script antes. Use apenas fontes oficiais.
- Repositório APT externo: baixe a chave, confira o fingerprint e use `signed-by` (formato `.sources`).

## Documentação (`docs/`)

- Português do Brasil, frases curtas e diretas, sem primeira pessoa.
- Cada passo traz: o comando, o que ele faz e como conferir o resultado.
- Só registre como executado o que foi de fato executado. O restante leva a marca *(proposta)*.
- Atualize o status em `docs/02-roteiro.md` na mesma mudança que altera o sistema.
- Termo novo entra em `docs/03-glossario.md`.

## Scripts (`scripts/`)

- Só depois de a fase ter sido feita à mão, testada e documentada.
- Um script por fase, numerado (`01-git.sh`), com `#!/usr/bin/env bash` e `set -euo pipefail`.
- Idempotente (rodar duas vezes dá o mesmo resultado) e sem perguntas interativas.
- Sem usuário ou caminho fixo: use `$HOME` e `$USER`.
- `shellcheck` sem avisos.

## Privacidade (repositório público)

- Nunca versione: senhas, tokens, chaves, números de série, UUIDs, MACs, e-mails pessoais,
  nome de empresa ou cargo, nome de usuário local.
- Antes de cada commit, revise `git diff --staged`.
- Commits usam o e-mail `noreply` do GitHub, já configurado.

## Git

- Trabalhe direto na `main`, sem branch nem PR (repositório de uso individual).
- Commits atômicos, em Conventional Commits, em português e no imperativo:
  `docs: descrever o disco extra`, `feat: adicionar script do git`.
- Faça `git push` só quando o usuário pedir, porque o repositório é público.

## Definition of Done

- O comando ou a verificação rodou e a saída foi conferida.
- Docs e roteiro estão atualizados.
- Há como desfazer, e está documentado.
- `shellcheck` passa, se houver script.
- `git diff --staged` não tem segredo nem dado pessoal.
- A mudança é mínima e intencional.

## Pendências

Marque o que ficou para depois, com o motivo: `# TODO(fase 4): ajustar quando testar a GPU`.

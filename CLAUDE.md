# CLAUDE.md

Este repositório documenta a instalação e a configuração do Debian 13 (GNOME) em um
ThinkPad E14 Gen 1. Contém **documentação** (`docs/`). Não há scripts: o repositório registra o que foi feito e o motivo.
O repositório é **público**.

## Estrutura

- `README.md`: apresentação, público, como usar e estado das fases.
- `docs/01-ambiente.md`: sistema, hardware de referência e software instalado.
- `docs/02-roteiro.md`: índice das fases, com status e escopo.
- `docs/fases/`: um arquivo por fase (`NN-nome.md`), com os comandos e as verificações.
- `docs/03-glossario.md`: termos em ordem alfabética.
- `LICENSE` (MIT, para os comandos) e `LICENSE-DOCS.txt` (CC BY 4.0, para os textos).

## Princípios

- **A Escada:** pare no primeiro degrau que resolve.
  1. Não precisa? Não faça. 2. Pacote oficial do Debian. 3. Recurso nativo do GNOME.
  4. Flatpak. 5. Repositório externo. 6. Script próprio.
- Faça o mínimo que funciona. Não instale o que não será usado. Não refatore sem pedido.
  Não deixe placeholder sem avisar.
- **Entender antes de executar:** explique, faça à mão, confira e documente.

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
- Cada passo traz: o comando, o que ele faz, como conferir o resultado e como desfazer.
- Cada fase abre com o cabeçalho padrão: título, **Status**, **Escopo** (geral, modelo de referência, perfil de uso ou ambiente), navegação e objetivo.
- Só registre como executado o que foi de fato executado. O restante leva a marca *(proposta)*.
- Texto atemporal: sem datas, sem horas, sem medidas de um momento e sem relato de tentativa ou de erro. Registre só o que serve de referência permanente.
- Atualize o status no arquivo da fase e em `docs/02-roteiro.md` na mesma mudança que altera o sistema.
- Termo novo entra em `docs/03-glossario.md`, em ordem alfabética.
- Ao mudar a estrutura, confira os links internos entre os arquivos.

## Privacidade (repositório público)

- Nunca versione: senhas, tokens, chaves, números de série, UUIDs, MACs, e-mails pessoais,
  nome de empresa ou cargo, nome de usuário local, nome da máquina e endereços de servidores.
- Antes de cada commit, revise `git diff --staged`.
- Commits usam o e-mail `noreply` do GitHub, já configurado.

## Git

- Trabalhe direto na `main`, sem branch nem PR (repositório de uso individual).
- Commits atômicos, em Conventional Commits, em português e no imperativo:
  `docs: descrever o disco extra`, `docs: registrar o backup`.
- Faça `git push` só quando o usuário pedir, porque o repositório é público.

## Definition of Done

- O comando ou a verificação rodou e a saída foi conferida.
- Docs, roteiro e fase estão atualizados.
- Há como desfazer, e está documentado.
- `git diff --staged` não tem segredo nem dado pessoal.
- A mudança é mínima e intencional.

## Pendências

Marque o que ficou para depois, com o motivo: `<!-- TODO: confirmar no próximo modelo de notebook -->`.

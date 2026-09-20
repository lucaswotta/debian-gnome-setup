# debian-gnome-setup

Guia para instalar e configurar o **Debian 13 (GNOME)** em um **Lenovo ThinkPad E14 Gen 1 (20RB)**,
do sistema recém-instalado até o ambiente pronto para trabalho.

Cada passo traz o comando, o que ele faz e como conferir o resultado. Pensado para quem usa
Debian pela primeira vez.

## Estado atual

Fases 0 a 4 concluídas (base, git e GitHub, sistema, atualizações e notebook). Em andamento: fase 5, aplicativos.
O detalhe de cada fase está em [docs/02-roteiro.md](docs/02-roteiro.md).

## Documentos

| Documento | Conteúdo |
|---|---|
| [docs/01-ambiente.md](docs/01-ambiente.md) | A distro, o notebook e o estado atual do sistema |
| [docs/02-roteiro.md](docs/02-roteiro.md) | Passo a passo em fases, com o que está feito e o que falta |
| [docs/03-glossario.md](docs/03-glossario.md) | Termos de Debian e Linux explicados de forma simples |

## Estrutura

```
debian-gnome-setup/
├── README.md
├── CLAUDE.md    instruções para o Claude Code ao trabalhar neste repositório
├── docs/        documentação
└── scripts/     (futuro) automação de cada fase
```

`scripts/` só será criada depois que uma fase for feita à mão, testada e documentada.

## Convenções

- Idioma: português do Brasil.
- Todo comando foi executado nesta máquina, exceto os marcados como *(proposta)*.
- Status das fases: `[x]` feito, `[ ]` pendente, `[~]` em discussão.
- Nunca versionar: senhas, tokens, chaves, números de série, UUIDs, endereços MAC ou e-mails.
- Foco em máquina de trabalho: estabilidade antes de novidade, e backup antes de tudo.

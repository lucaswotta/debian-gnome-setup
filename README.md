# debian-gnome-setup

Guia prático para instalar e configurar o **Debian 13 (GNOME)** como estação de trabalho, do sistema recém-instalado ao ambiente pronto, com atualizações automáticas, aplicativos e backup.

Cada passo traz o comando, o que ele faz, como conferir o resultado e como desfazer. Todo comando foi executado e verificado num **Lenovo ThinkPad E14 Gen 1 (20RB)**, o modelo de referência do guia.

## Para quem é

- Quem usa o Debian pela primeira vez e quer entender cada passo, e não só copiar comandos.
- Quem monta uma estação de desenvolvimento ou de trabalho de escritório com Debian e GNOME.
- Quem vem do Windows e do Microsoft Office e quer um ambiente familiar.

## O que o guia entrega

| Área | Resultado |
|---|---|
| Sistema | Debian 13 atualizado, com atualizações de segurança automáticas e firmware em dia |
| Notebook | Energia, suspensão, vídeo híbrido e boot direto ajustados |
| Desenvolvimento | Docker, Go, Node, Python, Java, VS Code, DBeaver e ferramentas de API |
| Escritório | LibreOffice com a aparência e os padrões do Microsoft Office |
| Rede | VPN pelo menu do GNOME e acesso a servidores pelo aplicativo Arquivos |
| GNOME | Extensões, atalhos e barra de aplicativos no estilo do Windows |
| Segurança dos dados | Snapshots do sistema com o Timeshift |

## Como usar

1. Leia o [ambiente](docs/01-ambiente.md) e compare com a sua máquina.
2. Siga o [roteiro](docs/02-roteiro.md) na ordem das fases. Cada fase é um arquivo em [`docs/fases/`](docs/fases/).
3. Confira o **escopo** de cada fase. Fases *gerais* servem a qualquer instalação. As de *modelo de referência*, *perfil de uso* e *ambiente* pedem adaptação.
4. Antes de qualquer comando com `sudo`, leia o que ele faz. Cada fase traz a seção **Como desfazer**.
5. Consulte o [glossário](docs/03-glossario.md) para os termos que não conhecer.

## Estado atual

Fases 0 a 7 concluídas: base, git e GitHub, sistema, atualizações, notebook, aplicativos, GNOME e backup.

## Documentos

| Documento | Conteúdo |
|---|---|
| [docs/01-ambiente.md](docs/01-ambiente.md) | A distribuição, o notebook de referência e o que está instalado |
| [docs/02-roteiro.md](docs/02-roteiro.md) | O passo a passo em fases, com o status e o escopo de cada uma |
| [docs/03-glossario.md](docs/03-glossario.md) | Termos de Debian e Linux explicados de forma simples |

## Estrutura

```
debian-gnome-setup/
├── README.md
├── CLAUDE.md          instruções para o Claude Code ao trabalhar neste repositório
├── LICENSE            licença dos comandos e trechos de código (MIT)
├── LICENSE-DOCS.txt   licença dos textos (CC BY 4.0)
└── docs/
    ├── 01-ambiente.md
    ├── 02-roteiro.md
    ├── 03-glossario.md
    └── fases/         um arquivo por fase
```

## Como replicar em outra máquina

O repositório não traz scripts, de propósito: é o registro do que foi feito e do motivo. Para repetir o processo em outro equipamento, peça a um assistente de IA, como o Claude Code, que leia os arquivos de `docs/`, adapte os passos ao seu hardware e execute as fases em ordem. Confirme cada comando com `sudo` antes de ele rodar.

## Convenções

- Idioma: português do Brasil.
- Todo comando foi executado na máquina de referência, exceto os marcados como *(proposta)*.
- Status das tarefas: `[x]` feito, `[ ]` pendente, `[~]` em discussão.
- Foco em máquina de trabalho: estabilidade antes de novidade, e backup antes de tudo.

## Privacidade

O repositório é público. Nunca entram nele senhas, tokens, chaves, números de série, UUIDs, endereços MAC, e-mails pessoais, nomes de empresa, endereços de servidores ou nomes de usuário. Nos exemplos, esses dados aparecem como `<usuario>`, `<ip-do-servidor>` e semelhantes.

## Licença

- Comandos, trechos de código e configurações: [MIT](LICENSE).
- Textos da documentação: [CC BY 4.0](LICENSE-DOCS.txt). Você pode copiar e adaptar, dando o crédito.

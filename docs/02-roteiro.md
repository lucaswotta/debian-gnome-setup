# 02 - Roteiro

Este roteiro leva um Debian 13 recém-instalado até um ambiente de trabalho completo, em fases curtas. Cada fase tem o seu arquivo em [`docs/fases/`](fases/).

## Como o guia funciona

- Uma fase por vez, explicada antes de executar.
- Comando com `sudo` só roda depois de confirmação.
- Só se registra como feito o que foi executado e conferido.
- Cada fase termina com uma verificação e um registro.

## Fases

| Fase | Tema | Escopo | Status |
|---|---|---|---|
| 0 | [Base mínima](fases/00-base-minima.md) | Geral | Concluída |
| 1 | [Git e GitHub](fases/01-git-e-github.md) | Geral | Concluída |
| 2 | [Base do sistema, com disco extra](fases/02-base-do-sistema.md) | Geral | Concluída |
| 3 | [Atualizações](fases/03-atualizacoes.md) | Geral | Concluída |
| 4 | [Notebook](fases/04-notebook.md) | Modelo de referência | Concluída |
| 5 | [Aplicativos](fases/05-aplicativos.md) | Perfil de uso | Concluída |
| 5 | [LibreOffice no estilo do Microsoft Office](fases/05a-libreoffice.md) | Perfil de uso | Concluída |
| 5 | [VPN e acesso a servidores](fases/05b-vpn-e-servidores.md) | Ambiente | Concluída |
| 6 | [GNOME](fases/06-gnome.md) | Perfil de uso | Concluída |
| 7 | [Backup](fases/07-backup.md) | Geral | Concluída |
| 8 | [Terminal e shell](fases/08-terminal.md) | Perfil de uso | Concluída |
| 9 | [Windows em máquina virtual](fases/09-windows-vm.md) | Perfil de uso | Concluída |

## Replicar em outra máquina

O roteiro é um registro, e não um instalador. Para repetir o processo, peça a um assistente de IA que leia as fases, adapte-as ao seu hardware e as execute em ordem, com a sua confirmação nos comandos com `sudo`. As seções **Como conferir** e **Como desfazer** de cada fase servem de checagem.

## Escopo: o que adaptar

| Escopo | Significado |
|---|---|
| **Geral** | Vale para qualquer instalação do Debian 13 com GNOME. |
| **Modelo de referência** | Depende do hardware do notebook usado como referência (ver [Ambiente](01-ambiente.md)). Confira o seu antes de repetir. |
| **Perfil de uso** | São escolhas de aplicativos e de preferências. Troque, remova ou acrescente. |
| **Ambiente** | Depende da rede e dos sistemas de quem usa, como a VPN e os servidores da empresa. |

## Como ler cada fase

Cada arquivo traz, nesta ordem: status e escopo, objetivo, lista de tarefas, os comandos, uma tabela com o que cada parte faz, **como conferir**, **como desfazer** e observações.

- `[x]` feito · `[ ]` pendente · `[~]` em discussão.
- *(proposta)* marca o que foi descrito, mas não executado.
- `<usuario>`, `<ip-do-servidor>`, `AAAA-MM-DD` e valores em maiúsculas, como `USUARIO`, são espaços para o seu dado.

## Decisões que dependem do ambiente

Estas perguntas não têm resposta única. Responda-as antes de aplicar o guia na sua máquina.

1. Existe política de TI que exija antivírus ou um software específico? Em equipamento corporativo, consulte a TI antes de instalar acesso remoto, contêineres ou VPN.
2. É preciso rodar Windows em máquina virtual para algum sistema legado? Se sim, veja a [fase 9](fases/09-windows-vm.md).
3. Dados pessoais e de trabalho ficarão no mesmo perfil de usuário?
4. O firmware (BIOS) será atualizado? O guia mantém a versão de fábrica quando o equipamento funciona bem.
5. O Secure Boot será usado? O Debian o suporta. No modelo de referência ele fica desativado.
6. O repositório `non-free` será habilitado? O guia o mantém desativado até um pacote exigir.

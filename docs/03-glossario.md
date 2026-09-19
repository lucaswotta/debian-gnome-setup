# 03 - Glossário

Termos que aparecem na documentação, explicados de forma direta. Este arquivo cresce
conforme aparecem novos conceitos.

| Termo | Explicação |
|---|---|
| **Distribuição (distro)** | Um "sabor" de Linux: kernel + programas + gerenciador de pacotes. O Debian é uma das mais antigas e é a base de outras, como Ubuntu e Mint. |
| **Kernel** | O núcleo do sistema: conversa com o hardware. O Debian 13 usa o Linux 6.12. |
| **Stable / testing / unstable** | Ramos do Debian. *Stable* (13 "trixie") é o que este guia usa: estável e conservador. |
| **Nome-código (trixie)** | Nome de cada versão do Debian. Aparece nos repositórios. |
| **APT** | Gerenciador de pacotes do Debian. Instala, atualiza e remove programas a partir dos repositórios. |
| **Pacote / `.deb`** | Programa empacotado para o Debian. O instalador do Chrome para Debian, por exemplo, é um `.deb`. |
| **Repositório** | Servidor com pacotes. Definido em `/etc/apt/sources.list`. **Não confundir** com o repositório Git. |
| **`main`, `contrib`, `non-free`, `non-free-firmware`** | Categorias de pacotes. `main` é 100% software livre; `non-free` tem licença restritiva; `non-free-firmware` traz firmwares de hardware. |
| **`sudo`** | Executa um comando como administrador (root). |
| **root** | O superusuário, com poder total sobre o sistema. |
| **GNOME** | O ambiente de área de trabalho (janelas, menus, painel). |
| **Wayland / X11** | Sistemas de exibição gráfica. O Wayland é o mais novo e é o padrão do GNOME atual. |
| **Extensão do GNOME** | Complemento que altera ou acrescenta funções ao GNOME. |
| **Flatpak / Flathub** | Formato alternativo de aplicativos (isolados do sistema) e sua loja principal. |
| **Firmware** | Software embutido em um componente (Wi-Fi, vídeo, BIOS). Precisa de atualização própria. |
| **UEFI / BIOS** | O programa que liga o computador antes do sistema. UEFI é a versão moderna. |
| **Secure Boot** | Recurso do UEFI que só deixa iniciar sistemas assinados. |
| **ext4 / btrfs** | Sistemas de arquivos. O ext4 é o clássico e estável; o btrfs suporta *snapshots* (fotografias do disco). |
| **Swap** | Área do disco usada como memória extra quando a RAM enche. |
| **LUKS** | Criptografia de disco padrão no Linux. |
| **TRIM** | Comando que avisa ao SSD quais blocos estão livres, mantendo o desempenho e a vida útil. |
| **TLP / power-profiles-daemon** | Ferramentas de economia de energia em notebooks. |
| **SSH** | Protocolo de acesso seguro; aqui, usado com chaves para autenticar no GitHub sem senha. |
| **Git / GitHub** | Git: controle de versão. GitHub: serviço online que hospeda repositórios Git. |
| **`PATH`** | Lista de pastas onde o terminal procura comandos. O Claude Code vive em `~/.local/bin`, que foi adicionada ao `PATH`. |
| **Commit** | Registro de um conjunto de mudanças no histórico do Git, com uma mensagem que explica o motivo. |
| **`main`** | Branch principal de um repositório Git. Neste projeto, todo o trabalho vai direto nela. |
| **Remoto (`origin`)** | Cópia do repositório em um servidor, como o GitHub. `git push` envia os commits para ele. |
| **E-mail `noreply`** | Endereço fornecido pelo GitHub para assinar commits sem expor o e-mail pessoal. |

# 03 - Glossário

Termos usados na documentação, em ordem alfabética.

| Termo | Explicação |
|---|---|
| **`.sources` (deb822)** | Formato moderno dos arquivos de repositório do APT, com um campo por linha e `Signed-By` para limitar cada chave ao seu repositório. |
| **APT** | Gerenciador de pacotes do Debian. Instala, atualiza e remove programas a partir dos repositórios. |
| **Branch `main`** | Ramo principal de um repositório Git. Neste projeto, todo o trabalho vai direto nele. |
| **Chaveiro (GNOME Keyring)** | Cofre do GNOME, protegido por senha, para credenciais como senhas de Wi-Fi e de VPN e tokens. É destravado no login. |
| **Commit** | Registro de um conjunto de mudanças no histórico do Git, com uma mensagem que explica o motivo. |
| **Componentes do APT (`main`, `contrib`, `non-free`, `non-free-firmware`)** | Categorias de pacotes. `main` é software livre; `non-free` tem licença restritiva; `non-free-firmware` traz firmwares de hardware. |
| **Condição de corrida (*race condition*)** | Falha que depende de qual de dois processos termina primeiro. Aparece de forma intermitente, e o que falha muda de uma vez para outra. |
| **Contêiner** | Ambiente isolado que executa um programa com tudo o que ele precisa. Comum para rodar bancos de dados e serviços de desenvolvimento. |
| **Distribuição (distro)** | Um "sabor" de Linux: kernel, programas e gerenciador de pacotes. O Debian é uma das mais antigas e serve de base para outras. |
| **Docker (Engine, Compose, Buildx)** | Motor de contêineres. O Compose descreve vários contêineres num arquivo, e o Buildx constrói imagens. |
| **E-mail `noreply`** | Endereço fornecido pelo GitHub para assinar commits sem expor o e-mail pessoal. |
| **ext4 / btrfs** | Sistemas de arquivos. O ext4 é o clássico e estável. O btrfs oferece *snapshots*, compressão e verificação de dados. |
| **Extensão do GNOME** | Complemento que altera ou acrescenta funções ao GNOME. |
| **Faixa em abas (*Tabbed*)** | Modo de interface do LibreOffice com a barra de ferramentas organizada em abas, como a faixa de opções do Office. |
| **Fingerprint (impressão digital da chave)** | Resumo curto e único de uma chave GPG. Serve para conferir que a chave baixada é a que o fabricante publicou. |
| **Firmware** | Software embutido em um componente (Wi-Fi, vídeo, BIOS). Tem atualização própria. |
| **Flatpak / Flathub** | Formato alternativo de aplicativos, isolados do sistema, e sua loja principal. |
| **`fontconfig`** | Sistema que escolhe e substitui fontes no Linux. As regras do usuário ficam em `~/.config/fontconfig/conf.d/`. |
| **Fontes métricas compatíveis** | Fontes livres que ocupam o mesmo espaço que as da Microsoft (Liberation, Carlito e Caladea), mantendo o layout dos documentos. |
| **`fstab`** | Arquivo `/etc/fstab`. Define quais discos são montados na inicialização e com quais opções. |
| **`fwupd` / LVFS** | O `fwupd` atualiza firmwares pelo catálogo LVFS (Linux Vendor Firmware Service). Nem todo fabricante ou modelo está no catálogo. |
| **Gerenciador de versões (fnm, uv, SDKMAN)** | Ferramenta que instala e alterna versões de uma linguagem na pasta pessoal, sem `sudo` e sem afetar o sistema. |
| **Git / GitHub** | Git: controle de versão. GitHub: serviço online que hospeda repositórios Git. |
| **GNOME** | Ambiente de área de trabalho: janelas, menus e painel. |
| **GPU híbrida (PRIME)** | Notebook com duas placas de vídeo: uma integrada, econômica, e uma dedicada, mais potente, usada sob demanda. |
| **Grupo (`docker`, `wireshark`)** | Conjunto de usuários com uma permissão. Entrar num grupo só vale a partir do próximo login. O grupo `docker` equivale a ser administrador. |
| **Journal / `journalctl`** | Log central do sistema. Para lê-lo sem `sudo`, o usuário precisa estar no grupo `systemd-journal`. |
| **Kernel** | Núcleo do sistema, que se comunica com o hardware. O Debian 13 usa o Linux 6.12. |
| **LTS (*Long Term Support*)** | Versão com suporte de longo prazo. É a recomendada para uso em produção e no dia a dia. |
| **Mascarar (unidade do systemd)** | Impedir que uma unidade inicie, mesmo por dependência, apontando-a para `/dev/null`. Desfaz-se com `unmask`. |
| **Menu rápido** | Painel do GNOME no canto superior direito, com Wi-Fi, VPN, volume e energia. |
| **Modelo (*template*) padrão** | Documento-base que o LibreOffice usa em *Arquivo > Novo*. Define fonte, espaçamento e página dos documentos novos. |
| **Montagem (*mount*)** | Ligar um disco ou partição a uma pasta do sistema. Enquanto não é montado, o disco não é acessível. |
| **NetworkManager** | Serviço que gerencia Wi-Fi, cabo e VPN. As Configurações de Rede do GNOME são a interface dele. |
| **Nome-código (trixie)** | Nome de cada versão do Debian. Aparece nos arquivos de repositório. |
| **OOXML** | Formato dos arquivos do Office (`.docx`, `.xlsx` e `.pptx`). O LibreOffice lê e grava esses formatos. |
| **Pacote / `.deb`** | Programa empacotado para o Debian. O instalador do Chrome para Debian, por exemplo, é um `.deb`. |
| **`PATH`** | Lista de pastas onde o terminal procura comandos. O Claude Code fica em `~/.local/bin`, que foi adicionada ao `PATH`. |
| **Perfil do LibreOffice** | Pasta `~/.config/libreoffice/4/user` com as preferências do usuário. O `registrymodifications.xcu` guarda as mudanças em relação ao padrão. |
| **Porta em escuta** | Porta de rede em que um serviço aguarda conexões. `0.0.0.0` aceita de qualquer interface, e `127.0.0.1` só da própria máquina. |
| **`power-profiles-daemon`** | Serviço com os perfis de energia (economia, equilibrado e desempenho) usados pelo menu do GNOME. O TLP faz papel parecido, mas os dois não devem ser usados juntos. |
| **Remoto (`origin`)** | Cópia do repositório em um servidor, como o GitHub. O `git push` envia os commits para ele. |
| **Repositório (APT)** | Servidor com pacotes, definido em `/etc/apt/sources.list` e `/etc/apt/sources.list.d/`. Não confundir com o repositório Git. |
| **Repositório do fabricante** | Repositório APT mantido pelo próprio autor do programa, fora do Debian. Exige adicionar a chave do fabricante com `Signed-By`. |
| **root** | Superusuário, com poder total sobre o sistema. |
| **Sandbox (Flatpak)** | Isolamento que limita o que um aplicativo pode acessar (arquivos, dispositivos, rede). As permissões de cada Flatpak são visíveis e ajustáveis. |
| **Secure Boot** | Recurso do UEFI que só permite iniciar sistemas assinados. |
| **SMART** | Autodiagnóstico de discos. Mostra saúde, horas de uso, desgaste e erros. Lido com `smartctl`. |
| **SSH** | Protocolo de acesso seguro. Aqui, usado com chaves para autenticar no GitHub sem senha. |
| **Stable / testing / unstable** | Ramos do Debian. O *stable* (13 "trixie") é o que este guia usa: estável e conservador. |
| **`sudo`** | Executa um comando como administrador (root). |
| **Suspensão** | Estado de baixo consumo que guarda o trabalho na memória e desliga o restante. |
| **Swap** | Área do disco usada como memória extra quando a RAM enche. |
| **TRIM** | Comando que informa ao SSD quais blocos estão livres, mantendo o desempenho e a vida útil. |
| **Túnel dividido (*split tunnel*)** | Modo em que só o tráfego das redes internas passa pela VPN. O restante segue direto pela internet. |
| **UEFI / BIOS** | Programa que liga o computador antes do sistema. O UEFI é a versão moderna. |
| **`unattended-upgrades`** | Pacote que instala atualizações automaticamente, na frequência e nas origens configuradas. |
| **UNO** | Interface de programação do LibreOffice, usada por scripts para criar documentos, ler estilos e alterar configurações. |
| **UPower** | Serviço que informa o estado da bateria e controla o limite de carga. |
| **UUID** | Identificador único de um disco ou partição. Não muda, ao contrário do nome `sda`. |
| **VPN** | Rede privada virtual: um túnel criptografado até a rede da empresa, para acessar recursos internos. |
| **Wayland / X11** | Sistemas de exibição gráfica. O Wayland é o mais novo e é o padrão do GNOME atual. |

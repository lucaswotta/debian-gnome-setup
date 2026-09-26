# 03 - Glossário

Termos usados na documentação, em ordem alfabética.

| Termo | Explicação |
|---|---|
| **`.sources` (deb822)** | Formato moderno dos arquivos de repositório do APT, com um campo por linha e `Signed-By` para limitar cada chave ao seu repositório. |
| **Alias** | Apelido para um comando ou uma sequência de comandos, definido no arquivo de configuração do shell. Exemplo: `gs` para `git status -sb`. |
| **APT** | Gerenciador de pacotes do Debian. Instala, atualiza e remove programas a partir dos repositórios. |
| **Ativação por socket (*socket activation*)** | O `systemd` escuta um socket e só inicia o serviço quando chega a primeira conexão. Poupa memória de serviços pouco usados, como o Docker e o libvirt. |
| **Branch `main`** | Ramo principal de um repositório Git. Neste projeto, todo o trabalho vai direto nele. |
| **Chaveiro (GNOME Keyring)** | Cofre do GNOME, protegido por senha, para credenciais como senhas de Wi-Fi e de VPN e tokens. É destravado no login. |
| **Commit** | Registro de um conjunto de mudanças no histórico do Git, com uma mensagem que explica o motivo. |
| **Componentes do APT (`main`, `contrib`, `non-free`, `non-free-firmware`)** | Categorias de pacotes. `main` é software livre; `non-free` tem licença restritiva; `non-free-firmware` traz firmwares de hardware. |
| **Condição de corrida (*race condition*)** | Falha que depende de qual de dois processos termina primeiro. Aparece de forma intermitente, e o que falha muda de uma vez para outra. |
| **Contêiner** | Ambiente isolado que executa um programa com tudo o que ele precisa. Comum para rodar bancos de dados e serviços de desenvolvimento. |
| **`cron`** | Agendador de tarefas do sistema. O Timeshift usa um trabalho do `cron` para conferir, de hora em hora, se um snapshot está na hora. |
| **Distribuição (distro)** | Um "sabor" de Linux: kernel, programas e gerenciador de pacotes. O Debian é uma das mais antigas e serve de base para outras. |
| **Docker (Engine, Compose, Buildx)** | Motor de contêineres. O Compose descreve vários contêineres num arquivo, e o Buildx constrói imagens. |
| **Dotfiles** | Arquivos de configuração pessoais, geralmente com nome iniciado por ponto, como `~/.zshrc`. Neste projeto, ficam copiados em `dotfiles/`. |
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
| **`gsettings`** | Comando que lê e altera as configurações do GNOME, as mesmas dos aplicativos de Ajustes. `gsettings reset` volta ao padrão. |
| **GTK (3 e 4) e libadwaita** | Bibliotecas de interface do GNOME. A cor de destaque escolhida nas Configurações só vale para aplicativos GTK 4 com libadwaita. Os GTK 3, como o `gnome-terminal` do Debian 13, usam um tema fixo. |
| **`hwdb` (banco de hardware do udev)** | Arquivos em `/etc/udev/hwdb.d/` que ajustam propriedades de dispositivos, como o mapa de teclas de um teclado. `systemd-hwdb update` recompila o banco. |
| **Journal / `journalctl`** | Log central do sistema. Para lê-lo sem `sudo`, o usuário precisa estar no grupo `systemd-journal`. |
| **Kernel** | Núcleo do sistema, que se comunica com o hardware. O Debian 13 usa o Linux 6.12. |
| **KVM e QEMU** | O KVM é o recurso do kernel que usa o processador para rodar máquinas virtuais quase na velocidade real. O QEMU é o programa que emula o resto do computador (disco, rede e vídeo) e usa o KVM. |
| **libvirt** | Serviço que gerencia máquinas virtuais, redes virtuais e pools de armazenamento. O `virsh` e o virt-manager conversam com ele. |
| **Limites do texto** | Linhas que o Writer desenha nos cantos da folha para mostrar a área das margens. Não são impressas. |
| **LTS (*Long Term Support*)** | Versão com suporte de longo prazo. É a recomendada para uso em produção e no dia a dia. |
| **LTSC (*Long-Term Servicing Channel*)** | Canal do Windows com suporte longo e sem atualizações de recursos. Não traz a Microsoft Store e vem com poucos aplicativos embutidos. |
| **Máquina virtual (VM)** | Computador simulado por software, que roda um sistema próprio dentro do sistema real. O disco dela é um arquivo. |
| **Mascarar (unidade do systemd)** | Impedir que uma unidade inicie, mesmo por dependência, apontando-a para `/dev/null`. Desfaz-se com `unmask`. |
| **Menu rápido** | Painel do GNOME no canto superior direito, com Wi-Fi, VPN, volume e energia. |
| **Modelo (*template*) padrão** | Documento-base que o LibreOffice usa em *Arquivo > Novo*. Define fonte, espaçamento e página dos documentos novos. |
| **Montagem (*mount*)** | Ligar um disco ou partição a uma pasta do sistema. Enquanto não é montado, o disco não é acessível. |
| **Nerd Font** | Fonte com milhares de ícones extras (git, pastas, linguagens) que os prompts usam. Sem ela, os ícones aparecem como quadrados. |
| **NetworkManager** | Serviço que gerencia Wi-Fi, cabo e VPN. As Configurações de Rede do GNOME são a interface dele. |
| **`nodatacow`** | Atributo (`chattr +C`) que desliga a cópia na escrita (*copy-on-write*) do `btrfs` em uma pasta ou arquivo. Evita a fragmentação de discos de VM. O que estiver na pasta perde a compressão e a soma de verificação. |
| **Nome-código (trixie)** | Nome de cada versão do Debian. Aparece nos arquivos de repositório. |
| **OOXML** | Formato dos arquivos do Office (`.docx`, `.xlsx` e `.pptx`). O LibreOffice lê e grava esses formatos. |
| **OVMF** | Firmware UEFI de código aberto para máquinas virtuais. Permite iniciar a VM em modo UEFI, com Secure Boot. |
| **Pacote / `.deb`** | Programa empacotado para o Debian. O instalador do Chrome para Debian, por exemplo, é um `.deb`. |
| **Pacote de transição** | Pacote vazio que existe só para levar ao pacote com o novo nome. O `dnsutils`, por exemplo, instala o `bind9-dnsutils`. |
| **Paleta ANSI** | As 16 cores que o terminal define, 8 normais e 8 brilhantes. Os programas pedem a cor pelo número, e o tema do terminal decide o tom exato. |
| **Paleta de cores (`.soc`)** | Arquivo com as cores oferecidas nos seletores do LibreOffice. Paletas próprias ficam em `~/.config/libreoffice/4/user/config/`. |
| **`PATH`** | Lista de pastas onde o terminal procura comandos. Programas instalados só para o usuário costumam ficar em `~/.local/bin`, que precisa estar no `PATH`. |
| **Perfil do LibreOffice** | Pasta `~/.config/libreoffice/4/user` com as preferências do usuário. O `registrymodifications.xcu` guarda as mudanças em relação ao padrão. |
| **Porta em escuta** | Porta de rede em que um serviço aguarda conexões. `0.0.0.0` aceita de qualquer interface, e `127.0.0.1` só da própria máquina. |
| **Powerline** | Estilo de prompt em blocos coloridos ligados por setas. Precisa de uma fonte com esses símbolos, como as Nerd Fonts. |
| **`power-profiles-daemon`** | Serviço com os perfis de energia (economia, equilibrado e desempenho) usados pelo menu do GNOME. O TLP faz papel parecido, mas os dois não devem ser usados juntos. |
| **PPK** | Formato de chave privada do PuTTY e do WinSCP. O Linux usa o formato do OpenSSH, e o `puttygen` converte de um para o outro. |
| **Prompt** | Linha em que o shell espera um comando. Pode mostrar a pasta, a branch do git e outras informações. |
| **Proton** | Camada de compatibilidade da Steam que roda jogos de Windows no Linux. |
| **`qcow2`** | Formato de disco virtual do QEMU. O arquivo cresce conforme o uso, até o tamanho definido. |
| **Remoto (`origin`)** | Cópia do repositório em um servidor, como o GitHub. O `git push` envia os commits para ele. |
| **Repositório (APT)** | Servidor com pacotes, definido em `/etc/apt/sources.list` e `/etc/apt/sources.list.d/`. Não confundir com o repositório Git. |
| **Repositório do fabricante** | Repositório APT mantido pelo próprio autor do programa, fora do Debian. Exige adicionar a chave do fabricante com `Signed-By`. |
| **root** | Superusuário, com poder total sobre o sistema. |
| **`rsync`** | Programa que copia só o que mudou entre duas pastas. O Timeshift o usa para criar os snapshots. |
| **Sandbox (Flatpak)** | Isolamento que limita o que um aplicativo pode acessar (arquivos, dispositivos, rede). As permissões de cada Flatpak são visíveis e ajustáveis. |
| **Scancode** | Número que o teclado envia ao computador quando uma tecla é pressionada. O kernel converte cada scancode em um código de tecla (como `KEY_RO`). Um remapeamento por `hwdb` troca essa conversão. |
| **Secure Boot** | Recurso do UEFI que só permite iniciar sistemas assinados. |
| **SFTP** | Transferência de arquivos por SSH. O aplicativo Arquivos do GNOME abre servidores com `sftp://usuario@endereco/`. |
| **SHA-256** | Resumo de 64 caracteres calculado a partir do conteúdo de um arquivo. Se o resumo do seu arquivo for igual ao publicado, o arquivo está íntegro (`sha256sum`). |
| **Shell (bash e zsh)** | Programa que lê os comandos digitados no terminal. O bash é o padrão do Debian. O zsh oferece sugestões e um completar mais completo. |
| **Slide mestre** | Slide-modelo do Impress que define posição, fonte e cor do título, do texto e do rodapé de todos os slides. |
| **SMART** | Autodiagnóstico de discos. Mostra saúde, horas de uso, desgaste e erros. Lido com `smartctl`. |
| **Snapshot (Timeshift)** | Cópia do sistema em um momento. Serve para voltar a um estado anterior. Os arquivos iguais entre cópias são compartilhados, então cada nova cópia ocupa pouco. |
| **SPICE** | Protocolo de exibição de máquinas virtuais. Com o agente instalado no sistema convidado, leva a área de transferência e ajusta a resolução à janela. |
| **SSH** | Protocolo de acesso remoto seguro. Neste guia, autentica no GitHub e nos servidores com chaves, no lugar de senhas. |
| **Stable / testing / unstable** | Ramos do Debian. O *stable* (13 "trixie") é o que este guia usa: estável e conservador. |
| **Starship** | Programa que desenha o prompt, em qualquer shell, a partir de um arquivo de configuração (`~/.config/starship.toml`). |
| **`sudo`** | Executa um comando como administrador (root). |
| **Super (tecla)** | Nome da tecla Windows no GNOME. `Super+E`, por exemplo, é a tecla Windows com a letra E. |
| **Suspensão** | Estado de baixo consumo que guarda o trabalho na memória e desliga o restante. |
| **Swap** | Área do disco usada como memória extra quando a RAM enche. |
| **Tema de ícones e de cursor** | Conjunto de imagens usadas pelo sistema para os ícones dos aplicativos e para o ponteiro do mouse. Pacotes do Debian ficam em `/usr/share/icons/`, e os do usuário, em `~/.local/share/icons/`. |
| **TRIM** | Comando que informa ao SSD quais blocos estão livres, mantendo o desempenho e a vida útil. |
| **Túnel dividido (*split tunnel*)** | Modo em que só o tráfego das redes internas passa pela VPN. O restante segue direto pela internet. |
| **UEFI / BIOS** | Programa que liga o computador antes do sistema. O UEFI é a versão moderna. |
| **`unattended-upgrades`** | Pacote que instala atualizações automaticamente, na frequência e nas origens configuradas. |
| **UNO** | Interface de programação do LibreOffice, usada por scripts para criar documentos, ler estilos e alterar configurações. |
| **UPower** | Serviço que informa o estado da bateria e controla o limite de carga. |
| **UUID** | Identificador único de um disco ou partição. Não muda, ao contrário do nome `sda`. |
| **UUID de extensão** | Nome único de uma extensão do GNOME, como `caffeine@patapon.info`. Aparece em `gnome-extensions list`. |
| **`virtio`** | Conjunto de dispositivos virtuais rápidos (disco, rede, vídeo e memória), feitos para máquinas virtuais. O sistema convidado precisa do driver `virtio`. |
| **VPN** | Rede privada virtual: um túnel criptografado até a rede da empresa, para acessar recursos internos. |
| **Wayland / X11** | Sistemas de exibição gráfica. O Wayland é o mais novo e é o padrão do GNOME atual. |

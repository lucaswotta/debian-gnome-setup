# Fase 5 - Aplicativos

**Status:** concluída · **Escopo:** depende do perfil de uso. Os lotes são independentes: instale só o que fizer sentido

[← Fase 4](04-notebook.md) · [Roteiro](../02-roteiro.md) · [Fase 6 →](06-gnome.md)

Objetivo: instalar apenas o que será usado, nesta ordem de preferência: pacote do Debian, recurso nativo do GNOME, Flatpak, repositório do fabricante e, por último, script próprio.

O perfil de referência é o de desenvolvimento de software, com uso geral. Troque, remova ou acrescente aplicativos conforme o seu.

- [x] **Navegador:** Google Chrome, pelo pacote `.deb` oficial. O pacote configura o repositório do Google, e as atualizações chegam pelo `apt`.
- [x] **VPN e acesso a servidores:** ver [VPN e acesso a servidores](05b-vpn-e-servidores.md).
- [x] **Lote 1:** Flatpak, base de desenvolvimento, fontes, Wireshark, Meld e Docker.
- [x] **Lote 2:** Go 1.27, Node 24 LTS, TypeScript 7, Python 3.14 e Java 25 LTS.
- [x] **Lote 3:** VS Code, DBeaver e AnyDesk (repositórios dos fabricantes), Postman, SoapUI e Discord (Flatpak).
- [x] **LibreOffice:** configurado para se parecer com o Office. Ver [LibreOffice no estilo do Microsoft Office](05a-libreoffice.md).
- [x] **Lote 4:** VLC, utilitários de diagnóstico e de rede (Debian) e Steam (Flatpak).

Aplicativos adicionais, como GIMP, OBS Studio e Inkscape, entram sob demanda, pelo Debian ou pelo Flatpak.

## Visão geral dos aplicativos

| Bloco | Escolha | Origem |
|---|---|---|
| Base de desenvolvimento | `build-essential` e Meld | Debian |
| Editor de código | Visual Studio Code | Repositório da Microsoft |
| Bancos de dados | DBeaver, para Oracle, MySQL e Postgres | Repositório do fabricante |
| Linguagens | Node 24 LTS, Java 25 LTS, Python 3.14, Go 1.27 e TypeScript 7 | Gerenciadores de versão na pasta pessoal |
| Contêineres | Docker Engine | Repositório oficial do Docker |
| APIs | Postman e SoapUI | Flatpak |
| Comunicação | WhatsApp Web, Teams, Meet e Zoom pelo Chrome. Discord | Navegador e Flatpak |
| Rede e acesso remoto | Wireshark, AnyDesk, `nmap` e `dnsutils` | Debian e repositório do fabricante |
| Diagnóstico | `htop`, `ncdu` e `tree` | Debian |
| Multimídia | VLC | Debian |
| Jogos | Steam, com a biblioteca em `/mnt/ssd/Jogos` | Flatpak |
| Escritório | LibreOffice, configurado para se parecer com o Office | Debian |
| Captura de tela | Recurso nativo do GNOME | GNOME |

O Debian 13 traz versões antigas de algumas linguagens (o Node 20 e o Go 1.24, por exemplo, deixam de receber suporte do projeto de origem), por isso o lote 2 usa
gerenciadores de versão na pasta pessoal: `fnm` para Node, SDKMAN para Java, `uv` para Python e o pacote oficial do Go.
O Python do sistema fica intocado.

## Lote 1: base de desenvolvimento, fontes e Docker

```bash
# Flatpak e Flathub
sudo apt install -y flatpak gnome-software-plugin-flatpak
sudo flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo

# Compilador, Meld, fontes e português do LibreOffice
sudo apt install -y build-essential meld fonts-crosextra-carlito fonts-crosextra-caladea fonts-noto-core fonts-firacode \
  libreoffice-help-pt-br hyphen-pt-br mythes-pt-br

# Wireshark, com captura permitida ao grupo wireshark
echo "wireshark-common wireshark-common/install-setuid boolean true" | sudo debconf-set-selections
sudo apt install -y wireshark && sudo usermod -aG wireshark "$USER"

# Habilitar o contrib no formato moderno de repositórios
sudo cp -a /etc/apt/sources.list /etc/apt/sources.list.bak-$(date +%F)
sudo apt modernize-sources -y
sudo sed -i 's/^Components: main non-free-firmware$/Components: main contrib non-free-firmware/' /etc/apt/sources.list.d/debian.sources
sudo sed -i 's/^Types: deb deb-src$/Types: deb/' /etc/apt/sources.list.d/debian.sources
sudo apt update

# Fontes da Microsoft (aceita o EULA delas)
echo "ttf-mscorefonts-installer msttcorefonts/accepted-mscorefonts-eula select true" | sudo debconf-set-selections
sudo apt install -y ttf-mscorefonts-installer && sudo fc-cache -f

# Repositório oficial do Docker e instalação
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
printf 'Types: deb\nURIs: https://download.docker.com/linux/debian\nSuites: trixie\nComponents: stable\nArchitectures: amd64\nSigned-By: /etc/apt/keyrings/docker.asc\n' \
  | sudo tee /etc/apt/sources.list.d/docker.sources > /dev/null
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Docker sem sudo e rede fora das faixas da VPN
sudo usermod -aG docker "$USER"
printf '{\n  "bip": "100.64.0.1/24",\n  "default-address-pools": [\n    {"base": "100.65.0.0/16", "size": 24}\n  ]\n}\n' \
  | sudo tee /etc/docker/daemon.json > /dev/null
sudo systemctl restart docker
```

| Parte | O que faz |
|---|---|
| `flatpak remote-add ... flathub` | Adiciona a loja Flathub. Os aplicativos Flatpak ficam no disco do sistema. |
| `fonts-crosextra-carlito` e `-caladea` | Substituem o Calibri e o Cambria com as mesmas métricas, então o layout dos documentos se mantém. |
| `install-setuid true` | Permite capturar pacotes sem ser administrador, para quem estiver no grupo `wireshark`. |
| `apt modernize-sources` | Converte `/etc/apt/sources.list` para o formato `.sources`, recomendado no Debian 13. |
| `Components: main contrib ...` | Habilita o `contrib`, que contém o `ttf-mscorefonts-installer`. |
| `Types: deb` | Remove o código-fonte (`deb-src`), que o `apt update` baixaria sem necessidade. |
| `ttf-mscorefonts-installer` | Baixa as fontes da Microsoft (Arial, Times New Roman, Verdana e outras) e as instala. |
| `Signed-By: ...docker.asc` | Só aceita pacotes assinados pela chave do Docker. Confira o fingerprint antes de usar. |
| `usermod -aG docker` | Permite usar o Docker sem `sudo`. Vale a partir do próximo login. |
| `bip` e `default-address-pools` | Fixam as redes do Docker na faixa `100.64.0.0/10`, fora das faixas da VPN. |

Como conferir:

```bash
fc-match Calibri; fc-match Cambria; fc-match Arial        # Carlito, Caladea e Arial
gpg --show-keys --with-fingerprint /etc/apt/keyrings/docker.asc   # 9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88
docker --version && docker compose version
docker run --rm hello-world                               # sem sudo, depois de um novo login
ip -br addr show docker0                                  # 100.64.0.1/24
ip route get 172.17.5.5                                   # com a VPN ativa: dev do túnel, não docker0
```

Como desfazer:

```bash
sudo apt remove docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo rm /etc/apt/sources.list.d/docker.sources /etc/apt/keyrings/docker.asc /etc/docker/daemon.json
sudo apt remove ttf-mscorefonts-installer wireshark meld flatpak
sudo rm /etc/apt/sources.list.d/debian.sources && sudo cp -a /etc/apt/sources.list.bak-AAAA-MM-DD /etc/apt/sources.list && sudo apt update
```

Observações:

- **Fontes:** o Calibri e o Cambria originais não têm fonte gratuita legal. O Carlito e o Caladea são clones métricos e
  mantêm o layout. Arial, Times New Roman e as demais fontes da Microsoft vêm do `ttf-mscorefonts-installer`,
  que exige aceitar o EULA delas.
- **Docker e VPN:** uma VPN corporativa costuma rotear as faixas privadas inteiras (`10.0.0.0/8`, `172.16.0.0/12` e
  `192.168.0.0/16`), e o Docker usa `172.17.0.0/16` até `172.31.0.0/16` e `192.168.0.0/16`. Com a VPN ativa, um pacote para
  `172.17.x.x` vai para o `docker0`, e não para a VPN. Confira com `ip route get 172.17.5.5`. A correção é fixar a rede do
  Docker numa faixa livre (`bip` e `default-address-pools`). Escolha a faixa comparando com as rotas reais da VPN.
- **Grupo `docker`:** equivale a ser administrador, porque quem controla o Docker pode montar o disco inteiro num contêiner.
  O modo *rootless* é mais seguro, mas limita a rede dos contêineres.
- **Grupos e sessão:** `docker` e `wireshark` só valem depois de sair da sessão gráfica e entrar de novo. Fechar o terminal não basta.
  Um novo login também faz os aplicativos Flatpak aparecerem no menu.
- **Docker e atualizações automáticas:** o Docker não entra no `unattended-upgrades`, porque atualizar o serviço o reinicia e para os contêineres.
  Atualize com o `sudo apt full-upgrade` semanal.
- **`deb-src`:** removido de `debian.sources` (`Types: deb`). Sem necessidade de código-fonte, o `apt update` baixa menos índices.
- **Docker sob demanda:** o serviço não sobe no boot, o que poupa memória e tempo de inicialização. O `docker.socket` fica ativo e acorda o serviço no primeiro comando `docker`,
  com uma pequena espera. Contêineres com política de reinício automático só voltam depois desse primeiro comando.
  `sudo systemctl disable --now docker.service containerd.service` liga o modo sob demanda, e `sudo systemctl enable --now docker.service containerd.service` o desfaz.
- **Repositórios externos:** o `debian.sources` e o `docker.sources` usam `Signed-By`, que limita cada chave ao seu repositório.

## Lote 2: linguagens

```bash
# Go: arquivo oficial, com conferência do SHA-256 publicado em go.dev/dl
# (versão e checksum abaixo são de exemplo: use os valores atuais de go.dev/dl)
curl -fsSL https://go.dev/dl/go1.27.1.linux-amd64.tar.gz -o go.tar.gz
echo "63d339f0da5ab53635a56f2490a7984dfe12dfcff22ad749f63edaf590168445  go.tar.gz" | sha256sum -c -
mkdir -p ~/.local && tar -C ~/.local -xzf go.tar.gz

# fnm, Node 24 LTS e TypeScript 7 (o instalador é lido antes de ser executado)
curl -fsSL https://fnm.vercel.app/install -o fnm-install.sh
bash fnm-install.sh --skip-shell
export PATH="$HOME/.local/share/fnm:$PATH" && eval "$(fnm env --shell bash)"
fnm install 24 && fnm default 24
npm install -g typescript@7

# uv e Python 3.14
curl -LsSf https://astral.sh/uv/install.sh -o uv-install.sh
UV_NO_MODIFY_PATH=1 sh uv-install.sh
uv python install 3.14

# SDKMAN e Java 25 LTS (Temurin). O SDKMAN exige zip e unzip
sudo apt install -y zip
curl -fsSL https://get.sdkman.io -o sdkman-install.sh
bash sdkman-install.sh
source ~/.sdkman/bin/sdkman-init.sh
sdk install java 25.0.4-tem

# PATH e integração do fnm no shell
cp -a ~/.bashrc ~/.bashrc.bak-$(date +%F)
cat >> ~/.bashrc <<'EOF'
export PATH="$HOME/.local/go/bin:$HOME/go/bin:$HOME/.local/share/fnm:$PATH"
eval "$(fnm env --use-on-cd --shell bash)"
EOF
```

| Parte | O que faz |
|---|---|
| `sha256sum -c` | Confere que o arquivo baixado é o publicado pelo Go. |
| `--skip-shell` | Impede que o instalador do fnm edite o `~/.bashrc`. A integração é feita à parte, de forma visível. |
| `fnm default 24` | Define o Node 24 como padrão em todo terminal novo. |
| `UV_NO_MODIFY_PATH=1` | Impede que o uv altere o `PATH`. O `~/.local/bin` já está nele. |
| `uv python install 3.14` | Instala o Python 3.14 na pasta pessoal, sem tocar no Python do sistema. |
| `fnm env --use-on-cd` | Troca a versão do Node ao entrar numa pasta com `.node-version` ou `.nvmrc`. |
| `sdk install java 25.0.4-tem` | Instala o Temurin 25 LTS e o define como padrão, com `JAVA_HOME` apontando para ele. |

Como conferir:

```bash
go version && node --version && npm --version && tsc --version
uv --version && python3.14 --version
java -version && javac -version
python3 --version        # continua sendo o Python do sistema
```

Como desfazer:

```bash
rm -rf ~/.local/go ~/go
rm -rf ~/.local/share/fnm ~/.local/state/fnm
rm -f ~/.local/bin/uv ~/.local/bin/uvx ~/.local/bin/python3.14 && rm -rf ~/.local/share/uv
rm -rf ~/.sdkman
cp -a ~/.bashrc.bak-AAAA-MM-DD ~/.bashrc
```

Observações:

- **Sem `sudo`:** tudo fica na pasta pessoal, e o Python do sistema não é tocado. O `python3.14` é um comando à parte.
- **Ler antes de executar:** o instalador do fnm edita o `~/.bashrc` se não receber `--skip-shell`, o do uv altera o `PATH`
  sem `UV_NO_MODIFY_PATH=1`, e o do SDKMAN sempre acrescenta um bloco ao `~/.bashrc`. O do fnm não confere checksum.
- **TypeScript global:** fica dentro da versão do Node. Ao instalar outro Node, reinstale com `npm install -g typescript@7`,
  ou use o TypeScript de cada projeto.
- **Node 24 LTS:** quando o Node 26 virar LTS, instale-o com `fnm install 26` e `fnm default 26`.
- **Go:** o `GOTOOLCHAIN=auto` baixa sozinho o toolchain que o `go.mod` de um projeto pedir.
- **Python:** não use `pip install` no Python do sistema, que é protegido (PEP 668). Use `uv venv`, `uv run` ou `pipx`.
- **SDKMAN:** o instalador sempre acrescenta um bloco ao `~/.bashrc`, que deve ficar no **fim** do arquivo. Só confere a integridade do zip,
  e não a autenticidade. Exige `zip` e `unzip`.
- **Java:** outras versões se instalam com `sdk install java <identificador>` e se alternam com `sdk use` ou `sdk default`.
  `sdk list java` mostra os identificadores. Maven e Gradle também vêm pelo SDKMAN (`sdk install maven`).

## Lote 3: aplicativos

Repositórios dos fabricantes (VS Code, AnyDesk e DBeaver) e Flatpak (Postman, SoapUI e Discord).

```bash
# 1. Baixar as chaves dos fabricantes e conferir o fingerprint
mkdir ~/lote3 && cd ~/lote3
curl -fsSL https://packages.microsoft.com/keys/microsoft.asc -o microsoft.asc
curl -fsSL https://keys.anydesk.com/repos/DEB-GPG-KEY -o anydesk.asc
curl -fsSL https://dbeaver.io/debs/dbeaver.gpg.key -o dbeaver.asc
gpg --show-keys --with-fingerprint microsoft.asc anydesk.asc dbeaver.asc

# 2. Provar que cada chave assina o repositório do fabricante
G=$(mktemp -d); chmod 700 $G
gpg --homedir $G --import microsoft.asc
curl -fsSL https://packages.microsoft.com/repos/code/dists/stable/InRelease -o $G/InRelease
gpg --homedir $G --verify $G/InRelease        # "Good signature"; repita para AnyDesk e DBeaver

# 3. Arquivos de repositório (formato .sources), por exemplo vscode.sources
#    Types: deb
#    URIs: https://packages.microsoft.com/repos/code
#    Suites: stable
#    Components: main
#    Architectures: amd64
#    Signed-By: /etc/apt/keyrings/microsoft.asc
#    (AnyDesk: URIs https://deb.anydesk.com, Suites all, Components main)
#    (DBeaver: URIs https://dbeaver.io/debs/dbeaver-ce, Suites /, sem Components)

# 4. Instalar chaves e repositórios
sudo install -d -m 755 /etc/apt/keyrings
sudo install -m 644 microsoft.asc anydesk.asc dbeaver.asc /etc/apt/keyrings/
sudo install -m 644 vscode.sources anydesk.sources dbeaver.sources /etc/apt/sources.list.d/
sudo apt update

# 5. Instalar (o debconf impede o pacote do VS Code de registrar o repositório por conta própria)
echo "code code/add-microsoft-repo boolean false" | sudo debconf-set-selections
sudo apt install -y code anydesk dbeaver-ce

# 6. Postman, SoapUI e Discord
sudo flatpak install -y flathub com.getpostman.Postman org.soapui.SoapUI com.discordapp.Discord

# 7. Incluir os repositórios nas atualizações automáticas: o arquivo 52unattended-upgrades-local passa a ter as regras
#    do Chrome (já existente) e estas três, uma por linha, no formato Unattended-Upgrade::Origins-Pattern:: "..."
#    origin=code stable,codename=stable
#    origin=philandro Software GmbH,codename=all
#    site=dbeaver.io                     (o repositório do DBeaver não informa Origin nem Label)
sudo install -m 644 52unattended-upgrades-local /etc/apt/apt.conf.d/52unattended-upgrades-local
sudo unattended-upgrade --dry-run --debug
```

| Fabricante | Fingerprint da chave | Repositório |
|---|---|---|
| Microsoft (VS Code) | `BC52 8686 B50D 79E3 39D3 721C EB3E 94AD BE12 29CF` | `packages.microsoft.com/repos/code` |
| AnyDesk | `06B5 EA2F AE20 8E7C DA97 61DC A2FB 21D5 A877 2835` | `deb.anydesk.com` |
| DBeaver | `BDFB 19F6 8151 4B43 875D 16FA 132C 13A8 A330 F403` | `dbeaver.io/debs/dbeaver-ce` |

Como conferir:

```bash
code --version && dbeaver --help >/dev/null && dpkg -s anydesk | grep Status
flatpak list --app
ls /etc/apt/sources.list.d/                      # sem vscode.list nem dbeaver.list duplicados
sudo unattended-upgrade --dry-run --debug        # as origens novas aparecem em "origens permitidas"
```

Como desfazer:

```bash
sudo apt remove code anydesk dbeaver-ce
sudo rm /etc/apt/sources.list.d/{vscode,anydesk,dbeaver}.sources /etc/apt/keyrings/{microsoft,anydesk,dbeaver}.asc
sudo flatpak uninstall com.getpostman.Postman org.soapui.SoapUI com.discordapp.Discord
```

Observações:

- **Conferência das chaves:** só a Microsoft publica o fingerprint de forma amplamente conhecida. Para os outros, a garantia vem do
  download por HTTPS do domínio do fabricante e de a chave **assinar de fato** os metadados do repositório (`gpg --verify`).
- **Teste em área isolada:** o `apt-get` aceita `-o Dir::State::Lists=... -o Dir::Etc::sourceparts=...` e roda sem `sudo`, o que permite
  testar um repositório novo sem tocar em `/etc`.
- **Duplicidade de repositório:** o pacote `code` registra o repositório da Microsoft sozinho, o que duplicaria o `vscode.sources`.
  O `debconf-set-selections` acima impede isso. Confira com `apt update`, que avisa de destinos configurados duas vezes.
- **Erro `Unit anydesk.service not loaded`:** aparece na instalação, porque o pacote tenta parar um serviço que ainda não existe.
  É inofensivo.
- **AnyDesk:** o pacote instala e habilita um serviço que escuta portas de entrada. Ver [AnyDesk sob demanda](#anydesk-sob-demanda).
- **DBeaver:** instala em `/usr/share/dbeaver-ce`, com um Java embutido (OpenJDK 25), e independe do SDKMAN.
- **Flatpak:** cada aplicativo pede uma versão diferente da base (24.08, 25.08 e 26.08), e cada uma ocupa cerca de 700 MB, além do Mesa.
  Os três aplicativos, com as bases, ocupam cerca de 5 GB. O Postman baixa o binário do fabricante na instalação (*extra-data*).
- **Permissões dos Flatpak:** o SoapUI só acessa Documentos. O Postman acessa a pasta pessoal inteira. O Discord acessa Downloads e
  **todos os dispositivos** (câmera e microfone). Dá para restringir com `flatpak override`.
- **Docker fora das atualizações automáticas:** os quatro repositórios externos restantes (Chrome, VS Code, AnyDesk e DBeaver) entram;
  o Docker não, porque atualizar o serviço reinicia os contêineres.
- **VS Code e a Fira Code:** em `~/.config/Code/User/settings.json`, use `editor.fontFamily`, `editor.fontLigatures: true` e
  `terminal.integrated.fontFamily`.

### AnyDesk sob demanda

O pacote instala e habilita um serviço (`anydesk --service`, como `root`) que escuta em `0.0.0.0:7070` (TCP) e `50001` (UDP), para
conexões diretas de entrada, e um autostart que abre o ícone da bandeja em todo login. Para usar o AnyDesk só de saída e só com o
aplicativo aberto:

```bash
sudo systemctl disable --now anydesk       # desliga o serviço e impede que suba no boot
mkdir -p ~/.config/autostart
printf '[Desktop Entry]\nType=Application\nName=AnyDesk Tray\nHidden=true\nX-GNOME-Autostart-enabled=false\n' \
  > ~/.config/autostart/anydesk_global_tray.desktop     # a bandeja deixa de abrir no login
```

Sem o serviço do sistema, o aplicativo abre um serviço local do próprio usuário (`--local-service`, sem `root`), registra-se na rede
do AnyDesk e o encerra sozinho ao fechar (`Initiating auto-shutdown` no registro `~/.anydesk/anydesk.trace`).

Como conferir:

```bash
systemctl is-active anydesk               # inactive
pgrep -c -x anydesk                       # 0 com o aplicativo fechado
ss -ltn | grep -c ':7070 '                # 0 com o aplicativo fechado
```

Como desfazer: `sudo systemctl enable --now anydesk` e `rm ~/.config/autostart/anydesk_global_tray.desktop`.

Observações:

- O registro do aplicativo mostra que ele se registra na rede do AnyDesk e se encerra sozinho ao fechar. A conexão de saída depende de uma
  máquina de destino e não foi validada neste guia.
- Se algo exigir o serviço do sistema (acesso sem supervisão, por exemplo), reative-o com o `enable --now` acima.

## Lote 4: multimídia, utilitários e Steam

| Bloco | Programas | Origem |
|---|---|---|
| Multimídia | VLC | Debian |
| Diagnóstico | `htop` (processos), `ncdu` (uso do disco), `tree` (árvore de pastas) e `dnsutils` (`dig` e `nslookup`) | Debian |
| Rede | `nmap` | Debian |
| Jogos | Steam | Flatpak |

O VLC entra pelo Debian, sem repositório externo. O Steam usa o Flatpak porque a base do Flatpak já traz as bibliotecas de 32 bits que os jogos pedem,
sem ativar a arquitetura `i386` no sistema.

```bash
apt-get -s install vlc htop ncdu tree dnsutils nmap        # simulação, sem root: só pacotes novos e nenhuma remoção
sudo apt-get install -y vlc htop ncdu tree dnsutils nmap
sudo flatpak install -y flathub com.valvesoftware.Steam
flatpak override --user --filesystem=/mnt/ssd/Jogos com.valvesoftware.Steam      # a Steam passa a enxergar a pasta de jogos
```

Como conferir:

```bash
vlc --version | head -1
dig -v && nmap --version | head -1
flatpak list --app --columns=application | grep Steam
flatpak override --user --show com.valvesoftware.Steam                           # filesystems=/mnt/ssd/Jogos;
```

Como desfazer: `sudo apt remove vlc htop ncdu tree dnsutils nmap` e `sudo flatpak uninstall com.valvesoftware.Steam`.
O `flatpak override --user --reset com.valvesoftware.Steam` remove a permissão da pasta.

Observações:

- **Biblioteca no SSD extra:** na Steam, em *Configurações > Armazenamento*, adicione `/mnt/ssd/Jogos` como pasta de biblioteca. O disco é `btrfs`
  e fica fora do NVMe do sistema.
- **Vídeo:** a Intel atende o uso comum. Para acionar a AMD num jogo, a opção de inicialização `DRI_PRIME=1 %command%` *(proposta)* segue a mesma lógica
  descrita em "GPU híbrida" na [fase 4](04-notebook.md). A placa é de entrada e roda só jogos leves ou antigos.
- **`dnsutils`:** é um pacote de transição. Quem instala o `dig` e o `nslookup` é o `bind9-dnsutils`.
- **`nmap`:** escaneie só equipamentos próprios ou com autorização. Uma varredura na rede corporativa pode disparar alertas da TI.
- **VLC como `root`:** o VLC se recusa a rodar com `sudo`. Confira a versão como usuário comum.

## Serviços e portas em escuta

Um serviço instalado junto com o sistema ou com um aplicativo pode abrir portas sem que isso fique evidente. Para listar o que está em escuta:

```bash
ss -ltn      # portas TCP em escuta
ss -lun      # portas UDP em escuta
```

`0.0.0.0` e `[::]` aceitam conexões de qualquer interface. `127.0.0.1` e `[::1]` aceitam só da própria máquina. O `ss -p`, sem `sudo`,
só mostra os processos do próprio usuário, então serviços do `root` aparecem sem nome.

O instalador do Debian pode ativar um servidor SSH. Para mantê-lo instalado, mas desligado, e ligá-lo só quando precisar:

```bash
sudo systemctl disable --now ssh ssh.socket     # desliga agora e impede que suba no boot
sudo systemctl start ssh                        # liga sob demanda (não sobrevive ao reinício)
sudo systemctl stop ssh                         # desliga ao terminar
```

Com isso, a única porta em escuta fica sendo a do serviço de impressão, restrita à própria máquina.

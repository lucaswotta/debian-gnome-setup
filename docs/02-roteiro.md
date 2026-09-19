# 02 - Roteiro

Legenda: `[x]` feito · `[ ]` pendente · `[~]` em discussão.

Regras:

- Uma fase por vez, explicada antes de executar.
- Comando com `sudo` só roda depois de confirmação.
- Só vira script (`scripts/`) o que já foi feito à mão e entendido.
- Cada fase termina com uma verificação e um registro neste arquivo.

---

## Fase 0 - Base mínima (concluída)

- [x] Instalar o Debian 13 com GNOME (UEFI, raiz em ext4, swap de 12 GB)
- [x] Configurar idioma pt_BR, fuso America/Sao_Paulo e teclado ABNT2
- [x] Colocar o usuário no grupo `sudo`
- [x] Instalar o `curl`
- [x] Instalar o Claude Code

```bash
sudo apt update && sudo apt install -y curl
curl -fsSL https://claude.ai/install.sh | bash
```

| Comando | O que faz |
|---|---|
| `sudo` | Executa o comando seguinte como administrador. |
| `apt update` | Baixa a lista atualizada de pacotes. Não instala nem atualiza nada. |
| `apt install -y curl` | Instala o `curl`. O `-y` responde "sim" às confirmações. |
| `&&` | Executa o próximo comando só se o anterior der certo. |
| `curl -fsSL URL` | Baixa o conteúdo de uma URL. `-f` falha em erro HTTP, `-s` fica em silêncio, `-S` mostra erros, `-L` segue redirecionamentos. |
| `\| bash` | Entrega o conteúdo baixado ao `bash` para executar. |

Aprendizados:

- O `apt` precisa de administrador. Sem `sudo`, o `apt update` falha por permissão.
- Se o `sudo` disser que o usuário não pode usá-lo, adicione-o ao grupo `sudo` como `root` (`su -`).
- `curl ... | bash` executa o que o site enviar. Use só com fontes confiáveis e leia o script nos outros casos.
- O Claude Code foi instalado só para o usuário (`~/.local/bin`), sem `sudo`.

---

## Fase 1 - Git e GitHub (concluída)

Objetivo: atualizar o sistema e versionar este guia.

- [x] Atualizar o sistema
- [x] Instalar `git` e `gh` (CLI do GitHub)
- [x] Configurar a identidade do git
- [x] Autenticar no GitHub com chave SSH
- [x] Ligar a pasta local ao repositório remoto e publicar o primeiro commit

```bash
sudo apt update && sudo apt full-upgrade -y && sudo apt install -y git gh

git config --global user.name "Seu Nome"
git config --global user.email "ID+usuario@users.noreply.github.com"
git config --global init.defaultBranch main

gh auth login --hostname github.com --git-protocol ssh --web
gh api meta --jq '.ssh_keys[] | "github.com " + .' >> ~/.ssh/known_hosts
ssh -T git@github.com

git init
git remote add origin git@github.com:USUARIO/REPOSITORIO.git
git add -A && git commit
git push -u origin main
```

| Comando | O que faz |
|---|---|
| `apt full-upgrade` | Atualiza todos os pacotes, instalando ou removendo dependências quando preciso. |
| `git config --global` | Grava a configuração do git para o usuário, em `~/.gitconfig`. |
| `gh auth login` | Faz login no GitHub e cria a chave SSH. O `--web` autoriza pelo navegador. |
| `gh api meta` | Lista as chaves oficiais do servidor do GitHub. Gravadas em `known_hosts`, evitam confirmar a chave "às cegas" na primeira conexão. |
| `ssh -T git@github.com` | Testa a autenticação. A resposta esperada é `Hi USUARIO! You've successfully authenticated`. |
| `git init` e `git remote add origin` | Iniciam o repositório local e apontam para o remoto. Usados no lugar de `git clone` porque a pasta já tinha conteúdo. |

Como conferir:

```bash
apt list --upgradable      # sem pacotes pendentes
git config --global -l     # nome, e-mail e branch padrão
gh auth status             # logado, protocolo ssh
git status -sb             # main acompanhando origin/main
```

Aprendizados:

- **Repositório público:** o e-mail de cada commit fica visível. Use o endereço `noreply` do GitHub
  (`ID+usuario@users.noreply.github.com`). O ID sai de `gh api user --jq .id`.
- **Chave SSH:** proteja com senha (passphrase).
- **Trabalho direto na `main`:** neste repositório de uso individual não há branches nem PRs.

---

## Fase 2 - Base do sistema (em andamento)

- [x] **TRIM do SSD:** já vem ativo, com o `fstrim.timer` rodando toda semana.
- [x] **Catálogo de firmware:** atualizado com `fwupd`. Nenhum componente tem atualização no LVFS.
- [ ] **BIOS:** comparar a versão instalada com a mais recente no site de suporte da Lenovo.
- [~] **`contrib` e `non-free`:** adiados. Só entram quando um pacote exigir (fontes da Microsoft, alguns codecs).
- [ ] **Disco SATA extra:** ver detalhes abaixo.

```bash
sudo fwupdmgr refresh --force
fwupdmgr get-updates
```

| Comando | O que faz |
|---|---|
| `fwupdmgr refresh --force` | Baixa do LVFS a lista atualizada de firmwares. Só atualiza o cache e não instala nada. |
| `fwupdmgr get-updates` | Mostra, por dispositivo, se há atualização de firmware disponível. |

Como conferir:

```bash
systemctl is-enabled fstrim.timer      # enabled
lsblk -D -o NAME,DISC-GRAN,DISC-MAX    # discos com suporte a TRIM
cat /sys/class/dmi/id/bios_version     # versão da BIOS instalada
```

Aprendizados:

- **Firmware:** o LVFS não tem todos os modelos. "Sem atualização" no `fwupd` não garante que a BIOS seja a última.
- **Atualização de BIOS:** só com o notebook na tomada e sem desligar durante o processo.
- **Escada:** um repositório novo só entra quando há um pacote que precise dele.

---

## Fases seguintes (proposta)

| Fase | Tema | Itens |
|---|---|---|
| 2 | Base do sistema | Em andamento (ver acima) |
| 3 | Segurança | Firewall (`ufw`), atualizações automáticas de segurança, criptografia, Secure Boot |
| 4 | Backup | Snapshots do sistema (Timeshift) e backup dos dados |
| 5 | Notebook | Bateria e temperatura, GPU AMD, leitor de digital, teclas Fn |
| 6 | Aplicativos | Navegador, Flatpak e Flathub, comunicação, VPN, ferramentas de trabalho |
| 7 | GNOME | Extensões, atalhos, gestos do touchpad, tema e fontes |
| 8 | Automação | Transformar o que foi validado em `scripts/` |

### Fase 2: disco SATA extra

Etapas, cada uma com confirmação antes de executar:

1. [ ] Montar a partição em **somente leitura** e listar o conteúdo.
2. [ ] Decidir: manter o btrfs ou recriar o sistema de arquivos (**apaga tudo**).
3. [ ] Escolher o ponto de montagem e criar a entrada no `/etc/fstab`, identificando o disco
   por **UUID** (o nome `sda` pode mudar, por exemplo ao conectar um pendrive).
4. [ ] Criar a estrutura de pastas com as permissões do usuário.
5. [ ] Apontar para ele o que ocupa espaço, como a biblioteca de jogos.
6. [ ] Incluí-lo no backup (fase 4). Um disco só não é backup.

Pontos de atenção:

- **btrfs ou ext4:** o btrfs oferece *snapshots* e compressão transparente (`compress=zstd`).
  O ext4 é mais simples. Como o disco já vem em btrfs, manter é uma boa opção.
- **Programas:** o APT instala em `/usr` e não permite escolher outro disco. Neste disco vão jogos,
  arquivos, Flatpaks e ferramentas portáteis. Os pacotes do sistema ficam no NVMe.
- **`nofail` no `fstab`:** se o disco falhar ou for removido, o sistema ainda deve iniciar.

---

## Decisões em aberto

1. Há política de TI que exija criptografia de disco, antivírus, VPN ou software específico?
2. Quais ferramentas de trabalho são necessárias (banco de dados, modelagem, Office, videoconferência, acesso remoto)?
   Alguma só existe para Windows?
3. Será preciso rodar Windows em máquina virtual para algum sistema legado?
4. Dados pessoais e de trabalho ficarão no mesmo perfil de usuário?

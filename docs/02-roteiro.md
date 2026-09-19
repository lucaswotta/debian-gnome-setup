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

## Fase 1 - Git e GitHub (próxima)

Objetivo: atualizar o sistema e versionar este guia.

- [ ] Atualizar o sistema: `sudo apt update && sudo apt full-upgrade`
- [ ] Instalar `git` e `gh` (CLI do GitHub)
- [ ] Configurar a identidade do git (`user.name` e `user.email`)
- [ ] Autenticar no GitHub com `gh auth login` (chave SSH)
- [ ] Ligar a pasta local ao repositório remoto e fazer o primeiro commit

A pasta local já tem conteúdo, e o `git clone` exige pasta vazia. Por isso o caminho é
`git init` + `git remote add origin`, em vez de clonar.

---

## Fases seguintes (proposta)

| Fase | Tema | Itens |
|---|---|---|
| 2 | Base do sistema | `contrib` e `non-free` no APT, TRIM do SSD, firmware (`fwupd`), disco extra |
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

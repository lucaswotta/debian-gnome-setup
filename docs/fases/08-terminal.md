# Fase 8 - Terminal e shell

**Status:** concluída · **Escopo:** perfil de uso. Shell, fonte, cores e prompt são preferências

[← Fase 7](07-backup.md) · [Roteiro](../02-roteiro.md)

Objetivo: trocar o terminal padrão por um mais confortável, com sugestões enquanto se digita, prompt informativo, fonte com ícones e paleta própria, sem trocar o shell do sistema.

- [x] **Shell:** zsh, aberto pelo perfil do `gnome-terminal`. O bash continua como shell de login e do sistema, e o `~/.bashrc` não muda.
- [x] **Plugins do zsh:** sugestão de comando enquanto se digita e realce de sintaxe, pelos pacotes do Debian.
- [x] **Prompt:** Starship, em duas linhas. Em cima, a pasta, a branch e o estado do git, as versões de Node, Go, Python e Java (só em pastas de projeto), o tempo do último comando e a hora. Embaixo, o `$`.
- [x] **Fonte:** FiraCode Nerd Font Mono, tamanho 12.
- [x] **Cores:** tema *Terminal Green 1999*, do catálogo Gogh, com texto verde-menta e cursor em sublinhado.
- [x] **Ferramentas:** `eza` (listagem), `bat` (leitura de arquivos), `fzf` (busca) e `fastfetch` (resumo do sistema).
- [x] **Atalhos de linha de comando:** *aliases* de git, listagem, navegação e Docker.
- [x] **Histórico:** 50 mil comandos, compartilhado entre as abas.

Os arquivos de configuração ficam em [`dotfiles/`](../../dotfiles/): [`zshrc`](../../dotfiles/zshrc) e [`starship.toml`](../../dotfiles/starship.toml).

```bash
# 1. Pacotes do Debian
sudo apt-get install -y zsh zsh-autosuggestions zsh-syntax-highlighting starship eza bat fzf fastfetch

# 2. Fonte com ícones (Nerd Font): baixar, conferir o checksum e instalar na pasta do usuário
cd "$(mktemp -d)"
B=https://github.com/ryanoasis/nerd-fonts/releases/latest/download
curl -fLO $B/FiraCode.tar.xz && curl -fLO $B/SHA-256.txt
grep ' FiraCode.tar.xz$' SHA-256.txt | sha256sum -c -
mkdir -p ~/.local/share/fonts/FiraCodeNerdFont
tar -xJf FiraCode.tar.xz -C ~/.local/share/fonts/FiraCodeNerdFont --wildcards '*.ttf' LICENSE
fc-cache -f ~/.local/share/fonts/FiraCodeNerdFont

# 3. Configuração do zsh e do Starship (a partir da raiz deste repositório)
cp dotfiles/zshrc ~/.zshrc
mkdir -p ~/.config && cp dotfiles/starship.toml ~/.config/starship.toml

# 4. Perfil do gnome-terminal: fonte, cores, cursor, sino e abertura em zsh
P=$(gsettings get org.gnome.Terminal.ProfilesList default | tr -d "'")
K="org.gnome.Terminal.Legacy.Profile:/org/gnome/terminal/legacy/profiles:/:$P/"
gsettings set "$K" visible-name 'Terminal Green 1999'
gsettings set "$K" use-system-font false
gsettings set "$K" font 'FiraCode Nerd Font Mono 12'
gsettings set "$K" use-theme-colors false
gsettings set "$K" background-color '#03120A'
gsettings set "$K" foreground-color '#B3FFCF'
gsettings set "$K" palette "['#32463A','#FF5C7A','#22C55E','#00D26A','#7DD3FC','#9DFF57','#39FF88','#8FD6A9','#274335','#FF7D95','#4ED17E','#2EDA85','#97DCFD','#B1FF79','#61FFA0','#D8FFE6']"
gsettings set "$K" bold-color-same-as-fg true
gsettings set "$K" cursor-shape 'underline'
gsettings set "$K" cursor-colors-set true
gsettings set "$K" cursor-background-color '#B3FFCF'
gsettings set "$K" cursor-foreground-color '#03120A'
gsettings set "$K" highlight-colors-set true
gsettings set "$K" highlight-background-color '#14532D'
gsettings set "$K" highlight-foreground-color '#D8FFE6'
gsettings set "$K" audible-bell false
gsettings set "$K" use-custom-command true
gsettings set "$K" custom-command '/usr/bin/zsh'
```

O que cada parte faz:

| Parte | Função |
|---|---|
| `zsh-autosuggestions` | Mostra, em cor apagada, o comando que completa o que foi digitado. A seta para a direita aceita. |
| `zsh-syntax-highlighting` | Colore o comando enquanto se digita: verde se existe, vermelho se não existe. |
| `starship` | Desenha o prompt. A configuração está em `~/.config/starship.toml`. |
| `fzf` | `Ctrl+R` busca no histórico, `Ctrl+T` acha arquivos e `Alt+C` entra em uma pasta. |
| `eza` | Substitui o `ls`, com ícones, cores e coluna do git (`ll`, `la` e `lt` para árvore). |
| `bat` | Mostra arquivos com cores e numeração. No Debian, o programa se chama `batcat`. |
| `fastfetch` | Resumo do sistema, sob demanda. |
| `custom-command` | Faz o perfil abrir o zsh sem trocar o shell de login. |

Como conferir:

```bash
zsh -n ~/.zshrc && echo 'sintaxe ok'
TERM=xterm-256color script -qec 'zsh -ic "echo carregou"' /dev/null   # carrega o zsh num terminal de verdade
fc-list | grep -c 'FiraCode Nerd Font Mono'                             # maior que zero
gsettings get "$K" font
```

Abra um terminal novo. Ele deve abrir em zsh, com a fonte nova, e o prompt deve mostrar a pasta e, dentro de um repositório git, a branch com ícone.

Como desfazer:

- **Perfil do terminal:** `dconf reset -f /org/gnome/terminal/legacy/profiles:/` volta ao padrão de fábrica.
- **Configuração:** `rm ~/.zshrc ~/.config/starship.toml`. O histórico fica em `~/.zsh_history`.
- **Fonte:** `rm -r ~/.local/share/fonts/FiraCodeNerdFont && fc-cache -f`.
- **Pacotes:** `sudo apt remove --autoremove zsh zsh-autosuggestions zsh-syntax-highlighting starship eza bat fzf fastfetch`.

Observações:

- **Sem `chsh`:** o shell de login continua sendo o bash. Scripts, o terminal integrado do VS Code e sessões remotas não mudam. Para abrir o bash no `gnome-terminal`, basta `gsettings set "$K" use-custom-command false`.
- **Nome do `bat`:** no Debian, o executável é `batcat`, porque o nome `bat` pertence a outro pacote. O `zshrc` cria o *alias* `bat`.
- **Alias `gs`:** o atalho para `git status` esconde o Ghostscript só no terminal interativo. Use `command gs` para chamá-lo.
- **Fonte:** o pacote traz três famílias. No terminal, use a **Mono**, em que cada ícone ocupa a largura de um caractere. A `Propo` não é monoespaçada.
- **Cores seguem o terminal:** o prompt, o `fzf`, o realce de sintaxe, o `man` e o `bat` usam as 16 cores da paleta do perfil, e não valores próprios. Trocar de tema é trocar `background-color`, `foreground-color`, `cursor-*` e `palette` no perfil. Só a sugestão do zsh e os comentários usam uma cor fixa (`#4A8F68`), porque nenhuma cor da paleta serve de tom médio apagado.
- **Tema:** o catálogo do Gogh (`github.com/Gogh-Co/Gogh`, pasta `themes`) traz um arquivo por tema, com `color_01` a `color_16` (a ordem da `palette`), `background`, `foreground` e `cursor`. Aplique esses valores com `gsettings`, como no passo 4, em vez de usar o instalador do projeto, que baixa e executa um script.
- **Contraste no *Terminal Green 1999*:** amarelo, magenta e ciano do tema são tons de verde. A cor 8 (`#274335`) tem contraste de 1,8:1 sobre o fundo e não serve para texto, por isso a hora e o tempo do comando usam a cor 7. Em outro tema, confira o contraste da cor 8 antes de usá-la.
- **Transparência:** o `gnome-terminal` 3.56 não tem opção de fundo transparente. Para isso, é preciso outro terminal, como o Ptyxis ou o Kitty.
- **Dependências do `zshrc`:** as linhas do `fnm`, do SDKMAN, do Go e do `uv` pressupõem as ferramentas da [fase 5](05-aplicativos.md). As do `fnm` e do SDKMAN só rodam se a ferramenta existir. As demais entradas do `PATH` são inofensivas sem elas.
- **Histórico:** `~/.zsh_history` guarda os comandos digitados, com o que houver neles. Não o versione nem o compartilhe.
- **Privacidade:** o `fastfetch` mostra o nome da máquina e o do usuário. Corte essas linhas antes de publicar uma captura de tela.

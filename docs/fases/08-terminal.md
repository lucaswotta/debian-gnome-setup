# Fase 8 - Terminal e shell

**Status:** concluída · **Escopo:** perfil de uso. Shell, fonte, cores e prompt são preferências

[← Fase 7](07-backup.md) · [Roteiro](../02-roteiro.md)

Objetivo: trocar o terminal padrão por um mais confortável, com sugestões enquanto se digita, prompt informativo, fonte com ícones e paleta própria, sem trocar o shell do sistema.

- [x] **Shell:** zsh, aberto pelo perfil do `gnome-terminal`. O bash continua como shell de login e do sistema, e o `~/.bashrc` não muda.
- [x] **Plugins do zsh:** sugestão de comando enquanto se digita e realce de sintaxe, pelos pacotes do Debian.
- [x] **Prompt:** Starship, em duas linhas. Em cima, a pasta e o git em blocos Powerline (seta cheia, degradê de verde, casa no lugar do `~` e ícones nas pastas do sistema), as versões de Node, Go, Python e Java (só em pastas de projeto), o tempo do último comando e a hora, em cor discreta. Embaixo, o `$`.
- [x] **Fonte:** FiraCode Nerd Font Mono, tamanho 12.
- [x] **Cores:** tema *Terminal Green 1999*, do catálogo Gogh, com texto verde-menta e cursor em sublinhado.
- [x] **Destaque verde no GTK 3:** o `gnome-terminal` usa GTK 3, que ignora a cor de destaque do GNOME. Um arquivo de estilo do usuário troca o azul das abas e da seleção pelo verde de destaque.
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

# 4. Destaque verde nos aplicativos GTK 3 (a linha das abas e a seleção de texto)
mkdir -p ~/.config/gtk-3.0
cat > ~/.config/gtk-3.0/gtk.css <<'EOF'
notebook > header.top > tabs > tab:checked { box-shadow: inset 0 -4px #3a944a; }
label selection { background-color: #3a944a; }
EOF

# 5. Perfil do gnome-terminal: fonte, cores, cursor, sino e abertura em zsh
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
| `starship` | Desenha o prompt, com a pasta e o git em blocos Powerline. A configuração está em `~/.config/starship.toml`. |
| `fzf` | `Ctrl+R` busca no histórico, `Ctrl+T` acha arquivos e `Alt+C` entra em uma pasta. |
| `eza` | Substitui o `ls`, com ícones, cores e coluna do git (`ll`, `la` e `lt` para árvore). |
| `bat` | Mostra arquivos com cores e numeração. No Debian, o programa se chama `batcat`. |
| `fastfetch` | Resumo do sistema, sob demanda. |
| `gtk.css` | Troca o azul do GTK 3 pelo verde de destaque nos aplicativos GTK 3, como as abas do `gnome-terminal`. |
| `custom-command` | Faz o perfil abrir o zsh sem trocar o shell de login. |

Como conferir:

```bash
zsh -n ~/.zshrc && echo 'sintaxe ok'
TERM=xterm-256color script -qec 'zsh -ic "echo carregou"' /dev/null   # carrega o zsh num terminal de verdade
fc-list | grep -c 'FiraCode Nerd Font Mono'                             # maior que zero
gsettings get "$K" font
```

Feche todas as janelas do terminal e abra uma nova. Ela deve abrir em zsh, com a fonte nova. O prompt deve mostrar a pasta em um bloco verde e, dentro de um repositório git, o bloco da branch. A linha sob a aba selecionada deve ser verde.

Como desfazer:

- **Destaque do GTK 3:** `rm ~/.config/gtk-3.0/gtk.css` e abrir os aplicativos de novo.
- **Perfil do terminal:** `dconf reset -f /org/gnome/terminal/legacy/profiles:/` volta ao padrão de fábrica.
- **Configuração:** `rm ~/.zshrc ~/.config/starship.toml`. O histórico fica em `~/.zsh_history`.
- **Fonte:** `rm -r ~/.local/share/fonts/FiraCodeNerdFont && fc-cache -f`.
- **Pacotes:** `sudo apt remove --autoremove zsh zsh-autosuggestions zsh-syntax-highlighting starship eza bat fzf fastfetch`.

Observações:

- **Sem `chsh`:** o shell de login continua sendo o bash. Scripts, o terminal integrado do VS Code e sessões remotas não mudam. Para abrir o bash no `gnome-terminal`, basta `gsettings set "$K" use-custom-command false`.
- **Nome do `bat`:** no Debian, o executável é `batcat`, porque o nome `bat` pertence a outro pacote. O `zshrc` cria o *alias* `bat`.
- **Alias `gs`:** o atalho para `git status` esconde o Ghostscript só no terminal interativo. Use `command gs` para chamá-lo.
- **Fonte:** o pacote traz três famílias. No terminal, use a **Mono**, em que cada ícone ocupa a largura de um caractere. A `Propo` não é monoespaçada.
- **Cores seguem o terminal:** o prompt, o `fzf`, o realce de sintaxe, o `man` e o `bat` usam as 16 cores da paleta do perfil, e não valores próprios. Trocar de tema é trocar `background-color`, `foreground-color`, `cursor-*` e `palette` no perfil. Têm cor fixa a sugestão do zsh e os comentários (`#4A8F68`, porque nenhuma cor da paleta serve de tom médio apagado) e os blocos do Powerline (quatro valores na seção `[palettes.powerline]` do `starship.toml`). Ao trocar de tema, revise esses valores.
- **Tema:** o catálogo do Gogh (`github.com/Gogh-Co/Gogh`, pasta `themes`) traz um arquivo por tema, com `color_01` a `color_16` (a ordem da `palette`), `background`, `foreground` e `cursor`. Aplique esses valores com `gsettings`, como no passo 5, em vez de usar o instalador do projeto, que baixa e executa um script.
- **Contraste no *Terminal Green 1999*:** amarelo, magenta e ciano do tema são tons de verde. A cor 8 (`#274335`) tem contraste de 1,8:1 sobre o fundo e não serve para texto, por isso o tempo do comando usa a cor 7. A hora usa a cor 8 de propósito, para ficar discreta. Em outro tema, confira o contraste da cor 8 antes de usá-la.
- **Powerline:** as setas (`U+E0B0` e `U+E0B1`) e os ícones de pasta existem na FiraCode Nerd Font Mono. Sem uma Nerd Font, aparecem como quadrados. A seta que fecha o bloco depende de a pasta estar ou não em um repositório git. Isso é decidido pelos módulos `custom.dir_end` e `custom.git_end`, que rodam `git rev-parse` a cada prompt, com custo desprezível. Os nomes das pastas com ícone estão em português (Documentos, Downloads, Imagens, Música, Vídeos e Área de trabalho). Para outro idioma, troque as chaves de `[directory.substitutions]`. Fora da pasta pessoal, o caminho começa com uma seta fina, que marca a raiz.
- **Destaque no GTK 3:** o destaque escolhido em *Configurações > Aparência* só vale para aplicativos GTK 4 com libadwaita. O `gnome-terminal` do Debian 13 é GTK 3, e o tema Adwaita dele é azul fixo, com a cor escrita dentro da regra, e não em uma variável. O `gtk.css` do passo 4 traz as duas regras principais. Uma versão completa repete todas as propriedades azuis do tema (foco, botões de ação, entradas de texto), com o azul trocado por tons de verde. Para gerá-la, extraia o CSS escuro embutido na biblioteca com `gresource extract /usr/lib/x86_64-linux-gnu/libgtk-3.so.0 /org/gtk/libgtk/theme/Adwaita/gtk-contained-dark.css` e troque cada azul, de matiz entre 205° e 225°, pelo verde de destaque do GNOME 48 (`#3a944a`, obtido da libadwaita), mantendo só as propriedades que mudam. O arquivo não fica no repositório, porque deriva do tema do GTK, que tem licença própria. Ele não acompanha o destaque: se a cor mudar, o arquivo precisa ser refeito. O `gnome-terminal` guarda o estilo em memória, então é preciso fechar todas as janelas para aplicar. Versões do `gnome-terminal` em GTK 4 com libadwaita seguem o destaque e dispensam o arquivo.
- **Transparência:** o `gnome-terminal` 3.56 não tem opção de fundo transparente. Para isso, é preciso outro terminal, como o Ptyxis ou o Kitty.
- **Dependências do `zshrc`:** as linhas do `fnm`, do SDKMAN, do Go e do `uv` pressupõem as ferramentas da [fase 5](05-aplicativos.md). As do `fnm` e do SDKMAN só rodam se a ferramenta existir. As demais entradas do `PATH` são inofensivas sem elas.
- **Histórico:** `~/.zsh_history` guarda os comandos digitados, com o que houver neles. Não o versione nem o compartilhe.
- **Privacidade:** o `fastfetch` mostra o nome da máquina e o do usuário. Corte essas linhas antes de publicar uma captura de tela.

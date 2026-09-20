# VPN e acesso a servidores

**Status:** concluída · **Escopo:** depende do ambiente. Cobre uma VPN Fortinet e servidores Linux acessados por SSH

[← Fase 5](05-aplicativos.md) · [Roteiro](../02-roteiro.md) · [Fase 6 →](06-gnome.md)

Duas partes: a VPN corporativa, ligada pelo menu do GNOME, e o acesso aos arquivos dos servidores internos, que só respondem com a VPN conectada.

## VPN Fortinet

Objetivo: ligar e desligar a VPN pelo menu rápido do GNOME, sem usar o terminal.

- [x] Instalar o plugin do NetworkManager para OpenConnect
- [x] Cadastrar a conexão pela tela de Rede, com o protocolo Fortinet
- [x] Conectar e validar

```bash
sudo apt install -y network-manager-openconnect-gnome
```

Cadastro, pela interface:

1. *Configurações > Rede*, botão **+** ao lado de *VPN*, *Multiprotocol VPN Client (openconnect)*.
2. **Protocolo:** Fortinet SSL VPN.
3. **Gateway:** `ENDERECO:PORTA`.
4. Salvar e conectar. Usuário e senha são pedidos na conexão.

Como conferir:

```bash
nmcli connection show --active     # a VPN aparece com o tipo "vpn"
ip route | grep vpn0               # rotas das redes internas pelo túnel
ip route show default              # a internet segue pela rota normal
```

Observações:

- **Plugin:** o Debian 13 não tem plugin do NetworkManager para o `openfortivpn`. O OpenConnect fala o
  protocolo Fortinet e resolve. O `openfortivpn` continua instalado como reserva.
- **Onde ficam os dados:** endereço e credenciais ficam no NetworkManager, fora do repositório.
- **Túnel dividido:** só as redes internas passam pela VPN. A internet segue pela rota normal.
- **DNS:** sem `systemd-resolved`, o NetworkManager grava os DNS da VPN em `/etc/resolv.conf`.
  Se um nome interno não resolver, comece por esse arquivo.
- **Certificado:** aceite o certificado do servidor só se reconhecer o servidor.

## Acesso a servidores por SFTP

Objetivo: abrir e editar arquivos de servidores Linux pelo aplicativo Arquivos, no lugar do WinSCP.

O aplicativo **Arquivos** abre servidores por SFTP, sem programa extra. As chaves do PuTTY e do WinSCP (`.ppk`) precisam ser convertidas para o formato do OpenSSH,
e o `puttygen`, do pacote `putty-tools`, faz isso.

```bash
sudo apt-get install -y putty-tools
mv ~/Downloads/<chave>.ppk ~/.ssh/chave-servidores.ppk && chmod 600 ~/.ssh/chave-servidores.ppk
puttygen ~/.ssh/chave-servidores.ppk -O private-openssh-new -o ~/.ssh/chave-servidores -P     # pede a senha atual e a nova
puttygen ~/.ssh/chave-servidores.ppk -O public-openssh -o ~/.ssh/chave-servidores.pub         # parte pública, sem senha
chmod 600 ~/.ssh/chave-servidores
```

Regra padrão para os servidores da rede interna, em `~/.ssh/config` (permissão `600`):

```
Host 10.* 172.* 192.168.*
    IdentityFile ~/.ssh/chave-servidores
    IdentitiesOnly yes
    AddKeysToAgent yes
    ServerAliveInterval 30
```

Uso:

- **Arquivos:** `Ctrl+L`, digite `sftp://<usuario>@<ip-do-servidor>/` e confirme. Com `Ctrl+D`, o servidor vai para a barra lateral e abre com um clique.
  O aplicativo também guarda os servidores recentes em *Outros locais > Conectar ao servidor*.
- **Favoritos:** o Chrome usa `google-chrome.desktop`. A entrada `com.google.Chrome.desktop` é oculta (`NoDisplay=true`) e aponta para o mesmo programa.
- **Terminal:** `ssh <usuario>@<ip-do-servidor>`.

Como conferir:

```bash
ssh -G <ip-do-servidor> | grep -E '^(identityfile|identitiesonly) '     # a chave de trabalho, e só ela
ssh -G github.com | grep '^identityfile '                                # não inclui a chave de trabalho
ssh-keygen -y -P '' -f ~/.ssh/chave-servidores > /dev/null; echo $?        # diferente de 0: a chave está protegida por senha
```

Como desfazer: apague o bloco `Host` do `~/.ssh/config`, os arquivos `chave-servidores*` e, se quiser, `sudo apt remove putty-tools`.

Observações:

- **Um usuário por servidor:** o usuário vai no endereço, então cada servidor pode ter o seu (`opc`, `ubuntu` e outros), com a mesma chave.
- **Alcance da regra:** só as faixas internas (`10.*`, `172.*` e `192.168.*`) usam a chave de trabalho. O GitHub segue com a chave própria, e a chave de trabalho não é oferecida a servidores externos.
  Para servidores acessados por nome, acrescente o padrão à linha `Host` (por exemplo, `*.empresa.interno`).
- **Senha da chave:** o GNOME pergunta a senha na primeira conexão e oferece guardá-la no chaveiro. Com `AddKeysToAgent`, a chave fica no agente até o fim da sessão.
- **Primeira conexão a cada servidor:** o programa mostra a impressão digital do servidor. Confirme com quem o administra e aceite. Ela fica em `~/.ssh/known_hosts`.
- **VPN:** os servidores internos só respondem com a VPN conectada.
- **Chaves fora do repositório:** nunca versione `~/.ssh`. Mantenha o `.ppk` original como cópia.
- **Editar direto no servidor:** um arquivo aberto pelo Arquivos, no Editor de Texto, é salvo no próprio servidor. Para projetos, a extensão *Remote - SSH* do VS Code usa a mesma configuração *(proposta)*.

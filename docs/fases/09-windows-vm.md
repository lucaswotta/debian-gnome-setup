# Fase 9 - Windows em máquina virtual

**Status:** concluída · **Escopo:** perfil de uso. Serve a quem precisa de um Windows, como para um programa legado

[← Fase 8](08-terminal.md) · [Roteiro](../02-roteiro.md)

Objetivo: rodar o Windows 10 LTSC em uma máquina virtual (VM) com aceleração por hardware. O disco da VM é um arquivo, e os discos e o boot do notebook não mudam.

- [x] **Virtualização:** QEMU/KVM, libvirt e virt-manager, pelos pacotes do Debian e sem as recomendações.
- [x] **Serviços:** o `libvirtd` e o `virtlogd` iniciam sob demanda, pelos sockets do `systemd`. O `libvirt-guests` fica habilitado.
- [x] **Rede:** a rede `default` do libvirt na faixa `100.66.0.0/24`, fora das faixas da VPN e do Docker.
- [x] **Armazenamento:** a pasta `/mnt/ssd/VMs`, no SSD extra, sem cópia na escrita (*nodatacow*), com o pool `vms` e um disco `qcow2` de 60 GB.
- [x] **Windows:** Windows 10 Enterprise LTSC 2019, em português, a partir do ISO oficial da Microsoft.
- [x] **Máquina:** UEFI, 4 GB de memória, 4 vCPUs, disco e placa de rede `virtio`, vídeo QXL e SPICE.
- [x] **Integração:** drivers `virtio-win`, agente SPICE (copiar e colar e resolução da janela) e agente QEMU.
- [x] **Navegador:** Microsoft Edge, instalado pelo PowerShell.

O LTSC (*Long-Term Servicing Channel*) não recebe atualizações de recursos, não traz a Microsoft Store e vem com poucos aplicativos embutidos. O LTSC 2019 tem suporte mais longo que o LTSC 2021 sem IoT, por isso é o escolhido. O Edge baseado em Chromium não vem incluído.

## 1. Pacotes e grupo (com `sudo`)

```bash
sudo apt-get install -y --no-install-recommends \
  qemu-system-x86 qemu-utils qemu-system-modules-spice \
  libvirt-daemon-system libvirt-clients virt-manager ovmf dnsmasq-base \
  gir1.2-spiceclientgtk-3.0 gir1.2-gtk-vnc-2.0
sudo adduser "$USER" libvirt
```

| Parte | O que faz |
|---|---|
| `--no-install-recommends` | Deixa de fora o suporte a armazenamento em rede (GlusterFS, Ceph e iSCSI), que a VM não usa. |
| `qemu-system-x86` e `qemu-utils` | O emulador, que usa o KVM do processador, e as ferramentas de imagem de disco. |
| `qemu-system-modules-spice` | Traz o vídeo QXL e o SPICE. Sem ele, o libvirt rejeita a VM com vídeo QXL. |
| `libvirt-daemon-system` e `libvirt-clients` | O serviço que gerencia as VMs e o `virsh`. |
| `virt-manager` e os `gir1.2-*` | A interface gráfica e os visualizadores de tela (SPICE e VNC). |
| `ovmf` | O firmware UEFI da VM. |
| `dnsmasq-base` | O DHCP e o DNS da rede virtual. |
| `adduser ... libvirt` | Permite gerenciar as VMs sem `sudo`. Vale a partir do próximo login. Até lá, use `sg libvirt -c '<comando>'`. |

### Serviços sob demanda (com `sudo`)

```bash
sudo systemctl disable libvirtd.service virtlogd.service
sudo systemctl enable libvirtd.socket libvirtd-ro.socket libvirtd-admin.socket virtlogd.socket virtlogd-admin.socket
sudo systemctl stop libvirtd.service virtlogd.service
```

| Parte | O que faz |
|---|---|
| `disable` dos dois serviços | Tira o `libvirtd` e o `virtlogd` do boot. |
| `enable` dos sockets | O `disable` também desliga os sockets, porque cada serviço os lista em `Also=`. Religá-los mantém a ativação sob demanda: o primeiro `virsh` ou o virt-manager inicia o `libvirtd`, e o `virtlogd` sobe quando uma VM começa. |
| `stop` | Encerra as instâncias que estão rodando, sem esperar o próximo boot. |
| `libvirt-guests` | Continua habilitado. Não tem processo residente e, no desligamento do notebook, suspende a VM que estiver rodando, em vez de encerrá-la à força. |

## 2. Rede da VM

A rede `default` do libvirt usa `192.168.122.0/24`, que fica dentro da faixa que a VPN roteia. A faixa `100.66.0.0/24` é do mesmo bloco `100.64.0.0/10` que o Docker usa na [fase 5](05-aplicativos.md).

```bash
export LIBVIRT_DEFAULT_URI=qemu:///system      # sem isto, o virsh usa a sessão do usuário
virsh net-dumpxml default | sed 's/192\.168\.122\./100.66.0./g' > /tmp/rede.xml
virsh net-define /tmp/rede.xml                 # a rede vem desligada depois da instalação
virsh net-start default
```

A rede `default` não inicia com o sistema, para manter o `libvirtd` sob demanda. Sem ela, o virt-manager recusa iniciar a VM com o erro `network 'default' is not active`.

### Ligar a rede ao abrir o virt-manager

Um script liga a rede quando ela está desligada e abre o virt-manager. Um atalho local, que tem prioridade sobre o do pacote, chama o script.

```bash
mkdir -p ~/.local/bin ~/.local/share/applications
cat > ~/.local/bin/virt-manager-rede <<'EOF'
#!/bin/sh
# Liga a rede 'default' do libvirt (se estiver desligada) e abre o virt-manager.
uri=qemu:///system
if ! virsh -c "$uri" net-info default 2>/dev/null | grep -q '^Active:.*yes'; then
    virsh -c "$uri" net-start default >/dev/null 2>&1
fi
exec virt-manager "$@"
EOF
chmod +x ~/.local/bin/virt-manager-rede
sed "s|^Exec=virt-manager$|Exec=$HOME/.local/bin/virt-manager-rede|" \
  /usr/share/applications/virt-manager.desktop > ~/.local/share/applications/virt-manager.desktop
desktop-file-validate ~/.local/share/applications/virt-manager.desktop && echo válido
```

| Parte | O que faz |
|---|---|
| `net-info ... grep` | Consulta o estado da rede. O `net-start` só roda se ela estiver inativa, porque ele falha em uma rede já ativa. |
| `exec virt-manager "$@"` | Substitui o script pelo virt-manager e repassa os argumentos. |
| `sed` | Copia o atalho do pacote e troca só a linha `Exec`, o que mantém nome, ícone e traduções. Uma atualização do pacote não altera a cópia. |

Para conferir, com `LIBVIRT_DEFAULT_URI=qemu:///system` definido: `virsh net-destroy default` (com a VM desligada), abra o virt-manager pelo menu e rode `virsh net-list`. A rede aparece como `active`.

O atalho cobre só a abertura pelo menu. Quem inicia a VM por outro caminho, como `virsh start`, liga a rede antes com `virsh net-start default`.

Para desfazer: `rm ~/.local/bin/virt-manager-rede ~/.local/share/applications/virt-manager.desktop`. Alternativa ao script: `virsh net-autostart default` *(proposta)*, que liga a rede quando o `libvirtd` inicia.

## 3. Armazenamento

```bash
mkdir /mnt/ssd/VMs && chattr +C /mnt/ssd/VMs   # +C vale para os arquivos criados depois
mkdir /mnt/ssd/VMs/iso
virsh pool-define-as vms dir --target /mnt/ssd/VMs && virsh pool-start vms
virsh vol-create-as vms w10ltsc.qcow2 60G --format qcow2
```

| Parte | O que faz |
|---|---|
| `chattr +C` | Desliga a cópia na escrita do `btrfs` na pasta. Imagem de VM com cópia na escrita fragmenta e fica lenta. A pasta perde a compressão e a soma de verificação. |
| `pool-define-as` | Registra a pasta no libvirt como um pool de discos. |
| `60G` e `qcow2` | O disco é fino: o arquivo cresce com o uso, até 60 GB. |

Os ISOs e os discos ficam fora da pasta pessoal, porque ela tem modo `700` e o usuário `libvirt-qemu`, que executa a VM, não a atravessa.

## 4. Os dois ISOs

- **Windows:** baixe em [my.visualstudio.com](https://my.visualstudio.com/) (assinatura do Visual Studio) o arquivo *Windows 10 Enterprise LTSC 2019 (x64) - DVD (Portuguese-Brazil)* e salve em `/mnt/ssd/VMs/iso`. O SHA-256 publicado é `438FC394A69A70C2DD2B575EE06243C95829ED73819A8114D9B726DDA005ADDE`. A versão em inglês tem o SHA-256 `B570DDFDC4672F4629A95316563DF923BD834AEC657DE5D4CA7C7EF9B58DF2B1`.
- **Drivers `virtio-win`:** a Fedora publica o ISO estável, com os drivers assinados para o Windows.

```bash
sha256sum /mnt/ssd/VMs/iso/*.iso               # compare com o hash publicado
curl -fLo /mnt/ssd/VMs/iso/virtio-win.iso \
  https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso
```

## 5. Criar a VM

```bash
virt-install --name w10ltsc --memory 4096 --vcpus 4 --cpu host-passthrough \
  --osinfo win10 --boot uefi \
  --disk vol=vms/w10ltsc.qcow2,bus=virtio,cache=none,discard=unmap \
  --cdrom /mnt/ssd/VMs/iso/<iso-do-windows>.iso \
  --disk path=/mnt/ssd/VMs/iso/virtio-win.iso,device=cdrom \
  --network network=default,model=virtio \
  --graphics spice --video model.type=qxl \
  --channel spicevmc \
  --channel unix,target.type=virtio,target.name=org.qemu.guest_agent.0 \
  --print-xml 1 > /tmp/w10ltsc.xml
sed -i 's#<on_reboot>destroy</on_reboot>#<on_reboot>restart</on_reboot>#' /tmp/w10ltsc.xml
virsh define /tmp/w10ltsc.xml
```

| Parte | O que faz |
|---|---|
| `--print-xml 1` | Só gera a definição da VM, sem ligá-la. O `1` escolhe o XML da fase de instalação. |
| `sed` de `on_reboot` | O XML de instalação vem com `on_reboot=destroy`, que desligaria a VM no primeiro reinício do Windows Setup. O `sed` troca por `restart`. |
| `--cpu host-passthrough` | Repassa o processador real, para o desempenho ficar próximo do nativo. |
| `--osinfo win10` | Aplica os ajustes recomendados para o Windows 10, como os relógios e os recursos do Hyper-V. |
| `--boot uefi` | Usa o firmware OVMF, com Secure Boot e as chaves da Microsoft. O Windows 10 não exige TPM. |
| `bus=virtio` e `model=virtio` | Disco e rede paravirtualizados, bem mais rápidos que os emulados. O Windows precisa dos drivers (passo 6). |
| `--video model.type=qxl` | Vídeo que aceita mudar a resolução com a janela. |
| `--channel spicevmc` | Canal do agente SPICE, para copiar e colar e ajustar a resolução. Sem ele, nada disso funciona, mesmo com o agente instalado no Windows. |
| `--channel unix,...guest_agent.0` | Canal do agente QEMU, que permite ao host consultar e comandar o Windows (`guest-ping`, `guest-exec`). |

A memória e as vCPUs são valores de partida. O virt-manager os altera com a VM desligada.

## 6. Instalar o Windows

Abra o `virt-manager`, dê duplo clique em `w10ltsc` e ligue a VM. O instalador pergunta *Press any key to boot from CD or DVD* por poucos segundos: aperte uma tecla dentro da janela. Se passar do tempo, reinicie a VM.

1. Escolha o idioma e *Instalar agora*. A chave do produto é a da assinatura.
2. Escolha **Personalizada: instalar somente o Windows**.
3. A lista de discos vem **vazia**, porque o Windows não traz o driver do disco `virtio`. Use **Carregar driver > Procurar**, escolha o CD `virtio-win` e a pasta `viostor\w10\amd64`, e selecione o *Red Hat VirtIO SCSI controller*. O disco de 60 GB aparece.
4. Na configuração inicial, sem rede, escolha **Não tenho internet** e crie uma conta local.

## 7. Depois de instalar

1. No Windows, abra o CD `virtio-win` e execute `virtio-win-guest-tools.exe`. Ele instala os drivers de rede, de vídeo e de memória, o agente SPICE e o agente QEMU. Reinicie a VM.
2. Com a VM desligada, remova do XML a linha `<boot dev='cdrom'/>` (`virsh edit w10ltsc`). A VM passa a iniciar direto pelo disco, sem o aviso de teclar.
3. No PowerShell da VM, instale o Edge. O LTSC 2019 não traz o `winget`, então o instalador vem direto da Microsoft:

```powershell
[Net.ServicePointManager]::SecurityProtocol = 'Tls12'
$ProgressPreference = 'SilentlyContinue'
iwr "https://go.microsoft.com/fwlink/?linkid=2109047&Channel=Stable&language=pt-br&brand=M100" -OutFile $env:TEMP\edge.exe
(Get-AuthenticodeSignature $env:TEMP\edge.exe).Status
Start-Process $env:TEMP\edge.exe
```

| Parte | O que faz |
|---|---|
| `SecurityProtocol = 'Tls12'` | O PowerShell 5.1 do LTSC 2019 pode tentar um protocolo antigo, e o download falha. Vale só para a sessão. |
| `$ProgressPreference` | Desliga a barra de progresso do `iwr`, que torna o download muito mais lento no PowerShell 5.1. |
| `Get-AuthenticodeSignature` | Deve mostrar `Valid`: o arquivo tem a assinatura da Microsoft. |

Como conferir:

```bash
virsh list --all                                     # w10ltsc: running ou shut off
systemctl is-enabled libvirtd.service virtlogd.service   # disabled
systemctl is-enabled libvirtd.socket virtlogd.socket     # enabled
virsh net-dhcp-leases default                        # com a VM ligada: um IP 100.66.0.x
virsh dumpxml w10ltsc | grep -E "name='(com.redhat.spice.0|org.qemu.guest_agent.0)'"
                                                     # os dois com state='connected', com o Windows no ar
virsh qemu-agent-command w10ltsc '{"execute":"guest-ping"}'     # {"return":{}}
```

No Windows: `ping 100.66.0.1` responde, e `Test-NetConnection go.microsoft.com -Port 443` mostra `TcpTestSucceeded : True`.

Como desfazer:

```bash
virsh destroy w10ltsc                                # se estiver ligada
virsh undefine w10ltsc --nvram                       # remove a VM e o firmware dela
virsh vol-delete w10ltsc.qcow2 --pool vms            # apaga o disco virtual
virsh pool-destroy vms && virsh pool-undefine vms
virsh net-destroy default
rm -r /mnt/ssd/VMs                                   # também apaga os ISOs
sudo deluser "$USER" libvirt
sudo systemctl enable libvirtd.service virtlogd.service   # só para voltar ao padrão, sem remover os pacotes
sudo apt remove --autoremove qemu-system-x86 qemu-utils qemu-system-modules-spice \
  libvirt-daemon-system libvirt-clients virt-manager ovmf dnsmasq-base
```

Para apagar também a configuração do libvirt, use `apt purge` no lugar de `apt remove`.

Observações:

- **Sem `sudo` no dia a dia:** depois do novo login, o grupo `libvirt` dá acesso ao `qemu:///system`. O virt-manager já usa essa conexão.
- **Docker ativo:** com o Docker rodando, a VM tem rede e internet. As regras de firewall do Docker e as do libvirt convivem.
- **Serviços:** o `libvirtd` e o `virtlogd` só iniciam quando alguma coisa os usa. O `libvirtd` roda com `--timeout 120` e sai sozinho depois de 2 minutos ociosos, desde que não haja VM rodando nem a rede `default` ligada. Depois de usar a VM, `virsh net-destroy default` desliga a rede e deixa o `libvirtd` sair. O `virtlogd` não tem timeout e fica ativo até o próximo boot. O `libvirt-guests` é `oneshot`: com `ON_BOOT=ignore`, o padrão, não conecta ao libvirt no boot.
- **Ativação:** a chave vem da assinatura ou do contrato de volume da organização. O guia não trata disso.
- **Cópia de segurança:** o disco da VM é um único arquivo, `/mnt/ssd/VMs/w10ltsc.qcow2`. Para guardá-lo, copie o arquivo com a VM desligada *(proposta)*.
- **Desempenho gráfico:** o QXL atende o uso de escritório e de sistemas legados. Não há aceleração 3D para jogos.

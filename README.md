# Tutorial Pós-instalação — Workstation Corporativa Linux

## Rocky Linux 10.1 Desktop — Ambiente KDE Plasma

---

## 1. Objetivo

Este documento descreve o procedimento técnico padrão de pós-instalação do **Rocky Linux 10.1 com ambiente gráfico KDE Plasma**, aplicado em estações de trabalho corporativas. O objetivo é garantir que o sistema esteja devidamente atualizado, seguro, integrado ao domínio Active Directory e com os softwares corporativos necessários instalados. Quando houver necessidade de compatibilidade com aplicações restritas ao ecossistema Microsoft, recomenda-se a utilização de uma máquina virtual Windows ou de um servidor Windows Server com Remote Desktop Services habilitado, a fim de manter a produtividade dos usuários finais.

### Escopo

**Inclui:**

* Workstations corporativas Linux
* Usuários finais
* Ambiente gráfico KDE Plasma

**Não inclui:**

* Servidores
* Uso pessoal ou doméstico (mas pode ser adaptado)
* Dual boot
* Hardening avançado de kernel

---

## 2. Pré-requisitos

* Rocky Linux 10.1 instalado com KDE Plasma
* Acesso administrativo (sudo)
* Conectividade com a internet
* Credenciais válidas para ingresso no domínio
* Acesso aos portais/clouds de distribuição do FortiClient e CrowdStrike

---

## 3. Atualização do Sistema Operacional e Ajustes Iniciais

Atualizar todos os pacotes do sistema para garantir correções de segurança e estabilidade, além de aplicar ajustes iniciais:

```bash
sudo hostnamectl set-hostname novo-hostname

sudo systemctl stop cockpit.socket
sudo systemctl stop cockpit.service
sudo systemctl disable cockpit.socket
sudo systemctl disable cockpit.service

sudo dnf install -y https://mirrors.rpmfusion.org/free/el/rpmfusion-free-release-10.noarch.rpm
sudo dnf install -y https://mirrors.rpmfusion.org/nonfree/el/rpmfusion-nonfree-release-10.noarch.rpm

sudo dnf update -y

sudo dnf install -y vlc vlc-plugins-all gnome-keyring libsecret libvirt libvirt-daemon-kvm qemu-kvm virt-install virt-manager virt-viewer

sudo dnf install -y ffmpeg ffmpeg-libs --allowerasing
```

> Recomenda-se reiniciar o sistema após a atualização, especialmente se o kernel ou bibliotecas críticas forem atualizados.

---

## 4. Habilitação de Atualizações Automáticas

### 4.1 Instalação do serviço

```bash
sudo dnf install -y dnf-automatic
```

### 4.2 Configuração

Editar o arquivo de configuração:

```bash
sudo nano /etc/dnf/automatic.conf
```

Ajustar o parâmetro:

```ini
apply_updates = yes
```

### 4.3 Ativação do timer

```bash
sudo systemctl enable --now dnf-automatic.timer
```

---

## 5. Instalação do FortiClient VPN (opcional)

1. Baixar a **última versão do FortiClient VPN** diretamente no site oficial da Fortinet.
2. Instalar conforme o pacote fornecido (.rpm).
3. Validar o funcionamento.

> A configuração da VPN deve seguir as diretrizes do time de Redes.

---

## 6. Instalação do CrowdStrike Falcon Sensor (opcional)

### 6.1 Instalação

* Baixar a **última versão do sensor CrowdStrike** a partir da cloud oficial.
* Instalar o pacote conforme instruções do fabricante.

### 6.2 Ativação do sensor

```bash
sudo /opt/CrowdStrike/falconctl -s --cid=<CROWDSTRIKE_CID>
```

### 6.3 Inicialização e persistência

```bash
sudo systemctl start falcon-sensor
sudo systemctl enable falcon-sensor
```

### 6.4 Validação

* Confirmar com o **time de Segurança da Informação** se o sensor está ativo no console.
* Validar se o **hostname** está correto no CrowdStrike.

---

## 7. Adição do Repositório Flathub

```bash
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

---

## 8. Instalação de Aplicações via Flatpak

### 8.1 Microsoft Teams (opcional)

```bash
flatpak install app/com.github.IsmaelMartinez.teams_for_linux/x86_64/stable
```

### 8.2 Outlook (opcional)

```bash
flatpak install app/io.github.mahmoudbahaa.outlook_for_linux/x86_64/stable
```

### 8.3 Slack (opcional)

```bash
flatpak install app/com.slack.Slack/x86_64/stable
```

---

## 9. Software de Acesso Remoto

Instalação do Remmina:

```bash
sudo dnf install -y remmina
```

---

## 10. Ingresso no Domínio Active Directory

### 10.1 Pacotes necessários

```bash
sudo dnf install -y realmd sssd oddjob oddjob-mkhomedir adcli samba-common-tools
```

### 10.2 Ingresso no domínio

```bash
sudo realm join --user=<usuario_administrador_ad> dominio.local
```

### 10.3 Validação

```bash
realm list
```

---

## 11. Ajustes na Tela de Login (SDDM – KDE)

### 11.1 Configuração

```bash
sudo nano /etc/sddm.conf
```

Adicionar ou ajustar:

```ini
HideUsers=(ocultar conta de admin local que foi criado na instalação)
MaximumUid=1000
```

### 11.2 Testes

* Reiniciar o sistema
* Validar login com usuário de domínio

### 11.3 Formato de logon

```text
usuario.sobrenome@dominio.local
```

---

## 12. Netskope (opcional)

Após instalar o cliente Linux fornecido pela Netskope, executar os comandos abaixo para corrigir a exibição do ícone na bandeja:

```bash
sudo dnf install -y webkit2gtk4 webkitgtk6

sudo ln -s /usr/lib64/libwebkit2gtk-4.1.so.0 /usr/lib64/libwebkit2gtk-4.0.so.37
sudo ln -s /usr/lib64/libjavascriptcoregtk-4.1.so.0 /usr/lib64/libjavascriptcoregtk-4.0.so.18
```

Validar se o ícone aparece corretamente e testar as configurações.

---

## 13. Virtualização Nativa via KVM (opcional)

Instalar dependências e ajustar serviços:

```bash
sudo systemctl start libvirtd
sudo systemctl enable libvirtd

sudo systemctl edit cockpit.socket
```

Adicionar ao arquivo:

```ini
[Socket]
ListenStream=
ListenStream=127.0.0.1:9090
```

Ativar serviços:

```bash
sudo systemctl enable cockpit.service
sudo systemctl enable cockpit.socket
sudo systemctl start cockpit.socket
sudo systemctl start cockpit.service
```

Acessar a interface via navegador:

```text
http://localhost:9090
```

* Ativar o recurso **Virtual Machines**
* Efetuar logoff e logar com o usuário de domínio
* Criar VMs utilizando storage dentro da **home do usuário** (recomendado criar a pasta `VM`)

---

## 14. Validação Final da Workstation

Checklist de conferência pós-configuração:

```text
[ ] Hostname configurado corretamente
[ ] Usuário de domínio realiza login no KDE
[ ] CrowdStrike aparece como ativo no console
[ ] FortiClient conecta à VPN
[ ] Atualizações automáticas ativas
[ ] Horário sincronizado (timedatectl)
```

---

## 15. Observações de Segurança

* SELinux em modo **enforcing** (padrão)
* firewalld ativo
* Cockpit restrito ao localhost (127.0.0.1)

---

## 16. Metadados do Documento

```text
Arquivo: README.md
Versão: 1.1
Última revisão: 2026-01-17
Responsável: Edson Batista do Carmo Junior
```

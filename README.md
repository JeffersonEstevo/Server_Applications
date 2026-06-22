# Server_Applications

## 🐧 Guia Rápido: Ubuntu Server

O **Ubuntu Server** é gerenciado quase inteiramente por linha de comando. Dominar a navegação, o gerenciamento de pacotes e a estrutura de pastas é o primeiro passo para operar o sistema com eficiência.

---

### 🛠️ Comandos Essenciais

* `sudo`: Executa comandos com privilégios de administrador (root).
* `cd [diretório]`: Navega entre as pastas do sistema.
* `ls -la`: Lista todos os arquivos da pasta atual com detalhes e permissões.
* `mkdir [nome]`: Cria uma nova pasta.
* `rm -rf [nome]`: Apaga arquivos ou pastas permanentemente sem pedir confirmação.
* `cp [origem] [destino]`: Copia arquivos ou pastas de um local para o outro.
* `mv [origem] [destino]`: Move ou renomeia arquivos e pastas.
* `sudo apt update && sudo apt upgrade`: Atualiza as listas e os pacotes do sistema.
* `sudo nano [arquivo]`: Abre um editor de texto simples diretamente no terminal.
* `systemctl [ação] [serviço]`: Controla serviços (ex: `sudo systemctl restart nginx`).
* `df -h`: Mostra o espaço em disco disponível em formato legível (GB, MB).
* `top` ou `htop`: Exibe os processos em execução e o uso de memória/CPU.

---

### 📂 Principais Diretórios

* `/`: A **raiz** do sistema. Tudo começa a partir deste ponto.
* `/etc`: Central de arquivos de **configuração** de programas e do sistema.
* `/home`: Pastas pessoais dos **usuários** comuns do servidor.
* `/root`: Pasta pessoal exclusiva do usuário **administrador** (root).
* `/var`: Arquivos **variáveis**, como os bancos de dados e logs do sistema.
* `/var/log`: Pasta que armazena os **logs** (histórico de eventos) para diagnóstico.
* `/tmp`: Arquivos **temporários** que costumam ser apagados ao reiniciar.
* `/bin` e `/sbin`: Arquivos **executáveis** e comandos essenciais do sistema.

---

### 🌐 Redes no Ubuntu Server (VirtualBox)

O Ubuntu Server utiliza o **Netplan** para gerenciar a rede através de arquivos YAML. 

#### 1. Comandos de Diagnóstico
* `ip a` ou `ip address`: Identifica as placas de rede disponíveis e seus IPs (ex: `enp0s3`).
* `ping [ip_ou_site]`: Testa a conectividade com a internet ou outros servidores.

#### 2. Dica de Ouro para VirtualBox
Antes de configurar o sistema, verifique o padrão da sua VM nas configurações do VirtualBox:
* **Modo NAT**: Dá internet à VM, mas seu computador físico não consegue acessar o servidor diretamente por IP.
* **Placa em modo Bridge (Rede em Ponte)**: A VM se torna um dispositivo real na sua rede física, recebendo um IP do seu roteador. Ideal para testes locais.

#### 3. Configurando IP Estático (Netplan)
Os arquivos de rede ficam armazenados no diretório `/etc/netplan/`. No Ubuntu Server, o arquivo padrão geralmente chama-se `00-installer-config.yaml`, `01-netcfg.yaml` ou `50-cloud-init.yaml`.

Para descobrir o nome exato da sua interface de rede antes de editar, execute:
```bash
ip a
```

Para editar o arquivo de configuração de rede:
```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

##### Estrutura Correta para IP Estático
O formato YAML é extremamente sensível a espaços. **Nunca utilize a tecla TAB** para fazer os recuos; utilize estritamente a barra de espaço, seguindo o alinhamento vertical abaixo:

```yaml
network:
  version: 2
  ethernets:
    enp0s3:                  # Nome da sua interface de rede
      dhcp4: no              # Desativa a busca por IP automático (IPv4)
      dhcp6: no              # Desativa a busca por IP automático (IPv6)
      addresses:
        - 192.168.1.70/24    # IP estático desejado para o servidor e a máscara (/24)
      routes:
        - to: default        # Rota padrão para saída de dados
          via: 192.168.1.254 # Endereço IP do seu Gateway (Roteador)
      nameservers:
        addresses:
          - 8.8.8.8          # Servidor DNS Primário (Google)
          - 1.1.1.1          # Servidor DNS Secundário (Cloudflare)
```

> **Dica de ouro sobre o alinhamento:** A linha `via:` deve iniciar **exatamente abaixo do hífen (`-`)** da linha `  - to: default`. Mover o `via:` para a frente ou para trás causará erros de sintaxe (`Invalid YAML`).

##### Automação via Terminal (Alternativa ao Editor)
Caso a formatação manual via editor apresente erros persistentes devido a caracteres invisíveis gerados pelo terminal ou conexão SSH, é possível reescrever o arquivo aplicando o espaçamento perfeito diretamente pelo terminal com o comando:

```bash
cat << 'EOF' | sudo tee /etc/netplan/00-installer-config.yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: no
      dhcp6: no
      addresses:
        - 192.168.1.70/24
      routes:
        - to: default
          via: 192.168.1.254
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
EOF
```

### 4. Aplicando as Alterações
Após salvar o arquivo, execute os comandos abaixo para validar e ativar as configurações:

* `sudo netplan try`: Comandos de teste recomendados. Valida a sintaxe e inicia uma contagem regressiva de 2 minutos. Se você perder o acesso ao servidor ou a sintaxe estiver incorreta, o Netplan reverte as alterações automaticamente ao fim do cronômetro. Basta apertar **Enter** se tudo estiver correto para confirmar.
* `sudo netplan apply`: Aplica as novas configurações imediatamente. 

> **Atenção ao acessar via SSH:** Ao executar o `sudo netplan apply` com um novo IP estático configurado, a sua sessão SSH atual cairá imediatamente com o erro `client_loop: send disconnect: Connection reset`. Isso é esperado, pois o servidor mudou de endereço. Para retomar o controle, abra um novo terminal em sua máquina local e conecte-se utilizando o novo IP definido: `ssh usuario@192.168.1.70`.
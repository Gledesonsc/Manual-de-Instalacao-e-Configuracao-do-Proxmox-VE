# Manual de Instalação e Configuração do Proxmox VE  
_Implementação de VMs/Containers para Easypanel e Portainer_

Este manual descreve, de forma detalhada, os passos para instalar e configurar o Proxmox VE em uma máquina com as seguintes características:  
- **CPU:** Intel i5 (7ª geração)  
- **Placa de Vídeo:** Radeon AMD  
- **Placa-Mãe:** ASUS  
- **Armazenamento:**  
  - SSD SATA de 240 GB (recomendado para o sistema Proxmox)  
  - HD de 4 TB (para armazenamento de VMs, containers ou dados)

O objetivo é preparar o ambiente para hospedar dois serviços:  
- **Easypanel**  
- **Portainer**

Além disso, o manual contempla a configuração para acesso externo, apontamento de domínio via Cloudflare e as liberações necessárias no firewall WatchGuard.

---

## 1. Pré-Requisitos

- **Download da ISO do Proxmox VE:**  
  Baixe a última versão do Proxmox VE em [https://www.proxmox.com/en/downloads](https://www.proxmox.com/en/downloads).

- **Mídia de Instalação:**  
  Prepare um pendrive bootável com a ISO. Você pode usar ferramentas como o [Rufus](https://rufus.ie/) (no Windows) ou o `dd` (no Linux).

- **Acesso ao Ambiente de Rede:**  
  Certifique-se de que a máquina terá um IP fixo na rede interna e que o endereço IP público (ou DDNS) estará configurado para apontar via Cloudflare.

- **Informações do Firewall WatchGuard:**  
  Anote as portas a serem liberadas (por exemplo, 80, 443, 8006, 8080, 9000 ou outras definidas pelos serviços).

---

## 2. Instalação do Proxmox VE

### 2.1 Preparação do Hardware e Boot

1. **Inserir a Mídia de Instalação:**
   - Conecte o pendrive bootável na máquina.
   - Configure na BIOS/UEFI para dar boot pela mídia USB.

2. **Iniciar a Instalação:**
   - Ao iniciar, será exibida a tela de boas-vindas do Proxmox.
   - Selecione `Install Proxmox VE` e pressione **Enter**.

### 2.2 Processo de Instalação

1. **Aceitar os Termos de Licença:**
   - Leia e aceite os termos de licença. Utilize a tecla **Enter** para prosseguir.

2. **Selecionar o Disco para Instalação:**
   - Escolha o **SSD de 240 GB** como disco de instalação do Proxmox VE.  
     _Dica:_ Utilize o HD de 4 TB posteriormente para armazenar os dados das VMs ou containers.
   - Clique em **Next**.

3. **Configuração do País, Fuso Horário e Layout do Teclado:**
   - Selecione o país, fuso horário e layout de teclado conforme sua preferência.

4. **Definir Senha e E-mail do Administrador:**
   - Informe uma senha forte para o usuário `root` e um e-mail para notificação (opcional).

5. **Configuração de Rede:**
   - Defina o endereço IP fixo para o Proxmox (recomendado para facilitar o encaminhamento no firewall e a configuração do DNS).
   - Informe o gateway e o servidor DNS adequado.
   - Clique em **Next** para confirmar as configurações.

6. **Resumo e Instalação:**
   - Revise as configurações escolhidas e clique em **Install** para iniciar o processo.
   - Aguarde a conclusão (o tempo pode variar conforme o hardware).

7. **Finalizar e Reiniciar:**
   - Uma vez concluída a instalação, remova o pendrive e reinicie o sistema.

---

## 3. Configuração Pós-Instalação do Proxmox VE

### 3.1 Acesso ao Painel Web do Proxmox

1. **Conectar na Rede:**
   - Após a reinicialização, conecte um computador à mesma rede e abra um navegador.
   - Acesse o painel do Proxmox pelo endereço:  
     `https://<IP_do_Proxmox>:8006`  
     _Nota:_ Se o navegador alertar sobre o certificado, aceite o aviso (o certificado autoassinado pode ser trocado por um certificado válido posteriormente).

### 3.2 Configuração do Armazenamento

1. **Adicionar o HD de 4 TB:**
   - No painel do Proxmox, acesse **Datacenter > Storage**.
   - Clique em **Add** e selecione o tipo de armazenamento desejado (por exemplo, **Directory** ou **LVM**).
   - Configure o HD de 4 TB para armazenar os dados das VMs/containers.

### 3.3 Atualizações e Configurações Básicas

1. **Atualizar o Sistema:**
   - Acesse o shell do Proxmox (via web ou SSH) e execute:
     ```bash
     apt update && apt dist-upgrade -y
     ```
2. **Configurar o Backup:**
   - Configure rotinas de backup para os containers e VMs através do painel, garantindo maior segurança dos dados.

---

## 4. Criação de VMs ou Containers para os Serviços

Você pode optar por criar **VMs** (máquinas virtuais completas) ou **containers LXC** (mais leves) para hospedar o Easypanel e o Portainer.

### 4.1 Criação de um Container LXC (Exemplo para Easypanel)

1. **Criar o Container:**
   - No painel do Proxmox, clique em **Create CT**.
   - Configure o ID, nome (por exemplo, `easypanel`), e defina os recursos (CPU, memória, etc.).
   
2. **Escolher a Template:**
   - Selecione uma template Linux adequada (por exemplo, Debian ou Ubuntu) para instalar o Easypanel.
   
3. **Definir Armazenamento e Rede:**
   - Configure o armazenamento utilizando o HD de 4 TB.
   - Configure o adaptador de rede para que o container receba um IP (recomendado IP fixo dentro da sub-rede).

4. **Concluir a Criação:**
   - Revise as configurações e clique em **Finish**.

5. **Instalar o Easypanel:**
   - Acesse o container via console do Proxmox ou SSH.
   - Siga a documentação do Easypanel para a instalação (normalmente envolve baixar a imagem Docker ou pacotes específicos, conforme a documentação oficial).

### 4.2 Criação de um Container LXC para Portainer

1. **Repetir os Passos:**
   - Crie outro container LXC com um sistema Linux leve.
   - Configure os recursos, armazenamento e rede, conforme feito para o Easypanel.

2. **Instalar o Docker e Portainer:**
   - Dentro do container, instale o Docker:
     ```bash
     apt update && apt install docker.io -y
     systemctl enable docker && systemctl start docker
     ```
   - Execute o container do Portainer:
     ```bash
     docker run -d -p 9000:9000 --name portainer \
       --restart=always \
       -v /var/run/docker.sock:/var/run/docker.sock \
       -v portainer_data:/data \
       portainer/portainer-ce
     ```

---

## 5. Configuração para Acesso Externo

### 5.1 Configuração do DNS no Cloudflare

1. **Adicionar o Domínio:**
   - Acesse sua conta Cloudflare e adicione seu domínio.
   - Atualize os servidores DNS no registrador de domínios para os fornecidos pelo Cloudflare.

2. **Criar Registros A/CNAME:**
   - Crie registros para apontar para o IP público da sua empresa. Por exemplo:
     - `easypanel.seudominio.com → IP público`
     - `portainer.seudominio.com → IP público`

3. **Configurar Proxy e SSL:**
   - Habilite o proxy do Cloudflare (ícone de nuvem laranja).
   - Configure a opção “Full” ou “Full (strict)” para o SSL, ou utilize certificados do Let’s Encrypt na camada do Proxmox/reverse proxy.

### 5.2 Configuração do Firewall WatchGuard

1. **Liberar Portas Necessárias:**
   - **Porta 8006:** Para acesso ao painel do Proxmox (opcional, se não exposto para a internet).
   - **Portas 80 e 443:** Para HTTP/HTTPS – utilizadas pelo Cloudflare e para acesso aos serviços.
   - **Portas Específicas dos Containers:** Caso os serviços estejam rodando em portas não padrão (por exemplo, 9000 para o Portainer ou a porta designada para o Easypanel).

2. **Configurar NAT/Port Forwarding:**
   - Configure regras de NAT no WatchGuard para encaminhar o tráfego das portas definidas para o IP fixo do Proxmox ou dos containers.
  
3. **Regras de Segurança:**
   - Se possível, limite os IPs que podem acessar os painéis administrativos ou utilize VPN para acesso interno.

---

## 6. Testes e Validação

1. **Teste o Acesso Interno:**
   - Acesse o painel do Proxmox e os consoles dos containers/VMs para confirmar que os serviços estão online.

2. **Verifique a Resolução DNS:**
   - Acesse os endereços `https://easypanel.seudominio.com` e `https://portainer.seudominio.com` de uma conexão externa para garantir que o apontamento via Cloudflare está funcionando.

3. **Teste as Regras do Firewall:**
   - Verifique se o tráfego externo está corretamente direcionado e se não há bloqueios inesperados.

4. **Certificados SSL:**
   - Confirme que os certificados SSL estão ativos e que o acesso é feito via HTTPS.

---

## 7. Considerações Finais e Manutenção

- **Backups Regulares:**  
  Configure backups automáticos dos containers/VMs e do armazenamento de dados.
  
- **Monitoramento:**  
  Utilize as ferramentas integradas do Proxmox para monitorar recursos e revise periodicamente as regras do firewall.
  
- **Atualizações:**  
  Mantenha o Proxmox VE e os sistemas instalados atualizados para garantir segurança e performance.

- **Documentação dos Serviços:**  
  Consulte sempre a documentação oficial dos serviços (Easypanel, Portainer, Cloudflare e WatchGuard) para atualizações e melhores práticas.

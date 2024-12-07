# Born To Be Root

**Desafio**: criar uma máquina virutal sob instruções específicas.

## Requisitos

- Usar obrigatoriamente VirtualBox (ou UTM se não for possívelusar o VirtualBox).
- Disponibilizar um arquivo `signature.txt` contendo à assinatura do disco virtual da máquina criada.

### Parte Mandatória 

- Escolher como sistema operacional a última versão estável do Debian ou do Rocky.
- Ter instalado o mínimo de serviços (apenas os necessários para avaliação do desafio). 
- A máquina virtual não deve ter qualquer interface gráfica.
- Ter criado pelo menos 2 partições criptografadas usando LVM.
- Estar com o serviço SSH rodando somente na porta 4242 e não deve serpossível conectar usando SSH como root.
- Ter configurado seu sistema operacional com o firewall UFW (ou firewalld para Rocky), deixando apenas a porta 4242 aberta.
- O nome do host da sua máquina virtual deve ser seu login terminando com 42.
- Ter implementado uma política de senha forte.
- Instalar e configurar o sudo seguindo regras rígidas.
- Ter além do usuário root, um usuário com o login como username.
- O usuário (não root) tem que pertencer aos grupos user42 e sudo.
- A política de senha deve ser forte, cumprindo os seguintes requisitos:
	- Expirar a cada 30 dias.
	- O número mínimo de dias permitido antes da modificação de uma senha erá definido como 2.
	- O usuário deve receber uma mensagem de aviso 7 dias antes da senha expirar.
	- Deve ter pelo menos 10 caracteres e ela deve conter:
		- Um número.
		- Uma letra maiúscula.
		- Uma letra minúscula.
		- Não deve conter mais de 3 caracteres idênticos consecutivos.
	- Não deve incluir o nome do usuário.
	- Deve ter pelo menos 7 caracteres que não façam parte da senha anterior (não aplicado ao root)
- As configurações de grupo sudo, devem cumprir os seguintes requisitos:
	- A autenticação usando sudo deve ser limitada a 3 tentativas no caso de uma senha incorreta.
	- Uma mensagem personalizada de sua escolha deve ser exibida se um erro devido à uma senha errada ocorrer ao usar sudo.
	- Cada ação usando sudo deve ser arquivada, tanto entradas quanto saídas. O arquivo de log deve ser salvo na pasta `/var/log/sudo/`.
	- O modo TTY deve ser habilitado por razões de segurança.
	- Por razões de segurança também, os caminhos que podem ser usados ​​pelo sudo devem ser restritos.
		`/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin`
- Ter um script simples chamado `monitoring.sh` (ele deve ser desenvolvido em bash), onde na inicialização do servidor, o script exibirá algumas informações (conforme abaixo) em todos os terminais a cada 10 minutos. Nenhum erro deve estar visível.
	- A arquitetura do seu sistema operacional e sua versão do kernel.
	- O número de processadores físicos.
	- O número de processadores virtuais.
	- A RAM disponível no seu servidor e sua taxa de utilização como uma porcentagem.
	- A memória disponível no seu servidor e sua taxa de utilização como uma porcentagem.
	- A taxa de utilização atual dos seus processadores como uma porcentagem.
	- A data e a hora da última reinicialização.
	- Se o LVM está ativo ou não.
	- O número de conexões ativas.
	- O número de usuários usando o servidor.
	- O endereço IPv4 do seu servidor e seu endereço MAC (Media Access Control).
	- O número de comandos executados com o programa sudo.

### Parte Bonus

- Configurar as partições corretamente para obter uma estrutura semelhante à abaixo:
- Configurar um site WordPress funcional com os seguintes serviços: lighttpd, MariaDB e PHP.
- Configurar um serviço de sua escolha que você acha útil (NGINX / Apache2 excluídos!).

### Procedimentos realizados

- Feito o download da última versão estável da distro Debian, em `https://www.debian.org`
- Criado uma nova máquina virtual, direcionando o conteúdo para uma pasta específca na raiz do usuário.
- Configurado como dispositivo de armazenamento a ISO baixada.
- Inicializado a máquina seguindo o guia de instação do sistema operacional.
	- Iniciado com particionamento manual, alocando a quantidade necessária conforme o subject do desafio (1 Partição primária como ponto de partida (boot) e o restante do espaço livre como partição lógica com volumes criptografados)
	- Criado e configurado cada volume lógico conforme particionamentos pedido no subject do projeto.
	- Desabilitado todas as pré definições de opções software para serem instalados;
- Realizado atualização dos pacotes do sistema com os comandos `update` e `upgrade` por meio do `apt-get`
- Instalado o pacote sudo atraavés do comando `apt install sudo`
- Reinicializado o sistema com `sudo reboot` e verificado se o sudo foi instalado corretamente, com o comando `sudo -V`.
- Criado um novo grupo com o comando: `sudo addgroup user42`
- Adicionado o usuário (não root) ao grupo `sudo` e `user42`, através do comando: `adduser <user> <group>`
- Verificado se de fato o usuário foi adicionado ao grupo `sudo` usando o comando `getent group <group>`
- Instalado o `ssh` por meio do comando `apt install openssh-serve`
- Testado a conexão ssh, acessando o usuário (não root) pela porta 4242 e outras portas.
	- Neste ponto, foi necessário configurar a rede da Virtual Box como Bridgeem vez de NAT, a infra da 42  não tem apenas um servidor (não estão todos na mesma rede), com a configuração de NAT, o endereço de IP ficaram com números bem diferentes e não dava para realizar a conexão.
	

- Alterado o arquivo `/etc/ssh/sshd_config` e `/etc/ssh/ssh_config` atualizando a porta a ser utilizada pelo servidor ssh (`4242`) e para não permitir acesso através do root.
- Verificado o status da instação com o comando: `systemctl status ssh` e se esta usando a porta `4242`
- Instaldo o serviço `ufw` por meio do comando `apt` e habilitado.
- Habilitado o serviço ufw com o comando: `ufw enable`
- Verificado o status do serviço com o comando `ufw status numbered`
- Adicionado como regra, a permissão para a porta `4242` através de `ufw allow <rule>`


- Configurado senha forte para os usuários
	- Criado a pasta onde os comandos de execução do sudo serão salvos `/var/log/sudo`
	- Criando um arquivo em `/etc/sudoers.d/sudo_config` e configurado variáveis para politícas específicas.
- Editado o arquivo `/etc/login.defs` para configurar o tempo para senha experiar e o mínimo para muda-lá.
- Instalado o pacote `limpam-pwquality` para gerenciar a politica de senha.
- Editado o arquivo `/etc/pam.d/common-password` adicionando mais configurações para politica como tamanho mínimo, letra maiscula, maximo de repetição, etc.
- Alterado os usuários para corresponder a politica de expiração de senha, usando `sudo chage -m <days> <user>` e `sudo chage -M <days> <user>` onde `m`refere-se a tempo minimo para mudar e `M` tempo maxímo para expirar
- Atualizado as senhas dos usuários exitntes (includo root) validando o comportamento da politica configurada.


- Configuração do script de monitoramento.
- Verifica à arquitetura do SO: `uname -a`
- Verifca o número de núcleos físicos:`grep "physical id" /proc/cpuinfo | wc -l`
- Verifca o número de núcleos virtuais: `grep processor /proc/cpuinfo | wc -l`
- Consulta a memória utilizada usando o `free` em conjunto do awk para filtrar a parte do processamento de texto desejado: `free --mega | awk '$1 == "Mem.:" {print $3}'`
- Idem a anterior, porém para verificar a memória total `free --mega | awk '$1 == "Mem.:" {print $2}'`
- Calcula o percentual da memória utilizada usando o `awk` para processamento e `printf` para formatação: `free --mega | awk '$1 == "Mem.:" {printf("(%.2f%%%%)\n", $3/$2*100)}'`
- Verifica a memória em disco ocupada: `df -m | grep "/dev/" | grep -v "/boot" | awk '{memory_use += $3} END {print memory_use}'`
- Verifica a memória total do disco: `df -m | grep "/dev/" | grep -v "/boot" | awk '{memory_result += $2} END {printf("%0fGB\n"), memory_result/1024}'`
- Calcula o percentual de disco utilizado `df -m | grep "/dev/" | grep -v "/boot" | awk '{use += $3} {total += $2} END {printf("(%d%%%%)}, use/total*100}'`
- Calcula o percentual de utilização da CPU, obtendo o percentual ocioso da CPU usando para subtrair de 100 e obter o percentual de uso: `expr 100 - $(vmstat | tail -1 | awk '{printf $15}')`
- Verifica o endereço de IP: `hostname -I`
- Verifica o endereço do MAC: `ip link | grep "link/ether" | awk '{print $2}'`
- Verifica os comandos sudo executados, aplicando processamento para contar as linhas: `journalctl _COMM=sudo | grep COMMAND | wc -l`
- Criação do arquivo `monitoring.sh` com base nos comandos de sistema acima utilizados
- Executa o `crontab` para configurar a execução do script de 10 em 10 minutos:
	- Inicializa um novo configuração através do root: `sudo crontab -u root -e`
	- Configura no arqivo o comando: `*/10 * * * * sh /home/<username>/monitoring.sh`


### Notas de execução

- `lsblk`: lista as informações sobre todos os dispositivos de bloco disponíveis no sistema.
- `apt`: ferramenta de linha de comando simples e direta.
- `aptitude`: oferece uma interface de linha de comando e uma interface de usuário baseada em texto (TUI) mais amigável.
- `sudo`: usado para permitir que um usuário execute comandos com privilégios de superusuário (root) ou outro usuário especificado.
- `apt-get`: ferramenta de linha de comando usada em distribuições Linux baseadas em Debian para gerenciar pacotes de software.
- `apt`: ferramenta de linha de comando usada para gerenciar pacotes em distribuições Linux baseadas em Debian, como Ubuntu. Ele combina funcionalidades de várias ferramentas de gerenciamento de pacotes (`apt-get`, `apt-cache`, etc.)
- `usermod`: usado para modificar as propriedades de uma conta de usuário existente, permitindo alterar as configurações de um usuário.
- `getent`: usado para consultar várias bases de dados do sistema, como as listadas em `/etc/nsswitch.conf`, também pode ser usado para obter informações sobre usuários, grupos, hosts, serviços, entre outros.
- `systemctl`: ferramenta de linha de comando usada para controlar o sistema de inicialização e gerenciamento de serviços no Linux, especificamente com o `systemd`, permitindo iniciar, parar, reiniciar, habilitar, desabilitar e verificar o status de serviços e unidades do sistema.
	- `systemd`: sistema de inicialização e gerenciamento de serviços para sistemas operacionais Linux, responsável por inicializar o sistema e gerenciar processos durante a execução.
- `grep`: usado para pesquisar texto dentro de arquivos, procurando por padrões específicos de texto e exibe as linhas que correspondem a esses padrões.
- `uname`: exibe informações sobre o sistema operacional, como o nome do kernel, a versão, a arquitetura, entre outros detalhes.
- `crontab`: arquivo que contém a lista de tarefas programadas para ser executadas em horários específicos, permitindo agendar a execução de scripts, comandos e programas em intervalos regulares.
- `awk`: ferramenta de processamento de texto usada para pesquisar e manipular dados em arquivos de texto, permitindo filtrar, formatar e processar informações de maneira eficiente.
- `printf`: função de formatação de saída usada para exibir texto formatado em um terminal, permitindo especificar o formato de saída, como números decimais, hexadecimais, octais, entre outros.
- `expr`: avalia expressões matemáticas e lógicas, permitindo realizar cálculos e comparações em scripts de shell.
- `journalctl`: ferramenta de linha de comando usada para consultar e exibir mensagens de log do sistema, permitindo visualizar registros de eventos, erros e informações do sistema.
- `hostname`: comando usado para exibir ou configurar o nome do host do sistema, permitindo identificar e acessar um computador em uma rede.
- `ip`: ferramenta de linha de comando usada para exibir e configurar informações de rede, permitindo gerenciar interfaces de rede, endereços IP, rotas, tabelas ARP, entre outros.
- `vmstat`: ferramenta de linha de comando usada para exibir informações sobre a utilização de memória, processador, E/S de disco, entre outros recursos do sistema.
- `tail`: comando usado para exibir as últimas linhas de um arquivo de texto, permitindo visualizar as últimas entradas de um log, por exemplo.
- `chage`: comando usado para alterar as configurações de expiração de senha de um usuário, permitindo definir a validade da senha, o tempo mínimo e máximo para alteração, entre outras opções.
- `df`: comando usado para exibir informações sobre o espaço em disco disponível e utilizado em sistemas de arquivos, permitindo verificar a capacidade, uso e disponibilidade de armazenamento.
- `limpam-pwquality`: biblioteca usada para verificar e aplicar políticas de qualidade de senha em sistemas Linux, permitindo definir requisitos de segurança para senhas de usuários.
- `free`: comando usado para exibir informações sobre a memória física e de troca disponível e utilizada no sistema, permitindo monitorar a utilização de memória.
- `adduser`: comando usado para adicionar um novo usuário ao sistema, permitindo criar contas de usuário com permissões e configurações específicas.
- `addgroup`: comando usado para adicionar um novo grupo ao sistema, permitindo agrupar usuários com permissões e configurações comuns.
- `ufw`: ferramenta de firewall usada para configurar e gerenciar regras de firewall no Linux, permitindo controlar o tráfego de rede e proteger o sistema contra acessos não autorizados.
- `sudoers`: arquivo de configuração usado para definir políticas de acesso ao comando `sudo`, permitindo especificar quais usuários e grupos têm permissão para executar comandos com privilégios de superusuário.
- `sudoers.d`: diretório usado para armazenar arquivos de configuração adicionais para o `sudo`, permitindo definir políticas de acesso personalizadas para usuários e grupos específicos.
- `ssh`: protocolo de rede usado para acessar e gerenciar sistemas remotos de forma segura, permitindo a transferência de arquivos, a execução de comandos e a interação com o sistema remoto.
- `ssh_config`: arquivo de configuração usado para definir as configurações padrão do cliente SSH, permitindo especificar opções de conexão, autenticação, chaves, entre outros.
- `sshd_config`: arquivo de configuração usado para definir as configurações do servidor SSH, permitindo configurar opções de segurança, portas, autenticação, chaves, entre outros.
- `sudo_config`: arquivo de configuração usado para definir políticas de acesso ao comando `sudo`, permitindo especificar regras e restrições para usuários e grupos.

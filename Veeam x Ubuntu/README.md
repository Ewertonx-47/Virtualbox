VINCULANDO VEEAM AGENT COM UBUNTU NO VIRTUALBOX

O Veeam é uma ferramenta poderosa para backup, mas para ele "enxergar" o Ubuntu dentro do VirtualBox, é preciso de um componente adicional. O Veeam que está no host (Windows) não consegue acessar o disco da máquina virtual diretamente de forma gerenciada. A chave para a comunicação é o Veeam Agent for Linux, um software que deve ser instalado dentro do sistema operacional convidado (o Ubuntu). Ele funciona como um "tradutor" que se comunica com o servidor Veeam no host, permitindo que o Veeam realize e gerencie os backups.

PROCESSO BÁSICO DE COMUNICAÇÃO
1. Baixar o Agente: Acesse o site da Veeam e baixe o instalador do Veeam Agent for Linux. Para o Ubuntu, você precisará do arquivo .deb.

2. Transferir o Arquivo: Passe o arquivo .deb para a máquina virtual Ubuntu. Uma forma fácil e segura é usar uma pasta compartilhada do VirtualBox.

3. Instalar o Agente: Dentro do seu Ubuntu, instale o pacote usando o terminal.

Instalando o Guest Additions (para Pastas Compartilhadas)
Primeiro, é necessário criar uma pasta compartilhada entre o host e o Ubuntu. O caminho para isso é:

clicar na VM > Configurações > Pasta compartilhada

![PASTA](../Assets/virtualbox/selecionar_pasta.png)

Depois de mapear a pasta, é preciso instalar um pacote de software que faz a ponte entre a VM e o host, para que a VM consiga enxergar a pasta compartilhada. Esse pacote é o Guest Additions. A ISO do Guest Additions já é inserida no drive virtual de DVD/CD da VM no momento da instalação do sistema operacional. O Linux enxerga todos os dispositivos como arquivos, e o drive onde está o Guest Additions é o arquivo /dev/cdrom.

Para que a máquina virtual possa usar os arquivos do Guest Additions, é preciso montar o drive cdrom em uma pasta vazia.

sudo mkdir -p /mnt/virtualbox (Cria uma pasta vazia dentro do diretório /mnt).

sudo mount /dev/cdrom /mnt/virtualbox (Monta o drive cdrom na pasta virtualbox).

O diretório /mnt é usado para montagens manuais e temporárias, uma prática padrão em distribuições Linux.

Agora, é preciso instalar alguns pacotes de desenvolvimento que são necessários para que o Guest Additions funcione corretamente.

sudo apt-get update && sudo apt-get install build-essential dkms

Em seguida, navegue até a pasta onde o drive está montado e execute o instalador do Guest Additions.

cd /mnt/virtualbox

sudo ./VBoxLinuxAdditions.run

![ERRO](../Assets/Erros/erro_instalador.png)

Se a instalação der um erro de falta de pacotes, instale os que estão faltando.

sudo apt-get install gcc make perl

Navegue até o ponto de montagem novamente e execute o instalador com sudo ./VBoxLinuxAdditions.run.

![OK](../Assets/Utilitarios/instalador_ok.png)

Conforme a imagem, a execução do instalador funcionou. Agora, é essencial reiniciar a máquina para que o pacote funcione corretamente.

Configurando Permissões para a Pasta Compartilhada
Depois de reiniciar, o VirtualBox monta a pasta compartilhada no diretório /media. 

![MEDIA](../Assets/Utilitarios/diretorio_media.png)

Não foi possível acessar o contéudo da pasta compartilhada.

![PERMISSAO](../Assets/Erros/erro_permissao.png)

No entanto, para acessar o conteúdo, é necessário incluir o seu usuário (ewerton) em um grupo específico chamado vboxsf.

sudo usermod -aG vboxsf ewerton

sudo: Executa o comando como superusuário (administrador).

usermod: É o comando para modificar um usuário.

-aG: Adiciona (-a) o usuário a um grupo suplementar (-G).

vboxsf: É o nome do grupo que controla as pastas compartilhadas.

ewerton: O nome do seu usuário.

Reinicie a máquina novamente para que a mudança de grupo tenha efeito.

reboot

Agora sim, o acesso à pasta compartilhada é permitido.

Instalando o Veeam Agent for Linux
A partir da pasta compartilhada, use o dpkg para instalar o arquivo do repositório da Veeam.

cd /media/sf_Downloads

![DOWNLOADS](../Assets/Utilitarios/sf_downloads.png)

sudo dpkg -i veeam-release-deb_1.0.9_amd64.deb

É crucial entender que o arquivo veeam-release-deb_1.0.9_amd64.deb não é o Veeam Agent completo. Ele é um pacote de configuração que informa ao seu sistema onde encontrar o Veeam Agent. A função do dpkg -i aqui é apenas instalar esse arquivo de configuração localmente.

Para instalar o Veeam Agent, é necessário usar o apt, um comando mais "inteligente" que consegue buscar na internet e instalar todos os pacotes do serviço e suas dependências.

sudo apt-get update

sudo apt-get install veeam

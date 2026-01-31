Nesse lab será feito uma analise das entradas do firmware UEFI do sistema Linux. A execução desse trabalho será feito no ambiente do Virtualbox, pois o padrão de firmware da máquina que utiliza o Proxmox é BIOS, no qual tem uma analise e execução diferente.

Antes de qualquer coisa, o UEFI ele tem um comportamente diferente da BIOS, enquanto a BIOS faz o processo de POST e busca o MBR nos primeiros 512bytes do primeiro disco da máquina, o UEFI ele faz o post e busca as informações na NVRAM (memória não volátil) que está atrelado na placa mãe. A partir desso ponto, o UEFI segue o caminho que foi visualizado na memória. 

Com um comando, conseguimos visualizar e lê as variáveis armazenadas na NVRAM.

* efibootmgr

(imagem)

Na imagem, podemos identificar as seguintes situações:

-BootCurrent: 003 - Qual entrada atual que foi utilizada.
-Timeout: 0 seconds - É o tempo que o menu da BIOS/UEFI espera antes de carregar a opção padrão.
-BooOrder: 003,000,001 - Esta é a hierarquia de boot. O computador tenta primeiro a 0003 (Ubuntu). Se ela falhar, tenta a 0000 e assim por diante.
-* (estrela) - Entrada ativa.

Por exemplo o 0003 que possui as seguintes informações:

"Ubuntu HD(1,GPT,b275e370...)/File(\EFI\ubuntu\shimx64.efi)"

HD(1,GPT,...): O UEFI identifica que deve olhar para a primeira partição de um disco com tabela de partição GPT. Aquele código longo é o UUID da partição ESP.

File(\EFI\ubuntu\shimx64.efi): Aqui está o "mapa" que o firmware segue.

Caso essa entrada não funcione, o firmware tentará a segunda opção e assim por diante. 

Utilizando um comando para listar as partições e discos que foram identificados pelos sistema operacional, podemos visualizar como está a divisão e formatação das partições.

* lsblk -f

(imagem)

Na imagem podemos observar que a partição sda1 ela tem uma formatação diferente da sda2. em sda1 temos FAT32 e em sda2 temos EXT4. A ESP é formatada em FAT32 por ser um requisito da especificação técnica UEFI. Como a ESP é lida pelo firmware antes do sistema operacional carregar, ela precisa de um sistema de arquivos simples e amplamente suportado que não exija drivers complexos (como NTFS ou EXT4). Pode-se perceber também que a partição está montada em boot/efi, que é o caminho que a NVRAM armazena para que a UEFI possa ver na hora de bootar. 

O que é /dev/sda1 nesse cenário?

-Ela NÃO contém kernel
-Ela NÃO contém /etc
-Ela NÃO é /boot
-Ela contém executáveis UEFI.

Agora, olhando dentro da partição ESP, sem mexer em nada, podemos visualizar os seguintes aquivos:

* ls -R /boot/efi

(imagem)

No path /boot/efi/EFI/ubuntu temos os arquivos .efi:

-grubx64.efi 
-shimx64.efi 

Esses são os executaveis que a UEFI procura para executar. Primeiro o UEFI executa shimx64.efi que é um arquivo de segurança para validar se o proximo executável de fato é o grubx64.efi. Nada mais que uma camada de segurança.




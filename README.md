Cenário Simplificado de Acesso Web no Packet Tracer (Sem ACL)
A ideia é que qualquer PC na rede local (192.168.1.0/24) consiga acessar o servidor web (208.67.222.10).
1. Dispositivos Necessários
• PCs (Hosts Clientes): Pelo menos 1, mas você pode adicionar mais.
• 1 Switch: (Ex: 2960) Para conectar os PCs e o Roteador1 na rede local.
• Roteador1 (Roteador Principal/Gateway da LAN): (Ex: 4331 ou 1941) Conecta a LAN à "internet
simulada".
• Roteador_ISP (Roteador do Provedor/Simulação de Internet): (Ex: 4331 ou 1941) Conecta o
Roteador1 ao Servidor Web.
• 1 Servidor Web: Onde a página será hospedada.
2. Conectando os Dispositivos
• PCs ao Switch: Cabos "Copper Straight-Through" (Cobre Direto).
• Switch ao Roteador1: Cabo "Copper Straight-Through" (da porta FastEthernet do Switch para
GigabitEthernet0/0/0 do Roteador1).
• Roteador1 ao Roteador_ISP:
o Verifique se ambos os roteadores possuem módulos seriais (ex: HWIC-2T). Se não, desligue o
roteador, adicione o módulo na aba "Physical" e ligue-o novamente.
o Use um cabo "Serial DCE". Conecte Serial0/1/0 do Roteador1 à Serial0/1/0 do Roteador_ISP. O
lado do cabo com o ícone de relógio no Packet Tracer é o DCE.
• Roteador_ISP ao Servidor Web: Cabo "Copper Straight-Through" (da porta GigabitEthernet0/0/0 do
Roteador_ISP para FastEthernet0 do Servidor Web).
3. Configurando os Hosts (PCs)
• PC1 (Exemplo):
o Aba "Desktop" -> "IP Configuration".
o IPv4 Address: 192.168.1.10
o Subnet Mask: 255.255.255.0
o Default Gateway: 192.168.1.1
o DNS Server: 208.67.222.10
• Outros PCs: IPs diferentes na mesma rede (ex: 192.168.1.11), mesma máscara, gateway e DNS.
4. Configurando o Roteador1 (Gateway da LAN)
Clique no Roteador1 e vá para a aba "CLI".
Router> enable
Router# configure terminal
Enter configuration commands, one per line. End with CNTL/Z.
Router(config)# hostname Roteador1
Roteador1(config)#
Roteador1(config)# ! Configurando a interface LAN (conectada ao switch dos clientes)COMENTÁRIOS
Roteador1(config)# interface GigabitEthernet0/0/0 //ou 0/0 somente...
Roteador1(config-if)# description >> Conexao com a LAN dos Clientes <<
Roteador1(config-if)# ip address 192.168.1.1 255.255.255.0
Roteador1(config-if)# ip nat inside
Roteador1(config-if)# no shutdown //ENTER
Roteador1(config-if)# exit
Roteador1(config)#
Roteador1(config)# ! Configurando a interface WAN (conectada ao Roteador_ISP)
Roteador1(config)# interface Serial0/1/0
Roteador1(config-if)# description >> Conexao com o Roteador_ISP (WAN) <<
Roteador1(config-if)# ip address 203.0.113.1 255.255.255.252
Roteador1(config-if)# ip nat outside
Roteador1(config-if)# clock rate 64000 ! Comando necessário no lado DCE da conexão serial
Roteador1(config-if)# no shutdown
Roteador1(config-if)# exit
Roteador1(config)#
Roteador1(config)# ! Configurando a lista de acesso para o NAT (permitindo a rede LAN sair)
Roteador1(config)# access-list 1 permit 192.168.1.0 0.0.0.255
Roteador1(config)#
Roteador1(config)# ! Configurando o NAT (PAT - Port Address Translation)
Roteador1(config)# ip nat inside source list 1 interface Serial0/1/0 overload
Roteador1(config)#
Roteador1(config)# ! Configurando uma rota padrão para a "internet"
Roteador1(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.2 ! IP do proximo salto (Roteador_ISP Serial0/1/0)
Roteador1(config)#
Roteador1(config)# end
Roteador1#
Roteador1# write memory
Building configuration...
[OK]
Roteador1#
5. Configurando o Roteador_ISP
Clique no Roteador_ISP e vá para a aba "CLI".
Plaintext
Router> enable
Router# configure terminal
Enter configuration commands, one per line. End with CNTL/Z.
Router(config)# hostname Roteador_ISP
Roteador_ISP(config)#
Roteador_ISP(config)# ! Configurando a interface conectada ao Roteador1
Roteador_ISP(config)# interface Serial0/1/0
Roteador_ISP(config-if)# description >> Conexao com o Roteador1 <<
Roteador_ISP(config-if)# ip address 203.0.113.2 255.255.255.252
Roteador_ISP(config-if)# no shutdown ! Clock rate esta no Roteador1 (lado DCE)
Roteador_ISP(config-if)# exit
Roteador_ISP(config)#
Roteador_ISP(config)# ! Configurando a interface conectada a rede do Servidor Web
Roteador_ISP(config)# interface GigabitEthernet0/0/0
Roteador_ISP(config-if)# description >> Conexao com a Rede do Servidor Web <<
Roteador_ISP(config-if)# ip address 208.67.222.1 255.255.255.0
Roteador_ISP(config-if)# no shutdown
Roteador_ISP(config-if)# exit
Roteador_ISP(config)#
Roteador_ISP(config)# ! Nao são necessárias rotas estáticas adicionais aqui para este cenário,
Roteador_ISP(config)# ! pois o Roteador_ISP conhece suas redes diretamente conectadas
Roteador_ISP(config)# ! (203.0.113.0/30 e 208.67.222.0/24) e o servidor terá
Roteador_ISP(config)# ! este roteador como gateway para responder ao IP NATted (203.0.113.1).
Roteador_ISP(config)#
Roteador_ISP(config)# end
Roteador_ISP#
Roteador_ISP# write memory
Building configuration...
[OK]
Roteador_ISP#
6. Configurando o Servidor Web
• Configuração IP:
o Aba "Desktop" -> "IP Configuration".
o IPv4 Address: 208.67.222.10
o Subnet Mask: 255.255.255.0
o Default Gateway: 208.67.222.1
o DNS Server: 208.67.222.10 (Ele mesmo)
• Serviço HTTP:
o Aba "Services" -> "HTTP".
o Certifique-se de que "HTTP" e "HTTPS" estejam "On".
o Edite o index.html para a página do prof. Sydnei.
• Serviço DNS:
o Aba "Services" -> "DNS".
o Certifique-se de que o serviço DNS esteja "On".
o Resource Record:
▪ Name: www.meusitefacil.com (ou o nome que preferir)
▪ Type: A Record
▪ Address: 208.67.222.10
▪ Clique em "Add".
7. Testando o Acesso
• Em qualquer PC configurado na LAN (192.168.1.x):
1. Abra o "Command Prompt" (Prompt de Comando) na aba "Desktop".
▪ Tente pingar o gateway: ping 192.168.1.1
▪ Tente pingar a interface serial do Roteador_ISP: ping 203.0.113.2
▪ Tente pingar o Servidor Web: ping 208.67.222.10
2. Abra o "Web Browser" (Navegador Web) na aba "Desktop".
▪ Na barra de endereço, digite o IP do Servidor Web: 208.67.222.10 e pressione "Go".
▪ Tente também acessar usando o nome DNS: www.meusitefacil.com.
Todos os testes devem ser bem-sucedidos, e a página web deve carregar. Se vocês adicionarem novos PCs à
LAN e os configurarem corretamente (IP na faixa 192.168.1.x, máscara 255.255.255.0, gateway 192.168.1.1 e
DNS 208.67.222.10), eles também terão acesso.
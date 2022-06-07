# SIRS-LETI-21-22-Project

Instituto Superior Técnico, Universidade de Lisboa

**Segurança Informática em Redes e Sistemas**

# Objetivos 

O objetivo do projeto consiste em aprender a instalar e configurar diversos mecanismos de segurança, aprofundando alguns conhecimentos das aulas teóricas. 
Em termos práticos, o objetivo do projeto é proteger a rede da CRYPTOTEKK, uma start-up tecnológica da área da segurança e ativos digitais (criptomoedas, NFTs). A empresa tem a sede num escritório em Oeiras e um segundo escritório em Silicon Valley, EUA. A comunicação entre os dois escritórios é feita através da Internet e portanto exposta a um risco elevado. A empresa está preocupada com a possibilidade de potenciais adversários terem acesso à sua propriedade intelectual e dados de negócio, bem como outras ameaças que possam comprometer a sua operação (vermes, ransomware, etc.). 
A rede da CRYPTOTEKK será emulada usando o software Kathará.
A entrega do projeto tem 3 componentes: 
*	um diagrama da rede da empresa; 
*	a configuração da rede; 
*	um questionário a reportar o que foi feito. O questionário é longo e detalhado, logo deve ser preenchido durante a realização do projeto.

#	Descrição da rede

A rede da CRYPTOTEKK contém os dispositivos habituais neste tipo de redes: servidores, estações de trabalho (PCs), routers, switches (também designados bridges) e dispositivos de segurança. A configuração básica da rede já está feita e é fornecida sob a forma de um laboratório Kathará, mas ainda não inclui mecanismos de segurança. 
A rede tem três partes:
1.	Escritório de Oeiras (Oeiras): contém 11 dispositivos e usa endereços da gama 1.2.0.0/20;
2.	Escritório de Silicon Valley (SValley): contém 4 dispositivos e usa endereços da gama 5.4.3.0/24;
3.	Internet: representada de forma extremamente simplificada por um único switch e um PC para efeito de testes.

O escritório de Oeiras (1) tem 3 subredes, interligadas por dois routers:
*	uma subrede que contém apenas uma firewall, em concreto, um packet filter;
*	a DMZ, onde se encontra: 
    *	um switch, 
    *	um servidor web, 
    *	um servidor de correio eletrónico e 
    *	um detetor de intrusões;
*	a subrede corporate, onde se encontra:
    *	um switch, 
    *	um servidor de ficheiros e 
    *	as estações de trabalho dos colaboradores da empresa.

Em relação ao software instalado nas estações de trabalho e nos servidores:
*	estações de trabalho: não contêm software específico, só o que já vem na imagem;
*	servidor web: executar o Apache2, contendo uma página simples de apresentação da empresa;
*	servidor de correio eletrónico: a configuração de um servidor deste tipo é complexa e está fora do âmbito da cadeira, de modo que vamos emulá-lo executando o comando nc (netcat) em modo servidor e à escuta nos portos TCP/465 (SMTPS) e TCP/993 (IMAPS).
*	servidor de ficheiros: também não vamos usar nenhum software específico, mas emulá-lo usando o comando nc em modo servidor à escuta no porto TCP/445 (SMB). 

Para testar os servidores emulados usando o comando nc usa-se o comando: `echo “mensagem” | nc <ip_destino> <porto_destino>`
O escritório de Silicon Valley (2) contém apenas um router, um switch e duas estações de trabalho. 
Por simplicidade não existe DNS, logo toda a comunicação tem de ser feita usando endereços IPv4.
A primeira tarefa do grupo de trabalho consiste em fazer um diagrama da rede usando software adequado. Deve representar todos os dispositivos, indicar o seu nome e os endereços de todas as interfaces de rede que os tenham. Inclua esse diagrama no relatório. Para o efeito deve estudar este enunciado e o ficheiro lab.conf fornecido. Este diagrama serve para o grupo usar como referência, mas tem também de ser entregue (ver prazos abaixo).

# Testes

Para testar o funcionamento da rede e dos serviços que serão concretizados é fundamental usar ferramentas de teste adequadas. Além de comandos bem conhecidos como o ping, o tcpdump e o traceroute recomenda-se o uso do netcat (nc) que permite estabelecer conexões TCP e enviar texto sobre essa conexão:
*	Para ficar à escuta numa porta na máquina de destino (servidor): `nc -l -p <porta>`
*	Para enviar pacote TCP executar no client: `echo <string_a_enviar> | nc <ip_destino> <porta_destino>`
*	A string deve aparecer no terminal do servidor.

# Mecanismos de Segurança 

A rede está configurada para o Kathará, mas não estão configurados os mecanismos de segurança. O projeto consiste em implementar 4 tipos de mecanismos de segurança: VPN, SSH, firewall (packet filter) e detetor de intrusões.

## VPN - OpenVPN

SValley está ligado a Oeiras de através de uma VPN. Configure essa VPN usando o software OpenVPN [2][14]. O router de acesso à Internet de SValley será um cliente OpenVPN e o router de acesso de Oeiras será um servidor OpenVPN. Todo o tráfego de SValley para Oeiras e vice-versa tem de ser encaminhado através de um túnel entre esses dois routers. Portanto, se alguém na Internet tentar escutar essa comunicação vai observar tráfego cifrado, logo ilegível. 
A comunicação tem de ser cifrada usando AES com chaves de 256 bits e modo CBC. As chaves do cliente OpenVPN devem ser geradas no cliente e não no servidor OpenVPN.
Teste se a comunicação está a ser efetivamente cifrada, escutando a comunicação na Internet.

## SSH - OpenSSH

A empresa tem um administrador de rede que tem de poder trabalhar remotamente no servidor web. O administrador tem uma conta nesse servidor que usa para fazer o acesso usando o protocolo SSH. O administrador viaja muito e por isso, apesar de sob o ponto de vista de segurança não ser muito boa ideia, abriu o acesso por SSH a esse servidor a partir de qualquer ponto do mundo. Por exemplo, pode fazer esse acesso a partir de um PC ligado à Internet, como o somepc no caso da nossa rede emulada. 
Configure o protocolo SSH fornecido pelo pacote OpenSSH [4][13] de modo a permitir esse acesso. A autenticação tem de ser baseada em criptografia de chave pública, não em password. 
Teste o funcionamento desses protocolos usando os comandos: 
*	ssh para fazer login remoto e 
*	scp para copiar ficheiros entre os dois computadores.
Teste se a comunicação está a ser efetivamente cifrada, escutando a comunicação na Internet.

## Firewall - iptables

A rede da CRYPTOTEKK tem três firewalls ou packet filters mas vamos configurar apenas duas delas:
*	packetfilter que filtra pacotes entre a Internet e Oeiras;
*	router3 que filtra pacotes entre a Internet e SValley.
Configure essas duas firewalls usando o netfilter / iptables [1][12][15] de modo a concretizar a seguinte política de segurança:
*	Todos os pacotes não explicitamente permitidos pelo resto da política são proibidos.
*	É permitido fazer ping e traceroute entre todas as máquinas (só para efeitos de depuração de erros; em termos de segurança não é boa ideia permitir da Internet executar esses comandos para dentro da rede da empresa).
*	Qualquer máquina da Internet e da rede da CRYPTOTEKK pode:
    *	aceder ao servidor web da empresa usando o protocolo HTTP (TCP/80) e SSH;
    *	entregar correio ao servidor de correio eletrónico da empresa (TCP/465, SMTPS);
*	Máquinas da DMZ da CRYPTOTEKK:
    *	podem responder aos pedidos que recebem (web e correio eletrónico);
    *	o servidor de correio eletrónico pode entregar mensagens a outros servidores de correio eletrónico da Internet (TCP/465, SMTPS).
*	Máquinas da rede da CRYPTOTEKK (i.e., da subrede corporate de Oeiras e da subrede de SValley):
    *	podem aceder aos servidores web (via SSH e HTTP) e correio eletrónico 
        *	da Internet (i.e. externos à empresa) e 
        *	da própria empresa;
    *	podem aceder ao servidor de ficheiros.
*	Nenhum pacote pode sair de Oeiras ou SValley com um endereço IP de origem fora da gama de endereços dessas subredes.
*	Nenhum pacote pode entrar em Oeiras ou SValley com um endereço IP de origem da gama de endereços dessas subredes (ou seja, é preciso bloquear IP Spoofing [11]).
Teste se cada firewall está a bloquear todo o tráfego que deve bloquear. Para testar se uma firewall está a negar o acesso a uma porta podem testar essa mesma porta usando o netcat.

## Detecção de Intrusões - snort

O detetor de intrusões snort [3][17] deve ser configurado de modo a detetar ataques e intrusões na rede. O sistema de deteção de intrusões (IDS) deve ser colocado na máquina designada *ids1*. O switch ao qual esse computador está ligado está configurado para funcionar como hub, de modo que o IDS receba todo o tráfego que passa por esse switch.

O primeiro passo é a instalação do software snort em si, pois este não está disponível na imagem quagga usada em todas as imagens do ficheiro lab.conf. Para o efeito é preciso criar uma nova imagem com o snort seguindo as instruções fornecidas no slide “Installing software inside a VM” [15].

No segundo passo, o snort tem de ser configurado para alertar para um conjunto de ataques. O snort contém um conjunto enorme de regras que é atualizado periodicamente. Estas regras e a configuração do snort estão geralmente disponíveis na pasta /etc/snort. No nosso caso interessam apenas dois ficheiros: o /etc/snort/snort.conf (onde está a configuração do snort) e o /etc/snort/rules/local.rules (onde devem colocar as vossas regras). Nota: no ficheiro /etc/snort/snort.conf é preciso dar à opção *config checksum_mode" o valor *none*.

Modifique as regras disponíveis de modo a gerar os seguintes alarmes (ou seja, as seguintes mensagens na consola ou ficheiro de log):

*	Alarme “1: Perigo telnet!”. Um atacante que obteve acesso ao servidor web, executa o comando telnet para outro computador do escritório de Oeiras. O comando telnet é inseguro, logo deve ser detetado. O que caracteriza o uso do telnet é simplesmente usar como porto de destino TCP/23.
*	Alarme “2: SSH forcing!”. Um ataque comum contra um computador que tenha acesso remoto SSH, ou seja, que esteja à escuta no porto o TCP/22, consiste em tentar descobrir a password de uma conta, tentando usar nomes de contas comuns e mudando a password em cada tentativa. Deve ser gerado um alarme sempre que um computador faça mais do que 10 tentativas falhadas de acesso SSH por minuto.
*	Alarme “3: Perigo SIRS!”. O ataque é hipotético e consiste em fazer uma ligação ao porto TCP/9000 enviando: 
    *	no 1º byte, o número do grupo de trabalho;
    *	no 2º byte, o número do grupo multiplicado por 3. 
Teste a deteção dos ataques acima começando por criar comandos ou scripts que os executem.

# Entrega

O projeto é entregue no Fénix. Prazos e forma de entrega (sendo XXX o nº do grupo):
*	Até dia 27 de Maio às 17 horas: entregar um diagrama da rede sob a forma de um ficheiro chamado diagramaXXX.pdf no Fénix;
*	Até dia 20 de Junho às 17 horas: entregar todos os ficheiros do laboratório sob a forma de um ficheiro katharaXXX.tgz no Fénix. Incluir o laboratório Kathará com todos os ficheiros .startup e as subpastas. Se alguns comandos não puderem ser automatizados dessa forma, explicar e indicá-los no laboratório. Não é preciso entregar a imagem que contém o snort;
*	Até dia 20 de Junho às 17 horas: entregar o relatório/questionário preenchido sob a forma de um ficheiro chamado relatorioXXX.pdf no Fénix. Esse relatório tem obrigatoriamente criado com base no ficheiro relatorio-questionario.docx fornecido.

# Bibliografia

* [1] iptables Tutorial 1.2.2 https://www.frozentux.net/iptables-tutorial/iptables-tutorial.html 
* [2] OpenVPN Howto, https://openvpn.net/index.php/open-source/documentation/howto.html 
* [3] Snort Users Manual 2.9.12 – The Snort Project, 2018 
* [4] OpenSSH, http://www.openssh.org/
* [5] Kathará web page: https://www.kathara.org/  
* [6] Kathará Wiki: https://github.com/KatharaFramework/Kathara/wiki 
* [7] Kathará Labs: https://github.com/KatharaFramework/Kathara-Labs/wiki 
* [8] Kathará man pages: https://www.kathara.org/man-pages/kathara.1.html 
* [9] Guia de Configuração do Kathará: https://github.com/tecnico-sec/Kathara-Setup 
* [10] Guia de Laboratório - Network Routing: https://github.com/tecnico-sec/Kathara-Route 
* [11] Guia de Laboratório - Network Vulnerabilities: https://github.com/tecnico-sec/Kathara-NetVulns 
* [12] Guia de Laboratório - Web Server and Firewall: https://github.com/tecnico-sec/Kathara-WebServer-Firewall 
* [13] Guia de Laboratório - Secure Shell: https://github.com/tecnico-sec/Kathara-SSH 
* [14] Guia de Laboratório - Virtual Private Network: https://github.com/tecnico-sec/Kathara-VPN 
* [15] Miguel Correia. “Kathará”. Slides de Segurança Informática em Redes e Sistemas, LETI, Instituto Superior Técnico, Abril de 2022
* [16] Miguel Correia. "iptables: a brief introduction". Slides de Segurança Informática em Redes e Sistemas, LETI, Instituto Superior Técnico, Maio de 2022
* [17] Miguel Correia. "snort: a brief introduction". Slides de Segurança Informática em Redes e Sistemas, LETI, Instituto Superior Técnico, Maio de 2022



---

Caso venham a surgir correções ou clarificações neste documento, podem ser consultadas no histórico (_History_).

**Bom trabalho!**

[Os docentes de SIRS](mailto:leti-sirs@disciplinas.tecnico.ulisboa.pt)





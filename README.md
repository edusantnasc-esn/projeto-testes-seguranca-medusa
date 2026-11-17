Projeto de Segurança da Informação — Testes de Autenticação com Kali Linux e Medusa
1. Apresentação
Este projeto tem como objetivo realizar testes controlados de segurança em um ambiente virtualizado, utilizando Kali Linux e Metasploitable 2.
Foram utilizadas ferramentas de reconhecimento, enumeração e análise de autenticação para demonstrar vulnerabilidades relacionadas ao uso de credenciais fracas e configurações inadequadas de serviços de rede.
O laboratório foi desenvolvido para fins educacionais e para demonstrar conhecimentos práticos em segurança da informação, infraestrutura e redes.
2. Objetivos
    • Identificar serviços de rede disponíveis no sistema-alvo.
    • Realizar reconhecimento de portas e versões de serviços.
    • Identificar usuários disponíveis por meio de enumeração.
    • Executar testes controlados de força bruta em FTP.
    • Realizar password spraying em SMB.
    • Analisar mecanismos de autenticação em aplicações web.
    • Interpretar os resultados e propor medidas de proteção.
3. Ambiente de laboratório
3.1 Equipamentos e ferramentas
Componente	Utilização
Oracle VirtualBox	Virtualização
Kali Linux	Máquina de testes
Metasploitable 2	Máquina-alvo vulnerável
Nmap	Reconhecimento de rede
Medusa v2.3	Testes de autenticação
enum4linux	Enumeração SMB
Netcat	Verificação de serviços
Cliente FTP	Validação de autenticação
3.2 Endereçamento IP
Máquina	Endereço IP
Kali Linux	192.168.56.103
Metasploitable 2	192.168.56.102
3.3 Configuração de rede
As máquinas virtuais foram conectadas por meio de uma rede Host-Only no VirtualBox.
Essa configuração permite a comunicação entre as máquinas virtuais e mantém os testes isolados da rede externa.
A conectividade entre as máquinas foi validada antes da execução dos testes.
4. Reconhecimento do sistema-alvo
Foi utilizada a ferramenta Nmap para identificar portas abertas e versões dos serviços.
Comando:
nmap -sV 192.168.56.102
Serviços identificados
Porta	Serviço	Tecnologia identificada
21	FTP	vsftpd 2.3.4
22	SSH	OpenSSH
23	Telnet	Telnet
25	SMTP	Postfix
53	DNS	BIND
80	HTTP	Apache
139	SMB	Samba
445	SMB	Samba
3306	Banco de dados	MySQL
5432	Banco de dados	PostgreSQL
5900	VNC	VNC
8180	HTTP	Apache Tomcat
Observação: o comando utilizado realiza a identificação de serviços nas portas TCP mais comuns consideradas pelo Nmap em sua varredura padrão, não em todas as portas existentes.
Verificação de conectividade FTP
Comando:
nc -nv 192.168.56.102 21
Resposta observada:
220 (vsFTPd 2.3.4)
A resposta confirmou que o serviço FTP estava disponível.
5. Preparação das listas de teste
Foi criada a pasta de trabalho:
mkdir -p ~/projeto-medusa
cd ~/projeto-medusa
Foram preparados arquivos de usuários e senhas para os testes.
5.1 Listas FTP
Arquivo usuarios.txt:
msfadmin
user
test
Arquivo senhas.txt:
senha123
123456
msfadmin
5.2 Listas SMB
Arquivo usuarios_smb.txt:
msfadmin
user
service
Arquivo senhas_smb.txt:
123456
password
msfadmin
As listas foram utilizadas exclusivamente no ambiente de laboratório.
6. Teste de autenticação FTP
6.1 Objetivo
Verificar se o serviço FTP aceitava credenciais fracas ou previsíveis utilizando o módulo FTP do Medusa.
6.2 Execução
Comando:
medusa -h 192.168.56.102 -U usuarios.txt -P senhas.txt -M ftp -t 1
Parâmetros utilizados:
    • -h: endereço IP do alvo.
    • -U: arquivo contendo os usuários.
    • -P: arquivo contendo as senhas.
    • -M ftp: módulo de autenticação FTP.
    • -t 1: uma tarefa simultânea.
6.3 Resultado
O Medusa identificou uma combinação válida para o usuário msfadmin.
Resultado observado:
ACCOUNT FOUND: [ftp]
Host: 192.168.56.102
User: msfadmin
[SUCCESS]
6.4 Validação manual
Para confirmar o resultado, foi realizada uma conexão manual:
ftp 192.168.56.102
Após informar as credenciais identificadas, o servidor retornou:
230 Login successful.
6.5 Conclusão do teste FTP
A autenticação foi confirmada manualmente.
O teste demonstrou que o serviço aceitava uma credencial previsível, permitindo acesso ao FTP.
7. Enumeração de usuários SMB
7.1 Objetivo
Identificar informações sobre contas disponíveis no serviço SMB do sistema-alvo.
7.2 Identificação das portas
Comando:
nmap -p 139,445 -sV 192.168.56.102
Resultado:
139/tcp open netbios-ssn Samba
445/tcp open microsoft-ds Samba
7.3 Enumeração
Foi utilizada a ferramenta enum4linux:
enum4linux -U 192.168.56.102
7.4 Resultado
Foram identificados aproximadamente 34 nomes de contas no ambiente.
Entre os usuários encontrados:
    • games
    • nobody
    • bind
    • proxy
    • syslog
    • user
    • www-data
    • root
    • postgres
    • mysql
    • backup
    • msfadmin
    • service
    • ftp
    • tomcat55
A enumeração também demonstrou que o serviço permitia obter informações por meio de uma sessão sem autenticação.
Observação: os nomes encontrados não representam necessariamente contas habilitadas para login. Algumas podem ser contas de serviço ou estar desativadas.
8. Teste de Password Spraying SMB
8.1 Objetivo
Realizar testes controlados de autenticação SMB utilizando uma senha por rodada contra uma lista reduzida de usuários.
8.2 Primeira rodada
Senha testada:
123456
Comando:
medusa -h 192.168.56.102 -U usuarios_smb.txt -p 123456 -M smbnt -t 1
Resultado:
Nenhuma credencial válida foi identificada.
8.3 Segunda rodada
Senha testada:
password
Comando:
medusa -h 192.168.56.102 -U usuarios_smb.txt -p password -M smbnt -t 1
Resultado:
Nenhuma credencial válida foi identificada.
8.4 Terceira rodada
Senha testada:
msfadmin
Comando:
medusa -h 192.168.56.102 -U usuarios_smb.txt -p msfadmin -M smbnt -t 1
Resultado observado:
ACCOUNT FOUND: [smbnt]
Host: 192.168.56.102
User: msfadmin
Password: msfadmin
SUCCESS (ADMIN$ - Access Allowed)
8.5 Interpretação
O resultado confirmou:
    • Autenticação válida para o usuário msfadmin.
    • Aceitação da senha testada.
    • Acesso permitido ao compartilhamento administrativo ADMIN$.
A mensagem ADMIN$ - Access Allowed indica que o Medusa conseguiu acessar o compartilhamento administrativo após a autenticação.
8.6 Resumo dos resultados SMB
Usuário	Senha testada	Resultado
msfadmin	123456	Não encontrada
user	123456	Não encontrada
service	123456	Não encontrada
msfadmin	password	Não encontrada
user	password	Não encontrada
service	password	Não encontrada
msfadmin	msfadmin	Acesso administrativo confirmado
user	msfadmin	Não encontrada
service	msfadmin	Não encontrada
9. Análise do DVWA — aplicação web
9.1 Objetivo
Analisar o módulo Brute Force da aplicação DVWA, disponível no sistema Metasploitable 2.
Endereço:
http://192.168.56.102/dvwa/
Foi realizado acesso à aplicação e análise inicial do formulário de autenticação.
9.2 Verificação de acesso
Foi utilizado o comando:
curl -i "http://192.168.56.102/dvwa/vulnerabilities/brute/"
Resposta observada:
HTTP/1.1 302 Found
Location: ../../login.php
A resposta demonstrou que a requisição realizada pelo curl, sem a sessão autenticada do navegador, foi redirecionada para a página de login.
9.3 Situação
A análise inicial do formulário foi realizada, mas a automação com Medusa ainda não foi concluída.
O módulo web-form foi consultado com:
medusa -M web-form -q
Foram identificadas opções para configurar método HTTP, campos do formulário, mensagem de falha e cabeçalhos personalizados.
Essa etapa permanece pendente de validação.
10. Resumo geral dos testes
Etapa	Ferramenta	Resultado
Reconhecimento de serviços	Nmap	Concluído
Verificação FTP	Netcat	Concluído
Autenticação FTP	Medusa	Credencial encontrada
Validação FTP	Cliente FTP	Login confirmado
Enumeração SMB	enum4linux	Aproximadamente 34 contas identificadas
Password spraying SMB	Medusa	Credencial encontrada
Acesso administrativo SMB	Medusa	ADMIN$ permitido
Análise inicial DVWA	Navegador e curl	Parcial
Automação DVWA	Medusa	Pendente
11. Recomendações de segurança
Com base nos resultados obtidos, são recomendadas as seguintes medidas:
FTP
    • Desabilitar FTP quando não houver necessidade operacional.
    • Priorizar SFTP ou FTPS.
    • Utilizar senhas fortes e exclusivas.
    • Restringir o acesso por firewall.
    • Monitorar tentativas de autenticação.
SMB
    • Restringir sessões anônimas.
    • Limitar o acesso às portas 139 e 445.
    • Desabilitar contas desnecessárias.
    • Revisar permissões de compartilhamentos administrativos.
    • Aplicar o princípio do menor privilégio.
    • Monitorar falhas de autenticação e acessos administrativos.
    • Utilizar versões modernas e configurações seguras do SMB.
Aplicações web
    • Implementar proteção contra tentativas automatizadas.
    • Utilizar tokens CSRF corretamente.
    • Aplicar limitação de tentativas de autenticação.
    • Monitorar tentativas repetidas de login.
    • Utilizar autenticação multifator quando disponível.
12. Conclusão
O laboratório permitiu praticar atividades de reconhecimento, enumeração de usuários e testes controlados de autenticação.
Os testes FTP e SMB demonstraram na prática os riscos associados ao uso de credenciais fracas ou padrão.
No serviço FTP, foi identificada uma credencial válida e realizada confirmação manual do acesso.
No serviço SMB, foi possível enumerar nomes de contas e identificar uma credencial que permitiu acesso ao compartilhamento administrativo ADMIN$.
A análise inicial do DVWA também permitiu observar o comportamento de uma aplicação web que exige autenticação por sessão.
O projeto reforçou a importância de configurações seguras, políticas de senhas, controle de acesso e monitoramento dos serviços de rede.
13. Escopo e responsabilidade
Todos os testes foram realizados exclusivamente no ambiente virtualizado de laboratório, utilizando Kali Linux e Metasploitable 2.
O projeto possui finalidade educacional e demonstração de conhecimentos práticos em segurança da informação.
14. Estrutura do projeto
projeto-medusa/
├── README.md
├── docs/
├── evidencias/
├── usuarios.txt
├── senhas.txt
├── usuarios_smb.txt
└── senhas_smb.txt
15. Competências praticadas
    • Linux e terminal.
    • Redes TCP/IP.
    • Reconhecimento de portas e serviços.
    • Nmap.
    • Enumeração SMB.
    • enum4linux.
    • Medusa.
    • Autenticação FTP.
    • Password spraying.
    • Análise de segurança de aplicações web.
    • Interpretação de resultados.
    • Documentação técnica.

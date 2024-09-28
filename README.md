# PowerPot

## Introdução

Esta documentação descreve o **PowerPot**, uma ferramenta multitarefas, um mini servidor web desenvolvido para testes de segurança e simulações de ataques. A proposta do PowerPot é fornecer um ambiente controlado para coletar dados de payloads, atuar como um honeypot e servir como uma plataforma para a proliferação de malwares em ambientes de teste para equipes de Red Team.

## Proposta do Projeto

O PowerPot tem os seguintes objetivos principais:

1. **Ambiente de Teste**: Facilitar a criação de um ambiente onde profissionais de segurança podem interagir com diferentes tipos de requisições HTTP, permitindo a observação de como atacantes podem explorar falhas em um servidor web.

2. **Honeypot**: Servir como um honeypot que atrai potenciais atacantes, possibilitando o monitoramento e registro de suas ações. Isso ajuda as equipes de segurança a compreender táticas e técnicas utilizadas por atacantes.

3. **Coleta de Dados**: Capturar informações detalhadas sobre cada interação, incluindo métodos de requisição, payloads enviados e dados do cliente, que podem ser analisados para entender comportamentos maliciosos.

### Funcionalidade de Honeypot

Um honeypot é um sistema intencionalmente configurado para ser vulnerável, com o objetivo de coletar informações sobre tentativas de ataques. O PowerPot é projetado para registrar:

- **Endereço IP de Origem**: Captura o endereço IP do cliente que faz a requisição, permitindo a análise da geolocalização dos atacantes e suas fontes.

- **Método HTTP Utilizado**: Registra o método da requisição (GET, POST, PUT, DELETE), permitindo uma análise de como os atacantes interagem com o servidor.

- **URL Requisitada**: Armazena a URL acessada, incluindo qualquer parâmetro de consulta, permitindo que a equipe de segurança compreenda quais recursos estão sendo alvo.

- **User-Agent**: Identifica o agente do usuário que fez a requisição, ajudando a discernir se o acesso foi feito por um script automatizado ou um navegador.

- **Tamanho do Payload**: Captura o tamanho do payload enviado, se houver, ajudando a entender o volume de dados que os atacantes tentam enviar ao servidor.

### Log de Payloads

Todos os detalhes de cada requisição são registrados em um arquivo de log. As informações logadas incluem:

- **Data e Hora**: Um timestamp é gerado para cada requisição, facilitando a auditoria e a análise de eventos.

- **Detalhes do Cliente**: Informações como IP, nome do computador e sistema operacional do cliente que fez a requisição são registradas para análise posterior.

- **Conteúdo do Payload**: O conteúdo enviado pelo cliente é registrado quando aplicável, o que é crucial para entender como os malwares tentam interagir com o servidor.

## Operação no Kernel do Windows

O PowerPot é implementado em PowerShell, permitindo que opere dentro do kernel do Windows. A execução no nível do sistema oferece vantagens significativas, incluindo:

- **Desempenho**: A interação próxima com o sistema operacional garante que o servidor funcione de maneira eficiente, suportando múltiplas conexões e requisições simultâneas.

- **Capacidade de Monitoramento**: A implementação em PowerShell permite que o servidor monitore e registre as atividades do sistema, coletando dados em tempo real que são valiosos para a análise de segurança.

### Função de C2 (Command and Control)

O servidor pode ser configurado para simular uma infraestrutura de C2, onde malwares podem se comunicar para receber comandos ou exfiltrar dados. Isso permite que as equipes de Red Team:

- **Testem a Comunicação de Malwares**: Observem como os malwares tentam se conectar ao servidor e quais dados tentam enviar ou receber.

- **Analisem o Comportamento do Malware**: Capturam como os malwares interagem com o servidor, permitindo um entendimento mais profundo de suas operações.

- **Compreendam a Configuração de C2**: Ajudam as equipes de segurança a implementar medidas defensivas contra sistemas de C2.

## Estrutura Técnica do Servidor

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

## Instruções de Uso

Para executar o script `powerpot.ps1`, siga estas etapas:

1. **Abra o PowerShell** com permissões administrativas.
2. **Navegue até o diretório** onde o script está localizado. Por exemplo:
   ```powershell
   cd C:\caminho\para\o\script
3. **No Powershell 5.1+ **
   ```powershell
   Powerpot.ps1 -DirectoryPath "C:\fake\website\path" -LogFilePath "C:\log\path\honeypot.log" -Port 8080
4. Caso não selecionado, os diretórios padrão serão `Web Site C:\inetpub\wwwroot\`, `Path para os log C:\inetpub\logs\LogFiles`, `Porta de comunicação :8080`

## Conclusão

O PowerPot oferece uma ferramenta robusta para testes de segurança e análise de malwares. Suas funcionalidades como honeypot e a capacidade de operar no kernel do Windows fazem dele um recurso valioso para equipes de segurança que buscam entender melhor as ameaças e fortalecer suas defesas.

### Nota

**Este servidor deve ser utilizado apenas em ambientes de teste controlados. O uso indevido pode violar leis e políticas de segurança.**

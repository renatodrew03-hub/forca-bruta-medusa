# Recomendações de Mitigação

## FTP

- Desativar FTP em texto claro; substituir por **SFTP/FTPS**.
- Implementar **bloqueio de conta** após N tentativas falhas (ex: fail2ban monitorando o log do vsftpd/proftpd).
- Desativar contas padrão/anônimas (ex: `msfadmin`, contas de fábrica).
- Restringir o acesso ao serviço por firewall/ACL a IPs conhecidos.

## Formulário Web (DVWA / aplicações em geral)

- **Rate limiting** por IP e por conta no endpoint de login.
- **Bloqueio progressivo** (ex: captcha após 3 tentativas, bloqueio temporário após 5).
- **Autenticação multifator (MFA)**.
- Mensagens de erro genéricas (não revelar se o usuário existe ou não).
- **WAF** (Web Application Firewall) para detectar padrões de automação (User-Agent, frequência de requisições).
- Monitorar logs de autenticação (ex: enviar para Wazuh/SIEM e criar regra de alerta para múltiplas falhas de login na mesma origem).

## SMB

- Desativar **SMBv1** (protocolo legado e vulnerável).
- Habilitar **SMB signing**.
- Política de **lockout de conta** no domínio/servidor (ex: 5 tentativas → bloqueio de 15 min).
- Segmentação de rede: SMB não deve estar acessível a qualquer host da rede, apenas aos que realmente precisam.
- Monitorar tentativas de enumeração (ex: `enum4linux`, `smbclient -L`) e autenticações SMB malsucedidas via SIEM.

## Mitigações gerais (aplicáveis a todos os vetores)

- **Senhas fortes e únicas**, sem reuso entre contas/serviços.
- **Monitoramento centralizado** (Wazuh/SIEM) com regras de correlação para múltiplas falhas de autenticação em curto intervalo de tempo — base para um alerta de força bruta/spraying.
- **Honeypots/honeytokens** para detectar reconhecimento e tentativas de acesso não autorizado precocemente.
- Revisão periódica de contas ativas, removendo contas de teste/padrão (como as presentes por padrão no Metasploitable2).
- Testes de força bruta/spraying **autorizados e documentados** como parte de um programa de testes de segurança (pentest/red team) recorrente.

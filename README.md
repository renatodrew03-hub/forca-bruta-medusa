# Simulação de Ataques de Força Bruta com Medusa (Kali Linux + Metasploitable2 + DVWA)

## Objetivo

Implementar, documentar e compartilhar um projeto prático de simulação de ataques de força bruta usando **Kali Linux** e a ferramenta **Medusa**, contra ambientes intencionalmente vulneráveis (**Metasploitable 2** e **DVWA**), exercitando tanto a execução dos ataques quanto as medidas de prevenção/mitigação.

> ⚠️ Este laboratório foi executado **inteiramente em rede isolada (host-only)**, em máquinas virtuais próprias, sem qualquer exposição à internet ou a sistemas de terceiros. Técnicas de força bruta só devem ser testadas em ambientes de laboratório ou com autorização explícita.

## Topologia do laboratório

| Máquina | Sistema | Papel | Rede |
|---|---|---|---|
| Kali Linux | Kali Linux (rolling) | Atacante | Host-only (VirtualBox) |
| Metasploitable 2 | Ubuntu 8.04 (vulnerável) | Alvo (FTP, SMB, DVWA via Apache) | Host-only (VirtualBox) |

- Adaptador de rede: **Host-only Adapter** (ex: `vboxnet1`, faixa `192.168.56.0/24`)
- Nenhuma das duas VMs possui rota para a internet durante os testes
- IP do Kali: `192.168.56.10` *(ajustar para o IP real do seu lab)*
- IP do Metasploitable2: `192.168.56.20` *(ajustar para o IP real do seu lab)*

## Ferramentas utilizadas

- **Nmap** — reconhecimento de portas e serviços
- **Medusa** — força bruta (FTP e SMB)
- **Hydra** — automação de formulário web do DVWA (módulo HTTP usado como alternativa ao Medusa, ver `comandos.md`)
- **enum4linux / smbclient** — enumeração de usuários SMB
- **Wazuh** *(opcional)* — monitoramento e detecção das tentativas, já existente no laboratório pessoal do autor

## Estrutura do repositório

```
forca-bruta-medusa/
├── README.md          <- este arquivo
├── comandos.md         <- comandos executados, por cenário, com saídas resumidas
├── mitigacoes.md        <- recomendações de prevenção por vetor de ataque
└── wordlists/
    ├── users.txt
    └── passwords.txt
```

## Cenários cobertos

1. **Força bruta em FTP** (porta 21) — Medusa
2. **Automação de tentativas no formulário de login do DVWA** (HTTP) — Medusa/Hydra
3. **Password spraying em SMB com enumeração prévia de usuários** (portas 139/445) — enum4linux + Medusa

## Principais aprendizados

- A eficácia de um ataque de força bruta depende muito mais da **qualidade da wordlist** e da ausência de controles (lockout, rate limiting) do que da ferramenta em si.
- **Enumeração antes de ataque**: enumerar usuários válidos (SMB) reduz drasticamente o espaço de tentativas necessário, tornando um *password spraying* bem mais eficiente que um brute force cego.
- Serviços legados (FTP em texto claro, SMBv1) são alvos triviais; a mitigação real passa por desativar serviços desnecessários, não só por "senhas fortes".
- Ver `mitigacoes.md` para as recomendações completas.

## Autor

Projeto desenvolvido como parte dos estudos práticos em Segurança da Informação (SOC/NOC), como laboratório pessoal.

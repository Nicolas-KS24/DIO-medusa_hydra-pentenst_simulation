# 🛡️ Cybersecurity Lab: Auditoria de Senhas com Medusa e Hydra

Este repositório contém a documentação prática de um laboratório de testes de intrusão focado em ataques de força bruta, desenvolvido como o desafio de projeto da **Digital Innovation One (DIO)**. O objetivo principal foi simular cenários reais de ataque em um ambiente controlado e seguro para entender o comportamento de protocolos comuns e propor medidas eficazes de mitigação.

---

## 💻 Cenário de Laboratório (Lab Setup)

O ambiente foi totalmente isolado e configurado utilizando a seguinte topologia de rede interna:

* **Hypervisor:** VirtualBox 7.x
* **Rede:** Placa de rede exclusiva de hospedeiro (*Host-Only*) - Garantindo isolamento total da internet.
* **Máquina Atacante:** Kali Linux 2026.1 (IP: `192.168.56.102`)
* **Máquina Alvo:** Metasploitable 2 (IP: `192.168.56.101`)

---

## ⚔️ Ataques Simulados e Resultados

As wordlists de teste utilizadas no laboratório continham credenciais comuns encontradas nas pastas `/wordlists`.

### 1. Força Bruta em FTP (Porta 21)
O protocolo FTP foi auditado utilizando o **Medusa** para cruzar a lista de usuários contra a lista de senhas comuns de forma paralela.

**Comando utilizado:**
```bash
medusa -h 192.168.56.101 -U usuarios.txt -P senhas.txt -M ftp
```

### Resultado:
O Medusa identificou credenciais válidas ativas no sistema `msfadmin`/`msfadmin`.

---

### 2. Password Spraying em SMB (Porta 445)

Simulação de um ataque de Password Spraying, que consiste em testar uma única senha comum contra múltiplos usuários na rede para evitar bloqueios de conta por políticas locais.

**Comando utilizado:**

```bash
medusa -h 192.168.56.101 -U usuarios.txt -p "msfadmin" -M smbnt
```

### Resultado:

O Medusa confirmou que o usuário `msfadmin` estava ativo no compartilhamento Samba com privilégios de acesso.

---

### 3. Força Bruta em Formulário Web - DVWA (Porta 80)

**Nota de Engenharia (Desafio e Solução):**

Durante os testes iniciais com o Medusa contra o formulário do DVWA, as requisições geraram redirecionamentos temporários **(HTTP 302)**, impossibilitando a leitura do corpo HTML estável (`200 OK`) pela ferramenta.

Como decisão técnica de arquitetura, foi adotado o **Hydra**, ferramenta padrão de mercado altamente robusta para seguir e interpretar redirecionamentos e cookies de sessão.

**Comando utilizado:**

```bash
hydra -l admin -P senhas.txt 192.168.56.101 http-get-form '/dvwa/vulnerabilities/brute/index.php:username=^USER^&password=^PASS^&Login=Login:H=Cookie\: security=low; PHPSESSID=bf80c786f9947bf7e3da8f8a32dc3a47:F=Username and/or password incorrect.'
```

### Resultado:

O Hydra injetou com sucesso o cookie de sessão do navegador do Kali, contornou o redirecionamento e identificou as senhas vulneráveis ativas para o usuário `admin`.

---

**🛡️ Medidas de Defesa e Mitigação Recomendadas**

Para cada protocolo explorado, as seguintes medidas defensivas são altamente recomendadas para proteção em ambientes de produção:
| **Serviço** | **Vulnerabilidade** | **Medida de Mitigação** |
|---|---|---|
| **FTP** | Transmissão de credenciais em texto claro e sucetibilidade a brute force | Migrar o tráfego para **SFTP (SSH)**. Implementar o **Fail2Ban** para bloquear IPs de origem após 3 a 5 tentativas de login falhas. |
| **SMB** | Enumeração de contas ativas na rede interna | Desabilitar suporte ao SMBv1 (manter apenas v2/v3). Aplicar GPOs de bloqueio temporário de conta de usuário (Account Lockout Threshold) após tentativas inválidas repetitivas. |
| **Web (DVWA)** | Ausência de limites de taxa de requisições por IP (Rate Limiting). | Implementar mecanismos de **reCAPTCHA** no fluxo de autenticação. Adicionar atrasos progressivos (delays) de rede entre falhas consecutivas e usar um WAF (Web Application Firewall).

---

### 🎓 Aprendizados
Este laboratório proporcionou uma sólida compreensão prática de como ferramentas de brute force interpretam diferentes protocolos de rede. Além disso, evidenciou a importância de saber diagnosticar respostas HTTP (como o código 302) e a necessidade de flexibilizar o arsenal de ferramentas (migrando do Medusa para o Hydra no cenário web) para atingir os objetivos da auditoria.

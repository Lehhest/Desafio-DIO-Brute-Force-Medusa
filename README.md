# 🛡️ Simulação de Ataque Brute Force com Medusa no Kali Linux

Projeto prático desenvolvido para o desafio da **Digital Innovation One (DIO)**, cobrindo varredura de portas, ataque de força bruta via protocolo SSH com a ferramenta Medusa, resolução de incompatibilidade de cifras legadas e validação de acesso remoto em ambiente controlado.

---

## 🛠️ Tecnologias e Ferramentas Utilizadas

- **Oracle VirtualBox**: Gerenciamento do ambiente de máquinas virtuais[cite: 5].
- **Kali Linux**: Sistema atacante (`192.168.1.10`)[cite: 1, 5].
- **Metasploitable 2**: Sistema alvo vulnerável (`192.168.1.20`)[cite: 1, 4].
- **Nmap**: Mapeamento de portas e identificação de serviços.
- **Medusa v2.3**: Auditoria de credenciais via Brute Force.
- **OpenSSH Client**: Acesso remoto com ajuste de algoritmos legados (`ssh-rsa`, `3des-cbc`).

---

## 📐 Topologia do Ambiente

As máquinas virtuais foram configuradas em rede interna (`intnet` / Host-Only) para garantir isolamento e segurança[cite: 5]:

| Máquina Virtual | Função | Endereço IP |
| :--- | :--- | :--- |
| **Kali Linux** | Atacante | `192.168.1.10` |
| **Metasploitable 2** | Alvo Vulnerável | `192.168.1.20`[cite: 1, 4] |

<img width="425" height="509" alt="1" src="https://github.com/user-attachments/assets/87e0a599-2f7a-432a-ae9e-68bdb456f876" />

![Configuração das Máquinas no VirtualBox](images/1.png)
*Figura 1: Ambiente virtualizado com Kali Linux e Metasploitable 2 ativos no VirtualBox[cite: 5].*

---

## 🚀 Passo a Passo da Execução

### 1. Mapeamento de Serviços com Nmap

Foi realizada uma varredura rápida sem resolução DNS (`-F -n`) para identificar serviços abertos no alvo `192.168.1.20`[cite: 4]:

```bash
sudo nmap -F -n 192.168.1.0/24<img width="425" height="509" alt="1" src="https://github.com/user-attachments/assets/8f654c6a-b60a-41c9-976f-702362820a81" />
```

<img width="417" height="175" alt="2" src="https://github.com/user-attachments/assets/7eed8e9e-5055-4b90-96c5-fb8fb2f0500b" />

### 2. Ataque de Força Bruta no SSH com Medusa

Com a porta 22 identificada, utilizou-se o Medusa para realizar um ataque de força bruta contra o usuário msfadmin utilizando uma wordlist customizada (wordlist.txt):

```bash
medusa -h 192.168.1.20 -u msfadmin -P /home/leh/wordlistdio/wordlist.txt -M ssh -m CODEC:SSH-CIPHERS=+3des-cbc
```

<img width="413" height="173" alt="3" src="https://github.com/user-attachments/assets/7d75965e-4c0c-457e-b83f-afbc8c93be38" />

Medusa testa as senhas sequencialmente e obtém sucesso (SUCCESS) na credencial msfadmin:msfadmin

### 3. Validação do Acesso SSH e Resolução de Incompatibilidade de Cifras

Ao tentar conectar diretamente via SSH padrão, o cliente do Kali Linux recusou a conexão por ausência de algoritmos de chave host legados

```bash
ssh msfadmin@192.168.1.20
# Erro: Unable to negotiate with 192.168.1.20 port 22: no matching host key type found. Their offer: ssh-rsa,ssh-dss
```
Para contornar o problema e validar as credenciais obtidas, os algoritmos e cifras legados foram habilitados diretamente na linha de comando do ssh

```bash
ssh -o HostKeyAlgorithms=+ssh-rsa -o KexAlgorithms=+diffie-hellman-group1-sha1 -o Ciphers=+3des-cbc msfadmin@192.168.1.20
```
<img width="353" height="49" alt="4" src="https://github.com/user-attachments/assets/dca24c10-4dc8-403a-9374-3f273fbc8f2b" />


Login realizado com sucesso no Metasploitable 2 através das credenciais capturadas


<img width="416" height="131" alt="5" src="https://github.com/user-attachments/assets/5ff222e6-0c19-460a-95f3-2b0e7936fcb3" />


##🛡️ Medidas de Mitigação e Recomendações
- Políticas de Senha Forte: Utilizar senhas complexas com mais de 12 caracteres, incluindo números, símbolos e letras maiúsculas/minúsculas.

- Bloqueio de Tentativas (Fail2ban): Implementar ferramentas para bloquear temporariamente ou banir IPs após múltiplas tentativas consecutivas de acesso.

- Desativação de Algoritmos Legados: Atualizar servidores SSH e desativar algoritmos fracos ou descontinuados como ssh-rsa, ssh-dss e 3des-cbc.

- Autenticação por Chaves Públicas: Desabilitar a autenticação por senha no arquivo /etc/ssh/sshd_config (PasswordAuthentication no) e utilizar pares de chaves SSH criptográficas.

- Alteração de Porta Padrão: Mudar a porta padrão do serviço SSH (porta 22) para reduzir tentativas automatizadas de varredura.

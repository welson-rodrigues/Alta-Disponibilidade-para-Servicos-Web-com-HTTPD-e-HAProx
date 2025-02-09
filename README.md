# HAProxy com Servidores Apache HTTPD

## 1. Objetivo
Garantir alta disponibilidade para serviços web utilizando o balanceador de carga HAProxy e servidores Apache HTTPD. O sistema deve continuar funcionando mesmo em caso de falha de um ou mais servidores backend.

## 2. Arquitetura do Sistema

### 2.1 Componentes Utilizados
- **HAProxy**: Balanceador de carga.
- **Servidores Apache HTTPD**: Servidores Web Backend.

### 2.2 Configuração de Rede
1. Instalamos e configuramos os servidores Apache e o HAProxy.
2. O cliente acessa o IP do HAProxy.
3. O HAProxy distribui as requisições entre os servidores backend (Apache).
4. Caso um servidor falhe, o HAProxy redireciona as requisições para os servidores restantes.

### 2.3 Definição de IPs Estáticos
Inicialmente, os IPs dos servidores mudavam ao reiniciar a máquina virtual. Para resolver isso, configuramos os seguintes IPs estáticos:
- Servidor 1: `10.49.10.10/16`
- Servidor 2: `10.49.10.11/16`
- Servidor 3: `10.49.10.12/16`

A configuração foi feita no arquivo `/etc/network/interfaces`, garantindo que as máquinas mantenham os mesmos IPs mesmo após reinicialização.

## 3. Configuração dos Servidores Apache (HTTPD)

### 3.1 Instalação e Configuração
```bash
sudo apt update && sudo apt install apache2 -y
```
Criamos uma página HTML personalizada em cada servidor para validar o balanceamento de carga.

#### **Arquivo `/var/www/html/index.html`**

**Servidor 1:**
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Servidor 1</title>
</head>
<body>
    <h1>Servidor 1</h1>
</body>
</html>
```

**Servidor 2:**
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Servidor 2</title>
</head>
<body>
    <h1>Servidor 2</h1>
</body>
</html>
```

Adicionamos CSS para estilizar as páginas.

## 4. Instalação e Configuração do HAProxy

### 4.1 Instalação
```bash
sudo apt update && sudo apt install haproxy -y
```

### 4.2 Configuração do HAProxy
Editamos o arquivo `/etc/haproxy/haproxy.cfg` e adicionamos:

```cfg
frontend http_front
    bind *:80
    default_backend http_back

backend http_back
    balance roundrobin
    option httpchk GET /
    server srv1 10.49.10.10:80 check
    server srv2 10.49.10.11:80 check
    server srv3 10.49.10.12:80 check
```

### 4.3 Explicação
- `balance roundrobin`: Define o algoritmo de balanceamento como "round-robin".
- `option httpchk GET /`: Ativa a verificação de saúde dos servidores backend.
- `check`: Monitora o estado dos servidores.

## 5. Monitoramento e Failover
Habilitamos o `option httpchk` no HAProxy para garantir que apenas servidores ativos recebam tráfego.

### Testes de Verificação
1. **Verificação de logs** nos servidores Apache:
   ```bash
   tail -f /var/log/apache2/access.log
   ```
2. **Teste de Failover:**
   - Desligamos um servidor (`sudo systemctl stop apache2`).
   - Repetimos o acesso ao IP do HAProxy.
   - Verificamos que todas as requisições foram redirecionadas para os servidores restantes.

## 6. Conclusão
- O HAProxy foi configurado com sucesso para balanceamento de carga e failover.
- Os testes demonstraram que o sistema pode continuar operando mesmo em caso de falha de um servidor backend.
- A configuração de IPs estáticos garantiu a estabilidade dos servidores.

## 7. Referências
WEB SERVER LOAD-BALANCING WITH HAPROXY ON UBUNTU 14.04. [s.l.]: [s.n.], [s.d.]. Disponível em: [URL]. Acesso em: 09 fev. 2025.

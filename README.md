# HAProxy com Servidores Apache HTTPD

## 1. Objetivo
Garantir alta disponibilidade para serviços web utilizando o balanceador de carga HAProxy e servidores Apache HTTPD. O sistema deve continuar funcionando mesmo em caso de falha de um ou mais servidores backend.

## 2. Arquitetura do Sistema

### 2.1 Componentes Utilizados
- **HAProxy**: Balanceador de carga.
- **Servidores Apache HTTPD**: Servidores Web Backend.

### 2.2 Configuração de Rede
Inicialmente, realizamos a instalação do Apache HTTPD e configuramos suas páginas nos servidores backend.
Os servidores recebiam IPs dinâmicos via DHCP, o que fazia com que um ou mais servidores tivessem seus endereços alterados ao serem reiniciados. Isso gerava inconsistências no balanceamento de carga, pois o HAProxy dependia de IPs fixos para direcionar as requisições corretamente.

### 2.3 Definição de IPs Estáticos
Inicialmente, os IPs dos servidores mudavam ao reiniciar a máquina virtual. Para resolver isso, configuramos os seguintes IPs estáticos:
- Servidor 1: `10.49.10.10/16`
- Servidor 2: `10.49.10.11/16`
- Servidor 3: `10.49.10.12/16`

A configuração foi feita no arquivo `/etc/network/interfaces`, garantindo que as máquinas mantenham os mesmos IPs mesmo após reinicialização.

## 3. Configuração dos Servidores Apache (HTTPD)

### 3.1 Instalação e Configuração
Para instalar o Apache em cada servidor backend, execute:

```bash
sudo apt update && sudo apt install apache2 -y
```
Cada servidor Apache deve ter uma página HTML personalizada para testar o balanceamento de carga.

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

## 4. Instalação e Configuração do HAProxy

### 4.1 Instalação
```bash
sudo apt update && sudo apt install haproxy -y
```

### 4.2 Configuração do HAProxy
O arquivo de configuração do HAProxy `(/etc/haproxy/haproxy.cfg)` deve ser atualizado com as seguintes definições:


```cfg
frontend http_front
    bind *:80
    default_backend http_back

backend http_back
    balance roundrobin
    option httpchk GET /
    server srv1 10.49.10.11:80 check
    server srv2 10.49.10.12:80 check
```
Após editar o arquivo, reinicie o HAProxy:

```cfg
sudo systemctl restart haproxy
```

### 4.3 Explicação
- `balance roundrobin`: Define o algoritmo de balanceamento como "round-robin".
- `option httpchk GET /`: Ativa a verificação de saúde dos servidores backend.
- `check`: Monitora o estado dos servidores.

## 5. Monitoramento e Failover
Habilitamos o `option httpchk` no HAProxy para garantir que apenas servidores ativos recebam tráfego.

```cfg
curl https://<IP_DO_HAPROXY>
```
Verifique se as respostas alternam entre os servidores.

```cfg
tall -f /var/log/apache2/accees.log
```
Resultado esperado: As requisições devem alternar entre os servidores 1 e 2. (clicando em F5)

### 5.1 Testes de Verificação de Saúde
No arquivo haproxy.cfg, ativamos o option httpchk, que verifica automaticamente a saúde dos servidores. Apenas servidores ativos recebem tráfego.

## 5.2 Testes de Balanciamenteo de Carga
1. Acesse o IP do HAProxy pelo navegador e recarregue a página diversas vezes.

## 5.2 Teste de Failover:**
   - Desligamos um servidor (`sudo systemctl stop apache2`).
   - Repetimos o acesso ao IP do HAProxy.
   - Verificamos que todas as requisições foram redirecionadas para os servidores restantes.

## 6. Conclusão
- O HAProxy foi configurado com sucesso para balanceamento de carga e failover.
- Os testes demonstraram que o sistema pode continuar operando mesmo em caso de falha de um servidor backend.
- A configuração de IPs estáticos garantiu a estabilidade dos servidores.

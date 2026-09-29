# Small Business Network Lab

Laboratório de rede de uma pequena empresa desenvolvido no **Cisco Packet Tracer**, com foco em fundamentos de redes, configuração de dispositivos, DHCP, roteamento e troubleshooting.

## 📌 Objetivo

Simular uma infraestrutura de rede de pequena empresa, permitindo praticar:

* Configuração de endereçamento IPv4;
* Configuração de interfaces de roteadores;
* Configuração de switches;
* DHCP;
* Gateway padrão;
* Comunicação entre diferentes redes;
* Rotas estáticas;
* Rota padrão;
* Simulação de uma rede externa/ISP;
* Diagnóstico e resolução de problemas de conectividade.

## 🖥️ Topologia

A topologia é composta por:

* 2 roteadores Cisco 2911;
* 2 switches Cisco 2960;
* 2 computadores;
* 3 servidores.

Estrutura principal:

```text
                    REDE EXTERNA / ISP
                         Server3
                       10.0.1.2/24
                           |
                       ISP G0/1
                       10.0.1.1/24
                           |
                       ISP Router
                       G0/0
                       10.0.0.2/30
                           |
                       10.0.0.1/30
                       R1 G0/2
                           |
                          R1
                    /             \
             G0/0 /                 \ G0/1
          192.168.1.1            192.168.2.1
               |                       |
           Switch1                 Switch2
          /   |    \                   |
        PC1  PC2  Server1           Server2
       .10   .11    .2                 .2
```

## 🌐 Endereçamento

| Dispositivo | Interface | Endereço IP | Máscara         | Gateway     |
| ----------- | --------- | ----------- | --------------- | ----------- |
| R1          | G0/0      | 192.168.1.1 | 255.255.255.0   | —           |
| R1          | G0/1      | 192.168.2.1 | 255.255.255.0   | —           |
| R1          | G0/2      | 10.0.0.1    | 255.255.255.252 | —           |
| ISP         | G0/0      | 10.0.0.2    | 255.255.255.252 | —           |
| ISP         | G0/1      | 10.0.1.1    | 255.255.255.0   | —           |
| Server1     | —         | 192.168.1.2 | 255.255.255.0   | 192.168.1.1 |
| Server2     | —         | 192.168.2.2 | 255.255.255.0   | 192.168.2.1 |
| Server3     | —         | 10.0.1.2    | 255.255.255.0   | 10.0.1.1    |
| PC1         | —         | DHCP        | 255.255.255.0   | 192.168.1.1 |
| PC2         | —         | DHCP        | 255.255.255.0   | 192.168.1.1 |

## ⚙️ Configurações realizadas

### DHCP

O R1 foi configurado como servidor DHCP para a rede `192.168.1.0/24`.

```text
ip dhcp pool LAN
network 192.168.1.0 255.255.255.0
default-router 192.168.1.1
dns-server 8.8.8.8
```

Foram excluídos os endereços de `192.168.1.1` até `192.168.1.9`, mantendo essa faixa disponível para o gateway, servidores e futuros dispositivos de infraestrutura.

Os computadores receberam endereços automaticamente, como:

```text
PC1 → 192.168.1.10
PC2 → 192.168.1.11
```

## 🔀 Roteamento

### Rotas diretamente conectadas

O R1 possui três redes diretamente conectadas:

```text
192.168.1.0/24
192.168.2.0/24
10.0.0.0/30
```

### Rota estática

Para alcançar a rede externa `10.0.1.0/24`, foi configurada uma rota estática:

```text
ip route 10.0.1.0 255.255.255.0 10.0.0.2
```

### Rota padrão

Também foi configurada uma rota padrão no R1:

```text
ip route 0.0.0.0 0.0.0.0 10.0.0.2
```

### Rota de retorno

No ISP foi configurada uma rota para retornar à rede interna:

```text
ip route 192.168.1.0 255.255.255.0 10.0.0.1
```

O ISP funciona neste projeto como uma **simulação de uma rede externa**, permitindo praticar comunicação entre a rede interna e uma rede externa.

## 🧪 Testes realizados

Foram realizados testes de conectividade utilizando `ping` e `traceroute`.

### Comunicação na rede local

```text
PC1 → PC2
```

Resultado:

```text
4 enviados
4 recebidos
0 perdidos
```

### Comunicação com o gateway

```text
PC1 → 192.168.1.1
```

Resultado:

```text
4 enviados
4 recebidos
0 perdidos
```

### Comunicação entre redes internas

```text
PC1 → Server2
```

Resultado após o primeiro teste:

```text
4 enviados
4 recebidos
0 perdidos
```

### Comunicação com a rede externa

```text
PC1 → Server3
```

Resultado:

```text
4 enviados
4 recebidos
0 perdidos
```

## 🔧 Troubleshooting

Durante o laboratório foram simuladas falhas para praticar diagnóstico de problemas de conectividade.

### Problema 1 — Interface do roteador desligada

**Sintoma:**

O PC1 possuía uma configuração IP válida, mas não conseguia alcançar o gateway.

```text
ping 192.168.1.1
→ 0/4
```

A tabela ARP também não apresentava uma entrada para o gateway.

Foi utilizado:

```text
show ip interface brief
```

Foi identificado:

```text
GigabitEthernet0/0
administratively down/down
```

**Causa:**

A interface G0/0 do R1 estava administrativamente desligada.

**Solução:**

```text
interface gigabitEthernet 0/0
no shutdown
```

Após a correção:

```text
PC1 → Gateway
4/4
```

E posteriormente:

```text
PC1 → Server3
4/4
```

### Problema 2 — Gateway incorreto no servidor externo

**Sintoma:**

O PC2 conseguia acessar o gateway, mas não conseguia acessar o Server3.

```text
PC2 → 192.168.1.1
4/4

PC2 → 10.0.1.2
0/4
```

Os testes no R1 e no ISP mostraram que o caminho até o Server3 estava funcionando.

Ao testar o caminho de retorno, foi identificado que o Server3 estava configurado com:

```text
IP:      10.0.1.2
Máscara: 255.255.255.0
Gateway: 192.168.2.1
```

O gateway estava incorreto, pois não pertencia à rede `10.0.1.0/24`.

**Correção:**

```text
Gateway: 10.0.1.1
```

Após a alteração, o Server3 conseguiu alcançar o R1 e a comunicação entre PC2 e Server3 foi restaurada.

### 🔎 Aprendizado de troubleshooting

Os problemas foram investigados de forma incremental, verificando:

1. Configuração IP do dispositivo;
2. Comunicação com o gateway;
3. Tabelas ARP;
4. Estado das interfaces;
5. Rotas;
6. Comunicação entre roteadores;
7. Caminho de retorno;
8. Configuração do gateway no destino.

Isso ajudou a praticar o princípio de que uma comunicação de rede precisa funcionar tanto no **caminho de ida quanto no caminho de retorno**.

## 🛠️ Ferramentas

* Cisco Packet Tracer
* Cisco IOS
* IPv4
* DHCP
* ARP
* ICMP
* Roteamento estático
* Traceroute

## 📚 Conhecimentos praticados

* Modelo de comunicação em redes;
* Endereçamento IPv4;
* Máscaras de sub-rede;
* Gateway padrão;
* DHCP;
* ARP;
* Switching;
* Routing;
* Rotas diretamente conectadas;
* Rotas estáticas;
* Rota padrão;
* Redes /24 e /30;
* Diagnóstico de conectividade;
* Troubleshooting de interfaces;
* Troubleshooting de gateway e rotas.

## 📁 Arquivos

```text
small-business-network-lab/
├── README.md
├── small-business-network.pkt
├── documentation/
│   ├── addressing.md
│   ├── routing.md
│   └── troubleshooting.md
└── screenshots/
    ├── topology.png
    ├── dhcp.png
    ├── routing.png
    └── tests.png
```

## 👨‍💻 Sobre o projeto

Este laboratório foi desenvolvido como projeto prático de estudos para consolidar conhecimentos fundamentais de **Suporte de TI, Redes e Infraestrutura**, utilizando o Cisco Packet Tracer para simular uma pequena rede empresarial e praticar configuração e troubleshooting.

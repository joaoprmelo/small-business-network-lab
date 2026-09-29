# Troubleshooting

Registro dos problemas de conectividade identificados e solucionados durante o laboratório.

## Problema 1 — Interface do R1 desligada

### Sintoma

O PC1 possuía uma configuração IP válida, porém não conseguia alcançar o gateway:

```text
ping 192.168.1.1
```

Resultado:

```text
4 enviados
0 recebidos
4 perdidos
```

### Diagnóstico

Primeiro foi verificada a configuração IP do PC1 utilizando:

```text
ipconfig
```

A configuração estava correta:

* IPv4: `192.168.1.11`
* Máscara: `255.255.255.0`
* Gateway: `192.168.1.1`

Em seguida, foi verificada a tabela ARP:

```text
arp -a
```

Não havia uma entrada para o gateway `192.168.1.1`.

No R1, foi executado:

```text
show ip interface brief
```

A interface `GigabitEthernet0/0` estava:

```text
administratively down/down
```

### Causa

A interface G0/0 do R1 estava administrativamente desligada.

### Solução

A interface foi ativada com:

```text
interface gigabitEthernet 0/0
no shutdown
```

Após a correção, o PC1 conseguiu alcançar o gateway:

```text
4 enviados
4 recebidos
0 perdidos
```

O acesso ao Server3 também foi restaurado.

---

## Problema 2 — Gateway incorreto no Server3

### Sintoma

O PC2 conseguia acessar seu gateway, porém não conseguia acessar o Server3:

```text
PC2 → 192.168.1.1
```

Resultado:

```text
4/4
```

Enquanto:

```text
PC2 → 10.0.1.2
```

Resultado:

```text
0/4
```

### Diagnóstico

Foi verificado se o R1 possuía uma rota para a rede do Server3:

```text
show ip route
```

A rota estava presente:

```text
S 10.0.1.0/24 via 10.0.0.2
```

Também foram realizados testes entre o R1 e o ISP:

```text
R1 → 10.0.0.2
```

Resultado:

```text
5/5
```

E entre o ISP e o Server3:

```text
ISP → 10.0.1.2
```

Resultado:

```text
5/5
```

Isso indicou que o caminho até o Server3 estava funcionando.

Foi então testado o caminho de retorno:

```text
Server3 → 10.0.0.1
```

O teste falhou.

Ao verificar a configuração de rede do Server3, foi identificado:

```text
IP:       10.0.1.2
Máscara:  255.255.255.0
Gateway:  192.168.2.1
```

O gateway estava incorreto.

### Causa

O Server3 estava configurado com um gateway pertencente à rede `192.168.2.0/24`, enquanto o próprio servidor pertence à rede `10.0.1.0/24`.

### Solução

O gateway foi alterado para:

```text
10.0.1.1
```

que corresponde à interface do ISP na rede do Server3.

Após a correção, o Server3 conseguiu alcançar o R1 e a comunicação entre PC2 e Server3 foi restaurada.

---

## Processo de diagnóstico utilizado

Os problemas foram investigados de forma incremental:

1. Verificação da configuração IP;
2. Teste do gateway;
3. Verificação da tabela ARP;
4. Verificação do estado das interfaces;
5. Verificação da tabela de roteamento;
6. Testes entre os roteadores;
7. Teste do caminho de retorno;
8. Verificação da configuração do dispositivo de destino;
9. Correção da causa identificada;
10. Novo teste para validar a solução.

## Principais aprendizados

Durante o troubleshooting, foram praticados conceitos importantes de redes:

* Uma interface `administratively down` precisa ser ativada;
* O gateway deve pertencer à rede local do dispositivo;
* O `ping` depende de comunicação de ida e de retorno;
* Uma rota existente não garante que todas as configurações necessárias estejam corretas;
* `show ip interface brief` ajuda a identificar problemas de interfaces;
* `show ip route` ajuda a verificar o caminho conhecido pelo roteador;
* `arp -a` ajuda a investigar a resolução de endereços IP para MAC;
* Testes realizados de diferentes dispositivos ajudam a isolar a origem de uma falha.

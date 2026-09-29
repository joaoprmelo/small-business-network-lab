# Routing

Documentação do roteamento utilizado no laboratório.

## Redes diretamente conectadas

O R1 possui três redes diretamente conectadas:

| Rede           | Interface          |
| -------------- | ------------------ |
| 192.168.1.0/24 | GigabitEthernet0/0 |
| 192.168.2.0/24 | GigabitEthernet0/1 |
| 10.0.0.0/30    | GigabitEthernet0/2 |

Essas redes são identificadas automaticamente pelo roteador quando as interfaces estão configuradas e ativas.

## Rota estática

Para permitir que o R1 alcance a rede externa simulada `10.0.1.0/24`, foi configurada uma rota estática:

```text
ip route 10.0.1.0 255.255.255.0 10.0.0.2
```

O próximo salto (`next-hop`) é `10.0.0.2`, que corresponde à interface do ISP conectada ao R1.

Fluxo:

```text
R1
 |
 | 10.0.0.1
 v
ISP
10.0.0.2
 |
 v
10.0.1.0/24
```

## Rota padrão

Também foi configurada uma rota padrão no R1:

```text
ip route 0.0.0.0 0.0.0.0 10.0.0.2
```

A rota padrão funciona como um caminho utilizado quando o R1 não possui uma rota mais específica para o destino.

## Rota de retorno

Para que dispositivos da rede externa possam responder aos dispositivos da LAN, o ISP possui uma rota de retorno para a rede `192.168.1.0/24`:

```text
ip route 192.168.1.0 255.255.255.0 10.0.0.1
```

O próximo salto é `10.0.0.1`, endereço da interface do R1 conectada ao ISP.

## Comunicação entre redes

Um exemplo de comunicação entre a rede interna e a rede externa:

```text
PC2
192.168.1.11
     |
     v
Gateway
192.168.1.1
     |
     v
R1
10.0.0.1
     |
     v
ISP
10.0.0.2
     |
     v
Server3
10.0.1.2
```

Para a comunicação funcionar corretamente, é necessário existir tanto um caminho de ida quanto um caminho de retorno.

## Testes

Foram realizados testes utilizando `ping` para validar o roteamento.

### R1 → ISP

```text
ping 10.0.0.2
```

Resultado:

```text
5 enviados
5 recebidos
0 perdidos
```

### ISP → Server3

```text
ping 10.0.1.2
```

Resultado:

```text
5 enviados
5 recebidos
0 perdidos
```

### PC1 → Server3

```text
ping 10.0.1.2
```

Resultado após a configuração do roteamento:

```text
4 enviados
4 recebidos
0 perdidos
```

## Comandos utilizados

Para verificar as interfaces:

```text
show ip interface brief
```

Para verificar a tabela de roteamento:

```text
show ip route
```

Para verificar uma rota específica:

```text
show ip route 10.0.1.0
```

Para testar conectividade:

```text
ping <endereço-IP>
```

Para verificar o caminho percorrido pelos pacotes:

```text
traceroute <endereço-IP>
```

# Tutorial: Configurando um Servidor PPPoE "Bypass" (Aceita-Tudo) por VLAN

Este tutorial documenta o processo de configuração de um BNG Huawei para aceitar qualquer cliente PPPoE (qualquer usuário e senha) em uma VLAN específica e atribuir-lhe um pool de IPs predefinido.

**Objetivo:** Permitir que clientes PPPoE na VLAN 3150 se conectem instantaneamente, independentemente do usuário ou senha fornecidos, usando um "domínio coringa".

## 1. Definição dos Esquemas de Autenticação e Contabilidade "Bypass"

Primeiro, criamos os esquemas de autenticação (`authentication-scheme`) e contabilidade (`accounting-scheme`) que efetivamente "burlam" a verificação.

```huawei
aaa
 #
 authentication-scheme open-radius-auth
  authentication-mode none
 #
 #
 accounting-scheme open-radius-acct
  accounting-mode none
 #
````

**Finalidade:**

  * **`authentication-scheme open-radius-auth`**: Cria um esquema de autenticação chamado `open-radius-auth`.
  * **`authentication-mode none`**: Este é o comando-chave. Ele instrui o BNG a **não realizar nenhuma autenticação** (nem local, nem RADIUS) e simplesmente aceitar a conexão.
  * **`accounting-mode none`**: Instrui o BNG a **não enviar** registros de contabilidade (Start, Stop, Interim) para este usuário.

-----

## 2\. Configuração do Domínio Coringa

Com os esquemas de "bypass" prontos, criamos o domínio (`temporario-usuarios-repetidos`) que será o "pacote de serviços" para esses clientes.

```huawei
domain temporario-usuarios-repetidos
 authentication-scheme open-radius-auth
 accounting-scheme open-radius-acct
 ip-pool cgnat-100.81.16.0-20
 ipv6-pool pool-ipv6-pd-2
 user-max-session 65530
 dns primary-ip 8.8.4.4
 dns second-ip 8.8.8.8
 dns primary-ipv6 2001:4860:4860::8888
 dns second-ipv6 2001:4860:4860::8844
 reallocate-ip-address
 user-basic-service-ip-type ipv4
#
```

**Finalidade:**

  * Este domínio é o destino para onde os clientes da VLAN 3150 serão forçados a ir.
  * Ele aplica os esquemas de "bypass" (`open-radius-auth` e `open-radius-acct`) que criamos no Passo 1.
  * Ele define quais recursos o cliente receberá após a conexão:
      * **IP Pool:** `cgnat-100.81.16.0-20`
      * **IPv6 Pool:** `pool-ipv6-pd-2`
      * **DNS:** Servidores DNS IPv4 e IPv6.
      * **`user-max-session 65530`**: Permite que o mesmo usuário logue múltiplas vezes.

-----

## 3\. Ajuste do Virtual-Template do Servidor PPPoE

O `Virtual-Template` é o modelo lógico que o BNG usa para criar a sessão PPP de cada cliente.

```huawei
interface Virtual-Template2
 ppp authentication-mode pap chap mschapv1 mschapv2
 ppp mru 1500
 pppoe-server service-name-parameter INTERNET
 pppoe-server ac-name PPPoE
 tcp adjust-mss 1420
 ip urpf strict enable check subnet
 ipv6 urpf strict enable check subnet
#
```

**Finalidade:**

  * **`ppp authentication-mode ...`**: Define os protocolos PPP que o servidor aceita (PAP, CHAP, etc.). Embora o BNG esteja em `authentication-mode none`, essa negociação PPP ainda precisa ocorrer.
  * **`tcp adjust-mss 1420`**: Essencial para evitar problemas de MTU em conexões PPPoE, garantindo que os pacotes TCP sejam fragmentados corretamente.
  * **`ip urpf ...`**: Uma medida de segurança (Unicast Reverse Path Forwarding) para mitigar ataques de spoofing.

-----

## 4\. Configuração da Interface de Acesso (BAS)

Esta é a etapa que "amarra" tudo. Configuramos a sub-interface da VLAN 3150 para forçar todos os clientes para o nosso domínio coringa.

```huawei
interface Eth-Trunk100.3150
 description VLANs-PPPoE
 user-vlan 3150
 pppoe-server bind Virtual-Template 2
 bas
  #
  access-type layer2-subscriber default-domain pre-authentication temporario-usuarios-repetidos authentication force temporario-usuarios-repetidos
  #
#
```

**Finalidade:**

  * **`user-vlan 3150`**: Escuta por clientes PPPoE nesta VLAN.
  * **`pppoe-server bind Virtual-Template 2`**: Aplica os parâmetros do `Virtual-Template 2` (do Passo 3) a todos que conectarem aqui.
  * **`access-type ... force ...`**: Este é o comando mais importante. Ele instrui o BNG a:
    1.  Ignorar qualquer domínio que o usuário envie (ex: `@algar.com.br`).
    2.  **Forçar** o cliente para o domínio `temporario-usuarios-repetidos` (do Passo 2) para autenticação e alocação de recursos.

-----

## 5\. Etapa de Troubleshooting (Crucial)

Durante a implementação, descobrimos que usuários específicos (como `algar@algar.com.br`) falhavam, mesmo com a configuração de "bypass". Os logs mostravam falhas de `LAM` (Local Authentication Module).

Isso ocorreu porque o usuário existia na base de usuários local do BNG (`local-aaa-server`), e essa verificação tem prioridade sobre o domínio da interface.

**Ação Corretiva (O passo que faltava):**
Foi necessário remover o usuário conflitante da base local.

```huawei
system-view
local-aaa-server
 undo user algar@algar.com.br
commit
```

**Finalidade:**

  * Ao remover o usuário da `local-aaa-server`, o BNG parou de "sequestrar" o login e permitiu que a regra `force` da interface (do Passo 4) funcionasse corretamente, enviando o usuário para o domínio de bypass.

-----

## 6\. Verificação Final

Após aplicar todas as etapas, o comando de verificação mostra o cliente conectado com sucesso, no domínio correto e com o IP do pool esperado.

```huawei
<100-SUA_FIBRA-BORDA_BNG>display access-user domain temporario-usuarios-repetidos
  ------------------------------------------------------------------------------
  UserID     Username                Interface      IP address       MAC
             Vlan          IPv6 address             Access type
  ------------------------------------------------------------------------------
  20529      algar@algar.com.br      Eth-Trunk100.3150  100.81.17.14     789a-1823-0759
             3150/-        -                        PPPoE                          
  27698      algar@algar.com.br      Eth-Trunk100.3150  100.81.25.43     f8b1-322f-80cb
             3150/-        -                        PPPoE                          
  ------------------------------------------------------------------------------
  Normal users                       : 2
  RUI Local users                    : 0
  RUI Remote users                   : 0
  Total users                        : 2
<100-SUA_FIBRA-BORDA_BNG>
```

**Resultado:**
O usuário `algar@algar.com.br` foi forçado para o domínio `temporario-usuarios-repetidos`, sua autenticação foi "bypassed" (`authentication-mode none`), e ele recebeu o IP `100.81.20.252` (do pool `cgnat-100.81.16.0-20`).

```
```

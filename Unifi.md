# Tutorial: Provisionando UniFi em Nuvem com DHCP Option 43 no Mikrotik

Este tutorial demonstra como configurar a Opção 43 do DHCP em um roteador Mikrotik. O objetivo é permitir que dispositivos UniFi (como Access Points, Switches, etc.) localizados em uma rede L3 (rede diferente da do controlador) possam encontrar e se conectar automaticamente a um controlador UniFi na nuvem ou em outra localidade.

Usaremos como exemplo o endereço IP do controlador **`45.234.128.130`**.

---

## O Conceito: Como Funciona a Option 43 para UniFi?

Quando um dispositivo UniFi novo é ligado em uma rede, ele pede um endereço IP ao servidor DHCP. Junto com o IP, ele também pergunta: "Existe alguma configuração especial para mim?". É aí que a Option 43 entra.

Para os equipamentos da Ubiquiti, o valor da Option 43 precisa seguir um formato hexadecimal específico:

`[Código da Sub-Opção]` + `[Tamanho do IP]` + `[Endereço IP do Controlador em Hexadecimal]`

- **Código da Sub-Opção:** Para UniFi, é sempre **`01`**.
- **Tamanho do IP:** Um endereço IPv4 tem 4 bytes, então o valor é **`04`**.
- **IP em Hex:** O IP do controlador convertido para hexadecimal.

Isso nos dá um prefixo fixo de **`0104`** que usaremos em todos os casos.

---

## Passo a Passo para a Configuração

Siga os passos abaixo para calcular e configurar o valor correto no seu Mikrotik.

### Passo 1: Converter o IP do Controlador para Hexadecimal

Primeiro, pegamos o endereço IP do nosso controlador (`45.234.128.130`) e convertemos cada um dos quatro números (octetos) para o seu equivalente hexadecimal.

- **45** (decimal) = **`2D`** (hexadecimal)
- **234** (decimal) = **`EA`** (hexadecimal)
- **128** (decimal) = **`80`** (hexadecimal)
- **130** (decimal) = **`82`** (hexadecimal)

> **Dica:** Você pode usar a calculadora do seu sistema operacional (Windows, macOS) no modo "Programador" para fazer essa conversão facilmente.

Ao juntar os valores, nosso IP em hexadecimal é: **`2DEA8082`**.

### Passo 2: Montar o Valor Final da Option 43

Agora, combinamos o prefixo fixo com o IP convertido:

**`0104`** + **`2DEA8082`** = **`01042DEA8082`**

Este é o valor final que iremos inserir no Mikrotik.

### Passo 3: Criar a Option 43 no Mikrotik

Vamos adicionar essa nova opção ao nosso servidor DHCP. Você pode fazer isso via terminal (recomendado) ou pela interface gráfica (WinBox).

**Via Terminal (mais rápido):**

Abra um novo terminal no seu Mikrotik e cole o seguinte comando:

```bash
/ip dhcp-server option add code=43 name=unifi-cloud-controller value='0x01042DEA8082'

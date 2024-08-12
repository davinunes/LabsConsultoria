# BGP Em Junos

> Utilizada Imagem VMX disponivel gratuitamente na web com simulador pnetlab/eve-ng

> Por  algum motivo, essa imagem de Junos demora uns 11 minutos pra subir e disponibilizar a CLI, não importa o quanto aumente memória e CPU.

Pra facilitar as coisas, é interessante conectar a fxp0 na cloud do simulador pra obter acesso ssh na vm

### Configurando acesso SSH para tornar o lab mais prático
```shell
configure
top edit interfaces fxp0 unit 0
set description "Interface de Gerencia"
set family inet address 192.168.3.5/24
commit
```

```shell
set system services ssh
set system root-authentication plain-text-password
```
> Coloque uma senha

```shell
top edit system login user lab 
set class super-user
set authentication plain-text-password
```
> Coloque uma senha

```shell
commit
```
Pronto, agora já pode acessar com um terminal melhor.

### Aplicando a licença de laboratório
```shell
request system license add terminal
```
Cole a chave e, numa linha vazia, pressione Ctrl + D para confirmar:
```
E435890758 aeaqic aiagij cpabsc idycyi giydco bqgiyd
           sdapoz gvqlkk ovxgs4 dfojcx mylma4 uhfs7p
           etdlfc ex2i5k 4mo4fa ew2xbz ujzbol 4fl44z
           ya3md2 drdgjf preowc 5ubrju kyq'
```

## Configurando uma Sessão Básica sem Filtros


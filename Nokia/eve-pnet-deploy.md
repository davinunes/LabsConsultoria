# blablabla

> Bibliografoa: https://tmsoft.com.br/temp/nokia-eve-setup.txt

Baixe o arquivo fornecido na aula

```bash
ls -lah
-rw-r--r--  1 root root 526M Aug 13 23:34 'vSIM R24.zip'
```

Descompacte

```bash
unzip vSIM\ R24.zip
cd vSIM/
unzip Nokia-vSIM-KVM-24.10.R2.zip
cd vm/vSIM-KVM/sros-x86-64/
ls
sros-vsim.qcow2
```

Crie os diretorios do CPM e IOM e depois copie o vHD pra dentro deles

```bash
mkdir /opt/unetlab/addons/qemu/timoscpm-ng-24.10.R2
mkdir /opt/unetlab/addons/qemu/timosiom-ng-24.10.R2

cp sros-vsim.qcow2 /opt/unetlab/addons/qemu/timoscpm-ng-24.10.R2/hda.qcow2
cp sros-vsim.qcow2 /opt/unetlab/addons/qemu/timosiom-ng-24.10.R2/hda.qcow2
```

Ajuste as permissões:

```bash
/opt/unetlab/wrappers/unl_wrapper -a fixpermissions
```

Crie um Lab e adicione um nó *CPM*, aumente a quantidade de CPU e RAM pra que o boot seja mais rápido.

> Utilizei 8 vcpu e 8192 de ram, mas ajuste conforme a sua disponibilidade

Ajuste a versão do Qemu para 2.4.0

Em UUID substitua pelo UUID presente na licença

<img width="897" height="645" alt="image" src="https://github.com/user-attachments/assets/18dd3e85-081a-43ab-8163-3c7812dbf269" />

Em *timos_line*, use: ``` slot=A chassis=SR-12e card=cpm5 ```

Em *management_address*, utilize ```198.18.22.2/24```

Em *timos_license* utilize ``` ftp://admin:admin@198.18.22.1/lic.txt ``` para iniciar automaticamente de um ftp ou utilize ``` cf3:/license.txt ``` para inserir na mão 

<img width="878" height="522" alt="image" src="https://github.com/user-attachments/assets/812a398a-f1bf-4e42-a8c2-40e135494822" />


### Licença com servidor FTP:

Crie um nó mikrotik, defina a senha de ```admin``` para ```admin```, acesse pelo winbox e coloque a licença no arquivo lic.txt. C

Configure uma porta com ip ```198.18.22.1/24```

Conecte a porta *mgmt* da vSim no mikrotik.

Assim, durante o boot, a vSim vai baixar a licença do ftp e aplicar.

<img width="857" height="561" alt="image" src="https://github.com/user-attachments/assets/7ff94602-fc7c-4c23-9556-47037494988a" />


### Licença sem servidor FTP:

Conectar no TELNET e esperar carregar

Login: ```admin```

Senha: ```admin```

entre na sessao file:

```
file
vi license.txt
```
cole o conteudo do arquivo txt da licenca, salve (é um editor vi)

ative a licenca: ```admin system license validate ```

reinicie com ```admin reboot```

Para conferir a licença use ```show system license available-licenses```

# Adicione o nó IOM

<img width="889" height="762" alt="image" src="https://github.com/user-attachments/assets/99f736ea-51d1-477f-90d5-5ad4ddf0536f" />

No IOM a unica alteração é a timos_line, ajuste para 

```
slot=1 chassis=SR-12e sfm=m-sfm5-12e card=iom4-e mda/1=me10-10gb-sfp+ mda/2=isa2-bb
```

Conecte a porta SF na porta SF da CPM

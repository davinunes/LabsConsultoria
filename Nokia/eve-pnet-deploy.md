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

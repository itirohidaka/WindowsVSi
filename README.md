# WindowsVSi
Procedimento para obter a senha do Windows no ambiente VPC na IBM Cloud.

### Antes de Começar
Você precisará dos seguintes componentes:
- Docker Community instalado no laptop
- Mac OS. Este tutorial foi elaborado utilizando um MacBook.

Criamos este tutorial passo a passo pois o Mac OS utiliza o LibreSSL que é incompatível com o procedimento padrão.

### Passo a Passo

1. Efetuar o Login na IBM Cloud usando a CLI.
```shell
ibmcloud login
```
2. No terminal, criar uma nova pasta (mkdir nova pasta). Acessar a pasta previamente criada (cd pasta)
```shell
mkdir winpass
cd winpass
```
3. Criar as chaves Publicas e Privada utilizando o comando ssh-keygen. Deixar o arquivo sem senha. Este passo criará dois arquivos na pasta previamente criada (um sem extensão e outro com a extensão .pub)
```shell
ssh-keygen -m PEM -t rsa -f winpass
```
4. Criar a SSH Key na IBM Cloud através do commando: (ibmcloud is key-create winless @winpass.pub)
```shell
ibmcloud is key-create winless @winpass.pub
```
5. Criar a instância Windows no ambiente VPC utilizando a interface gráfica, cli ou api. Selectionar a SSH-key previamente criada e aguardar até o termino da criação.

6. Listar as VSIs da VPC. Anotar o ID da máquina virtual Windows
```shell
ibmcloud is instances
```
7. Obter a Senha Criptografada do Windows e inserir no arquivo examplepwd64 com o comando abaixo. Substituir o <instance-id> pelo ID da máquina virtual que adquiriu na etapa anterior.
```shell
ibmcloud is instance-initialization-values <instance-id> | grep "Password" | sed 's/Password//g' | sed -e 's/^[ \t]*//' | base64 —decode > examplepwd64
```
8. Criação do arquivo Dockerfile (alpine linux + openssl) e criar uma imagem Docker chamada winpass.
```shell
printf "FROM alpine:latest\nRUN apk add openssl" > Dockerfile && docker build -t winpass .
```
9. Executar o container e obter a senha.
```docker
docker run -it -v $(pwd):/data winpass openssl pkeyutl -in /data/examplepwd64 -decrypt -inkey /data/winpass -pkeyopt rsa_padding_mode:oaep -pkeyopt rsa_oaep_md:sha256 -pkeyopt rsa_mgf1_md:sha256
```
A senha será exibida no terminal.

## Sumário
- [Antes de começar](#antes-de-começar)
- [Desinstalando o MySQL local](#desinstalando-o-mysql-local)
- [Usando o MYSQL via Docker](#usando-o-mysql-via-docker)
- [Instalação do Workbench via Snap](#instalando-o-workbench-via-snap)
- [Erros Comuns](#erros-comuns)
  - [MySQL Error: Access denied for user root@localhost](#problema-1)
  - [ER_NOT_SUPPORTED_AUTH_MODE: Client does not support authentication protocol requested by server; consider upgrading MySQL client](#problema-2)
  - [ERROR 1819 (HY000): Your password does not satisfy the current policy requirements.](#problema-3) 
  - [The name org.freedesktop.secrets was not provided by any .service files](#problema-4)
  - [TypeError: Cannot read property 'query' of undefined](#problema-5)


### Antes de começar

- Recomendamos **fortemente** que utilize o MySQL com Docker
- Utilize a versão 5.7 do MySQL

---

### Desinstalando o MySQL local

Caso você tenha instalado o MySQL localmente, recomendamos que o desinstale para não haver confusão.

```sh
sudo apt-get purge mysql-server mysql-client mysql-common mysql-server-core-* mysql-client-core-*
sudo rm -rf /etc/mysql /var/lib/mysql
sudo apt autoremove
sudo apt autoclean
```

### Usando o MySQL via Docker

Verifique se o serviço do **Docker** esta em funcionamento:

```sh
  // Verificando o status do serviço
  sudo service docker status

  // Caso não esteja rodando, inicie o serviço
  sudo service docker start
```


Para **baixar** e **criar** um container com a [imagem do MYSQL](https://hub.docker.com/_/mysql), usamos este comando:

```sh
docker run -p 3306:3306 --name trybe_mysql -e MYSQL_ROOT_PASSWORD=sua_senha -d mysql:5.7
```

Atente que você pode seta a senha que preferir no lugar de "sua_senha". 

Para **ver** se nosso container esta rodando corretamente, usamos este comando:

```sh
docker container ls
```

Para **parar** a execução do nosso container, usamos este comando:

```
docker container stop trybe_mysql
```

Para **iniciar** novamente a execução do nosso container, usamos este comando:

```
docker container start trybe_mysql
```

Caso precise **entrar no MYSQL via terminal**, usamos este comando:

```
docker exec -it trybe_mysql bash
```

Após executar o comando anterior, seu terminal irá prover um *shell bash* dentro do container do MYSQL, agora basta usar este comando:

``` sh
// visualização parecida com: root@d113f0f4d54f:/#
mysql -u root -p
```

> Obs. A senha é "sua_senha", conforme informamos comando `docker run`

Depois pode realizar um teste com o comando:

```sh
show databases;
```


### Instalando o Workbench via Snap

Caso sua distro **NÃO** fornecer suporte ao pacote oficial, então precisamos instalar o **MYSQL Workbench** usando **Snap**, siga os seguintes comandos:

```sh
sudo snap install mysql-workbench-community
snap connect mysql-workbench-community:password-manager-service
snap connect mysql-workbench-community:ssh-keys
snap connect mysql-workbench-community:cups-control
```

> Obs. Caso você use o Linux Mint, precisamos habilitar a instalação do snap com os comandos a seguir:

```sh
sudo rm /etc/apt/preferences.d/nosnap.pref
sudo apt update
sudo apt install snapd
```

---

### Erros Comuns

Algumas pessoas podem ter problemas com os testes locais nestes projetos de **MYSQL**, quatro possíveis problemas e suas respctivas soluções:


Antes de fazer o que esta descrito nos **problemas 1, 2 e 3**, entre no MYSQL via linha de comando:

```sh
// 'root' se o seu usuário chamar root
sudo mysql -u root -p
```

---

#### Problema 1

> MySQL Error: Access denied for user root@localhost

Primeiramente, qual o motivo desse erro acontecer?
- Você não está conseguindo se autenticar como o usuário `root`, provavelmente a senha que está inserindo está errada.

Execute os comandos abaixo, dentro do MYSQL:

```
  // 'root' se o seu usuário chamar root
  DROP USER 'root'@'localhost';
```

```
  CREATE USER 'root'@'%' IDENTIFIED BY 'sua_senha';
```

```
  GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
```

```
  FLUSH PRIVILEGES;
```

---

#### Problema 2

> ER_NOT_SUPPORTED_AUTH_MODE: Client does not support authentication protocol requested by server; consider upgrading MySQL client

Primeiramente, qual o motivo desse erro acontecer?
- A partir da versão 8 do mysql ele passou a ter suporte a um novo esquema de autenticação, então esse erro só ocorre pra quem usa mysql versão > 8;
o que é esse erro?
- Os testes são implementados em node.js, e o node utiliza uma biblioteca para acessar o mysql e realizar os comandos para teste, o que acontece é que a biblioteca do node ainda não tem suporte para a nova autenticação do mysql > 8;
- Então o recomendado é você usar a versão 5.7.
    - Caso não seja possível, a alternativa é mudar o modo de autenticação do seu usuário `root`.

##### Mudando o modo de autenticação usando docker

Caso você esteja utilizando docker-cli e rodando a imagem com o comando `run`, remova o container antigo e adicione essa flag junto ao comando `--default-authentication-plugin=mysql_native_password`. Logo abaixo existe o [comando completo](#criando-um-container-com-a-imagem-do-mysql)

Caso esteja utilizando docker compose, adicione a seguinte linha com a mesma identação dos comandos image, port.
`command: --default-authentication-plugin=mysql_native_password`


#### Problema 3

> ERROR 1819 (HY000): Your password does not satisfy the current policy requirements.

Esse problema está relacionado à política de requerimentos para uma senha. Com o comando abaixo, você flexibiliza essa política.


```
SET GLOBAL validate_password.policy=LOW;
```

---

#### Problema 4

> The name org.freedesktop.secrets was not provided by any .service files

Se ao tentar conectar com o banco pelo MySQLWorkbench uma pop up surgir com o erro acima, ou algum erro semelhante que envolva o termo `org.freedesktop.secrets`: é possível que o problema seja relativo à **keyring**, um programinha que o sistema operacional usa para armazenar senhas e credenciais.

Para resolver, vocês podem tentar instalar o **gnome-keyring**:

```sh
sudo apt-get update -y
```
```sh
sudo apt-get install -y gnome-keyring
```

Fechar e abrir novamente o workbench após a instalação e testar a conexão com o banco.

##### :warning: Vale ressaltar que em algumas máquinas os passos não dão certo :disappointed: então suba os requisitos para serem avaliados no GitHub :+1: :warning:

---

#### Problema 5 

> TypeError: Cannot read property 'query' of undefined

Provalmente vocês não colocou nas variáveis de ambiente suas credenciais corretas do `mysql` ou deixou de colocar uma ou mais vairáveis.
O comando correto para rodar os testes é:
```sh
MYSQL_USER=root MYSQL_PASSWORD=sua_senha HOSTNAME=localhost npm test
```
> Dica: para só usar o comando `npm test` vocẽ pode exportar para a sessão do seu terminal as variáveis de ambiente. `npm test` vai ser executado nesse terminam com as variáveis de ambiente injetadas nele, vai funcionar até fechar esse terminal. Um exemplo de exportação segue abaixo:

```sh
export MYSQL_USER=root MYSQL_PASSWORD=sua_senha HOSTNAME=localhost
```

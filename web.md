## Organização e instalação de um sistema web

### O que é um sistema web
Um sistema web pode ser visto como um conjunto de recursos, especialmente programas, necessários para ofertar uma aplicação acessível a partir de uma rede de computadores.

O Wordpress, por exemplo, é um dos sistemas de gerenciamento de conteúdo mais utilizados na Internet. Para viabilizar o Wordpress são necessárias diversas peças, dentre elas:
  - Uma máquina - real ou virtual - conectada à Internet;
  - Um serviço de nome capaz de converter o endereço IP da máquina conectada à Internet num nome fácil de lembrar como linux.ime.usp.br;
  - Um programa responsável por atender as requisições realizadas pelo navegador até a máquina que está conectada à Internet;
  - Um programa responsável por rodar o código PHP no qual o Wordpress foi escrito. Podemos dizer aqui que temos um programa que roda outro programa;
  - Um sistema de banco de dados que armazena, por exemplo, os posts e as configurações do Wordpress;
  - Um lugar para armazenar os uploads realizados associados eventualmente aos posts.

Vários desses requisitos podem ser entregues por serviços contratados, podem residir em máquinas distintas, podem ser oferecidos por uma única máquina... Diante de tantas possibilidades parece ser interessante saber o que esses modelos de organização trazem de bom e de ruim.

### Atividades
1) Instalar o LXC a partir do repositório. (usar o `apt-get`)

**OBS:** para instalar um programa é preciso virar `root`.

2) Criar um container, vamos chamá-lo de `backend`. O comando utilizado para criar um container será o `lxc-create`. É necessário privilégio de `root`.

Exemplo:

```bash
# o comando abaixo criará um container com o debian atual e com o nome mundial
# -t é de type
# -n é de name
lxc-create -t debian -n mundial

# o comando abaixo inicializará o container
lxc-start -n mundial

# o comando abaixo listará os containers criados
# -f é de fancy, para imprimir na tela bonitinho
lxc-ls -f

# o comando abaixo acessará o container
lxc-attach -n mundial
```

3) No container `backend`, instalar todos os pacotes abaixo:

```bash
apt install apache2 libapache2-mod-php curl php-curl php-gd php-imagick php-intl php-mbstring php-mysql php-soap php-xml php-xmlrpc php-zip 
```

Os pacotes instalados são o servidor web **Apache**, o módulo para processar arquivos **PHP** a partir do Apache e diversas bibliotecas de PHP utilizadas pelo **Wordpress**.

Verificar o IP do container utilizando o comando `ip address`.

Criar em `/var/www/html` o arquivo `x.php` com o conteúdo abaixo:

```php
<?php
phpinfo();
?>
```

Para verificar se a instalação foi correta, abrir o navegador no **host** e acessar como endereço o IP do container (e.g. http://10.0.3.14). Por fim, acessar o arquivo PHP recém criado (e.g. http://10.0.3.14/x.php).

4) Criar outro container. Chamá-lo-emos de `bd`.

5) No `bd`, instalar o `mariadb-server`.

Editar o arquivo `/etc/mysql/mariadb.conf.d/50-server.cnf`. Queremos trocar o `bind-address = 127.0.0.1` por `bind-address = 0.0.0.0`. O objetivo disso é garantir que o nosso servidor seja capaz de atender via rede. Lembrando que o endereço `127.0.0.1` é o endereço de *loopback*.

Quase sempre que um arquivo de configuração de um servidor for editado é razoável reiniciá-lo.

```bash
service mysql restart
```

**Pergunta:** instalamos o `mariadb-server`. Por que estamos reiniciando o `mysql`?

Acessar o mariadb, criar um usuário, criar um banco de dados e garantir as permissões adequadas.

```bash
mysql

# para cada comando abaixo, rodá-lo no mariadb (ou seria mysql???)
create user wordpress identified by 'mundialdopalmeiras';
create database wordpress;
grant all privileges on wordpress.* to wordpress;
```

6) Instalar o Wordpress: voltar ao `backend`. Lá:
  - trocar de diretório para `/var/www/html`;
  - instalar o `unzip` e o `wget` via `apt-get`;
  - baixar o Wordpress com o `wget` e o extrair;
  - trocar recursivamente o dono do diretório recém extraído para o usuário `www-data`.

Exemplos de comandos abaixo
```bash
# para baixar o wordpress
wget https://br.wordpress.org/latest-pt_BR.zip

# para extrair o wordpress recém baixado
unzip latest-pt_BR.zip

# para trocar o dono
chown -R www-data: wordpress
```

Abrir o instalador do Wordpress no navegador. O endereço deve ficar algo como `http://10.0.3.14/wordpress`.

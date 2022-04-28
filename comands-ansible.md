## Aula 01 - Introdução ao Ansible

```hosts```: um arquivo de inventário onde colocaremos todas as informações necessárias para o Ansible saber como acessar as máquinas.

Grupo: utilidade é informar ao Ansible para o servidor qual é a sua serventia. Exemplo: grupo ```wordpress```.

```$ ansible wordpress -u vagrant --private-key .vagrant/machines/wordpress/virtualbox/private_key -i hosts -m shell -a 'echo Hello, World'```: o Vagrant sempre cria um usuário chamado vagrant, além dele, passaremos o arquivo de inventário hosts e o módulo shell. Módulos são os comandos que o Ansible são capazes de rodar, o primeiro deles é responsável por executar um comando no Shell quando rodarmos o SSH na máquina. Nós queremos passar ```echo Hello, World```. Passamos também no comando o nome do grupo que será afetado pelo comando e uma chave ssh para acessar a máquina virtual.

__Resumo do comando:__

* grupo de hosts que queremos rodar o comando;
qual nome do usuário;
* a chave privada (e evitar passar senha na linha de comando);
* informamos o arquivo de inventário para ele conferir os hosts que configuraremos;
* informamos o módulo que executaremos: shell;
* quais os argumentos que estamos passando: echo Hello, World (comando que eu executaria direto no bash).

Podemos adicionar a flag ```-vvvv``` para que o comando tenha uma saída mais verbosa para que possamos depurar o que o Ansible executou:

```ansible -vvvv wordpress -u vagrant --private-key .vagrant/machines/wordpress/virtualbox/private_key -i hosts -m shell -a 'echo Hello, World'```
___________
## Aula 02 - O primeiro Playbook

O formato ```yml``` é o padrão adotado pelo Ansible.

O arquivo de configuração deve ser iniciado com três hífens (---) e o elemento de primeiro nível é sempre uma lista que começa com um filtro chamado hosts.

O código a seguir cria um arquivo de texto com a palavra 'hello' em uma pasta especificada na VM:
```
- hosts: all
  tasks: 
   - shell: 'echo hello > /vagrant/world.txt'
```

Para executar o provisionamento da máquina, digitamos o seguinte comando:
```ansible-playbook provisioning.yml -u vagrant -i hosts --private-key .vagrant/machines/wordpress/virtualbox/private_key```

```ansible-playbook```: é o comando para executar o provisionamento a partir de um arquivo de configuração.

```-u vagrant```: a flag ```-u``` indica qual o usuário da VM.

```-i hosts```: a flag ```-i``` indica qual o arquivo de invetário.

```--private-key .vagrant/machines/wordpress/virtualbox/private_key```: a flag ```--private-key``` indica o local onde está a chave privada para acessar a VM.
__________
### Instalando dependências
O módulo usado no Ansible para instalar pacotes em distribuições do Ubuntu é o ```apt```. Na maior parte dos módulos do Ansible, é relevante usar o parâmetro ```state``` para informar o estado desejado da execução de uma task.

E se nós estamos instalando um pacote em um sistema operacional, deve ser feito como um usuário root. No Ansible, informamos que queremos executar algo como root e não como o usuário que logamos via SSH, como o parâmetro ```become```. Iremos configurá-lo com yes e deixá-lo no mesmo nível de tasks - ou seja, ele não é outro parâmetro dentro dele.

Podemos adicionar um nome a nossa task por meio do parâmetro ```name```. O seguinte código instalar a última versão do php5:
```
- hosts: all
  tasks:
    - name: 'Instala o PHP5'
      apt:
        name: php5
        state: latest
      become: yes
```
__________
### O que aprendemos?
* Como criar um Playbook de Ansible;
* Como instalar dependências usando o Ansible;
* Como utilizar alguns parâmetros de forma correta no seu Playbook.
__________
## Aula 03 - Aplicando boas práticas

Podemos diminuir o nosso código usando um loop. Como todos os pacotes que estão sendo instalados usam o ```apt```, podemos generalizar e passar uma lista com os nomes dos pacotes que desejamos instalar. Faremos isso com o comando ```wit_items```:

```
---
- hosts: all
  tasks:
    - name: 'Instala pacotes de dependências do sistema operacional'
      apt:
        name: "{{ item }}"
        state: latest
      become: yes
      with_items:
        - php5
        - apache2
        - libapache2-mod-php5
        - php5-gd
        - libssh2-php
        - php5-mcrypt
        - mysql-server-5.6
        - python-mysqldb
        - php5-mysql
```

A versões mais recentes do Ansible descontinuaram essa forma de laço. Abaixo a nova forma mais enxuta:

```
- hosts: all
  tasks:
    - name: 'Instala pacotes de dependências do sistema operacional'
      apt:
        name:
          - php5
          - apache2
          - libapache2-mod-php5
          - php5-gd
          - libssh2-php
          - php5-mcrypt
          - mysql-server-5.6
          - python-mysqldb
          - php5-mysql
        state: latest
      become: yes
```
____________
### Trabalhando com vários logins
Para podermos simplificar o comando que executamos para provisionar com Ansible, podemos adicionar no arquivo de hosts algumas informações referentes a VM. Por exemplo, podemos adicionar o nome de usuário e o caminho da chave privada:
```
[wordpress]
192.168.56.20 ansible_user=vagrant ansible_ssh_private_key_file=".vagrant/machines/wordpress/virtualbox/private_key"
```
___Obs.: está tudo na mesma linha.___

Assim, não precisamos mais informar o usuário e chave privada na linha de comando:
```
ansible-playbook provisioning.yml -i hosts
```

Podemos também adicionar vários logins no arquivo de hosts:
```
[wordpress]
192.168.56.20 ansible_user=vagrant ansible_ssh_private_key_file=".vagrant/machines/wordpress/virtualbox/private_key"
168.586.20.85 ---------
```
__________
### O que aprendemos?
* Como deixar nosso código mais enxuto com o uso de ```item``` e ```with_items```;
* Como configurar nosso projeto para lidar com chaves privadas diferentes.
_________
## Aula 4 - Configuração do banco de dados
Podemos consultar as informações de todos os módulos do Ansible a partir da documentação (https://docs.ansible.com/ansible/latest/collections/index_module.html). O código abaixo tem um exemplo de como criar um banco a partir do usuário root:

```
---
- hosts: all
  tasks:
    - name: 'Cria o banco MySQL'
      mysql_db:
        name: wordpress_db
        login_user: root
        state: present 
```

Link da documentação de todos os módulos de banco de dados: https://docs.ansible.com/ansible/2.6/modules/list_of_database_modules.html
_____________
### Criação de usuário no banco
O código abaixo cria um usuário no banco de dados:
```
---
- hosts: all
  tasks:
    - name: 'Cria usuário do MySql'
      mysql_user:
        login_user: root
        name: wordpress_user
        password: 12345
        priv: 'wordpress_db.*:ALL'
        state: present
```
_____________
### O que aprendemos?
* Como criar e deletar um banco de dados;
* Como criar e manipular usuários em nosso banco de dados.
_____________
## Aula 05 - Instalação do servidor e deploy da aplicação
_________
### Fazendo downloads e descompactando arquivos
Iremos instalar o Wordpress por meio da url do site oficial, para isso vamos usar o módulo ```get_url``` do Ansible. O código abaixo faz o download do arquivo e o salva no caminho especificado:

```
- name: 'Baixa o arquivo de instalação do Wordpress'
      get_url:
        url: 'https://wordpress.org/latest.tar.gz'
        dest: '/tmp/wordpress.tar.gz'
```

Após isso precisamos descompactar o arquivo, para tal usamos o módulo ```unarchive```:

```
- name: 'Descompacta o arquivo'
  unarchive:
    src: '/tmp/wordpress.tar.gz'
    dest: /var/www/
    remote_src: yes
  become: yes
```
_________
### Copiando e alterando arquivos 
Agora precisamos configurar o acesso do wordpress ao banco de dados, para isso iremos criar uma cópia do arquivo wp-config-sample.php para a mesma pasta. Para isso usamos o módulo ```copy ```:

```
- name: 'Cria uma cópia do arquivo de configuração do Wordpress'
  copy:
    src: '/var/www/wordpress/wp-config-sample.php'
    dest: '/var/www/wordpress/wp-config.php'
    remote_src: yes
  become: yes
```

Agora temos que alterar algumas constantes definidas no arquivo, para isso usamos o módulo ```replace```:

```
- name: 'Configura o wp-config com as entradas do banco de dados'
  replace:
    path: '/var/www/wordpress/wp-config.php'
    regexp: "{{ item.regex }}"
    replace: "{{ item.value }}"
  with_items:
    - {regex: 'database_name_here', value: 'wordpress_db'}
    - {regex: 'username_here', value: 'wordpress_user'}
    - {regex: 'password_here', value: '12345'}
  become: yes
```
_________
### Reiniciando serviços com Handlers
Algumas configurações são específicas para o estudo de caso do curso, não há necessidade de anotar, apenas entender os comandos.

O comando abaixo faz a cópia de arquivo do host para a VM:

```
- name: 'Configura Apache para servir o Wordpress'
  copy:
   src: 'files/000-default.conf'
   dest: '/etc/apache2/sites-available/000-default.conf'
  become: yes
```
Podemos criar tasks que serão chamadas apenas em momentos específicos do nosso código por meio de notificações, esse tipo de task é chamada de ```handler```. A handler abaixo restarta o serviço apache2:
```
- hosts: all
  handlers:
    - name: restart apache
      service:
        name: apache2
        state: restarted
      become: yes
```
Para chamarmos esse handler no nosso código usamos o comando ```notify```:
```
- name: 'Configura Apache para servir o Wordpress'
  copy:
   src: 'files/000-default.conf'
   dest: '/etc/apache2/sites-available/000-default.conf'
  become: yes
  notify:
    - restart apache
```
_________
### O que aprendemos?
* Como baixar e instalar o WordPress através do Ansible;
* Como configurar o WordPress para encontrar o banco de dados;
* Como preparar o Apache para encontrar o WordPress.
_________
## Aula 06 - Separando banco e aplicação
Podemos separar o provisionamento do Ansible para cada VM, para isso primeiro definimos a conexão no arquivo ```hosts```:
```
[wordpress]
192.168.56.20 ansible_user=vagrant ansible_ssh_private_key_file=".vagrant/machines/wordpress/virtualbox/private_key"

[database]
192.168.56.21 ansible_user=vagrant ansible_ssh_private_key_file=".vagrant/machines/mysql/virtualbox/private_key"
```
Após, separamos as tasks específicas para cada host no arquivo provisioning.yml:
```
---
- hosts: database
  tasks:
  # tasks aqui

- hosts: wordpress
  tasks:
  # tasks aqui
```
____________________
Agora precisamos mostar para o wordpress onde se encontra o banco de dados (que está em outra VM). Para isso adicionamos mais um parâmetro no regex:
```
- name: 'Configura o wp-config com as entradas do banco de dados'
  replace:
    path: '/var/www/wordpress/wp-config.php'
    regexp: "{{ item.regex }}"
    replace: "{{ item.value }}"
  with_items:
    - {regex: 'database_name_here', value: 'wordpress_db'}
    - {regex: 'username_here', value: 'wordpress_user'}
    - {regex: 'password_here', value: '12345'}
    - {regex: 'localhost', value: '192.168.56.21'} # parâmetro adicionado (ip da VM onde está instalado o MySQL)
  become: yes
```
Ainda há dois problemas, a instalação padrão do MySQL, não aceita duas coisas:
* não aceita a conexão de nenhum IP que não seja de localhost (127.0.0.1);
* os usuários que criamos no MySQL, por padrão, quando colocamos algum tipo de permissão, é apenas para permissão local.

Quando criarmos um usuário do Wordpress, adicionaremos uma lista de hosts possíveis de se conectarem com o MySQL, que é a própria máquina local. Ela será acessada por dois nomes: localhost e pelo IP 127.0.0.1. Incluiremos também o IP da máquina que roda Wordpress, para definir que esta tenha permissão de utilizar o usuário quando se conectar com MySQL:

```
- name: 'Cria o usuário do MySQL'
  mysql_user:
    login_user: root
    name: wordpress_user
    password:12345
    priv: 'wordpress_db.*:ALL'
    state: present
    host: "{{ item }}"
  with_items:
  - 'localhost'
  - '127.0.0.1'
  - '172.17.177.40' # ip da VM com o Wordpress
```
Agora precisamos alterar o arquivo de configuração do MySQL, que por padrão não aceita conexões remotas, usaremos uma técnica semelhante a que usamos para o Apache.

Segundo vamos copiar manualmente o conteúdo do arquivo ```/etc/mysql/my.cnf``` para o nosso host na pasta ```files/my.cnf```.

Então, dentro desse arquivo vamos buscar pelo parâmetro ```bind-address``` e setar seu valor para '0.0.0.0' para aceitar conexões de qualquer IP ```bind-address = 0.0.0.0```.

Agora temos que informar ao Ansible a necessidade de fazer a cópia do arquivo:
```
- name: 'Configura o MySql para aceitar conexões remotas'
  copy:
    src: 'files/my.cnf'
    dest: '/etc/mysql/my.cnf'
  become: yes
```

Por fim precisamos configurar um handles para restartar o MySQL toda vez que o arquivo for copiado:

```
# Handler
- hosts: database
  handlers:
    - name: restart mysql
      service: 
        name: mysql
        state: restart
      become: yes
  tasks: 

# Task
- name: 'Configura o MySql para aceitar conexões remotas'
  copy:
    src: 'files/my.cnf'
    dest: '/etc/mysql/my.cnf'
  become: yes
  notify: restart mysql
```

__Obs: devemos fazer alguma alteração no arquivo a ser copiado (adicionar um caracter qualquer por exemplo) apenas para que seja percebida uma mudança no arquivo e ele seja copiado novamente.__
________
Revisando os passos seguidos para fazer a página do Wordpress funcionar:

* Separamos o playbook em dois grupos de hosts, que usam as duas máquina virtuais criadas anteriormente;

* Colocamos no inventário do Ansible os IPs e como eles se conectam nessas máquinas;

* Separamos as tarefas do MySQL, de forma que a máquina do Wordpress fique sem comandos do MySQL;

* E a máquina do MySQL fique sem comandos do Wordpress;

* Incluímos a informação de como o Wordpress conecta no servidor remoto, mudando o replace;

* Além disso, incluímos o arquivo de configuração novo my.cnf, liberando o MySQL para aceitar conexões remotas.

* Demos permissão para o usuário criado para o Wordpress de aceitar conexões de servidores remotos colocando a lista de IPs.
__________
### O que aprendemos?
* Como separar o playbook em dois grupos de hosts:
  * Esses dois grupos usam as duas máquinas virtuais que criamos;
  * Inclusive, essas máquinas foram adicionadas ao inventário do Ansible.
* Como separar as tarefas entre os grupos;
* Como conectar o WordPress em um servidor remoto;
* Como liberar o MySQL para receber conexões remotas.
__________
## Aula 07 - Trabalhando com variáveis e templates
Para deixarmos nosso código reaproveitável podemos fazer uso de variáveis, elas devem ser declaradas em arquivos separados e referenciadas em nosso arquivo ```provisioning.yml```.

Vamos primeiramente declarar as variáveis que serão usadas em todos os hosts num arquivo chamado ```all.yml``` dentro da pasta ```group_vars```:

```
---
wp_username: wordpress_user
wp_db_name: wordpress_db
wp_user_password: '12345'
wp_installation_dir: '/var/www/wordpress'
```

Podemos também separar essas variáveis entre os hosts onde serão usadas, para isso colocamos num arquivo com o mesmo nome do grupo, também dentro da pasta ```group_vars```. Por exemplo, a variável abaixo será usada apenas no grupo ```database```:

Arquivo ```database.yml```
```
---
wp_host_ip: '192.168.56.20'
```

Agora dentro do nosso arquivo ```provisioning.yml```, fazemos a substituição de valores "chumbados" pelo nome das variáveis. O padrão é o seguinte ```"{{ nome_da_variavel }}"```, ou com aspas simples.

Exemplo:
```
- name: 'Cria usuário do MySql'
  mysql_user:
    login_user: root
    name: "{{ wp_username }}"
    password: "{{ wp_user_password }}"
    priv: "{{ wp_db_name }}.*:ALL"
    state: present
    host: "{{ item }}"
  with_items:
    - 'localhost'
    - '127.0.0.1'
    - "{{ wp_host_ip }}"
```
________
### Usando templates
Quando temos arquivos de configuração que são "padrão", usados como se fossem templates, podemos passar variáveis dentro deles como passamos nos demais arquivos usados pelo Ansible.

Por exemplo, o arquivo ```000-default-conf``` usado para mostrar ao Apache onde está a instalação do Wordpress, podemos passar o caminho por meio de uma variável que está definida em nossos arquivos da pasta ```group_vars```.

Para fazer isso usamos o comando ```template```, passando para ele o cominho onde está o template e o destino para onde o arquivo será substituído:
```
- name: 'Configura Apache para servir o Wordpress'
  template:
    src: 'templates/000-default.conf.j2'
    dest: '/etc/apache2/sites-available/000-default.conf'
  become: yes
  notify: restart apache
```
Devemos criar á pasta templates e __ATENÇÃO__ todos os templates devem ter a extensão ```.j2``` para funcionarem.

Para fins de curiosidade, o template que usamos como exemplo durante o curso é o seguinte:

```
<VirtualHost *:80>
	ServerAdmin webmaster@localhost
	DocumentRoot {{ wp_installation_dir }}

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
______
### O que aprendemos?

* Como deixar o script mais organizado e menos repetido com variáveis e templates;
* Como declarar as variáveis tanto na linha de comando quanto em arquivos do diretório ```group_vars```:
  * Cada grupo pode ter seu próprio arquivo com seus valores;
  * Se o Ansible não encontrar as variáveis nos arquivos do grupo, ele irá utilizar as do arquivo ```all.yml```.
* Como separar responsabilidades com templates.
______
## Aula 08 - Usando roles, dependências e defaults
Para podermos reaproveitar nosso código podemos separá-lo em módulos chamados no Ansible de 'roles'. O Ansible encontra as roles em nosso projeto por meio de uma pasta de mesmo nome. Dentro dessa pasta criamos outros diretórios relacionamos a cada tarefa que queremos fazer, no nosso exemplo podemos criar três: mysql, webserver e wordpress. Dentro desses diretórios adicionamos outros para cada tipo de tarefa, tais como: tasks, handlers, templates e files. Dentro de cada diretório onde haverá código, devemos colocar dentro de um arquivo chamado ```main.yml```. Abaixo um exemplo de role para o mysql:


Caminho do arquivo: ```roles/mysql/tasks/main.yml```
```
---
- name: 'Instala pacotes de dependências do sistema operacional'
  apt:
    name:
      - mysql-server-5.6
      - python-mysqldb
    state: latest
  become: yes
  
- name: 'Cria o banco MySQL'
  mysql_db:
    name: "{{ wp_db_name }}"
    login_user: root
    state: present

# restante do código 
```

Não precisamos dizer no código que trata-se de uma task, o Ansible já identifica isso pelo diretório em que o arquivo se encontra.

Agora no nosso arquivo ```provisioning.yml``` apenas chamamos a execução dessa role:

```
---
- hosts: database
  roles:
    - mysql
```

Pronto! Nosso código está modularizado.
__________
### Valores default para variáveis
Caso em nosso código existam variáveis que precisem ser passadas pelo usuário, podemos declarar valores padrão para elas para que caso o usuário não as informe, nosso código ainda assim funcione corretamente.

Dentro da pasta mysql, por exemplo, adicionamos uma pasta ```default``` e dentro dela o arquivo ```main.yml```. Dentro desse arquivo vamos adicionar valores padrão para os ips aceitos pelo mysql na variável ```wp_host_ip```:

```
---
wp_host_ip:
  - 'localhost'
  - '127.0.0.1'
```
Agora no nosso arquivo ```roles/mysql/tasks/main.yml``` deixamos apenas a variável e não precisamos nos preocupar caso o usuário não passe um valor, o código ainda assim funcionará:
```
- name: 'Cria usuário do MySql'
  mysql_user:
    login_user: root
    name: "{{ wp_username }}"
    password: "{{ wp_user_password }}"
    priv: "{{ wp_db_name }}.*:ALL"
    state: present  
    host: "{{ item }}"
  with_items:
    - "{{ wp_host_ip }}"
```
Caso o usuário deseje adicionar mais valores (os que não são default), ele pode fazer isso por meio do arquivo do grupo na pasta 'group_vars'.
________
### Dependências
Quando temos roles que dependem de outras para funcionar corretamente podemos deixar isso explicito por meio de arquivos. No nosso exemplo a role do wordpress só funcionará se executada após a role do webserver, para especificar isso, adicionamos dentro da pasta do ```roles/wordpress``` a pasta ```meta``` e dentro o arquivo ```main.yml```, com o seguinte conteúdo:

Caminho do arquivo: ```roles/wordpress/meta/main.yml```
```
---
dependencies:
  - webserver
```
Agora a role do wordpress só será executada após a role do webserver.
_________
### O que aprendemos?
* Como pensar em um código que facilite o reuso;
* Como deixar o script menos acoplado com roles;
* Como definir os handlers dentro dos diretórios das roles;
* Como determinar valores padrões para as roles.
_____
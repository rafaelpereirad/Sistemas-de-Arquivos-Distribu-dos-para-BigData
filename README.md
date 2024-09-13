# Sistema de Arquivos Distribuídos para Big Data

## Criação de instâncias na Oracle

Criar 4 instâncias: 1 Master e 3 Worker nodes

<img width="549" alt="image" src="https://github.com/user-attachments/assets/87ebddb9-194e-4efb-a4ef-2b2704386f1c">

<img width="605" alt="image" src="https://github.com/user-attachments/assets/2e7ad807-4f9c-4655-8a28-47a3e5dd2f38">

Geração de chave ssh no terminal para acesso nas máquinas e colocar a chave pública na criação da instância:

```
ssh-keygen -t rsa -b 2048
```

<img width="239" alt="image" src="https://github.com/user-attachments/assets/14eed280-98f3-47fc-9fdf-2bee4108aa7c">

<img width="603" alt="image" src="https://github.com/user-attachments/assets/2fbfce9f-8cf8-4d1d-9859-67885ae42151">

Repetir o processo de criação das máquinas para as outras intâncias atribuindo IPs privados 10.0.0.2-10.0.0.5:

<img width="1003" alt="image" src="https://github.com/user-attachments/assets/d08a722b-9c0d-43b5-9aad-526e3100bb21">


Editar na lista de segurança da sub-rede abrindo todas as portas das aplicações nas faixas de IPs privados das instâncias (10.0.0.0/24):

<img width="863" alt="image" src="https://github.com/user-attachments/assets/c4288c29-0388-4118-92b5-54cde4d2cf6f">

## Acesso as máquinas virtuais por ssh e instalação das ferramentas

ssh nas máquinas usando o IP público gerado nas instâncias: 

```
ssh -i /caminho/da/chave/privada/gerada/private_key opc@IP_publico_instâncias
```

<img width="822" alt="image" src="https://github.com/user-attachments/assets/cff64b0e-cafc-46a7-8b99-2149570e2da2">


Rodar em todas as instâncias:

```
sudo apt-get update
sudo apt-get install ssh
```

Gerar par de chaves pública/privada no nó mestre:

```
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat .ssh/id_rsa.pub >> ~/.ssh/authorized_keys
cat ~/.ssh/authorized_keys
```

Nos Worker1, Worker2 e Worker3:

```
nano .ssh/authorized_keys
```

Em todas as máquinas:

```
sudo nano /etc/hosts
```

Copiar os hostnames de todas as máquinas:

```
10.0.0.5      worker1
10.0.0.4      worker2
10.0.0.3      worker3
10.0.0.2      master
```

Rodar para instalar pacotes hadoop e jvm:

```
sudo apt-get -y install openjdk-8-jdk-headless
sudo wget -P ~ https://dlcdn.apache.org/hadoop/common/hadoop-3.4.0/hadoop-3.4.0-aarch64.tar.gz
tar xzf hadoop-3.4.0-aarch64.tar.gz
mv hadoop-3.4.0 hadoop
nano hadoop/etc/hadoop/hadoop-env.sh
```

Copiar:

```
export JAVA_HOME="/usr/lib/jvm/java-8-openjdk-arm64/
```

```
sudo mv hadoop /usr/local/hadoop
sudo nano /etc/environment
```

Copiar:

```
JAVA_HOME=”/usr/lib/jvm/java-8-openjdk-arm64/jre”
```

Editar em todas:

```
nano ~/.bashrc
```

<img width="398" alt="image" src="https://github.com/user-attachments/assets/9147615a-4e44-4adb-9a31-31abc5cb5907">

```
#Hadoop Related Options
export HADOOP_HOME="/usr/local/hadoop"
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"
```

Rodar:

```
source ~/.bashrc
```

Reiniciar todas as máquinas e reconectar. Então:

```
nano /usr/local/hadoop/etc/hadoop/core-site.xml
```
Editar:

```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://10.0.0.2:9000</value>
    </property>
</configuration>
```

```
nano /usr/local/hadoop/etc/hadoop/hdfs-site.xml
```

Editar:

```
configuration>
    <property>
        <name>dfs.replication</name>
        <value>4</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///usr/local/hadoop/hdfs/data</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///usr/local/hadoop/hdfs/data</value>
    </property>
</configuration>
```

```
nano /usr/local/hadoop/etc/hadoop/yarn-site.xml
```

Editar:

```
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>
    <property>
       <name>yarn.resourcemanager.hostname</name>
       <value>10.0.0.2</value>
    </property>
</configuration>
```

```
nano /usr/local/hadoop/etc/hadoop/mapred-site.xml
```

Editar:
```
<configuration>
    <property>
        <name>mapreduce.jobtracker.address</name>
        <value>10.0.0.2:54311</value>
    </property>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

No mestre:

```
sudo mkdir -p /usr/local/hadoop/hdfs/data
sudo chown ubuntu:ubuntu -R /usr/local/hadoop/hdfs/data
chmod 700 /usr/local/hadoop/hdfs/data
```

Em todas:

```
nano /usr/local/hadoop/etc/hadoop/masters
```

Adicionar o IP privado do mestre:

```
10.0.0.2
```

```
nano /usr/local/hadoop/etc/hadoop/workers
```

Adicionar IP privado dos Workers:

```
10.0.0.3      
10.0.0.4      
10.0.0.5
```       

No mestre:

```
hdfs namenode -format
```

<img width="857" alt="image" src="https://github.com/user-attachments/assets/5c1d0d54-1e68-4840-8c85-ff20385a3807">

Rodar:

```
start-dfs.sh
```

Rodar comandos para instalar habilitar portas para a aplicação no firewall:

```
sudo apt install firewalld
sudo systemctl enable firewalld
sudo systemctl start firewalld
sudo firewall-cmd --permanent --add-port=9870/tcp
sudo firewall-cmd --permanent --add-port=9000/tcp
sudo firewall-cmd --permanent --add-port=8088/tcp
sudo firewall-cmd --permanent --add-port=50075/tcp
sudo firewall-cmd --reload
```

Em todas as máquinas:

```
start-all.sh 
```

Acessar http://IP_publico_mestre:50070 

<img width="977" alt="image" src="https://github.com/user-attachments/assets/aaba2fe1-c32a-46a0-ac5a-1dc6e3efdff6">

Assim, é possível verificar o Sistema de arquivos distribuídos reconhecendo os 3 DataNodes (Worker nodes), os quais armazenam o sistema de arquivos distribuídos HDFS de forma invisível para o cliente da aplicação

Desse modo, ao inserir dados no hdfs é possível armazenar de forma distribuída os dados nesse sistema de arquivos distribuídos sem se preocupar em qual dos nós está sendo armazenado os dados

O Master node (ou Data Node) faz o gerenciamento e replicação dos dados nos Worker nodes de forma distribuída.

Rodar no nó mestre para carregar e consultar arquivos no HDFS:

```
wget -O test.csv "http://sinca.mma.gob.cl/cgi-bin/APUB-MMA/apub.tsindico2.cgi?outtype=xcl&macro=./RII/237/Cal/PM25//PM25.diario.diario.ic&from=13060100&to=15110323&path=/usr/airviro/data/CONAMA/&lang=esp&rsrc=&macropath="
hdfs dfs -mkdir /user/local
hdfs dfs  -put /home/opc/test.csv  /user/test/test.csv  
sudo chmod 644 /home/opc/test.csv
hdfs dfs -ls
```

<img width="468" alt="image" src="https://github.com/user-attachments/assets/a907fa0a-9c88-414d-b106-b6b447103deb">

Instalação do Hive:

No nó mestre:

```
wget https://dlcdn.apache.org/hive/hive-4.0.0/apache-hive-4.0.0-bin.tar.gz
tar xzf apache-hive-4.0.0-bin.tar.gz 
mv apache-hive-4.0.0-bin hive
sudo mv hive /usr/local/hive
nano ~/.bashrc
```
Adicionar:

```
#Hive
export HIVE_HOME="/usr/local/hive"                
export HIVE_CONF_DIR="/usr/local/hive/conf"
export PATH=$HIVE_HOME/bin:$PATH
export CLASSPATH=$CLASSPATH:/usr/local/hadoop/lib/*:.
export CLASSPATH=$CLASSPATH:/usr/local/hive/lib/*:.
```

Rodar:

```
source ~/.bashrc
hdfs dfs -mkdir -p /user/hive/warehouse
hdfs dfs -mkdir -p /tmp
hdfs dfs -chmod g+w /tmp
hdfs dfs -chmod g+w /user/hive/warehouse
cd $HIVE_HOME/conf
sudo cp hive-env.sh.template hive-env.sh
sudo nano hive-env.sh
```

Adicionar:

```
export HADOOP_INSTALL="/usr/local/hadoop"
```

```
sudo nano /var/lib/pgsql/data/postgresql.conf
```

Adicionar:

```
   Listen_addresses = ‘*’
   port = 5432
```

```
sudo nano /var/lib/pgsql/data/pg_hba.conf
```

Acionar:

```
      host    all   hive              10.0.0.7/32         trust
```

```
sudo -u postgres psql -U postgres
```

Rodar:

```
CREATE DATABASE hive;
CREATE USER hive WITH PASSWORD 'hive';
GRANT ALL PRIVILEGES ON DATABASE hive TO hive;
```

Rodar:

```
sudo yum install postgresql-jdbc.noarch -y
```

Para acessar hive que interage com hdfs, rodar no mestre:

```
 beeline -u "jdbc:hive2://master:10000/default" -n hive -p hive 
```

Exemplo de consultas buscando dados armazenados anteriormente no sistema de arquivos distribuído:

<img width="564" alt="image" src="https://github.com/user-attachments/assets/2c72144c-8d68-4b7a-b519-0c14d65251e4">




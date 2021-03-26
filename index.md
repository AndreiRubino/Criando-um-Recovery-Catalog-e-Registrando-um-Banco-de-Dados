# RMAN – Criando um Recovery Catalog e Registrando um Banco de Dados

Este artigo tem como objetivo descrever como efetuar a criação de um recovery catalog e após isso, como registrar um banco de dados nele. As formas abaixo são úteis no dia a dia de um dba.

O primeiro passo é efetuar a conexão em seu banco de dados escolhido para armazenar o recovery catalog, em meu caso, este é o rcatpdb (pluggable database).

```
alter session set container = rcatpdb;
```
--imagem 1


Conectado com sucesso iremos criar uma tablespace própria para o catalog.
```
create tablespace tbs_catalog datafile '/u01/app/oracle/oradata/catalog01.dbf' size 100m autoextend on next 10m maxsize 1024m;
```
-- imagem 2


Agora precisarmos criar um owner para o recovery catalog e fornecer os privilégios necessários.
```
create user rman identified by rman temporary tablespace temp default tablespace tbs_catalog quota unlimited on tbs_catalog;
grant connect, resource, recovery_catalog_owner to rman;
```
-- imagem 3


Após a criação do usuário owner devemos inicar o RMAN e se conectar no recovery catalog com seu owner (em meu caso, usuário rman).

Com o comando CREATE CATALOG iremos criar as tabelas do catalog, observe que as tabelas do Catalog são criadas na tablespace default do usuário owner(rman). Se você deseja criar em outra tablespace, deve se usar o comando CREATE CATALOG TABLESPACE NOME_TABLESPACE;

A criação do catalog pode levar alguns minutos.
```
rman catalog rman/rman@rcatpdb
```

```
create catalog;
```
-- imagem 4


**Nessa segunda parte do tutorial irei mostrar como registrar um banco de dados no Recovery Catalog que acabou de ser criado.**

Registrar um banco de dados no recovery catalog nada mais é do que dar permissão ao RMAN para armazenar os metadados deste banco de dados no recovery catalog. Sabendo disso, iremos registrar nosso banco de dados server01 no recovery catalog.

Antes de registrar, irei testar a conexão com o server01 utilizando o comando tnsping.
```
tnsping server01
```
-- imagem 5


Agora que temos certeza que o server01 está ON vamos iniciar o RMAN e conectar nosso target database ao recovery catalog para registrar o banco de dados no catalog.
```
rman target sys/oracle@server01 catalog rman/rman@rcatpdb
```
-- imagem 6


Nesse momento ambos os banco de dados estão conectados e estamos pronto para registrá-lo. Quando um banco de dados é registrado no Recovery Catalog é feito uma sincronização completa dele.

Esse processo atualiza o Catalog com:

Data file e Archived Redo Log backup sets e backup pieces
Cópias Data file
Archived redo logs e suas cópias
Estrutura do banco de dados
Stored Scripts (comandos criadas pelo usuário RMAN)
Agora basta executar o comando REGISTER DATABASE;
```
register database;
```
-- imagem 7


Pronto ! Seu banco de dados já está registrado no recovery catalog, e para verificar se o registro foi realmente concluído, podemos usar o comando list db_unique_name all.
```
list db_unique_name all;
```
-- imagem 8


Com o comando report schema também podemos listar todos os datafiles e tablespaces do target database no repository do RMAN.
```
report schema;
```
-- imagem 9


Pronto ! Podemos observar que o target database (server01) foi registrado com sucesso no recovery catalog.


Referência
https://docs.oracle.com/database/121/BRADV/rcmcatdb.htm#BRADV11001
https://docs.oracle.com/cd/E25178_01/backup.1111/e10642/rcmcatdb.htm

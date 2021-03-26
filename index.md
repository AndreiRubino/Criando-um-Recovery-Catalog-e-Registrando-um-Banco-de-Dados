# RMAN – Criando um Recovery Catalog e Registrando um Banco de Dados

Este artigo tem como objetivo descrever como efetuar a criação de um recovery catalog e após isso, como registrar um banco de dados nele. As formas abaixo são úteis no dia a dia de um dba.

O primeiro passo é efetuar a conexão em seu banco de dados escolhido para armazenar o recovery catalog, em meu caso, este é o rcatpdb (pluggable database).

```
alter session set container = rcatpdb;
```
![Image](https://bn1301files.storage.live.com/y4mw0rV_qsC4Y9my0ygCPdduF_ftmP6i54LG-jcY3yA0u4TfnqFXRrOw_94k2t9FAzQHIX6lOKaoL0JSanjZ7aFTU0ptiXj-A5ttiESAwKd4IR2QRZe3mb-PBls4JjdukHbFfX_SRLSAMpoHLCkgxo2rjTz6CkJH7129mzT5GoqRrc4UKFzRMvlXwI8Xv265sKV?width=726&height=362&cropmode=none)


Conectado com sucesso iremos criar uma tablespace própria para o catalog.
```
create tablespace tbs_catalog datafile '/u01/app/oracle/oradata/catalog01.dbf' size 100m autoextend on next 10m maxsize 1024m;
```
![Image](https://bn1301files.storage.live.com/y4mKFrJduz4qM-AG1HJinQYaATcrHyhLZpxmF3HUO7yXUZs7RlojrgLlC7OChvkEIoIX8Mbyr6TT5ikLy7qkyMSaFlZ4VPj0tFzBR1AWjrY-uKXYWunP8mmZ4VehrxTbj9V_YDbXAYTqup3r0UZzvK5d5bIZSdfUC-WqBsxb3zs42_t1UzrHJq1aGKOJsXvu4cM?width=1023&height=70&cropmode=none)


Agora precisarmos criar um owner para o recovery catalog e fornecer os privilégios necessários.
```
create user rman identified by rman temporary tablespace temp default tablespace tbs_catalog quota unlimited on tbs_catalog;
grant connect, resource, recovery_catalog_owner to rman;
```
![Image](https://bn1301files.storage.live.com/y4mbJ9kYy9rWGZi5z-9vzQJxGOdwE5707maFao_uS-EXM8eNp5ZhSORWIAdaJi4HnnUgbzHftm1RsuxohsMPwkZaXBxxv7kDxHxKOTTywMWi557vtbNTdvCSv_SFka8UxVsnd89a1TptrbnMUMqyhiyR6BP8Etg3nkBl8WISwxGyVDz3sUWzyKBjjDI_IZt4JuT?width=1024&height=135&cropmode=none)


Após a criação do usuário owner devemos inicar o RMAN e se conectar no recovery catalog com seu owner (em meu caso, usuário rman).

Com o comando CREATE CATALOG iremos criar as tabelas do catalog, observe que as tabelas do Catalog são criadas na tablespace default do usuário owner(rman). Se você deseja criar em outra tablespace, deve se usar o comando CREATE CATALOG TABLESPACE NOME_TABLESPACE;

A criação do catalog pode levar alguns minutos.
```
rman catalog rman/rman@rcatpdb
```

```
create catalog;
```
![Image](https://bn1301files.storage.live.com/y4me4-rC5o0Co0Gym1RyYYtbwYqANKOepJtw2LHOlZ2wbz7xyjY0IK6zCfPIbMTI1CqMxRVYsF08hIc1xDnE64e_feZjLUskRmUi7ohSwTRR_ZenXI-ykaogFlExEmA6gTilh4s3VW4SUhFCD11ZUlRfj3pBbau6pPB02_cVoPvx9PseFx0dxqLQTwwJQrauvNM?width=721&height=288&cropmode=none)


**Nessa segunda parte do tutorial irei mostrar como registrar um banco de dados no Recovery Catalog que acabou de ser criado.**

Registrar um banco de dados no recovery catalog nada mais é do que dar permissão ao RMAN para armazenar os metadados deste banco de dados no recovery catalog. Sabendo disso, iremos registrar nosso banco de dados server01 no recovery catalog.

Antes de registrar, irei testar a conexão com o server01 utilizando o comando tnsping.
```
tnsping server01
```
![Image](https://bn1301files.storage.live.com/y4mLZs7WGDSnWlGeStJ_tVyFnzlLZESD4F9xxVvU4bJraLED7P8Z0bWojzYHwy1XRHFVmsNFr-FEmwOtPzlUXauGPOa8dRkOtuShQDyrGR8gW4006Nb6-A2Ox2D_UFn0cjDLKES6qNYTqJBG9iJBBgSQg-shfH_90hAwQ-B5JoPxop2aLcIfQtaRrem5R92ZOqy?width=1024&height=265&cropmode=none)


Agora que temos certeza que o server01 está ON vamos iniciar o RMAN e conectar nosso target database ao recovery catalog para registrar o banco de dados no catalog.
```
rman target sys/oracle@server01 catalog rman/rman@rcatpdb
```
![Image](https://bn1301files.storage.live.com/y4mIpOxdHOKvHeivG1G4L2hhPMYUQcLvFsLCPfBQLS7qVBzTT8zG9_BCGX_uiYENTfaQfmuJhLxw4T7X3GCtKFdrKMUyFuDDm4_QDbhDMVeAwbqp7jaA2gh-Ia3VeEh0DMuvlAZw5c4-Vv3O9GRe-bCPU3qUMjL2IG6-imlc7EMlrzyCy5ThjlxxbJbSruX2Xt1?width=742&height=309&cropmode=none)


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
![Image](https://bn1301files.storage.live.com/y4mFrTJ_GfpIq2a9HtAGapQ6U26kKztADE5svySoiw8K-xm7hZiS8SGc6z-lqgcGY18gIPs5saYmApMMbki5W6HEIpykYUHr_Xf-rI3NbyGYyTHhfOu9fhEFOqS3zVPkvzbCYYPXJ9vEe8of5iIyQE-b-HoXhbnlKGjANVA5yYzVvWBSid2HH1WjAL_mi2ewJju?width=706&height=115&cropmode=none)


Pronto ! Seu banco de dados já está registrado no recovery catalog, e para verificar se o registro foi realmente concluído, podemos usar o comando list db_unique_name all.
```
list db_unique_name all;
```
![Image](https://bn1301files.storage.live.com/y4mCrnFTfT8gLR8xWLfCnfuQX_z9tE8vCquhGQqu02Wxk1tXRKZ0t2qXK7WZF_0tmzqy7dUxqGequODGAmTnRXorej9GQgluntg-uP64_bWjadl7Txpgq8kB7H47YsN0XrfXciCJBEkv2lf1-aLqcuLDRKyy3QKQ2Xu5He7i6Xv0VSWRhTeDt0Dd3DDvUkJLSxf?width=718&height=266&cropmode=none)


Com o comando report schema também podemos listar todos os datafiles e tablespaces do target database no repository do RMAN.
```
report schema;
```
![Image](https://bn1301files.storage.live.com/y4msZcf4HNkC-v2rdjMrEDxm94u7upu7heRnSK6mbXdItpqu_yWeVFxsBoz9gFkmogRx_vKGLBmIefVGnndN1VTTcdKAqJ4jqDJQKN-W0lERfIO-0_K7UBEkqJv9hyAghzgw7TYJOz7Jt1nBtwGztU_oLgVQ45cmK_8Lpvu9Qkcs7XwfwOUp0RNML1GJCYXlffM?width=1024&height=577&cropmode=none)


Pronto ! Podemos observar que o target database (server01) foi registrado com sucesso no recovery catalog.


Referência
- [Database Backup and Recovery User's Guide](https://docs.oracle.com/database/121/BRADV/rcmcatdb.htm#BRADV11001)
- [Oracle® Database Backup and Recovery User's Guide](https://docs.oracle.com/cd/E25178_01/backup.1111/e10642/rcmcatdb.htm)

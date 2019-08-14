# Postgresql

## 内容

dockerでpostgresqlのコンテナを利用してDBを作成した際の**備忘録**

### やったこと

- サンプルデータベースを作成する
- 個人用dockerイメージ作成
    - DockerHub からダウンロードした PostgreSQL のイメージから作成しても var/lib/postgresql/data がコピーされないらしい
    - Dockerfile を作成してコンテナ化したものからイメージを作成する必要がある。（未実施）

## 手順

### Postgresqlコンテナ起動

- DockerHubのイメージを利用
    ```bash
    docker container run -d --name postgres -e POSTGRES_PASSWORD=postgres -p 5432:5432 postgres
    ```
- 起動確認
    ```bash
    docker container ls
    ```

### DB作成

- コンテナ内に入る
    ```bash
    docker exec -ti postgres /bin/bash
    ```
    実行後、ユーザ名が以下のように root に変わる
    ```bash
    root@5b5a03a2e869:/#
    ```
- 現在のDB確認
    ```bash
    su - postgres   #実行ユーザがpostgresに変わる
    psql -l         #DB一覧を表示
    ```
    以下が表示される
    ```bash
                                     List of databases
       Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges
    -----------+----------+----------+------------+------------+-----------------------
     postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
     template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
               |          |          |            |            | postgres=CTc/postgres
     template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
               |          |          |            |            |  postgres=CTc/postgres
    (3 rows)
    ```
- 作成するDBファイルをコンテナにコピー
PostgreSQLの公式チュートリアルにあるサンプルDBを利用する
    ```bash
    exit    #postgreユーザ終了
    exit    #rootユーザ終了
    cd [サンプルDBダウンロードディレクトリ]  #ディレクトリ移動
    docker cp dvdrental postgres:/home  #postgresコンテナの/homeディレクトリにコピー
    # postgreユーザに読み取り権限があるフォルダにコピーするべき
    ```
- ファイル確認
    ```bash
    docker exec -ti postgres bash
    ls /home
    ```
- DB仮作成
    ```bash
    su - postgres
    createdb dvdrental  #DB作成
    psql -l
    exit
    ```
- sql実行
    ```bash
    psql -U postgres dvdrental < /home/dvdrental/restore.sql
    ```
- 確認
    ```bash
    su - postgres   #postgresユーザに変更
    psql
    \c dvdretan     #接続DBをdvdrentalに変更
    \d
    ```
    以下が表示されていれば成功
    ```bash
                         List of relations
     Schema |            Name            |   Type   |  Owner
    --------+----------------------------+----------+----------
     public | actor                      | table    |  postgres
     public | actor_actor_id_seq         | sequence |  postgres
     public | actor_info                 | view     |  postgres
     public | address                    | table    |  postgres
     public | address_address_id_seq     | sequence |  postgres
     public | category                   | table    |  postgres
     public | category_category_id_seq   | sequence |  postgres
     public | city                       | table    |  postgres
     public | city_city_id_seq           | sequence |  postgres
     public | country                    | table    |  postgres
     public | country_country_id_seq     | sequence |  postgres
     public | customer                   | table    |  postgres
     public | customer_customer_id_seq   | sequence |  postgres
     public | customer_list              | view     |  postgres
     public | film                       | table    |  postgres
     public | film_actor                 | table    |  postgres
     public | film_category              | table    |  postgres
     public | film_film_id_seq           | sequence |  postgres
     public | film_list                  | view     |  postgres
     public | inventory                  | table    |  postgres
     public | inventory_inventory_id_seq | sequence |  postgres
     public | language                   | table    |  postgres
     public | language_language_id_seq   | sequence |  postgres
     public | nicer_but_slower_film_list | view     |  postgres
     public | payment                    | table    |  postgres
     public | payment_payment_id_seq     | sequence |  postgres
     public | rental                     | table    |  postgres
     public | rental_rental_id_seq       | sequence |  postgres
     public | sales_by_film_category     | view     |  postgres
     public | sales_by_store             | view     |  postgres
     public | staff                      | table    |  postgres
     public | staff_list                 | view     |  postgres
     public | staff_staff_id_seq         | sequence |  postgres
     public | store                      | table    |  postgres
     public | store_store_id_seq         | sequence |  postgres
     (35 rows)
    ```

### イメージ作成

- 現在のイメージ確認
    ```bash
    docker image ls
    ```
    以下が表示される
    （ダウンロードしたイメージのみ）
    ```bash
    REPOSITORY          TAG                    IMAGE ID            CREATED             SIZE
    postgres            latest                 6dc0e6af942c        a days ago          312MB
    ```
- 現在のコンテナ確認
    ```bash
    docker container ls -a
    ```
    以下が表示される
    ```bash
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
    5b5a03a2e869        postgres            "docker-entrypoint.s…"   25 hours ago        Up 25 hours         0.0.0.0:5432->5432/tcp   postgres
    ```
- イメージ化
    ```bash
    docker container commit 5b5a03a2e869 postgres:dvdrental
    ```
    作成されたことを確認
    ```bash
    docker image ls
    ```
    ```bash
    REPOSITORY          TAG                    IMAGE ID            CREATED             SIZE
    postgres            dvdrental              ea1cbad79dfc        7 minutes ago       315MB
    ```

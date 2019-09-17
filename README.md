# dockerお試し備忘録

## コンテナ起動エラー

### ログ確認

- docker logs { コンテナID }

## コンテナ接続

- docker exec -ti postgres /bin/bash

## コンテナ離脱

- ctrl+p,ctrl+q

## コンテナにファイルコピー

- docker cp {ORIGIN_PATH}/fileName {CONTAINER_ID}:{COPY_PATH}
  - docker cp ./xxx.sql xxxxxxxxx:/init
  - docker cp ./xxx/. xxxxxxxxx:/init

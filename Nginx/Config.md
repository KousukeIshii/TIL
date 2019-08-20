# Nginxのコンフィグ
## root
	サーバーの公開ディレクトリの設定
## index
	公開されたディレクトリにアクセスしたときに返されるファイル
	左から順に最初に見つかったものが返される。
	デフォルトはindex.html
## location
	アクセスされたURLパラメータによって処理内容を変えられる。
	プレフィックスがあってそれによりlocationのマッチ基準が異なる。
	おおまかにいうとプレフィックスなしのものなどよりは正規表現を用いたlocationのほうがつよく、
	＝などのプレフィックスがあるとそれが一番強い

## try_files
	ファイルもしくはディレクトリを探す処理
	課題２ではtry_files $uri $uri/ /index.php$is_args$args;
	となっていたがこれはつまり
	$uri というファイルを探す　→　（なければ）　$uriというディレクトリを探す　→　（それもなければ）index.phpをargs付きで返す。
	$is_argsはargsがあれば?になるだけ	

## fastcgi
	動的にページなどを生成してクライアントに返すための仕組み。
	サーバーから他のプログラムを呼び出して処理結果をクライアントに返す。

### 使うディレクティブ
### fastcgi_pass
	FastCGIが使うポートを書く
	課題２で使ったconfigでは
	app:9000
	だった。
	→Dockerではcopmoseにかいたサービス名がipアドレス欄に入る？

### fastcgi_split_path_info
	課題２では
	fastcgi_split_path_info ^(.+\.php)(/.+)$;
	であった。
	このディレクティブにより前半の正規表現は$fastcgi_script_nameの値になり
	後半の正規表現は$fastcgi_path_infoの値になる。
	意味は".php"までの文字列と".php"より後ろの"/"から始まる文字列を取り出す。

### fastcgi_param
	FastCGIにわたす情報を記述する。
	include fastcgi_params;
	で/etc/nginx/fastcgi_paramsに記述されているデフォルト設定を読み込んで
	その後に変更が必要な値については別個更新している。

### fastcgi_index
	indexディレクティブは上で書いたようにディレクトリにアクセスされたときに返すファイルである。
	つまりは、var/www/　などのようなパターンのときにこのファイルが返されるわけである。
	でもこのパターンではそもそも課題２のように"location /"があるとマッチされない。

	https://stackoverflow.com/questions/30802025/what-is-fastcgi-index-in-nginx-used-for
	上の質問によるとlocationの外にあるindexディレクティブによりindex.phpにアクセスがリダイレクトされることで
	phpスクリプトのlocationへのアクセスになるらしい。
	でもこの場合でも結局fastcgi_indexは参照されない。
	結局サーバーがリモートにあり、index.phpが見えないときに使うらしい（？）

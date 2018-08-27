# Rubyを汎用コマンドラインユーティリティに変えるthisredone/rb
thisredone/rbはたったの9行でコマンドライン上にRuby関数を組み込むライブラリ。面白そうなので読んでみる。

https://github.com/thisredone/rb


## コード全体
*README.md*



rbスクリプト全体。後は1行ずつ読んでいく。

```ruby
#!/usr/bin/env ruby
File.join(Dir.home, '.rbrc').tap { |f| load f if File.exists?(f) }

def execute(_, code)
  puts _.instance_eval(&code)
rescue Errno::EPIPE
  exit
end

single_line = ARGV.delete('-l')
code = eval("Proc.new { #{ARGV.join(' ')} }")
single_line ? STDIN.each { |l| execute(l.chomp, code) } : execute(STDIN.each_line, code)
```

## .rbrcファイルの読み込み
*rb*



`~/.rbrc`ファイルかあればloadする。.rbrcによく使う関数などを書いておくことでコマンドとして使うことができる。  
`tap`を使うことでローカル変数fへの`=`を省略して1文で書いているのが面白い。

```ruby
File.join(Dir.home, '.rbrc').tap { |f| load f if File.exists?(f) }

```

## execute関数
*rb*



肝となる関数。[instance_eval](https://docs.ruby-lang.org/ja/latest/method/BasicObject/i/instance_eval.html)はオブジェクトのコンテキストで文字列やブロックを実行するもの。
- 第1引数`_`: selfとなるオブジェクト
- 第2引数`code`: instance_evalされるブロック(&がついているのでブロック確定)

```ruby
def execute(_, code)
  puts _.instance_eval(&code)
rescue Errno::EPIPE
  exit
end

```

## オプション処理
*rb*



'-l'がrbコマンドの引数に渡されていたら`single_line != nil`になる。

```ruby
single_line = ARGV.delete('-l')
```

## コードブロックの生成
*rb*



ARGVをProc.newで囲んだものをevalする、つまりARGVを実行するブロックが作られる。
code(ブロック)をexecute関数に渡して実行することでレシーバーを省略して呼べるようになる。つまりexecute関数の第1引数がEnumratorだったら`self.drop(1)`は`drop 1`で呼べるようになる！

```ruby
code = eval("Proc.new { #{ARGV.join(' ')} }")
```

## 実行
*rb*



single_lineモードのときはSTDINをStringに分解して実行、通常時はSTDINのEnumeratorが渡されて実行される。

```ruby
single_line ? STDIN.each { |l| execute(l.chomp, code) } : execute(STDIN.each_line, code)
```


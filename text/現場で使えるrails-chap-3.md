# アプリの作成
## モデル設計時に考えること。
属性の意味、カラム名、データ型、nullを許容するか、デフォルト値を設定するかなど。
## t.timestamps
打刻用のカラムが自動生成される。
## Restful
- リクエストを処理するコントローラーとアクションは、ブラウザからのリクエストに含まれるURLとHTTPもメソッドによって決定される。
- コントローラーのアクションを設計する時には、入り口となるURL、HTTPメソッドを合わせて考える必要がある。
## コントローラを作成する時
HTTPメソッドがGETのアクションは同名のビューを使うことが多いので、ジェネレーターコマンドでアクション名を指定した方が便利。
## link_toヘルパーメソッドの引数
link_to 'キャプション',リンク先のURL,a要素につける属性（classなど）
## アクションの役割
アクションからビューに受け渡したい値をインスタンス変数に入れる事がアクションの基本的な役割
## リクエストパラメータ
リクエスト含まれたデータのこと。POSTとGETで送る事ができ、アクション側で受け取る事ができる。送られてきたURL中の、route.rbで定義された可変部分もリクエストパラメーターとして利用できる。route.rbにtasks/:idが定義されている時、tasks/7の7はアクション側でparams[:id]で受け取る事ができる。
## form_with(フォームのヘルパーメソッド)
form要素を作成するためのメソッド。引数に@モデル名を渡して対応づける。また、ブロック引数のfに対してlabelやtext_fieldなどのメソッドを呼び出すと、モデルオブジェクトに対応したフィールドを出力できる
```
= form_with model: @task,lacal:true do |f|
```
```
    = f.label :name
    = f.text_field :name,class:'form-control',id:'task_name'
    
    ↓

　　　<label for="task_name">名称</label>
　　　<input class="form-control" id="task_name" type="text" name="task[name]">
```

## ストロングパラメータ
```
  def task_params
    params.require(:task).permit(:name,:description)
  end
```
これでセキュリティーを強化できる。安全化。
## レンダーとリダイレクト
アクションに続けてビューを表示するのはレンダー。リダイレクトはもう一度ブラウザがリクエストし直して?(同じリクエスト？違うリクエスト？)、画面を表示させる。
## フラッシュメッセージ
redirect_toは特定のflashメッセージを渡す事ができる。セッションを利用している。
```
    redirect_to tasks_url,notice:"タスク「#{task.name}」を登録しました。"

   ↓ #同じ意味

　　flash[:notice] = "タスク「#{task.name}」を登録しました"
   redirect_to tasks_url
```
flash[:notice]にタスク完了のメッセージをセットしてからリダイレクトすると言う意味。
## フラッシュのキー設定
デフォルトで:notice、:alterが入っている。flash:の値にハッシュとして渡せばどの様なキーのデータも渡せる。
```
   redirect_to tasks_url,flash:{warning:"何かの警告メッセージ"}
```
noticeとalertはメソッドの形でもデータを渡せる
```
   flash.notice = "タスク「#{task.name}」を登録しました"
```
## フラッシュの役割
フラッシュは主に次のリクエストに対してデータを伝える仕組み。しかし同じリクエスト内で（直後にレンダーされるビュー）伝えることもできる。
```
   flash.now.alert = "登録できません"
```
2つ以上先のリクエストにデータを渡す`flash.keep`もある
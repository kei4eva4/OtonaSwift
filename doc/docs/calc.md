# チュートリアル①電卓アプリ

## プロジェクトの新規作成
- Xcodeを起動して「Create a new Xcode project」または、メニューから［File］→［New］→［Project］を選択し、新規プロジェクトを作成
- ［iOS］の［Single View Application］を選択して［Next］で進む
![/images/calc/calc1.jpg](/images/calc/calc1.jpg)

- 各項目を入力し［Next］で進む
- ［Organization Name］は任意の名前
- ［Organization Identifier］には、持っているドメイン等を入力する。（これはアプリの識別IDの一部となるため重要で、必須入力です。通常は、所有しているドメイン名を逆順にして入力します）
![/images/calc/calc2.jpg](/images/calc/calc2.jpg)

- プロジェクトを保存する任意のフォルダを選択
- [Create]で保存
![/images/calc/calc3.jpg](/images/calc/calc3.jpg)

## CocoaPodsでライブラリをインストール

- 一度Xcodeを終了し、ターミナルを起動して下記コマンドにてCocoapodsをインストールする

```
$ sudo gem install cocoapods
$ pod setup
$ pod --version
```

- CocoaPodsのインストールが完了したら、先ほど作成したプロジェクトのルートに移動してPodfileを作成する

```
$ cd /path/to/XcodeProject/CalculatorApp
$ pod init
```

- Podfileを下記のように編集する
- 「Expression」は、電卓に入力された文字列をSwiftコードとして評価するために使う 

```
platform :ios, '9.0'

target 'CalculatorApp' do
  use_frameworks!
  pod 'Expression'
end
```

![/images/calc/calc4.jpg](/images/calc/calc4.jpg)

- インストールが成功したらCalculatorApp.xcworkspaceが作成されるので、これをXcodeプロジェクトとして開く

```
$ open CalculatorApp.xcworkspace
```

## 起動確認

- デバイスを選択し、Runする

![/images/calc/calc5.jpg](/images/calc/calc5.jpg)

## UIの作成
- Main.storyboardを開く
- Library一覧から[Label]を選択し、ViewControllerの上にドラッグアンドドロップする

![/images/calc/calc6.jpg](/images/calc/calc6.jpg)

- Atrribute Inspectorを開き、ラベルのテキストを編集する

![/images/calc/calc7.jpg](/images/calc/calc7.jpg)

- 同じ手順で「=」と「答え用のラベル」も追加する

![/images/calc/calc8.jpg](/images/calc/calc8.jpg)

- Library一覧から[Button]を選択し、ViewControllerの上にドラッグアンドドロップする

![/images/calc/calc9.jpg](/images/calc/calc9.jpg)

- Atrribute Inspectorを開き、ボタンのテキストと背景色を編集する

![/images/calc/calc10.jpg](/images/calc/calc10.jpg)

- 同じ手順で他のボタンも追加する

![/images/calc/calc11.jpg](/images/calc/calc11.jpg)

- [Run]して、ボタントラベルが正しく表示されていたらUIはこれで完了

![/images/calc/calc12.jpg](/images/calc/calc12.jpg)

## Outlet接続
- Outlet接続とは、Storyboard上のオブジェクト（部品）を、ソースコード上の変数として使えるようにする仕組み

- Main.storyboardでViewControllerを選択、Assitant editorを開く
![/images/calc/calc13.jpg](/images/calc/calc13.jpg)

- 「式用のラベル」を選択肢、Cotrolキーを押しながら、Assistant editorで開いたView Controllerのソースコード上にドラッグアンドドロップする
![/images/calc/calc14.jpg](/images/calc/calc14.jpg)

- 同様に答え用のラベルも接続する
![/images/calc/calc15.jpg](/images/calc/calc15.jpg)

## Action接続
- 数字の0のボタンをControlを押しながらAssistant editorにドラッグ＆ドロップした後、次の画像のように［Connection］はAction、［Name］はinputFormula、［Type］にはUIButtonと入力して、［Connect］する
![/images/calc/calc16.jpg](/images/calc/calc16.jpg)

- 同様の手順を繰り返して、［C］ボタンと［=］ボタンを除くすべてのボタンで、Action接続を作成し、［Name］は、すべて同じinputFormulaとする
- ソースコード上に同名のinputFormulaアクションが追加さるが、1つだけ残して後は削除する。［C］と［=］以外のボタンが押されたときは、すべてこのinputFormulaメソッドで処理を行う。
- ［C］ボタンはclearCalculation、［=］ボタンはcalculateAnswerという名前にする
- 下記のようになっていればOK
![/images/calc/calc18.jpg](/images/calc/calc18.jpg)

## ボタンが押されたときの処理を記述
### 式の入力（inputFormula）
- ViewController.swiftを開いて、ソースコードのviewDidLoadメソッドおよびinputFormulaメソッドを、次のように編集する

```swift
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // ビューがロードされた時点で式と答えのラベルは空にする
        formulaLabel.text = ""
        answerLabel.text = ""
    }

    @IBAction func inputFormula(_ sender: UIButton) {
        // ボタン（Cと=以外）が押されたら式を表示する
        guard let formulaText = formulaLabel.text else {
            return
        }
        guard let senderedText = sender.titleLabel?.text else {
            return
        }
        formulaLabel.text = formulaText + senderedText
    }
```

- inputFormulaメソッドでは、sender.titleLabel?.textで押されたボタンのTitleを取得できる
- ボタンが押されるたびに、式を連結して、式用のラベルに表示するという処理

### 答えの表示（calculateAnswer）
- Expressionをインポート
```swift
import Expression
```

- 下記のように記述
```swift
    @IBAction func calculateAnswer(_ sender: UIButton) {
        // =ボタンが押されたら答えを計算して表示する
        guard let formulaText = formulaLabel.text else {
            return
        }
        let formula: String = formatFormula(formulaText)
        answerLabel.text = evalFormula(formula)
    }

    private func formatFormula(_ formula: String) -> String {
        // 入力された整数には`.0`を追加して小数として評価する
        // また`÷`を`/`に、`×`を`*`に置換する
        let formattedFormula: String = formula.replacingOccurrences(
                of: "(?<=^|[÷×\\+\\-\\(])([0-9]+)(?=[÷×\\+\\-\\)]|$)",
                with: "$1.0",
                options: NSString.CompareOptions.regularExpression,
                range: nil
            ).replacingOccurrences(of: "÷", with: "/").replacingOccurrences(of: "×", with: "*")
        return formattedFormula
    }

    private func evalFormula(_ formula: String) -> String {
        do {
            // Expressionで文字列の計算式を評価して答えを求める
            let expression = Expression(formula)
            let answer = try expression.evaluate()
            return formatAnswer(String(answer))
        } catch {
            // 計算式が不当だった場合
            return "式を正しく入力してください"
        }
    }

    private func formatAnswer(_ answer: String) -> String {
        // 答えの小数点以下が`.0`だった場合は、`.0`を削除して答えを整数で表示する
        let formattedAnswer: String = answer.replacingOccurrences(
                of: "\\.0$",
                with: "",
                options: NSString.CompareOptions.regularExpression,
                range: nil)
        return formattedAnswer
    }
```

### 式と答えのクリア（clearCalculation）
- 下記のように記述
```swift
    @IBAction func clearCalculation(_ sender: UIButton) {
        // Cボタンが押されたら式と答えをクリアする
        formulaLabel.text = ""
        answerLabel.text = ""
    }
```
## 動作確認
- [Run]して、動作に問題がないか確認する
![/images/calc/calc19.jpg](/images/calc/calc19.jpg)
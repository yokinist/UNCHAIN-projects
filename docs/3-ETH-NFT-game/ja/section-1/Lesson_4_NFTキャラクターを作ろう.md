### 🤔 NFT とは何か？

NFT の概要については、[こちら](https://github.com/yukis4san/Intro-NFT/blob/main/section-1/NFT-S1-lesson-1.md) をご覧ください。

一般的に NFT が何であるかが理解できたら、次のステップに進みましょう。

### 🌱 Mint とは何か？

NFT における「Mint（ミント）」とは、スマートコントラクトを用いて、NFT を新らしく作成・発行することを意味します。

### 🎰 ゲームで遊べる NFT を作る

このプロジェクトでは、**プレイヤーが協力して、ボスを倒すゲーム**を作成していきます。

ゲームの内容は以下のようになります。

- ゲームが始まると、プレイヤーは、一定の攻撃力**と HP** を持つ**キャラクター NFT** を発行します。

- プレイヤーは、その**キャラクター NFT** にボスを攻撃するよう命令し、ボスにダメージを与えます。

- しかし、ボスの HP は、100 万ですので、一人のプレイヤーの攻撃でボスを倒すのは不可能です。

- したがって、このゲームでは、プレイヤーは協力してボスを倒す必要があります。ボスの HP が 0 になれば、プレイヤーの勝利です。

- プレイヤーがボスを攻撃するたびに、ボスもプレイヤーを攻撃してきます。

- プレイヤーの NFT キャラクターの HP が 0 になると、そのキャラクターは**死亡**し、ボスを攻撃できなくなります。

- プレイヤーはウォレットの中に 1 つのキャラクターの NFT しか持つことができません。

- つまり、**複数のプレイヤーが力を合わせてボスを攻撃し、ボスを倒す必要があります。**

ここで知っておくべき重要なことは、キャラクターそのものが **NFT** であるということです。

ゲームの中では、以下のようなフローが必要になります。

1. プレイヤーは、ゲームを開始する際に、Web アプリケーションにウォレットを接続します。
2. Web アプリケーションは、プレイヤーがウォレットにキャラクター NFT をまだ持っていないことを検知します。
3. プレイヤーに、キャラクターを選んでもらい、自分のキャラクター NFT を Mint してゲームを開始してもらいます。
   - 各キャラクター NFT は、NFT に保存された独自の属性（HP、攻撃ダメージ、キャラクターのイメージなど）を持っています。
   - HP が 0 になると `hp: 0` と表示されます。

**これは、世界で最も人気のある NFT ゲームのしくみです。** これから、このしくみをゼロから構築していきます！

### ✨ NFT データをセットアップする

それでは、キャラクター NFT をセットアップしていきましょう。

各キャラクターに、画像、名前、HP 値、攻撃ダメージ値の属性を付与していきます。

キャラクターの NFT は、決まった数（例：3 体）だけ存在することになります。

**各キャラクターの NFT は無制限に Mint できます。**

- 最初のポケモンを選ぶ作業を想像してください。

- ポケモンでは、ゲームを開始する際に、火のポケモン、水のポケモン、草のポケモンの 3 体から一匹、旅をともにするキャラクターを選びます。

- 複数のプレイヤーが同じ種類の NFT キャラクターを Mint した場合、彼らは同じキャラクター NFT を持つことになりますが、プレイヤー各々が、固有の NFT を持つので、**各 NFT キャラクターは独自の状態を保持します。**

- つまり、あるプレイヤーの NFT キャラクターが攻撃を受けて HP を失ったとしても、ほかのプレイヤーの NFT キャラクターに影響はありません。

これから、NFT キャラクターの**デフォルト属性**（デフォルト HP、デフォルト攻撃力、デフォルト画像など）を初期化していきます。

- たとえば、「ピカチュウ」という名前のキャラクターがいた場合、ピカチュウのデフォルト HP、デフォルト攻撃力などを設定します。

それでは、`MyEpicGame.sol` を以下のように更新しましょう。

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;
import "hardhat/console.sol";
contract MyEpicGame {
  // キャラクターのデータを格納する CharacterAttributes 型の 構造体（`struct`）を作成しています。
  struct CharacterAttributes {
    uint characterIndex;
    string name;
    string imageURI;
    uint hp;
    uint maxHp;
    uint attackDamage;
  }
  // キャラクターのデフォルトデータを保持するための配列 defaultCharacters を作成します。それぞれの配列は、CharacterAttributes 型です。
  CharacterAttributes[] defaultCharacters;
  constructor(
	// プレイヤーが新しく NFT キャラクターを Mint する際に、キャラクターを初期化するために渡されるデータを設定しています。これらの値は フロントエンド（js ファイル）から渡されます。
    string[] memory characterNames,
    string[] memory characterImageURIs,
    uint[] memory characterHp,
    uint[] memory characterAttackDmg
  )
  {
  // ゲームで扱う全てのキャラクターをループ処理で呼び出し、それぞれのキャラクターに付与されるデフォルト値をコントラクトに保存します。
	// 後でNFTを作成する際に使用します。
    for(uint i = 0; i < characterNames.length; i += 1) {
      defaultCharacters.push(CharacterAttributes({
        characterIndex: i,
        name: characterNames[i],
        imageURI: characterImageURIs[i],
        hp: characterHp[i],
        maxHp: characterHp[i],
        attackDamage: characterAttackDmg[i]
      }));
      CharacterAttributes memory character = defaultCharacters[i];

	  // hardhat の console.log() では、任意の順番で最大4つのパラメータを指定できます。
	  // 使用できるパラメータの種類: uint, string, bool, address
      console.log("Done initializing %s w/ HP %s, img %s", character.name, character.hp, character.imageURI);
    }
  }
}
```

ここでは、`constructor` に複数の値を渡して、キャラクターをセットアップしています。

この処理により、プレイヤーが NFT キャラクターを発行する際、そのキャラクターにデフォルト HP、デフォルト攻撃力、デフォルト画像などのデータを付与できます。

それでは、一行ずつコードの理解を深めましょう。

```javascript
// MyEpicGame.sol
struct CharacterAttributes {
  uint characterIndex;
  string name;
  string imageURI;
  uint hp;
  uint maxHp;
  uint attackDamage;
}
```

ここでは、キャラクターのデータを格納する `CharacterAttributes` 型の構造体（`struct`）を定義しています。

`CharacterAttributes` には、下記の属性情報が格納されます。

```javascript
uint characterIndex; // キャラクターID（1番、2番.. N番）
string name; // キャラクターの名前
string imageURI; // キャラクターの画像情報
uint hp; // キャラクターの現在のHP
uint maxHp; // キャラクターの最大HP
uint attackDamage; // キャラクターの攻撃力
```

次に、下記のコードを見ていきましょう。

```javascript
// MyEpicGame.sol
CharacterAttributes[] defaultCharacters;
```

ここでは、キャラクターのデフォルトデータを保持するための配列 `defaultCharacters` を作成しています。

`defaultCharacters` 配列は、`CharacterAttributes` 型ですので、キャラクターそれぞれに、`characterIndex` から `attackDamage` までの 6 種類の情報が付与されます。

次に下記のコードを見ていきましょう。

```javascript
// MyEpicGame.sol
constructor(
	string[] memory characterNames,
	string[] memory characterImageURIs,
	uint[] memory characterHp,
	uint[] memory characterAttackDmg
)
```

**`constructor` は、コントラクトが呼び出されるに、一度だけ実行されます。**

今回のプロジェクトでは、プレイヤーが NFT キャラクターを新しく Mint する際に、コントラクトが呼び出されます。

上記の処理では、新しく NFT キャラクターが Mint される際に、キャラクターを初期化するために渡されるデータ（＝デフォルト属性）のセットアップを行っています。

- デフォルトデータは、`characterNames` から `characterAttackDmg` の 4 種類ですので、ここで定義しています。

最後に、次のコードを見ていきましょう。

```javascript
// MyEpicGame.sol
for(uint i = 0; i < characterNames.length; i += 1) {
	defaultCharacters.push(CharacterAttributes({
	characterIndex: i,
	name: characterNames[i],
	imageURI: characterImageURIs[i],
	hp: characterHp[i],
	maxHp: characterHp[i],
	attackDamage: characterAttackDmg[i]
	}));
	CharacterAttributes memory character = defaultCharacters[i];
	console.log("Done initializing %s w/ HP %s, img %s", character.name, character.hp, character.imageURI);
}
```

`constructor` が実行されると、上記のループ処理が実行されます。

ここでは、ゲームに登場するすべてのキャラクターをループ処理で呼び出し、それぞれのキャラクターのデフォルト値をコントラクトに保存しています。

この処理によって、プログラムは、各キャラクターにアクセスできます。

- たとえば、`defaultCharacters[0]` と実行するだけで、1 番目のキャラクターのデフォルト属性に簡単にアクセスできます。

- キャラクターは最初無傷ですので、`hp` ＝ `maxHp` = `characterHp[i]` となっています。

### ⭐️ テストを実行する

`MyEpicGame.sol` を更新したので、テストを行っていきます。

テスト用のスクリプト `run.js` を下記のように更新していきましょう。

```javascript
// run.js
const main = async () => {
  const gameContractFactory = await hre.ethers.getContractFactory("MyEpicGame");
  const gameContract = await gameContractFactory.deploy(
    ["ZORO", "NAMI", "USOPP"], // キャラクターの名前
    [
      "https://i.imgur.com/TZEhCTX.png", // キャラクターの画像
      "https://i.imgur.com/WVAaMPA.png",
      "https://i.imgur.com/pCMZeiM.png",
    ],
    [100, 200, 300], // キャラクターのHP
    [100, 50, 25] // キャラクターの攻撃力
  );
  await gameContract.deployed();
  console.log("Contract deployed to:", gameContract.address);
};
const runMain = async () => {
  try {
    await main();
    process.exit(0);
  } catch (error) {
    console.log(error);
    process.exit(1);
  }
};
runMain();
```

ここでは、3 体のキャラクターとその基本情報を定義しています。

- キャラクターは、ワンピースのゾロ、ナミ、ウソップを用意しました。
- 各キャラクターには、ID、名前、画像、HP、攻撃力の情報が付与されます。

**上記の `gameContractFactory.deploy()` の中に格納されている情報が、`MyEpicGame.sol` の `constructor` に渡されます。**

```javascript
// MyEpicGame.sol
// 例：["ZORO", "NAMI", "USOPP"] = キャラクターの名前 が `characterNames` 配列に渡されます。
string[] memory characterNames,
string[] memory characterImageURIs,
uint[] memory characterHp,
uint[] memory characterAttackDmg
```

[Imgur](https://imgur.com/) に画像をアップロードすれば、あなたの好きなキャラクターを設定できます！

> サンプルで使用しているワンピースの素材は、[こちら](https://www.irasutoya.com/2021/01/onepiece.html) から取得しています。※ 集英社様の許可を得て掲載しているイラストです。作品のイメージを傷つけないよう注意してご利用ください。

ぜひ、`run.js` の中の `https://i.imgur.com/...` の画像のリンクをあなたのオリジナルの画像に差し替えてください 😊

```javascript
// run.js
["ZORO", "NAMI", "USOPP"], // キャラクターの名前
["https://i.imgur.com/TZEhCTX.png",  // キャラクターの画像
 "https://i.imgur.com/WVAaMPA.png",
 "https://i.imgur.com/pCMZeiM.png"],
```

それでは、ターミナル上で、`scripts` ディレクトリに移動して下記を実行してみましょう。

```bash
npx hardhat run run.js
```

ターミナル上で `console.log` の中身とコントラクトアドレスが表示されていることを確認してください。

例）ターミナル上でのアウトプット:

```
Compiling 1 file with 0.8.4
Solidity compilation finished successfully
Done initializing ZORO w/ HP 100, img https://i.imgur.com/TZEhCTX.png
Done initializing NAMI w/ HP 200, img https://i.imgur.com/WVAaMPA.png
Done initializing USOPP w/ HP 300, img https://i.imgur.com/pCMZeiM.png
Contract deployed to: 0x5FbDB2315678afecb367f032d93F642f64180aa3
```

上記のようなアウトプットターミナルに表示されていればテストは成功です。

### 🙋‍♂️ 質問する

ここまでの作業で何かわからないことがある場合は、Discord の `#section-1` で質問をしてください。

ヘルプをするときのフローが円滑になるので、エラーレポートには下記の 3 点を記載してください ✨

```
1. 質問が関連しているセクション番号とレッスン番号
2. 何をしようとしていたか
3. エラー文をコピー&ペースト
4. エラー画面のスクリーンショット
```

---

テストの出力が完了したら次のレッスンに進んでください 🎉

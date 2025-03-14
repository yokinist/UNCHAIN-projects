### 🙉 GitHub に 関する注意点

**GitHub にコントラクト（ `epic-nfts`）のコードをアップロードする際は、秘密鍵を含むハードハット構成ファイルをリポジトリにアップロードしないよう注意しましょう。**

秘密鍵などのファイルを隠すために、ターミナルで `epic-nfts` に移動して、下記を実行してください。

```bash
npm install --save dotenv
```

`dotenv` モジュールに関する詳しい説明は、[こちら](https://maku77.github.io/nodejs/env/dotenv.html)を参照してください。

`dotenv` をインストールしたら、`.env` ファイルを更新します。

ファイルの先頭に `.` がついているファイルは、「不可視ファイル」です。

`.` がついているファイルやフォルダはその名の通り、見ることができないので、「隠しファイル」「隠しフォルダ」とも呼ばれます。

操作されては困るファイルについては、このように「不可視」の属性を持たせて、一般の人が触れられないようにします。

ターミナル上で `epic-nfts` ディレクトリにいることを確認し、下記を実行しましょう。VS Code から `.env` ファイルを開きます。

```
code .env
```

そして、`.env` ファイルを下記のように更新します。

```
PRIVATE_KEY = hardhat.config.jsにある秘密鍵（accounts）を貼り付ける
STAGING_ALCHEMY_KEY = hardhat.config.js内にあるAlchemyのyURLを貼り付ける
PROD_ALCHEMY_KEY = イーサリアムメインネットにデプロイする際に使用するAlchemyのURLを貼り付ける（今は何も貼り付ける必要はありません）
```

私の `.env` は、下記のようになります。

```javascript
PRIVATE_KEY = 0x...
STAGING_ALCHEMY_KEY = https://...
PROD_ALCHEMY_KEY = ""
```

`.env` を更新したら、 `hardhat.config.js` ファイルを次のように更新してください。

```javascript
// hardhat.config.js
require("@nomiclabs/hardhat-waffle");
require("dotenv").config();

module.exports = {
  solidity: "0.8.4",
  networks: {
    rinkeby: {
      url: process.env.STAGING_ALCHEMY_KEY,
      accounts: [process.env.PRIVATE_KEY],
    },
    mainnet: {
      chainId: 1,
      url: process.env.PROD_ALCHEMY_KEY,
      accounts: [process.env.PRIVATE_KEY],
    },
  },
};
```

最後に ` .gitignore` に `.env` が含まれていることを確認しましょう。

`cat .gitignore` をターミナル上で実行します。

下記のような結果が表示されていれば成功です。

```
node_modules
.env
coverage
coverage.json
typechain

#Hardhat files
cache
artifacts
```

これで、GitHub にあなたの秘密鍵をアップロードせずに、GitHub にコントラクトのコードをアップロードできます。

### 🌎 IPFS について

[IPFS](https://docs.ipfs.io/concepts/what-is-ipfs/) は誰にも所有されていない分散型データストレージシステムです。

たとえば、動画の NFT をあなたが発行したいと考えたとしましょう。
オンチェーンでそのデータを保存すると、ガス代が非常に高くなります。

このような場合、IPFS が役に立ちます。

[Pinata](https://www.pinata.cloud/) というサービスを使用すると、簡単に画像や動画を NFT にできます。

- NFT は、いくつかのメタデータにリンクする単なる JSON ファイルであることを思い出してください 💡

- この JSON ファイルを IPFS に配置できます。

**NFT データを保存する一般的な方法として、多くの人が IPSF を利用しています。**

### 📝 Etherscan を使ってコントラクトを verify（検証）する

Etherscan の **コントラクトの Verification（検証）** を行いましょう。

この機能を使えば、あなたの Solidity プログラムを世界中の人に公開できます。

また、あなたもほかの人の書いたコードを読むことができます。

まず、Etherscan のアカウントを取得して、`apiKey` を取得しましょう。

アカウントをまだお持ちでない場合は、[https://etherscan.io/register](https://etherscan.io/register) にアクセスして、無料のユーザーアカウントを作成してください。

アカウントが作成できたら、`My Profile` 画面に移動してください。

![](/public/images/2-ETH-NFT-collection/section-4/4_2_1.png)

`API Keys` タブを選択し、`+ Add` ボタンを押したら、`Create API Key` のポップアップが表示されるので、あなたの API に任意の名前をつけましょう。

![](/public/images/2-ETH-NFT-collection/section-4/4_2_2.png)

次に、あなたが作成した API の横の `Edit` ボタンを選択してください。ポップアップが表示されるので、`apiKey` を取得しましょう。

![](/public/images/2-ETH-NFT-collection/section-4/4_2_3.png)

次に、ターミナルで `epic-nfts` ディレクトリに移動して、次のコマンドを実行してください。 Etherscan で verification を行うために必要なツールをインストールします。

⚠️：Ethereum API key は誰にも教えてはいけません！

```bash
npm install @nomiclabs/hardhat-etherscan
```

そして、`epic-nfts/hardhat.config.js` を編集していきます。

先ほど Etherscan から取得した `apiKey` を `Your_Etherscan_apiKey` に貼り付けてください。

`require("@nomiclabs/hardhat-etherscan");` を含むのも忘れないようにしましょう。

```javascript
// hardhat.config.js

require("@nomiclabs/hardhat-etherscan");

module.exports = {
  solidity: "0.8.4",
  etherscan: {
    apiKey: "Your_Etherscan_apiKey",
  },
  networks: {
    rinkeby: {
      url: "Your_Alchemy_API_Key",
      accounts: ["Your_Private_Key"],
    },
  },
};
```

最後に、下記を実行して、あなたのコントラクトを verify して、世界に公開してみましょう。

```
npx hardhat clean
npx hardhat verify YOUR_CONTRACT_ADDRESS --network rinkeby
```

下記のような結果が表示されれば、verification は成功です。

```
Successfully submitted source code for contract
contracts/MyEpicNFT.sol:MyEpicNFT at 0xB3340071dc206d09170a7269331155ff1BeE64de
for verification on the block explorer. Waiting for verification result...

Successfully verified contract MyEpicNFT on Etherscan.
https://rinkeby.etherscan.io/address/0xB3340071dc206d09170a7269331155ff1BeE64de#code
```

出力された URL リンクをブラウザに貼り付け、中身を確認してみましょう。

私の [URL リンク](https://rinkeby.etherscan.io/address/0xB3340071dc206d09170a7269331155ff1BeE64de#code) の中身は下記のように表示されます。

![](/public/images/2-ETH-NFT-collection/section-4/4_2_6.png)

Etherscan で **Contract** タブを選択すると、下図のような `0x608060405234801 ...` で始まる長いテキストのリストが表示されます。

![](/public/images/2-ETH-NFT-collection/section-4/4_2_12.png)

実は、このテキストのリストは、デプロイされたコントラクトのバイトコードです。

- バイトコードに関する詳しい説明は、[こちら](https://e-words.jp/w/%E3%83%90%E3%82%A4%E3%83%88%E3%82%B3%E3%83%BC%E3%83%89.html) をご覧ください。

`Read Contract` と `Write Contract` の 2 つのサブタブが追加されたことを確認してださい。これらの機能を使えば、コントラクトをオンチェーンで簡単に操作できます。フロントエンドがなくても、コントラクトから直接関数を呼び出せるので、便利ですね 😊

![](/public/images/2-ETH-NFT-collection/section-4/4_2_11.png)

おめでとうございます！　これで、あなたのスマートコントラクトが世界中の誰でも見られるようになりました 🚀

### 🔮 プロジェクトを拡張する

このプロジェクトで学んだことは、Web3 への旅の始まりに過ぎません。

NFT とスマートコントラクトできることはたくさんあります。

あなたのプロジェクトに拡張性を与えるインサイトをいくつか共有します。

**🧞‍♂️: NFT を販売する**

- NFT をユーザーが Mint する際に、あなたに ETH を支払うしくみを実装しましょう。

- コントラクトに `payable` を追加したり、`require` を使用すれば、ユーザーが NFT を購入する際の最小金額を設定できます。

**🍁: ロイヤリティを追加する**

- スマートコントラクトにロイヤリティを追加して、NFT が転売されるごとに、あなたに、一定の資金が振り込まれるしくみを作りましょう。

- 詳細については、こちらをご覧ください: [EIP-2981：NFT Royally Standard ](https://eips.ethereum.org/EIPS/eip-2981)

### 🤟 Vercel に Web アプリケーションをデプロイする

最後に、[Vercel](https://vercel.com/) に Web アプリケーションをホストする方法を学びます。

Vercel はサーバレス機能のホスティングを提供するクラウドプラットフォームです。

スケーリングやサーバの監視は Vercel が行うため、開発者は Vercel へデプロイするだけでアプリケーションを公開・運用できます。

Vercel に関する詳しい説明は、[こちら](https://zenn.dev/lollipop_onl/articles/eoz-vercel-pricing-2020)をご覧ください。

まず、GitHub の `nft-collection-starter-project` にローカルファイルをアップロードしていきます。

ターミナル上で `nft-collection-starter-project` に移動して、下記を実行しましょう。

```
git add .
git commit -m "upload to github"
git push
```

次に、GitHub 上の `nft-collection-starter-project` に、ローカル環境に存在する `nft-collection-starter-project` のファイルとディレクトリが反映されていることを確認してください。

Vercel のアカウントを取得したら、下記を実行しましょう。

1\. `Dashboard` へ進んで、`New Project` を選択してください。

![](/public/images/2-ETH-NFT-collection/section-4/4_2_7.png)

2\. `Import Git Repository` で自分の GitHub アカウントを接続したら、`nft-collection-starter-project` を選択し、`Import` してください。

![](/public/images/2-ETH-NFT-collection/section-4/4_2_8.png)

3\. プロジェクトを作成します。Environment Variable に下記を追加します。

`NAME`＝`CI`、`VALUE`＝`false`（下図参照）

![](/public/images/2-ETH-NFT-collection/section-4/4_2_9.png)

4\. `Deploy`ボタンを推しましょう。

Vercel は GitHub と連動しているので、GitHub が更新されるたびに自動でデプロイを行ってくれます。

下記のように、`Building` ログが出力されます。

基本的に `warning` は無視して問題ありません。

![](/public/images/2-ETH-NFT-collection/section-4/4_2_10.png)

こちらが、今回のプロジェクトで作成される Web アプリケーションのデモです。

https://nft-minter-jl0e7c9yw-yukis4san.vercel.app/

### 🙋‍♂️ 質問する

ここまでの作業で何かわからないことがある場合は、Discord の `#section-4` で質問をしてください。

ヘルプをするときのフローが円滑になるので、エラーレポートには下記の 3 点を記載してください ✨

```
1. 質問が関連しているセクション番号とレッスン番号
2. 何をしようとしていたか
3. エラー文をコピー&ペースト
4. エラー画面のスクリーンショット
```

### 🎫 NFT を取得しよう！

NFT を取得する条件は、以下のようになります。

1. MVP の機能がすべて実装されている（実装 OK）

2. Web アプリケーションで MVP の機能が問題なく実行される（テスト OK）

3. このページの最後にリンクされている Project Completion Form に記入する

4. Discord の `🔥｜post-your-project` チャンネルに、あなたの Web サイトをシェアしてください 😉🎉 Discord に投稿する際に、追加実装した機能とその概要も教えていただけると幸いです！

プロジェクトを完成させていただいた方には、NFT をお送りします。※ 現在準備中ですので、今しばらくお待ちください！

### 🎉 おつかれさまでした！

あなたのオリジナルの NFT Collection が完成しました。

あなたは、コントラクトをデプロイし、NFT が Mint できる Web アプリケーションを立ち上げました。

これらは、分散型 Web アプリケーションがより一般的になる社会の中で、世界を変える 2 つの重要なスキルです。

これからも Web3 への旅をあなたが続けてくれることを願っています 🚀

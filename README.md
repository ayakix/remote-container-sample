VSCode remote containersでnodejs環境を構築する

# 目指すもの
VSCode remote containersを使ってホスト側の環境を汚すことなく、コンテナ側でnodejsの環境構築をします。
その際に、extensionでLintツールをインストールしたり、VSCodeの設定も行います。これは開発メンバー間で同一の環境を作成することを想定しています。

## VSCode remote containersとは？
開発環境をDockerコンテナ側に作成し、ホスト側のディレクトリをマウントすることで、開発者はホスト側のVSCode上で編集することで、コンテナ側のファイルを書き換えることができる機能(extension)です。

![image](https://code.visualstudio.com/assets/docs/remote/containers/architecture-containers.png)
[VSCodeドキュメントより](https://code.visualstudio.com/docs/remote/containers)より

## 構築手順
1. [Docker Desktop](https://www.docker.com/products/docker-desktop/)のインストール
1. Docker Desktopの起動
1. リポジトリクローン `git clone git@github.com:ayakix/remote-container-sample.git`
1. VSCode上より「Open Folder」 にて、ディレクトリを開く
1. コマンドパレットもしくは左下の`><`ボタンから`Reopen in Container`を開く
2. コンテナが起動し、必要ライブラリインストール後に本日の日付が表示されたら正しくコンテナが起動できています。
3. 起動した状態で、test.jsを編集し、例えば、`const now = dayjs();`を`const now=dayjs();`のように編集し、保存してください。コードが自動フォーマットされます。

## 説明
### ファイル構成
```
.
├── .devcontainer
│   ├── devcontainer.json
│   ├── postCreateCommand.sh
│   └── postStartCommand.sh
├── docker-compose.yml
├── package.json
└── test.js : サンプル実行ファイル
```

---

### docker-compose.yml
```docker-compose.yml
version: "3"

services:
  myservice:
    image: mcr.microsoft.com/vscode/devcontainers/javascript-node:0-14
    working_dir: /workspace
    volumes:
      - node-modules:/workspace/node_modules
      - .:/workspace:cached
    tty: true # コンテナを起動させ続ける

volumes:
  node-modules:
```
imageにはmicrosoftのjavascript-node14を使用します。
ベースOSはDebian11(bullseye)となります。

---

.devcontainerディレクトリ以下がVSCode remote containers特有のファイルとなります。

### .devcontainer/devcontainer.json
```.devcontainer/devcontainer.json
{
  "name": "remote container sample",
  "dockerComposeFile": ["../docker-compose.yml"],
  "service": "myservice",
  "workspaceFolder": "/workspace",
  "settings": {
    "terminal.integrated.defaultProfile.linux": "bash",
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.formatOnSave": true,
    "editor.formatOnPaste": true,
    "editor.formatOnType": true,
    "editor.codeActionsOnSave": {
      "source.organizeImports": true,
      "source.fixAll.eslint": true
    },
    "[json]": {
      "editor.formatOnSave": false
    }
  },
  "extensions": ["dbaeumer.vscode-eslint", "esbenp.prettier-vscode"],
  "postCreateCommand": "bash ./.devcontainer/postCreateCommand.sh",
  "postStartCommand": "bash ./.devcontainer/postStartCommand.sh"
}
```
docker-composeへのパス、VSCodeの設定やextensionを記述します。
また、postCreateCommandはコンテナ作成時に一度実行されるもの、postStartCommandはコンテナ起動時に毎回呼ばれるものです。直接コマンドを記述できますが、複数コマンドあると見づらいので、別途shellファイルで管理しています。

### postCreateCommand.sh
```postCreateCommand.sh
npm install
```

### postStartCommand.sh
```postStartCommand.sh
node test.js
```

## 補足
VSCodeコマンドパレット上で、「Close Remote Connection」をすると、10秒後ぐらいにコンテナがkillされます。

VSCodeのDocker extensionをインストールすると、VSCode上でコンテナの停止/削除とかもできます。
```
docker kill [container_id]
docker rm [container_id]
```

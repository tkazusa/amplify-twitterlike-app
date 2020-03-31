# amplify-twitterlike-app
つぶやきアプリ「Boyaki」を Amplify で実装。

## Setup
### Amplify CLI のインストール
```
$ npm install -g @aws-amplify/cli@4.16.1
```
### Java のインストール
[Debian ベースおよび RPM ベースの Linux ディストリビューション用の Amazon Corretto 8 のインストール手順](https://docs.aws.amazon.com/ja_jp/corretto/latest/corretto-8-ug/generic-linux-install.html)

```
 $ wget -O- https://apt.corretto.aws/corretto.key | sudo apt-key add - 
 $ sudo add-apt-repository 'deb https://apt.corretto.aws stable main'
$  sudo apt-get update; sudo apt-get install -y java-1.8.0-amazon-corretto-jdk
```


### Amplify CLI の設定
Amplify の設定を下記コマンドの後に指示に従って実行する。
```
amplify configure
```

## モックの作成
### Bootsrap
```
$ npx create-react-app boyaki
$ cd boyaki
$ amplify init
$ npm start
```
`amplify init` の途中でいくつか質問されるので適宜回答する。
`npm start` の後に `http://localhost_3000` へアクセスすると初期画面が表示される。


### 認証機能の追加
```
$ amplify add auth
$ amplify status
Current Environment: production

| Category | Resource name  | Operation | Provider plugin   |
| -------- | -------------- | --------- | ----------------- |
| Auth     | boyaki6ab6e661 | Create    | awscloudformation |

$ amplify push
```
`push` して数分待つと Amplify と Amazon Cognito が連携し、ユーザープールが作成される。。


aws-amplify-reactとAmplify Frameworkをアプリケーションに追加する。
```
$ npm install --save aws-amplify-react aws-amplify 
```

`./src/App.js` ファイルの中身を確認し、下記のように変更。
```
import React from 'react';

import Amplify from '@aws-amplify/core';
import awsmobile from './aws-exports';

import { withAuthenticator } from 'aws-amplify-react';

Amplify.configure(awsmobile);

function App() {
  return (
    <h1>
      Hello World!
    </h1>
  );
}

export default withAuthenticator(App, {
  signUpConfig: {
    hiddenDefaults: ['phone_number']
  }
});
```

変更が反映されたか確認する。

```
$ npm start
```
`http://localhost_3000` へアクセスすると、


### POST 機能の追加
```
$ amplify add api

Please select from one of the below mentioned services: GraphQL
Provide API name: BoyakiGql
Choose the default authorization type for the API: Amazon Cognito User Pool
Do you want to configure advanced settings for the GraphQL API: No, I am done.
Do you have an annotated GraphQL schema? No
Do you want a guided schema creation? No
Provide a custom type name: Post
```

`./amplify/backend/api/BoyakiGql/schema.graphql` を編集することで API の挙動を生やすことが出来る。

```
type Post
  @model (subscriptions: { level: public })
  @auth(rules: [
    {allow: owner, ownerField:"owner", provider: userPools, operations:[read, create]}
    {allow: private, provider: userPools, operations:[read]}
  ])
{
  type: String! # always set to 'post'. used in the SortByTimestamp GSI
  id: ID
  content: String!
  owner: String
  timestamp: AWSTimestamp!
}
```

`$ amplify push` は反映に時間がかかるので、mock を使うことで開発効率を上げることができる。

```
amplify mock api
```



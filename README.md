# try-azure-static-blazor-app
Blazor アプリを Azure static web apps へデプロイしてみます。

![Azure Static Web Apps CI/CD](https://github.com/MareMare/try-azure-static-blazor-app/workflows/Azure%20Static%20Web%20Apps%20CI/CD/badge.svg?branch=master)

## 前提条件

* Azure アカウント
* GitHub アカウント

## アプリケーションの作成

サクッと `dotnet` CLI で作ります。ついでなのでソリューションも追加しビルドしておきます。

```ps
mkdir TryAzureStaticBlazorApp; cd TryAzureStaticBlazorApp
dotnet new blazorwasm
cd ..
dotnet new sln
dotnet sln add TryAzureStaticBlazorApp
dotnet build
```

### VSCode の準備

`launch.json` と `tasks.json` の各ファイルを準備します。

> アプリケーションを作成したら、一度VSCodeを再起動すると次のメッセージが表示されるので「Yes」を選択すると、VSCodeの環境設定ファイルが用意されるので捗ります。
> 
> ![01](images/01.png)

ただ VSCode では、既定のブラウザが Chrome であるので、Edge で Blazor WebAssembly アプリを起動してデバッグするには、以下のように `browser` に `edge` を指定します。

* `launch.json`

```json
{
    // Use IntelliSense to find out which attributes exist for C# debugging
    // Use hover for the description of the existing attributes
    // For further information visit https://github.com/OmniSharp/omnisharp-vscode/blob/master/debugger-launchjson.md
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Launch and Debug Standalone Blazor WebAssembly App",
            "type": "blazorwasm",
            "request": "launch",
            "cwd": "${workspaceFolder}/TryAzureStaticBlazorApp",
            // "hosted": true,
            // "program": "${workspaceFolder}/TryAzureStaticBlazorApp/bin/Debug/netstandard2.1/TryAzureStaticBlazorApp.dll",
            "browser": "edge"
        }
    ]
}
```

## Azure Static Web Apps の作成

1. 静的 Web アプリの作成

    [Azure portal](https://portal.azure.com/) へサインインして「静的 Web アプリの作成(プレビュー)」を作成します。

    ![02](images/02.png)

    ここでは次を設定し「確認および作成」をクリックします。

    |項目|設定値|
    |---|---|
    |サブスクリプション|無料試用版|
    |リソースグループ|(新規) rgStaticWebApps|
    |名前|TryAzureStaticBlazorApp|
    |地域|East Asia|
    |SKU|Free|
    |GitHubアカウント|自分のGitHubアカウント|
    |組織|自分の組織|
    |リポジトリ|try-azure-static-blazor-app|
    |分岐|master|
    |ビルドのプリセット|Blazor|
    |アプリの場所|Client|
    |APIの場所|Api|
    |アプリの成果物の場所|wwwroot|

2. 静的 Web アプリの設定内容の確認

    ![03](images/03.png)

    とりあえず「作成」をクリックすると、デプロイが開始されるのでしばらく待ちます。

3. 静的 Web アプリのデプロイ
    
    デプロイ完了した後でリソースへ移動します。

    ![04](images/04.png)

    > Azure 静的 Web アプリをご利用いただき、ありがとうございます。サイトのコンテンツをまだ受け取っていません。ここをクリックして、GitHub アクション実行の状態をご確認ください。

    上記のメッセージが表示されている場合、GitHub アクションが実行されていない、または失敗しているので、GitHub のリポジトリで確認します。

4. GitHub アクションの修正と実行

    GitHub リポジトリを Azure Static Web Apps にリンクすると、そのリポジトリに GitHub アクションのワークフローファイルが追加されます。
    既定では次のファイルが追加されています。

    * `.github/workflows/azure-static-web-apps-<RANDOM_NAME>.yml`
    
        ```yml
        name: Azure Static Web Apps CI/CD

        on:
          push:
            branches:
              - main
          pull_request:
            types: [opened, synchronize, reopened, closed]
            branches:
              - main

        jobs:
          build_and_deploy_job:
            if: github.event_name == 'push' || (github.event_name == 'pull_request' && github.event.action != 'closed')
            runs-on: ubuntu-latest
            name: Build and Deploy Job
            steps:
              - uses: actions/checkout@v2
                with:
                  submodules: true
              - name: Build And Deploy
                id: builddeploy
                uses: Azure/static-web-apps-deploy@v1
                with:
                  azure_static_web_apps_api_token: ${{ secrets.  AZURE_STATIC_WEB_APPS_API_TOKEN_BRAVE_STONE_0645CC000 }}
                  repo_token: ${{ secrets.GITHUB_TOKEN }} # Used for Github integrations (i.e. PR comments)
                  action: "upload"
                  ###### Repository/Build Configurations - These values can be configured to match your app   requirements. ######
                  # For more information regarding Static Web App workflow configurations, please visit: https://aka.  ms/swaworkflowconfig
                  app_location: "/TryAzureStaticBlazorApp" # App source code path
                  api_location: "Api" # Api source code path - optional
                  output_location: "wwwroot" # Built app content directory - optional
                  ###### End of Repository/Build Configurations ######

        close_pull_request_job:
            if: github.event_name == 'pull_request' && github.event.action == 'closed'
            runs-on: ubuntu-latest
            name: Close Pull Request Job
            steps:
              - name: Close Pull Request
                id: closepullrequest
                uses: Azure/static-web-apps-deploy@v1
                with:
                  azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN_BRAVE_STONE_0645CC000 }}
                  action: "close"
        ```
    
    今回の環境用に `Build And Deploy` の設定を次のように変更してコミットします。

    |項目|修正前|修正後|
    |---|---|---|
    |app_location|"Client"|"/TryAzureStaticBlazorApp"|
    |api_location|"Api"|"Api"|
    |app_artifact_location|"wwwroot"|"wwwroot"|

    ```yml
    ###### Repository/Build Configurations - These values can be configured to match your app requirements. ######
    # For more information regarding Static Web App workflow configurations, please visit: https://aka.ms/swaworkflowconfig
    app_location: "/TryAzureStaticBlazorApp" # App source code path
    api_location: "Api" # Api source code path - optional
    output_location: "wwwroot" # Built app content directory - optional
    ###### End of Repository/Build Configurations ######
    ```

    コミットすると自動で GitHub アクションが実行されるのでしばらく待ちます。

    ![05](images/05.png)

5. 動作確認

    GitHub アクション実行に成功すると、静的 Web アプリの概要は次のようになります。

    ![06](images/06.png)

    ここで「URL」または「参照」をクリックすると、デプロイされた Web アプリが次のように表示されます。

    ![07](images/07.png)


## 参考サイト
* [チュートリアル:Azure Static Web Apps での Blazor を使用した静的 Web アプリのビルド \| Microsoft Docs](https://docs.microsoft.com/ja-jp/azure/static-web-apps/deploy-blazor?WT.mc_id=-blog-scottha)
* [Azure Static Web Apps with \.NET and Blazor \| ASP\.NET Blog](https://devblogs.microsoft.com/aspnet/azure-static-web-apps-with-blazor/)
* [Blazor WebAssembly を触ってみる \- その②デバッグしてみる \- Qiita](https://qiita.com/chyonek/items/ef76e97d18904053fcf6)
* [ASP\.NET Core Blazor WebAssembly をデバッグする \| Microsoft Docs](https://docs.microsoft.com/ja-jp/aspnet/core/blazor/debug?view=aspnetcore-3.1&tabs=visual-studio-code)
* [Azure Static Web Apps の GitHub Actions ワークフロー \| Microsoft Docs](https://docs.microsoft.com/ja-jp/azure/static-web-apps/github-actions-workflow#build-and-deploy)

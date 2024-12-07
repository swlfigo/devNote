# 通过Github Action 部署 Flutter Web

push到github前需要修改  Flutter 项目目录下web文件夹中的 index.html

```html
  <!--
    If you are serving your web app in a path other than the root, change the
    href value below to reflect the base path you are serving from.

    The path provided below has to start and end with a slash "/" in order for
    it to work correctly.

    For more details:
    * https://developer.mozilla.org/en-US/docs/Web/HTML/Element/base

    This is a placeholder for base href that will be replaced by the value of
    the `--base-href` argument provided to `flutter build`.
  -->
<!-- 这里改成仓库名称 -->
  <base href="/YOUR-REPO-NAME/">
```



```yaml
#通过以下Action 可以部署 GithubPage
name: Flutter Web
on:
  push:
    branches:
      - master
jobs:
  build:
    name: Build Web
    env:
  		#在 仓库 Setting 中设置秘钥
      my_secret: ${{secrets.PERSONAL_TOKEN}}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          channel: "stable"
          #根据所需Flutter Version 自行修改
          flutter-version: 3.24.5
      - run: flutter --version
      - run: flutter config --enable-web
      - run: flutter pub get
      - run: flutter channel master
      - run: flutter upgrade
      - run: flutter build web --release
      - run: |
          cd build/web
          git init
          #自行修改git push email
          git config --global user.email YOUR-EMAIL
          #自行修改git push username
          git config --global user.name YOUR-NAME
          git status
					#自行修改仓库地址网址
          git remote add origin https://${{secrets.PERSONAL_TOKEN}}@github.com/YOUR/REPO.git
          #push到仓库 gh-pages 分支, 在仓库 Setting 中 Pages启用该分支的Pages功能
          git checkout -b gh-pages
          git add --all
          git commit -m "update"
          git push origin gh-pages -f

```



# Reference

https://juejin.cn/post/7366062210932260918

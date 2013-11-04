Golang 中文文件
================

## 簡介 Introduction

這裡是 Golang TW 用來翻譯 [golang.org](http://golang.org/) 上各文件的地方。

## 現況 Current State

目前正在作業的只有 [Effective Go](http://golang.org/doc/effective_go.html) 一文。目前該文正基於 Go 1.1 內所附的版本進行初譯。

## 如何貢獻 How to Contribute

目前我們在 [Github](https://github.com/) 上面，以 [GitHub Flavored Markdown 格式](https://help.github.com/articles/github-flavored-markdown)管理／編寫這些翻譯文件。以下是翻譯與校潤流程：

1. [準備好自己的 Github 帳號](https://github.com/join)
2. 將[翻譯文件](https://github.com/golangtw/Docs)獨立 [fork 一份成為自己的 repository](https://help.github.com/articles/fork-a-repo)
3. 選好想要修改的部份，然後[在 golangtw/Docs 新增一條 issue](https://github.com/golangtw/Docs/issues)，昭告天下這部份已被誰所認領
4. 在剛剛 fork 出來的 repository 裡面開始快樂的作業
5. 完工並確認無誤之後，[向 golang/Docs 發出 pull request](https://github.com/golangtw/Docs/pulls)。請務必在 commit comment 中註明：
    - 這是哪部份的翻譯
    - 對應前述步驟 3 裡面的哪一條 issue。例如「Fixes #3924」
6. 靜待管理人員合併到 master branch 裡面。如果遲遲得不到回音，也可以在 issue tracker 上面詢問發生了什麼事。

## 翻譯原則 Translate Rules

1. 所有的文件使用 UTF-8 編碼，換行字元為 `<LF>`。
2. 使用台灣地區正體中文（Traditional Chinese）的字集與用語；如果特殊名詞實在沒有正體中文譯名且不在譯名表內，請保留英文原文。
3. 遵守 [GitHub Flavored Markdown 格式](https://help.github.com/articles/github-flavored-markdown)。
4. 文件內錨點請盡可能以 ASCII 字元命名，因此中文章節可能需要另外以 HTML 手動宣告錨點，例如 `<a name="anchor-name"></a>`。
5. 文中的範例程式碼，只翻譯註解與必要的字串，不對程式碼的變數／宣告式本身進行翻譯。
6. Markdown 的縮排為 4 空白字元，但範例程式碼則保留原文格式
7. 譯文內的標點符號盡量使用全形標點符號。

由於目前尚在起步階段，請不吝提出各種疑問或建議。

### 譯名表 Table of Translated Terms

（持續增加中）

- argument/parameter 參數
- function 函式
- memory 記憶體
- programming 程式設計
- case `case` 述句
- variable 變數

## 開發

致各位翻譯者，更新 repo 請愛用 git rebase, e.g.,

    git pull --rebase origin master


## 聯絡方式 Contact Us

目前 Golang TW [在 freenode 上有專門的 IRC channel](http://webchat.freenode.net/?channels=golang.tw)「#golang.tw」，歡迎與大家一起討論！

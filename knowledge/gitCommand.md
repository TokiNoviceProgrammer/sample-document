**●reset**

**通常コミットの取り消し**

以下では例として`テストコミット04`を取り消す

![image](https://github.com/user-attachments/assets/77be286e-f7e0-4459-9754-f44c045f6179)

`git reset --hard HEAD^`を実行→ローカルブランチが1つ前のコミットに戻る
(コミット前のステージングは残したい(ローカルでの変更は残したい)場合は`hard`ではなく`soft`を使用する)

![image](https://github.com/user-attachments/assets/2484b165-f443-44d3-bce4-a02a8633dd68)

`git push --force`で強制プッシュすることで、リモートブランチにもコミットがなかったことになる

![image](https://github.com/user-attachments/assets/b4cf9e4f-c679-4731-9619-b015553f926e)

**マージのコミットも通常のコミットと同様に取り消せる**
以下では例として`bug-xxxx`のマージを取り消す

![image](https://github.com/user-attachments/assets/7e68819c-44ad-43da-baa0-348d0d2b3848)

`git reset --hard HEAD^`を実行→ローカルブランチが1つ前のコミットに戻る

![image](https://github.com/user-attachments/assets/6be3fd17-fe28-4ef0-9f67-d33d416ce3d6)

`git push --force`で強制プッシュすることで、リモートブランチにもコミットがなかったことになる

![image](https://github.com/user-attachments/assets/f1e627c3-971d-4660-b343-669a72ae3eb0)

マージの取り消し後は再度プルリクを作成してマージ可能

![image](https://github.com/user-attachments/assets/f8152ed5-bbf2-4229-9034-72a8a211c3af)

プルリクを承認してマージすると取り消し前と同じ状態になっていることがわかる

![image](https://github.com/user-attachments/assets/bf0961cd-8247-4743-a28e-b40e3d107088)

---

**●revert**

**取り消したコミットの履歴を残したい場合**

以下は、テストコミット07を取り消す例

![image](https://github.com/user-attachments/assets/dc0e9a87-d50a-4aaf-9fc0-e44b3c47b975)

`git revert HEAD`を実行すると以下のようにコミットコメント編集モードになる

![image](https://github.com/user-attachments/assets/36321290-db96-459d-8a9e-6f4e86105777)

編集が終われば`esc`を押下し、`:wq`を入力してエンターを押下する
(コメントは基本はデフォルトのままで良いはずであるため、何もせずに上記の手順を実施すれば良い)

![image](https://github.com/user-attachments/assets/5da343f8-81ff-442f-9087-b07606318783)

上記を実施後、「テストコミット07を取り消す」内容の新たなコミットができる

![image](https://github.com/user-attachments/assets/517cf3be-8f07-4d2f-8265-22a9ac85bc60)

`git push origin feature`を実行し、指定のリモートブランチにプッシュする

![image](https://github.com/user-attachments/assets/275534c4-024a-449b-ae9b-efa9fdd54bdc)

テストコミット07を取り消した(revert した)コミットがpushされていることが確認できる

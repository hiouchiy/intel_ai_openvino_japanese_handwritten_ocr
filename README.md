# インテル® OpenVINO™ ツールキットを使った日本語手書き文字のOCRデモ
OpenVINOを使って日本語の手書き文字を認識するアプリをさくっと作ってみます。
 
## Getting Started / スタートガイド
### Prerequisites / 必要条件
- Intel CPU（Core or Xeon）を搭載したマシン
    - Core: 第6世代以上
    - Xeon: Sandy Bridge以上
- OS: Linux（Ubuntu 18.04がお薦め）／Windows 10／
macOS 10.15
- Docker（※以下にインストール手順記載）
### Installing / インストール
#### ホストOSのポート開放（リモートアクセスする場合のみ）
このハンズオンではJupyter LabおよびOpenVINO Model Serverを使用します。特にサーバーにリモートアクセスしながら実施する場合は各環境ごとの手順に則り、ホストOSのポート「8888」、「9000」番を開放ください。
#### Dockerインストール
##### Linux（Ubuntu 18.04）
```Bash
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
sudo apt update
apt-cache policy docker-ce
sudo apt install -y docker-ce
sudo usermod -aG docker ${USER}
su - ${USER}
id -nG
```
##### Windows 10
https://docs.docker.jp/docker-for-windows/install.html
##### macOS
https://docs.docker.jp/docker-for-mac/install.html
#### Dockerイメージのダウンロード
今回はDocker版のOpenVINOを使用します（2020年11月2日現在、バージョン2021R1がダウンロードされる）。OSに直接インストールされたい方は[公式ドキュメント（英語）](https://docs.openvinotoolkit.org/latest/install_directly.html)を参照ください。
```Bash
docker pull openvino/ubuntu18_dev
```
#### Dockerコンテナの起動
コンテナはRootで起動します。また、8888番ポートをホストOSとコンテナとでバインドしておきます。
```Bash
docker run -it -u 0 --privileged -p 8888:8888 openvino/ubuntu18_dev:latest /bin/bash
```
以降はコンテナ上での作業になります。
#### 追加モジュールのインストール
```Bash
apt-get update
apt-get install -y wget unzip git sudo
apt-get install -y ubuntu-restricted-extras　
(※↑でEULAにacceptを求められるのでyesと入力)
apt-get install -y ffmpeg
pip install jupyterlab munkres
```
#### 本レポジトリをClone
```Bash
cd ~
git clone https://github.com/hiouchiy/intel_ai_openvino_japanese_handwritten_ocr.git
```
#### Jupyter Labの起動
```Bash
jupyter lab --ip=0.0.0.0 --no-browser --allow-root
```
#### WebブラウザからJupyter Labにアクセス
前のコマンド実行すると以下のようなログが出力されまして、最後にローカルホスト（127.0.0.1）のトークン付きURLが表示されるはずです。こちらをWebブラウザにペーストしてアクセスください。リモートアクセスされている場合はIPアドレスをサーバーのホストOSのIPアドレスに変更してください。
```
root@f79f54d47c1b:~# jupyter lab --allow-root --ip=0.0.0.0 --no-browser
[I 09:13:08.932 LabApp] JupyterLab extension loaded from /usr/local/lib/python3.6/dist-packages/jupyterlab
[I 09:13:08.933 LabApp] JupyterLab application directory is /usr/local/share/jupyter/lab
[I 09:13:08.935 LabApp] Serving notebooks from local directory: /root
[I 09:13:08.935 LabApp] Jupyter Notebook 6.1.4 is running at:
[I 09:13:08.935 LabApp] http://f79f54d47c1b:8888/?token=2d6863a5b833a3dcb1a57e3252e641311ea7bc8e65ad9ca3
[I 09:13:08.935 LabApp]  or http://127.0.0.1:8888/?token=2d6863a5b833a3dcb1a57e3252e641311ea7bc8e65ad9ca3
[I 09:13:08.935 LabApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
[C 09:13:08.941 LabApp] 
    
    To access the notebook, open this file in a browser:
        file:///root/.local/share/jupyter/runtime/nbserver-33-open.html
    Or copy and paste one of these URLs:
        http://f79f54d47c1b:8888/?token=2d6863a5b833a3dcb1a57e3252e641311ea7bc8e65ad9ca3
     or http://127.0.0.1:8888/?token=2d6863a5b833a3dcb1a57e3252e641311ea7bc8e65ad9ca3
```
↑こちらの例の場合は、最後の "http://127.0.0.1:8888/?token=2d6863a5b833a3dcb1a57e3252e641311ea7bc8e65ad9ca3" です。
#### Notebookの起動
Jupyter Lab上で「intel_ai_openvino_japanese_handwritten_ocr」フォルダーに入り、その中の「handwritten-ocr.ipynb」を開き、後はノートブックの内容に従って進めてください。

---
## 応用編：OpenVINO Model Serverを使ってモデルをWeb API化
[OpenVINO Model Server](https://github.com/openvinotoolkit/model_server)を使うとOpenVINOのモデルを簡単にWeb API化できます。以下の手順通りにダウンロードおよび起動をしてください。
### Installing / インストール
#### OpenVINO Model ServerのDockerイメージをダウンロード
ホストOS上でもう一つターミナルを開き、下記コマンドを実行
```Bash
docker pull openvino/model_server:latest
```
#### OpenVINOの事前学習済みモデルをダウンロード
ハンズオンの中で使用した日本語の手書き文字認識用の事前学習済みモデルをダウンロードして、ホストOS上の適当なフォルダに格納しておく
#### OpenVINO Model Serverを起動
各パラメータの意味については[こちら](https://github.com/openvinotoolkit/model_server/blob/main/docs/docker_container.md)を参照ください。
```Bash
docker run -d -v <モデルを格納しているフォルダへの絶対パス>:/models/ocr/1 -p 9000:9000 openvino/model_server:latest \
--model_path /models/ocr --model_name ocr --port 9000 --log_level DEBUG --shape auto
```
#### NotebookからOpenVINO Model Serverへアクセス
前のコンテナ（Jupyter Lab実行中）のNotebook（handwritten-ocr.ipynb）に戻り、「【応用編】OpenVINO Model Serverを使う」から再開ください。
## License / ライセンス
このプロジェクトは Apache 2.0の元にライセンスされています。
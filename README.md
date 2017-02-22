[item]: # (slide)

![](http://imapex.io/images/imapex_standing_text_sm.png)

[item]: # (/slide)

[item]: # (slide)

----

# CICD Learning Lab (2017/2/22 ハンズオン用)

![](images/stage_final_diagram.png)

この README は、2016/12月のSEVTで行われた CI/CD ラボを、リデリバリ用に内容を修正したものです. 
元の README は https://github.com/imapex-training/cicd_learning_lab を参照してください

[item]: # (/slide)

<!-- ### Presentation Slides

A slide show version of this presentation is available at this link: [CICD Learning Lab Slides](https://rawgit.com/imapex-training/cicd_learning_lab/master/output/index.html#/)
 -->

[item]: # (slide)

----

# Lab Agenda
以下の順番にラボを進めます
<!-- Here are links to each part of the lab.  They do build on each other so be sure to go in order. -->

0. [Introduction](https://github.com/radiantmarch/cicd_learning_lab#introduction)
0. [Prerequisites](https://github.com/radiantmarch/cicd_learning_lab#prerequisites)
1. [Environment Prep](https://github.com/radiantmarch/cicd_learning_lab#environment-prep)
2. [Stage 1 - Continuous Integration](https://github.com/radiantmarch/cicd_learning_lab#cicd-stage-1-continuous-integration)
3. [Stage 2 - Continuous Delivery](https://github.com/radiantmarch/cicd_learning_lab#cicd-stage-2-continuous-delivery)
4. [Install the Application](https://github.com/radiantmarch/cicd_learning_lab#install-your-application)
5. [Stage 3 - Continuous Deployment](https://github.com/radiantmarch/cicd_learning_lab#cicd-stage-3-continuous-deployment)
6. [Stage 4 - Monitoring and Notify Phase](https://github.com/radiantmarch/cicd_learning_lab#monitor-and-notify-phase)
8. [Clean-Up](https://github.com/radiantmarch/cicd_learning_lab#cleanup---uninstall-your-application)

<!-- 7. [Bonus - CICD In Action!](bonus.md) -->

[item]: # (/slide)


[item]: # (slide)

# Introduction
[アジェンダに戻る](https://github.com/radiantmarch/cicd_learning_lab#lab-agenda)

![](images/labcomponents.png)

[item]: # (/slide)

This lab is intended to be an introduction to setting up a very basic CI/CD (Continuous Integration/Continuous Delivery) Pipeline.  There are many different technologies and methods that can be used for CI/CD, in this lab we will use:

* Cisco's Mantl Container Stack - [Mantl.io](http://mantl.io) which includes:
  * Docker Container Engine - [Docker](http://www.docker.com)
  * Mesos Scheduler - [Mesos/Marathon](http://mesos.apache.org)
* Development Language - Python and Flask
* Source Control - [github.com](https://github.com)
* Container Registry - [hub.docker.com](http://hub.docker.com)
* CICD Server - [drone.io](http://drone.io)
* Notifications - [CiscoSpark.com](http://CiscoSpark.com)

[item]: # (slide)

# Prerequisites
[アジェンダに戻る](https://github.com/radiantmarch/cicd_learning_lab#lab-agenda)

[item]: # (/slide)

ラボを開始する前に以下を読んで準備をお願いします
* [Cloud SEVT リデリバリ- CI/CD ハンズオン 事前準備](https://cisco.jiveon.com/docs/DOC-1691497)

このラボを進めるためには、以下のいくつかのサービスサイトのアカウントが必要です（すべてフリーアカウントでOK）
<!-- To run through this lab, you will need to have accounts (all free) created with the following services, as well as a set of tools properly setup on your laptop or workstation. -->

[item]: # (slide)

## Accounts

* [github.com](https://github.com)
	* 自身のフリーアカウントを作成してログイン
	* Note if you are using 2FA for your GitHub account, let the admin know, as that will change how you do authentication later in the lab exerices.

* [hub.docker.com](http://hub.docker.com)
	* 自身のフリーアカウントを作成してログイン

[item]: # (/slide)

* [CiscoSpark.com](http://CiscoSpark.com) Developer Token
	* Cisco社員は CEC パスワードでログインでOK
    * ![Spark Token](images/spark_token.png)
	* You will also use your Spark login to retrieve the Spark API token. 
	* Go to [developer.ciscospark.com](http://developer.ciscospark.com), where you will find your token by logging into the developer portal, clicking on your image in the upper right corner, and clicking **Copy**

[item]: # (slide)

## Laptop or Workstation

Mac / Windows で共通の手順にするため、Virtual Box を利用する方法に統一しています

※ Mac/Windows ネイティブ環境で利用する方法については [元のラボ資料](https://github.com/imapex-training/cicd_learning_lab) のほうを参照してください. または近くの講師に相談ください

[item]: # (/slide)

[item]: # (slide)

### Option: Run the Lab within a Container

以下のサイトの通り、Virtual Box をインストールして推奨設定を済ませて下さい
* [Cloud SEVT リデリバリ- CI/CD ハンズオン 事前準備](https://cisco.jiveon.com/docs/DOC-1691497)

<!-- [hpreston/devbox](https://hub.docker.com/r/hpreston/devbox) -->

Virtual Box でデプロイした VM (名前: Development Sandbox) を起動します。起動したら VM コンソール画面に入り、ログインします
* ユーザ名: devbox
* パスワード: devbox

最初からコンソールが開いていて、以下のようなプロンプトになっているはずです
```
[root@cf95a414877e coding]#
```

もしそうなっていない場合は以下の手順を実行します

0. Virtual Box VM のデスクトップアイコンからターミナルを起動する
0. 以下のコマンドを実行

```
[devbox@devbox ~]$ docker run -it --name cicdlab hpreston/devbox:cicdlab
[devbox@devbox ~]$ docker start -i cicdlab
[root@c0b2cc5a69fa coding]#  <<< Linux コンテナ内に入った
```

このコンテナは Linux ベースの作業環境で、以下のツールがすでにインストール済み

* nano 
	* Provided as a text editor to use for executing the lab steps
* git
* docker
  * the container has the docker tools installed, but the `docker run` command above will **NOT** enable you to run additional containers from inside
  * running containers is not a required step in the lab, but if you would like to do so, you can use the following command instead
    ```
    docker run -it --name cicdlab -v /var/run/docker.sock:/var/run/docker.sock hpreston/devbox:cicdlab
    ```
  * this command will link the docker daemon on the host machine into the container
* drone cli tools

[item]: # (slide)

もしコンテナから抜けてしまった場合、`docker run` コマンドは実行しないでください
実行してしまうと、それまでの作業内容が消え、新しいコンテナが起動してしまいます
<!-- If you exit out of the container before completing the lab and want to continue from where you left off, do not execute a `docker run` command again.  This will create a new clean container that lacks any of your work.  
 -->
その代わりに以下を実行して、停止中のコンテナがあるかどうか確認します
<!-- Instead follow the below to start the original container. -->

```
# Verify that you have  a container in a stopped state
docker ps -a

CONTAINER ID        IMAGE                         COMMAND             CREATED             STATUS                        PORTS               NAMES
cf95a414877e        hpreston/devbox:cicdlab       "/bin/bash"         2 minutes ago       Exited (0) 10 seconds ago                         cicdlab

# Restart your stopped container
docker start -i cicdlab

[root@cf95a414877e coding]#
```

[item]: # (/slide)

[item]: # (slide)

## Lab Environment Details

今回のラボ環境の情報は以下の通りです.
ラボを進める中で必要なので適宜参照してください

<!-- Your instructor will provide you an TXT file with this content needed to complete the lab.   -->
<!-- https://github.com/radiantmarch/cicd_learning_lab#lab-environment-details -->

```
# CICD Learning Lab Infrastructure Details

# Lab Guide
ハンズオン向け改変版: https://github.com/radiantmarch/cicd_learning_lab 
オリジナル: https://github.com/imapex-training/cicd_learning_lab/blob/master/README.md

# Build Server
drone server address: http://drone.lab.apps.imapex.io/

# Target Cloud Infrastructure
Mantl Control Server Address: <当日講師が説明>
Mantl Username: <当日講師が説明>
Mantl Password: <当日講師が説明>
Mantl Application Domain: <当日講師が説明>

# Spark Room Info
Spark Room Name: <当日講師が説明>
Spark RoomId: <当日講師が説明>. "CICD ハンズオン用" ルームの場合は "
SPARK_TOKEN: <下の説明参照>	
```

## Spark SPARK_TOKEN の取得方法
SPARK_TOKEN は https://developer.ciscospark.com/# にログインして自分の写真をクリックすると、表示されます
=> 手元にコピーしておいてください

## (参考) Spark RoomId の取得方法
いろいろな方法があるようですが、Mac の場合 Terminal を開いて以下を実行すればゲットできます
```
$ SPARK_TOKEN=<自分のトークン>
$ curl https://api.ciscospark.com/v1/rooms -X GET -H "Authorization:Bearer ${SPARK_TOKEN}"
<< これで Room 一覧が json 形式で表示される
<< 一覧の中から Room 名の含まれるエントリを探し、"id" のエントリの値をコピーする

## jq コマンドが使える場合は以下が便利
$ curl https://api.ciscospark.com/v1/rooms -X GET -H "Authorization:Bearer ${SPARK_TOKEN}" | jq . -a rooms.txt
<< テキストに書き出して検索すると見やすい
<< 出力例
    {
      "id": "Y2lzY29zcGFyazovL3VzL1JPT00vYTNhNjA4YjAtZjQyYS0xMWU2LWEwMDItMWY2MWUxNDU5ZTcy",
      "title": "CICD ハンズオン用",
      "type": "group",
      "isLocked": false,
      "lastActivity": "2017-02-16T09:31:57.395Z",
      "teamId": "Y2lzY29zcGFyazovL3VzL1RFQU0vYzc5ZmRhYzAtYzU5NS0xMWU2LWJiMGMtOGIyMDY3OTcyYzZl",
      "creatorId": "Y2lzY29zcGFyazovL3VzL1BFT1BMRS9iNWNmOTM5YS03ZWMwLTRkM2EtOGYzNy1kMjQzNWIyMjVkNzk",
      "created": "2017-02-16T09:31:05.301Z"
    },
```

[item]: # (/slide)

[item]: # (slide)


---

# Environment Prep
[アジェンダに戻る](https://github.com/radiantmarch/cicd_learning_lab#lab-agenda)

以下はコンテナ内で作業します. 

	* Tips: SSH で Virtual Box VM にログインして実施しても OK. 
	(別途ポートフォワーディング設定が必要. 参考: http://note.kurodigi.com/vbox-ssh/)

With all the pre-reqs completed, you are ready to start the lab.  We'll start by setting up our new application code repo, container repository, and continuous integration server configuration.

[item]: # (/slide)

[item]: # (slide)

## Forking the cicd_demoapp GitHub Repo

今回デプロイするアプリ cicd_demoapp のソースを GitHub 上で Fork します

![GitHub Fork](images/github_fork.png)

[item]: # (/slide)

[item]: # (slide)

### Steps
1. Log into GitHub and visit the demo app repo [imapex-training/cicd_demoapp](https://github.com/imapex-training/cicd_demoapp)
2. Click **Fork** to create a copy of the repo in your account

    ![GitHub Fork](images/github_fork.png)

[item]: # (/slide)

[item]: # (slide)

3. Make a local clone of **YOUR** repo on your laptop.  Do NOT clone the `imapex-training/cicd_demoapp` repo. In this case, we will create a directory called "coding". If you choose to deviate from that directory name, just remember what directory you created for your local repo, as this will be used multiple times through the lab. 

[item]: # (/slide)

[item]: # (slide)

**Cloning**

※ コンテナ内での作業

```
# if you don't have a local directory where you keep projects, create one
mkdir ~/coding

# enter this new directory (or where ever you keep your code)
cd ~/coding

# clone your fork of the demo app
# replace USERNAME with your GitHub user name
git clone https://github.com/USERNAME/cicd_demoapp

```
    
[item]: # (/slide)

[item]: # (slide)

## What is "cicd_demoapp"

Here are some basic details about the demoapp we are building.  

* Web Application written in Python using Flask Framework 
* It's a "Hello World" Application
* Deployed via a Docker Container
* Includes test cases for full coverage
* Includes Installation Scripts

Checkout the source repo and code:  [imapex-training/cicd_demoapp](https://github.com/imapex-training/cicd_demoapp)

[item]: # (/slide)

[item]: # (slide)
    
## Create Docker Repository 

![Docker Hub New Repo](images/docker_hub_new_repo.png)

[item]: # (/slide)

[item]: # (slide)

### Steps
1. Log into [hub.docker.com](http://hub.docker.com) with your account
2. Click the blue **Create Repository** button in the upper right corner

    ![Docker Hub New Repo](images/docker_hub1.png)

[item]: # (/slide)

[item]: # (slide)

3. Name the repo _cicd\_demoapp_ and provide a short description.

    ![Docker Hub New Repo](images/docker_hub_new_repo.png)

[item]: # (/slide)

[item]: # (slide)

## Activate the Repo in Drone

![Drone Repos](images/drone_available_repos.png)

[item]: # (/slide)

[item]: # (slide)

### Steps

1. **Make sure the lab administrator has enabled your GitHub account on the lab server.** (今回はスキップしてOK)
2. Navigate to the drone server address provided by the lab administrator, and click **Login**.

    ![Drone Login](images/drone_login.png)

    * http://drone.lab.apps.imapex.io/

[item]: # (/slide)

[item]: # (slide)

3. Drone uses GitHub for authentication, so you will either be prompted to log into your account, and then authorize drone, or simply authorize drone if you're already logged into GitHub.
4. Once you've logged in, click the **Available Repositories** tab in the upper right corner.

    ![Drone Repos](images/drone_available_repos.png)

[item]: # (/slide)

[item]: # (slide)

5. Find the _cicd\_demoapp_ repo in the list, and click to activate it.  Drone will then setup a WebHook in GitHub to be notified of relevant events, such as as code pushes, automatically.

    ![Drone Repos](images/drone_active_repos.png)

6. On the **Active Repositories** tab, you should now see a new entry for the cicd_demoapp.  If you click on it, there should be no builds reported yet.

[item]: # (/slide)

[item]: # (slide)

## Build Secrets File
シークレットファイルを作成します. 必要な情報をメモしてください.
上記の [ラボ環境情報](https://github.com/radiantmarch/cicd_learning_lab#lab-environment-details) も参照してください.

```
environment:
  SPARK_TOKEN: <当日講師が説明>
  SPARK_ROOM: <当日講師が説明>
  DOCKER_USERNAME: 自身で作成した docker.com アカウントのユーザ名
  DOCKER_PASSWORD: 自身で作成した docker.com アカウントのパスワード
  DOCKER_EMAIL: 自身で作成した docker.com アカウントの Email アドレス
  MANTL_USERNAME: <当日講師が説明>
  MANTL_PASSWORD: <当日講師が説明>
  MANTL_CONTROL: <当日講師が説明>
```

<!-- 
```
environment:
  SPARK_TOKEN: <FROM YOUR DEVELOPER.CISCOSPARK.COM ACCOUNT>
  SPARK_ROOM: <ROOMID PROVIDED BY THE LAB ADMIN>
  DOCKER_USERNAME: <YOUR HUB.DOCKER.COM USERNAME>
  DOCKER_PASSWORD: <YOUR HUB.DOCKER.COM PASSWORD>
  DOCKER_EMAIL: <YOUR HUB.DOCKER.COM EMAIL ADDRESS>
  MANTL_USERNAME: <MANTL USER PROVIDED BY LAB ADMIN>
  MANTL_PASSWORD: <MANTL PASSWORD PROVIDED BY LAB ADMIN>
  MANTL_CONTROL: <MANTL SERVER ADDRESS PROVIDED BY LAB ADMIN>
```
 -->

[item]: # (/slide)

In order for Drone to be able to do the hard work of testing your application, building a container and publishing it to your registry, notifying your team of status, and deploying the updated code to production, it requires credentials for several systems to act on your behalf.  These details are often referred to as _secrets_ in application development.   As a general rule, you want to protect these details through encryption to maintain their security.  Commiting your passwords to a code repo is never a good idea, but it is especially bad when using a public repo like github.com.

Drone provides a method to create an encrypted file with needed secrets that can be safely included in a code repo.  This is why we installed the drone command line utilities as part of the pre-reqs.

[item]: # (slide)

**_In this step you will be entering several commands in a terminal window.  These need to be run from your local repo directory.  If you followed the directions when cloning the repo locally, this command will place you in the correct directory_**

```
cd ~/coding/cicd_demoapp
```

[item]: # (/slide)

[item]: # (slide)

### Steps
※ コンテナ内での作業

1. The drone utilities on your laptop need to know the address and access information for the drone server you are using.  We use session environment variables for this. You will replace the variable's value with the information the lab admin gives you. The follow code can be copied and pasted directly into a terminal window if you'd like to do that, but you can also just type in the line that doesn't start with the hash mark.

    ```
    # Configure the drone server address,
    # Use the address provided by the lab administrator
    export DRONE_SERVER=http://drone.lab.apps.imapex.io/
    ```

[item]: # (/slide)

[item]: # (slide)

2.  Find your personal Drone token by viewing your profile in the drone console.  Click the down arrow next to your picture in the upper right corner and navigate to _Profile_.  Click the button to _Show Token_ and copy the displayed value.

    ![Drone Profile](images/drone_profile.png)

[item]: # (/slide)

[item]: # (slide)

3.  Copy/Paste or execute this command to store the value in your terminal session.

    ```
    # Configure your token
    export DRONE_TOKEN=<your token>
    ```

[item]: # (/slide)

[item]: # (slide)

3. Test the command line tools by listing the repositories configured.  You should see your cicd_demoapp listed like below. If you don't see something similar, please let the lab admin know.

    ```
    drone repo ls

    <yourusername>/cicd_demoapp
    ```

[item]: # (/slide)

[item]: # (slide)

4. Next we will create the clear text version of our secrets file.  This file is only used to build the encrypted version and should **NEVER** be added to or commited to your repo.  A sample template for the secrets file was included in the demo app.  We will copy this file and edit it to put in the values that we will use for the lab exercises.

    ```
    cp drone_secrets_sample.yml drone_secrets.yml
    ```

[item]: # (/slide)

[item]: # (slide)

5. Edit the copied file in whatever IDE or editor you prefer.  You'll need to provide the details on each line of the file.  This is a YML format, so be sure to maintain proper spacing, **including a single space after the colon in each line**. 
[Secrets File](https://github.com/radiantmarch/cicd_learning_lab#build-secrets-file) 参照.

    ```
    environment:
      SPARK_TOKEN: <FROM YOUR DEVELOPER.CISCOSPARK.COM ACCOUNT>
      SPARK_ROOM: <EITHER ONE OF YOUR OWN ROOMS, OR A ROOMID PROVIDED BY THE LAB ADMIN>
      DOCKER_USERNAME: <YOUR HUB.DOCKER.COM USERNAME>
      DOCKER_PASSWORD: <YOUR HUB.DOCKER.COM PASSWORD>
      DOCKER_EMAIL: <YOUR HUB.DOCKER.COM EMAIL ADDRESS>
      MANTL_USERNAME: <MANTL USER PROVIDED BY LAB ADMIN>
      MANTL_PASSWORD: <MANTL PASSWORD PROVIDED BY LAB ADMIN>
      MANTL_CONTROL: <MANTL SERVER ADDRESS PROVIDED BY LAB ADMIN>
    ```

	* コロンの後のシングルスペースに注意


[item]: # (/slide)

[item]: # (slide)

6. Save your secrets file, but do NOT add or commit it to your repo.  
	* **Pay particular attention that you do not mistakenly ADD your completed secrets file to git.  If you are leveraging an IDE or GUI to execute the git add/commit/push steps, it is very easy to send sensitive files to GitHub**

[item]: # (/slide)

[item]: # (slide)

7. Run this command to create the encrypted **.drone.sec** file.

    ```
    # Replace USERNAME with your GitHub username
    drone secure --repo USERNAME/cicd_demoapp --in drone_secrets.yml
    ```

[item]: # (/slide)

[item]: # (slide)

8. Add this file to git, commit and push it to GitHub.  When you `git push` you may be prompted for your GitHub credentials, provide them. The code below will commit and push ONLY the encrypted file, not the clear text file. 
	* **Note if you are using a graphical IDE and choose to use that for your commit/push actions, it's very easy to _accidentally_ save and commit all changed files in a directory, therefore _accidentally_ pushing your clear text secrets YML file. Be conscious of what you commit and push.**

[item]: # (/slide)

[item]: # (slide)

```
# add the file to the git repo
git add .drone.sec

# commit the change
git commit -m "Added .drone.sec file"

# push changes to GitHub
git push
```

[item]: # (/slide)

[item]: # (slide)

9. Return to the drone server web interface and look at your repo status.  You should see a build has kicked off.

    ![Drone Build](images/drone_1st_build.png)

[item]: # (/slide)


[item]: # (slide)

# CICD Stage 1: Continuous Integration

[アジェンダに戻る](https://github.com/radiantmarch/cicd_learning_lab#lab-agenda)

**In each of the following steps, you will be repeating the steps to secure your secrets.  If you close your terminal window, you will need to re-export the DRONE_SERVER and DRONE_TOKEN values**

The first step of the CICD process is to react to new code commits and test the changes against the larger application needs.  In this lab, we will configure drone to send an alert to a Spark room each time a commit has been made.

[item]: # (/slide)

[item]: # (slide)

## Update the .drone.yml configuration

In the root of the code repository is a file named _.drone.yml_ that provides the instructions to Drone on what actions to take upon each run.  We will be updating this file at each stage of the lab to move from Continuous Integration -> Delivery -> Deployment.

[item]: # (/slide)

**_In this step you will be entering several commands in a terminal window.  These need to be run from your local repo directory.  If you followed the directions when cloning the repo locally, this command will place you in the correct directory_**

```
cd ~/coding/cicd_demoapp
```

[item]: # (slide)

### Steps
1. Open the _.drone.yml_ file in your editor.  The file should begin looking like the code block below.  The _build_ directive at the top represents where the integration phase of the process will take place.  

In the _commands:_ list under _run_tests:_ are the different steps needed to test a successful code change.

```
build:
  build_starting:
    image: python:2
    commands:
      - echo "Beginning new build"	
```

[item]: # (/slide)

[item]: # (slide)

2. Now we will add Testing to the *build* phase.  Update `.drone.yml` to match the updated content below. In the _commands:_ list under _run\_tests:_ are the different steps needed to test a successful code change.  
	* **NOTE You can simply copy and paste the contents from below directly into your file.**

```
build:
    build_starting:
        image: python:2
        commands:
            - echo "Beginning new build"
    run_tests:
        image: python:2-alpine
        commands:
            - pip install -r requirements.txt
            - python testing.py
```

[item]: # (/slide)

[item]: # (slide)

3. As part of the security of drone, every change to the _.drone.yml_ file requires the secrets file to be recreated.  Since we've updated this file, we need to re-secure our secrets file.

    ```
    # Replace USERNAME with your GitHub username
    drone secure --repo USERNAME/cicd_demoapp --in drone_secrets.yml
    ```

[item]: # (/slide)

[item]: # (slide)

* If the command above gives the error `Error: you must provide the Drone server address.`, you likely closed the terminal window after the Environment Prep step.  Run these commands to reset the environment variables.

    ```
    export DRONE_SERVER=http://DRONE_SERVER
    export DRONE_TOKEN=<your token>
    ```

[item]: # (/slide)

[item]: # (slide)

4. Now commit and push the changes to the drone configuraiton and secrets file to GitHub.
	* **Remember if you are using an IDE, be careful not to commit & push the changes to the file drone_secrets.yml**

    ```
    # add the file to the git repo
    git add .drone.sec
    git add .drone.yml

    # commit the change
    git commit -m "Updated Build Phase to run UnitTests"

    # push changes to GitHub
    git push
    ```
    
[item]: # (/slide)

[item]: # (slide)

5. Now check the Drone web interface. 

    ![Drone Build](images/drone_2nd_build.png)

[item]: # (/slide)

[item]: # (slide)

6. Click on the build and scroll through the log displayed.  You can use this log to monitor the process in each stage as we add additional steps to the build process. The Drone log window also has an arrow at the bottom right that you can click and it will automatically tail the end of the log file as it progresses. 

    ![Drone Build](images/drone_2nd_build_details.png)

[item]: # (/slide)

[item]: # (slide)

7. If the build reports a **Failure**, check the log to see what might have gone wrong.  Common reasons include forgetting to re-create the secrets file, forgetting to commit the secrets file after recreating, or mis-entered credentials in the plain text secrets file.

[item]: # (/slide)

[item]: # (slide)

## Current Build Pipeline Status

![Stage 1 Diagram](images/stage_1_diagram.png)

[item]: # (/slide)

Okay, so drone said it did something... but you may be wondering what actually happened.  This image and walkthrough shows the steps that are occuring along the way.

1. You committed and pushed code to GitHub.com
2. GitHub sent a WebHook to the Drone server notifying it of the committed code.
3. Drone checks the _.drone.yml_ file and executes the commands in the _build_ phase. During this phase, Drone has two steps: 
	* *build_starting* 	
  		* Fetch a container from hub.docker.com to begin the build.  This container is identified in the `image: python:2` line of the drone config file.  Drone will run the commands listed in this section
	* *run_tests*
  		* Fetch a container from hub.docker.com to do the testing.  This container is identified in the `image: python:2-alpine` line of the drone config file.  Drone will run the tests described by the commands listed in this section

---

[item]: # (slide)

# CICD Stage 2: Continuous Delivery
[アジェンダに戻る](https://github.com/radiantmarch/cicd_learning_lab#lab-agenda)

In this step, we will take our successfully tested application, build a Docker Container, and publish it to a docker registry where it can be used as an artifact for our application.

[item]: # (/slide)

[item]: # (slide)

## Update the .drone.yml configuration

[item]: # (/slide)

In the root of the code repository is a file _.drone.yml_ that provides the instructions to drone on what actions to take upon each run.  We will be updating this file at each stage of the lab to move from Continuous Integration -> Delivery -> Deployment.

**_In this step you will be entering several commands in a terminal window.  These need to be run from your local repo directory.  If you followed the directions when cloning the repo locally, this command will place you in the correct directory_**

```
cd ~/coding/cicd_demoapp
```

[item]: # (slide)

### Steps
1. Open the _.drone.yml_ file in your editor.  We will be adding the next phase, _publish_ to the existing configuration.  Update your drone file to look like this.
    1. **Note You can simply copy and paste the contents from below directly into your file.  The notation of _$$VARIABLE_ references details stored encrypted within the .drone.sec file.  Do NOT replace them with clear text credentials.**

[item]: # (/slide)

[item]: # (slide)

```
build:
    build_starting:
        image: python:2
        commands:
            - echo "Beginning new build"
    run_tests:
        image: python:2-alpine
        commands:
            - pip install -r requirements.txt
            - python testing.py

publish:
    docker:
        repo: $$DOCKER_USERNAME/cicd_demoapp
        tag: latest
        username: $$DOCKER_USERNAME
        password: $$DOCKER_PASSWORD
        email: $$DOCKER_EMAIL 
```

[item]: # (/slide)

[item]: # (slide)

2. As part of the security of drone, every change to the _.drone.yml_ file requires the secrets file to be recreated.  Since we've updated this file, we need to re-secure our secrets file.

    ```
    # Replace USERNAME with your GitHub username
    drone secure --repo USERNAME/cicd_demoapp --in drone_secrets.yml
    ```

[item]: # (/slide)

* If the command above gives the error `Error: you must provide the Drone server address.`, you likely closed the terminal window after the Environment Prep step.  Run these commands to reset the environment variables.

    ```
    export DRONE_SERVER=http://DRONE_SERVER
    export DRONE_TOKEN=<your token>
    ```

[item]: # (slide)

3. Now commit and push the changes to the drone configuraiton and secrets file to GitHub.
	* **Remember if you are using an IDE, be careful not to commit & push the changes to the file drone_secrets.yml**

    ```
    # add the file to the git repo
    git add .drone.sec
    git add .drone.yml

    # commit the change
    git commit -m "Added Publish Phase to build and push container"

    # push changes to GitHub
    git push
    ```
    
[item]: # (/slide)

[item]: # (slide)

4. Now check the Drone web interface. A new build should have kicked off.  

    ![Drone Build](images/drone_3rd_build.png)

[item]: # (/slide)

5. If the build reports a **Failure**, check the log to see what might have gone wrong.  Common reasons include forgetting to re-create the secrets file, forgetting to commit the secrets file after recreating, or mis-entered credentials in the plain text secrets file.

[item]: # (slide)

6. Once the build completes, log into [hub.docker.com](hub.docker.com), and open your repo.  Click the _Tags_ tab.  You should see a new tag of _latest_ displayed.  This is the container that was build and pushed up by drone.

    ![Docker Hub](images/docker_hub_repo_tags.png)

[item]: # (/slide)

[item]: # (slide)

## Current Build Pipeline Status

![Stage 2 Diagram](images/stage_2_diagram.png)

[item]: # (/slide)

Okay, so drone said it did something... but you may be wondering what actually happened.  This image and walkthrough shows the steps that are occuring along the way.

1. You committed and pushed code to GitHub.com
2. GitHub sent a WebHook to the Drone server notifying it of the committed code.
3. Drone checks the _.drone.yml_ file and executes the commands in the _build_ phase. During this phase, Drone has two steps: 
	* *build_starting* 	
  		* Fetch a container from hub.docker.com to begin the build.  This container is identified in the `image: python:2` line of the drone config file.  Drone will run the commands listed in this section
	* *run_tests*
  		* Fetch a container from hub.docker.com to do the testing.  This container is identified in the `image: python:2-alpine` line of the drone config file.  Drone will run the tests described by the commands listed in this section

4. Drone checks the _.drone.yml_ file and executes the commands in the _publish_ phase. During this phase, Drone will: 
  * Build a Docker Container using the Dockerfile definition included in the Git repo
  * Push the container up to hub.docker.com using the credentials contained in the secrets file

---

[item]: # (slide)

# Install your application
[アジェンダに戻る](https://github.com/radiantmarch/cicd_learning_lab#lab-agenda)

Now that we have used CICD to automate the creation of a Docker container for our application, we can now deploy our application.  This step could certainly be automated as well, but in this lab we will manually install our application to illustrate the process and rely on automation to keep it up to date.

[item]: # (/slide)

[item]: # (slide)

## Investigate the installation process

### Key Files
* [sample-demoapp.json](https://github.com/hpreston/cicd_demoapp/blob/master/sample-demoapp.json) 
* [app_install.sh](https://github.com/hpreston/cicd_demoapp/blob/master/app_install.sh) 
* [app_uninstall.sh](https://github.com/hpreston/cicd_demoapp/blob/master/app_uninstall.sh) 

[item]: # (/slide)

[item]: # (slide)

### Steps
1. Included in your repository are three files that are used to define, install, and uninstall the application.

[item]: # (/slide)

[item]: # (slide)

* [sample-demoapp.json](https://github.com/hpreston/cicd_demoapp/blob/master/sample-demoapp.json) is a template for the Marathon Application definition that will be used.  A custom version will be created at installation that references **YOUR** Docker container.

[item]: # (/slide)

[item]: # (slide)

* [app_install.sh](https://github.com/hpreston/cicd_demoapp/blob/master/app_install.sh) is a bash script that installs the demo application.  This has three steps.
    1. Collect environment details: Lab Mantl Address, Lab Username, Lab Password, Your Docker Username, and the Lab Application Domain
    2. Create an application definition for your deployment
    3. Install your application using the REST API for Marathon.

[item]: # (/slide)

[item]: # (slide)

* [app_uninstall.sh](https://github.com/hpreston/cicd_demoapp/blob/master/app_uninstall.sh) is a bash script that uninstalls the demoapp.  This has two steps.
    1. Collect environment details: Lab Mantl Address, Lab Username, Lab Password, and Your Docker Username
    2. Destroy your application using the REST API for Marathon.

[item]: # (/slide)

[item]: # (slide)

2. You could manually install the application using the Marathon GUI, however the GUI lacks the ability to configure certain parameters we need for the lab.  Besides... deploying through APIs is so much cooler!

[item]: # (/slide)

[item]: # (slide)

## Install your application

[item]: # (/slide)

**_In this step you will be entering several commands in a terminal window.  These need to be run from your local repo directory.  If you followed the directions when cloning the repo locally, this command will place you in the correct directory_**

```
cd ~/coding/cicd_demoapp
```

[item]: # (slide)

From the root of your code repository...

### Steps
1. Execute the installation script. Remember to check the information the lab administrator has provided for the correct responses to the script prompts. Be sure to pay attention to the last few lines of the script output, as these will give you the URLs for your application (you'll need this to test the app) and the control interface for Marathon. 

[item]: # (/slide)

[item]: # (slide)

```
./app_install.sh

# You will be prompted to enter the information needed before the applicaiton deploys.
# The process will look like this

Please provide the following details on your lab environment.

What is the address of your Mantl Control Server?
eg: control.mantl.internet.com
control.mantl.domain.com

What is the username for your Mantl account?
admin

What is the password for your Mantl account?
**HIDDEN**

What is the your Docker Username?
hpreston

What is the Lab Application Domain?
mantl.domain.com


***************************************************
Installing the demoapp as  class/hpreston
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1508    0   997  100   511   1643    842 --:--:-- --:--:-- --:--:--  1642
{
    "acceptedResourceRoles": null,
    "args": null,
    "backoffFactor": 1.15,
    "backoffSeconds": 1,
    "cmd": null,
    "constraints": [],
    "container": {
        "docker": {
            "forcePullImage": true,
            "image": "hpreston/cicd_demoapp:latest",
            "network": "BRIDGE",
            "parameters": [],
            "portMappings": [
                {
                    "containerPort": 5000,
                    "hostPort": 0,
                    "protocol": "tcp",
                    "servicePort": 0
                }
            ],
            "privileged": false
        },
        "type": "DOCKER",
        "volumes": []
    },
    "cpus": 0.1,
    "dependencies": [],
    "deployments": [
        {
            "id": "7e659013-3b2e-4446-954e-2405c3ebc865"
        }
    ],
    "disk": 0,
    "env": {},
    "executor": "",
    "healthChecks": [
        {
            "gracePeriodSeconds": 300,
            "ignoreHttp1xx": false,
            "intervalSeconds": 60,
            "maxConsecutiveFailures": 3,
            "portIndex": 0,
            "protocol": "TCP",
            "timeoutSeconds": 20
        }
    ],
    "id": "/class/hpreston",
    "instances": 1,
    "labels": {},
    "maxLaunchDelaySeconds": 3600,
    "mem": 16,
    "ports": [
        0
    ],
    "requirePorts": false,
    "storeUrls": [],
    "tasks": [],
    "tasksHealthy": 0,
    "tasksRunning": 0,
    "tasksStaged": 0,
    "tasksUnhealthy": 0,
    "upgradeStrategy": {
        "maximumOverCapacity": 1,
        "minimumHealthCapacity": 1
    },
    "uris": [],
    "user": null,
    "version": "2016-07-01T12:45:10.685Z"
}
***************************************************

Installed
Wait 2-3 minutes for the service to deploy.
Then you can visit your application at:

http://class-hpreston.mantl.domain.com/hello/world


You can also watch the progress from the GUI at:

https://control.mantl.domain.com/marathon
```
    
[item]: # (/slide)

[item]: # (slide)

2. The application is now being deployed, and you can watch the progress from the lab console address (provided by the lab admin as well as at the end of the script output).

    ![Marathon App Install](images/marathon_app_install.png)

[item]: # (/slide)

[item]: # (slide)

3. Once the application is fully deployed (shouldn't take more than 3 minutes), and shows healthy status in the console, you can visit your applicaiton at the URL that was provided at the end of installation script.

    ![App Hello World](images/app_hello_world.png)

[item]: # (/slide)    

[item]: # (slide)

## Current Build Pipeline Status

![Manual Install Diagram](images/manual_install_diagram.png)

[item]: # (/slide)

Okay, so building on the process from the previous step, this diagram shows what we've added.

1. You committed and pushed code to GitHub.com
2. GitHub sent a WebHook to the Drone server notifying it of the committed code.
3. Drone checks the _.drone.yml_ file and executes the commands in the _build_ phase. During this phase, Drone will: 
  * Fetch a container from hub.docker.com.  This container is identified in the `image: python:2` line of the drone config file.  Drone will run the commands and tests described in this phase from the fetched container
  * Send a notification message to the subscribed Spark room stating that the build has begun. 
4. Drone checks the _.drone.yml_ file and executes the commands in the _publish_ phase. During this phase, Drone will: 
  * Build a Docker Container using the Dockerfile definition included in the Git repo
  * Push the container up to hub.docker.com using the credentials contained in the secrets file

* ***You manually Install the application on Mantl***

---

[item]: # (slide)

# CICD Stage 3: Continuous Deployment
[アジェンダに戻る](https://github.com/radiantmarch/cicd_learning_lab#lab-agenda)

In this step, we will automatically deploy our changes by issuing a restart command to Marathon that will pull down the new docker container.

[item]: # (/slide)

[item]: # (slide)

## Update the .drone.yml configuration

[item]: # (/slide)

In the root of the code repository is a file _.drone.yml_ that provides the instructions to drone on what actions to take upon each run.  We will be updating this file at each stage of the lab to move from Continuous Integration -> Delivery -> Deployment.

**_In this step you will be entering several commands in a terminal window.  These need to be run from your local repo directory.  If you followed the directions when cloning the repo locally, this command will place you in the correct directory_**

```
cd ~/coding/cicd_demoapp
```

[item]: # (slide)

### Steps
1. Open the _.drone.yml_ file in your editor.  We will be adding the next phase, _deploy_ to the existing configuration.  Update your drone file to look like this.
    1. **Note You can simply copy and paste the contents from below directly into your file.  The notation of _$$VARIABLE_ references details stored encrypted within the .drone.sec file.  Do NOT replace them with clear text credentials.**

[item]: # (/slide)

[item]: # (slide)

```
build:
    build_starting:
        image: python:2
        commands:
            - echo "Beginning new build"
    run_tests:
        image: python:2-alpine
        commands:
            - pip install -r requirements.txt
            - python testing.py
	
publish:
    docker:
        repo: $$DOCKER_USERNAME/cicd_demoapp
        tag: latest
        username: $$DOCKER_USERNAME
        password: $$DOCKER_PASSWORD
        email: $$DOCKER_EMAIL
	
deploy:
    webhook:
        image: plugins/drone-webhook
        skip_verify: true
        method: POST
        auth:
            username: $$MANTL_USERNAME
            password: $$MANTL_PASSWORD
        urls:
            - https://$$MANTL_CONTROL/marathon/v2/apps/class/$$DOCKER_USERNAME/restart?force=true
```

[item]: # (/slide)

[item]: # (slide)

2. As part of the security of drone, every change to the _.drone.yml_ file requires the secrets file to be recreated.  Since we've updated this file, we need to re-secure our secrets file.

    ```
    # Replace USERNAME with your GitHub username
    drone secure --repo USERNAME/cicd_demoapp --in drone_secrets.yml
    ```

[item]: # (/slide)

1. If the command above gives the error `Error: you must provide the Drone server address.`, you likely closed the terminal window after the Environment Prep step.  Run these commands to reset the environment variables.

    ```
    export DRONE_SERVER=http://DRONE_SERVER
    export DRONE_TOKEN=<your token>
    ```

[item]: # (slide)

3. Now commit and push the changes to the drone configuraiton and secrets file to GitHub.
	* **Remember if you are using an IDE, be careful not to commit & push the changes to the file drone_secrets.yml**

    ```
    # add the file to the git repo
    git add .drone.sec
    git add .drone.yml

    # commit the change
    git commit -m "Added Deploy phase to restart application"

    # push changes to GitHub
    git push
    ```
    
[item]: # (/slide)

[item]: # (slide)

4. Now check the Drone web interface. A new build should have kicked off.  Also, watch for the Spark message to come through in the client. 

    ![Drone Build](images/drone_4th_build.png)

[item]: # (/slide)    

5. If the build reports a **Failure**, check the log to see what might have gone wrong.  Common reasons include forgetting to re-create the secrets file, forgetting to commit the secrets file after recreating, or mis-entered credentials in the plain text secrets file.

[item]: # (slide)

6. While the build is running, open Marathon from the Lab Mantl installation, and watch your application.  When Drone gets to the deploy phase, you should see a new task get created as the application restarts.

    ![Marathon Deploy](images/marathon_restart.png)

[item]: # (/slide)

[item]: # (slide)

## Current Build Pipeline Status

![Stage 3 Diagram](images/stage_3_diagram.png)

[item]: # (/slide)

Okay, so drone said it did something... but you may be wondering what actually happened.  This image and walkthrough shows the steps that are occuring along the way.

1. You committed and pushed code to GitHub.com
2. GitHub sent a WebHook to the Drone server notifying it of the committed code.
3. Drone checks the _.drone.yml_ file and executes the commands in the _build_ phase. During this phase, Drone has two steps: 
	* *build_starting* 	
  		* Fetch a container from hub.docker.com to begin the build.  This container is identified in the `image: python:2` line of the drone config file.  Drone will run the commands listed in this section
	* *run_tests*
  		* Fetch a container from hub.docker.com to do the testing.  This container is identified in the `image: python:2-alpine` line of the drone config file.  Drone will run the tests described by the commands listed in this section

4. Drone checks the _.drone.yml_ file and executes the commands in the _publish_ phase. During this phase, Drone will: 
  * Build a Docker Container using the Dockerfile definition included in the Git repo
  * Push the container up to hub.docker.com using the credentials contained in the secrets file
5. Drone checks the _.drone.yml_ file and executes the commands in the _deploy_ phase. In this phase, the following actions will take place: 
  * Drone sends a WebHook command to Marathon to cause an application restart
  * Marathon pulls the new container from hub.docker.com containing the code changes

---

**_Before beginning this step, be sure to be at a command line prompt from your prepared working environment.  This will either be your local machine, or within the provided container._**

[item]: # (slide)

# Monitor and Notify Phase
[アジェンダに戻る](https://github.com/radiantmarch/cicd_learning_lab#lab-agenda)

In this step, we will configure the job to send a notification to the Spark room indicating whether the build was successful or failed.

[item]: # (/slide)

[item]: # (slide)

## Update the .drone.yml configuration

[item]: # (/slide)

In the root of the code repository is a file _.drone.yml_ that provides the instructions to drone on what actions to take upon each run.  We will be updating this file at each stage of the lab to move from Continuous Integration -> Delivery -> Deployment.

**_In this step you will be entering several commands in a terminal window.  These need to be run from your local repo directory.  If you followed the directions when cloning the repo locally, this command will place you in the correct directory_**

```
cd ~/coding/cicd_demoapp
```

[item]: # (slide)

### Steps
1. Open the _.drone.yml_ file in your editor.  We will be adding the next phase, _notify_ to the existing configuration.  Update your drone file to look like this.
    1. **Note You can simply copy and paste the contents from below directly into your file.  The notation of _$$VARIABLE_ references details stored encrypted within the .drone.sec file.  Do NOT replace them with clear text credentials.**

[item]: # (/slide)

[item]: # (slide)

```
build:
    build_starting:
        image: python:2
        commands:
            - echo "Beginning new build"
    run_tests:
        image: python:2-alpine
        commands:
            - pip install -r requirements.txt
            - python testing.py
	
publish:
    docker:
        repo: $$DOCKER_USERNAME/cicd_demoapp
        tag: latest
        username: $$DOCKER_USERNAME
        password: $$DOCKER_PASSWORD
        email: $$DOCKER_EMAIL
	
deploy:
    webhook:
        image: plugins/drone-webhook
        skip_verify: true
        method: POST
        auth:
            username: $$MANTL_USERNAME
            password: $$MANTL_PASSWORD
        urls:
            - https://$$MANTL_CONTROL/marathon/v2/apps/class/$$DOCKER_USERNAME/restart?force=true

notify:
    spark:
        image: hpreston/drone-spark
        auth_token: $$SPARK_TOKEN
        roomId: $$SPARK_ROOM            
```

[item]: # (/slide)

[item]: # (slide)

2. As part of the security of drone, every change to the _.drone.yml_ file requires the secrets file to be recreated.  Since we've updated this file, we need to re-secure our secrets file.

    ```
    # Replace USERNAME with your GitHub username
    drone secure --repo USERNAME/cicd_demoapp --in drone_secrets.yml
    ```

[item]: # (/slide)

1. If the command above gives the error `Error: you must provide the Drone server address.`, you likely closed the terminal window after the Environment Prep step.  Run these commands to reset the environment variables.

    ```
    export DRONE_SERVER=http://DRONE_SERVER
    export DRONE_TOKEN=<your token>
    ```

[item]: # (slide)

3. Now commit and push the changes to the drone configuraiton and secrets file to GitHub.
	* **Remember if you are using an IDE, be careful not to commit & push the changes to the file drone_secrets.yml**

    ```
    # add the file to the git repo
    git add .drone.sec
    git add .drone.yml

    # commit the change
    git commit -m "Added Notify Phase"

    # push changes to GitHub
    git push
    ```
    
[item]: # (/slide)

[item]: # (slide)

4. Now check the Drone web interface. A new build should have kicked off.  Also, watch for the Spark message to come through in the client. 

    ![Drone Build](images/drone_5th_build.png)

[item]: # (/slide)

5. If the build reports a **Failure**, check the log to see what might have gone wrong.  Common reasons include forgetting to re-create the secrets file, forgetting to commit the secrets file after recreating, or mis-entered credentials in the plain text secrets file.

[item]: # (slide)

6. At the end of the build check the Spark Room.  You should see a status notification regarding the state of the build.

    ![Spark Notification](images/spark_notify1.png)

[item]: # (/slide)

[item]: # (slide)

## Current Build Pipeline Status

![Final Diagram](images/stage_final_diagram.png)

[item]: # (/slide)

Okay, so drone said it did something and we got a Spark message... but you may be wondering what actually happened.  This image and walkthrough shows the steps that are occuring along the way.

1. You committed and pushed code to GitHub.com
2. GitHub sent a WebHook to the Drone server notifying it of the committed code.
3. Drone checks the _.drone.yml_ file and executes the commands in the _build_ phase. During this phase, Drone has two steps: 
	* *build_starting* 	
  		* Fetch a container from hub.docker.com to begin the build.  This container is identified in the `image: python:2` line of the drone config file.  Drone will run the commands listed in this section
	* *run_tests*
  		* Fetch a container from hub.docker.com to do the testing.  This container is identified in the `image: python:2-alpine` line of the drone config file.  Drone will run the tests described by the commands listed in this section

4. Drone checks the _.drone.yml_ file and executes the commands in the _publish_ phase. During this phase, Drone will: 
  * Build a Docker Container using the Dockerfile definition included in the Git repo
  * Push the container up to hub.docker.com using the credentials contained in the secrets file
5. Drone checks the _.drone.yml_ file and executes the commands in the _deploy_ phase. In this phase, the following actions will take place: 
  * Drone sends a WebHook command to Marathon to cause an application restart
  * Marathon pulls the new container from hub.docker.com containing the code changes
6. Drone checks the _.drone.yml_ file and executes the commands in the _notify_ phase. During notification, Drone will check the staus of the build and then send notifications to the Spark room.
  * If the build was successful, send a Success notification
  * If the build failed, send a Failure notificiation and blame someone.

---
 

[item]: # (slide)

## Let's put that pipeline to use!

Now that we have our full CICD pipeline up and running, let's make some application changes and watch them deploy.

[item]: # (/slide)

[item]: # (slide)

## Verify that the default app status

The default demo app has a single route `hello/world`.  We will be adding an additional route to the application, but let's verify that it doesn't exist already.

[item]: # (/slide)

[item]: # (slide)

1. Try to navigate to `hello/universe` at your application.  This would be available at a URL such as `http://class-USERNAME.mantl.domain.com/hello/universe`, but replace **USERNAME** with your Docker Username, and **mantl.domain.com** with the Lab Application Domain provided by your instructor.

[item]: # (/slide)

[item]: # (slide)

2. You should get back a page not found error.

    ![App Hello Universe Error](images/app_hello_universe_error.png)

[item]: # (/slide)

[item]: # (slide)

## Add /hello/universe to the app

[item]: # (/slide)

**_In this step you will be entering several commands in a terminal window.  These need to be run from your local repo directory.  If you followed the directions when cloning the repo locally, this command will place you in the correct directory_**

```
cd ~/coding/cicd_demoapp
```

[item]: # (slide)

## Build the Test

1. In your editor or IDE, open `testing.py` and add the new tests for /hello/universe.  You can simply copy and paste the below into your editor.

[item]: # (/slide)

[item]: # (slide)

```
import demoapp
import unittest


class FlaskTestCase(unittest.TestCase):

    def setUp(self):
        demoapp.app.config['TESTING'] = True
        self.app = demoapp.app.test_client()

    def test_correct_http_response(self):
        resp = self.app.get('/hello/world')
        self.assertEquals(resp.status_code, 200)

    def test_correct_content(self):
        resp = self.app.get('/hello/world')
        self.assertEquals(resp.data, '"Hello World!"\n')

    def test_universe_correct_http_response(self):
        resp = self.app.get('/hello/universe')
        self.assertEquals(resp.status_code, 200)

    def test_universe_correct_content(self):
        resp = self.app.get('/hello/universe')
        self.assertEquals(resp.data, '"Hello Universe!"\n')

    def tearDown(self):
        pass

if __name__ == '__main__':
    unittest.main()

```

[item]: # (/slide)

[item]: # (slide)

2. Since we are **NOT** changing the actual build process stored in `.drone.yml`, we don't need to recreate the secrets file.  Once the build process is complete, as long as new tests or steps aren't added these files can be left alone.  Developers can focus on their code and changes, not the build process.

[item]: # (/slide)

[item]: # (slide)

3. Commit and push our updated test to begin the CICD process.

    ```
    # add the file to the git repo
    git add testing.py

    # commit the change
    git commit -m "Added new Tests for hello/universe"

    # push changes to GitHub
    git push
    ```

[item]: # (/slide)

[item]: # (slide)

4. Check the drone server to check the status of the build.  You can also check in Spark for Status updates.  

    ![Drone Build](images/drone_5-5_build.png)

[item]: # (/slide)

[item]: # (slide)

5. The build failed because we built the test **BEFORE** the feature.  That's test driven development, and the failure is exactly what we want to see.  When drone fails in the testing, the new container isn't built and published.  

[item]: # (/slide)

[item]: # (slide)

### Build the Feature

1. In your editor or IDE, open `demoapp.py` and add the new class and resource for HelloUniverse.  You can simply copy and paste the below into your editor.

[item]: # (/slide)

[item]: # (slide)

```
from flask import Flask, request
from flask_restful import Resource, Api, reqparse


app = Flask(__name__)
app = Flask(__name__)
api = Api(app)

class HelloWorld(Resource):
    def get(self):
        text = "Hello World!"
        return text

api.add_resource(HelloWorld, '/hello/world')

class HelloUniverse(Resource):
    def get(self):
        text = "Hello Universe!"
        return text

api.add_resource(HelloUniverse, '/hello/universe')

if __name__ == '__main__':
    # Run Flask
    app.run(debug=True, host='0.0.0.0', port=int("5000"))

```

[item]: # (/slide)

[item]: # (slide)

2. Since we are **NOT** changing the actual build process stored in `.drone.yml`, we don't need to recreate the secrets file.  Once the build process is complete, as long as new tests or steps aren't added these files can be left alone.  Developers can focus on their code and changes, not the build process.

[item]: # (/slide)

[item]: # (slide)

3. Commit and push our updated application to begin the CICD process.

    ```
    # add the file to the git repo
    git add demoapp.py

    # commit the change
    git commit -m "Added hello/universe to the application"

    # push changes to GitHub
    git push
    ```

[item]: # (/slide)

[item]: # (slide)

4. Check the drone server to verify the build has begun.  You can also monitor the Spark room for the completed message.

    ![Drone Build](images/drone_6th_build.png)

[item]: # (/slide)

[item]: # (slide)

5. Once drone completes the build, check Marathon and watch as the application restarts.

    ![Marathon App Restart](images/marathon_app_restart.png)

[item]: # (/slide)

[item]: # (slide)

6. Wait for Marathon to show the application as healthy.

    ![Marathon App Healthy](images/marathon_app_healthy.png)

[item]: # (/slide)

[item]: # (slide)

7. Refresh the web page with the `hello/universe` page **Not Found**.  You should now have your new message available.

    ![App Hello Universe](images/app_hello_universe.png)

[item]: # (/slide)

[item]: # (slide)

8. Feel free to experiment with more changes to the application.  Each commit will result in the application restarting with the updated code.

[item]: # (/slide)

[item]: # (slide)

## Current Build Pipeline Status

![Final Diagram](images/stage_final_diagram.png)

[item]: # (/slide)

Okay, so let's review the steps in the full pipeline.

1. You committed and pushed code to GitHub.com
2. GitHub sent a WebHook to the Drone server notifying it of the committed code.
3. Drone checks the _.drone.yml_ file and executes the commands in the _build_ phase. During this phase, Drone has two steps: 
	* *build_starting* 	
  		* Fetch a container from hub.docker.com to begin the build.  This container is identified in the `image: python:2` line of the drone config file.  Drone will run the commands listed in this section
	* *run_tests*
  		* Fetch a container from hub.docker.com to do the testing.  This container is identified in the `image: python:2-alpine` line of the drone config file.  Drone will run the tests described by the commands listed in this section

4. Drone checks the _.drone.yml_ file and executes the commands in the _publish_ phase. During this phase, Drone will: 
  * Build a Docker Container using the Dockerfile definition included in the Git repo
  * Push the container up to hub.docker.com using the credentials contained in the secrets file
5. Drone checks the _.drone.yml_ file and executes the commands in the _deploy_ phase. In this phase, the following actions will take place: 
  * Drone sends a WebHook command to Marathon to cause an application restart
  * Marathon pulls the new container from hub.docker.com containing the code changes
6. Drone checks the _.drone.yml_ file and executes the commands in the _notify_ phase. During notification, Drone will check the staus of the build and then send notifications to the Spark room.
  * If the build was successful, send a Success notification
  * If the build failed, send a Failure notificiation and blame someone.

--- 
[item]: # (slide)

# CleanUp - Uninstall your Application
[アジェンダに戻る](https://github.com/radiantmarch/cicd_learning_lab#lab-agenda)

[item]: # (/slide)

In this lab we built and put to use a full CICD pipeline for a very simple application.  Full production application pipelines are more involved with a greater amount of testing to ensure only good code is deployed, but the basic steps and process are the same.

[item]: # (slide)

## Uninstall your application

[item]: # (/slide)

**_In this step you will be entering several commands in a terminal window.  These need to be run from your local repo directory.  If you followed the directions when cloning the repo locally, this command will place you in the correct directory_**

```
cd ~/coding/cicd_demoapp
```

[item]: # (slide)

From the root of your code repository...

1. Execute the uninstallation script.

[item]: # (/slide)

[item]: # (slide)

```
$ ./app_uninstall.sh

# You will be prompted to enter the information needed before the applicaiton deploys.
# The process will look like this

Please provide the following details on your lab environment.

What is the address of your Mantl Control Server?
eg: control.mantl.internet.com
control.sandbox.imapex.io

What is the username for your Mantl account?
<当日講師が説明>

What is the password for your Mantl account?
<当日講師が説明>

What is the your Docker Username?
(自分のユーザ名)

Uninstalling the demoapp at class/hpreston
{"version":"2016-07-01T13:26:43.421Z","deploymentId":"731e0df3-056c-4340-ae03-f5155a5e3b50"}

```

[item]: # (/slide)

[item]: # (slide)

2. The application has now been destroyed.  

[item]: # (/slide)

[item]: # (slide)

## Wasn't that Fun?

![](http://imapex.io/images/imapex_standing_text_sm.png)

[item]: # (/slide)


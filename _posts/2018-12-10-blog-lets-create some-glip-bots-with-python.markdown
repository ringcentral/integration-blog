---
layout: post
title:  "Let's create some glip bots with python"
date:   2018-12-10 13:31:07 +0800
categories: blog
---

![ ](https://github.com/zxdong262/ringcentral-chatbot-factory-py/blob/master/screenshots/wanted.jpg?raw=true)

This is a intro blog about my recent work: [ringcentral-chatbot-python](https://github.com/zxdong262/ringcentral-chatbot-python).

The project is part of our chatbot plan: Make writing a glip chatbot easy and fun. It is not mature enough for public yet, still need more documents and adding more features.

It do works and is really easy to work with, you do not even need to know any python to get it running and talking. So ringcentral developers, let's create some chatbot with python.

## Table of contents <!-- omit in toc -->

- [categories: blog](#categories-blog)
- [Prerequisites](#prerequisites)
- [Getting started](#getting-started)
- [Test bot](#test-bot)
- [Let's write some bot logic](#lets-write-some-bot-logic)
- [Example bots](#example-bots)
- [Deploy to AWS Lambda](#deploy-to-aws-lambda)
- [Use Extensions](#use-extensions)
- [Write a extension yourself](#write-a-extension-yourself)
- [Credits](#credits)

## Prerequisites

- Python3.6+ and Pip3
- Nodejs 8.10+/npm, recommend using [nvm](https://github.com/creationix/nvm) to install nodejs/npm
- Create the bot App: Login to [developer.ringcentral.com](https://developer.ringcentral.com) and create an `public` `Server/Bot` app with permissions: `ReadContacts, ReadMessages, ReadPresence, Contacts, ReadAccounts, SMS, InternalMessages, ReadCallLog, ReadCallRecording, WebhookSubscriptions, Glip`

## Getting started

```bash
pip3 install ringcentral_chatbot_factory

rcf my-ringcentral-chat-bot
# then carefully answer all questions, then the my-app folder will be created

# follow the instruction of my-app/README.md to dev/run/test the bot
cd my-ringcentral-chat-bot

# use virtualenv
pip3 install virtualenv # might need sudo

# init virtual env
virtualenv venv --python=python3

# use env
source ./venv/bin/activate

# install deps
pip install -r requirements.txt
npm i

# run ngrok proxy
# since bot need https server,
# so we need a https proxy for ringcentral to visit our local server
./bin/proxy
# will show:
# Forwarding https://xxxxx.ngrok.io -> localhost:8989

# create env file
cp .sample.env .env
# then edit .env, set proper setting,
# and goto your ringcentral app setting page, set OAuth Redirect URI to https://https://xxxxx.ngrok.io/bot-oauth
RINGCENTRAL_BOT_SERVER=https://xxxxx.ngrok.io

## for bots auth required, get them from your ringcentral app page
RINGCENTRAL_BOT_CLIENT_ID=
RINGCENTRAL_BOT_CLIENT_SECRET=

# create custom bot config file
# since all functions are optional, you could keep those you edited, delete others in config.py
cp config.sample.py config.py

# run local dev server
./bin/start
```

## Test bot

- Goto your ringcentral app's bot section, click 'Add to glip'
- Login to [https://glip-app.devtest.ringcentral.com](https://glip-app.devtest.ringcentral.com), find the bot by searching its name. Talk to the bot.
- Edit config.py to change bot bahavior and test in [https://glip-app.devtest.ringcentral.com](https://glip-app.devtest.ringcentral.com)

## Let's write some bot logic

let's write a bot that can tell server time/date as a hello world bot

Edit `config.py`, like this:

```py
"""
sample config module
run "cp config.sample.py config.py" to create local config
edit config.py functions to override default bot behavior
"""

from datetime import datetime

__name__ = 'localConfig'
__package__ = 'ringcentral_bot_framework'

def helpMsg(botId):
  return f'''Hello, I am a date/time chatbot. Please reply "@![:Person]({botId}) **cmd**" if you want to talk to me.

**cmd** list

**date** -- show current date
**time** -- show current time
  '''

def botJoinPrivateChatAction(bot, groupId, user, dbAction):
  """
  bot join private chat event handler
  bot could send some welcome message or help, or something else
  """
  text = helpMsg(bot.id)
  bot.sendMessage(
    groupId,
    {
      'text': text
    }
  )

def botGotPostAddAction(
  bot,
  groupId,
  creatorId,
  user,
  text,
  dbAction,
  handledByExtension
):
  """
  bot got group chat message: text
  bot could send some response
  """
  if handledByExtension:
    return

  if text == f'![:Person]({bot.id}) date':
    date = str(datetime.now().strftime('%Y-%m-%d'))
    bot.sendMessage(
      groupId,
      {
        'text': f'![:Person]({creatorId}), current date is {date}'
      }
    )
  elif text == f'![:Person]({bot.id}) time':
    time = str(datetime.now().strftime('%Y-%m-%d %H:%M:%S'))
    bot.sendMessage(
      groupId,
      {
        'text': f'![:Person]({creatorId}), current time is {time}'
      }
    )
  elif f'![:Person]({bot.id})' in text:
    bot.sendMessage(
      groupId,
      {
        'text': helpMsg(bot.id)
      }
    )


```

Then try talk to the bot in [https://glip-app.devtest.ringcentral.com](https://glip-app.devtest.ringcentral.com), see how it work.

That is it. You can just focus on the bot logic.

If you want to access user data, subscribe to user event, or use different database, just check these examples.

## Example bots

- [date-time-chatbot](https://github.com/zxdong262/ringcentral-date-time-chatbot) : Simple ringcentral chatbot which can tell time/date.
- [assistant-bot](https://github.com/zxdong262/ringcentral-assistant-bot) : Simple assistant Glip bot to show user/company information, this bot will show you how to access user data.
- [survey-bot](https://github.com/zxdong262/ringcentral-survey-bot) : Example survey bot, this bot will show you how to create/use custom database wrapper.

## Deploy to AWS Lambda

If you feel ok about the bot, time to deploy to production server, I would suggest AWS Lambda for these reasons:

1. Free tier gives 1000000 free run per month
2. Https ready, do not need extra efforts to handle domain/https staff
3. Very stable and reliable

*Be aware that AWS Lambda **ONLY works in linux** on an x64 architecture. For **non-linux os**, we need **docker** to build dependencies, should [install docker](https://docs.docker.com/docker-for-mac/) first.

Get an AWS account, create `aws_access_key_id` and `aws_secret_access_key` and place them in `~/.aws/credentials`, like this:

```bash
[default]
aws_access_key_id = <your aws_access_key_id>
aws_secret_access_key = <your aws_secret_access_key>
```

For more information, refer to https://docs.aws.amazon.com/general/latest/gr/aws-security-credentials.html

```bash
cp dev/lambda/serverless.sample.yml devlambda/serverless.yml
```

Edit `dev/lambda/serverless.yml`, and make sure you set the proper name and required env.

```yml
# you can define service wide environment variables here
  environment:
    NODE_ENV: production
    # ringcentral apps

    ## bots
    RINGCENTRAL_BOT_CLIENT_ID:
    RINGCENTRAL_BOT_CLIENT_SECRET:

    ## if you only have bot app, it is not needed
    RINGCENTRAL_USER_CLIENT_ID: xxxx
    RINGCENTRAL_USER_CLIENT_SECRET: xxxx

    ## common
    RINGCENTRAL_SERVER: https://platform.devtest.ringcentral.com
    RINGCENTRAL_BOT_SERVER: https://xxxx.execute-api.us-east-1.amazonaws.com/default/poc-your-bot-name-dev-bot

    # db
    DB_TYPE: dynamodb
    DYNAMODB_TABLE_PREFIX: rc_bot2
    DYNAMODB_REGION: us-east-1

```

Deploy to AWS Lambda with `bin/deploy`

```bash
# install serverless related modules
npm i

# Run this cmd to deploy to AWS Lambda, full build, may take more time
bin/deploy

## watch Lambda server log
bin/watch

```

- Create API Gateway for your Lambda function, shape as `https://xxxx.execute-api.us-east-1.amazonaws.com/default/poc-your-bot-name-dev-bot/{action+}`
- Make sure your Lambda function role has permission to read/write dynamodb(Set this from AWS IAM roles, could simply attach `AmazonDynamoDBFullAccess` and `AWSLambdaRole` policies to Lambda function's role)
- Make sure your Lambda function's timeout more than 5 minutes
- Do not forget to set your RingCentral app's redirect URL to Lambda's API Gateway URL, `https://xxxx.execute-api.us-east-1.amazonaws.com/default/poc-your-bot-name-dev-bot/bot-oauth` for bot app.

## Use Extensions

RingCentral Chatbot Framework for Python Extensions will extend bot command support with simple setting in `.env`.

Just set like this in `.env`

```bash
EXTENSIONS=ringcentral_bot_framework_extension_botinfo,ringcentral_bot_framework_extension_some_other_extension
```

And install these exetnsions by `pip install ringcentral_bot_framework_extension_botinfo ringcentral_bot_framework_extension_some_other_extension`, it is done.

![ ](https://github.com/zxdong262/ringcentral-chatbot-python-ext-bot-info/raw/master/screenshots/ss.png)

You can search for more extension in [pypi.org](https://pypi.org) with keyword `ringcentral_bot_framework_extension`.

## Write a extension yourself

Writing one extension will be simple, just check out [botinfo extension](https://github.com/zxdong262/ringcentral-chatbot-python-ext-bot-info) as an example, you just need to write one function `botGotPostAddAction` there.

Make sure you follow the naming rule: starts with `ringcentral_bot_framework_extension_` and publish to pypi.org and it is done!

```python
# botinfo extension's source code
# https://github.com/zxdong262/ringcentral-chatbot-python-ext-bot-info/blob/master/ringcentral_bot_framework_extension_botinfo/__init__.py
import json

name = 'ringcentral_bot_framework_extension_botinfo'

def botGotPostAddAction(
  bot,
  groupId,
  creatorId,
  user,
  text,
  dbAction
):
  """
  bot got group chat message: text
  bot extension could send some response
  return True when bot send message, otherwise return False
  """
  if not f'![:Person]({bot.id})' in text:
    return False

  if 'bot info' in text:
    botInfo = bot.platform.get('/account/~/extension/~')
    txt = json.loads(botInfo.text())
    txt = json.dumps(txt, indent=2)
    msg = f'![:Person]({creatorId}) bot info json is:\n' + txt

    bot.sendMessage(
      groupId,
      {
        'text': msg
      }
    )
    return True
  else:
    return False
```

## Credits

- The core bot framework logic is implanted from [ringcentral-ai-bot](https://github.com/ringcentral-tutorials/ringcentral-ai-bot) written by [@tylerlong](https://github.com/tylerlong)

- Also check tyler's js bot framework [https://github.com/tylerlong/ringcentral-chatbot-js](https://github.com/tylerlong/ringcentral-chatbot-js)

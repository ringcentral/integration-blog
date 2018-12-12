---
layout: post
title:  "Let's create some glip bots with python"
date:   2018-12-10 13:31:07 +0800
categories: blog
author: Drake Zhao
---

![ ](https://github.com/zxdong262/ringcentral-chatbot-factory-py/blob/master/screenshots/wanted.jpg?raw=true)

This is a intro blog about my recent work: [ringcentral-chatbot-python](https://github.com/zxdong262/ringcentral-chatbot-python).

The project is part of our chatbot plan: make writing a glip chatbot easy and fun. It is not mature enough for public yet, still need to add more documents and features.

It does work and is really easy to work with, you do not even need to know any python to get it running and talking. So RingCentral developers, let's create some chatbot with python, and give some feedback.

## Prerequisites

- Python3.6+ and Pip3
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

# install required modules
pip install -r requirements.txt

# run ngrok proxy
# since bot need https server,
# so we need a https proxy for RingCentral to visit our local server
./bin/proxy
# will show:
# Forwarding https://xxxxx.ngrok.io -> localhost:9890

# create env file
# .env already created from .sample.env
# just edit .env, set proper setting,
RINGCENTRAL_BOT_SERVER=https://xxxxx.ngrok.io

## for bots auth required, get them from your RingCentral app page
RINGCENTRAL_BOT_CLIENT_ID=
RINGCENTRAL_BOT_CLIENT_SECRET=

# and goto your RingCentral app setting page, set OAuth Redirect URI to https://https://xxxxx.ngrok.io/bot-oauth

# run local dev server
./bin/start
```

## Test bot

- Goto your RingCentral app's bot section, click 'Add to glip'
- Login to [https://glip-app.devtest.ringcentral.com](https://glip-app.devtest.ringcentral.com), find the bot by searching its name. Talk to the bot.
- Edit config.py to change bot bahavior and test in [https://glip-app.devtest.ringcentral.com](https://glip-app.devtest.ringcentral.com)

## Let's write some bot logic

let's write a bot that can tell server time/date as a hello world bot.

First, create bot config file by `cp config.sample.py config.py`

Then Edit `config.py`, like this:

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

- [date-time-chatbot](https://github.com/zxdong262/ringcentral-date-time-chatbot): simple RingCentral chatbot that can tell server time/date.
- [assistant-bot](https://github.com/zxdong262/ringcentral-assistant-bot): simple assistant Glip bot to show user/company information, this bot will show you how to access user data.
- [survey-bot](https://github.com/zxdong262/ringcentral-survey-bot): example survey bot, this bot will show you how to create/use custom database wrapper.

## Deploy to AWS Lambda

If you are satisfied with the bot, time to deploy to production server, I would suggest AWS Lambda for these reasons:

1. Free tier gives 1000000 free run per month
2. Https ready, do not need extra efforts to handle domain/https
3. Very stable and reliable

*Be aware that AWS Lambda **ONLY works in linux** on an x64 architecture. For **non-linux os**, we need **docker** to build dependencies, should [install docker](https://docs.docker.com/docker-for-mac/) first.

This requires Nodejs 8.10+/npm, recommend using [nvm](https://github.com/creationix/nvm) to install nodejs/npm.

Get an AWS account, create `aws_access_key_id` and `aws_secret_access_key` and place them in `~/.aws/credentials`, like this:

```bash
[default]
aws_access_key_id = <your aws_access_key_id>
aws_secret_access_key = <your aws_secret_access_key>
```

For more information, refer to [https://docs.aws.amazon.com/general/latest/gr/aws-security-credentials.html](https://docs.aws.amazon.com/general/latest/gr/aws-security-credentials.html)

```bash
cp dev/lambda/serverless.sample.yml dev/lambda/serverless.yml
```

Edit `dev/lambda/serverless.yml`, and make sure you set the proper name and required env.

```yml
# you can define service wide environment variables here
  environment:
    ENV: production
    # ringcentral apps

    ## for bots auth, required
    RINGCENTRAL_BOT_CLIENT_ID:
    RINGCENTRAL_BOT_CLIENT_SECRET:

    ## for user auth, could be empty if do not need user auth
    RINGCENTRAL_USER_CLIENT_ID:
    RINGCENTRAL_USER_CLIENT_SECRET:

    ## common
    RINGCENTRAL_SERVER: https://platform.devtest.ringcentral.com
    RINGCENTRAL_BOT_SERVER: https://xxxxx.execute-api.us-east-1.amazonaws.com/dev

    # db
    DB_TYPE: dynamodb
    DYNAMODB_TABLE_PREFIX: ringcentral-bot
    DYNAMODB_REGION: us-east-1
    DYNAMODB_ReadCapacityUnits: 1
    DYNAMODB_WriteCapacityUnits: 1

```

Deploy to AWS Lambda with `bin/deploy`

```bash
# Run this cmd to deploy to AWS Lambda
bin/deploy
```

After successful deploy, you will get the https api url:

```bash
Service Information
service: ringcentral-bot
stage: dev
region: us-east-1
stack: ringcentral-bot-dev
api keys:
  None
endpoints:
  ANY - https://dddddd.execute-api.us-east-1.amazonaws.com/dev/{action+}
  GET - https://dddddd.execute-api.us-east-1.amazonaws.com/dev/
```

Relpace `RINGCENTRAL_BOT_SERVER: https://xxxxx.execute-api.us-east-1.amazonaws.com/dev` in serverless.yml with
`RINGCENTRAL_BOT_SERVER: https://dddddd.execute-api.us-east-1.amazonaws.com/dev`
 and run `bin/deploy` to deploy again.

Watch Lambda server log by run:

```bash
bin/watch
```

Do not forget to set your RingCentral app's redirect URL to Lambda's API Gateway URL, `https://dddddd.execute-api.us-east-1.amazonaws.com/dev/bot-oauth` for bot app.

## Use Extensions

RingCentral Chatbot Framework for Python Extensions will extend bot command support with simple settings in `.env`.

Just set like this in `.env`

```bash
EXTENSIONS=ringcentral_bot_framework_extension_botinfo,ringcentral_bot_framework_extension_some_other_extension
```

And install these exetnsions by `pip install ringcentral_bot_framework_extension_botinfo ringcentral_bot_framework_extension_some_other_extension`, it is done.

![ ](https://github.com/zxdong262/ringcentral-chatbot-python-ext-bot-info/raw/master/screenshots/ss.png)

You can search for more extension in [pypi.org](https://pypi.org) with keyword `ringcentral_bot_framework_extension`.

## Write a extension yourself

Writing extension is simple, just check out [botinfo extension](https://github.com/zxdong262/ringcentral-chatbot-python-ext-bot-info) as an example, you just need to write one function `botGotPostAddAction` there.

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

- Also check tyler's js bot framework [https://github.com/ringcentral/ringcentral-chatbot-js](https://github.com/ringcentral/ringcentral-chatbot-js)

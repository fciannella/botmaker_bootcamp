# HelloWorld Bot

This is a very basic example of a bot that uses no models and is the most basic bot that one can build.

Here is how to run it:

## Prepare the environment

We need to have the botmaker quickscripts downloaded on a machine with docker and GPU available.

```shell
ngc registry resource download-version "ea-botmaker/release/bot_maker_quickstart:2.0.0-ea"
```

Once the quickscripts have been downloaed, you cen cd to the directory and get the git repo:

```shell
cd /mnt/nvdl/usr/fciannella/src/botmaker_ea2/bot_maker_quickstart_v2.0.0-ea
git clone ssh://git@gitlab-master.nvidia.com:12051/fciannella/botmaker_bootcamp.git
```

Finally you can run the bot in text mode to start with:

```shell
bash run_bot_launcher.sh -c botmaker_bootcamp/01_hello_world_bot/nlu_data_match_policy_bot  -m cli
```

Tha above command will look for the one file ending in `_bot_config.yaml`, inside the directory specified in the above command. So make sure that you name the main configuration file that way when you will build your own bot. 

Here is a sample of the interaction that you can have with this primitive both:

```text
FM details
         
         +---+------+-----+--------+
         | # | Name | URL | Status |
         +---+------+-----+--------+
         
         
NLP model details
         
         +------+------+-----+-------------+--------+
         | Name | Type | URL | Model Paths | Status |
         +------+------+-----+-------------+--------+
         
         
Bot started in cli mode successfully. Type queries below to interact.
[YOU] ok
[BOT] can i help you with anything else?
[YOU] Thanks
[BOT] The pleasure is mine!
[YOU] Thanks
[BOT] You are most welcome!
[YOU] Have a good day
[BOT] You as well!
[YOU] bla
[BOT] I don't know how to answer these type of questions yet.
[YOU] 
```

All that is happening is very easy to understand if you look at the files. Those are the mimimal files you need to run a bot.

The config yaml file (`hello_world_bot_config.yaml`) that you pass to the `run_bot_launcher.sh` command (actually it is automatically detected inside the directory that you pass to the command), is where you define the main component of your bot.

In this case it looks like this:

```yaml
bot: hello_world

configs:
    language: en-US
    health_check_interval: 750

policies:
  - name: custom_command_policy
    paths:    # Custom command tsv file paths
      - nlu_data_match_policy_bot_custom_commands.tsv

  - name: dialog_policy
    paths:
      - ./default.yaml
```

As you can see, you define the policies that need to be executed in order.

The dialog_policy is mandatory (not sure why, but things will not work if you use just the custom_command_policy).

Here is a list of the possible [policies](https://docs.google.com/document/d/18FyCecxoACOkZL9SH65cvZEtA47WnM9lNMHsKtjus8c/edit#heading=h.8ihktxvivfeo) that can be defined:

```text
custom_command_policy
knowledge_base_policy
nlu_data_match_policy
faq_kb_policy
fallback rules
dialog_policy
irqa_policy
```

We will study them all one by one in the next modules.




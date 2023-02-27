# NLU Data Match Policy Bot

In this example we will explore another policy, which will allow us to exploit some basic semantic similarity features, without having to use any Deep Learning module yet.

Here is how to run it:

## Prepare the environment

We need to have the botmaker quickscripts downloaded on a machine with docker and GPU available. You only need to do this the first time!

```shell
ngc registry resource download-version "ea-botmaker/release/bot_maker_quickstart:2.0.0-ea"
```

Once the quickscripts have been downloaed, you cen cd to the directory and get the git repo:

```shell
cd /mnt/nvdl/usr/fciannella/src/botmaker_ea2/bot_maker_quickstart_v2.0.0-ea
git clone ssh://git@gitlab-master.nvidia.com:12051/fciannella/botmaker_bootcamp.git
```

## Initialize botmaker:

You need to initialize botmaker. This will download the docker containers, so you need to do this only the first time.

```shell
bash bot_maker_init.sh
```

## Run the bot

Finally you can run the bot in text mode to start with:

```shell
bash run_bot_launcher.sh -c botmaker_bootcamp/03_using_dl_models_domain_intent_slot/using_dl_models_domain_intent_slot_configs  -m cli
```
The above command will look for a file which ends in `_bot_config.yaml` so make sure that that is how you name your botmaker configuration file.


# Extra Configurations

We have added some variations in the configurations so that we can experiment with different models. In the directory `extra_configs` you will find a config file called `intent_slot_bot_config.yaml`. You can copy that file one level up and onto the `bot_config.yaml` so that there is only one configuration file in the main directory.

You can use this command from insice the `03_semantic_similarity_bot` directory:

```shell
cp using_dl_models_domain_intent_slot_configs/all_configs/intent_slot_bot_config.yaml using_dl_models_domain_intent_slot_configs/03_bot_config.yaml
```


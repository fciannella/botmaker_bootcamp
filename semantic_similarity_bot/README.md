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
bash run_bot_launcher.sh -c botmaker_bootcamp/semantic_similarity_bot/semantic_similarity_bot_configs  -m cli
```
The above command will look for a file which ends in `_bot_config.yaml` so make sure that that is how you name your botmaker configuration file. 
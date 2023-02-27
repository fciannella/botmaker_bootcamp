# NLU Data Match Policy Bot

In this example we will explore another policy, which will allow us to exploit some basic semantic similarity features, without having to use any Deep Learning module yet.

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

You need to initialize botmaker:

```shell
bash bot_maker_init.sh
```

Finally you can run the bot in text mode to start with:

```shell
bash run_bot_launcher.sh -c botmaker_bootcamp/02_nlu_dmp_bot/nlu_dmp_bot_configs  -m cli
```

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
[YOU] Hello
[BOT] Hi there
[YOU] share your location with me
[BOT] I live at Nvidia Headquarters in Santa Clara CA.
[YOU] i wanna hear a joke
[BOT] I'm not really that funny.
[YOU] 
```

We have added a few things to the configuration. Let's look at it:

```yaml
bot: nlu_dmp

configs:
    language: en-US
    health_check_interval: 750

policies:
  - name: custom_command_policy
    paths:    # Custom command tsv file paths
      - nlu_dmp_bot_custom_commands.tsv

  - name: nlu_data_match_policy
    paths:    # Paths to Nemo-formatted NLU training data
      - nlu_data/smalltalk

  - name: dialog_policy
    paths:
      - smalltalk/domain_smalltalk.yaml
      - ./default.yaml
```

The first novelty is the nlu_data_match_policy. This policy uses tsv files that are inside the directory `nlu_data/smalltalk` to find domain / intent / entities / slots.

At the moment this works on a full match basis. For instance if the user's utterance is `i wanna hear a joke`, you can see that this maps to `i wanna hear a joke	30` in the `train.tsv` file. The number `30` is the identifier of the intent, which is expressed in the `dict.intents.csv` file: `smalltalk.personality_im_not_really_that_funny` (corresponds to line 31 in that file: there is an offset of 1 in the indices in that there is a header in `train.csv`). 
The same thing applies for the slots. If for instance you look at line `8` of the file `train_slots.tsv`, it looks like this: `0 0 1 2`. The slots are defined in the file `dict.slots.csv`:

```text
O
B-bodyaction
I-bodyaction
```
So if we look at the line `9` of the file `train.tsv` (remember the offset), it is: `can you throw up	0`. So basically this means this mapping:

```text
can: O
you: O
throw: B-bodyaction
up: I-bodyaction
```
Let's look at a debugging session:

```shell
fciannella@nvdl-a112-asus02:/mnt/nvdl/usr/fciannella/src/botmaker_ea2/bot_maker_quickstart_v2.0.0-ea$ bash run_bot_launcher.sh -c botmaker_bootcamp/02_nlu_dmp_bot/nlu_dmp_bot_configs  -m cli -db --dm-log-level DEBUG
Bot-Maker Version : 2.0.0-ea-x86_64
...
2023-02-22 19:54:11,213 [INFO] Getting models for deployment from bot config /mnt/nvdl/usr/fciannella/src/botmaker_ea2/bot_maker_quickstart_v2.0.0-ea/botmaker_bootcamp/02_nlu_dmp_bot/nlu_dmp_bot_configs/nlu_dmp_bot_config.yaml
2023-02-22 19:54:11,215 [INFO] Skipping Speech models for deployment
...
Debug mode status true
Starting Bot editor..
Starting Dialog Manager..
docker run --user 41975:30 -i -t --rm --net host --env OPENAPI_VALIDATION=false -v /mnt/nvdl/usr/fciannella/src/botmaker_ea2/bot_maker_quickstart_v2.0.0-ea:/mnt/nvdl/usr/fciannella/src/botmaker_ea2/bot_maker_quickstart_v2.0.0-ea --name dialog-manager nvcr.io/ea-botmaker/release/dialog-manager:2.0.0-ea-x86_64 -m cli -to 120 -w 1 -c /mnt/nvdl/usr/fciannella/src/botmaker_ea2/bot_maker_quickstart_v2.0.0-ea/botmaker_bootcamp/02_nlu_dmp_bot/nlu_dmp_bot_configs/nlu_dmp_bot_config.yaml -p 5000 --log_root /mnt/nvdl/usr/fciannella/src/botmaker_ea2/bot_maker_quickstart_v2.0.0-ea/log -db --dm-log-level DEBUG
Absolute path: /mnt/nvdl/usr/fciannella/src/botmaker_ea2/bot_maker_quickstart_v2.0.0-ea/botmaker_bootcamp/02_nlu_dmp_bot/nlu_dmp_bot_configs/nlu_dmp_bot_config.yaml detected for config file.
Starting Bot in cli mode..
...
2023-02-22 19:54:12,641 main INFO Creating DM App instance
2023-02-22 19:54:12,641 create_core INFO Creating DM App instance
...
2023-02-22 19:54:13,012 __init__ DEBUG Initialized subintent classification module using zero shot pipelines..
2023-02-22 19:54:13,012 __init__ DEBUG Initialized manager for zero shot pipelines..
2023-02-22 19:54:13,012 __init__ DEBUG Successfully initialized slot manager module.
2023-02-22 19:54:13,012 __init__ DEBUG Successfully initialized slot linking module..
2023-02-22 19:54:13,012 __init__ DEBUG Initialized slot resolution module
FM details
         
         +---+------+-----+--------+
         | # | Name | URL | Status |
         +---+------+-----+--------+
         
         
2023-02-22 19:54:13,012 print_fm_details INFO FM details
         
         +---+------+-----+--------+
         | # | Name | URL | Status |
         +---+------+-----+--------+
         
         
NLP model details
         
         +------+------+-----+-------------+--------+
         | Name | Type | URL | Model Paths | Status |
         +------+------+-----+-------------+--------+
         
         
2023-02-22 19:54:13,013 print_nlp_details INFO NLP model details
         
         +------+------+-----+-------------+--------+
         | Name | Type | URL | Model Paths | Status |
         +------+------+-----+-------------+--------+
         
         
Bot started in cli mode successfully. Type queries below to interact.
[YOU] can you throw up
2023-02-22 19:54:18,311 __get_user_data INFO Failed to get user data for user id 11005d36-f6f3-404a-be9b-0c22b4a1ee6e due to error 'Invalid user id 11005d36-f6f3-404a-be9b-0c22b4a1ee6e'. Creating new user_data instance.
2023-02-22 19:54:18,312 __preprocess_user_request INFO New session: f29d346f-276f-4303-8f6b-1272a20f4a89 created for user.
2023-02-22 19:54:18,312 reset_dialog_state INFO Clearing memory type: 
2023-02-22 19:54:18,312 reset_dialog_state INFO Clearing all fulfillment slots, fulfillment memory and dialog graph state.
2023-02-22 19:54:18,312 reset_dialog_state INFO Resetting both longterm and shortterm slots in memory.
2023-02-22 19:54:18,312 clean_text DEBUG Text after removing unicode characters: can you throw up
2023-02-22 19:54:18,313 __preprocess_user_request DEBUG User query after preprocessing: can you throw up
2023-02-22 19:54:18,313 __preprocess_user_request DEBUG Request Json: Language: en-US, ResponseLanguage: en-US, UserId: 11005d36-f6f3-404a-be9b-0c22b4a1ee6e, Query: can you throw up, QueryId: 4b9cc471-8809-4f01-bf9b-6a2c769a9906, SessionId: f29d346f-276f-4303-8f6b-1272a20f4a89, Domain: none, Intent: none, Event: none, Entities: [], Parameters: {}
2023-02-22 19:54:18,313 preprocess_user_request DEBUG Found defined processor: preprocessed, Processed Query: can you throw up
2023-02-22 19:54:18,313 process_query INFO Executing policy: custom_command_policy
2023-02-22 19:54:18,313 process_query DEBUG CustomParser::process called with transcript::can you throw up
2023-02-22 19:54:18,313 process_query INFO Custom parser module response: No match found
2023-02-22 19:54:18,313 process_query INFO Executing policy: nlu_data_match_policy
2023-02-22 19:54:18,313 process_query DEBUG Exact match found in NLU data match policy.
2023-02-22 19:54:18,313 process_query INFO Found nlp response from data match policy: Domain smalltalk Intent: smalltalk.personality_i_dont_have_a_body Entities: [{'Token': 'throw', 'EntityName': 'bodyaction', 'Span': {'start': 8, 'end': 12}}, {'Token': 'up', 'EntityName': 'bodyaction', 'Span': {'start': 14, 'end': 15}}]Anchor: can you throw up
2023-02-22 19:54:18,313 process_query INFO Executing policy: dialog_policy
2023-02-22 19:54:18,313 process_query INFO 

Determining entities associated with the query: can you throw up.
2023-02-22 19:54:18,313 __get_entities_for_query INFO Trying entity recognition with module type: intent_slot, model name:  and confidence_threshold: 0.0
2023-02-22 19:54:18,314 get_intent_slot_result INFO Making a NLU request to intent-slot model with name: smalltalk as result was not cached.
2023-02-22 19:54:18,314 get_intent_slot_result INFO No response from intent slot model with name: smalltalk. Either no models are defined in bot config file or defined model is not deployed.
2023-02-22 19:54:18,314 get_intent_slot_result INFO NLU Intent Slot Model Result:
Domain: none Score: 1.0
Intent: none Score: 1.0
2023-02-22 19:54:18,314 __get_entities_for_query INFO Trying entity recognition with module type: ner, model name:  and confidence_threshold: 0.0
2023-02-22 19:54:18,314 __get_prompts_for_slots DEBUG Tagging all qna based entities in domain: smalltalk
2023-02-22 19:54:18,314 get_ner_result INFO Call to NER model for domain: smalltalk failed as no NER models defined in bot config file.
2023-02-22 19:54:18,314 __get_entities_for_query INFO Trying entity recognition with module type: lookup, model name:  and confidence_threshold: 0.0
2023-02-22 19:54:18,314 process_query INFO Lookup NLP module result: TokenClassResponse(slots=[])
2023-02-22 19:54:18,314 __get_entities_for_query INFO Trying entity recognition with module type: regex, model name:  and confidence_threshold: 0.0
2023-02-22 19:54:18,314 process_query INFO Regex NLP module result: TokenClassResponse(slots=[])
2023-02-22 19:54:18,314 __get_entities_for_query INFO Final determined entities from entity recognition task: [{'Token': 'throw', 'EntityName': 'bodyaction', 'Span': {'start': 8, 'end': 12}}, {'Token': 'up', 'EntityName': 'bodyaction', 'Span': {'start': 14, 'end': 15}}]


2023-02-22 19:54:18,314 get_slots DEBUG Populating slot values from Dialog State Tracker..
2023-02-22 19:54:18,314 __get_dialog_state_slots INFO Available short term slots are: dict_keys([])
2023-02-22 19:54:18,314 __get_dialog_state_slots INFO Available long term slots are: dict_keys([])
2023-02-22 19:54:18,314 get_slots DEBUG Using entities: [{'Token': 'throw', 'EntityName': 'bodyaction', 'Span': {'start': 8, 'end': 12}}, {'Token': 'up', 'EntityName': 'bodyaction', 'Span': {'start': 14, 'end': 15}}] to form all types of slots after correcting their values if needed..
2023-02-22 19:54:18,314 get_slots DEBUG Slots formed for smalltalk: {'bodyaction': ['throw', 'up']}
2023-02-22 19:54:18,315 get_slots DEBUG Using entities: [{'Token': 'throw', 'EntityName': 'bodyaction', 'Span': {'start': 8, 'end': 12}}, {'Token': 'up', 'EntityName': 'bodyaction', 'Span': {'start': 14, 'end': 15}}] to form all types of slots after correcting their values if needed..
2023-02-22 19:54:18,315 get_slots DEBUG Slots formed for smalltalk: {'bodyaction': ['throw', 'up']}
2023-02-22 19:54:18,315 __get_dialog DEBUG Found matching dialog in top level.
2023-02-22 19:54:18,315 process_query DEBUG Valid dialog found by dialog policy.
2023-02-22 19:54:18,315 __create_dialog_id DEBUG Generating new dialog ID as no last dialog was found.
2023-02-22 19:54:18,315 __execute_enabled_processors DEBUG Query after spell correction which is used for pronoun replacement and query replacement is: can you throw up
2023-02-22 19:54:18,315 execute_actions INFO Executing action: ResponseRules
2023-02-22 19:54:18,315 get_default_need_user_response DEBUG Domain:  has default need_user_response: None.
2023-02-22 19:54:18,315 get_response DEBUG Possible text responses are: ["I don't have a body."]
2023-02-22 19:54:18,315 get_response DEBUG Selected Valid response: Response(text="I don't have a body.", action=None, json=None, need_user_response=None, omniverse_json=None, is_partial_response=None)
2023-02-22 19:54:18,315 execute_actions INFO Found valid response by executing action: ResponseRules
2023-02-22 19:54:18,315 process_query INFO Valid response found by executing: dialog_policy
2023-02-22 19:54:18,315 postprocess_dm_response DEBUG Cleaned up response text after markdown language tags removal is: I don't have a body.
2023-02-22 19:54:18,315 get_slots DEBUG Using entities: [{'Token': 'throw', 'EntityName': 'bodyaction', 'Span': {'start': 8, 'end': 12}}, {'Token': 'up', 'EntityName': 'bodyaction', 'Span': {'start': 14, 'end': 15}}] to form all types of slots after correcting their values if needed..
2023-02-22 19:54:18,316 get_slots DEBUG Slots formed for smalltalk: {'bodyaction': ['throw', 'up']}
2023-02-22 19:54:18,316 __update_slot_timestamps DEBUG Updated timestamp for bodyaction
2023-02-22 19:54:18,316 save_user_data INFO Adding a new data corresponding to user-id 11005d36-f6f3-404a-be9b-0c22b4a1ee6e to the store.Data of Total 1 users are available in store
2023-02-22 19:54:18,317 __display_user_data INFO {
    "ApiVersion": "2.0.0-ea",
    "Language": "en-US",
    "ResponseLanguage": "en-US",
    "Version": "1.2",
    "UserId": "11005d36-f6f3-404a-be9b-0c22b4a1ee6e",
    "QueryId": "4b9cc471-8809-4f01-bf9b-6a2c769a9906",
    "DialogId": "88374adf-4d80-4197-a31a-7ab4bfd7f1ed",
    "SessionId": "f29d346f-276f-4303-8f6b-1272a20f4a89",
    "TimeStamp": "2023-02-22T19:54:18.316Z",
    "Query": "can you throw up",
    "ProcessedQuery": "can you throw up",
    "Event": "none",
    "StreamId": "",
    "Response": {
        "Text": "I don't have a body.",
        "CleanedText": "I don't have a body.",
        "Json": null,
        "OmniverseJson": null,
        "Action": null,
        "NeedUserResponse": false,
        "IsPartialResponse": false
    },
    "Domain": "smalltalk",
    "Intent": "smalltalk.personality_i_dont_have_a_body",
    "Entities": [
        {
            "Token": "throw",
            "EntityName": "bodyaction",
            "Span": {
                "start": 8,
                "end": 12
            }
        },
        {
            "Token": "up",
            "EntityName": "bodyaction",
            "Span": {
                "start": 14,
                "end": 15
            }
        }
    ],
    "Slots": [
        {
            "Name": "system.policy.name",
            "Values": [
                "nlu_data_match_policy"
            ]
        },
        {
            "Name": "system.policy.intent",
            "Values": [
                "smalltalk.personality_i_dont_have_a_body"
            ]
        },
        {
            "Name": "system.policy.domain",
            "Values": [
                "smalltalk"
            ]
        },
        {
            "Name": "system.policy.entities",
            "Values": [
                {
                    "Text": "",
                    "Attributes": [
                        {
                            "Name": "Token",
                            "Values": [
                                "t",
                                "h",
                                "r",
                                "o",
                                "w"
                            ]
                        },
                        {
                            "Name": "EntityName",
                            "Values": [
                                "b",
                                "o",
                                "d",
                                "y",
                                "a",
                                "c",
                                "t",
                                "i",
                                "o",
                                "n"
                            ]
                        },
                        {
                            "Name": "Span",
                            "Values": [
                                "start",
                                "end"
                            ]
                        }
                    ]
                },
                {
                    "Text": "",
                    "Attributes": [
                        {
                            "Name": "Token",
                            "Values": [
                                "u",
                                "p"
                            ]
                        },
                        {
                            "Name": "EntityName",
                            "Values": [
                                "b",
                                "o",
                                "d",
                                "y",
                                "a",
                                "c",
                                "t",
                                "i",
                                "o",
                                "n"
                            ]
                        },
                        {
                            "Name": "Span",
                            "Values": [
                                "start",
                                "end"
                            ]
                        }
                    ]
                }
            ]
        },
        {
            "Name": "system.policy.anchor",
            "Values": [
                "can you throw up"
            ]
        },
        {
            "Name": "bodyaction",
            "Values": [
                "throw",
                "up"
            ]
        }
    ],
    "MissingSlots": [],
    "Parameters": {},
    "PolicyName": "dialog_policy",
    "Fulfillment": [],
    "NlpResult": {
        "Domain": {
            "Name": "none",
            "ConfidenceScore": 1.0,
            "NluType": ""
        },
        "Intent": {
            "Name": "none",
            "ConfidenceScore": 1.0,
            "NluType": ""
        },
        "Entities": [],
        "NaturalQuery": [],
        "InformationRetrieval": [],
        "SemanticSimilarity": [],
        "TextClassifier": [],
        "Entailment": []
    },
    "DialogState": {
        "SessionSlot": [
            {
                "Name": "bodyaction",
                "Values": [
                    "throw",
                    "up"
                ],
                "RemainingTurn": 2,
                "RemainingTime": 599.9998698234558
            }
        ],
        "GlobalSlot": []
    },
    "Latency": {
        "IntentSlotModel": 0,
        "NaturalQueryModel": 0,
        "InformationRetrievalModel": 0,
        "EntailmentModel": 0,
        "NERModel": 0,
        "MachineTranslation": 0,
        "SemanticSimilarityModel": 0,
        "TextClassifierModel": 0,
        "Fulfillment": 0,
        "DialogManager": 4,
        "EndToEnd": 4,
        "TimeUnit": "ms"
    }
}
[BOT] DEBUG INFO
         
         1. NLU
         +--------+------------------------------------------+
         |  Key   |                  Value                   |
         +========+==========================================+
         | Domain | smalltalk                                |
         +--------+------------------------------------------+
         | Intent | smalltalk.personality_i_dont_have_a_body |
         +--------+------------------------------------------+
         | Action | None                                     |
         +--------+------------------------------------------+
         Entities
         +------------+-------+
         |   Entity   | Value |
         +============+=======+
         | bodyaction | throw |
         +------------+-------+
         | bodyaction | up    |
         +------------+-------+
         +-------------------+-------+
         |        Key        | Value |
         +===================+=======+
         | Fulfillment Slots | None  |
         +-------------------+-------+
         
         2. Dialog State
         Session Slots
         +------------+-----------------+
         |    Slot    |     Values      |
         +============+=================+
         | bodyaction | ['throw', 'up'] |
         +------------+-----------------+
         +--------------+--------+
         |     Key      | Values |
         +==============+========+
         | Global Slots | None   |
         +--------------+--------+
         
         3. Details
         +------------------+--------------------------------------------------------+
         |      Field       |                        Details                         |
         +==================+========================================================+
         | ApiVersion       | "2.0.0-ea"                                             |
         +------------------+--------------------------------------------------------+
         | Language         | "en-US"                                                |
         +------------------+--------------------------------------------------------+
         | ResponseLanguage | "en-US"                                                |
         +------------------+--------------------------------------------------------+
         | Version          | "1.2"                                                  |
         +------------------+--------------------------------------------------------+
         | UserId           | "11005d36-f6f3-404a-be9b-0c22b4a1ee6e"                 |
         +------------------+--------------------------------------------------------+
         | QueryId          | "4b9cc471-8809-4f01-bf9b-6a2c769a9906"                 |
         +------------------+--------------------------------------------------------+
         | DialogId         | "88374adf-4d80-4197-a31a-7ab4bfd7f1ed"                 |
         +------------------+--------------------------------------------------------+
         | SessionId        | "f29d346f-276f-4303-8f6b-1272a20f4a89"                 |
         +------------------+--------------------------------------------------------+
         | TimeStamp        | "2023-02-22T19:54:18.316Z"                             |
         +------------------+--------------------------------------------------------+
         | Query            | "can you throw up"                                     |
         +------------------+--------------------------------------------------------+
         | ProcessedQuery   | "can you throw up"                                     |
         +------------------+--------------------------------------------------------+
         | Event            | "none"                                                 |
         +------------------+--------------------------------------------------------+
         | StreamId         | ""                                                     |
         +------------------+--------------------------------------------------------+
         | Response         | {                                                      |
         |                  |     "Text": "I don't have a body.",                    |
         |                  |     "CleanedText": "I don't have a body.",             |
         |                  |     "Json": null,                                      |
         |                  |     "OmniverseJson": null,                             |
         |                  |     "Action": null,                                    |
         |                  |     "NeedUserResponse": false,                         |
         |                  |     "IsPartialResponse": false                         |
         |                  | }                                                      |
         +------------------+--------------------------------------------------------+
         | Domain           | "smalltalk"                                            |
         +------------------+--------------------------------------------------------+
         | Intent           | "smalltalk.personality_i_dont_have_a_body"             |
         +------------------+--------------------------------------------------------+
         | Entities         | [                                                      |
         |                  |     {                                                  |
         |                  |         "Token": "throw",                              |
         |                  |         "EntityName": "bodyaction",                    |
         |                  |         "Span": {                                      |
         |                  |             "start": 8,                                |
         |                  |             "end": 12                                  |
         |                  |         }                                              |
         |                  |     },                                                 |
         |                  |     {                                                  |
         |                  |         "Token": "up",                                 |
         |                  |         "EntityName": "bodyaction",                    |
         |                  |         "Span": {                                      |
         |                  |             "start": 14,                               |
         |                  |             "end": 15                                  |
         |                  |         }                                              |
         |                  |     }                                                  |
         |                  | ]                                                      |
         +------------------+--------------------------------------------------------+
         | Slots            | [                                                      |
         |                  |     {                                                  |
         |                  |         "Name": "system.policy.name",                  |
         |                  |         "Values": [                                    |
         |                  |             "nlu_data_match_policy"                    |
         |                  |         ]                                              |
         |                  |     },                                                 |
         |                  |     {                                                  |
         |                  |         "Name": "system.policy.intent",                |
         |                  |         "Values": [                                    |
         |                  |             "smalltalk.personality_i_dont_have_a_body" |
         |                  |         ]                                              |
         |                  |     },                                                 |
         |                  |     {                                                  |
         |                  |         "Name": "system.policy.domain",                |
         |                  |         "Values": [                                    |
         |                  |             "smalltalk"                                |
         |                  |         ]                                              |
         |                  |     },                                                 |
         |                  |     {                                                  |
         |                  |         "Name": "system.policy.entities",              |
         |                  |         "Values": [                                    |
         |                  |             {                                          |
         |                  |                 "Text": "",                            |
         |                  |                 "Attributes": [                        |
         |                  |                     {                                  |
         |                  |                         "Name": "Token",               |
         |                  |                         "Values": [                    |
         |                  |                             "t",                       |
         |                  |                             "h",                       |
         |                  |                             "r",                       |
         |                  |                             "o",                       |
         |                  |                             "w"                        |
         |                  |                         ]                              |
         |                  |                     },                                 |
         |                  |                     {                                  |
         |                  |                         "Name": "EntityName",          |
         |                  |                         "Values": [                    |
         |                  |                             "b",                       |
         |                  |                             "o",                       |
         |                  |                             "d",                       |
         |                  |                             "y",                       |
         |                  |                             "a",                       |
         |                  |                             "c",                       |
         |                  |                             "t",                       |
         |                  |                             "i",                       |
         |                  |                             "o",                       |
         |                  |                             "n"                        |
         |                  |                         ]                              |
         |                  |                     },                                 |
         |                  |                     {                                  |
         |                  |                         "Name": "Span",                |
         |                  |                         "Values": [                    |
         |                  |                             "start",                   |
         |                  |                             "end"                      |
         |                  |                         ]                              |
         |                  |                     }                                  |
         |                  |                 ]                                      |
         |                  |             },                                         |
         |                  |             {                                          |
         |                  |                 "Text": "",                            |
         |                  |                 "Attributes": [                        |
         |                  |                     {                                  |
         |                  |                         "Name": "Token",               |
         |                  |                         "Values": [                    |
         |                  |                             "u",                       |
         |                  |                             "p"                        |
         |                  |                         ]                              |
         |                  |                     },                                 |
         |                  |                     {                                  |
         |                  |                         "Name": "EntityName",          |
         |                  |                         "Values": [                    |
         |                  |                             "b",                       |
         |                  |                             "o",                       |
         |                  |                             "d",                       |
         |                  |                             "y",                       |
         |                  |                             "a",                       |
         |                  |                             "c",                       |
         |                  |                             "t",                       |
         |                  |                             "i",                       |
         |                  |                             "o",                       |
         |                  |                             "n"                        |
         |                  |                         ]                              |
         |                  |                     },                                 |
         |                  |                     {                                  |
         |                  |                         "Name": "Span",                |
         |                  |                         "Values": [                    |
         |                  |                             "start",                   |
         |                  |                             "end"                      |
         |                  |                         ]                              |
         |                  |                     }                                  |
         |                  |                 ]                                      |
         |                  |             }                                          |
         |                  |         ]                                              |
         |                  |     },                                                 |
         |                  |     {                                                  |
         |                  |         "Name": "system.policy.anchor",                |
         |                  |         "Values": [                                    |
         |                  |             "can you throw up"                         |
         |                  |         ]                                              |
         |                  |     },                                                 |
         |                  |     {                                                  |
         |                  |         "Name": "bodyaction",                          |
         |                  |         "Values": [                                    |
         |                  |             "throw",                                   |
         |                  |             "up"                                       |
         |                  |         ]                                              |
         |                  |     }                                                  |
         |                  | ]                                                      |
         +------------------+--------------------------------------------------------+
         | MissingSlots     | []                                                     |
         +------------------+--------------------------------------------------------+
         | Parameters       | {}                                                     |
         +------------------+--------------------------------------------------------+
         | PolicyName       | "dialog_policy"                                        |
         +------------------+--------------------------------------------------------+
         | Fulfillment      | []                                                     |
         +------------------+--------------------------------------------------------+
         | NlpResult        | {                                                      |
         |                  |     "Domain": {                                        |
         |                  |         "Name": "none",                                |
         |                  |         "ConfidenceScore": 1.0,                        |
         |                  |         "NluType": ""                                  |
         |                  |     },                                                 |
         |                  |     "Intent": {                                        |
         |                  |         "Name": "none",                                |
         |                  |         "ConfidenceScore": 1.0,                        |
         |                  |         "NluType": ""                                  |
         |                  |     },                                                 |
         |                  |     "Entities": [],                                    |
         |                  |     "NaturalQuery": [],                                |
         |                  |     "InformationRetrieval": [],                        |
         |                  |     "SemanticSimilarity": [],                          |
         |                  |     "TextClassifier": [],                              |
         |                  |     "Entailment": []                                   |
         |                  | }                                                      |
         +------------------+--------------------------------------------------------+
         | DialogState      | {                                                      |
         |                  |     "SessionSlot": [                                   |
         |                  |         {                                              |
         |                  |             "Name": "bodyaction",                      |
         |                  |             "Values": [                                |
         |                  |                 "throw",                               |
         |                  |                 "up"                                   |
         |                  |             ],                                         |
         |                  |             "RemainingTurn": 2,                        |
         |                  |             "RemainingTime": 599.9998698234558         |
         |                  |         }                                              |
         |                  |     ],                                                 |
         |                  |     "GlobalSlot": []                                   |
         |                  | }                                                      |
         +------------------+--------------------------------------------------------+
         | Latency          | {                                                      |
         |                  |     "IntentSlotModel": 0,                              |
         |                  |     "NaturalQueryModel": 0,                            |
         |                  |     "InformationRetrievalModel": 0,                    |
         |                  |     "EntailmentModel": 0,                              |
         |                  |     "NERModel": 0,                                     |
         |                  |     "MachineTranslation": 0,                           |
         |                  |     "SemanticSimilarityModel": 0,                      |
         |                  |     "TextClassifierModel": 0,                          |
         |                  |     "Fulfillment": 0,                                  |
         |                  |     "DialogManager": 4,                                |
         |                  |     "EndToEnd": 4,                                     |
         |                  |     "TimeUnit": "ms"                                   |
         |                  | }                                                      |
         +------------------+--------------------------------------------------------+
         
[BOT] I don't have a body.
[YOU] 

```

Looking at the log it is easy to understand the logic:

- The domain is caught by the anchor in the nlu_data files. 
  - **ToDo** Is it the name of the corresponding directory?
  - Most likely it comes from the dictionary: `smalltalk.personality_thats_not_something_i_can_do` so first you get the intent in this case?
- Once the domain is chosen, then the intent is also coming from the anchor and the dictionary
- The slots are correctly detected

You can also see that the answer is produced by the dialog manager, by looking into the action for the corresponding intent. Here is the relevant part of the `domain_smalltalk.yaml` configuration file :

```yaml
- intent: smalltalk.personality_i_dont_have_a_body
  action:
    - response:
      - text: "I don't have a body."
```



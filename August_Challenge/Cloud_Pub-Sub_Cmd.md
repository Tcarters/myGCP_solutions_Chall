
## Task 1. Pub/Sub topics

```bash
    gcloud pubsub topics create myTopic

    gcloud pubsub topics create Test1

    gcloud pubsub topics create Test2

    ## List Topics
    gcloud pubsub topics list

```

- Delete a TOpic

```bash
    gcloud pubsub topics delete Test1
```

## Task 2. Pub/Sub subscriptions

```bash
    gcloud  pubsub subscriptions create --topic myTopic mySubscription

    ## List subscriptions
    gcloud pubsub topics list-subscriptions myTopic


    ## Delete a subscriptions
    gcloud pubsub subscriptions delete Test1

```

## Task 3. Pub/Sub publishing and pulling a single message

```bash
    ## Publish a message to a topic
    gcloud pubsub topics publish myTopic --message "Hello"

    gcloud pubsub topics publish myTopic --message "Publisher's name is <YOUR NAME>"

    ## Pull messages to topic

    gcloud pubsub subscriptions pull mySubscription --auto-ack

```

## Task 4. Pub/Sub pulling all messages from subscriptions

```bash
    gcloud pubsub subscriptions pull mySubscription --auto-ack --limit=3
```


------------------------------------------------------------------------------------------END_GAME-----------------------------------------------------------------------------------------------


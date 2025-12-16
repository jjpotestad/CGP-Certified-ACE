# Pub/Sub: Qwik Start - Command Line

## Setup and requirements

#### (Optional) You can list the active account name with this command:
```bash
gcloud auth list
```

#### (Optional) You can list the project ID with this command:
```bash
gcloud config list project
```

## Task 1. Pub/Sub topics

#### Run the following command to create a topic called myTopic:
```bash
gcloud pubsub topics create myTopic
```

#### For good measure, create two more topics; one called Test1 and the other called Test2:
```bash
gcloud pubsub topics create Test1
```

```bash
gcloud pubsub topics create Test2
```

#### To see the three topics you just created, run the following command:
```bash
gcloud pubsub topics list
```

#### Time to clean up. Delete Test1 and Test2 by running the following commands:
```bash
gcloud pubsub topics delete Test1
```

```bash
gcloud pubsub topics delete Test2
```

#### Run the <b>gcloud pubsub topics list</b> command one more time to verify the topics were deleted:
```bash
gcloud pubsub topics list
```

## Task 2. Pub/Sub subscriptions

#### Run the following command to create a subscription called mySubscription to topic myTopic:
```bash
gcloud  pubsub subscriptions create --topic myTopic mySubscription
```

#### Add another two subscriptions to myTopic. Run the following commands to make Test1 and Test2 subscriptions:
```bash
gcloud  pubsub subscriptions create --topic myTopic Test1
```

```bash
gcloud  pubsub subscriptions create --topic myTopic Test2
```

#### Run the following command to list the subscriptions to myTopic:
```bash
gcloud pubsub topics list-subscriptions myTopic
```

#### Now delete the Test1 and Test2 subscriptions. Run the following commands:
```bash
gcloud pubsub subscriptions delete Test1
```

```bash
gcloud pubsub subscriptions delete Test2
```

#### See if the Test1 and Test2 subscriptions were deleted. Run the list-subscriptions command one more time:
```bash
gcloud pubsub topics list-subscriptions myTopic
```

## Task 3. Pub/Sub publishing and pulling a single message

#### Run the following command to publish the message "hello" to the topic you created previously (myTopic):
```bash
gcloud pubsub topics publish myTopic --message "Hello"
```

#### Publish a few more messages to myTopic. Run the following commands (replacing <YOUR NAME> with your name and <FOOD> with a food you like to eat):

```bash
gcloud pubsub topics publish myTopic --message "Publisher's name is <YOUR NAME>"
```

```bash
gcloud pubsub topics publish myTopic --message "Publisher likes to eat <FOOD>"
```

```bash
gcloud pubsub topics publish myTopic --message "Publisher thinks Pub/Sub is awesome"
```

#### Use the following command to pull the messages you just published from the Pub/Sub topic:
```bash
gcloud pubsub subscriptions pull mySubscription --auto-ack
```


## Task 4. Pub/Sub pulling all messages from subscriptions

#### Run the following commands:
```bash
gcloud pubsub topics publish myTopic --message "Publisher is starting to get the hang of Pub/Sub"
```

```bash
gcloud pubsub topics publish myTopic --message "Publisher wonders if all messages will be pulled"
```

```bash
gcloud pubsub topics publish myTopic --message "Publisher will have to test to find out"
```

#### Add a <b>flag</b> to your command so you can output all three messages in one request. You may have not noticed, but you have actually been using a <b>flag</b> this entire time: the <b>--auto-ack</b> part of the <b>pull</b> command is a flag that has been formatting your messages into the neat boxes that you see your pulled messages in.
#### <b>limit</b> is another flag that sets an upper limit on the number of messages to pull. Wait a minute to let the topics get created. Run the pull command with the <b>limit</b> flag:
```bash
gcloud pubsub subscriptions pull mySubscription --limit=3
```

## Congratulations!



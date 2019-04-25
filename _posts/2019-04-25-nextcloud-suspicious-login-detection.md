---
layout: single
title: Detecting Suspicious Nextcloud Logins
comments: true
date: 2019-04-25
last_modified_at: 2019-04-25
tags:
  - nextcloud
  - security
  - machine learning
  - foss
header:
  image: /assets/20190424_nextcloud_suspiciou_login_detection/banner.png
---

With Nextcloud 16 we release a new security app: *[Suspicious Login](https://apps.nextcloud.com/apps/suspicious_login)*. This app can detect anomalies in IP addresses logging into a user's Nextcloud account using supervised learning of an artificial neural network.

## An Overview

Whenever a user logs into Nextcloud, a trained model of a neural network is loaded and used to classify the combination of IP and user ID (UID) as either suspicious or not suspicious. In the former case the user is informed immediately with a (push) notification to their desktop and mobile clients.

![Nextcloud Android app showing a suspicious login into the user's account](/assets/20190424_nextcloud_suspiciou_login_detection/notification_android.png)

An affected user can then change their password or kill any device token that looks unfamiliar to them.

## Your cloud, your data

In order to train and use a neural network to classify logins, we need some good training data. While we could ship a pre-trained model that has already learned the structure of IP addresses and a shorter time to practical use, we know that Nextcloud admins and users value privacy. Therefore the app only uses data from the instance.

The *Suspicious Login* app will collect (IP, UID) tuples from every login and every client connection. After a few weeks of collection, this data can be used for training. The training data is updated continuously as is the trained model.

As training data is only gather from the instance, the classifier should be less of a black box, but deliver highly customized results while valuing user data privacy.


## Gathering *good* training data

To get the best classifier possible, it is important to have a good set of training data for the neural network. Successful logins form the positive data set. But we also need negative samples. As we don't have such data in an ideal scenario we have to fake it.

### Random samples

The simplest way to get random data samples is to use a random number generator to generate IP addresses. Combined with existing users, they simulate logins from IP addresses that are very different to a user's previous login sources.

### Shuffled samples

The neural net should not only warn about IPs it hasn't seen before in general, but also when a user tries to log into another user's account. This is less relevant for environments where many users of the same instance use Nextcloud from the same network, but makes most sense for shared instances, e.g. on hosted providers, or remote companies.

The app therefore shuffles randomly picked data from the positive training set and mixes UIDs with IPs that a specific user has not logged in from before.

## Separation of training and validation data sets

The performance of a classifier can be determined by testing it with data sets it has not seen before. In many machine learning tasks this is done with randomly separated data rows into training and validation sets. As the login history of a user is not random but follows a certain pattern, only the most recent data is used for validation, while the historic data is used for training. 

## Building the input layer

Another important consideration for the design of the machine learning application is the selection of a good input representation. The raw data is an UID string and an IPv4 string. These have to be converted to numeric values, while both keeping balance between the input values and making it possible for the neural net to learn the structure and hierarchy of IP addresses.

This is the part where most effort went into but also where the biggest improvements in terms of classifier performance were made. I spare you the details of my failed attempts for a good input representation, but you can find it in the git history if you're curious 😉.

So far the best representation I have found is to hash the UID to a 16 bit binary vector. This lowers the value range while still having a relatively low conflict probability. The IP address is also represented as its 32bit binary vector. Combined, this makes for a 48bit input layer.

## Sensible training parameters

Now that we have the training data and the correct representation we can have a close look at the parameters of a training. The multi layer perceptron of [php-ml](https://php-ml.org/) has [these free parameters](https://php-ml.readthedocs.io/en/latest/machine-learning/neural-network/multilayer-perceptron-classifier/#constructor-parameters):
* layers
* iterations (epochs)
* learn rate
* activation function

I've ran numerous training runs with different sets of parameter values. The goal was to find something that has a high, but balanced precision and recall of logins classified as suspicious, while not consuming too much time to compute. The default set of parameters should also work relatively well for small, medium and large installations. After a few hundred training runs with many different combinations of parameter values I have found that 150 epochs, 14 layers and a learn rate of 7% to meet those requirements. I have also implemented a heuristic *simulated annealing* algorithm that starts with random parameters and tests those iteratively with decreasingly modified random values within certain boundaries. These runs take a very long time as each of the parameter sets is used multiple times to compensate for the randomness of training a neural net. In the end the best solution found was usually also in the area of my handcrafted ones.


In addition to the training parameters, the app is also configurable in the ratio of how much random and shuffled data to generate for the positive data available. As a sensible default the app takes 150% as many shuffled negative and 100% as many random samples as positive samples. With this configuration, precision and recall were the highest and the most balanced.


## Keeping the model up-to-date

Training the model once does not do the job as users travel or simply get new IPs by their ISPs. But the admin actually does not even have to trigger any training because training is automated in a Nextcloud background job that runs once a day. This ensures that the model used is ready to distinguish login patterns of recently used IPs from the ones of new IP patterns. Depending on the instance size the training may take a few minutes to complete, but should not consume many resources on any reasonably powerful server.

## Results

Precision and recall were already mentioned above. These binary metrics can tell how well the trained model might react to new login data. In this app we only care about the negative data sets, as in, how many of the suspicious logins are detected and how many of the detected are indeed from suspicious sources.

The **precision(n)** is the ratio of truly suspicious logins among the set of (UID, IP) tuples classified as suspicious. Therefore it tells us how likely a suspicious login alert should alarm us (as opposed to false alarms).

The **recall(n)** is the ratio of detected suspicious logins among the set of all suspicious logins. It can therefore indicate how many of the logins from unknown/uncommon sources will be captured as such.

![Historic precision and recall of suspicious login detection on a small Nextcloud instance](/assets/20190424_nextcloud_suspiciou_login_detection/admin_settings.png)

We have tested the app continuously on three instances and monitored the training results. The more stationary (no international travel, similar IPs assigned by ISPs and cell phone carriers) users of a Nextcloud are, the better the results. But also for larger instances with traveling users the results were always above 80% accuracy (both precision and recall).

## Conclusion

This is the first time I'm putting my knowledge about neural networks into a practical use case. So I'm very happy this turned out to work quite well so far and I hope this app will be used to protect the data of many Nextcloud instances.

As always, the code is 100% open source, licensed under the GNU APGL. You can find the source code on [GitHub](https://github.com/nextcloud/suspicious_login), where I hope to see your feedback, bug reports and questions.

If you are new to Nextcloud, check out our [website](https://nextcloud.com/)!

## Acknowledgement

Many ideas for this app came from [Amazon](https://aws.amazon.com/blogs/machine-learning/detect-suspicious-ip-addresses-with-the-amazon-sagemaker-ip-insights-algorithm/) and [this paper, which also gives a great overview on the topic](https://arxiv.org/pdf/1701.02145.pdf).

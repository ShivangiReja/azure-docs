---
title: Azure Service Bus duplicate message detection | Microsoft Docs
description: This article explains how you can detect duplicates in Azure Service Bus messages. The duplicate message can be ignored and dropped.
ms.topic: article
ms.date: 04/14/2021
---

# Duplicate detection

If an application fails due to a fatal error immediately after it sends a message, and the restarted application instance erroneously believes that the prior message delivery did not occur, a subsequent send causes the same message to appear in the system twice.

It's also possible for an error at the client or network level to occur a moment earlier, and for a sent message to be committed into the queue, with the acknowledgment not successfully returned to the client. This scenario leaves the client in doubt about the outcome of the send operation.

Duplicate detection takes the doubt out of these situations by enabling the sender resend the same message, and the queue or topic discards any duplicate copies.

> [!NOTE]
> The basic tier of Service Bus doesn't support duplicate detection. The standard and premium tiers support duplicate detection. For differences between these tiers, see [Service Bus pricing](https://azure.microsoft.com/pricing/details/service-bus/).

## How it works? 
Enabling duplicate detection helps keep track of the application-controlled *MessageId* of all messages sent into a queue or topic during a specified time window. If any new message is sent with *MessageId* that was logged during the time window, the message is reported as accepted (the send operation succeeds), but the newly sent message is instantly ignored and dropped. No other parts of the message other than the *MessageId* are considered.

Application control of the identifier is essential, because only that allows the application to tie the *MessageId* to a business process context from which it can be predictably reconstructed when a failure occurs.

For a business process in which multiple messages are sent in the course of handling some application context, the *MessageId* may be a composite of the application-level context identifier, such as a purchase order number, and the subject of the message, for example, **12345.2017/payment**.

The *MessageId* can always be some GUID, but anchoring the identifier to the business process yields predictable repeatability, which is desired for using the duplicate detection feature effectively.

> [!IMPORTANT]
>- When **partitioning** is **enabled**, `MessageId+PartitionKey` is used to determine uniqueness. When sessions are enabled, partition key and session ID must be the same. 
>- When **partitioning** is **disabled** (default), only `MessageId` is used to determine uniqueness.
>- For information about SessionId, PartitionKey, and MessageId, see [Use of partition keys](service-bus-partitioning.md#use-of-partition-keys).
>- The [premier tier](service-bus-premium-messaging.md) doesn't support partitioning, so we recommend that you use unique message IDs in your applications and not rely on partition keys for duplicate detection. 


## Enable duplicate detection

Apart from just enabling duplicate detection, you can also configure the size of the duplicate detection history time window during which message-ids are retained.
This value defaults to 10 minutes for queues and topics, with a minimum value of 20 seconds to maximum value of 7 days.

Enabling duplicate detection and the size of the window directly impact the queue (and topic) throughput, since all recorded message-ids must be matched against the newly submitted message identifier.

Keeping the window small means that fewer message-ids must be retained and matched, and throughput is impacted less. For high throughput entities that require duplicate detection, you should keep the window as small as possible.

### Using the portal

In the portal, the duplicate detection feature is turned on during entity creation with the **Enable duplicate detection** check box, which is off by default. The setting for creating new topics is equivalent.

![Screenshot of the Create queue dialog box with the Enable duplicate detection option selected and outlined in red.][1]

> [!IMPORTANT]
> You can't enable/disable duplicate detection after the queue is created. You can only do so at the time of creating the queue. 

The duplicate detection history time window can be changed in the queue and topic properties window in the Azure portal.

![Screenshot of the Service Bus feature with the Properties setting highlighted adn the Duplicate detection history option outlined in red.][2]

### Using SDKs

You can any of our SDKs across .NET, Java, JavaScript, Python and Go to enable duplicate detection feature when creating queues and topics. You can also change the duplicate detection history time window.
The properties to update when creating queues and topics to achieve this are:
- `RequiresDuplicateDetection`
- `DuplicateDetectionHistoryTimeWindow`

Please note that while the property names are provided in pascal casing here, JavaScript and Python SDKs will be using camel casing and snake casing respectively.

## Next steps

To learn more about Service Bus messaging, see the following topics:

* [Service Bus queues, topics, and subscriptions](service-bus-queues-topics-subscriptions.md)
* [Get started with Service Bus queues](service-bus-dotnet-get-started-with-queues.md)
* [How to use Service Bus topics and subscriptions](service-bus-dotnet-how-to-use-topics-subscriptions.md)

In scenarios where client code is unable to resubmit a message with the same *MessageId* as before, it is important to design messages that can be safely reprocessed. This [blog post about idempotence](https://particular.net/blog/what-does-idempotent-mean) describes various techniques for how to do that.

[1]: ./media/duplicate-detection/create-queue.png
[2]: ./media/duplicate-detection/queue-prop.png

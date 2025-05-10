ahmed mostafa 10005779

# cloudass
assignment1
///////////////////PIC////////////////////////////
![maqot4](https://github.com/user-attachments/assets/d94b34a8-5a69-4f62-a4ca-1839e465413a)
![maqot3](https://github.com/user-attachments/assets/1e618bf4-dfe5-4562-aa7a-1bcf0e032b82)
![maqot2](https://github.com/user-attachments/assets/3ed10423-ea01-43dd-9b6f-42dadc4555a8)
![maqot1](https://github.com/user-attachments/assets/5182a51b-6aaa-4201-9d3e-28dea3800813)
///////////////////////////////////SUMMAR//////////////////////////
Explanation: Visibility Timeout & DLQ Usage

### Visibility Timeout

The visibility timeout in Amazon SQS ensures that when a message is retrieved by a consumer (like a Lambda function), it becomes temporarily invisible to other consumers. This prevents the same message from being processed multiple times concurrently.

In this project:
- When the Lambda function starts processing a message from `OrderQueue`, the message becomes invisible.
- If processing succeeds, the message is deleted.
- If the Lambda crashes or throws an error, the message becomes visible again after the timeout expires, allowing for retry.

This guarantees **at-least-once delivery** and protects against race conditions or duplicate writes to DynamoDB.

### ✅ Dead-Letter Queue (DLQ)

The Dead-Letter Queue (DLQ) is a secondary queue where failed messages are sent after a specified number of failed processing attempts (e.g., 3 tries).

In this system:
- `OrderDLQ` is linked to `OrderQueue` with `maxReceiveCount = 3`.
- If a message causes an error (e.g., bad JSON, missing fields) and fails 3 times, it is automatically moved to `OrderDLQ`.
- This avoids repeated processing of broken data and keeps the main queue clean.

You can inspect `OrderDLQ` later to fix or reprocess failed messages.

---

###  Summary

By combining **Visibility Timeout** and **DLQ**, the system becomes more resilient:
-  Avoids infinite loops on bad messages
-  Allows safe retry logic without duplicates
-  Helps with debugging and future improvements

This design is essential for reliable, production-grade serverless applications.

  /////////////////////////LAMBDA CODE//////////////////////
  const { DynamoDBClient } = require("@aws-sdk/client-dynamodb");
const { DynamoDBDocumentClient, PutCommand } = require("@aws-sdk/lib-dynamodb");

const client = new DynamoDBClient({});
const docClient = DynamoDBDocumentClient.from(client);

exports.handler = async (event) => {
    for (const record of event.Records) {
        try {
            const snsMessage = JSON.parse(record.body);
            const order = JSON.parse(snsMessage.Message);

            const params = {
                TableName: "Orders",
                Item: {
                    orderId: order.orderId,
                    userId: order.userId,
                    itemName: order.itemName,
                    quantity: order.quantity,
                    status: order.status,
                    timestamp: order.timestamp
                }
            };

            await docClient.send(new PutCommand(params));
            console.log("✅ Order saved:", order.orderId);
        } catch (err) {
            console.error("❌ Error processing record:", err);
        }
    }
};





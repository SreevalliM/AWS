# 8. ⚡ DynamoDB

**AWS's fully managed NoSQL database**

![DynamoDB Overview](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2019/03/05/everything-dynamodb-5.gif)

![DynamoDB Partition Key](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2018/09/10/dynamodb-partition-key-2.gif)

![DynamoDB Secondary Indexes](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2018/12/19/DynamoDBSecondaryIndexes1.png)

![DynamoDB GSI](https://docs.aws.amazon.com/images/amazondynamodb/latest/developerguide/images/GSI_01.png)

## What is DynamoDB?

Fast, flexible NoSQL database service for applications that need consistent, single-digit millisecond latency at any scale. Fully managed with built-in security, backup/restore, and in-memory caching.

### Key Features
- ✅ **Fully managed** - No servers to manage
- ✅ **Performance at scale** - Consistent low latency
- ✅ **Flexible schema** - NoSQL design
- ✅ **Auto-scaling** - Scales up and down automatically
- ✅ **High availability** - Multi-AZ, durable
- ✅ **Security** - Encryption at rest, IAM integration

### Use Cases
- Mobile and web applications
- Gaming leaderboards and session stores
- IoT data storage
- Real-time bidding
- Shopping carts
- Serverless applications with Lambda

## DynamoDB Basics

### Tables
- Collection of items (like rows in relational DB)
- No schema required (except primary key)
- Items can have different attributes

### Items
- Individual records in a table
- Collected of attributes (like columns)
- Size limit: 400 KB per item

### Attributes 
- Data elements (like fields)
- Types: String, Number, Binary, Boolean, Null, List, Map, Set

### Example Structure
```json
Table: Users
Item 1: {
  "UserId": "123",
  "Name": "John Doe",
  "Email": "john@example.com",
  "Age": 30,
  "Premium": true,
  "Tags": ["developer", "aws"]
}

Item 2: {
  "UserId": "456",
  "Name": "Jane Smith",
  "Country": "USA",
  "LoginCount": 42
}
```

## Primary Keys

**MUST define when creating table - cannot change later!**

### 1. Partition Key (Simple Primary Key)
- Single attribute
- Must be unique
- Used to distribute data across partitions

```json
Table: Users
Primary Key schema Key: UserId (Partition Key)

Items:
- {"UserId": "123", "Name": "John"}
- {"UserId": "456", "Name": "Jane"}
```

### 2. Partition Key + Sort Key (Composite Primary Key)
- Two attributes
- Partition key groups items
- Sort key orders items within partition
- **Combination must be unique**

```json
Table: GameScores
Primary Key: UserId (Partition) + GameId (Sort Key)

Items:
- {"UserId": "123", "GameId": "chess", "Score": 1500}
- {"UserId": "123", "GameId": "poker", "Score": 2000}
- {"UserId": "456", "GameId": "chess", "Score": 1200}
```

### Choosing Primary Key
- **High cardinality** - Many distinct values
- **Even distribution** - Avoid hot partitions
- **Query patterns** - Design for your access patterns

## Secondary Indexes

Query on non-primary-key attributes.

### Global Secondary Index (GSI)
- **Different partition key** and optional sort key
- **Spans all partitions**
- **Eventually consistent** reads
- **Separate throughput** from base table
- Can add/delete anytime

**Example:**
```
Base Table: Users (UserId, Email, Country, Premium)
Primary Key: UserId

GSI: CountryIndex
- Partition Key: Country
- Sort Key: Premium

Query: All premium users in USA
```

### Local Secondary Index (LSI)
- **Same partition key** as table
- **Different sort key**
- **Shares throughput** with table
- **Must create with table** (cannot add later)
- Max 5 LSI per table

**Example:**
```
Table: GameScores
Primary Key: UserId (Partition) + GameId (Sort)

LSI: ScoreIndex
- Partition Key: UserId (same)
- Sort Key: Score

Query: User's games sorted by score
```

### GSI vs LSI

| Feature | GSI | LSI |
|---------|-----|-----|
| **Partition Key** | Different | Same as table |
| **Sort Key** | Optional, can differ | Required, must differ |
| **When to create** | Anytime | Only at table creation |
| **Throughput** | Independent | Shared with table |
| **Max per table** | 20 | 5 |
| **Consistency** | Eventually consistent | Strongly consistent option |

## Read/Write Capacity Modes

### 1. Provisioned Mode (Default)
**You specify reads/writes per second**

- **Read Capacity Unit (RCU)**: 
  - 1 RCU = 1 strongly consistent read/sec (up to 4 KB)
  - 1 RCU = 2 eventually consistent reads/sec (up to 4 KB)
  
- **Write Capacity Unit (WCU)**:
  - 1 WCU = 1 write/sec (up to 1 KB)

**Example Calculations:**
```
Read 10 KB item with strong consistency:
= 10 KB / 4 KB = 2.5 → Round up to 3 RCU

Write 3.5 KB item:
= 3.5 KB / 1 KB = 3.5 → Round up to 4 WCU

Read 10 items of 4 KB each (eventually consistent):
= (10 items * 4 KB) / 4 KB / 2 = 5 RCU
```

**Auto Scaling:**
- Automatically adjust capacity based on traffic
- Set target utilization (e.g., 70%)
- Min and max capacity thresholds

**Use Case:** Predictable workloads

### 2. On-Demand Mode
**Pay per request - no capacity planning**

- Automatically scales
- No concept of RCU/WCU
- Pay per million requests
- 2.5x more expensive than provisioned
- Good for:
  - Unpredictable workloads
  - New apps (unknown traffic)
  - Spiky traffic

**Cost comparison:**
- Provisioned: $0.00013 per RCU/hour
- On-Demand: $1.25 per million read requests

## DynamoDB Operations

### PutItem
Create or replace item.

```javascript
// AWS SDK v3
const params = {
  TableName: 'Users',
  Item: {
    userId: '123',
    name: 'John Doe',
    email: 'john@example.com'
  }
};
await dynamodb.putItem(params);
```

### GetItem
Retrieve single item by primary key.

```javascript
const params = {
  TableName: 'Users',
  Key: {
    userId: '123'
  }
};
const result = await dynamodb.getItem(params);
```

### UpdateItem
Modify attributes of existing item.

```javascript
const params = {
  TableName: 'Users',
  Key: { userId: '123' },
  UpdateExpression: 'SET email = :e, age = age + :inc',
  ExpressionAttributeValues: {
    ':e': 'newemail@example.com',
    ':inc': 1
  }
};
await dynamodb.updateItem(params);
```

### DeleteItem
Remove item from table.

```javascript
const params = {
  TableName: 'Users',
  Key: { userId: '123' }
};
await dynamodb.deleteItem(params);
```

### Query
Find items with specific partition key (efficient).

```javascript
const params = {
  TableName: 'GameScores',
  KeyConditionExpression: 'userId = :uid',
  ExpressionAttributeValues: {
    ':uid': '123'
  }
};
const result = await dynamodb.query(params);
```

### Scan
Read all items (expensive, avoid if possible).

```javascript
const params = {
  TableName: 'Users',
  FilterExpression: 'age > :age',
  ExpressionAttributeValues: {
    ':age': 25
  }
};
const result = await dynamodb.scan(params);
```

**⚠️ Scan is slow and expensive - avoid when possible!**

## Conditional Writes

Only write if condition is met.

```javascript
const params = {
  TableName: 'Users',
  Item: { userId: '123', name: 'John' },
  ConditionExpression: 'attribute_not_exists(userId)'  // Only if doesn't exist
};

```

**Common conditions:**
- `attribute_not_exists(attr)` - Attribute doesn't exist
- `attribute_exists(attr)` - Attribute exists
- `attr = :val` - Attribute equals value
- `attr < :val` - Less than value

## DynamoDB Streams

Capture time-ordered item-level modifications.

### Features
- **24-hour retention**
- **Real-time** stream of changes
- **Event types**: INSERT, MODIFY, REMOVE
- **Integration**: Lambda, Kinesis Data Streams

### Use Cases
- Real-time analytics
- Data replication
- Notifications on changes
- Audit logging

### EnableStream:
```javascript
StreamSpecification: {
  StreamEnabled: true,
  StreamViewType: 'NEW_AND_OLD_IMAGES'  // KEYS_ONLY, NEW_IMAGE, OLD_IMAGE, NEW_AND_OLD_IMAGES
}
```

### Process with Lambda:
```javascript
exports.handler = async (event) => {
  for (const record of event.Records) {
    console.log('Event:', record.eventName); // INSERT, MODIFY, REMOVE
    console.log('New:', record.dynamodb.NewImage);
    console.log('Old:', record.dynamodb.OldImage);
  }
};
```

## DynamoDB Accelerator (DAX)

**In-memory cache** for DynamoDB (microsecond latency).

### Benefits
- ✅ **10x performance** improvement
- ✅ **Microsecond latency** for reads
- ✅ **No code changes** (compatible API)
- ✅ **Reduce read load** on DynamoDB

### Use Cases
- Read-heavy workloads
- Real-time bidding
- Gaming leaderboards
- Social media feeds

### DAX vs ElastiCache

| Feature | DAX | ElastiCache |
|---------|-----|-------------|
| **Purpose** | DynamoDB cache only | General caching |
| **API** | DynamoDB API | Redis/Memcached |
| **Code changes** | Minimal | More changes |
| **Use case** | DynamoDB-specific | Any data source |

## Global Tables

**Multi-region, multi-master** replication.

### Features
- ✅ **Active-active** replication (read/write in any region)
- ✅ **< 1 second** replication
- ✅ **Automatic conflict resolution** (last writer wins)
- ✅ **High availability** across regions

### Use Cases
- Global applications
- Disaster recovery
- Low-latency access worldwide

### Requirements
- DynamoDB Streams must be enabled
- Table must have same name in all regions
- Replica lag monitoring

## Backup and Restore

### On-Demand Backup
- **Full backup** anytime
- **No performance impact**
- **Retention**: Until deleted
- **Restore** to new table (same or cross-region)

### Point-in-Time Recovery (PITR)
- **Continuous backups** for 35 days
- **Restore to any second** in retention period
- **No performance impact**
- Must **enable explicitly**

### Example Use Case:
```
Table created: Jan 1
PITR enabled: Jan 1
Oops, deleted data: Feb 10, 2:35 PM

Solution: Restore to Feb 10, 2:34 PM
```

## DynamoDB Best Practices

### Design Best Practices
1. ✅ **Understand access patterns first** before designing schema
2. ✅ **Use partition keys with high cardinality**
3. ✅ **Denormalize data** (it's NoSQL!)
4. ✅ **Use composite keys** for hierarchical data
5. ✅ **Design for even partition distribution**

### Query Best Practices
1. ✅ **Use Query instead of Scan** whenever possible
2. ✅ **Project only needed attributes**
3. ✅ **Use FilterExpression for additional filtering**
4. ✅ **Paginate large result sets**
5. ✅ **Use batch operations** for multiple items

### Cost Optimization
1. ✅ **Use on-demand for unpredictable workloads**
2. ✅ **Enable auto-scaling for provisioned mode**
3. ✅ **Delete unnecessary GSIs**
4. ✅ **Use TTL** to auto-delete old items
5. ✅ **Consider DAX** to reduce read costs

## ✅ Hands-on Lab: DynamoDB

### Lab 1: Create Table and Add Items

**Step 1: Create Table**
1. **DynamoDB Console** → **Create table**
2. Table name: `Users`
3. Partition key: `userId` (String)
4. Table settings: **Default settings**
5. **Create table**

**Step 2: Add Items via Console**
1. Select table → **Explore table items**
2. **Create item**
3. Add attributes:
```json
{
  "userId": "user001",
  "name": "John Doe",
  "email": "john@example.com",
  "age": 30,
  "country": "USA"
}
```
4. **Create item**
5. Add more items with different attributes

**Step 3: Query Items**
1. **Scan/Query items**
2. Select **Scan**
3. Add filter: `age > 25`
4. **Run**

### Lab 2: Create Table with Composite Key

**Create Table:**
```
Table name: GameScores
Partition key: userId (String)
Sort key: gameId (String)
```

**Add Items:**
```json
{"userId": "player1", "gameId": "chess", "score": 1500, "timestamp": "2026-02-14"}
{"userId": "player1", "gameId": "poker", "score": 2000, "timestamp": "2026-02-14"}
{"userId": "player2", "gameId": "chess", "score": 1200, "timestamp": "2026-02-14"}
```

**Query:**
- Partition key condition: `userId = player1`
- Result: All games for player1

### Lab 3: Create Global Secondary Index

1. **GameScores** table → **Indexes** → **Create index**
2. Index name: `GameIndex`
3. Partition key: `gameId` (String)
4. Sort key: `score` (Number)
5. **Create index**

**Query GSI:**
- Index: `GameIndex`
- Partition key: `gameId = chess`
- Result: All chess scores, sorted by score

### Lab 4: DynamoDB with AWS CLI

```bash
# Create table
aws dynamodb create-table \
  --table-name Products \
  --attribute-definitions AttributeName=productId,AttributeType=S \
  --key-schema AttributeName=productId,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST

# Put item
aws dynamodb put-item \
  --table-name Products \
  --item '{"productId": {"S": "P001"}, "name": {"S": "Laptop"}, "price": {"N": "999"}}'

# Get item
aws dynamodb get-item \
  --table-name Products \
  --key '{"productId": {"S": "P001"}}'

# Query
aws dynamodb query \
  --table-name Products \
  --key-condition-expression "productId = :id" \
  --expression-attribute-values '{":id": {"S": "P001"}}'

# Scan
aws dynamodb scan \
  --table-name Products \
  --filter-expression "price < :p" \
  --expression-attribute-values '{":p": {"N": "1000"}}'

# Delete item
aws dynamodb delete-item \
  --table-name Products \
  --key '{"productId": {"S": "P001"}}'
```

### Lab 5: DynamoDB with Lambda

**Python Lambda function:**
```python
import boto3
import json
from decimal import Decimal

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('Users')

def lambda_handler(event, context):
    # Create
    if event['operation'] == 'create':
        table.put_item(Item=event['payload'])
        return {'statusCode': 200, 'body': 'Item created'}
    
    # Read
    elif event['operation'] == 'read':
        response = table.get_item(Key={'userId': event['userId']})
        return {'statusCode': 200, 'body': json.dumps(response.get('Item'))}
    
    # Update
    elif event['operation'] == 'update':
        table.update_item(
            Key={'userId': event['userId']},
            UpdateExpression='SET #n = :name',
            ExpressionAttributeNames={'#n': 'name'},
            ExpressionAttributeValues={':name': event['name']}
        )
        return {'statusCode': 200, 'body': 'Item updated'}
    
    # Delete
    elif event['operation'] == 'delete':
        table.delete_item(Key={'userId': event['userId']})
        return {'statusCode': 200, 'body': 'Item deleted'}
```

## DynamoDB vs RDS

| Feature | DynamoDB | RDS |
|---------|----------|-----|
| **Type** | NoSQL | Relational (SQL) |
| **Schema** | Flexible | Fixed |
| **Scaling** | Horizontal | Vertical (read replicas for read) |
| **Queries** | Key-value, simple | Complex SQL, joins |
| **Performance** | Single-digit ms | varies |
| **Transactions** | Limited (ACID for single item) | Full ACID |
| **Use Case** | Web scale, simple access | Complex queries, relations |

## Common Interview Questions

1. **Q: What is DynamoDB?**
   - Fully managed NoSQL database with single-digit millisecond latency

2. **Q: Partition key vs Sort key?**
   - Partition key: Distributes data, Sort key: Orders data within partition

3. **Q: GSI vs LSI?**
   - GSI: Different partition key, create anytime
   - LSI: Same partition key, create with table only

4. **Q: How to improve DynamoDB performance?**
   - Use DAX caching, design good partition keys, use query instead of scan

5. **Q: DynamoDB consistency models?**
   - Eventually consistent (default), Strongly consistent (optional)

6. **Q: How does DynamoDB scale?**
   - Automatically partitions data, distributes across multiple servers

7. **Q: Maximum item size?**
   - 400 KB per item

# CRUD Products DynamoDB (Lambda + API Gateway)

This document contains a complete CRUD implementation for DynamoDB using AWS Lambda and API Gateway.
The code follows best practices while keeping a simple and consistent structure.

---

## CREATE PRODUCT
Endpoint: POST /products

```js
/**
 * CREATE ITEM FUNCTION
 * Purpose: Create product safely
 */

const { DynamoDBClient } = require('@aws-sdk/client-dynamodb');
const { DynamoDBDocumentClient, PutCommand } = require('@aws-sdk/lib-dynamodb');
const { randomUUID } = require('crypto');

const client = new DynamoDBClient({});
const dynamodb = DynamoDBDocumentClient.from(client);

const TABLE_NAME = process.env.PRODUCTS_TABLE;

exports.handler = async (event) => {
    console.log('Event:', JSON.stringify(event));

    try {
        const body = JSON.parse(event.body || '{}');

        if (!body.name || body.price === undefined) {
            return {
                statusCode: 400,
                headers: { 'Content-Type': 'application/json', 'Access-Control-Allow-Origin': '*' },
                body: JSON.stringify({ error: 'Invalid input' })
            };
        }

        const now = new Date().toISOString();

        const item = {
            productId: randomUUID(),
            name: body.name,
            description: body.description ?? '',
            price: Number(body.price),
            category: body.category ?? '',
            imageUrl: body.imageUrl ?? '',
            createdAt: now,
            updatedAt: now
        };

        await dynamodb.send(new PutCommand({
            TableName: TABLE_NAME,
            Item: item,
            ConditionExpression: 'attribute_not_exists(productId)'
        }));

        return {
            statusCode: 201,
            headers: { 'Content-Type': 'application/json', 'Access-Control-Allow-Origin': '*' },
            body: JSON.stringify(item)
        };
    } catch (error) {
        console.error(error);
        return {
            statusCode: 500,
            headers: { 'Content-Type': 'application/json', 'Access-Control-Allow-Origin': '*' },
            body: JSON.stringify({ error: 'Create failed', message: error.message })
        };
    }
};

```

---

## GET PRODUCT BY ID
Endpoint: GET /products/{id}

```js
/**
 * GET ITEM FUNCTION
 */

const { DynamoDBClient } = require('@aws-sdk/client-dynamodb');
const { DynamoDBDocumentClient, GetCommand } = require('@aws-sdk/lib-dynamodb');

const client = new DynamoDBClient({});
const dynamodb = DynamoDBDocumentClient.from(client);

const TABLE_NAME = process.env.PRODUCTS_TABLE;

exports.handler = async (event) => {
    const productId = event.pathParameters?.id;

    if (!productId) {
        return {
            statusCode: 400,
            headers: { 'Content-Type': 'application/json', 'Access-Control-Allow-Origin': '*' },
            body: JSON.stringify({ error: 'Missing id' })
        };
    }

    try {
        const { Item } = await dynamodb.send(new GetCommand({
            TableName: TABLE_NAME,
            Key: { productId }
        }));

        if (!Item) {
            return {
                statusCode: 404,
                headers: { 'Content-Type': 'application/json', 'Access-Control-Allow-Origin': '*' },
                body: JSON.stringify({ error: 'Not found' })
            };
        }

        return {
            statusCode: 200,
            headers: { 'Content-Type': 'application/json', 'Access-Control-Allow-Origin': '*' },
            body: JSON.stringify(Item)
        };
    } catch (error) {
        console.error(error);
        return {
            statusCode: 500,
            headers: { 'Content-Type': 'application/json', 'Access-Control-Allow-Origin': '*' },
            body: JSON.stringify({ error: 'Get failed', message: error.message })
        };
    }
};

```

---

## LIST PRODUCTS
Endpoint: GET /products

```js
/**
 * LIST ITEMS FUNCTION
 */

const { DynamoDBClient } = require('@aws-sdk/client-dynamodb');
const { DynamoDBDocumentClient, ScanCommand } = require('@aws-sdk/lib-dynamodb');

const client = new DynamoDBClient({});
const dynamodb = DynamoDBDocumentClient.from(client);

const TABLE_NAME = process.env.PRODUCTS_TABLE;

exports.handler = async (event) => {
    try {
        const limit = event.queryStringParameters?.limit
            ? Number(event.queryStringParameters.limit)
            : 20;

        const result = await dynamodb.send(new ScanCommand({
            TableName: TABLE_NAME,
            Limit: limit
        }));

        return {
            statusCode: 200,
            headers: { 'Content-Type': 'application/json', 'Access-Control-Allow-Origin': '*' },
            body: JSON.stringify({
                items: result.Items ?? [],
                nextKey: result.LastEvaluatedKey ?? null
            })
        };
    } catch (error) {
        console.error(error);
        return {
            statusCode: 500,
            headers: { 'Content-Type': 'application/json', 'Access-Control-Allow-Origin': '*' },
            body: JSON.stringify({ error: 'List failed', message: error.message })
        };
    }
};

```

---

## UPDATE PRODUCT
Endpoint: PUT /products/{id}

```js
/**
 * UPDATE ITEM FUNCTION
 */

const { DynamoDBClient } = require('@aws-sdk/client-dynamodb');
const { DynamoDBDocumentClient, UpdateCommand } = require('@aws-sdk/lib-dynamodb');

const client = new DynamoDBClient({});
const dynamodb = DynamoDBDocumentClient.from(client);

const TABLE_NAME = process.env.PRODUCTS_TABLE;

exports.handler = async (event) => {
    const productId = event.pathParameters?.id;

    if (!productId) {
        return {
            statusCode: 400,
            headers: { 'Content-Type': 'application/json', 'Access-Control-Allow-Origin': '*' },
            body: JSON.stringify({ error: 'Missing id' })
        };
    }

    try {
        const body = JSON.parse(event.body || '{}');

        const fields = ['name', 'description', 'price', 'category', 'imageUrl'];
        const sets = [];
        const names = {};
        const values = {};

        for (const field of fields) {
            if (body[field] !== undefined) {
                sets.push(`#${field} = :${field}`);
                names[`#${field}`] = field;
                values[`:${field}`] = field === 'price' ? Number(body[field]) : body[field];
            }
        }

        sets.push('#updatedAt = :updatedAt');
        names['#updatedAt'] = 'updatedAt';
        values[':updatedAt'] = new Date().toISOString();

        const result = await dynamodb.send(new UpdateCommand({
            TableName: TABLE_NAME,
            Key: { productId },
            UpdateExpression: `SET ${sets.join(', ')}`,
            ExpressionAttributeNames: names,
            ExpressionAttributeValues: values,
            ConditionExpression: 'attribute_exists(productId)',
            ReturnValues: 'ALL_NEW'
        }));

        return {
            statusCode: 200,
            headers: { 'Content-Type': 'application/json', 'Access-Control-Allow-Origin': '*' },
            body: JSON.stringify(result.Attributes)
        };
    } catch (error) {
        console.error(error);
        return {
            statusCode: 500,
            headers: { 'Content-Type': 'application/json', 'Access-Control-Allow-Origin': '*' },
            body: JSON.stringify({ error: 'Update failed', message: error.message })
        };
    }
};

```

---

## DELETE PRODUCT
Endpoint: DELETE /products/{id}

```js
/**
 * DELETE ITEM FUNCTION
 */

const { DynamoDBClient } = require('@aws-sdk/client-dynamodb');
const { DynamoDBDocumentClient, DeleteCommand } = require('@aws-sdk/lib-dynamodb');

const client = new DynamoDBClient({});
const dynamodb = DynamoDBDocumentClient.from(client);

const TABLE_NAME = process.env.PRODUCTS_TABLE;

exports.handler = async (event) => {
    const productId = event.pathParameters?.id;

    if (!productId) {
        return {
            statusCode: 400,
            headers: { 'Content-Type': 'application/json', 'Access-Control-Allow-Origin': '*' },
            body: JSON.stringify({ error: 'Missing id' })
        };
    }

    try {
        await dynamodb.send(new DeleteCommand({
            TableName: TABLE_NAME,
            Key: { productId },
            ConditionExpression: 'attribute_exists(productId)'
        }));

        return {
            statusCode: 200,
            headers: { 'Content-Type': 'application/json', 'Access-Control-Allow-Origin': '*' },
            body: JSON.stringify({ message: 'Deleted', productId })
        };
    } catch (error) {
        console.error(error);
        return {
            statusCode: 500,
            headers: { 'Content-Type': 'application/json', 'Access-Control-Allow-Origin': '*' },
            body: JSON.stringify({ error: 'Delete failed', message: error.message })
        };
    }
};

```

---

Notes:
- Full CRUD over DynamoDB
- Lambda + API Gateway ready
- Production-safe patterns

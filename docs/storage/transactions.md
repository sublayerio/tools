# Creating transactions

Transactions facilitate converting an action into a set of operations that can be translated into mutations for a specific database, be it a mysql database or a single static file. Currently `createTransaction` supports three kinds of actions:
- create
- update
- remove

## create

This action will create two operations:
- an insert operation that contains an entity created based on the schema, hooks and the data provided to the action.
- an insert operation that contains an event entity describing the create action.
```js
const createTransaction = require('@sublayer/tools/transaction/createTransaction')

const schema = {
    TableDatas: {
        Person: {
            id: "Person",
            fields: [
                {
                    id: "firstName",
                    type: "text"
                },
                {
                    id: "lastName",
                    type: "text"
                },
                {
                    id: "name",
                    type: "text"
                }
            ]
        }
    },
    TableDatas: [
        "Person"
    ]
}

const result = createTransaction(ctx)({
    type: "create",
    tableId: "Person",
    data: {
        firstName: "Luke",
        lastName: "Skywalker"
    }
})
```

Result:
```js
{
    operations: [
        {
            type: "insert",
            payload: {
                tableId: "Person",
                recordId: "0106c862-db09-4060-97fb-8716997b0c21",
                entity: {
                    id: "0106c862-db09-4060-97fb-8716997b0c21",
                    firstName: "Luke",
                    lastName: "Skywalker",
                    name: "Luke Skywalker"
                }
            } 
        },
        {
            type: "insert",
            payload: {
                tableId: "Event",
                recordId: "86910eef-ddad-4147-9ccd-7878b5341a0a",
                entity: {
                    id: "86910eef-ddad-4147-9ccd-7878b5341a0a",
                    type: "Person:created",
                    tableId: "Person",
                    recordId: "0106c862-db09-4060-97fb-8716997b0c21",
                    payload: {
                        tableId: "Person",
                        recordId: "0106c862-db09-4060-97fb-8716997b0c21",
                        entity: {
                            id: "0106c862-db09-4060-97fb-8716997b0c21",
                            firstName: "Luke",
                            lastName: "Skywalker",
                            name: "Luke Skywalker"
                        }
                    }
                }
            }
        }
    ]
}
```

## update
This action will create two (or more based on the fields that have been changed) operations:
- an insert operation that contains the updated entity based on the schema, hooks and the data provided to the action.
- an insert operation that contains an event entity describing the update action and each field that has been changed.
- an insert operation for each field that has been updated that contains an event entity describing the previous and current value of the field.

```js
const createTransaction = require('@sublayer/tools/transaction/createTransaction')

const schema = {
    TableDatas: {
        Person: {
            id: "Person",
            fields: [
                {
                    id: "firstName",
                    type: "text"
                },
                {
                    id: "lastName",
                    type: "text"
                },
                {
                    id: "name",
                    type: "text"
                }
            ]
        }
    },
    Table: [
        "Person"
    ]
}

const result = createTransaction(ctx)({
    type: "update",
    tableId: "Person",
    data: {
        id: "0106c862-db09-4060-97fb-8716997b0c21",
        firstName: "Anakin"
    }
})
```

Result:
```js
{
    operations: [
        {
            type: "update",
            payload: {
                tableId: "Person",
                recordId: "0106c862-db09-4060-97fb-8716997b0c21",
                entity: {
                    firstName: "Anakin"
                }
            } 
        },
        {
            type: "insert",
            payload: {
                tableId: "Event",
                recordId: "86910eef-ddad-4147-9ccd-7878b5341a0a",
                entity: {
                    id: "86910eef-ddad-4147-9ccd-7878b5341a0a",
                    type: "Person:firstName:changed",
                    tableId: "Person",
                    recordId: "0106c862-db09-4060-97fb-8716997b0c21",
                    payload: {
                        tableId: "Person",
                        recordId: "0106c862-db09-4060-97fb-8716997b0c21",
                        entity: {
                            id: "0106c862-db09-4060-97fb-8716997b0c21",
                            firstName: "Anakin",
                            lastName: "Skywalker",
                            name: "Anakin Skywalker"
                        },
                        field: "firstName",
                        previousValue: "Luke",
                        value: "Anakin"
                    }
                }
            }
        },
        {
            type: "insert",
            payload: {
                tableId: "Event",
                recordId: "86910eef-ddad-4147-9ccd-7878b5341a0a",
                entity: {
                    id: "86910eef-ddad-4147-9ccd-7878b5341a0a",
                    type: "Person:updated",
                    tableId: "Person",
                    recordId: "0106c862-db09-4060-97fb-8716997b0c21",
                    payload: {
                        tableId: "Person",
                        recordId: "0106c862-db09-4060-97fb-8716997b0c21",
                        previousEntity: {
                            id: "0106c862-db09-4060-97fb-8716997b0c21",
                            firstName: "Luke",
                            lastName: "Skywalker",
                            name: "Luke Skywalker"
                        },
                        entity: {
                            id: "0106c862-db09-4060-97fb-8716997b0c21",
                            firstName: "Anakin",
                            lastName: "Skywalker",
                            name: "Anakin Skywalker"
                        },
                        changes: [
                            {
                                field: "firstName",
                                previousValue: "Luke",
                                value: "Anakin"
                            }
                        ]
                    }
                }
            }
        }
    ]
}
```

## remove
This action will create two operations:
- a remove operation that contains the removed entity based.
- an insert operation that contains an event entity indicating the removal of the entity alongside the entity that has been removed.

```js
const createTransaction = require('@sublayer/tools/transaction/createTransaction')

const schema = {
    TableDatas: {
        Person: {
            id: "Person",
            fields: [
                {
                    id: "firstName",
                    type: "text"
                },
                {
                    id: "lastName",
                    type: "text"
                },
                {
                    id: "name",
                    type: "text"
                }
            ]
        }
    },
    Table: [
        "Person"
    ]
}

const result = createTransaction(ctx)({
    type: "remove",
    tableId: "Person",
    data: {
        id: "0106c862-db09-4060-97fb-8716997b0c21",
        firstName: "Anakin"
    }
})
```

Result:
```js
{
    operations: [
        {
            type: "remove",
            payload: {
                tableId: "Person",
                recordId: "0106c862-db09-4060-97fb-8716997b0c21"
            } 
        },
        {
            type: "insert",
            payload: {
                tableId: "Event",
                recordId: "86910eef-ddad-4147-9ccd-7878b5341a0a",
                entity: {
                    id: "86910eef-ddad-4147-9ccd-7878b5341a0a",
                    type: "Person:removed",
                    tableId: "Person",
                    recordId: "0106c862-db09-4060-97fb-8716997b0c21",
                    payload: {
                        tableId: "Person",
                        recordId: "0106c862-db09-4060-97fb-8716997b0c21",
                        entity: {
                            id: "0106c862-db09-4060-97fb-8716997b0c21",
                            firstName: "Luke",
                            lastName: "Skywalker",
                            name: "Luke Skywalker"
                        }
                    }
                }
            }
        }
    ]
}
```